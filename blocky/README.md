# Blocky

> DNS resolver and ad-blocker for the homelab network

## Overview

Blocky is an advanced DNS proxy and ad-blocker that provides network-level ad and tracker blocking, custom DNS mappings, DNS-over-HTTPS (DoH), and DNS-over-TLS (DoT) support. It exposes Prometheus metrics for monitoring query patterns and blocklist effectiveness.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Blocky](https://github.com/0xERR0R/blocky) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `v0.28.2` |
| **App Version** | `v0.28.2` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

No HTTP ingress. Blocky exposes DNS (UDP/TCP port 53) and an API port (4000) via Kubernetes services.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `logs` | optional | `longhorn` | Query logs (disabled by default) |

## Secrets

No secrets required.

## Notes

- Configuration is managed via a ConfigMap (`configmap.yaml`) mounted into the container
- Includes custom DNS mappings for internal services
- Prometheus metrics enabled; includes `ServiceMonitor` and `PrometheusRules` templates
- Supports upstream DoH and DoT resolvers with caching and prefetching
- Query logging to MySQL is configured but optional
