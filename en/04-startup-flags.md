---
layout: default
title: Startup Flags
parent: English
nav_order: 4
---

# Startup Flags

XOS requires no configuration file at startup. All parameters come from **etcd** or are passed as flags.

## Flags

| Flag | Required | Description |
|---|---|---|
| `--etcd <endpoint>` | ✅ | etcd endpoint, e.g. `localhost:2379` |
| `--bridge` | – | Starts XOS as MCP bridge for Claude Desktop (no browser) |

## Normal start

```bash
./xos --etcd localhost:2379
```

XOS loads all configuration from etcd and opens the browser for login.

## Bridge mode (Claude Desktop)

```bash
./xos --bridge --etcd localhost:2379
```

In bridge mode, XOS does not open a browser. It communicates with Claude Desktop via **MCP HTTP Streamable**. This mode is configured via `claude_desktop_config.json` — see [Demo Stack](45-demo.md).

## Configuration in etcd

Everything else — IAM, Vault, XOSP, HTML store — lives in etcd.  
→ See [Infrastructure Configuration](14-infra-conf.md)
