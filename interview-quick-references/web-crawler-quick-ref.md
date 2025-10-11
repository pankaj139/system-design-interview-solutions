# Web Crawler (Googlebot) - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a distributed web crawler to index 10B web pages efficiently and respectfully
- **Key Challenge:** Crawl 1000 pages/second while respecting robots.txt, avoiding duplicates, and handling failures gracefully
- **Scale:** 10B web pages, 1000 pages/second, 100+ crawler machines, 1M+ domains

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Total Pages | 10B | Given |
| Crawl Rate | 1000 pages/sec | Given |
| Crawler Machines | 100 | Given |
| Pages per Machine/sec | 10 | 1000 ÷ 100 |
| Average Page Size | 50KB | Given |
| Total Storage | 503.7TB | 10B × 50KB + metadata |
| Crawl Time | 115 days | 10B ÷ 1000 ÷ 86400 |
| Peak Bandwidth | 401Mbps | 1000 × 50KB + DNS |

## 🏗️ High-Level Architecture

```text
[Crawler Machines] -> [Load Balancer] -> [Master Service] -> [URL Frontier]
     |                      |                    |                |
     v                      v                    v                v
[DNS Resolver] -> [Content Storage] -> [Content Parser] -> [Bloom Filter]
     |                      |                    |                |
     v                      v                    v                v
[Robots.txt Cache] -> [Metadata DB] -> [Link Extractor] -> [Analytics]
```

**Components:**
- Crawler Machines: Go/Java, 100 machines, 10 pages/sec each
- Master Service: URL distribution, priority management, coordination
- URL Frontier: Redis, priority queues, domain-based scheduling
- Content Storage: S3/HDFS, 500TB raw content, distributed storage
- Content Parser: Python + BeautifulSoup, HTML parsing, link extraction
- Bloom Filter: Deduplication, URL normalization, fast duplicate detection
- Metadata DB: PostgreSQL, crawl statistics, domain management
- Analytics: Kafka, real-time monitoring, performance metrics

## 💾 Data Model (Essentials)

### Core Tables
- **domains**: domain_id (PK), domain_name, robots_txt, crawl_delay, priority_score
- **urls**: url_id (PK), url_hash, url, domain_id (FK), status, priority_score, depth
- **crawled_content**: url_hash (PK), crawl_timestamp, content, content_hash
- **crawl_logs**: crawler_id, timestamp, url_hash, status_code, response_time

### Key Indexes
- urls: (status, priority_score) - for frontier queries
- domains: (priority_score, last_crawled_at) - for scheduling
- crawled_content: (url_hash, crawl_timestamp) - for content retrieval

### Partitioning Strategy
- PostgreSQL: Shard by domain_id for URLs table
- Cassandra: Partition by url_hash for content, by crawler_id for logs
- Redis: Cluster by geographic regions for URL frontier

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /crawler/urls | Get URLs for crawling with priority |
| POST | /crawler/content | Submit crawled content and extracted links |
| POST | /crawler/failures | Report crawl failures for retry |
| POST | /domains | Add new domain for crawling |
| GET | /domains/{id}/statistics | Get domain crawl statistics |
| GET | /crawler/status | Get overall crawler system status |

## 🚀 Critical Talking Points

### URL Frontier Management
- **What:** Priority queue system with domain-based scheduling and politeness policies
- **Why:** Need to crawl important pages first while respecting robots.txt and crawl delays
- **Detail:** Redis sorted sets for priority, separate queues per domain, Bloom filter for deduplication
- **Alternative:** PostgreSQL with custom indexing (more complex, higher latency)

### Distributed Crawler Architecture
- **What:** Master-worker pattern with 100 crawler machines and centralized coordination
- **Why:** Scale to 1000 pages/second requires distributed processing across multiple machines
- **Detail:** Master distributes URLs, workers crawl pages, heartbeat monitoring, automatic failover
- **Alternative:** Peer-to-peer architecture (more complex coordination, harder to manage)

### Content Parser and Link Extraction
- **What:** HTML parsing with BeautifulSoup to extract content, links, and metadata
- **Why:** Need to extract meaningful content and discover new URLs for crawling
- **Detail:** Robust HTML parsing, URL normalization, content quality scoring, metadata extraction
- **Alternative:** Go with goquery (faster but less parsing features)

### Politeness and Rate Limiting
- **What:** Respect robots.txt, implement crawl delays, and avoid overwhelming target servers
- **Why:** Maintain good relationships with website owners and avoid being blocked
- **Detail:** Robots.txt parsing, domain-specific delays, exponential backoff on failures
- **Alternative:** Aggressive crawling (faster but unsustainable, risk of IP blocks)

### Deduplication Strategy
- **What:** Bloom filter for fast duplicate detection combined with content hashing
- **Why:** Avoid crawling the same content multiple times, save bandwidth and storage
- **Detail:** URL normalization, content hash comparison, probabilistic duplicate detection
- **Alternative:** Database-based deduplication (more accurate but slower)

### Fault Tolerance and Retry Logic
- **What:** Graceful handling of failures with exponential backoff and circuit breaker pattern
- **Why:** Network failures, server errors, and timeouts are common in web crawling
- **Detail:** Retry failed URLs, exponential backoff, circuit breaker for failing domains
- **Alternative:** Simple retry (less sophisticated, may overwhelm failing servers)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Database | Multi-DB (PostgreSQL/Cassandra/Redis) | Single database | Each optimized for specific access patterns |
| Crawling Strategy | Politeness-first | Speed-first | Sustainable long-term vs short-term gains |
| Content Storage | Raw + Processed | Processed only | Flexibility vs storage cost |
| Parser Language | Python + BeautifulSoup | Go + goquery | Rich parsing vs performance |
| Architecture | Master-Worker | P2P | Simpler coordination vs decentralization |
| Deduplication | Bloom Filter | Database | Speed vs accuracy |

## 🔥 Bottlenecks & Solutions

### URL Frontier Memory Usage
- **Problem:** Large URL frontier consuming too much memory
- **Solution:** Partitioning by domain, disk-based storage for low-priority URLs, compression

### DNS Resolution Bottleneck
- **Problem:** DNS lookups causing delays and limiting throughput
- **Solution:** DNS caching with TTL, multiple DNS servers, pre-resolution

### Network Bandwidth Limitations
- **Problem:** Bandwidth limiting crawl speed and causing timeouts
- **Solution:** Content compression, HTTP/2 multiplexing, intelligent bandwidth allocation

### Content Processing Bottleneck
- **Problem:** HTML parsing and link extraction slowing down crawlers
- **Solution:** Async processing, content caching, optimized parsing libraries

## 💡 Interview Tips

- **Start Here:** "Let me clarify the requirements - we need to crawl 10B pages at 1000 pages/second"
- **Emphasize:** Distributed systems, politeness policies, fault tolerance, deduplication
- **Be Ready For:** "How do you handle JavaScript-heavy sites?", "What about infinite loops?"
- **Don't Forget:** Robots.txt compliance, DNS caching, content compression, monitoring

---

**Scale:** 10B pages, 1000 pages/sec, 100 machines, 1M+ domains
**Key Tech:** Redis, PostgreSQL, Cassandra, Python, Go, Bloom Filter
**Complexity:** ⭐⭐⭐ Hard
**Interview Time:** 45-60 minutes
