# ssh-to-go

Browser terminal session manager. The server holds an ssh key, dials out to the hosts in
its config, runs `tmux list-sessions` against each one every 5 seconds, and relays the
panes to xterm.js over a websocket at `/ws/{host}/{session}`.

Upstream: [awkto/ssh-to-go](https://github.com/awkto/ssh-to-go), AGPL-3.0, tracking v1.29.0.

This is an ssh **client**. There is no sshd in the container, nothing listens on 22, and
the chart needs no LoadBalancer. One HTTP port on 8080 behind the usual Traefik ingress.

| | |
|---|---|
| Namespace | `app-ssh-to-go` |
| URL | https://ssh-to-go.k8s.firekatt.ca |
| In-cluster | `http://ssh-to-go.app-ssh-to-go:8080` |
| Image | `docker.io/awkto/ssh-to-go:v1.29.0` (Docker Hub only, no GHCR mirror) |
| Storage | two Longhorn RWO PVCs, 1Gi at `/data` and 128Mi at `/etc/ssh-to-go` |
| Database | none |

## Why an init container writes the config file

`internal/config/config.go` `Load()` falls back to `listen_addr: "127.0.0.1:8080"` when the
config file is missing, and relocates `data_dir` to `<configDir>/data` at the same time.

The image bakes `/etc/ssh-to-go/config.yaml` in, but this chart mounts a PVC over that
path, which masks it. Without the `seed-config` init container, first boot gives you a pod
that pulls, starts, passes admission, reports Running, binds loopback only, and cannot be
reached from its own Service. Its keys and `auth.json` land in `/etc/ssh-to-go/data`
instead of the `/data` volume, so the storage you provisioned sits empty while the state
you care about rides on the wrong claim.

Both behaviours are confirmed against the upstream parser, not inferred:

```
NO CONFIG FILE -> ListenAddr="127.0.0.1:8080" DataDir="/tmp/sstest-empty/data"
seeded config  -> ListenAddr="0.0.0.0:8080"   DataDir="/data"  PollInterval=5s
```

The seed script tests `[ -s ... ]` before writing, so a rollout never overwrites hosts you
added through the UI. `AppendHost` re-reads and rewrites the whole file, preserving
`listen_addr` and `data_dir`, which was also verified against the upstream code.

## Why the config directory is a PVC and not a ConfigMap

Adding, editing, or deleting a host in the web UI calls `config.AppendHost`,
`config.UpdateHost`, or `config.RemoveHost`, and each one rewrites `config.yaml` in place.
A ConfigMap mount is read-only, so every host button in the UI would fail.

The trade-off is real: the host list lives on a Longhorn volume, not in git. If you'd
rather manage hosts through pull requests, swap `persistence.config` for a ConfigMap and
accept that the UI can only read.

## Why it runs as root

The runtime image is `alpine:3.21` with no `USER` directive, so uid 0 is the mode upstream
ships and tests. The app creates `/data/keys` at 0700, writes ed25519 private keys at 0600,
then execs the openssh client against them; ssh refuses a key it considers group-readable
and wants a real home directory for its own bookkeeping.

Pinning `runAsUser` plus `fsGroup` would trade a known-good state for an uncertain one on a
container that already holds the private keys to every host it manages. The uid isn't the
security boundary here. The ingress class is.

## Why it isn't backed up

Everything on both volumes is reproducible in about two minutes: run the setup wizard
again, generate a keypair, re-add the hosts. `auth.json`, `settings.json`,
`session-registry.json`, `session-ids.json`, `session-icons.json`, and
`recent-commands.json` are all small and all disposable.

Separately, `daily-backup-r2` currently carries `groups: []`, so the `default` group label
selects nothing and 32 of 47 volumes in this cluster get no backup either way. Only the 15
with an explicit `recurring-job.longhorn.io/daily-backup-r2: enabled` label are covered.

## What you have to do by hand

The chart can't do these, and no host works until they're done.

1. Open https://ssh-to-go.k8s.firekatt.ca/setup and set the admin password. It's bcrypt
   hashed at cost 12 into `/data/auth.json`, and there's no recovery path short of deleting
   that file.
2. Generate an ed25519 keypair in the UI, then append the public half to
   `~/.ssh/authorized_keys` on every host you want to manage.
3. Install `tmux` on those hosts. The session list comes from `tmux list-sessions`, so a
   host without tmux connects fine and shows an empty list, which reads as a networking
   fault when it isn't one.
4. Add each host in the UI.

## Two behaviours to know before relying on it

Browser sessions live in an in-memory map and are never written to disk, so every pod
restart logs out every browser. Renovate bumps this image often; upstream went from v1.15.3
to v1.29.0 in about two weeks.

Authentication is one shared admin password with no OIDC, no LDAP, and no multi-user, sat
in front of an exec API that runs arbitrary shell on your hosts. The `internal` ingress
class keeps it on the LAN, and that's the only reason this is acceptable. Don't move it to
the `tailscale` class or expose it outward without an auth proxy in front.

`SSH_TO_GO_NO_AUTH=1` disables authentication entirely. It is deliberately not set.

## Verifying a deploy

```bash
kubectl -n app-ssh-to-go get pods,pvc,svc,ingress
kubectl -n app-ssh-to-go exec deploy/ssh-to-go -- cat /etc/ssh-to-go/config.yaml
```

`listen_addr` must read `0.0.0.0:8080`. If it says `127.0.0.1:8080`, the seed init
container didn't run and the Service can't reach the process even though the pod is
Running.

```bash
kubectl -n app-ssh-to-go run curl --rm -it --image=curlimages/curl --restart=Never -- \
  -s -o /dev/null -w '%{http_code}\n' http://ssh-to-go:8080/login
```

`/login` is public and returns 200 once the setup wizard has run. `/api/version` sits
behind auth, and there is no `/health` route, which is why the probes are TCP.

## Architecture pin

Every published tag is `linux/amd64` only. The manifest index carries one platform entry
plus an attestation stub, and upstream only builds arm64 on a manual `workflow_dispatch`
with `multiarch: true`, which no released tag has had. `talos-piwk-01` is arm64, so
`nodeSelector: kubernetes.io/arch: amd64` keeps the pod off it.
