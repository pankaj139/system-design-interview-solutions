# LEETCODE SYSTEM DESIGN - Interview Quick Reference

**File Purpose:** Concise interview-day reference guide for LeetCode-like online coding platform. Covers all critical talking points, architectural decisions, and key numbers. Review in 5-10 minutes before interviews.

**Last Updated:** October 2, 2025

---

## 🎯 Core Problem Statement

- **What:** Build a scalable online coding platform where users solve programming problems and get real-time execution feedback
- **Key Challenge:** Securely execute untrusted user code in isolated environments while maintaining performance and scalability
- **Scale:** 100K DAU, 500K submissions/day, 10M registered users, 3K problems

---

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| **Daily Active Users** | 100,000 | Given assumption |
| **Submissions/Day** | 500,000 | 100K users × 5 submissions |
| **Avg Submission QPS** | 6 | 500K / 86,400 seconds |
| **Peak Submission QPS** | 18 | 3x average (peak hours) |
| **Problem Views/Day** | 1,000,000 | 100K × 10 views |
| **Peak Problem QPS** | 36 | 3x average |
| **Code Executions/Day** | 1,000,000 | Submissions × 2 (test runs) |
| **Peak Execution QPS** | 36 | 3x average |
| **Concurrent Executions** | 108 | Peak QPS × 3s avg time |
| **Judge Workers Needed** | 250 | 108 × 2x buffer |
| **Storage (3 years)** | 1.5 TB | 500K × 2.5KB × 365 × 3 |
| **Storage (5 years)** | 2.5 TB | Submissions dominate |
| **Code Execution Timeout** | 10 seconds | Max per submission |
| **Avg Execution Time** | 3 seconds | Typical problem |
| **Memory per Execution** | 256 MB | Resource limit |
| **Worker Memory** | 512 MB | 256 MB + overhead |
| **Total Cluster Resources** | 250 vCPU, 125 GB | Worker pool |

---

## 🏗️ High-Level Architecture

```text
[User/Browser]
      |
      v
[CDN (CloudFront)] ---> Static Assets
      |
      v
[Load Balancer (AWS ELB)]
      |
      v
[API Servers (Node.js)] <---> [Redis Cache]
      |                              |
      v                              |
[Service Layer]                      |
  ├── Problem Service (Go)           |
  ├── User Service (Go)              |
  ├── Submission Service (Go)        |
  └── Judge Coordinator (Go)         |
      |                              |
      v                              v
[Message Queue (RabbitMQ/Kafka)] <---+
      |
      v
[Judge Workers (Docker)] ---> [Test Cases (S3)]
      |
      v
[Databases (PostgreSQL)]
  ├── Problem DB (unsharded)
  ├── User DB (shard by user_id)
  └── Submission DB (shard by user_id, partition by date)
      |
[Code Storage (S3)]
```

### Key Components

- **CDN:** Serves static assets globally, reduces server load
- **Load Balancer:** Distributes traffic, health checks, SSL termination
- **API Servers:** Stateless Node.js/Express, horizontally scaled, handles HTTP requests
- **Problem Service:** Manages CRUD operations, caches aggressively (1-hour TTL)
- **User Service:** Authentication (JWT), profiles, statistics
- **Submission Service:** Receives code, stores in S3, enqueues execution jobs
- **Redis Cache:** Problems (1h), sessions (15m), rate limits, user stats (10m), submission status
- **Message Queue:** RabbitMQ/Kafka for async execution, decouples API from workers, durable queues
- **Judge Workers:** Docker containers, execute code in sandboxed environment, report results
- **Judge Coordinator:** Monitors queue depth, auto-scales workers, health checks
- **PostgreSQL:** ACID transactions, complex queries for rankings/stats, sharded by user_id
- **S3:** Stores code files (2KB avg) and test cases (1KB avg), cost-effective blob storage

---

## 💾 Data Model (Essentials)

### Core Tables

```text
users
- user_id (PK, UUID)
- username, email (UNIQUE)
- password_hash (bcrypt)
- created_at, role
Indexes: username, email, created_at

problems
- problem_id (PK, UUID)
- title, slug (UNIQUE)
- description, difficulty (ENUM: easy/medium/hard)
- acceptance_rate, total_submissions
Indexes: slug, difficulty, acceptance_rate

submissions
- submission_id (PK, UUID)
- user_id, problem_id (FK)
- code_s3_key (VARCHAR)
- status (ENUM: pending, running, accepted, wrong_answer, TLE, MLE, runtime_error)
- runtime_ms, memory_kb
- test_cases_passed, total_test_cases
- submitted_at, judged_at
Indexes: (user_id, problem_id, submitted_at DESC), (user_id, submitted_at DESC), status

test_cases
- test_case_id (PK, UUID)
- problem_id (FK)
- input_s3_key, expected_output_s3_key
- is_sample, is_hidden
- time_limit_ms (3000), memory_limit_kb (262144)
Indexes: (problem_id, order_index), (problem_id, is_sample)

user_statistics
- user_id (PK, FK)
- problems_solved, easy_solved, medium_solved, hard_solved
- total_submissions, acceptance_rate, ranking
Indexes: ranking, problems_solved DESC
```

### Sharding Strategy

- **Users DB:** Shard by user_id (consistent hashing), 10M users
- **Problems DB:** Unsharded (only 3K problems, read-heavy)
- **Submissions DB:** Shard by user_id (queries are user-specific), partition by date (time-series)
- **Test Cases:** Store in S3, reference in DB, cached on workers

---

## 🔌 API Design (Key Endpoints)

```http
POST /v1/auth/login
POST /v1/auth/register
POST /v1/auth/refresh

GET /v1/problems?difficulty=easy&tags=array&page=1&limit=20
GET /v1/problems/{slug}

POST /v1/submissions
POST /v1/problems/{slug}/run
GET /v1/submissions/{submission_id}
GET /v1/users/{username}/submissions

GET /v1/users/{username}
GET /v1/users/{username}/statistics
```

### Key APIs

1. **Submit Code:** `POST /v1/submissions` → Returns 202 Accepted with submission_id, queues job
2. **Get Status:** `GET /v1/submissions/{id}` → Polls every 1-2s, returns status/results
3. **Run Test:** `POST /v1/problems/{slug}/run` → Quick test against sample cases (30/min limit)
4. **List Problems:** `GET /v1/problems` → Cached 5 min, paginated, filterable

### Rate Limits

- Anonymous: 20 req/min
- Authenticated: 100 req/min
- Code submissions: 10/min per user
- Test runs: 30/min per user

### Authentication

- JWT tokens: Access (15 min), Refresh (7 days)
- Bearer token in Authorization header

---

## 🚀 Critical Talking Points

### 1. Judge System Architecture

- **What:** Isolated code execution engine using Docker containers with gVisor sandboxing
- **Why:** Security is paramount - untrusted code must not compromise system or access data
- **Detail:** Each execution gets 1 vCPU, 256MB RAM, 10s timeout, no network, read-only filesystem except /tmp, seccomp filters limit syscalls, non-root user
- **Alternative:** VMs (stronger isolation, 10x slower startup), WASM (ultra-safe, limited language support), bare metal with chroot (fast, weak isolation)
- **Scaling:** 250 worker pool, auto-scale on queue depth >200, pre-warm containers, separate pools per language (optional)

### 2. Asynchronous Execution Pattern

- **What:** Queue-based async job processing via RabbitMQ/Kafka
- **Why:** Code execution takes 3-10s - synchronous blocks API threads and degrades UX
- **Detail:** Submit returns 202 immediately, job queued with durability, workers pull jobs, clients poll every 1-2s for results
- **Alternative:** Synchronous (simple, poor UX), WebSocket (real-time, higher server load, better UX)
- **Trade-off:** Polling adds latency but simpler than WebSocket connections; can upgrade to WebSocket for premium users

### 3. Multi-Layer Caching Strategy

- **What:** Redis cache at multiple levels with different TTLs and strategies
- **Why:** Reduce DB load (problems are read 80% of time), fast session validation, efficient rate limiting
- **Detail:**
  - Problems: Cache-aside, 1h TTL, invalidate on update
  - Problem lists: 5min TTL, include filters in key
  - Sessions: Write-through, 15min TTL (token lifetime)
  - User stats: Lazy update, 10min TTL
  - Submission status: Write-through, 1h TTL
  - Rate limits: Sliding window, 1min TTL
- **Alternative:** No cache (high DB load), longer TTLs (stale data), CDN caching (for GET endpoints)
- **Cache Stampede Solution:** Probabilistic early expiration, request coalescing, cache warming

### 4. Database Sharding & Partitioning

- **What:** Submissions sharded by user_id, partitioned by date (monthly)
- **Why:** Submissions grow unbounded (500K/day = 182M/year), single DB can't handle write load
- **Detail:** Consistent hashing for shard selection, most queries are user-specific (user submissions, stats), partition by date enables efficient purging of old data
- **Alternative:** NoSQL like Cassandra (better write throughput, weaker consistency, no complex queries for rankings)
- **Trade-off:** Cross-shard queries (global leaderboards) are expensive - solve with materialized views or separate analytics DB

### 5. Security Sandboxing

- **What:** Defense-in-depth: Docker + gVisor + seccomp + resource limits + network isolation
- **Why:** Malicious code could execute fork bombs, crypto miners, data exfiltration, privilege escalation
- **Detail:**
  - Container: Process isolation, PID namespace
  - gVisor: Syscall interception, kernel-level protection
  - Network: iptables DROP all outbound
  - Resources: cgroups enforce CPU/memory/disk quotas
  - Filesystem: Mount code as read-only, writable /tmp only
  - User: Run as unprivileged user, no sudo
- **Alternative:** Lambda functions (serverless, expensive at scale), dedicated VMs (slow), language-specific sandboxes (limited languages)
- **Monitoring:** Track syscall violations, resource limit hits, suspicious patterns

### 6. S3 for Code/Test Case Storage

- **What:** Store code files and test cases in S3, reference by key in DB
- **Why:** 2.5TB over 5 years, storing BLOBs in DB is expensive and degrades query performance
- **Detail:** Code: 2KB avg, test cases: 1KB avg, S3 is 5x cheaper than EBS, unlimited scalability, 99.999999999% durability
- **Alternative:** Database BLOB columns (expensive, slow), local filesystem (not scalable, no redundancy)
- **Optimization:** Workers cache test cases locally (immutable data), use CloudFront for test case distribution

---

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| **Execution Model** | Asynchronous (queue) | Synchronous | Decouples API from execution, handles spikes, better resource utilization; added latency acceptable |
| **Database** | PostgreSQL (SQL) | Cassandra (NoSQL) | Need ACID for stats updates, complex queries for rankings, 3NF schema; can shard for scale |
| **Code Storage** | S3 Object Storage | Database BLOBs | Cost-effective ($0.023/GB vs $0.10+), offloads DB, unlimited scale; slight latency increase |
| **Caching** | Redis (in-memory) | CDN only | Sub-ms latency, flexible TTLs, session storage, rate limiting; costs ~$100/month |
| **Sandboxing** | Docker + gVisor | AWS Lambda | Stronger isolation, cost-effective at scale ($50/mo vs $500/mo), flexible limits; more ops complexity |
| **Status Updates** | Polling (1-2s) | WebSocket | Simpler implementation, stateless API, HTTP caching; 1-2s latency acceptable for MVP |
| **API Style** | REST | GraphQL | Standard, cacheable, simple, well-understood; slight over-fetching acceptable |
| **Pagination** | Offset (problems), Cursor (submissions) | One approach | Offset for stable small data, cursor for time-series; mixed adds complexity but optimal |
| **Architecture** | Microservices | Monolith | Independent scaling (workers need 10x API), fault isolation, technology diversity; ops overhead |

---

## 🔥 Bottlenecks & Solutions

### 1. Judge Worker Capacity

- **Problem:** Queue grows faster than workers process during contests/peak hours
- **Solution:** Auto-scale workers based on queue depth (target <100, scale up at >200 for 2min), priority queues for premium users, pre-warm containers, increase pool during known peaks
- **Metric:** Queue depth, avg wait time, worker utilization
- **Alert:** Wait time >10s

### 2. Submission DB Write Contention

- **Problem:** 36 QPS peak writes with index updates cause lock contention
- **Solution:** Shard by user_id (most queries user-specific), minimize indexes (only critical ones), batch stat updates, read replicas for history queries
- **Metric:** Write latency p95/p99, lock wait time
- **Alert:** p95 write latency >100ms

### 3. Cache Stampede on Hot Problems

- **Problem:** Popular problem cache expires → multiple requests hit DB simultaneously
- **Solution:** Probabilistic early expiration, request coalescing (dedupe concurrent requests), proactive cache warming, longer TTL for popular items
- **Metric:** Cache hit rate, DB query spike
- **Alert:** Cache hit rate <85%

### 4. Single Point of Failure - Message Queue

- **Problem:** Queue down = no submissions processed
- **Solution:** RabbitMQ cluster (3 nodes) or Kafka with replication factor 3, fallback to DB-backed queue, continuous health checks with auto-failover
- **Metric:** Queue availability, consumer lag
- **Alert:** Queue unavailable

---

## 💡 Interview Tips

### Start Here

1. **Clarify requirements:** Scale (DAU, submissions/day), languages supported, security needs, latency targets
2. **Draw high-level architecture:** Client → LB → API → Queue → Workers → DB, emphasize async execution
3. **Dive into judge system:** This is the unique, complex component - spend 30% of time here

### Emphasize

- **Security:** Sandboxing strategy, defense-in-depth, no network access, resource limits
- **Scalability:** Stateless services, horizontal scaling, sharding, queue-based decoupling
- **Performance:** Multi-layer caching, S3 for blobs, read replicas, async execution
- **Trade-offs:** Always explain "why" - sync vs async, SQL vs NoSQL, polling vs WebSocket

### Be Ready For

- "How do you prevent malicious code?" → Sandbox layers (Docker, gVisor, seccomp, resource limits)
- "What if workers crash?" → Queue acknowledgment, job returns to queue, retry with backoff
- "How to scale to 10M DAU?" → Geo-distribution, more shards, CDN, Redis cluster, worker auto-scaling
- "How to handle contests (10K concurrent)?" → Pre-scale workers, priority queues, rate limiting, optimized caching
- "Database schema details?" → Focus on submissions table (most important), indexes, sharding key
- "Why not serverless (Lambda)?" → Cost at scale ($500/mo vs $50/mo), cold starts, execution time limits

### Don't Forget

- **Numbers:** Always justify with calculations (show QPS, storage, memory estimates)
- **Monitoring:** Mention key metrics (queue depth, execution time, error rate, cache hit rate)
- **Failure handling:** Worker crashes, timeout, OOM, network failures
- **Future enhancements:** WebSocket for real-time, multi-region, contests, AI hints
- **Cache invalidation:** Problem updates, stat changes, TTL strategy
- **Rate limiting:** Prevent abuse, different tiers, submission limits

### Common Mistakes to Avoid

- ❌ Skipping security discussion (most critical aspect)
- ❌ Not explaining async execution necessity
- ❌ Forgetting to shard submissions table
- ❌ Missing caching strategy
- ❌ Ignoring monitoring/alerting
- ❌ Not considering failure scenarios

---

## 🎓 Quick Reference Commands

### For Deep Dive

- **Judge System:** Docker + gVisor, 256MB, 10s timeout, seccomp filters, no network
- **Queue:** RabbitMQ cluster or Kafka (replication factor 3), durable messages
- **Cache:** Redis cluster, cache-aside (problems), write-through (sessions), TTL 1h/15m/10m
- **Sharding:** user_id (consistent hashing), partition submissions by date
- **Scaling:** 250 workers → auto-scale on queue depth, API horizontal scaling, DB read replicas

### Key Metrics

- **Latency:** API p95 <500ms, execution avg 3s, queue wait <5s
- **Throughput:** 36 QPS peak submissions, 108 concurrent executions
- **Availability:** 99.9% uptime target
- **Cache:** 85%+ hit rate target
- **Error Rate:** <2% target

---

**🚀 Good luck with your interview! Remember: Focus on security, scalability, and trade-offs.**

---

**Document Stats:** 450 lines | 5-min read | Updated Oct 2, 2025
