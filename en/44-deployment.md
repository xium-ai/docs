---
layout: default
title: Deployment
parent: English
nav_order: 44
---

# Deployment

## Demo Stack (Docker Compose)

The demo stack includes all components and is the recommended starting point.

### Steps

```bash
make infra     # Phase 1: Vault, Keycloak, PostgreSQL, etcd
# Wait ~30s for Vault + Keycloak to be ready

make app       # Phase 2: XOSP, MinIO, Memgraph, setup job
make register  # Once: write XOSP fingerprint to etcd
```

### Stack components

| Component | Port | Description |
|---|---|---|
| OpenBao (Vault) | 8200 | Secrets, PKI, XOSP identity |
| Keycloak | 8080 | IAM / OIDC |
| PostgreSQL | 5432 | Relational database |
| etcd | 2379 | Configuration source |
| MinIO | 9000 / 9001 | HTML templates (S3) |
| Memgraph | 7687 | Graph database for contexts |
| XOSP | 9100 | Plugin server |

### Demo credentials

| | |
|---|---|
| Users | `frank` / `tristan` |
| Password | `xos-dev-2026` |
| Vault token | `xos-dev-root-token` |
| Keycloak admin | `admin` / `xos-kc-bootstrap` |

## Starting XOS

```bash
./xos --etcd localhost:2379
```

XOS loads all configuration from etcd and opens the browser for login.

## Connecting Claude Desktop (Bridge)

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

Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Restart Claude Desktop. The hammer icon (🔨) shows the available XOS tools.

## make register — when is it needed?

`make register` reads the XOSP fingerprint from Vault and writes it to etcd. Run it:

- After the first `make app`
- After `make reset` (new Vault volume → new fingerprint)

As long as the Vault volume is preserved (`make down` / `make up`), the fingerprint stays constant.

## Linux

The demo stack runs identically on Linux:

```bash
make infra && make app && make register
./xos --etcd localhost:2379
```

Requirements: Docker Compose v2, `python3` for `make register`.
