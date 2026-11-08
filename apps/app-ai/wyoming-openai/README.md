# wyoming-openai

> Wyoming text-to-speech bridge for Home Assistant

## Overview

Home Assistant speaks Wyoming, and Kokoro speaks the OpenAI speech API. This app translates between them. It does TTS only.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [roryeckel/wyoming_openai](https://github.com/roryeckel/wyoming_openai) |
| **Image** | `ghcr.io/roryeckel/wyoming_openai:0.9.0` |
| **Chart Version** | `0.1.0` |
| **App Version** | `0.9.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

## Home Assistant

Settings, Devices and services, Add integration, Wyoming Protocol. Host `10.0.1.103`, port `10300`. Home Assistant then offers a text-to-speech entity named `kokoro`; pick a voice in the Assist pipeline.

Home Assistant runs with `hostNetwork` and cannot resolve cluster Service names, so the bridge has a MetalLB address (`10.0.1.103`, from `service-pool`) instead of a ClusterIP.

## Configuration

| Variable | Value | Why |
|---|---|---|
| `TTS_OPENAI_URL` | `http://kokoro:8880/v1` | The `kokoro` Service in this namespace |
| `TTS_MODELS` | `kokoro` | Model name Kokoro accepts |
| `TTS_STREAMING_MODELS` | `kokoro` | Sentence-chunked playback |
| `TTS_CONCURRENT_REQUESTS` | `1` | Kokoro serves one request at a time |
| `TTS_BACKEND` | `KOKORO_FASTAPI` | Backend-specific voice handling |
| `TTS_VOICES` | English voices | Keeps the picker to voices that match `WYOMING_LANGUAGES=en` |

Changing the engine behind `kokoro` means updating `TTS_BACKEND`, `TTS_MODELS` and `TTS_OPENAI_URL` here.
