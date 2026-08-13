# Basic Memory

> Self-hosted AI memory MCP server backed by a Markdown knowledge base and a SQLite index

## Overview

Basic Memory stores knowledge as plain Markdown files and exposes it to AI assistants over the Model Context Protocol. The assistant reads and writes notes through MCP tools; the notes stay as human-readable Markdown on disk, so the memory is inspectable and portable outside the tool. It runs as a single pod with the default SQLite backend (no Postgres or pgvector).

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Basic Memory](https://github.com/basicmachines-co/basic-memory) |
| **Docs** | [docs.basicmemory.com](https://docs.basicmemory.com/reference/docker) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `1.0.0` |
| **App Version** | `0.22.1` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `basic-memory.k8s.firekatt.ca` | MCP server (`/mcp`, SSE transport), internal ingress class |

## MCP Endpoint

- The container runs `basic-memory mcp --transport sse --host 0.0.0.0 --port 8000` and serves the MCP protocol at `/mcp` on port 8000.
- Transport is **SSE**, not streamableHttp. streamableHttp answers the initialize POST as an SSE stream, and Claude Code reuses that connection for the follow-up notification, which the cluster Traefik path turns into a 400. The same failure hit the node-red MCP sidecar. SSE keeps the long-lived stream on a dedicated request and avoids the pattern.
- The endpoint has **no authentication**. The ingress is on the `internal` class only, so it is reachable on the LAN (or Tailscale), never from the public internet.
- Health for the Uptime Kuma monitor is served by a sidecar at `/health` (see below), because Basic Memory itself ships no plain HTTP health route.

### Adding to Claude Code

```
claude mcp add --transport sse -s user basic-memory https://basic-memory.k8s.firekatt.ca/mcp
```

The client machine has to be on the LAN (or Tailscale) since the host resolves on the LAN only.

## Persistence

| Volume | Mount | Size | Storage Class | Notes |
|--------|-------|------|---------------|-------|
| `data` | `/app/data` | `5Gi` | `longhorn` | Markdown knowledge base (the memory content) |
| `config` | `/home/appuser/.basic-memory` | `2Gi` | `longhorn` | App config and SQLite index (rebuildable from Markdown) |

Both PVCs are `ReadWriteOnce`. The pod runs as the non-root `appuser` (UID/GID 1000); `fsGroup: 1000` makes the volumes group-writable so no init-container chown is needed.

## Health Sidecar

Basic Memory serves `/mcp` (which needs MCP headers) and no plain health route, so the default kuma-ingress-watcher root probe never sees a 200. A small stdlib-Python sidecar serves `GET /health` on port 8099 and returns 200 only when it can open a TCP connection to the Basic Memory listener on `127.0.0.1:8000`, else 503. The `/health` ingress path targets that sidecar through a dedicated `basic-memory-health` service, and the `uptime-kuma.autodiscovery.probe.path` annotation points the auto-created monitor at it.

## Secrets

No secrets required. The endpoint is unauthenticated and gated only by the internal ingress class.

## Notes

- Single replica, `Recreate` strategy. The SQLite backend writes to a `ReadWriteOnce` PVC, so the server is a singleton and must not scale above 1.
- Image tag is pinned to `0.22.1` (Renovate tracks the ghcr repository for updates).
- Liveness, readiness, and startup probes are TCP checks on port 8000, since the image exposes no HTTP health route. The startup probe is generous for the first-boot index build and Markdown sync.
- Timezone set to `America/Toronto`.
