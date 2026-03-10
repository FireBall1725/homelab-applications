# Prowlarr

> Indexer manager and aggregator for the *arr media stack

## Overview

Prowlarr manages torrent and NZB indexers in one place and syncs them automatically to Radarr and Sonarr. Rather than configuring indexers individually in each *arr application, Prowlarr acts as the central indexer hub for the entire media stack.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Prowlarr](https://github.com/Prowlarr/Prowlarr) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2.3.0` |
| **App Version** | `2.3.0` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `prowlarr.k8s.firekatt.ca` | Prowlarr web UI (port 9696) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and indexer cache |

## Secrets

No secrets required.

## Notes

- An init container sets correct file ownership (UID 568) before startup
- Prowlarr integrates with Radarr and Sonarr via their respective internal service URLs
