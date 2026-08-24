# Database Optimisation — Interview Quick Reference

**Difficulty:** ⭐⭐⭐⭐ Hard  
**Tags:** `Database Design`, `Indexing`, `Query Optimization`, `Sharding`, `Replication`, `Caching`, `Performance`, `High Availability`

---

## 🎯 The 5 Questions to Ask First

1. **Read/write ratio?** (10:1 read-heavy vs 1:10 write-heavy changes everything)
2. **Consistency requirement?** (Strong = sync replication; eventual = async + higher perf)
3. **Data model?** (Relational, document, graph, time-series?)
4. **Scale?** (QPS, storage GB, latency SLA, DAU)
5. **Primary access patterns?** (Point lookups, range queries, aggregations, full-text?)

---

## 📊 Numbers to Know Cold

| Metric | Value |
|--------|-------|
| PostgreSQL read QPS (single, indexed) | ~50,000 |
| PostgreSQL write QPS (single) | ~10,000 |
| Redis ops/sec (single instance) | ~1,000,000 |
| Redis latency | ~0.1ms |
| B-tree index: disk I/Os per lookup | 3–4 |
| Full scan vs index scan overhead | 100x–1000x |
| PgBouncer: client vs DB connections | 100–200x multiplier |
| Async replication lag (same AZ) | 1–5ms |
| Async replication lag (cross-region) | 50–100ms |
| Target cache hit rate | >90% |
| Connection overhead (PostgreSQL) | ~5MB/connection |

---

## ⚡ Optimisation Priority Order

```text
1. Schema design (normalisation, proper types)      ← Highest ROI
2. Indexes (cover hot queries, remove unused)
3. Query rewrites (eliminate N+1, bad JOINs)
4. Caching (Redis in front of hot reads)
5. Connection pooling (PgBouncer)
6. DB configuration (shared_buffers, work_mem)
7. Read replicas (scale reads horizontally)
8. Sharding (last resort — massive complexity)
```

---

## 🗂️ Index Quick Guide

### Choose index type by query:
| Query type | Index type |
|-----------|-----------|
| `WHERE col = ?` | B-tree (default) or Hash |
| `WHERE col > ? AND col < ?` | B-tree |
| `WHERE col LIKE 'prefix%'` | B-tree |
| `WHERE col LIKE '%suffix'` | Full-text (GIN) |
| `WHERE ts_vector @@ query` | GIN (full-text) |
| JSON/array contains | GIN |
| Geospatial | GiST / PostGIS |

### Composite index column order:
```text
(equality filter columns) → (range filter columns) → (included columns)

Example: WHERE user_id = 42 AND status = 'PENDING' AND created_at > '2024-01-01'
→ CREATE INDEX ON orders(user_id, status, created_at)
```

### Red flags (index killers):
```sql
WHERE DATE(created_at) = '2024-01-01'   -- ❌ function on column
WHERE LOWER(email) = 'alice@...'         -- ❌ function on column
WHERE id = '42'                          -- ❌ type mismatch (id is INT)
WHERE name LIKE '%john%'                 -- ❌ leading wildcard
```

---

## 🔍 Query Anti-Patterns → Fixes

| Anti-Pattern | Fix |
|-------------|-----|
| `SELECT *` | Select only needed columns |
| `OFFSET 900000 LIMIT 10` | Cursor-based pagination (keyset) |
| Correlated subquery per row | Rewrite as JOIN + aggregation |
| `OR` across different columns | Use `UNION ALL` |
| N+1 (loop + query per item) | JOIN with IN clause or batch fetch |
| `LIKE '%term%'` on large table | Elasticsearch or GIN full-text index |

---

## 🔁 Replication Cheat Sheet

```text
Async replication:  Primary → Replica (no wait)
  ✅ Low latency   ❌ Data loss on crash (RPO > 0)

Sync replication:   Primary → waits for 1 replica ACK
  ✅ No data loss  ❌ Latency increases by replica RTT

Semi-sync (MySQL):  Primary → waits for 1 of N replicas
  ✅ Balance       ⚠️ MySQL-specific
```

**Failover targets (with Patroni/automated HA):**

| SLA | Architecture |
|-----|-------------|
| RPO < 5 min, RTO < 30 min | Async + manual failover |
| RPO < 1 min, RTO < 5 min  | Async + Patroni auto-failover |
| RPO = 0, RTO < 30 sec     | Sync + Patroni + fencing |
| RPO = 0, RTO < 10 sec (global) | CockroachDB / Spanner |

---

## 🗄️ Sharding Decision

**Shard only when:**
- Single primary can't handle write QPS after all other optimisations
- Storage exceeds single server capacity
- Latency SLA requires geographic distribution

**Shard key rules:**
- High cardinality (many distinct values)
- Even distribution (no hotspots)
- Matches primary access pattern
- Avoid cross-shard JOINs for common queries

| Strategy | Pros | Cons |
|---------|------|------|
| Hash (user_id % N) | Even distribution | No range queries, hard resharding |
| Range | Range queries stay local | Hotspots on recent data |
| Consistent hashing | Minimal data moved on reshard | More complex routing |
| Directory | Full control | Extra lookup, SPOF routing table |

---

## 💾 Caching Patterns

```text
Cache-Aside (most common):
  Read: check cache → miss → DB → populate cache
  Write: update DB → delete/update cache key

Write-Through:
  Write: DB + cache simultaneously
  Use when: stale reads unacceptable

Write-Behind:
  Write: cache only → async flush to DB
  Use when: write-heavy, tolerate brief loss risk (gaming scores, IoT)
```

**Prevent cache failures:**
```python
# TTL jitter: prevent cache avalanche
ttl = base_ttl + random.randint(-base_ttl//10, base_ttl//10)

# Lock: prevent thundering herd on cache miss
if cache.setnx(f"lock:{key}", "1"):
    value = fetch_from_db()
    cache.setex(key, ttl, value)
```

---

## 🗃️ Database Selection Matrix

| Workload | Best Database |
|---------|--------------|
| OLTP, complex queries, ACID | PostgreSQL |
| Write-heavy, time-series, wide rows | Cassandra |
| Key-value, sessions, caching | Redis |
| Full-text search, log analytics | Elasticsearch |
| OLAP analytics (aggregate large datasets) | ClickHouse / BigQuery |
| Document / flexible schema | MongoDB |
| Graph (social network, recommendations) | Neo4j |
| Global ACID at scale | CockroachDB / Spanner |

**Default rule:** Use PostgreSQL unless you have a *measured* reason not to.

---

## 🔒 Security Essentials

```python
# ALWAYS use parameterised queries
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))  # ✅
query = f"SELECT * FROM users WHERE id = {user_id}"              # ❌ SQL injection

# Least privilege
GRANT SELECT, INSERT ON orders TO app_user;    # not superuser
REVOKE DELETE ON users FROM app_user;

# Row-level security (PostgreSQL)
ALTER TABLE patient_records ENABLE ROW LEVEL SECURITY;
CREATE POLICY access ON patient_records USING (doctor_id = current_user_id());
```

---

## 📈 Monitoring Checklist

**5 vital signs to alert on:**

| Metric | Warning | Critical |
|--------|---------|---------|
| P99 query latency | > 200ms | > 500ms |
| Active connections | > 80% of max | > 90% |
| Cache hit rate | < 85% | < 70% |
| Replication lag | > 10s | > 30s |
| Disk usage | > 70% | > 85% |

**Find slow queries:**
```sql
-- pg_stat_statements: top consumers
SELECT LEFT(query, 80), calls, ROUND(mean_exec_time::numeric, 2) AS mean_ms
FROM pg_stat_statements
ORDER BY total_exec_time DESC LIMIT 10;
```

---

## 🏗️ Reference Architecture: PostgreSQL Production Stack

```text
App Servers (×10)
     ↓
PgBouncer (pool: 50 connections) ← multiplexes 10K app conns
     ├── Primary (writes)
     └── Read Replica 1 / 2 / 3 (reads, round-robin)
              ↑ auto-failover via Patroni + etcd

Redis Cluster (3+3) ← absorbs ~90% of reads

Monitoring: pg_stat_statements → Prometheus → Grafana → PagerDuty
Backup: pgBackRest → S3 (continuous WAL + daily base)
```

---

## 💬 Interview Script Templates

**"How would you scale this database to 10M QPS?"**
> "I'd work through the optimisation hierarchy: first verify queries and indexes are optimal (most teams think they need 10M QPS but actually have 100K QPS with N+1 problems), then add Redis caching to absorb 90%+ of reads, then read replicas for the remainder. Sharding only if write QPS is the bottleneck after exhausting everything else."

**"SQL vs NoSQL?"**
> "I default to PostgreSQL — it handles 80% of use cases well and its flexibility prevents future migration pain. I'd reach for Cassandra for write-heavy time-series at scale, Redis for sub-millisecond key-value caching, and Elasticsearch for full-text search. Each addition is justified by a specific, measured limitation of PostgreSQL."

**"How do you handle schema migrations in production?"**
> "Expand-contract pattern: (1) Add new column as nullable + deploy code writing to both; (2) Backfill old rows in batches; (3) Make NOT NULL; (4) Drop old column in next release. Zero downtime because each step is backward compatible."

---

*Full guide: `database_optimization_system_design.md`*
