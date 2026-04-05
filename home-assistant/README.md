# Home Assistant

> Open-source home automation platform

## Overview

Home Assistant is the central smart home hub, integrating with hundreds of devices and services. It handles automations, scenes, dashboards, and device state management for the entire home. A code-server addon is enabled for direct configuration editing from the browser.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Home Assistant](https://www.home-assistant.io) |
| **Helm Chart** | `home-assistant` ([pajikos](http://pajikos.github.io/home-assistant-helm-chart/)) |
| **Chart Version** | `0.3.32` |
| **App Version** | `2026.3.0` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `ha.k8s.firekatt.ca` | Home Assistant web UI |
| `hacode.k8s.firekatt.ca` | Code server (browser-based config editor) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `config` | `100Gi` | `longhorn` | All Home Assistant configuration and data |

## Secrets

No secrets required at the Helm level. Home Assistant credentials and integration tokens are managed within the application.

## Notes

- `hostNetwork` is enabled for local network device discovery (mDNS, etc.)
- Trusted proxy configuration is set for Traefik reverse proxy
- Code server addon is enabled for in-browser YAML editing
- Configuration is templated and managed within the config volume
