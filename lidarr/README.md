# Lidarr

> Music collection manager and automated downloader

## Overview

Lidarr monitors music release feeds and automatically searches for, downloads, and organises albums into the media library. It integrates with SABnzbd for downloading and Prowlarr for indexer management, and is the music counterpart to Sonarr (TV) and Radarr (movies).

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Lidarr](https://github.com/Lidarr/Lidarr) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `1.0.0` |
| **App Version** | `3.1.0.4875-ls25` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `lidarr.k8s.firekatt.ca` | Lidarr web UI (port 8686) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and database |
| `media` | — | NFS | Shared NFS media storage (Music) |

## Secrets

No secrets required.

## Notes

- Shares the NFS media mount with Sonarr, Radarr, and SABnzbd so hardlinks work correctly
- An init container sets correct file ownership (UID 568) before startup
