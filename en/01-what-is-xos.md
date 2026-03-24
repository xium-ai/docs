---
layout: default
title: What is Xium OS?
parent: English
nav_order: 1
---

# What is Xium OS?

Xium OS (XOS) is an **AI-first enterprise data system**. It connects structured business data with an AI assistant — without traditional GUI development.

## The Idea

Traditional business software separates data, interface, and logic into distinct layers. XOS inverts this principle: the AI is the interface. The user describes what they want to see or do — XOS handles the rest.

## Core Principles

**Contexts instead of Screens**  
Data lives in so-called *Contexts*. A Context describes a data entity (e.g. `person`, `order`) as XML — with fields, types, relations and display rules. XOS automatically generates GraphQL queries and HTML views from this definition.

**MCP as the Backbone**  
XOS implements the [Model Context Protocol (MCP)](https://modelcontextprotocol.io). Claude Desktop (or any other MCP client) connects to XOS and receives a defined set of tools: load, display, edit, and save data.

**AI Workflow — No Direct Writes**  
Data changes always follow a three-step workflow:
1. `xos_query` — load data and display it in the Board
2. `xos_ui_change_required` — show AI changes as a preview
3. `xos_ui_save` — user confirms, data is saved

No direct database writes without user confirmation.

## Components

| Component | Description |
|---|---|
| **XOS** | Main process — MCP server, HTTP login, HTTPS dashboard, Board rendering |
| **XOSP (Plugin Server)** | Separate MCP server for data sources and GraphQL |
| **mcp-bridge** | stdio→MCP HTTP Streamable proxy for Claude Desktop |
| **DSL** | XML-based Context definition |
| **Board** | Browser-based display surface (HTTPS) |

## Technology Stack

- **Go** — server and MCP bridge
- **Browser-based** — HTTP login → Keycloak → HTTPS dashboard
- **GraphQL** — automatically generated from DSL
- **HTML Templates** — data binding via `field=`, `loop=`, `bind=` attributes
- **OpenBao** — secret management (Vault-compatible fork, MPL 2.0)
- **etcd** — central configuration source
- **OAuth 2.0 / OIDC** — authentication via Keycloak
