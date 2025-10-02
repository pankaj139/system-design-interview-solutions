# Pub/Sub Messaging System - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a publish-subscribe messaging system (like Google Pub/Sub, AWS SNS/SQS) supporting 100K messages/sec.

---

## 🎯 Core Problem Statement

- **What:** Design a scalable pub/sub messaging system for decoupling services
- **Key Challenge:** Deliver messages reliably to multiple subscribers with at-least-once guarantee
- **Scale:** 100K messages/sec, 1M subscribers, <50ms delivery latency, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value |
|--------|-------|
| Message Rate | 100K messages/sec |
| Subscribers | 1M total |
| Topics | 10K active |
| Delivery Latency | <50ms (p95) |
| Message Size | 10KB avg |
| Retention | 7 days |
| Throughput | 1GB/sec |
| Fan-out | 100 subscribers per topic avg |

## 🏗️ High-Level Architecture

```text
[Publishers] → [Load Balancer] → [API Gateway]
                                       ↓
                                  [Topic Service]
                                       ↓
                              [Message Queue (Kafka)]
                                       ↓
                              [Subscription Manager]
                                       ↓
                           [Push/Pull Delivery Workers]
                                       ↓
                                  [Subscribers]
```

## 🚀 Critical Talking Points

### Point 1: Topic-Based Routing

- **What:** Publishers send to topics, subscribers receive from subscriptions
- **Why:** Decouple senders from receivers, enable 1-to-many delivery
- **Detail:** Topics stored in PostgreSQL, message routing via Kafka partitions
- **Alternative:** Direct messaging (tight coupling), queue-based (no fan-out)

### Point 2: Push vs Pull Delivery

- **What:** Push (HTTP webhook) or Pull (subscriber polls) delivery models
- **Why:** Different use cases need different delivery mechanisms
- **Detail:**
  - Push: System calls subscriber webhook (real-time, requires endpoint)
  - Pull: Subscriber polls for messages (flexible, higher latency)
  - Choice: Push for real-time, Pull for batch processing
- **Alternative:** Push-only (inflexible) or pull-only (higher latency)

### Point 3: At-Least-Once Delivery

- **What:** Guarantee message delivered at least once, may have duplicates
- **Why:** Prevents message loss while maintaining high availability
- **Detail:** Retry until ACK received (exponential backoff: 1s, 2s, 4s, max 30s)
- **Alternative:** At-most-once (data loss) or exactly-once (complex/slow)

### Point 4: Dead Letter Queue

- **What:** Store messages that fail delivery after max retries
- **Why:** Prevent poison messages from blocking queue, enable debugging
- **Detail:** After 10 retries over 24 hours, move to DLQ for manual investigation
- **Alternative:** Drop failed messages (data loss) or retry forever (queue blocks)

### Point 5: Message Ordering

- **What:** Optional per-key ordering guarantee within partition
- **Why:** Some use cases require ordered processing (state machines)
- **Detail:** Use message key to route to same partition, order within partition
- **Alternative:** No ordering (simpler, more scalable) or global order (bottleneck)

## ⚖️ Key Trade-Offs

| Decision | Choice | Why |
|----------|--------|-----|
| Delivery Guarantee | At-Least-Once | Balance reliability and performance |
| Storage | Kafka | High throughput, durability, replay |
| Ordering | Per-Key Optional | Flexibility vs complexity |
| Delivery Model | Push + Pull | Support different use cases |

## 💡 Interview Tips

**Emphasize:**

- Topic/subscription model for fan-out
- At-least-once delivery with retries
- Push vs pull trade-offs
- Dead letter queue for failed messages

**Be Ready For:**

- "How to ensure delivery?" → Retries with exponential backoff + ACKs
- "What if subscriber is down?" → Push retries or pull when back online
- "How to scale?" → Partition topics, parallel workers
- "Message ordering?" → Per-key ordering via partitioning
