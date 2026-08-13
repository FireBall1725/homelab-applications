# Error Pages

> Custom HTTP error page server for Traefik

## Overview

Error Pages serves themed HTML error pages for all 4xx and 5xx HTTP status codes. It is integrated with Traefik as a global middleware, so any request that results in an error from any backend returns a consistent, styled error page instead of a raw browser error.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [error-pages](https://github.com/tarampampam/error-pages) |
| **Helm Chart** | Custom (no upstream dependency) |
| **Chart Version** | `3` |
| **App Version** | `3` |
| **Common Library** | Custom |

## Ingress

No direct ingress. Traffic is routed through Traefik middleware — error responses from any service are intercepted and served by this deployment.

## Persistence

No persistent storage. Stateless deployment.

## Secrets

No secrets required.

## Notes

- Uses the `connection` theme
- Deployed with 2 replicas for high availability
- Runs as non-root (UID 1000) with a read-only root filesystem
- Applied globally via a Traefik `Middleware` resource defined in `templates/middleware.yaml`
