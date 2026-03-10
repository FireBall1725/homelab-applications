# ttyd

> Web-based terminal with Claude Code CLI

## Overview

ttyd provides a browser-accessible terminal interface backed by a persistent home directory. It runs zsh with the Claude Code CLI pre-installed, giving full shell and AI-assisted access to the cluster from any browser. RBAC grants the pod cluster-admin permissions so it can interact with Kubernetes directly.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [ttyd](https://github.com/tsl0922/ttyd) |
| **Helm Chart** | Custom (no upstream dependency) |
| **Chart Version** | `1.7.7` |
| **App Version** | `1.7.7` |
| **Common Library** | Custom |

## Ingress

| Host | Description |
|------|-------------|
| `terminal.k8s.firekatt.ca` | Web terminal (ttyd, internal ingress class) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `home` | `10Gi` | `longhorn` | Persistent home directory (shell history, dotfiles, projects) |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `ttyd-auth` | Basic auth credentials (admin user password) |
| `ttyd-anthropic-key` | Anthropic API key for Claude Code CLI |

## Notes

- Maximum 5 concurrent client connections
- Shell: `zsh` with `xterm-256color`
- Package installation is managed via a ConfigMap (`configmap-packages.yaml`)
- Terminal theme is configurable via a ConfigMap (`configmap-theme.yaml`)
- RBAC grants `cluster-admin` — treat access credentials with care
- Ingress uses the `internal` ingress class — not publicly exposed
