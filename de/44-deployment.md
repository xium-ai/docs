---
layout: default
title: Deployment
parent: Deutsch
nav_order: 44
---

# Deployment

## Was XOS wirklich benötigt

XOS und XOSP sind schlanke Go-Binaries. Die einzige harte Laufzeitabhängigkeit ist **etcd** — alles andere wird von dort bezogen.

| Komponente | Benötigt von | Zweck |
|---|---|---|
| **etcd** | XOS, XOSP | Konfigurationsquelle (einzige Pflichtabhängigkeit) |
| **Keycloak** | XOS | IAM / OIDC-Login |
| **OpenBao (Vault)** | XOS, XOSP | Secrets, PKI, XOSP-Identity |
| **Memgraph** | XOSP | Graph-Datenbank für Contexts und Relationen |
| **MinIO** | XOS | HTML-Templates (S3-kompatibler Store) |
| **PostgreSQL** | XOSP | Relationale Daten (Demo-Daten, Anwendungsdaten) |

### Optionale Komponenten (Demo-Stack)

| Komponente | Zweck |
|---|---|
| **Redis** | Cache |
| **LiveKit** | Voice / Video (1:1 und Gruppen-Calls) |

---

## XOS starten

```bash
./xos --etcd localhost:2379
```

XOS lädt die gesamte Konfiguration aus etcd und öffnet den Browser für den Login.

## Claude Desktop verbinden (Bridge)

In `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS):

```json
{
  "mcpServers": {
    "xos": {
      "command": "/pfad/zu/xos",
      "args": ["--bridge", "--etcd", "localhost:2379"]
    }
  }
}
```

Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Claude Desktop neu starten. Das Hammer-Symbol (🔨) zeigt die verfügbaren XOS-Tools.

---

## Demo-Stack

Für lokales Testen und Vorführungen steht ein vollständiger Docker Compose Stack bereit.  
→ Siehe [Demo Stack](45-demo.md)
