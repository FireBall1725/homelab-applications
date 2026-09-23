# Openclaw

> Multi-channel AI agent gateway

## Overview

Openclaw is an AI agent that answers in Discord and can use tools, a headless browser and web fetch. It runs on the local Ollama host through the `ollama` Service in `app-ai`, so chats don't use any paid API.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Openclaw](https://github.com/openclaw/openclaw) |
| **Helm Chart** | `openclaw` ([serhanekicii](https://serhanekicii.github.io/openclaw-helm), archived 2026-05-24) |
| **Chart Version** | `1.5.40` |
| **App Version** | `2026.9.5` (set by `openclawVersion`, the chart default is 2026.5.22) |
| **Common Library** | bjw-s app-template 4.6.2 |

## Ingress

| Host | Description |
|------|-------------|
| `openclaw.k8s.firekatt.ca` | Openclaw API/UI (internal ingress class) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | 5Gi | `longhorn` | Mounted at `/home/node/.openclaw`: config, SQLite state, sessions, the npm-installed Discord plugin |
| `home` | n/a | emptyDir | `/home/node`, so plugin installs can write to `$HOME` under a read-only root |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `openclaw-secrets` | `DISCORD_BOT_TOKEN`, `OPENCLAW_GATEWAY_TOKEN`, `ANTHROPIC_API_KEY` (unused since the switch to Ollama) |

## Model

`ollama/qwen3.6:latest`, a 36B MoE with about 3B active per token. On the M1 Max it reads prompts at about 600 tok/s and writes at about 52 tok/s. The first message after a load takes 30 to 45 s. After that, Ollama reuses the cached prompt and replies come back in about 2 s. The provider sends `num_ctx: 65536` (the Ollama server default, so other clients share the loaded model), `keep_alive: -1` so the model stays in memory, and `think: false`. There's no cloud fallback.

## Notes

- `configMode: overwrite`: `openclaw.json` comes from this chart on every start, and edits made in the UI are lost on restart
- The `init-doctor` init container runs `openclaw doctor --fix --non-interactive` before the gateway. 2.0 exits with code 78 on a pre-2.0 session store until doctor migrates it to SQLite, and doctor also installs the Discord plugin from npm, so the pod needs npm registry egress
- The migration is one-way. Snapshot the `openclaw` PVC before moving to a new major version
- The admin terminal in the control UI is off (`gateway.terminal.enabled: false`); it would inherit the gateway's environment, secrets included
- Discord answers DMs from FireBall's user ID only (`dmPolicy: allowlist`) and ignores every server (`groupPolicy: disabled`). The bot has shell, browser and fetch tools, so nobody else gets to talk to it
- Sessions reset after 60 minutes of inactivity
- Ingress uses the `internal` ingress class, so it isn't publicly exposed
