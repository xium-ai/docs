---
layout: default
title: Secret Management
parent: English
nav_order: 37
---

# Secret Management

## Overview

XOS supports two secret backends:

| Provider | Usage |
|---|---|
| `env` | Environment variables — simple, for local development |
| `vault` | HashiCorp Vault KV v2 — secure, for production |

## Provider: `env`

```toml
[secrets]
provider = "env"
```

XOS reads secrets from environment variables with the prefix `XOS_`:

```bash
export XOS_PLUGIN_CERT="-----BEGIN CERTIFICATE-----..."
export XOS_PLUGIN_KEY="-----BEGIN EC PRIVATE KEY-----..."
```

Simple for local development — not recommended for production.

## Provider: `vault`

```toml
[secrets]
provider = "vault"
url      = "http://localhost:8200"
token    = "hvs.xxxxxxxxxxxx"
path     = "xos"
```

XOS uses HashiCorp Vault KV v2. All secrets are stored as fields under a single path.

## Vault KV v2 Setup

### 1. Enable KV v2 Mount

```bash
vault secrets enable -path=secret kv-v2
```

### 2. Create Policy

```hcl
# xos-policy.hcl
path "secret/data/xos" {
  capabilities = ["create", "read", "update"]
}
```

```bash
vault policy write xos-policy xos-policy.hcl
```

### 3. Create Token

```bash
vault token create -policy=xos-policy -ttl=8760h
```

### 4. Add to `xos.toml`

```toml
[secrets]
provider = "vault"
url      = "http://localhost:8200"
token    = "hvs.TOKEN_FROM_STEP_3"
path     = "xos"
```

## Automatic Certificate Management

XOS automatically manages TLS certificates for the plugin server via Vault:

| Key | Content |
|---|---|
| `XOS_PLUGIN_CERT` | PEM-encoded TLS certificate |
| `XOS_PLUGIN_KEY` | PEM-encoded private key |

When the plugin server starts:
1. XOS checks whether `XOS_PLUGIN_CERT` exists in Vault
2. If not: generates a self-signed certificate and stores it
3. Hub loads the certificate on connect and pins it

No certificates need to be created or managed manually.
