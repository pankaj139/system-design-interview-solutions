# System Design TODO Tracker

**File Purpose:** This file tracks the progress of all system design problems in this repository. Use this as your working checklist to plan, track, and complete designs systematically.

**Last Updated:** October 1, 2025

**Progress:** 5/52 (9.6% Complete)

---

## 📊 Progress Overview

```
✅ Completed: 5
🚧 In Progress: 0
📝 Planned: 47
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[███░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░] 9.6%
```

---

## ✅ Completed Designs

### 1. ✅ LeetCode System Design
- **Status:** Complete
- **File:** `leetcode_system_design.md`
- **Completed:** October 1, 2025
- **Scale:** 100K DAU, 500K submissions/day
- **Key Features:** Sandboxed code execution, async job processing, multi-language support
- **Lines:** 2,117

### 2. ✅ Google Photos System Design
- **Status:** Complete
- **File:** `google_photos_system_design.md`
- **Completed:** [Date TBD]
- **Scale:** Multi-million users, petabytes of storage
- **Key Features:** Distributed storage, image processing pipeline, cross-platform sync
- **Lines:** 3,215

### 3. ✅ URL Shortener (TinyURL) System Design
- **Status:** Complete
- **File:** `url_shortener_system_design.md`
- **Completed:** October 1, 2025
- **Scale:** 100M DAU, 1M URL creations/day, 100M redirects/day
- **Key Features:** Snowflake ID generation, multi-tier caching, analytics pipeline, geographic distribution
- **Lines:** 2,200+ (comprehensive)

### 4. ✅ Distributed Cache (Redis/Memcached) System Design
- **Status:** Complete
- **File:** `distributed_cache_system_design.md`
- **Completed:** October 1, 2025
- **Scale:** 1M ops/sec starting, 10M+ ops/sec peak, 100M+ keys
- **Key Features:** Consistent hashing, master-replica replication, multiple data types, optional persistence (RDB + AOF), pub/sub messaging, memory management with eviction policies
- **Lines:** 2,800+ (comprehensive)

### 5. ✅ Rate Limiter for API Gateway System Design
- **Status:** Complete
- **File:** `rate_limiter_system_design.md`
- **Completed:** October 1, 2025
- **Scale:** 10M requests/day, 348 QPS peak (3x burst), 100K API keys
- **Key Features:** Sliding window counter algorithm, multi-region deployment with eventual consistency, three-tier pricing support, graceful degradation, real-time rate limit status, comprehensive algorithm comparison (Token Bucket, Fixed Window, Sliding Log, Leaky Bucket, Sliding Window Counter)
- **Lines:** 1,900+ (comprehensive)

---

## 📋 Pending Designs (47)

### 🔥 High Priority - Core Infrastructure (Next 3)

#### 6. ⬜ Content Delivery Network (CDN)
- **Status:** Not Started
- **Priority:** P0 - Critical
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Edge servers, cache hierarchies, origin pull, DNS routing
- **Common Follow-ups:** Cache invalidation, geographic distribution, cost optimization
- **Notes:** Complex but extremely common

#### 7. ⬜ Distributed Key-Value Store
- **Status:** Not Started
- **Priority:** P0 - Critical
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Consistent hashing, vector clocks, gossip protocol, quorum
- **Common Follow-ups:** CAP theorem, conflict resolution, replication strategies
- **Notes:** Tests deep understanding of distributed systems

---

### 💬 Messaging & Real-Time Systems

#### 8. ⬜ Chat Application (WhatsApp)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** WebSocket, message queue, read receipts, group chat, end-to-end encryption

#### 9. ⬜ Pub/Sub Messaging System (Kafka)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Topics, partitions, consumer groups, offset management, durability

#### 10. ⬜ Notification System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2-3 hours
- **Key Concepts:** Push notifications, FCM/APNs, fan-out, priority queues, delivery guarantees

#### 11. ⬜ Distributed Stream Processing System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Stream processing, windowing, stateful operations, exactly-once semantics

---

### 📱 Social Media & Content Platforms

#### 12. ⬜ Social Media Platform (Instagram/Twitter)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Feed generation, follower/following, media storage, timeline algorithms

#### 13. ⬜ Newsfeed System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Fan-out on write vs read, ranking algorithms, personalization, caching

#### 14. ⬜ Video Streaming Service (Netflix/YouTube)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Adaptive bitrate, CDN, transcoding, content delivery, recommendations

#### 15. ⬜ Find Friends on Social Network
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Graph algorithms, recommendation systems, friend suggestions

---

### 🔍 Search & Discovery

#### 16. ⬜ Web Crawler
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** BFS/DFS, URL frontier, politeness, deduplication, distributed crawling

#### 17. ⬜ Search Engine
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Inverted index, ranking algorithms, crawling, indexing, query processing

#### 18. ⬜ Autocomplete/Typeahead System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2-3 hours
- **Key Concepts:** Trie data structure, caching, ranking, prefix matching

#### 19. ⬜ Proximity Service (Yelp)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Geohashing, QuadTree, R-tree, spatial indexing, nearby search

#### 20. ⬜ Google Maps
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Graph algorithms, Dijkstra, A*, road network, ETA, routing

#### 21. ⬜ Recommendation Engine
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Collaborative filtering, content-based, matrix factorization, ML pipelines

#### 22. ⬜ Book Review Aggregator
- **Status:** Not Started
- **Priority:** P3 - Low
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2-3 hours
- **Key Concepts:** Data aggregation, API integration, deduplication, ranking

---

### 🛒 E-commerce & Payments

#### 23. ⬜ E-commerce Website (Amazon)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Product catalog, inventory, cart, checkout, orders, payments, recommendations

#### 24. ⬜ Payment Gateway
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Transaction processing, idempotency, reconciliation, fraud detection, PCI compliance

#### 25. ⬜ Shopping Cart System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Session management, inventory reservation, cart persistence, consistency

#### 26. ⬜ Ticket Booking System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Seat locking, concurrency control, payment integration, inventory management

#### 27. ⬜ Warehouse Inventory System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Stock management, location tracking, FIFO/LIFO, replenishment, analytics

#### 28. ⬜ Promotion/Cashback System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Rule engine, eligibility checks, fraud prevention, budget management

---

### 🚗 On-Demand Services

#### 29. ⬜ Ride-Sharing Service (Uber)
- **Status:** Not Started
- **Priority:** P0 - Critical
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Geolocation, matching algorithm, real-time tracking, ETA, pricing, dispatch

#### 30. ⬜ Food Delivery System (Zomato/Swiggy)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Restaurant catalog, order management, delivery routing, real-time tracking

#### 31. ⬜ Restaurant Registration System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Queue management, wait time estimation, notifications, capacity planning

#### 32. ⬜ Shortest Delivery Routes
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Traveling salesman, route optimization, real-time traffic, multi-stop routing

---

### 🎮 Gaming & Entertainment

#### 33. ⬜ Online Multiplayer Game Backend
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4 hours
- **Key Concepts:** Real-time sync, state management, matchmaking, lag compensation, cheat detection

#### 34. ⬜ Blackjack/Poker Gaming Site
- **Status:** Not Started
- **Priority:** P3 - Low
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Game state, fairness, random number generation, fraud prevention

---

### 🔐 Security & Authentication

#### 35. ⬜ Single Sign-On (SSO)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** OAuth2, SAML, JWT, identity provider, session management, security

#### 36. ⬜ Global Authentication Service
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Multi-region auth, token management, session replication, security

#### 37. ⬜ Firewall System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Packet filtering, rule engine, DPI, logging, threat detection

#### 38. ⬜ Buffer Overflow Prevention
- **Status:** Not Started
- **Priority:** P3 - Low
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 1-2 hours
- **Key Concepts:** Stack protection, ASLR, DEP, input validation, sanitization

---

### 📊 Monitoring & Observability

#### 39. ⬜ Distributed Metrics Logging System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3-4 hours
- **Key Concepts:** Log aggregation, time-series DB, query language, retention, indexing

#### 40. ⬜ Health Monitoring System for Servers
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Metrics collection, alerting, dashboards, anomaly detection, distributed tracing

#### 41. ⬜ Performance Metrics Collection at Scale
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** Sampling, aggregation, time-series storage, visualization, downsampling

---

### 📦 Storage & File Systems

#### 42. ⬜ Text Storage Service (Pastebin)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2-3 hours
- **Key Concepts:** Object storage, URL generation, expiration, access control, syntax highlighting

#### 43. ⬜ File Storage Service (Dropbox/Google Drive)
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Chunking, deduplication, sync algorithm, versioning, conflict resolution

#### 44. ⬜ Distributed Storage System
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4-5 hours
- **Key Concepts:** Replication, sharding, consistency, erasure coding, HDFS-like architecture

#### 45. ⬜ In-Memory Database
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4 hours
- **Key Concepts:** Memory management, persistence, indexing, concurrency, recovery

---

### 🏢 Enterprise & Real-World Systems

#### 46. ⬜ Parking Lot System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Capacity management, pricing, payment, availability tracking, OOP design

#### 47. ⬜ Online Voting/Election System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4 hours
- **Key Concepts:** Security, anonymity, verification, auditing, fraud prevention, consensus

#### 48. ⬜ Elevator System
- **Status:** Not Started
- **Priority:** P2 - Medium
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Scheduling algorithms, optimization, state machine, OOP design

#### 49. ⬜ Smart Elevator (Optimization)
- **Status:** Not Started
- **Priority:** P3 - Low
- **Difficulty:** ⭐⭐⭐ Hard
- **Estimated Time:** 3 hours
- **Key Concepts:** ML optimization, grouping, prediction, real-time scheduling

---

### 🏗️ Architecture & Patterns

#### 50. ⬜ Microservices Architecture
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐⭐⭐ Very Hard
- **Estimated Time:** 4 hours
- **Key Concepts:** Service boundaries, communication, service discovery, API gateway, resilience

#### 51. ⬜ Monolithic vs Microservices Comparison
- **Status:** Not Started
- **Priority:** P1 - High
- **Difficulty:** ⭐⭐ Medium
- **Estimated Time:** 2 hours
- **Key Concepts:** Trade-offs, migration strategies, when to use what, pros/cons analysis

---

## 📅 Suggested Learning Path

### Phase 1: Foundations (Problems 3-7)
**Goal:** Master core infrastructure patterns
**Timeline:** 2-3 weeks
- URL Shortener
- Distributed Cache
- Rate Limiter
- CDN
- Distributed Key-Value Store

### Phase 2: Scalable Applications (Problems 8-14)
**Goal:** Build complex user-facing systems
**Timeline:** 3-4 weeks
- Chat Application
- Pub/Sub System
- Social Media Platform
- Newsfeed System
- Video Streaming

### Phase 3: Search & Discovery (Problems 16-22)
**Goal:** Master search and recommendation systems
**Timeline:** 3-4 weeks
- Web Crawler
- Search Engine
- Autocomplete
- Proximity Service
- Recommendation Engine

### Phase 4: E-commerce & Transactions (Problems 23-28)
**Goal:** Handle complex transactions and consistency
**Timeline:** 3-4 weeks
- E-commerce Website
- Payment Gateway
- Shopping Cart
- Ticket Booking

### Phase 5: Real-time & On-Demand (Problems 29-32)
**Goal:** Build location-aware real-time systems
**Timeline:** 2-3 weeks
- Ride-Sharing (Uber)
- Food Delivery
- Shortest Routes

### Phase 6: Infrastructure & Operations (Problems 35-46)
**Goal:** Deep dive into infrastructure and monitoring
**Timeline:** 4-5 weeks
- SSO & Authentication
- Monitoring Systems
- File Storage
- Distributed Storage

### Phase 7: Architecture Patterns (Problems 51-52)
**Goal:** Understand architectural trade-offs
**Timeline:** 1 week
- Microservices Architecture
- Monolithic vs Microservices

---

## 🎯 Weekly Goals Template

### Week 1
- [ ] Complete 1 P0 design
- [ ] Review and iterate on 1 previous design
- [ ] Study related engineering blogs

### Week 2
- [ ] Complete 1 P1 design
- [ ] Practice explaining 2 designs verbally
- [ ] Compare alternative approaches

---

## 📝 Design Completion Checklist

Before marking a design as complete, ensure:

- [ ] All 5 framework sections completed
- [ ] Mermaid diagram included and renders correctly
- [ ] Database schema with indexes defined
- [ ] At least 10 API endpoints documented
- [ ] 5+ trade-offs analyzed
- [ ] 3+ bottlenecks identified with solutions
- [ ] Security considerations covered
- [ ] Monitoring strategy defined
- [ ] Back-of-envelope calculations done
- [ ] Document is 1500+ lines (comprehensive)

---

## 🔄 Update Instructions

**When starting a design:**
1. Change status from ⬜ to 🚧
2. Update "In Progress" count
3. Add start date

**When completing a design:**
1. Change status from 🚧 to ✅
2. Move to "Completed Designs" section
3. Update progress percentage
4. Add completion date and file name
5. Update "Next Up" in README.md

**When postponing a design:**
1. Add reason in notes
2. Adjust priority if needed
3. Select next design

---

## 💡 Tips for Success

1. **Time-box your work:** Don't spend more than estimated time on first draft
2. **Iterate:** First version doesn't need to be perfect
3. **Practice explaining:** Verbal explanation is key in interviews
4. **Review regularly:** Revisit old designs monthly
5. **Focus on trade-offs:** Understanding "why" matters more than "what"
6. **Use real examples:** Reference actual tech company blogs
7. **Draw diagrams:** Visual thinking helps solidify understanding
8. **Question assumptions:** Always clarify requirements first

---

## 🎯 Current Focus

**Active Design:** None

**Next Up:** Content Delivery Network (CDN)

**Target Completion:** [Set date]

---

**Last Activity:** October 1, 2025 - Completed Rate Limiter for API Gateway System Design

**Next Milestone:** Complete 5 P0 designs (Phase 1) - 3/5 done

**Target:** Complete all 52 designs by [Set target date]

