---
layout: default
title: Deployment
parent: Deutsch
nav_order: 44
---

# Deployment

## Demo-Stack (Docker Compose)

Der Demo-Stack enthält alle Komponenten und ist der empfohlene Einstieg.

### Ablauf

```bash
make infra     # Phase 1: Vault, Keycloak, PostgreSQL, etcd
# ~30s warten bis Vault + Keycloak bereit sind

make app       # Phase 2: XOSP, MinIO, Memgraph, Setup-Job
make register  # Einmalig: XOSP Fingerprint in etcd schreiben
```

### Stack-Komponenten

| Komponente | Port | Beschreibung |
|---|---|---|
| OpenBao (Vault) | 8200 | Secrets, PKI, XOSP Identity |
| Keycloak | 8080 | IAM / OIDC |
| PostgreSQL | 5432 | Relationale Datenbank |
| etcd | 2379 | Konfigurationsquelle |
| MinIO | 9000 / 9001 | HTML-Templates (S3) |
| Memgraph | 7687 | Graph-Datenbank für Contexts |
| XOSP | 9100 | Plugin-Server |

### Login-Daten Demo

| | |
|---|---|
| Benutzer | `frank` / `tristan` |
| Passwort | `xos-dev-2026` |
| Vault Token | `xos-dev-root-token` |
| Keycloak Admin | `admin` / `xos-kc-bootstrap` |

## XOS starten

```bash
./xos --etcd localhost:2379
```

XOS lädt die gesamte Konfiguration aus etcd und öffnet den Browser für den Login.

## Claude Desktop verbinden (Bridge)

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

Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Claude Desktop neu starten. Das Hammer-Symbol (🔨) zeigt die verfügbaren XOS-Tools.

## make register — wann nötig?

`make register` schreibt den XOSP Fingerprint aus Vault in etcd. Es muss ausgeführt werden:

- Nach dem ersten `make app`
- Nach `make reset` (neues Vault-Volume → neuer Fingerprint)

Solange das Vault-Volume erhalten bleibt (`make down` / `make up`), bleibt der Fingerprint konstant.

## Linux

Der Demo-Stack läuft auf Linux identisch:

```bash
make infra && make app && make register
./xos --etcd localhost:2379
```

Voraussetzung: Docker Compose v2, `python3` für `make register`.
