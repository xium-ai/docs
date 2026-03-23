---
layout: default
title: Secret Management
parent: Deutsch
nav_order: 37
---

# Secret Management

## Übersicht

XOS unterstützt zwei Secret-Backends:

| Provider | Verwendung |
|---|---|
| `env` | Umgebungsvariablen — einfach, für lokale Entwicklung |
| `vault` | HashiCorp Vault KV v2 — sicher, für Produktion |

## Provider: `env`

```toml
[secrets]
provider = "env"
```

XOS liest Secrets aus Umgebungsvariablen mit dem Präfix `XOS_`:

```bash
export XOS_PLUGIN_CERT="-----BEGIN CERTIFICATE-----..."
export XOS_PLUGIN_KEY="-----BEGIN EC PRIVATE KEY-----..."
```

Einfach für lokale Entwicklung — nicht für Produktion empfohlen.

## Provider: `vault`

```toml
[secrets]
provider = "vault"
url      = "http://localhost:8200"
token    = "hvs.xxxxxxxxxxxx"
path     = "xos"
```

XOS verwendet HashiCorp Vault KV v2. Alle Secrets werden unter einem einzigen Pfad als Felder gespeichert.

## Vault KV v2 Setup

### 1. KV v2 Mount aktivieren

```bash
vault secrets enable -path=secret kv-v2
```

### 2. Policy erstellen

```hcl
# xos-policy.hcl
path "secret/data/xos" {
  capabilities = ["create", "read", "update"]
}
```

```bash
vault policy write xos-policy xos-policy.hcl
```

### 3. Token erstellen

```bash
vault token create -policy=xos-policy -ttl=8760h
```

### 4. In `xos.toml` eintragen

```toml
[secrets]
provider = "vault"
url      = "http://localhost:8200"
token    = "hvs.TOKEN_AUS_SCHRITT_3"
path     = "xos"
```

## Automatisches Cert-Management

XOS verwaltet TLS-Zertifikate für den Plugin-Server automatisch über Vault:

| Key | Inhalt |
|---|---|
| `XOS_PLUGIN_CERT` | PEM-kodiertes TLS-Zertifikat |
| `XOS_PLUGIN_KEY` | PEM-kodierter privater Schlüssel |

Beim Start des Plugin-Servers:
1. XOS prüft ob `XOS_PLUGIN_CERT` in Vault existiert
2. Falls nicht: generiert selbst-signiertes Zertifikat und speichert es
3. Hub lädt das Zertifikat beim Connect und pinnt es

Es müssen keine Zertifikate manuell erstellt oder verwaltet werden.
