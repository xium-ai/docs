---
layout: default
title: Quick Start
parent: Deutsch
nav_order: 3
---

# Quick Start

## Voraussetzungen

- XOS Binary (`xos` / `xos.exe`)
- Docker mit Docker Compose
- Claude Desktop mit MCP-Unterstützung

## 1. Demo-Stack starten

```bash
git clone https://github.com/xium-ai/demo
cd demo

make infra   # Phase 1: Vault, Keycloak, PostgreSQL, etcd
# Warten bis Vault + Keycloak bereit sind (~30s)

make app     # Phase 2: XOSP, MinIO, Memgraph und Setup-Job
make register  # XOSP Fingerprint in etcd schreiben (einmalig)
```

XOS bezieht die gesamte Konfiguration aus **etcd** — keine `xos.toml` nötig.

## 2. XOS starten

```bash
./xos --etcd localhost:2379
```

Der Browser öffnet sich automatisch für den Keycloak-Login.

```
Benutzer:  frank  /  xos-dev-2026
```

## 3. Claude Desktop verbinden

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/pfad/zu/xos",
      "args": ["--bridge"]
    }
  }
}
```

Claude Desktop neu starten. Das Hammer-Symbol (🔨) zeigt die verfügbaren XOS-Tools an.

## 4. Ersten Query stellen

In Claude Desktop:

> „Zeige mir alle Personen"

Claude lädt das Schema, fragt die Daten ab und rendert sie ins Board.

## Makefile-Übersicht

| Befehl | Beschreibung |
|---|---|
| `make infra` | Phase 1: Vault, Keycloak, PostgreSQL, etcd |
| `make app` | Phase 2: Anwendung (setzt Phase 1 voraus) |
| `make register` | XOSP Fingerprint in etcd schreiben (einmalig nach `make app`) |
| `make status` | Laufende Container |
| `make down` | Stack stoppen |
| `make reset` | Alles löschen inkl. Volumes |
