# Homelab Applications

Every Helm chart & raw manifest running in the home Kubernetes cluster, plus the
single ApplicationSet that turns them into ArgoCD Applications. 58 apps across 36
namespaces, one Talos cluster.

## How an app becomes an Application

The path is the configuration. `apps/<namespace>/<name>/` gives ArgoCD everything
it needs: the second path segment is the destination namespace, the third is both
the Application name & the Helm release name.

```
apps/app-media/sonarr/          -> Application "sonarr" in namespace "app-media"
apps/kube-system/headlamp/      -> Application "headlamp" in namespace "kube-system"
apps/app-firebin/firebin-api/   -> Application "firebin-api" in namespace "app-firebin"
```

`appsets/apps.yaml` holds the ApplicationSet that does this. It runs a git
*directory* generator over `apps/*/*`, so adding a directory adds an app & there's
no second file to keep in sync. A directory generator rather than a files
generator on `Chart.yaml`, because `cloudflare-ddns` & `unpoller` are raw manifest
directories with no chart at all.

Two rules on that ApplicationSet are load-bearing:

`applicationsSync: create-update` lets the controller create & update Applications
but never delete one. `preserveResourcesOnDeletion: true` makes it strip
`resources-finalizer.argocd.argoproj.io` from every Application it generates. Both
exist because no PVC in this repo sets `helm.sh/resource-policy: keep`, so before
this a deleted Application took its PersistentVolumes with it. Deleting a
directory now leaves the Application running until you remove it yourself:

```bash
kubectl delete application <name> -n core-argocd
```

`cloudnative-pg` is the one app with a per-app override. Its CRDs exceed the
262,144-byte annotation limit that client-side apply relies on, so a second
generator matches only that directory & sets `ServerSideApply=true`.

### Namespace conventions

| Pattern | Used for | Example |
|---|---|---|
| `app-<name>` | ArgoCD-managed apps | `app-wikijs`, `app-media` |
| `core-<name>` | Tofu-managed infrastructure | `core-argocd`, `core-longhorn` |
| Standard names | System components | `cert-manager`, `kube-system` |

A namespace can hold several apps. `app-media` holds ten, `app-ai` six,
`app-firebin` four.

## Where the rest of it lives

This repo holds charts & the ApplicationSet. It does not hold the ArgoCD install.
The `argo-cd` Helm release, the `homelab-gitops` AppProject & `Application/app-bootstrap`
are OpenTofu resources in `~/Documents/terraform/talos-cluster/core-argocd.tf`,
with local state. `app-bootstrap` points at `appsets/` in this repo & is what
applies the ApplicationSet. If you need to change the bootstrap Application,
that's the file, & use `-target` so a plan doesn't drag in the whole cluster.

Until 2026-08-13 there was a second repo, `homelab-config`, holding one
`app-config.json` per app. Its only real content was the destination namespace,
which the path now carries. Two of its four keys did nothing: `replace` was
`false` in all 57 files, & `createNamespace` was read through
`{{ or .createNamespace true }}`, which returns `true` for `or false true`, so the
five apps that set it to `false` had been getting `CreateNamespace=true` for
years.

## Directory layout

```
apps/<namespace>/<name>/
├── Chart.yaml          # chart metadata & upstream dependency
├── Chart.lock          # dependency lock (33 of 55 charts have one)
├── values.yaml         # configuration overrides
├── templates/          # extra resources: sealed secrets, CNPG clusters, ingresses
├── charts/             # vendored dependency tarballs, tracked in git
└── README.md           # app notes

appsets/apps.yaml       # the ApplicationSet
```

`charts/*.tgz` is committed, not ignored. 46 tarballs are tracked. Be aware that
11 of them are older than what their `Chart.yaml` declares: `traefik` vendors
35.4.0 against a declared 39.0.7, `homarr` vendors 5.3.0 against 8.16.1. ArgoCD
re-resolves at sync time, so the committed tarball is what a local `helm template`
uses & what a reviewer reads, & the two disagree. `cribl-edge` declares a floating
`^4.12.1`, which is the only chart here whose render isn't reproducible.

## Common library

Most apps wrap the [FireLabs Helm Common Library](https://github.com/FireBall1725/firelabs-helm-common),
a fork of the k8s-at-home common library, & put everything in `values.yaml`.

```yaml
# Chart.yaml
dependencies:
  - name: common
    version: 5.0.3
    repository: https://fireball1725.github.io/firelabs-helm-common/
```

```yaml
# templates/common.yaml
{{ include "common.all" . }}
```

`audiobookshelf` & `lazylibrarian` still point at the old k8s-at-home repo at
4.5.2.

Resource names come from the release name, which is the directory name. 12 charts
pin `global.fullnameOverride` so their names don't depend on it; the rest inherit.
Renaming a directory therefore renames every Deployment, Service **and PVC** the
chart owns, which orphans the volume. Don't rename a directory to move an app
between namespaces; that isn't what the layout is for.

## Applications

### Home automation
| App | Description |
|-----|-------------|
| [Home Assistant](apps/app-home-assistant/home-assistant/) | Smart home hub |
| [Node-RED](apps/app-node-red/node-red/) | Flow-based automation engine |
| [ESPHome](apps/app-esphome/esphome/) | ESP device firmware management |
| [Zigbee2MQTT](apps/app-zigbee2mqtt/zigbee2mqtt/) | Zigbee device bridge |
| [Mosquitto](apps/app-mosquitto/mosquitto/) | MQTT broker |
| [Matter Server](apps/app-matter-server/matter-server/) | Matter/Thread controller |
| [Music Assistant](apps/app-music-assistant/music-assistant/) | Multi-room audio server |
| [Homebridge](apps/app-homebridge/homebridge/) | HomeKit bridge |

### Monitoring
| App | Description |
|-----|-------------|
| [Kube Prometheus Stack](apps/app-monitoring/kube-prometheus-stack/) | Prometheus, Grafana, Alertmanager |
| [Uptime Kuma](apps/app-uptime-kuma/uptime-kuma/) | Uptime monitoring & status page |
| [Kuma Ingress Watcher](apps/app-uptime-kuma/kuma-ingress-watcher/) | Registers ingresses with Uptime Kuma |
| [Unpoller](apps/app-unpoller/unpoller/) | UniFi network metrics exporter |
| [Cribl Edge](apps/app-cribl-edge/cribl-edge/) | Log & metric collection agent |

### Media
| App | Description |
|-----|-------------|
| [Radarr](apps/app-media/radarr/) | Movie library management |
| [Sonarr](apps/app-media/sonarr/) | TV series management |
| [Lidarr](apps/app-media/lidarr/) | Music library management |
| [Prowlarr](apps/app-media/prowlarr/) | Indexer aggregator |
| [SABnzbd](apps/app-media/sabnzbd/) | Usenet downloader |
| [Pinchflat](apps/app-media/pinchflat/) | YouTube channel archival |
| [Sportarr](apps/app-media/sportarr/) | Sports event PVR |
| [Tautulli](apps/app-media/tautulli/) | Plex analytics |
| [Audiobookshelf](apps/app-media/audiobookshelf/) | Audiobook & podcast server |
| [LazyLibrarian](apps/app-media/lazylibrarian/) | Book library management |
| [Kometa](apps/app-kometa/kometa/) | Plex metadata & collections |

### Infrastructure
| App | Description |
|-----|-------------|
| [CloudNative-PG](apps/app-cloudnative-pg/cloudnative-pg/) | PostgreSQL operator |
| [Cert Manager](apps/cert-manager/cert-manager/) | TLS certificate automation |
| [Traefik](apps/core-traefik-ingress/traefik/) | Ingress controller |
| [Error Pages](apps/core-traefik-ingress/error-pages/) | Custom Traefik error pages |
| [Tailscale](apps/app-tailscale/tailscale/) | Mesh VPN |
| [Metrics Server](apps/kube-system/metrics-server/) | Kubernetes resource metrics |
| [Blocky](apps/app-blocky/blocky/) | DNS-based ad blocking |
| [Cloudflare DDNS](apps/app-cloudflare-ddns/cloudflare-ddns/) | Dynamic DNS updater |

### AI
| App | Description |
|-----|-------------|
| [Open WebUI](apps/app-ai/open-webui/) | Web UI for local LLMs |
| [Ollama](apps/app-ai/ollama/) | Bridge to the LLM host at 10.0.1.191 |
| [Ollama Admin](apps/app-ai/ollama-admin/) | Model management UI |
| [Openclaw](apps/app-openclaw/openclaw/) | AI agent gateway (Discord) |
| [ha-mcp](apps/app-ai/ha-mcp/) | Home Assistant MCP server |
| [Basic Memory](apps/app-ai/basic-memory/) | Persistent note store over MCP |
| [Basic Memory Viewer](apps/app-ai/basic-memory-viewer/) | Web UI for Basic Memory |

### FireBin & Librarium
| App | Description |
|-----|-------------|
| [FireBin API](apps/app-firebin/firebin-api/) | Electronics inventory backend |
| [FireBin Web](apps/app-firebin/firebin-web/) | Inventory frontend |
| [FireBin MCP](apps/app-firebin/firebin-mcp/) | Inventory MCP server |
| [FireBin KiCad](apps/app-firebin/firebin-kicad/) | KiCad HTTP library |
| [Librarium API](apps/app-librarium/librarium-api/) | Book catalogue backend |
| [Librarium Web](apps/app-librarium/librarium-web/) | Catalogue frontend |
| [Librarium MCP](apps/app-librarium/librarium-mcp/) | Catalogue MCP server |

### Dashboards & tools
| App | Description |
|-----|-------------|
| [Headlamp](apps/kube-system/headlamp/) | Kubernetes web UI |
| [Homarr](apps/app-homarr/homarr/) | Homelab dashboard |
| [Homer](apps/app-homer/homer/) | Static homepage |
| [Wiki.js](apps/app-wikijs/wikijs/) | Documentation wiki |
| [ttyd](apps/app-ttyd/ttyd/) | Web-based terminal |
| [ssh-to-go](apps/app-ssh-to-go/ssh-to-go/) | Browser SSH gateway |
| [Terminus](apps/app-terminus/terminus/) | Terminal workspace |
| [n8n](apps/app-n8n/n8n/) | Workflow automation |
| [Spoolman](apps/app-spoolman/spoolman/) | Filament inventory |
| [Bambuddy](apps/app-bambuddy/bambuddy/) | Bambu printer bridge |
| [PC Express MCP](apps/app-pcexpress/pcexpress-mcp/) | Grocery ordering MCP server |
| [Renovate](apps/app-renovate/renovate/) | Dependency update bot |

## Adding an application

Create the directory under the namespace it belongs in. That's the whole
registration step; there's no second repo & no config file.

```bash
mkdir -p apps/app-<name>/<name>
```

`Chart.yaml`:

```yaml
apiVersion: v2
name: <name>
type: application
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: <chart-name>
    version: <chart-version>
    repository: https://<chart-repo-url>/
```

`values.yaml`:

```yaml
<chart-name>:
  env:
    TZ: "America/Toronto"
  persistence:
    config:
      enabled: true
      storageClassName: "longhorn"
      size: 10Gi
  ingress:
    enabled: true
    hosts:
      - host: <name>.k8s.firekatt.ca
        paths:
          - path: /
            pathType: Prefix
```

Then `helm dependency update`, commit, push. The ApplicationSet picks it up on
the next refresh.

## Secrets

Sealed Secrets, so encrypted material can sit in a public repo & decrypt only
inside the cluster. The controller is `sealed-secrets-controller` in `kube-system`.

```bash
kubectl create secret generic <name> \
  --namespace=<namespace> \
  --from-literal=<key>=<value> \
  --dry-run=client -o yaml | \
  kubeseal --controller-name sealed-secrets-controller \
           --controller-namespace kube-system \
           -o yaml > apps/<namespace>/<name>/templates/sealed-secret.yaml
```

Seal secrets, don't generate them. ArgoCD caches rendered manifests per
`(revision, path)`, so a template using `randAlphaNum` mints a new value on every
commit to this repo while the running pods keep whatever they started with.
`terminus` did exactly that for 94 syncs before it was sealed on 2026-08-13. Its
`APP_SECRET` is read through `secretKeyRef` into an env var, which resolves at pod
start, so nothing broke until a pod restarted & logged everyone out. `lookup` is
not a way around this: the repo-server has no cluster access & returns empty.

See [SECRETS.md](SECRETS.md) for the full workflow.

## Cluster

| | |
|---|---|
| Platform | Talos Linux |
| Kubernetes | v1.33+ |
| ArgoCD | v3.0.3, namespace `core-argocd` |
| Storage | Longhorn, namespace `core-longhorn` |
| Ingress | Traefik v3 |
| Load balancer IP | 10.0.1.100 |

## Commands

```bash
# force a hard refresh on one app
kubectl -n core-argocd patch application <name> --type merge \
  -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'

# anything not healthy
kubectl get applications -n core-argocd --no-headers | grep -v 'Synced *Healthy'

# pods that aren't running
kubectl get pods -A --field-selector status.phase!=Running,status.phase!=Succeeded

# what the ApplicationSet would generate, before you merge
argocd appset generate appsets/apps.yaml -o yaml

# long-lived service account token
kubectl create token <service-account> -n <namespace> --duration=87600h
```

## Dependency management

```bash
helm repo add firelabs https://fireball1725.github.io/firelabs-helm-common/
helm repo update
helm dependency update apps/<namespace>/<name>
helm search repo <chart-name> --versions | head -10
```
