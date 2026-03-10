# Unpoller

> UniFi network metrics exporter for Prometheus

## Overview

Unpoller (UniFi Poller) collects metrics from a UniFi network controller and exports them to Prometheus. It provides detailed visibility into network device status, client connections, and traffic statistics, with pre-built Grafana dashboards for the UniFi network infrastructure.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [UniFi Poller](https://unpoller.com) |
| **Helm Chart** | Raw Kubernetes manifests (no Helm chart) |
| **Chart Version** | — |
| **App Version** | — |
| **Common Library** | None |

## Ingress

No HTTP ingress. Metrics are scraped by Prometheus via `ServiceMonitor`.

## Persistence

No persistent storage.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `unpoller-secret` | UniFi controller credentials (username, password, URL) |

## Notes

- Deployed as raw Kubernetes manifests rather than a Helm chart
- Includes Grafana dashboard definitions in the `dashboards/` directory (auto-discovered by kube-prometheus-stack)
- A `ServiceMonitor` resource enables Prometheus scraping
