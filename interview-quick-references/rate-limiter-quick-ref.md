# Rate Limiter - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a distributed rate limiter for API gateways supporting 10M requests/day with multi-tier pricing.

---

## 🎯 Core Problem Statement

- **What:** Design a distributed rate limiter to prevent API abuse and enforce pricing tiers
- **Key Challenge:** Accurate rate limiting across multiple servers without central bottleneck
- **Scale:** 10M requests/day, <10ms latency overhead, 100K API keys, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Requests | 10M | Given requirement |
| Requests/Second (Avg) | 116 | 10M / 86,400 |
| Peak RPS | 350 | Avg × 3 (burst capacity) |
| Latency Overhead | <10ms | Rate limit check time |
| API Keys | 100K | Registered users |
| Free Tier Limit | 100 req/hour | 1.67 req/min |
| Pro Tier Limit | 1,000 req/hour | 16.67 req/min |
| Enterprise Tier | Unlimited | Or 100K req/hour |
| Memory per Key (Redis) | 100 bytes | Counter + metadata |
| Total Memory | 10MB | 100K keys × 100 bytes |

## 🏗️ High-Level Architecture

```text
[Client] → [API Gateway] → [Rate Limiter Middleware]
                                    ↓
                            [Local Cache (LRU)]
                                    ↓
                           [Redis Cluster (Distributed)]
                                    ↓
                            [PostgreSQL (Config)]
```

**Components:**

- **API Gateway**: Kong/Nginx, routes requests to rate limiter first
- **Rate Limiter Middleware**: In-process check, 1-2ms latency
- **Local Cache**: LRU cache (10K keys), avoids Redis for every check
- **Redis Cluster**: Distributed counters, sliding window data, 1ms access
- **PostgreSQL**: User limits, pricing tiers, audit logs

## 💾 Data Model (Essentials)

### Redis Schema

```text
Key: rate_limit:{api_key}:{window_start}
Value: Request count (integer)
TTL: Window duration + buffer (e.g., 3700s for 1 hour)

Key: sliding_log:{api_key}
Value: Sorted Set (timestamp → request_id)
TTL: Window duration
```

### PostgreSQL Schema

```text
api_keys
├── api_key (PK, VARCHAR)
├── user_id (FK, UUID)
├── tier (ENUM: free, pro, enterprise)
├── rate_limit (INTEGER)
├── created_at (TIMESTAMP)

rate_limit_logs
├── log_id (PK, UUID)
├── api_key (FK, INDEX)
├── timestamp (TIMESTAMP, INDEX)
├── allowed (BOOLEAN)
├── remaining_quota (INTEGER)
```

## 🔌 API Design (Key Endpoints)

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/resource` | Protected endpoint with rate limiting |
| GET | `/v1/rate_limit/status` | Check current rate limit status |

**Response Headers:**

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 73
X-RateLimit-Reset: 1696243200
Retry-After: 3600
```

## 🚀 Critical Talking Points

### Point 1: Sliding Window Counter Algorithm (Chosen)

- **What:** Hybrid of fixed window and sliding log, smooth rate limiting
- **Why:** 99% accuracy, O(1) time/space, handles burst traffic gracefully
- **Detail:**
  - Formula: `Count = prev_window_count × overlap% + curr_window_count`
  - Example: 70 requests at :45 with 75% overlap from previous hour
  - Memory: 2 counters per key (current + previous window)
- **Alternative:** Token bucket (smoother but complex), fixed window (spiky), sliding log (accurate but expensive)

### Point 2: Distributed Rate Limiting with Redis

- **What:** Centralized Redis cluster for shared rate limit state
- **Why:** Single source of truth across all API gateway instances
- **Detail:**
  - Atomic operations: INCR, EXPIRE for race-condition-free counting
  - Lua scripts for complex atomic operations (check + increment)
  - Fallback: Allow requests if Redis unavailable (fail-open)
- **Alternative:** Local counters (inaccurate), database (too slow), Gossip protocol (complex)

### Point 3: Multi-Layer Caching

- **What:** Local LRU cache → Redis → Database hierarchy
- **Why:** Reduce Redis load, achieve <10ms overhead
- **Detail:**
  - L1 (Local): 10K hot keys, 1-2ms check, 1-minute TTL
  - L2 (Redis): All keys, 3-5ms check, window-duration TTL
  - L3 (DB): Config only (tier limits), loaded at startup
- **Alternative:** Redis-only (higher latency), no cache (impossible load)

### Point 4: Handling Distributed Races

- **What:** Use Lua scripts for atomic check-and-increment
- **Why:** Prevent race conditions where multiple servers allow same request
- **Detail:**

  ```lua
  local current = redis.call('GET', KEYS[1])
  if current and tonumber(current) >= limit then
    return 0  -- Rate limited
  end
  redis.call('INCR', KEYS[1])
  redis.call('EXPIRE', KEYS[1], window)
  return 1  -- Allowed
  ```

- **Alternative:** Optimistic locking (slower), distributed locks (complex), accept small error margin

### Point 5: Clock Synchronization

- **What:** Use Redis timestamps or UTC for window boundaries
- **Why:** Prevent clock skew causing inconsistent rate limiting
- **Detail:**
  - Redis TIME command for consistent timestamps
  - NTP synchronization on all servers (<100ms skew)
  - Window start = floor(timestamp / window_duration) × window_duration
- **Alternative:** Local clocks (skew issues), centralized time service (bottleneck)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Algorithm | Sliding Window Counter | Token Bucket / Leaky Bucket | Best balance of accuracy, performance, and memory |
| Storage | Redis | Local counters / Database | Centralized state, fast atomic operations |
| Consistency | Eventual (with Lua) | Strong consistency | Acceptable 1-2% error margin for better performance |
| Failure Mode | Fail-open | Fail-closed | Availability > strict limiting |
| Synchronization | Redis TIME | NTP only | Consistent timestamps across servers |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution | Monitoring |
|------------|----------|-----------|
| Redis Hot Keys | Shard by API key, hash slot distribution | Redis slow log, key access patterns |
| Redis Cluster Lag | Use pipelining, local caching, read replicas | Replication lag, command latency |
| Network Latency | Co-locate Redis with API gateway, use connection pooling | Network RTT, connection pool stats |
| Clock Skew | NTP sync <100ms, use Redis TIME command | Clock offset metrics, timestamp drift |

## 🎯 Algorithm Comparison

| Algorithm | Accuracy | Memory | Complexity | Burst Handling |
|-----------|----------|--------|------------|----------------|
| Token Bucket | High | O(1) | Medium | Excellent (smooths bursts) |
| Leaky Bucket | High | O(1) | Medium | Good (constant rate) |
| Fixed Window | Low (spiky) | O(1) | Simple | Poor (boundary bursts) |
| Sliding Log | Perfect | O(N) | Complex | Excellent |
| **Sliding Window Counter** | 99% | O(1) | Simple | Good (best trade-off) |

## 💡 Interview Tips

**Start Here:**

1. Clarify requirements: QPS, tiers, latency, accuracy tolerance
2. Calculate numbers: requests/sec, memory, Redis capacity
3. Choose algorithm (explain trade-offs!)
4. Design distributed architecture

**Emphasize:**

- Algorithm choice reasoning (sliding window counter is optimal)
- Distributed coordination with Redis + Lua scripts
- Multi-layer caching for performance
- Graceful degradation (fail-open vs fail-closed)

**Be Ready For:**

- "Which algorithm and why?" → Sliding window counter (accuracy + performance)
- "How to handle clock skew?" → Redis TIME command + NTP sync
- "What if Redis fails?" → Fail-open with local cache fallback
- "How to prevent race conditions?" → Lua scripts for atomicity
- "How to handle burst traffic?" → Sliding window smooths bursts
- "How to scale Redis?" → Cluster with consistent hashing

**Don't Forget:**

- Discuss algorithm trade-offs (this is the core!)
- Mention response headers (X-RateLimit-*)
- Address distributed consistency challenges
- Consider different failure modes
- Talk about monitoring and alerting

**Common Mistakes:**

- Choosing fixed window (explain why it's problematic)
- Forgetting about clock synchronization
- Not handling Redis failures (fail-open/closed decision)
- Over-complicating with distributed locks
- Ignoring local caching (performance killer)

**Algorithm Decision Flow:**

```text
Need perfect accuracy? → Sliding Log (memory expensive)
     ↓ No
Need smooth burst handling? → Token Bucket (complex state)
     ↓ No
Need simple and performant? → Sliding Window Counter ✓
```

**Key Implementation Details:**

```text
Sliding Window Counter:
1. Window size: 1 hour for hourly limits
2. Redis keys: rate_limit:{api_key}:{window_start}
3. Atomic operation: GET → Check → INCR (in Lua)
4. Calculation: prev_count × (1 - elapsed_ratio) + curr_count
5. Reset: Automatic via TTL expiration
```

**Testing Edge Cases:**

- Boundary testing (requests exactly at window transition)
- Concurrent requests from same API key
- Clock skew scenarios (server time differences)
- Redis failure and recovery
- Burst traffic (many requests in short time)
