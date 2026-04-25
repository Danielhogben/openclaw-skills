---
name: kotlin-libgdx-mmo
description: |
  Develop and operate the custom Kotlin/LibGDX MMO game. Use when the user wants to:
  build the game, run tests, start the game server, manage assets, debug networking,
  or work with Gradle modules (core, server, client). Triggers on keywords: mmo, game,
  libgdx, kotlin game, server run, gradle build game, Netty, tick, port 43594.
version: 1.0.0
author: donn
license: MIT
metadata:
  openclaw:
    emoji: 🎮
    tags: [kotlin, libgdx, mmo, game, gradle, netty, server, gamedev]
---

# Kotlin LibGDX MMO Development

Develop and operate the custom Kotlin/LibGDX MMO.

## Quick Commands

```bash
./gradlew build            # Build all modules
./gradlew :core:test       # Run core tests
./gradlew :server:run      # Start server on port 43594
./gradlew :desktop:run     # Run desktop client
```

## Architecture

- **Language**: Kotlin (with some Java)
- **Framework**: LibGDX
- **Networking**: Netty
- **Tick Rate**: 100ms
- **Server Port**: 43594

## Module Structure

Typical LibGDX multi-module setup:

```
├── core/           # Shared game logic, entities, world state
├── server/         # Netty server, game loop, player management
├── desktop/        # Desktop client launcher
├── android/        # Android client (if present)
├── build.gradle    # Root build script
└── gradle.properties
```

## Development Workflows

### Build and Test

```bash
./gradlew build
./gradlew test
```

Run specific module tests:

```bash
./gradlew :core:test
./gradlew :server:test
```

### Run Server

```bash
./gradlew :server:run
```

Server binds to port 43594. Verify it's running:

```bash
ss -tlnp | grep 43594
nc -zv localhost 43594
```

### Run Desktop Client

```bash
./gradlew :desktop:run
```

### Clean Build

```bash
./gradlew clean build
```

## Code Style

- `val` for immutables, `var` for mutables
- Data classes with `@Serializable` for network messages
- `when` expressions over if-else chains
- 4 spaces indent
- PascalCase classes, camelCase methods/variables

## Common Tasks

### Add a New Entity Type

1. Define in `core/src/.../entity/`
2. Add serialization in shared packet class
3. Register server-side handler
4. Add client-side renderer

### Debug Networking

```bash
# Monitor Netty connections
ss -tlnp | grep 43594

# Packet capture (server side)
sudo tcpdump -i any port 43594 -w mmo-traffic.pcap
```

### Performance Profiling

```bash
./gradlew :server:run --debug-jvm
# Attach JProfiler or VisualVM
```

### Asset Management

Assets typically live in `core/assets/` or `android/assets/`:

```bash
ls core/assets/
# textures/, sounds/, maps/, sprites/
```

After adding assets, trigger a Gradle sync:

```bash
./gradlew :core:processResources
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Port 43594 in use | `lsof -ti:43594 | xargs kill -9` or change port in config |
| Gradle daemon issues | `./gradlew --stop && ./gradlew build` |
| Out of memory | Increase `-Xmx` in `gradle.properties`: `org.gradle.jvmargs=-Xmx4g` |
| LibGDX version conflict | Check `gradle.properties` for `gdxVersion` |
| Kotlin serialization error | Ensure `@Serializable` is present, check proguard rules |

## Deployment

Build distribution:

```bash
./gradlew :server:distZip
./gradlew :desktop:distZip
```

Artifacts in `server/build/distributions/` and `desktop/build/distributions/`.

## CI/CD Validation

Before committing:

```bash
./gradlew build test
```

Ensure no compilation errors and all tests pass. The server module should start without crashing.
