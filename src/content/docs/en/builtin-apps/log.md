---
title: Log Application
description: Viewing and filtering system and error logs
---

The Log application allows viewing, filtering, and analyzing system logs. It currently supports displaying error logs; activity logs are under development.

## Overview

The Log application consists of two main parts:
- **Error Logs** — System errors, warnings, and debug messages
- **Activity Logs** — User action logging (under development)

### Main features

- Display error logs in tabular form
- Filter by log level (debug, info, warn, error, fatal)
- Filter by source
- Sort by column
- Pagination for large datasets
- Color-coded log levels
- Permission-based access

## File structure

```
apps/log/
├── index.svelte              # Main layout (AppLayout + menu)
├── menu.json                 # Menu definition (Error, Activity)
├── error-logs.remote.ts      # Server action for fetching error logs
└── components/
    ├── ErrorLog.svelte       # Error logs table with filters
    ├── ActivityLog.svelte    # Activity logs (placeholder)
    └── errorLogColumns.ts    # Table column definitions
```

## Menu structure

The `menu.json` defines the application menu items:

```json
[
  {
    "labelKey": "menu.error",
    "href": "#error",
    "icon": "ShieldAlert",
    "component": "ErrorLog",
    "requiredPermission": "log.error.view"
  },
  {
    "labelKey": "menu.activity",
    "href": "#activity",
    "icon": "ShieldAlert",
    "component": "ActivityLog"
  }
]
```

**Permissions:**
- `log.error.view` — View error logs (admin only)
- Activity Log is currently not permission-restricted

## Server Actions

### `error-logs.remote.ts`

#### `fetchErrorLogs` (command)

Fetch error logs with filtering and pagination parameters.

```typescript
const result = await fetchErrorLogs({
  page: 1,
  pageSize: 20,
  level: ['error', 'fatal'],  // optional, string or string[]
  source: 'server',            // optional
  search: 'database',          // optional (currently unused)
  sortBy: 'timestamp',         // optional, default: 'timestamp'
  sortOrder: 'desc'            // optional, 'asc' or 'desc'
});
```

**Validation:**
- `page`: minimum 1, default: 1
- `pageSize`: between 1-100, default: 20
- `level`: 'debug' | 'info' | 'warn' | 'error' | 'fatal' (single or array)
- `source`: string
- `sortBy`: string
- `sortOrder`: 'asc' | 'desc'

**Return value:**

```typescript
{
  success: true,
  data: LogEntry[],
  pagination: {
    page: number,
    pageSize: number,
    totalCount: number,
    totalPages: number
  }
}
```

## Components

### ErrorLog.svelte

Display error logs using the DataTable component with filters and pagination.

**State:**

```typescript
let data = $state<LogEntry[]>([]);
let loading = $state(false);
let paginationInfo = $state<PaginationInfo>({...});
let levelFilter = $state<string[]>([]);
let sourceFilter = $state('');
let tableState = $state<DataTableState>({
  page: 1,
  pageSize: 20,
  sortBy: 'timestamp',
  sortOrder: 'desc'
});
```

**Filters:**

1. **Level filter** — Faceted filter component (multi-select: debug, info, warn, error, fatal)
2. **Source filter** — Input field with 300ms debounce, partial match (ILIKE)

**Reactivity:**

```typescript
$effect(() => {
  tableState;
  levelFilter;
  sourceFilter;
  untrack(() => loadData());
});
```

### errorLogColumns.ts

Table column definitions for TanStack Table.

**Columns:**

1. **Level** — Log level with color coding (debug: gray, info: blue, warn: yellow, error: red, fatal: dark red bold)
2. **Message** — Error message (max width 500px, truncated, tooltip with full message)
3. **Source** — Source (e.g., 'server', 'client', 'database') — monospace font, badge style
4. **Timestamp** — Timestamp with localized format (`toLocaleString()`)

## Logging system

### Logger class

The `logger.ts` file contains the central Logger class.

**Initialization:**

```typescript
const config = createLogConfig(
  env.LOG_TARGETS,  // 'console' | 'file' | 'database'
  env.LOG_LEVEL,    // 'debug' | 'info' | 'warn' | 'error' | 'fatal'
  env.LOG_DIR       // File log directory
);
export const logger = new Logger(config);
```

**Usage:**

```typescript
import { logger } from '$lib/server/logging/logger';

await logger.debug('Debug message', { source: 'myModule' });
await logger.info('User logged in', { source: 'auth', userId: '123' });
await logger.warn('Deprecated API used', { source: 'api' });
await logger.error('Database connection failed', { source: 'database', stack: error.stack });
await logger.fatal('System crash', { source: 'server', stack: error.stack });
```

**Log level priority:**

```typescript
const LOG_LEVEL_PRIORITY: Record<LogLevel, number> = {
  debug: 0,
  info: 1,
  warn: 2,
  error: 3,
  fatal: 4
};
```

### Transports

The Logger supports three transports:

1. **ConsoleTransport** — Write to console (for development)
2. **FileTransport** — Write to file (for production)
3. **DatabaseTransport** — Write to database (for UI display)

**Configuration:**

```typescript
// .env
LOG_TARGETS=console,database  # Comma-separated
LOG_LEVEL=info                # Minimum log level
LOG_DIR=./logs                # File log directory
```

## Types

### LogEntry

```typescript
interface LogEntry {
  id: string;
  level: LogLevel;
  message: string;
  source: string;
  timestamp: string;
  stack?: string;
  context?: Record<string, unknown>;
  userId?: string;
  url?: string;
  method?: string;
  routeId?: string;
  userAgent?: string;
}
```

### LogLevel

```typescript
type LogLevel = 'debug' | 'info' | 'warn' | 'error' | 'fatal';
```

## Database schema

### `error_logs` table

```typescript
{
  id: serial('id').primaryKey(),
  level: varchar('level', { length: 10 }).notNull(),
  message: text('message').notNull(),
  source: varchar('source', { length: 100 }).notNull(),
  stack: text('stack'),
  context: jsonb('context'),
  userId: varchar('user_id', { length: 255 }),
  url: text('url'),
  method: varchar('method', { length: 10 }),
  routeId: varchar('route_id', { length: 255 }),
  userAgent: text('user_agent'),
  createdAt: timestamp('created_at').defaultNow().notNull()
}
```

**Indexes:** `level`, `source`, `createdAt`, `userId`

## Usage examples

### Logging an error

```typescript
import { logger } from '$lib/server/logging/logger';

try {
  await riskyOperation();
} catch (error) {
  await logger.error('Operation failed', {
    source: 'myModule',
    stack: error.stack,
    context: { operation: 'riskyOperation' }
  });
  throw error;
}
```

### HTTP request logging

```typescript
// hooks.server.ts
export async function handle({ event, resolve }) {
  try {
    const response = await resolve(event);
    await logger.info('HTTP request', {
      source: 'http',
      url: event.url.pathname,
      method: event.request.method,
      userId: event.locals.user?.id
    });
    return response;
  } catch (error) {
    await logger.error('HTTP request failed', {
      source: 'http',
      url: event.url.pathname,
      stack: error.stack
    });
    throw error;
  }
}
```

## Best practices

1. **Use appropriate log levels** — debug for development, info for normal operation, warn for warnings, error for errors, fatal for critical failures
2. **Add context** — Always add relevant context (userId, url, method, etc.)
3. **Stack trace** — Always include the stack trace for errors
4. **Sensitive data** — Never log passwords, API keys, or other sensitive data
5. **Database size** — Regularly clean up old log entries (e.g., delete entries older than 30 days)
6. **Source** — Always specify the source for easier filtering and debugging

## Troubleshooting

### Logs not appearing

**Problem**: No log entries visible in the Log application.

**Solution**:
1. Check the `LOG_TARGETS` environment variable — does it include `database`?
2. Check the `LOG_LEVEL` setting — it may be too high
3. Check the `error_logs` table directly in the database
4. Check permissions (`log.error.view`)

### Filtering not working

**Problem**: Filters don't filter the data.

**Solution**:
1. Check the browser console for errors
2. Check the network tab — is `fetchErrorLogs` being called?
3. Check `$effect` reactivity — are all filter variables included?
