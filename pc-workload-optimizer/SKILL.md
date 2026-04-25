---
name: pc-workload-optimizer
description: |
  Reduce CPU, memory, disk, and token workload on the local machine. Use when the user
  complains about slowness, high resource usage, fans spinning, OOM errors, disk full,
  or wants to optimize their AI agent setup. Covers Docker cleanup, process killing,
  cache pruning, Ollama model unloading, agent context reduction, log rotation, and
  swarm node shutdown. Triggers on: slow, lag, memory, cpu, optimize, workload,
  resources, disk full, cleanup, prune, unload model, stop agents, performance.
version: 1.0.0
author: donn
license: MIT
metadata:
  openclaw:
    emoji: ⚡
    tags: [performance, optimization, cleanup, resources, cpu, memory, disk, workload]
---

# PC Workload Optimizer

Cut resource usage across all running AI agents, containers, and services.

## Quick Diagnosis

```bash
# What's eating CPU/RAM?
ps aux --sort=-%mem | head -20
ps aux --sort=-%cpu | head -20

# What's eating disk?
du -sh ~/* 2>/dev/null | sort -rh | head -10
ncdu ~   # interactive disk explorer

# Docker bloat
docker system df
docker stats --no-stream
```

## Agent Workload Reduction

### Hermes

1. **Reduce context window** — edit `~/.hermes/config.yaml` or env:
   ```bash
   HERMES_MAX_CONTEXT=4000   # default may be 8k-128k
   ```
2. **Disable unused skills** — only load skills needed for current task:
   ```bash
   hermes skills list
   # move unused skills out of ~/.hermes/skills/
   ```
3. **Shorten session history** — prune old sessions:
   ```bash
   hermes sessions prune --older-than 7d
   ```

### OpenClaw

1. **Unload unused skills**:
   ```bash
   openclaw skills check   # see what's ready vs needs setup
   ```
2. **Reduce gateway workers** — check `~/.openclaw/config` for `maxConcurrentAgents`.
3. **Disable channels** — turn off Discord/Slack/Telegram if not needed:
   ```bash
   openclaw channels list
   openclaw channels disconnect <name>
   ```

### ZeroClaw

1. **Use smaller models** — switch from 70B to 8B or 0.5B for simple tasks.
2. **Reduce max_tokens** in config.
3. **Stop daemon if idle**:
   ```bash
   systemctl --user stop zeroclaw
   ```

### Ollama

1. **Unload all models** (frees VRAM/RAM):
   ```bash
   ollama stop $(ollama list | tail -n +2 | awk '{print $1}')
   # or
   ollama ps   # see loaded models
   ollama stop <name>
   ```
2. **Run smaller models**:
   ```bash
   ollama run llama3.2        # 3B params, fast
   ollama run qwen2.5:0.5b    # 0.5B params, tiny
   ollama run gemma2:2b       # 2B params, efficient
   ```
3. **Limit context window** in Modelfile:
   ```
   PARAMETER num_ctx 2048
   ```

## System Cleanup

### Docker

```bash
docker system prune -f          # remove stopped containers, unused networks, dangling images
docker volume prune -f          # remove unused volumes
docker image prune -a -f        # remove ALL unused images (aggressive)
docker builder prune -f         # clear build cache
```

### Caches

```bash
# Python
rm -rf ~/.cache/pip
rm -rf ~/.cache/uv
find ~ -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null

# Node
npm cache clean --force
pnpm store prune

# Rust
cargo cache --autoclean

# General
rm -rf ~/.cache/thumbnails/*
rm -rf ~/.local/share/Trash/files/*
journalctl --vacuum-time=3d   # trim logs to 3 days
```

### Logs

```bash
# Agent logs
rm -f ~/.hermes/logs/*.log.*   # old rotated logs
rm -f ~/.openclaw/logs/*.log.*
rm -f ~/.kimi/logs/*.log.*

# System logs
sudo journalctl --rotate
sudo journalctl --vacuum-time=7d
```

## Swarm Shutdown

If agents are idle, shut down non-essential nodes:

```bash
# Stop everything
kill-all-agents

# Or selective stop
systemctl --user stop zeroclaw
systemctl --user stop openclaw-gateway
systemctl --user stop swarm-orchestrator
sudo systemctl stop ollama
```

## Automation

Add these to Hermes cron for auto-cleanup:

- **Daily at 3 AM**: `docker system prune -f`
- **Weekly**: `journalctl --vacuum-time=3d`, clear `__pycache__`
- **On low disk**: alert + auto-prune caches
- **On high RAM**: unload Ollama models, stop idle agents

## Emergency Low-Resource Mode

When the system is struggling (RAM >90%, CPU pinned):

1. `kill-all-agents`
2. `ollama stop $(ollama list | tail -n +2 | awk '{print $1}')`
3. `docker system prune -f`
4. `sudo systemctl stop ollama`
5. Re-enable only what's needed with `wake-all-agents`
