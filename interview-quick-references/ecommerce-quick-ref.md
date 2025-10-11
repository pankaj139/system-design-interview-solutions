# E-commerce Website (Amazon) - Principal Engineer Level - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a full-featured e-commerce platform supporting 100M products, 500M users, and 1M orders/day
- **Key Challenge:** <500ms page load, 99.99% checkout uptime, strong inventory consistency, 10x traffic during flash sales
- **Scale:** 500M users, 100M products, 1M orders/day, peak 10x during sales, 100M page views/day

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Active Users | 50M | Given |
| Total Products | 100M | Given |
| Daily Orders | 1M | Given (10M during flash sales) |
| Daily Page Views | 1B | 50M DAU × 2 sessions × 10 pages |
| Peak QPS | 115K PV/s | 11.5K normal × 10x flash sale |
| Peak Orders/Second | 1,150 OPS | 115 normal × 10x flash sale |
| Total Storage | 263 TB | Products (251 TB) + Orders (9 TB) + Users (2.5 TB) |
| API Servers | 464 | 232 normal + 2x redundancy |
| Database Servers | 195 | 20 primary + 80 replicas + inventory (30) |

## 🏗️ High-Level Architecture

```text
[Web/Mobile Clients] -> [CDN (CloudFront, 250 TB images)] -> [Global LB] -> [Regional LB]
         |                         |                             |              |
         v                         v                             v              v
[API Gateway (Kong)] -> [Product Service] -> [Cart Service] -> [Order Service]
         |                    |                    |                  |
         v                    v                    v                  v
[PostgreSQL (Products)] -> [Redis (Cart)] -> [PostgreSQL (Orders)] -> [Inventory DB]
         |                    |                    |                  |
         v                    v                    v                  v
[Elasticsearch (Search)] -> [Kafka Events] -> [Payment Gateway] -> [Recommendation Engine]
```

**Components:**
- Product Catalog: PostgreSQL + Elasticsearch, 100M products, <100ms search
- Shopping Cart: Redis primary + PostgreSQL backup, <5ms add-to-cart
- Inventory: PostgreSQL with strong consistency, optimistic locking, flash sale Redis DECR
- Order Processing: PostgreSQL partitioned by date, Kafka for async processing
- Payment: Stripe integration, PCI DSS compliant, fraud detection ML
- Recommendation: Collaborative filtering + content-based + deep learning

## 💾 Data Model (Essentials)

### Core Tables
- **products**: product_id (PK), seller_id (FK), sku, name, category_id (FK), base_price, average_rating
- **inventory**: inventory_id (PK), product_id (FK), warehouse_id (FK), quantity_available, quantity_reserved
- **shopping_carts**: cart_id (PK), user_id (FK), status, expires_at
- **orders**: order_id (PK), user_id (FK), order_number, subtotal, tax, shipping, total, order_status, payment_status
- **order_items**: order_item_id (PK), order_id (FK), product_id (FK), quantity, unit_price, total_price

### Advanced Indexes
- products: (category_id, average_rating, status), (category_id, base_price), Full-text on (name, description, brand)
- inventory: (product_id, warehouse_id), (quantity_available) for stock checks
- orders: (user_id, created_at), (order_status), Partition by created_at (monthly)

### Caching Strategy
- Redis (cart): 62 GB per region, TTL 24h guest / 30d authenticated
- Redis (product): Top 1% hot products (1M), TTL 5 min, 10 GB
- Redis (search): Query result cache, TTL 5 min, 2 GB
- Redis (session): 50 GB for 10M concurrent sessions

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /products/search | Search products with filters, facets, pagination |
| GET | /products/{id} | Get complete product details with variants |
| POST | /cart/items | Add item to cart with inventory reservation |
| POST | /checkout/complete | Complete checkout with idempotency key |
| GET | /orders/{id} | Get order details with tracking info |

## 🚀 Critical Talking Points (Principal Engineer Level)

### Product Catalog with Elasticsearch
- **What:** PostgreSQL primary + Elasticsearch for search, CDC sync with Debezium + Kafka
- **Why:** 100x faster search vs PostgreSQL LIKE, faceted search, BM25 ranking
- **Detail:** 63 nodes (21 shards × 3 replicas), <100ms search, 85% cache hit rate
- **Tradeoff:** Complexity vs performance - eventual consistency (1-5s lag) acceptable for search
- **Alternative:** PostgreSQL only (simpler but 10x slower search, no facets)

### Shopping Cart with Redis + Inventory Reservation
- **What:** Redis primary (fast), PostgreSQL backup (persistence), 15-minute soft reservation
- **Why:** Sub-5ms add-to-cart, high throughput, prevent overselling
- **Detail:** 12-node Redis cluster, WebSocket sync across devices, background job releases expired
- **Tradeoff:** Complexity vs speed - Redis provides <5ms vs 50ms PostgreSQL
- **Alternative:** PostgreSQL only (simpler but 10x slower, lower throughput)

### Inventory Management with Strong Consistency
- **What:** PostgreSQL with optimistic locking (normal), pessimistic locking (flash sales), Redis DECR (flash sales)
- **Why:** Prevent overselling (0.01% rate), customer trust, strong consistency required
- **Detail:** Version column for CAS, SELECT FOR UPDATE for contention, Redis atomic operations
- **Tradeoff:** Consistency vs throughput - strong consistency prevents overselling, worth latency
- **Alternative:** Eventual consistency (higher throughput but overselling risk)

### Flash Sale Handling
- **What:** Pre-load hot inventory to Redis, use DECR for atomic operations, rate limiting per user
- **Why:** Handle 10x traffic (10M orders/day), prevent database overload
- **Detail:** 1000 hot SKUs in Redis, DECR + rollback on negative, async sync to PostgreSQL
- **Tradeoff:** Complexity vs scale - Redis handles 10,000 orders/second vs 100 PostgreSQL
- **Alternative:** PostgreSQL only (simpler but can't handle 10x traffic, timeouts)

### Order Processing Pipeline
- **What:** Synchronous payment + order creation (<2s), asynchronous fulfillment via Kafka
- **Why:** User expects immediate confirmation, background processing doesn't block
- **Detail:** ACID transaction for order+payment, Kafka events for email/shipping/inventory
- **Tradeoff:** Sync vs async - payment must be sync, everything else async for speed
- **Alternative:** All synchronous (simpler but 10s+ order creation, poor UX)

### Payment Integration with Stripe
- **What:** Third-party gateway (Stripe), tokenization, separate auth/capture, fraud detection ML
- **Why:** PCI DSS compliance out-of-box, global payment methods, fraud protection
- **Detail:** 3D Secure, CVV/AVS checks, ML risk scoring, circuit breaker for gateway failures
- **Tradeoff:** Transaction fees (2.9%) vs compliance - worth it for PCI DSS + reliability
- **Alternative:** Build in-house (no fees but years of compliance work, higher risk)

### Recommendation Engine
- **What:** Hybrid ML (collaborative filtering + content-based + deep learning), precomputed + real-time
- **Why:** 50% CTR increase, 52% conversion increase, $15M additional annual revenue
- **Detail:** Daily batch training, hourly incremental, Redis cache (1h TTL), <10ms p95
- **Tradeoff:** Complexity vs revenue - measurable $15M impact justifies ML investment
- **Alternative:** Simple "popular products" (easier but 50% lower engagement)

## ⚖️ Key Trade-Offs (Principal Engineer Analysis)

| Decision | Choice | Alternative | Principal Engineer Analysis |
|----------|--------|-------------|----------------------------|
| Primary DB | PostgreSQL | MongoDB | ACID compliance critical for orders/inventory, complex joins needed |
| Search | Elasticsearch | PostgreSQL LIKE | 100x faster, faceted search, better relevance ranking |
| Cart Storage | Redis + PostgreSQL | PostgreSQL only | Sub-5ms latency justifies dual storage complexity |
| Inventory Consistency | Strong (ACID) | Eventual | Prevent overselling (customer trust), worth latency tradeoff |
| Flash Sale | Redis DECR | PostgreSQL | 100x throughput (10K OPS vs 100 OPS) handles 10x traffic |
| Payment | Stripe | Build in-house | PCI compliance + reliability justify 2.9% transaction fees |
| Recommendation | ML Ensemble | Rules-based | $15M revenue increase justifies ML complexity |

## 🔥 Bottlenecks & Solutions (Principal Engineer Level)

### Product Search Performance
- **Root Cause:** 100M products, complex filters, need <100ms response
- **Impact:** Slow search kills conversions, 50% users abandon if >3s
- **Solutions:**
  - Elasticsearch with 21 shards × 3 replicas: 100x faster than PostgreSQL
  - Query result caching in Redis: 85% hit rate, <2ms cached queries
  - CDN for product images: 99% cache hit, reduces origin load 99%
- **Monitoring:** Search latency, Elasticsearch node health, cache hit ratio

### Flash Sale Inventory Contention
- **Root Cause:** 10x traffic (10M orders/day), database locks cause timeouts
- **Impact:** Overselling, angry customers, revenue loss
- **Solutions:**
  - Redis DECR for hot inventory: Atomic operations, 10K orders/second
  - Pessimistic locking (SELECT FOR UPDATE): Prevents overselling during contention
  - Rate limiting per user: Max 1 purchase, prevents hoarding
  - Pre-warming: Load hot inventory to Redis before sale
- **Monitoring:** Order success rate, overselling rate, Redis performance

### Cart Abandonment and Sync Issues
- **Root Cause:** Cart not syncing across devices, lost on session expiry
- **Impact:** 70% cart abandonment rate (industry average)
- **Solutions:**
  - Store by user_id not session: Persists across devices
  - WebSocket real-time sync: Updates all devices instantly
  - PostgreSQL backup: Recover carts after Redis failure
  - Abandoned cart emails: Recover 15% of abandoned carts
- **Monitoring:** Cart abandonment rate, sync latency, recovery rate

### Payment Gateway Failures
- **Root Cause:** Stripe downtime (0.1%), network issues, timeouts
- **Impact:** Lost orders, revenue loss, customer frustration
- **Solutions:**
  - Circuit breaker pattern: Failover to secondary gateway
  - Retry logic with exponential backoff: 3 retries (1s, 5s, 15s)
  - Idempotency keys: Prevent duplicate charges on retry
  - Multi-gateway support: Automatic failover (Stripe → PayPal)
- **Monitoring:** Payment success rate, gateway latency, error rates

### Database Write Bottlenecks
- **Root Cause:** 1M orders/day + inventory updates + reviews = high write load
- **Impact:** Slow writes, connection pool exhaustion, timeouts
- **Solutions:**
  - Sharding by date for orders: Distribute writes across databases
  - Read replicas: 1 primary + 4 replicas, 95% reads to replicas
  - Connection pooling: HikariCP with 200 connections, prevent exhaustion
  - Async writes via Kafka: Non-critical writes buffered and batched
- **Monitoring:** Database CPU/IO, connection pool utilization, replication lag

## 💡 Interview Tips (Principal Engineer Level)

- **Start Here:** "Let me clarify - we need 99.99% uptime for checkout with strong inventory consistency for 100M products"
- **Emphasize:** Strong consistency for inventory, flash sale handling, payment integration, recommendation impact
- **Be Ready For:** "How do you prevent overselling?", "What about PCI compliance?", "How do you handle refunds?"
- **Don't Forget:** Cost analysis ($263 TB storage), security (PCI DSS), monitoring (SLA/SLO/SLI), disaster recovery

---

**Scale:** 500M users, 100M products, 1M orders/day (10M during flash sales), 99.99% checkout uptime
**Key Tech:** PostgreSQL + Elasticsearch, Redis Cluster, Kafka, Stripe, TensorFlow, CloudFront CDN
**Complexity:** ⭐⭐⭐⭐⭐ Principal Engineer Level
**Interview Time:** 60-90 minutes
