<!--
File: distributed_cache_system_design.md
Purpose: Complete system design document for a Distributed Cache system (Redis/Memcached)
Author: System Design Framework
Created: October 1, 2025
Last Updated: October 1, 2025

Description:
This document provides a comprehensive system design for building a distributed caching solution
similar to Redis or Memcached. It covers all aspects from requirements gathering, capacity planning,
architecture design, database schemas, API design, to scaling considerations and trade-offs.

Usage:
- Use this as a reference for distributed cache system design interviews
- Follow the framework structure for consistent system design documentation
- Adapt requirements and scale based on specific use cases
-->

# DISTRIBUTED CACHE SYSTEM DESIGN

## Building a High-Performance In-Memory Data Store

---

## 1. REQUIREMENTS & CLARIFICATION

### User Stories

**As a backend developer**, I want to store frequently accessed data in memory so that I can reduce database load and improve application response times.

**As a system architect**, I want a distributed caching solution so that I can scale horizontally and handle increased traffic without performance degradation.

**As an operations engineer**, I want automatic failover and replication so that the cache remains available even when nodes fail.

**As an application developer**, I want to set expiration times on cached data so that stale data is automatically removed.

---

### Functional Requirements

**Core Features (MVP):**

1. **Key-Value Storage**: Store and retrieve data using string keys
2. **Data Expiration**: Support TTL (Time To Live) for automatic key expiration
3. **Basic Data Types**: Support strings, lists, sets, sorted sets, and hashes
4. **CRUD Operations**: GET, SET, DELETE operations on keys
5. **Atomic Operations**: Increment/decrement, compare-and-set
6. **Batch Operations**: MGET, MSET for multiple keys
7. **Pattern Matching**: Find keys by pattern (e.g., "user:*")
8. **Pub/Sub**: Basic message publishing and subscription

**Out of Scope for MVP:**

- Complex transactions (ACID guarantees)
- Full database replacement features
- Advanced Lua scripting
- Cluster auto-sharding (will handle manually)

---

### Non-Functional Requirements

1. **Performance**:
   - Sub-millisecond latency for GET/SET operations (< 1ms p99)
   - Support 100K+ operations per second per node
   - O(1) time complexity for basic operations

2. **Availability**:
   - 99.99% uptime
   - Automatic failover with replica promotion
   - No single point of failure

3. **Scalability**:
   - Horizontal scaling through sharding
   - Support for 1000+ nodes in a cluster
   - Linear performance scaling with added nodes

4. **Consistency**:
   - Eventual consistency for replicas
   - Strong consistency for single-node writes
   - Configurable consistency levels

5. **Durability** (Optional):
   - Periodic snapshots to disk
   - Append-only file (AOF) for write logging
   - Configurable persistence strategies

6. **Security**:
   - Authentication with password protection
   - TLS/SSL encryption for data in transit
   - Network isolation options

---

### Clarifying Questions & Assumptions

**Questions:**

- Q: What's the expected scale of operations per second?
  - A: Starting with 1M ops/sec, scaling to 10M+ ops/sec

- Q: What's the typical key-value size?
  - A: Average key: 50 bytes, Average value: 1KB, Max value: 512MB

- Q: What's the read/write ratio?
  - A: 80% reads, 20% writes (typical cache workload)

- Q: Do we need persistence or is it pure in-memory?
  - A: Persistence is optional but recommended for recovery

- Q: What's the expected data size?
  - A: Starting with 100GB per node, scaling to 1TB+ total

**Assumptions:**

1. Network bandwidth is not a bottleneck (10Gbps+ available)
2. Clients can implement consistent hashing for sharding
3. Cluster membership changes are infrequent
4. Most values are relatively small (< 10KB)
5. Memory is more critical than CPU

---

## 2. BACK-OF-THE-ENVELOPE CALCULATIONS

### Traffic Estimates

```text
Target Operations: 1M ops/sec (starting), 10M ops/sec (peak)

Read/Write Distribution:
- Reads: 1M * 0.8 = 800K reads/sec
- Writes: 1M * 0.2 = 200K writes/sec

Peak Load (3x average):
- Peak Reads: 2.4M reads/sec
- Peak Writes: 600K writes/sec

Operations per day:
- 1M ops/sec * 86,400 seconds = 86.4B operations/day
```text

### Storage Estimates

```text
Average Data Size:
- Key size: 50 bytes
- Value size: 1KB
- Total per entry: ~1.05KB

Number of Keys:
- Assuming 100M keys initially
- Storage = 100M * 1.05KB = 105GB

With Overhead (metadata, pointers, hash tables):
- Memory overhead: ~30%
- Total memory: 105GB * 1.3 = 136.5GB

5-Year Growth (assuming 50% yearly growth):
- Year 1: 136.5GB
- Year 2: 204.75GB
- Year 3: 307GB
- Year 4: 460GB
- Year 5: 690GB
```text

### Resource Estimates

```text
Number of Nodes Needed:

For Performance (100K ops/sec per node):
- 1M ops/sec / 100K = 10 nodes minimum
- With 3x peak: 30 nodes for peak load
- With replication (1 primary + 1 replica): 20 nodes (10 primary + 10 replicas)

For Storage (64GB RAM per node):
- 136.5GB / 64GB = 3 nodes minimum
- With replication factor of 2: 6 nodes
- With sharding: 10 shards × 2 replicas = 20 nodes

Final Node Count: 20-30 nodes initially

Memory per Node:
- 64GB RAM standard
- 80% usable for cache (20% for OS and overhead)
- Usable: 51.2GB per node

CPU Requirements:
- Single-threaded performance critical
- Event-driven I/O model
- 8-16 cores per node (for handling connections)
```text

### Bandwidth Estimates

```text
Average Request Size:
- GET request: 50 bytes (key only)
- SET request: 1.05KB (key + value)
- Average: (0.8 * 50) + (0.2 * 1050) = 40 + 210 = 250 bytes

Average Response Size:
- GET response: 1KB (value)
- SET response: 10 bytes (OK/ERROR)
- Average: (0.8 * 1000) + (0.2 * 10) = 800 + 2 = 802 bytes

Bandwidth Calculation (1M ops/sec):
- Incoming: 1M * 250 bytes = 250MB/sec = 2 Gbps
- Outgoing: 1M * 802 bytes = 802MB/sec = 6.4 Gbps
- Total per node (10 nodes): ~200Mbps incoming, 640Mbps outgoing

Peak Bandwidth (3x):
- Per node: 600Mbps incoming, 1.9Gbps outgoing
- Well within 10Gbps network capacity
```text

---

## 3. HIGH-LEVEL DESIGN

### Architecture Overview

```mermaid
graph TB
    subgraph Client Layer
        Client1[Application Client 1]
        Client2[Application Client 2]
        Client3[Application Client 3]
    end
    
    subgraph Client-Side Components
        CH[Client Library<br/>with Consistent Hashing]
    end
    
    subgraph Cache Cluster - Shard 1
        M1[Master Node 1<br/>Redis/Memcached]
        R1[Replica Node 1<br/>Redis/Memcached]
    end
    
    subgraph Cache Cluster - Shard 2
        M2[Master Node 2<br/>Redis/Memcached]
        R2[Replica Node 2<br/>Redis/Memcached]
    end
    
    subgraph Cache Cluster - Shard N
        MN[Master Node N<br/>Redis/Memcached]
        RN[Replica Node N<br/>Redis/Memcached]
    end
    
    subgraph Coordination Layer
        Sentinel[Sentinel Cluster<br/>Health Monitoring]
        Config[Configuration Service<br/>etcd/Consul]
    end
    
    subgraph Persistence Layer
        Disk1[(Snapshot Storage<br/>Disk/S3)]
        AOF1[(AOF Logs<br/>Disk/S3)]
    end
    
    subgraph Monitoring
        Metrics[Metrics Collector<br/>Prometheus]
        Logs[Log Aggregator<br/>ELK Stack]
    end
    
    Client1 -->|Request| CH
    Client2 -->|Request| CH
    Client3 -->|Request| CH
    
    CH -->|1. Hash Key| M1
    CH -->|1. Hash Key| M2
    CH -->|1. Hash Key| MN
    
    M1 -.->|2. Async Replication| R1
    M2 -.->|2. Async Replication| R2
    MN -.->|2. Async Replication| RN
    
    M1 -->|3. Periodic Snapshot| Disk1
    M1 -->|4. Write AOF| AOF1
    
    Sentinel -->|Health Check| M1
    Sentinel -->|Health Check| M2
    Sentinel -->|Health Check| R1
    Sentinel -->|Health Check| R2
    Sentinel -->|Failover| Config
    
    M1 -->|Metrics| Metrics
    M2 -->|Metrics| Metrics
    M1 -->|Logs| Logs
```text

### Data Flow Explanation

**Write Operation Flow:**
1. **Client Request**: Application sends SET command with key-value pair to client library
2. **Key Hashing**: Client library applies consistent hashing to determine target shard
3. **Route to Master**: Request is routed to the appropriate master node
4. **Write to Memory**: Master node writes data to in-memory hash table (O(1) operation)
5. **Async Replication**: Master asynchronously replicates write to replica nodes
6. **Persistence (Optional)**: Write is logged to AOF file and/or included in next snapshot
7. **Response**: Master returns success acknowledgment to client

**Read Operation Flow:**
1. **Client Request**: Application sends GET command with key to client library
2. **Key Hashing**: Client library applies consistent hashing to determine target shard
3. **Route to Node**: Request is routed to master node (or replica for read-heavy workloads)
4. **Memory Lookup**: Node performs O(1) hash table lookup
5. **Check Expiration**: If TTL exists, verify key hasn't expired
6. **Response**: Return value to client (or null if expired/missing)

**Failover Flow:**
1. **Failure Detection**: Sentinel nodes detect master failure through heartbeat timeout
2. **Quorum Decision**: Sentinels reach quorum agreement on failure
3. **Replica Promotion**: Sentinel promotes a replica to master
4. **Configuration Update**: Configuration service updates cluster topology
5. **Client Notification**: Clients are notified of new master location
6. **Resume Operations**: Traffic is redirected to new master

---

## 4. DATABASE DESIGN

### In-Memory Data Structures

**Hash Table (Primary Storage):**
```text
Structure: dict (hash table with chaining)
- Entry:
  - key (string, pointer to key object)
  - value (pointer to value object)
  - hash (uint64, cached hash value)
  - next (pointer, for collision chaining)

Implementation:
- Load factor: 0.75 (resize when 75% full)
- Collision handling: Chaining with linked lists
- Hash function: SipHash or MurmurHash3
```text

**Key Object:**
```text
Key Structure:
- type (uint8: STRING, LIST, SET, ZSET, HASH)
- encoding (uint8: RAW, INT, HT, ZIPLIST, etc.)
- lru (uint24: LRU timestamp for eviction)
- refcount (uint32: reference count)
- ptr (pointer to actual data)
```text

**Value Objects by Type:**

```text
1. String Value:
   - len (uint32: string length)
   - data (char array: actual string data)
   - Max size: 512MB

2. List Value:
   - encoding (LINKEDLIST or ZIPLIST)
   - length (uint32: number of elements)
   - head, tail pointers (for linkedlist)
   - Max size: 2^32 - 1 elements

3. Set Value:
   - encoding (HASHTABLE or INTSET)
   - size (uint32: number of elements)
   - elements (hash table or sorted int array)
   - Max size: 2^32 - 1 elements

4. Sorted Set Value:
   - encoding (SKIPLIST or ZIPLIST)
   - length (uint32: number of elements)
   - skiplist (for range queries)
   - dict (for O(1) score lookup by member)
   - Max size: 2^32 - 1 elements

5. Hash Value:
   - encoding (HASHTABLE or ZIPLIST)
   - size (uint32: number of fields)
   - fields (key-value pairs)
   - Max size: 2^32 - 1 fields
```text

### Expiration Management

**Expiration Dictionary:**
```text
Structure: Separate hash table for keys with TTL
- Entry:
  - key (pointer to key object)
  - expire_time (int64: Unix timestamp in milliseconds)

Eviction Strategies:
- Passive: Check expiration on access
- Active: Periodic random sampling (100 keys every 100ms)
- Lazy: Remove during iteration operations
```text

### Persistence Structures

**RDB Snapshot Format:**
```text
Header:
- Magic string: "REDIS"
- Version: uint16
- Metadata: Creation time, Redis version

Database Section (per database):
- db_number (uint8)
- hash_table_size (uint32)
- expire_table_size (uint32)
- Key-value pairs:
  - type (uint8)
  - key (length-prefixed string)
  - expire (int64, optional)
  - value (type-specific encoding)

Footer:
- CRC64 checksum
```text

**AOF Log Format:**
```text
Format: Redis Protocol (RESP)
- Command: *<arg_count>\r\n
- Arguments: $<length>\r\n<data>\r\n

Example:
*3\r\n$3\r\nSET\r\n$5\r\nmykey\r\n$7\r\nmyvalue\r\n

Rewrite Strategy:
- Trigger: When AOF size exceeds 100% of base size
- Process: Fork child process to rewrite from memory snapshot
```text

---

## 5. API DESIGN

### Base Configuration

**Protocol**: Redis Serialization Protocol (RESP) or REST API wrapper

**Connection**:
- TCP socket connection (default port: 6379)
- Connection pooling recommended
- Pipelining support for batching

**Authentication**:
```text
AUTH password
```text

**Response Format**:
- Simple Strings: +OK\r\n
- Errors: -Error message\r\n
- Integers: :1000\r\n
- Bulk Strings: $6\r\nfoobar\r\n
- Arrays: *2\r\n$3\r\nfoo\r\n$3\r\nbar\r\n

---

### Core String Operations

#### SET - Store a key-value pair

```text
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```text

**Parameters**:
- `key`: String key (required)
- `value`: String value (required)
- `EX seconds`: Set expiration in seconds (optional)
- `PX milliseconds`: Set expiration in milliseconds (optional)
- `NX`: Only set if key doesn't exist (optional)
- `XX`: Only set if key exists (optional)

**Request Example**:
```text
SET user:1001:session "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9" EX 3600
```text

**Response**:
```text
Success: +OK
Error: -ERR invalid expiration time
```text

**Time Complexity**: O(1)

---

#### GET - Retrieve a value by key

```http
GET key
```text

**Parameters**:
- `key`: String key (required)

**Request Example**:
```http
GET user:1001:session
```text

**Response**:
```text
Success: $43\r\neyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9\r\n
Not Found: $-1\r\n (null bulk string)
```text

**Time Complexity**: O(1)

---

#### DEL - Delete one or more keys

```text
DEL key [key ...]
```text

**Parameters**:
- `key`: One or more keys to delete

**Request Example**:
```text
DEL user:1001:session user:1001:cart
```text

**Response**:
```text
:(integer) 2  # Number of keys deleted
```text

**Time Complexity**: O(N) where N is number of keys

---

#### MGET - Get multiple values

```http
MGET key [key ...]
```text

**Parameters**:
- `key`: One or more keys to retrieve

**Request Example**:
```http
MGET user:1001:name user:1001:email user:1001:age
```text

**Response**:
```text
*3
$10
John Smith
$18
john@example.com
$2
25
```text

**Time Complexity**: O(N) where N is number of keys

---

#### MSET - Set multiple key-value pairs

```text
MSET key value [key value ...]
```text

**Parameters**:
- `key value`: Pairs of keys and values

**Request Example**:
```text
MSET user:1001:name "John Smith" user:1001:email "john@example.com"
```text

**Response**:
```text
+OK
```text

**Time Complexity**: O(N) where N is number of keys

---

### Expiration Operations

#### EXPIRE - Set key expiration

```text
EXPIRE key seconds
```text

**Parameters**:
- `key`: Key to set expiration on
- `seconds`: Time to live in seconds

**Request Example**:
```text
EXPIRE session:abc123 3600
```text

**Response**:
```text
:1  # Success
:0  # Key doesn't exist
```text

**Time Complexity**: O(1)

---

#### TTL - Get time to live

```text
TTL key
```text

**Parameters**:
- `key`: Key to check expiration

**Request Example**:
```text
TTL session:abc123
```text

**Response**:
```text
:3599  # Seconds remaining
:-1    # Key exists but no expiration
:-2    # Key doesn't exist
```text

**Time Complexity**: O(1)

---

#### PERSIST - Remove expiration

```text
PERSIST key
```text

**Parameters**:
- `key`: Key to remove expiration from

**Request Example**:
```text
PERSIST user:1001:preferences
```text

**Response**:
```text
:1  # Expiration removed
:0  # Key doesn't exist or no expiration
```text

**Time Complexity**: O(1)

---

### Atomic Operations

#### INCR - Increment integer value

```text
INCR key
```text

**Parameters**:
- `key`: Key containing integer value

**Request Example**:
```text
INCR page:views:homepage
```text

**Response**:
```text
:1001  # New value after increment
```text

**Time Complexity**: O(1)

---

#### INCRBY - Increment by amount

```text
INCRBY key increment
```text

**Parameters**:
- `key`: Key containing integer value
- `increment`: Amount to add

**Request Example**:
```text
INCRBY user:1001:points 100
```text

**Response**:
```text
:1500  # New value after increment
```text

**Time Complexity**: O(1)

---

#### DECR / DECRBY - Decrement operations

```text
DECR key
DECRBY key decrement
```text

Similar to INCR/INCRBY but subtracts value.

**Time Complexity**: O(1)

---

### List Operations

#### LPUSH - Prepend to list

```text
LPUSH key value [value ...]
```text

**Parameters**:
- `key`: List key
- `value`: One or more values to prepend

**Request Example**:
```text
LPUSH queue:jobs "process-video-123" "send-email-456"
```text

**Response**:
```text
:2  # New length of list
```text

**Time Complexity**: O(N) where N is number of values

---

#### RPUSH - Append to list

```text
RPUSH key value [value ...]
```text

**Parameters**:
- `key`: List key
- `value`: One or more values to append

**Request Example**:
```text
RPUSH notifications:user:1001 "New message from Jane"
```text

**Response**:
```text
:3  # New length of list
```text

**Time Complexity**: O(N) where N is number of values

---

#### LPOP - Remove and return first element

```text
LPOP key
```text

**Parameters**:
- `key`: List key

**Request Example**:
```text
LPOP queue:jobs
```text

**Response**:
```text
$17
process-video-123
```text

**Time Complexity**: O(1)

---

#### LRANGE - Get range of elements

```text
LRANGE key start stop
```text

**Parameters**:
- `key`: List key
- `start`: Start index (0-based, can be negative)
- `stop`: Stop index (inclusive)

**Request Example**:
```text
LRANGE notifications:user:1001 0 9
```text

**Response**:
```text
*10
$23
New message from Jane
$19
New follower: Bob
...
```text

**Time Complexity**: O(S+N) where S is start offset, N is range size

---

### Set Operations

#### SADD - Add members to set

```text
SADD key member [member ...]
```text

**Parameters**:
- `key`: Set key
- `member`: One or more members to add

**Request Example**:
```text
SADD user:1001:interests "technology" "sports" "music"
```text

**Response**:
```text
:3  # Number of members added (excluding duplicates)
```text

**Time Complexity**: O(N) where N is number of members

---

#### SMEMBERS - Get all set members

```text
SMEMBERS key
```text

**Parameters**:
- `key`: Set key

**Request Example**:
```text
SMEMBERS user:1001:interests
```text

**Response**:
```text
*3
$10
technology
$6
sports
$5
music
```text

**Time Complexity**: O(N) where N is set size

---

#### SISMEMBER - Check membership

```text
SISMEMBER key member
```text

**Parameters**:
- `key`: Set key
- `member`: Member to check

**Request Example**:
```text
SISMEMBER user:1001:interests "technology"
```text

**Response**:
```text
:1  # Member exists
:0  # Member doesn't exist
```text

**Time Complexity**: O(1)

---

### Hash Operations

#### HSET - Set hash field

```text
HSET key field value [field value ...]
```text

**Parameters**:
- `key`: Hash key
- `field value`: Pairs of fields and values

**Request Example**:
```text
HSET user:1001 name "John Smith" email "john@example.com" age 25
```text

**Response**:
```text
:3  # Number of fields added
```text

**Time Complexity**: O(N) where N is number of fields

---

#### HGET - Get hash field value

```http
HGET key field
```text

**Parameters**:
- `key`: Hash key
- `field`: Field name

**Request Example**:
```http
HGET user:1001 name
```text

**Response**:
```text
$10
John Smith
```text

**Time Complexity**: O(1)

---

#### HGETALL - Get all hash fields and values

```text
HGETALL key
```text

**Parameters**:
- `key`: Hash key

**Request Example**:
```text
HGETALL user:1001
```text

**Response**:
```text
*6
$4
name
$10
John Smith
$5
email
$18
john@example.com
$3
age
$2
25
```text

**Time Complexity**: O(N) where N is size of hash

---

### Sorted Set Operations

#### ZADD - Add members with scores

```text
ZADD key score member [score member ...]
```text

**Parameters**:
- `key`: Sorted set key
- `score member`: Pairs of scores and members

**Request Example**:
```text
ZADD leaderboard:global 1500 "player:1001" 1450 "player:1002"
```text

**Response**:
```text
:2  # Number of members added
```text

**Time Complexity**: O(log(N)) per member

---

#### ZRANGE - Get range by rank

```text
ZRANGE key start stop [WITHSCORES]
```text

**Parameters**:
- `key`: Sorted set key
- `start`: Start rank (0-based)
- `stop`: Stop rank (inclusive)
- `WITHSCORES`: Include scores (optional)

**Request Example**:
```text
ZRANGE leaderboard:global 0 9 WITHSCORES
```text

**Response**:
```text
*20
$12
player:1003
$4
1500
$12
player:1001
$4
1450
...
```text

**Time Complexity**: O(log(N)+M) where M is range size

---

#### ZRANK - Get member rank

```text
ZRANK key member
```text

**Parameters**:
- `key`: Sorted set key
- `member`: Member to find rank

**Request Example**:
```text
ZRANK leaderboard:global "player:1001"
```text

**Response**:
```text
:5  # Rank (0-based)
```text

**Time Complexity**: O(log(N))

---

### Pattern Matching Operations

#### KEYS - Find keys by pattern

```text
KEYS pattern
```text

**Parameters**:
- `pattern`: Glob-style pattern (* and ? supported)

**Request Example**:
```text
KEYS user:*:session
```text

**Response**:
```text
*3
$17
user:1001:session
$17
user:1002:session
$17
user:1003:session
```text

**⚠️ Warning**: O(N) operation - use SCAN in production

**Time Complexity**: O(N) where N is database size

---

#### SCAN - Iterate keys incrementally

```text
SCAN cursor [MATCH pattern] [COUNT count]
```text

**Parameters**:
- `cursor`: Cursor position (0 to start)
- `MATCH pattern`: Filter by pattern (optional)
- `COUNT count`: Hint for number of keys to return (optional)

**Request Example**:
```text
SCAN 0 MATCH user:* COUNT 100
```text

**Response**:
```text
*2
$4
1357  # Next cursor (0 means complete)
*50   # Keys returned
$15
user:1001:name
$15
user:1002:name
...
```text

**Time Complexity**: O(1) per call, O(N) for full iteration

---

### Pub/Sub Operations

#### PUBLISH - Publish message to channel

```text
PUBLISH channel message
```text

**Parameters**:
- `channel`: Channel name
- `message`: Message to publish

**Request Example**:
```text
PUBLISH notifications:realtime "New order received"
```text

**Response**:
```text
:5  # Number of subscribers that received message
```text

**Time Complexity**: O(N+M) where N is subscribers, M is patterns

---

#### SUBSCRIBE - Subscribe to channels

```text
SUBSCRIBE channel [channel ...]
```text

**Parameters**:
- `channel`: One or more channels to subscribe

**Request Example**:
```text
SUBSCRIBE notifications:realtime alerts:critical
```text

**Response** (per message):
```text
*3
$7
message
$23
notifications:realtime
$18
New order received
```text

**Time Complexity**: O(N) where N is number of channels

---

#### PSUBSCRIBE - Subscribe to patterns

```text
PSUBSCRIBE pattern [pattern ...]
```text

**Parameters**:
- `pattern`: One or more channel patterns

**Request Example**:
```text
PSUBSCRIBE notifications:* alerts:*
```text

**Response** (similar to SUBSCRIBE with pattern info)

**Time Complexity**: O(N) where N is number of patterns

---

### Administrative Operations

#### INFO - Get server information

```text
INFO [section]
```text

**Parameters**:
- `section`: Optional section (server, memory, stats, etc.)

**Request Example**:
```text
INFO memory
```text

**Response**:
```text
$200
# Memory
used_memory:1073741824
used_memory_human:1.00G
used_memory_peak:1610612736
...
```text

**Time Complexity**: O(1)

---

#### FLUSHDB - Clear current database

```text
FLUSHDB [ASYNC]
```text

**Parameters**:
- `ASYNC`: Delete asynchronously (optional)

**Request Example**:
```text
FLUSHDB ASYNC
```text

**Response**:
```text
+OK
```text

**Time Complexity**: O(N) where N is database size

---

#### CONFIG GET/SET - Get/set configuration

```http
CONFIG GET parameter
CONFIG SET parameter value
```text

**Parameters**:
- `parameter`: Configuration parameter name
- `value`: New value (for SET)

**Request Example**:
```http
CONFIG GET maxmemory
CONFIG SET maxmemory 2gb
```text

**Response**:
```text
*2
$9
maxmemory
$10
2147483648
```text

**Time Complexity**: O(1)

---

### Cross-Cutting Concerns

**Rate Limiting**:
- Per-connection limits: 10K commands/sec
- Global limits: 1M commands/sec per node
- Implemented at connection handler level

**Error Response Format**:
```text
-ERR Error message here
-WRONGTYPE Operation against a key holding the wrong kind of value
-NOAUTH Authentication required
```text

**Pipelining**:
- Send multiple commands without waiting for responses
- Responses returned in order
- Reduces RTT overhead
- Example: Send 100 SETs at once, receive 100 responses

**Transactions (Optional)**:
```text
MULTI           # Start transaction
SET key1 value1
SET key2 value2
EXEC            # Execute atomically
```text

**Blocking Operations**:
- BLPOP/BRPOP: Block until list element available
- Timeout parameter supported
- Used for queue implementations

**Memory Management**:
- MAXMEMORY configuration parameter
- Eviction policies: noeviction, allkeys-lru, volatile-lru, etc.
- MEMORY commands for detailed analysis

**Security**:
- AUTH command for password authentication
- ACL (Access Control Lists) for fine-grained permissions
- Rename dangerous commands (FLUSHDB, FLUSHALL)
- Bind to specific network interfaces

---

### API Trade-Offs

**Decision: Binary Protocol (RESP) vs REST API**
- **Choice**: Binary Protocol (RESP) with optional REST wrapper
- **Pros**: Minimal parsing overhead, lower latency, smaller payload size
- **Cons**: Less familiar to developers, requires client library
- **Justification**: Performance is critical for cache; sub-millisecond latency requires efficient protocol

**Decision: Synchronous vs Asynchronous Replication**
- **Choice**: Asynchronous replication by default
- **Pros**: Lower write latency, higher throughput, no blocking
- **Cons**: Potential data loss on failure, eventual consistency
- **Justification**: Cache can tolerate some data loss; performance > durability

**Decision: Single-threaded vs Multi-threaded Event Loop**
- **Choice**: Single-threaded with I/O multiplexing (epoll/kqueue)
- **Pros**: No lock contention, simpler code, predictable performance
- **Cons**: Limited to single core for command execution
- **Justification**: Memory operations are extremely fast; I/O is bottleneck, not CPU

**Decision: Text-based vs Binary Value Storage**
- **Choice**: Binary-safe storage
- **Pros**: Can store any data format, no encoding overhead
- **Cons**: No built-in serialization
- **Justification**: Maximum flexibility; let clients choose serialization format

---

## 6. DEEP-DIVE COMPONENTS & TRADE-OFFS

### Component 1: Consistent Hashing for Sharding

**Purpose**: Distribute keys evenly across multiple cache nodes while minimizing redistribution when nodes are added/removed.

**Architecture**:

```text
Consistent Hashing Ring:

Hash Space: 0 to 2^32 - 1 (or 2^160 for better distribution)

Node Placement:
- Each node is hashed to multiple points (virtual nodes)
- Typically 150-200 virtual nodes per physical node
- Example: node1 -> hash("node1-0"), hash("node1-1"), ..., hash("node1-199")

Key Assignment:
- Hash the key: hash_value = hash(key)
- Walk clockwise on ring to find first node
- That node is responsible for the key

Virtual Nodes Benefits:
- More even distribution of keys
- When node fails, load distributes to multiple nodes
- When node added, steals from multiple nodes
```text

**Implementation Details**:

```python
class ConsistentHashRing:
    def __init__(self, nodes, virtual_nodes=150):
        self.virtual_nodes = virtual_nodes
        self.ring = {}  # hash -> node mapping
        self.sorted_keys = []  # sorted hash values
        
        for node in nodes:
            self.add_node(node)
    
    def add_node(self, node):
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}-{i}"
            hash_value = self.hash(virtual_key)
            self.ring[hash_value] = node
            self.sorted_keys.append(hash_value)
        self.sorted_keys.sort()
    
    def get_node(self, key):
        hash_value = self.hash(key)
        # Binary search for next node clockwise
        idx = bisect.bisect_right(self.sorted_keys, hash_value)
        if idx == len(self.sorted_keys):
            idx = 0
        return self.ring[self.sorted_keys[idx]]
```text

**Trade-offs**:

**Decision: Consistent Hashing vs Hash Slot-based Sharding**
- **Choice**: Consistent Hashing with Virtual Nodes
- **Pros**: Smooth scaling, automatic load balancing, minimal key redistribution
- **Cons**: More complex to implement, requires client-side logic
- **Justification**: Better for dynamic environments where nodes frequently join/leave

**Decision: Number of Virtual Nodes (150-200 vs 1)**
- **Choice**: 150 virtual nodes per physical node
- **Pros**: More even distribution, better load balancing on node changes
- **Cons**: Higher memory overhead for ring structure
- **Justification**: The overhead (150 * 10 nodes * 64 bytes = 96KB) is negligible

---

### Component 2: Replication and Failover

**Purpose**: Provide high availability by maintaining replicas and automatically promoting them on failure.

**Architecture**:

```text
Replication Topology:

Master-Replica Model:
+----------+
|  Master  | <----- Writes go here
+----------+
     |
     | Async Replication
     | (replication stream)
     |
     v
+----------+     +----------+
| Replica1 |     | Replica2 |
+----------+     +----------+
     |               |
     |               |
     v               v
  Reads           Reads
  (optional)      (optional)
```text

**Replication Process**:

1. **Write Propagation**:
   - Master appends command to replication buffer
   - Buffer is sent to replicas asynchronously
   - Replicas apply commands in order
   - Replicas ACK offset periodically

2. **Replica Synchronization**:
   - **Full Sync**: New replica connects
     - Master creates background snapshot (RDB)
     - Sends snapshot to replica
     - Buffers writes during snapshot
     - Sends buffered writes after snapshot
   - **Partial Sync**: Reconnected replica
     - Master maintains replication backlog (circular buffer)
     - Replica sends last offset received
     - Master sends only missing commands

**Sentinel-based Failover**:

```text
Sentinel Cluster (3-5 nodes for quorum):

Responsibilities:
1. Monitor master and replica health
2. Detect failures through gossip protocol
3. Reach quorum agreement on failures
4. Elect new master from replicas
5. Update configuration and notify clients

Failure Detection:
- Send PING to master every second
- Wait for PONG response
- After N consecutive failures (e.g., 5 = 5 seconds), mark as SDOWN (subjectively down)
- Exchange status with other Sentinels
- When quorum Sentinels agree, mark as ODOWN (objectively down)

Failover Process:
1. Select best replica:
   - Replica with lowest replication lag
   - Replica with highest priority
   - Replica with lowest run_id (tiebreaker)

2. Promote replica:
   - Send REPLICAOF NO ONE to selected replica
   - Wait for promotion to complete

3. Reconfigure other replicas:
   - Send REPLICAOF <new_master> to other replicas

4. Update configuration:
   - Update Sentinel configuration
   - Notify clients via pub/sub

5. Monitor old master:
   - When old master comes back, demote to replica
```text

**Trade-offs**:

**Decision: Synchronous vs Asynchronous Replication**
- **Choice**: Asynchronous Replication
- **Pros**: 
  - Lower write latency (no waiting for replica ACK)
  - Higher throughput
  - Master not blocked by slow replicas
- **Cons**: 
  - Potential data loss (unreplicated writes lost on master failure)
  - Eventually consistent reads from replicas
- **Justification**: Cache workloads prioritize performance over durability

**Decision: Sentinel vs Embedded Cluster Mode**
- **Choice**: Sentinel-based (similar to Redis Sentinel)
- **Pros**: 
  - Simpler client implementation
  - Separate failure detection from data nodes
  - Can monitor multiple masters
- **Cons**: 
  - Additional infrastructure (Sentinel nodes)
  - Slightly slower failover (network round trips)
- **Justification**: Cleaner separation of concerns, easier to manage

**Decision: Automatic vs Manual Failover**
- **Choice**: Automatic Failover with Manual Override
- **Pros**: Faster recovery, less downtime, no human intervention needed
- **Cons**: Risk of split-brain scenarios, false positives
- **Justification**: Availability is critical; use quorum to prevent split-brain

---

### Component 3: Memory Management and Eviction

**Purpose**: Efficiently manage limited memory resources and automatically evict data when memory limit is reached.

**Architecture**:

```text
Memory Management Layers:

1. Object-level Memory Tracking:
   - Each object tracks its memory usage
   - Includes overhead: pointers, metadata, reference counts
   - Aggregated to calculate total memory usage

2. Memory Allocator (jemalloc):
   - Optimized for frequent small allocations
   - Reduces fragmentation
   - Provides memory usage statistics

3. Eviction Engine:
   - Triggered when memory exceeds threshold
   - Selects victims based on eviction policy
   - Frees memory by deleting keys
```text

**Eviction Policies**:

1. **noeviction** (Default):
   - Returns error when memory limit reached
   - No automatic eviction
   - Use case: When cache misses are acceptable

2. **allkeys-lru** (Least Recently Used):
   - Evict least recently accessed keys from all keys
   - Maintains approximate LRU using sampling
   - Use case: General-purpose caching

3. **volatile-lru**:
   - Evict least recently accessed keys with TTL set
   - Only considers keys in expiration dictionary
   - Use case: Mix of permanent and temporary data

4. **allkeys-random**:
   - Evict random keys from all keys
   - Fastest eviction policy
   - Use case: When all data has equal importance

5. **volatile-random**:
   - Evict random keys with TTL
   - Use case: Temporary data with random access patterns

6. **volatile-ttl**:
   - Evict keys with nearest expiration time
   - Use case: Prioritize keeping fresh data

**LRU Implementation (Approximate)**:

```text
Traditional LRU Problems:
- Requires doubly-linked list + hash map
- O(1) operations but high memory overhead
- Every access updates list (expensive)

Redis-style Approximation:
1. Each key has 24-bit LRU timestamp
2. Updated on access (LRU clock / 1000)
3. When eviction needed:
   - Sample N keys randomly (default: 5)
   - Evict key with oldest LRU timestamp
   - Repeat until enough memory freed

4. LRU clock:
   - Global counter, incremented every millisecond
   - 24-bit wraps every 194 days (acceptable)

5. Sample size trade-off:
   - Larger sample = closer to true LRU
   - Sample of 10 is very close to true LRU
   - Default 5 is good balance
```text

**Memory Usage Calculation**:

```text
Per-key Overhead:
- Hash table entry: 16 bytes (key ptr, value ptr, hash, next)
- Key object: 16 bytes (type, encoding, lru, refcount, ptr)
- Value object header: 16 bytes (minimum)
- String data: len + 1 bytes
- Total: ~50 bytes + key length + value length

Hash Table Resize:
- Triggers at 75% load factor
- Incremental rehashing (doesn't block)
- Temporarily uses 2x memory during resize

Fragmentation:
- External: Unused memory between allocations
- Internal: Wasted space within allocations
- Measured: (RSS - used_memory) / RSS
- Mitigated by jemalloc and periodic defragmentation
```text

**Trade-offs**:

**Decision: True LRU vs Approximate LRU**
- **Choice**: Approximate LRU with sampling
- **Pros**: 
  - Minimal memory overhead (24 bits vs linked list)
  - No performance penalty on every access
  - Good enough for cache workloads
- **Cons**: 
  - Not perfectly LRU
  - May evict slightly sub-optimal keys
- **Justification**: Performance and memory savings outweigh perfect accuracy

**Decision: Active vs Passive Expiration**
- **Choice**: Hybrid (both active and passive)
- **Pros**: 
  - Passive: No CPU overhead, checked on access
  - Active: Gradually frees memory, doesn't wait for access
- **Cons**: 
  - Expired keys may linger if not accessed
- **Justification**: Best of both worlds for cache

**Decision: Memory Limit Hard vs Soft**
- **Choice**: Hard limit with configurable threshold
- **Pros**: 
  - Prevents OOM crashes
  - Predictable behavior
- **Cons**: 
  - May reject writes even if memory available soon
- **Justification**: Stability over flexibility

---

### Component 4: Persistence (Optional)

**Purpose**: Provide durability options for cache data to survive restarts and failures.

**Architecture**:

Two persistence mechanisms:

**1. RDB Snapshots**:
```text
Process:
1. Fork child process (copy-on-write)
2. Child iterates through memory and writes to temp file
3. Parent continues serving requests
4. COW ensures parent's writes don't affect child's view
5. When complete, atomically rename temp to .rdb file

Triggers:
- Time-based: save 900 1 (after 900s if 1+ keys changed)
- Manual: SAVE (blocking) or BGSAVE (background)

Pros:
- Compact file format
- Fast restart (load entire file)
- Minimal performance impact (forked process)

Cons:
- Potential data loss (since last snapshot)
- Fork can be slow for large datasets
- Uses extra memory for COW
```text

**2. AOF (Append-Only File)**:
```text
Process:
1. After executing write command, append to AOF buffer
2. Periodically fsync buffer to disk (configurable)
3. On restart, replay all commands from AOF

Fsync Policies:
- always: fsync after every command (slowest, safest)
- everysec: fsync every second (good balance)
- no: let OS decide (fastest, least safe)

AOF Rewrite:
- Background process creates new AOF from memory state
- Compacts multiple operations into single ones
- Example: 100 INCRs -> 1 SET with final value

Pros:
- More durable (can lose at most 1 second of data)
- Log is human-readable (can edit if corrupted)
- Automatic background rewrite

Cons:
- Larger file size than RDB
- Slower restart (must replay commands)
- Slightly slower writes (fsync overhead)
```text

**Hybrid Approach**:
```text
Combine RDB + AOF:
1. Use RDB for fast restarts
2. Use AOF for durability
3. On restart:
   - Load RDB snapshot
   - Replay AOF from snapshot point onwards

Result: Fast restarts + minimal data loss
```text

**Trade-offs**:

**Decision: No Persistence vs RDB vs AOF vs Hybrid**
- **Choice**: Optional (RDB by default, AOF for critical data)
- **Pros (No Persistence)**: 
  - Maximum performance
  - Simpler code
- **Pros (RDB)**: 
  - Fast restarts
  - Minimal performance impact
- **Pros (AOF)**: 
  - Better durability
  - Can recover from corruption
- **Pros (Hybrid)**: 
  - Best of both worlds
- **Justification**: Let users choose based on use case; cache typically doesn't need full durability

**Decision: Fork vs Thread-based Persistence**
- **Choice**: Fork-based (COW)
- **Pros**: 
  - Isolates persistence from serving traffic
  - No locks needed
- **Cons**: 
  - Memory spike from COW
  - Fork can be slow on large datasets
- **Justification**: Better isolation and simpler code

---

### Caching Strategy

**Cache-aside Pattern**:
```text
Application-managed caching:

Read Flow:
1. Check cache (GET key)
2. If hit: return value
3. If miss:
   - Query database
   - Write to cache (SET key value EX ttl)
   - Return value

Write Flow:
1. Write to database
2. Invalidate cache (DEL key)
3. Next read will cache new value
```text

**Read-through Cache**:
```text
Cache-managed loading:

Read Flow:
1. Application requests key from cache
2. If miss, cache automatically loads from DB
3. Cache returns value
4. Subsequent reads hit cache

Implementation requires cache to know about database
(Less common in practice)
```text

**Write-through Cache**:
```text
Synchronous cache + DB writes:

Write Flow:
1. Write to cache
2. Cache synchronously writes to DB
3. Return success

Pros: Always consistent
Cons: Higher write latency
```text

**Write-behind Cache**:
```text
Asynchronous DB writes:

Write Flow:
1. Write to cache
2. Return success immediately
3. Cache asynchronously writes to DB later

Pros: Lower write latency
Cons: Risk of data loss, eventual consistency
```text

**Recommended Strategy**: **Cache-aside** for most use cases
- Simple to implement
- Application has full control
- Can use with any database
- Clear invalidation logic

---

## 7. BOTTLENECKS & IMPROVEMENTS

### Potential Bottlenecks

#### Bottleneck 1: Single Master Write Bottleneck

**Problem**: All writes for a shard go to single master node, limiting write throughput to ~100K ops/sec.

**Solution**:
1. **Horizontal Sharding**: 
   - Increase number of shards
   - Each shard handles subset of keys
   - Linear scaling of write capacity

2. **Write Batching**: 
   - Use MSET instead of multiple SETs
   - Use pipelining to reduce RTT overhead
   - Can achieve 10x improvement

3. **Client-side Caching**: 
   - Cache reads in application layer
   - Reduces write pressure by handling more reads

**Monitoring**:
- Track write QPS per master
- Alert when approaching 80K ops/sec
- Monitor command queue length

---

#### Bottleneck 2: Network Bandwidth Saturation

**Problem**: High throughput can saturate 10Gbps network link, especially for large values.

**Solution**:
1. **Compression**: 
   - Compress large values before storing
   - Trade CPU for bandwidth
   - Typical compression ratio: 3-5x for text

2. **Value Size Optimization**: 
   - Store references instead of full objects
   - Use hash data structure instead of JSON strings
   - Split large values across multiple keys

3. **Local Caching**: 
   - Implement L1 cache in application
   - Reduces network requests for hot keys

**Monitoring**:
- Track network utilization per node
- Alert when exceeding 8Gbps (80% of capacity)
- Monitor bandwidth per client

---

#### Bottleneck 3: Memory Fragmentation

**Problem**: Over time, memory becomes fragmented, reducing effective capacity by 20-30%.

**Solution**:
1. **Active Defragmentation**: 
   - Background process to defragment memory
   - Move allocations to consolidate free space
   - Redis 4.0+ feature

2. **Periodic Restarts**: 
   - Schedule maintenance windows
   - Promote replica to master
   - Restart old master to clear fragmentation

3. **Better Allocator**: 
   - Use jemalloc instead of default libc malloc
   - Designed to minimize fragmentation

**Monitoring**:
- Track fragmentation ratio: (RSS - used_memory) / RSS
- Alert when ratio > 1.5 (50% waste)
- Monitor allocation patterns

---

#### Bottleneck 4: Hot Key Problem

**Problem**: Single popular key (e.g., celebrity user profile) receives massive traffic, overloading single shard.

**Solution**:
1. **Client-side Caching**: 
   - Cache hot keys in application memory
   - Short TTL (1-5 seconds)
   - Can handle 100K+ reads/sec per app instance

2. **Read Replicas**: 
   - Route reads to replicas
   - Distribute load across multiple nodes
   - Slight inconsistency acceptable for reads

3. **Key Replication**: 
   - Store hot key with multiple suffix variations
   - E.g., "hot_key_1", "hot_key_2", ... "hot_key_10"
   - Randomly select variant on read

**Monitoring**:
- Track per-key access frequency
- Alert when single key exceeds 10K ops/sec
- Use Redis MONITOR (sparingly) to identify hot keys

---

#### Bottleneck 5: Large Key Problem

**Problem**: Very large values (10MB+) cause blocking operations and slow down entire node.

**Solution**:
1. **Value Size Limits**: 
   - Enforce max value size (e.g., 1MB)
   - Reject oversized values at API layer

2. **Chunking**: 
   - Split large values into chunks
   - Store as list or multiple keys
   - Retrieve in parallel

3. **External Storage**: 
   - Store large objects in S3/blob storage
   - Cache only metadata and URLs

**Monitoring**:
- Track value size distribution
- Alert on values > 1MB
- Monitor slow commands (>10ms)

---

### Scalability Improvements

#### 1. Geographic Distribution

**Multi-Region Deployment**:
```text
Strategy:
- Deploy independent clusters in each region
- US-East, US-West, EU, Asia-Pacific
- Clients connect to nearest region

Benefits:
- Lower latency (sub-10ms in region)
- Compliance with data residency laws
- Isolation of failures

Implementation:
- Use Route53 latency-based routing
- Replicate configuration across regions
- Per-region monitoring
```text

**Cross-Region Replication** (Optional):
```text
Strategy:
- Async replication from primary to secondary regions
- Use for disaster recovery or global data

Challenges:
- High WAN latency (50-200ms)
- Potential conflicts (last-write-wins)
- Increased bandwidth costs

When to Use:
- Read-heavy workloads with global users
- Disaster recovery scenarios
- Eventually consistent global cache
```text

---

#### 2. Service Optimization

**Multi-Level Caching**:
```text
L1 Cache: Application In-Memory
- Size: 100MB per app instance
- TTL: 1-10 seconds
- Hit rate: 40-50% (hot keys)

L2 Cache: Distributed Cache (Redis)
- Size: 100GB per cluster
- TTL: 1-24 hours
- Hit rate: 80-90%

L3 Cache: Database Query Cache
- Size: 10GB
- TTL: 5-60 minutes
- Hit rate: 60-70%

Result: 95%+ combined hit rate
```text

**CDN for Static Data**:
```text
Use Cases:
- User profile images
- Static configuration
- Public reference data

Implementation:
- Store in S3 with CloudFront
- Cache-Control headers: max-age=86400
- Invalidate on updates

Benefits:
- Offload 90%+ of image traffic
- Global edge presence
- Lower costs
```text

**Query Optimization**:
```text
Techniques:
1. Use hash data structure instead of JSON strings
   - Allows field-level updates (HSET)
   - Saves bandwidth and parsing overhead

2. Pipelining for bulk operations
   - Batch 100 SETs into one network round trip
   - 10x improvement in throughput

3. Lua scripting for complex operations
   - Execute multi-step logic atomically on server
   - Reduces network round trips
```text

---

#### 3. Real-Time Features

**WebSocket for Cache Invalidation**:
```text
Problem: Clients cache data, need to know when to invalidate

Solution:
1. Clients subscribe to WebSocket connection
2. On cache update, publish invalidation message
3. Clients evict stale data from local cache

Benefits:
- Real-time consistency
- Reduced polling traffic
- Better user experience

Implementation:
- Use Redis Pub/Sub as message bus
- WebSocket server subscribes to channels
- Pushes updates to connected clients
```text

**Server-Sent Events (SSE) for Monitoring**:
```text
Use Case: Real-time cache statistics dashboard

Implementation:
1. Dashboard connects to SSE endpoint
2. Server pushes metrics every second:
   - Hit rate
   - QPS
   - Memory usage
   - Eviction rate

Benefits:
- Simple to implement (HTTP-based)
- Auto-reconnect on disconnect
- Works through firewalls
```text

---

### Monitoring and Observability

#### Metrics to Track

**System Metrics**:
```text
Latency:
- p50, p95, p99 command latency
- Target: <1ms p99
- Alert: >5ms p99

Throughput:
- Commands per second (total and by type)
- Target: 100K+ ops/sec per node
- Alert: <50K ops/sec (underutilization) or >150K (overload)

Error Rate:
- Errors per second
- Error types: timeout, connection refused, OOM
- Target: <0.01% error rate
- Alert: >0.1%

Memory:
- Used memory vs max memory
- Fragmentation ratio
- Eviction rate
- Target: 70-80% utilization
- Alert: >90% utilization
```text

**Business Metrics**:
```text
Cache Hit Rate:
- Hits / (Hits + Misses)
- Target: >80%
- Alert: <70%

Eviction Rate:
- Evictions per second
- High rate indicates undersized cache
- Alert: >1000/sec

Key Count:
- Total keys in database
- Growth rate
- Alert: Unexpected spikes or drops

TTL Distribution:
- Percentage of keys with/without expiration
- Average TTL
```text

**Infrastructure Metrics**:
```text
CPU:
- User CPU time (should be low, <30%)
- System CPU time (context switches)
- Alert: >70% total CPU

Network:
- Bandwidth usage (in/out)
- Connections count
- Alert: >8Gbps or >10K connections

Disk (for persistence):
- Write throughput for AOF
- RDB snapshot duration
- Alert: Slow disk operations
```text

---

#### Alerting Strategy

**Critical Alerts (Page immediately)**:
```text
Condition: Master node down
Threshold: No PONG response for 5 seconds
Action: Sentinel initiates failover

Condition: Memory usage critical
Threshold: >95% max memory
Action: Aggressive eviction or reject writes

Condition: Error rate spike
Threshold: >1% of requests failing
Action: Investigate cause, check logs

Time Window: Real-time (30-60 second evaluation)
```text

**Warning Alerts (Slack notification)**:
```text
Condition: High latency
Threshold: p99 > 5ms for 5 minutes
Action: Check slow log, investigate hot keys

Condition: Low hit rate
Threshold: <70% for 10 minutes
Action: Review caching strategy, increase TTLs

Condition: High eviction rate
Threshold: >1000/sec for 5 minutes
Action: Consider increasing memory

Time Window: 5-10 minutes
```text

**Informational Alerts (Log/dashboard)**:
```text
Condition: Configuration change
Threshold: CONFIG SET command executed
Action: Log for audit trail

Condition: Replica lag increasing
Threshold: >1MB replication buffer
Action: Monitor, may indicate network issues

Condition: Memory fragmentation
Threshold: >1.5 ratio
Action: Schedule maintenance for restart

Time Window: 30-60 minutes
```text

---

### Security Considerations

#### 1. Authentication & Authorization

**Basic Authentication**:
```text
CONFIG SET requirepass "strong_password_here"
AUTH strong_password_here

Limitations:
- Single password for all users
- No per-command permissions
```text

**ACL (Redis 6+)**:
```text
Define per-user permissions:

ACL SETUSER readonly on >readonly_pass ~* -@all +@read
ACL SETUSER writer on >writer_pass ~user:* +@all -@dangerous

Benefits:
- Granular permissions
- Per-user password
- Command restrictions
```text

**API Key Authentication** (for REST wrapper):
```text
HTTP Header: X-API-Key: <key>

Implementation:
- Store API keys in separate auth service
- Validate on each request
- Rate limit per key
```text

---

#### 2. Network Security

**Network Isolation**:
```text
Best Practices:
- Place cache in private subnet
- Only allow connections from application tier
- Use security groups/firewall rules
- Disable public internet access

Example (AWS):
- VPC with private subnet
- Security group: allow port 6379 from app SG only
```text

**TLS/SSL Encryption**:
```text
Enable encrypted connections:

redis.conf:
  tls-port 6380
  tls-cert-file /path/to/redis.crt
  tls-key-file /path/to/redis.key
  tls-ca-cert-file /path/to/ca.crt

Client:
  redis-cli --tls --cert redis.crt --key redis.key

Considerations:
- 10-20% performance overhead
- Required for sensitive data
- Use for cross-region replication
```text

---

#### 3. Input Validation

**Command Validation**:
```text
Risks:
- Large keys/values causing memory issues
- Dangerous commands (FLUSHDB, SHUTDOWN)

Mitigations:
1. Rename dangerous commands:
   rename-command FLUSHDB ""
   rename-command FLUSHALL ""
   rename-command CONFIG ""

2. Limit value size:
   proto-max-bulk-len 1mb

3. Validate input at application layer
```text

---

#### 4. DDoS Protection

**Rate Limiting**:
```text
Implementation:
- Per-IP connection limits
- Per-connection command rate limits
- Global QPS limits

Example:
maxclients 10000  # Max concurrent connections
timeout 300       # Close idle connections
```text

**Connection Throttling**:
```text
Strategy:
- Use load balancer with rate limiting
- Implement exponential backoff on clients
- Use connection pooling to reduce overhead

AWS NLB:
- Built-in DDoS protection
- Connection tracking
- Automatic scaling
```text

---

#### 5. Data Encryption

**Encryption at Rest** (for persistence):
```text
Options:
1. Encrypted disk volumes (AWS EBS encryption)
2. Application-level encryption before caching
3. Encrypted filesystems (LUKS, dm-crypt)

Recommendation:
- Use encrypted EBS for RDB/AOF files
- Minimal performance impact (<5%)
```text

**Encryption in Transit**:
```text
TLS for all connections:
- Client to cache
- Master to replica replication
- Sentinel to cache nodes

Required for:
- Sensitive data (PII, financial)
- Compliance (HIPAA, PCI-DSS)
- Cross-region replication
```text

---

### Future Enhancements

#### 1. Advanced Features

**Geo-Distributed CRDT-based Cache**:
- Support for multi-master writes across regions
- Conflict-free replicated data types
- Automatic conflict resolution
- Use case: Global applications with local writes

**Machine Learning Integration**:
- Predict hot keys before they become hot
- Adaptive TTL based on access patterns
- Anomaly detection for unusual access

**Intelligent Caching**:
- Auto-tune eviction policies per key pattern
- Predictive prefetching
- Cache warming strategies

---

#### 2. Performance Optimizations

**I/O Multi-Threading**:
- Keep main thread single-threaded
- Offload I/O operations to threads
- Redis 6.0 feature: 2x improvement

**GPU Acceleration**:
- Use GPU for compression/decompression
- Pattern matching operations
- Cryptographic operations

**NVMe/Optane Integration**:
- Hybrid memory architecture
- Hot data in DRAM, warm data in NVMe
- Expand capacity beyond RAM limits

---

#### 3. Operational Improvements

**Auto-Scaling**:
- Detect load patterns
- Automatically add/remove nodes
- Trigger based on CPU, memory, or QPS

**Chaos Engineering**:
- Randomly kill nodes to test resilience
- Inject latency to test timeout handling
- Verify failover procedures

**Backup and Restore**:
- Automated scheduled backups to S3
- Point-in-time recovery
- Cross-region backup replication

---

#### 4. Developer Experience

**Client Libraries**:
- Auto-retry with exponential backoff
- Connection pooling built-in
- Automatic sharding and failover handling

**Web UI Dashboard**:
- Real-time metrics visualization
- Query slow log
- Key inspection and editing
- Configuration management

**CLI Tools**:
- redis-benchmark for load testing
- redis-cli for administration
- redis-check-rdb for snapshot validation

---

#### 5. Advanced Use Cases

**Session Store**:
- Store user sessions with automatic expiration
- Fast login/logout
- Scalable across multiple web servers

**Leaderboards**:
- Use sorted sets for ranking
- Real-time score updates
- Efficient range queries

**Rate Limiting**:
- Sliding window rate limiter
- Token bucket algorithm
- Per-user or per-IP limits

**Message Queue**:
- List-based queue (LPUSH/RPOP)
- Blocking operations (BLPOP)
- Reliability with ACKs

**Real-Time Analytics**:
- HyperLogLog for cardinality estimation
- Bitmap for user activity tracking
- Aggregations with sorted sets

---

## CONCLUSION

This distributed cache system design provides a comprehensive foundation for building a high-performance, scalable, and reliable caching solution. The architecture prioritizes:

1. **Performance**: Sub-millisecond latency through in-memory storage and efficient data structures
2. **Scalability**: Horizontal scaling through sharding and replication
3. **Availability**: High availability through automatic failover and replication
4. **Flexibility**: Support for multiple data types and persistence options

The design balances trade-offs between performance, consistency, and durability based on typical caching workload requirements. Key decisions include using asynchronous replication for speed, approximate LRU for memory efficiency, and consistent hashing for scalable sharding.

With proper monitoring, security measures, and operational practices, this system can serve as the foundation for applications requiring fast, reliable access to frequently accessed data at massive scale.

---

**Note**: This design is inspired by Redis and Memcached architectures but represents a generalized approach suitable for educational and interview purposes. Production implementations should consider specific requirements, constraints, and available technologies.

