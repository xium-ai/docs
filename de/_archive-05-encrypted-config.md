---
layout: default
title: Encrypted Config
parent: Deutsch
nav_order: 5
---

# Encrypted Config (xos.enc)

XOS unterstützt verschlüsselte Konfiguration. Damit können sensible Werte aus `xos.toml` in eine verschlüsselte Datei `xos.enc` ausgelagert werden.

## Verschlüsselung

```bash
./xos crypt encrypt xos.toml
```

Das Passwort wird interaktiv abgefragt. Die verschlüsselte Datei wird als `xos.enc` gespeichert.

## Verwendung

```bash
XOS_PASSWORD=meinPasswort ./xos
```

XOS erkennt automatisch ob `xos.enc` vorhanden ist und entschlüsselt die Konfiguration beim Start.

## Sicherheitshinweis

`xos.toml` sollte nach der Verschlüsselung aus dem Dateisystem und aus der Versionskontrolle entfernt werden. Nur `xos.enc` committen.
