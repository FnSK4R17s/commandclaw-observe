<p align="center">
  <img src="logo.png" alt="Command Claw Observe" height="88">
</p>

<h1 align="center">Command Claw Observe</h1>

<p align="center">
  <strong>Self-hosted observability stack for CommandClaw agents.</strong><br>
  <em>Langfuse v3 tracing + Prometheus metrics + Grafana dashboards — one compose, zero config.</em><br>
  <sub>For teams that need full data sovereignty. Hosted alternatives work great for everyone else.</sub>
</p>

---

> [!WARNING]
> **🚧 Beta Software** — This project is under active development. Your feedback helps make this better!
>
> 💬 **Have feedback or found a bug?**  Reach out at [**@_Shikh4r_** on X](https://x.com/_Shikh4r_)

## Quick Start

```bash
# 1. Clone the observe stack
gh repo clone FnSK4R17s/commandclaw-observe
cd commandclaw-observe

# 2. Start everything
docker compose up -d

# 3. Open Langfuse at http://localhost:3000
#    (login: admin@commandclaw.dev / admin)
```

Then add to your agent's `.env`:

```
COMMANDCLAW_LANGFUSE_PUBLIC_KEY=pk-lf-local
COMMANDCLAW_LANGFUSE_SECRET_KEY=sk-lf-local
COMMANDCLAW_LANGFUSE_HOST=http://localhost:3000
```

That's it — traces start flowing immediately.

## Repositories

| Repo | Purpose |
|------|---------|
| [commandclaw](https://github.com/FnSK4R17s/commandclaw) | Agent runtime, Telegram I/O, tracing |
| [commandclaw-vault](https://github.com/FnSK4R17s/commandclaw-vault) | Vault template — cloned per agent workspace |
| [commandclaw-mcp](https://github.com/FnSK4R17s/commandclaw-mcp) | MCP gateway — credential proxy with rotating keys |
| [commandclaw-gateway](https://github.com/FnSK4R17s/commandclaw-gateway) | LLM routing layer — provider credentials, virtual keys, budgets, rate limits, multi-provider fallback |
| [commandclaw-skills](https://github.com/FnSK4R17s/commandclaw-skills) | Skills library — `npx skills add FnSK4R17s/commandclaw-skills` |
| [commandclaw-memory](https://github.com/FnSK4R17s/commandclaw-memory) | Recall service — wiki validation, LanceDB + BM25 indexing, distillation, hybrid retrieval |
| [commandclaw-wiki](https://github.com/FnSK4R17s/commandclaw-wiki) | LLM Wiki — persistent, compounding knowledge base per agent (Karpathy pattern) |
| [openclaw](https://github.com/FnSK4R17s/openclaw) | Original personal AI assistant — predecessor to CommandClaw |

## Do I Need This?

**Most users don't.** Hosted [Langfuse Cloud](https://langfuse.com) and [Grafana Cloud](https://grafana.com/products/cloud/) work great and require zero infrastructure.

**Self-host if:**

- Your LLM traffic can't leave your network (compliance, privacy, air-gapped environments)
- You want full control over trace retention and data lifecycle
- You're running CommandClaw in an enterprise environment with strict data sovereignty requirements

## Services

| Service | Port | Purpose |
|---------|------|---------|
| **Langfuse v3** | [localhost:3000](http://localhost:3000) | LLM tracing — multimodal, vision, tool calls |
| Langfuse Worker | (internal :3030) | Async event processing |
| **Grafana** | [localhost:3001](http://localhost:3001) | Dashboards — MCP gateway metrics |
| **Prometheus** | [localhost:9092](http://localhost:9092) | Metrics scraping |
| ClickHouse | (internal) | OLAP analytics for Langfuse |
| Redis | (internal) | Queue + cache for Langfuse |
| Postgres | (internal) | OLTP store for Langfuse |
| MinIO | [localhost:9091](http://localhost:9091) | S3-compatible media storage |

## Default Credentials

| Service | User | Password |
|---------|------|----------|
| Langfuse | admin@commandclaw.dev | admin |
| Grafana | admin | admin |
| MinIO | minio | miniosecret |

## Why Langfuse v3?

v3 adds ClickHouse (OLAP) + S3 media storage, enabling:

- **Multimodal tracing** — images, audio, and vision model I/O stored and viewable in the dashboard
- **Faster analytics** — ClickHouse handles large trace volumes without Postgres bottleneck
- **Media persistence** — MinIO stores attachments so traces aren't just text logs

This matters for CommandClaw because agents use vision models, process images, and interact with multimodal tools through MCP. Without v3, that context is invisible in traces.

## Pre-loaded Dashboards

### MCP Gateway

Grafana ships with a pre-loaded dashboard for the [commandclaw-mcp](https://github.com/FnSK4R17s/commandclaw-mcp) gateway:

- Tool call rates per tool
- Latency p95 per tool
- Active sessions
- Auth failures / RBAC denials
- Key rotation events

Prometheus scrapes `host.docker.internal:8100/metrics` by default. Edit `prometheus/prometheus.yml` to change the target.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  commandclaw-observe                     │
│                                                         │
│  ┌──────────┐  traces   ┌───────────────┐              │
│  │ Langfuse │◄──────────│ Langfuse      │              │
│  │ Web :3000│           │ Worker :3030  │              │
│  └────┬─────┘           └──────┬────────┘              │
│       │                        │                        │
│  ┌────▼─────┐  ┌───────▼──┐  ┌▼─────┐  ┌──────────┐  │
│  │ Postgres │  │ClickHouse│  │Redis │  │  MinIO   │  │
│  │ (OLTP)   │  │ (OLAP)   │  │(queue)│  │(media/S3)│  │
│  └──────────┘  └──────────┘  └──────┘  └──────────┘  │
│                                                         │
│  ┌────────────┐  scrapes  ┌──────────┐                 │
│  │ Prometheus │◄──────────│ MCP GW   │ (external)      │
│  │ :9092      │           │ :8100    │                 │
│  └─────┬──────┘           └──────────┘                 │
│        │                                                │
│  ┌─────▼──────┐                                        │
│  │  Grafana   │                                        │
│  │  :3001     │                                        │
│  └────────────┘                                        │
└─────────────────────────────────────────────────────────┘
```

## Data Persistence

All data is stored in Docker named volumes. To reset everything:

```bash
docker compose down -v
```
