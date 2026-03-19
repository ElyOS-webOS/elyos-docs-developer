---
title: Creating Your First App
description: Creating an app project with the create-elyos-app CLI tool – project structure, manifest, first build
---

## Prerequisites

- [Bun](https://bun.sh) installed (`bun --version` ≥ 1.0)
- A running ElyOS instance (via Docker or locally) — required for uploading the app and testing within the ElyOS system (not just standalone mode). See: [Docker-based setup](/en/getting-started#docker-based-setup)

## Creating a Project

Use the ElyOS CLI to create a new app project:

```bash
bunx @elyos-dev/create-app
```

The interactive wizard walks you through the setup:

1. **App ID** — kebab-case identifier (e.g. `my-app`)
2. **Display name** — what users see
3. **Description** — short description
4. **Author** — `Name <email>` format
5. **Template** — `blank`, `basic`, `advanced`, `datatable`, or `sidebar`
6. **Permissions** — `database`, `notifications`, `remote_functions`

Or provide the name and template directly:

```bash
bunx @elyos-dev/create-app my-app --template blank
bunx @elyos-dev/create-app my-app --template basic
bunx @elyos-dev/create-app my-app --template advanced
bunx @elyos-dev/create-app my-app --template datatable --no-install
```

### Available Templates

| Template | Best for |
|---|---|
| `starter` | Clean slate — only the SDK, you decide what goes in |
| `basic` | Simple UI app, no server-side logic |
| `advanced` | With server functions and a Settings component |
| `datatable` | CRUD app with DataTable and server CRUD operations |
| `sidebar` | App with sidebar navigation (AppLayout mode, `menu.json`) |

#### `blank` template

The `blank` template is the most minimal starting point: only the SDK and required files are included. After selecting the template, the wizard asks three follow-up questions to optionally add the most common features:

- **Sidebar navigation?** — if yes, creates the `src/components/` folder with an `Overview.svelte` component and `menu.json`
- **Server functions?** — if yes, creates `server/functions.ts` with an example function
- **Database migrations?** — if yes, creates `migrations/001_init.sql` with a sample table definition. The ElyOS installer automatically runs migrations on install and prefixes table names with the plugin schema (`plugin__app_id`)
- **i18n translations?** — if yes, creates the `locales/` folder with `hu.json` and `en.json` files

Generated structure (with all options):

```
my-app/
├── manifest.json
├── src/
│   ├── App.svelte           # Main component
│   ├── main.ts              # Entry point
│   └── components/          # (if sidebar: yes)
│       └── Overview.svelte
├── server/                  # (if server functions: yes)
│   └── functions.ts
├── migrations/              # (if database migrations: yes)
│   └── 001_init.sql
├── locales/                 # (if i18n: yes)
│   ├── hu.json
│   └── en.json
└── assets/
    └── icon.svg
```

#### `basic` template

The `basic` template generates a simple single-page app. Ideal for beginners and simple apps.

Generated structure:

```
my-app/
├── manifest.json
├── src/
│   ├── App.svelte       # Main component
│   └── main.ts          # Entry point
├── locales/
│   ├── hu.json
│   └── en.json
└── assets/
    └── icon.svg
```

#### `advanced` template

The `advanced` template includes server-side logic and a Settings component. Ideal for apps that need server functions.

Generated structure:

```
my-app/
├── manifest.json
├── src/
│   ├── App.svelte       # Main component
│   ├── main.ts          # Entry point
│   └── components/
│       └── Settings.svelte
├── server/
│   └── functions.ts     # Server-side functions
├── locales/
│   ├── hu.json
│   └── en.json
└── assets/
    └── icon.svg
```

#### `datatable` template

The `datatable` template generates a full CRUD app with a DataTable component and server-side CRUD operations. Ideal for data management apps.

Generated structure:

```
my-app/
├── manifest.json
├── src/
│   ├── App.svelte       # Main component
│   ├── main.ts          # Entry point
│   └── components/
│       ├── DataTable.svelte
│       └── Settings.svelte
├── server/
│   └── functions.ts     # CRUD server functions
├── locales/
│   ├── hu.json
│   └── en.json
└── assets/
    └── icon.svg
```

#### `sidebar` template

The `sidebar` template generates a sidebar navigation layout where the app consists of multiple pages. ElyOS uses its `AppLayout` component for display — a navigation bar appears on the left side of the app window, with the selected page content on the right.

Generated structure:

```
my-app/
├── manifest.json
├── menu.json            # Sidebar navigation definition
├── src/
│   ├── App.svelte       # Main component (sidebar + routing)
│   └── components/
│       ├── Overview.svelte
│       └── Settings.svelte
├── locales/
│   ├── hu.json
│   └── en.json
└── migrations/          # Database migrations (if database permission)
    └── 001_init.sql
```

The `menu.json` defines the sidebar menu items:

```json
[
  { "id": "overview", "labelKey": "menu.overview", "component": "Overview" },
  { "id": "settings", "labelKey": "menu.settings", "component": "Settings" }
]
```

The `labelKey` values have no namespace — the system automatically prepends the `app:{id}.` prefix when looking up translations.

## Project Structure

The generated project structure:

```
my-app/
├── manifest.json        # App metadata (required)
├── package.json         # Dependencies and scripts
├── vite.config.ts       # Build configuration
├── tsconfig.json        # TypeScript configuration
├── src/
│   ├── main.ts          # Entry point, Mock SDK init
│   ├── App.svelte       # Main component
│   └── components/      # Sub-components (in advanced/datatable template)
├── server/              # Server-side functions (advanced/datatable)
│   └── functions.ts
├── locales/             # Translations
│   ├── hu.json
│   └── en.json
└── assets/
    └── icon.svg         # App icon
```

:::note
The `sdk-demo` example app is available in the monorepo: `examples/apps/sdk-demo/`. This is the most complete reference implementation — worth reviewing before development.
:::

## Manual Creation (Copying sdk-demo)

If you're working within the monorepo, you can copy the sdk-demo example:

```bash
cp -r examples/apps/sdk-demo examples/apps/my-app
cd examples/apps/my-app
# Modify manifest.json (id, name, description)
bun install
```

## Installing Dependencies

```bash
cd my-app
bun install
```

The `@elyos/sdk` package is available on the npm registry and JSR — it includes TypeScript type definitions and the developer Mock SDK.

## First Build

```bash
bun run build
```

This creates the `dist/index.iife.js` file — the bundle loaded by ElyOS.

## Next Steps

- [Developer workflow](/en/plugins-development/) — standalone dev mode and Mock SDK
- [manifest.json reference](/en/plugins-manifest/) — all fields in detail
- [Build and upload](/en/plugins-build/) — `.elyospkg` package and installation
