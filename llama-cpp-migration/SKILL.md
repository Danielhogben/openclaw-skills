---
name: llama-cpp-migration
description: |
  Replace Ollama with llama.cpp server for lower resource usage. Use when Ollama is
  consuming excessive CPU/RAM, the user wants a lighter local LLM inference engine,
  or wants to reduce Docker container count. Covers model extraction, llama-server setup,
  OpenAI-compatible API configuration, and agent migration. Triggers on: ollama slow,
  ollama CPU high, replace ollama, lighter llm, llama.cpp, local inference, reduce
  docker containers, ollama alternative, model server.
---

# Ollama → llama.cpp Migration

Replace Ollama with `llama.cpp` server. Same models, 15-25% faster, near-zero idle usage, no Docker required.

## Why Switch

| | Ollama | llama.cpp |
|--|--------|-----------|
| Idle RAM | ~1.7GB (container) | ~0MB |
| Idle CPU | Background services | None |
| Overhead | 15-25% abstraction tax | Raw engine |
| Format | Wrapped GGUF | Native GGUF |
| Docker needed | Yes | No |

## Extract Models from Ollama

Ollama stores models as GGUF blobs. Extract them:

```bash
python3 ~/.kimi/skills/llama-cpp-migration/scripts/extract_ollama_models.py ~/llama-models
```

Or manually:

```bash
# Find model blobs
ls -la /var/lib/docker/volumes/ollama/_data/models/ 2>/dev/null || \
ls -la ~/.ollama/models/

# Copy blobs out and rename to .gguf
# Then verify with: llama-cli --model model.gguf --verbose
```

## Install llama.cpp

### Option A: Prebuilt Binary

```bash
cd ~/.local/bin
curl -L -o llama-server https://github.com/ggerganov/llama.cpp/releases/download/b3662/llama-server-x86_64-linux-cpu-avx2
chmod +x llama-server
curl -L -o llama-cli https://github.com/ggerganov/llama.cpp/releases/download/b3662/llama-cli-x86_64-linux-cpu-avx2
chmod +x llama-cli
```

### Option B: Build from Source

```bash
git clone https://github.com/ggerganov/llama.cpp ~/llama.cpp
cd ~/llama.cpp
make -j$(nproc)
# Binaries: ./llama-server, ./llama-cli
```

## Run llama-server

Start with a small model for testing:

```bash
llama-server \
  -m ~/llama-models/qwen2.5-0.5b.gguf \
  --port 11434 \
  -ngl 0 \
  -np 4 \
  --ctx-size 2048 \
  --verbose 0
```

Flags:
- `-m`: model path
- `-ngl 0`: CPU only (no GPU layers)
- `-np 4`: 4 parallel slots
- `--ctx-size 2048`: context window
- `--verbose 0`: quiet

## Systemd Service

Create `~/.config/systemd/user/llama-server.service`:

```ini
[Unit]
Description=llama.cpp server
After=network.target

[Service]
Type=simple
ExecStart=%h/.local/bin/llama-server -m %h/llama-models/qwen2.5-0.5b.gguf --port 11434 -ngl 0 -np 4 --ctx-size 2048 --verbose 0
Restart=on-failure

[Install]
WantedBy=default.target
```

Enable:

```bash
systemctl --user daemon-reload
systemctl --user enable llama-server
systemctl --user start llama-server
```

## Agent Migration

Update agent configs to point to `http://localhost:11434` (same port, drop-in replacement).

### Hermes

Edit `~/.hermes/.env` or config:

```bash
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=qwen2.5:0.5b
```

### OpenClaw

Update model provider in `~/.openclaw/config` to use the local OpenAI-compatible endpoint.

### ZeroClaw

Point provider to `http://localhost:11434/v1/chat/completions`.

## Stop Ollama

```bash
docker stop ollama
docker rm ollama
# Or if system service:
sudo systemctl stop ollama
```

Free disk space:

```bash
docker image prune -a -f
rm -rf ~/.ollama/models/*  # if extracted
```

## Quick Benchmark

Compare token speed:

```bash
# llama.cpp
llama-cli -m ~/llama-models/qwen2.5-0.5b.gguf -p "Hello" -n 50

# Ollama (if still running)
OLLAMA_HOST=http://localhost:11434 ollama run qwen2.5:0.5b "Hello"
```

## Safety

- Keep Ollama running until llama.cpp is verified working.
- Extract models before deleting Ollama volumes.
- Test one agent at a time after switching endpoints.
