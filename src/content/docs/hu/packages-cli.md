---
title: "@elyos-dev/create-app"
description: Az ElyOS CLI eszköz, amellyel új alkalmazás projektet hozhatsz létre egyetlen paranccsal
next:
  link: /hu/plugins/
  label: Alkalmazás fejlesztés
---

A `@elyos-dev/create-app` egy interaktív parancssori eszköz, amellyel másodpercek alatt létrehozhatsz egy teljesen felkonfigurált ElyOS alkalmazás projektet. Nem kell kézzel beállítani a Vite konfigurációt, a TypeScript-et, a Mock SDK-t vagy a build scripteket — a CLI mindent előkészít.

```bash
bunx @elyos-dev/create-app
```

A wizard végigvezet az alapbeállításokon (alkalmazás ID, név, template, jogosultságok), majd legenerálja a projekt struktúrát. Ha tudod előre, mit szeretnél, megadhatod a paramétereket közvetlenül is:

```bash
bunx @elyos-dev/create-app my-app --template starter
```

## Elérhető template-ek

| Template | Leírás |
|---|---|
| `starter` | Minimális kiindulópont, opcionális funkciókkal |
| `basic` | Egyszerű egyoldalas alkalmazás |
| `advanced` | Szerver függvényekkel és Settings komponenssel |
| `datatable` | CRUD alkalmazás DataTable-lel |
| `sidebar` | Oldalsávos navigációval rendelkező alkalmazás |

## Kapcsolódó

- [Első alkalmazás létrehozása](/hu/plugins-getting-started/) — részletes útmutató a CLI használatához és a generált projekt struktúrához
