# CloudNative-PG

> Kubernetes-native PostgreSQL operator

## Overview

CloudNative-PG (CNPG) is a Kubernetes operator that manages the full lifecycle of PostgreSQL clusters — provisioning, high availability, backups, and failover. It is used in this homelab to provide a managed PostgreSQL instance for Wiki.js.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [CloudNative-PG](https://cloudnative-pg.io) |
| **Helm Chart** | `cloudnative-pg` ([CloudNative-PG](https://cloudnative-pg.github.io/charts)) |
| **Chart Version** | `v0.27.0` |
| **App Version** | `1.28.0` |
| **Common Library** | Upstream chart |

## Ingress

No HTTP ingress. This is a cluster operator; databases are accessed internally via Kubernetes services.

## Persistence

Storage is provisioned by the operator per-cluster when a `Cluster` resource is created. See [wikijs](../wikijs/) for the PostgreSQL cluster configuration.

## Secrets

No secrets required at the operator level. Per-cluster credentials are managed by the operator.

## Notes

- `PodMonitor` is enabled for Prometheus metrics scraping
- Grafana dashboard auto-discovery is configured
- Resource limits: 50m CPU, 128Mi memory
