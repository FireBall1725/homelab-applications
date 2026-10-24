# zot

OCI registry at `registry.k8s.firekatt.ca`. Anonymous pull, push as `fireball`
(password in the macOS Keychain, service `homelab/zot-registry-push`).

Keeps the 10 most recent tags per repo and garbage-collects the rest daily.
Build for `linux/amd64`; the laptop is arm64 and most nodes are not.

```bash
security find-generic-password -s homelab/zot-registry-push -w \
  | docker login registry.k8s.firekatt.ca -u fireball --password-stdin
docker buildx build --platform linux/amd64 -t registry.k8s.firekatt.ca/<name>:<tag> --push .
```
