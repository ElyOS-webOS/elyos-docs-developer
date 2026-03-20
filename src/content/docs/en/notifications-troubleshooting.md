---
title: Troubleshooting
description: Notification system troubleshooting and fallback behavior
next:
  link: /en/ui-components/
  label: UI Components
---

This page helps resolve the most common issues and explains the fallback mechanism in detail.

## Notifications Not Appearing

### 1. Check the Socket.IO Connection

```typescript
const notificationStore = getNotificationStore();
console.log('Connected:', notificationStore.isConnected);
```

**Expected result:**
- `true` — Socket.IO is working, real-time notifications
- `false` — Fallback mode, 30s polling

### 2. Check the Console

**Server console:**

```bash
[Socket.IO] Client connected: abc123
[Socket.IO] User registered: 123
[Socket.IO] Notification sent to user: 123
```

**Client console:**

```bash
[NotificationStore] Connected to Socket.IO
[NotificationStore] New notification received via WebSocket: {...}
[NotificationStore] About to show toast for notification: 1
```

### 3. Socket.IO Not Working

If Socket.IO is unavailable, fallback activates automatically:

```bash
# Client console
[NotificationStore] Connection error: ...
[NotificationStore] WebSocket disconnected, polling for notifications
```

**Solution:** No action needed, the system automatically switches to fallback mode.

### 4. Use the Reload Function in Dev Mode

```typescript
await notificationStore.reload();
```

## Socket.IO vs REST API Fallback

| Aspect | Socket.IO (Primary) | REST API (Fallback) |
|--------|---------------------|---------------------|
| **Latency** | Immediate (< 100ms) | 30 seconds |
| **Network traffic** | Minimal | Moderate (polling) |
| **Reliability** | High | Very high |
| **Usage** | Automatic | Automatic fallback |
| **Activation** | On connection | On Socket.IO error |

## Testing Fallback Mode

### 1. Stop Socket.IO

```bash
# Stop the Socket.IO server
# The client will automatically switch to fallback mode
```

### 2. Console Messages

```bash
[NotificationStore] Disconnected from Socket.IO
[NotificationStore] WebSocket disconnected, polling for notifications
```

### 3. Send a Notification

```typescript
await sendNotification({
  userId: 123,
  title: { hu: 'Teszt', en: 'Test' },
  message: { hu: 'Fallback mód teszt', en: 'Fallback mode test' },
  type: 'info'
});

// The notification will appear within 30 seconds
```

## Toast Notifications Not Appearing

### 1. Check the Toaster Component

```svelte
<!-- +layout.svelte -->
<script>
  import { Toaster } from 'svelte-sonner';
</script>

<Toaster />
```

**Solution:** Add the `Toaster` component to the layout.

### 2. Check the Notification Type

Critical type notifications don't appear as toasts — they appear in a modal dialog.

```typescript
// ❌ Won't show as toast
await sendNotification({
  userId: 123,
  title: 'Critical',
  message: 'This appears in a modal dialog',
  type: 'critical'
});

// ✅ Will show as toast
await sendNotification({
  userId: 123,
  title: 'Info',
  message: 'This appears as a toast',
  type: 'info'
});
```

### 3. Check the Console

```bash
[NotificationStore] About to show toast for notification: 1
[NotificationStore] Toast imported successfully
[NotificationStore] Showing toast: { title: '...', message: '...', type: 'info' }
```

## Dev Mode Specific Issues

### Issue: Socket.IO Not Running in Dev Mode

**Symptom:** `isConnected` is always `false`

**Solution:** The system automatically switches to fallback mode. Use the `reload()` method for manual refresh:

```typescript
const notificationStore = getNotificationStore();
await notificationStore.reload();
```

### Issue: Toast Notifications Not Appearing in Dev Mode

**Symptom:** New notifications don't appear automatically

**Solution:** The `reload()` method automatically shows toasts for new notifications:

```typescript
// In notification-demo app
if (import.meta.env.DEV) {
  setTimeout(() => {
    notificationStore.reload();
  }, 500);
}
```

### Issue: Notifications Don't Update Automatically

**Symptom:** New notifications only appear after page refresh

**Cause:** Socket.IO is not running and polling hasn't started yet

**Solution:** Wait 30 seconds, or use the `reload()` method:

```typescript
await notificationStore.reload();
```

## Notifications Not Deleting

### Check Permissions

Only your own notifications can be deleted. Check that the `userId` matches the logged-in user.

```typescript
// ✅ Good - deleting own notification
await notificationStore.deleteNotification(myNotificationId);

// ❌ Error - deleting another user's notification
await notificationStore.deleteNotification(otherUserNotificationId);
```

### Check the Console

```bash
# Successful deletion
[API] Notification deleted: 123

# Failed deletion
[API] Error deleting notification: Unauthorized
```

## Browser Notifications Not Appearing

### 1. Check Permissions

```typescript
console.log('Notification permission:', Notification.permission);
```

**Possible values:**
- `granted` — Allowed
- `denied` — Blocked
- `default` — Not yet requested

### 2. Request Permission

```typescript
if (Notification.permission === 'default') {
  await Notification.requestPermission();
}
```

### 3. Check Browser Support

```typescript
if (!('Notification' in window)) {
  console.error('Browser does not support notifications');
}
```

## Critical Notifications Not Appearing

### Check the CriticalNotificationDialog Component

```svelte
<!-- Desktop.svelte -->
<script>
  import CriticalNotificationDialog from '$lib/components/core/CriticalNotificationDialog.svelte';
</script>

<CriticalNotificationDialog />
```

**Solution:** Add the `CriticalNotificationDialog` component to the Desktop component.

### Check the Type

```typescript
// ✅ Good - critical type
await sendNotification({
  userId: 123,
  title: 'Critical',
  message: 'This appears in a modal dialog',
  type: 'critical'
});

// ❌ Wrong - error type (appears as toast)
await sendNotification({
  userId: 123,
  title: 'Error',
  message: 'This appears as a toast',
  type: 'error'
});
```

## Performance Issues

### Too Many Notifications

If there are too many notifications, the list may be slow.

**Solution:** Use pagination or delete old notifications:

```typescript
// Delete all notifications
await notificationStore.deleteAllNotifications();

// Or delete only old ones (server-side)
await notificationRepository.deleteOlderThan(30); // older than 30 days
```

### Polling Too Frequent

If polling is too frequent, increase the interval:

```typescript
// notificationStore.svelte.ts
setInterval(() => {
  if (browser && !this.state.isConnected) {
    this.loadNotifications();
  }
}, 60000); // 60 seconds instead of 30
```

## Debugging Tips

### 1. Enable Console Messages

```typescript
// notificationStore.svelte.ts
console.log('[NotificationStore] Connected:', this.state.isConnected);
console.log('[NotificationStore] Notifications:', this.state.notifications);
console.log('[NotificationStore] Unread count:', this.state.unreadCount);
```

### 2. Check the Network Tab

Open the browser DevTools Network tab and filter for:
- `socket.io` — WebSocket connection
- `/api/notifications` — REST API calls

### 3. Svelte DevTools

Use Svelte DevTools to track state:

```bash
# Install
npm install -D @sveltejs/vite-plugin-svelte-inspector
```

### 4. Server-Side Logging

```typescript
// lib/server/socket/index.ts
logger.info('[Socket.IO] Notification sent to user:', userId);
logger.info('[Socket.IO] Unread count:', unreadCount);
```

## Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| Notifications not appearing | Socket.IO not running | Wait 30s (fallback) or `reload()` |
| Toast not appearing | Toaster component missing | Add to layout |
| Critical dialog not appearing | CriticalNotificationDialog missing | Add to Desktop |
| Notifications not deleting | Permission error | Can only delete own notifications |
| Browser notifications not working | Permission missing | Request permission from user |

## Next Steps

- [NotificationStore →](/en/notifications-store)
- [API & Socket.IO →](/en/notifications-api)
- [Notification System Overview →](/en/notifications)
