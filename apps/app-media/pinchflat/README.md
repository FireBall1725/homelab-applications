# Pinchflat

> Automated YouTube channel archival and downloader

## Overview

Pinchflat is a self-hosted YouTube downloader that automatically archives videos from subscribed channels using yt-dlp under the hood. It monitors channels on a schedule and downloads new content to the NFS media share, making videos available for local playback without relying on YouTube availability.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Pinchflat](https://github.com/kieraneglin/pinchflat) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2025.01.30` |
| **App Version** | `2025.01.30` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `pinchflat.k8s.firekatt.ca` | Pinchflat web UI (internal ingress class) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `1Gi` | `longhorn` | Application configuration and database |
| `downloads` | — | NFS | NFS media storage for downloaded videos |

## Secrets

No secrets required.

## Notes

- Downloads are written to the shared NFS media storage
- An init container sets correct file ownership (UID 1000) before startup
- Timezone set to `America/Edmonton`
