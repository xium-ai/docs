---
layout: default
title: Quick Start
parent: English
nav_order: 3
---

# Quick Start

## Prerequisites

- XOS binary (`xos` / `xos.exe`)
- Docker with Docker Compose
- Claude Desktop with MCP support

## 1. Start the demo stack

```bash
git clone https://github.com/xium-ai/demo
cd demo

make infra   # Phase 1: Vault, Keycloak, PostgreSQL, etcd
# Wait ~30s for Vault + Keycloak to be ready

make app     # Phase 2: XOSP, MinIO, Memgraph and setup job
make register  # Write XOSP fingerprint to etcd (once)
```

XOS reads all configuration from **etcd** — no `xos.toml` needed.

## 2. Start XOS

```bash
./xos --etcd localhost:2379
```

The browser opens automatically for Keycloak login.

```
User:  frank  /  xos-dev-2026
```

## 3. Connect Claude Desktop

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/path/to/xos",
      "args": ["--bridge"]
    }
  }
}
```

Restart Claude Desktop. The hammer icon (🔨) shows the available XOS tools.

## 4. First query

In Claude Desktop:

> "Show me all persons"

Claude loads the schema, queries the data and renders it into the board.

## Makefile overview

| Command | Description |
|---|---|
| `make infra` | Phase 1: Vault, Keycloak, PostgreSQL, etcd |
| `make app` | Phase 2: Application (requires Phase 1) |
| `make register` | Write XOSP fingerprint to etcd (once after `make app`) |
| `make status` | Running containers |
| `make down` | Stop stack |
| `make reset` | Delete everything including volumes |
