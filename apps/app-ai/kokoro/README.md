# kokoro

> CPU text-to-speech with an OpenAI-compatible API

## Overview

Runs Kokoro-FastAPI (CPU image). It serves `POST /v1/audio/speech` on `:8880`. Kisuke calls it directly, and the `wyoming-openai` app in this namespace exposes it to Home Assistant over the Wyoming protocol.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [remsky/Kokoro-FastAPI](https://github.com/remsky/Kokoro-FastAPI) |
| **Image** | `ghcr.io/remsky/kokoro-fastapi-cpu:v0.9.2` |
| **Chart Version** | `0.1.0` |
| **App Version** | `v0.9.2` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Access

| From | URL |
|---|---|
| Cluster | `http://kokoro.app-ai.svc.cluster.local:8880/v1` |
| LAN (Kisuke) | `https://tts.k8s.firekatt.ca/v1` |

No API key is set (`API_KEYS` is unset). Both routes are LAN-trust. The ingress is on the `internal` class.

```bash
curl -s https://tts.k8s.firekatt.ca/v1/audio/speech \
  -H 'Content-Type: application/json' \
  -d '{"model":"kokoro","input":"Hello from Kisuke.","voice":"af_heart","response_format":"mp3"}' \
  -o test.mp3
```

`/v1/audio/voices` lists voices. Model names `kokoro`, `tts-1`, `tts-1-hd` and `gpt-4o-mini-tts` all map to the one Kokoro model. The OpenAI voice names (`alloy`, `nova`, and so on) map to Kokoro voices.

## Persistence

None. The model and voice packs are baked into the image and `DOWNLOAD_MODEL=false` stops the start-time fetch, so a restart does not download anything. Upgrading the image tag replaces the model with the one in that image.

## Resources

Requests 2 CPU and 3Gi, limits 6 CPU and 6Gi. Inference is single-process and synchronous, so a second request waits for the first.

## Swapping the engine

Keep the `kokoro` Service and the `tts.k8s.firekatt.ca` host. A replacement only has to answer `POST /v1/audio/speech` and `GET /v1/audio/voices`. Update `TTS_BACKEND`, `TTS_MODELS` and `TTS_OPENAI_URL` in `wyoming-openai` if the new engine's names differ.
