---
layout: default
title: Deployment
parent: English
nav_order: 44
---

# Deployment

## What XOS actually requires

XOS and XOSP are lean Go binaries. The only hard runtime dependency is **etcd** — everything else is loaded from there.

| Component | Required by | Purpose |
|---|---|---|
| **etcd** | XOS, XOSP | Configuration source (only mandatory dependency) |
| **Keycloak** | XOS | IAM / OIDC login |
| **OpenBao (Vault)** | XOS, XOSP | Secrets, PKI, XOSP identity |
| **Memgraph** | XOSP | Graph database for contexts and relations |
| **MinIO** | XOS | HTML templates (S3-compatible store) |
| **PostgreSQL** | XOSP | Relational data (demo data, application data) |

### Optional components (demo stack)

| Component | Purpose |
|---|---|
| **Redis** | Cache |
| **LiveKit** | Voice / video (1:1 and group calls) |

---

## Starting XOS

```bash
./xos --etcd localhost:2379
```

XOS loads all configuration from etcd and opens the browser for login.

## Connecting Claude Desktop (Bridge)

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/path/to/xos",
      "args": ["--bridge", "--etcd", "localhost:2379"]
    }
  }
}
```

Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Restart Claude Desktop. The hammer icon (🔨) shows the available XOS tools.

---

## Demo stack

A complete Docker Compose stack is available for local testing and demonstrations.  
→ See [Demo Stack](45-demo.md)
