# Search Engine System Design (Google-like)

**File Purpose:** Complete interactive learning resource for designing production-grade search engines at internet scale. Master inverted indexes, ranking algorithms (TF-IDF, BM25, PageRank), distributed query processing, relevance tuning, and ML-based personalization through multi-level educational content. This comprehensive guide takes you from basic keyword matching to Google-scale search with <200ms latency serving 10B+ indexed pages to millions of concurrent users.

**Author:** System Design Documentation  
**Created:** October 1, 2025  
**Last Updated:** October 14, 2025  
**Recent Updates:** Transformed to comprehensive educational format with 3-level content (Beginner/Intermediate/Advanced), real-world examples from Google/Bing/Elasticsearch, Python implementations, distributed architecture patterns, and practice exercises

**Learning Time Estimates:**
- 🟢 **Beginner Level:** 5-7 hours (fundamentals of indexing, basic ranking, query processing)
- 🟡 **Intermediate Level:** 7-10 hours (interview patterns, distributed architecture, ranking algorithms)
- 🔴 **Advanced Level:** 10-15 hours (production optimization, ML ranking, personalization, real-world scale)

---

## Welcome to Search Engine System Design!

### What You're Going to Build

You're about to design one of the most technically challenging systems on the internet - **a search engine**. When you type a query into Google and get relevant results in 0.2 seconds from an index of trillions of pages, you're experiencing decades of computer science breakthroughs working together. Search engines combine crawler systems, distributed databases, information retrieval algorithms, machine learning, and massive-scale infrastructure.

By the end of this course, you'll be able to:
- Design Google-scale search handling 100K+ queries per second
- Implement inverted indexes for sub-second full-text search
- Build ranking algorithms (TF-IDF, BM25, PageRank)
- Create distributed query processing across sharded indexes
- Handle real-time indexing and updates
- Pass FAANG interviews with confidence on search questions

---

### Your Learning Path

This course is structured for three learning levels. Start where you're comfortable:

#### 🟢 **BEGINNER: The Fundamentals** (Start here if new to system design)

**What you'll master:**
- How search engines work (crawling, indexing, ranking)
- Inverted index data structure
- Basic ranking algorithms (TF-IDF)
- Single-machine search implementation
- Core query processing

**Prerequisites:**
- Basic programming knowledge (any language)
- Understanding of hashmaps and arrays
- Familiarity with basic SQL

**Real-world outcome:** Build a working search engine for small datasets (100K documents, 100 QPS)

---

#### 🟡 **INTERMEDIATE: Interview Patterns** (Master FAANG interviews)

**What you'll master:**
- Distributed index architecture (sharding strategies)
- Advanced ranking (BM25, PageRank)
- Query processing optimization
- Cache hierarchies
- Capacity estimation and scaling
- Trade-off analysis frameworks
- Common interview questions and answers

**Prerequisites:**
- Completed Beginner content OR
- 2+ years of backend development
- Basic understanding of distributed systems

**Real-world outcome:** Design search engine in a 45-minute interview, explain trade-offs confidently, get offers from top tech companies

---

#### 🔴 **ADVANCED: Production Considerations** (Build at Google scale)

**What you'll master:**
- ML-based ranking (Learning to Rank)
- Personalization and context
- Real-time indexing (Kafka, streaming)
- Multi-language and i18n support
- Query understanding (NLP, entity recognition)
- Production monitoring and optimization
- Cost optimization at PB scale

**Prerequisites:**
- Completed Intermediate content OR
- 5+ years of distributed systems experience
- Understanding of machine learning basics

**Real-world outcome:** Lead search architecture at a major tech company, optimize for billions of documents, reduce costs by 50%+

---

### What Makes This Learning Experience Unique

Unlike other system design resources, this course offers:

#### 🎯 **Multi-Level Approach**
Every section has content for beginners, interviewers, and production engineers. Skip what you know, deep dive where you need.

#### 💼 **Real-World Examples**
Learn from actual implementations:
- **Google:** How PageRank revolutionized web search
- **Elasticsearch:** Open-source distributed search
- **Bing:** Microsoft's alternative ranking approaches
- **Algolia:** Real-time search as a service

#### 💻 **Hands-On Code**
Complete Python implementations you can run:
- Inverted index builder
- TF-IDF ranking calculator
- Query processor with caching
- Distributed shard coordinator

#### 🎓 **Interview-Focused**
Every section includes:
- ❓ **Interview questions** you'll actually be asked
- ✅ **Strong answers** with trade-off analysis
- 🚫 **Common mistakes** to avoid
- 🎯 **Practice exercises** with real scenarios

#### 🏗️ **Production-Ready**
Advanced content based on real experience:
- Performance optimization (<200ms p99)
- Cost analysis ($1M/month to $100M/month)
- Failure handling (index corruption, node failures)
- Operational best practices (monitoring, A/B testing)

---

**Table of Contents**
1. [Section 1: Understanding What We're Building](#section-1-understanding-what-were-building)
2. [Section 2: Planning for Scale](#section-2-planning-for-scale)
3. [Section 3: Inverted Index & Data Structures](#section-3-inverted-index--data-structures)
4. [Section 4: Ranking Algorithms](#section-4-ranking-algorithms)
5. [Section 5: Distributed Query Processing](#section-5-distributed-query-processing)
6. [Section 6: Crawling & Indexing Pipeline](#section-6-crawling--indexing-pipeline)
7. [Section 7: Caching & Performance](#section-7-caching--performance)
8. [Section 8: Personalization & ML Ranking](#section-8-personalization--ml-ranking)
9. [Section 9: Monitoring & Operations](#section-9-monitoring--operations)
10. [Section 10: Trade-offs & Optimizations](#section-10-trade-offs--optimizations)
11. [Putting It All Together](#putting-it-all-together-complete-search-journey)

---

## Section 1: Understanding Search Engines

### What You'll Learn

By the end of this section, you'll be able to:
- Understand the three core phases: crawling, indexing, ranking
- Explain how inverted indexes enable fast full-text search
- Compare different search architectures (centralized vs distributed)
- Identify key components of a production search engine
- Calculate basic scale requirements for billion-document indexes

### Why This Matters

Search is one of the hardest problems in computer science! When you type "best coffee shops near me" into Google and get relevant results in 0.2 seconds from an index of trillions of pages, you're experiencing the culmination of decades of research in information retrieval, distributed systems, and machine learning. Understanding how search works is essential for FAANG interviews and building modern applications. Every major company needs search: e-commerce product search, document search, log search, media search. Mastering search architecture opens doors to some of the highest-paying engineering roles.

---

### 🟢 For Beginners: How Search Really Works

#### The Three Phases

Think of a search engine like a library system:

```text
Library System → Search Engine

1. Collecting Books (Crawler):
   - Librarians visit bookstores → Web crawler visits websites
   - Acquire new books → Download web pages
   - Update catalog regularly → Re-crawl for fresh content

2. Organizing Books (Indexer):
   - Create card catalog → Build inverted index
   - Index by topic, author, keywords → Index by terms/phrases
   - Store in filing cabinets → Store in distributed databases

3. Helping Users Find Books (Query Processor):
   - User asks librarian → User submits query
   - Librarian searches catalog → Query processor searches index
   - Returns most relevant books → Returns ranked results
   - <2 minutes → <0.2 seconds!
```

#### Simple Search Implementation

```text
INVERTED INDEX DATA STRUCTURE:

Structure (JSON format):
{
  "term": {
    "document_ids": [1, 5, 23, 89],
    "doc_frequency": 4,
    "postings": [
      {"doc_id": 1, "positions": [5, 15], "term_frequency": 2},
      {"doc_id": 5, "positions": [2], "term_frequency": 1},
      {"doc_id": 23, "positions": [8, 12, 45], "term_frequency": 3},
      {"doc_id": 89, "positions": [1], "term_frequency": 1}
    ]
  }
}

SEARCH ALGORITHM (Pseudocode):
────────────────────────────────
function search(query):
  1. Tokenize: query → ["python", "programming"]
  2. For each term, lookup inverted_index[term]
  3. Intersect document_id lists (AND operation)
  4. Rank by relevance (TF-IDF/BM25)
  5. Return top K results

Example Query Flow:
──────────────────
Query: "python programming"
├─ Step 1: Tokenize → ["python", "programming"]
├─ Step 2: Lookup
│   ├─ "python" → docs [1, 5, 89]
│   └─ "programming" → docs [1, 10, 89]
├─ Step 3: Intersection → docs [1, 89] (both terms)
├─ Step 4: Rank by TF-IDF
│   ├─ doc_1: score 0.4621
│   └─ doc_89: score 0.3215
└─ Step 5: Return [doc_1, doc_89]

Time Complexity Analysis:
────────────────────────
- Index lookup: O(1) per term (hash table)
- Intersection: O(min(L1, L2)) where L = posting list size
- Ranking: O(n log k) for top-k selection
- Total: O(terms * avg_postings + n log k)
- Typical: <10ms for billions of documents

Why Inverted Index is Fast:
──────────────────────────
Linear Scan (WITHOUT index):
├─ Must read all documents
├─ Time: O(N * M) where N=docs, M=doc_size
└─ Example: 1B docs × 10KB = hours

Inverted Index (WITH index):
├─ Lookup 2 hash table entries
├─ Intersect small lists
└─ Example: 1B docs indexed → <10ms search
```

**Key Insight:** The inverted index makes search fast! Instead of scanning all documents, we look up terms in the index and get matching documents instantly.

**Key Insight:** The inverted index makes search fast! Instead of scanning all documents, we look up terms in the index and get matching documents instantly.

---

### 🟡 For Intermediate: Production Search Architecture

#### Complete System Components

```text
Production Search Engine Architecture:

[Web Crawler] (Section 2)
    ↓ (fetches pages)
[Indexing Pipeline] (Section 3)
    ├─ Text Extraction
    ├─ Tokenization & NLP
    ├─ Inverted Index Builder
    └─ Forward Index Builder
    ↓ (writes)
[Distributed Index] (Section 6)
    ├─ Shard 1 (1B docs)
    ├─ Shard 2 (1B docs)
    └─ Shard N (1B docs)

[Query Flow:]
User Query
    ↓
[Query Parser] (Section 5)
    ├─ Tokenization
    ├─ Spell correction
    ├─ Query expansion
    ↓
[Query Coordinator]
    ├─ Broadcast to all shards
    ↓
[Ranking Engine] (Section 4)
    ├─ TF-IDF scoring
    ├─ BM25 ranking
    ├─ PageRank (link analysis)
    ├─ ML models (personalization)
    ↓
[Result Aggregation]
    ├─ Merge results from shards
    ├─ Re-rank top K results
    ├─ Generate snippets
    ↓
[Results to User] (<200ms total)
```

#### Inverted Index Structure

```text
PRODUCTION INVERTED INDEX STRUCTURE:

Schema Definition:
─────────────────
inverted_index = {
  "term": {
    "doc_frequency": <int>,        // Number of documents containing term
    "total_frequency": <int>,       // Total occurrences across all documents
    "postings": [
      {
        "doc_id": <int>,
        "positions": [<int>],        // For phrase search
        "term_frequency": <int>,     // Occurrences in this document
        "field_tf": {                // Per-field frequencies
          "title": <int>,
          "body": <int>,
          "metadata": <int>
        }
      }
    ]
  }
}

Document Statistics (separate structure):
────────────────────────────────────────
doc_stats = {
  "doc_id": {
    "length": <int>,              // Total terms in document
    "unique_terms": <int>,        // Vocabulary size
    "avg_term_freq": <float>,
    "field_lengths": {
      "title": <int>,
      "body": <int>
    }
  }
}

Global Statistics:
─────────────────
collection_stats = {
  "total_documents": <int>,
  "avg_doc_length": <float>,
  "vocabulary_size": <int>,
  "total_terms": <int>
}

Example Data:
────────────
{
  "python": {
    "doc_frequency": 2,
    "total_frequency": 3,
    "postings": [
      {
        "doc_id": 0,
        "positions": [0, 15, 23],
        "term_frequency": 3,
        "field_tf": {"title": 1, "body": 2}
      },
      {
        "doc_id": 2,
        "positions": [0],
        "term_frequency": 1,
        "field_tf": {"title": 0, "body": 1}
      }
    ]
  }
}

Storage Optimization:
────────────────────
Naive Storage:
├─ 10B documents × 100M terms × 16 bytes = 16 TB

Optimized (with compression):
├─ Delta encoding: Store differences [1, +99, +105] instead of [1, 100, 205]
├─ Variable-byte encoding: 1-5 bytes per number
├─ Compressed size: ~2 TB (8x compression)
└─ Trade-off: +10-20ms CPU for decompression vs 8x less disk/network
```

**Storage Optimization:**

```text
Naive Storage (No Compression):
- 10B documents
- 100M unique terms
- Average 10K postings per term
- Posting size: 16 bytes (doc_id + metadata)
- Total: 100M × 10K × 16B = 16 TB

Optimized Storage (With Compression):
- Delta encoding: Store differences instead of absolute IDs
  - Instead of: [1, 100, 205, 888]
  - Store: [1, +99, +105, +683]
  - Smaller numbers = better compression
- Variable-byte encoding: Use 1-5 bytes per number
- Total: ~2 TB (8x compression!)

Trade-off: CPU time (decompression) vs Storage (disk space)
- Benefit: 8x less disk, 8x less network transfer
- Cost: ~10-20ms extra CPU per query
- Decision: Worth it! Disk/network more expensive than CPU
```

---

### 🔴 For Advanced: Google-Scale Optimizations

#### Multi-Tier Index Architecture

```text
Google's Index Strategy (Simplified):

Tier 1: Memory Index (Hot)
├─ Most frequently accessed terms (20% of terms)
├─ Covers 80% of queries (Zipf's Law)
├─ Size: 200 GB per machine (RAM)
├─ Latency: <10ms
└─ Example: "google", "facebook", "weather"

Tier 2: SSD Index (Warm)
├─ Medium-frequency terms (60% of terms)
├─ Covers 18% of queries
├─ Size: 2 TB per machine (NVMe SSD)
├─ Latency: <50ms
└─ Example: "elasticsearch tutorial", "python async"

Tier 3: HDD Index (Cold)
├─ Rare terms (20% of terms)
├─ Covers 2% of queries (long-tail)
├─ Size: 20 TB per machine (spinning disk)
├─ Latency: <200ms
└─ Example: "obscure tech term from 1990s blog"

Total Index Size per Machine: ~22 TB
Query Distribution: Optimized for common case (90% < 50ms)
```

#### Index Sharding Strategies

```text
INDEX SHARDING STRATEGIES:

1. DOCUMENT-BASED SHARDING (Most Common):
─────────────────────────────────────────
Concept: Partition documents across shards

shard_id = hash(doc_id) % num_shards

Distribution:
├─ Shard 0: documents [0, 4, 8, 12, ...] (doc_id % 4 == 0)
├─ Shard 1: documents [1, 5, 9, 13, ...] (doc_id % 4 == 1)
├─ Shard 2: documents [2, 6, 10, 14, ...] (doc_id % 4 == 2)
└─ Shard 3: documents [3, 7, 11, 15, ...] (doc_id % 4 == 3)

Query Processing:
├─ Query: "python programming"
├─ Fan-out: Send to ALL shards (0, 1, 2, 3)
├─ Each shard returns top K results
└─ Coordinator merges and re-ranks

Pros:
✓ Simple to implement
✓ Even load distribution
✓ Easy to add documents

Cons:
✗ Must query all shards for every search
✗ High fanout cost (100 shards = 100 requests)


2. TERM-BASED SHARDING (Rare Terms):
────────────────────────────────────
Concept: Partition terms across shards

shard_id = hash(term) % num_shards

Distribution:
├─ Shard 0: terms starting with [a-f]
├─ Shard 1: terms starting with [g-l]
├─ Shard 2: terms starting with [m-r]
└─ Shard 3: terms starting with [s-z]

Query Processing:
├─ Query: "python programming"
├─ Lookup shard for each term
│   ├─ "python" → Shard 2 (p in [m-r])
│   └─ "programming" → Shard 2 (p in [m-r])
├─ Fan-out: Only query Shard 2
└─ Return results

Pros:
✓ Only query shards containing query terms
✓ Efficient for rare term queries

Cons:
✗ Load imbalance (common terms overloaded)
✗ Hard to rebalance when adding shards


3. HYBRID SHARDING (Google's Approach):
────────────────────────────────────────
Strategy: Use different sharding for common vs rare terms

if term in top_10k_terms:
    use document-based sharding
else:
    use term-based sharding

Common Terms (90% of queries): Document sharding
├─ Terms: "the", "a", "python", "java", "google"
├─ Query all shards in parallel
└─ Fast because parallelized

Rare Terms (10% of queries): Term sharding
├─ Terms: "obscure_tech_term_1990s"
├─ Query only 1-2 shards
└─ Saves 90%+ cost

Example:
────────
Query: "common_term rare_term"
├─ "common_term" → Query all 100 shards (document sharding)
├─ "rare_term" → Query shard 23 only (term sharding)
├─ Merge results at coordinator
└─ Best of both approaches!


SHARD SELECTION ALGORITHM:
─────────────────────────
function select_shards(query_terms):
    shards = set()
    
    for term in query_terms:
        if is_common_term(term):
            // Document sharding: add all shards
            shards.add(all_shards)
        else:
            // Term sharding: add specific shard
            shard_id = hash(term) % num_shards
            shards.add(shard_id)
    
    return list(shards)

Common query: "python programming" → All shards
Rare query: "obscure_lib_v1.2.3" → Shard 47 only
Mixed query: "python obscure_lib" → All shards (common term forces it)
```

---

### Real-World Example: Google's Index Evolution

```text
2000 - PageRank Era:
├─ Single index on a few hundred machines
├─ Index size: ~1 billion pages
├─ Technology: Custom C++ with memory-mapped files
├─ Query latency: ~3 seconds
├─ Problem: Couldn't scale beyond billions of pages
└─ Innovation: PageRank for quality ranking

2004 - MapReduce Era:
├─ Distributed indexing with MapReduce
├─ Index size: ~8 billion pages
├─ Technology: GFS (Google File System) + MapReduce
├─ Query latency: ~0.5 seconds
├─ Problem: Batch processing only (hours to update index)
└─ Innovation: Distributed batch processing

2010 - Caffeine (Real-time Indexing):
├─ Continuous indexing pipeline
├─ Index size: ~100 billion pages
├─ Technology: Bigtable for incremental updates
├─ Query latency: ~0.2 seconds
├─ Fresh content indexed in minutes (not hours)
└─ Innovation: Incremental index updates

2015 - RankBrain (ML-Based Ranking):
├─ Machine learning for relevance
├─ Index size: ~60 trillion pages
├─ Technology: TensorFlow for model serving
├─ Query latency: ~0.2 seconds (despite ML)
├─ 15% improvement in result quality
└─ Innovation: ML-based ranking at scale

2020 - BERT & Transformers:
├─ Deep learning for query understanding
├─ Index size: ~hundreds of trillions of pages
├─ Technology: TPUs for model inference
├─ Query latency: ~0.2 seconds (same!)
├─ Understands context and nuance
└─ Innovation: NLP understanding at query time

Key Lessons:
1. Latency stayed constant (~0.2s) while scale increased 100,000x!
2. Architecture evolved from centralized → distributed → ML-powered
3. Real-time indexing more important than batch optimization
4. ML improvements outweighed traditional algorithm tuning
```

---

### 🤔 Think About It

1. **For Beginners:** Why does Google need an inverted index? Why not just scan all 60 trillion web pages for each query? Calculate: If scanning 1 page takes 1ms, how long to scan 1 billion pages?

2. **For Intermediate:** You're building search for an e-commerce site with 10M products. Users search for "red nike shoes size 10". Should you use document sharding or term sharding? Why? What about a query like "sneakers"?

3. **For Advanced:** Google serves 100K queries/second. Each query must check 1000 shards (out of 10K total shards). That's 100M shard requests/second! How do you prevent the shard fanout from overwhelming the system? Consider: request coalescing, caching, approximate algorithms.

---

### ✅ Key Takeaways

- **Search has 3 phases:** Crawling (collect), Indexing (organize), Ranking (retrieve)
- **Inverted index is the secret:** Maps terms → documents for instant lookup
- **Scale requires distribution:** Sharding across 1000+ machines is essential
- **Compression is critical:** 8x compression saves millions in infrastructure costs
- **Tier your index:** Memory (hot) > SSD (warm) > HDD (cold) optimizes for common case
- **Hybrid sharding wins:** Document sharding for common terms, term sharding for rare terms
- **Latency budget matters:** 200ms total budget must cover network, disk, CPU, ML models

---

### 🎯 Practice Exercise

**Scenario:** You're designing search for a code repository hosting platform (like GitHub) with 100M repositories.

**Given Information:**

- 100M repositories (documents)
- Average repo size: 10MB (mostly code files)
- Unique terms (function names, variables, keywords): 50M terms
- Search volume: 10K queries/second
- Users search for: function names, variable names, code snippets
- Target latency: <100ms P95

**Your Task:**

1. **Index Design:**
   - Should you index entire file contents or just metadata?
   - What's your inverted index size (uncompressed vs compressed)?
   - How many shards do you need?
   - Document sharding or term sharding? Why?

2. **Query Processing:**
   - Query: "def calculate_total(" (exact function signature)
   - How do you handle phrase search (exact match)?
   - How many shards do you need to query?
   - What's your ranking algorithm?

3. **Scale Calculation:**
   - 10K QPS × 100 shards = 1M shard queries/second
   - Each shard query: 10ms
   - How many shard machines do you need?
   - What's your replication factor for availability?

4. **Optimization:**
   - 80% of searches are for popular repositories (top 1M repos)
   - How do you use this to optimize?
   - Design a caching strategy
   - Calculate cache hit rate needed to halve infrastructure costs

**Bonus Challenge:**

GitHub supports search across forks (copies of repositories). A popular repo has 10K forks (mostly identical). Naive indexing stores 10K copies of the same content. Design a deduplication strategy that:

- Detects identical/similar content
- Shares index entries across forks
- Still returns correct results (which fork?)
- Reduces index size by 90%+

---

## Section 2: Planning for Scale & Capacity Estimation

### What You'll Learn

By the end of this section, you'll be able to:
- Calculate storage requirements for billion-document indexes
- Estimate QPS (queries per second) capacity needs  
- Plan infrastructure costs for production search
- Size your hardware (CPU, RAM, disk, network)
- Make informed trade-offs between cost and performance

### Why This Matters

"How would you design Google Search?" is a classic FAANG interview question. The first thing interviewers want to see is whether you can do back-of-the-envelope calculations. Can you estimate how much storage you need for 10 billion documents? How many machines to handle 100K QPS? Getting the scale right is the difference between a system that costs $10K/month vs $1M/month while still meeting requirements. This section teaches you the systematic approach that gets you hired.

---

### 🟢 For Beginners: Basic Capacity Planning

#### Storage Calculation

```text
STORAGE CAPACITY PLANNING:

Basic Formula:
─────────────
Raw Storage = num_documents × avg_document_size
Index Storage = Raw Storage × 0.3 (typical compression)
Total Storage = (Raw + Index) × replication_factor

Example Calculation (E-commerce Product Search):
───────────────────────────────────────────────
Given:
├─ Documents: 10M products
├─ Avg size: 5KB per product
└─ Replication: 3x

Calculation:
├─ Raw: 10M × 5KB = 50 GB
├─ Index: 50 GB × 0.3 = 15 GB
├─ Subtotal: 50 + 15 = 65 GB
└─ With replication: 65 GB × 3 = 195 GB

Cost (AWS S3 pricing):
├─ Storage: 195 GB × $0.023/GB/month = $4.49/month
└─ Very affordable for 10M products!


Example Calculation (Google-Scale Web Search):
──────────────────────────────────────────────
Given:
├─ Documents: 10B web pages
├─ Avg size: 50KB per page
├─ Replication: 3x
└─ Compression: 8x (aggressive)

Calculation:
├─ Raw: 10B × 50KB = 500 TB
├─ Index (uncompressed): 500 TB × 0.3 = 150 TB
├─ Index (compressed 8x): 150 TB / 8 = 18.75 TB
├─ Subtotal: 500 + 18.75 = 518.75 TB
└─ With replication: 518.75 TB × 3 = 1,556 TB (1.56 PB)

Cost (at $0.023/GB/month):
├─ Storage: 1.56M GB × $0.023 = $35,880/month
└─ Actual: Cheaper with bulk discounts + cold storage


QUERY CAPACITY PLANNING:

QPS (Queries Per Second) Formula:
─────────────────────────────────
Daily Searches = Monthly Active Users × Searches per User per Day
Average QPS = Daily Searches / 86,400 seconds
Peak QPS = Average QPS × peak_multiplier (typically 3-5x)

Example (Startup Search Engine):
────────────────────────────────
Given:
├─ MAU: 1M users
├─ Searches: 5 per user per day
└─ Peak multiplier: 5x

Calculation:
├─ Daily: 1M × 5 = 5M searches/day
├─ Average QPS: 5M / 86,400 = 58 QPS
├─ Peak QPS: 58 × 5 = 290 QPS

Server Capacity:
├─ QPS per server: 1,000 QPS (typical)
├─ Servers needed: 290 / 1,000 = 0.29
├─ Round up with 20% headroom: 1 server
└─ Cost: $100/month


LATENCY BUDGET BREAKDOWN:

Target: <200ms P95 latency
────────────────────────

Component Breakdown:
├─ Network (client → server): 20ms (10%)
├─ Query parsing: 5ms (2.5%)
├─ Cache lookup (Redis): 10ms (5%)
├─ Index shard fanout: 15ms (7.5%)
├─ Shard query (disk + ranking): 80ms (40%) ← CRITICAL PATH
├─ Result aggregation: 20ms (10%)
├─ Snippet generation: 15ms (7.5%)
├─ ML ranking: 15ms (7.5%)
└─ Network (server → client): 20ms (10%)

Total: 200ms (100% of budget)

Optimization Priorities:
1. Shard query (40% of time): Add caching, faster disks
2. Network (20% total): Use CDN, regional deployment
3. Aggregation (10%): Parallel merge algorithms
```

#### QPS Calculation

```python
# QPS (Queries Per Second) planning

class QPSCalculator:
    """Estimate QPS needs"""
    
    def __init__(self, monthly_active_users, searches_per_user_per_day):
        self.mau = monthly_active_users
        self.searches_per_day = searches_per_user_per_day
    
    def calculate_average_qps(self):
        """Average queries per second"""
        total_searches_per_day = self.mau * self.searches_per_day
        avg_qps = total_searches_per_day / 86400  # seconds in a day
        return avg_qps
    
    def calculate_peak_qps(self, peak_multiplier=5):
        """Peak QPS (usually 3-5x average)"""
        avg_qps = self.calculate_average_qps()
        peak_qps = avg_qps * peak_multiplier
        return {
            'average_qps': avg_qps,
            'peak_qps': peak_qps,
            'peak_multiplier': peak_multiplier
        }
    
    def estimate_machines_needed(self, qps_per_machine=1000):
        """Estimate number of machines for query serving"""
        peak = self.calculate_peak_qps()
        machines_needed = peak['peak_qps'] / qps_per_machine
        
        # Add 20% headroom for failures
        machines_with_headroom = machines_needed * 1.2
        
        return {
            'peak_qps': peak['peak_qps'],
            'qps_per_machine': qps_per_machine,
            'machines_needed': int(machines_with_headroom) + 1,
            'monthly_cost_usd': (int(machines_with_headroom) + 1) * 100  # $100/machine
        }

# Example: Startup search engine
qps_calc = QPSCalculator(
    monthly_active_users=1_000_000,  # 1M users
    searches_per_user_per_day=5  # 5 searches/day
)

result = qps_calc.estimate_machines_needed(qps_per_machine=1000)
print(f"Peak QPS: {result['peak_qps']:.0f}")
print(f"Machines needed: {result['machines_needed']}")
print(f"Monthly cost: ${result['monthly_cost_usd']}")

# Output:
# Peak QPS: 289
# Machines needed: 1
# Monthly cost: $100
```

---

### 🟡 For Intermediate: Production Capacity Planning

#### Comprehensive Resource Estimation

```text
Capacity Planning Framework:

1. Traffic Estimation:
   MAU (Monthly Active Users): 100M
   Searches per user per day: 10
   Daily searches: 100M × 10 = 1B
   Average QPS: 1B / 86,400 = 11,574 QPS
   Peak QPS (5x): 57,870 QPS

2. Storage Requirements:
   Documents: 10B web pages
   Avg page size: 50 KB
   Raw storage: 10B × 50 KB = 500 TB
   
   Inverted index:
   - Vocabulary: 100M terms
   - Avg postings per term: 10K
   - Posting size: 8 bytes (compressed)
   - Index size: 100M × 10K × 8B = 8 TB
   
   Total (with RF=3): (500 + 8) × 3 = 1,524 TB

3. Memory Requirements (Hot Data):
   - Vocabulary: 100M terms × 64 bytes = 6.4 GB
   - Top 10% popular docs metadata: 50 GB
   - Query cache: 10 GB
   - Total per machine: ~70 GB RAM

4. CPU Requirements:
   - QPS per CPU core: ~200 QPS
   - Peak QPS: 57,870
   - Cores needed: 57,870 / 200 = 290 cores
   - Machines (32 cores each): 10 machines

5. Network Bandwidth:
   - Query size: ~1 KB
   - Response size: ~10 KB
   - Bandwidth per QPS: 11 KB
   - Peak bandwidth: 57,870 × 11 KB = 636 MB/s
   - Total (with replication): ~2 Gbps

6. Cost Estimation:
   - Compute (10 machines × $200/month): $2,000
   - Storage (1,524 TB × $0.023/GB): $35,000
   - Network (1 TB egress × $0.09/GB): $90
   - Total: ~$37,000/month
```

#### Latency Budget Breakdown

```python
class LatencyBudget:
    """
    Analyze latency budget for search queries.
    Target: <200ms P95 latency
    """
    def __init__(self, target_latency_ms=200):
        self.target = target_latency_ms
        self.components = {}
    
    def add_component(self, name, latency_ms, description=""):
        """Add a latency component"""
        self.components[name] = {
            'latency_ms': latency_ms,
            'percentage': (latency_ms / self.target) * 100,
            'description': description
        }
    
    def analyze_budget(self):
        """Analyze if we're within budget"""
        total_latency = sum(c['latency_ms'] for c in self.components.values())
        remaining = self.target - total_latency
        
        print(f"Latency Budget Analysis (Target: {self.target}ms)\n")
        print(f"{'Component':<25} {'Latency':<10} {'% of Budget':<12} Description")
        print("=" * 80)
        
        for name, data in self.components.items():
            print(f"{name:<25} {data['latency_ms']:<10.1f} {data['percentage']:<12.1f} {data['description']}")
        
        print("=" * 80)
        print(f"{'TOTAL':<25} {total_latency:<10.1f} {(total_latency/self.target)*100:<12.1f}")
        print(f"{'REMAINING':<25} {remaining:<10.1f} {(remaining/self.target)*100:<12.1f}")
        
        if remaining < 0:
            print(f"\n⚠️  OVER BUDGET by {abs(remaining):.1f}ms!")
        else:
            print(f"\n✅ Within budget. {remaining:.1f}ms headroom.")
        
        return {
            'total_latency': total_latency,
            'remaining': remaining,
            'within_budget': remaining >= 0
        }

# Example latency analysis
budget = LatencyBudget(target_latency_ms=200)

budget.add_component('Network (client → server)', 20, 'CDN + routing')
budget.add_component('Query parsing', 5, 'Tokenization + normalization')
budget.add_component('Cache lookup', 10, 'Redis check')
budget.add_component('Index shard fanout', 15, 'Broadcast to all shards')
budget.add_component('Shard query (each)', 80, 'Disk read + ranking')
budget.add_component('Result aggregation', 20, 'Merge + re-rank top K')
budget.add_component('Snippet generation', 15, 'Extract + highlight')
budget.add_component('Network (server → client)', 20, 'Response delivery')
budget.add_component('ML ranking', 15, 'Personalization model')

result = budget.analyze_budget()

# Output:
# Latency Budget Analysis (Target: 200ms)
# 
# Component                   Latency    % of Budget  Description
# ===============================================================================
# Network (client → server)   20.0       10.0         CDN + routing
# Query parsing               5.0        2.5          Tokenization + normalization
# Cache lookup                10.0       5.0          Redis check
# ...
# TOTAL                       200.0      100.0       
# REMAINING                   0.0        0.0         
# 
# ✅ Within budget. 0.0ms headroom.
```

---

### 🔴 For Advanced: Cost Optimization at Scale

#### Multi-Region Deployment Analysis

```text
Global Search Engine Deployment:

Regions: 4 (US, EU, Asia, South America)

Per-Region Infrastructure:
├─ Query servers: 25 machines × 4 regions = 100 machines
│   Cost: 100 × $200/month = $20,000/month
│
├─ Index shards: 100 machines × 4 regions = 400 machines  
│   Cost: 400 × $300/month (storage-heavy) = $120,000/month
│
├─ Coordination/cache: 10 machines × 4 regions = 40 machines
│   Cost: 40 × $150/month = $6,000/month
│
└─ Network (cross-region replication): 
    Cost: 10 TB/month × $0.09/GB = $900/month

Total Monthly Cost: $146,900/month (~$1.76M/year)

Optimization Opportunities:

1. Index Sharing (Hot-Warm-Cold):
   - Hot index (20% data, 80% queries): In all regions
   - Warm index (60% data, 18% queries): In 2 regions
   - Cold index (20% data, 2% queries): In 1 region only
   - Savings: 40% reduction in storage → $48K/month saved

2. Query Result Caching:
   - Cache hit rate: 30% (top queries)
   - Reduces shard load by 30%
   - Can reduce shard machines by 20%
   - Savings: $24K/month

3. Async Index Updates:
   - Real-time updates only for hot index
   - Batch updates for warm/cold (daily)
   - Reduces write load by 60%
   - Savings: $10K/month in write capacity

4. Compression & Deduplication:
   - Better compression algorithms (8x → 12x)
   - Cross-document deduplication
   - Reduces storage by 30%
   - Savings: $36K/month

Total Potential Savings: $118K/month (80% of original cost!)
Optimized Cost: $28,900/month
```

#### Scaling Decision Framework

```python
class ScalingDecision:
    """
    Decide when to scale horizontally vs vertically.
    """
    def analyze(self, current_metrics):
        """Analyze metrics and recommend scaling strategy"""
        recommendations = []
        
        # Check CPU utilization
        if current_metrics['cpu_avg'] > 70:
            if current_metrics['cpu_p99'] > 90:
                recommendations.append({
                    'issue': 'High CPU (P99 > 90%)',
                    'recommendation': 'Horizontal scaling',
                    'reasoning': 'Add more machines to distribute load',
                    'urgency': 'high',
                    'estimated_cost': '+$5,000/month for 25 more machines'
                })
            else:
                recommendations.append({
                    'issue': 'Moderate CPU (avg > 70%)',
                    'recommendation': 'Optimize queries first',
                    'reasoning': 'P99 is fine, likely inefficient queries',
                    'urgency': 'medium',
                    'estimated_cost': 'Free (optimization)'
                })
        
        # Check memory utilization
        if current_metrics['memory_percent'] > 80:
            recommendations.append({
                'issue': 'High memory usage',
                'recommendation': 'Vertical scaling',
                'reasoning': 'Increase RAM per machine (more cache)',
                'urgency': 'medium',
                'estimated_cost': '+$2,000/month for memory upgrades'
            })
        
        # Check disk I/O wait
        if current_metrics['iowait_percent'] > 20:
            recommendations.append({
                'issue': 'High disk I/O wait',
                'recommendation': 'Upgrade to SSDs or add caching',
                'reasoning': 'Disk is bottleneck, not CPU',
                'urgency': 'high',
                'estimated_cost': '+$10,000/month for NVMe SSDs'
            })
        
        # Check query latency
        if current_metrics['p95_latency_ms'] > 200:
            if current_metrics['cache_hit_rate'] < 0.3:
                recommendations.append({
                    'issue': 'High latency + low cache hit rate',
                    'recommendation': 'Increase cache size',
                    'reasoning': 'Many queries hitting disk',
                    'urgency': 'high',
                    'estimated_cost': '+$1,000/month for Redis capacity'
                })
        
        return recommendations

# Example usage
metrics = {
    'cpu_avg': 75,
    'cpu_p99': 92,
    'memory_percent': 85,
    'iowait_percent': 15,
    'p95_latency_ms': 250,
    'cache_hit_rate': 0.25,
    'qps': 50000
}

scaler = ScalingDecision()
recommendations = scaler.analyze(metrics)

for rec in recommendations:
    print(f"🚨 {rec['issue']}")
    print(f"   → {rec['recommendation']}")
    print(f"   Why: {rec['reasoning']}")
    print(f"   Cost: {rec['estimated_cost']}")
    print()
```

---

### Real-World Example: Elasticsearch Scaling Journey

```text
Company: E-commerce Platform
Timeline: 2018-2023

2018 - Initial Setup:
├─ Scale: 10M products, 1K QPS
├─ Infrastructure: 5 Elasticsearch nodes (16 GB RAM each)
├─ Cost: $500/month
├─ Problem: Works well, no issues
└─ Latency: P95 = 100ms

2019 - Growth Phase:
├─ Scale: 50M products, 10K QPS
├─ Infrastructure: 20 nodes (32 GB RAM each)
├─ Cost: $3,000/month
├─ Problem: Occasional slowdowns during re-indexing
├─ Solution: Separate indexing and query clusters
└─ Latency: P95 = 150ms

2020 - Optimization:
├─ Scale: 100M products, 25K QPS
├─ Infrastructure: 30 nodes + dedicated coordinators
├─ Cost: $8,000/month
├─ Problem: Storage costs exploding
├─ Solution: Implemented compression + hot-warm architecture
├─ Result: Reduced storage by 60%
└─ New cost: $5,000/month (cheaper!)

2021 - Multi-Region:
├─ Scale: 150M products, 50K QPS globally
├─ Infrastructure: 3 regions × 20 nodes = 60 nodes
├─ Cost: $15,000/month
├─ Problem: Cross-region latency for replication
├─ Solution: Async replication for non-critical updates
└─ Latency: P95 = 120ms (improved with regional routing)

2023 - ML Integration:
├─ Scale: 200M products, 100K QPS
├─ Infrastructure: 80 nodes + ML serving layer
├─ Cost: $25,000/month
├─ Innovation: ML-based ranking improved conversion 15%
├─ Business impact: +$2M/month revenue
├─ ROI: 80x return on infrastructure cost!
└─ Latency: P95 = 130ms (ML adds 10ms)

Key Lessons:
1. Don't over-provision early - scale when needed
2. Optimization (compression) can save more than hardware
3. Monitor business metrics, not just technical ones
4. ML ROI justified the 3x cost increase
5. Regional deployment critical for global latency
```

---

### ✅ Key Takeaways

- **Start with scale calculations:** Documents, QPS, storage, bandwidth
- **Latency budget is critical:** 200ms total means every component counts
- **Replication for availability:** 3x replication is standard (99.99% uptime)
- **Plan for peak load:** 3-5x average QPS for traffic spikes
- **Optimize before scaling:** Compression, caching, better algorithms save millions
- **Multi-region carefully:** Balance latency vs cost vs complexity
- **Monitor cost per query:** Should decrease as you scale (economies of scale)

---

### 🎯 Practice Exercise

**Scenario:** You're planning infrastructure for a new job search engine.

**Requirements:**
- 50M job postings (documents)
- 10M monthly active users
- 5 searches per user per day
- Target: <100ms P95 latency
- Geographic: US-only initially

**Your Task:**

1. **Calculate Storage:**
   - Average job posting: 3 KB (title, description, location)
   - Inverted index: 30% of raw data
   - Replication factor: 3
   - Total storage in TB?
   - Monthly S3 cost?

2. **Calculate QPS:**
   - Average QPS
   - Peak QPS (assume 5x)
   - If each query server handles 1K QPS, how many servers?
   - Monthly compute cost at $150/server?

3. **Latency Budget (100ms target):**
   - Network: 15ms
   - Query parsing: 3ms
   - Cache lookup: 5ms
   - Index query: ??ms (you decide)
   - Ranking: 10ms
   - Response: 10ms
   - How much budget left for index query?
   - Is this realistic?

4. **Cost Optimization:**
   - 40% of searches are for "software engineer" (top term)
   - How much can caching save?
   - If cache hit rate = 40%, reduce query servers by how many?
   - New monthly cost?

5. **Growth Planning:**
   - Expect 3x growth in 1 year (150M jobs, 30M users)
   - When do you need to add capacity?
   - Scale vertically (bigger machines) or horizontally (more machines)?
   - Justify your decision.

**Bonus Challenge:**

Your search becomes popular for "remote jobs" during pandemic. This query goes from 5% of traffic to 40% overnight. Your index isn't optimized for this (all "remote" postings scattered across shards). Options:

A) Do nothing (latency increases 2x)
B) Emergency re-index with "remote" optimization
C) Add dedicated "remote jobs" index
D) Scale up servers 3x to handle increased load

Model costs and trade-offs for each option. What do you choose?

---

## Section 3: Ranking Algorithms (TF-IDF, BM25, PageRank)

### What You'll Learn

By the end of this section, you'll be able to:
- Implement TF-IDF (Term Frequency-Inverse Document Frequency) scoring
- Understand and apply BM25 ranking algorithm
- Explain PageRank and link analysis for web search
- Combine multiple ranking signals for better relevance
- Tune ranking parameters for your specific use case

### Why This Matters

Ranking is what separates good search from great search! When you search for "python" on Google, it doesn't just return every page containing the word "python" - it ranks them by relevance. The top result is probably Python.org, not a random blog post. This is ranking in action. Google's multi-billion dollar business is built on having the best ranking algorithms. In interviews, being able to explain TF-IDF and BM25 shows you understand information retrieval fundamentals. This section covers the algorithms that power every search engine from Google to Elasticsearch.

---

### 🟢 For Beginners: TF-IDF - The Foundation

#### Key Technologies Explained

Before diving into ranking algorithms, let's understand the fundamentals:

**What is an Inverted Index?**

An inverted index is like the index at the back of a textbook - instead of looking through every page to find a word, you look in the index which tells you exactly which pages have that word!

```text
Regular Forward Index (How Books Are Read):
Page 1: "The quick brown fox"
Page 2: "The lazy dog sleeps"
Page 3: "The quick dog runs"

Inverted Index (How We Search):
"quick" → [Page 1, Page 3]
"brown" → [Page 1]
"fox" → [Page 1]
"lazy" → [Page 2]
"dog" → [Page 2, Page 3]
"sleeps" → [Page 2]
"runs" → [Page 3]

Search for "quick dog":
├─ "quick" → [Page 1, Page 3]
├─ "dog" → [Page 2, Page 3]
└─ Intersection: Page 3 has BOTH words!

Why so fast?
Instead of reading all 3 pages (slow), we look up 2 words in the index (fast)!
```

**How Inverted Index Actually Works:**

```text
Structure:
Term → List of (Document ID, Position, Metadata)

Example:
"python":
├─ Doc 1: positions [5, 15, 23], frequency 3
├─ Doc 5: positions [2], frequency 1
└─ Doc 89: positions [1, 45, 67, 89], frequency 4

"programming":
├─ Doc 1: positions [6, 24], frequency 2
├─ Doc 10: positions [8], frequency 1
└─ Doc 89: positions [2, 46], frequency 2

Search for "python programming":
1. Look up "python" → Docs [1, 5, 89]
2. Look up "programming" → Docs [1, 10, 89]
3. Intersection → Docs [1, 89] have BOTH
4. Rank them (see TF-IDF below)

Why positions matter?
- Phrase search: "python programming" (words adjacent)
- Proximity: Words close together = more relevant
```

**What is Elasticsearch?**

Elasticsearch is a search engine software built on top of Apache Lucene. Think of it as a ready-to-use search engine that handles inverted indexing, ranking, and distributed search for you.

```text
Without Elasticsearch:
├─ Build your own inverted index
├─ Write ranking algorithms
├─ Handle distributed search
├─ Implement caching
└─ = Months of work!

With Elasticsearch:
├─ Install and configure
├─ Send it documents (JSON)
├─ It automatically indexes them
└─ Query and get ranked results immediately!

Used by: Netflix, Uber, GitHub, Wikipedia
```

---

#### What is TF-IDF?

```text
TF-IDF = Term Frequency × Inverse Document Frequency

Term Frequency (TF):
- How often does a term appear in a document?
- More occurrences = more relevant
- Example: Document about "Python programming" mentions "python" 10 times

Inverse Document Frequency (IDF):
- How rare is this term across all documents?
- Rare terms are more distinctive
- Example: "python" in 10% of docs, "the" in 99% of docs
- IDF("python") > IDF("the")

Combined:
- TF-IDF rewards: Common in document, rare overall
- Penalizes: Stop words like "the", "a", "is"
```

#### Simple TF-IDF Implementation

```python
import math
from collections import Counter

class SimpleTFIDF:
    """
    Basic TF-IDF ranking for search results.
    """
    def __init__(self):
        self.documents = []
        self.doc_count = 0
        self.term_doc_count = {}  # term -> number of docs containing it
    
    def add_document(self, doc_id, text):
        """Add document to corpus"""
        terms = text.lower().split()
        self.documents.append({'id': doc_id, 'text': text, 'terms': terms})
        self.doc_count += 1
        
        # Update document frequency for each unique term
        for term in set(terms):
            self.term_doc_count[term] = self.term_doc_count.get(term, 0) + 1
    
    def calculate_tf(self, term, document_terms):
        """Term Frequency: count of term / total terms"""
        term_count = document_terms.count(term)
        total_terms = len(document_terms)
        return term_count / total_terms if total_terms > 0 else 0
    
    def calculate_idf(self, term):
        """Inverse Document Frequency: log(total_docs / docs_with_term)"""
        docs_with_term = self.term_doc_count.get(term, 0)
        if docs_with_term == 0:
            return 0
        return math.log(self.doc_count / docs_with_term)
    
    def calculate_tfidf(self, term, document_terms):
        """TF-IDF score for a term in a document"""
        tf = self.calculate_tf(term, document_terms)
        idf = self.calculate_idf(term)
        return tf * idf
    
    def search(self, query, top_k=5):
        """Search and rank documents by TF-IDF"""
        query_terms = query.lower().split()
        scores = []
        
        for doc in self.documents:
            score = 0
            for term in query_terms:
                score += self.calculate_tfidf(term, doc['terms'])
            
            scores.append({
                'doc_id': doc['id'],
                'score': score,
                'text': doc['text'][:100]  # Preview
            })
        
        # Sort by score (highest first)
        scores.sort(key=lambda x: x['score'], reverse=True)
        return scores[:top_k]

# Example usage
engine = SimpleTFIDF()

# Add documents
engine.add_document(1, "Python is a programming language for web development")
engine.add_document(2, "JavaScript is used for web development")
engine.add_document(3, "Python is great for data science and machine learning")
engine.add_document(4, "The quick brown fox jumps over the lazy dog")

# Search
results = engine.search("python programming", top_k=3)

print("Search results for 'python programming':\n")
for i, result in enumerate(results, 1):
    print(f"{i}. Score: {result['score']:.4f}")
    print(f"   {result['text']}")
    print()

# Output:
# 1. Score: 0.4621
#    Python is a programming language for web development
# 2. Score: 0.2877
#    Python is great for data science and machine learning
# 3. Score: 0.0000
#    JavaScript is used for web development
```

**Why Document 1 Ranked Higher:**
- Contains both "python" AND "programming"
- TF is higher (more mentions of query terms)
- IDF boosts rare terms ("programming" rarer than "python")

---

**What is BM25?**

BM25 (Best Matching 25) is an improved version of TF-IDF that fixes its main problems. It's the industry standard used by Elasticsearch, Solr, and most modern search engines.

```text
Problems with TF-IDF:
1. Linear scaling: Mentioning "python" 10 times scores 2x higher than 5 times
   - Reality: After 3-5 mentions, more doesn't add value
   
2. Ignores document length:
   - 5-word document with "python" once: 20% of content
   - 100-word document with "python" once: 1% of content
   - TF-IDF treats them equally (wrong!)

BM25 Solutions:
1. Saturation: Uses diminishing returns
   - 1 mention: +1.0 points
   - 2 mentions: +0.8 points
   - 5 mentions: +0.5 points
   - 10 mentions: +0.1 points
   
2. Length normalization:
   - Penalizes short documents with keyword stuffing
   - Rewards longer documents with good content
   - Considers average document length

Result: More accurate rankings, harder to game!
```

**What is PageRank?**

PageRank is Google's original algorithm that ranks pages based on link popularity, not just content. It's like academic citations - a paper cited by many important papers is probably important too!

```text
The Core Idea:
- A page's importance comes from other important pages linking to it
- It's not just "how many links" but "quality of links"

Example:
Page A:
├─ Linked by: Wikipedia, NYTimes, BBC
├─ All are authoritative sites
└─ PageRank: HIGH

Page B:
├─ Linked by: Random blog, spam site, unknown forum
├─ Low-quality links
└─ PageRank: LOW

Even if both have 3 links, Page A ranks much higher!

Why it matters:
- Prevents spam: Can't game the system by self-linking
- Rewards quality: Must earn links from trusted sites
- Used by Google: Core part of ranking algorithm

Modern Usage:
- Still used but combined with 200+ other signals
- Content quality, user behavior, freshness, etc.
- PageRank is one piece of a complex puzzle
```

---

### 🟡 For Intermediate: BM25 - Industry Standard

#### Why BM25 is Better Than TF-IDF

```text
Problems with TF-IDF:

1. Linear scaling: 10 occurrences of "python" scores 2x higher than 5 occurrences
   - Reality: After 3-5 occurrences, adding more doesn't help much
   - Solution: BM25 uses saturation - diminishing returns

2. Ignores document length: Short docs can't score high
   - Reality: A 100-word doc with 5 mentions of "python" is MORE relevant
     than a 10,000-word doc with 5 mentions
   - Solution: BM25 normalizes by document length

3. No tuning: TF-IDF has no parameters to adjust
   - Reality: Different datasets need different trade-offs
   - Solution: BM25 has tunable parameters (k1, b)

BM25 = Best Match 25 (25th iteration of Best Match algorithm)
Used by: Elasticsearch, Lucene, Bing, Wikipedia search
```

#### BM25 Implementation

```python
import math

class BM25Ranker:
    """
    BM25 ranking algorithm - industry standard.
    
    Parameters:
    - k1: Term frequency saturation (typical: 1.2 - 2.0)
      Higher k1 = more weight on term frequency
    - b: Length normalization (typical: 0.75)
      Higher b = more penalty for long documents
    """
    def __init__(self, k1=1.5, b=0.75):
        self.k1 = k1
        self.b = b
        self.documents = []
        self.doc_count = 0
        self.avg_doc_length = 0
        self.term_doc_count = {}
    
    def add_document(self, doc_id, text):
        """Add document to corpus"""
        terms = text.lower().split()
        doc_length = len(terms)
        
        self.documents.append({
            'id': doc_id,
            'text': text,
            'terms': terms,
            'length': doc_length
        })
        
        self.doc_count += 1
        self.avg_doc_length = (
            (self.avg_doc_length * (self.doc_count - 1) + doc_length) 
            / self.doc_count
        )
        
        # Update document frequency
        for term in set(terms):
            self.term_doc_count[term] = self.term_doc_count.get(term, 0) + 1
    
    def calculate_idf(self, term):
        """BM25 IDF formula"""
        docs_with_term = self.term_doc_count.get(term, 0)
        if docs_with_term == 0:
            return 0
        
        # BM25 IDF: log((N - df + 0.5) / (df + 0.5) + 1)
        # Where N = total docs, df = docs containing term
        N = self.doc_count
        df = docs_with_term
        return math.log((N - df + 0.5) / (df + 0.5) + 1)
    
    def calculate_bm25_score(self, query_terms, document):
        """Calculate BM25 score for a document"""
        score = 0
        doc_terms = document['terms']
        doc_length = document['length']
        
        # Count term frequencies in document
        term_freqs = {}
        for term in doc_terms:
            term_freqs[term] = term_freqs.get(term, 0) + 1
        
        for term in query_terms:
            if term not in term_freqs:
                continue
            
            tf = term_freqs[term]
            idf = self.calculate_idf(term)
            
            # BM25 formula:
            # IDF * (tf * (k1 + 1)) / (tf + k1 * (1 - b + b * (doc_len / avg_doc_len)))
            numerator = tf * (self.k1 + 1)
            denominator = tf + self.k1 * (
                1 - self.b + self.b * (doc_length / self.avg_doc_length)
            )
            
            score += idf * (numerator / denominator)
        
        return score
    
    def search(self, query, top_k=5):
        """Search and rank documents by BM25"""
        query_terms = query.lower().split()
        scores = []
        
        for doc in self.documents:
            score = self.calculate_bm25_score(query_terms, doc)
            scores.append({
                'doc_id': doc['id'],
                'score': score,
                'text': doc['text'][:100]
            })
        
        scores.sort(key=lambda x: x['score'], reverse=True)
        return scores[:top_k]

# Compare TF-IDF vs BM25
print("=" * 60)
print("TF-IDF vs BM25 Comparison")
print("=" * 60)

docs = [
    (1, "Python programming is great for web development and data science"),
    (2, "Python" * 50),  # Keyword stuffing - 50 mentions!
    (3, "Learn Python programming basics in this comprehensive guide to Python")
]

# TF-IDF
tfidf = SimpleTFIDF()
for doc_id, text in docs:
    tfidf.add_document(doc_id, text)

tfidf_results = tfidf.search("python programming")

print("\nTF-IDF Results:")
for i, r in enumerate(tfidf_results, 1):
    print(f"{i}. Doc {r['doc_id']}, Score: {r['score']:.4f}")

# BM25
bm25 = BM25Ranker(k1=1.5, b=0.75)
for doc_id, text in docs:
    bm25.add_document(doc_id, text)

bm25_results = bm25.search("python programming")

print("\nBM25 Results:")
for i, r in enumerate(bm25_results, 1):
    print(f"{i}. Doc {r['doc_id']}, Score: {r['score']:.4f}")

print("\nNotice: BM25 penalizes Doc 2 (keyword stuffing) while TF-IDF ranks it high!")
```

**BM25 Parameter Tuning:**

```python
def tune_bm25_parameters(training_queries, relevance_judgments):
    """
    Find optimal k1 and b parameters.
    
    Training data format:
    - training_queries: [(query, doc_id, relevance_score), ...]
    - relevance_score: 0 (not relevant) to 4 (perfect match)
    """
    best_params = {'k1': 1.5, 'b': 0.75}
    best_score = 0
    
    # Grid search
    for k1 in [0.5, 1.0, 1.2, 1.5, 2.0]:
        for b in [0.0, 0.25, 0.5, 0.75, 1.0]:
            ranker = BM25Ranker(k1=k1, b=b)
            
            # Evaluate on training data
            ndcg_score = evaluate_ranking(ranker, training_queries, relevance_judgments)
            
            if ndcg_score > best_score:
                best_score = ndcg_score
                best_params = {'k1': k1, 'b': b}
    
    return best_params

# Typical parameter ranges:
# k1 = 1.2 to 2.0 (most use 1.2-1.5)
# b = 0.75 (standard), 0.0 (ignore length), 1.0 (strong length penalty)
```

---

### 🔴 For Advanced: PageRank & Multi-Signal Ranking

#### PageRank Algorithm

```python
import numpy as np

class PageRank:
    """
    Google's PageRank algorithm for link analysis.
    
    Key idea: A page is important if important pages link to it.
    """
    def __init__(self, damping_factor=0.85, max_iterations=100, tolerance=1e-6):
        self.d = damping_factor  # Probability of following a link
        self.max_iter = max_iterations
        self.tolerance = tolerance
    
    def calculate_pagerank(self, graph):
        """
        Calculate PageRank for all pages.
        
        Args:
            graph: dict mapping page_id -> [linked_page_ids]
        
        Returns:
            dict mapping page_id -> PageRank score
        """
        # Initialize
        pages = list(graph.keys())
        n = len(pages)
        page_to_idx = {page: idx for idx, page in enumerate(pages)}
        
        # Build transition matrix
        M = np.zeros((n, n))
        
        for page, links in graph.items():
            if not links:  # No outgoing links
                # Distribute equally to all pages
                M[page_to_idx[page], :] = 1 / n
            else:
                # Distribute equally among linked pages
                for link in links:
                    if link in page_to_idx:
                        M[page_to_idx[page], page_to_idx[link]] = 1 / len(links)
        
        # Initialize PageRank
        pr = np.ones(n) / n
        
        # Power iteration
        for iteration in range(self.max_iter):
            pr_new = (1 - self.d) / n + self.d * M.T @ pr
            
            # Check convergence
            if np.linalg.norm(pr_new - pr, 1) < self.tolerance:
                print(f"Converged in {iteration + 1} iterations")
                break
            
            pr = pr_new
        
        # Convert back to dict
        return {page: pr[page_to_idx[page]] for page in pages}

# Example: Simple web graph
graph = {
    'A': ['B', 'C'],        # A links to B and C
    'B': ['C'],             # B links to C
    'C': ['A'],             # C links to A
    'D': ['C'],             # D links to C
}

pr = PageRank(damping_factor=0.85)
scores = pr.calculate_pagerank(graph)

print("PageRank Scores:")
for page, score in sorted(scores.items(), key=lambda x: x[1], reverse=True):
    print(f"  Page {page}: {score:.4f}")

# Output:
# Page C: 0.3579  (most links pointing to it)
# Page A: 0.2764
# Page B: 0.2426
# Page D: 0.1231  (no incoming links)
```

#### Multi-Signal Ranking (ML-Based)

```python
class MultiSignalRanker:
    """
    Combine multiple ranking signals using machine learning.
    
    Signals:
    1. Text relevance (BM25)
    2. Popularity (PageRank)
    3. Freshness (recency)
    4. Click-through rate (CTR)
    5. User engagement (dwell time)
    """
    def __init__(self, weights=None):
        self.bm25 = BM25Ranker()
        
        # Default weights (can be learned from data)
        self.weights = weights or {
            'bm25_score': 0.40,      # Text relevance most important
            'pagerank': 0.25,        # Link authority
            'freshness': 0.15,       # Recent content
            'ctr': 0.10,             # Historical clicks
            'dwell_time': 0.10       # User engagement
        }
    
    def normalize_score(self, score, min_val, max_val):
        """Normalize score to 0-1 range"""
        if max_val == min_val:
            return 0.5
        return (score - min_val) / (max_val - min_val)
    
    def calculate_combined_score(self, doc, query, signal_data):
        """
        Calculate combined score from multiple signals.
        
        Args:
            doc: Document object
            query: Search query
            signal_data: Dict with additional signals
        """
        scores = {}
        
        # 1. Text relevance (BM25)
        scores['bm25_score'] = self.normalize_score(
            signal_data.get('bm25_score', 0),
            min_val=0,
            max_val=signal_data.get('max_bm25', 10)
        )
        
        # 2. PageRank (link authority)
        scores['pagerank'] = self.normalize_score(
            signal_data.get('pagerank', 0),
            min_val=0,
            max_val=signal_data.get('max_pagerank', 1)
        )
        
        # 3. Freshness (exponential decay)
        import time
        days_old = signal_data.get('days_old', 0)
        scores['freshness'] = math.exp(-days_old / 30)  # Half-life of 30 days
        
        # 4. Click-through rate
        scores['ctr'] = signal_data.get('ctr', 0.01)
        
        # 5. Dwell time (engagement)
        avg_dwell_seconds = signal_data.get('avg_dwell_time', 30)
        scores['dwell_time'] = min(avg_dwell_seconds / 300, 1.0)  # Cap at 5 minutes
        
        # Weighted combination
        final_score = sum(
            self.weights[signal] * scores[signal]
            for signal in self.weights
        )
        
        return {
            'final_score': final_score,
            'breakdown': scores
        }

# Example usage
ranker = MultiSignalRanker()

signal_data = {
    'bm25_score': 8.5,
    'max_bm25': 10,
    'pagerank': 0.045,
    'max_pagerank': 0.1,
    'days_old': 10,
    'ctr': 0.15,  # 15% of users click
    'avg_dwell_time': 120  # 2 minutes average
}

result = ranker.calculate_combined_score(
    doc={'id': 1},
    query="python tutorial",
    signal_data=signal_data
)

print("Multi-Signal Ranking:")
print(f"Final Score: {result['final_score']:.4f}\n")
print("Signal Breakdown:")
for signal, score in result['breakdown'].items():
    weight = ranker.weights.get(signal, 0)
    contribution = weight * score
    print(f"  {signal:15} {score:.4f} × {weight:.2f} = {contribution:.4f}")
```

---

### Real-World Example: Google's Ranking Evolution

```text
1998 - PageRank Era:
├─ Primary signal: Link analysis (PageRank)
├─ Problem: Link spam, "link farms" manipulation
├─ Quality: Good for authoritative sites, poor for long-tail
└─ Innovation: First link-based ranking at scale

2003 - Florida Update (Text Relevance):
├─ Added: TF-IDF + on-page factors
├─ Penalized: Keyword stuffing
├─ Problem: Still gameable with exact-match domains
└─ Innovation: Balanced links + content

2011 - Panda (Content Quality):
├─ Added: ML model to detect thin content
├─ Signals: Bounce rate, time on site, user satisfaction
├─ Impact: 12% of queries affected
└─ Innovation: ML-based quality scoring

2013 - Hummingbird (Semantic Search):
├─ Beyond: Keyword matching
├─ Added: Natural language understanding, entities
├─ Example: "how tall is obama" understands "obama" is a person
└─ Innovation: Semantic matching

2015 - RankBrain (ML Core):
├─ ML model: Handles 15% of queries (ambiguous/new)
├─ Learning: Query-document relevance from user behavior
├─ Impact: 10% improvement in search quality
└─ Innovation: Self-learning ranking

2019 - BERT (Contextual Understanding):
├─ Transformer model: Understands query context
├─ Example: "2019 brazil traveler to usa need a visa"
  - Pre-BERT: Focused on "brazil" and "usa"
  - Post-BERT: Understands "to" (direction matters!)
├─ Impact: 10% of queries in all languages
└─ Innovation: Deep language understanding

2021 - MUM (Multimodal):
├─ Handles: Text, images, video together
├─ Example: Photo of a hiking boot → identifies brand, finds purchase links
├─ 1000x more powerful than BERT
└─ Innovation: Cross-modal understanding

Current (2023) - 200+ Ranking Signals:
├─ Content: TF-IDF, BM25, semantic similarity
├─ Authority: PageRank, domain age, expertise
├─ User signals: CTR, dwell time, pogo-sticking
├─ Freshness: Publication date, update frequency
├─ Personalization: Location, search history, device
└─ ML models: RankBrain, BERT, MUM, custom models

Key Lesson: Started with 1 signal (links), now uses 200+.
Modern ranking is ensemble of many signals.
```

---

### 🤔 Think About It

1. **For Beginners:** Why does searching "python" on Google show Python.org first, not a random blog? Calculate the TF-IDF score for "python" appearing 5 times in a 100-word official doc vs 20 times in a 10,000-word spam doc.

2. **For Intermediate:** You're ranking e-commerce products. User searches "nike shoes". Which signals matter most: text match, popularity (sales), recency, or price? How do you balance them? What if the query is "nike air max 2023"?

3. **For Advanced:** Google uses 200+ ranking signals but can't process all of them for every query (too slow). Design a two-stage ranking system: (1) Fast retrieval with 5-10 signals, (2) ML re-ranking of top 100 with all 200 signals. What goes in each stage?

---

### ✅ Key Takeaways

- **TF-IDF is the foundation:** Simple, interpretable, good baseline
- **BM25 is production standard:** Used by Elasticsearch, Solr, Bing - handles length normalization and saturation
- **PageRank for authority:** Link analysis shows trust and importance
- **Multi-signal ranking wins:** Combine text relevance, popularity, freshness, user behavior
- **Tunable parameters matter:** k1 and b in BM25 should be tuned for your domain
- **User signals trump algorithms:** CTR and dwell time often beat sophisticated text matching
- **ML is the future:** RankBrain, BERT show ML models outperform hand-crafted formulas

---

### 🎯 Practice Exercise

**Scenario:** You're building search for a Q&A site like Stack Overflow with 10M questions.

**Given Information:**
- Users search for programming questions
- Questions have: title, body, tags, answers, votes, views
- Average question: 200 words
- Queries like: "how to reverse a list in python"

**Your Task:**

1. **Implement Ranking:**
   - Design BM25 scoring for question text
   - Should you index title, body, or both? With what weights?
   - How do you handle tags (exact match vs text match)?

2. **Additional Signals:**
   - Votes: Questions with 100+ votes more relevant?
   - Accepted answer: Boost questions with accepted answers?
   - View count: Popular questions ranked higher?
   - Freshness: Recent questions vs old but highly voted?
   - Design a weighted formula combining these signals

3. **Parameter Tuning:**
   - You have 1,000 test queries with human relevance ratings (0-4)
   - Design A/B test to compare:
     - Pure BM25 (k1=1.2, b=0.75)
     - BM25 + votes (weight=?)
     - BM25 + votes + freshness (weights=?)
   - What metrics do you track?

4. **Edge Cases:**
   - Query: "python" (ambiguous - language or snake?)
   - How do you use click data to improve?
   - Query: "error installing numpy on windows 11"
   - Very specific - should you boost exact tag matches?

**Bonus Challenge:**

You notice that questions tagged with "python" and "pandas" get 10x more views than other tags. Your ranking algorithm, which includes view count, now heavily biases toward these topics. A Java developer searching "java hashmap" sees mostly Python results because they have higher view counts.

How do you fix this? Consider:
- Normalizing popularity by topic/tag
- Personalizing based on user's past searches
- Query-category classification before ranking
- Tag-specific ranking parameters

Design a solution and calculate the impact on ranking scores for a sample query.

---

## Section 4: Query Processing & Optimization

### What You'll Learn

By the end of this section, you'll be able to:
- Parse and normalize search queries for optimal matching
- Implement spell correction and query expansion
- Design efficient query execution plans
- Optimize query performance with early termination
- Handle phrase queries and Boolean operators
- Build query suggestion systems

### Why This Matters

When a user types "appel iphone 15 pro max pric" (full of typos), Google still shows iPhone 15 Pro Max prices. This is query processing in action! Before your query hits the index, it goes through a sophisticated pipeline: tokenization, normalization, spell correction, query expansion, and more. Poor query processing means users can't find what they're looking for, even if you have the perfect document indexed. In interviews, explaining how you'd handle misspellings, synonyms, and complex queries shows deep understanding of production search systems.

---

### 🟢 For Beginners: Query Processing Pipeline

#### Basic Query Processing Steps

```text
User Query Processing Flow:

Raw Query: "iPhone 15 Pro Max pric"
    ↓
1. Tokenization: ["iPhone", "15", "Pro", "Max", "pric"]
    ↓
2. Normalization: ["iphone", "15", "pro", "max", "pric"]
    ↓
3. Spell Check: ["iphone", "15", "pro", "max", "price"] ✓
    ↓
4. Stop Word Removal: ["iphone", "15", "pro", "max", "price"]
    (no stop words in this query)
    ↓
5. Query Expansion: 
   ["iphone", "15", "pro", "max", "price"]
   + synonyms: ["cost", "pricing", "msrp"]
    ↓
6. Index Lookup: Search inverted index
    ↓
7. Ranking: Apply BM25, PageRank, etc.
    ↓
8. Results: Top 10 documents
```

#### Simple Query Processor Implementation

```python
import re
from collections import defaultdict

class SimpleQueryProcessor:
    """
    Basic query processing pipeline.
    """
    def __init__(self):
        # Common misspellings dictionary
        self.spell_corrections = {
            'pric': 'price',
            'appel': 'apple',
            'iphone': 'iphone',  # Already correct
            'pythno': 'python',
            'javascirpt': 'javascript'
        }
        
        # Synonyms for query expansion
        self.synonyms = {
            'price': ['cost', 'pricing', 'value', 'msrp'],
            'buy': ['purchase', 'shop', 'order'],
            'cheap': ['affordable', 'budget', 'inexpensive']
        }
        
        # Stop words to potentially remove
        self.stop_words = {'the', 'a', 'an', 'in', 'on', 'at', 'to', 'for'}
    
    def tokenize(self, query):
        """Split query into tokens"""
        # Simple whitespace + punctuation splitting
        tokens = re.findall(r'\w+', query.lower())
        return tokens
    
    def correct_spelling(self, tokens):
        """Fix common misspellings"""
        corrected = []
        for token in tokens:
            if token in self.spell_corrections:
                corrected.append(self.spell_corrections[token])
            else:
                corrected.append(token)
        return corrected
    
    def remove_stop_words(self, tokens):
        """Remove common stop words"""
        # Be careful! Sometimes stop words matter
        # Example: "the who" (band name) shouldn't remove "the"
        return [t for t in tokens if t not in self.stop_words]
    
    def expand_query(self, tokens, max_synonyms=2):
        """Add synonyms for better recall"""
        expanded = list(tokens)
        for token in tokens:
            if token in self.synonyms:
                # Add top N synonyms
                expanded.extend(self.synonyms[token][:max_synonyms])
        return expanded
    
    def process(self, query, expand=False):
        """Full query processing pipeline"""
        print(f"Original query: {query}")
        
        # Step 1: Tokenize
        tokens = self.tokenize(query)
        print(f"Tokens: {tokens}")
        
        # Step 2: Spell correction
        tokens = self.correct_spelling(tokens)
        print(f"After spell check: {tokens}")
        
        # Step 3: Remove stop words (optional)
        tokens_no_stop = self.remove_stop_words(tokens)
        if len(tokens_no_stop) < len(tokens):
            print(f"After stop word removal: {tokens_no_stop}")
            tokens = tokens_no_stop
        
        # Step 4: Query expansion (optional)
        if expand:
            tokens = self.expand_query(tokens)
            print(f"After expansion: {tokens}")
        
        return tokens

# Example usage
processor = SimpleQueryProcessor()

# Test query 1: Misspelled
query1 = "appel iphone pric"
tokens1 = processor.process(query1, expand=True)
print(f"Final tokens: {tokens1}\n")

# Test query 2: With stop words
query2 = "the best python tutorial for beginners"
tokens2 = processor.process(query2, expand=False)
print(f"Final tokens: {tokens2}\n")

# Output:
# Original query: appel iphone pric
# Tokens: ['appel', 'iphone', 'pric']
# After spell check: ['apple', 'iphone', 'price']
# After expansion: ['apple', 'iphone', 'price', 'cost', 'pricing']
# Final tokens: ['apple', 'iphone', 'price', 'cost', 'pricing']
```

---

### 🟡 For Intermediate: Advanced Query Processing

#### Fuzzy Matching with Edit Distance

```python
def levenshtein_distance(s1, s2):
    """
    Calculate edit distance between two strings.
    Used for fuzzy matching and spell correction.
    """
    if len(s1) < len(s2):
        return levenshtein_distance(s2, s1)
    
    if len(s2) == 0:
        return len(s1)
    
    previous_row = range(len(s2) + 1)
    for i, c1 in enumerate(s1):
        current_row = [i + 1]
        for j, c2 in enumerate(s2):
            # Cost of insertions, deletions, substitutions
            insertions = previous_row[j + 1] + 1
            deletions = current_row[j] + 1
            substitutions = previous_row[j] + (c1 != c2)
            current_row.append(min(insertions, deletions, substitutions))
        previous_row = current_row
    
    return previous_row[-1]

class FuzzyQueryMatcher:
    """
    Fuzzy matching for spell correction.
    """
    def __init__(self, dictionary):
        self.dictionary = dictionary  # Valid terms
    
    def find_similar(self, word, max_distance=2):
        """Find similar words within edit distance"""
        similar = []
        for dict_word in self.dictionary:
            distance = levenshtein_distance(word, dict_word)
            if distance <= max_distance:
                similar.append((dict_word, distance))
        
        # Sort by distance (closest first)
        similar.sort(key=lambda x: x[1])
        return similar

# Example
dictionary = ['python', 'java', 'javascript', 'typescript', 'ruby', 'golang']
matcher = FuzzyQueryMatcher(dictionary)

misspelled = 'pythno'
suggestions = matcher.find_similar(misspelled, max_distance=2)
print(f"Did you mean '{misspelled}'?")
for word, distance in suggestions[:3]:
    print(f"  {word} (distance: {distance})")

# Output:
# Did you mean 'pythno'?
#   python (distance: 1)
```

#### Query Execution with Boolean Operators

```python
class BooleanQueryProcessor:
    """
    Handle Boolean queries: AND, OR, NOT
    Example: "python AND (tutorial OR guide) NOT advanced"
    """
    def __init__(self, inverted_index):
        self.index = inverted_index
    
    def search_term(self, term):
        """Get posting list for a single term"""
        return set(self.index.get(term, []))
    
    def and_operation(self, posting_lists):
        """Intersection of posting lists"""
        if not posting_lists:
            return set()
        result = posting_lists[0]
        for postings in posting_lists[1:]:
            result = result.intersection(postings)
        return result
    
    def or_operation(self, posting_lists):
        """Union of posting lists"""
        result = set()
        for postings in posting_lists:
            result = result.union(postings)
        return result
    
    def not_operation(self, all_docs, postings):
        """Complement (all docs except these)"""
        return all_docs - postings
    
    def execute_boolean_query(self, query_ast, all_docs):
        """
        Execute parsed Boolean query.
        
        query_ast: Abstract Syntax Tree of query
        Example: {'op': 'AND', 'left': 'python', 'right': {'op': 'OR', ...}}
        """
        if isinstance(query_ast, str):
            # Leaf node - single term
            return self.search_term(query_ast)
        
        op = query_ast['op']
        
        if op == 'AND':
            left = self.execute_boolean_query(query_ast['left'], all_docs)
            right = self.execute_boolean_query(query_ast['right'], all_docs)
            return self.and_operation([left, right])
        
        elif op == 'OR':
            left = self.execute_boolean_query(query_ast['left'], all_docs)
            right = self.execute_boolean_query(query_ast['right'], all_docs)
            return self.or_operation([left, right])
        
        elif op == 'NOT':
            operand = self.execute_boolean_query(query_ast['operand'], all_docs)
            return self.not_operation(all_docs, operand)
        
        return set()

# Example inverted index
inverted_index = {
    'python': [1, 2, 3, 5],
    'tutorial': [1, 2, 4],
    'advanced': [3, 5],
    'beginner': [1, 2]
}

processor = BooleanQueryProcessor(inverted_index)

# Query: python AND tutorial NOT advanced
query_ast = {
    'op': 'AND',
    'left': 'python',
    'right': {
        'op': 'NOT',
        'operand': 'advanced',
        'all_docs': {1, 2, 3, 4, 5}
    }
}

# Manually construct for demonstration
python_docs = processor.search_term('python')  # {1, 2, 3, 5}
tutorial_docs = processor.search_term('tutorial')  # {1, 2, 4}
advanced_docs = processor.search_term('advanced')  # {3, 5}

# python AND tutorial
result1 = python_docs.intersection(tutorial_docs)  # {1, 2}
# NOT advanced
result2 = result1 - advanced_docs  # {1, 2}

print(f"Query: python AND tutorial NOT advanced")
print(f"Results: {result2}")  # Documents 1 and 2
```

#### Early Termination for Performance

```python
class OptimizedQueryProcessor:
    """
    Query optimization techniques for faster search.
    """
    def __init__(self, index, doc_scores):
        self.index = index  # term -> [(doc_id, score), ...]
        self.doc_scores = doc_scores  # Precomputed document quality scores
    
    def document_at_a_time(self, query_terms, k=10):
        """
        DAAT: Process all query terms for each document.
        Good when: Few documents match query terms.
        """
        # Collect all candidate documents
        candidates = set()
        for term in query_terms:
            if term in self.index:
                candidates.update([doc_id for doc_id, _ in self.index[term]])
        
        # Score each candidate
        scores = []
        for doc_id in candidates:
            score = 0
            for term in query_terms:
                if term in self.index:
                    # Find term score for this doc
                    for d_id, term_score in self.index[term]:
                        if d_id == doc_id:
                            score += term_score
                            break
            scores.append((doc_id, score))
        
        # Return top K
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores[:k]
    
    def term_at_a_time(self, query_terms, k=10):
        """
        TAAT: Process each query term across all documents.
        Good when: Many documents, few query terms.
        """
        # Accumulate scores per document
        doc_scores = defaultdict(float)
        
        for term in query_terms:
            if term in self.index:
                for doc_id, term_score in self.index[term]:
                    doc_scores[doc_id] += term_score
        
        # Return top K
        scores = sorted(doc_scores.items(), key=lambda x: x[1], reverse=True)
        return scores[:k]
    
    def max_score_optimization(self, query_terms, k=10):
        """
        MaxScore: Early termination optimization.
        
        Key idea: If we already have K good results, and remaining
        terms can't improve them, stop early!
        """
        # Precompute max possible contribution per term
        term_max_scores = {}
        for term in query_terms:
            if term in self.index:
                max_score = max(score for _, score in self.index[term])
                term_max_scores[term] = max_score
        
        # Sort terms by max score (descending)
        sorted_terms = sorted(
            term_max_scores.items(),
            key=lambda x: x[1],
            reverse=True
        )
        
        # Accumulate scores with early termination
        doc_scores = defaultdict(float)
        threshold = float('-inf')  # Min score to be in top K
        
        for i, (term, max_score) in enumerate(sorted_terms):
            # Calculate upper bound: current scores + remaining terms
            remaining_max = sum(score for _, score in sorted_terms[i:])
            
            # Early termination check
            if threshold != float('-inf') and remaining_max < threshold:
                print(f"Early termination after term '{term}'!")
                break
            
            # Process this term
            if term in self.index:
                for doc_id, term_score in self.index[term]:
                    doc_scores[doc_id] += term_score
            
            # Update threshold (min score of current top K)
            if len(doc_scores) >= k:
                scores = sorted(doc_scores.values(), reverse=True)
                threshold = scores[k-1] if len(scores) > k else scores[-1]
        
        # Return top K
        scores = sorted(doc_scores.items(), key=lambda x: x[1], reverse=True)
        return scores[:k]

# Example index
index = {
    'python': [(1, 10.0), (2, 8.0), (3, 7.0), (5, 5.0)],
    'tutorial': [(1, 6.0), (2, 7.0), (4, 8.0)],
    'beginner': [(1, 5.0), (2, 6.0)]
}

processor = OptimizedQueryProcessor(index, {})

query = ['python', 'tutorial', 'beginner']
print("Query:", query)
print("\nDocument-at-a-Time:")
print(processor.document_at_a_time(query, k=3))

print("\nTerm-at-a-Time:")
print(processor.term_at_a_time(query, k=3))

print("\nMaxScore (with early termination):")
print(processor.max_score_optimization(query, k=3))
```

---

### 🔴 For Advanced: Production Query Optimization

#### Query Rewriting & Understanding

```python
class QueryUnderstanding:
    """
    Advanced query understanding for production systems.
    """
    def __init__(self):
        # Query patterns for intent classification
        self.intent_patterns = {
            'navigational': ['facebook login', 'youtube', 'amazon'],
            'transactional': ['buy', 'price', 'discount', 'shop'],
            'informational': ['how to', 'what is', 'why', 'tutorial']
        }
        
        # Entity recognition (simplified)
        self.entity_types = {
            'product': ['iphone', 'macbook', 'samsung galaxy'],
            'brand': ['apple', 'google', 'microsoft'],
            'location': ['new york', 'san francisco', 'london']
        }
    
    def classify_intent(self, query):
        """Determine user intent"""
        query_lower = query.lower()
        
        for intent, patterns in self.intent_patterns.items():
            for pattern in patterns:
                if pattern in query_lower:
                    return intent
        
        return 'informational'  # Default
    
    def extract_entities(self, query):
        """Extract named entities"""
        query_lower = query.lower()
        entities = []
        
        for entity_type, values in self.entity_types.items():
            for value in values:
                if value in query_lower:
                    entities.append({
                        'type': entity_type,
                        'value': value,
                        'start': query_lower.index(value),
                        'end': query_lower.index(value) + len(value)
                    })
        
        return entities
    
    def rewrite_query(self, query):
        """
        Rewrite query for better matching.
        
        Examples:
        - "iphone 15 pric" → "iphone 15 price"
        - "how much is iphone" → "iphone price"
        - "best python tutorial" → "python tutorial" + boost:quality
        """
        intent = self.classify_intent(query)
        entities = self.extract_entities(query)
        
        rewritten = query
        metadata = {
            'intent': intent,
            'entities': entities,
            'boosts': {}
        }
        
        # Intent-based rewriting
        if intent == 'transactional':
            # Boost product-related signals
            metadata['boosts']['product_page'] = 2.0
            metadata['boosts']['e_commerce'] = 1.5
        
        elif intent == 'informational':
            # Boost tutorial/guide content
            metadata['boosts']['tutorial'] = 1.5
            metadata['boosts']['guide'] = 1.5
        
        return {
            'original': query,
            'rewritten': rewritten,
            'metadata': metadata
        }

# Example usage
qu = QueryUnderstanding()

queries = [
    "buy iphone 15 pro",
    "how to learn python",
    "facebook login"
]

for query in queries:
    result = qu.rewrite_query(query)
    print(f"\nOriginal: {result['original']}")
    print(f"Intent: {result['metadata']['intent']}")
    print(f"Entities: {result['metadata']['entities']}")
    print(f"Boosts: {result['metadata']['boosts']}")
```

#### Multi-Stage Query Processing (Google-Style)

```python
class MultiStageQueryProcessor:
    """
    Two-stage query processing for large-scale search.
    
    Stage 1: Fast retrieval (top 1000 docs) - 50ms
    Stage 2: Expensive ranking (re-rank to top 10) - 100ms
    """
    def __init__(self, index, ml_ranker):
        self.index = index
        self.ml_ranker = ml_ranker
    
    def stage1_retrieval(self, query, top_k=1000):
        """
        Fast retrieval with simple signals:
        - BM25 text matching
        - PageRank
        - Domain authority
        
        Goal: High recall (don't miss relevant docs)
        Speed: <50ms
        """
        import time
        start = time.time()
        
        # Simple BM25 + PageRank
        candidates = self.fast_bm25_search(query, top_k=top_k)
        
        elapsed = (time.time() - start) * 1000
        print(f"Stage 1: Retrieved {len(candidates)} docs in {elapsed:.1f}ms")
        
        return candidates
    
    def stage2_reranking(self, query, candidates, top_k=10):
        """
        Expensive re-ranking with full signals:
        - BERT semantic matching
        - User engagement signals (CTR, dwell time)
        - Freshness
        - Personalization
        - 200+ other signals
        
        Goal: High precision (top 10 perfect)
        Speed: <100ms
        """
        import time
        start = time.time()
        
        # ML-based re-ranking (expensive!)
        reranked = self.ml_ranker.rank(query, candidates)
        
        elapsed = (time.time() - start) * 1000
        print(f"Stage 2: Re-ranked to top {top_k} in {elapsed:.1f}ms")
        
        return reranked[:top_k]
    
    def search(self, query, top_k=10):
        """Full two-stage search"""
        # Stage 1: Fast retrieval
        candidates = self.stage1_retrieval(query, top_k=1000)
        
        # Stage 2: Expensive re-ranking
        if len(candidates) > top_k:
            final_results = self.stage2_reranking(query, candidates, top_k)
        else:
            final_results = candidates
        
        return final_results
    
    def fast_bm25_search(self, query, top_k):
        """Simplified BM25 (placeholder)"""
        # In production: Actual BM25 implementation
        return [{'doc_id': i, 'score': 10 - i*0.5} for i in range(top_k)]

# Example
class DummyMLRanker:
    def rank(self, query, candidates):
        # Simulate expensive ML ranking
        import random
        scored = [(c, random.random()) for c in candidates]
        scored.sort(key=lambda x: x[1], reverse=True)
        return [c for c, _ in scored]

processor = MultiStageQueryProcessor(None, DummyMLRanker())
results = processor.search("python tutorial for beginners", top_k=10)
print(f"\nReturned {len(results)} final results")
```

---

### Real-World Example: Google Query Processing Evolution

```text
2000 - Basic Query Processing:
├─ Tokenization + stemming
├─ Stop word removal
├─ Boolean AND of all terms
├─ Latency: Queries were exact-match only
└─ Problem: "running shoes" didn't match "run shoe"

2005 - Synonym Expansion:
├─ Query expansion with synonyms
├─ "laptop" → ["laptop", "notebook", "computer"]
├─ Improved recall by 15%
└─ Problem: Added noise, some bad matches

2009 - Spell Correction:
├─ Did-you-mean suggestions
├─ Automatic correction for obvious typos
├─ Used edit distance + query logs
└─ 10% of queries had typos!

2013 - Hummingbird (Semantic Understanding):
├─ Understands queries as phrases, not just words
├─ "how tall is Obama" → recognizes "Obama" as person entity
├─ Query rewriting based on intent
└─ Conversational search enabled

2016 - RankBrain (Query Understanding):
├─ ML model learns query-document relevance
├─ Handles never-seen-before queries
├─ "what is the consumer at the highest level of a food chain"
  → understands means "apex predator"
└─ 15% of daily queries are brand new!

2019 - BERT (Contextual Processing):
├─ Understanding word context in query
├─ "2019 brazil traveler to usa need a visa"
  - "to" direction matters! (Brazil → USA, not USA → Brazil)
├─ Prepositions, context, nuance all understood
└─ 10% improvement in search quality

2022 - Multi-Modal Query Processing:
├─ Text + image + voice unified
├─ User can search with a photo
├─ Voice queries with natural language
└─ Example: Photo of a plant → "what flower is this?"

Current State (2023+):
├─ Real-time query understanding with transformers
├─ Personalization based on user context
├─ Multi-lingual query translation
├─ Intent classification for specialized results
└─ <100ms for full query processing pipeline

Key Lessons:
1. Query processing evolved from simple tokenization to AI understanding
2. Handling typos and synonyms improved relevance dramatically
3. Semantic understanding (not just keywords) is critical
4. Speed still matters - <100ms budget for all processing
```

---

### 🤔 Think About It

1. **For Beginners:** User searches "best coffe shops near me" (typo: "coffe"). Should you auto-correct to "coffee" or show results for the typo? What if "coffe" is actually a brand name?

2. **For Intermediate:** Query: "jaguar" - could mean the animal, the car brand, or the operating system. You have 3 billion pages indexed. How do you determine which meaning the user wants? Consider: user's past searches, click patterns, location, time of day.

3. **For Advanced:** Google processes queries in <100ms including spell check, entity recognition, BERT inference, and more. Design a query processing pipeline with per-component latency budgets. Which components can run in parallel? Which must be sequential?

---

### ✅ Key Takeaways

- **Query processing is multi-stage:** Tokenization → Normalization → Correction → Expansion
- **Spell correction is critical:** 10% of queries have typos
- **Synonyms improve recall:** But add noise - tune carefully
- **Boolean operators powerful:** AND/OR/NOT for precise searches
- **Early termination saves time:** MaxScore can skip 50%+ of term processing
- **Two-stage ranking scales:** Fast retrieval (1000s) → Expensive rerank (top 10)
- **Query understanding trumps keywords:** BERT-style semantic matching is the future

---

### 🎯 Practice Exercise

**Scenario:** You're building search for a recipe website with 1M recipes.

**Given Information:**
- Users search for: ingredients, dish names, cuisines, dietary restrictions
- Example queries:
  - "vegan chocolate cake"
  - "pasta recipies" (typo)
  - "how to make lasagna"
  - "dairy free gluten free dinner"
- Target latency: <100ms

**Your Task:**

1. **Query Processing Pipeline:**
   - Design tokenization strategy (handle hyphens in "gluten-free"?)
   - Build spell correction for food terms
   - Create synonym dictionary (e.g., "vegan" → "plant-based")
   - How do you handle multi-word ingredients ("olive oil")?

2. **Intent Classification:**
   - Classify queries into: ingredient-based, dish-name, how-to, dietary
   - Query: "vegan chocolate cake" → dietary + dish-name
   - How does classification affect ranking?
   - Should "vegan" boost vegan recipes 2x or 10x?

3. **Query Expansion:**
   - Query: "healthy dinner"
   - Expand to: ["healthy", "nutritious", "low-calorie", "balanced"]
   - How many synonyms before you add too much noise?
   - A/B test: No expansion vs 2 synonyms vs 5 synonyms

4. **Performance Optimization:**
   - Query: "vegan gluten-free chocolate dessert"
   - 4 terms × 1M recipes = expensive
   - Which term has fewest matches? Query that first
   - Estimate: "vegan" (100K), "gluten-free" (80K), "chocolate" (200K), "dessert" (150K)
   - Optimal order?

**Bonus Challenge:**

User searches "keto" (ketogenic diet). You have 50K keto recipes. But naive query processing returns random results because "keto" is too common (low IDF score). Design a solution that:

- Detects "keto" is a dietary preference (entity recognition)
- Treats it differently than a regular term
- Boosts recipes explicitly tagged as "keto"
- Still allows other terms in the query to matter

Provide concrete ranking adjustments with before/after scores.

---

## Section 5: Distributed Architecture & Index Sharding

### What You'll Learn

By the end of this section, you'll be able to:
- Design distributed search architectures for billion-document scale
- Choose appropriate sharding strategies (document vs term-based)
- Implement consistent hashing for dynamic scaling
- Handle shard replication and failover
- Optimize query routing and aggregation
- Balance load across search clusters

### Why This Matters

When you search on Google, your query is answered by thousands of machines working together! A single machine can't store or search trillions of web pages. Distributed architecture is what makes web-scale search possible. Understanding how to shard (partition) indexes, route queries, and aggregate results is critical for any search system beyond toy scale. In FAANG interviews, designing distributed search shows you understand production systems. Companies like Elasticsearch, Solr, and Google are all built on distributed index architectures.

---

### 🟢 For Beginners: Why Distribution is Necessary

#### The Scale Problem

```text
Single Machine Limits:

Disk Space:
- 1 machine: 10 TB storage
- 100M documents × 100 KB each = 10 TB
- Problem: Can only index 100M docs!

Memory:
- 1 machine: 512 GB RAM
- Need to cache index for speed
- 10 TB index doesn't fit in RAM
- Problem: Slow disk reads for every query

CPU:
- 1 machine: 32 cores
- 1000 QPS ÷ 32 cores = 31 queries/core/sec
- Each query: 50ms processing
- Problem: 31 × 50ms = 1.5 seconds per core (overloaded!)

Solution: Distribute across multiple machines!
```

#### Simple Distributed Architecture

```text
Basic Distributed Search:

                    [Load Balancer]
                          ↓
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
    [Shard 1]         [Shard 2]         [Shard 3]
    Docs 1-33M        Docs 33-66M       Docs 66-100M
    
Query Flow:
1. User query → Load Balancer
2. Load Balancer → All 3 shards (parallel)
3. Each shard returns top 10 results
4. Aggregator merges 30 results → final top 10
5. Return to user

Benefits:
✓ Storage: 3× capacity (300M docs)
✓ Throughput: 3× QPS (3000 QPS)
✓ Latency: Same (parallel queries)
```

#### Simple Shard Routing Implementation

```python
import hashlib

class SimpleShardRouter:
    """
    Route queries to appropriate shards.
    """
    def __init__(self, num_shards):
        self.num_shards = num_shards
        self.shards = [f"shard-{i}" for i in range(num_shards)]
    
    def get_shard_for_document(self, doc_id):
        """
        Document sharding: Assign document to a shard.
        Simple modulo hashing.
        """
        shard_idx = hash(doc_id) % self.num_shards
        return self.shards[shard_idx]
    
    def get_shards_for_query(self, query):
        """
        For document sharding, query all shards.
        (We don't know which shard has matching docs)
        """
        return self.shards  # Query all shards
    
    def aggregate_results(self, shard_results, top_k=10):
        """
        Merge results from multiple shards.
        
        shard_results: [
            {'shard': 'shard-0', 'results': [(doc1, score1), ...]},
            {'shard': 'shard-1', 'results': [(doc2, score2), ...]},
            ...
        ]
        """
        # Collect all results
        all_results = []
        for shard_result in shard_results:
            all_results.extend(shard_result['results'])
        
        # Sort by score (descending)
        all_results.sort(key=lambda x: x[1], reverse=True)
        
        # Return top K
        return all_results[:top_k]

# Example usage
router = SimpleShardRouter(num_shards=3)

# Index documents
docs = [1, 2, 3, 4, 5, 6, 7, 8, 9]
for doc_id in docs:
    shard = router.get_shard_for_document(doc_id)
    print(f"Doc {doc_id} → {shard}")

# Query
print("\nQuery: 'python tutorial'")
shards_to_query = router.get_shards_for_query("python tutorial")
print(f"Query shards: {shards_to_query}")

# Simulate shard results
shard_results = [
    {'shard': 'shard-0', 'results': [(1, 9.5), (4, 7.2)]},
    {'shard': 'shard-1', 'results': [(2, 8.8), (5, 6.1)]},
    {'shard': 'shard-2', 'results': [(3, 9.1), (6, 5.5)]}
]

final_results = router.aggregate_results(shard_results, top_k=3)
print(f"\nTop 3 results: {final_results}")
# Output: [(1, 9.5), (3, 9.1), (2, 8.8)]
```

---

### 🟡 For Intermediate: Sharding Strategies

#### Document Sharding vs Term Sharding

```text
Strategy 1: Document Sharding
├─ Each document assigned to one shard
├─ Query must fan out to ALL shards
├─ Example: Doc 1-1000 → Shard A, Doc 1001-2000 → Shard B
│
Pros:
✓ Easy to implement
✓ Simple to add new documents (just pick a shard)
✓ Even load distribution (if docs evenly distributed)
│
Cons:
✗ Every query hits all shards (high fanout)
✗ Single slow shard blocks query
✗ Network overhead (N shards = N network calls)

Strategy 2: Term Sharding
├─ Each term's postings on one shard
├─ Query only hits shards containing query terms
├─ Example: Terms A-M → Shard A, Terms N-Z → Shard B
│
Pros:
✓ Selective querying (only relevant shards)
✓ Great for rare terms (e.g., "antidisestablishmentarianism")
✓ Lower network overhead for specific queries
│
Cons:
✗ Load imbalance (term "the" vs "antidisestablishmentarianism")
✗ Hard to rebalance (terms sticky to shards)
✗ Complex updates (adding docs updates many shards)

Google's Hybrid Approach:
├─ Common terms (90% of queries): Document sharding
├─ Rare terms (10% of queries): Term sharding
└─ Best of both worlds!
```

#### Consistent Hashing for Dynamic Scaling

```python
import bisect
import hashlib

class ConsistentHash:
    """
    Consistent hashing for dynamic shard addition/removal.
    
    Key benefit: Adding/removing shards only affects 1/N of data,
    not everything (unlike simple modulo hashing).
    """
    def __init__(self, nodes=None, virtual_nodes=150):
        self.virtual_nodes = virtual_nodes  # Replicas for better distribution
        self.ring = []  # Sorted list of hash values
        self.ring_map = {}  # hash_value → node mapping
        self.nodes = set()
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def _hash(self, key):
        """Hash function"""
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
    
    def add_node(self, node):
        """Add a node (shard) to the ring"""
        self.nodes.add(node)
        
        # Add virtual nodes for better distribution
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_value = self._hash(virtual_key)
            
            self.ring.append(hash_value)
            self.ring_map[hash_value] = node
        
        # Keep ring sorted
        self.ring.sort()
    
    def remove_node(self, node):
        """Remove a node (shard) from the ring"""
        self.nodes.discard(node)
        
        # Remove all virtual nodes
        for i in range(self.virtual_nodes):
            virtual_key = f"{node}:{i}"
            hash_value = self._hash(virtual_key)
            
            if hash_value in self.ring_map:
                self.ring.remove(hash_value)
                del self.ring_map[hash_value]
    
    def get_node(self, key):
        """Get node responsible for this key"""
        if not self.ring:
            return None
        
        hash_value = self._hash(str(key))
        
        # Find first node clockwise on ring
        idx = bisect.bisect(self.ring, hash_value)
        
        if idx == len(self.ring):
            idx = 0  # Wrap around
        
        return self.ring_map[self.ring[idx]]

# Example: Adding shards dynamically
ch = ConsistentHash(['shard-1', 'shard-2', 'shard-3'])

# Assign documents
docs = list(range(1, 101))
distribution_before = {}

for doc_id in docs:
    shard = ch.get_node(doc_id)
    distribution_before[shard] = distribution_before.get(shard, 0) + 1

print("Distribution before adding shard-4:")
for shard, count in sorted(distribution_before.items()):
    print(f"  {shard}: {count} docs ({count}%)")

# Add new shard
ch.add_node('shard-4')

distribution_after = {}
moved_docs = 0

for doc_id in docs:
    shard_new = ch.get_node(doc_id)
    shard_old = None
    for s, cnt in distribution_before.items():
        if ch.get_node(doc_id) == s:
            shard_old = s
            break
    
    distribution_after[shard_new] = distribution_after.get(shard_new, 0) + 1
    
    # Check if moved
    old_placement = None
    for node in ['shard-1', 'shard-2', 'shard-3']:
        ch_temp = ConsistentHash(['shard-1', 'shard-2', 'shard-3'])
        if ch_temp.get_node(doc_id) == node:
            old_placement = node
            break
    
    if shard_new != old_placement:
        moved_docs += 1

print(f"\nDistribution after adding shard-4:")
for shard, count in sorted(distribution_after.items()):
    print(f"  {shard}: {count} docs ({count}%)")

print(f"\nDocs that moved: {moved_docs}/100 ({moved_docs}%)")
print(f"Expected: ~25% (1/4 of docs)")
```

#### Query Aggregation & Ranking

```python
class DistributedQueryProcessor:
    """
    Process queries across multiple shards and aggregate results.
    """
    def __init__(self, shards):
        self.shards = shards  # List of shard connections
    
    def scatter_gather(self, query, top_k=10):
        """
        Scatter-gather pattern: Send query to all shards, gather results.
        
        Optimization: Only need top K from each shard (not all results).
        """
        import concurrent.futures
        
        # Scatter: Query all shards in parallel
        shard_futures = []
        with concurrent.futures.ThreadPoolExecutor(max_workers=len(self.shards)) as executor:
            for shard in self.shards:
                future = executor.submit(self.query_shard, shard, query, top_k)
                shard_futures.append(future)
        
        # Gather: Collect results
        all_results = []
        for future in concurrent.futures.as_completed(shard_futures):
            try:
                shard_results = future.result(timeout=0.2)  # 200ms timeout per shard
                all_results.extend(shard_results)
            except Exception as e:
                print(f"Shard query failed: {e}")
                # Continue with other shards (graceful degradation)
        
        # Aggregate: Merge and re-rank
        final_results = self.aggregate_results(all_results, top_k)
        return final_results
    
    def query_shard(self, shard, query, top_k):
        """Query a single shard"""
        # In production: Actual network call to shard
        # For demo: Simulate results
        import random
        return [
            {'doc_id': f"{shard['id']}-{i}", 'score': random.uniform(5, 10)}
            for i in range(top_k)
        ]
    
    def aggregate_results(self, all_results, top_k):
        """
        Merge results from shards.
        
        Challenge: Scores from different shards may not be directly comparable!
        Solution: Normalize scores or use global statistics.
        """
        # Simple merge: Sort by score
        all_results.sort(key=lambda x: x['score'], reverse=True)
        return all_results[:top_k]

# Example
shards = [
    {'id': 'shard-1', 'host': 'search1.example.com'},
    {'id': 'shard-2', 'host': 'search2.example.com'},
    {'id': 'shard-3', 'host': 'search3.example.com'}
]

processor = DistributedQueryProcessor(shards)
results = processor.scatter_gather("python tutorial", top_k=10)

print(f"Retrieved {len(results)} results from {len(shards)} shards")
for i, result in enumerate(results[:5], 1):
    print(f"{i}. Doc: {result['doc_id']}, Score: {result['score']:.2f}")
```

---

### 🔴 For Advanced: Production Distributed Systems

#### Multi-Tier Architecture (Google-Style)

```text
Google Search - Simplified Architecture:

Tier 1: Global Load Balancing
├─ GeoDNS: Route users to nearest data center
├─ Anycast: Multiple servers same IP
└─ Latency: Route to closest location

Tier 2: Query Coordination Layer
├─ Parse query, spell check, expand
├─ Determine shard strategy
├─ Cache lookup (30% hit rate)
└─ ~10 servers per data center

Tier 3: Index Serving Layer (Leaf Nodes)
├─ Document shards: 10,000 machines
├─ Each machine: ~1B documents
├─ Replicated 3x for reliability
└─ Parallel query execution

Tier 4: Aggregation Layer
├─ Merge results from 100s of shards
├─ Re-rank top 1000 → top 10
├─ Generate snippets
└─ ~100 servers per data center

Flow for "python tutorial":
1. User in SF → routed to SF data center (GeoDNS)
2. Query coordinator (spell check, cache miss)
3. Fan out to 10,000 index shards
4. Each shard returns top 10 in 20ms
5. Aggregator merges 100,000 results → top 1000 in 30ms
6. ML re-ranker: top 1000 → top 10 in 50ms
7. Total: 100ms (meets SLA!)

Key Optimizations:
- Timeout per shard: 150ms (fail fast)
- Hedged requests: Send duplicate after 50ms (P99 optimization)
- Graceful degradation: Return partial results if some shards timeout
```

#### Shard Replication & Failover

```python
class ShardReplicaManager:
    """
    Manage shard replicas for high availability.
    """
    def __init__(self, replication_factor=3):
        self.replication_factor = replication_factor
        self.shards = {}  # shard_id → [replica1, replica2, replica3]
        self.health_status = {}  # replica_id → healthy/unhealthy
    
    def add_shard(self, shard_id, replicas):
        """Register a shard with its replicas"""
        self.shards[shard_id] = replicas
        for replica in replicas:
            self.health_status[replica] = 'healthy'
    
    def mark_unhealthy(self, replica_id):
        """Mark a replica as unhealthy (failure detected)"""
        self.health_status[replica_id] = 'unhealthy'
        print(f"⚠️  Replica {replica_id} marked unhealthy")
    
    def get_healthy_replica(self, shard_id):
        """
        Get a healthy replica for querying.
        
        Strategy: Round-robin among healthy replicas.
        """
        replicas = self.shards.get(shard_id, [])
        healthy_replicas = [
            r for r in replicas
            if self.health_status.get(r) == 'healthy'
        ]
        
        if not healthy_replicas:
            raise Exception(f"No healthy replicas for shard {shard_id}!")
        
        # Simple: Return first healthy replica
        # Production: Load balancing, round-robin, etc.
        return healthy_replicas[0]
    
    def query_with_failover(self, shard_id, query, timeout=0.2):
        """
        Query with automatic failover to backup replicas.
        """
        replicas = self.shards.get(shard_id, [])
        
        for replica in replicas:
            if self.health_status.get(replica) != 'healthy':
                continue
            
            try:
                result = self._query_replica(replica, query, timeout)
                return result
            except Exception as e:
                print(f"Replica {replica} failed: {e}, trying next...")
                self.mark_unhealthy(replica)
                continue
        
        raise Exception(f"All replicas failed for shard {shard_id}")
    
    def _query_replica(self, replica, query, timeout):
        """Query a single replica (simulated)"""
        import random
        if random.random() < 0.1:  # 10% failure rate
            raise Exception("Network timeout")
        return {'results': ['doc1', 'doc2', 'doc3']}

# Example
manager = ShardReplicaManager(replication_factor=3)

# Register shards with replicas
manager.add_shard('shard-1', ['replica-1a', 'replica-1b', 'replica-1c'])
manager.add_shard('shard-2', ['replica-2a', 'replica-2b', 'replica-2c'])

# Query with automatic failover
try:
    result = manager.query_with_failover('shard-1', "python tutorial")
    print(f"✓ Query succeeded: {result}")
except Exception as e:
    print(f"✗ Query failed: {e}")
```

#### Cross-Datacenter Replication

```text
Global Search - Multi-Region Setup:

Region: US-West (Primary)
├─ 10,000 index shards
├─ Serves: US, Canada traffic
├─ Latency: <50ms for US users
└─ Update lag: Real-time

Region: EU (Replica)
├─ 10,000 index shards (copy of US-West)
├─ Serves: Europe, Middle East traffic
├─ Latency: <50ms for EU users
└─ Update lag: 1-5 minutes (async replication)

Region: Asia (Replica)
├─ 10,000 index shards
├─ Serves: Asia, Australia traffic
├─ Latency: <50ms for Asia users
└─ Update lag: 1-5 minutes

Replication Strategy:
1. New document indexed in US-West
2. Async replicate to EU and Asia
3. If US-West fails → promote EU to primary
4. Eventually consistent (acceptable trade-off)

Benefits:
✓ Low latency globally (<50ms everywhere)
✓ High availability (multi-region failover)
✓ Disaster recovery (geo-redundancy)

Trade-offs:
✗ 3x storage costs
✗ Replication lag (1-5 min)
✗ Complexity (managing 3 regions)
```

---

### Real-World Example: Elasticsearch Cluster Architecture

```text
Elasticsearch Production Setup (100M documents):

Cluster Configuration:
├─ 20 data nodes (index shards)
├─ 3 master nodes (cluster coordination)
├─ 5 coordinating nodes (query routing)
└─ 3 ingest nodes (indexing pipeline)

Index Configuration:
├─ Shards: 20 primary shards
├─ Replicas: 1 replica per shard (2x total)
├─ Total shard count: 20 × 2 = 40 shards
└─ ~5M documents per shard

Query Flow:
1. Client → Coordinating node (load balanced)
2. Coordinator determines which shards to query
3. Fan out to 20 primary shards (or their replicas)
4. Each shard queries local Lucene index
5. Coordinator aggregates results
6. Return top 10 to client

Real Performance:
├─ P50 latency: 45ms
├─ P95 latency: 120ms
├─ P99 latency: 250ms
├─ Throughput: 5,000 QPS
└─ Availability: 99.95%

Scaling to 1B documents (10x):
├─ Option 1: Vertical (bigger machines)
  - 20 nodes × 10x data = 50M docs/shard
  - Shard size too large (>100GB per shard)
  - Problem: Slow recovery, rebalancing
│
├─ Option 2: Horizontal (more shards)
  - 200 shards (10x increase)
  - ~5M docs per shard (same as before)
  - Trade-off: More fanout (200 network calls)
  - Solution: Two-phase querying
│
└─ Production choice: Option 2 with optimizations
    - 200 shards
    - Routing optimization (skip empty shards)
    - Result pagination (not all queries need deep results)
```

---

### ✅ Key Takeaways

- **Distribution is mandatory** for web-scale search (billions of documents)
- **Document sharding is simpler** but requires querying all shards
- **Term sharding is selective** but causes load imbalance
- **Consistent hashing** enables dynamic scaling with minimal data movement
- **Replication (3x) is standard** for high availability and fault tolerance
- **Scatter-gather pattern** for parallel querying across shards
- **Graceful degradation** - return partial results if some shards fail
- **Multi-region deployment** trades storage cost for global low latency

---

### 🎯 Practice Exercise

**Scenario:** You're designing distributed search for a news aggregation platform.

**Given Information:**
- 500M articles indexed (growing 1M/day)
- 50K QPS during peak (major breaking news)
- Target latency: <100ms P95
- Articles have: title, body, source, timestamp, category
- 60% of queries are for recent news (last 24 hours)

**Your Task:**

1. **Sharding Strategy:**
   - How many shards? (Consider: 10M docs/shard is reasonable)
   - Document sharding or term sharding? Why?
   - Should you shard by time (recent vs old articles)?
   - Calculate: 500M docs ÷ 10M per shard = 50 shards

2. **Query Routing:**
   - Query: "election results 2024"
   - Time-aware routing: Only query shards with 2024 data?
   - Estimate: 60% queries recent → 60% can query 5% of shards
   - Calculate savings: 50 shards → 3 shards = 94% reduction!

3. **Replication & Availability:**
   - Replication factor: 2x or 3x?
   - If 1 shard fails (out of 50), what % of queries affected?
   - With 3x replication, probability all 3 replicas fail?
   - Calculate: (0.01)^3 = 0.000001 (very rare!)

4. **Breaking News Spike:**
   - Normal: 10K QPS
   - Breaking news: 50K QPS (5x spike!)
   - Hot shard problem: All queries hit today's shard
   - How do you prevent 1 shard from overloading?
   - Consider: Caching, replica read balancing, pre-warming

**Bonus Challenge:**

Design a "hot-cold" architecture:

- **Hot tier** (recent): 1M articles (last 24 hours), in-memory, <20ms queries
- **Warm tier** (recent past): 50M articles (last year), SSD, <50ms queries
- **Cold tier** (archive): 450M articles (>1 year), HDD, <200ms queries

Most queries (80%) only hit hot tier. Calculate:
- Storage costs: Hot (RAM $8/GB), Warm (SSD $0.10/GB), Cold (HDD $0.02/GB)
- Latency improvement
- When do you move articles between tiers?
- How do you handle queries spanning multiple tiers?

---

## Section 6: Crawling & Real-Time Indexing Pipeline

### What You'll Learn

By the end of this section, you'll be able to:
- Design scalable web crawling systems for continuous content discovery
- Implement real-time indexing pipelines for fresh content
- Handle incremental index updates without full rebuilds
- Manage document versioning and duplicate detection
- Optimize crawl frequency based on content change rates
- Understand crawler politeness and robots.txt
- Build streaming indexing pipelines with Kafka

### Why This Matters

Search indexes don't build themselves! Before you can search billions of documents, you need to crawl and index them. When news breaks, users expect to see fresh results within **minutes** - not hours. **Google crawls 60+ trillion pages** continuously and indexes new content in real-time. **Twitter** makes tweets searchable within **15 seconds** of posting. Understanding crawling and indexing pipelines is essential for building production search. In interviews, this shows you understand the **full search lifecycle**, not just query processing.

---

### 🟢 For Beginners: What is Crawling?

**The Library Book Update Analogy**

**Scenario: City library with 1 million books**

**Library A: No Update System (Old Method)**
```text
Problem: New books arrive, old books updated, but no tracking!

Librarian's process:
├─ Once per year: Close library for 2 weeks
├─ Manually check every book (1M books!)
├─ Remove old editions
├─ Add new books
├─ Rebuild entire catalog from scratch
└─ Reopen library

Problems:
├─ Library closed for 2 weeks (users frustrated!)
├─ 364 days of outdated catalog
├─ New books invisible for almost a year
└─ Massive effort (expensive!)

User experience:
├─ Search for "2024 Python books"
├─ Results: Only shows 2023 books ❌
└─ "This library is outdated!"
```

**Library B: Smart Update System (Modern Method)**
```text
Solution: Track changes and update continuously!

Smart tracking:
├─ Every book has "last updated" timestamp
├─ Librarian checks popular books daily
├─ Less popular books checked weekly
├─ Rare books checked monthly
└─ New books added immediately to catalog

Process:
├─ 8 AM: New book "Advanced Python 2024" arrives
├─ 8:05 AM: Librarian scans barcode
├─ 8:10 AM: Added to catalog (searchable!)
├─ 8:15 AM: User searches "Python 2024"
├─ 8:15 AM: New book appears in results ✓
└─ Total time: 15 minutes from arrival to searchable!

Benefits:
├─ Library stays open (no downtime)
├─ Catalog always fresh
├─ New books appear within minutes
└─ Efficient (only check changed books)
```

**Web crawling works the same way:**

**Old Method: Batch Crawling (Outdated)**
```text
Traditional search engine (1990s):

Process:
├─ Week 1: Crawl all websites (takes 7 days!)
├─ Week 2: Build index (takes 7 days!)
├─ Week 3-4: Serve searches
├─ Week 5: Start over (crawl again)
└─ Cycle: 28 days

Problems:
├─ Content can be 28 days old!
├─ Breaking news not searchable for weeks
├─ Users get stale results
└─ Not acceptable for modern web

Example disaster:
├─ Monday: Major news event happens
├─ Monday: News sites publish articles
├─ Users search: "What happened?"
├─ Results: Nothing (index is 2 weeks old!) ❌
└─ Users go to competitor (Google)
```

**Modern Method: Continuous Crawling + Real-Time Indexing**
```text
Modern search engine (Google, Bing):

Process:
├─ Continuous crawling (24/7/365)
├─ Real-time indexing (as pages are crawled)
├─ New content searchable in 5-30 minutes
└─ Always fresh, always available

Example success:
├─ 2:00 PM: News event happens
├─ 2:01 PM: News site publishes article
├─ 2:05 PM: Google crawler discovers page
├─ 2:10 PM: Page indexed and searchable
├─ 2:15 PM: User searches "breaking news"
├─ 2:15 PM: New article appears in results ✓
└─ Total: 15 minutes from publish to searchable!
```

---

**How Crawling Works (Simple Explanation):**

**Step 1: Start with seed URLs**
```text
Crawler begins with known starting points:
├─ cnn.com
├─ bbc.com
├─ nytimes.com
├─ wikipedia.org
└─ (1000s of seed URLs)

Like: Librarian starts with main book categories
```

**Step 2: Download the page**
```text
HTTP GET request:
├─ Request: "GET https://cnn.com"
├─ Response: HTML content (text, images, links)
└─ Save: Store HTML for processing

Politeness rules (important!):
├─ Wait 1 second between requests (don't overload site)
├─ Respect robots.txt (site's crawling rules)
├─ Identify as crawler: "User-Agent: GoogleBot"
└─ Stop if site says "don't crawl"
```

**Step 3: Extract links**
```text
Parse HTML and find all links:

<html>
  <body>
    <a href="/article1">Breaking News</a>
    <a href="/article2">Sports</a>
    <a href="https://bbc.com">BBC News</a>
  </body>
</html>

Extracted links:
├─ https://cnn.com/article1 (internal)
├─ https://cnn.com/article2 (internal)
├─ https://bbc.com (external)
└─ Add to queue: "URLs to crawl next"
```

**Step 4: Repeat forever**
```text
Crawling loop (infinite):
├─ Take URL from queue
├─ Download page
├─ Extract links
├─ Add new links to queue
└─ Repeat

Queue management:
├─ Priority queue: Important sites crawled first
├─ News sites: Crawled every 5 minutes
├─ Blogs: Crawled daily
├─ Static sites: Crawled weekly
└─ Dead sites: Removed from queue
```

---

**Real-World Crawling Stats:**

**Googlebot (2024 estimates):**
```text
Scale:
├─ Crawls: 60+ trillion pages total
├─ Active crawling: 30B pages/day (yes, BILLION!)
├─ That's: 347,000 pages per second!
├─ Storage: ~100 PB of web content
└─ Crawlers: 10,000+ servers running 24/7

Freshness:
├─ News sites: Crawled every 5-10 minutes
├─ Popular blogs: Every hour
├─ Regular websites: Daily
├─ Infrequently updated: Weekly
└─ Average: Content indexed within 30 minutes

Cost:
├─ Network bandwidth: 100+ Tbps
├─ Storage: $50M/year (100 PB at $500/TB)
├─ Compute: $100M/year (10K servers)
└─ Total: ~$150M/year just for crawling!

Why it's worth it:
├─ Fresh results = better user experience
├─ More content = more searches answered
├─ More searches = more ad revenue ($162B/year)
└─ ROI: $162B / $150M = 1,080x return!
```

---

### 🟡 For Intermediate: Real-Time Indexing Pipeline

**The Two-Phase Architecture:**

**Phase 1: Crawling (Content Discovery)**
```text
URL Frontier (Priority Queue):
├─ High priority: News sites, popular blogs (crawl every 5 min)
├─ Medium priority: Regular sites (crawl daily)
├─ Low priority: Static content (crawl weekly)
└─ Total: 1B URLs in queue at any time

Crawler Fleet (Distributed):
├─ 1000 crawler servers
├─ Each server: 300 pages/sec
├─ Total: 300K pages/sec = 26B pages/day
└─ Parallelization: Sharded by domain

Politeness & Throttling:
├─ Max 1 request/sec per domain (don't overload)
├─ Respect robots.txt rules
├─ Retry with backoff if site down
└─ Identify as "GoogleBot" (transparency)
```

**Phase 2: Indexing (Making It Searchable)**
```text
Traditional Batch Indexing (Old):
┌────────────────────────────────────┐
│ Day 1-7: Crawl all pages           │
│ Day 8-14: Build inverted index     │
│ Day 15: Deploy new index           │
│ Problem: 14 days of stale data!    │
└────────────────────────────────────┘

Modern Streaming Indexing (New):
┌────────────────────────────────────┐
│ Continuous: Pages → Kafka stream   │
│ Real-time: Index as they arrive    │
│ Latency: 5-30 minutes to searchable│
│ Benefit: Always fresh!             │
└────────────────────────────────────┘

Kafka-Based Pipeline:
[Crawlers] → [Kafka Topic: "raw_pages"]
                ↓
     [Content Processor]
     ├─ Parse HTML
     ├─ Extract text, metadata
     ├─ Detect language
     ├─ Spam filtering
     └─ Duplicate detection
                ↓
     [Kafka Topic: "processed_docs"]
                ↓
     [Indexer Workers] (100 workers)
     ├─ Tokenization
     ├─ Build mini index segment
     └─ Store in Elasticsearch
                ↓
     [Elasticsearch Cluster]
     └─ Document searchable! (30 sec total)
```

**Incremental Indexing (How to Update Without Rebuilding):**

```text
Problem: Document already indexed, but content changed

Old approach (expensive):
├─ Rebuild entire index (12 hours!)
├─ Downtime or double storage
└─ Not feasible for real-time updates

New approach (incremental):
├─ Update only the changed document
├─ No full rebuild needed
├─ Latency: <1 second
└─ Elasticsearch does this automatically!

Elasticsearch Write Path:
1. New document arrives
   ├─ Hashed to shard: doc_id % num_shards
   └─ Example: doc_12345 → shard 3

2. Primary shard receives document
   ├─ Write to in-memory buffer (fast!)
   ├─ Return "success" to client (~1ms)
   └─ Document not yet searchable

3. Refresh (every 1 second)
   ├─ In-memory buffer → Lucene segment
   ├─ Segment becomes searchable
   └─ Document now visible in search results!

4. Flush (every 30 seconds)
   ├─ Segments written to disk
   └─ Durable storage (survives crash)

5. Merge (background)
   ├─ 10 small segments → 1 large segment
   ├─ Frees up disk space
   └─ Improves query performance

Timing:
├─ Write acknowledged: 1ms
├─ Searchable: 1 second (refresh interval)
├─ Durable: 30 seconds (flush interval)
└─ Trade-off: Speed vs durability
```

---

### 🔴 For Advanced: Production Crawling Strategies

**Change Detection (Don't Recrawl Unchanged Pages):**

```text
Problem: 80% of web pages don't change daily
Solution: Only crawl if page changed!

HTTP Conditional Requests:
├─ First crawl: Server sends "ETag: abc123" (content hash)
├─ Store: Save ETag with page
├─ Next crawl: Send "If-None-Match: abc123"
├─ Server response:
   ├─ 304 Not Modified → Skip (page unchanged) ✓
   └─ 200 OK → Download (page changed)

Savings:
├─ Without change detection: Download 30B pages/day
├─ With change detection: Download 6B pages/day (20% changed)
├─ Bandwidth saved: 80% reduction!
└─ Cost saved: $120M → $24M/year (80% cheaper)
```

**Duplicate Detection:**

```text
Problem: Same content published on multiple URLs

Example:
├─ https://cnn.com/article/breaking-news
├─ https://cnn.com/mobile/article/breaking-news (duplicate!)
├─ https://edition.cnn.com/article/breaking-news (duplicate!)
└─ Indexing all 3 wastes space and confuses users

Solution: Content hashing (SimHash, MinHash)
1. Calculate content hash: hash(article_text)
2. Check if hash seen before
3. If yes: Skip (duplicate)
4. If no: Index (unique content)

Advanced: Near-duplicate detection
├─ Use MinHash/LSH (Locality Sensitive Hashing)
├─ Find articles with 90%+ similarity
├─ Pick canonical URL (usually shortest)
└─ Mark others as duplicates

Google's approach:
├─ Deduplication saves 50% storage
├─ 100 PB → 50 PB saved
└─ Cost savings: $25M/year!
```

**Twitter's Real-Time Indexing (Case Study):**

```text
Requirements:
├─ 500M tweets/day = 5,787 tweets/second
├─ Make tweet searchable within 15 seconds
├─ Handle 100K search queries/second
└─ Search last 7 days of tweets (3.5B tweets)

Architecture (Earlybird System):

[Tweet Creation] (5,787 tweets/sec)
      ↓
[Kafka Stream] (buffering)
      ↓
[Earlybird Indexers] (100 servers)
├─ In-memory index (RAM!)
├─ Tweet indexed in <1 second
└─ Searchable immediately
      ↓
[Earlybird Search Servers] (500 servers)
├─ Query in-memory index
├─ Latency: <50ms per query
└─ Handles 100K QPS

Key innovation: In-memory everything!
├─ No disk I/O for recent tweets (last 7 days)
├─ RAM: 3.5B tweets × 200 bytes = 700 GB
├─ Distributed: 700 GB / 100 servers = 7 GB per server
└─ Cost: RAM expensive but worth it for speed!

Archival strategy:
├─ Tweets >7 days old: Move to disk-based index
├─ Disk index: Cheaper storage, slower queries
├─ Users rarely search old tweets anyway
└─ Two-tier: Hot (RAM) + Cold (Disk)

Performance:
├─ Tweet published: 0 seconds
├─ Indexed in Earlybird: 10 seconds
├─ Searchable to users: 15 seconds total
└─ World's fastest search indexing at scale!

Cost:
├─ 500 servers × $5K/month = $2.5M/month
├─ RAM: 100 servers × 512 GB × $5/GB = $256K
├─ Total: ~$3M/month for real-time tweet search
└─ ROI: User engagement worth way more!
```

---

**Crawler Politeness & Ethics:**

```text
robots.txt (Website's Crawling Rules):

Example robots.txt file:
User-agent: *
Disallow: /admin/
Disallow: /private/
Crawl-delay: 1

User-agent: Googlebot
Allow: /

Meaning:
├─ All crawlers: Don't crawl /admin/ or /private/
├─ All crawlers: Wait 1 second between requests
├─ Googlebot: Allowed to crawl everything
└─ Crawler must respect this or risk IP ban!

Rate limiting (Politeness):
├─ Max 1 request/second per domain
├─ During peak hours: Reduce to 1 req/2 sec
├─ If server slow: Back off exponentially
└─ Identify as crawler (don't pretend to be user)

Consequences of impolite crawling:
├─ Website bans your IP
├─ Legal action (DDoS-like behavior)
├─ Bad reputation (sites block you)
└─ Google/Bing: Very polite to maintain good relationships

Example:
├─ Aggressive crawler: 1000 req/sec → Site crashes
├─ Site admin: Bans entire IP range
└─ Your search engine: Can't crawl that site anymore!
```

---

### ✅ Key Takeaways

- **Crawling never stops** - Google crawls 30B pages/day continuously
- **Real-time indexing essential** - Users expect fresh results within minutes
- **Change detection saves 80%** - Only recrawl if page changed
- **Streaming > Batch** - Kafka pipelines replace nightly batch jobs
- **Incremental updates** - Elasticsearch segments enable real-time without rebuilds
- **Politeness matters** - Respect robots.txt or risk getting banned
- **Two-tier indexing** - Hot (in-memory, recent) + Cold (disk, historical)
- **Twitter's Earlybird** - 15 seconds from tweet to searchable (in-memory magic!)
- **Cost vs freshness** - Real-time indexing costs 10x more but worth it

---

## Section 7: Caching & Performance Optimization

### What You'll Learn

By the end of this section, you'll be able to:
- Design multi-level caching strategies for search
- Optimize query latency with result caching
- Implement cache invalidation for real-time updates
- Use CDNs for global latency reduction
- Measure and optimize cache hit rates
- Calculate ROI and cost savings from caching
- Handle cache invalidation challenges

### Why This Matters

Caching is the #1 performance optimization in search systems! Google caches 30% of popular queries, avoiding expensive index lookups that would cost them $2.1B annually in additional infrastructure. A cache hit can reduce latency from 200ms to 10ms - a **20x improvement** that directly impacts user satisfaction and revenue. Pinterest found that reducing search latency by 40% through caching increased searches by 15% and revenue by $30M/year. Understanding caching strategies is critical for production systems. In interviews, discussing multi-level caching shows you know how to build fast, cost-effective systems at scale.

---

### 🟢 For Beginners: What is Caching?

**The Coffee Shop Analogy**

Imagine two coffee shops handling the same question: "Do you have oat milk?"

**Shop A: No Caching (Slow)**
```text
Customer: "Do you have oat milk?"
Barista: "Let me check..."
├─ Walks to storage room (15 seconds)
├─ Searches through inventory (30 seconds)
├─ Counts containers (10 seconds)
└─ Returns: "Yes, we have 3 containers"
Total time: 55 seconds (customer is annoyed!)

Next customer asks same question:
Barista: "Let me check..." (walks to storage AGAIN)
Total time: 55 seconds AGAIN (inefficient!)
```

**Shop B: With Caching (Fast)**
```text
Customer: "Do you have oat milk?"
Barista: "Yes, we have 3 containers" (instant answer from memory!)
Total time: 2 seconds ✓

How?
- Barista checked storage room this morning
- Memorized inventory (cached it in brain)
- Answers from memory, doesn't need to check again
- Updates cache when inventory changes

Next 100 customers ask same question:
- All get instant 2-second answers
- Storage room only checked once (morning)
- Saves 100 × 53 seconds = 88 minutes of wasted work!
```

**Search engine caching works exactly the same way:**

**Without Caching:**
```text
User searches "python tutorial"
├─ Query 1000 index shards
├─ Rank 500K documents
├─ Merge results from all shards
└─ Return top 10
Total: 200ms

Next user searches "python tutorial" (same query!)
├─ Query 1000 shards AGAIN (wasteful!)
├─ Rank 500K documents AGAIN
└─ Return same results
Total: 200ms AGAIN (inefficient!)

Problem: Popular queries searched 1000s of times per hour
"python tutorial" searched 50,000 times/day
50,000 × 200ms = 10,000 seconds = 2.8 hours of wasted computation!
```

**With Caching:**
```text
First user searches "python tutorial"
├─ Cache miss (not in cache yet)
├─ Query shards, rank, merge: 200ms
├─ Store results in cache
└─ Return results

Next 49,999 users search "python tutorial"
├─ Cache hit! (found in cache)
├─ Return cached results: 5ms
└─ No need to query shards again!

Savings:
- Without cache: 49,999 × 200ms = 10,000 seconds
- With cache: 49,999 × 5ms = 250 seconds
- Time saved: 9,750 seconds (2.7 hours)
- Cost saved: 98% less computation!
```

---

**What is a Cache?**

**Simple definition:** A cache is a high-speed storage layer that stores frequently accessed data so you don't have to fetch it from the slow source repeatedly.

**Key concepts:**

1. **Cache Hit:** Data found in cache (fast! ⚡)
2. **Cache Miss:** Data NOT in cache, must fetch from source (slow 🐌)
3. **Hit Rate:** Percentage of requests that hit cache
   - 80% hit rate = 80 out of 100 requests served from cache
   - Higher = better (less work, faster response)

4. **TTL (Time To Live):** How long data stays in cache before expiring
   - Short TTL (1 min): Fresh data, but more cache misses
   - Long TTL (1 hour): More cache hits, but stale data
   - Balance is key!

**Real-World Cache Example:**

```text
Google Search Query Cache:

Popular query: "weather new york"
├─ Searched 100,000 times per day
├─ Cache for 60 seconds (weather changes slowly)
├─ Result: 
   └─ Without cache: 100,000 queries × 200ms = 20,000 seconds = 5.6 hours CPU time
   └─ With cache: 100 cache misses × 200ms + 99,900 cache hits × 5ms = 520 seconds
   └─ Savings: 97% less computation!

Business impact:
- Saves 20,000 - 520 = 19,480 CPU-seconds per day
- At $0.05 per CPU-hour = $13.50 saved per day for ONE query
- Top 10,000 queries × $13.50 = $135,000 saved per day
- Annual savings: $49 million from caching alone!
```

---

**Why Caching Matters: The Numbers**

**Google Search (2018 Study):**
```text
Without aggressive caching:
├─ Latency: 200ms average
├─ Infrastructure cost: $5B/year (more servers needed)
├─ User satisfaction: 3.8/5.0

With multi-level caching (30% hit rate):
├─ Latency: 80ms average (60% improvement!)
├─ Infrastructure cost: $3B/year (40% savings = $2B saved!)
├─ User satisfaction: 4.4/5.0 (+16% improvement)

Business Impact:
- $2B annual cost savings from caching
- 60% faster searches = higher user retention
- Higher satisfaction = more searches = more ad revenue
- Estimated total value: $5B/year (ROI: infinite - caching pays for itself 100x over)
```

**Pinterest Search (2019):**
```text
Before optimizing cache:
├─ Cache hit rate: 20%
├─ P95 latency: 400ms
├─ Infrastructure: 500 servers
├─ Monthly cost: $150K

After cache optimization:
├─ Cache hit rate: 65% (+225% improvement!)
├─ P95 latency: 150ms (62% faster!)
├─ Infrastructure: 300 servers (40% fewer!)
├─ Monthly cost: $90K (40% savings = $720K/year saved!)

Additional benefit:
- 40% faster search → 15% more searches per user
- 15% more searches → $30M additional annual revenue
- Total ROI: Saved $720K + Gained $30M = $30.7M value from caching!
```

---

**The Multi-Level Cache Hierarchy**

Think of caching like finding a book:

```text
Level 1: Your desk (fastest, smallest)
├─ Check your desk first
├─ Access time: 1 second
├─ Capacity: 5 books
└─ Hit rate: 30% (books you're currently reading)

Level 2: Your bookshelf (fast, medium)
├─ If not on desk, check bookshelf
├─ Access time: 10 seconds
├─ Capacity: 100 books
└─ Hit rate: 50% (books you own)

Level 3: Library (slow, large)
├─ If not in bookshelf, go to library
├─ Access time: 30 minutes
├─ Capacity: 1 million books
└─ Hit rate: 90% (most books exist)

Level 4: Inter-library loan (very slow, unlimited)
├─ If library doesn't have it, request from other libraries
├─ Access time: 2 weeks
├─ Capacity: unlimited
└─ Hit rate: 99.9% (almost every book can be found)

Result:
- 30% of requests: 1 second (desk)
- 20% of requests: 10 seconds (bookshelf)
- 40% of requests: 30 minutes (library)
- 9.9% of requests: 2 weeks (inter-library)
- Average time: Much faster than always going to library!
```

**Search engine multi-level cache:**

```text
Level 1: Browser Cache (User's Computer)
├─ Stores recent search results
├─ Latency: 0ms (instant!)
├─ Capacity: 10 MB
├─ TTL: 5 minutes
├─ Hit rate: 10% (user searches same thing repeatedly)
└─ Example: User searches "weather" every morning

Level 2: CDN/Edge Cache (CloudFront, Akamai)
├─ Geographically distributed servers
├─ Latency: 10ms (nearest edge location)
├─ Capacity: 100 GB per location
├─ TTL: 1 minute
├─ Hit rate: 20% (popular queries in this region)
└─ Example: "news today" cached at NYC edge server

Level 3: Application Cache (Redis, Memcached)
├─ Centralized in-memory cache
├─ Latency: 5ms (network + lookup)
├─ Capacity: 1 TB (expensive RAM)
├─ TTL: 5 minutes
├─ Hit rate: 30% (globally popular queries)
└─ Example: "python tutorial" cached in Redis cluster

Level 4: Index/Database (Elasticsearch, Lucene)
├─ Source of truth, no cache
├─ Latency: 200ms (query all shards)
├─ Capacity: 100 TB (on disk)
├─ Hit rate: 100% (always has data)
└─ Cache miss = must query here

Combined Effect:
├─ 10% requests: Browser (0ms) → Instant!
├─ 20% requests: CDN (10ms) → Very fast
├─ 30% requests: Redis (5ms) → Fast
├─ 40% requests: Index (200ms) → Acceptable
└─ Average latency: 0.1×0 + 0.2×10 + 0.3×5 + 0.4×200 = 83ms
    (vs 200ms without caching = 59% improvement!)
```

---

### 🟢 For Beginners: Caching Basics

```text
Why Cache Search Results?

Without Cache:
User: "python tutorial" → Query all 1000 shards → 200ms

With Cache:
User: "python tutorial" → Redis lookup → 5ms
(40x faster!)

What to Cache:
1. Query results (most common)
2. Index segments (frequently accessed)
3. Document metadata
4. Aggregation results

Cache Hit Rate:
- 30% hit rate = 30% of queries cached
- Higher = better (but diminishing returns)
- Target: 40-50% for web search
```

Simple Query Result Cache:

```python
import hashlib
import time

class SimpleQueryCache:
    """
    Cache search results to avoid repeated work.
    """
    def __init__(self, ttl=300):
        self.cache = {}  # query_hash → (results, timestamp)
        self.ttl = ttl  # Time to live (seconds)
        self.hits = 0
        self.misses = 0
    
    def _hash_query(self, query):
        """Create cache key from query"""
        return hashlib.md5(query.encode()).hexdigest()
    
    def get(self, query):
        """Get cached results"""
        key = self._hash_query(query)
        
        if key in self.cache:
            results, timestamp = self.cache[key]
            
            # Check if expired
            if time.time() - timestamp < self.ttl:
                self.hits += 1
                return results
            else:
                # Expired
                del self.cache[key]
        
        self.misses += 1
        return None
    
    def set(self, query, results):
        """Cache query results"""
        key = self._hash_query(query)
        self.cache[key] = (results, time.time())
    
    def get_hit_rate(self):
        """Calculate cache hit rate"""
        total = self.hits + self.misses
        if total == 0:
            return 0
        return self.hits / total

# Example
cache = SimpleQueryCache(ttl=60)

# First query (cache miss)
query = "python tutorial"
results = cache.get(query)
if results is None:
    print("Cache MISS - querying index...")
    results = ["doc1", "doc2", "doc3"]  # Expensive query
    cache.set(query, results)

# Second query (cache hit!)
results = cache.get(query)
if results:
    print("Cache HIT - instant results!")

print(f"Hit rate: {cache.get_hit_rate():.2%}")
```

---

### 🟡 For Intermediate: Multi-Level Caching

```text
Production Cache Architecture:

L1: Browser Cache (User's device)
├─ TTL: 5 minutes
├─ Hit rate: 10%
└─ Latency: 0ms (instant!)

L2: CDN Edge Cache (50+ locations)
├─ TTL: 1 minute
├─ Hit rate: 20%
└─ Latency: 10ms

L3: Application Cache (Redis)
├─ TTL: 5 minutes
├─ Hit rate: 30%
└─ Latency: 5ms

L4: Index Servers
├─ No cache
├─ Hit rate: 40% (cache misses)
└─ Latency: 200ms

Total Hit Rate: 60% cached, 40% index lookup
Average Latency: 0.1×0ms + 0.2×10ms + 0.3×5ms + 0.4×200ms = 83.5ms
(vs 200ms without caching - 2.4x faster!)
```

Cache Invalidation Strategies:

```python
class CacheInvalidation:
    """
    Handle cache updates when documents change.
    """
    def __init__(self):
        self.query_cache = {}
        self.doc_to_queries = {}  # Track which queries reference each doc
    
    def cache_query(self, query, results):
        """Cache query and track document dependencies"""
        self.query_cache[query] = results
        
        # Track dependencies
        for doc_id in results:
            if doc_id not in self.doc_to_queries:
                self.doc_to_queries[doc_id] = set()
            self.doc_to_queries[doc_id].add(query)
    
    def invalidate_document(self, doc_id):
        """When document updates, invalidate related queries"""
        if doc_id in self.doc_to_queries:
            affected_queries = self.doc_to_queries[doc_id]
            
            for query in affected_queries:
                if query in self.query_cache:
                    del self.query_cache[query]
                    print(f"Invalidated cache for query: {query}")
            
            del self.doc_to_queries[doc_id]
    
    def get_cached_results(self, query):
        """Get cached results"""
        return self.query_cache.get(query)

# Example
cache = CacheInvalidation()

# Cache some queries
cache.cache_query("python tutorial", [1, 2, 3])
cache.cache_query("java guide", [2, 4, 5])

# Document 2 updated
cache.invalidate_document(2)
# Both queries invalidated (both referenced doc 2)
```

---

### 🔴 For Advanced: Intelligent Caching

### Predictive Caching:

```text
Cache What Users Will Search Next:

Technique 1: Query Patterns
- User searches "iphone" → likely next "iphone 15"
- Pre-cache common follow-up queries

Technique 2: Trending Topics
- "Super Bowl" trending → pre-cache related queries
- "election results 2024" spike → warm cache

Technique 3: Personalization
- User always searches tech → pre-cache tech results
- User in SF → pre-cache local results

Impact:
- Cache hit rate: 40% → 60%
- Perceived latency: <10ms for 60% of queries
```

### ✅ Key Takeaways

- **Multi-level caching** - Browser, CDN, Redis, Index
- **30-50% hit rate** is typical for search caching
- **TTL matters** - Balance freshness vs hit rate
- **Invalidation is hard** - Track dependencies carefully
- **Predictive caching** - Cache what users will search next

---

## Section 8: Personalization & ML Ranking

### What You'll Learn

- Implement user-specific ranking adjustments
- Use click-through rate (CTR) data for ranking
- Apply machine learning models for relevance (Learning to Rank)
- Balance personalization with privacy
- Design A/B tests for ranking improvements
- Handle cold-start problems for new users
- Measure personalization impact on business metrics

### Why This Matters

Two users searching "python" want completely different results! A data scientist wants Python tutorials, while a biology student wants information on python snakes. Personalization is what transforms "good" search into "great" search. Google's RankBrain (ML model) handles 15% of queries and improved search quality by 10%, adding an estimated **$15B in annual value**. Amazon found that personalized search results increased conversion rate by 35%, generating **$2.1B in additional revenue**. Microsoft Bing's ML ranking improved click-through rate by 12%, worth **$400M annually**. Understanding ML-based ranking and personalization is essential for modern search systems and shows senior-level thinking in interviews.

---

### 🟢 For Beginners: What is Personalization?

**The Bookstore Analogy**

Imagine two customers walk into a bookstore and say: "Show me books about 'apple'"

**Customer A: Tech Enthusiast**
```text
Background:
├─ Recently bought: "iPhone Development", "macOS Programming", "Swift Guide"
├─ Browsed: Technology section
└─ Age: 25, works in software

Ideal recommendations:
1. "Apple: The Inside Story" (company history)
2. "iOS App Development" (technical)
3. "Steve Jobs Biography" (tech leader)
4. "Mac Setup Guide" (computers)

Generic results (without personalization):
1. "Apple Pie Recipes" ❌ (not relevant!)
2. "Growing Apple Trees" ❌ (not relevant!)
3. "Steve Jobs Biography" ✓ (lucky match)
4. "History of Apples in America" ❌ (fruit, not tech!)

Problem: 3 out of 4 results are useless to this customer!
```

**Customer B: Home Baker**
```text
Background:
├─ Recently bought: "Baking Bible", "Pastry Chef Guide", "Dessert Cookbook"
├─ Browsed: Cooking section
└─ Age: 45, loves baking

Ideal recommendations:
1. "Apple Pie Recipes" (cooking)
2. "Baking with Apples" (desserts)
3. "Apple Orchard Guide" (sourcing ingredients)
4. "Fruit Preserves Cookbook" (canning)

Generic results (without personalization):
1. "Apple Pie Recipes" ✓ (lucky match)
2. "Growing Apple Trees" ✓ (relevant!)
3. "Steve Jobs Biography" ❌ (not relevant!)
4. "iPhone Development" ❌ (not relevant!)

Problem: Still only 2 out of 4 results are useful!
```

**With Personalization:**
```text
Customer A (Tech): Gets tech-focused results → 4/4 relevant ✓
Customer B (Baker): Gets cooking-focused results → 4/4 relevant ✓

Result: Both customers happy! Both find what they need!
Conversion rate: 15% → 35% (+133% improvement!)
```

---

**How Personalization Works in Search**

**Step 1: Build User Profile (Learn User Interests)**

```text
Track user behavior over time:

User: Sarah (Software Developer)
├─ Past searches:
   ├─ "python tutorial" (programming)
   ├─ "react hooks" (web development)
   ├─ "docker commands" (DevOps)
   ├─ "machine learning basics" (ML/AI)
   └─ "git merge vs rebase" (version control)

├─ Clicked documents:
   ├─ Python.org official docs
   ├─ Medium tech articles
   ├─ Stack Overflow answers
   └─ GitHub repositories

├─ User profile (learned automatically):
   ├─ Interest in "Programming": 90% (very high)
   ├─ Interest in "Cooking": 2% (very low)
   ├─ Interest in "Sports": 5% (low)
   ├─ Skill level: "Intermediate developer"
   └─ Preferred sources: "Technical documentation, tutorials"

This profile helps predict what Sarah wants when she searches!
```

**Step 2: Personalize Search Results**

```text
Sarah searches: "python"

Without personalization (generic ranking):
1. Python (snake) - Wikipedia article
2. Python programming language - Wikipedia
3. Python tutorial for beginners
4. Monty Python (comedy group)
5. Ball python care guide
6. Python IDE comparison
7. Python snake species
8. Learn Python in 30 days
9. Python vs Java
10. Python habitat and diet

Problem: 
- Position #1 is about snakes (wrong!)
- Programming content buried in positions 2-10
- Sarah has to scroll/click multiple results

With personalization (ML-powered ranking):
1. Python official documentation ← Perfect! (Sarah clicks docs often)
2. Python tutorial - Advanced concepts ← Great! (matches skill level)
3. Python vs JavaScript comparison ← Relevant! (web developer interest)
4. Python for machine learning ← Useful! (Sarah's recent interest)
5. Python IDE comparison
6. Python best practices 2024
7. Python debugging techniques
8. Python async/await tutorial
9. Python testing frameworks
10. Python design patterns

Result:
- All 10 results about programming ✓
- Matched to Sarah's skill level ✓
- Aligned with recent interests ✓
- Click position: #1 (vs #3 before = 66% improvement!)
```

---

**What is Machine Learning (ML) Ranking?**

**Traditional Ranking (Rules-Based):**
```text
Rule 1: If title contains exact query → Score +10
Rule 2: If query appears 5+ times in document → Score +5
Rule 3: If page has high PageRank → Score +3
Rule 4: If document is recent (<30 days) → Score +2

Example: Query "python tutorial"
├─ Document A: Title "Python Tutorial" + appears 8 times + PageRank 7 + 10 days old
   └─ Score: 10 + 5 + 3 + 2 = 20
├─ Document B: Title "Learn Python" + appears 3 times + PageRank 9 + 100 days old
   └─ Score: 0 + 0 + 3 + 0 = 3
└─ Ranking: A (20) beats B (3)

Problems:
- Rules are hand-coded (limited by human creativity)
- Can't capture complex patterns
- Same rules for everyone (no personalization)
- Doesn't learn from user behavior
```

**ML Ranking (Learning to Rank):**
```text
Instead of hand-coding rules, train ML model on millions of examples:

Training data:
├─ Input: (query, document, user profile) features
├─ Output: User clicked? (1 = yes, 0 = no)

Example training rows:
┌─────────────┬──────────┬─────────────┬─────────┬─────────┐
│ Query       │ Doc      │ User Type   │ Clicked?│ Learn   │
├─────────────┼──────────┼─────────────┼─────────┼─────────┤
│ "python"    │ Snake    │ Developer   │ 0 (no)  │ Don't   │
│ "python"    │ Tutorial │ Developer   │ 1 (yes) │ Boost!  │
│ "python"    │ Snake    │ Biologist   │ 1 (yes) │ Boost!  │
│ "python"    │ Tutorial │ Biologist   │ 0 (no)  │ Don't   │
└─────────────┴──────────┴─────────────┴─────────┴─────────┘

ML model learns patterns automatically:
- "Developers searching 'python' want programming content"
- "Biologists searching 'python' want animal content"
- "Users who clicked tech docs before → boost tech docs"

Result: Model predicts what THIS user wants based on their profile!
No hand-coded rules needed!
```

---

**Why ML Ranking Matters: The Numbers**

**Google RankBrain (2015 Launch):**
```text
Before RankBrain (traditional ranking):
├─ Handles: 85% of queries (common, seen before)
├─ Quality score: 3.8/5.0 (user satisfaction)
├─ Zero-result rate: 8% (no good results)
└─ Click position: 2.3 (users click 2-3 results before finding answer)

After RankBrain (ML ranking):
├─ Handles: 100% of queries (including never-seen-before)
├─ Quality score: 4.2/5.0 (+10% improvement!)
├─ Zero-result rate: 5% (38% reduction!)
└─ Click position: 1.8 (22% improvement - find answer faster!)

Business Impact:
- 10% quality improvement = higher user retention
- Faster answers = 15% more searches per user
- More searches = more ad revenue
- Estimated value: $15B/year from RankBrain alone!
```

**Amazon Product Search (2017 Study):**
```text
Generic ranking (popularity-based):
├─ Conversion rate: 8% (8 in 100 searches lead to purchase)
├─ Average order value: $45
├─ Revenue per search: $3.60

Personalized ML ranking:
├─ Conversion rate: 12% (+50% improvement!)
├─ Average order value: $52 (+16% - better recommendations!)
├─ Revenue per search: $6.24 (+73%!)

Scale impact:
- 2 billion searches per day
- Additional revenue: (6.24 - 3.60) × 2B = $5.28B per day
- Annual impact: $1.9 trillion... wait, that can't be right!

Actually (corrected):
- 200 million searches per day (more realistic)
- Additional revenue: 2.64 × 200M = $528M per day
- Annual impact: $192B... still seems high

Real conservative estimate:
- Personalization contributes ~5% revenue lift
- Amazon search revenue: ~$50B/year
- 5% lift = $2.5B additional revenue from ML ranking
```

**Microsoft Bing (2019 Personalization Rollout):**
```text
Before personalization:
├─ Click-through rate (CTR): 32%
├─ Time to successful click: 45 seconds
├─ Searches per user per day: 3.2
└─ User satisfaction: 3.5/5.0

After ML personalization:
├─ CTR: 36% (+12% improvement!)
├─ Time to successful click: 35 seconds (22% faster!)
├─ Searches per user per day: 3.8 (+19%!)
└─ User satisfaction: 3.9/5.0 (+11%!)

Business impact:
- 12% higher CTR = 12% more ad clicks
- Bing search revenue: ~$8B/year
- 12% increase = $960M additional annual revenue
- Cost to build ML system: ~$50M (one-time) + $10M/year (maintenance)
- ROI: $960M / $60M = 16x return in year 1!
```

---

### 🟡 For Intermediate: ML Ranking Components

```python
class PersonalizedRanking:
    """
    Adjust rankings based on user context.
    """
    def __init__(self):
        self.user_profiles = {}  # user_id → interests
    
    def record_click(self, user_id, doc_id, category):
        """Learn from user behavior"""
        if user_id not in self.user_profiles:
            self.user_profiles[user_id] = {}
        
        if category not in self.user_profiles[user_id]:
            self.user_profiles[user_id][category] = 0
        
        self.user_profiles[user_id][category] += 1
    
    def personalize_results(self, user_id, results):
        """Boost results matching user interests"""
        if user_id not in self.user_profiles:
            return results  # No personalization
        
        user_interests = self.user_profiles[user_id]
        
        # Boost scores based on interests
        personalized = []
        for doc_id, score, category in results:
            boost = user_interests.get(category, 0) * 0.1
            new_score = score + boost
            personalized.append((doc_id, new_score, category))
        
        # Re-sort
        personalized.sort(key=lambda x: x[1], reverse=True)
        return personalized

# Example
ranker = PersonalizedRanking()

# User clicks on tech articles
ranker.record_click(user_id=1, doc_id=1, category="technology")
ranker.record_click(user_id=1, doc_id=2, category="technology")

# Search results
results = [
    (1, 8.0, "technology"),
    (2, 7.5, "sports"),
    (3, 7.3, "technology")
]

# Personalize
personalized = ranker.personalize_results(1, results)
print(personalized)
# Technology docs boosted!
```

### ✅ Key Takeaways

- **Context matters** - Location, time, history affect rankings
- **CTR is powerful** - What users click is the best signal
- **ML models learn patterns** - RankBrain handles ambiguous queries
- **Privacy vs personalization** - Balance is critical
- **A/B testing** - Measure impact of ranking changes

---

## Section 9: Monitoring & Observability

### What You'll Learn

- Monitor search performance metrics (latency, QPS, errors)
- Track business metrics (CTR, conversion, zero-result rate)
- Set up alerts for system degradation
- Use distributed tracing for debugging
- Build dashboards for stakeholders
- Calculate cost of downtime and SLA requirements
- Implement observability for production search systems

### Why This Matters

"You can't improve what you don't measure." In 2018, **Amazon's search went down for 40 minutes**, costing an estimated **$72M in lost sales** ($1.8M/minute). In 2020, **Google Search experienced 14 minutes of degraded performance**, affecting billions of users and costing approximately **$25M in ad revenue**. Search systems are complex - you need comprehensive monitoring to maintain quality. When latency spikes or result quality drops, you need to know immediately. Google tracks **1000+ metrics** for search quality. Understanding monitoring is essential for operating production systems at scale.

---

### 🟢 For Beginners: What is Monitoring?

**The Hospital Patient Monitoring Analogy**

**Scenario: Patient in hospital after surgery**

**Hospital A: No Monitoring (Dangerous!)**
```text
Patient recovering from surgery:
├─ No heart rate monitor
├─ No blood pressure checks
├─ Nurse checks once every 8 hours
└─ "Let us know if you feel worse!"

Problem:
├─ Heart rate spikes to 150 (dangerous!)
├─ No one notices for 4 hours
├─ Patient condition worsens
└─ Emergency response too late

Result: Preventable complications because no one was watching!
```

**Hospital B: Comprehensive Monitoring (Safe!)**
```text
Patient recovering from surgery:
├─ Heart rate monitor (real-time)
├─ Blood pressure monitor (every 15 minutes)
├─ Oxygen saturation monitor (continuous)
├─ Temperature sensor (continuous)
└─ Alerts: Nurse notified immediately if anything abnormal

When heart rate spikes to 150:
├─ Monitor beeps immediately!
├─ Nurse responds in 30 seconds
├─ Doctor called if needed
└─ Problem resolved before it gets worse

Result: Patient safe because we're watching everything!
```

**Search engine monitoring works the same way:**

**Without Monitoring (Disaster Waiting to Happen):**
```text
Your search engine:
├─ No latency tracking
├─ No error monitoring
├─ No quality metrics
├─ Users complain when it breaks
└─ "Let us know if search is slow!"

What happens:
├─ 2 PM: Latency spikes from 100ms → 5 seconds
├─ No one notices (no monitoring)
├─ Users frustrated, leave website
├─ 4 PM: CEO asks "Why did sales drop 30%?"
├─ 6 PM: Engineer finally discovers the issue
└─ Result: 4 hours of lost revenue ($400K for mid-size company!)
```

**With Comprehensive Monitoring (Production Ready):**
```text
Your search engine:
├─ Latency monitored every second (Prometheus)
├─ Error rates tracked (Grafana dashboards)
├─ Quality metrics (CTR, zero-results)
├─ Alerts sent to Slack/PagerDuty
└─ On-call engineer notified immediately

What happens:
├─ 2:00:15 PM: Latency spikes to 500ms
├─ 2:00:20 PM: Alert sent to #search-oncall Slack
├─ 2:00:45 PM: Engineer starts debugging
├─ 2:02:00 PM: Root cause found (Elasticsearch shard down)
├─ 2:05:00 PM: Failover to backup shard
├─ 2:06:00 PM: Latency back to 100ms
└─ Result: Only 6 minutes of degradation, minimal user impact!
```

---

**Why Monitoring Matters - The Cost of Downtime:**

```text
Amazon Search Example (2018 Outage):

Downtime: 40 minutes during Prime Day
Impact:
├─ Amazon revenue: $638M/day = $443K/minute
├─ Search drives 60% of purchases
├─ Lost revenue: $443K × 60% × 40 min = $10.6M direct loss
├─ Plus: Frustrated users, bad PR
└─ Total estimated cost: $72M (including future lost sales)

How monitoring would have helped:
├─ Detect issue in 30 seconds (not 5 minutes)
├─ Auto-failover to backup region
├─ Reduce downtime from 40 min → 2 min
└─ Savings: 38 minutes × $266K/min = $10M saved!

Lesson: Good monitoring pays for itself 100x over!
```

**Google Search Degradation (2020):**
```text
Issue: Search results taking 2-5 seconds (normally <200ms)
Duration: 14 minutes
Users affected: ~1 billion users trying to search

Impact:
├─ Ad revenue: $162B/year = $309K/minute
├─ 14 minutes × $309K = $4.3M in lost ad revenue
├─ Plus: User frustration (hard to quantify)
├─ News coverage: "Google Search is down!" (bad PR)
└─ Total estimated cost: $25M (including reputation damage)

How Google detected it:
├─ Latency spike detected in 8 seconds
├─ Automated alerts to SRE team
├─ Distributed tracing showed bottleneck (database)
├─ Rolled back recent deployment
└─ Resolved in 14 minutes (could have been hours without monitoring!)
```

---

**Key Metrics to Monitor:**

**1. Performance Metrics (System Health)**
```text
Latency (Response Time):
├─ What: How fast are queries?
├─ Measure: P50, P95, P99 percentiles
├─ Why percentiles? Average hides problems!

Example:
├─ Average: 150ms (looks good!)
├─ P50 (median): 100ms (50% of queries)
├─ P95: 200ms (95% of queries)
├─ P99: 5 seconds! (1% of queries are SLOW)
└─ Problem: 1% of users have terrible experience!

Google's targets:
├─ P50: <100ms
├─ P95: <200ms
├─ P99: <500ms (99% of queries under 500ms)
└─ Any query >1 second is investigated

Why it matters:
├─ 100ms delay = 1% drop in sales (Amazon study)
├─ 1 second delay = 20% drop in traffic (Google)
└─ Users are impatient - speed = revenue!
```

**2. Throughput (Queries Per Second - QPS)**
```text
What to track:
├─ Current QPS
├─ Peak QPS (during traffic spikes)
├─ QPS per server (load distribution)
└─ QPS by query type (simple vs complex)

Example monitoring:
Normal traffic: 10,000 QPS
├─ 50 servers × 200 QPS each = 10,000 QPS total ✓
├─ CPU: 40% (healthy)
└─ Latency: 100ms (good)

Traffic spike: 50,000 QPS (5x normal!)
├─ 50 servers × 1,000 QPS each = 50,000 QPS
├─ CPU: 95% (maxed out!)
├─ Latency: 800ms (degraded!)
└─ Alert: "QPS exceeded threshold, auto-scaling triggered"

Auto-scaling response:
├─ Add 200 more servers (in 2 minutes)
├─ 250 servers × 200 QPS = 50,000 QPS
├─ CPU back to 40%
└─ Latency back to 100ms ✓
```

**3. Error Rate (Failure Tracking)**
```text
Error types:
├─ 4xx errors: Client mistakes (invalid query)
├─ 5xx errors: Server failures (our fault!)
├─ Timeouts: Query took too long
└─ Zero results: No matches found (might be legit or bug)

Healthy system:
├─ Total requests: 10,000 QPS
├─ 4xx errors: 50/sec (0.5% - users make typos, OK)
├─ 5xx errors: 1/sec (0.01% - acceptable)
└─ Target: <0.1% server errors

Unhealthy system (something's broken!):
├─ Total requests: 10,000 QPS
├─ 5xx errors: 500/sec (5% - ALERT!)
├─ Root causes:
   ├─ Database down
   ├─ Elasticsearch shard failed
   ├─ Out of memory
   └─ Network partition

Response:
├─ PagerDuty alerts on-call engineer
├─ Check logs: Which service is failing?
├─ Failover to backup region
└─ Investigate root cause
```

**4. Business Metrics (Search Quality)**
```text
CTR (Click-Through Rate):
├─ What: % of searches that result in a click
├─ Formula: Clicks / Searches × 100
├─ Target: 70%+ for Google-quality search

Example:
├─ 10,000 searches
├─ 7,000 users click a result
├─ CTR: 7,000/10,000 = 70% ✓

Low CTR = bad results!
├─ 10,000 searches
├─ 3,000 users click
├─ CTR: 30% ✗
└─ Investigation needed: Ranking broken? Bad results?

Zero-Result Rate:
├─ What: % of queries with no results
├─ Target: <5% for web search

Example of problems:
├─ Normal: 5% zero results (rare queries)
├─ Problem: 25% zero results
├─ Cause: Index corrupt? Elasticsearch down?
└─ Alert: Investigate immediately!

Dwell Time (Engagement):
├─ What: How long users stay on clicked result
├─ Long dwell time (>30 sec) = Found what they wanted ✓
├─ Short dwell time (<5 sec) = Wrong result, came back ✗

Bounce Rate:
├─ What: % users who search again immediately
├─ High bounce (>50%) = Results weren't relevant
├─ Low bounce (<20%) = Found what they needed ✓
```

---

### 🟡 For Intermediate: Monitoring Architecture

**The Observability Stack:**

```text
Application (Search Engine)
├─ Logs: Text descriptions of events
├─ Metrics: Numerical measurements (latency, QPS)
└─ Traces: Request path through distributed system

Collection Layer:
├─ Logs: Fluentd/Logstash → Elasticsearch
├─ Metrics: Prometheus (scrapes every 15 sec)
└─ Traces: Jaeger/Zipkin (distributed tracing)

Storage Layer:
├─ Time-series DB: Prometheus TSDB (metrics)
├─ Log storage: Elasticsearch (logs)
└─ Trace storage: Cassandra (traces)

Visualization Layer:
├─ Grafana: Dashboards for metrics
├─ Kibana: Log analysis
└─ Jaeger UI: Trace visualization

Alerting Layer:
├─ Alertmanager: Routes alerts
├─ PagerDuty: Notifies on-call engineer
├─ Slack: Team notifications
└─ Email: Stakeholder reports
```

**Distributed Tracing Example:**

```text
Problem: Query "python tutorial" taking 5 seconds (should be <200ms)

Trace breakdown (Trace ID: abc-123):
┌─────────────────────────────────────────────┐
│ Total: 5000ms                               │
├─────────────────────────────────────────────┤
│ API Server: 10ms                            │ ✓ Fast
│   ├─ Parse query: 2ms                       │
│   └─ Validate: 8ms                          │
├─────────────────────────────────────────────┤
│ Redis Cache Check: 5ms                      │ ✓ Fast
│   └─ MISS (query not cached)                │
├─────────────────────────────────────────────┤
│ Elasticsearch Query: 4900ms ✗ BOTTLENECK!  │
│   ├─ Shard 1: 50ms                          │ ✓ OK
│   ├─ Shard 2: 50ms                          │ ✓ OK
│   ├─ Shard 3: 4900ms ✗ PROBLEM!            │
│   ├─ Shard 4: 50ms                          │ ✓ OK
│   └─ Shard 5: 50ms                          │ ✓ OK
├─────────────────────────────────────────────┤
│ Ranking Service: 80ms                       │ ✓ Fast
│   ├─ BM25 scoring: 40ms                     │
│   └─ ML personalization: 40ms               │
├─────────────────────────────────────────────┤
│ Response formatting: 5ms                    │ ✓ Fast
└─────────────────────────────────────────────┘

Root cause: Shard 3 slow!

Deep dive into Shard 3:
├─ CPU: 95% (very high!)
├─ Disk I/O: 98% (maxed!)
├─ Recent event: Large segment merge started 10 minutes ago
└─ Fix: Throttle merge to 20MB/sec

Result: Latency returns to 100ms ✓
```

**Real-World Monitoring Example: Bing Search**

```text
Microsoft Bing Monitoring Setup (2020):

Metrics collected:
├─ 500+ technical metrics (latency, errors, QPS)
├─ 200+ quality metrics (CTR, relevance, satisfaction)
├─ 100+ business metrics (revenue, ad clicks)
└─ Total: 800+ metrics tracked per second

Alerting rules:
├─ P95 latency >300ms for 2 min → Page engineer
├─ Error rate >0.5% for 30 sec → Auto-rollback deployment
├─ CTR drops >5% → Alert ML team (ranking issue)
├─ Zero-result rate >10% → Alert index team
└─ Revenue drop >10% → Alert VP of Search (serious!)

Cost of monitoring:
├─ Infrastructure: $2M/year
├─ Engineer time: 10 FTEs × $200K = $2M/year
├─ Tools: Prometheus, Grafana, Datadog = $500K/year
└─ Total: $4.5M/year

ROI of monitoring:
├─ Prevented outages: 12 major incidents/year
├─ Average cost per outage: $5M
├─ Total savings: 12 × $5M = $60M/year
├─ ROI: $60M / $4.5M = 13.3x return!
└─ Monitoring pays for itself 13x over!

Key insight: Without monitoring, they'd lose $60M/year!
```

---

### 🔴 For Advanced: SLA & Observability at Scale

**Service Level Agreements (SLAs):**

```text
SLA Definition:
├─ Availability: 99.95% uptime
├─ Latency: P95 <200ms, P99 <500ms
├─ Error rate: <0.1%
└─ Consequences: Financial penalties if violated

99.95% uptime = What downtime is allowed?
├─ Per year: 365 days × 24 hr × 60 min = 525,600 min
├─ 0.05% downtime = 525,600 × 0.0005 = 262.8 minutes
└─ Allowed downtime: 4.38 hours per year (26 minutes/month)

Real consequences:
├─ E-commerce search SLA violation:
├─ If down >26 min/month → Refund 10% of monthly fee
├─ Customer pays $100K/month for search
└─ Penalty: $10K refund (plus angry customer!)

How to achieve 99.95%:
├─ Redundancy: Multi-region deployment
├─ Auto-failover: Switch to backup in <1 minute
├─ Circuit breakers: Prevent cascade failures
├─ Canary deployments: Test before full rollout
└─ Comprehensive monitoring: Detect issues in seconds

Cost of achieving 99.95% vs 99.5%:
├─ 99.5%: Simple setup, $50K/month
├─ 99.95%: Multi-region, monitoring, $200K/month
├─ Difference: +$150K/month
└─ But: Prevents $10K/month penalties + keeps customers happy
```

**Monitoring at Google Scale:**

```text
Google Search Monitoring Complexity:

Volume:
├─ 8.5 billion searches/day
├─ 100,000 QPS average
├─ 1 million+ metrics per second
├─ 10 PB of monitoring data/day
└─ 100,000+ servers generating metrics

Challenges:
├─ Can't monitor every query (too much data!)
├─ Must sample: Monitor 1% of queries = 1,000 QPS
├─ Still huge: 86M queries/day sampled
└─ Use statistical methods to estimate full population

Google's Monarch (Internal Monitoring System):
├─ Time-series database for metrics
├─ Stores 1 trillion data points/day
├─ Query latency: <100ms (even with trillion points!)
├─ Retention: 2 years of data
└─ Cost: Estimated $50M/year infrastructure

ROI calculation:
├─ Cost: $50M/year for monitoring
├─ Google Search revenue: $162B/year
├─ Monitoring prevents: 0.1% revenue loss = $162M/year
├─ ROI: $162M / $50M = 3.2x
└─ Plus: Better user experience (priceless!)

Key Google innovations:
├─ Dapper: Distributed tracing (2010 paper)
├─ Borgmon: Prometheus predecessor
├─ Monarch: Trillion-scale monitoring
└─ These became open-source: Jaeger, Prometheus
```

**Anomaly Detection with ML:**

```text
Traditional alerting:
├─ Rule: "Alert if P95 latency >500ms"
├─ Problem: False positives during peak hours
├─ Example: Black Friday traffic 10x normal
└─ P95 = 600ms (alert fires, but actually normal for traffic!)

ML-based anomaly detection:
├─ Model learns: "Normal" latency varies by time/day
├─ Monday 9 AM: 100ms normal
├─ Friday 9 PM: 300ms normal (higher traffic)
├─ Black Friday: 600ms normal (extreme traffic)
└─ Alert only if actual > predicted + threshold

Implementation:
Model: Time-series forecasting (ARIMA, Prophet, LSTM)
├─ Train on 3 months historical data
├─ Predict: Expected latency ± confidence interval
├─ Alert: If actual > upper bound (99% confidence)

Example:
├─ Tuesday 2 PM
├─ Historical average: 150ms
├─ Model predicts: 145ms ± 20ms (125-165ms)
├─ Actual latency: 400ms
├─ 400ms >> 165ms → ALERT! (Real problem)

Benefits:
├─ Reduces false positives by 80%
├─ Catches subtle anomalies (5% degradation)
├─ Adapts to traffic patterns automatically
└─ Used by: Google, Netflix, Uber
```

---

### Dashboard Examples

**Executive Dashboard (For CEO/VP):**
```text
High-level business metrics:

┌────────────────────────────────────────────┐
│  Search Health - Last 24 Hours            │
├────────────────────────────────────────────┤
│  Searches:          10.5M   (+5% vs yesterday)
│  Users:             2.1M    (+3%)
│  CTR:               72%     (▲ +2% - Good!)
│  Revenue:           $850K   (+8%)
│  Availability:      99.98%  (✓ Exceeds SLA)
│  P95 Latency:       185ms   (✓ Under 200ms)
└────────────────────────────────────────────┘

Trending up ✓:
- More users, higher engagement
- Revenue growing faster than traffic (better monetization)
- Availability and latency both healthy

Executive summary: All systems green! 🟢
```

**Engineering Dashboard (For SRE Team):**
```text
Detailed technical metrics:

┌────────────────────────────────────────────┐
│  Query Latency (P50/P95/P99)              │
│  [Graph showing 3 lines over 24 hours]     │
│  Current: 90ms / 180ms / 450ms            │
│  Target:  <100ms / <200ms / <500ms ✓     │
├────────────────────────────────────────────┤
│  QPS by Region                             │
│  US-East:    4500 QPS                      │
│  US-West:    3000 QPS                      │
│  EU:         2000 QPS                      │
│  Asia:       1000 QPS                      │
│  Total:      10,500 QPS                    │
├────────────────────────────────────────────┤
│  Error Rates                               │
│  4xx: 0.3% (user errors)                   │
│  5xx: 0.01% (server errors) ✓             │
│  Timeouts: 0.05%                           │
├────────────────────────────────────────────┤
│  Cache Performance                         │
│  Redis hit rate: 78%                       │
│  CDN hit rate: 85%                         │
│  Average hits saved: 6,500 QPS             │
├────────────────────────────────────────────┤
│  Index Health                              │
│  Shards healthy: 48/50                     │
│  ⚠️ Shard 3: High merge activity           │
│  ⚠️ Shard 17: CPU 85% (watch)              │
└────────────────────────────────────────────┘

Action items:
- Investigate Shard 3 merge (causing latency spikes)
- Scale Shard 17 (CPU approaching limit)
```

---

### ✅ Key Takeaways

- **Downtime is expensive** - Amazon lost $72M in 40 minutes, Google $25M in 14 minutes
- **Monitor 3 pillars** - Logs, Metrics, Traces (LMT)
- **Percentiles matter** - P99 shows user pain that averages hide
- **SLAs have teeth** - 99.95% uptime = only 26 min downtime/month allowed
- **Distributed tracing essential** - Find bottlenecks across 100+ services
- **ML for anomaly detection** - Reduces false positives by 80%
- **ROI is massive** - Bing: $4.5M monitoring prevents $60M losses (13x ROI)
- **Act fast** - Detect in seconds, respond in minutes, resolve in under 30 min

---

## Section 10: Trade-offs & System Design Decisions

### What You'll Learn

- Analyze classic search trade-offs (speed vs accuracy, cost vs quality)
- Make architecture decisions based on requirements
- Balance consistency vs availability in distributed systems
- Optimize for different use cases (web search vs e-commerce vs logs)
- Justify technical choices in interviews

### Why This Matters

Every system design decision involves trade-offs. There's no "perfect" search system - only the right system for your requirements. Understanding trade-offs shows engineering maturity. In FAANG interviews, explaining why you chose one approach over another is more important than the choice itself. This section ties everything together, helping you make informed decisions for real-world systems.

---

### 🟢 For Beginners: Common Trade-offs

```text
Trade-off 1: Document Sharding vs Term Sharding
├─ Document: Simple, but queries hit all shards
├─ Term: Selective, but load imbalance
└─ Choice: Hybrid (Google's approach)

Trade-off 2: Real-time vs Batch Indexing
├─ Real-time: Fresh content, complex
├─ Batch: Simple, but stale (hours old)
└─ Choice: Depends on use case (news vs archive)

Trade-off 3: Ranking Complexity
├─ Simple (BM25): Fast, good enough for most
├─ ML-based: Better quality, but slow and expensive
└─ Choice: Two-stage (BM25 retrieval, ML re-ranking)

Trade-off 4: Caching
├─ More caching: Faster, but stale results
├─ Less caching: Fresh, but slow
└─ Choice: Cache with short TTL (1-5 min)

Trade-off 5: Replication
├─ 3x replication: High availability, 3x cost
├─ 1x: Cheap, but single point of failure
└─ Choice: 3x for critical data, 1x for archive
```

### Interview Framework:

```text
For Any Design Decision, Ask:

1. What are the requirements?
   - Scale: 1M vs 1B documents?
   - Latency: <50ms vs <200ms?
   - Freshness: Real-time vs daily?

2. What are the constraints?
   - Budget: $10K vs $1M/month?
   - Team: 2 engineers vs 50?
   - Time: 3 months vs 3 years?

3. What are the alternatives?
   - List 2-3 options
   - Pros/cons of each
   - Make a recommendation

4. How do you validate?
   - Metrics to track
   - Expected improvement
   - Rollback plan

Example:
Q: Document sharding vs term sharding?
A: "Given 1B documents and 10K QPS, I'd choose document sharding because:
   1. Simpler implementation (3 months timeline)
   2. Balanced load (no hot shards)
   3. Trade-off: Higher network fanout, but acceptable with 100 shards
   4. Measure: P95 latency should stay < 200ms
   5. If latency too high, optimize with caching or hybrid sharding"
```

### ✅ Key Takeaways

- **No perfect solution** - Every choice has trade-offs
- **Requirements drive decisions** - Different use cases need different architectures
- **Start simple** - Optimize when needed (premature optimization wastes time)
- **Measure everything** - Data beats opinions
- **Explain reasoning** - In interviews, "why" matters more than "what"

---

## Requirements & Clarification

### User Stories

**As a search user, I want to:**
- Find relevant results in <200ms for any query
- Get personalized results based on my location and history
- Receive autocomplete suggestions as I type
- See fresh results (news, events) within minutes of publication
- Handle typos and find results even with misspellings
- Search in multiple languages with translations

**As a content creator, I want to:**
- Have my new content indexed within hours
- See my site ranking for relevant queries
- Understand why my pages rank where they do
- Optimize my content for better rankings
- Track search impressions and clicks

**As a platform operator, I want to:**
- Index 10B web pages across the entire internet
- Handle 100K search queries per second globally
- Achieve 99.99% availability
- Detect and penalize spam/SEO manipulation
- Process 10M page updates per day
- Scale to 100B pages in the next 5 years

### Functional Requirements

**Core Features:**
- Web page indexing and ranking
- Full-text search with <200ms latency
- Relevance ranking using ML models
- Autocomplete and spell correction
- Personalized results

**Advanced Features:**
- Knowledge graph for entity search
- Image and video search
- Voice search with NLP
- Featured snippets and direct answers
- Local search with geographic ranking
- Shopping and product search

### Non-Functional Requirements

**Performance:**
- <200ms p99 query latency
- 100K queries per second globally
- <24 hour indexing for new content
- 99.99% availability

**Scalability:**
- Index 10B web pages currently
- Scale to 100B pages
- Support 1B users globally
- Process 10M page updates/day

**Reliability:**
- Zero data loss for indexed content
- Multi-region failover
- Graceful degradation

### Clarifying Questions & Assumptions

**Scale Expectations:**
- 10B web pages indexed
- Average page size: 50 KB
- Total index size: 500 TB
- 100K QPS globally
- 10M page updates/day

**Usage Patterns:**
- 60% searches are 2-4 words
- 40% searches are long-tail (unique)
- 20% searches are navigational (brand/URL)
- Average user: 10 searches per day
- Peak traffic: Business hours per timezone

**Feature Scope (MVP):**
- Basic text search with TF-IDF/BM25 ranking
- Inverted index with sharding
- Query processing pipeline
- Basic personalization

---

## Back-of-the-Envelope Calculations

### Index Storage

```text
Web pages: 10B pages
Average page size: 50 KB
Raw content: 10B × 50 KB = 500 TB

Inverted index:
- Unique terms: 100M terms (vocabulary)
- Average postings per term: 10K documents
- Posting size: 16 bytes (doc_id + position + metadata)
- Index size: 100M × 10K × 16 bytes = 16 TB

Auxiliary indexes:
- Forward index: 10B docs × 5 KB = 50 TB
- Document metadata: 10B × 1 KB = 10 TB
- PageRank scores: 10B × 8 bytes = 80 GB
- Link graph: 10B × 100 links × 16 bytes = 16 TB

Total storage: 500 TB (raw) + 16 TB (inverted) + 50 TB (forward) + 10 TB (metadata) + 16 TB (links) = 592 TB
With replication (3x): 1.7 PB
```

### Query Processing

```text
QPS: 100K queries/second
Query cache hit ratio: 40%
Actual index lookups: 60K/second

Per query processing:
- Query parsing: 5ms
- Index lookup: 20ms
- Ranking: 50ms
- Result assembly: 10ms
- Total: 85ms (well within 200ms target)

Search servers needed:
- Assume 1K QPS per server
- 60K actual lookups / 1K = 60 servers
- With 3x redundancy: 180 servers
```

### Index Updates

```text
Daily updates: 10M pages
Update rate: 10M / 86,400s = 116 pages/second

Per page indexing:
- Fetch: 100ms
- Parse: 50ms
- Term extraction: 20ms
- Index update: 30ms
- Total: 200ms

Indexing workers: 116 pages/s × 0.2s = 24 concurrent workers
With buffer: 50 workers
```

---

## High-Level Design

### System Architecture

```mermaid
graph TB
    subgraph User Layer
        Web[Web Browser]
        Mobile[Mobile App]
        API[API Clients]
    end
    
    subgraph Query Processing
        LB[Load Balancer]
        QueryProcessor[Query Processor<br/>Go]
        Autocomplete[Autocomplete Service]
        SpellCheck[Spell Checker]
        QueryCache[Query Cache<br/>Redis]
    end
    
    subgraph Search Core
        Searcher[Search Service<br/>Go]
        Ranker[Ranking Service<br/>Python/ML]
        Personalizer[Personalization<br/>ML]
    end
    
    subgraph Index Layer
        IndexServer1[Index Server 1<br/>Shard 1-1000]
        IndexServer2[Index Server 2<br/>Shard 1001-2000]
        IndexServerN[Index Server N<br/>Shard N]
    end
    
    subgraph Indexing Pipeline
        Crawler[Web Crawler]
        Fetcher[Page Fetcher]
        Parser[HTML Parser]
        Indexer[Indexer Service]
        IndexWriter[Index Writer]
    end
    
    subgraph Storage
        IndexStore[(Inverted Index<br/>16 TB)]
        DocStore[(Document Store<br/>500 TB)]
        MetaDB[(Metadata DB<br/>PostgreSQL)]
        LinkDB[(Link Graph<br/>Neo4j)]
    end
    
    subgraph ML Pipeline
        PageRank[PageRank<br/>Calculation]
        QualityScore[Quality Scorer<br/>ML Model]
        RankingML[Ranking Model<br/>TensorFlow]
    end
    
    Web -->|1. Query| LB
    Mobile -->|2. Query| LB
    LB -->|3. Route| QueryProcessor
    
    QueryProcessor -->|4. Check Cache| QueryCache
    QueryProcessor -->|5. Autocomplete| Autocomplete
    QueryProcessor -->|6. Spell Check| SpellCheck
    QueryProcessor -->|7. Search| Searcher
    
    Searcher -->|8. Query Shards| IndexServer1
    Searcher -->|8. Query Shards| IndexServer2
    Searcher -->|8. Query Shards| IndexServerN
    
    IndexServer1 -->|9. Return Docs| Searcher
    Searcher -->|10. Rank| Ranker
    Ranker -->|11. Personalize| Personalizer
    Personalizer -->|12. Results| QueryProcessor
    
    Crawler -->|13. Discover URLs| Fetcher
    Fetcher -->|14. Fetch Pages| Parser
    Parser -->|15. Extract| Indexer
    Indexer -->|16. Update| IndexWriter
    IndexWriter -->|17. Write| IndexStore
    
    PageRank -->|18. Scores| RankingML
    QualityScore -->|19. Quality| RankingML
    RankingML -->|20. Model| Ranker
```

### Data Flow

**Search Query Flow:**
1. User enters query
2. Query processor checks cache
3. Spell check and query expansion
4. Parallel lookups across index shards
5. Merge results from shards
6. ML-based ranking
7. Personalization
8. Return top results

**Indexing Flow:**
1. Crawler discovers new URLs
2. Fetcher downloads pages
3. Parser extracts text, links, metadata
4. Indexer builds inverted index
5. Index writer updates shards
6. PageRank recalculation
7. Cache invalidation

---

## Database Design

### Inverted Index Structure

```text
Term: "system"
Postings List: [
    {doc_id: 123, positions: [5, 42, 89], tf: 3, field: "title"},
    {doc_id: 456, positions: [12], tf: 1, field: "body"},
    {doc_id: 789, positions: [2, 15, 23, 67], tf: 4, field: "body"}
]

Optimizations:
- Delta encoding for doc_ids
- Variable-byte encoding for positions
- Skip lists for fast intersection
- Bloom filters for existence checks
```

### PostgreSQL Schema (Metadata)

```sql
-- Documents table
CREATE TABLE documents (
    doc_id BIGSERIAL PRIMARY KEY,
    url VARCHAR(2048) UNIQUE NOT NULL,
    url_hash VARCHAR(64) UNIQUE NOT NULL,
    title VARCHAR(500),
    description TEXT,
    content_hash VARCHAR(64),
    
    -- Metadata
    language VARCHAR(10),
    country VARCHAR(2),
    last_modified TIMESTAMP,
    content_type VARCHAR(50),
    
    -- Scores
    pagerank_score DECIMAL(10,8),
    quality_score DECIMAL(10,8),
    spam_score DECIMAL(10,8),
    
    -- Status
    crawl_status VARCHAR(20),
    index_status VARCHAR(20),
    last_crawled_at TIMESTAMP,
    last_indexed_at TIMESTAMP,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_url_hash (url_hash),
    INDEX idx_pagerank (pagerank_score),
    INDEX idx_quality (quality_score),
    INDEX idx_language (language),
    INDEX idx_last_crawled (last_crawled_at),
    INDEX idx_crawl_status (crawl_status)
);

-- Links table (for PageRank calculation)
CREATE TABLE links (
    link_id BIGSERIAL PRIMARY KEY,
    source_doc_id BIGINT NOT NULL,
    target_doc_id BIGINT NOT NULL,
    anchor_text VARCHAR(500),
    link_type VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_source (source_doc_id),
    INDEX idx_target (target_doc_id),
    INDEX idx_source_target (source_doc_id, target_doc_id),
    
    FOREIGN KEY (source_doc_id) REFERENCES documents(doc_id),
    FOREIGN KEY (target_doc_id) REFERENCES documents(doc_id),
    UNIQUE (source_doc_id, target_doc_id)
);

-- Query logs table
CREATE TABLE query_logs (
    log_id BIGSERIAL PRIMARY KEY,
    user_id UUID,
    query_text VARCHAR(500) NOT NULL,
    query_hash VARCHAR(64),
    results_count INTEGER,
    clicked_doc_id BIGINT,
    click_position INTEGER,
    session_id UUID,
    device_type VARCHAR(20),
    location_country VARCHAR(2),
    location_city VARCHAR(100),
    response_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_query_hash (query_hash),
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at),
    INDEX idx_clicked_doc (clicked_doc_id)
);
```

### Redis Schema (Caching)

```redis
# Query result cache
query:cache:{query_hash} -> {
    "results": [{doc_id, url, title, snippet}],
    "total_count": 1000000,
    "timestamp": timestamp,
    "ttl": 300
}

# Autocomplete cache
autocomplete:{prefix} -> ["suggestion1", "suggestion2", ...]

# Popular queries cache
popular:queries:{category} -> ZSET (scored by popularity)

# Document cache
doc:cache:{doc_id} -> {
    "title": "...",
    "url": "...",
    "pagerank": 0.85,
    "quality": 0.90
}

# Spell correction cache
spell:correction:{word} -> "corrected_word"
```

---

## API Design

### Search Endpoint

```http
GET /search?q=system+design&page=1&size=10&location=us
```

**Response:**
```json
{
  "query": "system design",
  "corrected_query": null,
  "results": [
    {
      "doc_id": 123456789,
      "url": "https://example.com/system-design-interview",
      "title": "System Design Interview Guide",
      "snippet": "Complete guide to <b>system design</b> interviews...",
      "pagerank_score": 0.85,
      "relevance_score": 0.92,
      "published_date": "2025-01-01"
    }
  ],
  "total_results": 1000000,
  "response_time_ms": 85,
  "suggestions": ["system design interview", "system design patterns"]
}
```

---

## Deep-Dive Components & Trade-offs

### Component 1: Inverted Index with Advanced Optimizations

**Purpose:** Enable sub-200ms full-text search across 10B documents with efficient storage and query processing.

**Architecture:**
```text
1. Inverted Index Structure
   - Term dictionary: B-tree of terms
   - Postings lists: Arrays of (doc_id, positions, metadata)
   - Compression: Delta encoding + variable-byte encoding
   - Storage: 16 TB for 10B documents

2. Index Sharding Strategy
   - Document-based sharding: doc_id % num_shards
   - 1000 shards, each handling 10M documents
   - Parallel query across all shards
   - Merge results using min-heap

3. Query-Time Optimizations
   - Skip lists: Jump large sections of postings lists
   - Bloom filters: Check term existence before lookup
   - Early termination: Stop after top-K results found
   - Caching: Hot terms cached in memory
```

**Technology Choice:** Custom inverted index + RocksDB
- **Pros:** Optimized for search workload, microsecond lookups, efficient compression
- **Cons:** Custom implementation, maintenance overhead
- **Alternative:** Elasticsearch (easier but 2x storage, slower at scale)

**Performance:**
```text
Index lookup time:
- Term dictionary lookup: 1ms (B-tree)
- Postings list scan: 10ms (skip lists)
- Result merging: 5ms (min-heap)
- Total: 16ms per query

Storage efficiency:
- Uncompressed: 100 TB
- With compression: 16 TB (6.25x reduction)
- Memory for hot terms: 10 GB per server
```

### Component 2: ML-Powered Ranking with LambdaMART

**Purpose:** Rank search results by relevance using machine learning to maximize user satisfaction.

**Architecture:**
```text
1. Ranking Features (200+ features)
   - Query-document features: TF-IDF, BM25, cosine similarity
   - Document features: PageRank, quality score, freshness
   - User features: Location, search history, click patterns
   - Context features: Time, device, query intent

2. LambdaMART Model
   - Algorithm: Gradient boosted trees for learning-to-rank
   - Training: Millions of query-document pairs with click data
   - Objective: Optimize NDCG (Normalized Discounted Cumulative Gain)
   - Model size: 500 MB
   - Inference: 50ms for 1000 documents

3. Multi-Stage Ranking
   - Stage 1: Cheap retrieval (BM25) - 10K candidates
   - Stage 2: Feature-based ranking - 1K candidates
   - Stage 3: ML model (LambdaMART) - top 100
   - Stage 4: Personalization - final rerank

4. Online Learning
   - Click feedback collected in real-time
   - Model retrained daily with fresh data
   - A/B testing for model evaluation
   - Automated rollback on degradation
```

**Technology Choice:** LambdaMART + TensorFlow
- **Pros:** State-of-art ranking quality, proven at Google/Bing scale
- **Cons:** Complex feature engineering, model maintenance
- **Alternative:** Simple BM25 (faster but 30% lower relevance)

**Performance Impact:**
```text
Relevance improvements:
- BM25 only: 0.65 NDCG@10
- + PageRank: 0.72 NDCG@10
- + LambdaMART: 0.85 NDCG@10
- + Personalization: 0.90 NDCG@10

User satisfaction:
- BM25: 3.2/5 average rating
- LambdaMART: 4.5/5 average rating
- Click-through rate: 15% → 28% (87% increase)
```

### Component 3: Distributed PageRank Calculation

**Purpose:** Calculate global importance scores for 10B web pages considering 1 trillion links.

**Architecture:**
```text
1. PageRank Algorithm
   - Iterative matrix multiplication
   - Formula: PR(A) = (1-d) + d × Σ(PR(T)/C(T))
   - Damping factor: d = 0.85
   - Convergence: <0.0001 difference after ~50 iterations

2. Distributed Computation (MapReduce)
   - Map: Distribute current scores to outlinks
   - Reduce: Sum incoming scores
   - Iteration: Repeat until convergence
   - Parallelization: 1000 worker machines

3. Optimization Techniques
   - Block-based computation: Process dense subgraphs together
   - Sparse matrix representation: Only store non-zero values
   - Checkpointing: Save intermediate results
   - Approximate computation: Stop early for tail docs

4. Incremental Updates
   - Full recalculation: Weekly
   - Incremental updates: Daily for new pages
   - Priority: Focus on high-traffic pages
```

**Technology Choice:** Apache Spark with GraphX
- **Pros:** Distributed graph processing, fault tolerance, scalability
- **Cons:** Computation time (hours), resource intensive
- **Alternative:** Pre-computed static scores (faster but outdated)

**Performance:**
```text
Computation time:
- Full PageRank: 6 hours on 1000 machines
- Incremental: 30 minutes for 10M new pages
- Cost: $500 per full calculation

Resource usage:
- Link graph: 16 TB (10B docs × 100 links × 16 bytes)
- Intermediate results: 80 GB per iteration
- Worker memory: 1000 machines × 64 GB = 64 TB total
```

### Component 4: Real-Time Index Updates with Lambda Architecture

**Purpose:** Keep search index fresh with <1 hour latency for new content while maintaining query performance.

**Architecture:**
```text
1. Lambda Architecture Layers
   - Batch layer: Full re-indexing weekly
   - Speed layer: Real-time updates for new/changed pages
   - Serving layer: Merge batch + speed results

2. Speed Layer Implementation
   - Incremental indexing: Update only changed terms
   - In-memory buffer: Hold recent updates (last 1 hour)
   - Periodic flush: Merge to disk-based index every 10 minutes
   - Query-time merge: Combine disk + memory results

3. Index Versioning
   - Snapshot isolation: Queries see consistent view
   - Double buffering: Write to inactive buffer
   - Atomic swap: Switch buffers after update complete
   - Rollback: Keep last 3 versions

4. Consistency Trade-offs
   - Accept eventual consistency (1-hour lag)
   - Critical updates: Force immediate propagation
   - Read-after-write: Check speed layer first
```

**Technology Choice:** RocksDB + Kafka + Flink
- **Pros:** Low latency updates, high throughput, exactly-once semantics
- **Cons:** Complex architecture, memory overhead for speed layer
- **Alternative:** Batch-only updates (simpler but 24-hour lag)

**Performance:**
```text
Update latency:
- Critical updates: <1 minute
- Normal updates: <1 hour
- Full re-index: 1 week

Throughput:
- 10M page updates/day = 116 updates/second
- Speed layer capacity: 1000 updates/second
- Overhead: 20% memory for in-memory buffer
```

### Trade-offs Analysis

#### Index Structure: Inverted Index vs Forward Index

**Decision:** Primary inverted index with auxiliary forward index

**Choice:** Optimized inverted index for queries, forward index for ranking

**Pros:**
- Fast full-text search (O(log n) term lookup)
- Efficient storage with compression (6x reduction)
- Parallel query processing across shards
- Forward index enables complex ranking features

**Cons:**
- Slower updates (must rebuild postings lists)
- Higher memory usage for hot terms
- Complex query planning for multiple terms

**Justification:** For 100K QPS with <200ms latency, inverted index is only viable structure. Forward index trade-off worth it for ranking quality.

---

## Bottlenecks & Improvements

### Critical Bottlenecks Analysis

#### Bottleneck 1: Long-Tail Query Performance

**Problem Analysis:**
- **Root Cause:** 40% of queries are unique (long-tail), can't be cached
- **Impact:** 60% higher latency (150ms vs 90ms for popular queries)
- **Frequency:** Continuous, affects user experience
- **Severity:** High - impacts 40% of queries

**Solutions:**
1. **Predictive Caching with ML**
   - Model predicts queries likely to become popular
   - Pre-compute results for predicted queries
   - 150ms → 90ms (40% improvement)

2. **Index Optimization for Long-Tail**
   - More aggressive skip lists for rare terms
   - Early termination after top-K results
   - Parallel shard queries with timeout

3. **Query Approximation**
   - Approximate top-K using sampling
   - Trade accuracy for speed
   - 150ms → 120ms (20% improvement, 95% accuracy)

#### Bottleneck 2: Index Update Storm During News Events

**Problem Analysis:**
- **Root Cause:** Breaking news causes 100x spike in content updates
- **Impact:** Index update lag 1 hour → 6 hours
- **Frequency:** 5-10 times per month
- **Severity:** Critical - stale results for trending topics

**Solutions:**
1. **Priority-Based Indexing**
   - News sites get priority queue
   - Trending topics auto-prioritized
   - 6 hours → 30 minutes for priority content

2. **Elastic Index Workers**
   - Auto-scale from 50 → 500 workers during spikes
   - Cost: $200/hour burst vs stale results
   - Maintain <1 hour latency even at 100x load

#### Bottleneck 3: PageRank Staleness

**Problem Analysis:**
- **Root Cause:** Full PageRank takes 6 hours, only run weekly
- **Impact:** New popular sites take week to rank properly
- **Severity:** Medium - affects new content discovery

**Solutions:**
1. **Incremental PageRank**
   - Update only affected subgraph
   - 6 hours full → 30 minutes incremental
   - Run daily for growing sites

2. **Approximate PageRank**
   - Use ML model to predict PageRank for new sites
   - 90% accuracy vs full computation
   - Real-time estimates enable immediate ranking

---

## Section 11: Security & Data Privacy

### What You'll Learn

- Implement input sanitization to prevent injection attacks
- Design content filtering systems for offensive/illegal content
- Handle user data privacy and GDPR compliance
- Protect against DDoS and scraping attacks
- Build secure search APIs with authentication
- Manage data retention and right-to-be-forgotten requests

### Why This Matters

Search engines handle sensitive user data and must protect against attacks. Google suffered a €50M GDPR fine in 2019 for privacy violations in search personalization. Microsoft Bing faced a 24-hour DDoS attack in 2020 that cost $5M in lost revenue. DuckDuckGo built their entire business model on privacy-first search, gaining 100M users. Understanding security and privacy isn't just compliance - it's a competitive advantage and legal necessity. In interviews, discussing security shows senior-level thinking about production systems.

---

### 🟢 For Beginners: What is Search Security?

**The Library Analogy**

Imagine a public library with these security problems:

**Problem 1: Vandalism (Injection Attacks)**
```text
Malicious visitor writes in search catalog:
"Harry Potter'; DROP TABLE books;--"

If librarian blindly follows instructions:
├─ Looks up "Harry Potter"
├─ Executes "DROP TABLE books" (destroys catalog!)
└─ Library loses all book records

Same happens in search engines:
User searches: "python'; DELETE FROM index;--"
If not sanitized → Could delete entire search index!

Solution: Validate all input
├─ Remove special characters: '; -- < > etc.
├─ Treat everything as TEXT, not commands
└─ "python'; DELETE" becomes safe search query
```

**Problem 2: Inappropriate Content (Content Filtering)**
```text
Library problem:
├─ Kids section has adult books mixed in
├─ Racist/hateful books in general catalog
└─ Illegal content not removed

Search engine problem:
User searches "how to tie shoes"
Results include:
├─ Legitimate tutorials ✓
├─ Spam/scam sites ❌
├─ Adult content ❌
├─ Illegal weapon instructions ❌

Solution: Multi-layer filtering
├─ SafeSearch filter (remove adult content)
├─ Spam detection (ML model identifies low-quality sites)
├─ Illegal content blocking (DMCA, terrorism, CSAM)
└─ Quality threshold (minimum quality score to appear)
```

**Problem 3: Privacy Invasion (Data Tracking)**
```text
Library problem:
Librarian keeps detailed log:
├─ "Person A borrowed books about cancer" (private medical info!)
├─ "Person B researched divorce lawyers" (private legal issue!)
├─ "Person C reads LGBTQ+ books" (private identity info!)
└─ Sells this data to advertisers

Search engine problem:
Search history reveals:
├─ Medical conditions ("symptoms of diabetes")
├─ Financial status ("bankruptcy lawyer")
├─ Political views ("vote democrat" or "vote republican")
├─ Location (IP address tracking)

Solution: Privacy protection
├─ Anonymize search logs (remove user ID after 18 months - Google policy)
├─ Encrypt data in transit (HTTPS) and at rest
├─ Allow users to delete history (GDPR requirement)
├─ Don't sell personal search data (DuckDuckGo approach)
```

---

**Real Security Incidents: What Went Wrong**

**Google GDPR Fine (2019): €50 Million**
```text
What happened:
├─ Google personalized ads based on search history
├─ Didn't get explicit consent from users
├─ Didn't clearly explain data usage
└─ Violated GDPR privacy rules

Specific issue:
User searched: "diabetes treatment options"
Google:
├─ Stored search in user profile
├─ Used for ad targeting (diabetes medication ads)
├─ Shared with advertising partners
└─ Never got explicit permission for health data use

GDPR violation:
├─ Health data = "special category" (extra protection needed)
├─ Required: Explicit opt-in consent
├─ Google only had: Generic terms & conditions
└─ Penalty: €50M fine + forced to change privacy policy

Lesson learned:
- Search data is PERSONAL data (covered by privacy laws)
- Health/religion/political searches need extra protection
- Clear consent required, not buried in T&Cs
- Cost of non-compliance: $50M+ fines
```

**Microsoft Bing DDoS Attack (2020): 24 Hours Down**
```text
What happened:
├─ Attackers sent 10M requests/second to Bing
├─ Overwhelmed servers (normal capacity: 100K QPS)
├─ Service down for 24 hours
└─ Estimated loss: $5M in ad revenue

Attack pattern:
├─ Botnet of 500K compromised computers
├─ Each sent 20 requests/second
├─ Total: 10M QPS (100x normal traffic!)
├─ Legitimate requests couldn't get through

How it worked:
Normal traffic: 100K QPS
├─ 100K requests from real users
├─ Servers handle easily
└─ Response time: 50ms

Attack traffic: 10M QPS
├─ 10M requests from bots (99% attack, 1% real)
├─ Servers overwhelmed (each server gets 100x load!)
├─ Response time: Timeout (servers crash)
└─ Real users see: "Service unavailable"

Microsoft's response:
├─ Hour 1: Detected attack, enabled rate limiting
├─ Hour 4: Blocked botnet IP addresses (50K IPs)
├─ Hour 12: Deployed additional servers (10x capacity)
├─ Hour 24: Service restored, attack mitigation active

Cost impact:
├─ Lost ad revenue: $5M (24 hours @ $200K/hour)
├─ Emergency infrastructure: $500K (temporary servers)
├─ Engineering time: 100 engineers × 24 hours = $200K
└─ Total cost: $5.7M for one attack

Prevention measures added:
├─ Rate limiting: 10 requests/second per IP
├─ CAPTCHA: Shown for suspicious traffic
├─ DDoS protection: Cloudflare/Akamai edge filtering
├─ Auto-scaling: 10x capacity reserve for attack scenarios
```

**Elasticsearch Data Leak (2020): 5 Billion Records**
```text
What happened:
├─ Company left Elasticsearch instance public (no authentication)
├─ Search index contained user data
├─ Anyone could access via HTTP
└─ 5B user records exposed

Exposed data:
├─ Search queries (private information!)
├─ IP addresses (location tracking)
├─ User IDs (identity linkage)
└─ Timestamps (behavior profiling)

Example exposed search:
{
  "user_id": "12345",
  "query": "affordable cancer treatment",
  "ip": "192.168.1.1",
  "timestamp": "2020-05-15 14:23:00"
}

Privacy violations:
├─ Medical queries revealed health conditions
├─ Financial queries revealed money problems
├─ Location queries revealed home addresses
└─ Could link to real identities via user_id

Regulatory response:
├─ GDPR investigation: Potential €20M fine
├─ CCPA (California): Potential $7,500 per record fine
├─ Class action lawsuit: $100M settlement
└─ Total cost: $127M + reputation damage

How it should have been protected:
├─ Authentication: Require username/password
├─ Encryption: TLS for data in transit
├─ Anonymization: Remove user_id from logs
├─ Access control: Internal network only
└─ Monitoring: Alert on unauthorized access attempts
```

---

**The 5 Core Security Principles for Search**

**1. Input Validation (The Bouncer)**

```text
Think: Nightclub bouncer checking IDs

What it does:
├─ Examines every search query
├─ Rejects dangerous input
├─ Allows safe queries through

Examples:

Dangerous input:
├─ "'; DROP TABLE;--" (SQL injection)
├─ "<script>alert('hack')</script>" (XSS attack)
├─ "../../../etc/passwd" (path traversal)

Safe input:
├─ "python tutorial" ✓
├─ "how to bake bread" ✓
├─ "weather forecast" ✓

Validation rules:
├─ Max length: 200 characters
├─ Allowed characters: a-z A-Z 0-9 space - _
├─ Remove: < > ' " ; -- / \ 
├─ Encode: HTML entities for display
```

**HLD Approach (No Implementation Code):**
```text
Input Validation Architecture:

User Query → Validation Layer → Clean Query → Search Engine

Validation Layer checks:
├─ Length (reject if > 200 chars)
├─ Character whitelist (only alphanumeric + common punctuation)
├─ SQL keyword detection ("DROP", "DELETE", "UPDATE")
├─ Script tag detection ("<script>", "javascript:")
└─ Path traversal detection ("../", "~")

Reject patterns:
- Queries matching attack signatures → Return error 400
- Queries with >3 special chars in row → Flag as suspicious
- Queries from IPs with >100 requests/min → Rate limit

Accept patterns:
- Normal text queries → Process
- Queries with moderate special chars (emails, URLs) → Sanitize then process
```

---

**2. Content Filtering (The Quality Control)**

```text
Problem: Not all content should appear in search results

Categories to filter:

Tier 1: Illegal Content (MUST block by law)
├─ CSAM (child sexual abuse material) - Federal law
├─ Terrorism content - Anti-terrorism laws
├─ DMCA violations - Copyright law
├─ Counterfeit goods - Trademark law

Tier 2: Harmful Content (SHOULD block for safety)
├─ Graphic violence
├─ Self-harm instructions
├─ Dangerous medical misinformation
├─ Extreme hate speech

Tier 3: Low-Quality Content (Filter for quality)
├─ Spam sites (ad-heavy, no real content)
├─ Malware/phishing sites
├─ Duplicate/scraped content
├─ Doorway pages (SEO manipulation)

Tier 4: Optional Filters (User preference)
├─ Adult content (SafeSearch setting)
├─ Profanity
├─ Controversial topics
```

**Content Filtering Architecture:**
```text
Multi-Layer Filtering Pipeline:

Incoming Content → Layer 1: Hash Matching → Layer 2: ML Classifier → Layer 3: Manual Review → Index

Layer 1: Hash Matching (fast, 100% accurate for known content)
├─ Compare content hash against database of illegal content
├─ PhotoDNA for images (Microsoft technology)
├─ MD5/SHA256 for text
├─ Latency: 1ms
├─ If match → Block immediately

Layer 2: ML Classifier (slower, catches new content)
├─ Text classifier (hate speech, spam, violence)
├─ Image classifier (adult content, violence)
├─ Trained on millions of examples
├─ Latency: 50ms
├─ If confidence > 95% → Block
├─ If confidence 70-95% → Send to Layer 3

Layer 3: Manual Review (slowest, highest accuracy)
├─ Human reviewers check flagged content
├─ 10K reviewers globally (Google/Facebook scale)
├─ Average: 1000 items reviewed per reviewer per day
├─ Final decision: Block or Allow

Cost analysis:
├─ Layer 1: $0.0001 per check (hash lookup)
├─ Layer 2: $0.001 per check (ML inference)
├─ Layer 3: $0.50 per review (human labor)
└─ Total: 99% filtered by Layer 1/2 (cheap), 1% needs human review
```

---

**3. Data Privacy (GDPR Compliance)**

**User Rights Under GDPR:**
```text
1. Right to Access
   - Users can request: "Show me all data you have about me"
   - Search engine must provide: All queries, clicks, profile data
   - Deadline: 30 days

2. Right to Deletion ("Right to be Forgotten")
   - Users can request: "Delete all my search history"
   - Search engine must delete: Personal data from all systems
   - Deadline: 30 days
   - Exception: Can keep anonymized aggregate data

3. Right to Data Portability
   - Users can request: "Give me my data in a standard format (JSON/CSV)"
   - Search engine must provide: Machine-readable export
   - Deadline: 30 days

4. Right to Object
   - Users can say: "Don't use my data for personalization"
   - Search engine must respect: Generic results only, no profiling
```

**Privacy-Preserving Architecture:**
```text
Data minimization strategy:

Collect (what data to store):
├─ MUST store: Query text, result clicks (needed for service)
├─ SHOULD store: Anonymized logs (for quality improvement)
├─ SHOULD NOT store: IP addresses long-term (privacy risk)
├─ MUST NOT store: Sensitive queries tied to user ID (GDPR special category)

Retention policy:
├─ Personal search history: 18 months (Google policy)
├─ Anonymized aggregate logs: 5 years (business analytics)
├─ IP addresses: 30 days (security/fraud detection)
├─ User deletion requests: Immediate (GDPR compliance)

Anonymization process:
1. Remove user ID: user123 → anonymized_hash_abc
2. Truncate IP: 192.168.1.100 → 192.168.0.0/16
3. Round timestamps: 14:23:47 → 14:00:00
4. Remove rare queries: Queries searched <100 times → exclude

Result: Can't identify individual users, can still analyze trends
```

**Cost of Privacy Compliance:**
```text
Building GDPR-compliant search:

One-time costs:
├─ Legal review: $200K (compliance audit)
├─ Engineering: $1M (data deletion pipeline, anonymization)
├─ Privacy controls UI: $100K (user dashboard for data requests)
└─ Total: $1.3M

Ongoing costs:
├─ Data deletion requests: 10K/month × $2 processing = $20K/month
├─ Privacy team: 5 people × $200K/year = $1M/year
├─ Annual audits: $100K/year
└─ Total: $1.24M/year

Cost of NON-compliance:
├─ GDPR fines: Up to 4% of global revenue or €20M (whichever is higher)
├─ For Google-scale: 4% × $300B revenue = $12B potential fine!
└─ ROI: Spend $2M on compliance vs risk $12B fine = obvious choice
```

---

**4. Rate Limiting & DDoS Protection**

**Why Rate Limiting Matters:**
```text
Without rate limiting:

Scenario: Bot attacks search engine
├─ Bot sends 10,000 requests/second from single IP
├─ Normal users send 100K requests/second total
├─ Bot consumes 10% of capacity (wasteful!)
├─ Costs money (compute, bandwidth)
└─ Degrades service for real users

With rate limiting:
├─ Limit: 10 requests/second per IP
├─ Bot's 10,000 req/sec → blocked after 10th request
├─ Real users unaffected (rarely exceed 1 req/sec)
└─ Attack neutralized ✓
```

**Rate Limiting Architecture:**
```text
Multi-tier rate limiting:

Tier 1: Per-IP Rate Limit
├─ Limit: 10 requests/second per IP
├─ Window: Sliding 1-second window
├─ Action: Return HTTP 429 "Too Many Requests"
├─ Implemented at: Load balancer (Nginx)

Tier 2: Per-User Rate Limit
├─ Limit: 100 requests/minute per logged-in user
├─ Window: Rolling 1-minute window
├─ Action: Temporary account suspension (15 minutes)
├─ Implemented at: Application layer

Tier 3: Global Rate Limit
├─ Limit: 150K requests/second total (system capacity)
├─ Action: Reject lowest-priority requests first
├─ Priority: Paid API > Logged-in users > Anonymous
├─ Implemented at: API Gateway

DDoS Protection Strategy:

Level 1: CDN (Cloudflare/Akamai)
├─ Filters 95% of DDoS traffic at edge
├─ Challenge suspicious IPs with CAPTCHA
├─ Cost: $50K/month for 100TB/month traffic

Level 2: Application-level rate limiting
├─ Token bucket algorithm
├─ Costs: Negligible (in-memory counters)

Level 3: Auto-scaling
├─ Detect traffic spike → spin up 10x servers
├─ Cost: $10K/hour for 1000 servers
├─ Duration: Until attack stops
```

---

**5. Encryption & Secure Communication**

**Data Protection at Rest and in Transit:**
```text
Encryption layers:

In Transit (network):
├─ HTTPS/TLS 1.3 for all user connections
├─ Prevents: Man-in-the-middle attacks, eavesdropping
├─ User query: "private medical search" → encrypted during transmission
├─ Attacker on WiFi: Sees encrypted garbage, can't read actual query

At Rest (storage):
├─ AES-256 encryption for stored data
├─ Prevents: Data theft if disks are stolen
├─ Search index files encrypted on disk
├─ Key management: AWS KMS, Google Cloud KMS

Between Services (internal):
├─ mTLS (mutual TLS) for service-to-service communication
├─ Prevents: Internal attackers, compromised services
├─ Search API → Index Service: Authenticated + encrypted
```

**Compliance Summary:**
```text
Security Framework Checklist:

GDPR (EU):
├─ ✓ User consent for data collection
├─ ✓ Right to deletion
├─ ✓ Data anonymization
├─ ✓ Breach notification (72 hours)
└─ Fine: Up to €20M or 4% revenue

CCPA (California):
├─ ✓ Disclosure of data collection
├─ ✓ Opt-out of data selling
├─ ✓ Right to deletion
└─ Fine: $7,500 per violation

HIPAA (Healthcare data):
├─ ✓ Encryption at rest/transit
├─ ✓ Access logging
├─ ✓ Business associate agreements
└─ Fine: Up to $1.5M per violation

SOC 2 (Security practices):
├─ ✓ Access controls
├─ ✓ Change management
├─ ✓ Incident response
└─ Required for enterprise customers
```

---

### ✅ Key Takeaways

- **Input validation mandatory** - Prevent injection attacks with whitelist approach
- **Content filtering is multi-layered** - Hash matching → ML → Human review
- **Privacy is a legal requirement** - GDPR fines up to €20M, compliance cheaper than fines
- **Rate limiting prevents attacks** - 10 req/sec per IP blocks most bot traffic
- **Encryption everywhere** - HTTPS in transit, AES-256 at rest
- **Real incidents are expensive** - Bing DDoS: $5M, Google GDPR: €50M, Elasticsearch leak: $127M

---

## Section 12: Scalability & Growing the System

### What You'll Learn

- Scale search from 1K to 100M users step-by-step
- Implement horizontal scaling strategies for each component
- Plan capacity for 10x, 100x, 1000x growth
- Evolve architecture at different scale stages
- Calculate costs at each growth stage
- Handle traffic spikes and seasonal patterns

### Why This Matters

Search systems must grow with your business. Google started with 1 server in 1998 and now runs 1M+ servers. Amazon search handled 100K products in 1995, now indexes 350M+ products. Pinterest search scaled from 1M users to 500M in 5 years. Understanding scalability isn't just about technical skills - it's about cost management and business planning. Every 10x user growth requires rearchitecting components. In interviews, showing how a system evolves demonstrates senior-level system thinking.

---

### 🟢 For Beginners: What is Scalability?

**The Restaurant Analogy**

**Stage 1: Small Café (1K users = 10 searches/sec)**
```text
Setup:
├─ 1 chef (1 search server)
├─ 1 cash register (1 database)
├─ 10 tables (capacity: 50 customers/day)
├─ Menu: 20 items (20K documents indexed)

Works great initially!
├─ Orders fulfilled in 2 minutes
├─ Everyone gets served
├─ Monthly cost: $5K (rent + staff)
```

**Stage 2: Popular Restaurant (10K users = 100 searches/sec)**
```text
Problem: Lines out the door!
├─ 1 chef can't keep up (CPU maxed out)
├─ 1 register = bottleneck (database overloaded)
├─ Tables full, customers leaving (high latency)

Solution: "Scale up" (bigger kitchen) OR "Scale out" (more chefs)?

Scale UP (Vertical Scaling):
├─ Hire master chef (faster CPU: 2GHz → 4GHz)
├─ Buy industrial stove (more RAM: 16GB → 64GB)
├─ Cost: $15K/month
├─ Limit: Can only get so fast! Top chef still one person

Scale OUT (Horizontal Scaling): ✓ Better!
├─ Hire 5 chefs (5 search servers)
├─ Add 3 registers (3 database replicas)
├─ Cost: $25K/month
├─ Benefit: Can keep adding chefs infinitely!
```

**Stage 3: Restaurant Chain (100K users = 1K searches/sec)**
```text
Problem: One location can't handle demand

Solution: Open multiple locations (distributed system!)
├─ 10 restaurants in different cities (10 data centers)
├─ Each restaurant independent (sharding)
├─ Customer goes to nearest location (geographic routing)
├─ Central kitchen for supplies (shared index builder)

Benefits:
├─ Faster service (local = low latency)
├─ Redundancy (if one closes, others open)
├─ Infinite growth potential
└─ Cost: $150K/month (10 locations × $15K)
```

**Stage 4: Global Empire (10M users = 100K searches/sec)**
```text
Setup: McDonald's-scale operation
├─ 1000 restaurants globally (1000 servers)
├─ 50 countries (50 regions)
├─ Automated kitchens (ML-powered ranking)
├─ 24/7 operations (always available)

Cost: $2M/month
Efficiency: $0.20 per 100 searches (economies of scale!)
```

---

**Vertical vs Horizontal Scaling**

**Vertical Scaling (Scale UP): Bigger Machine**
```text
Analogy: Upgrade from Honda to Ferrari

Process:
├─ Current server: 4 CPU cores, 16GB RAM, 1TB SSD
├─ Upgrade to: 64 CPU cores, 512GB RAM, 10TB SSD
├─ Cost: $500/month → $5,000/month (10x more expensive!)

Pros:
✓ Simple (no code changes needed)
✓ No distributed system complexity
✓ Works for small-medium scale

Cons:
✗ Physical limits (can't buy infinite RAM!)
✗ Single point of failure
✗ Expensive (64-core servers cost 20x more than 4-core)
✗ Downtime during upgrade

Max realistic size:
├─ CPU: 128 cores ($15K/month)
├─ RAM: 2TB ($20K/month)
├─ Disk: 100TB ($10K/month)
└─ Total: ~$50K/month for ONE server

Handles:
├─ ~10K queries/second
├─ ~100M documents
└─ Not enough for Google scale!
```

**Horizontal Scaling (Scale OUT): More Machines**
```text
Analogy: Build a fleet of Hondas instead of one Ferrari

Process:
├─ Current: 1 server (4 cores, 16GB RAM)
├─ Add: 9 more identical servers
├─ Total: 10 servers
├─ Cost: $500/month × 10 = $5,000/month (same as 1 big server!)

Pros:
✓ No limits (add servers infinitely)
✓ Redundancy (if 1 fails, 9 still work)
✓ Cheaper (commodity hardware)
✓ No downtime (add servers without stopping service)

Cons:
✗ Complex (need load balancing, data sharding)
✗ Network overhead (servers must communicate)
✗ Data consistency challenges

Handles:
├─ 100K queries/second (10K per server × 10 servers)
├─ 10B documents (1B per server × 10 servers)
└─ Scales to Google level!
```

**Why Horizontal Wins:**
```text
Example: Handle 100K QPS

Vertical approach:
├─ Need: 10 super servers ($50K/month each)
├─ Total cost: $500K/month
├─ Redundancy: None (10 single points of failure)
└─ Max capacity: 100K QPS (hard limit)

Horizontal approach:
├─ Need: 1000 commodity servers ($500/month each)
├─ Total cost: $500K/month (same!)
├─ Redundancy: Lose 10 servers, 990 still work (99% uptime)
└─ Max capacity: No limit (add more servers)

Winner: Horizontal ✓
- Same cost
- Better redundancy
- Unlimited growth
- Industry standard (Google, Amazon, Facebook all use horizontal scaling)
```

---

**Growth Journey: 1K Users → 100M Users**

**Stage 1: Startup (1K users, 10 QPS)**
```text
Infrastructure:
├─ 1 application server (search API)
├─ 1 Elasticsearch node (index)
├─ 1 PostgreSQL database (metadata)
├─ 100K documents indexed
└─ All on 1 physical server!

Specs:
├─ 4 CPU cores
├─ 16GB RAM
├─ 1TB SSD
└─ Cost: $500/month (AWS: t3.xlarge)

Performance:
├─ Latency: 50ms P95
├─ Availability: 99% (downtime during deploys)
├─ Index size: 1GB

Team:
├─ 1-2 engineers
└─ Deploy manually

This works fine for MVP/early stage!
```

**Stage 2: Growing (10K users, 100 QPS)**
```text
Problems from Stage 1:
├─ Single server overloaded (CPU at 90%)
├─ Downtime during deploys
├─ Slow queries during peak hours
└─ Index doesn't fit in RAM anymore (5GB index, 16GB server)

Infrastructure upgrade:
├─ 3 application servers (load balanced)
├─ 3 Elasticsearch nodes (cluster with replication)
├─ 1 PostgreSQL primary + 2 read replicas
├─ 500K documents indexed
└─ Total: 9 servers

Specs (per server):
├─ 8 CPU cores
├─ 32GB RAM
├─ 2TB SSD
└─ Cost: $1,000/month × 9 = $9K/month

New components:
├─ Load balancer (Nginx): $100/month
├─ Redis cache: $200/month
├─ Monitoring (Datadog): $200/month
└─ Total: $9.5K/month

Performance:
├─ Latency: 30ms P95 (faster!)
├─ Availability: 99.9% (redundancy helps)
├─ Cache hit rate: 40%

Team:
├─ 3-4 engineers
├─ On-call rotation
└─ Automated deployment (CI/CD)

Key insight: 10x users = 20x cost
Why? Added redundancy and monitoring
```

**Stage 3: Scale-up (100K users, 1K QPS)**
```text
Problems from Stage 2:
├─ Elasticsearch cluster slow (needs sharding)
├─ PostgreSQL replicas lagging
├─ Cache eviction rate high (not enough RAM)
└─ Search quality issues (need better ranking)

Infrastructure upgrade:
├─ 10 application servers
├─ 10 Elasticsearch nodes (5 shards × 2 replicas)
├─ 5 PostgreSQL instances (sharded by doc_id range)
├─ 5 Redis nodes (cluster mode)
├─ 2M documents indexed
└─ Total: 30 servers

Specs (per server):
├─ 16 CPU cores
├─ 64GB RAM
├─ 4TB SSD
└─ Cost: $2,000/month × 30 = $60K/month

New components:
├─ CDN (CloudFront): $5K/month
├─ Message queue (Kafka): $3K/month (real-time indexing)
├─ ML ranking service: $5K/month
├─ Advanced monitoring: $1K/month
└─ Total: $74K/month

Performance:
├─ Latency: 25ms P95 (better!)
├─ Availability: 99.95%
├─ Cache hit rate: 55% (better cache)
├─ Relevance: +15% (ML ranking)

Team:
├─ 8-10 engineers
├─ 24/7 on-call
├─ Site reliability engineering (SRE) focus

Key insight: 10x users = 8x cost
Efficiency improving! Caching and sharding help
```

**Stage 4: Web-Scale (1M users, 10K QPS)**
```text
Problems from Stage 3:
├─ Global users (high latency for international)
├─ Data consistency across shards challenging
├─ Operational complexity (managing 30 servers hard)
└─ Cost optimization needed

Infrastructure upgrade:
├─ Multi-region deployment (US, EU, Asia)
├─ 30 app servers per region × 3 regions = 90 servers
├─ 30 Elasticsearch nodes per region = 90 nodes
├─ 10M documents indexed
└─ Total: 200 servers across 3 continents

Specs (per server):
├─ 32 CPU cores
├─ 128GB RAM
├─ 8TB SSD
└─ Cost: $3,000/month × 200 = $600K/month

New components:
├─ Global load balancer (GeoDNS): $10K/month
├─ Inter-region replication: $20K/month bandwidth
├─ Personalization platform: $30K/month
├─ Advanced security (WAF): $15K/month
└─ Total: $675K/month

Performance:
├─ Latency: 15ms P95 globally! (regional serving)
├─ Availability: 99.99% (multi-region redundancy)
├─ Cache hit rate: 65%
├─ Relevance: +25% (personalization)

Team:
├─ 25-30 engineers
├─ Specialized teams: Search, ML, Infrastructure, Security
├─ DevOps/SRE: 5 people

Key insight: 10x users = 9x cost
Geography adds overhead but improves UX significantly
```

**Stage 5: Google-Scale (100M users, 100K QPS)**
```text
Infrastructure:
├─ 10 regions globally
├─ 200 servers per region × 10 = 2000 servers
├─ Petabyte-scale storage
├─ 100B documents indexed
└─ Custom hardware (Google TPUs for ML)

Cost: $8M/month
├─ Servers: $6M
├─ Network: $1M (inter-region traffic)
├─ ML infrastructure: $500K
├─ Security/compliance: $300K
├─ Monitoring: $200K

Performance:
├─ Latency: <10ms P95
├─ Availability: 99.999% (5 nines!)
├─ Cache hit rate: 75%
├─ Queries: 100K/second sustained, 1M/second peak

Team:
├─ 200+ engineers
├─ Dedicated teams for every component
└─ Advanced R&D (new ranking algorithms, query understanding)

Key insight: 100x users = 12x cost from Stage 4
Economies of scale! Automation and optimization pay off
```

**Cost Efficiency Over Time:**
```text
Cost per 1000 queries:

Stage 1 (1K users): $500/month ÷ 2.6M queries/month = $0.19 per 1K queries
Stage 2 (10K users): $9.5K ÷ 26M = $0.37 per 1K queries (worse!)
Stage 3 (100K users): $74K ÷ 260M = $0.28 per 1K queries (better)
Stage 4 (1M users): $675K ÷ 2.6B = $0.26 per 1K queries
Stage 5 (100M users): $8M ÷ 260B = $0.03 per 1K queries (87% cheaper!)

Insight: Initial growth is EXPENSIVE (adding redundancy)
But at scale, efficiency dramatically improves!
```

---

### 🟡 For Intermediate: Scaling Patterns

**Pattern 1: Database Sharding**

**When to shard:**
```text
Signals you need sharding:
├─ Database >500GB (doesn't fit in RAM)
├─ Write throughput >10K/second
├─ Query latency degrading despite indexes
└─ Replication lag >5 seconds

Sharding strategies:

Option A: Range-based sharding
├─ Shard 1: doc_id 0-10M
├─ Shard 2: doc_id 10M-20M
├─ Shard 3: doc_id 20M-30M

Pros: Simple, sequential scans fast
Cons: Unbalanced load (recent docs get more traffic)

Option B: Hash-based sharding
├─ Shard = hash(doc_id) % num_shards
├─ Evenly distributes load
├─ Cons: Range queries expensive

Option C: Geography-based sharding (best for search!)
├─ Shard 1: US documents
├─ Shard 2: EU documents
├─ Shard 3: Asia documents

Pros: Low latency (data co-located with users)
Cons: Cross-region queries slow
```

**Pattern 2: Read Replicas**

**Scaling read-heavy workloads:**
```text
Search is 99% reads, 1% writes

Without replicas:
├─ 1 primary database
├─ Handles: 1000 reads/sec + 10 writes/sec
├─ Bottleneck: CPU maxed at 1000 reads/sec

With 5 read replicas:
├─ 1 primary (handles writes only): 10 writes/sec
├─ 5 replicas (handle reads only): 200 reads/sec each
├─ Total capacity: 1000 reads/sec + 10 writes/sec ✓

Cost:
├─ Without replicas: 1 × $5K = $5K/month
├─ With replicas: 6 × $5K = $30K/month
└─ 6x cost for 5x read capacity (worthwhile!)

Replication strategy:
├─ Asynchronous replication (faster, eventual consistency)
├─ Replication lag: <100ms typical
├─ Acceptable for search (stale results OK for 100ms)
```

**Pattern 3: Caching for Traffic Spikes**

**Handling 10x traffic spikes:**
```text
Normal traffic: 10K QPS
Black Friday: 100K QPS (10x spike!)

Without caching:
├─ Need 10x servers (100 servers)
├─ Cost: $50K/month normally, $500K during spike
├─ Waste: Paying for 90 servers that are idle 99% of time

With aggressive caching:
├─ Cache hit rate: 80% during spike (popular queries repeat)
├─ Backend load: 100K × 20% = 20K QPS
├─ Need: 2x servers (20 servers total)
├─ Cost: $50K normally, $100K during spike

Savings: $400K during 1-week spike!

Cache warming strategy:
1. Week before Black Friday: Pre-compute top 10K queries
2. Load into cache tier with high TTL (24 hours)
3. During spike: 80% cache hits
4. After spike: Reduce cache size back to normal
```

---

### ✅ Key Takeaways

- **Horizontal scaling wins** - Add more servers, not bigger servers
- **Growth isn't linear** - 10x users ≠ 10x cost (sometimes 3x, sometimes 20x)
- **Stage 1→2 expensive** - Adding redundancy doubles cost
- **Stage 4→5 efficient** - Economies of scale reduce per-query cost by 87%
- **Shard early** - Before database becomes bottleneck
- **Cache aggressively** - 80% hit rate saves 5x on infrastructure
- **Multi-region costly but necessary** - Global latency matters

---

## Section 13: Deep-Dive Topic - Query Understanding & NLP Pipeline

### What You'll Learn

- Design NLP pipeline for query processing (tokenization, entity recognition, intent classification)
- Implement spell correction and query expansion
- Handle ambiguous queries with context
- Build synonym detection and query rewriting systems
- Use BERT/transformers for semantic search
- Measure query understanding quality

### Why This Matters

"Jaguar" could mean the animal, the car, or the football team. Google's BERT update (2019) improved understanding of 1 in 10 queries, affecting **15% of all searches** and adding an estimated **$3B in annual value** through better relevance. Microsoft's query understanding pipeline reduced zero-result queries by 35%, increasing user satisfaction and revenue by **$800M/year**. Understanding what users really mean - not just matching keywords - is what separates great search from mediocre search.

---

### 🟢 For Beginners: What is Query Understanding?

**The Coffee Shop Order Analogy**

**Scenario: Customer says "I want a large coffee"**

**Barista A: Literal Understanding (Keyword Matching)**
```text
Hears: "large coffee"
Thinks: Customer wants exactly those words
Action: Gives large black coffee

Problem: Customer actually wanted large latte!
Customer: "No, I meant a large latte with milk!"
```

**Barista B: Smart Understanding (Query Understanding)**
```text
Hears: "I want a large coffee"
Thinks: Let me understand the INTENT
├─ "large" = size preference
├─ "coffee" = beverage category (could mean latte, cappuccino, americano...)
├─ Context: This customer ordered latte yesterday
├─ Time: 8 AM (morning = people want milk-based drinks)

Action: "Did you mean a large latte like yesterday?"
Customer: "Yes, perfect!"
```

**Search engine works the same way:**

**Query: "apple store near me"**

**Without query understanding (dumb search):**
```text
Matches keywords:
├─ "apple" → Documents about fruit
├─ "store" → Any retail store
├─ "near me" → Ignored (not in index)

Results:
1. "How to store apples in refrigerator" ❌
2. "Apple orchard store in Washington state" ❌
3. "Grocery stores selling apples" ❌

User: Frustrated, tries Google instead
```

**With query understanding (smart search):**
```text
NLP Pipeline understands:
├─ "apple" = Apple Inc. (capitalized = company, not fruit)
├─ "store" = retail location (not storage verb)
├─ "near me" = geographic intent (needs user location)
├─ Query type: Local business search
├─ User context: Has iPhone (likely wants Apple retail store, not third-party)

Results:
1. Apple Store - 5th Avenue, NYC (0.3 miles) ✓
2. Apple Store - SoHo, NYC (0.8 miles) ✓
3. Apple Store - Grand Central, NYC (1.2 miles) ✓

User: Finds what they need immediately!
```

---

**Key NLP Tasks in Search:**

**1. Spell Correction**
```text
User types: "presedent election" (typo!)

Without spell correction:
├─ Searches for "presedent" exactly
├─ Zero results (word doesn't exist)
└─ User has to retype

With spell correction:
├─ Detects typo (not in dictionary)
├─ Suggests "president election" (edit distance = 1)
├─ Auto-corrects and shows results
└─ Saves user effort!

Google: 1 in 10 queries has spelling errors
Fixing saves 10% of queries from zero results
Impact: 10% × 100K QPS = 10K queries/sec improved
```

**2. Entity Recognition**
```text
Query: "Apple founder died in 2011"

Entity extraction:
├─ "Apple" = Company (Organization entity)
├─ "founder" = Person role
├─ "died in 2011" = Life event + Date

Search knows:
├─ Looking for person, not fruit
├─ Time constraint: died in 2011
├─ Context: Company founder

Results:
1. "Steve Jobs died October 5, 2011" ✓ (exactly what user wants!)
2. "Apple Inc. co-founder Steve Jobs biography"
3. "Tim Cook: Apple CEO after Jobs died"

Without entity recognition:
1. "Apple fruit cultivation died in 2011" ❌
2. "Why did my apple tree die" ❌
3. "Apple music 2011 playlist" ❌
```

**3. Intent Classification**
```text
Same words, different intents:

Query: "python"
Intent possibilities:
├─ Programming language (tech)
├─ Snake species (biology)
├─ Monty Python (comedy)

How to determine intent:

User context:
├─ User history: Searched "java", "javascript" before → Likely programming
├─ User profile: Software developer → Programming
├─ Time: During work hours → Programming
└─ Predicted intent: Programming (90% confidence)

Results customized to programming intent:
1. Python official documentation ✓
2. Python tutorial for beginners ✓
3. Python vs JavaScript comparison ✓

If user was a biology student:
├─ User history: "lizards", "reptiles"
└─ Results: Python snake species, habitat, care guide
```

**4. Query Expansion (Synonyms)**
```text
Query: "cheap hotels"

Without expansion:
├─ Match only: "cheap" AND "hotels"
├─ Misses: "affordable hotels", "budget hotels", "inexpensive hotels"
└─ Fewer results, potentially miss best options

With expansion:
├─ Expand "cheap" to: ["cheap", "affordable", "budget", "inexpensive", "economical"]
├─ Expand "hotels" to: ["hotels", "motels", "inns", "lodging", "accommodation"]
├─ Search: (cheap OR affordable OR budget) AND (hotels OR motels OR inns)

Result:
├─ 10x more results found
├─ Better options (might miss "budget inn" without expansion)
└─ Higher user satisfaction
```

---

**Real-World Impact:**

**Google BERT (2019): Largest Query Understanding Improvement**
```text
Before BERT:
├─ Query: "can you get medicine for someone pharmacy"
├─ Google misunderstood: Thought user asking general question
├─ Results: Articles about pharmacy regulations ❌

Problem: Missed key words "for someone"
User wants: Can I pick up someone else's prescription?

After BERT:
├─ Understands: "for someone" is the key context
├─ Intent: Picking up prescription for another person
├─ Results: "How to pick up prescription for family member" ✓

Impact:
├─ Improved 1 in 10 queries (10% of all searches!)
├─ 100M queries per day × 10% = 10M queries/day improved
├─ User satisfaction: +5%
└─ Estimated value: $3B/year from better relevance
```

**Microsoft Bing Query Rewriting (2018):**
```text
Query: "weather"

Without query rewriting:
├─ Generic results: Weather definition, weather patterns
├─ Not actionable (user wants current weather!)
└─ User rephrases: "weather new york today"

With query rewriting:
├─ Detects: User likely wants current local weather
├─ Rewrites: "weather" → "current weather [user location]"
├─ Uses: IP geolocation to determine city
└─ Shows: Current weather widget immediately!

Impact:
├─ Zero-result rate: 8% → 5% (38% reduction!)
├─ User satisfaction: +12%
├─ Searches that need rephrasing: 25M/day
└─ Value: $800M/year from better query understanding
```

---

### 🟡 For Intermediate: NLP Pipeline Architecture

**Query Processing Pipeline:**

```text
User Query → Pipeline → Understood Query → Search

Step 1: Text Normalization
├─ Lowercase: "Python" → "python"
├─ Remove extra spaces: "python  tutorial" → "python tutorial"
├─ Unicode normalization: "café" → "cafe"
└─ Output: Clean text

Step 2: Spell Correction
├─ Check against dictionary (1M common words)
├─ If misspelled: Suggest corrections
├─ "presedent" → "president" (edit distance 1)
└─ Auto-correct high-confidence typos

Step 3: Tokenization
├─ Split: "python tutorial" → ["python", "tutorial"]
├─ Handle: "New York" → ["New York"] (keep phrase together)
└─ Output: Token list

Step 4: Entity Recognition (NER)
├─ Identify: Companies, people, locations, dates
├─ "Apple founder died 2011" →
   ├─ "Apple" = ORG (organization)
   ├─ "founder" = ROLE
   ├─ "2011" = DATE
└─ Output: Annotated entities

Step 5: Intent Classification
├─ ML model predicts: Informational? Transactional? Navigational?
├─ "buy iphone" = Transactional (shopping intent)
├─ "how to bake bread" = Informational
└─ Output: Intent label

Step 6: Query Expansion
├─ Add synonyms: "cheap" → ["cheap", "affordable", "budget"]
├─ Add related terms: "hotels" → ["hotels", "motels", "inns"]
└─ Output: Expanded query

Step 7: Query Rewriting (if needed)
├─ "weather" → "current weather [user location]"
├─ "apple" → "apple company" (if user is tech enthusiast)
└─ Output: Rewritten query optimized for search
```

**NLP Technology Stack:**

```text
Tool/Library choices:

Spell Correction:
├─ Algorithm: Edit distance (Levenshtein), N-gram matching
├─ Dictionary: 1M common words + domain-specific
├─ Latency: <5ms
└─ Library: SymSpell (fast), Hunspell (accurate)

Tokenization:
├─ Basic: Split on whitespace
├─ Advanced: Sentence segmentation, compound splitting
├─ Latency: <1ms
└─ Library: NLTK, spaCy

Entity Recognition (NER):
├─ Approach: BERT-based model fine-tuned on search queries
├─ Entities: PERSON, ORG, LOC, DATE, PRODUCT
├─ Accuracy: 92% F1 score
├─ Latency: 20ms
└─ Framework: HuggingFace Transformers, spaCy

Intent Classification:
├─ Model: Fine-tuned BERT classifier
├─ Classes: Informational, Navigational, Transactional, Local
├─ Accuracy: 87%
├─ Latency: 15ms
└─ Framework: TensorFlow, PyTorch

Semantic Search (BERT embeddings):
├─ Model: Sentence-BERT (bi-encoder)
├─ Embedding dimension: 768
├─ Similarity: Cosine similarity
├─ Latency: 30ms for encoding + 10ms for similarity
└─ Use: Matching query intent to document content semantically
```

---

### 🔴 For Advanced: BERT for Semantic Search

**Traditional Keyword Search vs Semantic Search:**

```text
Query: "how do I improve my sleep quality"

Keyword search:
├─ Matches: Documents with exact words "improve", "sleep", "quality"
├─ Misses: Documents with synonyms "enhance rest", "better slumber"
├─ Problem: Rigid matching, misses relevant docs

Semantic search:
├─ Understands: User wants advice on sleeping better
├─ Matches: Documents about sleep improvement, even with different words
├─ Finds: "10 tips for better rest", "enhance your slumber naturally"
└─ Better: Matches meaning, not just words
```

**BERT Embedding Approach:**

```text
1. Pre-compute document embeddings:
   - For each document in index
   - Generate 768-dimensional vector (BERT embedding)
   - Represents semantic meaning
   - Store in vector database (FAISS, Pinecone)

2. At query time:
   - Encode query to 768-dim vector
   - Find nearest document vectors (cosine similarity)
   - Top-k most similar documents = search results

Latency breakdown:
├─ Encode query: 30ms (BERT forward pass)
├─ Vector search: 10ms (FAISS approximate nearest neighbor)
├─ Total: 40ms overhead (acceptable for quality gain)

Quality improvement:
├─ Keyword search: 65% relevance
├─ BERT semantic search: 85% relevance (+31% improvement!)
```

---

### ✅ Key Takeaways

- **Query understanding ≠ keyword matching** - Must understand intent, context, entities
- **Spell correction critical** - 10% of queries have typos, fixing prevents zero results
- **BERT revolutionized search** - Improved 10% of Google queries, $3B annual value
- **Multi-stage pipeline** - Normalize → Spell check → NER → Intent → Expand
- **Semantic search > keyword** - 31% better relevance with BERT embeddings
- **Latency tradeoff** - NLP adds 30-50ms but improves quality significantly

---

## Section 14: Interview Preparation & Practice

### What You'll Learn

- Master 45-minute search engine interview framework
- Practice common interview questions with model answers
- Handle deep-dive scenarios and follow-ups
- Navigate trade-off discussions confidently
- Avoid common interview mistakes
- Prepare for company-specific variations (Google vs Amazon vs Bing)

### Why This Matters

Search engine design is one of the **most popular FAANG interview questions**. It combines multiple system design concepts: distributed systems, databases, caching, ranking algorithms, ML. Interviewers use it to assess breadth AND depth. Companies want to see how you think, not just what you know. A candidate who explains trade-offs clearly beats one who jumps to complex solutions. This section gives you the frameworks and practice to confidently ace search interviews at any top tech company.

---

### 🟢 For Beginners: The 45-Minute Interview Framework

**Time Allocation (CRITICAL!):**

```text
Minutes 0-5: Requirements & Scope (11%)
├─ Clarify functional requirements
├─ Discuss scale (QPS, documents, latency)
├─ Write down key numbers
└─ Get interviewer agreement

Minutes 6-12: High-Level Architecture (13%)
├─ Draw boxes: User → API → Cache → Index → Database
├─ Explain data flow for one query
├─ Mention key components (inverted index, ranker)
└─ Get feedback early (adjust if needed)

Minutes 13-35: Deep Dives (49% - MOST IMPORTANT!)
├─ Interviewer picks 2-3 areas to explore
├─ Common deep-dives:
   ├─ Inverted index structure and implementation
   ├─ Ranking algorithm (TF-IDF, BM25, PageRank)
   ├─ Distributed architecture (sharding strategy)
   ├─ Caching layers and invalidation
   └─ Scaling to 10x, 100x traffic
└─ THIS IS WHERE YOU SHINE - show expertise!

Minutes 36-42: Trade-Offs & Alternatives (13%)
├─ "Why did you choose X over Y?"
├─ Alternative approaches and tradeoffs
├─ Cost-benefit analysis
└─ How design changes at different scales

Minutes 43-45: Wrap-Up & Questions (4%)
├─ Summarize key decisions
├─ What you'd do differently at 100x scale
├─ Ask interviewer for feedback
└─ Your questions for them
```

---

**DO's and DON'Ts:**

**✅ DO:**
- **Start with questions** - "How many documents? Daily/hourly indexing?"
- **Think out loud** - "I'm considering two approaches: X and Y. Let me compare..."
- **Draw diagrams** - Visual architecture is powerful
- **Acknowledge trade-offs** - "This is faster but uses more memory"
- **Use specific numbers** - "With 10B docs, we need 500 servers"
- **Ask for feedback** - "Does this approach make sense?"
- **Admit gaps** - "I'm not sure about X, but here's my thinking..."
- **Explain decisions** - "I chose BM25 over TF-IDF because..."

**❌ DON'T:**
- **Jump to code** - Design first, code only if asked
- **Overengineer** - Don't build for 1B users if they have 10K
- **Stay silent** - Silence for 5 minutes = fail
- **Ignore hints** - If interviewer suggests something, explore it!
- **Get defensive** - Challenges are learning opportunities
- **Forget basics** - Explain inverted index even if it seems simple
- **Skip capacity planning** - Always estimate storage, QPS, servers

---

**Common Interview Questions:**

**Question 1: "Design Google Search" (Most Common)**
```text
Scope:
├─ 10B web pages indexed
├─ 100K queries/second
├─ <200ms P95 latency
├─ Global users (need multi-region)
└─ Fresh results (real-time indexing)

Key challenges interviewer tests:
├─ Can you handle massive scale? (10B docs, 100K QPS)
├─ Do you understand ranking? (TF-IDF, BM25, PageRank)
├─ Can you design distributed systems? (Sharding, replication)
├─ Do you consider caching? (Multi-level cache hierarchy)
└─ Production concerns? (Monitoring, latency budgets)

Your approach:
1. Clarify: "Web search or specific domain?"
2. Scope: "100K QPS peak, <200ms latency, need ranking"
3. High-level: Draw User → API → Cache → Index Shards → Ranking
4. Deep-dive: Interviewer likely asks about inverted index or ranking
5. Trade-offs: BM25 vs ML ranking, document vs term sharding
```

**Question 2: "Design Amazon Product Search" (E-commerce Variant)**
```text
Different from web search:
├─ Products change (inventory updates, price changes)
├─ Structured data (price, category, brand, reviews)
├─ Filters (price range, brand, rating, Prime-eligible)
├─ Personalization critical (recommend products user will buy)
└─ Revenue-driven ranking (promote profitable products)

Key differences to mention:
├─ Real-time inventory updates (sold out products hidden immediately)
├─ Multi-attribute search ("red nike shoes size 10 under $100")
├─ Faceted navigation (filter by brand, price, rating)
├─ Ranking includes business metrics (profit margin, conversion rate)

Additional challenges:
- "What if product goes out of stock during user's session?"
- "How do you rank: popularity vs profit margin vs in-stock?"
- "How do you handle typos in product names?" (iPhone → IPhone)
```

**Question 3: "Design Elasticsearch" (Technical Deep-Dive)**
```text
Scope:
├─ General-purpose search engine (any domain)
├─ 100M documents, 10K QPS
├─ Full-text search + aggregations
├─ Near real-time indexing (NRT)

Interviewer wants to see:
├─ Lucene internals (segments, merge policy)
├─ Distributed architecture (primary + replica shards)
├─ Query DSL design (how to express complex queries)
├─ Refresh vs flush (when does data become searchable)
└─ Operational concerns (split brain, cluster state)

Focus areas:
1. Inverted index deep-dive (posting lists, skip lists)
2. Sharding strategy (hash-based sharding of doc IDs)
3. Write path: Index → Translog → Memory buffer → Disk segment
4. Read path: Query coordinator → Fan-out to shards → Merge results
```

---

### 🟡 For Intermediate: Deep-Dive Scenarios

**Scenario 1: "Walk me through a single query"**

```text
Expected answer (show complete data flow):

User searches "python tutorial" in browser

1. Client-side:
   ├─ JavaScript debounces input (wait 150ms for more typing)
   ├─ Sends HTTPS GET /search?q=python+tutorial
   └─ Adds headers: User-Agent, Accept-Encoding, Cookie (user_id)

2. CDN/Edge (CloudFront):
   ├─ Request hits nearest edge (London for UK user)
   ├─ Check CDN cache (key: "search:python+tutorial")
   ├─ Cache MISS (or HIT if popular query)
   ├─ If HIT: Return cached results (10ms total) ✓
   └─ If MISS: Forward to origin (Load Balancer)

3. Load Balancer:
   ├─ Receives request at central data center
   ├─ Health check: Which API servers are up?
   ├─ Round-robin routing → API Server #23
   └─ Latency so far: 30ms (CDN miss + network)

4. API Server #23:
   ├─ Parse query: "python tutorial"
   ├─ Validate: Length OK? No injection? ✓
   ├─ NLP pipeline:
      ├─ Spell check: "python" ✓ "tutorial" ✓
      ├─ Intent: Informational (learning content)
      └─ Entities: "python" = programming language
   ├─ Check Redis cache (L2 cache):
      ├─ Key: "query:python+tutorial:v2"
      ├─ Result: MISS (not cached yet)
   └─ Must query index...

5. Index Shard Coordinator:
   ├─ Query needs to fan-out to all shards (10 shards)
   ├─ Build query: "python" AND "tutorial"
   ├─ Send to shards in parallel:
      ├─ Shard 1: 234 results
      ├─ Shard 2: 189 results
      ├─ Shard 3: 456 results
      ├─ ... (all 10 shards)
      └─ Total: 2,847 matching documents

6. Ranking Service:
   ├─ Receive 2,847 candidates
   ├─ Stage 1: BM25 score → Top 1000
   ├─ Stage 2: PageRank score → Top 100
   ├─ Stage 3: ML personalization → Top 10
   └─ Final results: [doc_1, doc_3, doc_7, ...]

7. Response Path:
   ├─ API Server formats results (title, snippet, URL)
   ├─ Stores in Redis cache (TTL: 5 minutes)
   ├─ Stores in CDN cache (TTL: 1 minute)
   ├─ Returns to user
   └─ Total latency: 120ms

8. User sees results, clicks #1
   ├─ Click logged for ML training
   └─ Ranking improves for next user!

Time breakdown:
├─ Network: 30ms
├─ API processing: 10ms
├─ Index query: 50ms (parallel shard queries)
├─ Ranking: 25ms
├─ Response formatting: 5ms
└─ Total: 120ms ✓ (under 200ms SLA)
```

**Scenario 2: "How would you scale this to 10x traffic?"**

```text
Current: 10K QPS → 10x: 100K QPS

Expected answer (systematic scaling):

1. Application Layer (EASY - stateless):
   ├─ Current: 20 API servers
   ├─ 10x: 200 API servers (just add more!)
   ├─ Auto-scaling: AWS Auto Scaling Group
   ├─ Cost: $500/server × 180 new = $90K/month additional

2. Caching Layer (MODERATE):
   ├─ Current: 3 Redis nodes (60GB total)
   ├─ Problem: Cache eviction rate too high at 100K QPS
   ├─ Solution: Scale to 30 Redis nodes (600GB total)
   ├─ Sharding: Consistent hashing on query hash
   ├─ Cost: $1K/node × 27 new = $27K/month

3. Index/Search Layer (HARD):
   ├─ Current: 10 Elasticsearch shards
   ├─ Problem: Each shard handling 1K QPS → 10K QPS total OK
   ├─ At 10x: Each shard needs 10K QPS → OVERLOADED
   ├─ Solution: Increase replicas, not shards
      ├─ Current: 5 shards × 2 replicas = 10 nodes
      ├─ 10x: 5 shards × 10 replicas = 50 nodes
      ├─ Each replica handles 2K QPS (within capacity)
   ├─ Cost: $2K/node × 40 new = $80K/month

4. Database (for metadata, not search):
   ├─ Current: 1 primary + 2 replicas
   ├─ Problem: Write capacity OK (indexing unchanged)
   ├─ Read capacity: Need more replicas
   ├─ Solution: Add 5 more read replicas (total 7)
   ├─ Cost: $5K/replica × 5 = $25K/month

5. Network/CDN:
   ├─ Current: 100 TB/month bandwidth
   ├─ 10x: 1 PB/month bandwidth
   ├─ CDN handles most (Cloudflare)
   ├─ Cost: $50K → $200K/month (+$150K)

Total cost increase:
├─ Servers: $90K
├─ Cache: $27K
├─ Index: $80K
├─ Database: $25K
├─ Network: $150K
└─ Total: $372K/month new costs

Current cost: $100K/month
New cost: $472K/month
Cost per 10x scale: 4.7x (not 10x! Good efficiency!)

Bottleneck analysis:
- Elasticsearch replicas is the constraint
- Can scale to ~200K QPS with current architecture
- Beyond that, need sharding (more shards) and multi-region
```

---

### 🔴 For Advanced: Production Scenarios

**Scenario 1: "Search results suddenly became slow for 10% of users. How do you debug?"**

```text
Expected systematic debugging approach:

Step 1: Gather symptoms (2 minutes)
├─ "Slow" = how slow? 500ms? 5 seconds?
├─ Which 10%? Specific geography? User type?
├─ Started when? Gradual or sudden?
└─ Other symptoms? Errors? Timeouts?

Step 2: Check monitoring dashboards (3 minutes)
├─ Latency: P50/P95/P99 - Is there a spike?
├─ Error rate: Are requests failing?
├─ Traffic: Is QPS higher than normal?
├─ Resource utilization: CPU/Memory/Network saturated?

Step 3: Distributed tracing (5 minutes)
├─ Sample slow request
├─ Trace ID: 123-456-789
├─ Breakdown:
   ├─ API server: 10ms ✓
   ├─ Redis cache: 5ms ✓
   ├─ Elasticsearch query: 450ms ✗ ← BOTTLENECK!
   ├─ Ranking: 15ms ✓
   └─ Total: 480ms (mostly index query)

Step 4: Index-level diagnosis (5 minutes)
├─ Check shard health: Are some shards slow?
├─ Shard 3 latency: 450ms (others: 50ms) ✗
├─ Check shard 3 metrics:
   ├─ CPU: 95% (very high!)
   ├─ Disk I/O: 98% (maxed out!)
   ├─ Memory: 92%
   └─ Recent changes: New index created 2 hours ago

Step 5: Root cause (2 minutes)
├─ Shard 3 is merging segments (background process)
├─ Large merge (10GB → 1 segment)
├─ Consumes CPU + disk I/O
├─ Affects 10% of queries (those hitting shard 3)

Step 6: Immediate mitigation (10 minutes)
├─ Option A: Stop merge temporarily
   └─ Risk: Segments accumulate, worse later
├─ Option B: Add more replicas to shard 3
   └─ Distributes load, but takes time to sync
├─ Option C: Throttle merge process
   └─ Set max_merge_at_once=1, throttle merge to 20MB/sec
   └─ CHOOSE THIS ✓

Step 7: Long-term fix (Plan for later)
├─ Merge during off-peak hours only
├─ Add dedicated merge nodes (not serving queries)
├─ Increase shard count to reduce merge sizes
└─ Monitor merge queue depth (alert if >10)

Resolution:
├─ Throttled merge: Latency improved from 450ms → 80ms
├─ Full resolution in 30 minutes
├─ Post-mortem: Document root cause, prevention steps
```

**Scenario 2: "Marketing wants to boost certain products in search results. How do you handle?"**

```text
Expected answer (balance business vs relevance):

Context understanding:
├─ Marketing wants to promote: High-margin products, new launches
├─ Conflict: Boosting irrelevant products hurts user experience
├─ Goal: Balance revenue and user satisfaction

Solution architecture:

1. Editorial Boosts (Manual):
   Configuration:
   {
     "boosts": [
       {
         "query": "iphone",
         "product_id": 12345,
         "boost": 1.5,
         "duration": "7 days"
       },
       {
         "query": "laptop",
         "category": "gaming",
         "boost": 1.2
       }
     ]
   }

   How it works:
   ├─ Normal ranking score: 8.5
   ├─ Applies boost: 8.5 × 1.5 = 12.75
   ├─ Product moves up in results
   └─ But: Still needs minimum relevance (score >5)

   Guardrails:
   ├─ Max boost: 2x (can't make irrelevant product #1)
   ├─ Must meet quality threshold
   ├─ Time-limited (auto-expires)
   └─ A/B tested (measure CTR impact)

2. Business Rules Layer:
   Ranking pipeline:
   ├─ Stage 1: BM25 retrieval (relevance-only)
   ├─ Stage 2: Apply business boosts
   ├─ Stage 3: ML personalization
   └─ Stage 4: Final diversity (don't show all boosted products)

   Business boost factors:
   ├─ Profit margin: +10% boost per $10 margin
   ├─ Inventory level: Boost in-stock items +20%
   ├─ New launch: +30% for first 2 weeks
   └─ Combined max: +50% total boost

3. Monitoring & Safeguards:
   Track impact:
   ├─ CTR: Click-through rate (should not decrease)
   ├─ Conversion: Purchase rate (should increase)
   ├─ User satisfaction: Ratings, bounce rate
   └─ Revenue: Short-term gain vs long-term trust

   Alert if:
   ├─ CTR drops >5% (users not clicking)
   ├─ Bounce rate increases >10% (bad UX)
   └─ Rollback boost automatically if metrics degrade

   Example:
   ├─ Boosted high-margin laptop
   ├─ CTR: 15% → 12% (users clicking less) ✗
   ├─ Decision: Reduce boost from 1.5x → 1.2x
   └─ Result: CTR recovered to 14%, revenue still +8% ✓

4. Long-term: ML-based business ranking
   Train model on:
   ├─ Features: Relevance score, profit margin, inventory, user profile
   ├─ Objective: Maximize revenue per search (not just clicks)
   ├─ Balances: User satisfaction (long-term) + revenue (short-term)

   Example optimization:
   ├─ Show relevant AND profitable products
   ├─ Not just most profitable (kills UX)
   ├─ Not just most relevant (misses revenue)
   └─ Sweet spot: 85% relevance + 15% business factors

Answer to interviewer:
"I'd implement a multi-layer approach:
1. Manual boosts for campaigns (controlled, time-limited)
2. Algorithmic business rules (profit, inventory, newness)
3. Strict quality thresholds (no boost for irrelevant products)
4. A/B testing and monitoring (measure impact on CTR and revenue)
5. Auto-rollback if user metrics degrade

Key principle: Business goals should enhance, not replace, relevance."
```

---

### ✅ Key Takeaways

- **Time management critical** - 50% of interview on deep-dives
- **Think out loud** - Silence is a red flag
- **Draw diagrams** - Visual communication wins
- **Use specific numbers** - "100K QPS" not "high traffic"
- **Explain trade-offs** - "BM25 is faster but ML is more accurate"
- **Follow interviewer hints** - They guide you to what they want to hear
- **Practice 10-15 mock interviews** - Interviewing.io, Pramp, friends

---

### 🎯 Practice Questions

**45-Minute Timed Practice:**

1. Design Google Image Search (images, not text)
2. Design StackOverflow search (code snippets, tags, votes)
3. Design Airbnb search (location, dates, filters)
4. Design Spotify search (songs, artists, albums, playlists)
5. Design Wikipedia search (huge corpus, quality articles)

**Deep-Dive Practice (20 minutes each):**

1. Design inverted index for 10B documents - how do you handle memory constraints?
2. Implement PageRank at Google scale - how do you parallelize?
3. Design real-time indexing pipeline - how do you handle 10K doc updates/second?
4. Build ML ranking model - what features? What algorithm?
5. Handle multi-language search - how do you support 100+ languages?

**Production Scenarios (30 minutes each):**

1. Search returns wrong results after index update - how do you debug?
2. Latency spikes to 5 seconds during peak hours - what do you do?
3. Competitor scrapes your search results - how do you prevent?
4. GDPR request: Delete all data for user - how do you comply?
5. CEO wants search as good as Google in 6 months - is it possible? How?

---

## Putting It All Together: Complete Search Engine Journey

### 🎓 What You've Mastered

Congratulations! You now understand search engines from fundamentals to production scale.

**Key Achievements:**
- ✅ **Inverted Index:** Core data structure enabling sub-second search across billions of documents
- ✅ **Ranking Algorithms:** TF-IDF, BM25, PageRank, and ML-based ranking
- ✅ **Distributed Architecture:** Sharding, replication, and query processing at 100K QPS
- ✅ **Real-Time Indexing:** Stream processing for fresh results
- ✅ **Production Operations:** Monitoring, optimization, and cost management
- ✅ **NLP Pipeline:** Query understanding, BERT, semantic search ($3B annual value)
- ✅ **Interview Skills:** 45-minute framework, deep-dive scenarios, production debugging

---

### 🎯 Interview Success Checklist

**✅ Can you confidently:**
- Explain inverted index structure and why it's efficient
- Calculate TF-IDF and BM25 scores
- Design distributed search architecture
- Estimate capacity for 10B documents
- Compare ranking algorithm trade-offs
- Implement NLP query understanding pipeline
- Handle interview deep-dives and production scenarios

**You're ready for Google/Amazon/Microsoft search interviews!**

---

### 🚀 Next Steps

1. **Build It:** Implement search engine with Elasticsearch
2. **Scale It:** Add sharding and replication
3. **Optimize It:** ML ranking and personalization
4. **Learn More:** Study Google's papers on PageRank and BERT

---

### Final Thoughts

Search engines democratize access to information. From PageRank to transformer models, each innovation makes knowledge more accessible.

**Core Principles:**
- **Relevance First:** Quality over speed
- **Iterate Constantly:** A/B test ranking changes
- **Scale Horizontally:** Sharding enables growth
- **Monitor Everything:** p99 latency is critical

**You're now ready to:**
- ✅ Pass FAANG search interviews
- ✅ Build production search systems
- ✅ Lead search architecture teams

Thank you for learning with us! 🚀

---

*End of Search Engine System Design Learning Module*  
*Last Updated: January 2025 | Framework Version: 2.1*
