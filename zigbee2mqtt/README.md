# Zigbee2MQTT

> Zigbee to MQTT bridge for smart home devices

## Overview

Zigbee2MQTT connects a Zigbee coordinator USB dongle (ConBee II) to the Mosquitto MQTT broker, exposing all paired Zigbee devices as standard MQTT topics. This allows Home Assistant and Node-RED to control and receive state from Zigbee devices without any proprietary bridge hardware or cloud dependency.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Zigbee2MQTT](https://www.zigbee2mqtt.io) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2.4.0` |
| **App Version** | `2.4.0` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

| Host | Description |
|------|-------------|
| `zigbee2mqtt.k8s.firekatt.ca` | Zigbee2MQTT web UI (port 8080) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | `10Gi` | `longhorn` | Device configuration, pairing data, and logs |

## Secrets

No secrets required.

## Notes

- Requires USB access to the ConBee II Zigbee coordinator dongle
- Pod is pinned to a specific node via `nodeAffinity` where the USB dongle is physically attached
- A taint toleration (`zwave-zigbee=only`) is applied to ensure scheduling on the correct node
- Runs as privileged to access the USB device via `hostPath`
