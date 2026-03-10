# Metrics Server

> Kubernetes resource metrics provider

## Overview

Metrics Server collects resource usage data (CPU, memory) from each kubelet and exposes it via the Kubernetes Metrics API. It is required for Horizontal Pod Autoscaler (HPA) and the `kubectl top` command to function.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Metrics Server](https://github.com/kubernetes-sigs/metrics-server) |
| **Helm Chart** | `metrics-server` ([Kubernetes SIGs](https://kubernetes-sigs.github.io/metrics-server/)) |
| **Chart Version** | `3.13.0` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

No HTTP ingress. Metrics Server is a Kubernetes system component accessed via the API server.

## Persistence

No persistent storage.

## Secrets

No secrets required.

## Notes

- Kubelet insecure TLS is enabled (`--kubelet-insecure-tls`) as required for Talos Linux
- Configured to prefer `InternalIP` for kubelet communication
- Runs as non-root (UID 1000) with a read-only root filesystem
