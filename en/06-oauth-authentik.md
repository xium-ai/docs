---
layout: default
title: OAuth / Authentik Setup
parent: English
nav_order: 6
---

# OAuth / Authentik Setup

XOS uses OAuth 2.0 / OIDC for authentication. [Authentik](https://goauthentik.io) is recommended, but any OIDC-compliant provider works.

## Authentik Configuration

### 1. Create OAuth2/OIDC Provider

In Authentik under **Applications → Providers → Create**:

- Type: `OAuth2/OpenID Provider`
- Client ID: note this down (goes into `xos.toml`)
- Client Secret: not required (PKCE)
- Redirect URIs: `http://localhost:9999/callback`
- Scopes: `openid`, `profile`, `email` + custom scope `xium`

### 2. Define Custom Scope `xium`

Under **System → Scopes → Create**:

```
Name: xium
Expression:
return {
  "xos_ctx_dir":   "/your/path/xosb/ctx",
  "xos_html_dir":  "/your/path/xosb/html",
  "xos_html_type": "fs",
  "xos_xosp_url":  "https://localhost:9100",
  "xos_infra_path": "/your/path/xosb/ctx/infra.conf.xml"
}
```

### 3. Fill in xos.toml

```toml
[auth]
client_id              = "YOUR_CLIENT_ID"
authorization_endpoint = "http://localhost:9000/application/o/authorize/"
token_endpoint         = "http://localhost:9000/application/o/token/"
redirect_uri           = "http://localhost:9999/callback"
scope                  = "openid profile email xium"
```

## JWT Claims

After a successful login, XOS reads the following claims from the JWT:

| Claim | Meaning |
|---|---|
| `xos_ctx_dir` | Context directory |
| `xos_html_dir` | HTML template directory |
| `xos_html_type` | `"fs"` or `"embed"` |
| `xos_xosp_url` | Plugin server URL (http or https) |
| `xos_infra_path` | Path to infra.conf.xml |

JWT claims take precedence over `xos.toml`. This allows different users to receive different contexts and data sources.

## HTTPS for the Plugin Server

When `xos_xosp_url` starts with `https://`, XOS automatically enables mTLS for the connection to the plugin server. The certificate is loaded from Vault.
