# Text Storage Service (Pastebin) - Interview Quick Reference

**File Purpose:** This document provides a concise interview-ready quick reference for the Text Storage Service (Pastebin) system design, covering key metrics, architecture decisions, trade-offs, and critical talking points for technical interviews.

**Last Updated:** October 2, 2025

---

## 🎯 Core Problem Statement

- **What:** Build a scalable text storage service (like Pastebin) for sharing code snippets and text
- **Key Challenge:** Handle 10M pastes/day with sub-100ms latency while managing storage costs
- **Scale:** 100K concurrent users, 1B reads/day, up to 10MB per paste

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Pastes | 10M | Given requirement |
| Daily Reads | 1B | 10M × 100 (read/write ratio) |
| Write QPS (Peak) | 350/sec | 10M ÷ 86,400 × 3 (peak factor) |
| Read QPS (Peak) | 35K/sec | 1B ÷ 86,400 × 3 (peak factor) |
| Storage (5 years) | 60TB | 10M × 10.5KB × 365 × 5 × 0.3 (after expiration) |
| Bandwidth (Peak) | 3Gbps | 35K reads/sec × 10.5KB |
| URL Space | 3.5T | 62^7 (Base62, 7 characters) |

## 🏗️ High-Level Architecture

```text
[User] -> [CDN] -> [Load Balancer] -> [API Gateway] -> [Services]
                                                          |
                                                          v
[Write Service] -> [URL Generator] -> [Redis Counter]
       |                |
       v                v
[Object Storage]   [PostgreSQL] <- [Read Service] <- [Redis Cache]
       |                |
       v                v
[Expiration Worker] [Stats Worker]
```

**Components:**

- **CDN:** CloudFront, 95% cache hit, <10ms latency
- **Load Balancer:** Nginx, consistent hashing, health checks
- **API Gateway:** Rate limiting (10/hour anonymous, 100/hour auth)
- **Write Service:** Paste creation, URL generation, storage
- **Read Service:** Paste retrieval, cache-aside pattern
- **URL Generator:** Counter-based, Base62 encoding, distributed ranges
- **Object Storage:** S3 for content >1KB, database for <1KB
- **Cache:** Redis cluster, LRU eviction, 80% hit rate
- **Database:** PostgreSQL master + 5 read replicas

## 💾 Data Model (Essentials)

### Main Table: pastes

```text
- paste_id (PK, VARCHAR(7)) - Base62 encoded
- content_url (VARCHAR) - S3 key or 'db:paste_id'
- expires_at (TIMESTAMP, nullable)
- visibility (ENUM: public/private/unlisted)
- access_key (VARCHAR(64), nullable)
- view_count (BIGINT)
- size_bytes (INTEGER)
- created_at (TIMESTAMP)
```

**Key Indexes:**

- `idx_expires_at` - For expiration cleanup
- `idx_creator_ip` - For rate limiting

**Partitioning:** Shard by paste_id hash (future scaling)

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose | Key Features |
|--------|------|---------|--------------|
| POST | `/v1/pastes` | Create paste | Rate limited, size validation |
| GET | `/v1/pastes/{id}` | Retrieve paste | Cache-first, expiration check |
| DELETE | `/v1/pastes/{id}` | Delete paste | Access key required |
| GET | `/v1/health` | Health check | Service status monitoring |

## 🔄 Data Flows (Critical for Interviews)

### Write Flow (Create Paste)

```text
1. User → CDN → Load Balancer → API Gateway
2. API Gateway → Write Service (rate limit check)
3. Write Service → URL Generator (get unique ID)
4. Write Service → Object Storage (store content if >1KB)
5. Write Service → PostgreSQL (store metadata)
6. Write Service → Redis Cache (cache hot paste)
7. Response → User (return short URL)
```

**Key Points:** Synchronous for immediate URL, async for stats

### Read Flow (View Paste)

```text
1. User → CDN (95% cache hit for static assets)
2. CDN miss → Load Balancer → API Gateway
3. API Gateway → Read Service
4. Read Service → Redis Cache (80% hit rate)
5. Cache miss → PostgreSQL (metadata) + S3 (content)
6. Update cache → Return to user
```

**Key Points:** Multi-tier caching, expiration check, view count async update

## 🚀 Critical Talking Points

### Point 1: URL Generation Strategy

- **What:** Counter-based with range partitioning using Base62 encoding
- **Why:** Guaranteed uniqueness, no collision handling, predictable performance
- **Detail:** Each service pre-allocates 1000 IDs from Redis counter, converts to 7-char Base62
- **Alternative:** Hash-based (collision risk) or UUID (too long for short URLs)

### Point 2: Storage Architecture Decision

- **What:** Hybrid storage - small pastes (<1KB) in database, large pastes in S3
- **Why:** Cost optimization - S3 is 10x cheaper for large files, database faster for small
- **Detail:** 40% of pastes <1KB stay in DB, 60% go to S3 with metadata in DB
- **Alternative:** Database-only (expensive) or S3-only (slower for small files)

### Point 3: Expiration Handling

- **What:** Hybrid approach - lazy deletion on read + active cleanup worker
- **Why:** Immediate 404 for users, efficient batch cleanup for storage costs
- **Detail:** Read-time expiration check + 15-minute cleanup job deleting 10K expired/batch
- **Alternative:** TTL-only (delayed cleanup) or active-only (higher DB load)

### Point 4: Caching Strategy

- **What:** Multi-tier caching with cache-aside pattern
- **Why:** 80/20 rule - 20% of pastes generate 80% of traffic, need sub-50ms reads
- **Detail:** Redis cache with dynamic TTL (1h recent, 24h popular, 10min others)
- **Alternative:** Write-through (complex invalidation) or no cache (high DB load)

### Point 5: Rate Limiting Implementation

- **What:** Token bucket algorithm with Redis counters
- **Why:** Prevent spam/abuse while allowing burst traffic for legitimate users
- **Detail:** 10 tokens/hour for anonymous, 100/hour for authenticated, per-IP tracking
- **Alternative:** Fixed window (burst issues) or sliding window (more complex)

### Point 6: Private Paste Security

- **What:** Access key approach (secret URL) instead of user authentication
- **Why:** MVP simplicity - no user accounts needed, easy sharing via URL
- **Detail:** 32-byte random access key, stored as hash, included in URL query param
- **Alternative:** OAuth (complex for MVP) or IP whitelist (inflexible)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Database | PostgreSQL | DynamoDB | Simple queries, strong consistency, lower ops complexity |
| Syntax Highlighting | Client-side | Server-side | Zero server CPU, faster response, 99% have JavaScript |
| ID Generation | Counter-based | Hash-based | Guaranteed unique, no collisions, predictable latency |
| Content Storage | Hybrid (DB+S3) | S3-only | Cost optimization for small files, performance for large |
| View Counting | Async updates | Sync updates | Lower read latency, eventual consistency acceptable |

## 🔥 Bottlenecks & Solutions

### Bottleneck 1: Database Write Contention

- **Problem:** Single master handling 350 writes/sec at peak
- **Solution:** Write-through cache + async DB flush, reduces write latency by 80%

### Bottleneck 2: Object Storage Latency

- **Problem:** S3 GET requests average 50-100ms
- **Solution:** CloudFront CDN with 95% hit rate, reduces to <10ms for hot content

### Bottleneck 3: URL Generator SPOF

- **Problem:** Single point of failure for paste creation
- **Solution:** Multiple instances with pre-allocated ID ranges, Redis failover

### Bottleneck 4: Cache Memory Limits

- **Problem:** Redis memory constraints with growing dataset
- **Solution:** LRU eviction + intelligent TTL based on access patterns

## 💡 Interview Tips

### Start Here

- Begin with requirements clarification (scale, consistency needs, features)
- Draw simple box diagram first, then add details
- Mention read-heavy nature (100:1 ratio) early

### Emphasize

- **Scalability:** Horizontal scaling at every layer (stateless services, read replicas, sharding)
- **Performance:** Multi-tier caching, CDN, optimized storage decisions
- **Cost Efficiency:** Hybrid storage approach saves 60% on storage costs
- **Reliability:** Redundancy, health checks, graceful degradation

### Be Ready For

- "How do you handle hot pastes?" → Multi-tier caching with dynamic TTL
- "What if Redis goes down?" → Graceful degradation, cache rebuilds from DB
- "How do you prevent spam?" → Multi-layer rate limiting + CAPTCHA triggers
- "How do you scale writes?" → Database sharding by paste_id hash
- "What about data consistency?" → Eventual consistency for views, strong for creation

### Don't Forget

- **Monitoring:** Mention key metrics (latency p99, cache hit rate, error rate)
- **Security:** Rate limiting, input validation, access control for private pastes
- **Operational:** Health checks, logging, alerting, deployment strategy
- **Future:** User accounts, collaboration features, geographic distribution

## 🔒 Security Deep Dive

### Multi-Layer Protection

```text
Layer 1: CDN (CloudFlare)
- WAF, 1000 req/min per IP, bot challenges

Layer 2: API Gateway
- 100 req/min per IP, payload limits

Layer 3: Application
- Token bucket: 10 pastes/hour (anon), 100/hour (auth)
- Content validation: 10MB max, XSS prevention
```

### Private Paste Security

- **Access Keys:** 32-byte random, stored as hash
- **URL Format:** `https://pastebin.com/aB3xY7z?key=sk_a1b2c3d4e5f6g7h8`
- **No User Auth:** MVP simplicity, upgrade to OAuth later

## 📊 Monitoring & Operations

### Critical Metrics

| Metric | Threshold | Action |
|--------|-----------|--------|
| API Latency p99 | >200ms | Scale servers |
| Cache Hit Rate | <75% | Tune TTL/memory |
| Error Rate | >1% | Page on-call |
| DB Replication Lag | >60s | Check DB health |

### Alerting Strategy

- **Critical:** Page immediately (availability, errors)
- **Warning:** Email (performance degradation)
- **Info:** Dashboard (trends, costs)

## 🌍 Scaling Strategies

### Geographic Distribution

```text
Current: Single region (US-East)
Future: Multi-region deployment

Regions: US-East, EU-West, Asia-Pacific
- Each region: Full stack (API, DB, Cache)
- S3 cross-region replication
- GeoDNS routing to nearest region
- Cost: +200% infrastructure, +150% performance
```

### Database Scaling

```text
Current: 1 master + 5 read replicas
Bottleneck: 350 writes/sec at single master

Solutions:
1. Write sharding by paste_id hash
2. Write-through cache with async DB flush
3. Separate analytics DB (ClickHouse)
```

### Cache Scaling

```text
Current: Redis cluster (6 nodes, 64GB each)
Future: Multi-tier caching

Tier 1: Application cache (1000 hottest, <1ms)
Tier 2: Redis distributed (current, <10ms)
Tier 3: CDN edge cache (<20ms)
Result: 99% hit rate, <10ms average
```

## 🚨 Failure Scenarios & Recovery

### URL Generator Failure

- **Problem:** No new pastes can be created
- **Solution:** Pre-allocated ID ranges per instance
- **Fallback:** Timestamp-based IDs (collision risk)

### Database Failure

- **Master Down:** Promote read replica (30s downtime)
- **Replica Down:** Route reads to other replicas
- **Full DB Down:** Serve from cache only (degraded mode)

### Cache Failure

- **Redis Down:** Direct DB queries (higher latency)
- **Partial Failure:** Consistent hashing redistributes load
- **Recovery:** Warm cache from DB access patterns

### S3 Failure

- **Regional Outage:** Serve from cross-region replica
- **Complete Failure:** Serve metadata only, show "content unavailable"

---

**Total Infrastructure Cost:** ~$13K/month for 10M pastes/day
**Cost per paste:** $0.000043 (4.3 cents per 1000 pastes)
