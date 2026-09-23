# upstream

[Upstream](https://github.com/fireball1725/upstream) scans this repo on a schedule, checks every pinned image tag and chart version for something newer, and opens bump PRs for the ones you pick.

- UI: http://upstream.k8s.firekatt.ca/ (internal ingress, no login)
- Image: `ghcr.io/fireball1725/upstream`, pinned to an immutable tag in `values.yaml` and `appVersion`
- Data: 1 Gi Longhorn PVC at `/data` (SQLite database and the repo clone)

## GitHub token

Reading release notes and opening PRs needs a fine-grained PAT scoped to `homelab-applications` only, with Contents and Pull requests set to read and write. Store it as a SealedSecret named `upstream-github` with the key `GITHUB_TOKEN`:

```sh
kubectl create secret generic upstream-github -n app-upstream \
  --from-literal=GITHUB_TOKEN=github_pat_... --dry-run=client -o yaml \
  | kubeseal -o yaml > templates/sealed-secret-github.yaml
```

Until it exists the pod still starts, and the UI lists the missing token.
