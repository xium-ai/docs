---
layout: default
title: Plugin-Server
parent: Deutsch
nav_order: 32
---

# Plugin-Server (XOSP)

## Was ist XOSP?

Der **XOSP Plugin-Server** ist ein eigenständiger MCP-Server, der von XOS als Datenquelle genutzt wird. Er stellt Contexts, GraphQL-Queries und Mutations bereit und verbindet sich mit Memgraph, PostgreSQL oder anderen Backends.

Im Docker-Stack läuft XOSP als eigenständiger Container (`docker.io/xium/xosp`).

## Konfiguration via Env-Variablen

| Variable | Beschreibung | Default |
|---|---|---|
| `XOSP_SERVER_ADDR` | Adresse und Port | `:9100` |
| `XOSP_VAULT_URL` | Vault / OpenBao URL | — |
| `XOSP_VAULT_TOKEN` | Vault Token | — |
| `XOSP_ETCD_ENDPOINTS` | etcd Endpoints (kommagetrennt) | — |

Backend und DSN kommen aus etcd (`/xos/services/xosp/backend`, `/xos/services/xosp/dsn`).

## TLS und Fingerprint-Pinning

Beim Start (wenn Vault konfiguriert):

1. XOSP lädt den Private Key aus Vault (`secret/data/xosp/identity`) — oder generiert einen neuen
2. Erstellt ein Self-signed TLS-Zertifikat (SANs: `localhost`, `127.0.0.1`)
3. Berechnet den SHA256-Fingerprint des Public Key
4. Speichert Key + Fingerprint in Vault

XOS liest den Fingerprint aus etcd und verbindet sich per **Fingerprint-Pinning** — kein CA, kein Ablaufdatum.

> **Wichtig:** XOSP schreibt **nicht** selbst in etcd. URL und Fingerprint werden ausschließlich vom Admin / Setup-Job gesetzt (`make register`).

## MCP Tools

| Tool | Beschreibung |
|---|---|
| `xosp_ast` | XOS Abstract Syntax Tree laden |
| `xosp_me` | Aktuellen Benutzer und Gruppen abfragen |
| `xosp_query` | GraphQL-Query ausführen |
| `xosp_mutation` | GraphQL-Mutation ausführen |
| `xosp_save` | Mutation aus JSON-Map bauen und ausführen |
| `xosp_sources` | Verfügbare DSN-Namen auflisten |

## XOSP im Demo-Stack

XOSP wird via `make app` gestartet. Danach einmalig:

```bash
make register   # Fingerprint aus Vault lesen und in etcd schreiben
```

Ohne diesen Schritt startet XOS ohne XOSP-Verbindung.

## Unsecure-Modus (lokale Entwicklung)

```bash
xosp --unsecure
```

Plain HTTP, kein Vault, kein TLS. Nur für lokale Entwicklung.
