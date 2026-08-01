# bambuddy

> Self-hosted control plane for Bambu Lab printers

## Overview

Bambuddy archives every print as a 3MF with its metadata, thumbnails, and cost, runs a multi-printer queue, tracks filament spools and AMS slots, and streams the chamber camera. It talks to printers over the LAN using Developer Mode credentials, so no print data goes through Bambu's cloud. This chart deploys the app, a CloudNative-PG managed PostgreSQL cluster, and the OrcaSlicer sidecar that backs the server-side Slice button.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [bambuddy](https://github.com/maziggy/bambuddy) ([docs](https://wiki.bambuddy.cool)) |
| **Image** | `ghcr.io/maziggy/bambuddy:1.2.5.1` (public, multi-arch amd64/arm64) |
| **Sidecar Image** | `ghcr.io/maziggy/orca-slicer-api:bambuddy-1.2.5.1` (linux/amd64 only) |
| **Chart Version** | `0.1.1` |
| **App Version** | `1.2.5.1` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

Image tags carry no leading `v` even though the upstream git tags do: git `v1.2.5.1` publishes as image `1.2.5.1`.

## Why this does not use hostNetwork

Upstream's `docker-compose.yml` sets `network_mode: host` and the docs call it required. It is required for two things only: SSDP auto-discovery and the virtual printer's SSDP announcements. Everything else is an outbound connection from Bambuddy to the printer, and those route fine from a pod to `10.0.0.0/24`:

| Path | Port | Direction |
|---|---|---|
| Printer status and control | 8883 TCP (MQTT over TLS) | outbound |
| File transfer | 990 TCP (FTPS) | outbound |
| Camera, X1/H2/P2 | 322 TCP (RTSP) | outbound |
| Camera, A1/P1 | 6000 TCP (`chamber_image`) | outbound |

The cost is the "discover printers" button. Add printers by IP instead.

## Adding a printer

Bambuddy needs three values per printer, all entered in the UI:

1. **IP address.** Set a DHCP reservation first so it does not move.
2. **Serial number.** X1 series starts `01S` or `01P`, P1 `01P`, A1 `01A`, H2 `03W`.
3. **Access code.** Eight characters. On the printer: Settings, Network, LAN Only Mode, then Developer Mode appears and shows the code.

In the slicer, enable "Store sent files on external storage" so Bambuddy can archive what gets sent.

## Ingress

`bambuddy.k8s.firekatt.ca` on ingressClass `internal`. No per-host TLS block; Traefik terminates with the cluster-wide `*.k8s.firekatt.ca` wildcard cert. The zone resolves inside the LAN only.

The Uptime Kuma autodiscovery annotation points at `/health` rather than `/`, because the root redirects to the login page once authentication is switched on.

## Virtual printer

Enabled, on its own MetalLB address `10.0.1.102` from `service-pool` (10.0.1.101-119, where mosquitto holds .101). `templates/service-virtual-printer.yaml` publishes 37 ports: 322, 990, 2021/udp, 3000, 3002, 6000, 8883, and the FTP passive range 50000-50029. Thirty passive ports is three virtual printers at ten each; widen `virtualPrinter.passivePortCount` to add more.

Two consequences of using a LoadBalancer VIP instead of hostNetwork:

- `VIRTUAL_PRINTER_PASV_ADDRESS` is set to `10.0.1.102`. Without it, FTPS passive mode advertises the pod IP and transfers stall after the control channel connects.
- SSDP announcements leave with the pod's source address, not the VIP, so Bambu Studio and OrcaSlicer will not auto-discover it. Add it manually at `10.0.1.102`.

Set `virtualPrinter.enabled: false` to drop the Service and the 37 NodePort allocations that come with it.

## Slicer sidecar

`orca-slicer-api` runs in the same pod on port **3003**, not its default 3000, because the virtual printer binds 3000 and 3002 in the same network namespace. `SLICER_API_URL` points at `http://127.0.0.1:3003`.

The image publishes linux/amd64 only, so the pod carries `nodeSelector: kubernetes.io/arch=amd64`. `talos-piwk-01` is the one arm64 node in this cluster.

The second slicer backend, `bambu-studio-api`, is not deployed. Add it as another entry under `additionalContainers` with `BAMBU_STUDIO_API_URL` if a Bambu Studio preset bundle needs it.

## Persistence

| Volume | Type | Size | Storage Class | Notes |
|--------|------|------|---------------|-------|
| `bambuddy-data` | pvc | 100Gi | `longhorn` | `/app/data`: archive 3MFs, thumbnails, timelapses, library files, `backups/` (RWO) |
| `logs` | emptyDir | 1Gi | | `/app/logs`, ephemeral |
| `slicer` | emptyDir | 20Gi | | sidecar scratch, mounted at `/app/data` in the sidecar only |
| `bambuddy-db` (CNPG) | pvc | 10Gi | `longhorn` | PostgreSQL 17.5-3 |

Timelapse video drives the growth. Upstream describes 10+ GB archives as normal, which is why the data volume starts at 100Gi. The `slicer` volume uses `mountPath: "-"` so the common library skips it in the main container; the sidecar mounts it explicitly.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents | Source |
|--------|----------|--------|
| `bambuddy-db-creds` | `username`, `password`, type `kubernetes.io/basic-auth` | SealedSecret (`templates/sealed-secret-db-creds.yaml`) |
| `bambuddy-database-url` | `DATABASE_URL` | SealedSecret (`templates/sealed-secret-database-url.yaml`) |

Both hold the same password. This is the one place the chart diverges from `firebin-api`, which reads `DATABASE_URL` straight out of the CNPG-generated `firebin-db-app` secret. Two reasons that does not work here:

1. Bambuddy runs SQLAlchemy with asyncpg and needs the `postgresql+asyncpg://` scheme prefix. CNPG's `uri` key is plain `postgresql://`, and a Kubernetes env var cannot concatenate a prefix onto a secret value.
2. Assembling the URL from separate `username` and `password` env vars would work, but a CNPG-generated password can contain symbols that need percent-encoding inside a URL. That fails at connection time with no useful error.

So the chart supplies the credentials instead. `postgres.ownerSecret` points `bootstrap.initdb.secret.name` at `bambuddy-db-creds`, and CNPG does not generate a `bambuddy-db-app` secret at all.

To rotate, regenerate both together with the same password:

```bash
PW=$(LC_ALL=C tr -dc 'A-Za-z0-9' < /dev/urandom | head -c 32)

kubectl create secret generic bambuddy-db-creds \
  --namespace=app-bambuddy --type=kubernetes.io/basic-auth \
  --from-literal=username=bambuddy \
  --from-literal=password="$PW" \
  --dry-run=client -o yaml | \
  kubeseal --controller-namespace kube-system --format yaml \
  > bambuddy/templates/sealed-secret-db-creds.yaml

kubectl create secret generic bambuddy-database-url \
  --namespace=app-bambuddy \
  --from-literal=DATABASE_URL="postgresql+asyncpg://bambuddy:${PW}@bambuddy-db-rw:5432/bambuddy" \
  --dry-run=client -o yaml | \
  kubeseal --controller-namespace kube-system --format yaml \
  > bambuddy/templates/sealed-secret-database-url.yaml
```

Keep the password alphanumeric. Rotating the CNPG side alone will not change the password on an already-bootstrapped cluster; `bootstrap.initdb` runs once.

## Authentication

Off at deploy time. The first visit lands on a setup page, and the first account created goes into the Administrators group. Group permissions, TOTP, email OTP, and OIDC are all configured in the UI afterwards.

One OIDC provider can be driven entirely from `BAMBUDDY_OIDC_*` env vars if that becomes preferable to the built-in login. That would need a third sealed secret for `BAMBUDDY_OIDC_CLIENT_SECRET`.

## Metrics

`templates/servicemonitor.yaml` scrapes `/api/v1/metrics` every 30s. `kube-prometheus-stack-prometheus` runs with empty `serviceMonitorSelector` and `serviceMonitorNamespaceSelector`, so no release label is needed.

The endpoint is a UI toggle under Settings, Network, Prometheus Metrics, not an env var. The target reads as down until that is switched on. While off, `/api/v1/metrics` answers 404 with `{"detail":"Prometheus metrics not enabled"}`, which confirms the path is right and only the switch is missing. Exported series cover connection state, print progress, bed and nozzle and chamber temperatures, fan speeds, WiFi signal, filament grams, and queue depth, all labelled by printer id, name, serial, and model.

The CNPG cluster exports separately through `monitoring.enablePodMonitor`, matching `firebin-db` and `librarium-db`.

## Notes

- PostgreSQL is provisioned via a `Cluster` CR managed by [CloudNative-PG](../cloudnative-pg/). Bambuddy creates its own tables on first start, so there is no migration job.
- Liveness and readiness probes hit `GET /health`, which is unauthenticated and returns `{"status":"healthy"}`. The wiki documents this as `/api/v1/health`, which 404s on 1.2.5.1; `/openapi.json` from the running container gives `/health` with `security: null`. `/api/v1/system/health` also exists but sits behind HTTPBearer.
- Do not substitute `/healthz` or `/metrics` as a probe target. Both answer HTTP 200, but they are the SPA catch-all serving `index.html`, so they stay green with a dead backend.
- `securityContext.capabilities.add: [NET_BIND_SERVICE]` is what lets the virtual printer bind 322 and 990. That capability sits on the Pod Security Admission baseline allowlist, so `app-bambuddy` needs no privileged PSA labels.
- `podSecurityContext.fsGroup: 1000` matches the `PUID`/`PGID` the container drops to after startup, so the Longhorn volume stays writable.
- Camera streams are transcoded to MJPEG by ffmpeg inside the container. If a stream connects but stalls in the browser, suspect the reverse proxy buffering the MJPEG response; upstream's documented workaround is relaying through go2rtc.
- Backups are configured in the UI and land in `/app/data/backups`, inside the same PVC. They export to portable SQLite regardless of the live backend, so a restore can move between SQLite and PostgreSQL.
