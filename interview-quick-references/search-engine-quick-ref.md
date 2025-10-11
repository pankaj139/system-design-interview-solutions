# Search Engine (Google) - Principal Engineer Level - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a search engine indexing 10B web pages with ML-powered ranking
- **Key Challenge:** <200ms p99 query latency for 100K QPS with relevance optimization
- **Scale:** 10B pages, 100K QPS, 10M updates/day, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Total Pages Indexed | 10B | Given |
| QPS | 100K queries/sec | Given |
| Raw Content Storage | 500 TB | 10B × 50 KB |
| Inverted Index Size | 16 TB | 100M terms × 10K docs × 16 bytes (compressed) |
| Link Graph Size | 16 TB | 10B × 100 links × 16 bytes |
| Total Storage (3x) | 1.7 PB | (500 + 16 + 50 + 16) TB × 3 |
| Daily Updates | 10M pages | Given |
| Search Servers | 180 | 60K actual lookups ÷ 1K + 3x redundancy |
| Index Shards | 1000 | 10B docs ÷ 10M per shard |

## 🏗️ High-Level Architecture

```text
[Web/Mobile Clients] -> [Load Balancer] -> [Query Processor] -> [Searcher]
         |                     |                  |                 |
         v                     v                  v                 v
[Query Cache (Redis)] -> [Autocomplete] -> [Index Servers (1000 shards)]
         |                     |                  |                 |
         v                     v                  v                 v
[Ranker (ML)] -> [Personalizer] -> [Inverted Index (16 TB)] -> [Document Store (500 TB)]
         |
         v
[Crawler] -> [Fetcher] -> [Parser] -> [Indexer] -> [Index Writer]
```

**Components:**
- Inverted Index: Custom B-tree + RocksDB, delta encoding, skip lists, 16 TB
- Ranker: LambdaMART (gradient boosted trees), 200+ features, 50ms inference
- PageRank: Apache Spark GraphX, 6-hour full computation, weekly updates
- Index Updates: Lambda architecture (batch + speed layer), <1 hour latency
- Query Cache: Redis, 40% hit rate, <5ms latency

## 💾 Data Model (Essentials)

### Core Structures
- **Inverted Index**: term → [{doc_id, positions, tf, field}] with delta encoding
- **Forward Index**: doc_id → {terms, frequencies, fields} for ranking features
- **Link Graph**: source_doc → [{target_doc, anchor_text}] for PageRank
- **Document Metadata**: doc_id, url, title, pagerank_score, quality_score, last_crawled_at

### PostgreSQL Tables
- **documents**: doc_id (PK), url, url_hash, pagerank_score, quality_score, crawl_status
- **links**: link_id (PK), source_doc_id (FK), target_doc_id (FK), anchor_text
- **query_logs**: log_id (PK), query_text, clicked_doc_id, response_time_ms, created_at

### Index Optimizations
- Compression: Delta encoding (6x reduction)
- Skip lists: O(√n) intersection vs O(n)
- Bloom filters: Fast term existence checks
- Sharding: 1000 shards, 10M docs each

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /search?q={query}&page=1&size=10 | Search with pagination and filtering |
| GET | /autocomplete?q={prefix} | Get query suggestions |
| GET | /spell-check?q={query} | Spell correction suggestions |

## 🚀 Critical Talking Points (Principal Engineer Level)

### Inverted Index with Advanced Optimizations
- **What:** B-tree term dictionary + compressed postings lists with skip lists
- **Why:** Enable sub-200ms search across 10B documents with 16 TB storage
- **Detail:** Delta + variable-byte encoding (6x compression), skip lists (O(√n) intersection)
- **Tradeoff:** Compression vs update speed - 6x space savings worth slower updates
- **Alternative:** Uncompressed (simple but 100 TB storage, 10x cost increase)

### ML-Powered Ranking with LambdaMART
- **What:** Gradient boosted trees with 200+ features, multi-stage ranking pipeline
- **Why:** 0.65 → 0.85 NDCG@10, 87% CTR increase, 4.5/5 user satisfaction
- **Detail:** Stage 1 (BM25, 10K), Stage 2 (features, 1K), Stage 3 (ML, 100), Stage 4 (personalization)
- **Tradeoff:** 50ms inference overhead vs relevance - worth for 30% better results
- **Alternative:** BM25 only (faster but 30% lower relevance, poor UX)

### Distributed PageRank Calculation
- **What:** MapReduce-style iterative computation on 1 trillion links
- **Why:** Global importance scores for ranking, 10-20% relevance improvement
- **Detail:** Apache Spark GraphX, 6-hour full computation on 1000 machines, weekly updates
- **Tradeoff:** 6-hour computation cost vs ranking quality - $500 per run justified
- **Alternative:** Simple domain authority (faster but misses link structure, 20% worse ranking)

### Real-Time Index Updates (Lambda Architecture)
- **What:** Batch layer (weekly full) + speed layer (real-time) + serving layer (merge)
- **Why:** <1 hour latency for new content vs 24-hour batch-only lag
- **Detail:** In-memory buffer → 10-minute flush → query-time merge with disk index
- **Tradeoff:** 20% memory overhead vs freshness - worth for breaking news/trending
- **Alternative:** Batch-only (simpler but 24-hour lag, miss trending content)

### Query Processing with Multi-Stage Retrieval
- **What:** Cheap BM25 retrieval → feature ranking → ML model → personalization
- **Why:** Balance speed and quality - can't afford ML on all 10M candidates
- **Detail:** 10K → 1K → 100 → 10 results, total 85ms (5ms parse + 20ms lookup + 50ms rank + 10ms assemble)
- **Tradeoff:** Multi-stage complexity vs performance - necessary for <200ms target
- **Alternative:** ML on all candidates (simple but 500ms latency, violates SLA)

### Index Sharding Strategy
- **What:** Document-based sharding (doc_id % 1000), parallel query across all shards
- **Why:** Scale to 10B documents, each shard handles 10M documents
- **Detail:** Scatter-gather pattern, merge with min-heap, early termination
- **Tradeoff:** Network overhead vs parallelism - 50ms saved worth 10ms network cost
- **Alternative:** Term-based sharding (simpler but load imbalance for popular terms)

## ⚖️ Key Trade-Offs (Principal Engineer Analysis)

| Decision | Choice | Alternative | Principal Engineer Analysis |
|----------|--------|-------------|----------------------------|
| Index Structure | Custom inverted index | Elasticsearch | 2x storage savings + microsecond lookups justify custom implementation |
| Ranking | LambdaMART (200+ features) | BM25 only | 30% relevance improvement + 87% CTR increase justify 50ms overhead |
| PageRank | Distributed weekly | Real-time approximate | Weekly sufficient for stability, $500/run justified by ranking quality |
| Index Updates | Lambda architecture | Batch-only | <1 hour freshness for trending content justifies 20% memory overhead |
| Query Processing | Multi-stage (4 stages) | Single-stage ML | <200ms latency requirement necessitates staged retrieval |
| Compression | Delta + variable-byte | Uncompressed | 6x storage reduction (100 TB → 16 TB) justifies slower updates |

## 🔥 Bottlenecks & Solutions (Principal Engineer Level)

### Long-Tail Query Performance
- **Root Cause:** 40% queries are unique (can't cache), 60% higher latency (150ms vs 90ms)
- **Impact:** Poor UX for 40% of queries, higher P95/P99 latency
- **Solutions:**
  - Predictive caching with ML: Predict queries likely to trend, pre-compute results
  - Aggressive skip lists for rare terms: Early termination after top-K
  - Query approximation: Sampling-based top-K (150ms → 120ms, 95% accuracy)
- **Monitoring:** Latency by query popularity, cache hit ratio, approximation accuracy

### Index Update Storm During News Events
- **Root Cause:** Breaking news causes 100x spike in updates (10M/day → 1B/day)
- **Impact:** Index lag 1 hour → 6 hours, stale results for trending topics
- **Solutions:**
  - Priority-based indexing: News sites + trending topics in priority queue
  - Elastic index workers: Auto-scale 50 → 500 workers ($200/hour burst cost)
  - Incremental indexing: Update only affected terms (6 hours → 30 minutes)
- **Monitoring:** Index update queue depth, latency by priority, worker utilization

### PageRank Staleness for New Sites
- **Root Cause:** Full PageRank weekly (6 hours), new popular sites take week to rank
- **Impact:** Poor discoverability for viral new content, missed traffic
- **Solutions:**
  - Incremental PageRank: Update only affected subgraph daily (30 minutes)
  - ML-predicted PageRank: Estimate for new sites (90% accuracy vs full)
  - Freshness boost: Temporary ranking boost for new high-quality content
- **Monitoring:** Ranking accuracy for new sites, prediction accuracy, user clicks on new content

### Shard Hotspots for Popular Terms
- **Root Cause:** Popular terms (e.g., "google") query single shard repeatedly
- **Impact:** Shard overload, 200ms+ latency vs 20ms normal
- **Solutions:**
  - Term replication: Replicate hot terms across multiple shards
  - Bloom filter optimization: Fast negative lookups reduce unnecessary shard queries
  - Caching layer: Cache postings lists for hot terms in memory
- **Monitoring:** Per-shard QPS, latency distribution, cache hit ratio for hot terms

## 💡 Interview Tips (Principal Engineer Level)

- **Start Here:** "Let me clarify - we need to index 10B pages with <200ms query latency for 100K QPS"
- **Emphasize:** Inverted index structure, ML ranking (LambdaMART), distributed PageRank, lambda architecture, query optimization
- **Be Ready For:** "How do you handle spam?", "What about duplicate content?", "How do you rank for local queries?"
- **Don't Forget:** Cost optimization (compression, caching), monitoring (latency, relevance metrics), scalability (100B pages future)

---

**Scale:** 10B pages, 100K QPS, 1.7 PB storage, 10M updates/day, <200ms p99 latency
**Key Tech:** Custom Inverted Index, LambdaMART, Apache Spark GraphX, Lambda Architecture, RocksDB
**Complexity:** ⭐⭐⭐⭐⭐ Principal Engineer Level
**Interview Time:** 60-90 minutes
