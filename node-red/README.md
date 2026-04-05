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
