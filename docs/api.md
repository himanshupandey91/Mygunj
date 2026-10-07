# MyGunj API Documentation

## Base URL
- Development: `http://localhost:3000`
- Production: `https://api.mygunj.app`

## Authentication

All authenticated endpoints require an `Authorization` header:
```
Authorization: Bearer {access_token}
```

## API Endpoints

### Authentication

#### Send OTP
```
POST /api/auth/send-otp
Body: { phone_number: string }
Response: { session_id: string, expires_in: number }
```

#### Verify OTP
```
POST /api/auth/verify-otp
Body: { session_id: string, otp: string }
Response: {
  user: { id, phone_number, display_name, username, profile_photo_url },
  access_token: string,
  refresh_token: string,
  expires_in: number
}
```

#### Refresh Token
```
POST /api/auth/refresh
Body: { refresh_token: string }
Response: { access_token: string, expires_in: number }
```

#### Logout
```
POST /api/auth/logout
Headers: { Authorization: Bearer {token} }
Response: { success: true }
```

### Users

#### Get Current User
```
GET /api/users/me
Headers: { Authorization: Bearer {token} }
Response: { id, phone_number, display_name, username, profile_photo_url, status, is_verified }
```

#### Update Profile
```
PUT /api/users/me
Headers: { Authorization: Bearer {token} }
Body: {
  display_name?: string,
  username?: string,
  profile_photo_url?: string,
  status?: string
}
Response: { user: {...} }
```

#### Search Users
```
GET /api/users/search?q={query}&limit={limit}
Headers: { Authorization: Bearer {token} }
Response: {
  results: [{
    id, phone_number, display_name, username, profile_photo_url, status
  }]
}
```

#### Get User By ID
```
GET /api/users/{user_id}
Headers: { Authorization: Bearer {token} }
Response: { id, phone_number, display_name, username, profile_photo_url, status }
```

### Conversations

#### List Conversations
```
GET /api/conversations?limit={limit}&offset={offset}
Headers: { Authorization: Bearer {token} }
Response: {
  conversations: [{
    id, type, last_message, updated_at,
    members: [{ id, display_name, username, profile_photo_url }],
    unread_count: number
  }]
}
```

#### Create Conversation
```
POST /api/conversations
Headers: { Authorization: Bearer {token} }
Body: {
  participant_ids: [string] // For direct: 1 other user, for group: multiple
}
Response: { id, type, members, created_at }
```

#### Get Conversation
```
GET /api/conversations/{conversation_id}
Headers: { Authorization: Bearer {token} }
Response: {
  id, type, members,
  last_message, updated_at,
  unread_count: number
}
```

#### Get Messages
```
GET /api/conversations/{conversation_id}/messages?limit={limit}&offset={offset}
Headers: { Authorization: Bearer {token} }
Response: {
  messages: [{
    id, content, sender_id, state, created_at,
    sender: { id, display_name, profile_photo_url }
  }]
}
```

#### Mark Messages as Read
```
POST /api/conversations/{conversation_id}/mark-read
Headers: { Authorization: Bearer {token} }
Body: { up_to_message_id: string }
Response: { success: true }
```

### Messages (WebSocket)

Connect to WebSocket at `ws://localhost:3000/ws?token={access_token}`

#### Send Message
```json
{
  "type": "send_message",
  "conversation_id": "conv_123",
  "content": "Hello!"
}
```

Response:
```json
{
  "type": "message_sent",
  "message": {
    "id": "msg_456",
    "conversation_id": "conv_123",
    "content": "Hello!",
    "state": "sent",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

#### Message Delivered
```json
{
  "type": "message_delivered",
  "message_id": "msg_456",
  "delivered_at": "2024-01-15T10:30:01Z"
}
```

#### Message Received (broadcast to other members)
```json
{
  "type": "message_received",
  "message": {
    "id": "msg_456",
    "conversation_id": "conv_123",
    "sender_id": "user_789",
    "content": "Hello!",
    "state": "sent",
    "created_at": "2024-01-15T10:30:00Z"
  }
}
```

#### Mark Message as Read
```json
{
  "type": "mark_read",
  "message_id": "msg_456",
  "conversation_id": "conv_123"
}
```

Broadcast to other members:
```json
{
  "type": "message_read",
  "message_id": "msg_456",
  "read_by_user_id": "user_123",
  "read_at": "2024-01-15T10:30:02Z"
}
```

### Blocks

#### Block User
```
POST /api/users/{user_id}/block
Headers: { Authorization: Bearer {token} }
Response: { success: true }
```

#### Unblock User
```
DELETE /api/users/{user_id}/block
Headers: { Authorization: Bearer {token} }
Response: { success: true }
```

#### Get Blocked Users
```
GET /api/users/blocked
Headers: { Authorization: Bearer {token} }
Response: {
  blocked_users: [{
    id, display_name, username, profile_photo_url
  }]
}
```

## Error Responses

### 400 Bad Request
```json
{
  "error": {
    "code": "INVALID_INPUT",
    "message": "Phone number is required"
  }
}
```

### 401 Unauthorized
```json
{
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Access token is invalid or expired"
  }
}
```

### 403 Forbidden
```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "You do not have permission to access this resource"
  }
}
```

### 404 Not Found
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found"
  }
}
```

### 409 Conflict
```json
{
  "error": {
    "code": "ALREADY_EXISTS",
    "message": "User with this phone number already exists"
  }
}
```

### 500 Internal Server Error
```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred. Please try again later."
  }
}
```
