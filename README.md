# Homelab Applications

This repository contains all Helm charts for applications running in my home Kubernetes cluster. Applications are deployed and managed using ArgoCD with a GitOps workflow.

## Architecture

This homelab uses a **two-repository pattern**:

| Repository | Purpose |
|---|---|
| **homelab-applications** (this repo) | Helm chart wrappers with configuration overrides |
| **homelab-config** | ArgoCD `Application` and `ApplicationSet` manifests |

ArgoCD monitors this repository and automatically syncs applications when changes are pushed. The `homelab-config` repo defines *which* apps to deploy and into *which* namespaces; this repo defines *how* each app is configured.

### Namespace Conventions

| Pattern | Used For | Example |
|---|---|---|
| `app-<name>` | ArgoCD-managed apps | `app-wikijs`, `app-traefik` |
| `core-<name>` | Tofu-managed infrastructure | `core-argocd`, `core-longhorn` |
| Standard names | System components | `cert-manager`, `kube-system` |

## Directory Structure

Each application has its own top-level folder containing a complete Helm chart:

```
<app-name>/
├── Chart.yaml          # Chart metadata and upstream dependency
├── Chart.lock          # Dependency lock file (generated)
├── values.yaml         # Configuration overrides
├── templates/          # Custom resource templates (optional)
├── charts/             # Downloaded dependencies (gitignored)
└── README.md           # App-specific documentation
```

## Common Library

Many applications use the **[FireLabs Helm Common Library](https://github.com/FireBall1725/firelabs-helm-common)** (`v5.0.3+`), a maintained fork of the k8s-at-home common library. It provides reusable templates for Deployments, Services, Ingresses, and persistence, reducing boilerplate across apps.

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

All configuration then lives in `values.yaml` using the common library schema.

## Applications

### Home Automation

| App | Description |
|-----|-------------|
| [Home Assistant](home-assistant/) | Smart home hub |
| [Node-RED](node-red/) | Flow-based automation engine |
| [ESPHome](esphome/) | ESP device firmware management |
| [Zigbee2MQTT](zigbee2mqtt/) | Zigbee device bridge |
| [Mosquitto](mosquitto/) | MQTT broker |
| [Music Assistant](music-assistant/) | Multi-room audio server |

### Monitoring & Observability

| App | Description |
|-----|-------------|
| [Kube Prometheus Stack](kube-prometheus-stack/) | Prometheus, Grafana, Alertmanager |
| [Uptime Kuma](uptime-kuma/) | Uptime monitoring and status page |
| [Kuma Ingress Watcher](kuma-ingress-watcher/) | Automatic ingress monitor registration |
| [Unpoller](unpoller/) | UniFi network metrics exporter |
| [Cribl Edge](cribl-edge/) | Log and metric collection agent |

### Media Management

| App | Description |
|-----|-------------|
| [Radarr](radarr/) | Movie library management |
| [Sonarr](sonarr/) | TV series management |
| [Prowlarr](prowlarr/) | Indexer aggregator for *arr apps |
| [SABnzbd](sabnzbd/) | Usenet downloader |
| [Pinchflat](pinchflat/) | YouTube channel archival |
| [Tautulli](tautulli/) | Plex analytics |
| [Kometa](kometa/) | Plex metadata and collections manager |

### Infrastructure & Core Services

| App | Description |
|-----|-------------|
| [CloudNative-PG](cloudnative-pg/) | PostgreSQL operator |
| [Cert Manager](cert-manager/) | TLS certificate automation |
| [Traefik](traefik/) | Ingress controller |
| [Tailscale](tailscale/) | Mesh VPN |
| [Metrics Server](metrics-server/) | Kubernetes resource metrics |

### Networking & DNS

| App | Description |
|-----|-------------|
| [Blocky](blocky/) | DNS-based ad blocking |

### AI & Automation

| App | Description |
|-----|-------------|
| [Open WebUI](open-webui/) | Web UI for local LLMs |
| [Ollama](ollama/) | External LLM inference server (bridge) |
| [Openclaw](openclaw/) | AI agent gateway (Discord) |
| [ha-mcp](ha-mcp/) | Home Assistant MCP server for AI integration |

### Dashboards & Management

| App | Description |
|-----|-------------|
| [Headlamp](headlamp/) | Kubernetes web UI |
| [Homarr](homarr/) | Homelab dashboard |
| [Homer](homer/) | Static homepage |
| [Wiki.js](wikijs/) | Documentation wiki |
| [ttyd](ttyd/) | Web-based terminal |

### Utilities

| App | Description |
|-----|-------------|
| [Error Pages](error-pages/) | Custom Traefik error pages |
| [Renovate](renovate/) | Automated dependency update bot |

## Adding a New Application

### Step 1: Create Chart Structure (this repo)

```bash
mkdir <app-name>
```

**`Chart.yaml`** — upstream chart dependency:
```yaml
apiVersion: v2
name: <app-name>
type: application
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: <chart-name>
    version: <chart-version>
    repository: https://<chart-repo-url>/
```

**`values.yaml`** — configuration overrides:
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
      - host: <app-name>.k8s.firekatt.ca
        paths:
          - path: /
            pathType: Prefix
```

Download dependencies:
```bash
helm dependency update
```

### Step 2: Create App Config (homelab-config repo)

Create `apps/<app-name>/app-config.json`:
```json
{
    "targetRevision": "HEAD",
    "namespace": "app-<app-name>",
    "createNamespace": true,
    "replace": false
}
```

### Step 3: Commit & Push

ArgoCD automatically detects and syncs new applications when `selfHeal: true` is enabled.

## Secret Management

This homelab uses **Sealed Secrets** to safely store sensitive data in git. Encrypted secrets can be committed and are decrypted only inside the cluster.

```bash
# Create and seal a secret
kubectl create secret generic <name> \
  --namespace=<namespace> \
  --from-literal=<key>=<value> \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml
```

See [SECRETS.md](SECRETS.md) for full documentation.

## Cluster Information

| | |
|---|---|
| **Platform** | Talos Linux |
| **Kubernetes** | v1.33+ |
| **Storage** | Longhorn |
| **Ingress** | Traefik v3 |
| **GitOps** | ArgoCD |
| **Load Balancer IP** | 10.0.1.100 |

## Dependency Management

```bash
# Add FireLabs common library
helm repo add firelabs https://fireball1725.github.io/firelabs-helm-common/
helm repo update

# Download/update chart dependencies
helm dependency update

# Find latest chart versions
helm search repo <chart-name> --versions | head -10
```

## Useful Commands

```bash
# Force ArgoCD hard refresh
kubectl -n core-argocd patch application <app-name> --type merge \
  -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'

# Check ArgoCD app status
kubectl get applications -n core-argocd

# Check pod health across namespaces
kubectl get pods --all-namespaces --field-selector status.phase!=Running,status.phase!=Succeeded

# Generate a long-lived service account token
kubectl create token <service-account> -n <namespace> --duration=87600h
```
