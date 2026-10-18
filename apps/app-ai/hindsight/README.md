# hindsight

> Long-term memory for AI agents (retain, recall, reflect)

## Overview

Hindsight reads conversation text, extracts typed facts, consolidates them into observations, and answers recall queries with semantic, keyword, graph and time search. This chart runs the standalone image (API on `:8888`, control-plane UI on `:9999`) and a CloudNativePG Postgres cluster with pgvector.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [vectorize-io/hindsight](https://github.com/vectorize-io/hindsight) |
| **Image** | `ghcr.io/vectorize-io/hindsight:1.0.0` |
| **Chart Version** | `0.1.0` |
| **App Version** | `1.0.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

Upstream ships a Helm chart, but it is at 1.0.0 with no release and bundles its own Postgres. This one is ours, in the `ollama-admin` shape.

## LLM

Fact extraction runs on `gpt-oss:20b` on the M1 Max, reached through the `ollama` Service in this namespace. Nothing leaves the LAN. `HINDSIGHT_API_LLM_OLLAMA_NUM_CTX` is 65536 to match the server default; any other value makes Ollama reload the model and evict what other apps have loaded. `HINDSIGHT_API_LLM_STRICT_SCHEMA` is on because the soft JSON path wedges retain on a local model. If the Mac is asleep, retain fails and retries.

Embeddings and the reranker run in-process from models baked into the image.

## Postgres

`hindsight-db` is a CNPG cluster on `postgresql:18-minimal-trixie`. The minimal image has no extensions, so pgvector comes from the `pgvector` image volume (`postgresql.extensions`, needs Kubernetes 1.35+). The owner password is a SealedSecret (`hindsight-db-creds`) and the DSN is a second one (`hindsight-db-url`) built from the same password.

## Access

| Host | Target | Auth |
|---|---|---|
| `hindsight.k8s.firekatt.ca` | UI `:9999` | none |
| `hindsight-api.k8s.firekatt.ca` | API and MCP `:8888` | bearer key |

Both are on the `internal` ingress class. The API key is `HINDSIGHT_API_TENANT_API_KEY` (SealedSecret `hindsight-api-key`); the value is in the macOS Keychain as `hindsight-api-key`. Each memory bank has an MCP endpoint at `/mcp/<bank>/`.

## Persistence

None on the pod. All state is in Postgres (10Gi Longhorn).
