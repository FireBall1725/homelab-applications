# Radarr

> Movie collection manager and automated downloader

## Overview

Radarr monitors movie release feeds and automatically searches for, downloads, and organises movies into the media library. It integrates with SABnzbd for downloading and Prowlarr for indexer management, and notifies Plex when new content is available.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Radarr](https://github.com/Radarr/Radarr) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `6.0.4` |
| **App Version** | `6.0.4` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `radarr.k8s.firekatt.ca` | Radarr web UI (port 7878) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and database |
| `media` | — | NFS | Shared NFS media storage (movies) |

## Secrets

No secrets required.

## Notes

- Shares the NFS media mount with Sonarr and SABnzbd so hardlinks work correctly
- An init container sets correct file ownership (UID 568) before startup
