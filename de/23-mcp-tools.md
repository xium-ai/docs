---
layout: default
title: MCP Tools
parent: Deutsch
nav_order: 23
---

# MCP Tools — Übersicht

XOS stellt Claude eine definierte Menge von MCP-Tools zur Verfügung. Diese Tools sind das einzige Interface zwischen Claude und XOS.

## Tool-Liste

| Tool | Beschreibung |
|---|---|
| `xos_ast` | AST laden — Contexts, Felder, Relationen, Prompts |
| `xos_query` | Daten laden und ins Board rendern |
| `xos_render` | JSON direkt ins Board rendern |
| `xos_ui_change_required` | Änderungen als Preview ins Board |
| `xos_ui_save` | Daten speichern |
| `xos_delete` | Datensatz löschen |
| `xos_system_status` | Systemstatus abrufen |
| `xos_call` | Video- oder Audiocall starten |
| `xos_presence` | Online-Benutzer anzeigen |

---

# `xos_ast`

Lädt den vollständigen XOS-AST und gibt ihn an Claude zurück.

**Claude verwendet dieses Tool immer zuerst**, bevor er Daten abfragt oder ändert.

**Rückgabe:**
- Alle verfügbaren Contexts mit Feldern, Typen, list_fields
- Globale und context-spezifische AI-Prompts
- Locales, Relationen, Navigate-Definitionen

**Wichtig:** Das Schema wird intern verarbeitet. Claude antwortet immer nur mit: `"Schema geladen."`

---

# `xos_query`

Lädt Daten per GraphQL und rendert sie ins Board.

**Parameter:**
- `context` — Name des Contexts (z.B. `person_list`)
- `query` — GraphQL-Query-String

**Beispiel:**
```
context: "person_list"
query:   "{ person_list { id firstname lastname email city age net_worth } }"
```

**Rückgabe:** `"All data was successfully displayed in the UI."`

**Regeln:**
- Immer `xos_ast` zuerst aufrufen
- Exakt die `list_fields` des Contexts verwenden
- Keine Rohdaten in den Chat schreiben — Daten gehören ins Board

---

# `xos_render`

Rendert JSON-Daten direkt ins Board — ohne GraphQL.

**Parameter:**
- `context` — Context für das HTML-Template
- `json` — Daten als JSON-String

**Verwendung:** Wenn Daten bereits vorliegen (z.B. aus einer externen Quelle) und nur angezeigt werden sollen.

---

# `xos_ui_change_required`

Schreibt KI-Änderungen als Preview ins Board. Die Daten werden noch **nicht** gespeichert.

**Parameter:**
- `context` — Context des aktuellen Formulars
- `json` — Geänderte Felder als JSON-Objekt (nur die geänderten Felder — Rest wird aus dem Board übernommen)

**Workflow:** Immer nach `xos_query`, immer vor `xos_ui_save`.

---

# `xos_ui_save`

Speichert die aktuellen Board-Daten in die Datenbank.

**Keine Parameter.**

XOS liest automatisch die FormData aus dem Board, baut die GraphQL-Mutation und führt sie aus.

**Voraussetzung:** Der Benutzer muss explizit bestätigt haben, dass er speichern möchte.

---

# `xos_delete`

Löscht einen Datensatz dauerhaft.

**Parameter:**
- `context` — Context des zu löschenden Datensatzes
- `id` — ID des Datensatzes

**Voraussetzung:** Der Benutzer muss explizit bestätigt haben, dass er löschen möchte.

---

# `xos_system_status`

Gibt den aktuellen Systemstatus zurück:
- Aktiver Context
- Verfügbare Tools
- Verbindungsstatus

---

# `xos_call`

Startet einen Video- oder Audiocall mit einem anderen Benutzer über LiveKit.

**Parameter:**
- `callee` — Username des anzurufenden Benutzers

XOS generiert einen LiveKit-Token, schreibt eine Einladung in etcd (der Callee reagiert sofort über einen Watch) und rendert die Call-UI ins Board.

**Voraussetzung:** LiveKit muss im Stack laufen und konfiguriert sein.

---

# `xos_presence`

Zeigt welche Benutzer gerade online sind.

**Keine Parameter.**

**Voraussetzung:** Cluster/etcd muss aktiv sein.

---

# Der vollständige AI-Workflow

```
1. xos_ast                 → Schema laden (einmal pro Session)
2. xos_query               → Daten laden und anzeigen
3. [Benutzer möchte etwas ändern]
4. xos_ui_change_required  → Änderungen als Preview
5. [Benutzer bestätigt]
6. xos_ui_save             → Speichern
```
