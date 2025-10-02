# Distributed Key-Value Store - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a distributed key-value store (DynamoDB/Cassandra) with 100K writes/sec and 500K reads/sec across multiple datacenters.

---

## 🎯 Core Problem Statement

- **What:** Design a highly available, partition-tolerant distributed key-value store
- **Key Challenge:** Maintain availability during partitions while handling conflicts and failures
- **Scale:** 100K writes/sec, 500K reads/sec, 10TB+ storage, 100+ nodes, multi-datacenter

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Write Throughput | 100K writes/sec | Given requirement |
| Read Throughput | 500K reads/sec | 5:1 read-to-write ratio |
| Daily Writes | 8.64B | 100K × 86,400 |
| Daily Reads | 43.2B | 500K × 86,400 |
| Storage (per key-value) | 1KB avg | Key (256B) + Value + metadata |
| Total Storage | 10TB+ | Across 100+ nodes |
| Replication Factor | 3 | N = 3 replicas |
| Quorum Reads (R) | 2 | R + W > N for consistency |
| Quorum Writes (W) | 2 | W = 2, N = 3 |
| Node Count | 100-1000 | Linear scaling |
| p99 Latency (Read) | <50ms | Performance target |
| p99 Latency (Write) | <100ms | Including replication |

## 🏗️ High-Level Architecture

```text
[Client] → [Coordinator Node]
               ↓
      [Consistent Hash Ring]
               ↓
    [Node 1] [Node 2] [Node 3] ... [Node N]
    (Replica 1, 2, 3 for each key)
               ↓
      [Gossip Protocol] ← Failure Detection
               ↓
     [Hinted Handoff] ← Temporary Failures
               ↓
      [Merkle Trees] ← Anti-entropy Repair
```

**Key Components:**

- **Coordinator Node**: Any node can coordinate requests, no master
- **Consistent Hashing**: Distribute keys evenly across nodes with virtual nodes
- **Replication**: N=3 replicas, quorum-based for consistency
- **Gossip Protocol**: Peer-to-peer failure detection (every second)
- **Vector Clocks**: Detect and resolve concurrent write conflicts
- **Hinted Handoff**: Queue writes for temporarily down nodes
- **Merkle Trees**: Compare replicas for anti-entropy repair

## 💾 Data Model (Essentials)

### Key-Value Structure

```text
Key: String (max 256 bytes)
Value: Binary blob (max 1MB)
Metadata:
├── vector_clock: [(node_id, counter), ...]
├── timestamp: Last write time
├── ttl: Optional expiration
└── version: Increment on each write
```

### Partitioning Strategy

```text
partition = hash(key) mod virtual_nodes
Virtual nodes per physical node: 100-200
Benefits: Even distribution, easy rebalancing
```

## 🔌 API Design (Key Endpoints)

| Method | Endpoint | Purpose | Consistency |
|--------|----------|---------|-------------|
| PUT | `/v1/kv/{key}` | Write key-value | Quorum (W=2) |
| GET | `/v1/kv/{key}` | Read value | Quorum (R=2) |
| DELETE | `/v1/kv/{key}` | Delete key | Quorum (W=2) |
| MGET | `/v1/kv?keys=k1,k2,k3` | Batch read | Parallel queries |

**Consistency Levels:**

- ONE: Fast but risky (1 replica)
- QUORUM: Balanced (R=2, W=2 with N=3)
- ALL: Slow but consistent (all replicas)

## 🚀 Critical Talking Points

### Point 1: Consistent Hashing with Virtual Nodes

- **What:** Distribute data using hash ring with 100-200 virtual nodes per physical node
- **Why:** Even distribution, easy node addition/removal without massive data movement
- **Detail:**
  - Hash function: MD5 or SHA-1 of key
  - Virtual nodes reduce load imbalance from 40% to <5%
  - When node added: Only adjacent nodes affected
  - Replication: Store on N successive nodes on ring
- **Alternative:** Range-based (manual rebalancing), hash-mod-N (complete reshuffling)

### Point 2: Quorum-Based Replication

- **What:** Configurable R (read) and W (write) quorums where R + W > N
- **Why:** Balance consistency and availability (tune for use case)
- **Detail:**
  - N=3 (replication factor), W=2, R=2 → Read-your-writes consistency
  - If W=1, R=3 → Fast writes, slower reads, eventual consistency
  - Coordinator waits for W responses before success
  - Sloppy quorum: Use next healthy nodes if primary replicas down
- **Alternative:** Master-slave (single point of failure), all-or-nothing (low availability)

### Point 3: Vector Clocks for Conflict Resolution

- **What:** Track causality of updates using per-node counters
- **Why:** Detect concurrent writes and resolve conflicts correctly
- **Detail:**
  - Vector clock: `{(A,1), (B,2)}` means node A wrote once, B wrote twice
  - Concurrent if neither clock is ancestor of other
  - Client resolves conflicts (or LWW - last-write-wins)
  - Garbage collection when clock grows too large
- **Alternative:** Timestamps (clock skew issues), versions only (can't detect concurrency)

### Point 4: Gossip Protocol for Failure Detection

- **What:** Peer-to-peer heartbeat where nodes randomly exchange membership info
- **Why:** Decentralized failure detection, no single point of failure
- **Detail:**
  - Every 1 second, each node picks 1-3 random nodes to gossip with
  - Exchange: Node states, heartbeat counters, suspected failures
  - Mark node as down after 10 failed heartbeats (10 seconds)
  - O(log N) convergence time for cluster-wide updates
- **Alternative:** Centralized monitor (SPOF), all-to-all (O(N²) messages)

### Point 5: Anti-Entropy with Merkle Trees

- **What:** Background process to sync replicas using hash trees
- **Why:** Repair divergent replicas efficiently without comparing all keys
- **Detail:**
  - Merkle tree: Hash tree where leaves are key ranges
  - Compare root hashes first, then drill down to different ranges
  - Only sync differing key ranges (efficient bandwidth)
  - Run periodically or on-demand after partition heals
- **Alternative:** Compare all keys (expensive), read repair only (incomplete)

### Point 6: Hinted Handoff

- **What:** Store writes for temporarily unavailable nodes on healthy nodes
- **Why:** Maintain availability during transient failures
- **Detail:**
  - Coordinator writes to next healthy node with "hint" (intended recipient)
  - When original node recovers, hinted node replays writes
  - TTL on hints (typically 3 hours) to prevent indefinite storage
  - Works with sloppy quorum for write availability
- **Alternative:** Fail writes (lower availability), client retries (complex)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| CAP Theorem | AP (Availability + Partition tolerance) | CP (Consistency + Partition) | E-commerce needs high availability > strict consistency |
| Partitioning | Consistent Hashing | Range-based / Hash-mod-N | Even distribution, easy scaling |
| Replication | Quorum (N=3, W=2, R=2) | Master-slave / All-or-nothing | Balance of consistency and availability |
| Conflict Resolution | Vector Clocks | Timestamps / Versions | Accurately detect concurrent writes |
| Failure Detection | Gossip Protocol | Centralized monitor | No SPOF, scalable |
| Consistency | Eventual (tunable) | Strong consistency | Better availability and performance |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution | Monitoring |
|------------|----------|-----------|
| Hot Keys | Replicate hot keys more, cache frequently accessed | Key access frequency, load distribution |
| Network Partitions | Sloppy quorum + hinted handoff | Partition detection, hint queue size |
| Large Vector Clocks | Prune old entries, set size limits | Clock size distribution |
| Coordinator Overload | Any node can coordinate, client-side load balancing | Request rate per node |
| Data Skew | More virtual nodes, monitor distribution | Data size per node, rebalancing frequency |

## 💡 Interview Tips

**Start Here:**

1. Clarify CAP choice (AP vs CP) - critical decision!
2. Calculate numbers: QPS, storage, nodes needed
3. Draw consistent hash ring diagram
4. Explain replication and quorum

**Emphasize:**

- CAP theorem application (AP for high availability)
- Consistent hashing with virtual nodes (scaling)
- Quorum-based replication (tunable consistency)
- Vector clocks (conflict resolution)
- Gossip protocol (failure detection)

**Be Ready For:**

- "How do you handle network partitions?" → Sloppy quorum + hinted handoff
- "How do you detect conflicts?" → Vector clocks for causality
- "How do you add new nodes?" → Consistent hashing redistributes adjacent ranges
- "What if coordinator fails?" → Any node can coordinate (no master)
- "How do you ensure consistency?" → Tunable quorums (R + W > N)
- "How do you detect failures?" → Gossip protocol (decentralized)
- "How do you repair replicas?" → Merkle trees + hinted handoff

**Don't Forget:**

- Explain CAP trade-off explicitly (AP vs CP choice)
- Mention virtual nodes (critical for even distribution)
- Discuss conflict resolution strategy (vector clocks)
- Address failure scenarios (node down, partition)
- Talk about anti-entropy mechanisms

**Common Mistakes:**

- Choosing CP when problem needs high availability
- Forgetting virtual nodes (causes load imbalance)
- Using timestamps for conflicts (clock skew)
- Not explaining quorum formula (R + W > N)
- Ignoring network partitions (must handle gracefully)

**Quorum Math:**

```text
For N=3 replicas:
├── W=1, R=3: Fast writes, eventual consistency
├── W=2, R=2: Balanced, read-your-writes
├── W=3, R=1: Fast reads, slow writes
└── W=2, R=1: Fast but inconsistent (R+W ≤ N)
```

**Key Algorithms:**

1. **Consistent Hashing**: O(log N) to find node (binary search on ring)
2. **Gossip Protocol**: O(log N) message complexity for convergence
3. **Vector Clocks**: O(number_of_nodes) space per key
4. **Merkle Trees**: O(log N) comparisons to find differences

**Failure Scenarios to Discuss:**

- Single node failure → Other replicas serve requests
- Multiple node failures → Sloppy quorum if < N-W nodes down
- Network partition → Both sides accept writes (resolve with vector clocks)
- Datacenter failure → Other datacenters continue serving
- Coordinator crash → Client retries with different coordinator

**Evolution Path:**

1. **MVP**: Single datacenter, N=3, gossip, basic quorums
2. **Scale**: Add virtual nodes, optimize Merkle trees
3. **Multi-DC**: Cross-datacenter replication with local quorums
4. **Advanced**: Compression, encryption, secondary indexes

