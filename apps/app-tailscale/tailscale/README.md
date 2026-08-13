# Tailscale

> Mesh VPN for secure remote access to the homelab

## Overview

The Tailscale Kubernetes operator runs a subnet router that advertises internal cluster and node network CIDRs into the Tailscale mesh network. This allows secure, zero-config remote access to all homelab services without port forwarding or a traditional VPN server.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Tailscale](https://tailscale.com) |
| **Helm Chart** | `tailscale-operator` ([Tailscale](https://pkgs.tailscale.com/helmcharts)) |
| **Chart Version** | `1.92.5` |
| **App Version** | — |
| **Common Library** | Upstream chart |

## Ingress

No HTTP ingress. Tailscale provides VPN connectivity at the network level.

## Persistence

No persistent storage.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `tailscale-oauth` | OAuth client credentials for Tailscale API authentication |

## Notes

- Configured as a subnet router `Connector` resource
- Advertises pod CIDR (10.244.0.0/16), node network (10.0.1.0/24), and service CIDR (10.96.0.0/12)
- Accepts routes from other Tailscale nodes (split tunneling)
- Tags: `k8s-connector`, `homelab`
