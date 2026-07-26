# firebin-web

> React frontend (nginx) for the self-hosted FireBin parts inventory

## Overview

This chart deploys the nginx bundle that serves the FireBin React UI. It is the companion to [`firebin-api`](../firebin-api/) and the only public entrypoint: nginx serves the static UI and reverse-proxies `/api` to the api in-cluster.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [firebin](https://github.com/fireball1725/firebin) |
| **Image** | `ghcr.io/fireball1725/firebin-web:26.7.0` (public, multi-arch amd64/arm64) |
| **Chart Version** | `0.1.0` |
| **App Version** | `26.7.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Ingress

| Host | Description |
|------|-------------|
| `firebin.k8s.firekatt.ca` | FireBin UI (internal LAN only, HTTP) |

The `*.k8s.firekatt.ca` zone resolves inside the LAN only; blocky maps the wildcard to the Traefik LB at `10.0.1.100`, so no per-host DNS entry is needed.

## Namespace

This chart **must** deploy into `app-firebin`, the same namespace as `firebin-api`. The image's bundled `nginx.conf` proxies `/api` to `http://firebin-api:8080` by bare service name, which Kubernetes DNS only resolves when both workloads share a namespace.

## Persistence

None. The web tier is a stateless static nginx bundle, 2 replicas by default.

## Secrets

None. The browser talks to `firebin-api` through the same ingress host via the nginx `/api` proxy, and the api handles all authentication server-side.

## Notes

- The service listens on port 3000 (matches the nginx config baked into the image).
- Probes hit `/` on the http port; a 200 from the SPA index means nginx is serving.
- Scale horizontally by raising `controller.replicas`; no sticky sessions required.
- The camera scanner and WebUSB label printer need a secure origin (HTTPS or localhost). Over plain HTTP on the LAN they stay disabled; put the host behind TLS if you need them.
