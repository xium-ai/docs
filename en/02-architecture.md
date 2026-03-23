---
layout: default
title: Architecture
parent: English
nav_order: 2
---

# Architecture Overview

## Component Diagram

```
Claude Desktop (stdio)
        │
        ▼
mcp-bridge (:stdio → HTTP)
        │
        ▼
XOS Hub (:8000/mcp)          ← MCP server + Wails desktop
        │
        ├── DSL / AST         ← XML Context definitions
        ├── GraphQL Engine    ← auto-generated from DSL
        ├── HTML Board        ← Wails window
        │
        └── HTTPS/mTLS ──────▶ xosp Plugin Server (:9100)
                                      │
                                      └── custom data sources
```

## XOS Hub

The Hub is the central process. It starts as a Wails desktop app and simultaneously provides an MCP server on port `8000`.

**Responsibilities:**
- OAuth login (Authentik or local mode)
- Load DSL and build AST
- Generate GraphQL schema
- Manage database connections (DSN registry)
- Provide MCP tools for Claude
- Render HTML into the Wails Board

## xosp (Plugin Server)

The plugin server is a separate process that also runs as an MCP server — but with its own context directory and data sources.

The Hub connects to the plugin server via **HTTPS/mTLS**. The TLS certificate is automatically generated and stored in Vault.

**When is xosp needed?**  
When data sources need to run separately from the Hub process — e.g. on a different server or as an isolated microservice.

## mcp-bridge

Claude Desktop communicates via **stdio** (JSON-RPC over stdin/stdout). The XOS Hub is an HTTP server. The `mcp-bridge` translates between both protocols.

```
Claude Desktop
    stdio (JSON-RPC)
         │
    mcp-bridge
         │
    HTTP POST /mcp
         │
    XOS Hub
```

## Data Flow: Query

```
Claude → xos_query("person_list", "{ person_list { id firstname ... } }")
    → GraphQL Execute
    → SQL SELECT ... FROM person
    → JSON data
    → render HTML template
    → update Wails Board
    → "Data displayed in Board"
```

## Data Flow: Mutation

```
Claude → xos_ui_change_required("person_detail", {"lastname": "Smith"})
    → merge stage data
    → render HTML template (preview)
    → update Wails Board (not yet saved)

User confirms →

Claude → xos_ui_save()
    → scrape FormData from Board
    → build GraphQL mutation
    → SQL UPDATE ... WHERE id = ?
    → success
```

## Secret Management

XOS supports two secret backends:

| Provider | Usage |
|---|---|
| `env` | Environment variables — for local development |
| `vault` | HashiCorp Vault KV v2 — for production |

All keys are stored as fields under a single Vault path (e.g. `secret/xos`).
