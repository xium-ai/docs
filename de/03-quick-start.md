---
layout: default
title: Quick Start
parent: Deutsch
nav_order: 3
---

# Quick Start

## Voraussetzungen

- XOS Binary (`xos` / `xos.exe`) — [Download](https://github.com/xium-ai/releases)
- Docker mit Docker Compose v2
- Claude Desktop mit MCP-Unterstützung

## 1. Demo-Stack starten

Der schnellste Einstieg ist der Demo-Stack. Er startet alle benötigten Komponenten und konfiguriert etcd automatisch.

```bash
git clone https://github.com/xium-ai/demo
cd demo

make infra      # Phase 1: OpenBao, Keycloak, PostgreSQL, etcd
make app        # Phase 2: XOSP, MinIO, Memgraph und weitere
make register   # XOSP Fingerprint einmalig registrieren
```

→ Details und Makefile-Referenz: [Demo Stack](45-demo.md)

## 2. XOS starten

```bash
./xos --etcd localhost:2379
```

Der Browser öffnet sich für den Keycloak-Login (`frank` / `xos-dev-2026`).

## 3. Claude Desktop verbinden

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/pfad/zu/xos",
      "args": ["--bridge", "--etcd", "localhost:2379"]
    }
  }
}
```

Claude Desktop neu starten. Das Hammer-Symbol (🔨) zeigt die verfügbaren XOS-Tools.

## 4. Ersten Query stellen

In Claude Desktop:

> „Zeige mir alle Personen"

Claude lädt das Schema, fragt die Daten ab und rendert sie ins Board.
