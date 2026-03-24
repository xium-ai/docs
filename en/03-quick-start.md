---
layout: default
title: Quick Start
parent: English
nav_order: 3
---

# Quick Start

## Prerequisites

- XOS binary (`xos` / `xos.exe`) — [Download](https://github.com/xium-ai/releases)
- Docker with Docker Compose v2
- Claude Desktop with MCP support

## 1. Start the demo stack

The fastest way to get started is the demo stack. It starts all required components and configures etcd automatically.

```bash
git clone https://github.com/xium-ai/demo
cd demo

make infra      # Phase 1: OpenBao, Keycloak, PostgreSQL, etcd
make app        # Phase 2: XOSP, MinIO, Memgraph and more
make register   # Register XOSP fingerprint (once)
```

→ Details and Makefile reference: [Demo Stack](45-demo.md)

## 2. Start XOS

```bash
./xos --etcd localhost:2379
```

The browser opens for Keycloak login (`frank` / `xos-dev-2026`).

## 3. Connect Claude Desktop

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/path/to/xos",
      "args": ["--bridge", "--etcd", "localhost:2379"]
    }
  }
}
```

Restart Claude Desktop. The hammer icon (🔨) shows the available XOS tools.

## 4. First query

In Claude Desktop:

> "Show me all persons"

Claude loads the schema, queries the data and renders it into the board.
