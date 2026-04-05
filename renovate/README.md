# Renovate

> Self-hosted Renovate Bot for automated dependency updates

## Overview

Renovate Bot automatically opens pull requests to update Helm chart versions, container image tags, and other dependency versions across the homelab repositories. It runs on an hourly schedule via a CronJob, scanning the configured repositories and raising PRs when newer versions are available.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Renovate](https://github.com/renovatebot/renovate) |
| **Helm Chart** | Custom (no upstream dependency) |
| **Chart Version** | `1.0.0` |
| **App Version** | `43.59.2` |
| **Common Library** | Custom |

## Ingress

No HTTP ingress. Runs as a CronJob only.

## Persistence

No persistent storage.

## Secrets

Uses [Sealed Secrets](../SECRETS.md) for sensitive values.

| Secret | Contents |
|--------|----------|
| `renovate-secrets` | `RENOVATE_TOKEN` — GitHub personal access token for opening PRs |

## Notes

- Runs hourly via a Kubernetes `CronJob` (`0 * * * *`)
- Scans the `homelab-applications` and `homelab-config` GitHub repositories
- PR assignments are sent to the configured GitHub user
- Renovate configuration (ignore paths, rules, etc.) is managed in `renovate.json` at the repo root
- The `legacy/` directory is excluded from Renovate scans
