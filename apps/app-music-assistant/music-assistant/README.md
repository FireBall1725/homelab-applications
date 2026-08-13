# Music Assistant

> Unified multi-room music streaming server

## Overview

Music Assistant is a free, open-source music player and library manager that aggregates multiple music providers (Spotify, Tidal, local files, etc.) and streams to a wide range of output targets including Sonos, Chromecast, and Home Assistant media players. It integrates with Home Assistant for media player control.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Music Assistant](https://music-assistant.io) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2.5.5` |
| **App Version** | `2.5.5` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `ma.k8s.firekatt.ca` | Music Assistant web interface (port 8095) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `10Gi` | `longhorn` | Application configuration and library database |

## Secrets

No secrets required.

## Notes

- `hostNetwork` is enabled to support mDNS/multicast for device discovery (e.g., Chromecast, AirPlay)
- An init container sets correct file ownership (UID 568) before startup
