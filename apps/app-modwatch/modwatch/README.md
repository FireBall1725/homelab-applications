# modwatch

[Modwatch](https://github.com/fireball1725/modwatch) polls CurseForge and Modrinth once an hour for the download counts of my mods, keeps a snapshot of each, and shows them on a dashboard. The Grafana dashboard is `grafana-dashboard-modwatch` in `app-monitoring/kube-prometheus-stack/templates/`.

- UI: http://modwatch.k8s.firekatt.ca/ (internal ingress, no login)
- Image: `ghcr.io/fireball1725/modwatch`, pinned to an immutable tag in `values.yaml` and `appVersion`
- Data: a CloudNativePG cluster, `modwatch-db` (2 Gi Longhorn). CurseForge only reports lifetime totals, so the history exists nowhere else.
- Metrics: a ServiceMonitor scrapes `/metrics`; `ModwatchPollFailing`, `ModwatchSourceDown` and `ModwatchPollsStale` alert when polls fail, a source can't be read, or polls stop

## CurseForge key

Polling needs a Core API key from [console.curseforge.com](https://console.curseforge.com). The tokens on the older "API Tokens" page are upload tokens and get a 403. Store the key as a SealedSecret named `modwatch-curseforge` with the key `CURSEFORGE_API_KEY`:

```sh
kubectl create secret generic modwatch-curseforge -n app-modwatch \
  --from-literal=CURSEFORGE_API_KEY=... --dry-run=client -o yaml \
  | kubeseal -o yaml > templates/sealed-secret-curseforge.yaml
```

Until it exists the pod still starts, and the UI lists the missing key.
