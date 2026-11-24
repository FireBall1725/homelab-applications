# lookout

[Lookout](https://github.com/fireball1725/lookout) polls GitHub every 10 minutes for the repos I maintain, grouped into projects, and shows releases, unreleased commits, open PRs and issues, CI, and the tag each app runs here. The Grafana dashboard is `grafana-dashboard-lookout` in `app-monitoring/kube-prometheus-stack/templates/`.

- UI: https://lookout.k8s.firekatt.ca/ (internal ingress, no login; anyone who reaches it can change every setting)
- Image: `ghcr.io/fireball1725/lookout`, pinned to an immutable tag in `values.yaml` and `appVersion`. Private package, pulled with the sealed `ghcr-pull` secret.
- Data: a CloudNativePG cluster, `lookout-db` (2 Gi Longhorn). It holds the current state of each repo, not a history.
- Metrics: a ServiceMonitor scrapes `/metrics` every 60s.
- Deployed tags: `LOOKOUT_GITOPS_REPO` points at this repo, so a release that never got pinned here shows up as undeployed.

## GitHub token

There isn't one in the chart. Paste a read-only fine-grained token on the Settings page; Lookout stores it in its database and polls nothing until it's there. It needs read-only Metadata, Contents, Issues, Pull requests, Actions and Commit statuses on every tracked repo and on `homelab-applications`.

## Seed file

`configmap.seed` in `values.yaml` lists the six projects to start with, mounted at `/seed/lookout.seed.yaml`. Lookout reads it only while it has no projects, so after the first start the Settings page is where projects change. Editing the ConfigMap does nothing until the database is empty again.

## MCP token

`/mcp` takes a bearer token, sealed as `lookout-mcp` with the key `LOOKOUT_MCP_TOKEN`. The plain value is in the Keychain as `homelab/lookout-mcp-token`. To rotate it:

```sh
security add-generic-password -U -a "$USER" -s homelab/lookout-mcp-token -w "$(openssl rand -hex 32)"
kubectl create secret generic lookout-mcp -n app-lookout \
  --from-literal=LOOKOUT_MCP_TOKEN="$(security find-generic-password -a "$USER" -s homelab/lookout-mcp-token -w)" \
  --dry-run=client -o yaml | kubeseal -o yaml > templates/sealed-secret-mcp.yaml
```

A value set on the Settings page wins over the secret.
