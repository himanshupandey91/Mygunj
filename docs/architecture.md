# MyGunj Architecture

## System Overview

MyGunj is a three-tier messaging application:

```
┌─────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│   Web Client    │         │  Mobile Client   │         │   Admin/Tools    │
│   (React)       │         │  (React Native)  │         │   (CLI/Scripts)  │
└────────┬────────┘         └────────┬─────────┘         └────────┬─────────┘
         │                           │                            │
         │      HTTP REST API        │                            │
         ├─────────────────────────────────────────────────────────┤
         │                                                         │
         ▼                                                         ▼
┌──────────────────────────────────────────────────────────────────────────┐
│                        Backend (Express.js)                              │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    API Routes                                   │   │
│  │  • /auth - Authentication                                      │   │
│  │  • /users - User management                                    │   │
│  │  • /conversations - Chat management                            │   │
│  │  • /messages - Message handling                                │   │
│  │  • /ws - WebSocket upgrade                                     │   │
│  └─────────────────────────────────────────────────────────────────┘   │
│                              ▲                                          │
│                              │                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │         Middleware Layer                                      │   │
│  │  • Authentication (JWT verification)                          │   │
│  │  • Authorization (role-based access control)                  │   │
│  │  • Input validation                                           │   │
│  │  • Error handling                                             │   │
│  │  • Logging                                                    │   │
│  │  • CORS                                                       │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                              ▲                                          │
│                              │                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │         Business Logic Layer                                  │   │
│  │  • User service (registration, profile, search)               │   │
│  │  • Authentication service (OTP, JWT, sessions)                │   │
│  │  • Conversation service (create, list, manage)                │   │
│  │  • Message service (send, receive, delivery states)           │   │
│  │  • WebSocket service (connections, subscriptions)             │   │
│  │  • Block service (block/unblock management)                   │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                              ▲                                          │
│                              │                                          │
│  ┌────────────────────────────────────────────────────────────────┐   │
│  │           Data Access Layer (DAL)                             │   │
│  │  • User repository                                            │   │
│  │  • Session repository                                         │   │
│  │  • Conversation repository                                    │   │
│  │  • Message repository                                         │   │
│  │  • Block repository                                           │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                              ▲                                          │
└──────────────────────────────┼──────────────────────────────────────────┘
                               │
                               ▼
                    ┌──────────────────┐
                    │   SQLite DB      │
                    │ (Development)    │
                    │ PostgreSQL       │
                    │ (Production)     │
                    └──────────────────┘
```

## WebSocket Architecture

Real-time messaging is handled via WebSocket connections:

```
Client                           Server                        Database
  │                               │                              │
  ├─────── WS Connect ───────────→│                              │
  │                    (with JWT token)                          │
  │                               │                              │
  │  ←─ Connection Established ───┤                              │
  │                               │                              │
  ├─── Send Message ─────────────→│                              │
  │  { conversation_id, text }    │                              │
  │                               ├─ Create Message Record ─────→│
  │                               │                              │
  │                               ├─ Emit to Recipients ─────────┤
  │  ←─ Message Delivered ────────┤                              │
  │  (with message_id, timestamp) │                              │
  │                               │                              │
  │  ←─ Recipient Message ────────┤  (if recipient online)       │
  │  (real-time update)           │                              │
  │                               │                              │
  ├─── Mark Message Read ────────→│                              │
  │                               ├─ Update Message State ──────→│
  │                               │                              │
  │  ←─ Read Receipt ──────────────┤                              │
  │  (sent to recipient)          │                              │
```

## Data Model

### Users Table
```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY,
  phone_number TEXT UNIQUE NOT NULL,
  display_name TEXT NOT NULL,
  username TEXT UNIQUE NOT NULL,
  profile_photo_url TEXT,
  status TEXT,
  is_verified BOOLEAN DEFAULT FALSE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Sessions Table
```sql
CREATE TABLE sessions (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  token TEXT NOT NULL UNIQUE,
  refresh_token TEXT NOT NULL UNIQUE,
  device_info TEXT,
  ip_address TEXT,
  expires_at DATETIME NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### Conversations Table
```sql
CREATE TABLE conversations (
  id TEXT PRIMARY KEY,
  type TEXT DEFAULT 'direct',  -- 'direct' or 'group'
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Conversation Members Table
```sql
CREATE TABLE conversation_members (
  id TEXT PRIMARY KEY,
  conversation_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  last_read_message_id TEXT,
  last_read_at DATETIME,
  joined_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(conversation_id, user_id)
);
```

### Messages Table
```sql
CREATE TABLE messages (
  id TEXT PRIMARY KEY,
  conversation_id TEXT NOT NULL,
  sender_id TEXT NOT NULL,
  content TEXT NOT NULL,
  state TEXT DEFAULT 'sending',  -- sending, sent, delivered, read, failed
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
  FOREIGN KEY (sender_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### Blocks Table
```sql
CREATE TABLE blocks (
  id TEXT PRIMARY KEY,
  blocker_id TEXT NOT NULL,
  blocked_user_id TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (blocker_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (blocked_user_id) REFERENCES users(id) ON DELETE CASCADE,
  UNIQUE(blocker_id, blocked_user_id)
);
```

## Authentication Flow

```
1. User enters phone number
   └─→ POST /auth/send-otp {phone_number}
       └─→ Backend generates OTP
       └─→ OTP sent via SMS (or stored in dev mode)
       └─→ Returns {session_id}

2. User enters OTP
   └─→ POST /auth/verify-otp {session_id, otp}
       └─→ Backend verifies OTP
       └─→ Creates/retrieves user
       └─→ Generates JWT + Refresh token
       └─→ Returns {user, tokens, expires_at}

3. Client stores tokens
   └─→ Access token in memory (15 min expiry)
   └─→ Refresh token in secure storage (7 day expiry)
   └─→ User ID in local storage

4. Authenticated requests
   └─→ All API calls include: Authorization: Bearer {access_token}
   └─→ Server validates token before processing
   └─→ If expired, client uses refresh token to get new access token

5. WebSocket connection
   └─→ WS connection includes token as query param or header
   └─→ Server validates token before accepting connection
   └─→ Each message is associated with authenticated user
```

## Message Delivery States

```
SENDING
  ↓
SENT (server confirmed)
  ↓
DELIVERED (recipient received via WebSocket)
  ↓
READ (recipient opened message)

Or at any point:
  ↓
FAILED (network error, recipient blocked, etc.)
```

## Error Handling

### HTTP Status Codes
- `200 OK`: Successful request
- `201 Created`: Resource created
- `400 Bad Request`: Invalid input
- `401 Unauthorized`: Missing/invalid auth token
- `403 Forbidden`: Authorized but not permitted
- `404 Not Found`: Resource not found
- `409 Conflict`: Resource already exists
- `500 Internal Server Error`: Unexpected server error

### Error Response Format
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "User-friendly error message",
    "details": { }
  }
}
```

## Security Considerations

1. **Authentication**
   - Phone number as primary identifier
   - OTP-based login (no passwords)
   - Short-lived access tokens (15 min)
   - Refresh tokens for session persistence

2. **Authorization**
   - User can only access own conversations
   - User cannot read/delete others' messages
   - User cannot modify others' profiles
   - Blocked users cannot initiate conversations

3. **Data Protection**
   - All credentials stored hashed/encrypted
   - Messages not end-to-end encrypted (architecture-ready)
   - Tokens never logged
   - Sensitive data excluded from error messages

4. **Input Validation**
   - Phone number format validation
   - Message length limits
   - Username validation
   - XSS prevention

5. **Rate Limiting**
   - OTP request limits
   - API call rate limits
   - WebSocket message limits

## Deployment

### Development
- Backend: `npm run dev` (nodemon + ts-node)
- Web: `npm start` (Vite dev server)
- Mobile: `npm start` (Expo dev server)

### Production
- Backend: Node.js + PM2 or Docker
- Web: Nginx/CDN + static build
- Mobile: EAS build → APK/IPA → app stores
- Database: PostgreSQL + backups
- WebSocket: Native support via Node.js cluster

## Performance Considerations

1. **Message Pagination**
   - Load 50 messages per request
   - Infinite scroll pattern
   - Cache last 10 conversations

2. **WebSocket Optimization**
   - Connection pooling
   - Message batching for high volume
   - Heartbeat every 30 seconds

3. **Database Indexes**
   - Composite index on (conversation_id, created_at)
   - Index on (user_id, created_at)
   - Index on (phone_number, username) for search

4. **Caching**
   - User profile cache (5 min)
   - Conversation list cache (30 sec)
   - Block list cache (5 min)
