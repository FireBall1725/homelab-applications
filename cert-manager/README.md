# Cert Manager

> Automated TLS certificate management for Kubernetes

## Overview

cert-manager automates the issuance and renewal of TLS certificates from Let's Encrypt using DNS-01 challenges via Cloudflare. It provides cluster-wide certificate automation for all ingress endpoints in the homelab, eliminating manual certificate management.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [cert-manager](https://cert-manager.io) |
| **Helm Chart** | `cert-manager` ([Jetstack](https://charts.jetstack.io)) |
| **Chart Version** | `v1.19.4` |
| **App Version** | `1.19.2` |
| **Common Library** | Upstream chart |

## Ingress

No HTTP ingress. cert-manager is an operator that provisions certificates for other applications.

## Persistence

No persistent storage. Certificate data is stored as Kubernetes secrets.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `cloudflare-api-token` | Cloudflare API token used for DNS-01 ACME challenges |

## Notes

- Two `ClusterIssuer` resources are deployed: staging (for testing) and production (Let's Encrypt)
- DNS-01 challenge type is used, allowing certificate issuance without exposing ports publicly
- CRD installation is enabled in the chart
- Prometheus metrics are enabled for monitoring certificate health
