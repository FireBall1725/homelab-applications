# ESPHome

> Configuration and management platform for ESP8266/ESP32 IoT devices

## Overview

ESPHome compiles and flashes firmware for ESP-based IoT devices directly from YAML configuration files. This deployment includes the ESPHome web dashboard, a code-server addon for browser-based editing, and a web terminal sidecar with the Claude Code CLI pre-installed.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [ESPHome](https://esphome.io) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2026.2.4` |
| **App Version** | `2026.2.4` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `esphome.k8s.firekatt.ca` | ESPHome web dashboard (port 6052) |
| `esphomecode.k8s.firekatt.ca` | Code server (browser-based editor) |
| `esphometerminal.k8s.firekatt.ca` | Web terminal (ttyd, port 7681) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `10Gi` | `longhorn` | ESPHome device configurations |
| `terminal-home` | `5Gi` | `longhorn` | Terminal home directory |
| `terminal-packages` | ConfigMap | — | Pre-installed packages list |

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `esphome-terminal-anthropic-key` | Anthropic API key for Claude Code CLI in the terminal |

## Notes

- `hostNetwork` is enabled to allow ESPHome to discover devices on the local network
- An init container handles file permission setup before startup
- The terminal sidecar runs a full development environment (zsh + Claude Code CLI)
- Package installation for the terminal is managed via a ConfigMap
