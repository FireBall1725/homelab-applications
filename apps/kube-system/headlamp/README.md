# Headlamp

> Lightweight Kubernetes web UI

## Overview

Headlamp is a feature-rich, extensible Kubernetes dashboard for browsing cluster resources, viewing logs, and managing workloads. It provides a clean alternative to the default Kubernetes dashboard with a modern UI.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Headlamp](https://headlamp.dev) |
| **Helm Chart** | `headlamp` ([Kubernetes SIGs](https://kubernetes-sigs.github.io/headlamp/)) |
| **Chart Version** | `0.39.0` |
| **App Version** | `1.0.0` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `headlamp.k8s.firekatt.ca` | Headlamp web UI |

## Persistence

No persistent storage.

## Secrets

No secrets required.

## Notes

- Authentication uses a Kubernetes service account token (generate a long-lived token via `kubectl create token`)
- Access to the cluster is controlled by the service account's RBAC permissions
