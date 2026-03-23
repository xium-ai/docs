---
layout: default
title: OAuth / Authentik Setup
parent: Deutsch
nav_order: 6
---

# OAuth / Authentik Setup

XOS verwendet OAuth 2.0 / OIDC zur Authentifizierung. Empfohlen wird [Authentik](https://goauthentik.io), es funktioniert aber mit jedem OIDC-konformen Provider.

## Authentik Konfiguration

### 1. OAuth2/OIDC Provider anlegen

In Authentik unter **Applications → Providers → Create**:

- Type: `OAuth2/OpenID Provider`
- Client ID: notieren (kommt in `xos.toml`)
- Client Secret: nicht benötigt (PKCE)
- Redirect URIs: `http://localhost:9999/callback`
- Scopes: `openid`, `profile`, `email` + custom scope `xium`

### 2. Custom Scope `xium` definieren

Unter **System → Scopes → Create**:

```
Name: xium
Expression:
return {
  "xos_ctx_dir":   "/dein/pfad/xosb/ctx",
  "xos_html_dir":  "/dein/pfad/xosb/html",
  "xos_html_type": "fs",
  "xos_xosp_url":  "https://localhost:9100",
  "xos_infra_path": "/dein/pfad/xosb/ctx/infra.conf.xml"
}
```

### 3. xos.toml befüllen

```toml
[auth]
client_id              = "DEINE_CLIENT_ID"
authorization_endpoint = "http://localhost:9000/application/o/authorize/"
token_endpoint         = "http://localhost:9000/application/o/token/"
redirect_uri           = "http://localhost:9999/callback"
scope                  = "openid profile email xium"
```

## JWT Claims

Nach erfolgreichem Login liest XOS folgende Claims aus dem JWT:

| Claim | Bedeutung |
|---|---|
| `xos_ctx_dir` | Context-Verzeichnis |
| `xos_html_dir` | HTML-Template-Verzeichnis |
| `xos_html_type` | `"fs"` oder `"embed"` |
| `xos_xosp_url` | Plugin-Server URL (http oder https) |
| `xos_infra_path` | Pfad zur infra.conf.xml |

JWT-Claims haben Vorrang vor `xos.toml`. So können verschiedene Benutzer unterschiedliche Contexts und Datenquellen erhalten.

## HTTPS für Plugin-Server

Wenn `xos_xosp_url` mit `https://` beginnt, aktiviert XOS automatisch mTLS für die Verbindung zum Plugin-Server. Das Zertifikat wird aus Vault geladen.
