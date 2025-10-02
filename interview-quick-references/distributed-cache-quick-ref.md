# Distributed Cache - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a distributed in-memory cache (Redis/Memcached) supporting 1M QPS with sub-millisecond latency.

---

## 🎯 Core Problem Statement

- **What:** Design a high-performance distributed in-memory key-value cache
- **Key Challenge:** Achieve sub-millisecond latency at scale with efficient memory usage
- **Scale:** 1M QPS, <1ms p99 latency, 1TB memory distributed, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Read QPS | 1M reads/sec | Target throughput |
| Write QPS | 100K writes/sec | 10:1 read-write ratio |
| p99 Latency | <1ms | In-memory performance |
| Memory per Node | 64-128GB | RAM limits |
| Total Cluster Memory | 1TB | Distributed across nodes |
| Eviction Rate | 10-20% | Cache churn |
| Cache Hit Rate | >80% | Effectiveness target |
| Key Size | 250 bytes avg | Typical web cache |
| Value Size | 1KB avg | Serialized objects |
| TTL Default | 1 hour | Expiration policy |

## 🏗️ High-Level Architecture

```text
[Application] → [Cache Client (Consistent Hash)]
                        ↓
          [Cache Node 1] [Cache Node 2] ... [Cache Node N]
                        ↓
              [LRU Eviction Engine]
                        ↓
              [Optional Persistence]
           [RDB Snapshots | AOF Log]
```

**Components:**

- **Cache Client**: Smart client with consistent hashing, connection pooling
- **Cache Cluster**: 10-100 nodes, sharded by key hash
- **Eviction Engine**: LRU/LFU algorithm for memory management
- **Persistence**: Optional RDB snapshots + AOF for durability
- **Replication**: Master-replica for read scaling and availability

## 💾 Data Model (Essentials)

### Data Structures

```text
String: SET/GET (simple key-value)
List: LPUSH/RPUSH/LRANGE (queues, timelines)
Set: SADD/SREM/SISMEMBER (unique items)
Sorted Set: ZADD/ZRANGE (leaderboards, rankings)
Hash: HSET/HGET (objects, nested data)
```

### Memory Layout

```text
Key: String (max 512MB in Redis)
Value: Binary-safe (max 512MB)
Metadata: TTL, LRU timestamp, encoding
Memory overhead: ~50 bytes per key-value
```

## 🔌 API Design (Key Commands)

| Operation | Command | Time Complexity |
|-----------|---------|-----------------|
| Set | `SET key value [EX seconds]` | O(1) |
| Get | `GET key` | O(1) |
| Delete | `DEL key` | O(1) |
| Increment | `INCR key` | O(1) |
| Batch Set | `MSET k1 v1 k2 v2` | O(N) |
| Batch Get | `MGET k1 k2 k3` | O(N) |
| Expire | `EXPIRE key seconds` | O(1) |
| Check Exists | `EXISTS key` | O(1) |

## 🚀 Critical Talking Points

### Point 1: Consistent Hashing for Sharding

- **What:** Distribute keys across cache nodes using hash ring
- **Why:** Minimize data movement when nodes added/removed
- **Detail:**
  - Hash key with CRC32/MD5, map to ring position
  - Virtual nodes (100-200 per physical) for even distribution
  - Only 1/N keys rehashed when node added
  - Client-side sharding for performance
- **Alternative:** Modulo hashing (all keys rehashed), proxy-based (extra hop)

### Point 2: LRU Eviction Policy

- **What:** Least Recently Used eviction when memory limit reached
- **Why:** Keep hot data in cache, remove cold data automatically
- **Detail:**
  - LRU list ordered by access time
  - On access: Move to head of list
  - On eviction: Remove from tail
  - Redis optimization: Sample-based LRU (not true LRU for performance)
- **Alternative:** LFU (frequency), FIFO (order), Random (simple but ineffective)

### Point 3: Cache Aside Pattern

- **What:** Application manages cache population (not write-through)
- **Why:** Gives application control, prevents cache pollution
- **Detail:**

  ```text
  1. Check cache for key
  2. If miss: Query database
  3. Store result in cache with TTL
  4. On write: Invalidate cache, write to DB
  ```

- **Alternative:** Write-through (slower writes), write-behind (complexity)

### Point 4: Replication for Availability

- **What:** Master-replica architecture for read scaling and failover
- **Why:** Increase read capacity and survive node failures
- **Detail:**
  - Async replication from master to replicas
  - Reads can go to replicas (eventually consistent)
  - Writes always to master
  - Automatic failover with Sentinel/Cluster
- **Alternative:** No replication (data loss), synchronous replication (slow)

### Point 5: Persistence Options

- **What:** RDB snapshots + AOF log for durability
- **Why:** Recover cache on restart, prevent cold start
- **Detail:**
  - RDB: Point-in-time snapshot (fork + COW), every 5-15 min
  - AOF: Append-only log of write commands, fsync every second
  - Hybrid: RDB for bulk, AOF for recent changes
  - Trade-off: Durability vs performance
- **Alternative:** No persistence (volatile cache), full sync replication

### Point 6: Connection Pooling

- **What:** Reuse TCP connections to cache nodes
- **Why:** Avoid connection overhead (TCP handshake ~1ms)
- **Detail:**
  - Pool size: 10-50 connections per app server
  - Idle timeout: Close unused connections after 5 min
  - Health checks: Ping before borrowing connection
  - Benefit: 50-100x throughput improvement
- **Alternative:** One connection per request (slow), single connection (bottleneck)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Sharding | Consistent Hashing | Proxy / Range | Minimize rehashing, client-side performance |
| Eviction | LRU | LFU / TTL-only | Good balance of recency and frequency |
| Consistency | Eventually Consistent | Strong | Cache is read-heavy, eventual is acceptable |
| Persistence | Optional (RDB+AOF) | None / Always | Durability vs pure speed trade-off |
| Replication | Async Master-Replica | Sync / None | Read scaling without write penalty |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution | Monitoring |
|------------|----------|-----------|
| Hot Keys | Replicate heavily accessed keys, local caching | Key access frequency |
| Memory Pressure | Increase eviction, add nodes, optimize data structures | Memory usage, eviction rate |
| Network Bandwidth | Pipeline commands, use compression, batch operations | Network I/O, throughput |
| Single-Threaded (Redis) | Multiple instances per host, cluster mode | CPU usage, command latency |
| Connection Limits | Connection pooling, multiplexing | Active connections, pool exhaustion |

## 💡 Interview Tips

**Start Here:**

1. Clarify use case (session cache, DB query cache, etc.)
2. Calculate memory needed (keys × value_size)
3. Draw sharding architecture with consistent hashing
4. Discuss eviction policy

**Emphasize:**

- Consistent hashing for scalability
- Cache aside pattern (application-managed)
- LRU eviction for memory management
- Replication for availability
- Sub-millisecond latency from in-memory

**Be Ready For:**

- "How do you handle cache invalidation?" → TTL + explicit invalidation on writes
- "What if cache node fails?" → Replicas serve requests, consistent hashing routes around
- "How do you prevent cache stampede?" → Locking, probabilistic early expiration
- "What eviction policy?" → LRU (or discuss LFU vs LRU trade-offs)
- "How to scale reads?" → Add replicas for read scaling
- "How to scale writes?" → Shard across more master nodes

**Don't Forget:**

- Mention cache hit rate as key metric
- Discuss TTL strategy (prevents stale data)
- Address cold start problem (cache warming)
- Talk about cache aside vs write-through
- Consider thundering herd problem

**Common Mistakes:**

- Using modulo hashing (breaks on node changes)
- No eviction policy (memory explosion)
- Ignoring connection pooling (huge performance impact)
- Synchronous replication (defeats purpose of cache)
- No monitoring of cache hit rate

**Cache Patterns:**

```text
Cache Aside (Read-Through):
1. Check cache → If hit, return
2. If miss → Query DB → Store in cache → Return

Write-Through:
1. Write to cache
2. Synchronously write to DB
3. Return success

Write-Behind (Write-Back):
1. Write to cache
2. Asynchronously write to DB (batch)
3. Return success immediately
```

**Key Formulas:**

```text
Cache Hit Rate = (Cache Hits / Total Requests) × 100%
Target: >80% for effectiveness

Memory Needed = (Keys × (Key_Size + Value_Size + Overhead)) / Hit_Rate
Example: 1B keys × 1KB × 1.5 / 0.8 = 1.875TB

Eviction Rate = Evictions / Time_Window
Monitor: Should be < 20% of insertions
```

**Performance Tips:**

- Use pipelining for batch operations (10-100x faster)
- Compress large values (CPU vs network trade-off)
- Use appropriate data structures (Hash vs String)
- Set appropriate TTL (balance freshness vs load)
- Monitor slow queries (>1ms operations)

**Failure Scenarios:**

- Cache miss storm → Rate limit DB queries, serve stale data
- Cache node crash → Replicas take over, slight latency increase
- Network partition → Serve from other nodes, some misses
- Memory full → Eviction kicks in, hit rate drops
- Cold start → Gradual cache warming, DB load spike
