# ha-mcp

> Home Assistant Model Context Protocol (MCP) server for AI assistant integration

## Overview

ha-mcp exposes the Home Assistant API as a Model Context Protocol server, allowing AI assistants (such as Claude) to query and control Home Assistant entities, automations, and services. It acts as a bridge between AI tools and the home automation system.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [ha-mcp](https://github.com/homeassistant-ai/ha-mcp) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `1.0.0` |
| **App Version** | `latest` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `ha-mcp.k8s.firekatt.ca` | MCP server endpoint (internal ingress class) |

## Persistence

No persistent storage. This is a stateless proxy service.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `ha-mcp-secrets` | `HOMEASSISTANT_TOKEN` — long-lived Home Assistant access token |

## Notes

- Uses HTTP transport (`fastmcp-http.json`) to serve the MCP protocol
- Connects to the `home-assistant` service within the cluster
- The ingress uses the `internal` ingress class — not publicly exposed
- Resource limits: 50m CPU, 128Mi memory
