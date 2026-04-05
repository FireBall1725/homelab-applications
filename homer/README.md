# Homer

> Simple, static homelab dashboard

## Overview

Homer is a lightweight, static homepage for the homelab. It renders a configurable dashboard of bookmarks and service links from a YAML configuration file. The configuration is loaded externally from GitHub, keeping it separate from this repo.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Homer](https://github.com/bastienwirths/homer) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `v25.05.2` |
| **App Version** | `v25.05.2` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `homepage.k8s.firekatt.ca` | Homer dashboard |

## Persistence

No persistent storage. Assets volume is disabled; configuration is loaded from an external source.

## Secrets

No secrets required.

## Notes

- Dashboard configuration is fetched from an external GitHub repository rather than stored locally
- Asset persistence is disabled (stateless deployment)
