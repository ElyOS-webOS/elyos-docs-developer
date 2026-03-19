---
title: Navigation Components
description: Navigation components and menus
---

Navigation components help users navigate within the application and access actions.

## Dropdown Menu

The Dropdown Menu component is a dropdown that appears on button click.

### Import

```svelte
<script>
  import * as DropdownMenu from '$lib/components/ui/dropdown-menu';
</script>
```

### Basic Usage

```svelte
<DropdownMenu.Root>
  <DropdownMenu.Trigger asChild let:builder>
    <Button builders={[builder]} variant="outline">
      Open menu
    </Button>
  </DropdownMenu.Trigger>
  <DropdownMenu.Content>
    <DropdownMenu.Item>Profile</DropdownMenu.Item>
    <DropdownMenu.Item>Settings</DropdownMenu.Item>
    <DropdownMenu.Separator />
    <DropdownMenu.Item>Sign out</DropdownMenu.Item>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

### Component Parts

- **DropdownMenu.Root** — Main container
- **DropdownMenu.Trigger** — Trigger button
- **DropdownMenu.Content** — Menu content
- **DropdownMenu.Item** — Menu item
- **DropdownMenu.Separator** — Divider line
- **DropdownMenu.Label** — Label
- **DropdownMenu.Group** — Group of menu items
- **DropdownMenu.Sub** — Submenu
- **DropdownMenu.CheckboxItem** — Checkbox menu item
- **DropdownMenu.RadioGroup** — Radio button group
- **DropdownMenu.RadioItem** — Radio menu item

### Example: With Icons

```svelte
<DropdownMenu.Root>
  <DropdownMenu.Trigger asChild let:builder>
    <Button builders={[builder]} variant="ghost" size="icon">
      <MoreVertical class="size-4" />
    </Button>
  </DropdownMenu.Trigger>
  <DropdownMenu.Content align="end">
    <DropdownMenu.Item>
      <Edit class="mr-2 size-4" />
      Edit
    </DropdownMenu.Item>
    <DropdownMenu.Item>
      <Copy class="mr-2 size-4" />
      Copy
    </DropdownMenu.Item>
    <DropdownMenu.Separator />
    <DropdownMenu.Item class="text-destructive">
      <Trash2 class="mr-2 size-4" />
      Delete
    </DropdownMenu.Item>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

### Example: With Submenu

```svelte
<DropdownMenu.Root>
  <DropdownMenu.Trigger asChild let:builder>
    <Button builders={[builder]}>Actions</Button>
  </DropdownMenu.Trigger>
  <DropdownMenu.Content>
    <DropdownMenu.Item>New item</DropdownMenu.Item>

    <DropdownMenu.Sub>
      <DropdownMenu.SubTrigger>Export</DropdownMenu.SubTrigger>
      <DropdownMenu.SubContent>
        <DropdownMenu.Item>PDF</DropdownMenu.Item>
        <DropdownMenu.Item>Excel</DropdownMenu.Item>
        <DropdownMenu.Item>CSV</DropdownMenu.Item>
      </DropdownMenu.SubContent>
    </DropdownMenu.Sub>

    <DropdownMenu.Separator />
    <DropdownMenu.Item>Close</DropdownMenu.Item>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

### Example: With Checkbox Items

```svelte
<script>
  let showToolbar = $state(true);
  let showSidebar = $state(false);
</script>

<DropdownMenu.Root>
  <DropdownMenu.Trigger asChild let:builder>
    <Button builders={[builder]} variant="outline">View</Button>
  </DropdownMenu.Trigger>
  <DropdownMenu.Content>
    <DropdownMenu.Label>Display</DropdownMenu.Label>
    <DropdownMenu.Separator />
    <DropdownMenu.CheckboxItem bind:checked={showToolbar}>
      Toolbar
    </DropdownMenu.CheckboxItem>
    <DropdownMenu.CheckboxItem bind:checked={showSidebar}>
      Sidebar
    </DropdownMenu.CheckboxItem>
  </DropdownMenu.Content>
</DropdownMenu.Root>
```

## Context Menu

The Context Menu component is a right-click menu that appears over an element.

### Import

```svelte
<script>
  import * as ContextMenu from '$lib/components/ui/context-menu';
</script>
```

### Basic Usage

```svelte
<ContextMenu.Root>
  <ContextMenu.Trigger>
    <div class="border rounded-lg p-8 text-center">
      Right-click here
    </div>
  </ContextMenu.Trigger>
  <ContextMenu.Content>
    <ContextMenu.Item>Open</ContextMenu.Item>
    <ContextMenu.Item>Edit</ContextMenu.Item>
    <ContextMenu.Separator />
    <ContextMenu.Item class="text-destructive">Delete</ContextMenu.Item>
  </ContextMenu.Content>
</ContextMenu.Root>
```

### Example: File Actions

```svelte
<ContextMenu.Root>
  <ContextMenu.Trigger>
    <div class="flex items-center gap-2 p-2 hover:bg-muted rounded">
      <FileText class="size-4" />
      <span>document.pdf</span>
    </div>
  </ContextMenu.Trigger>
  <ContextMenu.Content>
    <ContextMenu.Item>
      <Eye class="mr-2 size-4" />
      Open
    </ContextMenu.Item>
    <ContextMenu.Item>
      <Download class="mr-2 size-4" />
      Download
    </ContextMenu.Item>
    <ContextMenu.Item>
      <Share class="mr-2 size-4" />
      Share
    </ContextMenu.Item>
    <ContextMenu.Separator />
    <ContextMenu.Item>
      <Edit class="mr-2 size-4" />
      Rename
    </ContextMenu.Item>
    <ContextMenu.Item class="text-destructive">
      <Trash2 class="mr-2 size-4" />
      Delete
    </ContextMenu.Item>
  </ContextMenu.Content>
</ContextMenu.Root>
```

## Popover

The Popover component is a popup panel that appears next to an element.

### Import

```svelte
<script>
  import * as Popover from '$lib/components/ui/popover';
</script>
```

### Basic Usage

```svelte
<Popover.Root>
  <Popover.Trigger asChild let:builder>
    <Button builders={[builder]} variant="outline">
      Information
    </Button>
  </Popover.Trigger>
  <Popover.Content>
    <div class="space-y-2">
      <h4 class="font-medium">Popover title</h4>
      <p class="text-sm text-muted-foreground">
        This is popover content.
      </p>
    </div>
  </Popover.Content>
</Popover.Root>
```

### Example: Combobox (with search)

```svelte
<script>
  import { tick } from 'svelte';
  import { useId } from 'bits-ui';
  import * as Popover from '$lib/components/ui/popover';
  import * as Command from '$lib/components/ui/command';
  import { Button } from '$lib/components/ui/button';
  import Check from 'lucide-svelte/icons/check';
  import ChevronsUpDown from 'lucide-svelte/icons/chevrons-up-down';
  import { cn } from '$lib/utils/utils';

  let open = $state(false);
  let selectedValue = $state('');
  const triggerId = useId();

  const items = [
    { value: '1', label: 'First item' },
    { value: '2', label: 'Second item' },
    { value: '3', label: 'Third item' }
  ];

  function closeAndFocusTrigger() {
    open = false;
    tick().then(() => {
      document.getElementById(triggerId)?.focus();
    });
  }
</script>

<Popover.Root bind:open>
  <Popover.Trigger asChild let:builder>
    <Button
      builders={[builder]}
      variant="outline"
      role="combobox"
      aria-expanded={open}
      class="w-[200px] justify-between"
      id={triggerId}
    >
      {selectedValue
        ? items.find((i) => i.value === selectedValue)?.label
        : 'Select...'}
      <ChevronsUpDown class="ml-2 size-4 shrink-0 opacity-50" />
    </Button>
  </Popover.Trigger>
  <Popover.Content class="w-[200px] p-0">
    <Command.Root>
      <Command.Input placeholder="Search..." />
      <Command.Empty>No results.</Command.Empty>
      <Command.Group>
        {#each items as item}
          <Command.Item
            value={item.value}
            onSelect={(currentValue) => {
              selectedValue = currentValue === selectedValue ? '' : currentValue;
              closeAndFocusTrigger();
            }}
          >
            <Check
              class={cn(
                'mr-2 size-4',
                selectedValue === item.value ? 'opacity-100' : 'opacity-0'
              )}
            />
            {item.label}
          </Command.Item>
        {/each}
      </Command.Group>
    </Command.Root>
  </Popover.Content>
</Popover.Root>
```

## Command

The Command component is a command palette with search and keyboard shortcuts.

### Import

```svelte
<script>
  import * as Command from '$lib/components/ui/command';
</script>
```

### Basic Usage

```svelte
<Command.Root>
  <Command.Input placeholder="Search..." />
  <Command.List>
    <Command.Empty>No results.</Command.Empty>
    <Command.Group heading="Actions">
      <Command.Item>New file</Command.Item>
      <Command.Item>New folder</Command.Item>
      <Command.Item>Save</Command.Item>
    </Command.Group>
    <Command.Separator />
    <Command.Group heading="Navigation">
      <Command.Item>Home</Command.Item>
      <Command.Item>Settings</Command.Item>
    </Command.Group>
  </Command.List>
</Command.Root>
```

### Example: In a Dialog

```svelte
<script>
  import * as Dialog from '$lib/components/ui/dialog';
  import * as Command from '$lib/components/ui/command';

  let open = $state(false);

  // Ctrl+K keyboard shortcut
  $effect(() => {
    const down = (e: KeyboardEvent) => {
      if (e.key === 'k' && (e.metaKey || e.ctrlKey)) {
        e.preventDefault();
        open = !open;
      }
    };

    document.addEventListener('keydown', down);
    return () => document.removeEventListener('keydown', down);
  });
</script>

<Dialog.Root bind:open>
  <Dialog.Content class="p-0">
    <Command.Root>
      <Command.Input placeholder="Search commands..." />
      <Command.List>
        <Command.Empty>No results.</Command.Empty>
        <Command.Group heading="Actions">
          <Command.Item>
            <FileText class="mr-2 size-4" />
            New document
          </Command.Item>
          <Command.Item>
            <Folder class="mr-2 size-4" />
            New folder
          </Command.Item>
        </Command.Group>
      </Command.List>
    </Command.Root>
  </Dialog.Content>
</Dialog.Root>

<p class="text-sm text-muted-foreground">
  Press <kbd>Ctrl+K</kbd> to open the command palette
</p>
```

## Best Practices

1. **Dropdown Menu with icons** — Use icons next to menu items
2. **Destructive actions** — Use `text-destructive` class for dangerous actions
3. **Separator usage** — Group related menu items
4. **Context Menu** — Use right-click menu for file/item actions
5. **Popover align** — Set the `align` prop (`start`, `center`, `end`)
6. **Combobox pattern** — Use Popover + Command combination with search
7. **Command Dialog** — Use Command in Dialog for command palettes
8. **Keyboard shortcuts** — Add keyboard shortcuts to Command
9. **Popover width** — Set width appropriate to content
10. **Dropdown trigger** — Use `asChild` prop for custom trigger button

## Related Components

- [Basic Components](./basic) — Button, Input, Label
- [Dialog Components](./dialogs) — Modal windows
- [Icons](./icons) — Lucide icon usage
- [Tailwind CSS](./tailwind) — Styling utilities
