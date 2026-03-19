---
title: Help Application
description: Context-sensitive help system for applications
---

:::caution[Under Development]
The Help application is currently under development. The basic infrastructure is complete (help button in windows, parameter passing), but the actual help content rendering system is not yet finished. This documentation describes the planned behavior and the already implemented parts.
:::

The Help application is a context-sensitive help system that allows each application to have its own help content. Help is accessible via the help button in the window header.

## Overview

The help system consists of two main parts:
- **Help button in windows** — A help button can appear in every window header
- **Help application** — Displays the context-sensitive help content

### Main features (planned)

- Context-sensitive help for each application
- Help button in the window header
- Automatic title bar update (e.g., "Help - Settings")
- Multilingual help content
- Search within help content
- Navigation between help topics

## File structure

```
apps/help/
├── index.svelte              # Main component (currently placeholder)
├── icon.svg                  # Help icon
├── components/               # Help components (under development)
├── stores/                   # Help state management (under development)
├── types/                    # Type definitions
└── utils/                    # Helper functions
```

## Using the help system

### 1. Adding a help ID to an application

The `helpId` field must be specified in the application metadata:

```typescript
// apps/settings/index.svelte or app registry
const appMetadata: AppMetadata = {
  appName: 'settings',
  title: 'Settings',
  defaultSize: { width: 800, height: 600 },
  icon: 'icon.svg',
  helpId: 1,  // Help identifier
  // ...
};
```

### 2. Displaying the help button

The help button automatically appears in the window header if `helpId` is specified in the application metadata:

```typescript
// Window.svelte (automatic)
{#if windowState.helpId}
  <WindowControlButton
    controlType="help"
    onClick={() => help(windowState.helpId)}
  />
{/if}
```

### 3. Opening help

When the user clicks the help button, the system opens the Help application with the appropriate `helpId` parameter:

```typescript
async function help(helpId: number | undefined) {
  const helpApp = await getAppByName('help');
  if (helpApp) {
    windowManager.openWindow(helpApp.appName, helpApp.title, helpApp, {
      helpId  // Parameter passing
    });
  }
}
```

## Current implementation

### Help application (index.svelte)

The current implementation is a simple placeholder:

```svelte
<script lang="ts">
  import { getParameter, getWindowId } from '$lib/services/client/appContext';
  import { getWindowManager } from '$lib/stores';

  const helpId = getParameter<number | undefined>('helpId', undefined);

  // Hardcoded example data (temporary)
  const helps = [
    {
      id: 1,
      title: 'Settings',
      content: 'Settings help content goes here.'
    },
    {
      id: 2,
      title: 'Users',
      content: 'Users help content goes here.'
    },
    // ...
  ];

  const help = helps.find((h) => h.id === helpId);

  // Update window title bar
  if (help) {
    const windowManager = getWindowManager();
    const windowId = getWindowId();
    const windowData = windowManager.windows.find((w) => w.id === windowId);
    if (windowData) {
      windowManager.updateWindowTitle(windowId, windowData.title + ' - ' + help.title);
    }
  }
</script>

<div>
  {#if helpId}
    {#if help}
      <p>{help.content}</p>
    {:else}
      <p>Help not found</p>
    {/if}
  {:else}
    <p>General help application content.</p>
  {/if}
</div>
```

## Types

### AppMetadata (helpId field)

```typescript
// lib/types/window.ts
export interface AppMetadata {
  appName: string;
  title: string;
  defaultSize: WindowSize;
  icon?: string;
  // ...
  helpId?: number;  // Help identifier
  // ...
}
```

### WindowState (helpId field)

```typescript
// lib/stores/windowStore.svelte.ts
export type WindowState = {
  id: string;
  appName: string;
  title: string;
  // ...
  helpId?: number;  // Help identifier
  parameters?: AppParameters;
  // ...
};
```

### AppParameters

```typescript
// lib/types/window.ts
export interface AppParameters {
  [key: string]: unknown;
}

// Example usage
const parameters: AppParameters = {
  helpId: 1
};
```

## Planned features

### Help content structure

The help content will be stored in the following structure (draft):

```typescript
interface HelpContent {
  id: number;
  appName: string;
  title: Record<string, string>;        // Multilingual title
  content: Record<string, string>;      // Multilingual content (Markdown)
  sections?: HelpSection[];             // Subsections
  keywords?: string[];                  // Search keywords
  relatedTopics?: number[];             // Related topic IDs
  createdAt: Date;
  updatedAt: Date;
}

interface HelpSection {
  id: string;
  title: Record<string, string>;
  content: Record<string, string>;
  order: number;
}
```

### Help content storage

Help content will be stored in the **database**. This allows:

- Adding new help content without restarting the application
- Modifying help content without build and deploy
- Dynamic content management through an admin interface
- Version tracking and audit trail for changes

**Format**: The content format (Markdown, HTML, or other) is not yet finalized, but **Markdown** is the likely choice for ease of editing.

**Database schema** (draft):

```sql
CREATE TABLE help_contents (
  id SERIAL PRIMARY KEY,
  help_id INTEGER UNIQUE NOT NULL,
  app_name VARCHAR(100) NOT NULL,
  title JSONB NOT NULL,              -- Multilingual title
  content JSONB NOT NULL,            -- Multilingual content (Markdown)
  sections JSONB,                    -- Subsections
  keywords TEXT[],                   -- Search keywords
  related_topics INTEGER[],          -- Related topic IDs
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

### Planned components

```
components/
├── HelpBrowser.svelte        # Main browser component
├── HelpContent.svelte        # Content renderer (Markdown)
├── HelpNavigation.svelte     # Navigation (TOC)
├── HelpSearch.svelte         # Search
└── RelatedTopics.svelte      # Related topics
```

### Server Actions (planned)

```typescript
// help.remote.ts

// Fetch help content
export const fetchHelpContent = command(
  v.object({ helpId: v.number() }),
  async ({ helpId }) => {
    const content = await helpRepository.findById(helpId);
    return { success: true, content };
  }
);

// Search help
export const searchHelp = command(
  v.object({ query: v.string() }),
  async ({ query }) => {
    const results = await helpRepository.search(query);
    return { success: true, results };
  }
);

// Fetch related topics
export const fetchRelatedTopics = command(
  v.object({ helpId: v.number() }),
  async ({ helpId }) => {
    const topics = await helpRepository.findRelated(helpId);
    return { success: true, topics };
  }
);
```

## Usage examples

### Adding help to a new application

1. **Determine the help ID** — Choose a unique help ID for your application (e.g., 10 = Settings, 20 = Users, etc.)

2. **Add the help ID to app metadata**

```typescript
// apps/my-app/index.svelte
const appMetadata: AppMetadata = {
  appName: 'my-app',
  title: 'My Application',
  defaultSize: { width: 800, height: 600 },
  helpId: 100,  // Unique help ID
  // ...
};
```

3. **Create help content** (when the system is ready)

```typescript
const helpContent: HelpContent = {
  id: 100,
  appName: 'my-app',
  title: {
    hu: 'Saját Alkalmazás Súgó',
    en: 'My App Help'
  },
  content: {
    hu: '# Saját Alkalmazás\n\nEz az alkalmazás...',
    en: '# My App\n\nThis application...'
  },
  keywords: ['app', 'help'],
  // ...
};
```

### Opening help programmatically

```typescript
import { getWindowManager } from '$lib/stores';
import { getAppByName } from '$lib/services/client/appRegistry';

async function openHelp(helpId: number) {
  const windowManager = getWindowManager();
  const helpApp = await getAppByName('help');

  if (helpApp) {
    windowManager.openWindow(helpApp.appName, helpApp.title, helpApp, {
      helpId
    });
  }
}

// Usage
openHelp(100);
```

## Help ID conventions

Suggested help ID ranges:

- **1-99**: System applications
  - 1: Settings
  - 2: Users
  - 3: Help (meta)
  - 10: Chat
  - 20: Logs
  - 30: Plugin Manager

- **100-999**: Built-in applications

- **1000+**: Plugin applications
  - 1000: General plugin help
  - 1001+: Custom plugin help

## Window title bar update

The Help application automatically updates the window title bar to include the help topic:

```typescript
// Original title: "Help"
// Updated title: "Help - Settings"

const windowManager = getWindowManager();
const windowId = getWindowId();
const windowData = windowManager.windows.find((w) => w.id === windowId);

if (windowData && help) {
  windowManager.updateWindowTitle(
    windowId,
    windowData.title + ' - ' + help.title
  );
}
```

## Multilingual support

Help content will be multilingual, integrated with the system's i18n system:

```typescript
// Get current locale
const { locale } = useI18n();

// Display localized content
const localizedTitle = help.title[locale] || help.title['hu'];
const localizedContent = help.content[locale] || help.content['hu'];
```

## Best practices

1. **Unique help IDs**: Give each application a unique help ID
2. **Meaningful ranges**: Use the suggested ID ranges
3. **Multilingual content**: Always provide Hungarian and English versions
4. **Markdown format**: Use Markdown for help content formatting
5. **Keywords**: Add relevant keywords for search
6. **Related topics**: Link related help topics together
7. **Screenshots**: Use screenshots for illustration
8. **Keep updated**: Keep help content up to date with application changes

## Development plan

Next steps for Help application development:

1. ✅ Help button in windows
2. ✅ Parameter passing (helpId)
3. ✅ Window title bar update
4. ⏳ Help content database schema
5. ⏳ Help content CRUD operations
6. ⏳ Markdown rendering
7. ⏳ Navigation and TOC
8. ⏳ Search functionality
9. ⏳ Related topics
10. ⏳ Admin interface for help editing

## Troubleshooting

### Help button not appearing

**Problem**: The help button is not visible in the window header.

**Solution**:
1. Check that `helpId` is specified in the application metadata
2. Check the `WindowState` object — does it contain the `helpId` field?
3. Check the `Window.svelte` component — is it rendering the help button?

### Help not opening

**Problem**: Nothing happens when clicking the help button.

**Solution**:
1. Check the browser console for errors
2. Check that the `help` application is registered in the app registry
3. Check the return value of `getAppByName('help')`

### Incorrect helpId parameter

**Problem**: The Help application cannot find the appropriate content.

**Solution**:
1. Check that `helpId` is correctly passed in the parameters
2. Check the return value of `getParameter<number>('helpId')`
3. Check that help content exists for the given ID
