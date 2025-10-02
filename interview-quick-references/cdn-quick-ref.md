# CDN (Content Delivery Network) - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a global CDN (CloudFlare/Akamai) serving 1M requests/sec globally.

---

## 🎯 Core Problem Statement

- **What:** Design a geographically distributed caching system for static content delivery
- **Key Challenge:** Minimize latency globally while maintaining cache consistency and availability
- **Scale:** 1M requests/sec, 10Tbps bandwidth, <50ms p95 latency globally, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value |
|--------|-------|
| Global RPS | 1M requests/sec |
| Bandwidth | 10Tbps |
| PoP Count | 200+ locations |
| Cache Hit Rate | >90% |
| Origin Hit Rate | <10% |
| p95 Latency | <50ms |
| Avg Object Size | 100KB |

## 🏗️ High-Level Architecture

```text
[User] → [DNS (GeoDNS)] → [Nearest PoP]
                                ↓
                        [Edge Cache (L1)]
                                ↓
                        [Regional Cache (L2)]
                                ↓
                        [Origin Servers]
```

## 🚀 Critical Talking Points

### Point 1: GeoDNS Routing

- **What:** Route users to geographically nearest PoP (Point of Presence)
- **Why:** Minimize network latency (each 1000 miles ~50ms RTT)
- **Detail:**
  - Anycast IP: Same IP announced from multiple locations
  - BGP routing to nearest PoP
  - Health checks: Remove failed PoPs from DNS
- **Alternative:** Single datacenter (high latency) or round-robin (suboptimal routing)

### Point 2: Multi-Tier Caching

- **What:** Edge caches (L1) backed by regional caches (L2)
- **Why:** Reduce origin load while maintaining high hit rate
- **Detail:**
  - L1 (Edge): 200+ locations, 10TB cache each, serves 95% requests
  - L2 (Regional): 10 locations, 100TB cache each, serves 4% requests
  - Origin: Serves only 1% (cache misses)
- **Alternative:** Single-tier (more origin load) or no regional (slower misses)

### Point 3: Cache Invalidation

- **What:** Remove or update cached content when origin changes
- **Why:** Prevent serving stale content
- **Detail:**
  - TTL-based: Auto-expire after configurable time (15min - 1 year)
  - Purge API: Immediate invalidation by URL/tag
  - Versioned URLs: cdn.com/v2/style.css (cache-busting)
  - Propagation: 30-60s globally via message bus
- **Alternative:** No invalidation (stale content) or always fresh (defeats caching)

### Point 4: Cache Key Design

- **What:** Unique identifier for cached objects including variations
- **Why:** Serve different content based on device, location, etc.
- **Detail:**
  - Base: URL + query params
  - Vary: Add headers (Accept-Encoding, User-Agent)
  - Example: `GET /image.jpg?size=large + gzip + mobile`
- **Alternative:** URL-only (can't vary by headers) or too many keys (low hit rate)

### Point 5: Pull vs Push Model

- **What:** Pull: Cache fetches from origin on miss. Push: Origin uploads to CDN
- **Why:** Pull is simpler and automatic, Push for high-traffic events
- **Detail:**
  - Pull (default): Lazy loading, origin hit on first request
  - Push: Pre-warm cache before launch/event
  - Hybrid: Pull with strategic pre-warming
- **Alternative:** Push-only (manual overhead) or no pre-warming (launch spike)

### Point 6: Origin Shield

- **What:** Additional cache layer to protect origin from cache miss storms
- **Why:** Collapse concurrent misses for same object into single origin request
- **Detail:**
  - Regional shield between PoPs and origin
  - Deduplicates simultaneous requests
  - Reduces origin load by 90%+
- **Alternative:** Direct to origin (thundering herd) or wait/lock (higher latency)

## ⚖️ Key Trade-Offs

| Decision | Choice | Why |
|----------|--------|-----|
| Routing | GeoDNS + Anycast | Lowest latency to users |
| Cache Tiers | 2-tier (Edge + Regional) | Balance hit rate and origin load |
| Invalidation | TTL + Purge API | Flexibility for different content types |
| Model | Pull (with pre-warm) | Simplicity with performance option |

## 🔥 Bottlenecks & Solutions

| Bottleneck | Solution |
|------------|----------|
| Cache Miss Storm | Origin shield for request collapsing |
| Hot Files | Increase replication, serve from memory |
| Slow Origin | Longer TTL, origin optimization |
| Stale Content | Aggressive purging, shorter TTL |

## 💡 Interview Tips

**Start Here:**

1. Draw multi-tier architecture (Edge → Regional → Origin)
2. Explain GeoDNS routing
3. Discuss cache invalidation strategies
4. Calculate storage needed

**Emphasize:**

- GeoDNS for geographic routing
- Multi-tier caching for efficiency
- Cache invalidation strategies
- Origin shield for protection
- >90% cache hit rate target

**Be Ready For:**

- "How to route users?" → GeoDNS + Anycast to nearest PoP
- "How to invalidate cache?" → TTL + Purge API + versioned URLs
- "What if origin is slow?" → Longer TTL, shield, origin optimization
- "How to handle cache miss storm?" → Origin shield collapses requests
- "How to vary by device?" → Cache key includes User-Agent header
- "How to pre-warm cache?" → Push model before events

**Don't Forget:**

- Mention cache hit rate as key metric (>90%)
- Discuss TTL strategy (static: 1 year, dynamic: 5 min)
- Address cache key design (URL + headers)
- Talk about compression (gzip/brotli)
- Consider DDoS protection at edge

**Key Formulas:**

```text
Cache Hit Rate = (Cache Hits / Total Requests) × 100%
Target: >90% for effectiveness

Origin Reduction = 1 - (1 - Edge_Hit_Rate) × (1 - Regional_Hit_Rate)
Example: 1 - (1 - 0.95) × (1 - 0.80) = 99% reduction

Storage per PoP = (Objects × Size × Replication) / PoP_Count
Example: 1B × 100KB × 3 / 200 = 1.5TB per PoP
```

**Common Mistakes:**

- Not using GeoDNS (poor global latency)
- No origin shield (vulnerable to cache miss storms)
- Poor cache key design (low hit rate)
- No invalidation strategy (stale content)
- Single-tier caching (high origin load)
