---
layout: default
title: Startup Flags
parent: Deutsch
nav_order: 4
---

# Startup Flags

XOS benötigt beim Start keine Konfigurationsdatei. Alle Parameter kommen aus **etcd** oder werden als Flag übergeben.

## Flags

| Flag | Pflicht | Beschreibung |
|---|---|---|
| `--etcd <endpoint>` | ✅ | etcd-Endpoint, z.B. `localhost:2379` |
| `--bridge` | – | Startet XOS als MCP-Bridge für Claude Desktop (kein Browser) |

## Normaler Start

```bash
./xos --etcd localhost:2379
```

XOS lädt die gesamte Konfiguration aus etcd und öffnet den Browser für den Login.

## Bridge-Modus (Claude Desktop)

```bash
./xos --bridge --etcd localhost:2379
```

Im Bridge-Modus öffnet XOS keinen Browser. Es kommuniziert über **MCP HTTP Streamable** mit Claude Desktop. Dieser Modus wird über `claude_desktop_config.json` konfiguriert — siehe [Demo Stack](45-demo.md).

## Konfiguration in etcd

Alles weitere — IAM, Vault, XOSP, HTML-Store — liegt in etcd.  
→ Siehe [Infrastruktur-Konfiguration](14-infra-conf.md)
