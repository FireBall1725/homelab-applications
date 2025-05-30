# ArgoCD Applications Repo

This repository contains all the Helm charts for the applications running in my home kubernetes cluster. All applications are deployed and managed using ArgoCD.

## Structure

Each application has its own top-level folder. Applications must be lowercase and can use - for space. This folder contains the full helm chart required for deployment.

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