# Kube Prometheus Stack

> Full observability stack for Kubernetes (Prometheus, Grafana, Alertmanager)

## Overview

The kube-prometheus-stack bundles Prometheus, Grafana, Alertmanager, node-exporter, and kube-state-metrics into a single deployment. It provides cluster-wide metrics collection, dashboards, and alerting for all homelab applications and infrastructure. Custom Longhorn backup dashboards and alerting rules are included.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts) |
| **Helm Chart** | `kube-prometheus-stack` ([Prometheus Community](https://prometheus-community.github.io/helm-charts)) |
| **Chart Version** | `81.2.2` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `grafana.k8s.firekatt.ca` | Grafana dashboards |
| `prometheus.k8s.firekatt.ca` | Prometheus UI |
| `alertmanager.k8s.firekatt.ca` | Alertmanager UI |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `prometheus` | `50Gi` | `longhorn` | Prometheus TSDB (30-day retention) |
| `grafana` | `10Gi` | `longhorn` | Grafana dashboards and configuration |
| `alertmanager` | `5Gi` | `longhorn` | Alertmanager state |

## Secrets

No secrets at the Helm level. Alert webhook URLs (Discord) are configured in `values.yaml`.

## Notes

- `ServiceMonitor` selector covers all namespaces, scraping any app with Prometheus annotations
- Custom templates included: Longhorn backup alert rules and Grafana dashboard
- Alert notifications route to Discord webhooks
- Grafana admin credentials should be rotated after initial setup
