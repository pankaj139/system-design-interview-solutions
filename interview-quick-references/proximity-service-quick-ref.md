# Proximity Service (Yelp) - Principal Engineer Level - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a location-based business search service with advanced geospatial indexing and ML-powered ranking
- **Key Challenge:** Sub-100ms response time for 50K QPS while handling high-density areas (10K+ businesses/km²)
- **Scale:** 100M businesses, 500M users, 50K QPS, 100M searches/day, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Total Businesses | 100M | Given |
| Total Users | 500M | Given |
| Peak QPS | 50K queries/sec | Given |
| Daily Searches | 100M | 100M DAU × 3 searches |
| Business Data Storage | 500GB | 100M × 5KB |
| Photo Storage | 200TB | 100M × 10 photos × 500KB (CDN optimized) |
| Geospatial Index Size | 60GB | 100M × 200 bytes × 3x overhead |
| API Servers Needed | 200 | 50K QPS ÷ 500 + 2x redundancy |
| Cache Size | 170GB/region | Hot locations + queries + user prefs |

## 🏗️ High-Level Architecture

```text
[Mobile/Web Clients] -> [CDN + Geographic LB] -> [Regional LB] -> [API Gateway]
         |                      |                      |              |
         v                      v                      v              v
[Rate Limiter] -> [Search Service] -> [Business Service] -> [Review Service]
         |              |                      |                      |
         v              v                      v                      v
[Geospatial Cache] -> [Geospatial Search] -> [PostgreSQL+PostGIS] -> [Cassandra]
         |              |                      |                      |
         v              v                      v                      v
[Redis GeoHash] -> [Elasticsearch] -> [ML Ranking Service] -> [Analytics]
```

**Components:**
- Search Service: Go, geospatial queries, <100ms latency
- Geospatial Search Engine: Elasticsearch + PostGIS, R-tree indexes, QuadTree for high density
- Geospatial Cache: Redis with GEOADD, 95% hit rate, sub-5ms latency
- ML Ranking Service: Python/TensorFlow, multi-factor scoring, personalization
- Business Database: PostgreSQL + PostGIS, ACID compliance, spatial indexes
- Review Database: Cassandra, high write throughput, time-series data
- Photo Storage: AWS S3 with CloudFront CDN, 200TB storage

## 💾 Data Model (Essentials)

### Core Tables
- **businesses**: business_id (PK), name, location (GEOGRAPHY), category_id (FK), rating, price_level, is_active, geohash (computed)
- **categories**: category_id (PK), name, parent_category_id (FK), icon_url, display_order
- **business_hours**: hours_id (PK), business_id (FK), day_of_week, open_time, close_time, is_closed
- **reviews** (Cassandra): business_id (PK), review_id, user_id, rating, review_text, created_at (clustering key)
- **users**: user_id (PK), email, username, location (GEOGRAPHY), city, state, country

### Advanced Indexes
- businesses: GIST index on location for spatial queries, composite index on (category_id, rating, is_active)
- businesses: Geohash index for fast bucketing, Full-text index on (name, description, address)
- reviews: Partition by business_id, clustering by created_at DESC

### Geospatial Strategy
- Geohash: 6-8 characters for prefix filtering (6 chars = ±0.61km, 8 chars = ±0.019km)
- PostGIS R-tree: Precise distance calculations with ST_DWithin
- QuadTree: Dynamic subdivision for high-density areas (>1000 businesses/km²)
- Redis GEOADD: Hot location caching with GEORADIUS queries

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| POST | /search/nearby | Search businesses by location with advanced filters |
| GET | /businesses/{id} | Get complete business details with reviews |
| POST | /businesses | Create/update business profile |
| POST | /businesses/{id}/reviews | Submit review with rating |
| GET | /businesses/{id}/analytics | Get business analytics and metrics |

## 🚀 Critical Talking Points (Principal Engineer Level)

### Advanced Geospatial Indexing
- **What:** Hybrid approach with Geohash + PostGIS R-tree + QuadTree + Redis
- **Why:** Handle 100M businesses with <100ms queries, support high-density areas (10K/km²)
- **Detail:** 6-8 char Geohash for bucketing, PostGIS for precise distance, QuadTree for >1000/km² density
- **Tradeoff:** Complexity vs performance - hybrid provides consistent <100ms across all densities
- **Alternative:** Single approach (Geohash only) fails in high-density areas (2500ms → timeouts)

### Multi-Tier Geospatial Caching
- **What:** 3-tier caching (L1 application cache, L2 Redis GeoHash, L3 query result cache)
- **Why:** Achieve sub-10ms response time for 95% of queries, reduce database load by 85%
- **Detail:** L1 (10K hot locations, <1ms), L2 (1M locations, <5ms), L3 (10M queries, <10ms)
- **Tradeoff:** Consistency vs performance - accept 2-5 minute eventual consistency
- **Alternative:** No caching (simple but 50ms+ latency, can't handle 50K QPS)

### ML-Powered Ranking and Personalization
- **What:** Multi-factor ranking (distance, quality, popularity, personalization) with ML models
- **Why:** Increase user engagement by 40% CTR, 75% ad revenue increase, 4.2 → 4.6 satisfaction
- **Detail:** Collaborative filtering + GBT + neural networks, <3ms inference with quantization
- **Tradeoff:** Complexity vs engagement - measurable 40% improvement justifies investment
- **Alternative:** Rule-based ranking (simple but 40% lower engagement)

### High-Density Area Optimization
- **What:** QuadTree with adaptive subdivision for areas >1000 businesses/km²
- **Why:** Manhattan 5km radius = 785K businesses, need efficient handling
- **Detail:** Dynamic subdivision (max 500 per leaf), 8 levels deep, ~100MB per city
- **Tradeoff:** Memory vs performance - 100MB overhead provides 96% latency improvement
- **Alternative:** Simple distance sorting (easier but 2500ms timeouts in Manhattan)

### Database Sharding Strategy
- **What:** Shard businesses by geohash prefix with 10 shards total
- **Why:** Handle 10M business updates/day, reduce write contention by 90%
- **Detail:** Consistent hashing on geohash, 10M businesses per shard, scatter-gather for cross-shard
- **Tradeoff:** Complexity vs scalability - enables 10x write throughput
- **Alternative:** Single database (simple but write contention causes 200ms+ latency during peaks)

### Elasticsearch Near Real-Time Indexing
- **What:** CDC pipeline with Debezium + Kafka + Elasticsearch for <15 second lag
- **Why:** Business hours updates must appear immediately in search results
- **Detail:** Bulk API (500 docs/10s), separate indexes for core vs dynamic data
- **Tradeoff:** Infrastructure complexity vs freshness - 5-10 min → 15 sec (95% improvement)
- **Alternative:** Batch indexing (simple but 5-10 minute delays, user complaints)

## ⚖️ Key Trade-Offs (Principal Engineer Analysis)

| Decision | Choice | Alternative | Principal Engineer Analysis |
|----------|--------|-------------|----------------------------|
| Geospatial Indexing | Hybrid (Geohash+PostGIS+QuadTree) | Single approach | Hybrid handles all density scenarios, prevents timeouts |
| Database | PostgreSQL+PostGIS | MongoDB | ACID compliance + best spatial functions justify complexity |
| Caching | Multi-tier + Short TTL | Single tier/No cache | 95% hit rate + sub-10ms latency justifies memory cost |
| ML Ranking | Multi-model Ensemble | Rule-based | 40% CTR improvement + 75% revenue increase justifies investment |
| High-Density | QuadTree Optimization | Linear Scan | 2500ms → 85ms (96% improvement) prevents timeouts |
| Writes | Write-behind + Sharding | Synchronous | 200ms → 5ms (97% improvement) handles peak traffic |

## 🔥 Bottlenecks & Solutions (Principal Engineer Level)

### Geospatial Query Performance in High-Density Areas
- **Root Cause:** Linear scan of 785K businesses in Manhattan 5km radius (10K/km² × 78.5km²)
- **Impact:** 2500ms query time vs 50ms target, causes timeouts, poor UX
- **Solutions:**
  - Hierarchical QuadTree: 2500ms → 85ms (96% improvement)
  - Geohash prefix filtering: Narrows to ~1000 businesses before distance calc
  - PostGIS bounding box pre-filter: ST_Expand before ST_DWithin
  - Result limiting: Max 500 results, score threshold filtering
- **Monitoring:** Query latency by geohash prefix, density distribution, QuadTree depth

### ML Model Inference Latency
- **Root Cause:** Real-time ML model inference taking 30-50ms per query
- **Impact:** Total response time 80-120ms, violates 100ms SLA for 30% of queries
- **Solutions:**
  - Model quantization: FP32 → INT8 (4x speedup, 40ms → 3ms)
  - Precomputed embeddings: Business/user embeddings cached in Redis
  - Hybrid ranking: Rule-based for 95% of queries, ML for complex cases
  - TensorRT GPU acceleration: 10x speedup for batch inference
- **Monitoring:** Inference latency percentiles, model accuracy (NDCG), fallback rate

### Database Write Contention During Peak Hours
- **Root Cause:** 10M business updates/day causing write locks on primary database
- **Impact:** 200ms+ write latency during lunch/dinner rush, cascading delays
- **Solutions:**
  - Write-behind caching: Accept writes to Redis (5ms ack), batch to DB
  - Database sharding: 10 shards by geohash prefix, 90% contention reduction
  - Read replicas: 1 primary + 5 replicas per shard, 95/5 read/write split
  - Async processing: Non-critical writes (photos, reviews) via Kafka
- **Monitoring:** Write queue depth, connection pool utilization, replication lag

### Elasticsearch Index Update Lag
- **Root Cause:** Business updates taking 5-10 minutes to appear in search results
- **Impact:** Stale business hours, incorrect open/closed status, user complaints
- **Solutions:**
  - CDC pipeline: Debezium + Kafka + Elasticsearch (<1 second lag)
  - Near real-time indexing: Bulk API (500 docs/10s), 5-second refresh
  - Hybrid search: Recent updates from PostgreSQL, older from Elasticsearch
  - Index optimization: Separate core vs dynamic indexes (70% overhead reduction)
- **Monitoring:** Index update latency, CDC pipeline lag, query hybrid strategy usage

## 💡 Interview Tips (Principal Engineer Level)

- **Start Here:** "Let me clarify the requirements - we need to handle high-density areas like Manhattan with 10K+ businesses per km²"
- **Emphasize:** Geospatial algorithms (QuadTree, R-tree), hybrid caching, ML ranking, database sharding, real-time indexing
- **Be Ready For:** "How do you handle boundary cases in geospatial queries?", "What about GDPR compliance for location data?", "How do you prevent review spam?"
- **Don't Forget:** Security (rate limiting, input validation), monitoring (comprehensive metrics), disaster recovery (multi-region), cost optimization (CDN, caching)

---

**Scale:** 100M businesses, 500M users, 50K QPS, 100M searches/day, 99.99% availability
**Key Tech:** PostGIS, Redis GEOADD, Elasticsearch, QuadTree, TensorFlow, Cassandra
**Complexity:** ⭐⭐⭐⭐⭐ Principal Engineer Level
**Interview Time:** 60-90 minutes
