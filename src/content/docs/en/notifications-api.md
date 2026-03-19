---
title: API and Socket.IO
description: REST API endpoints and Socket.IO events
---

The notification system uses REST API endpoints and Socket.IO events for communication.

## REST API Endpoints

The API endpoints serve as a **fallback solution** when Socket.IO is unavailable. The NotificationStore automatically starts polling every 30 seconds if the WebSocket connection is lost.

### Automatic Fallback Behavior

```typescript
// notificationStore.svelte.ts
// Poll for new notifications only if WebSocket is disconnected
setInterval(() => {
  if (browser && !this.state.isConnected) {
    console.log('[NotificationStore] WebSocket disconnected, polling for notifications');
    this.loadNotifications();
  }
}, 30000);
```

**Important:** The API endpoints do not need to be called manually during normal usage. The store automatically handles the fallback.

### GET /api/notifications

Fetch all notifications for the current user.

**Response:**

```json
{
  "notifications": [
    {
      "id": 1,
      "userId": 123,
      "appName": "users",
      "title": { "hu": "Új csoport", "en": "New group" },
      "message": { "hu": "Csoport létrehozva", "en": "Group created" },
      "details": null,
      "type": "success",
      "isRead": false,
      "data": { "section": "groups", "groupId": "456" },
      "createdAt": "2024-01-15T10:30:00Z",
      "readAt": null
    }
  ]
}
```

### POST /api/notifications

Send a new notification.

**Request:**

```json
{
  "userId": 123,
  "appName": "users",
  "title": { "hu": "Új csoport", "en": "New group" },
  "message": { "hu": "Csoport létrehozva", "en": "Group created" },
  "type": "success",
  "data": { "section": "groups", "groupId": "456" }
}
```

### POST /api/notifications/[id]/read

Mark a notification as read.

### POST /api/notifications/[id]/delete

Delete a notification.

### POST /api/notifications/read-all

Mark all notifications as read.

### POST /api/notifications/delete-all

Delete all notifications.

## Socket.IO Events

### Client → Server

#### register

Register a user on Socket.IO.

```typescript
socket.emit('register', userId);
```

**Happens automatically:** The NotificationStore automatically registers the user on connection.

#### notification:mark-read

Mark a notification as read.

```typescript
socket.emit('notification:mark-read', notificationId);
```

#### notification:mark-all-read

Mark all notifications as read.

```typescript
socket.emit('notification:mark-all-read', userId);
```

### Server → Client

#### notification:new

A new notification has arrived.

```typescript
socket.on('notification:new', (notification: Notification) => {
  console.log('New notification:', notification);
});
```

**Handled automatically:** The NotificationStore automatically handles this event.

#### notification:unread-count

Unread notification count updated.

```typescript
socket.on('notification:unread-count', (count: number) => {
  console.log('Unread notifications:', count);
});
```

## Socket.IO Server Implementation

### Initialization

```typescript
// lib/server/socket/index.ts
export function initializeSocketIO(serverOrIo: HTTPServer | SocketIOServer) {
  if (io) {
    logger.warn('[Socket.IO] Already initialized');
    return io;
  }

  io = new SocketIOServer(serverOrIo, {
    cors: { origin: '*', methods: ['GET', 'POST'] },
    path: '/socket.io/',
    pingTimeout: 60000,
    pingInterval: 25000,
    transports: ['websocket', 'polling']
  });

  // Event handlers...
}
```

### Sending a Notification

```typescript
export async function sendNotification(payload: NotificationPayload): Promise<void> {
  let socketIO: SocketIOServer | null = null;

  try {
    socketIO = getSocketIO();
  } catch (error) {
    console.warn('[sendNotification] Socket.IO not initialized, will save to database only');
  }

  let targetUserIds: number[] = [];

  if (payload.broadcast) {
    const allUsers = await db.select({ id: users.id }).from(users);
    targetUserIds = allUsers.map((u) => u.id);
  } else if (payload.userId) {
    targetUserIds = [payload.userId];
  } else if (payload.userIds) {
    targetUserIds = payload.userIds;
  }

  for (const userId of targetUserIds) {
    const notification: NewNotification = {
      userId,
      appName: payload.appName || null,
      title: normalizeContent(payload.title) as any,
      message: normalizeContent(payload.message) as any,
      type: payload.type || 'info',
      data: payload.data || null
    };

    const saved = await notificationRepository.create(notification);

    if (socketIO) {
      socketIO.to(`user:${userId}`).emit('notification:new', saved);
      const unreadCount = await notificationRepository.getUnreadCount(userId);
      socketIO.to(`user:${userId}`).emit('notification:unread-count', unreadCount);
    }
  }
}
```

## Notification Repository

```typescript
export const notificationRepository = {
  async create(notification: NewNotification): Promise<Notification> {
    const [created] = await db.insert(notifications).values(notification).returning();
    return created;
  },

  async getByUserId(userId: number, limit = 50): Promise<Notification[]> {
    return db.select().from(notifications)
      .where(eq(notifications.userId, userId))
      .orderBy(desc(notifications.createdAt))
      .limit(limit);
  },

  async getUnreadCount(userId: number): Promise<number> {
    const result = await db.select().from(notifications)
      .where(and(eq(notifications.userId, userId), eq(notifications.isRead, false)));
    return result.length;
  },

  async markAsRead(id: number): Promise<Notification | undefined> {
    const [updated] = await db.update(notifications)
      .set({ isRead: true, readAt: new Date() })
      .where(eq(notifications.id, id))
      .returning();
    return updated;
  },

  async markAllAsRead(userId: number): Promise<void> {
    await db.update(notifications)
      .set({ isRead: true, readAt: new Date() })
      .where(and(eq(notifications.userId, userId), eq(notifications.isRead, false)));
  },

  async delete(id: number): Promise<void> {
    await db.delete(notifications).where(eq(notifications.id, id));
  },

  async deleteAllByUserId(userId: number): Promise<void> {
    await db.delete(notifications).where(eq(notifications.userId, userId));
  }
};
```

## Next Steps

- [Troubleshooting →](/en/notifications-troubleshooting)
- [NotificationStore →](/en/notifications-store)
- [Notification System Overview →](/en/notifications)
