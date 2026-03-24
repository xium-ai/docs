---
layout: default
title: Architektur
parent: Deutsch
nav_order: 2
---

# Architektur-Übersicht

## Komponenten-Diagramm

```
Claude Desktop (stdio)
        │
        ▼
mcp-bridge (stdio → MCP HTTP Streamable)
        │
        ▼
XOS (:8000/mcp)              ← MCP-Server + HTTP/HTTPS Server
        │
        ├── DSL / AST         ← XML Context-Definitionen
        ├── GraphQL Engine    ← auto-generiert aus DSL
        ├── Board             ← Browser-basiert (HTTPS)
        │
        └── MCP HTTP ────────▶ XOSP Plugin-Server (:9100)
              (TLS Fingerprint-Pinning)
                                      │
                                      ├── Memgraph (Graph-DB)
                                      └── PostgreSQL / weitere Quellen
```

## XOS

XOS ist der zentrale Prozess. Er startet einen HTTP-Server für den Login und einen HTTPS-Server für das Dashboard (Board).

**Login-Ablauf:**
1. Browser öffnet `http://localhost:8000` → Weiterleitung zu Keycloak
2. Keycloak-Login (OIDC)
3. Nach erfolgreicher Anmeldung: zwei Browser-Tabs öffnen sich mit HTTPS

**Aufgaben:**
- Keycloak OIDC-Login (JWKS-Verifikation)
- Konfiguration aus etcd laden
- DSL laden und AST aufbauen
- GraphQL-Schema generieren
- MCP-Tools für Claude bereitstellen
- HTML ins Board rendern

## XOSP (Plugin-Server)

XOSP ist ein separater MCP-Server für Datenquellen und GraphQL-Ausführung. Er verbindet sich mit Memgraph, PostgreSQL oder anderen Backends.

XOS kommuniziert mit XOSP über **MCP HTTP Streamable** mit TLS Fingerprint-Pinning — kein CA, kein Ablaufdatum. Der Fingerprint wird einmalig via `make register` in etcd geschrieben.

**Wann wird XOSP benötigt?**  
Immer — XOSP ist die Datenschicht. XOS rendert, XOSP liefert.

## mcp-bridge

Claude Desktop kommuniziert per **stdio** (JSON-RPC). XOS ist ein HTTP-Server. Die mcp-bridge übersetzt zwischen beiden Protokollen via MCP HTTP Streamable.

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

Jede neue Claude-Chat-Session startet die Bridge neu — `xos_ast` muss daher einmal pro Session aufgerufen werden.

## Datenfluss: Query

```
Claude → xos_query("person_list", "{ person_list { id firstname ... } }")
    → XOSP: GraphQL Execute
    → Memgraph / PostgreSQL
    → JSON-Daten
    → HTML-Template rendern
    → Board im Browser aktualisieren
    → "All data was successfully displayed in the UI."
```

## Datenfluss: Mutation

```
Claude → xos_ui_change_required("person_detail", {"lastname": "Müller"})
    → Stage-Daten mergen
    → HTML-Template rendern (Preview)
    → Board aktualisieren (noch nicht gespeichert)

Benutzer bestätigt →

Claude → xos_ui_save()
    → FormData aus Board lesen
    → GraphQL Mutation bauen
    → XOSP: Mutation ausführen
    → Datenbank aktualisiert
```

## Konfiguration

Alle Infrastruktur-Parameter (IAM, Vault, XOSP, HTML-Store) kommen aus **etcd**.  
→ Siehe [Infrastruktur-Konfiguration](14-infra-conf.md)

## Secret Management

Secrets (TLS-Keys, XOSP-Identity) werden in **OpenBao** (Vault-kompatibler Fork, MPL 2.0) gespeichert. XOS authentifiziert sich über das JWT des eingeloggten Benutzers (JWKS).
