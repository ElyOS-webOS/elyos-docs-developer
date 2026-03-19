---
title: Basic Components
description: Basic UI building blocks - Button, Input, Select, Checkbox, Switch
---

The basic UI components are the building blocks of applications. They come from the shadcn-svelte library and provide a consistent appearance.

## Button

Button component with various variants and sizes.

### Usage

```svelte
<script>
  import { Button } from '$lib/components/ui/button';
</script>

<Button>Default</Button>
<Button variant="destructive">Delete</Button>
<Button variant="outline">Outline</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="link">Link</Button>
```

### Sizes

```svelte
<Button size="sm">Small</Button>
<Button size="default">Default</Button>
<Button size="lg">Large</Button>
<Button size="icon">
  <User size={16} />
</Button>
```

### Variants

| Variant | Usage | Color |
|---------|-------|-------|
| `default` | Primary action | Blue |
| `destructive` | Dangerous action (delete) | Red |
| `outline` | Secondary action | Outlined |
| `secondary` | Tertiary action | Gray |
| `ghost` | Minimal emphasis | No background |
| `link` | Link style | Underlined |

### Props

| Prop | Type | Description |
|------|------|-------------|
| `variant` | `string` | Button variant (default: "default") |
| `size` | `string` | Button size (default: "default") |
| `disabled` | `boolean` | Disabled state |
| `onclick` | `() => void` | Click event |

### Examples

**Delete button:**

```svelte
<Button variant="destructive" onclick={handleDelete}>
  <Trash2 size={16} class="mr-2" />
  Delete
</Button>
```

**Loading state:**

```svelte
<script>
  let loading = $state(false);
</script>

<Button disabled={loading} onclick={handleSave}>
  {#if loading}
    Saving...
  {:else}
    Save
  {/if}
</Button>
```

---

## Input

Text input field component.

### Usage

```svelte
<script>
  import { Input } from '$lib/components/ui/input';
  import { Label } from '$lib/components/ui/label';

  let value = $state('');
</script>

<div class="space-y-2">
  <Label for="email">Email</Label>
  <Input
    id="email"
    type="email"
    placeholder="example@email.com"
    bind:value
  />
</div>
```

### Types

```svelte
<Input type="text" placeholder="Text" />
<Input type="email" placeholder="Email" />
<Input type="password" placeholder="Password" />
<Input type="number" placeholder="Number" />
<Input type="tel" placeholder="Phone" />
<Input type="url" placeholder="URL" />
<Input type="date" />
```

### Props

| Prop | Type | Description |
|------|------|-------------|
| `type` | `string` | Input type (default: "text") |
| `placeholder` | `string` | Placeholder text |
| `value` | `string` | Value (bindable) |
| `disabled` | `boolean` | Disabled state |
| `required` | `boolean` | Required field |

### Examples

**With validation:**

```svelte
<script>
  let email = $state('');
  let error = $state('');

  function validate() {
    if (!email.includes('@')) {
      error = 'Invalid email address';
    } else {
      error = '';
    }
  }
</script>

<div class="space-y-2">
  <Label for="email">Email *</Label>
  <Input
    id="email"
    type="email"
    bind:value={email}
    oninput={validate}
    class={error ? 'border-destructive' : ''}
  />
  {#if error}
    <p class="text-sm text-destructive">{error}</p>
  {/if}
</div>
```

---

## Select

Dropdown menu component.

### Usage

```svelte
<script>
  import * as Select from '$lib/components/ui/select';
  import { Label } from '$lib/components/ui/label';

  let selected = $state('');
</script>

<div class="space-y-2">
  <Label>Role</Label>
  <Select.Root bind:value={selected}>
    <Select.Trigger>
      <Select.Value placeholder="Select a role..." />
    </Select.Trigger>
    <Select.Content>
      <Select.Item value="admin">Admin</Select.Item>
      <Select.Item value="user">User</Select.Item>
      <Select.Item value="guest">Guest</Select.Item>
    </Select.Content>
  </Select.Root>
</div>
```

### Grouped Options

```svelte
<Select.Root bind:value={selected}>
  <Select.Trigger>
    <Select.Value placeholder="Select..." />
  </Select.Trigger>
  <Select.Content>
    <Select.Group>
      <Select.Label>Administrators</Select.Label>
      <Select.Item value="superadmin">Super Admin</Select.Item>
      <Select.Item value="admin">Admin</Select.Item>
    </Select.Group>
    <Select.Separator />
    <Select.Group>
      <Select.Label>Users</Select.Label>
      <Select.Item value="user">User</Select.Item>
      <Select.Item value="guest">Guest</Select.Item>
    </Select.Group>
  </Select.Content>
</Select.Root>
```

---

## Checkbox

Checkbox component.

### Usage

```svelte
<script>
  import { Checkbox } from '$lib/components/ui/checkbox';
  import { Label } from '$lib/components/ui/label';

  let checked = $state(false);
</script>

<div class="flex items-center space-x-2">
  <Checkbox id="terms" bind:checked />
  <Label for="terms">I accept the terms</Label>
</div>
```

### Multiple Checkboxes

```svelte
<script>
  let permissions = $state({
    read: false,
    write: false,
    delete: false
  });
</script>

<div class="space-y-2">
  <div class="flex items-center space-x-2">
    <Checkbox id="read" bind:checked={permissions.read} />
    <Label for="read">Read</Label>
  </div>
  <div class="flex items-center space-x-2">
    <Checkbox id="write" bind:checked={permissions.write} />
    <Label for="write">Write</Label>
  </div>
  <div class="flex items-center space-x-2">
    <Checkbox id="delete" bind:checked={permissions.delete} />
    <Label for="delete">Delete</Label>
  </div>
</div>
```

---

## Switch

Toggle switch component for on/off state.

### Usage

```svelte
<script>
  import { Switch } from '$lib/components/ui/switch';
  import { Label } from '$lib/components/ui/label';

  let enabled = $state(false);
</script>

<div class="flex items-center space-x-2">
  <Switch id="notifications" bind:checked={enabled} />
  <Label for="notifications">Enable notifications</Label>
</div>
```

### Disabled State

```svelte
<div class="flex items-center space-x-2">
  <Switch id="feature" disabled />
  <Label for="feature" class="text-muted-foreground">
    Coming soon
  </Label>
</div>
```

---

## Label

Label component for input fields.

### Usage

```svelte
<script>
  import { Label } from '$lib/components/ui/label';
  import { Input } from '$lib/components/ui/input';
</script>

<div class="space-y-2">
  <Label for="username">Username *</Label>
  <Input id="username" required />
</div>
```

### Required Field Indicator

```svelte
<Label for="email">
  Email <span class="text-destructive">*</span>
</Label>
```

---

## Badge

Label component for indicating statuses and categories.

### Usage

```svelte
<script>
  import { Badge } from '$lib/components/ui/badge';
</script>

<Badge>Default</Badge>
<Badge variant="secondary">Secondary</Badge>
<Badge variant="destructive">Destructive</Badge>
<Badge variant="outline">Outline</Badge>
```

### Status Indicator

```svelte
<script>
  function getStatusBadge(status: string) {
    switch (status) {
      case 'active':
        return { variant: 'default', label: 'Active' };
      case 'inactive':
        return { variant: 'secondary', label: 'Inactive' };
      case 'error':
        return { variant: 'destructive', label: 'Error' };
      default:
        return { variant: 'outline', label: 'Unknown' };
    }
  }
</script>

{#each users as user}
  {@const badge = getStatusBadge(user.status)}
  <Badge variant={badge.variant}>{badge.label}</Badge>
{/each}
```

---

## Separator

Divider line component.

### Usage

```svelte
<script>
  import { Separator } from '$lib/components/ui/separator';
</script>

<div class="space-y-4">
  <div>First section</div>
  <Separator />
  <div>Second section</div>
</div>
```

### Vertical Separator

```svelte
<div class="flex items-center space-x-4">
  <span>Item 1</span>
  <Separator orientation="vertical" class="h-4" />
  <span>Item 2</span>
</div>
```

---

## Best Practices

1. **Always use Label** — Every input should have a label
2. **Required field indicators** — Use `*` or "Required" text
3. **Placeholder text** — Give an example of the expected format
4. **Validation** — Display errors below the input
5. **Disabled state** — Use the `disabled` prop, not CSS
6. **Button variants** — `destructive` only for deletion, `default` for primary action
7. **Badge colors** — Use semantic colors (red = error, green = success)
8. **Accessibility** — Use `id` and `for` attributes with Label

## Related

- [Dialog Components →](./dialogs) — Modal windows
- [DataTable →](./datatable) — Data tables
- [Icons →](./icons) — Lucide icons
