# Tautulli

> Plex media server monitoring and analytics

## Overview

Tautulli tracks Plex playback activity, providing detailed statistics on what's being watched, by whom, and when. It also sends notifications (e.g., when new content is added) and generates watch history reports. A Prometheus `ServiceMonitor` is included for metrics scraping.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Tautulli](https://tautulli.com) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2.16.1` |
| **App Version** | `2.16.1` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `tautulli.k8s.firekatt.ca` | Tautulli web UI (port 8181) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and watch history database |

## Secrets

No secrets required.

## Notes

- Includes a `ServiceMonitor` template for Prometheus metrics scraping
- An init container sets correct file ownership (PUID/PGID 1000) before startup
- Timezone set to `America/Toronto`
- Memory limit: 512Mi
