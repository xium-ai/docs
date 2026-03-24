---
layout: default
title: Infrastructure Configuration
parent: English
nav_order: 14
---

# Infrastructure Configuration

XOS reads all infrastructure configuration from **etcd**. There is no `xos.toml` and no `infra.conf.xml`.

## etcd as single source of truth

XOS only needs the etcd endpoint at startup:

```bash
./xos --etcd localhost:2379
```

All further parameters are loaded from etcd. If required keys are missing, XOS exits with a clear error message.

---

## etcd Keys

### IAM (Keycloak)

XOS uses Keycloak as its OIDC provider. The connection is configured entirely via etcd — no `xos.toml [auth]` anymore.

| Key | Description |
|---|---|
| `/xos/config/iam_issuer_url` | Keycloak realm URL, e.g. `http://keycloak:8080/realms/xos` |
| `/xos/config/iam_client_id` | OAuth client ID |
| `/xos/config/iam_scope` | OAuth scopes, e.g. `openid profile email` |

In the demo stack the setup job (`make app`) sets these keys automatically.

**Redirect URI:** XOS uses `http://localhost:9999/callback` as a fixed value — this must be registered in the Keycloak client.

### Vault / Secrets

| Key | Description |
|---|---|
| `/xos/config/vault_url` | OpenBao / Vault URL, e.g. `http://openbao:8200` |

XOS authenticates against Vault using the logged-in user's JWT (JWKS verification). The Vault token is not stored in etcd.

### HTML Store (MinIO / S3)

| Key | Description |
|---|---|
| `/xos/config/html_type` | `s3` or `fs` |
| `/xos/config/html_dir` | S3: bucket name / FS: path |

### XOSP Plugin Server

| Key | Description |
|---|---|
| `/xos/services/xosp/url` | XOSP URL, e.g. `https://xosp:9100` |
| `/xos/services/xosp/fp` | XOSP TLS fingerprint (SHA256 hex) |
| `/xos/services/xosp/backend` | Database backend (`memgraph`, `postgres`) |
| `/xos/services/xosp/dsn` | Primary DSN (e.g. Bolt URI for Memgraph) |

> **Note:** DSN management via etcd is being reworked. Connection strings belong in Vault long-term. Details to follow.

---

## Setting keys manually

```bash
curl -X POST http://localhost:2379/v3/kv/put \
  -d "{\"key\":\"$(echo -n '/xos/config/iam_issuer_url' | base64)\",
       \"value\":\"$(echo -n 'http://keycloak:8080/realms/xos' | base64)\"}"
```

In the demo stack, the setup job (`make app`) sets all keys automatically.

---

## XOSP Fingerprint

XOSP generates an Ed25519 keypair on first start and stores the fingerprint in Vault. The admin writes it to etcd once:

```bash
make register
```

XOS uses **fingerprint pinning** (no CA, no expiry) for the TLS connection to XOSP. As long as the Vault volume is preserved, the fingerprint stays constant — `make register` only needs to be run again after `make reset`.

---

## Watch

XOS watches `/xos/services/xosp/url` and `/xos/services/xosp/fp` via etcd watch. If URL or fingerprint changes, XOS reconnects automatically — no restart needed.
