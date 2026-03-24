---
layout: default
title: Encrypted Config
parent: English
nav_order: 5
---

# Encrypted Config (xos.enc)

XOS supports encrypted configuration. Sensitive values from `xos.toml` can be stored in an encrypted file `xos.enc`.

## Encryption

```bash
./xos crypt encrypt xos.toml
```

The password is prompted interactively. The encrypted file is saved as `xos.enc`.

## Usage

```bash
XOS_PASSWORD=myPassword ./xos
```

XOS automatically detects whether `xos.enc` is present and decrypts the configuration on startup.

## Security Note

`xos.toml` should be removed from the filesystem and version control after encryption. Only commit `xos.enc`.
