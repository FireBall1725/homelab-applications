# Basic Memory Viewer

> Read-only web UI for browsing and searching the Basic Memory knowledge base

## Overview

Basic Memory stores notes as Markdown and exposes them over MCP, which is convenient for an assistant and awkward for a human. This viewer is a server-rendered FastAPI app that opens a short-lived MCP session per page and turns the same tools (`recent_activity`, `read_note`, `search_notes`, `list_memory_projects`) into a browsable UI: a recent feed grouped by day, live search, rendered Markdown, and a lazy-loaded Related panel driven by semantic search.

It writes nothing. Every route is a GET, and the app holds no state beyond an in-memory description cache.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [basic-memory-viewer](https://github.com/manelpb/basic-memory-viewer) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `1.0.0` |
| **App Version** | `49842aa` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `basic-memory-viewer.k8s.firekatt.ca` | Web UI, internal ingress class |

The viewer has no authentication and renders the whole knowledge base, so it stays on the `internal` ingress class, same as Basic Memory itself. Reachable on the LAN or over Tailscale, never from the public internet.

## MCP Connection

`MCP_URL` points at the in-cluster Service, `http://basic-memory.app-ai.svc.cluster.local:8000/mcp`, rather than back out through Traefik at `basic-memory.k8s.firekatt.ca`. The traffic stays inside the namespace and does not depend on ingress routing or LAN DNS.

`MCP_TRANSPORT` is `sse`. The viewer defaults to streamable HTTP, but our Basic Memory container runs `basic-memory mcp --transport sse`, so the default would fail to connect.

## Image Tag

Upstream CI publishes one tag per build, the short commit SHA (`type=sha,prefix=,format=short` in `.github/workflows/build.yml`). The `latest` tag still sitting on the registry predates that change and no longer moves, so it is not a useful pointer. The chart pins `49842aa`, the commit that added the `MCP_TRANSPORT` switch this deployment depends on. Renovate tracks the ghcr repository.

## NOTES_DIR

Not set, on purpose.

`NOTES_DIR` points the viewer at the Markdown files on disk. It buys two things: card descriptions read from frontmatter instead of a `read_note` fan-out over MCP, and the git-backed file-history panel. Both require mounting the same volume Basic Memory writes to.

That volume is `basic-memory-data`, a Longhorn `ReadWriteOnce` PVC. A second pod can attach it only if it happens to schedule onto the same node, and if it schedules anywhere else the pod hangs waiting for an attachment that cannot happen. Switching the PVC to `ReadWriteMany` means recreating the volume that holds the actual memory content. Neither risk is worth a description-loading shortcut, so the feature stays off: descriptions resolve over MCP and the history panel is hidden.

## Persistence

None. The root filesystem is read-only and the only mount is an `emptyDir` at `/tmp` for scratch space.

## Health

| Route | Behaviour |
|-------|-----------|
| `/livez` | 200 whenever uvicorn is serving |
| `/healthz` | 200 when an MCP session opens, 503 when it does not |

Liveness uses `/livez` so a Basic Memory outage cannot restart the viewer. Readiness uses `/healthz` so the pod drops out of the Service endpoints while its only dependency is down. The `uptime-kuma.autodiscovery.probe.path` annotation points the auto-created monitor at `/healthz`, which alerts on a broken MCP link and not just a missing pod.

## Secrets

None. The endpoint is unauthenticated and gated only by the internal ingress class.

## Notes

- Runs as the non-root `appuser` (UID/GID 10001) baked into the upstream image, with `readOnlyRootFilesystem: true` and all capabilities dropped.
- No init container. Basic Memory needs one because it chmods its own config dir at startup; the viewer touches no volume and has no such problem.
- Single replica with a `RollingUpdate` strategy. The app is stateless, so overlapping pods during a rollout are harmless.
- Container listens on port 8000 (the Dockerfile `CMD` runs uvicorn on 8000).
- Timezone set to `America/Toronto`.
