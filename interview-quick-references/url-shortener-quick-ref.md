# URL SHORTENER (TINYURL) - Interview Quick Reference

**File Purpose:** Concise interview-day reference guide for URL shortening service like TinyURL. Covers all critical talking points, architectural decisions, and key numbers. Review in 5-10 minutes before interviews.

**Last Updated:** October 2, 2025

---

## 🎯 Core Problem Statement

- **What:** Build a highly scalable URL shortening service that converts long URLs into short, shareable links with redirect capability and analytics
- **Key Challenge:** Generate unique short codes at scale, handle 100:1 read-to-write ratio, achieve <100ms redirect latency globally
- **Scale:** 100M DAU, 1M URL creations/day, 100M redirects/day, 10B redirects/month

---

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| **Daily Active Users** | 100M | Given assumption |
| **URLs Created/Day** | 1M | 100M × 1% creation rate |
| **Redirects/Day** | 100M | 1M × 100 (read:write ratio) |
| **Write QPS (Avg)** | 12 | 1M / 86,400 seconds |
| **Read QPS (Avg)** | 1,160 | 100M / 86,400 seconds |
| **Peak Write QPS** | 36 | 3x average |
| **Peak Read QPS** | 3,480 | 3x average |
| **URL Storage/Entry** | 600 bytes | 7B hash + 500B URL + metadata |
| **Daily Storage** | 600 MB | 1M × 600 bytes |
| **Annual Storage** | 216 GB | 600MB × 365 days |
| **5-Year Storage** | 1.6 TB | 216GB × 5 × 1.5 overhead |
| **Analytics/Click** | 230 bytes | Timestamp + IP + referrer + UA |
| **Analytics Storage (2yr)** | 16.6 TB | 100M × 230B × 365 × 2 |
| **Short URL Length** | 7 characters | Base62 encoding |
| **Character Set** | 62 chars | [a-z, A-Z, 0-9] |
| **Total Combinations** | 3.5 trillion | 62^7 |
| **Years to Exhaust** | 9,589 years | 3.5T / 1M per day |
| **Cache Memory Needed** | 216 GB | 20% of URLs (hot data) |
| **Redirect Latency Target** | <100ms | P99 requirement |
| **URL Creation Latency** | <500ms | P99 requirement |

---

## 🏗️ High-Level Architecture

```text
[User/Browser]
      |
      v
[CDN (CloudFlare)] ---> Cache popular redirects (1h TTL)
      |
      v
[Load Balancer (AWS ALB)]
      |
      v
[API Gateway (Kong)] <---> [Rate Limiter (Redis)]
      |
      +---> [URL Shortening Service (Write Path)]
      |           |
      |           v
      |     [ID Generation Service (Snowflake)]
      |           |
      |           v
      |     [Primary DB (PostgreSQL)]
      |           |
      |           v
      |     [Redis Cache] (proactive write)
      |
      +---> [URL Redirect Service (Read Path)]
                  |
                  v
            [Redis Cache] (cache-aside, 90% hit rate)
                  |
                  v (cache miss)
            [Read Replicas (PostgreSQL)] (2+ replicas)
                  |
                  v (async)
            [Kafka Queue] ---> [Analytics Worker]
                                      |
                                      v
                              [ClickHouse Analytics DB]
```

### Key Components

- **CDN:** Caches HTTP 302 redirects for extremely popular URLs, 1-hour TTL, global edge distribution
- **Load Balancer:** AWS ALB for traffic distribution, SSL termination, health checks
- **API Gateway:** Kong/AWS API Gateway for routing, auth, rate limiting orchestration
- **Rate Limiter:** Redis-based token bucket, 10 req/hr (anon), 1K req/hr (auth)
- **URL Shortening Service:** Generates short codes, stores mappings, Node.js/Go stateless instances
- **URL Redirect Service:** Handles redirects, cache-first strategy, logs analytics events
- **ID Generation Service:** Snowflake-based distributed ID generation (1024 machines, 4M IDs/sec/machine)
- **Redis Cache:** Cluster mode (6 nodes: 3 master + 3 replica), 384GB total, no TTL for URL mappings, LRU eviction
- **Primary PostgreSQL:** ACID compliance, strong consistency, URL mappings table, master-slave replication
- **Read Replicas:** 2+ PostgreSQL replicas for read scaling, eventual consistency acceptable
- **Kafka Queue:** Async analytics pipeline, decouples redirect from analytics, durable message storage
- **Analytics Worker:** Consumes events, enriches with GeoIP, batch writes to ClickHouse
- **ClickHouse:** Columnar analytics DB, time-series optimized, 100K events/sec, 10x compression

---

## 💾 Data Model (Essentials)

### Core Tables

```text
url_mappings (PostgreSQL)
- short_url_hash (PK, VARCHAR(7), UNIQUE)
- original_url (TEXT, NOT NULL)
- user_id (BIGINT, NULLABLE)
- created_at (TIMESTAMP, NOT NULL)
- expires_at (TIMESTAMP, NULLABLE)
- custom_alias (BOOLEAN, DEFAULT FALSE)
- is_active (BOOLEAN, DEFAULT TRUE)
- click_count (BIGINT, DEFAULT 0)
Indexes: PRIMARY KEY (short_url_hash), idx_user_id, idx_expires_at, idx_created_at, idx_active
Partitioning: Monthly partitions by created_at

users (Optional)
- user_id (PK, BIGINT, AUTO_INCREMENT)
- email (VARCHAR(255), UNIQUE)
- username (VARCHAR(50), UNIQUE)
- api_key_hash (VARCHAR(64), UNIQUE)
- created_at (TIMESTAMP)
- is_active (BOOLEAN)
Indexes: PRIMARY KEY (user_id), UNIQUE idx_email, UNIQUE idx_api_key

click_events (ClickHouse)
- event_id (UUID, PK)
- short_code (VARCHAR(7))
- clicked_at (TIMESTAMP, PARTITION KEY)
- ip_address_hash (VARCHAR(64))
- referrer (TEXT)
- user_agent (TEXT)
- country_code (CHAR(2))
- city (VARCHAR(100))
Partitioning: Daily partitions by clicked_at
Clustering: ORDER BY (short_code, clicked_at)
```

### Sharding Strategy

- **URL Mappings:** Shard by first 2 characters of short_code (distributes writes)
- **Users:** Shard by user_id (if needed at massive scale)
- **Analytics:** Partition by time (daily), cluster by short_code

---

## 🔌 API Design (Key Endpoints)

```http
POST /v1/shorten
GET /{short_code}
GET /v1/analytics/{short_code}
GET /v1/urls
DELETE /v1/urls/{short_code}
GET /v1/health
```

### Key APIs

1. **Create Short URL:** `POST /v1/shorten` → Returns short_url, stores in DB, proactively caches (201 Created)
2. **Redirect:** `GET /{short_code}` → Checks cache first, returns 302 Found with Location header, logs analytics async (<100ms)
3. **Get Analytics:** `GET /v1/analytics/{short_code}` → Returns aggregated click data with time-series, referrers, geo (requires auth)
4. **List URLs:** `GET /v1/urls` → Paginated user URLs, sorted by clicks/created_at (requires auth)

### Rate Limits

- Anonymous: 10 URL creations/hour per IP, unlimited redirects
- Authenticated Free: 1,000 URL creations/hour
- Authenticated Pro: 100,000 URL creations/hour

### Authentication

- API Key in `Authorization: Bearer {api_key}` header
- Anonymous access allowed for basic shortening

---

## 🚀 Critical Talking Points

### 1. Snowflake ID Generation

- **What:** Distributed 64-bit ID generator using timestamp (41 bits) + machine ID (10 bits) + sequence (12 bits), encoded to Base62 (7 chars)
- **Why:** Guaranteed uniqueness without database coordination, eliminates collision checks, enables horizontal scaling
- **Detail:** Supports 1024 machines, generates 4M IDs/sec per machine, 69 years of timestamps from custom epoch (2025-01-01), no database calls needed
- **Alternative:** MD5 hash with collision detection (requires DB check, slower, database bottleneck), UUID (too long, not user-friendly), Auto-increment (single point of failure)
- **Trade-off:** IDs are sequential/predictable (mitigated by Base62 encoding), requires clock sync and machine ID coordination

### 2. Multi-Tier Caching Strategy

- **What:** Three-tier caching: CDN (edge) → Redis (application) → Database (source)
- **Why:** 100:1 read:write ratio means caching is critical; reduces DB load by 90%+, achieves <100ms latency
- **Detail:**
  - Tier 1 (CDN): Extremely popular URLs, 1-hour TTL, geographic edge caching
  - Tier 2 (Redis): Cache-aside pattern, 90% hit rate, 6-node cluster (384GB), no TTL, LRU eviction
  - Tier 3 (DB): Read replicas for cache misses, eventual consistency acceptable
- **Alternative:** Write-through caching (more complex, cache always in sync), longer TTLs (stale data risk), single-tier (not enough capacity)
- **Cache warming:** Daily job preloads top 10K URLs by click_count, scheduled at 3 AM off-peak

### 3. Async Analytics Pipeline

- **What:** Event-driven analytics using Kafka message queue and ClickHouse columnar database
- **Why:** Analytics processing must not block redirects (0ms impact on latency), handles 100M events/day, enables real-time and historical analysis
- **Detail:**
  - Redirect service publishes event to Kafka (fire-and-forget, <1ms)
  - Analytics worker consumes in batches (1000 events), enriches with GeoIP data
  - ClickHouse stores with 10x compression, sub-second aggregation queries on billions of rows
  - Batch writes every 10 seconds or 1000 events
- **Alternative:** Synchronous DB writes (blocks redirects, poor UX), log files + batch processing (complex, delayed insights), separate microservice with sync calls (still blocks)
- **Monitoring:** Track Kafka consumer lag (<10 min), ClickHouse insert throughput, queue depth

### 4. Read-Write Path Separation

- **What:** Separate services and databases for URL creation (write) vs redirection (read)
- **Why:** Different performance characteristics: writes need ACID + uniqueness, reads need ultra-low latency + high throughput
- **Detail:**
  - Write path: API Gateway → Shortening Service → ID Gen → Primary DB → Cache update
  - Read path: CDN → Load Balancer → Redirect Service → Cache → Read Replica → Async analytics
  - Read replicas handle all redirect traffic, primary only handles writes
- **Alternative:** Single service + single DB (simpler, bottlenecks at scale), separate DBs but same service (less optimized)
- **Scaling:** Write path scales to 1K QPS with sharding, read path scales to 100K+ QPS with replicas + cache

### 5. Base62 Encoding & URL Space

- **What:** Encode 64-bit integer IDs to 7-character Base62 strings ([a-z, A-Z, 0-9])
- **Why:** Short, user-friendly URLs; 3.5 trillion combinations = 9,589 years at 1M/day; URL-safe without escaping
- **Detail:** 62^7 = 3,521,614,606,208 possible URLs, consistent 7-char length, case-sensitive for more space
- **Alternative:** Base58 (Bitcoin-style, avoids ambiguous chars like 0/O, smaller space), Hex (longer, 11 chars), UUID (too long, 36 chars)
- **Custom aliases:** Allow user-specified codes (check availability, validate uniqueness, handle conflicts with 409 Conflict)

### 6. Rate Limiting & Security

- **What:** Multi-layer rate limiting using Redis token bucket algorithm + malicious URL detection
- **Why:** Prevent abuse, DDoS protection, spam prevention, maintain service quality
- **Detail:**
  - Layer 1: CDN (1000 req/sec per IP)
  - Layer 2: API Gateway (user-specific limits: 10/hr anon, 1K/hr auth)
  - Layer 3: Application (per-endpoint limits)
  - URL validation: Blocklist check, format validation, suspicious pattern detection (javascript:, data:)
- **Alternative:** No rate limiting (vulnerable to abuse), single-layer (insufficient), CAPTCHA (poor UX)
- **Spam detection:** Track creation rate (50/5min = spam), domain diversity (many URLs to same domain), integration with Google Safe Browsing API

---

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| **ID Generation** | Snowflake (distributed) | MD5 hash + collision check | Guaranteed uniqueness, no DB coordination, 4M IDs/sec, eliminates bottleneck; predictability mitigated by Base62 |
| **Primary Database** | PostgreSQL (SQL) | Cassandra (NoSQL) | ACID guarantees for URL creation, strong consistency needed, complex queries for analytics; can shard for write scale |
| **Analytics Database** | ClickHouse (columnar) | PostgreSQL (same DB) | Optimized for time-series, 10x compression, sub-second aggregation on billions of rows, 100K inserts/sec |
| **Caching Strategy** | Cache-aside, no TTL | Write-through with TTL | Efficient memory (only hot data), simple invalidation, URLs are immutable; cache miss on first access acceptable |
| **Analytics Pipeline** | Async (Kafka) | Synchronous DB writes | Zero impact on redirect latency (critical), decoupled processing, handles spikes, scales independently; eventual consistency acceptable |
| **API Style** | REST | GraphQL | Simple CRUD operations, excellent HTTP caching, well-understood, matches URL resources; slight over-fetching acceptable |
| **Redirect Status** | 302 Found (temporary) | 301 Moved Permanently | Allows URL changes, better analytics tracking (browsers don't cache), flexible expiration; slight overhead vs 301 |
| **Read Replicas** | Master-slave replication | Multi-master | Read scaling for 100:1 ratio, simple consistency model, proven at scale; writes to single master only |
| **Partitioning** | Monthly by created_at | No partitioning | Efficient archival, query optimization, manageable table sizes; adds operational complexity |

---

## 🔥 Bottlenecks & Solutions

### 1. Database Write Contention (Primary DB)

- **Problem:** Single master PostgreSQL limits write throughput; 36 QPS peak can cause contention with index updates
- **Solution:** Shard by first 2 chars of short_code (distributes across masters), batch inserts where possible, use PgBouncer connection pooling (100 connections), optimize indexes (only essential), consider Cassandra for write-heavy workloads (linear scaling, eventual consistency trade-off)
- **Metric:** Write latency p95/p99, connection pool utilization
- **Alert:** p95 write latency >200ms, pool >80% utilized

### 2. Cache Memory Limitations

- **Problem:** 1B URLs = 600GB to cache all; Redis cluster memory fills, evicts potentially popular URLs
- **Solution:** LRU eviction with popularity scoring (track access frequency), cache only hot 20% (~216GB distributed), multi-tier caching (L1: local 1%, L2: Redis 20%, L3: DB 79%), Bloom filters to avoid unnecessary lookups, daily cache warming for top 10K URLs
- **Metric:** Cache hit ratio (target >90%), memory per node
- **Alert:** Hit rate <85%, memory >90%

### 3. Analytics Write Throughput

- **Problem:** 100K redirects/sec during peak creates massive Kafka queue backlog if ClickHouse can't keep up
- **Solution:** Batch writes (1000-5000 events), horizontal ClickHouse scaling (more nodes, parallel inserts), sampling for viral URLs (10% sample still accurate), increase Kafka retention (24h buffer), async buffering tolerates delays
- **Metric:** Kafka consumer lag, ClickHouse insert QPS
- **Alert:** Consumer lag >10 minutes, queue depth growing

### 4. Geographic Latency

- **Problem:** Users far from primary datacenter see 200-500ms latency (cross-continent), violates <100ms SLA
- **Solution:** Multi-region deployment (US East, US West, EU, APAC), GeoDNS routes to nearest region, replicate URL mappings to all regions (eventual consistency), CDN for popular redirects (<10ms at edge), split-brain protection with Raft consensus for coordination
- **Metric:** Latency by region, replication lag
- **Alert:** Regional latency p95 >150ms, replication lag >1s

### 5. ID Generation Service Availability

- **Problem:** If ID gen service fails, URL creation stops completely (single point of failure)
- **Solution:** Deploy multiple instances with unique machine IDs (1024 max), load balance across instances, pre-generate ID pool in Redis (background worker replenishes), fallback to hash-based generation temporarily (accept small collision risk during outage), health checks + auto-failover
- **Metric:** Service health, ID pool levels in Redis
- **Alert:** Service down, pool <1000 IDs remaining

---

## 💡 Interview Tips

### Start Here

1. **Clarify requirements:** Read:write ratio (100:1), DAU (100M), latency targets (<100ms redirect, <500ms creation), retention policy, custom aliases?
2. **Draw high-level architecture:** User → CDN → LB → API Gateway → Services → Cache → DB, emphasize read/write path separation
3. **Discuss ID generation:** This is the unique, critical component - spend 20% of time here explaining Snowflake approach

### Emphasize

- **Read optimization:** Multi-tier caching (CDN + Redis + DB), 90% cache hit rate, read replicas, <100ms latency
- **Scalability:** Snowflake IDs enable horizontal scaling, sharding for writes, replicas for reads, stateless services
- **Analytics decoupling:** Async pipeline ensures zero impact on redirect performance, eventual consistency acceptable
- **Trade-offs:** Always explain "why" - Snowflake vs hash, PostgreSQL vs Cassandra, cache-aside vs write-through, sync vs async analytics

### Be Ready For

- "How do you generate unique short codes?" → Snowflake algorithm: timestamp + machine ID + sequence, Base62 encoding, 3.5T space
- "What if two users create same URL?" → Same long_url can have different short codes (by design) OR deduplication logic (check before creating, return existing)
- "How to handle URL expiration?" → expires_at column, check on redirect (lazy), background job for cleanup, cache invalidation on expiry
- "How to scale to 1B QPS?" → Geographic distribution, more cache tiers, CDN edge caching, database sharding, ClickHouse horizontal scaling
- "What if cache goes down?" → Fallback to read replicas, performance degrades but service continues, automatic failover in Redis cluster
- "How to prevent abuse?" → Multi-layer rate limiting (Redis token bucket), URL validation (blocklist, pattern detection), spam detection (creation rate, domain diversity), Google Safe Browsing integration

### Don't Forget

- **Numbers:** Always calculate QPS, storage, cache size, justify with back-of-envelope math
- **Monitoring:** Cache hit rate, latency (p50/p95/p99), error rate, queue lag, database health
- **Security:** Rate limiting, malicious URL detection, input validation, API key auth, HTTPS only
- **Edge cases:** Expired URLs (410 Gone), custom alias conflicts (409 Conflict), clock skew in Snowflake (detect and handle)
- **Future enhancements:** Custom domains, QR codes, A/B testing redirects, link previews, bulk operations
- **Consistency:** Strong for URL creation (ACID), eventual for analytics (acceptable), eventual for read replicas (acceptable)

### Common Mistakes to Avoid

- ❌ Using auto-increment IDs (single DB bottleneck, not distributed)
- ❌ Synchronous analytics (blocks redirects, poor latency)
- ❌ Not discussing caching strategy (critical for 100:1 read ratio)
- ❌ Ignoring geographic distribution (global service needs multi-region)
- ❌ Missing rate limiting discussion (abuse prevention essential)
- ❌ Not explaining Base62 encoding (why 7 characters, why Base62)

---

## 🎓 Quick Reference Commands

### For Deep Dive

- **Snowflake ID:** 64-bit = 41b timestamp + 10b machine + 12b sequence → Base62 (7 chars), 4M IDs/sec/machine
- **Caching:** 3-tier (CDN 1h / Redis no-TTL / DB), cache-aside, LRU eviction, 90% hit rate target
- **Analytics:** Kafka → Worker (batch 1K events) → ClickHouse (columnar, 10x compression, 100K inserts/sec)
- **Sharding:** URL mappings by first 2 chars of short_code, time-based partitioning (monthly)
- **Rate Limiting:** Redis token bucket, 10/hr (anon), 1K/hr (auth), 100K/hr (pro)

### Key Metrics

- **Latency:** Redirect p99 <100ms, creation p99 <500ms
- **Throughput:** 12 write QPS (36 peak), 1160 read QPS (3480 peak)
- **Cache:** 90%+ hit rate, 216GB memory (20% of URLs)
- **Availability:** 99.99% uptime, <5.2 hours downtime/year
- **Storage:** 1.6TB (5 years URLs), 16.6TB (2 years analytics)

---

**🚀 Good luck with your interview! Remember: Focus on caching, ID generation, and read/write separation.**

---

**Document Stats:** 480 lines | 6-min read | Updated Oct 2, 2025

