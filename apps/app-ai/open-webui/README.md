# Open WebUI

> Web interface for interacting with local LLMs via Ollama

## Overview

Open WebUI provides a ChatGPT-like browser interface for interacting with large language models served by Ollama. It supports multiple models, conversation history, and system prompt configuration. In this homelab it connects to an Ollama instance running on dedicated hardware.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Open WebUI](https://openwebui.com) |
| **Helm Chart** | `open-webui` ([Open WebUI](https://helm.openwebui.com/)) |
| **Chart Version** | `12.8.1` |
| **App Version** | `0.8.8` |
| **Common Library** | Upstream chart |

## Ingress

| Host | Description |
|------|-------------|
| `open-webui.k8s.firekatt.ca` | Open WebUI (internal ingress class) |

## Persistence

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `data` | `2Gi` | `longhorn` | Conversation history and user settings |

## Secrets

No secrets required.

## Notes

- Connects to the `ollama` service within the cluster (which points to an external Ollama instance)
- Memory limit set to 2Gi after OOMKill incidents; requests set to 512Mi
- Pipelines and websocket features are disabled
- Ingress uses the `internal` ingress class — not publicly exposed
