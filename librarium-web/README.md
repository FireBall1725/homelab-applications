# librarium-web

> React frontend for the self-hosted Librarium library tracker

## Overview

Librarium is a self-hosted library tracker (books, ebooks, audiobooks) built around an abstract work / concrete edition (FRBR) data model. This chart deploys the static nginx bundle that serves the React UI. It is the companion to [`librarium-api`](../librarium-api/).

## Chart Details

| | |
|---|---|
| **Upstream Project** | [librarium](https://github.com/fireball1725/librarium) |
| **Upstream Chart** | `librarium-web` (vendored from `librarium/web/deploy/helm/librarium-web`) |
| **Chart Version** | `0.1.0` |
| **App Version** | `26.4.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

This is Option A (self-contained): the upstream chart is copied verbatim into `templates/` and `charts/`. Only `values.yaml` contains homelab overrides. A future upstream refresh is a clean re-copy + reapply-overrides.

## Ingress

| Host | Description |
|------|-------------|
| `librarium.k8s.firekatt.ca` | Librarium Web UI (internal LAN only) |

The `*.k8s.firekatt.ca` zone resolves inside the LAN only. No TLS is terminated at this ingress.

## Namespace

This chart **must** deploy into the `app-librarium` namespace, the same namespace as `librarium-api`. The web image's bundled `nginx.conf` proxies `/api` and `/health` to `http://librarium-api:8080` by the bare service name, so Kubernetes DNS only resolves that when both workloads share a namespace.

## Persistence

None. The web tier is a stateless static nginx bundle - 2 replicas by default.

## Secrets

None. No database credentials, no JWT, no tokens - the browser talks to `librarium-api` through the same ingress host via the nginx `/api` proxy path, and the API handles all authentication server-side.

## Notes

- The service listens on port 3000 (matches the nginx config baked into the image).
- Probes hit `/` on the http port - a 200 from the SPA index means nginx is alive and serving static assets.
- Scale horizontally by raising `controller.replicas`; no sticky sessions are required.
