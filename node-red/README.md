# Node-RED

> Flow-based automation and IoT logic engine

## Overview

Node-RED provides a browser-based visual editor for wiring together automation flows. In this homelab it acts as the primary automation engine, consuming MQTT messages from Mosquitto (Zigbee2MQTT, ESPHome devices) and triggering actions in Home Assistant, sending notifications, and orchestrating multi-step automations.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Node-RED](https://nodered.org) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `4.1.6` |
| **App Version** | `4.1.6` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `nodered.k8s.firekatt.ca` | Node-RED editor and dashboard |
| `node-red-mcp.k8s.firekatt.ca` | MCP server (`/mcp`, streamableHttp), internal class |

## MCP Sidecar

A `node-red-mcp` sidecar runs in the Node-RED pod and exposes the Node-RED Admin API to MCP clients.

- The upstream server ([node-red-mcp-server](https://github.com/karavaev-evgeniy/node-red-mcp-server)) is stdio-only and ships no image, so [supergateway](https://github.com/supercorp-ai/supergateway) (`supercorp/supergateway:3.4.3`) wraps it and bridges stdio to a streamableHttp listener on port 8000.
- supergateway runs in `--stateful` mode. Claude Code follows the full streamableHttp session lifecycle, which the default stateless mode does not satisfy (it answers `initialize` but the connection is then marked failed). `--sessionTimeout` reaps idle sessions. The pod is single-replica, so sessions have no affinity concern.
- supergateway runs `npx -y node-red-mcp-server@1.0.2` at startup, so the package is pulled on first boot. The startup probe is sized to allow for that.
- The MCP server talks to Node-RED over the Admin API at `http://localhost:1880` (same pod). Node-RED `adminAuth` is disabled, so no token is set. If `adminAuth` is enabled later, add `--token $NODE_RED_TOKEN` to the sidecar args and source the token from a SealedSecret.
- Endpoint: `https://node-red-mcp.k8s.firekatt.ca/mcp`. Health: `/healthz` (unauthenticated, used by probes).
- The `/mcp` endpoint has no authentication. The ingress is on the `internal` class only.
- A Traefik `Middleware` (`node-red-mcp-accept`, in `templates/mcp-accept-middleware.yaml`) forces the `Accept: application/json, text/event-stream` request header on this ingress. supergateway's streamableHttp transport requires both media types; Claude Code's `http` transport currently omits the header ([claude-agent-sdk #202](https://github.com/anthropics/claude-agent-sdk-typescript/issues/202)), so without it the server returns 406, surfaced as a 400 by the cluster error-pages middleware. Drop the middleware once that client bug is fixed upstream.

### Adding to Claude Code

```
claude mcp add --transport http -s user node-red http://node-red-mcp.k8s.firekatt.ca/mcp
```

The client machine has to be on the LAN (or Tailscale) since the host resolves on the LAN only.

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | `10Gi` | `longhorn` | Flows, credentials, and installed nodes |

## Secrets

No secrets required. Credentials for external services are managed within Node-RED's encrypted credential store.

## Notes

- Projects feature is enabled, allowing flow versioning via git
- An init container sets correct file ownership (UID 1000) before startup
- Timezone set to `America/Toronto`
- A `node-red-mcp` sidecar exposes the Admin API over MCP (see above)
