# 🔌 API Documentation

Complete API reference for ShareUpTime social media platform

## 📋 Table of Contents

1. [API Overview](#api-overview)
2. [Authentication](#authentication)
3. [User Management](#user-management)
4. [Posts & Content](#posts--content)
5. [Social Features](#social-features)
6. [Media Handling](#media-handling)
7. [Real-time Features](#real-time-features)
8. [Error Handling](#error-handling)

---

## 🌐 API Overview

### Base Configuration

**Base URL**: `http://localhost:3000` (Development)
**Production URL**: `https://api.shareuptime.com`

**API Version**: v1
**Content-Type**: `application/json`
**Authentication**: Bearer Token (JWT)

### Request/Response Format

All API endpoints follow RESTful conventions:

```http
GET /api/{resource}           # List resources
GET /api/{resource}/{id}      # Get specific resource
POST /api/{resource}          # Create new resource
PUT /api/{resource}/{id}      # Update resource (full)
PATCH /api/{resource}/{id}    # Update resource (partial)
DELETE /api/{resource}/{id}   # Delete resource
```

### Standard Response Format

```json
{
  "success": true,
  "data": {},
  "message": "Operation completed successfully",
  "timestamp": "2025-09-18T10:00:00.000Z",
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Email is required"
      }
    ]
  },
  "timestamp": "2025-09-18T10:00:00.000Z"
}
```

---

## 🔐 Authentication

### Registration

**Endpoint**: `POST /api/auth/register`

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "name": "John Doe",
  "username": "johndoe",
  "dateOfBirth": "1990-01-01"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe",
      "username": "johndoe",
      "avatar": null,
      "isVerified": false,
      "createdAt": "2025-09-18T10:00:00.000Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "refresh_token_here"
  }
}
```

### Login

**Endpoint**: `POST /api/auth/login`

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe",
      "username": "johndoe",
      "avatar": "https://cdn.shareuptime.com/avatars/user_123.jpg"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "24h"
  }
}
```

### Token Verification

**Endpoint**: `GET /api/auth/verify`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "valid": true,
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "name": "John Doe"
    },
    "expiresAt": "2025-09-19T10:00:00.000Z"
  }
}
```

### Password Reset

**Endpoint**: `POST /api/auth/forgot-password`

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Response**:
```json
{
  "success": true,
  "message": "Password reset email sent"
}
```

### Logout

**Endpoint**: `POST /api/auth/logout`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "message": "Successfully logged out"
}
```

---

## 👤 User Management

### Get User Profile

**Endpoint**: `GET /api/users/{userId}`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "username": "johndoe",
      "name": "John Doe",
      "email": "user@example.com",
      "bio": "Software developer passionate about technology",
      "avatar": "https://cdn.shareuptime.com/avatars/user_123.jpg",
      "coverPhoto": "https://cdn.shareuptime.com/covers/user_123.jpg",
      "location": "San Francisco, CA",
      "website": "https://johndoe.com",
      "joinedDate": "2025-01-01T00:00:00.000Z",
      "stats": {
        "followers": 1250,
        "following": 345,
        "posts": 87,
        "likes": 2340
      },
      "isFollowing": false,
      "isPrivate": false
    }
  }
}
```

### Update Profile

**Endpoint**: `PUT /api/users/profile`
**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "name": "John Doe Updated",
  "bio": "Updated bio text",
  "location": "New York, NY",
  "website": "https://newsite.com",
  "privacy": {
    "isPrivate": false,
    "showEmail": false,
    "showLocation": true
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123",
      "name": "John Doe Updated",
      "bio": "Updated bio text",
      "updatedAt": "2025-09-18T10:00:00.000Z"
    }
  }
}
```

### Search Users

**Endpoint**: `GET /api/users/search`
**Headers**: `Authorization: Bearer {token}`
**Query Parameters**: `?query=john&limit=20&page=1`

**Response**:
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "user_123",
        "username": "johndoe",
        "name": "John Doe",
        "avatar": "https://cdn.shareuptime.com/avatars/user_123.jpg",
        "isFollowing": false,
        "mutualFriends": 5
      }
    ]
  },
  "meta": {
    "total": 1,
    "page": 1,
    "limit": 20
  }
}
```

### Follow User

**Endpoint**: `POST /api/users/{userId}/follow`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "following": true,
    "followerId": "current_user_id",
    "followingId": "user_123"
  }
}
```

### Unfollow User

**Endpoint**: `DELETE /api/users/{userId}/follow`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "following": false
  }
}
```

---

## 📝 Posts & Content

### Create Post

**Endpoint**: `POST /api/posts`
**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "content": "This is my first post on ShareUpTime! 🎉",
  "type": "text",
  "visibility": "public",
  "mediaUrls": [
    "https://cdn.shareuptime.com/media/image_123.jpg"
  ],
  "tags": ["shareuptime", "social", "firstpost"],
  "location": {
    "name": "San Francisco, CA",
    "coordinates": {
      "lat": 37.7749,
      "lng": -122.4194
    }
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "post": {
      "id": "post_123",
      "content": "This is my first post on ShareUpTime! 🎉",
      "type": "text",
      "visibility": "public",
      "author": {
        "id": "user_123",
        "username": "johndoe",
        "name": "John Doe",
        "avatar": "https://cdn.shareuptime.com/avatars/user_123.jpg"
      },
      "mediaUrls": [
        "https://cdn.shareuptime.com/media/image_123.jpg"
      ],
      "stats": {
        "likes": 0,
        "comments": 0,
        "shares": 0,
        "views": 1
      },
      "createdAt": "2025-09-18T10:00:00.000Z",
      "updatedAt": "2025-09-18T10:00:00.000Z"
    }
  }
}
```

### Get Feed

**Endpoint**: `GET /api/feed`
**Headers**: `Authorization: Bearer {token}`
**Query Parameters**: `?page=1&limit=20&type=all`

**Response**:
```json
{
  "success": true,
  "data": {
    "posts": [
      {
        "id": "post_123",
        "content": "Amazing sunset today! 🌅",
        "type": "image",
        "author": {
          "id": "user_456",
          "username": "jane_photographer",
          "name": "Jane Smith",
          "avatar": "https://cdn.shareuptime.com/avatars/user_456.jpg"
        },
        "mediaUrls": [
          "https://cdn.shareuptime.com/media/sunset_123.jpg"
        ],
        "stats": {
          "likes": 42,
          "comments": 8,
          "shares": 3,
          "views": 156
        },
        "userInteraction": {
          "liked": false,
          "saved": true,
          "shared": false
        },
        "createdAt": "2025-09-18T09:30:00.000Z"
      }
    ]
  },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 250,
    "hasNext": true
  }
}
```

### Get Post Details

**Endpoint**: `GET /api/posts/{postId}`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "post": {
      "id": "post_123",
      "content": "Amazing sunset today! 🌅",
      "type": "image",
      "author": {
        "id": "user_456",
        "username": "jane_photographer",
        "name": "Jane Smith",
        "avatar": "https://cdn.shareuptime.com/avatars/user_456.jpg"
      },
      "mediaUrls": [
        "https://cdn.shareuptime.com/media/sunset_123.jpg"
      ],
      "stats": {
        "likes": 42,
        "comments": 8,
        "shares": 3,
        "views": 156
      },
      "comments": [
        {
          "id": "comment_123",
          "content": "Beautiful photo! 📸",
          "author": {
            "id": "user_789",
            "username": "photographer_pro",
            "name": "Alex Wilson"
          },
          "createdAt": "2025-09-18T09:45:00.000Z"
        }
      ],
      "createdAt": "2025-09-18T09:30:00.000Z"
    }
  }
}
```

### Like Post

**Endpoint**: `POST /api/posts/{postId}/like`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "liked": true,
    "likeCount": 43
  }
}
```

### Unlike Post

**Endpoint**: `DELETE /api/posts/{postId}/like`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "liked": false,
    "likeCount": 42
  }
}
```

### Add Comment

**Endpoint**: `POST /api/posts/{postId}/comments`
**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "content": "Great photo! Love the colors 🎨",
  "parentCommentId": null
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "comment": {
      "id": "comment_456",
      "content": "Great photo! Love the colors 🎨",
      "author": {
        "id": "user_123",
        "username": "johndoe",
        "name": "John Doe",
        "avatar": "https://cdn.shareuptime.com/avatars/user_123.jpg"
      },
      "parentCommentId": null,
      "likes": 0,
      "createdAt": "2025-09-18T10:15:00.000Z"
    }
  }
}
```

### Share Post

**Endpoint**: `POST /api/posts/{postId}/share`
**Headers**: `Authorization: Bearer {token}`

**Request Body**:
```json
{
  "content": "Check out this amazing sunset! 🌅",
  "visibility": "public"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "shareId": "share_123",
    "originalPost": "post_123",
    "shareCount": 4
  }
}
```

---

## 👥 Social Features

### Get Friends List

**Endpoint**: `GET /api/users/{userId}/friends`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "friends": [
      {
        "id": "user_456",
        "username": "jane_photographer",
        "name": "Jane Smith",
        "avatar": "https://cdn.shareuptime.com/avatars/user_456.jpg",
        "mutualFriends": 12,
        "friendshipDate": "2024-12-01T00:00:00.000Z"
      }
    ]
  }
}
```

### Send Friend Request

**Endpoint**: `POST /api/users/{userId}/friend-request`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "requestId": "request_123",
    "status": "pending",
    "sentAt": "2025-09-18T10:00:00.000Z"
  }
}
```

### Accept Friend Request

**Endpoint**: `POST /api/friend-requests/{requestId}/accept`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "friendship": {
      "id": "friendship_123",
      "user1": "user_123",
      "user2": "user_456",
      "createdAt": "2025-09-18T10:00:00.000Z"
    }
  }
}
```

### Get Notifications

**Endpoint**: `GET /api/notifications`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "notification_123",
        "type": "like",
        "title": "New Like",
        "message": "Jane Smith liked your post",
        "data": {
          "postId": "post_123",
          "userId": "user_456"
        },
        "isRead": false,
        "createdAt": "2025-09-18T09:45:00.000Z"
      },
      {
        "id": "notification_124",
        "type": "comment",
        "title": "New Comment",
        "message": "Alex Wilson commented on your post",
        "data": {
          "postId": "post_123",
          "commentId": "comment_456",
          "userId": "user_789"
        },
        "isRead": false,
        "createdAt": "2025-09-18T09:30:00.000Z"
      }
    ]
  }
}
```

---

## 📁 Media Handling

### Upload Media

**Endpoint**: `POST /api/media/upload`
**Headers**: 
- `Authorization: Bearer {token}`
- `Content-Type: multipart/form-data`

**Request Body** (FormData):
```
file: [binary file data]
type: "image" | "video" | "document"
purpose: "post" | "avatar" | "cover" | "story"
```

**Response**:
```json
{
  "success": true,
  "data": {
    "media": {
      "id": "media_123",
      "url": "https://cdn.shareuptime.com/media/image_123.jpg",
      "thumbnailUrl": "https://cdn.shareuptime.com/media/thumbs/image_123_thumb.jpg",
      "type": "image",
      "size": 2048576,
      "dimensions": {
        "width": 1920,
        "height": 1080
      },
      "metadata": {
        "filename": "sunset.jpg",
        "mimeType": "image/jpeg"
      },
      "uploadedAt": "2025-09-18T10:00:00.000Z"
    }
  }
}
```

### Get Media

**Endpoint**: `GET /api/media/{mediaId}`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "data": {
    "media": {
      "id": "media_123",
      "url": "https://cdn.shareuptime.com/media/image_123.jpg",
      "type": "image",
      "uploader": {
        "id": "user_123",
        "username": "johndoe"
      },
      "metadata": {
        "size": 2048576,
        "dimensions": {
          "width": 1920,
          "height": 1080
        }
      }
    }
  }
}
```

### Delete Media

**Endpoint**: `DELETE /api/media/{mediaId}`
**Headers**: `Authorization: Bearer {token}`

**Response**:
```json
{
  "success": true,
  "message": "Media deleted successfully"
}
```

---

## ⚡ Real-time Features

### WebSocket Connection

**URL**: `ws://localhost:3007/ws`
**Headers**: `Authorization: Bearer {token}`

### Message Types

#### Connection Event
```json
{
  "type": "connected",
  "data": {
    "userId": "user_123",
    "connectionId": "conn_456"
  }
}
```

#### New Message
```json
{
  "type": "message",
  "data": {
    "id": "message_123",
    "conversationId": "conv_456",
    "sender": {
      "id": "user_789",
      "name": "Alex Wilson"
    },
    "content": "Hello! How are you?",
    "timestamp": "2025-09-18T10:00:00.000Z"
  }
}
```

#### Live Notification
```json
{
  "type": "notification",
  "data": {
    "id": "notification_123",
    "type": "like",
    "message": "Someone liked your post",
    "timestamp": "2025-09-18T10:00:00.000Z"
  }
}
```

#### Typing Indicator
```json
{
  "type": "typing",
  "data": {
    "conversationId": "conv_456",
    "userId": "user_789",
    "isTyping": true
  }
}
```

### Send Message via WebSocket

```json
{
  "type": "send_message",
  "data": {
    "conversationId": "conv_456",
    "content": "Hello there!",
    "messageType": "text"
  }
}
```

---

## ❌ Error Handling

### HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Resource already exists |
| 422 | Unprocessable Entity | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |

### Error Codes

#### Authentication Errors
- `AUTH_TOKEN_INVALID` - Invalid or expired token
- `AUTH_TOKEN_MISSING` - Authorization header missing
- `AUTH_CREDENTIALS_INVALID` - Invalid login credentials
- `AUTH_USER_NOT_FOUND` - User account not found
- `AUTH_ACCOUNT_LOCKED` - Account temporarily locked

#### Validation Errors
- `VALIDATION_ERROR` - General validation failure
- `VALIDATION_EMAIL_INVALID` - Invalid email format
- `VALIDATION_PASSWORD_WEAK` - Password doesn't meet requirements
- `VALIDATION_USERNAME_TAKEN` - Username already exists
- `VALIDATION_REQUIRED_FIELD` - Required field missing

#### Resource Errors
- `RESOURCE_NOT_FOUND` - Requested resource doesn't exist
- `RESOURCE_ACCESS_DENIED` - Insufficient permissions for resource
- `RESOURCE_ALREADY_EXISTS` - Resource with same identifier exists
- `RESOURCE_LIMIT_EXCEEDED` - User has reached resource limit

#### Media Errors
- `MEDIA_FILE_TOO_LARGE` - File size exceeds limit
- `MEDIA_INVALID_FORMAT` - Unsupported file format
- `MEDIA_UPLOAD_FAILED` - File upload failed
- `MEDIA_PROCESSING_ERROR` - Error processing media file

### Example Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request contains invalid data",
    "details": [
      {
        "field": "email",
        "code": "VALIDATION_EMAIL_INVALID",
        "message": "Please provide a valid email address"
      },
      {
        "field": "password",
        "code": "VALIDATION_PASSWORD_WEAK",
        "message": "Password must be at least 8 characters long"
      }
    ]
  },
  "timestamp": "2025-09-18T10:00:00.000Z",
  "requestId": "req_123456"
}
```

---

## 🔧 Rate Limiting

### Rate Limit Headers

Every API response includes rate limiting information:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642781400
```

### Rate Limits by Endpoint Type

| Endpoint Type | Limit | Window |
|---------------|--------|--------|
| Authentication | 5 requests | 15 minutes |
| Post Creation | 10 requests | 1 hour |
| Media Upload | 20 requests | 1 hour |
| General API | 1000 requests | 1 hour |
| WebSocket | 100 messages | 1 minute |

### Rate Limit Exceeded Response

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "retryAfter": 900
  }
}
```

---

## 🧪 Testing the API

### Using cURL

#### Login
```bash
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password123"}'
```

#### Get Feed
```bash
curl -X GET http://localhost:3000/api/feed \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json"
```

#### Create Post
```bash
curl -X POST http://localhost:3000/api/posts \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{"content": "Hello ShareUpTime!", "visibility": "public"}'
```

### Using JavaScript (Fetch)

```javascript
// Login
const response = await fetch('http://localhost:3000/api/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'test@example.com',
    password: 'password123'
  })
});

const { data } = await response.json();
const token = data.token;

// Use token for authenticated requests
const feedResponse = await fetch('http://localhost:3000/api/feed', {
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json'
  }
});

const feedData = await feedResponse.json();
```

This comprehensive API documentation provides all the information needed to integrate with the ShareUpTime platform backend services.