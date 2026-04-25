---
name: omniclaw-swarm
description: |
  Manage the OmniClaw Collective multi-agent swarm. Use when the user wants to:
  wake agents, check swarm health, broadcast messages to all agents, kill/stop agents,
  check node status (Jupiter, Atlas, Luna, Vega, Cosmo, Hive), or coordinate between
  Hermes, OpenClaw, and ZeroClaw. Triggers on keywords: swarm, agents, wake, kill,
  broadcast, omniclaw, collective, node status.
version: 1.0.0
author: donn
license: MIT
metadata:
  openclaw:
    emoji: 🤖
    tags: [swarm, agents, omniclaw, hermes, zeroclaw, openclaw, orchestration]
---

# OmniClaw Swarm Management

Manage the OmniClaw Collective — a multi-agent swarm spanning Hermes, OpenClaw, and ZeroClaw across multiple nodes.

## Swarm Architecture

| Node | Host | Role | IP | Status Check |
|------|------|------|-----|--------------|
| Jupiter | pop-os | MASTER | 192.168.0.76 | localhost |
| Atlas | aspen | WORKER | 192.168.0.210 | SSH + ping |
| Luna | pi | EDGE | 192.168.0.49 | SSH + ping |
| Vega | lichee-rv | EDGE | — | PENDING CONFIGURATION |
| Cosmo | openclaw-gateway | GATEWAY | 127.0.0.1 | localhost service |
| Hive | esp32-mesh | IOT | — | STANDBY |

## Core Commands

### Wake All Agents

Start the entire swarm:

```bash
wake-all-agents
```

Full mode (includes NemoClaw):

```bash
wake-all-agents --full
```

This starts: Ollama → OpenClaw Gateway → ZeroClaw Daemon → Swarm Orchestrator → (optionally NemoClaw).

### Kill All Agents (Emergency Stop)

```bash
kill-all-agents
```

Stops: Swarm Orchestrator → NemoClaw → ZeroClaw → OpenClaw Gateway → remaining agent processes.

### Broadcast Message

Send a message to all active agents:

```bash
talk-to-agents "Your message here"
```

Individual agent messaging:

```bash
zeroclaw agent -m "Message to ZeroClaw"
openclaw agent --agent main -m "Message to OpenClaw main"
openclaw agent --agent aspen -m "Message to OpenClaw aspen"
openclaw agent --agent pi -m "Message to OpenClaw pi"
```

### Check Swarm Status

```bash
swarm-status
```

Or manually check services:

```bash
systemctl --user is-active swarm-orchestrator
systemctl --user is-active zeroclaw
systemctl --user is-active openclaw-gateway
systemctl is-active ollama
```

### View Logs

```bash
swarm-logs
journalctl --user -u swarm-orchestrator -f
journalctl --user -u zeroclaw -f
journalctl --user -u openclaw-gateway -f
```

## Node Health Checks

### Jupiter (Local Master)

```bash
systemctl is-active ollama
systemctl --user is-active openclaw-gateway
systemctl --user is-active zeroclaw
systemctl --user is-active swarm-orchestrator
```

### Atlas (Aspen Worker)

```bash
ping -c 1 -W 1 192.168.0.210
nc -zv 192.168.0.210 22
ssh aspen "systemctl --user is-active openclaw-agent" 2>/dev/null
```

### Luna (Pi Edge)

```bash
ping -c 1 -W 1 192.168.0.49
nc -zv 192.168.0.49 22
ssh pi "systemctl --user is-active openclaw-agent" 2>/dev/null
```

## Cron Job Integration

Hermes cron jobs can coordinate with the swarm. Example cron prompts:

- **Health Check**: "Ping all swarm nodes. Report which are offline. Send Telegram alert if any critical node is down."
- **Daily Wake**: "Run wake-all-agents at 7 AM if not already running."
- **Broadcast**: "Send daily status summary to all agents via talk-to-agents."

## Troubleshooting

| Issue | Command |
|-------|---------|
| Agent not responding | `kill-all-agents && wake-all-agents` |
| OpenClaw gateway down | `systemctl --user restart openclaw-gateway` |
| ZeroClaw daemon down | `systemctl --user restart zeroclaw` or `zeroclaw daemon &` |
| Ollama model server down | `sudo systemctl restart ollama` |
| Swarm orchestrator stuck | `systemctl --user restart swarm-orchestrator` |
| Node offline | Check physical power/network, then `wake-all-agents` |

## Safety Rules

1. Always prefer `wake-all-agents` over manual service starts — it ensures correct startup order.
2. Use `kill-all-agents` only as emergency stop — it force-kills processes without graceful shutdown.
3. Verify node health before broadcasting — `talk-to-agents` fails silently for offline nodes.
4. NemoClaw requires NVIDIA Docker setup — skip unless `--full` flag is explicitly requested.
