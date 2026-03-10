# Kuma Ingress Watcher

> Automatic Uptime Kuma monitor creation for Kubernetes ingresses

## Overview

Kuma Ingress Watcher watches Kubernetes `Ingress` resources and automatically creates corresponding monitors in Uptime Kuma. When a new ingress is deployed, a monitor is registered without any manual intervention. This ensures all services are automatically included in uptime tracking.

## Chart Details

| | |
|---|---|
| **Upstream Project** | Internal / Custom |
| **Helm Chart** | Custom (no upstream dependency) |
| **Chart Version** | `1.4.1` |
| **App Version** | `1.4.1` |
| **Common Library** | Custom |

## Ingress

No HTTP ingress. This is a controller that runs in the background.

## Persistence

No persistent storage.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `kuma-ingress-watcher-credentials` | Uptime Kuma API key for creating monitors |

## Notes

- Requires RBAC with read-only access to cluster-wide `Ingress` resources (configured in `templates/rbac.yaml`)
- Watch interval: 30 seconds
- Communicates with Uptime Kuma via its internal cluster service URL
- Pairs with [uptime-kuma](../uptime-kuma/)
