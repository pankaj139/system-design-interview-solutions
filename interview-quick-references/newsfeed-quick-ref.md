# Newsfeed System - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a personalized newsfeed system (Facebook/LinkedIn style) supporting 300M DAU with sub-300ms feed load times.

---

## 🎯 Core Problem Statement

- **What:** Design a personalized newsfeed system that delivers relevant content to users with ranking
- **Key Challenge:** Efficiently fan-out posts to millions of followers while maintaining real-time performance
- **Scale:** 300M DAU, 100M posts/day, <300ms feed generation, 99.9% uptime

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Active Users | 300M | Given requirement |
| Posts per Day | 100M | ~0.33 posts/user/day |
| Feed Read QPS (Peak) | 105K | (300M × 10 refreshes) / 86,400 × 3 |
| Post Write QPS (Peak) | 3,600 | (100M / 86,400) × 3 |
| Feed Generation Time | <300ms | P95 latency target |
| Feed Cache Memory | 5TB | 50M users × 100KB/feed |
| 5-Year Storage | 28PB | Posts: 365TB + Images: 27.5PB |
| Redis Cache Hit Rate | >85% | Target for performance |
| Average Friends/User | 500 | Social graph size |
| Celebrity Threshold | >1M followers | Fan-out strategy switch |

## 🏗️ High-Level Architecture

```text
[Mobile/Web] → [CDN] → [Load Balancer] → [API Gateway]
                                              ↓
                                   [Feed Generation Service] ← [ML Ranking]
                                      ↓           ↓
                              [Feed Cache]   [Post Service]
                              (Redis 5TB)          ↓
                                          [PostgreSQL + Cassandra]
                                                   ↓
                                            [Kafka Events]
                                                   ↓
                              [Fan-out Workers] → [Real-time Updates]
```

**Key Components:**

- **CDN**: CloudFront for media (images/videos), 7-day cache
- **Load Balancer**: AWS ALB with path-based routing, sticky sessions for WebSocket
- **Feed Generation Service**: Go/Java, checks cache → ML ranking → aggregates posts
- **Feed Cache**: Redis Cluster, 15-min TTL, 5TB hot data
- **ML Ranking**: Python/TensorFlow, XGBoost model, 200+ features
- **Post Service**: PostgreSQL sharded by user_id (64 shards)
- **Message Storage**: Cassandra for high-write throughput
- **Fan-out Workers**: Kafka consumers, handles post distribution
- **Real-time**: WebSocket + Redis Pub/Sub for live updates

## 💾 Data Model (Essentials)

### Posts Table (PostgreSQL)

```text
posts
├── post_id (PK, UUID)
├── user_id (FK, UUID, INDEX)
├── content (TEXT)
├── created_at (TIMESTAMP, INDEX)
├── engagement_score (FLOAT, INDEX)
└── privacy_level (ENUM, INDEX)
```

### Friendships Table

```text
friendships
├── user_id (FK, INDEX)
├── friend_id (FK, INDEX)
├── status (ENUM: accepted, pending)
└── INDEX(user_id, status)
```

### Feed Cache (Redis)

```text
Key: feed:{user_id}:{page}
Value: JSON array of post objects
TTL: 15 minutes
```

**Sharding Strategy:** Hash partitioning by user_id (64 shards initially, expandable to 1024)

**Indexes:** Composite indexes on (user_id, created_at), (created_at, post_id) for pagination

## 🔌 API Design (Key Endpoints)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/v1/feed?cursor={cursor}&limit=20` | Get personalized feed with pagination |
| POST | `/v1/posts` | Create new post, triggers fan-out |
| POST | `/v1/posts/{id}/like` | Like post, update engagement |
| WSS | `/v1/ws/feed` | WebSocket for real-time updates |

## 🚀 Critical Talking Points

### Point 1: Hybrid Fan-out Strategy

- **What:** Combines push (write) and pull (read) fan-out based on follower count
- **Why:** Celebrity users (>1M followers) cause write amplification with pure push model
- **Detail:**
  - Regular users (<10K): Fan-out on Write (pre-compute feeds)
  - Celebrities (>1M): Fan-out on Read (generate on demand)
  - Hybrid (10K-1M): Push to active followers, pull for inactive
- **Alternative:** Pure push (simple but doesn't scale) or pure pull (slower for regular users)

### Point 2: ML-Based Feed Ranking

- **What:** Machine learning model ranks posts by predicted engagement probability
- **Why:** Personalization improves user engagement by 30-40%
- **Detail:**
  - Features: User behavior, post recency, social proof, content type
  - Model: XGBoost with 200+ features, trained weekly
  - Scoring: `Score = w1×Engagement + w2×Recency + w3×Relevance + w4×Social + w5×Diversity`
- **Alternative:** Time-based ranking (simpler, less engaging) or simple rule-based (doesn't personalize)

### Point 3: Multi-Level Caching

- **What:** CDN → Redis → Database hierarchy for fast feed retrieval
- **Why:** 85%+ cache hit rate reduces DB load from 105K QPS to <15K QPS
- **Detail:**
  - L1 (CDN): Static media, 7-day TTL
  - L2 (Redis): Hot feeds, 15-min TTL, 5TB memory
  - L3 (Database): Read replicas with query result caching
- **Alternative:** Single-level cache (simpler but less effective) or no cache (impossible at scale)

### Point 4: Pagination with Cursors

- **What:** Cursor-based pagination for consistent results during real-time updates
- **Why:** Offset-based fails when new posts inserted, causes duplicate/missing items
- **Detail:**
  - Cursor: Base64(`{timestamp}:{post_id}:{user_id}:{score}:{version}`)
  - Query: `WHERE (created_at, post_id) < (cursor) ORDER BY created_at DESC LIMIT 20`
  - Handles: Deleted posts, ranking changes, concurrent updates
- **Alternative:** Offset-based (simpler but inconsistent) or time-based only (no deterministic ordering)

### Point 5: Celebrity User Handler

- **What:** Specialized processing for users with >1M followers
- **Why:** Posting by celebrity causes 10M+ cache writes, system overload
- **Detail:**
  - Tiered distribution: Active (push), Semi-active (cache warm), Inactive (on-demand)
  - Rate-limited fan-out: Max 100K writes/second per celebrity
  - Dedicated queues and workers for celebrity content
- **Alternative:** Treat all users same (system crashes) or no special handling (poor UX)

### Point 6: Real-Time Updates

- **What:** WebSocket connections for live feed updates without polling
- **Why:** Reduces server load by 10x vs polling, better UX
- **Detail:**
  - WebSocket servers with consistent hashing by user_id
  - Redis Pub/Sub for cross-server message distribution
  - Fallback to long-polling for compatibility
- **Alternative:** Polling (higher load) or SSE (one-way only)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Database | PostgreSQL + Cassandra | Single database | Posts need high-write throughput (Cassandra), users need ACID (PostgreSQL) |
| Fan-out | Hybrid (Push + Pull) | Pure Push or Pure Pull | Push fast for regular users, pull scales for celebrities |
| Consistency | Eventual | Strong | Feed data can be slightly stale, availability > consistency |
| Ranking | ML-based | Time-based | 30-40% engagement improvement justifies complexity |
| Pagination | Cursor-based | Offset-based | Consistent results during real-time updates |
| Cache Storage | Redis | Memcached | Redis has richer data structures (sorted sets for feeds) |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution | Monitoring |
|------------|----------|-----------|
| Feed Generation Latency | Pre-compute for active users, simpler fallback algorithm | P95 latency, ML service response time |
| Database Write Contention | Sharding by user_id, write-through caching, batch updates | Write queue depth, connection pool usage |
| Cache Memory Pressure | LRU with activity weighting, compression, separate clusters | Memory usage, eviction rate, hit rate |
| Celebrity Post Fan-out | Lazy evaluation, sampled fan-out, tiered delivery | Fan-out queue depth, processing time |

## 💡 Interview Tips

**Start Here:**

1. Clarify requirements: DAU, posts/day, friends/user, real-time needs
2. Calculate numbers: QPS, storage, cache size
3. Draw high-level architecture
4. Deep-dive into fan-out strategy (most interesting part!)

**Emphasize:**

- Hybrid fan-out strategy (shows understanding of scale challenges)
- ML ranking (demonstrates modern system thinking)
- Multi-level caching (performance optimization)
- Celebrity user handling (edge case awareness)

**Be Ready For:**

- "How do you handle celebrity users?" → Hybrid fan-out strategy
- "How do you rank posts?" → ML features and scoring function
- "What if Redis fails?" → Fallback to DB, graceful degradation
- "How do you ensure consistency?" → Eventually consistent, explain trade-offs
- "How do you paginate?" → Cursor-based vs offset-based trade-offs
- "How do you handle real-time updates?" → WebSocket + Redis Pub/Sub

**Don't Forget:**

- Mention sharding strategy (critical for scale)
- Discuss cache invalidation (cache is useless without it)
- Address monitoring and alerting (production readiness)
- Consider failure scenarios (what breaks at 3x load?)
- Talk about consistency model (CAP theorem application)

**Common Mistakes to Avoid:**

- Don't use pure fan-out on write for all users (celebrity problem)
- Don't use offset-based pagination (breaks during updates)
- Don't forget to cache friend lists (reduce DB load)
- Don't over-engineer MVP (start with simpler time-based ranking)
- Don't ignore monitoring (how do you know it works?)

**Architecture Evolution:**

1. **MVP:** Simple time-based feed, fan-out on write, single Redis
2. **Scale:** Add ML ranking, hybrid fan-out, Redis cluster
3. **Optimize:** Multi-region, edge caching, celebrity handlers
4. **Advanced:** Predictive caching, real-time ML, analytics pipeline

**Key Numbers to Remember:**

- 300M DAU, 100M posts/day
- <300ms P95 latency
- 105K read QPS (peak), 3.6K write QPS (peak)
- 5TB Redis cache, 28PB storage (5 years)
- 85% cache hit rate target
- 500 average friends per user
- 1M followers = celebrity threshold
