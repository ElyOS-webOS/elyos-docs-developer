---
title: Layout Components
description: Components for structuring content
---

Layout components help with the visual structuring and grouping of content.

## Card

The Card component is used for visually grouping content units.

### Import

```svelte
<script>
  import * as Card from '$lib/components/ui/card';
</script>
```

### Basic Usage

```svelte
<Card.Root>
  <Card.Header>
    <Card.Title>Card title</Card.Title>
    <Card.Description>Card description</Card.Description>
  </Card.Header>
  <Card.Content>
    <p>The card content goes here.</p>
  </Card.Content>
  <Card.Footer>
    <Button>Action</Button>
  </Card.Footer>
</Card.Root>
```

### Component Parts

- **Card.Root** — Main container
- **Card.Header** — Header section (optional)
- **Card.Title** — Title
- **Card.Description** — Description
- **Card.Content** — Main content
- **Card.Footer** — Footer (optional, often with buttons)

### Example: Empty State

```svelte
<Card.Root class="border-dashed">
  <Card.Content class="flex flex-col items-center justify-center py-16 text-center">
    <div class="bg-primary/10 mb-6 flex size-20 items-center justify-center rounded-full">
      <Store class="text-primary size-10" />
    </div>

    <h3 class="mb-2 text-xl font-semibold">Coming soon</h3>
    <p class="text-muted-foreground mb-6 max-w-md">
      This feature is currently under development.
    </p>

    <Button>Request notification</Button>
  </Card.Content>
</Card.Root>
```

### Example: Information Card

```svelte
<Card.Root>
  <Card.Header>
    <Card.Title>User profile</Card.Title>
    <Card.Description>Manage personal data</Card.Description>
  </Card.Header>
  <Card.Content class="space-y-4">
    <div>
      <Label for="name">Name</Label>
      <Input id="name" value="John Doe" />
    </div>
    <div>
      <Label for="email">Email</Label>
      <Input id="email" type="email" value="john@example.com" />
    </div>
  </Card.Content>
  <Card.Footer class="flex justify-end gap-2">
    <Button variant="outline">Cancel</Button>
    <Button>Save</Button>
  </Card.Footer>
</Card.Root>
```

### Styling

The Card component supports Tailwind classes:

```svelte
<!-- Dashed border -->
<Card.Root class="border-dashed">

<!-- No shadow -->
<Card.Root class="shadow-none">

<!-- Background color -->
<Card.Root class="bg-muted">

<!-- Hover effect -->
<Card.Root class="hover:shadow-lg transition-shadow">
```

## Tabs

The Tabs component is used for switching between multiple content sections with tabs.

### Import

```svelte
<script>
  import * as Tabs from '$lib/components/ui/tabs';
</script>
```

### Basic Usage

```svelte
<script>
  let activeTab = $state('members');
</script>

<Tabs.Root bind:value={activeTab}>
  <Tabs.List>
    <Tabs.Trigger value="members">Members</Tabs.Trigger>
    <Tabs.Trigger value="permissions">Permissions</Tabs.Trigger>
    <Tabs.Trigger value="apps">Apps</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="members">
    <p>Member list...</p>
  </Tabs.Content>

  <Tabs.Content value="permissions">
    <p>Permission list...</p>
  </Tabs.Content>

  <Tabs.Content value="apps">
    <p>App list...</p>
  </Tabs.Content>
</Tabs.Root>
```

### Component Parts

- **Tabs.Root** — Main container, controls active tab with `value` prop
- **Tabs.List** — Tab button container
- **Tabs.Trigger** — Tab button, identified by `value` prop
- **Tabs.Content** — Tab content, identified by `value` prop

### Example: With DataTable

```svelte
<script>
  let activeTab = $state('members');
</script>

<Tabs.Root bind:value={activeTab}>
  <Tabs.List>
    <Tabs.Trigger value="members">Members</Tabs.Trigger>
    <Tabs.Trigger value="permissions">Permissions</Tabs.Trigger>
    <Tabs.Trigger value="apps">Apps</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="members">
    <DataTable
      columns={userColumns}
      data={users}
      pagination={userPaginationInfo}
      loading={userLoading}
      striped
      initialSortBy="name"
      initialSortOrder="asc"
      initialPageSize={10}
      onStateChange={handleUserStateChange}
    />
  </Tabs.Content>

  <Tabs.Content value="permissions">
    <DataTable
      columns={permColumns}
      data={permissions}
      pagination={permPaginationInfo}
      loading={permLoading}
      striped
      initialSortBy="name"
      initialSortOrder="asc"
      initialPageSize={10}
      onStateChange={handlePermStateChange}
    />
  </Tabs.Content>
</Tabs.Root>
```

### Programmatic Tab Switching

```svelte
<script>
  let activeTab = $state('tab1');

  function goToNextTab() {
    if (activeTab === 'tab1') activeTab = 'tab2';
    else if (activeTab === 'tab2') activeTab = 'tab3';
  }
</script>

<Tabs.Root bind:value={activeTab}>
  <Tabs.List>
    <Tabs.Trigger value="tab1">Step 1</Tabs.Trigger>
    <Tabs.Trigger value="tab2">Step 2</Tabs.Trigger>
    <Tabs.Trigger value="tab3">Step 3</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="tab1">
    <p>Step 1 content</p>
    <Button onclick={goToNextTab}>Next</Button>
  </Tabs.Content>

  <Tabs.Content value="tab2">
    <p>Step 2 content</p>
    <Button onclick={goToNextTab}>Next</Button>
  </Tabs.Content>

  <Tabs.Content value="tab3">
    <p>Step 3 content</p>
    <Button>Finish</Button>
  </Tabs.Content>
</Tabs.Root>
```

## Separator

The Separator component is a visual divider line between content sections.

### Import

```svelte
<script>
  import { Separator } from '$lib/components/ui/separator';
</script>
```

### Horizontal Separator

```svelte
<div>
  <h3>First section</h3>
  <p>Content...</p>
</div>

<Separator class="my-4" />

<div>
  <h3>Second section</h3>
  <p>Content...</p>
</div>
```

### Vertical Separator

```svelte
<div class="flex items-center gap-4">
  <Button>Save</Button>
  <Separator orientation="vertical" class="h-6" />
  <Button variant="outline">Cancel</Button>
</div>
```

### Example: Menu Separator

```svelte
<div class="space-y-1">
  <Button variant="ghost" class="w-full justify-start">
    <User class="mr-2 size-4" />
    Profile
  </Button>
  <Button variant="ghost" class="w-full justify-start">
    <Settings class="mr-2 size-4" />
    Settings
  </Button>

  <Separator class="my-2" />

  <Button variant="ghost" class="w-full justify-start text-destructive">
    <LogOut class="mr-2 size-4" />
    Sign out
  </Button>
</div>
```

## Best Practices

1. **Card usage** — Group related content into Cards
2. **Card.Footer** — Place action buttons in the Footer
3. **Tabs with DataTable** — Use Tabs to organize content for large datasets
4. **Separator spacing** — Use appropriate margin classes (`my-4`, `my-6`)
5. **Vertical Separator** — Set height (`h-6`, `h-8`)
6. **Card styling** — Use `border-dashed` for empty states
7. **Tabs initial value** — Always provide an initial `value` to Tabs.Root
8. **Card.Description** — Use description to provide context
9. **Responsive Card** — Use responsive classes (`sm:`, `md:`, `lg:`)
10. **Separator color** — Separator automatically uses theme colors

## Related Components

- [Basic Components](./basic) — Button, Input, Label
- [Dialog Components](./dialogs) — Modal windows
- [DataTable](./datatable) — Data tables in Tabs
- [Tailwind CSS](./tailwind) — Styling utilities
