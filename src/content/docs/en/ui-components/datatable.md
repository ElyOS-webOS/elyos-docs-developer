---
title: DataTable
description: Server-side data table component with sorting, filtering and pagination
---

The DataTable is a full-featured data table component that supports server-side pagination, sorting and filtering. It is built on the **Tanstack Table** library.

## Basic Usage

```svelte
<script lang="ts">
  import { DataTable, createActionsColumn } from '$lib/components/ui/data-table';
  import type { ColumnDef } from '@tanstack/table-core';

  interface User {
    id: number;
    name: string;
    email: string;
    role: string;
  }

  let users = $state<User[]>([]);
  let loading = $state(false);
  let pagination = $state({
    page: 1,
    pageSize: 20,
    totalCount: 0,
    totalPages: 0
  });

  const columns: ColumnDef<User>[] = [
    {
      accessorKey: 'name',
      header: 'Name',
      meta: { title: 'Name' }
    },
    {
      accessorKey: 'email',
      header: 'Email',
      meta: { title: 'Email' }
    },
    {
      accessorKey: 'role',
      header: 'Role',
      meta: { title: 'Role' }
    }
  ];

  async function loadData(state: any) {
    loading = true;
    const response = await fetch(
      `/api/users?page=${state.page}&pageSize=${state.pageSize}&sortBy=${state.sortBy}&sortOrder=${state.sortOrder}`
    );
    const data = await response.json();
    users = data.users;
    pagination = data.pagination;
    loading = false;
  }

  $effect(() => {
    loadData({ page: 1, pageSize: 20, sortBy: '', sortOrder: 'desc' });
  });
</script>

<DataTable
  {columns}
  data={users}
  {pagination}
  {loading}
  onStateChange={loadData}
/>
```

## Column Definitions

### Simple Column

```typescript
{
  accessorKey: 'name',
  header: 'Name',
  meta: { title: 'Name' } // For column visibility menu
}
```

### Disable Sorting

```typescript
{
  accessorKey: 'actions',
  header: 'Actions',
  enableSorting: false,
  enableHiding: false
}
```

### Custom Cell Rendering

```typescript
{
  accessorKey: 'status',
  header: 'Status',
  cell: ({ row }) => {
    const status = row.getValue('status');
    return renderComponent(Badge, {
      variant: status === 'active' ? 'default' : 'secondary',
      children: status === 'active' ? 'Active' : 'Inactive'
    });
  }
}
```

## Actions Column

The `createActionsColumn` function automatically handles primary and secondary actions.

### Single Action

```typescript
createActionsColumn<User>([
  {
    label: 'Open',
    onClick: (user) => openUser(user)
  }
])
```

**Result:** A simple button appears.

### Multiple Actions — Primary + Dropdown

```typescript
createActionsColumn<User>([
  {
    label: 'Edit',
    primary: true, // This will be the main button (left side)
    onClick: (user) => editUser(user)
  },
  {
    label: 'Activate',
    onClick: (user) => toggleActive(user)
  },
  {
    label: 'Delete',
    variant: 'destructive', // Red text
    separator: true, // Separator line before it
    onClick: (user) => deleteUser(user)
  }
])
```

**Result:**
- **Left side:** "Edit" button (primary action)
- **Right side:** 3 vertical dots button → dropdown menu (Activate, Delete)

### Dynamic Actions

```typescript
createActionsColumn<User>((user) => {
  const actions = [
    {
      label: 'Edit',
      primary: true,
      onClick: () => editUser(user)
    }
  ];

  // Conditional actions
  if (user.isActive) {
    actions.push({
      label: 'Deactivate',
      onClick: () => deactivateUser(user)
    });
  } else {
    actions.push({
      label: 'Activate',
      onClick: () => activateUser(user)
    });
  }

  // Delete always last
  actions.push({
    label: 'Delete',
    variant: 'destructive',
    separator: true,
    onClick: () => confirmDelete(user)
  });

  return actions;
})
```

### RowAction Type

```typescript
interface RowAction<TData> {
  label: string;              // Action name
  icon?: string;              // Lucide icon name (optional)
  onClick: (row: TData) => void; // Callback
  variant?: 'default' | 'destructive'; // Visual variant
  separator?: boolean;        // Separator line before it
  primary?: boolean;          // Primary action (main button)
}
```

## Toolbar Usage

The toolbar snippet allows adding custom controls above the table.

```svelte
<script>
  import { Input } from '$lib/components/ui/input';
  import { Button } from '$lib/components/ui/button';
  import Plus from 'lucide-svelte/icons/plus';

  let searchQuery = $state('');
</script>

<DataTable
  {columns}
  data={users}
  {pagination}
  {loading}
  onStateChange={loadData}
>
  {#snippet toolbar({ table, handleSort })}
    <Input
      placeholder="Search by name or email..."
      value={searchQuery}
      oninput={(e) => {
        searchQuery = e.currentTarget.value;
        // Trigger search
      }}
      class="max-w-sm"
    />
    <Button onclick={() => openCreateDialog()}>
      <Plus size={16} class="mr-2" />
      Create new
    </Button>
  {/snippet}
</DataTable>
```

## Filters

### Faceted Filter

```svelte
<script>
  import { DataTableFacetedFilter } from '$lib/components/ui/data-table';

  const roleOptions = [
    { label: 'Admin', value: 'admin' },
    { label: 'User', value: 'user' },
    { label: 'Guest', value: 'guest' }
  ];

  let selectedRoles = $state<string[]>([]);
</script>

<DataTable {columns} data={users} {pagination} {loading} onStateChange={loadData}>
  {#snippet toolbar({ table })}
    <DataTableFacetedFilter
      {table}
      column="role"
      title="Role"
      options={roleOptions}
      bind:selected={selectedRoles}
    />
  {/snippet}
</DataTable>
```

## Props

| Prop | Type | Description |
|------|------|-------------|
| `columns` | `ColumnDef<TData>[]` | Column definitions |
| `data` | `TData[]` | Data array |
| `pagination` | `PaginationInfo` | Pagination information |
| `loading` | `boolean` | Loading state |
| `striped` | `boolean` | Striped rows (default: false) |
| `pageSizes` | `number[]` | Available page size options (default: [10, 20, 50, 100]) |
| `initialSortBy` | `string` | Initial sort column |
| `initialSortOrder` | `'asc' \| 'desc'` | Initial sort direction (default: 'desc') |
| `initialPageSize` | `number` | Initial page size (default: 20) |
| `onStateChange` | `(state) => void` | State change callback |
| `toolbar` | `Snippet` | Custom toolbar snippet |

### PaginationInfo Type

```typescript
interface PaginationInfo {
  page: number;        // Current page (starts at 1)
  pageSize: number;    // Rows per page
  totalCount: number;  // Total row count
  totalPages: number;  // Total page count
}
```

### DataTableState Type

```typescript
interface DataTableState {
  page: number;
  pageSize: number;
  sortBy: string;
  sortOrder: 'asc' | 'desc';
}
```

## Full Example

```svelte
<script lang="ts">
  import { DataTable, createActionsColumn } from '$lib/components/ui/data-table';
  import { Input } from '$lib/components/ui/input';
  import { Button } from '$lib/components/ui/button';
  import { ConfirmDialog, CustomDialog } from '$lib/components/ui';
  import { toast } from 'svelte-sonner';
  import type { ColumnDef } from '@tanstack/table-core';
  import Plus from 'lucide-svelte/icons/plus';

  interface User {
    id: number;
    name: string;
    email: string;
    role: string;
    isActive: boolean;
  }

  let users = $state<User[]>([]);
  let loading = $state(false);
  let pagination = $state({ page: 1, pageSize: 20, totalCount: 0, totalPages: 0 });
  let searchQuery = $state('');
  let deleteDialogOpen = $state(false);
  let selectedUser = $state<User | null>(null);

  const columns: ColumnDef<User>[] = [
    { accessorKey: 'name', header: 'Name', meta: { title: 'Name' } },
    { accessorKey: 'email', header: 'Email', meta: { title: 'Email' } },
    { accessorKey: 'role', header: 'Role', meta: { title: 'Role' } },
    createActionsColumn<User>((user) => [
      { label: 'Edit', primary: true, onClick: () => openEditDialog(user) },
      { label: user.isActive ? 'Deactivate' : 'Activate', onClick: () => toggleActive(user) },
      { label: 'Delete', variant: 'destructive', separator: true, onClick: () => openDeleteDialog(user) }
    ])
  ];

  async function loadData(state: any) {
    loading = true;
    try {
      const params = new URLSearchParams({
        page: state.page.toString(),
        pageSize: state.pageSize.toString(),
        sortBy: state.sortBy,
        sortOrder: state.sortOrder,
        search: searchQuery
      });
      const response = await fetch(`/api/users?${params}`);
      const data = await response.json();
      users = data.users;
      pagination = data.pagination;
    } catch (error) {
      toast.error('Error loading data');
    } finally {
      loading = false;
    }
  }

  async function handleDelete() {
    try {
      await fetch(`/api/users/${selectedUser!.id}`, { method: 'DELETE' });
      toast.success('User deleted');
      loadData({ page: pagination.page, pageSize: pagination.pageSize, sortBy: '', sortOrder: 'desc' });
    } catch (error) {
      toast.error('Error deleting user');
    }
  }

  $effect(() => {
    loadData({ page: 1, pageSize: 20, sortBy: '', sortOrder: 'desc' });
  });
</script>

<DataTable {columns} data={users} {pagination} {loading} striped onStateChange={loadData}>
  {#snippet toolbar()}
    <Input
      placeholder="Search..."
      value={searchQuery}
      oninput={(e) => {
        searchQuery = e.currentTarget.value;
        loadData({ page: 1, pageSize: pagination.pageSize, sortBy: '', sortOrder: 'desc' });
      }}
      class="max-w-sm"
    />
    <Button onclick={() => createDialogOpen = true}>
      <Plus size={16} class="mr-2" />
      New user
    </Button>
  {/snippet}
</DataTable>

<ConfirmDialog
  bind:open={deleteDialogOpen}
  title="Delete user"
  description="Are you sure you want to delete {selectedUser?.name}?"
  confirmText="Delete"
  confirmVariant="destructive"
  onConfirm={handleDelete}
  onCancel={() => deleteDialogOpen = false}
/>
```

## Best Practices

1. **Server-side pagination** — Always use server-side pagination for large datasets
2. **Loading state** — Always display loading state
3. **Primary action** — The most common action should be `primary: true`
4. **Delete confirmation** — Always use ConfirmDialog for deletion
5. **Toast feedback** — Give feedback after every operation
6. **Column visibility** — Provide `meta.title` for every column
7. **Sorting** — Only enable for meaningful columns
8. **Striped rows** — Use for large tables for better readability
9. **Responsive** — Hide less important columns on mobile
10. **Error handling** — Handle loading errors

## Related

- [Dialog Components →](./dialogs) — Confirmation dialogs
- [Toast Notifications →](./notifications) — Feedback
- [Basic Components →](./basic) — Button, Input
