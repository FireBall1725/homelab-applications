# Mosquitto

> MQTT message broker for IoT and home automation

## Overview

Eclipse Mosquitto is a lightweight MQTT broker used as the messaging backbone for home automation. All Zigbee2MQTT messages, ESPHome sensor readings, and other IoT device events are routed through Mosquitto. Node-RED subscribes to topics from Mosquitto to drive automation logic.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Eclipse Mosquitto](https://mosquitto.org) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `2.0.20` |
| **App Version** | `2.0.20` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Ingress

No HTTP ingress. Mosquitto is exposed via a `LoadBalancer` service on a fixed internal IP (port 1883).

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | `10Gi` | `longhorn` | MQTT message persistence store |

## Secrets

No secrets required.

## Notes

- Broker configuration is managed via a ConfigMap (`configmap.yaml`)
- Anonymous authentication is disabled; clients must authenticate
- Exposed via LoadBalancer with a fixed internal IP for direct device connectivity
- Message persistence is enabled so in-flight messages survive restarts
