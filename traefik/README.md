# Traefik

> Kubernetes ingress controller and reverse proxy

## Overview

Traefik is the cluster's primary ingress controller, routing external HTTP/HTTPS traffic to the appropriate services. It handles TLS termination using certificates from cert-manager, applies global middleware (error pages, rate limiting), and exposes Prometheus metrics. It is deployed with 2 replicas for high availability.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Traefik](https://traefik.io) |
| **Helm Chart** | `traefik` ([Traefik Labs](https://traefik.github.io/charts)) |
| **Chart Version** | `35.4.0` |
| **App Version** | `3.4.0` |
| **Common Library** | Upstream chart |

## Ingress

Traefik is the ingress controller itself. It receives traffic on ports 80/443 via a `LoadBalancer` service at a fixed internal IP (10.0.1.100).

## Persistence

No persistent storage.

## Secrets

No secrets required.

## Notes

- `internal` is the default `IngressClass` used by all other apps
- The error-pages middleware is applied globally to all entrypoints for consistent error handling
- TLS certificates are managed by cert-manager and referenced via custom `Certificate` templates
- Prometheus metrics are enabled and scraped by the kube-prometheus-stack
- 2 replicas deployed for availability
