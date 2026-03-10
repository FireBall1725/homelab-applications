# Ollama

> Kubernetes service pointing to an external Ollama LLM inference server

## Overview

This chart does not deploy Ollama itself — it creates a Kubernetes `Service` and `Endpoints` resource that points to an Ollama instance running on dedicated external hardware (a machine on the local network). This allows in-cluster applications like Open WebUI to reference Ollama via a stable internal service name.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Ollama](https://ollama.ai) |
| **Helm Chart** | Custom (external service bridge) |
| **Chart Version** | `1.0.0` |
| **App Version** | `1.0.0` |
| **Common Library** | Custom |

## Ingress

No HTTP ingress.

## Persistence

No persistent storage — models are stored on the external Ollama host.

## Secrets

No secrets required.

## Notes

- Deploys a headless `Service` + `Endpoints` pointing to an external host on the local network (port 11434)
- Allows cluster workloads to reach Ollama via `http://ollama.<namespace>.svc.cluster.local:11434`
- Used by [Open WebUI](../open-webui/)
