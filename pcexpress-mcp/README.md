# pcexpress-mcp

> Model Context Protocol server for PC Express grocery ordering (Zehrs/Loblaws banners)

## Overview

pcexpress-mcp exposes the PC Express (Zehrs/Loblaws) grocery API as an MCP server over SSE (`/sse`, posts to `/messages/`), so MCP-aware clients (Claude Desktop, Cursor, Claude Code) can browse and order groceries. It authenticates to PC Express with a rotating OAuth refresh token and runs in its own namespace (`app-pcexpress`).

## Chart Details

| | |
|---|---|
| **Upstream Project** | [pcexpress-mcp-server](https://github.com/fireball1725/pcexpress-mcp-server) |
| **Chart Version** | `0.1.0` |
| **App Version** | `0.1.0` (placeholder until the first image publishes) |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Ingress

| Host | Description |
|------|-------------|
| `pcexpress-mcp.k8s.firekatt.ca` | PC Express MCP server (internal LAN only) |

The `*.k8s.firekatt.ca` zone resolves inside the LAN only. `/health` is unauthenticated for probes; the SSE endpoint can be bearer-gated via `PCEXPRESS_MCP_BEARER`.

## Single-instance constraint

`replicas: 1`, `strategy: Recreate`. The PC Express OAuth refresh token is single-use and rotates on every refresh. Two pods would consume each other's rotated tokens and one would die with `invalid_grant`. Do not scale up and do not switch to RollingUpdate.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `pcexpress-mcp-data` | 32Mi | `longhorn` | Rotating OAuth refresh token at `/data` (RWO). Must persist across restarts. |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents | Source |
|--------|----------|--------|
| `pcexpress-mcp` | `PCEXPRESS_REFRESH_TOKEN`, `PCEXPRESS_BANNER`, `PCEXPRESS_STORE_ID`, optional `PCEXPRESS_MCP_BEARER` | SealedSecret (`templates/sealed-secret-refresh-token.yaml`) |

The app client secret is baked into the image (not stored here). The cart id is auto-discovered by the server (not stored here).

## Before this can deploy

1. Merge pcexpress-mcp-server PR #3 (adds CI) and push a version tag so CI publishes `ghcr.io/fireball1725/pcexpress-mcp:<tag>`.
2. Run `login_pcid.py` once to mint a refresh token dedicated to this cluster, then `kubeseal` it (plus banner/store id) into `templates/sealed-secret-refresh-token.yaml`, replacing the PLACEHOLDER values.
3. Set the real image tag in `values.yaml` (`image.tag`).

## Notes

- `PCEXPRESS_STATE_DIR` points at the `/data` PVC where the rotating refresh token is stored.
- The refresh token rotates on every refresh; the seed token is single-use. Losing the PVC means re-seeding via `login_pcid.py`.
