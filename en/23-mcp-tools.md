---
layout: default
title: MCP Tools
parent: English
nav_order: 23
---

# MCP Tools — Overview

XOS provides Claude with a defined set of MCP tools. These tools are the only interface between Claude and XOS.

## Tool List

| Tool | Description |
|---|---|
| `xos_ast` | Load AST — contexts, fields, relations, prompts |
| `xos_query` | Load data and render into Board |
| `xos_render` | Render JSON directly into Board |
| `xos_ui_change_required` | Write changes as preview into Board |
| `xos_ui_save` | Save data |
| `xos_delete` | Delete a record |
| `xos_system_status` | Get system status |
| `xos_call` | Start a video or audio call |
| `xos_presence` | Show online users |

---

# `xos_ast`

Loads the complete XOS AST and returns it to Claude.

**Claude always uses this tool first**, before querying or changing data.

**Returns:**
- All available contexts with fields, types, list_fields
- Global and context-specific AI prompts
- Locales, relations, navigate definitions

**Important:** The schema is processed internally. Claude always responds with just: `"Schema geladen."`

---

# `xos_query`

Loads data via GraphQL and renders it into the Board.

**Parameters:**
- `context` — name of the context (e.g. `person_list`)
- `query` — GraphQL query string

**Example:**
```
context: "person_list"
query:   "{ person_list { id firstname lastname email city age net_worth } }"
```

**Returns:** `"All data was successfully displayed in the UI."`

**Rules:**
- Always call `xos_ast` first
- Use exactly the `list_fields` of the context
- Never write raw data into the chat — data belongs in the Board

---

# `xos_render`

Renders JSON data directly into the Board — without GraphQL.

**Parameters:**
- `context` — context for the HTML template
- `json` — data as JSON string

**Usage:** When data is already available (e.g. from an external source) and only needs to be displayed.

---

# `xos_ui_change_required`

Writes AI changes as a preview into the Board. Data is **not yet** saved.

**Parameters:**
- `context` — context of the current form
- `json` — changed fields as a JSON object (only changed fields — the rest is merged from the Board)

**Workflow:** Always after `xos_query`, always before `xos_ui_save`.

---

# `xos_ui_save`

Saves the current Board data to the database.

**No parameters.**

XOS automatically reads the FormData from the Board, builds the GraphQL mutation, and executes it.

**Prerequisite:** The user must have explicitly confirmed they want to save.

---

# `xos_delete`

Permanently deletes a record.

**Parameters:**
- `context` — context of the record to delete
- `id` — record ID

**Prerequisite:** The user must have explicitly confirmed they want to delete.

---

# `xos_system_status`

Returns the current system status:
- Active context
- Available tools
- Connection status

---

# `xos_call`

Starts a video or audio call with another user via LiveKit.

**Parameters:**
- `callee` — username of the user to call

XOS generates a LiveKit token, writes an invitation to etcd (the callee reacts immediately via a watch) and renders the call UI into the Board.

**Prerequisite:** LiveKit must be running and configured in the stack.

---

# `xos_presence`

Shows which users are currently online.

**No parameters.**

**Prerequisite:** Cluster/etcd must be active.

---

# The Complete AI Workflow

```
1. xos_ast                 → load schema (once per session)
2. xos_query               → load and display data
3. [user wants to change something]
4. xos_ui_change_required  → show changes as preview
5. [user confirms]
6. xos_ui_save             → save
```
