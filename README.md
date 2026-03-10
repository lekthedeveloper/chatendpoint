# Chat Backend API Specification

This document provides exact JSON structures and behaviors for the Chat API. **Please follow these structures exactly** to ensure the mobile app can parse the data without errors.

## General Requirements
- **JSON Casing**: Use `camelCase` for all keys.
- **Date Format**: ISO 8601 (e.g., `2024-03-10T15:30:00Z`).
- **Response Wrapper**: All endpoints must wrap their data in this standard object:
  ```json
  {
    "success": true,
    "message": "Processed successfully",
    "data": { ... see specific endpoints below ... },
    "errors": []
  }
  ```

---

## 1. Get Conversations List
This returns the list of active chats (inboxes) for a user or provider.

### Endpoints
- `GET /api/messages/customers/{customerId}/conversations`
- `GET /api/messages/providers/{providerId}/conversations`

### Full Response Structure
```json
{
  "success": true,
  "message": null,
  "data": {
    "records": [
      {
        "providerId": 123,
        "customerId": 456,
        "providerName": "John's Plumbing",
        "customerName": "Alice Smith",
        "lastMessage": "See you at 4 PM",
        "lastMessageTime": "2024-03-10T12:00:00Z",
        "unreadCount": 3
      },
      {
        "providerId": 124,
        "customerId": 456,
        "providerName": "Quick Fix Electrical",
        "customerName": "Alice Smith",
        "lastMessage": "How much per hour?",
        "lastMessageTime": "2024-03-10T10:30:00Z",
        "unreadCount": 0
      }
    ],
    "totalCount": 58,
    "totalPages": 3,
    "pageNumber": 1,
    "pageSize": 20,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "errors": []
}
```

---

## 2. Get Chat History
Returns the actual messages between a customer and a provider.

### Endpoint
- `GET /api/messages/chat-history?providerId=123&customerId=456&pageNumber=1&pageSize=50`

### Full Response Structure
```json
{
  "success": true,
  "message": null,
  "data": {
    "messages": [
      {
        "id": 1001,
        "senderId": 456,
        "recipientId": 123,
        "message": "Hello, is the service available?",
        "timestamp": "2024-03-10T11:45:00Z",
        "isRead": true,
        "isFromProvider": false
      },
      {
        "id": 1002,
        "senderId": 123,
        "recipientId": 456,
        "message": "Yes, we are available!",
        "timestamp": "2024-03-10T11:46:00Z",
        "isRead": false,
        "isFromProvider": true
      }
    ],
    "totalCount": 150,
    "totalPages": 3,
    "pageNumber": 1,
    "pageSize": 50,
    "hasNextPage": true,
    "hasPreviousPage": false
  },
  "errors": []
}
```

---

## 3. Send New Message
### Endpoint
- `POST /api/messages`

### Request Body
```json
{
  "providerId": 123,
  "customerId": 456,
  "message": "I will be there in 10 minutes",
  "isFromProvider": true
}
```

### Response Structure (Should return the created message)
```json
{
  "success": true,
  "message": "Message sent",
  "data": {
    "id": 1003,
    "senderId": 123,
    "recipientId": 456,
    "message": "I will be there in 10 minutes",
    "timestamp": "2024-03-10T11:55:00Z",
    "isRead": false,
    "isFromProvider": true
  },
  "errors": []
}
```

---

## 4. Unread Counts & Status

### A. Total Global Unread Count
Used to show the badge icon on the app's bottom navigation bar.
- `GET /api/messages/unread-count?forProvider=false`

**Full Response Structure:**
```json
{
  "success": true,
  "message": null,
  "data": {
    "totalUnread": 5
  },
  "errors": []
}
```

### B. Batch Mark Conversation as Read
Called when the user opens a chat room. The backend should mark all messages from the *other* party in this specific conversation as read.

- `PATCH /api/messages/mark-as-read`

**Request Body:**
```json
{
  "providerId": 123,
  "customerId": 456,
  "markedBy": "customer"
}
```

**Full Response Structure:**
```json
{
  "success": true,
  "message": "Conversation marked as read",
  "data": {
    "providerId": 123,
    "customerId": 456,
    "updatedCount": 3
  },
  "errors": []
}
```
