# Kometa

> Plex library metadata and collections manager

## Overview

Kometa (formerly Plex Meta Manager) automatically manages Plex library collections, metadata overlays, and artwork using configurable YAML definitions. It runs as a daily CronJob, pulling movie/TV metadata from TMDB and applying collection rules and overlays to the Plex library.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Kometa](https://kometa.wiki) |
| **Helm Chart** | Custom (no upstream dependency) |
| **Chart Version** | `2.1.0` |
| **App Version** | `2.1.0` |
| **Common Library** | Custom |

## Ingress

No HTTP ingress. Runs as a CronJob only.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `media` | — | NFS | Read access to NFS media storage for library scanning |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `kometa-secrets` | Plex API token and TMDB API key |

## Notes

- Runs daily at midnight UTC via a Kubernetes `CronJob`
- Collection and overlay configurations are managed via ConfigMaps
- Requires NFS media mount for library path resolution
