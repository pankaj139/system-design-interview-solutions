# Social Media Platform System Design (Instagram/Twitter)

**Document Purpose:** This document provides a comprehensive system design for a large-scale social media platform supporting photo/video sharing with social features. It covers architecture decisions, scalability considerations, and implementation details for handling 500M daily active users with high availability and performance requirements.

**Last Updated:** October 2, 2025

---

## Table of Contents

1. [Requirements & Clarification](#requirements--clarification)
2. [Back-of-the-Envelope Calculations](#back-of-the-envelope-calculations)
3. [High-Level Design](#high-level-design)
4. [Database Design](#database-design)
5. [API Design](#api-design)
6. [Deep-Dive Components](#deep-dive-components)
7. [Trade-Offs Analysis](#trade-offs-analysis)
8. [Caching Strategy](#caching-strategy)
9. [Bottlenecks & Improvements](#bottlenecks--improvements)
10. [Security Considerations](#security-considerations)
11. [Future Enhancements](#future-enhancements)

---

## Requirements & Clarification

### User Stories

- **As a user**, I want to create and share photos/videos so that I can express myself and connect with others
- **As a user**, I want to follow other users so that I can see their content in my feed
- **As a user**, I want to like, comment, and share posts so that I can engage with content
- **As a user**, I want to view a personalized feed so that I see relevant content quickly
- **As a content creator**, I want my posts to reach my followers instantly so that I can maintain engagement
- **As a celebrity user**, I want to efficiently broadcast to millions of followers without system degradation

### Functional Requirements

**Core Features (MVP):**

1. **User Management**
   - User registration and authentication
   - Profile creation and management
   - Follow/unfollow functionality

2. **Content Creation**
   - Upload photos and videos
   - Create stories (24-hour expiration)
   - Add captions, hashtags, and mentions
   - Support multiple media formats

3. **Feed Generation**
   - Home timeline (posts from followed users)
   - Explore feed (trending/recommended content)
   - User profile feed

4. **Engagement Features**
   - Like posts
   - Comment on posts
   - Share posts
   - Hashtag browsing and trending topics

5. **Media Processing**
   - Image resizing and optimization
   - Video transcoding
   - Thumbnail generation
   - CDN distribution

### Non-Functional Requirements

1. **Availability:** 99.9% uptime for core features (8.76 hours downtime/year)
2. **Performance:**
   - Feed loads in <500ms (p95)
   - Media upload acknowledgment in <2s
   - Real-time updates for likes/comments (<1s delay)
3. **Scalability:**
   - Support 500M daily active users
   - Handle 200M posts per day
   - Serve 10B feed impressions per day
   - Support celebrity accounts with 100M+ followers
4. **Consistency:**
   - Eventual consistency for feeds (acceptable)
   - Strong consistency for financial operations (if any)
   - Immediate consistency for user actions (like/unlike)
5. **Durability:**
   - Zero data loss for uploaded media
   - Reliable delivery of notifications

### Clarifying Questions & Assumptions

**Scale Questions:**

- **Q:** What's the geographic distribution?
  - **A:** Global platform with concentration in US, Europe, India, and Southeast Asia
- **Q:** What's the read-to-write ratio?
  - **A:** Very read-heavy (100:1 ratio - users browse more than post)
- **Q:** Average media file sizes?
  - **A:** Photos ~2MB, Videos ~50MB, Stories ~10MB

**Feature Scope:**

- **Q:** Do we need direct messaging?
  - **A:** Out of scope for MVP
- **Q:** Do we need video calls or live streaming?
  - **A:** Out of scope for MVP
- **Q:** Support for ads?
  - **A:** Architecture should accommodate future ad insertion

**Assumptions:**

1. Average user follows 200 accounts
2. Celebrity accounts: 0.1% of users have >1M followers
3. 40% of DAU post content (200M posts from 500M DAU)
4. Peak traffic is 3x average (US evening hours)
5. Users scroll through 50 posts per session on average
6. Stories have 24-hour TTL and are viewed more frequently than posts
7. Average post has 5 hashtags
8. Mobile-first platform (80% mobile, 20% web)

---

## Back-of-the-Envelope Calculations

### Traffic Estimates

**Daily Active Users (DAU):**

```text
DAU: 500,000,000 users
Monthly Active Users (MAU): ~1.5B users (assuming 3x DAU)
```

**Post Creation:**

```text
Posts per day: 200,000,000
Posts per second (avg): 200M / 86,400 = ~2,315 posts/sec
Posts per second (peak): 2,315 * 3 = ~7,000 posts/sec

Breakdown:
- Photos: 60% = 120M/day = 1,400/sec (peak: 4,200/sec)
- Videos: 30% = 60M/day = 700/sec (peak: 2,100/sec)
- Stories: 10% = 20M/day = 230/sec (peak: 700/sec)
```

**Feed Generation:**

```text
Feed impressions per day: 10,000,000,000 (10B)
Feed requests per second (avg): 10B / 86,400 = ~115,740 req/sec
Feed requests per second (peak): 115,740 * 3 = ~347,000 req/sec

Average posts viewed per user per day: 10B / 500M = 20 posts
Sessions per user per day: 20 / 50 posts per session = ~5 sessions
```

**Engagement Actions:**

```text
Assuming:
- 50% of viewed posts get liked
- 10% of viewed posts get commented
- 5% of viewed posts get shared

Likes per day: 10B * 0.5 = 5B likes = 57,870 likes/sec (peak: 173,600/sec)
Comments per day: 10B * 0.1 = 1B comments = 11,574/sec (peak: 34,700/sec)
Shares per day: 10B * 0.05 = 500M shares = 5,787/sec (peak: 17,360/sec)
```

**Follow/Unfollow Actions:**

```text
Assuming 5% of DAU follow/unfollow per day:
Follow actions per day: 500M * 0.05 = 25M
Follow actions per second: 25M / 86,400 = ~290/sec (peak: 870/sec)
```

### Storage Estimates

**User Data:**

```text
Total users: 1.5B (MAU)
User record size: ~1KB (profile info, metadata)
User data storage: 1.5B * 1KB = 1.5TB
```

**Post Metadata:**

```text
Posts per day: 200M
Days of data to keep: Indefinite (assume 5 years = 1,825 days)
Total posts in 5 years: 200M * 1,825 = 365B posts

Post metadata size: ~2KB (caption, hashtags, metadata, engagement counts)
Post metadata storage (5 years): 365B * 2KB = 730TB = ~730TB
```

**Media Storage:**

**Photos:**

```text
Photos per day: 120M
Photo size (original): 2MB
Photo size (thumbnails): 100KB + 300KB + 500KB = 900KB
Total per photo: 2MB + 900KB = 2.9MB

Daily storage: 120M * 2.9MB = 348TB/day
Yearly storage: 348TB * 365 = 127PB/year
5-year storage: 127PB * 5 = 635PB
```

**Videos:**

```text
Videos per day: 60M
Video size (original): 50MB
Video size (processed formats): 10MB + 20MB + 30MB = 60MB
Thumbnail: 500KB
Total per video: 50MB + 60MB + 0.5MB = 110.5MB

Daily storage: 60M * 110.5MB = 6,630TB = 6.6PB/day
Yearly storage: 6.6PB * 365 = 2,409PB/year
5-year storage: 2,409PB * 5 = 12,045PB = ~12EB
```

**Stories:**

```text
Stories per day: 20M
Story size (avg): 10MB with processing = 25MB total
Stories are deleted after 24 hours

Active storage: 20M * 25MB = 500TB (rolling 24-hour window)
```

**Total Storage (5 years):**

```text
User data: 1.5TB
Post metadata: 730TB
Photos: 635PB
Videos: 12EB
Stories: 500TB (active)

Total: ~12.6EB (primarily video content)
```

### Bandwidth Estimates

**Upload Bandwidth:**

```text
Media uploads per second (avg): 2,315 posts/sec
Average media size: (120M * 2MB + 60M * 50MB + 20M * 10MB) / 200M
                  = (240TB + 3,000TB + 200TB) / 200M
                  = 3,440TB / 200M = ~17.2MB per post

Upload bandwidth (avg): 2,315 * 17.2MB = ~40GB/sec = 320Gbps
Upload bandwidth (peak): 320Gbps * 3 = 960Gbps = ~1Tbps
```

**Download Bandwidth (Feed Serving):**

```text
Feed requests per second (avg): 115,740/sec
Average posts per feed load: 20 posts
Media per post (thumbnail): 300KB

Download per request: 20 * 300KB = 6MB
Download bandwidth (avg): 115,740 * 6MB = 694TB/sec = ~5.5Pbps
Download bandwidth (peak): 5.5Pbps * 3 = 16.5Pbps

Note: CDN will cache majority of this traffic (~95% cache hit rate)
Actual origin bandwidth: 16.5Pbps * 0.05 = ~825Tbps at peak
```

### Resource Estimates

**Database Sizing:**

```text
Primary databases:
- User DB: 1.5TB + indexes (~2x) = 3TB
- Post metadata DB: 730TB + indexes = 1.5PB (sharded)
- Social graph DB: 500M users * 200 follows * 16 bytes = 1.6TB
```

**Cache Sizing:**

```text
Feed cache (hot data):
- Active users at peak: 500M * 0.2 (concurrent) = 100M
- Cache per user: 50 posts * 2KB metadata = 100KB
- Total cache: 100M * 100KB = 10TB

Hot content cache:
- Top 10% of posts account for 90% of views
- Recent posts (last 24h): 200M posts
- Top posts to cache: 20M posts
- Cache size: 20M * 300KB (thumbnail) = 6TB
```

**Worker Pool Sizing:**

```text
Media processing workers:
- Peak uploads: 7,000/sec
- Processing time per media: ~10 seconds
- Concurrent workers needed: 7,000 * 10 = 70,000 workers
- With buffering and queuing: ~100,000 workers
```

---

## High-Level Design

### System Architecture Diagram

```mermaid
graph TB
    subgraph Client Layer
        Mobile[Mobile Apps<br/>iOS/Android]
        Web[Web Client<br/>React]
    end
    
    subgraph CDN Layer
        CDN[CDN<br/>CloudFront/Akamai]
    end
    
    subgraph Load Balancing Layer
        LB[Load Balancer<br/>AWS ALB/Nginx]
    end
    
    subgraph API Gateway Layer
        API[API Gateway<br/>Kong/AWS API Gateway]
        Auth[Auth Service<br/>OAuth 2.0]
    end
    
    subgraph Application Services
        UserSvc[User Service<br/>Node.js/Go]
        PostSvc[Post Service<br/>Go]
        FeedSvc[Feed Service<br/>Go]
        EngageSvc[Engagement Service<br/>Go]
        MediaSvc[Media Upload Service<br/>Go]
    end
    
    subgraph Feed Generation
        FanoutSvc[Fanout Service<br/>Go]
        RankSvc[Ranking Service<br/>Python/ML]
    end
    
    subgraph Message Queue Layer
        MQ[Message Queue<br/>Kafka]
        PubSub[Pub/Sub<br/>Redis Streams]
    end
    
    subgraph Processing Layer
        MediaProc[Media Processor<br/>FFmpeg Workers]
        NotifProc[Notification Processor<br/>Go]
    end
    
    subgraph Data Layer - Databases
        UserDB[(User DB<br/>PostgreSQL)]
        PostDB[(Post DB<br/>Cassandra)]
        GraphDB[(Graph DB<br/>Neo4j/DGraph)]
        AnalyticsDB[(Analytics DB<br/>ClickHouse)]
    end
    
    subgraph Data Layer - Cache
        RedisCluster[Redis Cluster<br/>Feed Cache]
        RedisTimeline[Redis Sorted Sets<br/>Timeline Cache]
        Memcached[Memcached<br/>Object Cache]
    end
    
    subgraph Storage Layer
        S3[Object Storage<br/>S3/GCS]
        Search[Search Index<br/>Elasticsearch]
    end
    
    Mobile -->|1. Request| CDN
    Web -->|1. Request| CDN
    CDN -->|2. Cache Miss| LB
    CDN -.->|2a. Cache Hit| Mobile
    LB -->|3. Route| API
    API -->|4. Authenticate| Auth
    
    API -->|5a. User Request| UserSvc
    API -->|5b. Post Request| PostSvc
    API -->|5c. Feed Request| FeedSvc
    API -->|5d. Like/Comment| EngageSvc
    API -->|5e. Upload| MediaSvc
    
    UserSvc --> UserDB
    UserSvc --> Memcached
    
    PostSvc --> PostDB
    PostSvc -->|Publish Event| MQ
    PostSvc --> Memcached
    
    FeedSvc --> RedisTimeline
    FeedSvc --> RankSvc
    FeedSvc --> RedisCluster
    
    EngageSvc --> PostDB
    EngageSvc -->|Real-time Update| PubSub
    EngageSvc --> RedisCluster
    
    MediaSvc -->|Upload| S3
    MediaSvc -->|Queue Job| MQ
    
    MQ -->|New Post Event| FanoutSvc
    MQ -->|Process Media| MediaProc
    MQ -->|Send Notification| NotifProc
    
    FanoutSvc --> GraphDB
    FanoutSvc --> RedisTimeline
    FanoutSvc --> PostDB
    
    MediaProc --> S3
    MediaProc --> PostDB
    
    RankSvc --> AnalyticsDB
    RankSvc --> PostDB
    
    PostSvc --> Search
    UserSvc --> Search
```

### Data Flow Explanation

#### Post Creation Flow

1. **User uploads media** via mobile/web client
2. **CDN/Load Balancer** routes request to API Gateway
3. **API Gateway** authenticates user via Auth Service
4. **Media Upload Service** receives upload request
5. **Media Service** uploads original file to **Object Storage (S3)**
6. **Media Service** publishes job to **Message Queue (Kafka)**
7. **Media Processor workers** consume job and process media:
   - Generate thumbnails
   - Transcode videos to multiple formats
   - Extract metadata
   - Upload processed files back to S3
8. **Post Service** creates post metadata in **Post Database (Cassandra)**
9. **Post Service** publishes "new post" event to **Kafka**
10. **Fanout Service** consumes event and performs fan-out:
    - For regular users (<10k followers): Fan-out on write
    - For celebrities (>10k followers): Fan-out on read (lazy loading)
11. **Fanout Service** writes to followers' **Timeline Cache (Redis)**
12. **CDN** distributes media globally
13. User receives success response

#### Feed Generation Flow

1. **User requests feed** via mobile/web client
2. **CDN** serves if cached (for public/explore feeds)
3. **Load Balancer** routes to API Gateway
4. **Feed Service** receives request
5. **Feed Service** checks **Timeline Cache (Redis Sorted Sets)**:
   - If cache hit: Return cached timeline
   - If cache miss: Generate timeline on-demand
6. For timeline generation:
   - **Feed Service** queries **Graph DB** for followed users
   - For regular users: Read from pre-computed timeline
   - For celebrity followers: Merge celebrity content on-read
7. **Ranking Service** applies ML ranking algorithm:
   - Fetch user preferences from **Analytics DB**
   - Score posts based on engagement likelihood
   - Rerank timeline
8. **Feed Service** fetches post metadata from **Post DB**
9. **Feed Service** returns feed with CDN URLs for media
10. User's client downloads media from **CDN**

#### Engagement Flow (Like/Comment)

1. **User performs action** (like, comment, share)
2. **API Gateway** routes to **Engagement Service**
3. **Engagement Service** updates **Post Database** (increment counter)
4. **Engagement Service** updates **Cache** for real-time counts
5. **Engagement Service** publishes to **Pub/Sub (Redis Streams)**
6. **Real-time service** pushes update to connected clients via WebSocket
7. **Analytics DB** receives event for ranking algorithm updates

---

## Database Design

### Database Selection Strategy

**Multi-Database Approach:**

- **PostgreSQL**: User data, authentication (ACID compliance needed)
- **Cassandra**: Post metadata, high write throughput, horizontal scalability
- **Neo4j/DGraph**: Social graph, follower/following relationships
- **Redis**: Caching, timeline storage, real-time counters
- **Elasticsearch**: Full-text search for users, hashtags, posts
- **ClickHouse**: Analytics data, aggregations, reporting

### Schema Definitions

#### Users Table (PostgreSQL)

```text
Table: users
- user_id (PK, UUID)
- username (VARCHAR(30), UNIQUE, INDEXED)
- email (VARCHAR(255), UNIQUE, INDEXED)
- password_hash (VARCHAR(255))
- full_name (VARCHAR(100))
- bio (TEXT, max 500 chars)
- profile_picture_url (VARCHAR(500))
- is_verified (BOOLEAN, DEFAULT false)
- is_private (BOOLEAN, DEFAULT false)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
- last_login_at (TIMESTAMP)
- follower_count (INTEGER, DEFAULT 0, INDEXED)
- following_count (INTEGER, DEFAULT 0)
- post_count (INTEGER, DEFAULT 0)

Indexes:
- PRIMARY KEY (user_id)
- UNIQUE INDEX idx_username (username)
- UNIQUE INDEX idx_email (email)
- INDEX idx_follower_count (follower_count) -- for celebrity detection
```

#### Posts Table (Cassandra)

```text
Table: posts
Partition Key: user_id
Clustering Key: created_at DESC, post_id

Columns:
- post_id (UUID)
- user_id (UUID)
- post_type (VARCHAR) -- 'photo', 'video', 'carousel', 'story'
- caption (TEXT, max 2200 chars)
- media_urls (LIST<VARCHAR>) -- S3 URLs for original media
- thumbnail_urls (MAP<VARCHAR, VARCHAR>) -- size -> URL
- processed_urls (MAP<VARCHAR, VARCHAR>) -- format -> URL
- hashtags (SET<VARCHAR>)
- mentions (SET<UUID>)
- location (VARCHAR)
- like_count (COUNTER)
- comment_count (COUNTER)
- share_count (COUNTER)
- view_count (COUNTER)
- is_deleted (BOOLEAN)
- created_at (TIMESTAMP)
- expires_at (TIMESTAMP) -- for stories
- processing_status (VARCHAR) -- 'pending', 'processing', 'completed', 'failed'

Indexes:
- PRIMARY KEY ((user_id), created_at, post_id)
- INDEX idx_hashtags (hashtags) -- for hashtag queries
- INDEX idx_created_at (created_at) -- for trending queries
```

#### Post Metadata Table (Cassandra - for global post lookups)

```text
Table: posts_by_id
Partition Key: post_id

Columns:
- post_id (UUID, PK)
- user_id (UUID)
- [... same columns as posts table ...]

Indexes:
- PRIMARY KEY (post_id)
```

#### Likes Table (Cassandra)

```text
Table: likes
Partition Key: post_id
Clustering Key: created_at DESC, user_id

Columns:
- post_id (UUID)
- user_id (UUID)
- created_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((post_id), created_at, user_id)

Table: likes_by_user
Partition Key: user_id
Clustering Key: created_at DESC, post_id

Columns:
- user_id (UUID)
- post_id (UUID)
- created_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((user_id), created_at, post_id)
```

#### Comments Table (Cassandra)

```text
Table: comments
Partition Key: post_id
Clustering Key: created_at DESC, comment_id

Columns:
- comment_id (UUID)
- post_id (UUID)
- user_id (UUID)
- parent_comment_id (UUID, NULL for root comments)
- comment_text (TEXT, max 500 chars)
- like_count (COUNTER)
- reply_count (COUNTER)
- is_deleted (BOOLEAN)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((post_id), created_at, comment_id)
- INDEX idx_user_id (user_id)
```

#### Follows Table (Graph DB - Neo4j)

```text
Node: User
Properties:
- user_id (UUID, INDEXED)
- username (STRING)
- follower_count (INTEGER)
- is_celebrity (BOOLEAN) -- followers > 10k

Relationship: FOLLOWS
Properties:
- followed_at (TIMESTAMP)
- notification_enabled (BOOLEAN)

Cypher Queries:
// Get followers
MATCH (follower:User)-[:FOLLOWS]->(user:User {user_id: $userId})
RETURN follower

// Get following
MATCH (user:User {user_id: $userId})-[:FOLLOWS]->(following:User)
RETURN following

// Check if A follows B
MATCH (a:User {user_id: $userA})-[r:FOLLOWS]->(b:User {user_id: $userB})
RETURN r IS NOT NULL
```

#### Timeline Cache (Redis Sorted Sets)

```text
Key Pattern: timeline:{user_id}
Type: Sorted Set
Score: timestamp (for chronological ordering)
Member: post_id

Commands:
ZADD timeline:{user_id} {timestamp} {post_id}
ZREVRANGE timeline:{user_id} 0 49 -- Get top 50 posts
ZCARD timeline:{user_id} -- Get count
EXPIRE timeline:{user_id} 86400 -- 24-hour TTL
```

#### Feed Cache (Redis Hash)

```text
Key Pattern: feed:ranked:{user_id}
Type: Hash
Fields: post_id -> score

Commands:
HSET feed:ranked:{user_id} {post_id} {relevance_score}
HGETALL feed:ranked:{user_id}
EXPIRE feed:ranked:{user_id} 3600 -- 1-hour TTL
```

#### Engagement Counters (Redis Strings)

```text
Key Patterns:
- post:likes:{post_id} -> COUNT
- post:comments:{post_id} -> COUNT
- post:shares:{post_id} -> COUNT
- post:views:{post_id} -> COUNT

Commands:
INCR post:likes:{post_id}
DECR post:likes:{post_id} -- for unlike
GET post:likes:{post_id}
EXPIRE post:*:{post_id} 604800 -- 7-day TTL, sync to DB
```

#### Hashtag Index (Elasticsearch)

```json
{
  "index": "hashtags",
  "mappings": {
    "properties": {
      "hashtag": {
        "type": "keyword"
      },
      "post_count": {
        "type": "long"
      },
      "trending_score": {
        "type": "float"
      },
      "recent_posts": {
        "type": "nested",
        "properties": {
          "post_id": { "type": "keyword" },
          "timestamp": { "type": "date" }
        }
      },
      "created_at": {
        "type": "date"
      }
    }
  }
}
```

#### User Search Index (Elasticsearch)

```json
{
  "index": "users_search",
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword"
      },
      "username": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "full_name": {
        "type": "text",
        "analyzer": "standard"
      },
      "bio": {
        "type": "text"
      },
      "follower_count": {
        "type": "long"
      },
      "is_verified": {
        "type": "boolean"
      }
    }
  }
}
```

#### Analytics Events (ClickHouse)

```text
Table: user_events

Columns:
- event_id (UUID)
- user_id (UUID)
- event_type (String) -- 'post_view', 'like', 'comment', 'share', 'follow'
- post_id (Nullable(UUID))
- target_user_id (Nullable(UUID))
- session_id (UUID)
- timestamp (DateTime)
- device_type (String)
- platform (String)

Engine: MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, timestamp)
```

### Database Sharding Strategy

#### Post Database Sharding (Cassandra)

```text
Sharding Strategy: User-based partitioning
Shard Key: user_id

Rationale:
- All posts from a user are co-located
- Efficient user profile page queries
- Natural distribution (users are well-distributed)

Number of shards: 512 (for future growth)

Example routing:
shard_id = hash(user_id) % 512
```

#### Timeline Cache Sharding (Redis)

```text
Sharding Strategy: Consistent hashing on user_id
Number of Redis clusters: 64

Rationale:
- Distribute load evenly
- Allow independent scaling
- Fault isolation
```

---

## API Design

### Base Configuration

**Base URL:**

```text
https://api.socialmedia.com/v1
```

**Authentication:**

- Mechanism: JWT (JSON Web Tokens)
- Token expiration: Access token (15 min), Refresh token (30 days)
- Header: `Authorization: Bearer <access_token>`

**Versioning:**

- URL-based versioning: `/v1/`, `/v2/`

**Rate Limiting:**

- Authenticated users: 5,000 requests/hour
- Unauthenticated: 100 requests/hour
- Media uploads: 100 uploads/hour
- Engagement actions: 1,000 actions/hour

### Authentication Endpoints

#### Register User

```http
POST /v1/auth/register
```

**Request:**

```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecurePass123!",
  "full_name": "John Doe",
  "date_of_birth": "1990-01-15"
}
```

**Response (201 Created):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "johndoe",
  "email": "john@example.com",
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

#### Login

```http
POST /v1/auth/login
```

**Request:**

```json
{
  "email": "john@example.com",
  "password": "SecurePass123!"
}
```

**Response (200 OK):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

#### Refresh Token

```http
POST /v1/auth/refresh
```

**Request:**

```json
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl..."
}
```

**Response (200 OK):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 900
}
```

#### Logout

```http
POST /v1/auth/logout
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response:** 204 No Content

### User Endpoints

#### Get User Profile

```http
GET /v1/users/{user_id}
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "johndoe",
  "full_name": "John Doe",
  "bio": "Photography enthusiast 📸",
  "profile_picture_url": "https://cdn.socialmedia.com/users/550e.../profile.jpg",
  "is_verified": true,
  "is_private": false,
  "follower_count": 15420,
  "following_count": 328,
  "post_count": 142,
  "is_following": false,
  "is_followed_by": false,
  "created_at": "2023-01-15T10:30:00Z"
}
```

#### Update User Profile

```http
PATCH /v1/users/{user_id}
```

**Headers:**

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request:**

```json
{
  "full_name": "John Doe Updated",
  "bio": "New bio text",
  "is_private": true
}
```

**Response (200 OK):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "username": "johndoe",
  "full_name": "John Doe Updated",
  "bio": "New bio text",
  "is_private": true,
  "updated_at": "2025-10-02T14:30:00Z"
}
```

#### Search Users

```http
GET /v1/users/search?q={query}&limit={limit}&offset={offset}
```

**Query Parameters:**

- `q` (required): Search query string
- `limit` (optional): Results per page (default: 20, max: 50)
- `offset` (optional): Pagination offset (default: 0)

**Response (200 OK):**

```json
{
  "users": [
    {
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "full_name": "John Doe",
      "profile_picture_url": "https://cdn.socialmedia.com/users/550e.../profile.jpg",
      "is_verified": true,
      "follower_count": 15420
    }
  ],
  "total_count": 156,
  "has_more": true,
  "next_offset": 20
}
```

### Follow/Unfollow Endpoints

#### Follow User

```http
POST /v1/users/{user_id}/follow
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "success": true,
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_following": true,
  "follower_count": 15421
}
```

#### Unfollow User

```http
DELETE /v1/users/{user_id}/follow
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "success": true,
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_following": false,
  "follower_count": 15420
}
```

#### Get Followers

```http
GET /v1/users/{user_id}/followers?limit={limit}&cursor={cursor}
```

**Query Parameters:**

- `limit` (optional): Results per page (default: 20, max: 100)
- `cursor` (optional): Cursor for pagination

**Response (200 OK):**

```json
{
  "followers": [
    {
      "user_id": "660e8400-e29b-41d4-a716-446655440000",
      "username": "janedoe",
      "full_name": "Jane Doe",
      "profile_picture_url": "https://cdn.socialmedia.com/users/660e.../profile.jpg",
      "is_verified": false,
      "followed_at": "2025-09-15T10:30:00Z"
    }
  ],
  "next_cursor": "eyJsYXN0X2lkIjoi...",
  "has_more": true
}
```

### Post Endpoints

#### Create Post

```http
POST /v1/posts
```

**Headers:**

```http
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Request (multipart/form-data):**

```text
media[]: <binary file 1>
media[]: <binary file 2>
caption: "Beautiful sunset! #nature #photography"
location: "San Francisco, CA"
```

**Response (202 Accepted):**

```json
{
  "post_id": "770e8400-e29b-41d4-a716-446655440000",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "processing",
  "message": "Post is being processed",
  "estimated_time": 30
}
```

#### Get Post

```http
GET /v1/posts/{post_id}
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "post_id": "770e8400-e29b-41d4-a716-446655440000",
  "user": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "johndoe",
    "profile_picture_url": "https://cdn.socialmedia.com/users/550e.../profile.jpg",
    "is_verified": true
  },
  "post_type": "photo",
  "caption": "Beautiful sunset! #nature #photography",
  "media": [
    {
      "media_id": "880e8400-e29b-41d4-a716-446655440000",
      "type": "photo",
      "url": "https://cdn.socialmedia.com/posts/770e.../original.jpg",
      "thumbnail_url": "https://cdn.socialmedia.com/posts/770e.../thumb.jpg",
      "width": 1080,
      "height": 1350
    }
  ],
  "hashtags": ["nature", "photography"],
  "location": "San Francisco, CA",
  "like_count": 1542,
  "comment_count": 87,
  "share_count": 23,
  "view_count": 45230,
  "is_liked": false,
  "is_saved": false,
  "created_at": "2025-10-02T18:30:00Z"
}
```

### Feed Endpoints

#### Get Home Feed

```http
GET /v1/feed/home?limit={limit}&cursor={cursor}
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Query Parameters:**

- `limit` (optional): Results per page (default: 20, max: 50)
- `cursor` (optional): Cursor for pagination
- `algorithm` (optional): "chronological" or "ranked" (default: "ranked")

**Response (200 OK):**

```json
{
  "posts": [
    {
      "post_id": "770e8400-e29b-41d4-a716-446655440000",
      "user": {
        "user_id": "550e8400-e29b-41d4-a716-446655440000",
        "username": "johndoe",
        "profile_picture_url": "https://cdn.socialmedia.com/users/550e.../profile.jpg",
        "is_verified": true
      },
      "post_type": "photo",
      "caption": "Beautiful sunset!",
      "media": [],
      "like_count": 1542,
      "comment_count": 87,
      "created_at": "2025-10-02T18:30:00Z",
      "relevance_score": 0.87
    }
  ],
  "next_cursor": "eyJsYXN0X2lkIjoi...",
  "has_more": true,
  "refresh_token": "feed_refresh_xyz123"
}
```

### Engagement Endpoints

#### Like Post

```http
POST /v1/posts/{post_id}/like
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "success": true,
  "post_id": "770e8400-e29b-41d4-a716-446655440000",
  "is_liked": true,
  "like_count": 1543
}
```

#### Create Comment

```http
POST /v1/posts/{post_id}/comments
```

**Headers:**

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

**Request:**

```json
{
  "comment_text": "Amazing photo!",
  "parent_comment_id": null
}
```

**Response (201 Created):**

```json
{
  "comment_id": "990e8400-e29b-41d4-a716-446655440000",
  "post_id": "770e8400-e29b-41d4-a716-446655440000",
  "user": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "johndoe",
    "profile_picture_url": "https://cdn.socialmedia.com/users/550e.../profile.jpg"
  },
  "comment_text": "Amazing photo!",
  "like_count": 0,
  "created_at": "2025-10-02T19:30:00Z"
}
```

### Hashtag Endpoints

#### Get Posts by Hashtag

```http
GET /v1/hashtags/{hashtag}/posts?limit={limit}&cursor={cursor}
```

**Response (200 OK):**

```json
{
  "hashtag": "nature",
  "post_count": 1542890,
  "posts": [
    {
      "post_id": "770e8400-e29b-41d4-a716-446655440000",
      "thumbnail_url": "https://cdn.socialmedia.com/posts/770e.../thumb.jpg",
      "like_count": 1542,
      "comment_count": 87
    }
  ],
  "next_cursor": "eyJsYXN0X2lkIjoi...",
  "has_more": true
}
```

#### Get Trending Hashtags

```http
GET /v1/hashtags/trending?limit={limit}
```

**Query Parameters:**

- `limit` (optional): Number of results (default: 10, max: 50)
- `timeframe` (optional): "24h", "7d", "30d" (default: "24h")

**Response (200 OK):**

```json
{
  "trending_hashtags": [
    {
      "hashtag": "nature",
      "post_count": 45230,
      "trending_score": 0.95,
      "change_percent": 125.3
    }
  ],
  "updated_at": "2025-10-02T20:00:00Z"
}
```

### Story Endpoints

#### Create Story

```http
POST /v1/stories
```

**Headers:**

```http
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

**Response (202 Accepted):**

```json
{
  "story_id": "aa0e8400-e29b-41d4-a716-446655440000",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "processing",
  "expires_at": "2025-10-03T20:00:00Z"
}
```

#### Get Stories Feed

```http
GET /v1/stories/feed
```

**Headers:**

```http
Authorization: Bearer <access_token>
```

**Response (200 OK):**

```json
{
  "stories": [
    {
      "user": {
        "user_id": "660e8400-e29b-41d4-a716-446655440000",
        "username": "janedoe",
        "profile_picture_url": "https://cdn.socialmedia.com/users/660e.../profile.jpg"
      },
      "story_count": 3,
      "latest_story": {
        "story_id": "bb0e8400-e29b-41d4-a716-446655440000",
        "media_url": "https://cdn.socialmedia.com/stories/bb0e.../media.jpg",
        "created_at": "2025-10-02T15:30:00Z",
        "expires_at": "2025-10-03T15:30:00Z"
      },
      "is_seen": false
    }
  ]
}
```

### Cross-Cutting API Concerns

#### Error Response Format

All error responses follow this structure:

```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "The requested post does not exist",
    "details": {
      "post_id": "invalid-id"
    },
    "request_id": "req_xyz123abc",
    "timestamp": "2025-10-02T20:00:00Z"
  }
}
```

**Common Error Codes:**

- `UNAUTHORIZED` (401): Invalid or expired token
- `FORBIDDEN` (403): Insufficient permissions
- `RESOURCE_NOT_FOUND` (404): Resource doesn't exist
- `VALIDATION_ERROR` (422): Invalid input data
- `RATE_LIMIT_EXCEEDED` (429): Too many requests
- `INTERNAL_SERVER_ERROR` (500): Server error

#### Pagination Strategy

**Cursor-based pagination:**

- Used for feeds and timelines (unpredictable data)
- Cursor is an opaque token
- Ensures consistent results even with new data

**Offset-based pagination:**

- Used for search results
- Simple but can miss items with concurrent updates

#### Rate Limiting Headers

```http
X-RateLimit-Limit: 5000
X-RateLimit-Remaining: 4872
X-RateLimit-Reset: 1696276800
Retry-After: 3600
```

#### Caching Headers

```http
Cache-Control: public, max-age=300
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 02 Oct 2025 20:00:00 GMT
```

### API Trade-Offs

#### Decision: REST vs GraphQL

**Choice:** REST API

**Pros:**

- Simpler for mobile clients
- Better caching with CDN
- Mature ecosystem and tooling
- Easier rate limiting per endpoint

**Cons:**

- Over-fetching data in some cases
- Multiple requests for related data
- Less flexible for clients

**Justification:** REST provides better performance with CDN caching for read-heavy workloads. The mobile-first approach benefits from predictable request patterns and easier offline support.

#### Decision: Cursor vs Offset Pagination

**Choice:** Cursor-based for feeds, offset for search

**Pros:**

- Cursor: Consistent results with real-time updates
- Cursor: Better performance for large datasets
- Offset: Simpler for random access (search)

**Cons:**

- Cursor: Can't jump to specific pages
- Offset: Can miss items with concurrent inserts

**Justification:** Feeds are constantly updated, so cursor-based pagination prevents duplicate or missing posts. Search results are relatively static, so offset pagination is acceptable.

---

## Deep-Dive Components

### 1. Feed Generation System

The feed generation system is the heart of the platform, responsible for serving 10B impressions per day with <500ms latency.

#### Architecture

##### Hybrid Approach: Fan-out on Write + Fan-out on Read

```text
┌─────────────────────────────────────────────────────────────┐
│                    Feed Generation System                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  New Post Event                                               │
│       │                                                       │
│       v                                                       │
│  ┌──────────────┐                                            │
│  │  Fanout      │                                            │
│  │  Dispatcher  │                                            │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ├──────────────────┬──────────────────┐              │
│         v                  v                  v              │
│  ┌────────────┐     ┌────────────┐     ┌──────────────┐     │
│  │ Regular    │     │ Celebrity  │     │  Story       │     │
│  │ User Path  │     │ User Path  │     │  Handler     │     │
│  │ (Fan-out   │     │ (Lazy      │     │  (TTL-based) │     │
│  │  on Write) │     │  Loading)  │     │              │     │
│  └─────┬──────┘     └─────┬──────┘     └──────┬───────┘     │
│        │                  │                   │              │
│        v                  v                   v              │
│  ┌──────────────────────────────────────────────────┐        │
│  │       Redis Timeline Cache (Sorted Sets)         │        │
│  │  Key: timeline:{user_id}                         │        │
│  │  Score: timestamp, Member: post_id               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

#### Regular User Path (Fan-out on Write)

For users with <10,000 followers:

1. **Post Creation:**
   - User creates post
   - Post metadata saved to Cassandra
   - Event published to Kafka topic `post.created`

2. **Fanout Service Processes Event:**
   - Query Graph DB for all followers (batch query)
   - For each follower, add post to their timeline cache
   - Use Redis pipeline for batch writes
   - Target: 10,000 writes in <100ms

3. **Timeline Cache Update:**

   ```text
   For each follower_id:
     ZADD timeline:{follower_id} {post_timestamp} {post_id}
     LTRIM timeline:{follower_id} 0 999  # Keep only top 1000 posts
     EXPIRE timeline:{follower_id} 86400  # 24-hour TTL
   ```

4. **Feed Request:**
   - User requests feed
   - Read from `timeline:{user_id}` (Redis sorted set)
   - Fetch post metadata from cache/DB
   - Return results

#### Celebrity User Path (Fan-out on Read)

For users with >10,000 followers (celebrity threshold):

1. **Post Creation:**
   - Post saved to Cassandra
   - Mark user as celebrity (if follower_count > 10k)
   - NO fanout to followers

2. **Feed Request (Merge on Read):**
   - User requests feed
   - Fetch regular timeline from cache
   - Query: "Get recent posts from celebrity users I follow"
   - Merge celebrity posts with regular timeline
   - Rank combined results
   - Return merged feed

3. **Celebrity Post Query:**

   ```text
   SELECT post_id, created_at
   FROM posts_by_user
   WHERE user_id IN (SELECT celebrity_ids FROM user_following WHERE user_id = {requesting_user})
   AND created_at > {last_24_hours}
   ORDER BY created_at DESC
   LIMIT 50
   ```

4. **Merge Logic:**
   - Fetch regular timeline (200 posts from cache)
   - Fetch celebrity posts (50 most recent)
   - Merge based on timestamp
   - Apply ranking algorithm
   - Return top 50 posts

#### Ranking Algorithm

**Input Features:**

```python
features = {
    # Content features
    'post_age': time_since_post_creation,
    'media_type': 'photo' | 'video' | 'carousel',
    'has_multiple_media': boolean,
    
    # Author features
    'author_follower_count': int,
    'author_is_verified': boolean,
    'user_follows_author': boolean,
    'user_author_interaction_score': float,  # Past engagement
    
    # Engagement features
    'like_count': int,
    'comment_count': int,
    'share_count': int,
    'view_count': int,
    'engagement_rate': (likes + comments + shares) / views,
    'engagement_velocity': engagement_count / time_since_post,
    
    # User preference features
    'user_past_engagement_with_type': float,
    'user_past_engagement_with_hashtags': float,
    'user_location_match': boolean,
    'user_interest_match': float,
    
    # Contextual features
    'time_of_day': hour,
    'day_of_week': int,
    'user_session_length': float
}
```

**Scoring Model:**

```python
def calculate_relevance_score(post, user, context):
    """
    Calculate relevance score for a post
    
    Returns:
        float: Score between 0 and 1
    """
    # Time decay (exponential)
    age_hours = (now - post.created_at).hours
    time_decay = exp(-age_hours / 24)  # 24-hour half-life
    
    # Author score
    author_score = (
        0.3 * user.following_affinity[post.author_id] +  # How much user engages with author
        0.2 * post.author.is_verified +
        0.1 * normalize(post.author.follower_count)
    )
    
    # Engagement score
    engagement_score = (
        0.4 * normalize(post.like_count / post.view_count) +  # Like rate
        0.3 * normalize(post.comment_count) +  # Absolute comments
        0.3 * normalize(post.engagement_velocity)  # Trending factor
    )
    
    # Content match score
    content_score = (
        0.5 * cosine_similarity(user.interests, post.hashtags) +
        0.3 * (1 if post.media_type in user.preferred_types else 0) +
        0.2 * (1 if post.location == user.location else 0)
    )
    
    # Final score (weighted combination)
    final_score = (
        0.25 * time_decay +
        0.30 * author_score +
        0.30 * engagement_score +
        0.15 * content_score
    )
    
    return final_score
```

**Implementation:**

- Model trained using TensorFlow/PyTorch
- Inference via TensorFlow Serving or ONNX Runtime
- Batch prediction for feed generation
- Model updated daily with recent engagement data
- A/B testing framework for model experiments

#### Performance Optimizations

1. **Caching Layers:**
   - L1: User timeline cache (Redis) - 100GB per cluster
   - L2: Post metadata cache (Memcached) - 50GB per cluster
   - L3: Ranked feed cache (Redis) - 1-hour TTL

2. **Batch Operations:**
   - Batch fetch post metadata (100 posts per query)
   - Batch scoring (1000 posts per ML inference call)
   - Redis pipelining for multiple ZADD operations

3. **Precomputation:**
   - Explore feed pre-generated every 10 minutes
   - Trending content updated every 5 minutes
   - User interest vectors updated daily

4. **Query Optimization:**
   - Fetch only necessary fields (lean queries)
   - Parallel queries to multiple shards
   - Query result pagination

---

### 2. Media Processing Pipeline

Handles 7,000 uploads/sec at peak, processing photos and videos with <30s latency.

#### Architecture

```text
┌────────────────────────────────────────────────────────────┐
│                 Media Processing Pipeline                   │
├────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Upload          2. Queue         3. Process             │
│  ┌──────────┐      ┌──────────┐      ┌──────────────┐      │
│  │  API     │─────>│  Kafka   │─────>│   Workers    │      │
│  │ Gateway  │      │  Topic:  │      │  (FFmpeg)    │      │
│  │          │      │  media.  │      │              │      │
│  │ Direct   │      │  process │      │ 100k workers │      │
│  │ to S3    │      └──────────┘      └──────┬───────┘      │
│  └──────────┘                               │              │
│                                              v              │
│                                       ┌────────────┐        │
│                                       │  Generate  │        │
│  S3 Buckets                           │ Thumbnails │        │
│  ┌──────────────────────────────┐    │  Transcode │        │
│  │ /original/{post_id}/         │<───┤   Videos   │        │
│  │ /thumbnails/{post_id}/       │    │  Extract   │        │
│  │ /processed/{post_id}/        │    │  Metadata  │        │
│  └──────────────────────────────┘    └────────────┘        │
│                                                              │
│  4. CDN Distribution                                         │
│  ┌────────────────────────────────────────────────┐         │
│  │  CloudFront / Akamai                           │         │
│  │  - 200+ edge locations                         │         │
│  │  - Cache hit ratio: 95%+                       │         │
│  └────────────────────────────────────────────────┘         │
│                                                              │
└────────────────────────────────────────────────────────────┘
```

#### Processing Steps

**Photo Processing:**

1. **Upload:**
   - Multipart upload to S3 (client-side chunking)
   - Generate pre-signed URL for direct upload
   - Max file size: 10MB

2. **Thumbnail Generation:**
   - Small: 150x150 (profile grid)
   - Medium: 640x640 (feed)
   - Large: 1080x1080 (full view)

3. **Optimization:**
   - Format: Convert to WebP (30% smaller than JPEG)
   - Compression: 85% quality
   - Strip EXIF data (privacy)
   - Apply content-aware cropping

4. **Processing Time:** ~2 seconds per photo

**Video Processing:**

1. **Upload:**
   - Chunked upload (10MB chunks)
   - Resumable upload support
   - Max file size: 500MB
   - Max duration: 10 minutes

2. **Transcoding:**
   - Formats: 360p, 480p, 720p, 1080p
   - Codec: H.264 (compatibility), H.265 (efficiency)
   - Adaptive bitrate streaming (HLS/DASH)
   - Audio: AAC, 128kbps

3. **Thumbnail Generation:**
   - Extract frame at 10% duration
   - Generate 3 preview frames
   - Create animated GIF preview (3 seconds)

4. **Optimization:**
   - Two-pass encoding for better quality
   - Content-aware bitrate adjustment
   - Remove audio if silent

5. **Processing Time:** ~30 seconds per minute of video

#### Worker Architecture

**Horizontal Scaling:**

- Kubernetes-based worker pools
- Auto-scaling based on queue depth
- Target: Queue depth < 1000 messages
- Scale-up trigger: Depth > 1000 for 2 minutes
- Scale-down trigger: Depth < 100 for 10 minutes

**Worker Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: media-processor
spec:
  replicas: 1000
  template:
    spec:
      containers:
      - name: processor
        image: media-processor:v1
        resources:
          requests:
            cpu: "2"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        env:
        - name: KAFKA_TOPIC
          value: "media.process"
        - name: S3_BUCKET
          value: "socialmedia-media"
        - name: WORKER_THREADS
          value: "4"
```

**Processing Queue (Kafka):**

- Topic: `media.process`
- Partitions: 64 (for parallelism)
- Replication factor: 3
- Retention: 7 days
- Compression: LZ4

**Message Format:**

```json
{
  "job_id": "uuid",
  "post_id": "uuid",
  "user_id": "uuid",
  "media_type": "photo" | "video",
  "source_url": "s3://bucket/original/post_id/file.jpg",
  "output_bucket": "socialmedia-media",
  "priority": "high" | "normal" | "low",
  "processing_options": {
    "thumbnail_sizes": [150, 640, 1080],
    "video_formats": ["360p", "720p", "1080p"],
    "optimize_for_mobile": true
  },
  "created_at": "2025-10-02T20:00:00Z"
}
```

#### Error Handling

1. **Retry Logic:**
   - Max retries: 3
   - Backoff: Exponential (1s, 2s, 4s)
   - Dead letter queue for failed jobs

2. **Validation:**
   - File format validation
   - Malware scanning (ClamAV)
   - Content moderation (AWS Rekognition)
   - NSFW detection

3. **Monitoring:**
   - Processing time per job
   - Success/failure rates
   - Queue depth alerts
   - Worker utilization

---

### 3. Fanout Service (Celebrity Problem Solution)

Solves the "celebrity problem" where a single post to 100M followers would cause system overload.

#### Problem Statement

**Naive Fanout (Doesn't Scale):**

```text
Celebrity with 100M followers posts:
- Need to write to 100M timelines
- At 1ms per write = 100,000 seconds = 27+ hours
- Completely unacceptable
```

#### Solution: Hybrid Fanout Strategy

**Decision Tree:**

```text
New Post Created
    │
    ├─> Follower Count < 10k
    │   └─> Fan-out on Write (push to all follower timelines)
    │
    ├─> Follower Count 10k - 1M
    │   └─> Partial Fanout (push to active followers only)
    │
    └─> Follower Count > 1M
        └─> Fan-out on Read (no pre-computation, merge at read time)
```

#### Implementation

**User Classification:**

```python
def classify_user(user):
    """
    Classify user type based on follower count
    """
    if user.follower_count < 10_000:
        return 'regular'
    elif user.follower_count < 1_000_000:
        return 'influencer'
    else:
        return 'celebrity'
```

**Regular User Fanout (< 10k followers):**

```python
def fanout_regular_user(post, author):
    """
    Push post to all follower timelines
    """
    # Get all followers (from Graph DB or cached)
    follower_ids = graph_db.get_followers(author.user_id)
    
    # Batch write to Redis (use pipeline)
    pipeline = redis_client.pipeline()
    for follower_id in follower_ids:
        pipeline.zadd(
            f'timeline:{follower_id}',
            {post.post_id: post.created_at.timestamp()}
        )
        pipeline.ltrim(f'timeline:{follower_id}', 0, 999)  # Keep top 1000
        pipeline.expire(f'timeline:{follower_id}', 86400)  # 24h TTL
    
    pipeline.execute()
```

**Celebrity User Handling (> 1M followers):**

```python
def handle_celebrity_post(post, author):
    """
    Store post in celebrity feed, no fanout
    """
    # Just save to celebrity's own timeline
    redis_client.zadd(
        f'celebrity:posts:{author.user_id}',
        {post.post_id: post.created_at.timestamp()}
    )
    
    # Mark in database that this user is celebrity
    db.update(
        'users',
        {'user_id': author.user_id},
        {'is_celebrity': True}
    )
```

**Feed Generation for Celebrity Followers:**

```python
def generate_feed_with_celebrities(user_id, limit=50):
    """
    Generate feed merging regular timeline and celebrity posts
    """
    # Step 1: Get user's regular timeline (from fanout)
    regular_timeline = redis_client.zrevrange(
        f'timeline:{user_id}',
        0, 200,  # Get top 200 posts
        withscores=True
    )
    
    # Step 2: Get celebrities this user follows
    celebrity_ids = graph_db.get_following_celebrities(user_id)
    
    # Step 3: Fetch recent posts from celebrities (parallel)
    celebrity_posts = []
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [
            executor.submit(
                redis_client.zrevrange,
                f'celebrity:posts:{celeb_id}',
                0, 50,
                withscores=True
            )
            for celeb_id in celebrity_ids
        ]
        for future in as_completed(futures):
            celebrity_posts.extend(future.result())
    
    # Step 4: Merge and sort by timestamp
    all_posts = regular_timeline + celebrity_posts
    all_posts.sort(key=lambda x: x[1], reverse=True)  # Sort by score (timestamp)
    
    # Step 5: Fetch post metadata
    post_ids = [post[0] for post in all_posts[:limit]]
    posts_metadata = batch_fetch_posts(post_ids)
    
    # Step 6: Apply ranking algorithm
    ranked_posts = rank_posts(posts_metadata, user_id)
    
    return ranked_posts[:limit]
```

**Optimization: Partial Fanout for Influencers (10k - 1M followers):**

```python
def fanout_influencer(post, author):
    """
    Push to active followers only
    """
    # Get active followers (logged in last 7 days)
    active_followers = db.query("""
        SELECT follower_id
        FROM follows
        WHERE following_id = :author_id
        AND follower_id IN (
            SELECT user_id
            FROM users
            WHERE last_login_at > NOW() - INTERVAL '7 days'
        )
        LIMIT 100000
    """, author_id=author.user_id)
    
    # Fanout to active followers only
    pipeline = redis_client.pipeline()
    for follower_id in active_followers:
        pipeline.zadd(
            f'timeline:{follower_id}',
            {post.post_id: post.created_at.timestamp()}
        )
    pipeline.execute()
```

#### Performance Metrics

**Regular Users (<10k followers):**

- Fanout time: <100ms (10k writes via pipeline)
- Feed load time: <50ms (Redis read + metadata fetch)

**Influencers (10k-1M followers):**

- Fanout time: <500ms (100k active followers)
- Feed load time: <100ms (merge with celebrity content)

**Celebrities (>1M followers):**

- Fanout time: 0ms (no fanout)
- Feed load time: <200ms (merge from multiple celebrities)

---

## Trade-Offs Analysis

### Decision 1: SQL vs NoSQL for Post Storage

**Choice:** Cassandra (NoSQL)

**Pros:**

- Horizontal scalability (easily add nodes)
- High write throughput (200M posts/day = 2,315 writes/sec)
- Natural partitioning by user_id
- Time-series data model fits use case
- Linear scalability for reads and writes

**Cons:**

- Limited query flexibility (no JOINs)
- Eventual consistency (acceptable for social media)
- More complex data modeling
- Requires careful partition key selection

**Justification:** The write-heavy workload (200M posts/day) and need for horizontal scalability make Cassandra ideal. Posts are naturally partitioned by user, and we rarely need complex queries across users.

---

### Decision 2: PostgreSQL for User Data

**Choice:** PostgreSQL (SQL)

**Pros:**

- ACID compliance for critical user data
- Rich querying capabilities
- Strong consistency for auth operations
- Mature tooling and ecosystem
- Good for <10TB datasets

**Cons:**

- Vertical scaling limitations
- Sharding complexity
- Slower writes compared to NoSQL

**Justification:** User data requires strong consistency (can't have duplicate usernames) and ACID guarantees. The dataset size (1.5TB) fits comfortably in PostgreSQL with read replicas.

---

### Decision 3: Graph Database for Social Graph

**Choice:** Neo4j/DGraph

**Pros:**

- Native graph traversal (efficient follower queries)
- Multi-hop queries (friends-of-friends)
- Relationship properties (followed_at, notification_enabled)
- Graph algorithms (influencer detection, community detection)

**Cons:**

- Specialized database (operational complexity)
- Scaling challenges for write-heavy workloads
- Higher cost compared to relational DB

**Justification:** Social relationships are inherently graph-structured. Neo4j excels at traversal queries like "get all followers" or "mutual friends", which are performance-critical for feed generation.

**Alternative Considered:** Store follows in PostgreSQL with proper indexes. Would work for simple queries but poor performance for multi-hop traversals.

---

### Decision 4: Redis for Timeline Cache

**Choice:** Redis Sorted Sets

**Pros:**

- In-memory speed (<1ms latency)
- Sorted sets perfect for timelines (score = timestamp)
- Built-in operations (ZADD, ZREVRANGE, ZCARD)
- Horizontal scaling (Redis Cluster)
- TTL support for automatic cleanup

**Cons:**

- Volatile data (cache miss requires regeneration)
- Memory cost (10TB for 100M active users)
- Complex cluster management

**Justification:** Feed load time requirement (<500ms) demands in-memory caching. Redis sorted sets are purpose-built for ranked timelines with O(log N) insert and O(1) retrieval.

---

### Decision 5: Kafka for Message Queue

**Choice:** Apache Kafka

**Pros:**

- High throughput (millions of messages/sec)
- Durability (replicated, persistent)
- Replay capability (reprocess failed jobs)
- Partitioning for parallelism
- Mature ecosystem

**Cons:**

- Operational complexity
- Over-engineering for simple queues
- Storage costs (7-day retention)

**Justification:** Handles 7,000 media processing jobs/sec at peak. Durability ensures no lost uploads. Replay capability allows reprocessing if bugs occur.

**Alternative Considered:** AWS SQS - simpler but less throughput and no replay capability.

---

### Decision 6: CDN for Media Delivery

**Choice:** CloudFront/Akamai

**Pros:**

- 95%+ cache hit ratio (massive bandwidth savings)
- <100ms latency globally (200+ edge locations)
- DDoS protection
- Automatic failover
- Pay-as-you-go pricing

**Cons:**

- Cache invalidation complexity
- Initial cache warming takes time
- Vendor lock-in

**Justification:** Serving 10B impressions/day directly from origin would require 16.5Pbps bandwidth. CDN reduces this by 95% and improves latency from 500ms to <100ms.

---

### Decision 7: Elasticsearch for Search

**Choice:** Elasticsearch

**Pros:**

- Full-text search with relevance scoring
- Fuzzy matching for typos
- Aggregations for trending hashtags
- Near real-time indexing
- Horizontal scaling

**Cons:**

- Resource-intensive (CPU, memory)
- Eventual consistency (search lags writes)
- Complex cluster management

**Justification:** Hashtag search and user search require full-text capabilities. Elasticsearch provides sub-100ms search with fuzzy matching and ranking.

**Alternative Considered:** PostgreSQL full-text search - sufficient for small scale but doesn't scale to billions of posts.

---

### Decision 8: Hybrid Fanout Strategy

**Choice:** Fan-out on write for regular users, fan-out on read for celebrities

**Pros:**

- Solves celebrity problem (no 100M writes per post)
- Fast feed load for most users (<50ms from cache)
- Scalable to any follower count

**Cons:**

- Complex implementation (two code paths)
- Slower feed load for celebrity followers (200ms vs 50ms)
- Threshold tuning required (10k followers)

**Justification:** Pure fan-out on write doesn't scale for celebrities (27+ hours to fanout to 100M followers). Pure fan-out on read is slow for everyone. Hybrid approach optimizes for the common case (regular users) while handling edge case (celebrities).

---

## Caching Strategy

### Cache Layers

#### L1: User Timeline Cache (Redis)

**Purpose:** Store pre-computed timelines for fast feed generation

**Data:**

```text
Key: timeline:{user_id}
Type: Sorted Set
Score: post_timestamp
Member: post_id
Size per user: ~50KB (1000 posts * 50 bytes/post_id)
Total size: 100M users * 50KB = 5TB
TTL: 24 hours
```

**Operations:**

```text
Write: ZADD timeline:{user_id} {timestamp} {post_id}
Read: ZREVRANGE timeline:{user_id} 0 49
Eviction: Automatic (TTL) + LRU
```

**Hit Ratio Target:** 90%+

**Miss Handling:** Regenerate from Graph DB + Post DB

---

#### L2: Post Metadata Cache (Memcached)

**Purpose:** Cache frequently accessed post details

**Data:**

```text
Key: post:{post_id}
Type: String (serialized JSON)
Size per post: ~2KB
Total size: 20M hot posts * 2KB = 40GB
TTL: 6 hours
```

**Cached Fields:**

```json
{
  "post_id": "uuid",
  "user_id": "uuid",
  "caption": "text",
  "media_urls": [],
  "like_count": 1542,
  "comment_count": 87,
  "created_at": "timestamp"
}
```

**Hit Ratio Target:** 95%+

**Miss Handling:** Fetch from Cassandra

---

#### L3: User Profile Cache (Redis)

**Purpose:** Cache user profile data for feed rendering

**Data:**

```text
Key: user:{user_id}
Type: Hash
Fields: username, full_name, profile_picture_url, is_verified
Size per user: ~500 bytes
Total size: 100M active users * 500B = 50GB
TTL: 1 hour
```

**Hit Ratio Target:** 98%+

---

#### L4: Ranked Feed Cache (Redis)

**Purpose:** Cache ML-ranked feeds to avoid recomputation

**Data:**

```text
Key: feed:ranked:{user_id}
Type: List
Members: [post_id_1, post_id_2, ..., post_id_50]
Size per user: ~2KB
Total size: 100M users * 2KB = 200GB
TTL: 15 minutes (fresh rankings)
```

**Hit Ratio Target:** 70% (many users don't return within 15 min)

---

### Cache Invalidation Strategy

#### 1. Time-Based Invalidation (TTL)

**Use Cases:**

- Timeline cache: 24 hours (old posts naturally expire)
- Post metadata: 6 hours (engagement counts eventually consistent)
- Ranked feed: 15 minutes (ensure fresh rankings)

**Rationale:** Social media can tolerate eventual consistency. Stale engagement counts for a few hours are acceptable.

---

#### 2. Event-Based Invalidation

**Trigger Events:**

```python
# User updates profile
on_profile_update(user_id):
    redis.delete(f'user:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')

# User follows someone
on_follow(user_id, target_id):
    redis.delete(f'timeline:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')

# User unfollows someone
on_unfollow(user_id, target_id):
    redis.delete(f'timeline:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')
    # Remove target's posts from timeline
    target_post_ids = get_user_recent_posts(target_id)
    redis.zrem(f'timeline:{user_id}', *target_post_ids)

# Post deleted
on_post_delete(post_id, author_id):
    redis.delete(f'post:{post_id}')
    # Remove from all follower timelines (async)
    publish_event('post.deleted', {post_id, author_id})
```

---

#### 3. Approximate Counters

**Problem:** Incrementing like_count in cache and DB creates race conditions and consistency issues.

**Solution:** Use Redis for real-time counters, periodically sync to Cassandra.

```python
# Increment counter in Redis
def like_post(post_id, user_id):
    # Atomic increment
    redis.incr(f'post:likes:{post_id}')
    
    # Record individual like
    cassandra.insert('likes', {
        'post_id': post_id,
        'user_id': user_id,
        'created_at': now()
    })
    
    # Publish event for analytics
    kafka.publish('engagement.like', {post_id, user_id})

# Periodic sync (every 5 minutes)
def sync_counters_to_db():
    for post_id in get_active_posts():
        like_count = redis.get(f'post:likes:{post_id}')
        if like_count:
            cassandra.update('posts', {
                'post_id': post_id,
                'like_count': like_count
            })
```

**Tradeoff:** Counters may be slightly off (±1%) but eventual consistency is acceptable for social media.

---

### What NOT to Cache

1. **User Password Hashes:** Security risk
2. **Private/Sensitive Data:** GDPR compliance issues
3. **Real-Time Engagement:** Like/comment actions must be immediately visible to actor
4. **Financial Data:** Requires strong consistency
5. **Audit Logs:** Must be durable and accurate

---

## Bottlenecks & Improvements

### Potential Bottlenecks

#### Bottleneck 1: Database Write Contention

**Problem:**

- 173,600 like actions/sec at peak
- Hot posts can receive thousands of likes/sec
- Cassandra counter updates can create contention

**Solution:**

- Use Redis for real-time counters (INCR is atomic and fast)
- Batch sync to Cassandra every 5 minutes
- Use Cassandra COUNTER type for approximate counts
- Implement optimistic locking for critical updates

**Monitoring:**

```text
Metric: database_write_latency_p99
Threshold: > 100ms
Alert: Page on-call engineer
```

---

#### Bottleneck 2: Media Processing Queue Depth

**Problem:**

- Peak: 7,000 uploads/sec
- If workers fall behind, queue grows indefinitely
- User experience degrades (posts stuck in "processing")

**Solution:**

- Auto-scaling based on queue depth
- Scale-up trigger: Depth > 1000 for 2 minutes
- Scale-down trigger: Depth < 100 for 10 minutes
- Priority queue for verified users
- Reject uploads if queue > 10,000 (circuit breaker)

**Monitoring:**

```text
Metric: kafka_topic_lag (media.process)
Threshold: > 5000 messages
Alert: Auto-scale workers + notify team
```

---

#### Bottleneck 3: Graph Database Read Load

**Problem:**

- Every feed generation queries "get followers"
- 115,740 feed requests/sec = 115,740 graph queries/sec
- Neo4j read capacity: ~50,000 queries/sec per node

**Solution:**

- Cache follower lists in Redis
- Update cache on follow/unfollow events
- Use read replicas (3-5 replicas)
- Denormalize: Store follower_count in User DB

**Monitoring:**

```text
Metric: neo4j_query_latency_p95
Threshold: > 50ms
Alert: Add read replica
```

---

#### Bottleneck 4: CDN Origin Bandwidth

**Problem:**

- Cache miss rate: 5%
- Peak bandwidth: 16.5Pbps * 0.05 = 825Tbps
- Single origin can't handle this

**Solution:**

- Multi-region origin (10+ regions)
- Origin shield (secondary cache layer before origin)
- Increase cache TTL (1 hour → 6 hours for immutable media)
- Proactive cache warming for popular content

**Monitoring:**

```text
Metric: cdn_origin_requests_per_second
Threshold: > 100,000 req/sec per region
Alert: Add origin capacity
```

---

#### Bottleneck 5: Single Point of Failure (API Gateway)

**Problem:**

- All traffic goes through API Gateway
- If gateway fails, entire system is down

**Solution:**

- Multi-AZ deployment (3+ availability zones)
- Auto-scaling (target: 60% CPU utilization)
- Circuit breaker pattern (fail fast on downstream errors)
- Health checks with automatic failover
- Canary deployments (1% → 10% → 100%)

**Monitoring:**

```text
Metric: api_gateway_error_rate
Threshold: > 1%
Alert: Rollback deployment + page on-call
```

---

### Scalability Improvements

#### 1. Geographic Distribution

**Strategy:**

- Deploy in 5 regions: US-East, US-West, Europe, Asia, South America
- Route users to nearest region (latency-based routing)
- Replicate critical data across regions (user data, social graph)
- Local caching in each region

**Benefits:**

- Reduced latency (500ms → 100ms)
- Better availability (region failure doesn't affect other regions)
- Compliance (GDPR data residency)

**Implementation:**

```text
User Location: US-East
└─> Route to US-East cluster
    ├─> API Gateway (us-east-1)
    ├─> Feed Service (us-east-1)
    ├─> User DB (primary in us-east-1, replicas in us-west-2)
    └─> CDN (nearest edge location)
```

---

#### 2. Read Replicas

**Databases:**

- User DB (PostgreSQL): 1 primary + 5 read replicas
- Post DB (Cassandra): 3 replicas per datacenter
- Graph DB (Neo4j): 1 primary + 3 read replicas

**Read/Write Split:**

```python
# Writes go to primary
def create_user(user_data):
    return primary_db.insert('users', user_data)

# Reads go to replicas (round-robin)
def get_user(user_id):
    return replica_db.query('SELECT * FROM users WHERE user_id = :id', id=user_id)
```

**Benefits:**

- 5x read capacity
- Zero-downtime maintenance (rotate replicas)
- Disaster recovery (promote replica to primary)

---

#### 3. Sharding Strategy

**User DB Sharding:**

```text
Shard by user_id (consistent hashing)
Shard 0: user_id hash % 16 == 0
Shard 1: user_id hash % 16 == 1
...
Shard 15: user_id hash % 16 == 15

Start with 16 shards, grow to 256 shards over time
```

**Post DB Sharding:**

```text
Already sharded by user_id (Cassandra native)
512 virtual nodes per physical node
Automatic rebalancing on node addition
```

**Benefits:**

- Horizontal scalability (add shards as needed)
- Isolated failures (one shard down doesn't affect others)
- Parallel queries (query all shards simultaneously)

---

#### 4. Real-Time Updates (WebSocket)

**Current:** Polling (client requests feed every 30 seconds)

**Improvement:** WebSocket for live updates

```python
# Server-side
websocket_server = WebSocketServer()

@websocket_server.on_connect
def on_user_connect(user_id, websocket):
    # Subscribe to user's real-time updates
    pubsub.subscribe(f'user:{user_id}:updates', websocket)

@event_handler('engagement.like')
def on_like(post_id, user_id):
    # Notify post author
    post = get_post(post_id)
    pubsub.publish(f'user:{post.author_id}:updates', {
        'type': 'like',
        'post_id': post_id,
        'user_id': user_id
    })
```

**Benefits:**

- Instant updates (no 30-second polling delay)
- Reduced server load (no repeated polling requests)
- Better UX (real-time notifications)

---

### Monitoring and Observability

#### Key Metrics

**System Metrics:**

```text
- Latency (p50, p95, p99):
  - API Gateway: < 100ms p95
  - Feed Generation: < 500ms p95
  - Media Upload: < 2s p95

- Throughput:
  - Requests per second: 347,000/sec peak
  - Posts per second: 7,000/sec peak
  - Engagement actions per second: 220,000/sec peak

- Error Rates:
  - API errors: < 0.1%
  - Media processing failures: < 0.5%
  - Database errors: < 0.01%

- Resource Utilization:
  - CPU: < 70% average
  - Memory: < 80% average
  - Disk: < 75% used
  - Network: < 60% of capacity
```

**Business Metrics:**

```text
- Daily Active Users (DAU)
- Posts per day
- Engagement rate (likes + comments + shares / impressions)
- Feed load time
- Media upload success rate
- User retention (D1, D7, D30)
```

#### Alerting Strategy

**P0 Alerts (Page Immediately):**

```text
- API error rate > 1% for 5 minutes
- Database connection failures
- Payment system down
- Data loss detected
```

**P1 Alerts (Notify During Business Hours):**

```text
- Feed load time > 1s (p95)
- Media processing queue depth > 5000
- Cache hit rate < 80%
- Disk usage > 85%
```

**P2 Alerts (Weekly Review):**

```text
- Unusual traffic patterns
- Slow queries
- Cost anomalies
```

#### Logging

**Structured Logging Format:**

```json
{
  "timestamp": "2025-10-02T20:00:00Z",
  "level": "INFO",
  "service": "feed-service",
  "trace_id": "abc123",
  "user_id": "550e8400...",
  "action": "generate_feed",
  "duration_ms": 245,
  "result": "success",
  "metadata": {
    "posts_returned": 50,
    "cache_hit": true
  }
}
```

**Log Aggregation:**

- Tool: ELK Stack (Elasticsearch, Logstash, Kibana)
- Retention: 30 days (hot), 1 year (cold storage)
- Indexing: By service, timestamp, user_id, trace_id

#### Distributed Tracing

**Tool:** Jaeger or OpenTelemetry

**Trace Example:**

```text
User Request: Get Feed
├─ API Gateway (10ms)
├─ Auth Service (5ms)
├─ Feed Service (230ms)
│  ├─ Redis: Get Timeline (2ms)
│  ├─ Graph DB: Get Celebrities (15ms)
│  ├─ Post Service: Batch Fetch Metadata (50ms)
│  ├─ Ranking Service: ML Inference (150ms)
│  └─ Format Response (13ms)
└─ Response to Client (5ms)

Total: 245ms
```

---

## Security Considerations

### 1. Authentication & Authorization

**Mechanism:**

- JWT tokens (access token + refresh token)
- Access token: 15-minute expiration
- Refresh token: 30-day expiration (stored in HttpOnly cookie)

**Token Structure:**

```json
{
  "sub": "user_id",
  "username": "johndoe",
  "iat": 1696276800,
  "exp": 1696277700,
  "roles": ["user"],
  "permissions": ["read:feed", "write:post", "delete:own_post"]
}
```

**Implementation:**

```python
def generate_access_token(user_id):
    payload = {
        'sub': user_id,
        'iat': now(),
        'exp': now() + timedelta(minutes=15)
    }
    return jwt.encode(payload, SECRET_KEY, algorithm='HS256')

def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload['sub']
    except jwt.ExpiredSignatureError:
        raise AuthenticationError('Token expired')
    except jwt.InvalidTokenError:
        raise AuthenticationError('Invalid token')
```

---

### 2. Input Validation

**Validation Rules:**

```python
post_validation = {
    'caption': {
        'max_length': 2200,
        'allowed_chars': 'unicode',
        'xss_filter': True
    },
    'media': {
        'max_files': 10,
        'allowed_types': ['image/jpeg', 'image/png', 'video/mp4'],
        'max_size': '500MB'
    },
    'hashtags': {
        'max_count': 30,
        'max_length': 50,
        'pattern': r'^[a-zA-Z0-9_]+$'
    }
}

def validate_post(post_data):
    # Length check
    if len(post_data['caption']) > 2200:
        raise ValidationError('Caption too long')
    
    # XSS prevention
    caption = bleach.clean(post_data['caption'], strip=True)
    
    # SQL injection prevention (use parameterized queries)
    # Done automatically by ORM
    
    return sanitized_data
```

---

### 3. Rate Limiting

**Implementation:**

```python
def rate_limit(user_id, action, limit, window):
    """
    Token bucket algorithm
    """
    key = f'rate_limit:{user_id}:{action}'
    current = redis.get(key) or 0
    
    if current >= limit:
        raise RateLimitExceeded(f'Max {limit} {action}s per {window}')
    
    redis.incr(key)
    redis.expire(key, window)

# Usage
@rate_limit(user_id=request.user_id, action='post', limit=100, window=3600)
def create_post(post_data):
    ...
```

**Rate Limits:**

```text
- Login attempts: 5 per 15 minutes
- Post creation: 100 per hour
- Comment creation: 1000 per hour
- Follow/unfollow: 200 per hour
- API requests: 5000 per hour (authenticated), 100 per hour (unauthenticated)
```

---

### 4. Data Encryption

**Encryption at Rest:**

- User passwords: bcrypt with salt (cost factor: 12)
- Sensitive data: AES-256 encryption
- Database: Encrypted volumes (AWS KMS)
- Backups: Encrypted before upload

**Encryption in Transit:**

- TLS 1.3 for all API endpoints
- Certificate pinning for mobile apps
- HTTPS redirect (HTTP → HTTPS)

**Implementation:**

```python
# Password hashing
def hash_password(password):
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

def verify_password(password, hash):
    return bcrypt.checkpw(password.encode(), hash)

# Sensitive data encryption
def encrypt_sensitive_data(data):
    cipher = AES.new(ENCRYPTION_KEY, AES.MODE_GCM)
    ciphertext, tag = cipher.encrypt_and_digest(data.encode())
    return base64.b64encode(cipher.nonce + tag + ciphertext)
```

---

### 5. Content Moderation

**Automated Moderation:**

```python
def moderate_content(media_url, caption):
    """
    Multi-stage moderation pipeline
    """
    # Stage 1: NSFW detection (AWS Rekognition)
    nsfw_score = aws_rekognition.detect_moderation_labels(media_url)
    if nsfw_score > 0.8:
        return 'rejected', 'NSFW content'
    
    # Stage 2: Violence/gore detection
    violence_score = detect_violence(media_url)
    if violence_score > 0.7:
        return 'rejected', 'Violent content'
    
    # Stage 3: Text moderation (caption)
    profanity_detected = profanity_filter.check(caption)
    if profanity_detected:
        return 'flagged', 'Profanity detected'
    
    # Stage 4: Spam detection
    spam_score = spam_classifier.predict(caption)
    if spam_score > 0.9:
        return 'rejected', 'Spam content'
    
    return 'approved', None

# Apply moderation before publishing
def create_post(post_data):
    status, reason = moderate_content(post_data['media_url'], post_data['caption'])
    
    if status == 'rejected':
        raise ContentViolationError(reason)
    elif status == 'flagged':
        # Send to manual review queue
        queue_for_human_review(post_data)
        return {'status': 'pending_review'}
    else:
        # Publish post
        return publish_post(post_data)
```

---

### 6. DDoS Protection

**Measures:**

- CloudFlare/AWS Shield for L3/L4 protection
- API Gateway rate limiting
- Challenge-response for suspicious traffic (CAPTCHA)
- Geographic blocking (block high-risk countries)
- Anomaly detection (ML-based traffic analysis)

---

### 7. Privacy & Compliance

**GDPR Compliance:**

- Right to access: API endpoint to export user data
- Right to erasure: Delete user and all content
- Data portability: Export in JSON format
- Consent management: Explicit opt-in for data processing

**Implementation:**

```python
def export_user_data(user_id):
    """
    Export all user data (GDPR Article 15)
    """
    user = db.get_user(user_id)
    posts = db.get_user_posts(user_id)
    comments = db.get_user_comments(user_id)
    likes = db.get_user_likes(user_id)
    
    return {
        'user': user,
        'posts': posts,
        'comments': comments,
        'likes': likes,
        'exported_at': now()
    }

def delete_user_data(user_id):
    """
    Permanently delete user data (GDPR Article 17)
    """
    # Delete from primary databases
    db.delete_user(user_id)
    db.delete_user_posts(user_id)
    db.delete_user_comments(user_id)
    
    # Delete from caches
    redis.delete(f'user:{user_id}')
    redis.delete(f'timeline:{user_id}')
    
    # Delete from search index
    elasticsearch.delete('users', user_id)
    
    # Delete media from S3 (async)
    s3.delete_user_media(user_id)
```

---

## Future Enhancements

### 1. Direct Messaging (DM)

**Architecture:**

- WebSocket-based real-time messaging
- Message storage in Cassandra (partition by conversation_id)
- End-to-end encryption (Signal Protocol)
- Read receipts and typing indicators
- Media sharing in DMs

**Implementation:**

```python
# Message schema
{
    'message_id': 'uuid',
    'conversation_id': 'uuid',
    'sender_id': 'uuid',
    'content': 'encrypted_blob',
    'media_url': 's3_url',
    'created_at': 'timestamp',
    'read_at': 'timestamp'
}

# WebSocket handler
@websocket.on_message
def handle_dm(message):
    recipient = message['recipient_id']
    
    # Store message
    db.insert('messages', message)
    
    # Send to recipient (if online)
    if is_user_online(recipient):
        websocket.send(recipient, message)
    else:
        # Queue for push notification
        push_notification(recipient, 'New message from {sender}')
```

**Estimated Resources:**

- 50M DM users (10% of DAU)
- 500M messages/day
- Storage: 500M * 1KB = 500GB/day = 180TB/year

---

### 2. Live Streaming

**Architecture:**

- WebRTC for peer-to-peer streaming
- Media servers for distribution (Wowza/Ant Media Server)
- Adaptive bitrate streaming
- Chat integration
- Live reactions

**Implementation:**

```python
# Live stream schema
{
    'stream_id': 'uuid',
    'user_id': 'uuid',
    'title': 'string',
    'status': 'live' | 'ended',
    'viewer_count': 'int',
    'started_at': 'timestamp',
    'ended_at': 'timestamp',
    'chat_enabled': 'boolean'
}

# Start stream
def start_live_stream(user_id, title):
    stream = create_stream(user_id, title)
    
    # Notify followers
    followers = get_followers(user_id)
    notify_followers(followers, f'{user.username} is live!')
    
    # Start media server
    media_server.start_stream(stream.stream_id)
    
    return stream
```

**Estimated Resources:**

- 100k concurrent streams at peak
- 10M viewers at peak
- Bandwidth: 10M * 2Mbps = 20Tbps

---

### 3. Stories Enhancements

**Features:**

- Interactive stickers (polls, questions, quizzes)
- Augmented reality (AR) filters
- Story replies (DM responses)
- Story highlights (save stories beyond 24 hours)
- Story analytics (view counts, engagement)

**Implementation:**

```python
# Interactive poll sticker
{
    'sticker_type': 'poll',
    'question': 'Favorite color?',
    'options': ['Red', 'Blue', 'Green'],
    'votes': {
        'Red': 42,
        'Blue': 35,
        'Green': 23
    }
}

# AR filter application
def apply_ar_filter(story_media, filter_id):
    # Use OpenCV/TensorFlow for face detection
    faces = detect_faces(story_media)
    
    # Apply filter (e.g., dog ears)
    filtered_media = apply_filter(story_media, faces, filter_id)
    
    return filtered_media
```

---

### 4. Recommendation System

**ML-Based Content Discovery:**

- Collaborative filtering (users like you also liked...)
- Content-based filtering (similar to posts you've liked)
- Hybrid approach
- Trending content boost

**Implementation:**

```python
# User embedding
user_embedding = embed_user_preferences(user_id)
# [interest_tech: 0.8, interest_sports: 0.3, ...]

# Post embedding
post_embedding = embed_post_content(post_id)
# [topic_tech: 0.9, topic_sports: 0.1, ...]

# Similarity score
similarity = cosine_similarity(user_embedding, post_embedding)

# Ranking
def recommend_posts(user_id, num_posts=50):
    # Candidate generation (narrow down from billions)
    candidates = get_candidate_posts(user_id)  # ~10k candidates
    
    # Scoring
    scores = []
    for post in candidates:
        score = calculate_relevance_score(post, user_id)
        scores.append((post, score))
    
    # Ranking
    ranked_posts = sorted(scores, key=lambda x: x[1], reverse=True)
    
    return ranked_posts[:num_posts]
```

**Model Training:**

- Training data: 100B engagement events
- Model: Two-tower neural network
- Features: 500+ features
- Training frequency: Daily
- Serving: TensorFlow Serving (5ms inference latency)

---

### 5. Ads Platform

**Sponsored Content:**

- Native ads (looks like regular posts)
- Targeting (demographics, interests, behavior)
- Auction-based pricing (CPM, CPC, CPA)
- Ad frequency capping
- Performance analytics

**Implementation:**

```python
def insert_ads_in_feed(feed_posts, user_id):
    """
    Insert ads every N organic posts
    """
    AD_FREQUENCY = 5  # Show ad every 5 posts
    
    # Fetch relevant ads
    ads = ad_targeting_service.get_ads(
        user_demographics=get_user_demographics(user_id),
        user_interests=get_user_interests(user_id),
        context='home_feed'
    )
    
    # Insert ads in feed
    feed_with_ads = []
    ad_index = 0
    for i, post in enumerate(feed_posts):
        feed_with_ads.append(post)
        
        if (i + 1) % AD_FREQUENCY == 0 and ad_index < len(ads):
            feed_with_ads.append({
                'type': 'ad',
                'content': ads[ad_index],
                'ad_id': ads[ad_index]['id']
            })
            ad_index += 1
    
    return feed_with_ads
```

**Revenue Model:**

- CPM (Cost Per Mille): $5-20 per 1000 impressions
- Estimated revenue: 10B impressions/day * $10 CPM = $100k/day = $36M/year

---

### 6. Analytics Dashboard

**User Analytics:**

- Profile views
- Follower growth over time
- Post reach and engagement
- Best time to post
- Audience demographics

**Content Creator Tools:**

- Detailed post analytics
- Story insights
- Hashtag performance
- Competitor analysis
- Engagement rate trends

**Implementation:**

```python
# Analytics schema (ClickHouse)
{
    'event_id': 'uuid',
    'user_id': 'uuid',
    'post_id': 'uuid',
    'event_type': 'view' | 'like' | 'comment' | 'share',
    'timestamp': 'datetime',
    'device_type': 'mobile' | 'web',
    'location': 'country_code'
}

# Analytics query
def get_post_analytics(post_id):
    return {
        'views': count_events(post_id, 'view'),
        'likes': count_events(post_id, 'like'),
        'comments': count_events(post_id, 'comment'),
        'shares': count_events(post_id, 'share'),
        'engagement_rate': (likes + comments + shares) / views,
        'demographics': get_viewer_demographics(post_id),
        'top_locations': get_top_locations(post_id)
    }
```

---

### 7. Machine Learning Enhancements

**Content Understanding:**

- Object detection in images
- Scene classification
- Auto-tagging
- Auto-captioning

**User Safety:**

- Cyberbullying detection
- Self-harm content detection
- Misinformation flagging
- Deepfake detection

**Personalization:**

- Feed ranking optimization
- Notification timing optimization
- Content format preferences
- Language translation

---

## Conclusion

This social media platform design handles 500M daily active users with 200M posts/day and 10B feed impressions/day, meeting all performance requirements:

- **Feed loads in <500ms** via hybrid fanout strategy and multi-layer caching
- **Supports celebrity accounts** (100M+ followers) through fan-out on read
- **Real-time updates** for likes/comments using Redis Streams and WebSocket
- **Scalable media pipeline** with 100k workers processing 7,000 uploads/sec
- **99.9% uptime** through multi-AZ deployment, redundancy, and failover

**Key Architectural Decisions:**

1. **Hybrid fanout**: Solves celebrity problem while maintaining performance
2. **Multi-database approach**: Right tool for each data type
3. **Aggressive caching**: 90%+ cache hit rates for sub-100ms latency
4. **Asynchronous processing**: Handles spiky workloads gracefully
5. **Horizontal scalability**: Every component can scale independently

The system is designed for growth, with clear paths to scale to billions of users through sharding, geographic distribution, and optimized algorithms.

---

Document created for interview preparation. Last updated: October 2, 2025
