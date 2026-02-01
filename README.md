# ArgoCD Applications Repo

This repository contains all the Helm charts for the applications running in my home kubernetes cluster. All applications are deployed and managed using ArgoCD.

## Architecture Overview

This homelab uses a **two-repository pattern**:

1. **homelab-applications** (this repo): Contains Helm chart wrappers with configuration overrides
2. **homelab-config**: Contains ArgoCD application metadata (namespace, sync policy, etc.)

ArgoCD monitors this repository (`https://github.com/FireBall1725/homelab-applications.git`) and automatically syncs applications when changes are detected.

## Structure

Each application has its own top-level folder. Applications must be lowercase and can use `-` for space. This folder contains the full helm chart required for deployment.

### Directory Layout

```
<app-name>/
├── Chart.yaml          # Helm chart with upstream dependency
├── Chart.lock          # Generated lock file (after helm dependency update)
├── values.yaml         # Configuration overrides
└── charts/             # Downloaded chart dependencies (gitignored)
```

## Categories & Apps

### Home Automation
- [**Home Assistant**](home-assistant/) – Main smart home hub
- [**Node-RED**](node-red/) – Logic/automation layer
- [**ESPHome**](esphome/) – Sensor/device integration

### Monitoring & Observability
- [**Prometheus**](prometheus/) – Metrics backend
- [**Grafana**](grafana/) – Dashboards
- [**Alertmanager**](alertmanager/) – Notifications
- [**Cribl Edge**](cribl-edge/) – Lightweight log and metric collector

### Media Management
- [**Radarr**](radarr/) – Movies
- [**Sonarr**](sonarr/) – TV shows
- [**Bazarr**](bazarr/) – Subtitles
- [**Prowlarr**](prowlarr/) – Indexer management
- [**SABnzbd**](sabnzbd/) – NZB downloader
- [**Tautulli**](tautulli/) – Plex analytics

### Networking & DNS
- [**Blocky**](blocky/) – DNS filtering

### Dashboards & Management
- [**Headlamp**](headlamp/) – Kubernetes dashboard
- [**Homarr**](homarr/) – Application dashboard
- [**Homer**](homer/) – Static homepage

## Adding a New Application

### Step 1: Create Chart Structure (this repo)

```bash
mkdir -p <app-name>
cd <app-name>
```

Create `Chart.yaml` with upstream chart dependency:
```yaml
apiVersion: v2
name: <app-name>
type: application
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: <upstream-chart-name>
    version: <chart-version>
    repository: https://<chart-repo-url>/
```

Create `values.yaml` with configuration overrides:
```yaml
<chart-name>:
  # Common patterns for this homelab:

  env:
    TZ: "America/Toronto"

  persistence:
    <volume-name>:
      enabled: true
      storageClassName: "longhorn"
      size: 10Gi

  ingress:
    enabled: true
    hosts:
      - host: <app-name>.k8s.firekatt.ca
        paths:
          - path: /
            # NOTE: Check upstream chart - some use 'pathType', others use 'type'
            pathType: Prefix  # or type: Prefix
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

**Namespace Conventions:**
- Most apps: `app-<app-name>`
- Core infrastructure: `core-<app-name>` (e.g., `core-longhorn`, `core-argocd`)
- System apps: `kube-system` (for cluster management tools like headlamp)

### Step 3: Commit & Push

```bash
# This repo
git add <app-name>/
git commit -m "Add <app-name>"
git push

# homelab-config repo
cd /path/to/homelab-config
git add apps/<app-name>/
git commit -m "Add <app-name> config"
git push
```

ArgoCD will automatically detect and sync the new application (if `selfHeal: true` is enabled).

## Example Helm Chart

In the application folder, create `Chart.yaml`, use this as an example for what is required.

```yaml
apiVersion: v2
name: web-server
type: application
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: nginx-php
    version: 1.2.2
    repository: https://k8s-at-home.com/charts/
```

## Managing Helm Dependencies

To download and compile a chart's dependencies, run:

```bash
helm dependency update
```

## Secret Management

This homelab uses **Sealed Secrets** to safely store sensitive data (tokens, passwords, API keys) in git.

### Quick Guide

```bash
# Create and encrypt a secret
kubectl create secret generic <name> \
  --namespace=<namespace> \
  --from-literal=<key>=<secret-value> \
  --dry-run=client -o yaml | \
  kubeseal -o yaml > <app-name>/templates/sealed-secret.yaml

# Reference it in values.yaml
<chart-name>:
  env:
    - name: MY_SECRET
      valueFrom:
        secretKeyRef:
          name: <secret-name>
          key: <key>
```

**See [SECRETS.md](SECRETS.md) for complete documentation.**

## Common Patterns & Tips

### Finding Latest Chart Versions

```bash
# Add helm repo
helm repo add <repo-name> https://<chart-repo-url>/

# Search for versions
helm search repo <chart-name> --versions | head -10

# Or check GitHub releases
curl -sL https://api.github.com/repos/<org>/<repo>/releases/latest | jq -r '.tag_name'
```

### Ingress Configuration

**Important:** Different Helm charts use different structures for ingress paths.

Standard Kubernetes format:
```yaml
ingress:
  enabled: true
  hosts:
    - host: app.k8s.firekatt.ca
      paths:
        - path: /
          pathType: Prefix  # Kubernetes standard
```

Some charts (like Headlamp) use:
```yaml
ingress:
  enabled: true
  hosts:
    - host: app.k8s.firekatt.ca
      paths:
        - path: /
          type: Prefix  # Custom implementation
```

**Always check the upstream chart's `values.yaml` for the correct structure.**

### Image Tag Overrides

When a chart's default image doesn't exist or needs updating:

```yaml
<chart-name>:
  image:
    tag: v1.2.3  # Override default tag
```

## Troubleshooting

### ArgoCD Not Syncing Latest Changes

If ArgoCD shows synced but is using an old git revision:

```bash
# Force a hard refresh
kubectl -n core-argocd patch application <app-name> --type merge \
  -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'
```

### Ingress Not Creating

Common causes:
1. **Wrong path field**: Check if chart uses `pathType` or `type`
2. **Missing ingressClassName**: Some charts require explicit class
3. **Validation errors**: Check ArgoCD application status for details

```bash
# Check ArgoCD error message
kubectl get application -n core-argocd <app-name> -o jsonpath='{.status.operationState.message}'

# Check if ingress was created
kubectl get ingress -n <namespace> <app-name>
```

### Pod Not Starting

```bash
# Check pod status
kubectl get pods -n <namespace> -l app.kubernetes.io/name=<app-name>

# Check pod events
kubectl describe pod -n <namespace> <pod-name>

# Check logs
kubectl logs -n <namespace> <pod-name>
```

### Volume Mount Issues

After cluster network disruptions, volumes may get stuck. Check:

```bash
# Check Longhorn volume status
kubectl get volumes.longhorn.io -n core-longhorn

# Check volume attachments
kubectl get volumeattachments | grep <pvc-name>

# If needed, reboot the node
talosctl -n <node-ip> reboot
```

## Cluster Information

- **Platform**: Talos Linux v1.10.2
- **Kubernetes**: v1.33.0
- **Container Runtime**: containerd 2.0.5
- **Storage**: Longhorn v1.9.0
- **Ingress Controller**: Traefik
- **GitOps**: ArgoCD
- **Domain**: `*.k8s.firekatt.ca`
- **Load Balancer IP**: 10.0.1.100
- **Control Plane Nodes**: 3 (talos-cp-01, talos-cp-02, talos-cp-03)
- **Worker Nodes**: 8 (talos-wk-*, talos-vwk-*, talos-piwk-01)

## Useful Commands

### Generate Service Account Token

For apps requiring cluster access (like Headlamp):

```bash
# Create long-lived token (10 years)
kubectl create token <service-account> -n <namespace> --duration=87600h

# Copy to clipboard (macOS)
kubectl create token <service-account> -n <namespace> --duration=87600h | pbcopy
```

### Check Cluster Health

```bash
# Node status
kubectl get nodes

# Pod health across all namespaces
kubectl get pods --all-namespaces --field-selector status.phase!=Running,status.phase!=Succeeded

# ArgoCD application status
kubectl get applications -n core-argocd
```

## Notes for AI Assistants

When helping with this homelab:

- **Always verify** upstream chart structure before configuring (especially ingress paths)
- **Test** `helm dependency update` succeeds before committing
- **Update both repos**: homelab-applications (chart) + homelab-config (metadata)
- **Never auto-push**: User handles git commits and PRs
- **Use pbcopy** for copying tokens/secrets on macOS
- **Timezone**: America/Toronto
- **Storage class**: longhorn
- **Domain pattern**: `<app-name>.k8s.firekatt.ca`
- **Check ArgoCD** application status if deployment fails