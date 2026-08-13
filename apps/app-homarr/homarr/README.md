# Homarr

> Modern homelab service dashboard

## Overview

Homarr is a sleek, configurable dashboard for the homelab with app tiles, integrations, and widgets. It provides a single-pane-of-glass view for all homelab services with live status indicators and app shortcuts.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Homarr](https://homarr.dev) |
| **Helm Chart** | `homarr` ([Homarr Labs](https://homarr-labs.github.io/charts/)) |
| **Chart Version** | `5.3.0` |
| **App Version** | `v1.51.0` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `home.k8s.firekatt.ca` | Homarr dashboard |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `homarrDatabase` | — | `longhorn` | Dashboard configuration and user data |

## Secrets

No secrets required.

## Notes

- Timezone set to `America/Toronto`
