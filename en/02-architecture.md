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
mcp-bridge (stdio → MCP HTTP Streamable)
        │
        ▼
XOS (:8000/mcp)              ← MCP server + HTTP/HTTPS server
        │
        ├── DSL / AST         ← XML Context definitions
        ├── GraphQL Engine    ← auto-generated from DSL
        ├── Board             ← browser-based (HTTPS)
        │
        └── MCP HTTP ────────▶ XOSP Plugin Server (:9100)
              (TLS fingerprint pinning)
                                      │
                                      ├── Memgraph (graph DB)
                                      └── PostgreSQL / other sources
```

## XOS

XOS is the central process. It starts an HTTP server for login and an HTTPS server for the dashboard (Board).

**Login flow:**
1. Browser opens `http://localhost:8000` → redirected to Keycloak
2. Keycloak login (OIDC)
3. After successful login: two browser tabs open with HTTPS

**Responsibilities:**
- Keycloak OIDC login (JWKS verification)
- Load configuration from etcd
- Load DSL and build AST
- Generate GraphQL schema
- Provide MCP tools for Claude
- Render HTML into the Board

## XOSP (Plugin Server)

XOSP is a separate MCP server for data sources and GraphQL execution. It connects to Memgraph, PostgreSQL, or other backends.

XOS communicates with XOSP over **MCP HTTP Streamable** with TLS fingerprint pinning — no CA, no expiry. The fingerprint is written to etcd once via `make register`.

**When is XOSP needed?**  
Always — XOSP is the data layer. XOS renders, XOSP delivers.

## mcp-bridge

Claude Desktop communicates via **stdio** (JSON-RPC). XOS is an HTTP server. The mcp-bridge translates between both protocols via MCP HTTP Streamable.

```
Claude Desktop
    stdio (JSON-RPC)
         │
    mcp-bridge
         │
    MCP HTTP Streamable
         │
    XOS (:8000/mcp)
```

Each new Claude chat session restarts the bridge — `xos_ast` must therefore be called once per session.

## Data Flow: Query

```
Claude → xos_query("person_list", "{ person_list { id firstname ... } }")
    → XOSP: GraphQL execute
    → Memgraph / PostgreSQL
    → JSON data
    → render HTML template
    → update Board in browser
    → "All data was successfully displayed in the UI."
```

## Data Flow: Mutation

```
Claude → xos_ui_change_required("person_detail", {"lastname": "Smith"})
    → merge stage data
    → render HTML template (preview)
    → update Board (not yet saved)

User confirms →

Claude → xos_ui_save()
    → read FormData from Board
    → build GraphQL mutation
    → XOSP: execute mutation
    → database updated
```

## Configuration

All infrastructure parameters (IAM, Vault, XOSP, HTML store) come from **etcd**.  
→ See [Infrastructure Configuration](14-infra-conf.md)

## Secret Management

Secrets (TLS keys, XOSP identity) are stored in **OpenBao** (Vault-compatible fork, MPL 2.0). XOS authenticates via the logged-in user's JWT (JWKS verification).
