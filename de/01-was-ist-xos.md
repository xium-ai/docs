---
layout: default
title: Was ist Xium OS?
parent: Deutsch
nav_order: 1
---

# Was ist Xium OS?

Xium OS (XOS) ist ein **AI-first Enterprise-Datensystem**. Es verbindet strukturierte Unternehmensdaten mit einem KI-Assistenten — ohne klassische GUI-Entwicklung.

## Die Idee

Traditionelle Business-Software trennt Daten, Oberfläche und Logik in separate Schichten. XOS dreht dieses Prinzip um: Die KI ist die Oberfläche. Der Anwender beschreibt, was er sehen oder tun möchte — XOS erledigt den Rest.

## Kernprinzipien

**Contexts statt Screens**  
Daten leben in sogenannten *Contexts*. Ein Context beschreibt eine Datenentität (z.B. `person`, `order`) als XML — mit Feldern, Typen, Relationen und Anzeigeregeln. XOS generiert daraus automatisch GraphQL-Abfragen und HTML-Views.

**MCP als Rückgrat**  
XOS implementiert das [Model Context Protocol (MCP)](https://modelcontextprotocol.io). Claude Desktop (oder ein anderer MCP-Client) verbindet sich mit XOS und erhält eine definierte Werkzeugpalette: Daten laden, anzeigen, bearbeiten, speichern.

**AI-Workflow — kein direktes Schreiben**  
Änderungen an Daten folgen immer einem dreistufigen Workflow:
1. `xos_query` — Daten laden und im Board anzeigen
2. `xos_ui_change_required` — KI-Änderungen als Preview einblenden
3. `xos_ui_save` — Benutzer bestätigt, Daten werden gespeichert

Kein direktes Schreiben in die Datenbank ohne Benutzerbestätigung.

## Komponenten

| Komponente | Beschreibung |
|---|---|
| **XOS** | Hauptprozess — MCP-Server, HTTP-Login, HTTPS-Dashboard, Board-Rendering |
| **XOSP (Plugin-Server)** | Separater MCP-Server für Datenquellen und GraphQL |
| **mcp-bridge** | stdio→HTTP Proxy für Claude Desktop |
| **DSL** | XML-basierte Context-Definition |
| **Board** | Browser-basierte Anzeigeoberfläche (HTTPS) |

## Technologie-Stack

- **Go** — Server und MCP-Bridge
- **Browser-basiert** — HTTP Login → Keycloak → HTTPS Dashboard
- **GraphQL** — automatisch aus DSL generiert
- **HTML Templates** — data-binding via `field=`, `loop=`, `bind=` Attribute
- **OpenBao** — Secret Management (Vault-kompatibler Fork, MPL 2.0)
- **etcd** — zentrale Konfigurationsquelle
- **OAuth 2.0 / OIDC** — Authentifizierung via Keycloak
