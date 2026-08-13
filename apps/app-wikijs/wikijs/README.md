# Wiki.js

> Markdown-based documentation wiki

## Overview

Wiki.js is a powerful, self-hosted wiki platform for documentation and knowledge management. In this homelab it stores infrastructure notes, runbooks, and configuration documentation. It uses a CloudNative-PG managed PostgreSQL cluster for the database and includes an MCP server sidecar for AI assistant integration.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Wiki.js](https://js.wiki) |
| **Helm Chart** | `wiki` ([js.wiki](https://charts.js.wiki)) |
| **Chart Version** | `2.2.24` |
| **App Version** | `2.5.x` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `wikijs.k8s.firekatt.ca` | Wiki.js web UI (custom ingress template) |
| `wikijs-mcp.k8s.firekatt.ca` | MCP server sidecar |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `postgresql` | — | CloudNative-PG | Managed PostgreSQL cluster (`wikijs-postgres-rw`) |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `wikijs-mcp-api-token` | API token for the MCP server sidecar |

## Notes

- PostgreSQL is provisioned via a `Cluster` resource managed by [CloudNative-PG](../cloudnative-pg/)
- The MCP server sidecar (`mcp-deployment.yaml`) exposes the wiki content over HTTP (port 3200)
- Custom templates handle ingress, MCP deployment/service/ingress, and the PostgreSQL cluster definition
