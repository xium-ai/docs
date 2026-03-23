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
| `xos_schema` | Load AST — contexts, fields, prompts |
| `xos_query` | Load data and render into Board |
| `xos_render` | Render JSON directly into Board |
| `xos_ui_change_required` | Write changes as preview into Board |
| `xos_ui_save` | Save data |
| `xos_system_status` | Get system status |
| `xos_contacts` | Load contacts |
| `xos_mail_compose` | Prepare a mail |
| `xos_mail_send` | Send a mail |

---

# `xos_schema`

Loads the complete XOS AST and returns it to Claude.

**Claude always uses this tool first**, before querying or changing data.

**Returns:**
- All available contexts with fields, types, list_fields
- Global and context-specific AI prompts
- Locales, relations, navigate definitions

**Important:** The schema is processed internally. Claude does not output raw schema data in the chat.

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
- Always call `xos_schema` first
- Use exactly the `list_fields` of the context
- Do not call `xos_query` without loading the schema first

---

# `xos_render`

Renders JSON data directly into the Board — without GraphQL.

**Parameters:**
- `context` — context for the HTML template
- `json` — data as JSON string

**Usage:** When data is already available (e.g. from an external source) and only needs to be displayed.

```
context: "person_list"
json:    "[{\"id\":1,\"firstname\":\"John\",\"lastname\":\"Smith\"}]"
```

---

# `xos_ui_change_required`

Writes AI changes as a preview into the Board. Data is **not yet** saved.

**Parameters:**
- `context` — context of the current form
- `changes` — changed fields as a JSON object

**Example:**
```
context: "person_detail"
changes: {"lastname": "Smith", "age": 35}
```

**Workflow:** Always after `xos_query`, always before `xos_ui_save`.

---

# `xos_ui_save`

Saves the current Board data to the database.

**No parameters.**

XOS automatically scrapes the FormData from the Board, builds the GraphQL mutation, and executes it.

**Prerequisite:** The user must have explicitly confirmed they want to save.

---

# `xos_system_status`

Returns the current system status:
- Active context
- Available tools
- DSN status

---

# `xos_contacts`

Loads contacts from the configured provider (CardDAV, Google, Microsoft).

**Parameters:**
- `query` — search term (name, email)

**Returns:** List of contacts with name, email, phone.

---

# `xos_mail_compose`

Prepares an email and shows a preview in the Board.

**Parameters:**
- `to` — recipient (name or email from contacts)
- `subject` — subject line
- `body` — message body

---

# `xos_mail_send`

Sends the prepared email from the current session.

**No parameters.**

**Prerequisite:** `xos_mail_compose` was called and the user has confirmed with "send".

---

# The Complete AI Workflow

```
1. xos_schema              → load schema (once per session)
2. xos_query               → load and display data
3. [user wants to change something]
4. xos_ui_change_required  → show changes as preview
5. [user confirms]
6. xos_ui_save             → save
```
