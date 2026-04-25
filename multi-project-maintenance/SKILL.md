---
name: multi-project-maintenance
description: |
  Maintain and monitor multiple AI/development projects on the Desktop. Use when the user
  wants to: build, test, lint, check dependencies, update packages, run Docker commands,
  or perform health checks across deer-flow, potpie, zeroclaw, dify, eliza, superset,
  PraisonAI, or fish-speech. Triggers on keywords: build, test, lint, deps, update,
  maintain, project health, docker cleanup, Desktop projects.
version: 1.0.0
author: donn
license: MIT
metadata:
  openclaw:
    emoji: 🛠️
    tags: [maintenance, projects, build, test, lint, docker, python, go, kotlin, typescript]
---

# Multi-Project Maintenance

Maintain the active project portfolio on `~/Desktop/`.

## Project Registry

| Project | Type | Path | Build | Test | Lint |
|---------|------|------|-------|------|------|
| deer-flow | Python/TS AI | `~/Desktop/deer-flow/` | `make dev` (Docker) | — | — |
| potpie | Python knowledge graph | `~/Desktop/potpie/` | `uv sync` | `pytest` | `ruff check .`, `mypy .` |
| zeroclaw | Go/Rust AI assistant | `~/Desktop/zeroclaw/` | `make build` | `cargo test` | `cargo fmt --all -- --check`, `cargo clippy --all-targets -- -D warnings` |
| dify | AI workflow platform | `~/Desktop/dify/` | Docker-based | — | — |
| eliza | AI agent framework | `~/Desktop/eliza/` | `pnpm install` | `npm test` | `npm run lint` |
| superset | Data visualization | `~/Desktop/superset/` | Docker-based | — | — |
| PraisonAI | AI framework | `~/Desktop/PraisonAI/` | Python | `pytest` | `ruff check .` |
| fish-speech | TTS | `~/Desktop/fish-speech/` | Python | — | — |

## Lab Projects

| Project | Type | Path |
|---------|------|------|
| Ghidra | Java/Gradle | `~/Lab/Projects/ghidra*` |
| router-tool | Node/Puppeteer | `~/Lab/Projects/router-tool/` |
| custom-mmo | Kotlin/LibGDX | `~/Lab/Games/` or separate workspace |

## Quick Maintenance Workflows

### Daily Health Check

Check all projects for uncommitted changes and recent test status:

```bash
for dir in ~/Desktop/deer-flow ~/Desktop/potpie ~/Desktop/zeroclaw ~/Desktop/dify ~/Desktop/eliza ~/Desktop/superset ~/Desktop/PraisonAI ~/Desktop/fish-speech; do
  echo "=== $(basename $dir) ==="
  cd "$dir" 2>/dev/null && git status --short || echo "Not a git repo"
done
```

### Python Projects (potpie, PraisonAI, fish-speech)

```bash
cd ~/Desktop/potpie && uv sync && pytest && ruff check . && mypy .
cd ~/Desktop/PraisonAI && pip install -e . && pytest && ruff check .
```

### Go/Rust Project (zeroclaw)

```bash
cd ~/Desktop/zeroclaw
make build
make build-launcher
make build-all
./dev/ci.sh all   # Full validation
```

### Node Projects (deer-flow frontend, eliza, router-tool)

```bash
cd ~/Desktop/eliza && pnpm install && npm test && npm run lint
cd ~/Desktop/deer-flow && pnpm install  # frontend deps
```

### Docker Projects (deer-flow, dify, superset)

```bash
cd ~/Desktop/deer-flow && make dev   # needs Docker, TAVILY_API_KEY, JINA_API_KEY
cd ~/Desktop/dify && docker compose up -d
cd ~/Desktop/superset && docker compose up -d
```

### Kotlin/Gradle (custom-mmo)

```bash
./gradlew build
./gradlew test
./gradlew :server:run   # port 43594
```

## Dependency Updates

### Python (uv/pip)

```bash
cd ~/Desktop/potpie && uv sync --upgrade
```

### Node (pnpm/npm)

```bash
cd ~/Desktop/eliza && pnpm update
```

### Rust (cargo)

```bash
cd ~/Desktop/zeroclaw && cargo update
```

### Go

```bash
cd ~/Desktop/zeroclaw && go get -u ./...
```

## Docker Cleanup

Weekly cleanup to reclaim disk space:

```bash
docker system prune -f
docker volume prune -f
docker image prune -f
```

## Config Backups

Backup important project configs:

```bash
tar czf ~/backups/project-configs-$(date +%Y%m%d).tar.gz \
  ~/Desktop/*/.*env* \
  ~/Desktop/*/.envrc \
  ~/Desktop/*/docker-compose.yml \
  ~/Desktop/*/Makefile \
  ~/Desktop/*/Justfile \
  2>/dev/null
```

## Automation Ideas

- **Daily at 6 AM**: Run tests on modified projects, report failures.
- **Weekly on Sunday at 2 AM**: Docker cleanup + dependency check.
- **Daily at 8 AM**: Git status check, report uncommitted work.
