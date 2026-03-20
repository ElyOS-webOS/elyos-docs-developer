---
title: "@elyos-dev/create-app"
description: The ElyOS CLI tool for scaffolding new application projects with a single command
next:
  link: /en/plugins/
  label: Application development
---

`@elyos-dev/create-app` is an interactive CLI tool that lets you create a fully configured ElyOS application project in seconds. No need to manually set up Vite, TypeScript, the Mock SDK, or build scripts — the CLI handles everything.

```bash
bunx @elyos-dev/create-app
```

The wizard walks you through the basic settings (app ID, name, template, permissions) and generates the project structure. If you already know what you want, you can pass parameters directly:

```bash
bunx @elyos-dev/create-app my-app --template starter
```

## Available templates

| Template | Description |
|---|---|
| `starter` | Minimal starting point with optional features |
| `basic` | Simple single-page application |
| `advanced` | With server functions and a Settings component |
| `datatable` | CRUD application with DataTable |
| `sidebar` | Application with sidebar navigation |

## Related

- [Getting Started](/en/plugins-getting-started/) — detailed guide on using the CLI and the generated project structure
