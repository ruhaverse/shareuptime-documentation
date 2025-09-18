# 🏗️ Backend Architecture Documentation

**Complete guide to ShareUpTime's microservices backend architecture**

## 📋 Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Service Details](#service-details)
3. [Communication Patterns](#communication-patterns)
4. [Database Architecture](#database-architecture)
5. [Security Implementation](#security-implementation)
6. [Scalability Considerations](#scalability-considerations)

---

## 🔍 Architecture Overview

ShareUpTime uses a **microservices architecture** with 7 independent services, each handling specific business domains. This design ensures scalability, maintainability, and fault tolerance.

### High-Level Architecture Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                        Client Layer                          │
├─────────────────────────┬───────────────────────────────────┤
│     Web Frontend        │        Mobile App                 │
│    (Next.js - :3007)    │    (React Native)                 │
└─────────────────────────┴───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway (:3000)                      │
│            • Request Routing                                │
│            • Authentication & Authorization                 │
│            • Rate Limiting                                  │
│            • Request/Response Transformation               │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Auth Service │    │User Service │    │Post Service │
│   (:3001)   │    │   (:3002)   │    │   (:3003)   │
└─────────────┘    └─────────────┘    └─────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Feed Service │    │Media Service│    │Notification │
│   (:3004)   │    │   (:3005)   │    │Service(:3006│
└─────────────┘    └─────────────┘    └─────────────┘
        │                     │                     │
        ▼                     ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                             │
│  PostgreSQL  │  MongoDB  │  Redis  │  Neo4j  │  File System│
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Service Details

### 1. API Gateway (Port 3000)
**Purpose**: Central entry point for all client requests

**Key Responsibilities**:
- Route requests to appropriate microservices
- Handle authentication and authorization
- Implement rate limiting and throttling
- Request/response transformation and validation
- Load balancing and health checks

**Key Files**:
```
api-gateway/
├── index.js                 # Main server setup
├── middleware/
│   ├── rateLimiting.js     # Rate limiting configuration
│   └── validation.js       # Request validation
├── package.json            # Dependencies
└── swagger.config.js       # API documentation
```

**Environment Variables**:
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=your-jwt-secret-key
AUTH_SERVICE_URL=http://localhost:3001
USER_SERVICE_URL=http://localhost:3002
POST_SERVICE_URL=http://localhost:3003
FEED_SERVICE_URL=http://localhost:3004
MEDIA_SERVICE_URL=http://localhost:3005
NOTIFICATION_SERVICE_URL=http://localhost:3006
```

**Key Endpoints**:
- `GET /health` - Health check
- `POST /api/auth/*` - Proxy to Auth Service
- `GET /api/users/*` - Proxy to User Service
- `POST /api/posts/*` - Proxy to Post Service
- `GET /api/feed/*` - Proxy to Feed Service

### 2. Auth Service (Port 3001)
**Purpose**: Handle user authentication and authorization

**Key Responsibilities**:
- User registration and login
- JWT token generation and validation
- Password management and reset
- Session management
- OAuth integration (future)

**Key Files**:
```
auth-service/
├── index.js              # Main authentication server
├── package.json          # Auth-specific dependencies
├── swagger.config.js     # Auth API documentation
└── README.md            # Service-specific documentation
```

**Database Schema**:
```sql
-- Users table (PostgreSQL)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Sessions table (Redis)
-- Key: session:user_id
-- Value: { token, expires_at, user_data }
```

**Key Endpoints**:
- `POST /auth/register` - User registration
- `POST /auth/login` - User login
- `POST /auth/logout` - User logout
- `GET /auth/verify` - Token verification
- `POST /auth/forgot-password` - Password reset

### 3. User Service (Port 3002)
**Purpose**: Manage user profiles and social connections

**Key Responsibilities**:
- User profile management
- Friend connections and followers
- User preferences and settings
- Social graph relationships
- User search and discovery

**Database Schema**:
```sql
-- User profiles (PostgreSQL)
CREATE TABLE user_profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    username VARCHAR(50) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    bio TEXT,
    avatar_url VARCHAR(500),
    location VARCHAR(100),
    website VARCHAR(200),
    birth_date DATE,
    privacy_settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW()
);

-- Friendships (Neo4j for social graph)
-- Relationships: (:User)-[:FOLLOWS]->(:User)
-- Relationships: (:User)-[:FRIENDS_WITH]->(:User)
```

**Key Endpoints**:
- `GET /users/profile/:id` - Get user profile
- `PUT /users/profile` - Update profile
- `POST /users/follow/:id` - Follow user
- `DELETE /users/follow/:id` - Unfollow user
- `GET /users/search` - Search users

### 4. Post Service (Port 3003)
**Purpose**: Handle content creation and management

**Key Responsibilities**:
- Post creation, editing, deletion
- Comment management
- Like and reaction systems
- Content moderation
- Post visibility and privacy

**Key Files**:
```
post-service/
├── index.js              # Main post service server
├── business.js           # Business logic layer
├── persistence.js        # Data persistence layer
├── test-business.js      # Business logic tests
└── package.json          # Dependencies
```

**Database Schema**:
```sql
-- Posts (PostgreSQL)
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    media_urls JSONB DEFAULT '[]',
    post_type VARCHAR(20) DEFAULT 'text', -- text, image, video, story, reel
    visibility VARCHAR(20) DEFAULT 'public', -- public, friends, private
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Comments (PostgreSQL)
CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    post_id INTEGER REFERENCES posts(id),
    user_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    parent_comment_id INTEGER REFERENCES comments(id),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Likes (Redis for performance)
-- Key: likes:post:{post_id}
-- Value: Set of user_ids
```

**Key Endpoints**:
- `POST /posts` - Create new post
- `GET /posts/:id` - Get specific post
- `PUT /posts/:id` - Update post
- `DELETE /posts/:id` - Delete post
- `POST /posts/:id/like` - Like/unlike post
- `POST /posts/:id/comments` - Add comment

### 5. Feed Service (Port 3004)
**Purpose**: Generate personalized content feeds

**Key Responsibilities**:
- Timeline generation algorithms
- Content ranking and filtering
- Feed caching and optimization
- Trending content discovery
- Personalized recommendations

**Algorithm Overview**:
```javascript
// Feed Generation Algorithm
function generateFeed(userId, page = 1, limit = 20) {
  const feedItems = [
    ...getFollowingPosts(userId),    // Posts from followed users
    ...getTrendingContent(),         // Trending posts
    ...getRecommendedContent(userId) // ML-based recommendations
  ];
  
  return rankAndSort(feedItems)
    .slice((page - 1) * limit, page * limit);
}
```

**Caching Strategy**:
```
Redis Cache Layers:
├── feed:user:{user_id}           # User's feed cache (15min TTL)
├── trending:global              # Global trending content (30min TTL)
├── recommendations:{user_id}    # User recommendations (1hr TTL)
└── post:metadata:{post_id}      # Post metadata cache (1hr TTL)
```

**Key Endpoints**:
- `GET /feed/:userId` - Get user's personalized feed
- `GET /feed/trending` - Get trending content
- `GET /feed/explore/:userId` - Get explore page content
- `POST /feed/refresh/:userId` - Refresh user's feed

### 6. Media Service (Port 3005)
**Purpose**: Handle file uploads and media processing

**Key Responsibilities**:
- Image and video uploads
- Media compression and optimization
- Thumbnail generation
- CDN integration
- Media metadata extraction

**File Processing Pipeline**:
```
Upload → Validation → Virus Scan → Compression → Storage → CDN → Database
```

**Storage Structure**:
```
uploads/
├── images/
│   ├── original/           # Original uploaded images
│   ├── thumbnails/         # Generated thumbnails
│   └── compressed/         # Compressed versions
├── videos/
│   ├── original/           # Original uploaded videos
│   ├── thumbnails/         # Video thumbnails
│   └── processed/          # Processed videos
└── documents/              # Document uploads
```

**Key Endpoints**:
- `POST /media/upload` - Upload media files
- `GET /media/:id` - Get media file
- `DELETE /media/:id` - Delete media file
- `POST /media/process/:id` - Process uploaded media
- `GET /media/metadata/:id` - Get media metadata

### 7. Notification Service (Port 3006)
**Purpose**: Handle real-time notifications and messaging

**Key Responsibilities**:
- Real-time WebSocket connections
- Push notification delivery
- Email notifications
- In-app notification management
- Notification preferences

**Real-time Architecture**:
```
WebSocket Server (Port 3007)
├── Connection Manager      # Handle client connections
├── Room Manager           # Group users by interests/friends
├── Message Queue          # Queue notifications for delivery
└── Delivery Engine        # Send notifications to connected clients
```

**Notification Types**:
```javascript
const NotificationTypes = {
  LIKE: 'post_liked',
  COMMENT: 'post_commented',
  FOLLOW: 'user_followed',
  MENTION: 'user_mentioned',
  FRIEND_REQUEST: 'friend_request',
  MESSAGE: 'direct_message'
};
```

**Key Endpoints**:
- `GET /notifications/:userId` - Get user notifications
- `POST /notifications/send` - Send notification
- `PUT /notifications/:id/read` - Mark as read
- `WebSocket /ws` - Real-time connection

---

## 🔄 Communication Patterns

### 1. Synchronous Communication
**Used for**: Real-time operations, immediate responses
```javascript
// API Gateway → Service Communication
const response = await fetch(`${USER_SERVICE_URL}/users/${userId}`);
```

### 2. Asynchronous Communication
**Used for**: Background processing, event-driven operations
```javascript
// Event Publishing (Redis Pub/Sub)
redis.publish('user.created', JSON.stringify(userEvent));

// Event Listening
redis.subscribe('post.created', (message) => {
  const postEvent = JSON.parse(message);
  generateNotifications(postEvent);
});
```

### 3. Circuit Breaker Pattern
```javascript
const circuitBreaker = new CircuitBreaker(callService, {
  timeout: 3000,
  errorThresholdPercentage: 50,
  resetTimeout: 30000
});
```

---

## 🗄️ Database Architecture

### Database per Service Principle
Each service manages its own database to ensure loose coupling:

```
├── Auth Service → PostgreSQL (auth_db)
│   └── Tables: users, sessions, tokens
├── User Service → PostgreSQL (user_db) + Neo4j (social_graph)
│   └── Tables: profiles, settings, connections
├── Post Service → PostgreSQL (content_db) + MongoDB (media_metadata)
│   └── Tables: posts, comments, likes
├── Feed Service → Redis (cache_db)
│   └── Cached: feeds, trending, recommendations
├── Media Service → MongoDB (media_db) + File System
│   └── Collections: media_files, metadata
└── Notification Service → Redis (notifications_db)
    └── Queues: notifications, real-time events
```

### Data Consistency Strategy

**Eventual Consistency**: Accept temporary inconsistency for better performance
**Saga Pattern**: Coordinate transactions across services
**Event Sourcing**: Store events for audit and replay capabilities

---

## 🔒 Security Implementation

### 1. Authentication Flow
```
1. User Login → Auth Service generates JWT
2. JWT contains: user_id, permissions, expiry
3. API Gateway validates JWT on each request
4. Services trust API Gateway authentication
```

### 2. Authorization Levels
```javascript
const PermissionLevels = {
  PUBLIC: 0,     // Anyone can access
  USER: 1,       // Authenticated users only
  FRIEND: 2,     // Friends only
  ADMIN: 3       // Admin users only
};
```

### 3. Security Middleware Stack
```
Request → Rate Limiter → Auth Validator → Permission Checker → Service
```

### 4. Data Protection
- **Encryption**: All passwords hashed with bcrypt
- **HTTPS**: All communication encrypted in transit
- **Input Validation**: All inputs validated and sanitized
- **SQL Injection Protection**: Parameterized queries only

---

## 📈 Scalability Considerations

### Horizontal Scaling
```yaml
# Docker Compose scaling example
services:
  user-service:
    image: shareuptime/user-service
    deploy:
      replicas: 3
    ports:
      - "3002-3004:3002"
```

### Caching Strategy
```
├── Level 1: Application Cache (In-memory)
├── Level 2: Redis Cache (Distributed)
├── Level 3: CDN Cache (Global)
└── Level 4: Database Query Cache
```

### Load Balancing
```
Client → Load Balancer → API Gateway → Service Discovery → Service Instances
```

### Database Scaling
- **Read Replicas**: For read-heavy operations
- **Sharding**: Partition data across multiple databases
- **Connection Pooling**: Optimize database connections

---

## 📊 Monitoring and Health Checks

### Health Check Endpoints
Each service exposes a `/health` endpoint:
```javascript
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.npm_package_version,
    uptime: process.uptime()
  });
});
```

### Service Dependencies
```
API Gateway depends on: All services
Feed Service depends on: User Service, Post Service
Notification Service depends on: User Service, Post Service
Other services: Independent
```

---

## 🚀 Performance Optimization

### 1. Connection Pooling
```javascript
const pool = new Pool({
  host: 'localhost',
  database: 'shareuptime',
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

### 2. Query Optimization
```sql
-- Add indexes for frequent queries
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);
CREATE INDEX idx_comments_post_created ON comments(post_id, created_at);
```

### 3. Caching Strategies
```javascript
// Multi-level caching
const cachedData = 
  memory.get(key) || 
  await redis.get(key) || 
  await database.query(sql);
```

---

## 🔧 Development Tools

### API Documentation
- **Swagger UI**: Available at `http://localhost:3000/docs`
- **Postman Collection**: Available in `/docs/postman/`

### Testing
```bash
# Unit Tests
npm test

# Integration Tests
npm run test:integration

# Load Tests
npm run test:load
```

### Debugging
```javascript
// Enable debug logs
DEBUG=shareuptime:* npm start
```

---

This architecture provides a solid foundation for a scalable social media platform while maintaining flexibility for future enhancements and modifications.