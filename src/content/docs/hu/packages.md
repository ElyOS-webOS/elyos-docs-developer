---
title: Kiegészítő csomagok
description: Az ElyOS ökoszisztéma npm és JSR csomagjainak áttekintése – SDK, CLI
---

Az ElyOS fejlesztői eszközei önálló csomagként is elérhetők, így nem szükséges a teljes monorepo-t klónozni az alkalmazásfejlesztéshez.

## @elyos/sdk

A WebOS SDK TypeScript típusdefiníciókat és fejlesztői Mock SDK-t tartalmaz. Alkalmazásfejlesztéshez szükséges — a `window.webOS` globális objektum típusait és a standalone fejlesztői módhoz szükséges mock implementációt biztosítja.

Elérhető:
- **npm:** `@elyos/sdk`
- **JSR:** `@elyos/sdk`

```bash
bun add @elyos/sdk
```

Részletes dokumentáció: [SDK API referencia](/hu/plugins-sdk/)

---

## @elyos-dev/create-app

Interaktív CLI eszköz új ElyOS alkalmazás projekt létrehozásához. Végigvezet a projekt beállításain és legenerálja a kiindulási struktúrát a választott template alapján.

```bash
bunx @elyos-dev/create-app
```

Elérhető template-ek: `blank`, `basic`, `advanced`, `datatable`, `sidebar`

Részletes dokumentáció: [Első alkalmazás létrehozása](/hu/plugins-getting-started/)
