# AIOps demo workspace

This repository combines the existing OpenTelemetry/Grafana demo and AIOps portal as a starting point for a later generic AIOps platform. The current portal still assumes the demo's ClickHouse `otel` schema and order-service metrics.

## Layout

| Path | Purpose |
| --- | --- |
| `compose.yaml` | Combined local stack and shared Docker network |
| `observability/` | Demo services, load generator, OTel Collector, Grafana configuration |
| `portal/backend/` | FastAPI analytics and AI endpoints |
| `portal/frontend/` | Next.js static UI served by nginx |
| `portal/frontend_old/` | Earlier HTML portal, retained for reference |
| `observability/other-non-included-services/` | Original standalone ClickHouse reference configuration |

## Run locally

1. Copy `.env.example` to `.env` and replace both example passwords. Add `GEMINI_API_KEY` if using the chat and AI incident summary features.
2. Run `docker compose -f compose.yaml up --build -d` from this directory. The backend image compiles CmdStan and may take time to build.
3. Open the portal at `http://localhost:3001`, Grafana at `http://localhost:3000`, and the API docs at `http://localhost:8080/docs`.

The portal's nginx proxy sends browser `/api/` requests to the backend. ClickHouse, the collector, and the backend communicate by service name on the shared Docker network. Grafana provisions the demo dashboard from `observability/grafana/provisioning/dashboards/otel-demo.json`.

The nested Compose files are preserved as source references. Use the root `compose.yaml` for the combined stack. This copy contains no backend `.env` file or source Git metadata.

## Existing host deployment

The current VM uses `compose.deployment.yaml` with the root Compose file. It reuses the original Grafana, Tempo, Loki, Prometheus, and CmdStan volumes, and connects to the existing external ClickHouse database through values in the ignored `.env` file. It also names the existing Python images so the cutover does not depend on rebuilding their large dependency sets. The frontend image is built from `portal/frontend/` in this repo.

```bash
docker compose -f compose.yaml -f compose.deployment.yaml ps
docker compose -f compose.yaml -f compose.deployment.yaml up -d --no-build
```

The standalone deployment uses `docker compose -f compose.yaml up --build -d` and its own local ClickHouse volume. Do not run both forms together on the same host because they publish the same ports.

## Source snapshots

- `observability/` came from `/home/jonathan_jordy/grafana-otel-demo` at `003a74c1195ed8320ae90040470143854d505f98` (`main`, clean).
- `portal/` came from `/home/jonathan_jordy/web-portal-for-grafana-otel-demo` at `10b94513fd94e4e67eed7a51dc1fffd4dd0f7977` (`antigravity`). Its source `backend/.env` had local modifications and was deliberately excluded.

## Current limits

The analytics code is coupled to the existing `otel` tables, metric names, and service names. The original demo documentation may describe its standalone deployment; this root README and `compose.yaml` define the combined deployment. The running services at `35.219.90.43` were not changed by this copy.
