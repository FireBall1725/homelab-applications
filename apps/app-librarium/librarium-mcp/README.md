# librarium-mcp

> Model Context Protocol server for the self-hosted Librarium library tracker

## Overview

Librarium-mcp exposes the Librarium API as an MCP server over streamable HTTP, so MCP-aware clients (Claude Desktop, Cursor, Claude Code) can chat with your library. It is just another API client — no special backend access — and runs alongside `librarium-api` and `librarium-web` in `app-librarium`.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [librarium-mcp](https://github.com/fireball1725/librarium-mcp) |
| **Chart Version** | `0.1.0` |
| **App Version** | `26.4.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Ingress

| Host | Description |
|------|-------------|
| `librarium-mcp.k8s.firekatt.ca` | Librarium MCP server (internal LAN only) |

The `*.k8s.firekatt.ca` zone resolves inside the LAN only. The `/mcp` endpoint is bearer-auth gated; `/health` is unauthenticated for probes.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `librarium-mcp-data` | 1Gi | `longhorn` | Persisted inbound MCP bearer token (RWO) |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents | Source |
|--------|----------|--------|
| `librarium-mcp-access-token` | `LIBRARIUM_ACCESS_TOKEN` (PAT for librarium-api) | SealedSecret (`templates/sealed-secret-access-token.yaml`) |

## Notes

- `LIBRARIUM_API_URL` points at the in-cluster `librarium-api:8080` service. The chart MUST deploy into the same namespace as `librarium-api` (`app-librarium`) for the bare service name to resolve.
- The inbound MCP bearer token is auto-generated on first run and persisted to `/data/mcp-token`. Find it in the container's startup banner: `kubectl -n app-librarium logs deploy/librarium-mcp`.
- To rotate the inbound token, delete the file and restart the pod.
