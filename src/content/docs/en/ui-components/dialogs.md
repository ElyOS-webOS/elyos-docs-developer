---
title: Dialog Components
description: Modal windows and dialog panels - ConfirmDialog, CustomDialog
---

Dialog components display modal windows that show important information or ask for confirmation from the user.

## ConfirmDialog

Simple confirmation dialog with a two-button interface. Ideal for confirming deletion and other irreversible operations.

### Usage

```svelte
<script>
  import { ConfirmDialog } from '$lib/components/ui';
  import { Button } from '$lib/components/ui/button';

  let open = $state(false);

  function handleDelete() {
    console.log('Deleted');
  }
</script>

<Button onclick={() => open = true}>Delete</Button>

<ConfirmDialog
  bind:open
  title="Delete user"
  description="Are you sure you want to delete this user? This action cannot be undone."
  confirmText="Delete"
  cancelText="Cancel"
  confirmVariant="destructive"
  onConfirm={handleDelete}
  onCancel={() => console.log('Cancelled')}
/>
```

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `open` | `boolean` | Yes | Dialog open state (bindable) |
| `title` | `string` | Yes | Dialog title |
| `description` | `string` | Yes | Description/question text |
| `confirmText` | `string` | No | Confirm button text (default: "Continue") |
| `cancelText` | `string` | No | Cancel button text (default: "Cancel") |
| `confirmVariant` | `string` | No | Button variant (default: "default") |
| `onConfirm` | `() => void` | Yes | Confirm callback |
| `onCancel` | `() => void` | Yes | Cancel/close callback |

### Variants

**Delete confirmation (destructive):**

```svelte
<ConfirmDialog
  bind:open
  title="Delete group"
  description="Are you sure you want to delete this group? All members will lose their group membership."
  confirmText="Delete"
  cancelText="Cancel"
  confirmVariant="destructive"
  onConfirm={deleteGroup}
  onCancel={() => open = false}
/>
```

**Normal confirmation (default):**

```svelte
<ConfirmDialog
  bind:open
  title="Save settings"
  description="Are you sure you want to save the changes?"
  confirmText="Save"
  cancelText="Cancel"
  confirmVariant="default"
  onConfirm={saveSettings}
  onCancel={() => open = false}
/>
```

### Examples

**Delete with toast message:**

```svelte
<script>
  import { toast } from 'svelte-sonner';
  import { ConfirmDialog } from '$lib/components/ui';

  let deleteDialogOpen = $state(false);
  let userToDelete = $state(null);

  function openDeleteDialog(user) {
    userToDelete = user;
    deleteDialogOpen = true;
  }

  async function handleDelete() {
    try {
      await deleteUser(userToDelete.id);
      toast.success('User deleted');
    } catch (error) {
      toast.error('Error deleting user');
    }
  }
</script>

<ConfirmDialog
  bind:open={deleteDialogOpen}
  title="Delete user"
  description="Are you sure you want to delete {userToDelete?.name}?"
  confirmText="Delete"
  confirmVariant="destructive"
  onConfirm={handleDelete}
  onCancel={() => deleteDialogOpen = false}
/>
```

---

## CustomDialog

Custom content dialog component for more complex forms and content. Uses snippets for customizing content and buttons.

### Usage

```svelte
<script>
  import { CustomDialog } from '$lib/components/ui';
  import { Button } from '$lib/components/ui/button';
  import { Input } from '$lib/components/ui/input';
  import { Label } from '$lib/components/ui/label';

  let open = $state(false);
  let name = $state('');
  let email = $state('');

  function handleSave() {
    console.log('Save:', { name, email });
    open = false;
  }
</script>

<Button onclick={() => open = true}>New user</Button>

<CustomDialog
  bind:open
  title="Create new user"
  description="Enter the new user's details"
  onClose={() => console.log('Closed')}
>
  {#snippet content()}
    <div class="space-y-4">
      <div>
        <Label for="name">Name *</Label>
        <Input id="name" bind:value={name} required />
      </div>
      <div>
        <Label for="email">Email *</Label>
        <Input id="email" type="email" bind:value={email} required />
      </div>
    </div>
  {/snippet}

  {#snippet actions()}
    <Button variant="outline" onclick={() => open = false}>
      Cancel
    </Button>
    <Button onclick={handleSave}>
      Create
    </Button>
  {/snippet}
</CustomDialog>
```

### Props

| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `open` | `boolean` | Yes | Dialog open state (bindable) |
| `title` | `string` | Yes | Dialog title |
| `description` | `string` | No | Optional description below title |
| `content` | `Snippet` | Yes | Custom content snippet |
| `actions` | `Snippet` | Yes | Action buttons snippet |
| `onClose` | `() => void` | No | Close callback |

### Examples

**Edit form:**

```svelte
<script>
  import { CustomDialog } from '$lib/components/ui';
  import { Button } from '$lib/components/ui/button';
  import { Input } from '$lib/components/ui/input';
  import { Label } from '$lib/components/ui/label';
  import * as Select from '$lib/components/ui/select';
  import { toast } from 'svelte-sonner';

  let editDialogOpen = $state(false);
  let user = $state({ name: '', email: '', role: '' });

  async function handleUpdate() {
    try {
      await updateUser(user);
      toast.success('User updated');
      editDialogOpen = false;
    } catch (error) {
      toast.error('Error updating user');
    }
  }
</script>

<CustomDialog
  bind:open={editDialogOpen}
  title="Edit user"
  description="Modify the user's details"
>
  {#snippet content()}
    <div class="space-y-4">
      <div>
        <Label for="edit-name">Name</Label>
        <Input id="edit-name" bind:value={user.name} />
      </div>
      <div>
        <Label for="edit-email">Email</Label>
        <Input id="edit-email" type="email" bind:value={user.email} />
      </div>
      <div>
        <Label>Role</Label>
        <Select.Root bind:value={user.role}>
          <Select.Trigger>
            <Select.Value placeholder="Select a role..." />
          </Select.Trigger>
          <Select.Content>
            <Select.Item value="admin">Admin</Select.Item>
            <Select.Item value="user">User</Select.Item>
          </Select.Content>
        </Select.Root>
      </div>
    </div>
  {/snippet}

  {#snippet actions()}
    <Button variant="outline" onclick={() => editDialogOpen = false}>
      Cancel
    </Button>
    <Button onclick={handleUpdate}>
      Save
    </Button>
  {/snippet}
</CustomDialog>
```

**Multi-step form:**

```svelte
<script>
  import { CustomDialog } from '$lib/components/ui';
  import { Button } from '$lib/components/ui/button';

  let open = $state(false);
  let step = $state(1);

  function nextStep() { if (step < 3) step++; }
  function prevStep() { if (step > 1) step--; }

  function handleFinish() {
    console.log('Finished');
    open = false;
    step = 1;
  }
</script>

<CustomDialog bind:open title="Create new project" description="Step {step} / 3">
  {#snippet content()}
    {#if step === 1}
      <div>Step 1 content</div>
    {:else if step === 2}
      <div>Step 2 content</div>
    {:else}
      <div>Step 3 content</div>
    {/if}
  {/snippet}

  {#snippet actions()}
    {#if step > 1}
      <Button variant="outline" onclick={prevStep}>Back</Button>
    {/if}
    {#if step < 3}
      <Button onclick={nextStep}>Next</Button>
    {:else}
      <Button onclick={handleFinish}>Finish</Button>
    {/if}
  {/snippet}
</CustomDialog>
```

---

## AlertDialog (native)

If ConfirmDialog and CustomDialog are not sufficient, you can use the native AlertDialog component for full control.

### Usage

```svelte
<script>
  import * as AlertDialog from '$lib/components/ui/alert-dialog';
  import { Button } from '$lib/components/ui/button';

  let open = $state(false);
</script>

<AlertDialog.Root bind:open>
  <AlertDialog.Trigger>
    <Button>Open</Button>
  </AlertDialog.Trigger>
  <AlertDialog.Content>
    <AlertDialog.Header>
      <AlertDialog.Title>Are you sure?</AlertDialog.Title>
      <AlertDialog.Description>
        This action cannot be undone.
      </AlertDialog.Description>
    </AlertDialog.Header>
    <AlertDialog.Footer>
      <AlertDialog.Cancel>Cancel</AlertDialog.Cancel>
      <AlertDialog.Action>Continue</AlertDialog.Action>
    </AlertDialog.Footer>
  </AlertDialog.Content>
</AlertDialog.Root>
```

---

## Drawer

Side panel component for mobile views or quick actions.

### Usage

```svelte
<script>
  import * as Drawer from '$lib/components/ui/drawer';
  import { Button } from '$lib/components/ui/button';

  let open = $state(false);
</script>

<Drawer.Root bind:open>
  <Drawer.Trigger>
    <Button>Open</Button>
  </Drawer.Trigger>
  <Drawer.Content>
    <Drawer.Header>
      <Drawer.Title>Drawer title</Drawer.Title>
      <Drawer.Description>Drawer description</Drawer.Description>
    </Drawer.Header>
    <div class="p-4">
      Drawer content
    </div>
    <Drawer.Footer>
      <Button onclick={() => open = false}>Close</Button>
    </Drawer.Footer>
  </Drawer.Content>
</Drawer.Root>
```

---

## Best Practices

1. **ConfirmDialog for deletion** — Always use a confirmation dialog for dangerous operations
2. **CustomDialog for complex forms** — Use snippets for flexibility
3. **Destructive variant** — Only for deletion and irreversible operations
4. **Toast feedback** — Always give a toast message after closing the dialog
5. **Validation** — Validate form fields before saving
6. **Loading state** — Display loading on buttons
7. **Escape to close** — The dialog automatically closes on Escape
8. **Background click** — The dialog closes when clicking the background
9. **Titles and descriptions** — Always provide clear titles and descriptions
10. **Button order** — Cancel on the left, confirm on the right

## Related

- [Basic Components →](./basic) — Button, Input, Select
- [Toast Notifications →](./notifications) — Feedback
- [DataTable →](./datatable) — Delete confirmation in tables
