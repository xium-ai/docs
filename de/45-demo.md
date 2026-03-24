---
layout: default
title: Demo Stack
parent: Deutsch
nav_order: 45
---

# Demo Stack

Der XOS Demo Stack ist eine vollständige Docker Compose Umgebung für lokale Entwicklung und Vorführungen.

**Repository:** [github.com/xium-ai/demo](https://github.com/xium-ai/demo)

## Voraussetzungen

- Docker mit Docker Compose v2
- `python3`
- XOS Binary — [Download](https://github.com/xium-ai/releases)

## Start

```bash
git clone https://github.com/xium-ai/demo.git
cd demo

make infra      # Phase 1: OpenBao, Keycloak, PostgreSQL, etcd
# ~30s warten bis OpenBao + Keycloak bereit sind

make app        # Phase 2: XOSP, MinIO, Memgraph, Redis, LiveKit, Setup-Job
make register   # XOSP Fingerprint in etcd schreiben (einmalig)

./xos --etcd localhost:2379
```

## Die zwei Phasen

Der Stack ist bewusst zweiphasig aufgebaut — analog zu realen Enterprise-Deployments, wo Vault und Keycloak immer vor den Anwendungen bereitstehen müssen.

**Phase 1 — Infrastruktur (`make infra`)**

Startet: OpenBao (Vault), Keycloak, PostgreSQL, etcd. Diese Dienste müssen vollständig bereit sein bevor Phase 2 startet.

**Phase 2 — Anwendung (`make app`)**

Startet: XOSP, MinIO, Memgraph, Redis, LiveKit und den Setup-Job. Der Setup-Job konfiguriert OpenBao, MinIO, Keycloak und schreibt alle Konfigurationskeys nach etcd.

## XOSP Fingerprint registrieren

```bash
make register
```

Liest den XOSP Fingerprint aus Vault und schreibt ihn in etcd. XOS nutzt ihn für Fingerprint-Pinning beim TLS-Connect zu XOSP. Nach einem `make reset` muss `make register` erneut ausgeführt werden.

## Login

| Benutzer | Passwort | Rolle |
|---|---|---|
| `frank` | `xos-dev-2026` | xos-admin |
| `tristan` | `xos-dev-2026` | xos-user |

## Endpunkte

| Service | URL |
|---|---|
| Keycloak Admin | http://keycloak.127.0.0.1.nip.io:8080/admin |
| OpenBao UI | http://openbao.127.0.0.1.nip.io:8200/ui |
| MinIO Console | http://localhost:9001 |
| Memgraph Lab | http://localhost:3000 |
| XOSP | https://localhost:9100 |

## Stack-Komponenten

| Komponente | Port | Beschreibung |
|---|---|---|
| OpenBao | 8200 | Secrets, PKI, XOSP Identity |
| Keycloak | 8080 | IAM / OIDC |
| PostgreSQL | 5432 | Relationale Datenbank |
| etcd | 2379 | Konfigurationsquelle |
| MinIO | 9000 / 9001 | HTML-Templates (S3) |
| Memgraph | 7687 | Graph-Datenbank für Contexts |
| Redis | 6379 | Cache |
| LiveKit | 7880 | Voice / Video |
| XOSP | 9100 | Plugin-Server |

## Makefile-Referenz

| Befehl | Beschreibung |
|---|---|
| `make infra` | Phase 1 starten |
| `make app` | Phase 2 starten |
| `make register` | XOSP Fingerprint registrieren (einmalig) |
| `make upload` | HTML-Templates nach MinIO hochladen |
| `make seed-ctx` | Context-Gruppen in Memgraph laden |
| `make install-ca` | OpenBao CA im Mac Keychain installieren (sudo) |
| `make status` | Laufende Container |
| `make down` | Stack stoppen (Volumes bleiben erhalten) |
| `make reset` | Stack + alle Volumes löschen |

## Reset

```bash
make reset    # Löscht alle Container und Volumes
make infra
make app
make register
```
