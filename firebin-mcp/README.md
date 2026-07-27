# firebin-mcp

> Model Context Protocol server for the self-hosted FireBin electronics parts inventory

## Overview

firebin-mcp exposes the FireBin API as an MCP server over streamable HTTP, so MCP-aware clients (Claude Code, Claude Desktop, Cursor) can search parts, check what is in a bin, look up distributor pricing, book stock in, and work out whether a board can be built. It is another API client with no special backend access, and runs alongside `firebin-api` and `firebin-web` in `app-firebin`.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [firebin-mcp](https://github.com/fireball1725/firebin-mcp) |
| **Chart Version** | `0.1.0` |
| **App Version** | `26.7.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Ingress

| Host | Description |
|------|-------------|
| `firebin-mcp.k8s.firekatt.ca` | FireBin MCP server (internal LAN only) |

The `*.k8s.firekatt.ca` zone resolves inside the LAN only. `/mcp` is bearer-auth gated; `/health` is unauthenticated for probes.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `firebin-mcp-data` | 1Gi | `longhorn` | Holds the inbound MCP bearer token (RWO) |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents | Source |
|--------|----------|--------|
| `firebin-mcp-access-token` | `FIREBIN_ACCESS_TOKEN`, the outbound `fbin_pat_` PAT | SealedSecret (`templates/sealed-secret-access-token.yaml`) |
| `firebin-mcp-token` | `FIREBIN_MCP_TOKEN`, the inbound bearer clients present on `/mcp` | SealedSecret (`templates/sealed-secret-mcp-token.yaml`) |

## Notes

- `FIREBIN_API_URL` points at the in-cluster `firebin-api:8080` service, so the chart MUST deploy into the same namespace as `firebin-api` (`app-firebin`) for the bare service name to resolve. Do not route through `firebin-web`: its nginx proxies `/api` for the browser and adds a hop for no benefit.
- The outbound PAT carries the full authority of the FireBin account that minted it. `firebin-api` stores per-token scopes but no middleware reads them, so the account's role is the only limit. This token belongs to a dedicated `mcp` user with role `member`, which leaves `RequireAdmin` blocking `/users`, `/settings/*`, `/export`, `/import` and `/stock/cleanup-empty`.
- Both tokens are set explicitly rather than letting the server generate the inbound one on first run, so the value survives a lost PVC and is known before the first boot.
- To rotate either token, re-seal it per [SECRETS.md](../SECRETS.md) and restart the pod. Rotating the outbound PAT also means revoking the old one in the FireBin web UI under Settings, API tokens.
