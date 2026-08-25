# Sportarr

> Sports PVR for usenet and torrents

## Overview

Sportarr monitors the leagues and athletes you follow, searches your indexers for event releases, hands them to a download client, and renames the results into a sports library Plex can read. It is the Sonarr/Radarr model applied to fights, races, and games rather than series and films. Like the rest of `app-media`, it points at Prowlarr for indexers and SABnzbd for downloads; both are configured in the web UI after first start.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Sportarr](https://github.com/Sportarr/Sportarr) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `4.1.3` |
| **App Version** | `4.1.3.1113` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `sportarr.k8s.firekatt.ca` | Sportarr web UI (port 1867) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `2Gi` | `longhorn` | Application configuration and SQLite database |
| `media` | — | NFS | Shared NFS media storage; the library goes in `/media/sports` |

## Secrets

No secrets required. Sportarr runs on SQLite by default and the chart does not enable the optional PostgreSQL backend.

## Notes

- No `update-volume-permission` init container, unlike its siblings here. The Sportarr entrypoint starts as root and runs `chown -R $PUID:$PGID /config /app` itself before dropping privileges, so a busybox chown pass would only repeat work the image already does.
- `PUID=1026` and `PGID=100` match the existing ownership of `/media/sports` on the NAS, so downloaded events land with the same uid and gid as the files already there.
- `TZ` is `America/Toronto`, not the `UTC` the other media charts use. Event start times drive both the calendar view and the release search windows, so local time is the useful one.
- Probes are HTTP against `/ping` rather than the library's default TCP check. Kestrel binds port 1867 before the database is open, which makes a TCP probe report ready too early.
- Shares the NFS media mount with Radarr, Sonarr and SABnzbd at the same `/media` path, which is what lets hardlinks work across all four.
- `/volume2/media` was at 17 GB free of 58 TB when this was added (2026-08-25). A PVR that downloads full event recordings will hit that before it does anything useful.
