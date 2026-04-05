# Openclaw

> Multi-channel AI agent gateway

## Overview

Openclaw is an AI agent framework that exposes Claude as a conversational assistant across multiple channels, including Discord. It integrates with the Claude API to provide an AI assistant accessible from within the homelab Discord server.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Openclaw](https://github.com/serhanekicii/openclaw) |
| **Helm Chart** | `openclaw` ([serhanekicii](https://serhanekicii.github.io/openclaw-helm)) |
| **Chart Version** | `1.3.23` |
| **App Version** | `2026.2.24` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `openclaw.k8s.firekatt.ca` | Openclaw API/UI (internal ingress class) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | — | `longhorn` | Session state and configuration |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `openclaw-secrets` | `DISCORD_BOT_TOKEN` — Discord bot authentication token |

## Notes

- Configured to use Claude Sonnet 4.6 as the primary model
- Sessions reset after 60 minutes of inactivity
- Browser/CDP support is enabled
- Ingress uses the `internal` ingress class — not publicly exposed
