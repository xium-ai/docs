---
layout: default
title: DSL Übersicht
parent: Deutsch
nav_order: 7
---

# DSL Übersicht

Die XOS DSL (Domain Specific Language) beschreibt Daten-Contexts in XML. Jede `.ctx.xml`-Datei definiert einen oder mehrere Contexts.

## Was ist ein Context?

Ein Context ist die zentrale Dateneinheit in XOS. Er beschreibt:
- Welche Datenbanktabelle (oder Quelle) verwendet wird
- Welche Felder angezeigt werden
- Wie die Daten formatiert und dargestellt werden
- Welche Navigation möglich ist
- Welche Regeln die KI beim Umgang mit diesen Daten befolgen soll

## Zwei Arten von Contexts

### collection
Eine **collection** ist eine Liste von Datensätzen — entspricht einem `SELECT` mit mehreren Zeilen.

```xml
<context name="person_list" kind="collection" source="person" dsn="demo"
         view="html/pages/person/person.table.html">
  <list_fields>id, firstname, lastname, email</list_fields>
  ...
</context>
```

- Benötigt `<list_fields>` — definiert welche Felder die KI abfragt
- Rendert typischerweise eine Tabelle

### entity
Eine **entity** ist ein einzelner Datensatz — entspricht einem `SELECT WHERE id = ?`.

```xml
<context name="person_detail" kind="entity" source="person" dsn="demo"
         view="html/pages/person/person.detail.html">
  ...
</context>
```

- Kein `<list_fields>` nötig
- Rendert typischerweise ein Detailformular
- Kann eine `gql_mutation` haben (für Updates)

## Datei-Struktur

```
ctx/
  person.ctx.xml      ← person_list + person_detail
  note.ctx.xml        ← note_list + note_detail
  global.conf.xml     ← Locales, globale AI-Prompts
```

Alle `.ctx.xml`-Dateien werden beim Start automatisch geladen und zu einem AST zusammengeführt. DSN-Quellen und Infrastruktur-Parameter kommen aus **etcd** — nicht mehr aus einer `infra.conf.xml`.

## global.conf.xml

Enthält locales und globale KI-Anweisungen, die für alle Contexts gelten:

```xml
<xos>
  <ai>
    <prompt name="system">XOS ist ein AI-first Enterprise-Datensystem...</prompt>
    <prompt name="query_behavior">Nutze immer exakt die list_fields...</prompt>
  </ai>

  <locale name="standard"  language="de-DE" currency="EUR"/>
  <locale name="us_market" language="en-US" currency="USD"/>
</xos>
```
