---
title: NotificationStore
description: Global Svelte store for managing notifications
---

The NotificationStore is a global Svelte 5 store that manages notification state and the Socket.IO connection. It automatically falls back to REST API polling if WebSocket is unavailable.

## Automatic Fallback

The store intelligently handles connection issues:

1. **Socket.IO connection succeeds** → Real-time notifications via WebSocket
2. **Socket.IO connection fails** → Automatic polling every 30 seconds
3. **Socket.IO connection drops** → Automatic switch to polling
4. **Socket.IO reconnects** → Automatic switch back to WebSocket

```typescript
// Automatic initialization
const notificationStore = getNotificationStore();
await notificationStore.connect(userId);

// The store automatically:
// 1. Tries to connect via Socket.IO
// 2. If it fails, starts polling
// 3. Loads notifications via REST API
// 4. Refreshes every 30 seconds if no WebSocket connection
```

## Usage

### Getting the Store

```typescript
import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

const notificationStore = getNotificationStore();
```

### Reactive State

```typescript
// Notification list
const notifications = $derived(notificationStore.notifications);

// Unread notification count
const unreadCount = $derived(notificationStore.unreadCount);

// Socket.IO connection status
const isConnected = $derived(notificationStore.isConnected);

// Current critical notification (if any)
const currentCritical = $derived(notificationStore.currentCritical);

// Whether there are unread critical notifications
const hasUnreadCritical = $derived(notificationStore.hasUnreadCritical);
```

### In a Svelte Component

```svelte
<script>
  import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

  const notificationStore = getNotificationStore();

  const notifications = $derived(notificationStore.notifications);
  const unreadCount = $derived(notificationStore.unreadCount);
</script>

<div>
  <p>Unread notifications: {unreadCount}</p>

  {#each notifications as notification}
    <div>
      <h3>{notification.title}</h3>
      <p>{notification.message}</p>
    </div>
  {/each}
</div>
```

## API Methods

### connect(userId)

Initialize Socket.IO connection and load notifications.

```typescript
await notificationStore.connect(123);
```

**Happens automatically:** `hooks.client.ts` calls this automatically on login.

### disconnect()

Disconnect Socket.IO connection.

```typescript
notificationStore.disconnect();
```

### loadNotifications(showToast?)

Load notifications via REST API.

```typescript
await notificationStore.loadNotifications();

// With toast display (in dev mode)
await notificationStore.loadNotifications(true);
```

**Happens automatically:**
- On initialization
- Every 30 seconds if Socket.IO is unavailable

### reload()

Reload notifications with toast display (useful in dev mode).

```typescript
await notificationStore.reload();
```

### markAsRead(notificationId)

Mark a notification as read.

```typescript
await notificationStore.markAsRead(123);
```

**Effect:**
- Updates local state
- REST API call
- Socket.IO emit (if available)

### markAllAsRead()

Mark all notifications as read.

```typescript
await notificationStore.markAllAsRead();
```

### deleteNotification(notificationId)

Delete a notification.

```typescript
await notificationStore.deleteNotification(123);
```

### deleteAllNotifications()

Delete all notifications.

```typescript
await notificationStore.deleteAllNotifications();
```

### sendNotification(payload)

Send a new notification.

```typescript
await notificationStore.sendNotification({
  userId: 123,
  title: { hu: 'Teszt', en: 'Test' },
  message: { hu: 'Teszt üzenet', en: 'Test message' },
  type: 'info'
});
```

**Note:** Prefer using the `sendNotification` function from `notificationService`.

### getAppNotifications(appName)

Get notifications for a specific app.

```typescript
const userNotifications = notificationStore.getAppNotifications('users');
```

### getAppUnreadCount(appName)

Get unread notification count for a specific app.

```typescript
const unreadCount = notificationStore.getAppUnreadCount('users');
```

### acknowledgeCritical()

Acknowledge the current critical notification (remove from queue).

```typescript
notificationStore.acknowledgeCritical();
```

**Happens automatically:** The `CriticalNotificationDialog` component calls this when the OK button is clicked.

## State Properties

### notifications

Array of notifications in descending chronological order.

```typescript
const notifications: Notification[] = notificationStore.notifications;
```

### unreadCount

Number of unread notifications.

```typescript
const unreadCount: number = notificationStore.unreadCount;
```

### isConnected

Socket.IO connection status.

```typescript
const isConnected: boolean = notificationStore.isConnected;
```

### currentCritical

Current critical notification (first in queue).

```typescript
const currentCritical: Notification | null = notificationStore.currentCritical;
```

### hasUnreadCritical

Whether there are unread critical notifications.

```typescript
const hasUnreadCritical: boolean = notificationStore.hasUnreadCritical;
```

## Initialization

The store is automatically initialized in `hooks.client.ts`:

```typescript
// hooks.client.ts
import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

export async function handleSession({ data }) {
  if (data.user?.id) {
    const notificationStore = getNotificationStore();
    await notificationStore.connect(parseInt(data.user.id));
  }
}
```

## Internal Behavior

### Socket.IO Event Handling

```typescript
// New notification received
socket.on('notification:new', (notification: Notification) => {
  // Add to list
  this.state.notifications = [notification, ...this.state.notifications];
  this.state.unreadCount++;

  // Show toast (except critical)
  this.showToastNotification(notification);

  // Browser notification
  this.showBrowserNotification(notification);
});

// Update unread counter
socket.on('notification:unread-count', (count: number) => {
  this.state.unreadCount = count;
});
```

### Automatic Polling

```typescript
// Polling every 30 seconds if Socket.IO is unavailable
setInterval(() => {
  if (browser && !this.state.isConnected) {
    console.log('[NotificationStore] WebSocket disconnected, polling for notifications');
    this.loadNotifications();
  }
}, 30000);
```

### Toast Display

```typescript
private showToastNotification(notification: Notification) {
  // Critical notifications go to the dialog
  if (notification.type === 'critical') {
    this._criticalQueue = [...this._criticalQueue, notification];
    return;
  }

  // Show toast
  import('svelte-sonner').then(({ toast }) => {
    const toastFn = toast[notification.type] || toast.info;
    toastFn(title, {
      description: message,
      duration: 5000,
      action: {
        label: 'Open',
        onClick: () => this.openNotificationInApp(notification.id)
      }
    });
  });
}
```

## Examples

### Display Notifications in a List

```svelte
<script>
  import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

  const notificationStore = getNotificationStore();
  const notifications = $derived(notificationStore.notifications);

  async function handleMarkAsRead(id: number) {
    await notificationStore.markAsRead(id);
  }

  async function handleDelete(id: number) {
    await notificationStore.deleteNotification(id);
  }
</script>

<div>
  {#each notifications as notification}
    <div class:unread={!notification.isRead}>
      <h3>{notification.title}</h3>
      <p>{notification.message}</p>
      <p>{new Date(notification.createdAt).toLocaleString()}</p>

      {#if !notification.isRead}
        <button onclick={() => handleMarkAsRead(notification.id)}>
          Mark as read
        </button>
      {/if}

      <button onclick={() => handleDelete(notification.id)}>
        Delete
      </button>
    </div>
  {/each}
</div>
```

### Unread Count Badge

```svelte
<script>
  import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

  const notificationStore = getNotificationStore();
  const unreadCount = $derived(notificationStore.unreadCount);
</script>

<button>
  Notifications
  {#if unreadCount > 0}
    <span class="badge">{unreadCount}</span>
  {/if}
</button>
```

### App-Specific Notifications

```svelte
<script>
  import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

  const notificationStore = getNotificationStore();
  const appNotifications = $derived(notificationStore.getAppNotifications('users'));
  const appUnreadCount = $derived(notificationStore.getAppUnreadCount('users'));
</script>

<div>
  <h2>Users App Notifications ({appUnreadCount} unread)</h2>

  {#each appNotifications as notification}
    <div>{notification.message}</div>
  {/each}
</div>
```

### Connection Status Display

```svelte
<script>
  import { getNotificationStore } from '$lib/stores/notificationStore.svelte';

  const notificationStore = getNotificationStore();
  const isConnected = $derived(notificationStore.isConnected);
</script>

<div class="status">
  {#if isConnected}
    <span class="online">● Online (WebSocket)</span>
  {:else}
    <span class="offline">● Offline (Polling)</span>
  {/if}
</div>
```

## Next Steps

- [UI Components →](/en/notifications-ui)
- [API & Socket.IO →](/en/notifications-api)
- [Troubleshooting →](/en/notifications-troubleshooting)
- [Notification System Overview →](/en/notifications)
