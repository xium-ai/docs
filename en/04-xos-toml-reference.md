---
layout: default
title: xos.toml Reference
parent: English
nav_order: 4
---

# xos.toml Reference

`xos.toml` is the central configuration file. It lives in the same directory as the XOS binary.

---

## [secrets]

| Key | Type | Description |
|---|---|---|
| `provider` | string | `"env"` or `"vault"` |
| `url` | string | Vault URL (only for `vault`) |
| `token` | string | Vault token (only for `vault`) |
| `path` | string | Vault KV path (e.g. `"xos"`) |

```toml
[secrets]
provider = "vault"
url      = "http://localhost:8200"
token    = "hvs.xxxxxxxxxxxx"
path     = "xos"
```

---

## [xos]

| Key | Type | Description |
|---|---|---|
| `ctx_dir` | string | Path to context directory (`.ctx.xml` files) |
| `ctx_zip` | string | Alternative: path to a ZIP file containing contexts |
| `html_type` | string | `"fs"` (filesystem) or `"embed"` (embedded) |
| `html_dir` | string | Path to HTML template directory |
| `infra` | string | Path to `infra.conf.xml` |
| `xosp_url` | string | Plugin server URL (can be overridden by JWT) |

```toml
[xos]
ctx_dir   = "/your/path/xosb/ctx"
html_type = "fs"
html_dir  = "/your/path/xosb/html"
infra     = "/your/path/xosb/ctx/infra.conf.xml"
xosp_url  = "http://localhost:9100"
```

---

## [auth]

| Key | Type | Description |
|---|---|---|
| `client_id` | string | OAuth client ID |
| `authorization_endpoint` | string | OAuth authorization URL |
| `token_endpoint` | string | OAuth token URL |
| `redirect_uri` | string | Callback URL (must be registered in OAuth app) |
| `scope` | string | OAuth scopes (at minimum: `openid profile email`) |

```toml
[auth]
client_id              = "w4GMl3g0vKLerJtg2W4OxXkgPXZBRY2vZUN8Rz35"
authorization_endpoint = "http://localhost:9000/application/o/authorize/"
token_endpoint         = "http://localhost:9000/application/o/token/"
redirect_uri           = "http://localhost:9999/callback"
scope                  = "openid profile email xium"
```

---

## [chat]

| Key | Type | Description |
|---|---|---|
| `ollama_url` | string | Ollama API URL for local LLMs |
| `model` | string | Model name (e.g. `"qwen2.5:14b"`) |
| `mcp_url` | string | MCP URL for the integrated chat |

```toml
[chat]
ollama_url = "http://localhost:11434"
model      = "qwen2.5:14b"
mcp_url    = "http://localhost:8000/mcp"
```

---

## [contacts]

| Key | Type | Description |
|---|---|---|
| `provider` | string | `"carddav"`, `"google"` or `"microsoft"` |

### [contacts.carddav]

```toml
[contacts.carddav]
url      = "https://p63-contacts.icloud.com"
username = "user@icloud.com"
password = "app-specific-password"
```

### [contacts.google]

```toml
[contacts.google]
token_file = "/your/path/.xos/google_token.json"
```

### [contacts.microsoft]

```toml
[contacts.microsoft]
client_id     = "AZURE_CLIENT_ID"
client_secret = "AZURE_CLIENT_SECRET"
tenant_id     = "AZURE_TENANT_ID"
token_file    = "/your/path/.xos/ms_token.json"
```

---

## [mail]

| Key | Type | Description |
|---|---|---|
| `smtp_host` | string | SMTP server |
| `smtp_port` | int | SMTP port (587 for STARTTLS) |
| `smtp_username` | string | SMTP username |
| `smtp_password` | string | SMTP password |
| `from_name` | string | Sender name |
| `from_email` | string | Sender email address |

```toml
[mail]
smtp_host     = "smtp.mail.me.com"
smtp_port     = 587
smtp_username = "user@icloud.com"
smtp_password = "app-specific-password"
from_name     = "John Smith"
from_email    = "user@icloud.com"
```

---

## JWT Override

Certain `[xos]` values can be overridden by JWT claims from the OAuth login. This enables per-user configuration without restarting.

| JWT Claim | Overrides |
|---|---|
| `xos_ctx_dir` | `[xos] ctx_dir` |
| `xos_html_dir` | `[xos] html_dir` |
| `xos_html_type` | `[xos] html_type` |
| `xos_xosp_url` | `[xos] xosp_url` |
| `xos_infra_path` | `[xos] infra` |
