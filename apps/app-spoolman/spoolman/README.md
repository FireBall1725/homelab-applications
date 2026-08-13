# spoolman

> Filament spool inventory for 3D printing

## Overview

Spoolman tracks filament spools: weight remaining, material, vendor, cost, and consumption per print. It exposes a REST API that other tools write to, so it works as the inventory backend for [`bambuddy`](../bambuddy/) (AMS slot sync and per-print usage), Home Assistant, and Moonraker/Klipper. This chart deploys the app and its CloudNative-PG managed PostgreSQL cluster.

## Chart Details

| | |
|---|---|
| **Upstream Project** | [Spoolman](https://github.com/Donkie/Spoolman) |
| **Image** | `ghcr.io/donkie/spoolman:0.26.0` (public, amd64/arm64/armv7) |
| **Chart Version** | `0.1.0` |
| **App Version** | `0.26.0` |
| **Common Library** | `common` 5.0.3 ([firelabs-helm-common](https://fireball1725.github.io/firelabs-helm-common/)) |

Image tags carry no leading `v` even though the upstream git tags do: git `v0.26.0` publishes as image `0.26.0`.

## The database, and why this chart has no sealed secrets

`bambuddy` has to hand-seal its Postgres credentials. Spoolman does not, and the reason is worth writing down because it decides the whole shape of the chart.

Spoolman takes **discrete database fields**, not a connection URL:

```
SPOOLMAN_DB_TYPE  SPOOLMAN_DB_HOST  SPOOLMAN_DB_PORT
SPOOLMAN_DB_NAME  SPOOLMAN_DB_USERNAME  SPOOLMAN_DB_PASSWORD
```

`spoolman/database/database.py` then builds the connection with SQLAlchemy's `URL.create(drivername=..., host=..., port=..., database=..., username=..., password=...)`. `URL.create` takes each component as a raw value and escapes it internally; it never parses a URL string. A password containing symbols is therefore safe.

That removes both problems that forced sealed credentials in `bambuddy`: there is no DSN scheme prefix to bolt on, and no percent-encoding hazard. So:

- The operator-generated `spoolman-db-app` secret wires straight in through five `secretKeyRef` entries.
- `templates/cnpg-cluster.yaml` is a **verbatim copy** of `firebin-api/templates/cnpg-cluster.yaml`, with no `bootstrap.initdb.secret` extension.
- There are no `sealed-secret-*.yaml` templates at all.

The CNPG `<cluster>-app` secret supplies exactly the keys needed: `host` (which points at the `-rw` service), `port`, `dbname`, `username`, `password`.

Internally the driver still resolves to `postgresql+asyncpg`, the same as Bambuddy. The difference is only who assembles the URL.

**Migrations run themselves.** `spoolman/main.py` executes `alembic upgrade head` in a subprocess during FastAPI lifespan startup, so there is no init container and no Job. Upstream notes it has to be a subprocess because running alembic in-process hangs under the uvicorn worker.

## The port is 8000

`Dockerfile` has `EXPOSE 8000` and `entrypoint.sh` defaults `SPOOLMAN_PORT=8000`. The `7912` that appears throughout upstream documentation is only the conventional *host* port in their compose file. Do not set `SPOOLMAN_PORT`.

## enableServiceLinks is false on purpose

Kubernetes injects a Docker-style service link variable for every Service in the namespace, so a Service named `spoolman` produces `SPOOLMAN_PORT=tcp://<clusterIP>:8000` and shadows the app's own port variable. Upstream's `entrypoint.sh` names this case in a comment and recovers by resetting to 8000 with a warning.

Setting `enableServiceLinks: false` avoids the collision instead of surviving it, and also removes the risk that a future Service named `spoolman-db` clobbers `SPOOLMAN_DB_PORT`.

## Host checking

`SPOOLMAN_ALLOWED_HOSTS` is set:

```
spoolman.k8s.firekatt.ca,spoolman.app-spoolman
```

It is a comma-separated list of **hostnames**: no scheme, no port. A leading `*.` matches a domain and its subdomains. Once the variable names at least one host, `spoolman/security.py` installs `TrustedHostMiddleware` and every request whose `Host` fails the check gets a hard **400**.

`_is_allowed_hostname` allows, without listing: an explicit match, an IP literal, a single-label name, and anything ending in `.local`, `.localhost`, `.lan`, `.home`, `.home.arpa` or `.internal`. Everything else is refused.

Both entries above are load-bearing:

- `spoolman.k8s.firekatt.ca` is a registrable domain, so it must be listed.
- `spoolman.app-spoolman` is the in-cluster short name Bambuddy uses. It contains a dot, is not an IP, and does not end in a local suffix. **Omitting it breaks the Bambuddy integration with a 400** that looks nothing like a name-resolution problem.

The `.svc.cluster.local` FQDN happens to be auto-allowed because it ends in `.local`, but do not rely on that.

Unaffected: kubelet probes and Prometheus scrapes address the pod IP, and IP literals always pass. Traefik forwards both `Host` and `X-Forwarded-Host`, and the middleware checks both.

**If the ingress host ever changes, change this variable in the same commit.**

`SPOOLMAN_CORS_ORIGIN` is left unset. It is only needed for a browser UI served from a different host, such as Fluidd or Mainsail. Bambuddy, Home Assistant and Moonraker are server-side and send no `Origin`. Setting it to `*` would switch host checking off as a side effect, since `is_host_checking_enabled()` returns False when origin checks are disabled.

## Persistence

None, deliberately. With `SPOOLMAN_DB_TYPE=postgres` there is no durable state on disk:

| Path | Why it does not need a volume |
|---|---|
| `SPOOLMAN_DIR_DATA` | only holds `spoolman.db`, which is never created |
| `SPOOLMAN_DIR_BACKUPS` | the nightly backup and `POST /api/v1/backup` are SQLite-only; `Database.backup()` no-ops when `is_file_based_sqlite()` is False |
| `SPOOLMAN_DIR_LOGS` | logs also go to stdout through a console handler |

| Volume | Size | Storage Class | Notes |
|--------|------|---------------|-------|
| `spoolman-db` (CNPG) | 5Gi | `longhorn` | PostgreSQL 17.5-3, the only durable state |

The pod can move nodes freely as a result.

## Consumers

**Bambuddy.** Settings, Filament, Spoolman card. URL `http://spoolman.app-spoolman:8000`, no API key. It matches AMS slots against Spoolman records by RFID and reports per-filament usage after a print completes. Only official Bambu spools with RFID sync automatically; third-party and refilled spools are skipped.

**Home Assistant.** A community integration exists. It is server-side, so no CORS configuration is needed. Use the same in-cluster URL.

**Prometheus.** `SPOOLMAN_METRICS_ENABLED=TRUE` plus `templates/servicemonitor.yaml` scraping `/metrics`. Unlike Bambuddy this is a real environment variable rather than a UI toggle, so the target comes up green on first boot. Note the path is top level, not under `/api/v1`: `main.py` mounts it outside the API router so it does not collide with the single-page-app catch-all at the root.

The CNPG cluster exports separately through `monitoring.enablePodMonitor`, matching `firebin-db` and `bambuddy-db`.

## Notes

- Liveness and readiness probes hit `GET /api/v1/health`. The API is mounted at `/api/v1` in `main.py`, and the health route is defined in `spoolman/api/v1/router.py`. Verified against the running container's `/openapi.json`, not just the docs.
- `PUID` must be a non-zero integer. `entrypoint.sh` hard-fails on `PUID=0` because the container drops root through `gosu` and a PUID of 0 would make that re-exec loop forever.
- `EXTERNAL_DB_URL` defaults to `https://donkie.github.io/SpoolmanDB/` and syncs the filament catalogue hourly, so the cluster needs to resolve and reach that host. Set `EXTERNAL_DB_SYNC_INTERVAL=0` for startup-only, or an empty `EXTERNAL_DB_URL` to turn it off.
- `SPOOLMAN_BASE_PATH` (sub-path hosting) and `SPOOLMAN_LEGACY_CLIENT` (the previous React UI, shipped in the same image as a rollback) are available if ever needed.
- Rolling back the image tag is safe for data since Postgres holds it, but alembic does not auto-downgrade. Check the release notes before stepping back across a migration.
