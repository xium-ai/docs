---
layout: default
title: Infrastruktur-Konfiguration
parent: Deutsch
nav_order: 14
---

# Infrastruktur-Konfiguration

XOS bezieht die gesamte Infrastruktur-Konfiguration aus **etcd**. Es gibt keine `xos.toml` und keine `infra.conf.xml` mehr.

## etcd als einzige Konfigurationsquelle

XOS benötigt beim Start nur den etcd-Endpoint:

```bash
./xos --etcd localhost:2379
```

Alle weiteren Parameter (IAM, Vault, XOSP, HTML-Store) werden aus etcd geladen. Fehlen erforderliche Keys, bricht XOS mit einer klaren Fehlermeldung ab.

## etcd Keys

### IAM (Keycloak)

| Key | Beschreibung |
|---|---|
| `/xos/config/iam_issuer_url` | Keycloak Realm URL |
| `/xos/config/iam_client_id` | OAuth Client ID |
| `/xos/config/iam_scope` | OAuth Scopes |

### Vault / Secrets

| Key | Beschreibung |
|---|---|
| `/xos/config/vault_url` | OpenBao / Vault URL |

### HTML-Store (MinIO / S3)

| Key | Beschreibung |
|---|---|
| `/xos/config/html_type` | `s3` oder `fs` |
| `/xos/config/html_dir` | S3: Bucket-Name / FS: Pfad |

### XOSP Plugin-Server

| Key | Beschreibung |
|---|---|
| `/xos/services/xosp/url` | XOSP URL (z.B. `https://localhost:9100`) |
| `/xos/services/xosp/fp` | XOSP TLS Fingerprint (SHA256 hex) |
| `/xos/services/xosp/backend` | Datenbank-Backend (`memgraph`, `postgres`) |
| `/xos/services/xosp/dsn` | Primäre DSN (z.B. Bolt URI für Memgraph) |
| `/xos/services/xosp/dsn_demo` | Weitere DSN (beliebig erweiterbar) |

## Keys manuell setzen

```bash
curl -X POST http://localhost:2379/v3/kv/put \
  -d "{\"key\":\"$(echo -n '/xos/services/xosp/url' | base64)\",
       \"value\":\"$(echo -n 'https://localhost:9100' | base64)\"}"
```

Im Demo-Stack übernimmt der Setup-Job (`make app`) alle Keys automatisch.

## XOSP Fingerprint

XOSP generiert beim ersten Start ein Ed25519-Keypair und speichert den Fingerprint in Vault. Der Admin schreibt ihn einmalig in etcd:

```bash
make register
```

XOS nutzt dann **Fingerprint-Pinning** (kein CA, kein Ablaufdatum) für die TLS-Verbindung zu XOSP. Solange das Vault-Volume erhalten bleibt, bleibt der Fingerprint konstant — `make register` muss nur nach einem `make reset` erneut ausgeführt werden.

## Watch

XOS beobachtet `/xos/services/xosp/url` und `/xos/services/xosp/fp` via etcd Watch. Ändert sich die URL oder der Fingerprint, verbindet sich XOS automatisch neu — ohne Neustart.
