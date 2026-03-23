---
layout: default
title: Plugin Server
parent: English
nav_order: 32
---

# Plugin Server (XOSP)

## What is XOSP?

The **XOSP plugin server** is a standalone MCP server used by XOS as a data source. It provides contexts, GraphQL queries and mutations, and connects to Memgraph, PostgreSQL or other backends.

In the Docker stack, XOSP runs as a dedicated container (`docker.io/xium/xosp`).

## Configuration via environment variables

| Variable | Description | Default |
|---|---|---|
| `XOSP_SERVER_ADDR` | Address and port | `:9100` |
| `XOSP_VAULT_URL` | Vault / OpenBao URL | — |
| `XOSP_VAULT_TOKEN` | Vault token | — |
| `XOSP_ETCD_ENDPOINTS` | etcd endpoints (comma-separated) | — |

Backend and DSN come from etcd (`/xos/services/xosp/backend`, `/xos/services/xosp/dsn`).

## TLS and Fingerprint Pinning

On startup (when Vault is configured):

1. XOSP loads the private key from Vault (`secret/data/xosp/identity`) — or generates a new one
2. Creates a self-signed TLS certificate (SANs: `localhost`, `127.0.0.1`)
3. Computes the SHA256 fingerprint of the public key
4. Stores key + fingerprint in Vault

XOS reads the fingerprint from etcd and connects via **fingerprint pinning** — no CA, no expiry.

> **Important:** XOSP does **not** write to etcd itself. URL and fingerprint are set exclusively by the admin / setup job (`make register`).

## MCP Tools

| Tool | Description |
|---|---|
| `xosp_ast` | Load XOS abstract syntax tree |
| `xosp_me` | Query current user and groups |
| `xosp_query` | Execute GraphQL query |
| `xosp_mutation` | Execute GraphQL mutation |
| `xosp_save` | Build and execute mutation from JSON map |
| `xosp_sources` | List available DSN names |

## XOSP in the demo stack

XOSP is started via `make app`. Afterwards, once:

```bash
make register   # Read fingerprint from Vault and write to etcd
```

Without this step, XOS starts without an XOSP connection.

## Unsecure mode (local development)

```bash
xosp --unsecure
```

Plain HTTP, no Vault, no TLS. For local development only.
