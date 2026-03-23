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
| `xos_schema` | AST laden — Contexts, Felder, Prompts |
| `xos_query` | Daten laden und ins Board rendern |
| `xos_render` | JSON direkt ins Board rendern |
| `xos_ui_change_required` | Änderungen als Preview ins Board |
| `xos_ui_save` | Daten speichern |
| `xos_system_status` | Systemstatus abrufen |
| `xos_contacts` | Kontakte laden |
| `xos_mail_compose` | Mail vorbereiten |
| `xos_mail_send` | Mail senden |

---

# `xos_schema`

Lädt den vollständigen XOS-AST und gibt ihn an Claude zurück.

**Claude verwendet dieses Tool immer zuerst**, bevor er Daten abfragt oder ändert.

**Rückgabe:**
- Alle verfügbaren Contexts mit Feldern, Typen, list_fields
- Globale und context-spezifische AI-Prompts
- Locales, Relationen, Navigate-Definitionen

**Wichtig:** Das Schema wird intern verarbeitet. Claude gibt keine Schema-Rohdaten im Chat aus.

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
- Immer `xos_schema` zuerst aufrufen
- Exakt die `list_fields` des Contexts verwenden
- Kein `xos_query` ohne vorheriges Schema-Laden

---

# `xos_render`

Rendert JSON-Daten direkt ins Board — ohne GraphQL.

**Parameter:**
- `context` — Context für das HTML-Template
- `json` — Daten als JSON-String

**Verwendung:** Wenn die Daten bereits vorliegen (z.B. aus einer externen Quelle) und nur angezeigt werden sollen.

```
context: "person_list"
json:    "[{\"id\":1,\"firstname\":\"Max\",\"lastname\":\"Mustermann\"}]"
```

---

# `xos_ui_change_required`

Schreibt KI-Änderungen als Preview ins Board. Die Daten werden noch **nicht** gespeichert.

**Parameter:**
- `context` — Context des aktuellen Formulars
- `changes` — Geänderte Felder als JSON-Objekt

**Beispiel:**
```
context: "person_detail"
changes: {"lastname": "Schmidt", "age": 35}
```

**Workflow:** Immer nach `xos_query`, immer vor `xos_ui_save`.

---

# `xos_ui_save`

Speichert die aktuellen Board-Daten in die Datenbank.

**Keine Parameter.**

XOS scraped automatisch die FormData aus dem Board, baut die GraphQL-Mutation und führt sie aus.

**Voraussetzung:** Der Benutzer muss explizit bestätigt haben, dass er speichern möchte.

---

# `xos_system_status`

Gibt den aktuellen Systemstatus zurück:
- Aktiver Context
- Verfügbare Tools
- DSN-Status

---

# `xos_contacts`

Lädt Kontakte aus dem konfigurierten Provider (CardDAV, Google, Microsoft).

**Parameter:**
- `query` — Suchbegriff (Name, E-Mail)

**Rückgabe:** Liste von Kontakten mit Name, E-Mail, Telefon.

---

# `xos_mail_compose`

Bereitet eine E-Mail vor und zeigt eine Preview im Board.

**Parameter:**
- `to` — Empfänger (Name oder E-Mail aus Kontakten)
- `subject` — Betreff
- `body` — Nachrichtentext

---

# `xos_mail_send`

Sendet die vorbereitete E-Mail aus der aktuellen Session.

**Keine Parameter.**

**Voraussetzung:** `xos_mail_compose` wurde aufgerufen und der Benutzer hat mit "senden" bestätigt.

---

# Der vollständige AI-Workflow

```
1. xos_schema              → Schema laden (einmal pro Session)
2. xos_query               → Daten laden und anzeigen
3. [Benutzer möchte etwas ändern]
4. xos_ui_change_required  → Änderungen als Preview
5. [Benutzer bestätigt]
6. xos_ui_save             → Speichern
```
