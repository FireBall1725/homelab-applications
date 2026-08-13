# SABnzbd

> Usenet binary downloader and post-processor

## Overview

SABnzbd is a feature-rich Usenet downloader that handles NZB files, automating the download, verification, repair, and extraction pipeline. It serves as the download client for Radarr and Sonarr in the media stack.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [SABnzbd](https://sabnzbd.org) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `4.5.5` |
| **App Version** | `4.5.5` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `sabnzbd.k8s.firekatt.ca` | SABnzbd web UI (port 8080) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration |
| `media` | — | NFS | Shared NFS media storage (downloads) |

## Secrets

No secrets required. Usenet server credentials are stored within SABnzbd's own configuration.

## Notes

- Shares the NFS media mount with Radarr and Sonarr so hardlinks work correctly
- An init container sets correct file ownership (UID 568) before startup
