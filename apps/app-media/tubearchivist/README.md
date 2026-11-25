# TubeArchivist

> Self-hosted YouTube archive: subscribe to channels, download with yt-dlp, browse and search.

## Overview

TubeArchivist downloads the videos from channels and playlists you subscribe to and keeps them in its own library. The Jellyfin TubeArchivist plugin reads that library, so each channel shows up in Jellyfin as a show.

Elasticsearch and Redis run as sidecars in the same pod, so TubeArchivist reaches them on `localhost` and nothing else in the cluster can.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [TubeArchivist](https://github.com/tubearchivist/tubearchivist) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `0.1.0` |
| **App Version** | `v0.6.0` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

`https://tubearchivist.k8s.firekatt.ca` (internal)

## Storage

| Mount | Source | Holds |
|---|---|---|
| `/cache` | Longhorn, 10Gi | thumbnails and channel art |
| `/media` | NFS `10.0.1.241:/volume2/media` | the whole media share, like the other media apps |
| `/youtube` | NFS `10.0.1.241:/volume2/media/youtube` | downloaded videos, one folder per channel ID |
| Elasticsearch | Longhorn, 10Gi | the index |
| Redis | Longhorn, 1Gi | task queue |

`/volume2/media/youtube` has to exist on the NAS before the pod starts, or the NFS mount fails.

## Login

The admin username is `admin`. The password is in the `tubearchivist` SealedSecret (`TA_PASSWORD`) and in the macOS Keychain item `tubearchivist-admin`.

## Jellyfin plugin

In the plugin settings, set the address to `https://tubearchivist.k8s.firekatt.ca` and paste the API token from TubeArchivist under Settings, Application.
