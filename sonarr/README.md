# Sonarr

> TV series collection manager and automated downloader

## Overview

Sonarr monitors TV series release feeds and automatically searches for, downloads, and organises episodes into the media library. It integrates with SABnzbd for downloading and Prowlarr for indexer management, and notifies Plex when new content is available.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Sonarr](https://github.com/Sonarr/Sonarr) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `4.0.16` |
| **App Version** | `4.0.16` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `sonarr.k8s.firekatt.ca` | Sonarr web UI (port 8989) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and database |
| `media` | — | NFS | Shared NFS media storage (TV shows) |

## Secrets

No secrets required.

## Notes

- Shares the NFS media mount with Radarr and SABnzbd so hardlinks work correctly
- An init container sets correct file ownership (UID 568) before startup
