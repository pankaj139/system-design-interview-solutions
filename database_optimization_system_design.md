# Understanding and Optimising Databases

**Difficulty Level:** ⭐⭐⭐⭐ Hard  
**Tags:** `Database Design`, `Distributed Systems`, `Indexing`, `Query Optimization`, `Caching`, `Sharding`, `Replication`, `High Availability`, `Performance`, `Scalability`

**File Purpose:** Interactive, multi-level learning resource for understanding and optimising databases at scale. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to design database layers that handle 1M+ QPS with <10ms p99 query latency, 99.99% availability, and petabyte-scale data across distributed systems.

**Author:** System Design Documentation  
**Created:** August 2026  
**Last Updated:** August 2026  
**Recent Updates:** Initial creation covering database internals, indexing, query optimization, sharding, replication, and production operations

---

## 🎓 Welcome to Database Understanding & Optimisation!

### What You're Going to Learn

Imagine being the engineer on call at 3 AM when your company's primary database is running at 100% CPU and queries are taking 30 seconds instead of 30 milliseconds. Thousands of users are experiencing timeouts, revenue is falling, and your on-call phone won't stop ringing.

**Would you know what to do?**

By the end of this learning journey, you'll be able to:
- Diagnose and fix slow queries that are killing production databases
- Design indexing strategies that make queries 1000x faster
- Scale databases to handle 1M+ queries per second
- Prevent catastrophic data loss with proven replication strategies
- Make informed decisions between SQL, NoSQL, and NewSQL databases
- Optimise databases for read-heavy, write-heavy, and mixed workloads
- Navigate FAANG-level database design interviews with confidence

### 📚 Your Learning Path

```text
🟢 BEGINNER LEVEL (5-7 hours)
├─ Learn what happens inside a database
├─ Understand indexes and why they matter
├─ Learn basic query tuning techniques
└─ Perfect for: Developers new to database performance

🟡 INTERMEDIATE LEVEL (7-10 hours)
├─ Master query execution plans
├─ Design optimal indexing strategies
├─ Understand sharding and replication trade-offs
└─ Perfect for: Engineers preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-15 hours)
├─ Database internals (B-trees, LSM trees, WAL)
├─ Distributed transaction protocols
├─ Production incident management
└─ Perfect for: Senior engineers and DBAs
```

### 🎯 Prerequisites

**For Beginners:** Basic SQL knowledge, familiarity with tables and rows, no database performance experience needed.

**For Intermediate:** Comfortable writing complex SQL queries, familiar with indexes conceptually, basic understanding of distributed systems.

**For Advanced:** Experience running databases in production, knowledge of CAP theorem, understanding of transaction isolation levels.

💡 **Pro Tip:** Even if you're advanced, read the beginner sections — analogies here will help you explain complex topics in interviews.

---

## 📋 Table of Contents

1. [Understanding the Problem Space](#section-1-understanding-the-problem-space)
2. [Capacity Planning & Performance Baselines](#section-2-capacity-planning--performance-baselines)
3. [Database Internals — How Storage Works](#section-3-database-internals--how-storage-works)
4. [Indexing Deep Dive](#section-4-indexing-deep-dive)
5. [Query Optimization](#section-5-query-optimization)
6. [Schema Design & Data Modelling](#section-6-schema-design--data-modelling)
7. [Replication & High Availability](#section-7-replication--high-availability)
8. [Sharding & Horizontal Scaling](#section-8-sharding--horizontal-scaling)
9. [Caching Strategies](#section-9-caching-strategies)
10. [Database Selection Guide](#section-10-database-selection-guide)
11. [Scalability & Performance Optimisation](#section-11-scalability--performance-optimisation)
12. [Security Considerations](#section-12-security-considerations)
13. [Monitoring & Observability](#section-13-monitoring--observability)
14. [Trade-Offs & Design Decisions](#section-14-trade-offs--design-decisions)
15. [Interview Preparation Framework](#section-15-interview-preparation-framework)
16. [Putting It All Together](#section-16-putting-it-all-together)

---

## Section 1: Understanding the Problem Space

### What You'll Learn
- Why databases become bottlenecks
- How to frame database problems in interviews
- Real-world consequences of poor database design

### Why This Matters

Databases are the **heart of almost every application**. When they fail or slow down, everything stops. Understanding databases deeply separates senior engineers from junior ones.

#### 🟢 For Beginners: What Is a Database, Really?

Think of a database like a **highly organised filing cabinet** in a giant library:

- The **cabinet** is the database server (hardware + software)
- Each **drawer** is a table (a collection of related records)
- Each **folder** is a row (one record, e.g., one user)
- Each **label** on a folder is a column (an attribute, e.g., `email`)
- An **index** is like the card catalogue — it lets you find folders without opening every drawer

Now imagine 10 million people all trying to find and update files in that cabinet simultaneously, 24 hours a day. That's the database optimisation problem.

**Common reasons databases become slow:**

| Problem | Analogy | Database Reality |
|---------|---------|-----------------|
| No index | Looking through every folder | Full table scan (O(n)) |
| Bad query | Asking for every folder, then filtering | `SELECT *` with WHERE in application |
| Missing hardware | Only one cabinet for 1M users | Single server saturation |
| Too many connections | 1000 people at one counter | Connection pool exhaustion |
| Locking | One person hogging a drawer | Long-running transactions |

#### 🟡 For Intermediate: Framing Database Problems in Interviews

When an interviewer says "design a system for X", your database decisions must address:

**The 5 Database Interview Questions:**
1. **What is the read/write ratio?** (10:1 read-heavy? 1:10 write-heavy?)
2. **What consistency do we need?** (Strong consistency vs. eventual consistency)
3. **What is the data model?** (Relational, document, graph, time-series?)
4. **What scale must we handle?** (QPS, storage, latency SLAs)
5. **What are the access patterns?** (Point lookups, range queries, aggregations?)

**Interview Script:**
> "Before choosing a database, I want to understand the access patterns. Are we primarily doing point lookups by user ID, or do we need complex range queries and aggregations? This will determine whether we choose a relational database with careful indexing, or a purpose-built NoSQL store. Could you tell me more about the query patterns?"

#### 🔴 For Advanced: The True Cost of Poor Database Design

In production, bad database design has direct business impact:

| Issue | Real-world Cost |
|-------|----------------|
| N+1 query problem | Amazon: 100ms extra latency = 1% revenue drop |
| Missing index on hot query | Twitter: 2013 outage caused by full table scan |
| Unoptimised schema | Airbnb: 10x storage cost reduction after normalisation |
| No connection pooling | Slack: 2019 incident — 70K connections exhausted PostgreSQL |
| Wrong isolation level | Financial systems: phantom reads causing double-charges |

**Database Failure Categories by Root Cause (industry data):**
- 40% — Slow/missing indexes
- 25% — Poor schema design
- 20% — Capacity (not scaled out in time)
- 10% — Configuration errors
- 5% — Software bugs

#### Think About It
- What's the read/write ratio of your current application?
- How many rows does your largest table have? Do all queries use indexes?

#### Key Takeaways
- Databases are the most common application bottleneck at scale
- Most performance problems come from indexing and query issues, not hardware
- Always understand access patterns before choosing or optimising a database

#### Practice Exercise
Take any SQL query in your current project. Use `EXPLAIN` to see its execution plan. Identify whether it uses an index. We'll deep-dive into this in Section 5.

---

## Section 2: Capacity Planning & Performance Baselines

### What You'll Learn
- How to calculate database capacity requirements
- Industry performance baselines to use in interviews
- Back-of-envelope formulas for database sizing

### Why This Matters

Over-provisioning costs money. Under-provisioning causes outages. Capacity planning is the skill that separates reactive engineers from proactive ones.

#### 🟢 For Beginners: Database Performance Intuition

**What limits database performance?**

Think of a database server as a kitchen:
- **CPU** = The chef (processes queries)
- **RAM** = The counter space (working memory for joins, sorts, indexes)
- **Disk I/O** = Reaching into the refrigerator (slowest operation)
- **Network** = Orders coming in and food going out

A slow dish (query) usually means too much refrigerator trips (disk reads) or the chef is too busy (CPU saturation).

**Order of magnitude performance guide (single server, SSD):**

| Operation | Approximate Latency |
|-----------|-------------------|
| Read 1 row by primary key (in cache) | 0.1ms |
| Read 1 row by primary key (from disk) | 1-2ms |
| Full table scan (1M rows) | 500ms - 5s |
| Write 1 row | 1-5ms |
| Read 1 row by secondary index | 2-10ms |
| Complex JOIN across 3 tables (1M rows each) | 100ms - 10s |

#### 🟡 For Intermediate: Back-of-Envelope Calculations

**Worked Example: E-commerce Platform Database Sizing**

Assumptions:
- 10M registered users
- 50M product listings
- 500M orders (historical)
- 10M daily active users (DAU)
- Average 20 queries per user session

**Traffic Estimation:**
```text
Daily queries  = 10M DAU × 20 queries/session = 200M queries/day
QPS average    = 200M / 86,400 seconds ≈ 2,300 QPS
QPS peak       = 2,300 × 3 (3x peak multiplier) ≈ 7,000 QPS

Write ratio = 5% (orders, cart updates, reviews)
Write QPS   = 7,000 × 0.05 = 350 writes/sec
Read QPS    = 7,000 × 0.95 = 6,650 reads/sec
```

**Storage Estimation:**
```text
Users table    = 10M × 500 bytes/row  = 5 GB
Products table = 50M × 2 KB/row       = 100 GB
Orders table   = 500M × 1 KB/row      = 500 GB
Order items    = 500M × 3 items × 200 bytes = 300 GB
Indexes        = ~50% of raw data overhead = 450 GB
Total          ≈ 1.4 TB (primary data)
With 3x replication = 4.2 TB
```

**Server Sizing:**
```text
A well-tuned PostgreSQL on 32-core server with NVMe SSD can handle:
- Read QPS: ~50,000 (simple indexed queries)
- Write QPS: ~10,000
- RAM needed: 2x working set (hot data) = ~200 GB RAM ideal
- For 6,650 reads/sec → 1 primary + 2 read replicas is sufficient
```

**Key capacity planning formulas:**

```text
QPS required   = (DAU × queries_per_user) / 86,400 × peak_multiplier
Storage (GB)   = rows × avg_row_size_bytes / 1,000,000,000 × replication_factor
RAM target     = hot_data_size × 1.5 (for buffer pool)
Connections    = app_servers × threads_per_server × connection_multiplier
```

#### 🔴 For Advanced: Production Capacity Modelling

**The Database Performance Triangle:**

```text
        Throughput (QPS)
              /\
             /  \
            /    \
           /      \
          /________\
   Latency            Concurrency

You can optimise for 2 of 3 simultaneously.
Adding concurrency without hardware → latency degrades.
Reducing latency often reduces throughput.
```

**Working Set Estimation (critical for caching/RAM sizing):**

The 80/20 rule: 80% of queries hit 20% of data.

```text
Working set = total_data × 0.20
RAM needed  = working_set × 1.5 (buffer pool + overhead)

Example: 1 TB database
Working set = 200 GB
RAM needed  = 300 GB minimum for good cache hit ratio
```

**Connection Pool Sizing Formula:**

```
connections = (core_count * 2) + effective_spindle_count
```

PostgreSQL recommends: If you have 8 CPU cores and SSD (1 spindle):
```
connections = (8 * 2) + 1 = 17 connections per application tier
```

📊 **Example — Instagram's Database Scale:**
- Peak QPS: 500,000+ reads, 50,000+ writes
- Storage: 400+ TB of PostgreSQL data
- Approach: 100+ PostgreSQL shards, each with 1 primary + 2 replicas
- Cache: Redis cluster absorbs 95%+ of reads before hitting DB

#### Think About It
- What's the current QPS of your production database? Do you know?
- Is your RAM large enough to hold your working set?

#### Key Takeaways
- Most web applications need far less database power than engineers think
- Working set caching is the highest-leverage optimisation
- Always calculate before adding hardware — often the answer is an index, not a server

#### Practice Exercise
Estimate the QPS, storage, and RAM requirements for a Twitter-like system with 100M DAU and 500M tweets per day. Compare your answer against Twitter's actual architecture.

---

## Section 3: Database Internals — How Storage Works

### What You'll Learn
- How B-trees (PostgreSQL, MySQL) and LSM trees (Cassandra, RocksDB) work
- What the Write-Ahead Log (WAL) is and why it matters
- How MVCC enables concurrent access without locking

### Why This Matters

Understanding internals lets you predict performance, avoid surprises, and make confident claims in senior-level interviews. "I know this will be fast because it's a B-tree range scan" signals senior engineering depth.

#### 🟢 For Beginners: How Data Gets Stored

**The Library Catalogue Analogy:**

When you save a row to a database, it doesn't just "go in". There's a whole process:

1. **The Intake Desk (Write-Ahead Log / WAL):** Every change is first written to a journal (the WAL) — like a librarian writing in a log before they reshelve anything. This protects against crashes.

2. **Working Memory (Buffer Pool):** Frequently accessed pages are kept in RAM so the librarian doesn't have to go to the stacks (disk) every time.

3. **The Stacks (Data Files):** The actual organised storage on disk, arranged in a tree structure for fast lookup.

4. **The Card Catalogue (Indexes):** Separate data structures that point to where records live, so searches are fast.

**Two types of database storage engines:**

| Feature | B-Tree (PostgreSQL, MySQL) | LSM Tree (Cassandra, RocksDB) |
|---------|--------------------------|-------------------------------|
| Analogy | Sorted binder — fast reads | Append-only log — fast writes |
| Read speed | Very fast | Slower (may merge multiple files) |
| Write speed | Moderate | Very fast |
| Best for | Read-heavy OLTP | Write-heavy time-series |
| Space amplification | Low | High (compaction needed) |
| Examples | PostgreSQL, MySQL, SQLite | Cassandra, RocksDB, LevelDB |

#### 🟡 For Intermediate: B-Trees — The Foundation of Relational Databases

**B-Tree Structure:**

```text
                    [50 | 100]                    ← Root node (in memory)
                   /    |    \
           [10|30]  [60|80]  [110|150]            ← Internal nodes
          /  |  \   / | \    /  |  \
        [5] [20] [40] [55] [70] [95] [120]...    ← Leaf nodes (data pointers)
```

**Why B-Trees are fast for reads:**
- Balanced: every path root → leaf is the same length
- Branching factor ~100-1000: a 1TB database needs only 3-4 node reads (disk I/Os) to find any row
- Leaf nodes form a doubly linked list → range scans are sequential reads

**B-Tree performance characteristics:**

| Operation | Complexity | Disk I/Os |
|-----------|-----------|-----------|
| Point lookup by indexed column | O(log N) | 3-4 |
| Range scan on indexed column | O(log N + K) | 3-4 + K/page |
| Insert (no index) | O(1) | 1-2 |
| Insert (with index update) | O(log N) | 4-8 |
| Full table scan | O(N) | N/page_size |

**Why inserts are expensive with many indexes:**

Each additional index means an additional B-tree write on every INSERT/UPDATE/DELETE. A table with 10 indexes costs ~10x more to write than a table with 1 index.

```sql
-- This table has 4 indexes — every INSERT writes to 5 B-trees
CREATE TABLE orders (
    id         BIGSERIAL PRIMARY KEY,          -- index 1
    user_id    BIGINT REFERENCES users(id),    -- index 2
    status     VARCHAR(20),                    -- index 3
    created_at TIMESTAMP DEFAULT NOW(),        -- index 4
    total_amount DECIMAL(10,2)
);
CREATE INDEX idx_orders_user_id    ON orders(user_id);
CREATE INDEX idx_orders_status     ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

#### 🔴 For Advanced: LSM Trees, WAL, and MVCC

**LSM Tree (Log-Structured Merge Tree) — How Cassandra Writes:**

```text
Write path:
  Client → Commit Log (WAL) → Memtable (RAM) → SSTables (Disk)
                                    ↓ (when full, flush)
                             [SSTable 1] [SSTable 2] [SSTable 3]
                                    ↓ (compaction)
                             [Merged SSTable]

Read path:
  Client → Memtable → [SSTable 1] → [SSTable 2] → [SSTable 3]
  (Bloom filters skip SSTables that can't contain the key)
```

**LSM vs B-Tree trade-off in numbers:**

| Metric | B-Tree (PostgreSQL) | LSM Tree (Cassandra) |
|--------|--------------------|--------------------|
| Write amplification | 5-10x | 2-4x during flush, 10-30x total |
| Read amplification | 1-4x | 1-100x (before compaction) |
| Space amplification | 1.1-1.3x | 1.3-3x |
| Peak write throughput | ~50K/s | ~200K/s (single node) |
| Compaction impact | None | CPU/IO spikes during compaction |

**Multi-Version Concurrency Control (MVCC):**

MVCC is how modern databases allow concurrent reads and writes without blocking each other:

```text
Transaction T1 begins (sees snapshot at time t=100)
                    ↓
    Row version history:
    [id=5, name="Alice", txn_id=99]  ← T1 sees this
    [id=5, name="Bob",   txn_id=101] ← T2's update, invisible to T1

T1 reads id=5 → gets "Alice" (from its snapshot)
T2 updates id=5 → creates new version "Bob" (txn_id=101)
T1 reads id=5 again → still gets "Alice" (REPEATABLE READ isolation)
T1 commits → snapshot discarded
VACUUM runs → old version [Alice, txn_id=99] is garbage collected
```

**Write-Ahead Log (WAL) — Durability Guarantee:**

```text
Client sends: INSERT INTO orders (user_id, total) VALUES (42, 99.99)

Step 1: Append to WAL on disk (fsync)  ← guarantees durability
Step 2: Update in-memory buffer pool
Step 3: Return success to client
Step 4 (async): Flush dirty buffer pages to actual data files

If server crashes AFTER step 1 but BEFORE step 4:
→ On restart, replay WAL from last checkpoint
→ Database is fully consistent
```

📊 **Example — How Netflix Handles Database Writes:**
- Primary storage: MySQL clusters with InnoDB (B-tree)
- For real-time data: Cassandra (LSM tree) for viewing history — writes 100K/s per region
- WAL shipped in near-real-time to S3 for point-in-time recovery

#### Think About It
- If your database has many frequent updates, which storage engine is better: B-tree or LSM tree?
- What happens to your data if the database process is killed while a write is in-progress?

#### Key Takeaways
- B-trees are optimal for read-heavy, mixed workloads with complex queries
- LSM trees excel at write-heavy, append-heavy workloads (logs, metrics, events)
- WAL ensures durability even without expensive synchronous disk flushes
- MVCC enables high concurrency by avoiding read-write locks

---

## Section 4: Indexing Deep Dive

### What You'll Learn
- How different index types work and when to use each
- Common indexing mistakes that kill performance
- How to design an indexing strategy from scratch

### Why This Matters

Indexes are the single highest-leverage database optimisation. A correctly placed index can turn a 30-second query into a 3-millisecond query. An incorrectly designed index table can slow writes by 10x.

#### 🟢 For Beginners: What Is an Index?

**The Phone Book Analogy:**

Imagine searching for "Smith, John" in a phone book:
- **Without index:** Read every page from A to Z — 1000 pages
- **With alphabetical index (B-tree):** Turn to "S", then "Sm", then "Smith" — 3 page turns

A database index works identically. Without it, the database must read every row (a "full table scan"). With it, it jumps directly to the right location.

**What an index looks like internally:**

```text
Table: users (1 million rows)
┌────────────────────────────────────────────┐
│ id  │ email              │ name    │ age  │
├────────────────────────────────────────────┤
│ 1   │ alice@example.com  │ Alice   │ 28   │
│ 2   │ bob@example.com    │ Bob     │ 35   │
│ 3   │ carol@example.com  │ Carol   │ 22   │
│ ... │ ...                │ ...     │ ...  │
└────────────────────────────────────────────┘

Index on email (B-tree):
alice@example.com → row 1
bob@example.com   → row 2
carol@example.com → row 3

Query: SELECT * FROM users WHERE email = 'bob@example.com'
→ Index lookup: find 'bob@example.com' → pointer to row 2
→ Fetch row 2: O(log N) instead of O(N) ✅
```

**The cost of indexes:**

| Benefit | Cost |
|---------|------|
| Reads are much faster | Writes are slower (maintain index) |
| Queries are more predictable | Storage increases (~50-100% overhead) |
| Enables sorting efficiently | Memory usage increases |

#### 🟡 For Intermediate: Index Types and When to Use Them

**B-Tree Index (default — use for most cases):**

```sql
-- Point lookups
SELECT * FROM users WHERE id = 42;

-- Range queries
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-03-31';

-- Prefix matches
SELECT * FROM users WHERE name LIKE 'John%';  -- works with B-tree
-- NOT: WHERE name LIKE '%John%'              -- full scan regardless
```

**Hash Index (equality-only, fastest for point lookups):**

```sql
-- Only for = comparisons, NOT for ranges or LIKE
SELECT * FROM sessions WHERE session_token = 'abc123xyz';

-- PostgreSQL: hash indexes are crash-safe since v10
CREATE INDEX idx_sessions_token ON sessions USING HASH (session_token);
```

**Composite (Multi-column) Index:**

The **column order matters**. Follow the "leftmost prefix rule":

```sql
-- Index on (user_id, status, created_at)
CREATE INDEX idx_orders_user_status_date ON orders(user_id, status, created_at);

-- This query uses the full index ✅
SELECT * FROM orders WHERE user_id = 42 AND status = 'PENDING' AND created_at > '2024-01-01';

-- This query uses the first 2 columns of the index ✅
SELECT * FROM orders WHERE user_id = 42 AND status = 'PENDING';

-- This uses only the first column ✅ (less efficient)
SELECT * FROM orders WHERE user_id = 42;

-- This DOES NOT use the index ❌ (skips user_id)
SELECT * FROM orders WHERE status = 'PENDING';
```

**Partial Index (index only a subset of rows):**

```sql
-- Only active users need fast lookups — index just them
CREATE INDEX idx_users_active_email ON users(email) WHERE is_active = true;

-- Only recent orders are accessed frequently
CREATE INDEX idx_recent_orders ON orders(created_at) WHERE created_at > '2024-01-01';
-- Much smaller index → fits in RAM → faster
```

**Covering Index (include all columns needed by a query):**

```sql
-- Query: SELECT status, total FROM orders WHERE user_id = 42
-- Without covering index: index lookup + heap fetch for each row
-- With covering index: single index-only scan
CREATE INDEX idx_orders_covering ON orders(user_id) INCLUDE (status, total_amount);
```

**Index Selection Decision Tree:**

```text
What kind of queries do you have?
│
├─ Exact match only (=)?
│   └─ Hash index (fastest) or B-tree
│
├─ Range queries (<, >, BETWEEN, ORDER BY)?
│   └─ B-tree index
│
├─ Full-text search (LIKE '%word%', natural language)?
│   └─ Full-text index (PostgreSQL GIN/GiST)
│
├─ Geospatial queries (nearest, within radius)?
│   └─ Spatial index (PostGIS, R-tree)
│
├─ JSON/array queries?
│   └─ GIN index (PostgreSQL)
│
└─ Multiple columns frequently queried together?
    └─ Composite index (column order: = filters first, then ranges)
```

#### 🔴 For Advanced: Index Internals, Pathologies, and Strategy

**Index Bloat and Maintenance:**

```sql
-- Check index bloat (PostgreSQL)
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan AS index_scans,
    idx_tup_read AS tuples_read
FROM pg_stat_user_indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- An index with idx_scan = 0 after 30+ days is a candidate for removal
-- Dead indexes consume write overhead with zero read benefit
```

**The Index Selectivity Rule:**

Index selectivity = distinct values / total rows

- High selectivity (e.g., `user_id`, `email`) → great index candidate
- Low selectivity (e.g., `status` with 3 values, `is_deleted`) → often not worth indexing alone
- Exception: partial indexes and covering indexes can make low-selectivity useful

```sql
-- Calculate selectivity
SELECT
    COUNT(DISTINCT status) * 1.0 / COUNT(*) AS selectivity
FROM orders;
-- 0.000003 → very low → B-tree index on status alone is useless
-- BUT:
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'PENDING';
-- Partial index on just pending orders → very selective ✅
```

**The N+1 Index Problem:**

```sql
-- Common ORM anti-pattern: fetches orders, then queries user for each
-- Query 1: SELECT * FROM orders LIMIT 100  → 100 orders
-- Query 2-101: SELECT * FROM users WHERE id = ?  → 100 separate queries

-- Solution: JOIN with proper indexes
SELECT o.*, u.name, u.email
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.status = 'PENDING'
AND o.created_at > NOW() - INTERVAL '7 days';

-- Required indexes:
-- orders(status, created_at) for the filter
-- users(id) for the JOIN (primary key, already exists)
```

**Index-Aware Schema Design:**

```sql
-- Anti-pattern: function on indexed column prevents index use
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-15';  -- ❌ full scan
-- Fix: use range instead
SELECT * FROM orders WHERE created_at >= '2024-01-15' AND created_at < '2024-01-16';  -- ✅

-- Anti-pattern: implicit type conversion
SELECT * FROM users WHERE id = '42';  -- id is BIGINT, '42' is VARCHAR
-- Causes type cast → may not use index
-- Fix: match types
SELECT * FROM users WHERE id = 42;  -- ✅

-- Anti-pattern: OR across different columns
SELECT * FROM orders WHERE user_id = 42 OR status = 'PENDING';  -- ❌ often full scan
-- Fix: UNION
SELECT * FROM orders WHERE user_id = 42
UNION
SELECT * FROM orders WHERE status = 'PENDING';  -- ✅ uses separate indexes
```

**Covering Index Strategy for Hot Queries:**

```sql
-- Hot query (runs 100K/day): dashboard summary for user
SELECT
    COUNT(*) FILTER (WHERE status = 'COMPLETED') AS completed,
    COUNT(*) FILTER (WHERE status = 'PENDING')   AS pending,
    SUM(total_amount) FILTER (WHERE status = 'COMPLETED') AS revenue
FROM orders
WHERE user_id = 42;

-- Optimal covering index: all columns in the query are in the index
-- No heap fetch needed → index-only scan
CREATE INDEX idx_orders_user_summary
ON orders(user_id, status, total_amount);
```

📊 **Example — Shopify's Indexing Strategy:**
- Index audit process: weekly automated report of low-selectivity and unused indexes
- All foreign keys are indexed (non-negotiable)
- Composite index naming convention: `idx_{table}_{col1}_{col2}_{purpose}`
- Background index creation (CONCURRENTLY) to avoid table locks in production

#### Think About It
- Which queries in your application run most frequently? Do they use indexes?
- How many indexes does your largest table have? What's the write overhead?

#### Key Takeaways
- Indexes transform O(N) scans into O(log N) lookups — the single highest-leverage optimisation
- Composite index column order follows: equality filters → range filters → included columns
- Remove unused indexes — they hurt write performance with no benefit
- Covering indexes eliminate heap fetches for high-frequency queries

#### Practice Exercise
Pick your three most important production queries. Find their execution plans with `EXPLAIN ANALYZE`. Identify which ones are doing full table scans. Design the optimal indexes to fix them.

---

## Section 5: Query Optimization

### What You'll Learn
- How to read and understand query execution plans
- Common query anti-patterns and how to fix them
- Query optimisation methodology for production databases

### Why This Matters

A single slow query running 10,000 times per day can consume more resources than 1,000 fast queries. Query optimisation directly translates to cost savings and user experience improvement.

#### 🟢 For Beginners: Reading Query Execution Plans

**The Restaurant Order Analogy:**

When you place a complex order at a restaurant ("I want the chicken, cooked medium, with the sauce on the side, and a diet coke"), the waiter relays it to the kitchen which *plans* how to execute it. The database does the same thing — it creates an execution plan before running any query.

**How to see the plan:**

```sql
-- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42 AND status = 'PENDING';

-- MySQL
EXPLAIN SELECT * FROM orders WHERE user_id = 42 AND status = 'PENDING';
```

**Plan output and what it means:**

```text
Bitmap Heap Scan on orders  (cost=4.58..58.33 rows=15 width=89)
                             ↑ estimated       ↑ row count estimate
  Recheck Cond: (user_id = 42)
  Filter: ((status)::text = 'PENDING'::text)
  ->  Bitmap Index Scan on idx_orders_user_id  (cost=0.00..4.58 rows=15)
                              ↑ using this index ← GOOD: index used
        Index Cond: (user_id = 42)
Planning Time: 0.8 ms
Execution Time: 2.1 ms   ← actual time
```

**The 3 operations to look for:**

| Plan Node | Meaning | Fast or Slow? |
|-----------|---------|---------------|
| `Index Scan` | Used an index | ✅ Fast |
| `Bitmap Index Scan` | Used index, batched reads | ✅ Usually good |
| `Seq Scan` | Full table scan | ⚠️ Slow on large tables |
| `Hash Join` | Joining via hash table | ✅ Good for large joins |
| `Nested Loop` | Row-by-row join | ⚠️ Slow on large tables |
| `Sort` | Had to sort (no index for ORDER BY) | ⚠️ Can be slow |

#### 🟡 For Intermediate: Common Anti-Patterns and Fixes

**Anti-Pattern 1: SELECT \***

```sql
-- BAD: fetches all columns, many may be large (TEXT, BLOB)
SELECT * FROM users WHERE id = 42;

-- GOOD: fetch only what you need
SELECT id, email, name, created_at FROM users WHERE id = 42;

-- BETTER: if using an index covering all needed columns → index-only scan
```

**Anti-Pattern 2: Non-SARGable Predicates (breaks index use)**

```sql
-- BAD: function on indexed column
SELECT * FROM orders WHERE YEAR(created_at) = 2024;         -- MySQL
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-15'; -- PostgreSQL
-- Both cause full scans even with index on created_at

-- GOOD: range predicate is SARGable
SELECT * FROM orders WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- BAD: wildcard leading LIKE
SELECT * FROM users WHERE email LIKE '%@gmail.com';         -- full scan

-- GOOD: trailing wildcard uses index
SELECT * FROM users WHERE email LIKE 'john%';
-- Or use full-text search for contains queries
```

**Anti-Pattern 3: Implicit Conversions**

```sql
-- users.phone is VARCHAR, but we're searching with an INTEGER
SELECT * FROM users WHERE phone = 5551234567;  -- implicit cast, no index

-- GOOD: match types
SELECT * FROM users WHERE phone = '555-123-4567';
```

**Anti-Pattern 4: OR Causing Full Scans**

```sql
-- BAD: OR across different indexed columns often causes full scan
SELECT * FROM orders 
WHERE user_id = 42 OR assigned_driver_id = 42;

-- GOOD: UNION uses separate indexes
SELECT * FROM orders WHERE user_id = 42
UNION ALL
SELECT * FROM orders WHERE assigned_driver_id = 42;
```

**Anti-Pattern 5: Correlated Subquery (runs once per row)**

```sql
-- BAD: for each order row, this subquery runs once
SELECT o.id, 
    (SELECT COUNT(*) FROM order_items WHERE order_id = o.id) AS item_count
FROM orders o
WHERE o.user_id = 42;
-- If user has 1000 orders → 1000 subquery executions

-- GOOD: aggregation with JOIN
SELECT o.id, COUNT(oi.id) AS item_count
FROM orders o
LEFT JOIN order_items oi ON oi.order_id = o.id
WHERE o.user_id = 42
GROUP BY o.id;
-- Single JOIN pass → much faster
```

**Anti-Pattern 6: OFFSET Pagination on Large Tables**

```sql
-- BAD: OFFSET 900000 LIMIT 10 still reads and discards 900000 rows
SELECT * FROM posts ORDER BY created_at DESC LIMIT 10 OFFSET 900000;

-- GOOD: cursor-based pagination (keyset pagination)
-- First page
SELECT * FROM posts WHERE created_at < NOW() ORDER BY created_at DESC LIMIT 10;
-- Next page (use last row's created_at as cursor)
SELECT * FROM posts WHERE created_at < '2024-01-15 10:30:00' ORDER BY created_at DESC LIMIT 10;
```

**Anti-Pattern 7: Missing JOIN Indexes**

```sql
-- Always ensure JOIN columns are indexed
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
JOIN orders o ON o.user_id = u.id    -- requires index on orders.user_id
WHERE u.created_at > '2024-01-01'    -- requires index on users.created_at
GROUP BY u.id, u.name;

-- Required indexes:
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_users_created_at ON users(created_at);
```

#### 🔴 For Advanced: Query Optimisation Methodology

**Systematic Query Tuning Process:**

```text
Step 1: IDENTIFY
   → Find slow queries: pg_stat_statements (PostgreSQL), slow query log (MySQL)
   → Sort by: total_time DESC (highest cumulative impact)

Step 2: ISOLATE
   → Run EXPLAIN ANALYZE on the query in isolation
   → Look for: Seq Scan on large tables, Nested Loop on large datasets, Sort without index

Step 3: HYPOTHESIZE
   → Which predicate is most selective?
   → What index would help?
   → Can the query be rewritten?

Step 4: IMPLEMENT
   → Add index CONCURRENTLY (non-blocking)
   → Rewrite query if needed

Step 5: VALIDATE
   → Compare EXPLAIN ANALYZE before/after
   → Monitor in production: query time, index scans

Step 6: MONITOR
   → Set up alerts: p99 query time > threshold
   → Weekly slow query review
```

**Advanced: Partitioning for Query Optimisation:**

```sql
-- Range partitioning: queries on recent data scan only recent partition
CREATE TABLE orders (
    id          BIGSERIAL,
    user_id     BIGINT,
    created_at  TIMESTAMP,
    total       DECIMAL(10,2)
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- Query automatically targets only the relevant partition
SELECT * FROM orders WHERE created_at BETWEEN '2024-01-01' AND '2024-03-31';
-- Without partitioning: scans 5 years of data
-- With partitioning: scans only Q1 2024 partition (1/20th of data)
```

**Query Planner Statistics:**

```sql
-- PostgreSQL uses statistics to estimate cardinality (row counts)
-- If estimates are wrong, plans are wrong

-- Check statistics age
SELECT schemaname, tablename, last_autoanalyze, last_analyze
FROM pg_stat_user_tables
ORDER BY last_autoanalyze ASC NULLS FIRST;

-- Force statistics refresh
ANALYZE orders;

-- For highly skewed distributions, increase statistics target
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 500;  -- default is 100
ANALYZE orders;
```

📊 **Example — GitHub's Query Optimisation Story:**
- Problem: Repository search was causing full table scans, 15s latency
- Root cause: `LIKE '%search_term%'` on a 400M-row table
- Solution: Elasticsearch for full-text, MySQL only for structured queries
- Result: Latency dropped from 15s to 200ms, MySQL CPU dropped 80%

#### Think About It
- Can you identify 3 anti-patterns from the list above in your current codebase?
- When does cursor-based pagination perform better than OFFSET, and when might OFFSET still be acceptable?

#### Key Takeaways
- `EXPLAIN ANALYZE` is the most important database debugging tool
- Avoid functions on indexed columns — they break index usage
- Cursor-based pagination is 100x+ faster than OFFSET on large tables
- Correlated subqueries are almost always replaceable with more efficient JOINs

---

## Section 6: Schema Design & Data Modelling

### What You'll Learn
- Normalisation and when to denormalise
- Database schema design for high-performance systems
- Schema evolution strategies for live databases

### Why This Matters

A schema is forever — or at least, changing it in production is expensive and risky. Good schema design upfront prevents painful migrations later.

#### 🟢 For Beginners: Normalisation Explained Simply

**The Address Book Analogy:**

Imagine you have a contacts app. You can store data in two ways:

**Unnormalised (everything in one table):**
```text
┌───────────────────────────────────────────────────────────┐
│ contact_id │ name  │ email           │ city    │ country  │
├───────────────────────────────────────────────────────────┤
│ 1          │ Alice │ alice@email.com │ London  │ UK       │
│ 2          │ Bob   │ bob@email.com   │ London  │ UK       │
│ 3          │ Carol │ carol@email.com │ London  │ UK       │
└───────────────────────────────────────────────────────────┘
```

Problem: "London, UK" is stored 3 times. If London gets renamed, you update 3+ rows.

**Normalised (split into related tables):**
```text
contacts:            cities:
id │ name │ city_id  id │ name   │ country
1  │ Alice │ 5        5  │ London │ UK
2  │ Bob   │ 5
3  │ Carol │ 5
```

Now "London, UK" is stored once. Update in one place. No inconsistency.

**Normal forms (simplified):**
- **1NF:** Each column has one value (no comma-separated lists)
- **2NF:** Every column depends on the whole primary key
- **3NF:** No column depends on another non-key column

**Rule of thumb:** Start normalised (3NF). Denormalise only when you have measured performance problems.

#### 🟡 For Intermediate: Schema Design for Common Patterns

**E-commerce Order Schema:**

```sql
-- Users
CREATE TABLE users (
    id          BIGSERIAL PRIMARY KEY,
    email       VARCHAR(255) UNIQUE NOT NULL,
    name        VARCHAR(100) NOT NULL,
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT chk_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Products
CREATE TABLE products (
    id          BIGSERIAL PRIMARY KEY,
    sku         VARCHAR(50) UNIQUE NOT NULL,
    name        VARCHAR(255) NOT NULL,
    price       DECIMAL(10,2) NOT NULL CHECK (price >= 0),
    stock       INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Orders (immutable after creation — soft delete only)
CREATE TABLE orders (
    id              BIGSERIAL PRIMARY KEY,
    user_id         BIGINT NOT NULL REFERENCES users(id),
    status          VARCHAR(20) NOT NULL DEFAULT 'PENDING'
                        CHECK (status IN ('PENDING', 'CONFIRMED', 'SHIPPED', 'DELIVERED', 'CANCELLED')),
    total_amount    DECIMAL(10,2) NOT NULL,
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Order items (snapshot of price at time of purchase)
CREATE TABLE order_items (
    id              BIGSERIAL PRIMARY KEY,
    order_id        BIGINT NOT NULL REFERENCES orders(id),
    product_id      BIGINT NOT NULL REFERENCES products(id),
    quantity        INTEGER NOT NULL CHECK (quantity > 0),
    unit_price      DECIMAL(10,2) NOT NULL,  -- snapshot, not live product price
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Key indexes
CREATE INDEX idx_orders_user_id     ON orders(user_id);
CREATE INDEX idx_orders_status_date ON orders(status, created_at DESC);
CREATE INDEX idx_order_items_order  ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
```

**Important Schema Decisions:**

| Decision | Choice | Why |
|---------|---------|-----|
| `unit_price` in order_items | Snapshot (not FK to products) | Product prices change; orders are immutable records |
| `status` as VARCHAR with CHECK | Enum-like constraint | Easier to add new statuses without ALTER TYPE |
| `BIGSERIAL` for IDs | Auto-increment 64-bit | UUID is 4x larger; BIGSERIAL safe for petabyte scale |
| `TIMESTAMP WITH TIME ZONE` | Always with timezone | Avoids DST bugs in multi-region deployments |
| `updated_at` trigger | Auto-update on change | Required for CDC (Change Data Capture) and sync |

#### 🔴 For Advanced: Denormalisation Strategies and Schema Evolution

**When and How to Denormalise:**

```sql
-- Scenario: social media feed
-- Normalised: every feed load requires JOINs across users, posts, follows
SELECT p.*, u.name, u.avatar_url, COUNT(l.id) AS like_count
FROM posts p
JOIN users u ON u.id = p.user_id
LEFT JOIN likes l ON l.post_id = p.id
WHERE p.user_id IN (SELECT followed_id FROM follows WHERE follower_id = 42)
ORDER BY p.created_at DESC
LIMIT 20;
-- → Complex join, hard to cache, slow at 100M users

-- Denormalised: pre-computed feed table (materialised)
CREATE TABLE user_feeds (
    user_id         BIGINT,
    post_id         BIGINT,
    author_name     VARCHAR(100),   -- denormalised from users
    author_avatar   VARCHAR(500),   -- denormalised from users
    like_count      INTEGER,        -- denormalised from likes
    created_at      TIMESTAMP,
    PRIMARY KEY (user_id, created_at DESC, post_id)  -- time-ordered for fast range reads
);

-- Feed load query becomes:
SELECT * FROM user_feeds WHERE user_id = 42 ORDER BY created_at DESC LIMIT 20;
-- → Single table scan, no joins, easily cached
```

**Schema Evolution Without Downtime:**

The "expand-contract" pattern for zero-downtime migrations:

```text
Phase 1 — EXPAND (backward compatible):
  → Add new column as nullable (ALTER TABLE ... ADD COLUMN nullable)
  → Deploy new code that writes to both old and new column
  
Phase 2 — MIGRATE:
  → Backfill new column for existing rows (in batches, not one UPDATE)
  → ALTER TABLE ... ALTER COLUMN SET NOT NULL once all rows filled

Phase 3 — CONTRACT:
  → Remove reads/writes to old column
  → Drop old column in separate migration

Example:
-- Old: status VARCHAR(20)
-- New: status_code INTEGER (more efficient)

-- Step 1: Add nullable column
ALTER TABLE orders ADD COLUMN status_code INTEGER;

-- Step 2: Backfill (run as background job)
UPDATE orders SET status_code = CASE status
    WHEN 'PENDING'    THEN 1
    WHEN 'CONFIRMED'  THEN 2
    WHEN 'SHIPPED'    THEN 3
    WHEN 'DELIVERED'  THEN 4
    WHEN 'CANCELLED'  THEN 5
END
WHERE status_code IS NULL
LIMIT 10000;  -- run in batches

-- Step 3 (after deploy): NOT NULL constraint
ALTER TABLE orders ALTER COLUMN status_code SET NOT NULL;

-- Step 4 (future deploy): drop old column
ALTER TABLE orders DROP COLUMN status;
```

**Partitioning Strategy for Large Tables:**

```sql
-- Time-series data: partition by month
CREATE TABLE events (
    id          BIGSERIAL,
    user_id     BIGINT,
    event_type  VARCHAR(50),
    properties  JSONB,
    created_at  TIMESTAMP NOT NULL
) PARTITION BY RANGE (created_at);

-- Automatic partition creation (via pg_partman or application)
CREATE TABLE events_2024_01 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE events_2024_02 PARTITION OF events
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Retention: drop old partitions (instant — no row-by-row DELETE)
DROP TABLE events_2022_01;  -- drops entire partition instantly vs DELETE taking hours
```

📊 **Example — Airbnb's Schema Evolution Story:**
- Challenge: adding `instant_book` flag to 100M listings without downtime
- Used expand-contract pattern: added nullable column, backfilled over 3 days, then made NOT NULL
- Zero-downtime migration with 0 errors

#### Think About It
- Should you store a user's age in the database, or their date of birth?
- At what table size do you typically need to consider partitioning?

#### Key Takeaways
- Start normalised — denormalise only with measured evidence
- Price and quantity snapshots in order items prevent data corruption when source data changes
- Schema migrations must be done in backward-compatible phases for zero downtime
- Partition large time-series tables to enable instant data retention (DROP PARTITION)

---

## Section 7: Replication & High Availability

### What You'll Learn
- Primary-replica replication architecture
- Failover strategies and automatic promotion
- Consistency trade-offs with replication lag

### Why This Matters

Your database will fail. The question is not if, but when. Replication is the difference between a 5-minute blip and a 4-hour outage.

#### 🟢 For Beginners: What Is Replication?

**The Newspaper Printing Analogy:**

A newspaper is printed at one press (the primary), then physically copied and distributed to many locations (replicas). If the main press breaks down, another location can start printing copies. Readers continue getting their newspapers with minimal disruption.

Database replication works the same way:
- **Primary** (master): handles all writes
- **Replicas** (read replicas): receive copies of changes, handle reads

**Why we replicate:**
1. **High availability:** if primary fails, promote a replica
2. **Read scaling:** distribute reads across replicas
3. **Backup:** replicas can be used as backup sources
4. **Geographic distribution:** replica in each region for low-latency reads

#### 🟡 For Intermediate: Replication Architectures

**Single Primary, Multiple Read Replicas:**

```text
           Writes                        Reads
              ↓                             ↓
        ┌──────────┐         ┌─────────────────────────┐
        │ Primary  │────WAL──►  Replica 1  │  Replica 2  │
        │ (writes) │         │  (reads)   │  (reads)    │
        └──────────┘         └─────────────────────────┘
              │
              └──WAL──► Replica 3 (cross-region DR)
```

**Replication modes and their trade-offs:**

| Mode | How it works | Durability | Write Latency | Risk |
|------|-------------|-----------|---------------|------|
| Async (default) | Primary doesn't wait for replica acknowledgment | Potential data loss on failover (RPO > 0) | Low (<1ms extra) | Replica lag; data loss on crash |
| Sync (1 replica) | Primary waits for 1 replica to confirm | No data loss (RPO = 0) | Medium (+network RTT) | Replica becomes bottleneck |
| Quorum | Primary waits for majority | No data loss | Medium | More complex setup |

**Application-Level Routing:**

```python
# Route reads to replicas, writes to primary
import psycopg2
from psycopg2 import pool

primary_pool   = pool.ThreadedConnectionPool(5, 20, host="db-primary")
replica_pool   = pool.ThreadedConnectionPool(10, 50, host="db-replica-lb")

def get_user(user_id):
    # Read → replica
    conn = replica_pool.getconn()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    return cursor.fetchone()

def update_user(user_id, email):
    # Write → primary
    conn = primary_pool.getconn()
    cursor = conn.cursor()
    cursor.execute("UPDATE users SET email = %s WHERE id = %s", (email, user_id))
    conn.commit()
```

**Replication Lag Problem:**

```text
Timeline:
t=0ms:  User updates profile on primary
t=5ms:  WAL sent to replica
t=50ms: User immediately reads their profile
         → Reads from replica (still has OLD data!)
         → User sees stale data → confused

Solutions:
1. Read from primary for "read-your-own-writes" operations
2. Route user to same replica for a session (sticky sessions)  
3. Wait for replica lag to clear before reading (if eventual consistency is acceptable)
4. Use a cache invalidation approach
```

#### 🔴 For Advanced: Failover, Consensus, and Multi-Master

**Automated Failover with Patroni (PostgreSQL HA):**

```text
Architecture:
  ┌─────────────┐    ┌─────────────────────┐
  │   Patroni   │───►│  etcd/Consul/ZooKeeper│  (distributed consensus store)
  │  (primary)  │    └─────────────────────┘
  └──────┬──────┘             ↑
         │                    │ monitor
         WAL stream           │ + leader election
         │              ┌─────┴──────┐
         ↓              │  Patroni   │
  ┌─────────────┐       │ (replica)  │
  │  PostgreSQL │       └─────────────┘
  │   Replica   │
  └─────────────┘

Failover sequence (automatic, ~30 seconds):
1. Patroni on primary detects heartbeat failure
2. etcd leader election: replica wins
3. Replica promotes itself to primary
4. HAProxy/pgBouncer health check updates routing
5. Application automatically connects to new primary
```

**Split-Brain Prevention:**

Split-brain occurs when both old primary and new primary believe they are the leader:

```text
DANGER: Without fencing, both nodes accept writes → data divergence

Prevention strategies:
1. STONITH (Shoot The Other Node In The Head): 
   → Winner sends fencing command to loser (power off, network isolation)
   → Guarantees only one primary exists

2. Lease-based fencing:
   → Primary holds a "lease" from consensus store (e.g., etcd)
   → Lease expires after 30 seconds without renewal
   → If primary can't reach etcd, it stops accepting writes
   
3. Epoch fencing:
   → Each primary has an epoch number
   → Old primary rejects writes if it receives message from higher epoch
```

**Multi-Master Replication (Active-Active):**

```text
Region US-East         Region EU-West
┌──────────────┐       ┌──────────────┐
│ Primary US   │◄─────►│ Primary EU   │
│ (writes)     │       │ (writes)     │
└──────────────┘       └──────────────┘

Challenges:
- Conflict resolution: same row written in both regions simultaneously
  → Last-write-wins (LWW): simple but can lose data
  → CRDT (Conflict-free Replicated Data Type): for counters, sets
  → Application-level: custom merge logic
  
Used by: CockroachDB, Cassandra (tunable consistency), Google Spanner (TrueTime)
```

**RPO and RTO Targets:**

| Scenario | RPO (data loss) | RTO (recovery time) | Solution |
|---------|-----------------|---------------------|---------|
| Development | Hours | Hours | Single instance |
| Small production | < 1 hour | < 30 min | Async replica + manual failover |
| Standard production | < 1 min | < 5 min | Async replica + Patroni |
| Critical (finance, healthcare) | 0 | < 30 sec | Sync replica + Patroni + fencing |
| Global (multi-region) | 0 | < 10 sec | Multi-master + CockroachDB/Spanner |

📊 **Example — GitHub's Database HA Architecture:**
- Uses Orchestrator for MySQL topology management
- Automatic failover completes in < 30 seconds
- Semi-sync replication: at least 1 replica must acknowledge before primary confirms
- Zonal isolation: primary and replicas in different availability zones

#### Think About It
- What is your current database RPO and RTO? Are they documented and tested?
- Do you perform regular failover drills to validate that HA actually works?

#### Key Takeaways
- Async replication is the default for performance; use sync replication when RPO = 0 is required
- Replication lag can cause "read your own writes" bugs — design applications to account for this
- Automated failover requires consensus (etcd/ZooKeeper) and fencing to prevent split-brain
- Test your failover — an untested DR plan is not a DR plan

---

## Section 8: Sharding & Horizontal Scaling

### What You'll Learn
- When and how to shard databases
- Sharding strategies and their trade-offs
- How to handle cross-shard queries and resharding

### Why This Matters

Vertical scaling (bigger server) has limits and diminishing returns. Sharding is how companies like Instagram, Shopify, and Uber handle petabytes of data and millions of QPS.

#### 🟢 For Beginners: What Is Sharding?

**The Post Office Analogy:**

Imagine all the mail in a city going through one post office — it would be overwhelmed. Instead, mail is routed to different post offices by neighborhood (ZIP code). Each post office only handles its area's mail.

Database sharding works the same way:
- Each "post office" is a database shard (separate server)
- Mail (data) is routed to the right shard based on a "shard key" (like ZIP code)
- Each shard is an independent database that handles only its portion of data

**Before and after sharding:**

```text
Before (single server — limit reached):
┌──────────────────────────────────┐
│  All 1 billion user rows         │
│  CPU: 100% │ Disk: 95% full      │
│  P99 latency: 2000ms ← too slow  │
└──────────────────────────────────┘

After (4 shards):
┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
│ Shard 0    │  │ Shard 1    │  │ Shard 2    │  │ Shard 3    │
│ user_id    │  │ user_id    │  │ user_id    │  │ user_id    │
│ % 4 = 0   │  │ % 4 = 1   │  │ % 4 = 2   │  │ % 4 = 3   │
│ 250M rows  │  │ 250M rows  │  │ 250M rows  │  │ 250M rows  │
│ CPU: 25%   │  │ CPU: 25%   │  │ CPU: 25%   │  │ CPU: 25%   │
└────────────┘  └────────────┘  └────────────┘  └────────────┘
```

#### 🟡 For Intermediate: Sharding Strategies

**1. Hash-Based Sharding:**

```python
def get_shard(user_id, num_shards=64):
    return user_id % num_shards

# user_id=1 → shard 1
# user_id=65 → shard 1
# user_id=64 → shard 0
```

| Pros | Cons |
|------|------|
| Even data distribution | Can't do range queries (e.g., "users created this month") |
| Simple routing logic | Adding shards requires rehashing all data |
| Predictable performance | Cross-shard scatter-gather for non-shard-key queries |

**2. Range-Based Sharding:**

```text
Shard 0: user_id 1 - 250,000,000
Shard 1: user_id 250,000,001 - 500,000,000
Shard 2: user_id 500,000,001 - 750,000,000
Shard 3: user_id 750,000,001 - 1,000,000,000
```

| Pros | Cons |
|------|------|
| Range queries stay on one shard | Hotspots: newest data goes to last shard |
| Easy to understand | Uneven distribution over time |

**3. Directory-Based Sharding (most flexible):**

```text
Routing table (stored in fast cache):
user_id 1-1000       → Shard A
user_id 1001-2000    → Shard B
user_id 2001-50000   → Shard C
...
```

| Pros | Cons |
|------|------|
| Full control over data placement | Extra lookup per query |
| Easy to rebalance | Routing table is a single point of failure |
| Can handle hotspots | Operational complexity |

**4. Consistent Hashing (handles resharding):**

```text
Hash ring (0 to 2^32):
        0
       /|\
  Shard D | Shard A
   270° / \ 90°
       |   |
  Shard C | Shard B
       \ 180°/
```

When adding a new shard: only move ~1/N of the data (vs 100% with simple hash).

#### 🔴 For Advanced: Cross-Shard Operations and Resharding

**Cross-Shard Query Patterns:**

```python
# Scatter-gather: query all shards, aggregate results
def count_orders_by_status(status: str) -> int:
    results = []
    for shard_id in range(NUM_SHARDS):
        conn = get_shard_connection(shard_id)
        count = conn.execute(
            "SELECT COUNT(*) FROM orders WHERE status = %s", (status,)
        ).scalar()
        results.append(count)
    return sum(results)  # merge at application layer

# Problem: latency = max(latency of each shard)
# Solution: parallel execution
import concurrent.futures

def count_orders_parallel(status: str) -> int:
    def query_shard(shard_id):
        conn = get_shard_connection(shard_id)
        return conn.execute("SELECT COUNT(*) FROM orders WHERE status = %s", (status,)).scalar()
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=NUM_SHARDS) as executor:
        counts = list(executor.map(query_shard, range(NUM_SHARDS)))
    return sum(counts)
```

**Hotspot Problem and Solutions:**

```text
Problem: Shard 3 gets 80% of traffic (e.g., Twitter celebrity with 100M followers)

Solutions:

1. Sub-sharding: split hot shard into multiple shards
   Shard 3 → Shard 3a, 3b, 3c, 3d

2. Application-level routing: direct hot users to dedicated shard

3. Caching: serve celebrity data from cache, not DB
   → 99%+ cache hit rate for celebrity profiles

4. Fan-out at write time: pre-materialise data for followers
   → Avoids hotspot at read time
```

**Online Resharding Strategy (zero downtime):**

```text
Goal: Expand from 16 to 32 shards without downtime

Phase 1 — Dual-write (2 weeks):
  → Every write goes to both old shard and new shard
  → Reads still come from old shard

Phase 2 — Backfill (background):
  → Copy historical data to new shard layout
  → Verify consistency

Phase 3 — Gradual cutover:
  → Route 1% of reads to new shards → verify
  → Increase to 5%, 10%, 25%, 50%, 100%

Phase 4 — Cleanup:
  → Remove old shards
  → Remove dual-write logic
```

📊 **Example — Instagram's Sharding Journey:**
- Started: single PostgreSQL instance (2010)
- 2012: First shard (user data split into 12 PostgreSQL shards)
- 2019: 5,500+ PostgreSQL instances, hundreds of shards
- Shard key: user_id (consistent hashing)
- Migration tool: Shard rehoming (move shard data with zero downtime)

#### Think About It
- What would be the optimal shard key for a multi-tenant SaaS application?
- What consistency guarantees can you provide across shards?

#### Key Takeaways
- Shard only when you have measured need — it adds significant operational complexity
- Hash sharding distributes evenly but breaks range queries; choose based on access patterns
- Consistent hashing minimises data movement when adding shards
- Cross-shard queries are expensive — design to minimise them

---

## Section 9: Caching Strategies

### What You'll Learn
- Cache-aside, read-through, and write-through patterns
- Cache invalidation strategies
- Multi-tier caching architecture

### Why This Matters

The fastest database query is one that never reaches the database. Caching can reduce database load by 90%+ and cut latency by 10-100x.

#### 🟢 For Beginners: What Is a Database Cache?

**The Chef's Prep Station Analogy:**

A chef prepares frequently-used ingredients at the start of the day and keeps them on the prep station (cache). When a dish needs diced onions, instead of going to the cold storage (database), the chef grabs from the prep station — 10 seconds vs 2 minutes.

But the prep station has limited space, so you only prep what you'll use frequently. Less-used ingredients go back to cold storage.

**Cache vs Database:**

| Aspect | Cache (Redis) | Database (PostgreSQL) |
|--------|--------------|----------------------|
| Storage medium | RAM | SSD/HDD |
| Latency | ~0.1ms | 1-10ms |
| Capacity | GBs | TBs |
| Persistence | Optional | Yes |
| Query power | Key-value, sorted sets | Full SQL |
| Cost per GB | ~$10-20/GB | ~$0.10/GB |

#### 🟡 For Intermediate: Caching Patterns

**Cache-Aside (Lazy Loading) — Most Common:**

```python
import redis
import json

cache = redis.Redis(host='redis-cluster', decode_responses=True)

def get_user(user_id: int) -> dict:
    cache_key = f"user:{user_id}"
    
    # 1. Check cache
    cached = cache.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # 2. Cache miss — fetch from database
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    
    # 3. Populate cache (TTL = 1 hour)
    cache.setex(cache_key, 3600, json.dumps(user))
    
    return user
```

**When to use:** Most OLTP workloads. Read-heavy. Tolerates brief inconsistency.

**Read-Through — Cache manages DB access:**

```python
# Cache library handles miss automatically
# Application only talks to cache interface
user = cache_service.get("user:42")  # cache_service fetches from DB on miss
```

**Write-Through — Always write to cache AND DB:**

```python
def update_user(user_id: int, email: str):
    # Write to DB
    db.execute("UPDATE users SET email = %s WHERE id = %s", (email, user_id))
    
    # Immediately update cache
    cache_key = f"user:{user_id}"
    user_data = db.query("SELECT * FROM users WHERE id = %s", user_id)
    cache.setex(cache_key, 3600, json.dumps(user_data))
```

**When to use:** When stale reads are unacceptable. Costs more writes.

**Write-Behind (Write-Back) — Write to cache, flush to DB asynchronously:**

```python
# Write to cache immediately
cache.hset(f"user:{user_id}", "email", new_email)
# Queue for async DB write
write_queue.publish({"table": "users", "id": user_id, "email": new_email})

# Background worker
async def flush_to_db(write_event):
    db.execute("UPDATE users SET email = %s WHERE id = %s",
               (write_event["email"], write_event["id"]))
```

**When to use:** Write-heavy, can tolerate brief risk of loss. Used in gaming (scores), IoT.

**Cache Invalidation Strategies:**

| Strategy | How | When to use |
|----------|-----|------------|
| TTL expiry | `SETEX key 3600 value` | Tolerable eventual consistency |
| Delete on write | `DEL user:42` on UPDATE | Strong consistency needed |
| Cache-aside version | Include version in key: `user:42:v3` | Avoid thundering herd |
| Publish/subscribe | Notify all cache nodes on change | Distributed caches |

#### 🔴 For Advanced: Cache Failure Patterns and Multi-Tier Caching

**The Thundering Herd Problem:**

```text
Scenario: user:42's cache expires at exactly the same moment 10,000 requests arrive
→ All 10,000 requests hit the database simultaneously
→ Database spike → potential cascade failure

Solutions:

1. Cache stampede prevention via locking:
def get_with_lock(cache_key: str, fetch_fn, ttl: int):
    value = cache.get(cache_key)
    if value:
        return json.loads(value)
    
    lock_key = f"lock:{cache_key}"
    if cache.setnx(lock_key, "1"):  # acquire lock
        cache.expire(lock_key, 5)   # 5-second lock timeout
        try:
            value = fetch_fn()
            cache.setex(cache_key, ttl, json.dumps(value))
        finally:
            cache.delete(lock_key)  # release lock
    else:
        # Another process is fetching — wait and retry
        time.sleep(0.1)
        return get_with_lock(cache_key, fetch_fn, ttl)

2. Probabilistic early expiry:
   → Randomly refresh cache slightly before TTL expires
   → Prevents all requests hitting DB at exact expiry moment
```

**Cache Avalanche (all keys expire simultaneously):**

```python
import random

def set_with_jitter(key: str, value: dict, base_ttl: int = 3600):
    # Add random jitter ±10% to prevent simultaneous expiry
    ttl = base_ttl + random.randint(-base_ttl // 10, base_ttl // 10)
    cache.setex(key, ttl, json.dumps(value))
```

**Multi-Tier Caching Architecture:**

```text
Request flow:
Browser request
      ↓
1. CDN (CloudFront/Fastly)          — Hit rate: 80%, latency: <5ms
      ↓ miss
2. Application-level cache (in-process)  — Hit rate: 50% of remainder, latency: 0.01ms
      ↓ miss
3. Redis cluster (distributed cache)     — Hit rate: 90% of remainder, latency: 0.5ms
      ↓ miss
4. Database read replica                 — latency: 5-50ms
      ↓ miss (cold data)
5. Database primary / archive            — latency: 10-200ms

Overall hit rate: 1 - (1-0.80)(1-0.50)(1-0.90) = 98.5%
i.e., only 1.5% of requests reach the database
```

**Redis Cluster for High Availability:**

```text
Redis Cluster (6 nodes: 3 primary + 3 replica):

Primary 1 (slots 0-5460)     → Replica 1
Primary 2 (slots 5461-10922) → Replica 2
Primary 3 (slots 10923-16383)→ Replica 3

If Primary 1 fails:
→ Replica 1 automatically promoted
→ Cluster continues serving with no downtime
→ Recovery time: ~15 seconds
```

📊 **Example — Twitter's Cache Architecture:**
- Twemcache (Twitter's Memcached fork): billions of cached objects
- Fatcache: SSD-backed Memcache for cold data
- Manhattan (distributed KV): persistent cache for tweet metadata
- Cache hit rate: 99%+ for home timeline reads
- Database queries avoided per second: ~1.5 million

#### Think About It
- What is the cache hit rate on your most accessed database tables?
- What happens to your application if Redis goes down? Is there a fallback?

#### Key Takeaways
- Cache-aside is the most common pattern; use write-through when strong consistency is needed
- Always add TTL jitter to prevent cache avalanche
- Multi-tier caching (CDN → in-process → Redis → DB) can achieve 98%+ hit rates
- Design for cache failures: graceful degradation to database reads

---

## Section 10: Database Selection Guide

### What You'll Learn
- When to choose SQL vs NoSQL vs NewSQL
- Purpose-built databases for specific workloads
- How to justify database choices in interviews

### Why This Matters

Choosing the wrong database type for your workload leads to years of pain. Choosing correctly makes everything easier.

#### 🟢 For Beginners: Database Types Simply Explained

**The storage analogy:**

| Database Type | Real-world Analogy |
|--------------|-------------------|
| Relational (PostgreSQL) | Excel spreadsheet with enforced relationships |
| Document (MongoDB) | Filing cabinet of folders, each folder can have different contents |
| Key-Value (Redis) | Dictionary/hashmap — instant lookup by key |
| Time-Series (InfluxDB) | Stock ticker tape — ordered by time |
| Graph (Neo4j) | Social network map with labeled connections |
| Search (Elasticsearch) | Library index card catalogue with full-text search |
| Wide-column (Cassandra) | Sparse spreadsheet that scales horizontally |

#### 🟡 For Intermediate: Database Selection Decision Framework

**Decision Tree:**

```text
Does your data have complex relationships needing JOIN queries?
│
├─ YES → Do you need ACID transactions across multiple tables?
│         ├─ YES → PostgreSQL or MySQL
│         └─ NO  → Still probably PostgreSQL (ACID is free here)
│
└─ NO → What is the primary access pattern?
         │
         ├─ Key-value lookups at very high QPS?
         │   └─ Redis (in-memory) or DynamoDB (persistent)
         │
         ├─ Write-heavy time-series/append data?
         │   └─ Cassandra, InfluxDB, or TimescaleDB
         │
         ├─ Documents with flexible schema?
         │   └─ MongoDB or DynamoDB
         │
         ├─ Full-text search or complex aggregations?
         │   └─ Elasticsearch
         │
         ├─ Graph traversals (social graph, recommendations)?
         │   └─ Neo4j or Amazon Neptune
         │
         └─ OLAP analytics on large datasets?
             └─ ClickHouse, BigQuery, or Redshift
```

**Database Comparison Matrix:**

| Database | Best For | QPS (single node) | Consistency | Query Power |
|---------|----------|------------------|-------------|-------------|
| PostgreSQL | OLTP, general purpose | 50K reads, 10K writes | Strong (ACID) | Full SQL |
| MySQL | Web OLTP, read-heavy | 80K reads, 15K writes | Strong (ACID) | Full SQL |
| Cassandra | Write-heavy, time-series, wide data | 200K writes | Eventual (tunable) | Limited (CQL) |
| Redis | Caching, sessions, leaderboards | 1M ops | None (fire-and-forget) | Key-value, sorted sets |
| MongoDB | Documents, variable schema | 50K reads | Document-level ACID | Rich query language |
| Elasticsearch | Full-text search, logs | 10K complex, 50K simple | Near real-time | Rich DSL |
| ClickHouse | OLAP analytics | 1B rows/sec (scans) | Eventually consistent | Full SQL (column) |
| DynamoDB | Key-value, serverless scaling | Unlimited (managed) | Eventually / Strong | Limited |

#### 🔴 For Advanced: Polyglot Persistence and When to Use Multiple Databases

**Real-world Multi-Database Architecture (E-commerce):**

```text
┌─────────────────────────────────────────────────────────────┐
│                      E-commerce Platform                     │
│                                                              │
│  PostgreSQL          Redis              Elasticsearch        │
│  ├─ users            ├─ sessions        └─ product search   │
│  ├─ orders           ├─ cart (TTL)          (full-text,     │
│  ├─ payments         ├─ rate limiting        filters, facets)│
│  └─ inventory        └─ product cache                        │
│                                                              │
│  Cassandra           Kafka              ClickHouse           │
│  └─ user events      └─ order events    └─ analytics         │
│     (clickstream,       (CDC to sync        (dashboards,     │
│      views, searches)    other DBs)          reports)        │
└─────────────────────────────────────────────────────────────┘
```

**Database Synchronisation with CDC (Change Data Capture):**

```text
PostgreSQL ──WAL──► Debezium ──► Kafka ──► [Elasticsearch, ClickHouse, Redis]

Every INSERT/UPDATE/DELETE in PostgreSQL:
1. Written to WAL (always happens)
2. Debezium reads WAL as a "change event"
3. Publishes to Kafka topic
4. Consumers update their respective stores:
   - Elasticsearch: update search index
   - ClickHouse: append to analytics table
   - Redis: invalidate/update cache

Lag: typically < 1 second end-to-end
```

**NewSQL Databases — Best of Both Worlds:**

| Database | Use Case | Key Feature |
|---------|----------|-------------|
| CockroachDB | Global OLTP, survive region failures | Distributed SQL + strong consistency |
| Google Spanner | Global financial-grade transactions | TrueTime API for global serialisability |
| TiDB | MySQL-compatible horizontal scale | Raft consensus, automatic sharding |
| YugabyteDB | PostgreSQL-compatible global scale | Multi-region active-active |

**When to use NewSQL:**
- Need SQL (complex queries, JOINs)
- AND need horizontal scale (beyond single node)
- AND need strong consistency
- AND can accept 2-5x higher latency than single-node PostgreSQL

📊 **Example — Uber's Polyglot Persistence:**
- PostgreSQL: trip records, payment transactions (ACID required)
- MySQL: driver profile data (legacy, highly optimised)
- Cassandra: real-time driver locations (write-heavy, 500K writes/sec)
- Redis: surge pricing, session data (sub-millisecond latency)
- Elasticsearch: customer support search, trip history search
- Schemaless (internal): document store for flexible attributes

#### Think About It
- In your current system, could any table be better served by a different database type?
- What are the operational costs of running 5 different database systems vs 1?

#### Key Takeaways
- PostgreSQL handles 80% of use cases well — don't add complexity without evidence
- Add purpose-built databases for specific, proven needs (search, caching, analytics)
- CDC (Change Data Capture) enables safe polyglot persistence by keeping databases in sync
- NewSQL fills the gap when you need SQL semantics at horizontal scale

---

## Section 11: Scalability & Performance Optimisation

### What You'll Learn
- The full performance optimisation hierarchy
- Connection pooling and prepared statements
- Database configuration tuning

### Why This Matters

Performance work without a hierarchy leads to premature optimisation. This section gives you the framework to prioritise correctly and make impactful changes.

#### 🟢 For Beginners: The Performance Optimisation Hierarchy

Always optimise in this order — higher up = more impactful, easier to implement:

```text
1. Application design (schema, queries, indexes)     ← highest impact
2. Caching                                            ← second highest
3. Database configuration tuning                      ← medium impact
4. Hardware (more RAM, faster SSD)                    ← low impact on bad design
5. Horizontal scaling (more servers)                  ← last resort
```

A common mistake: buying a 32-core server to fix a slow query, when the fix is a single `CREATE INDEX`.

#### 🟡 For Intermediate: Connection Pooling

**Why connection pooling matters:**

```text
Without pooling:
  10,000 concurrent users each open a DB connection
  → 10,000 connections × 5MB overhead = 50GB RAM just for connections!
  → PostgreSQL max_connections: typically 100-500
  → Result: connection refused errors, crashes

With pooling (PgBouncer):
  10,000 app requests → PgBouncer → 50 actual DB connections
  → Each connection shared across many requests
  → Database handles 50 connections, not 10,000
```

**PgBouncer Configuration:**

```ini
[databases]
mydb = host=postgresql-primary port=5432 dbname=production

[pgbouncer]
pool_mode = transaction        ; best for most apps (connection released after each txn)
max_client_conn = 10000        ; total app connections accepted
default_pool_size = 50         ; connections to actual PostgreSQL per database
min_pool_size = 10             ; keep minimum warm connections
reserve_pool_size = 5          ; emergency connections for admin
server_idle_timeout = 600      ; close idle DB connections after 10 min
```

**Application-side connection pool (Python SQLAlchemy):**

```python
from sqlalchemy import create_engine

engine = create_engine(
    "******pgbouncer-host:6432/mydb",
    pool_size=10,           # persistent connections in pool
    max_overflow=20,        # extra connections under load
    pool_timeout=30,        # wait up to 30s for available connection
    pool_recycle=3600,      # recycle connections every hour (prevents stale)
    pool_pre_ping=True,     # verify connection alive before use
)
```

#### 🔴 For Advanced: PostgreSQL Configuration Tuning

**Critical PostgreSQL parameters (for 64GB RAM server):**

```ini
# Memory settings
shared_buffers = 16GB              # 25% of RAM for PostgreSQL buffer pool
effective_cache_size = 48GB        # hint to planner (total RAM available for caching)
work_mem = 64MB                    # per sort/hash operation (multiply by connections!)
maintenance_work_mem = 2GB         # for VACUUM, CREATE INDEX

# Write performance
wal_buffers = 64MB                 # WAL buffer size (16MB minimum, 64MB for heavy writes)
checkpoint_completion_target = 0.9 # spread checkpoint writes (reduce I/O spikes)
max_wal_size = 4GB                 # allow more WAL before checkpoint
synchronous_commit = off           # async commit for non-critical data (risk: last ~1ms)

# Connection settings
max_connections = 200              # use PgBouncer to handle thousands
autovacuum_max_workers = 5         # more vacuum workers for high-write tables

# Query planner
random_page_cost = 1.1             # for SSD (default 4.0 is for HDD)
effective_io_concurrency = 200     # for SSD (how many parallel I/Os planner can assume)
```

**VACUUM and Table Bloat:**

```sql
-- Check table bloat from dead tuples
SELECT
    schemaname,
    tablename,
    n_dead_tup,
    n_live_tup,
    ROUND(n_dead_tup * 100.0 / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
    last_autovacuum,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY n_dead_tup DESC;

-- Tables with > 20% dead tuples need manual VACUUM
VACUUM ANALYZE orders;

-- Full bloat cleanup (needs exclusive lock — use during maintenance window)
VACUUM FULL ANALYZE orders;
```

**Autovacuum Tuning for High-Traffic Tables:**

```sql
-- Tune autovacuum per table for high-write tables
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.01,   -- vacuum when 1% of rows are dead (default 20%)
    autovacuum_analyze_scale_factor = 0.005, -- analyze when 0.5% are changed
    autovacuum_vacuum_cost_delay = 2         -- less throttling (default 20ms)
);
```

📊 **Example — Cloudflare's PostgreSQL Configuration:**
- 1.5M QPS through PostgreSQL
- `shared_buffers` = 40% of RAM (above typical 25% guideline)
- Horizontal read scaling via 10 read replicas per shard
- PgBouncer in transaction mode: 500K client connections → 2K DB connections

#### Key Takeaways
- Optimise schema and indexes before tuning configuration or adding hardware
- PgBouncer reduces actual database connections by 100-200x vs direct connections
- `random_page_cost = 1.1` for SSDs changes planner to prefer index scans
- Autovacuum tuning for high-write tables prevents table bloat and degraded performance

---

## Section 12: Security Considerations

### What You'll Learn
- Database access control best practices
- SQL injection prevention
- Encryption at rest and in transit

### Why This Matters

Database breaches are the most expensive type of security incident — average cost $4.35M per breach (IBM 2022). Most are preventable.

#### 🟢 For Beginners: Basic Database Security

**The "least privilege" principle:** A user should only have exactly the permissions they need.

```sql
-- Create application user with minimal permissions
CREATE USER app_user WITH PASSWORD 'strong_random_password_from_secrets_manager';

-- Read-only access (for read replicas)
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

-- Write access (only tables the app needs to modify)
GRANT SELECT, INSERT, UPDATE ON orders, order_items TO app_user;
GRANT SELECT ON users, products TO app_user;  -- app can't delete users

-- Explicitly deny dangerous operations
REVOKE DROP, TRUNCATE ON ALL TABLES IN SCHEMA public FROM app_user;

-- No direct table access — use stored procedures/functions
-- (prevents SQL injection via parameter binding enforcement)
```

#### 🟡 For Intermediate: SQL Injection Prevention

**SQL injection is still the #1 database attack vector:**

```python
# VULNERABLE: string interpolation
user_id = request.get("user_id")  # attacker sends: "1 OR 1=1 --"
query = f"SELECT * FROM users WHERE id = {user_id}"
# Result: SELECT * FROM users WHERE id = 1 OR 1=1 --
# Returns ALL users!

# SAFE: parameterised queries (always use this)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
# Database treats user_id as DATA, not SQL code
```

**Always use parameterised queries:**

```python
# Python + SQLAlchemy (safe)
from sqlalchemy import text

result = db.execute(
    text("SELECT * FROM users WHERE email = :email AND is_active = :active"),
    {"email": user_email, "active": True}
)

# Python + psycopg2 (safe)
cursor.execute(
    "SELECT * FROM users WHERE email = %s AND is_active = %s",
    (user_email, True)
)
```

#### 🔴 For Advanced: Encryption, Auditing, and Compliance

**Encryption at rest (PostgreSQL with pgcrypto):**

```sql
-- Enable pgcrypto extension
CREATE EXTENSION pgcrypto;

-- Store PII encrypted (AES-256)
INSERT INTO users (email, ssn_encrypted)
VALUES (
    'alice@example.com',
    pgp_sym_encrypt('123-45-6789', current_setting('app.encryption_key'))
);

-- Decrypt for authorised reads
SELECT email, pgp_sym_decrypt(ssn_encrypted::bytea, current_setting('app.encryption_key'))
FROM users
WHERE id = 42;
```

**Column-level encryption for PCI DSS / HIPAA compliance:**

| Data | Encryption | Tokenisation |
|------|-----------|-------------|
| Credit card number | Not stored | Token stored, real number in PCI vault |
| SSN | AES-256 encrypted | Tokenised for most operations |
| Name, email | At-rest disk encryption | Plain (needed for search) |
| Password | bcrypt + salt (hash) | Never stored |

**Database audit logging:**

```sql
-- PostgreSQL: log all DDL (structure changes)
ALTER SYSTEM SET log_statement = 'ddl';

-- Log slow queries
ALTER SYSTEM SET log_min_duration_statement = '1000';  -- log queries > 1 second

-- Dedicated audit extension (pgaudit)
CREATE EXTENSION pgaudit;
ALTER SYSTEM SET pgaudit.log = 'write, role';  -- log all writes and permission changes
```

📊 **Example — Healthcare Database Security (HIPAA):**
- Database in private subnet (no public internet access)
- Application connects via SSL certificate authentication (not password)
- All PHI columns encrypted with customer-managed KMS keys (AWS KMS)
- Complete audit trail: every SELECT on PHI tables logged with user, timestamp, query
- Row-level security: doctors can only see their own patients' data

```sql
-- Row-level security (PostgreSQL)
ALTER TABLE patient_records ENABLE ROW LEVEL SECURITY;

CREATE POLICY patient_access ON patient_records
    USING (doctor_id = current_setting('app.current_doctor_id')::INT);

-- Now each doctor only sees their own patients' records
-- Even SELECT * FROM patient_records is safe — RLS filters automatically
```

#### Key Takeaways
- Parameterised queries are non-negotiable — always use them
- Least privilege: application users should only have the minimum required permissions
- Encrypt PII and regulated data at the column level, not just at disk level
- Audit logging is required for HIPAA, PCI DSS, SOX compliance

---

## Section 13: Monitoring & Observability

### What You'll Learn
- Key database metrics to monitor
- Alerting strategy for database health
- Slow query analysis tooling

### Why This Matters

You cannot optimise what you don't measure. Good monitoring enables proactive performance management instead of reactive firefighting.

#### 🟢 For Beginners: The 5 Essential Database Metrics

Think of these as vital signs for your database:

| Metric | Analogy | Normal | Alert |
|--------|---------|--------|-------|
| QPS (queries/sec) | Heartbeat | Baseline ± 30% | > 2x normal |
| Query latency (p99) | Blood pressure | < 50ms | > 500ms |
| Active connections | Breathing rate | < 80% of max | > 90% |
| Cache hit rate | Efficiency score | > 90% | < 80% |
| Disk usage | Body weight | < 70% | > 85% |

#### 🟡 For Intermediate: Prometheus + Grafana Setup

**Key PostgreSQL metrics to collect (via postgres_exporter):**

```yaml
# prometheus.yml scrape config
scrape_configs:
  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
```

**Essential Grafana dashboards:**

```text
Dashboard 1: Database Health Overview
  - QPS (reads + writes) over time
  - P50, P95, P99 query latency
  - Active connections vs max_connections
  - Cache hit ratio (shared_buffers)
  - Replication lag (seconds)

Dashboard 2: Slow Query Analysis
  - Top 10 queries by total time
  - Top 10 queries by avg time
  - Queries causing most lock waits
  - Table/index bloat trend

Dashboard 3: Storage & Growth
  - Table sizes (top 20 tables)
  - Index sizes vs data sizes
  - Dead tuple percentages
  - Disk IOPS utilisation
```

**Alerting rules:**

```yaml
# prometheus alert rules
groups:
  - name: database
    rules:
    - alert: DatabaseHighLatency
      expr: histogram_quantile(0.99, rate(pg_query_duration_seconds_bucket[5m])) > 0.5
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "P99 query latency > 500ms"

    - alert: DatabaseConnectionsHigh
      expr: pg_stat_activity_count / pg_settings_max_connections > 0.85
      for: 2m
      labels:
        severity: critical

    - alert: ReplicationLagHigh
      expr: pg_replication_lag_seconds > 30
      for: 1m
      labels:
        severity: warning
```

#### 🔴 For Advanced: pg_stat_statements and Slow Query Analysis

**pg_stat_statements — the most important PostgreSQL extension:**

```sql
-- Enable (add to postgresql.conf)
-- shared_preload_libraries = 'pg_stat_statements'
CREATE EXTENSION pg_stat_statements;

-- Top 10 queries by total execution time (identify biggest consumers)
SELECT
    LEFT(query, 100) AS query_snippet,
    calls,
    ROUND(total_exec_time::NUMERIC, 2) AS total_ms,
    ROUND(mean_exec_time::NUMERIC, 2) AS mean_ms,
    ROUND(stddev_exec_time::NUMERIC, 2) AS stddev_ms,
    rows,
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS cache_hit_pct
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Queries with highest cache miss rate (candidates for index improvement)
SELECT
    LEFT(query, 100) AS query_snippet,
    calls,
    ROUND(mean_exec_time::NUMERIC, 2) AS mean_ms,
    shared_blks_read,     -- blocks read from disk
    shared_blks_hit,      -- blocks read from cache
    100.0 * shared_blks_hit / NULLIF(shared_blks_hit + shared_blks_read, 0) AS cache_hit_pct
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC
LIMIT 10;
```

**Lock Monitoring:**

```sql
-- Find blocking queries
SELECT
    pid,
    query_start,
    state,
    wait_event_type,
    wait_event,
    LEFT(query, 100) AS query,
    pg_blocking_pids(pid) AS blocked_by
FROM pg_stat_activity
WHERE cardinality(pg_blocking_pids(pid)) > 0;

-- Kill long-running queries (> 5 minutes)
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE state != 'idle'
AND query_start < NOW() - INTERVAL '5 minutes'
AND query NOT LIKE '%pg_stat_activity%';
```

📊 **Example — Datadog's Database Monitoring:**
- Tracks 15M+ metrics/second across customer databases
- Anomaly detection alerts on unusual query patterns
- Automatic index recommendations based on query patterns
- Integration: pg_stat_statements → Datadog APM → distributed tracing

#### Key Takeaways
- Monitor the 5 vital signs: QPS, latency p99, connections, cache hit rate, disk usage
- `pg_stat_statements` is the single most valuable tool for query performance analysis
- Replication lag is a critical alert — lag > 30 seconds means potential RPO breach
- Lock waits are often the hidden cause of latency spikes

---

## Section 14: Trade-Offs & Design Decisions

### What You'll Learn
- The CAP theorem in database decisions
- Consistency vs availability trade-offs
- Making and defending database choices in interviews

### Why This Matters

System design interviews often hinge on trade-off discussions. Being able to articulate "I chose X over Y because of constraints A, B, C" demonstrates senior-level thinking.

#### 🟢 For Beginners: The Core Trade-Offs

**Everything in databases is a trade-off:**

| If you want... | You sacrifice... |
|---------------|-----------------|
| Faster reads | Slower writes (more indexes) |
| Faster writes | Slower reads (fewer indexes) |
| Strong consistency | Higher latency (sync replication) |
| Lower latency | Weaker consistency (async replication) |
| Horizontal scalability | Simpler queries (no cross-shard JOINs) |
| Rich query capability | Vertical scalability limit |
| Less storage | CPU overhead (compression) |
| Schema flexibility | Query performance (NoSQL) |

#### 🟡 For Intermediate: The CAP Theorem Applied

**CAP Theorem: you can only have 2 of 3:**
- **C**onsistency: every read sees the latest write
- **A**vailability: every request gets a non-error response
- **P**artition tolerance: system works when network splits occur

In distributed systems, network partitions are unavoidable. So the real choice is **CP vs AP**:

| System | Choice | Example |
|--------|--------|---------|
| PostgreSQL (single node) | CA (no partition) | Ideal for OLTP |
| CockroachDB, Spanner | CP | Financial transactions |
| Cassandra, DynamoDB | AP | Shopping carts, social feeds |
| Riak, Couchbase | AP (tunable) | Session data |

**Interview Decision Script:**
> "For a payment system, I'd choose CP (consistency over availability). A user would rather see an error 'try again' than have money debited twice due to a split-brain scenario. I'd use PostgreSQL with synchronous replication, accepting the small latency increase for strong consistency guarantees."

#### 🔴 For Advanced: Practical Consistency Levels

**PACELC Extension (more practical than CAP):**

Even without partitions, there's a trade-off between latency and consistency:

| Database | Partition handling | Normal operation | Use case |
|---------|-------------------|-----------------|---------|
| Spanner | CP (sacrifice availability) | Low latency + consistency | Finance |
| DynamoDB | AP (sacrifice consistency) | Low latency + availability | Shopping cart |
| Cassandra | AP (tunable) | User-chosen per-query | IoT, time-series |
| PostgreSQL + sync replica | CP | Higher latency | OLTP |
| PostgreSQL + async replica | AP | Low latency + eventual | Most web apps |

**Read Consistency Levels (Cassandra as example):**

```text
Consistency level ONE:
→ Response returns after 1 replica responds
→ Latency: lowest
→ Risk: may read stale data

Consistency level QUORUM:
→ Response returns after majority of replicas respond (2 of 3)
→ Latency: medium
→ Guarantee: consistent with QUORUM writes

Consistency level ALL:
→ Response returns after ALL replicas respond
→ Latency: highest
→ Risk: any single replica failure = request failure
```

**When to Accept Eventual Consistency:**

```text
✅ Acceptable:
- Social media likes/views counters (±1 doesn't matter)
- Product inventory display (slightly stale is fine)
- Recommendation feeds
- Activity logs

❌ Not acceptable:
- Bank account balances
- Inventory deduction (must prevent overselling)
- User authentication (security-critical)
- Financial transactions
```

#### Key Takeaways
- Every database decision is a trade-off — know what you're giving up
- CAP theorem: in a network partition, choose between consistency (CP) or availability (AP)
- Financial and safety-critical systems need CP; most user-facing features can tolerate AP
- Eventual consistency is powerful — know precisely when it is and isn't acceptable

---

## Section 15: Interview Preparation Framework

### What You'll Learn
- A structured approach to database questions in system design interviews
- How to handle follow-up questions on database optimisation
- Common interview mistakes and how to avoid them

### Why This Matters

Knowing the concepts is half the battle. Knowing how to communicate them clearly in a 45-minute interview is the other half.

#### 🟢 For Beginners: The 5-Step Database Interview Framework

When asked about a database in an interview, always follow these 5 steps:

```text
Step 1: REQUIREMENTS (2 minutes)
  "What is the read/write ratio?"
  "What consistency level do we need?"
  "What are the primary query patterns?"

Step 2: SCALE ESTIMATE (2 minutes)
  "Let me estimate QPS, storage, and connections needed..."
  [Do the math out loud]

Step 3: SCHEMA (3 minutes)
  "Here are the key tables and relationships..."
  [Draw or list tables with key columns]

Step 4: INDEXES (2 minutes)
  "For this access pattern, I'd add these indexes..."
  [Justify each index]

Step 5: SCALING (3 minutes)
  "Starting with one PostgreSQL, I'd add caching first, then read replicas,
   then sharding only if needed..."
  [Justify each step]
```

#### 🟡 For Intermediate: Common Interview Questions and Answers

**Q: "How would you scale a database to 1M QPS?"**

> "I'd approach this in layers:
> 1. **Query optimisation first** — many systems at '1M QPS' are actually 100K QPS because of N+1 problems
> 2. **Caching** — Redis cluster in front of DB can absorb 90-95% of reads (brings effective DB QPS to 50-100K)
> 3. **Read replicas** — 5-10 replicas handles 95% of the remaining reads
> 4. **Connection pooling** — PgBouncer prevents connection exhaustion
> 5. **Sharding** — only if we've exhausted the above and write QPS is the bottleneck"

**Q: "How do you handle database migrations in production?"**

> "I use the expand-contract pattern:
> 1. Expand: add new column as nullable, deploy code that writes both old and new
> 2. Migrate: backfill in batches (1000 rows at a time to avoid lock contention)
> 3. Contract: add NOT NULL constraint, remove old column in a future deploy
> Zero downtime because each step is backward compatible."

**Q: "When would you use NoSQL instead of SQL?"**

> "I default to PostgreSQL for most use cases because its flexibility and ACID guarantees
> prevent many bugs. I'd reach for NoSQL when:
> - Write throughput exceeds what a single PostgreSQL instance can handle (Cassandra)
> - The data model is truly key-value with no complex queries (Redis, DynamoDB)
> - Schema flexibility is genuinely needed and queries don't require JOINs (MongoDB)
> But I'd validate each assumption — many teams switch to NoSQL and then miss SQL."

**Q: "How do you find and fix slow queries?"**

> "My process:
> 1. Find via pg_stat_statements — sort by total_exec_time DESC
> 2. Run EXPLAIN ANALYZE on the top offenders
> 3. Look for Seq Scans on large tables, Nested Loops on large datasets
> 4. Add indexes CONCURRENTLY (non-blocking)
> 5. Rewrite N+1 queries as JOINs
> 6. Verify improvement with before/after EXPLAIN ANALYZE
> 7. Monitor query latency in production"

#### 🔴 For Advanced: Troubleshooting Scenarios

**Scenario 1: Production database suddenly at 100% CPU**

```text
Investigation steps:

1. Identify the culprit (< 30 seconds):
   SELECT pid, query, state, query_start
   FROM pg_stat_activity
   WHERE state = 'active'
   ORDER BY query_start ASC;

2. Check for lock waits:
   SELECT pid, wait_event, query FROM pg_stat_activity WHERE wait_event IS NOT NULL;

3. If runaway query found: pg_terminate_backend(pid)

4. Root cause analysis:
   - New deploy with missing index? (check pg_stat_statements — new query with many rows)
   - Autovacuum running on large table? (check pg_stat_activity for VACUUM)
   - Statistics stale? (run ANALYZE on affected tables)
   - Connection storm? (check pg_stat_activity — many connections in 'idle in transaction')

5. Permanent fix:
   - Add missing index
   - Kill idle-in-transaction with idle_in_transaction_session_timeout = '5min'
   - Tune autovacuum
```

**Scenario 2: Replication lag growing out of control**

```text
Diagnosis:
  SELECT now() - pg_last_xact_replay_timestamp() AS replication_lag FROM pg_stat_replication;

Causes:
  1. Long-running transaction on primary (holding WAL) → find and kill
  2. High write load → increase max_wal_senders, wal_sender_timeout
  3. Network bottleneck between primary and replica → check network throughput
  4. Replica disk slow → check I/O stats, upgrade to NVMe

If lag unrecoverable:
  → pg_basebackup: rebuild replica from primary (requires downtime for that replica)
  → Use WAL-E/pgBackRest for continuous archiving to S3 as cold standby
```

**Architecture Evolution Story (Interview Framework):**

```text
Interview question: "How would you design the database layer for Twitter?"

Stage 1 (0-10K users): Single PostgreSQL — tweets, users, follows all in one DB
Stage 2 (10K-1M users): Add Redis cache + 1 read replica
Stage 3 (1M-10M users): Shard by user_id (8 PostgreSQL shards), Elasticsearch for search
Stage 4 (10M-100M users): 64 shards, Cassandra for timeline (fan-out), Redis cluster
Stage 5 (100M+ users): 1000+ shards, multi-region active-passive, Kafka CDC
```

#### Key Takeaways
- Always ask clarifying questions before jumping to a solution
- Start simple — add complexity only when justified by scale estimates
- Know how to diagnose live production issues, not just design systems
- Frame every decision as a trade-off with clear reasoning

---

## Section 16: Putting It All Together

### 🎯 The Complete Database Optimisation Checklist

Use this checklist when optimising any database:

```text
SCHEMA
[ ] Tables are normalised to 3NF (denormalise only where measured)
[ ] Price/quantity snapshots in order items (immutable records)
[ ] Timestamps use TIMESTAMP WITH TIME ZONE
[ ] Integer primary keys (not UUID unless globally distributed required)
[ ] NOT NULL constraints where applicable
[ ] CHECK constraints for enums/ranges

INDEXING
[ ] All foreign key columns are indexed
[ ] All columns in frequent WHERE clauses are indexed
[ ] Composite indexes follow leftmost-prefix rule
[ ] Covering indexes for top-5 hottest queries
[ ] Partial indexes for filtered queries (e.g., WHERE status = 'ACTIVE')
[ ] No functions on indexed columns in WHERE clauses
[ ] Unused indexes are removed (check idx_scan = 0)

QUERIES
[ ] No SELECT * (specify columns)
[ ] No OFFSET pagination on large tables (use cursor-based)
[ ] No correlated subqueries (rewrite as JOINs)
[ ] No implicit type conversions in WHERE clauses
[ ] No leading wildcards in LIKE (use full-text index instead)
[ ] N+1 query problems eliminated (use JOINs or DataLoader)

REPLICATION & HA
[ ] Read replicas for read scaling
[ ] Automated failover configured (Patroni/Orchestrator)
[ ] Failover tested in the last 3 months
[ ] RPO and RTO documented and validated
[ ] Replication lag monitored with alert

CACHING
[ ] Redis cache in front of hot read paths
[ ] TTL jitter applied to prevent cache avalanche
[ ] Cache invalidation strategy defined
[ ] Cache hit rate > 90% on hot data
[ ] Application handles cache-miss gracefully (no thundering herd)

CONNECTIONS
[ ] PgBouncer (or equivalent) in front of database
[ ] Connection pool sized correctly (core_count * 2 + spindle_count)
[ ] idle_in_transaction_session_timeout set

MONITORING
[ ] pg_stat_statements enabled
[ ] Slow query log enabled (> 1 second)
[ ] Replication lag dashboard
[ ] Autovacuum monitoring
[ ] Alerts for p99 latency > 500ms, connections > 90%, replication lag > 30s

SECURITY
[ ] Parameterised queries everywhere
[ ] Least-privilege database users
[ ] PII encrypted at column level
[ ] Database in private subnet (no public IP)
[ ] SSL enforced for all connections
```

### 🏗️ Reference Architecture: Production PostgreSQL Stack

```text
Internet
    │
    ▼
[Load Balancer (Nginx/HAProxy)]
    │
    ├──► [Application Servers (× 10)]
    │            │ (read/write)
    │            ▼
    │    [PgBouncer connection pool]
    │         │          │
    │    (writes)    (reads, round-robin)
    │         │          │
    │         ▼          ▼
    │    [PostgreSQL  [Read Replica 1]
    │     Primary]   [Read Replica 2]  ← auto-promoted on primary failure
    │                [Read Replica 3 (DR, different AZ)]
    │
    └──► [Redis Cluster (3 primary + 3 replica)]
              │
              └── Cache-aside for: users, products, sessions, computed aggregates

Monitoring:
    postgres_exporter → Prometheus → Grafana → PagerDuty alerts
    Application APM → Datadog/New Relic → slow query attribution
    
Backup:
    WAL-E/pgBackRest → S3 (continuous WAL archiving + daily base backup)
    Point-in-time recovery (PITR): any moment in the last 7 days
```

### 📊 Quick Reference: Numbers to Know for Interviews

| Metric | Number |
|--------|--------|
| Single PostgreSQL read QPS (indexed) | ~50K |
| Single PostgreSQL write QPS | ~10K |
| PgBouncer connection multiplier | 100-200x |
| Redis QPS (single instance) | ~1M ops/sec |
| Redis latency | ~0.1ms |
| B-tree index lookup disk I/Os | 3-4 |
| Full table scan overhead vs index | 100x-1000x |
| Cache hit rate target | >90% |
| Async replication lag (same AZ) | 1-5ms |
| Async replication lag (cross-region) | 50-100ms |
| PostgreSQL WAL segment size | 16MB |
| Connection overhead (PostgreSQL) | ~5MB/connection |
| Checkpoint interval (default) | 5 minutes |

### 🎓 Learning Path Forward

**To deepen your database knowledge:**

1. **Hands-on practice:** Set up a PostgreSQL instance locally, load 10M rows of fake data, benchmark queries with and without indexes using `pgbench`
2. **Read the source:** "Designing Data-Intensive Applications" by Martin Kleppmann — the definitive book
3. **Database internals:** "Database Internals" by Alex Petrov — B-trees, LSM trees, consensus
4. **Practice query tuning:** Use `EXPLAIN ANALYZE` on real queries in your projects
5. **Run a failure drill:** Practice PostgreSQL failover with Patroni in a test environment
6. **Monitor real systems:** Set up pg_stat_statements + Grafana dashboard on any PostgreSQL you have access to

---

## 📚 Resources

### Essential Reading
- **"Designing Data-Intensive Applications"** — Martin Kleppmann (the bible of distributed databases)
- **"Database Internals"** — Alex Petrov (deep dive into B-trees, LSM, consensus)
- **PostgreSQL Official Documentation** — always up-to-date, excellent quality
- **"High Performance MySQL"** — Baron Schwartz (MySQL-specific but broadly applicable)

### Online Tools
- **explain.dalibo.com** — visualise PostgreSQL EXPLAIN plans
- **pgbadger** — slow query log analyser for PostgreSQL
- **pgbench** — built-in PostgreSQL benchmarking tool
- **USE Method** — Brendan Gregg's methodology for resource utilisation analysis

### Real-World Case Studies
- Instagram Engineering Blog: "Sharding & IDs at Instagram"
- GitHub Engineering Blog: "How we make GitHub fast"  
- Shopify Engineering Blog: "Upgrading GitHub to MySQL 8.0"
- Cloudflare Blog: "How we run PostgreSQL at scale"

---

*This guide covers the full spectrum from database internals to production operations. Revisit sections as you encounter real-world database challenges — the concepts become clearer with hands-on experience.*

*Last Updated: August 2026*
