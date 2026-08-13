# Homebridge

> Bridges UniFi Protect cameras into Apple Home with HomeKit Secure Video

## Overview

Homebridge runs the official `homebridge/homebridge` image with the Config UI X admin interface. The `homebridge-unifi-protect` plugin (by hjdhjd) is installed through the UI on first run and exposes the UniFi Protect cameras as HomeKit accessories, including HomeKit Secure Video recording.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Homebridge](https://homebridge.io) |
| **Plugin** | [homebridge-unifi-protect](https://github.com/hjdhjd/homebridge-unifi-protect) |
| **Helm Chart** | `common` ([FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common)) |
| **Chart Version** | `1.0.0` |
| **Common Library** | [FireLabs Helm Common](https://github.com/FireBall1725/firelabs-helm-common) `v5.0.3` |

## Networking

Homebridge needs `hostNetwork: true` so it can emit HAP mDNS/Bonjour (multicast `224.0.0.251:5353`) directly onto the LAN. Apple Home discovers accessories over mDNS, then connects to the advertised HAP port. A pod on the CNI network cannot put multicast on the physical wire, and a ClusterIP or LoadBalancer only forwards unicast to a VIP, so neither carries the advertisement HomeKit needs. This is the same mechanism `matter-server` uses.

Every cluster node sits on the flat `10.0.1.0/23` LAN, the same broadcast domain as the Apple Home hub and the UniFi Protect NVR (`10.0.1.254`), so no VLAN crossing or mDNS reflector is involved. The pod is free to schedule on any node; the only constraint is `kubernetes.io/arch: amd64`, which keeps the HomeKit Secure Video ffmpeg workload off the single arm64 node. HomeKit re-resolves the address over mDNS if the pod reschedules, and the pairing state lives on the PVC, so a move does not break the pairing.

The namespace runs at the `privileged` Pod Security level because the cluster baseline rejects `hostNetwork`.

## Storage

A Longhorn `ReadWriteOnce` PVC mounts at `/homebridge` and holds `config.json`, installed plugins, and the HAP/HKSV pairing state.

## Secrets

The UniFi Protect host and a local Protect admin account are entered through the plugin UI on first run. They persist on the PVC only and are never committed to git. Use a dedicated local Protect user (created under UniFi OS Admins, local access, no SSO, no MFA) with camera view and recordings access.

## First-run setup

1. Open `https://homebridge.k8s.firekatt.ca`.
2. Install the `homebridge-unifi-protect` plugin from the Plugins tab and run it in its own child bridge.
3. Log into Protect with host `10.0.1.254` and the local admin account.
4. Pair the bridge into the Home app, then enable HomeKit Secure Video per camera. HKSV requires an active iCloud+ plan and a HomePod or Apple TV set as a home hub.
