# Homepage

> Dynamic homelab landing page with service integrations

## Overview

Homepage is a modern, highly customisable static homepage with built-in integrations for popular self-hosted services. It displays live service status, statistics, and bookmarks in a clean, responsive layout.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Homepage](https://gethomepage.dev) |
| **Helm Chart** | `homepage` ([m0nsterrr](https://m0nsterrr.github.io/helm-charts/)) |
| **Chart Version** | `0.1.5` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `homepage.k8s.firekatt.ca` | Homepage dashboard |

## Persistence

No persistent storage. Configuration is loaded from an external source.

## Secrets

No secrets required.

## Notes

- Dashboard configuration is fetched from an external GitHub repository
