# Cribl Edge

> Edge data collection and streaming agent

## Overview

Cribl Edge is a lightweight agent that collects logs and metrics from the Kubernetes cluster and forwards them to a central Cribl Leader for processing and routing. It handles log aggregation and telemetry collection from cluster workloads.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Cribl](https://cribl.io) |
| **Helm Chart** | `edge` ([Cribl Helm Charts](https://criblio.github.io/helm-charts/)) |
| **Chart Version** | `^4.12.1` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

No HTTP ingress. Cribl Edge communicates outbound to the Cribl Leader.

## Persistence

No persistent storage.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `cribl-leader-secret` | Leader URL and connection credentials |
| `cribl-api-secret` | API authentication credentials |

## Notes

- `NODE_TLS_REJECT_UNAUTHORIZED` is enabled to handle self-signed certificates on the Leader
