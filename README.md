# CommandClaw Observe

Shared observability stack for CommandClaw services.

## Services

| Service | Port | Purpose |
|---------|------|---------|
| **Langfuse** | [localhost:3000](http://localhost:3000) | LLM tracing — every agent run, tool call, and LLM invocation |
| **Prometheus** | [localhost:9090](http://localhost:9090) | Metrics collection — scrapes MCP gateway /metrics |
| **Grafana** | [localhost:3001](http://localhost:3001) | Dashboards — pre-loaded MCP Gateway dashboard |
| Postgres | (internal) | Backing store for Langfuse |

## Quick Start

```bash
docker compose up -d
```

## Default Credentials

| Service | User | Password |
|---------|------|----------|
| Langfuse | admin@commandclaw.dev | admin |
| Grafana | admin | admin |

## Connecting Services

### CommandClaw Agent

Add to your agent's `.env`:

```
COMMANDCLAW_LANGFUSE_PUBLIC_KEY=pk-lf-local
COMMANDCLAW_LANGFUSE_SECRET_KEY=sk-lf-local
COMMANDCLAW_LANGFUSE_HOST=http://localhost:3000
```

Or if running in Docker on the same network:

```
COMMANDCLAW_LANGFUSE_HOST=http://langfuse:3000
```

### MCP Gateway

Prometheus scrapes `host.docker.internal:8100/metrics` by default.
Edit `prometheus/prometheus.yml` to change the target.

## Pre-loaded Dashboards

- **MCP Gateway** — tool call rates, latency p95, active sessions, auth failures, RBAC denials, key rotations
