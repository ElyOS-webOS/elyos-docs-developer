---
title: Chat Application
description: Real-time messaging system with Socket.IO
---

The Chat application is a real-time messaging system that enables direct communication between users. It features a Socket.IO-based real-time connection with a REST API fallback.

## Overview

The Chat application consists of three main parts:
- **User list** (right sidebar) - with online/offline status
- **Conversation list** - with unread message counts
- **Chat window** - message display and sending

### Key Features

- Real-time messaging with Socket.IO
- Online/offline status tracking
- Typing indicator
- Unread message counting
- Automatic fallback to REST API (dev mode)
- Toast notifications for new messages
- Automatic conversation sorting (newest first)

## File Structure

```
apps/chat/
├── index.svelte              # Main layout (sidebar + conversations + chat window)
├── chat.remote.ts            # Server actions (messages, conversations)
├── components/
│   ├── UserList.svelte       # User list with online/offline grouping
│   ├── ConversationList.svelte  # Conversation list
│   └── ChatWindow.svelte     # Message display and sending
└── stores/
    └── chatStore.svelte.ts   # Chat state management and Socket.IO connection
```

## Server Actions

### `chat.remote.ts`

The Chat application defines 8 server actions:

#### 1. `getChatUsers` (query)

Returns all users (except the current user) for starting a chat.

```typescript
const result = await getChatUsers();
// { success: true, users: ChatUser[] }
```

#### 2. `getConversations` (query)

Fetches all conversations for the user with the last message and unread count.

```typescript
const result = await getConversations();
// { success: true, conversations: ConversationWithLastMessage[] }
```

#### 3. `getMessages` (command)

Fetches messages for a conversation with pagination.

```typescript
const result = await getMessages({
  conversationId: 1,
  limit: 50,      // optional, default: 50
  offset: 0       // optional, default: 0
});
// { success: true, messages: MessageWithSender[] }
```

**Validation:**
- `conversationId`: minimum 1
- `limit`: between 1-100
- `offset`: minimum 0
- Checks that the user is a member of the conversation

#### 4. `sendMessage` (command)

Send a new message to a user.

```typescript
const result = await sendMessage({
  recipientId: 2,
  content: "Hello!"
});
// { success: true, message: MessageWithSender, conversationId: number }
```

**Validation:**
- `recipientId`: minimum 1
- `content`: between 1-5000 characters

**How it works:**
1. Gets or creates the conversation
2. Saves the message to the database
3. Returns the message with sender data

#### 5. `markMessagesAsRead` (command)

Mark conversation messages as read.

```typescript
const result = await markMessagesAsRead({
  conversationId: 1
});
// { success: true }
```

#### 6. `getUnreadCount` (query)

Get the total number of unread messages.

```typescript
const result = await getUnreadCount();
// { success: true, count: number }
```

#### 7. `getCurrentUserId` (query)

Get the current user's ID.

```typescript
const result = await getCurrentUserId();
// { success: true, userId: number }
```

#### 8. `getOrCreateConversation` (command)

Get or create a conversation with a user.

```typescript
const result = await getOrCreateConversation({
  otherUserId: 2
});
// { success: true, conversationId: number }
```

## ChatStore

`chatStore.svelte.ts` manages the chat state and Socket.IO connection.

### State

```typescript
interface ChatState {
  conversations: ConversationWithLastMessage[];
  activeConversationId: number | null;
  messages: MessageWithSender[];
  unreadCount: number;
  isConnected: boolean;              // Socket.IO connection state
  onlineUsers: Set<number>;          // Online user IDs
  typingUsers: Map<number, boolean>; // Typing users per conversation
}
```

### Key Methods

#### `connect(userId: number)`

Initialize Socket.IO connection and set up event listeners.

```typescript
const chatStore = getChatStore();
await chatStore.connect(userId);
```

**Socket.IO events:**
- `chat:new-message` - New message received
- `chat:user-online` - User came online
- `chat:user-offline` - User went offline
- `chat:online-users` - List of online users
- `chat:user-typing` - Typing indicator

**Fallback behavior:**
- If Socket.IO is unavailable, polls every 10 seconds
- In dev mode, automatically uses polling

#### `disconnect()`

Disconnect Socket.IO and stop polling.

```typescript
chatStore.disconnect();
```

#### `loadConversations()`

Reload conversations from the API.

```typescript
await chatStore.loadConversations();
```

#### `loadMessages(conversationId: number)`

Load messages for a conversation and make it active.

```typescript
await chatStore.loadMessages(1);
```

**Side effects:**
- Sets `activeConversationId`
- Automatically marks messages as read
- Updates unread count

#### `sendMessage(recipientId: number, content: string)`

Send a message via Socket.IO.

```typescript
const result = await chatStore.sendMessage(2, "Hello!");
```

**How it works:**
1. Calls the `sendMessage` server action
2. Sends via Socket.IO (`chat:send-message` event)
3. Adds the message to local state (if active conversation)
4. Updates the conversation list

#### `markAsRead(conversationId: number)`

Mark messages as read.

```typescript
await chatStore.markAsRead(1);
```

#### `sendTypingIndicator(recipientId, conversationId, isTyping)`

Send typing indicator via Socket.IO.

```typescript
chatStore.sendTypingIndicator(2, 1, true);  // Start typing
chatStore.sendTypingIndicator(2, 1, false); // Stop typing
```

#### `isUserOnline(userId: number): boolean`

Check if a user is online.

```typescript
if (chatStore.isUserOnline(2)) {
  console.log('User is online');
}
```

#### `isUserTyping(conversationId: number): boolean`

Check if someone is typing in a conversation.

```typescript
if (chatStore.isUserTyping(1)) {
  console.log('Other user is typing...');
}
```

## Components

### UserList.svelte

User list with online/offline grouping and search.

**Features:**
- Search by name and username
- Collapsible online/offline groups
- Status indicator (green/grey)
- Click to start a conversation

**Usage:**

```svelte
<UserList />
```

### ConversationList.svelte

Conversation list with last message and unread count.

**Features:**
- Automatic sorting (newest first)
- Unread message badge
- Active conversation highlight
- Refresh button
- Relative timestamps (e.g. "2 minutes ago")

**Usage:**

```svelte
<ConversationList />
```

### ChatWindow.svelte

Message display and sending.

**Features:**
- Messages grouped by sender
- Avatar display
- Typing indicator with animation
- Send with Enter key
- Auto-scroll to new messages
- Empty state handling

**Usage:**

```svelte
<ChatWindow currentUserId={userId} />
```

**Props:**
- `currentUserId`: Current user's ID (to distinguish messages)

## Socket.IO Integration

### Server Side

The Socket.IO server is configured in `server.js` (Express + Socket.IO).

**Events (server → client):**
- `chat:new-message` - New message received
- `chat:user-online` - User came online
- `chat:user-offline` - User went offline
- `chat:online-users` - List of online users
- `chat:user-typing` - Typing indicator

**Events (client → server):**
- `register` - Register user (userId)
- `chat:send-message` - Send message
- `chat:mark-read` - Mark messages as read
- `chat:typing` - Send typing indicator

### Client Side

ChatStore automatically manages the Socket.IO connection:

```typescript
// Connect
const chatStore = getChatStore();
await chatStore.connect(userId);

// Automatic reconnection
// - Infinite retries
// - 1-5 second delay
// - WebSocket + polling fallback
```

## Toast Notifications

When a new message arrives (if not in the active conversation):

```typescript
toast.info(senderName, {
  description: messagePreview,
  duration: 5000,
  action: {
    label: 'Open',
    onClick: () => openMessageInChat(conversationId)
  }
});
```

**How it works:**
1. Dynamically imports `svelte-sonner` toast
2. Shows the sender's name and message preview
3. "Open" button opens the conversation

## Database Schema

### `conversations` table

```typescript
{
  id: number;
  participant1Id: number;
  participant2Id: number;
  lastMessageAt: Date | null;
  createdAt: Date;
}
```

### `messages` table

```typescript
{
  id: number;
  conversationId: number;
  senderId: number;
  content: string;
  isRead: boolean;
  readAt: Date | null;
  sentAt: Date;
}
```

## ChatRepository

`chatRepository.ts` handles database operations.

### Key Methods

#### `getOrCreateConversation(userId1, userId2)`

Get or create a conversation between two users.

```typescript
const conversation = await chatRepository.getOrCreateConversation(1, 2);
```

**How it works:**
- Checks both directions (participant1 ↔ participant2)
- Creates if it doesn't exist

#### `getUserConversations(userId)`

Get all conversations for a user.

```typescript
const conversations = await chatRepository.getUserConversations(1);
```

**Returns:**
- Conversation data
- Other user's name and image
- Last message
- Unread message count

#### `getConversationMessages(conversationId, limit, offset)`

Get messages for a conversation.

```typescript
const messages = await chatRepository.getConversationMessages(1, 50, 0);
```

**How it works:**
- Paginated (limit + offset)
- Chronological order (oldest → newest)
- Sender name and image attached

#### `sendMessage(conversationId, senderId, content)`

Save a new message.

```typescript
const message = await chatRepository.sendMessage(1, 2, "Hello!");
```

**Side effects:**
- Updates the conversation's `lastMessageAt` field

#### `markMessagesAsRead(conversationId, userId)`

Mark conversation messages as read.

```typescript
await chatRepository.markMessagesAsRead(1, 2);
```

**How it works:**
- Only marks the other user's messages as read
- Sets `isRead` and `readAt` fields

#### `getUserUnreadCount(userId)`

Get the total number of unread messages.

```typescript
const count = await chatRepository.getUserUnreadCount(1);
```

## Usage Examples

### Starting the Chat Application

```typescript
import { getChatStore } from '$apps/chat/stores/chatStore.svelte';
import { getCurrentUserId } from '$apps/chat/chat.remote';

// Get user ID
const result = await getCurrentUserId();
if (result.success && result.userId) {
  // Initialize ChatStore
  const chatStore = getChatStore();
  await chatStore.connect(result.userId);
}
```

### Starting a New Conversation

```typescript
import { getChatStore } from '$apps/chat/stores/chatStore.svelte';
import { getOrCreateConversation } from '$apps/chat/chat.remote';

const chatStore = getChatStore();

// Create conversation
const result = await getOrCreateConversation({ otherUserId: 2 });

if (result.success && result.conversationId) {
  // Refresh conversations
  await chatStore.loadConversations();

  // Open conversation
  await chatStore.loadMessages(result.conversationId);
}
```

### Sending a Message

```typescript
import { getChatStore } from '$apps/chat/stores/chatStore.svelte';

const chatStore = getChatStore();

// Send message
const result = await chatStore.sendMessage(2, "Hello, how are you?");

if (result.success) {
  console.log('Message sent');
}
```

### Typing Indicator

```typescript
import { getChatStore } from '$apps/chat/stores/chatStore.svelte';

const chatStore = getChatStore();
let typingTimeout: ReturnType<typeof setTimeout> | null = null;

function handleInput(recipientId: number, conversationId: number) {
  // Start typing
  chatStore.sendTypingIndicator(recipientId, conversationId, true);

  // Clear timeout
  if (typingTimeout) clearTimeout(typingTimeout);

  // Stop typing after 3 seconds
  typingTimeout = setTimeout(() => {
    chatStore.sendTypingIndicator(recipientId, conversationId, false);
  }, 3000);
}
```

## Translations

Chat application translations are in the `translations.chat` namespace:

```sql
-- packages/database/src/seeds/translations/chat.ts
INSERT INTO translations (namespace, key, locale, value) VALUES
  ('chat', 'title', 'hu', 'Chat'),
  ('chat', 'title', 'en', 'Chat'),
  ('chat', 'users', 'hu', 'Felhasználók'),
  ('chat', 'users', 'en', 'Users'),
  ('chat', 'conversations', 'hu', 'Beszélgetések'),
  ('chat', 'conversations', 'en', 'Conversations'),
  -- ...
```

**Usage in component:**

```svelte
<script>
  import { I18nProvider } from '$lib/i18n/components';
</script>

<I18nProvider namespaces={['chat', 'common']}>
  <!-- Chat components -->
</I18nProvider>
```

## Best Practices

1. **Socket.IO connection management**: Always call `disconnect()` when the application closes
2. **Typing indicator**: Use a timeout to avoid sending continuous events
3. **Message pagination**: Implement "load more" for large conversations
4. **Offline operation**: The fallback polling ensures it works in dev mode too
5. **Toast notifications**: Only show when the message is not in the active conversation
6. **Unread counting**: Automatically updated with Socket.IO events
7. **Avatar images**: Use `referrerpolicy="no-referrer"` and `crossorigin="anonymous"` attributes
8. **Message grouping**: Only show avatar when there is a new sender

## Troubleshooting

### Socket.IO Not Connecting

**Problem**: `isConnected` always stays `false`.

**Solution**:
1. Check that the Socket.IO server is running (`server.js`)
2. Check the browser console for Socket.IO errors
3. In dev mode, polling fallback activates automatically

### Messages Not Showing

**Problem**: New message doesn't appear in the chat window.

**Solution**:
1. Check that the active conversation ID is correct
2. Check if the `chat:new-message` event is arriving (DevTools Network tab)
3. Check the `currentUserId` prop on the `ChatWindow` component

### Typing Indicator Not Working

**Problem**: The typing indicator doesn't appear.

**Solution**:
1. Check that Socket.IO is connected
2. Check the sending and receiving of the `chat:typing` event
3. Check the `typingUsers` Map in the store

### Online Status Not Updating

**Problem**: Users always appear offline.

**Solution**:
1. Check the `chat:online-users` event reception
2. Check the `onlineUsers` Set in the store
3. In dev mode, polling updates the online status
