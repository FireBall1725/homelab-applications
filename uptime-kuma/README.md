# Uptime Kuma

> Self-hosted uptime monitoring and status page

## Overview

Uptime Kuma monitors the availability of homelab services and generates a public-facing status page. It is paired with the [kuma-ingress-watcher](../kuma-ingress-watcher/) which automatically registers Kubernetes ingress endpoints as monitors, so new services are tracked without manual configuration.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Uptime Kuma](https://github.com/louislam/uptime-kuma) |
| **Helm Chart** | `uptime-kuma` ([dirsigler](https://dirsigler.github.io/uptime-kuma-helm)) |
| **Chart Version** | `2.24.0` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `status.k8s.firekatt.ca` | Uptime Kuma dashboard and status page |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | `4Gi` | `longhorn` | Monitor configuration and uptime history |

## Secrets

No secrets required.

## Notes

- Works in conjunction with [kuma-ingress-watcher](../kuma-ingress-watcher/) for automatic monitor creation
- Memory limit: 512Mi
- Port: 3001
