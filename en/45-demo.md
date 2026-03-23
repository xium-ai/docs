---
layout: default
title: Demo Stack
parent: English
nav_order: 45
---

# Demo Stack

The XOS demo stack is a complete Docker Compose environment for quick onboarding and demonstrations.

**Repository:** [github.com/xium-ai/demo](https://github.com/xium-ai/demo)

## Prerequisites

- Docker with Docker Compose v2
- `python3`
- XOS binary — [Download](https://github.com/xium-ai/releases)

## Getting started

```bash
git clone https://github.com/xium-ai/demo.git
cd demo

make infra      # Phase 1: Vault, Keycloak, PostgreSQL, etcd
# Wait ~30s

make app        # Phase 2: XOSP, MinIO, Memgraph, setup job
make register   # Write XOSP fingerprint to etcd (once)

./xos --etcd localhost:2379
```

## Phases

The stack is split into two phases — mirroring real enterprise deployments where Vault and Keycloak always run before applications.

**Phase 1 — Infrastructure (`make infra`)**

Starts the base services: OpenBao (Vault), Keycloak, PostgreSQL, etcd. These must be fully ready before Phase 2 starts.

**Phase 2 — Application (`make app`)**

Starts XOSP, MinIO, Memgraph, Redis, LiveKit and the setup job. The setup job configures Vault, MinIO, Keycloak and writes all configuration keys to etcd.

## XOSP Fingerprint

After `make app`, register the XOSP fingerprint once:

```bash
make register
```

This reads the fingerprint from Vault and writes it to etcd. XOS uses it for fingerprint pinning on the TLS connection to XOSP. After a `make reset`, run `make register` again.

## Login

| User | Password | Role |
|---|---|---|
| `frank` | `xos-dev-2026` | xos-admin |
| `tristan` | `xos-dev-2026` | xos-user |

## Endpoints

| Service | URL |
|---|---|
| Keycloak Admin | http://keycloak.127.0.0.1.nip.io:8080/admin |
| OpenBao UI | http://openbao.127.0.0.1.nip.io:8200/ui |
| MinIO Console | http://localhost:9001 |
| Memgraph Lab | http://localhost:3000 |
| XOSP | https://localhost:9100 |

## Stack components

| Component | Port | Description |
|---|---|---|
| OpenBao | 8200 | Secrets, PKI, XOSP identity |
| Keycloak | 8080 | IAM / OIDC |
| PostgreSQL | 5432 | Relational database |
| etcd | 2379 | Configuration source |
| MinIO | 9000 / 9001 | HTML templates (S3) |
| Memgraph | 7687 | Graph database for contexts |
| Redis | 6379 | Cache |
| LiveKit | 7880 | Voice / video |
| XOSP | 9100 | Plugin server |

## Makefile reference

| Command | Description |
|---|---|
| `make infra` | Start Phase 1 |
| `make app` | Start Phase 2 |
| `make register` | Register XOSP fingerprint (once) |
| `make upload` | Upload HTML templates to MinIO |
| `make seed-ctx` | Load context groups into Memgraph |
| `make install-ca` | Install OpenBao CA in Mac keychain (sudo) |
| `make status` | Show running containers |
| `make down` | Stop stack |
| `make reset` | Delete stack and volumes |

## Reset

```bash
make reset    # Deletes all containers and volumes
make infra
make app
make register
```
