---
title: Packages
description: Overview of ElyOS ecosystem npm and JSR packages – SDK, CLI
---

ElyOS developer tools are available as standalone packages, so you don't need to clone the full monorepo to build applications.

## @elyos/sdk

The WebOS SDK provides TypeScript type definitions and a developer Mock SDK. Required for application development — it provides types for the `window.webOS` global object and the mock implementation needed for standalone development mode.

Available on:
- **npm:** `@elyos/sdk`
- **JSR:** `@elyos/sdk`

```bash
bun add @elyos/sdk
```

Full documentation: [SDK API Reference](/en/plugins-sdk/)

---

## @elyos-dev/create-app

An interactive CLI tool for scaffolding new ElyOS application projects. It walks you through project configuration and generates the initial structure based on your chosen template.

```bash
bunx @elyos-dev/create-app
```

Available templates: `blank`, `basic`, `advanced`, `datatable`, `sidebar`

Full documentation: [Getting Started](/en/plugins-getting-started/)
