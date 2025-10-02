# Distributed Stream Processing - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a distributed stream processing system (Apache Kafka + Flink) handling 1M events/sec with exactly-once semantics.

---

## 🎯 Core Problem Statement

- **What:** Design a real-time stream processing system for analytics and event-driven applications
- **Key Challenge:** Process millions of events per second with low latency and exactly-once guarantees
- **Scale:** 1M events/sec, <100ms processing latency, exactly-once semantics, fault-tolerant

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Event Ingestion Rate | 1M events/sec | Target throughput |
| Daily Events | 86.4B | 1M × 86,400 |
| Event Size | 1KB avg | JSON event payload |
| Daily Data Volume | 86TB | 86.4B × 1KB |
| Processing Latency | <100ms | End-to-end (p95) |
| Kafka Partitions | 1000 | Parallelism = partitions |
| Retention Period | 7 days | Hot data window |
| Storage (7 days) | 600TB | 86TB × 7 |
| Consumer Lag | <1 minute | Real-time target |

## 🏗️ High-Level Architecture

```text
[Event Producers] → [Kafka Cluster] → [Stream Processors] → [Sinks]
                    (Ingestion)      (Flink/Spark)         (DB/Cache)
                         ↓
                  [ZooKeeper] ← Coordination
                         ↓
               [State Backend] ← Checkpointing
```

**Components:**

- **Kafka**: Distributed log for event ingestion (1000 partitions)
- **Stream Processor**: Flink/Spark Streaming for transformations
- **State Backend**: RocksDB for stateful operations
- **Checkpointing**: Periodic snapshots for exactly-once semantics
- **Sinks**: Elasticsearch, PostgreSQL, Redis for output

## 💾 Data Model (Essentials)

### Event Schema

```json
{
  "event_id": "uuid",
  "event_type": "page_view",
  "timestamp": "2025-10-02T10:30:00Z",
  "user_id": "user123",
  "properties": {...}
}
```

### Kafka Topic Structure

```text
Topic: events.{domain}.{event_type}
Partitions: 1000 (key-based partitioning)
Replication: 3
Retention: 7 days
```

## 🚀 Critical Talking Points

### Point 1: Kafka for Event Ingestion

- **What:** Distributed commit log for high-throughput event streaming
- **Why:** Handle 1M+ events/sec with durability and replay capability
- **Detail:**
  - Partitions for parallelism (1 consumer per partition)
  - Immutable log with offset-based consumption
  - Zero-copy transfers for performance
  - Producer batching (batch size: 16KB, linger: 10ms)
- **Alternative:** RabbitMQ (lower throughput), SQS (managed but limited), Kinesis (AWS-only)

### Point 2: Exactly-Once Semantics

- **What:** Guarantee each event processed exactly once, no duplicates or losses
- **Why:** Critical for financial transactions, billing, aggregations
- **Detail:**
  - Checkpointing: Periodic state snapshots (every 60s)
  - Two-phase commit: Coordinate state + output atomically
  - Idempotent producers: Kafka deduplication with producer IDs
  - Transactional writes: Atomic multi-partition writes
- **Alternative:** At-least-once (duplicates), at-most-once (data loss)

### Point 3: Stateful Processing with RocksDB

- **What:** Embedded key-value store for maintaining state across events
- **Why:** Enable windowed aggregations, joins, and complex event processing
- **Detail:**
  - State: Counters, windows, session data stored locally
  - Checkpoint: Async snapshots to HDFS/S3 for recovery
  - Size: 10-100GB per task for hot state
  - Performance: <1ms local lookups vs 10ms+ remote
- **Alternative:** External database (slow), in-memory only (data loss)

### Point 4: Windowing Strategies

- **What:** Group events into time-based or count-based windows
- **Why:** Compute aggregations over meaningful time periods
- **Detail:**
  - Tumbling: Fixed non-overlapping windows (0-60s, 60-120s)
  - Sliding: Overlapping windows (every 10s, 60s duration)
  - Session: Gap-based windows (idle timeout: 30 min)
  - Watermarks: Handle late events (max lateness: 5 min)
- **Alternative:** Global aggregation (unbounded), microbatching (higher latency)

### Point 5: Backpressure Handling

- **What:** Slow down producers when consumers can't keep up
- **Why:** Prevent OOM and maintain system stability
- **Detail:**
  - Monitor: Consumer lag (events behind real-time)
  - React: Reduce batch size, add more consumers, drop events
  - Buffer: Kafka holds data (7 days), consumers catch up
  - Alert: Lag > 1 minute triggers scaling/investigation
- **Alternative:** Drop events (data loss), crash (downtime)

### Point 6: Fault Tolerance

- **What:** Recover from failures without data loss or duplication
- **Why:** System must be resilient to node crashes
- **Detail:**
  - Checkpoints: State snapshots every 60s to distributed storage
  - Recovery: Reload state from last checkpoint + replay events
  - Kafka retention: Events available for replay (7 days)
  - Task restart: Automatic rescheduling on different node
- **Alternative:** No checkpointing (data loss), synchronous replication (slow)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Message Broker | Kafka | RabbitMQ / Kinesis | High throughput, replay capability |
| Processing | Flink | Spark Streaming / Storm | True streaming, exactly-once support |
| State Backend | RocksDB | In-memory / External DB | Balance of performance and durability |
| Semantics | Exactly-Once | At-least-once | Critical for accuracy despite complexity |
| Windowing | Event Time | Processing Time | Correct results despite late events |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution | Monitoring |
|------------|----------|-----------|
| Kafka Write Throughput | Increase partitions, batch producer writes | Producer throughput, broker CPU |
| Consumer Lag | Add parallel consumers, increase processing capacity | Consumer lag, processing rate |
| State Size Growth | TTL on state, incremental checkpoints | State size, checkpoint duration |
| Checkpoint Duration | Async snapshots, faster storage (SSD) | Checkpoint time, alignment time |
| Network Bandwidth | Compress events, co-locate services | Network I/O, data transfer |

## 💡 Interview Tips

**Start Here:**

1. Clarify requirements: event rate, latency, semantics (exactly-once?)
2. Calculate: events/sec, storage, partitions needed
3. Draw architecture: Kafka → Processor → Sinks
4. Explain exactly-once semantics

**Emphasize:**

- Kafka for high-throughput ingestion
- Exactly-once processing guarantees
- Stateful processing with checkpointing
- Windowing for time-based aggregations
- Backpressure and fault tolerance

**Be Ready For:**

- "How do you ensure exactly-once?" → Checkpointing + idempotent writes
- "What if processor fails?" → Reload state from checkpoint + replay events
- "How to handle late events?" → Watermarks with allowed lateness
- "How to scale?" → Add partitions + parallel consumers
- "What about hot partitions?" → Better partition key (user_id, not timestamp)
- "How to monitor?" → Consumer lag, processing rate, checkpoint duration

**Don't Forget:**

- Mention partition key design (critical for scaling)
- Discuss windowing strategy (tumbling vs sliding)
- Address backpressure handling
- Talk about checkpoint frequency trade-offs
- Consider late event handling with watermarks

**Key Concepts:**

```text
Event Time vs Processing Time:
- Event Time: When event actually occurred (use this!)
- Processing Time: When system processes event
- Watermark: Tracks event time progress

Checkpointing:
1. Pause processing
2. Snapshot state to durable storage
3. Record Kafka offsets
4. Resume processing
Frequency: Every 60s (trade-off: overhead vs recovery)
```

**Partition Key Selection:**

- User_id: Groups user events together (user-level state)
- Session_id: Session-based processing
- Geographic region: Regional processing
- Avoid: Timestamp (creates hot partitions), random (no ordering)

**Performance Optimization:**

- Producer batching: 16KB batches, 10ms linger time
- Consumer fetch size: 1MB min for throughput
- Parallelism: Match partition count
- State: Local RocksDB for <1ms access
- Network: Co-locate Kafka and processors

**Common Mistakes:**

- Using processing time instead of event time
- Not handling late events (data loss)
- Ignoring backpressure (OOM crashes)
- No checkpointing (data loss on failure)
- Too many partitions (coordination overhead)
- Poor partition key (hot partitions)
