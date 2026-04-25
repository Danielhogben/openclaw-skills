---
name: stack-optimizer
description: |
  Optimize the entire OmniClaw tech stack for lower resource usage and better performance.
  Use when the user wants to reduce CPU, RAM, disk, or container overhead across their
  systems. Covers alternatives to Docker, databases, media servers, LLM inference, and
  AI frameworks. Triggers on: optimize stack, reduce resources, lightweight alternatives,
  performance, make it run better, swap docker, swap ollama, swap redis, swap postgres.
---

# Full Stack Optimization Guide

Optimize every layer of the OmniClaw stack.

## Current Bottlenecks (from workload report)

| Component | Issue |
|-----------|-------|
| Ollama container | 611% CPU, 1.7GB RAM |
| Docker images | 20.6GB reclaimable |
| 18 containers | Zero resource limits |
| Agent swarm | 5 services running idle |

## LLM Inference

**Replace Ollama → llama.cpp server**

Already validated. 91 tok/s on qwen2.5-0.5b. Near-zero idle usage.

```bash
~/llama.cpp-bin/llama-b8925/llama-server -m ~/llama-models/qwen2.5-0.5b.gguf --port 11434
```

## Container Runtime

**Keep Docker for now, but migrate new workloads to Podman.**

| | Docker | Podman |
|--|--------|--------|
| Daemon | Yes (background service) | No |
| Root required | Often | Rootless possible |
| Idle RAM | ~150MB+ | ~0MB |
| Startup | 200-300ms | Faster |

```bash
# Install Podman
sudo apt install podman podman-compose
alias docker=podman
```

For existing Docker stack: add resource limits immediately.

## Databases

| Current | Better Alt | When to Switch |
|---------|-----------|----------------|
| Redis | Valkey | Drop-in replacement, BSD license |
| Redis | KeyDB | Multi-threaded, 25x throughput |
| PostgreSQL (small apps) | SQLite | <1GB data, single-user |
| PostgreSQL (analytics) | DuckDB | OLAP workloads, embedded |
| Neo4j | Kuzu | Embedded graph DB, lighter |
| Chroma | SQLite + vec | If only simple vector search |

**Action for Hermes stack:**
- Keep PostgreSQL + Neo4j (Hermes needs them)
- Replace Redis container with Valkey: `docker run valkey/valkey`
- Add `--memory=256m` to Redis/Valkey container

## Media Server

**Jellyfin is already efficient.** Just optimize:

```bash
# Use SQLite instead of PostgreSQL for Jellyfin (libraries <15k items)
# Disable trickplay image generation
# Set library scan interval to 1h
```

**Navidrome is already optimal** for music (47MB RAM idle).

## AI Frameworks

| Project | Optimization |
|---------|-------------|
| deer-flow | Uses `make dev` (Docker). Run backend natively with `uv` if possible. |
| potpie | Already uses `uv sync`. Good. |
| zeroclaw | Rust project. Already efficient. `cargo build --release` is optimized. |
| dify | Docker-only. Hard to lighten. |
| eliza | Node-based. Use `pnpm` instead of `npm`. |
| superset | Docker-only. Hard to lighten. |
| PraisonAI | Python. Use `uv` instead of `pip`. |
| fish-speech | Python. Heavy. Use only when needed. |

## Immediate Actions

Run these now for instant relief:

```bash
# 1. Stop Ollama
docker stop ollama && docker rm ollama

# 2. Start llama-server
systemctl --user start llama-server

# 3. Clean Docker
docker system prune -f
docker image prune -a -f

# 4. Limit all containers
python3 ~/.kimi/skills/stack-optimizer/scripts/limit_all_containers.py

# 5. Stop idle agents
systemctl --user stop zeroclaw
systemctl --user stop swarm-orchestrator
```

## Monitoring

Track before/after:

```bash
python3 ~/.kimi/skills/pc-workload-optimizer/scripts/workload_report.py
```
