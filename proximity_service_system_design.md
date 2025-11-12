<!--
File Purpose: Interactive, multi-level learning resource for designing a Proximity Service (like Yelp or Google Maps). 
This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to 
build a location-based search system that handles 100M businesses globally, serves 50K queries per second with <100ms 
response time, supports complex geospatial queries within configurable radius (1-50km), handles high-density urban 
areas with 10K+ businesses per km², and achieves 99.99% availability across multiple regions.

Author: System Design Documentation
Created: January 2, 2025
Last Updated: January 2, 2025
Recent Updates: Complete transformation to educational template with three learning levels (Beginner, Intermediate, 
Advanced), integrated interview questions per section, detailed explanations without heavy code focus, and 
comprehensive real-world examples from Yelp, Google Maps, and Foursquare.
-->

# Proximity Service System Design (Yelp/Google Maps)

**Difficulty Level:** ⭐⭐⭐⭐ Very Hard  
**Tags:** `Geospatial Search`, `Geohash`, `QuadTree`, `R-tree`, `Location-based Services`, `Radius Search`, `Spatial Indexing`, `Multi-criteria Filtering`, `Caching`, `Horizontal Scaling`, `PostGIS`, `Distributed Systems`

**File Purpose:** Interactive, multi-level learning resource for designing a Proximity Service like Yelp or Google Maps. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a location-based search system that handles 100M businesses globally, serves 50K queries per second with <100ms response time, supports radius searches within 50km, handles high-density urban areas with 10K+ businesses per km², and achieves 99.99% availability (52 minutes of downtime per year).

**Author:** System Design Documentation  
**Created:** January 2, 2025  
**Last Updated:** January 2, 2025  
**Recent Updates:** Complete transformation to educational template with beginner/intermediate/advanced levels and integrated interview questions

---

## 🎓 Welcome to Proximity Service System Design!

### What You're Going to Build

Imagine creating a system where someone in New York City can instantly find the best pizza place within walking distance, see real-time reviews, check if it's open right now, and get directions - all in under 100 milliseconds. Or picture a business owner in Tokyo updating their restaurant hours at 9 PM, and within seconds, millions of hungry diners worldwide see the updated information in their searches. That's the power of a well-designed proximity service!

By the end of this learning journey, you'll understand how to design a production-grade proximity service that:
- **Handles massive scale**: Index 100M businesses globally and serve 50K queries per second
- **Delivers blazing speed**: <100ms response time for proximity searches within configurable radius (1-50km)
- **Manages complex geography**: Support high-density urban areas with 10K+ businesses per km² without performance degradation
- **Provides rich filtering**: Multi-criteria search by category, rating, price, hours, and features
- **Maintains high availability**: 99.99% uptime (only 52 minutes of downtime per year) with geographic redundancy

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (5-7 hours)
├─ Learn fundamental concepts (what is geospatial indexing?)
├─ Understand WHY we make design choices (why Geohash over simple coordinates?)
├─ Build intuition with everyday analogies (spatial search like organizing a library)
└─ Perfect for: New to system design or location-based services

🟡 INTERMEDIATE LEVEL (7-10 hours)  
├─ Master interview techniques (how to approach proximity service questions)
├─ Learn trade-off analysis (Geohash vs QuadTree vs S2, caching vs consistency)
├─ Practice common interview questions (50+ questions with answer frameworks)
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-15 hours)
├─ Production considerations (handling 785K businesses in Manhattan 5km radius)
├─ Performance optimization techniques (multi-tier caching, QuadTree for density)
├─ Handle edge cases and failures (cross-border searches, index inconsistency)
└─ Perfect for: Senior engineers and architects building real systems
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of databases (tables, indexes, queries)
- Familiarity with web APIs (GET/POST requests, JSON responses)
- Basic geography knowledge (latitude, longitude, distance)
- No prior system design experience needed!

**For Intermediate:**
- Understanding of distributed systems basics (load balancing, caching, replication)
- Familiarity with SQL and NoSQL databases
- Basic knowledge of search engines (Elasticsearch concepts)
- Experience with REST API design

**For Advanced:**
- Deep understanding of distributed systems (CAP theorem, consistency models)
- Production experience with databases at scale
- Knowledge of geospatial algorithms (spatial indexing, distance calculations)
- Understanding of performance optimization and profiling

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:
1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context
3. **Multi-Level Content** - Tailored explanations for your level
4. **Real-World Examples** - How Yelp, Google Maps, and Foursquare actually do it
5. **Think About It** - Questions to deepen understanding
6. **Interview Questions** - Practice what you just learned
7. **Key Takeaways** - Summary of main points
8. **Practice Exercise** - Hands-on challenge

💡 **Pro Tip:** Don't skip the "Think About It" sections - they're designed to help you internalize concepts so you can explain them in interviews or to your team!

---

## TABLE OF CONTENTS

- [Section 1: Understanding What We're Building](#section-1-understanding-what-were-building)
- [Section 2: Planning for Scale (Capacity Estimation)](#section-2-planning-for-scale-capacity-estimation)
- [Section 3: Geospatial Indexing Strategies](#section-3-geospatial-indexing-strategies)
- [Section 4: System Architecture Design](#section-4-system-architecture-design)
- [Section 5: Database Design](#section-5-database-design)
- [Section 6: API Design](#section-6-api-design)
- [Section 7: Proximity Search Algorithm](#section-7-proximity-search-algorithm)
- [Section 8: Ranking and Personalization](#section-8-ranking-and-personalization)
- [Section 9: Caching Strategy](#section-9-caching-strategy)
- [Section 10: High-Density Area Optimization](#section-10-high-density-area-optimization)
- [Section 11: Growing the System (Scalability)](#section-11-growing-the-system-scalability)
- [Section 12: Protecting the System (Security)](#section-12-protecting-the-system-security)
- [Section 13: Keeping It Healthy (Monitoring)](#section-13-keeping-it-healthy-monitoring)
- [Section 14: Making Design Decisions](#section-14-making-design-decisions)
- [Section 15: Interview Preparation & Practice](#section-15-interview-preparation--practice)
- [Putting It All Together](#putting-it-all-together)
- [Resources & Next Steps](#resources--next-steps)

---

## Section 1: Understanding What We're Building

### What You'll Learn

By the end of this section, you'll be able to:
- Explain what a proximity service is and identify its core features
- Define functional and non-functional requirements for a location-based search system
- Ask the right clarifying questions in a system design interview
- Distinguish between MVP features and advanced capabilities

### Why This Matters

Before writing a single line of code or drawing any diagrams, you need to understand WHAT you're building and WHY. This is often where interviews are won or lost - spending 5-10 minutes on requirements can save you from designing the wrong system! Real-world example: When Google Maps first launched proximity search, they underestimated the complexity of high-density areas like Manhattan (10K+ businesses per km²). This led to performance issues that required a complete architecture redesign. Understanding requirements upfront prevents costly mistakes!

### 🟢 For Beginners: The Fundamentals

#### What is a Proximity Service?

Think of a proximity service like a super-smart restaurant recommendation friend. When you say "I'm hungry and I'm at Times Square," your friend instantly knows all the restaurants nearby, which ones are open right now, which have good reviews, and can even tell you which ones match your taste preferences. That's exactly what Yelp, Google Maps, and similar services do - but for millions of people simultaneously!

A proximity service has three core components:

1. **Location Database**: Stores information about businesses (restaurants, shops, hotels) with their exact geographic coordinates (latitude and longitude)
2. **Search Engine**: Quickly finds businesses near a given location within a specified radius
3. **Filtering & Ranking**: Sorts results by relevance, considering distance, ratings, price, and user preferences

**Simple Example:**
When you search "coffee shops near me" on your phone:
- Your phone sends your coordinates: (37.7749° N, 122.4194° W) - San Francisco
- The system searches within 5km radius
- Returns 20 best-matching coffee shops sorted by distance and rating
- Shows which are open right now
- All of this happens in under 100 milliseconds!

#### Why Can't We Just Use a Simple Database Query?

Let's say you have a database with 100 million businesses. Each business has latitude and longitude. You might think: "Why not just calculate the distance from my location to every business and find the closest ones?"

Here's why that doesn't work:

```text
Simple approach (checking every business):
├─ Check business 1: Calculate distance... 
├─ Check business 2: Calculate distance...
├─ ... (repeat 100 million times)
└─ Total time: 100 million × 0.001ms = 100,000ms = 100 seconds!

Smart approach (using geospatial indexes):
├─ Use Geohash to narrow down to ~1000 businesses in the area
├─ Calculate precise distance for only those 1000 businesses
├─ Sort and return top 20 results
└─ Total time: <100ms (1000x faster!)
```

This is why we need special data structures called **geospatial indexes** (like Geohash, QuadTree, or R-tree) that organize businesses by location, making searches incredibly fast.

#### What Are the Core Features?

**Must-Have Features (MVP):**
- Search businesses by location and radius (e.g., "restaurants within 5km")
- Filter by category (restaurants, hotels, gas stations)
- Filter by basic criteria (rating > 4 stars, price level)
- Show business details (name, address, phone, hours)
- Display results on a map

**Nice-to-Have Features:**
- Personalized recommendations based on your history
- Real-time business status (open/closed right now)
- User reviews and photos
- Turn-by-turn navigation
- Check-in and social features

💡 **Pro Tip:** In interviews, always start with MVP features. Don't try to build everything at once! Yelp started with just business listings and reviews - they added personalization and advanced features later.

#### Understanding Geographic Coordinates

Every location on Earth can be described with two numbers:

**Latitude** (North-South position):
- Ranges from -90° (South Pole) to +90° (North Pole)
- Equator is at 0°
- San Francisco: 37.7749° N

**Longitude** (East-West position):
- Ranges from -180° (West) to +180° (East)
- Prime Meridian (Greenwich, UK) is at 0°
- San Francisco: -122.4194° W (or 122.4194° W)

**Distance Calculation:**
The Earth is roughly spherical, so calculating distance between two points isn't just subtraction - we use the **Haversine formula** which accounts for Earth's curvature. For a 5km radius search, there's about a 0.045° difference in coordinates (roughly 0.045° × 111km/degree ≈ 5km).

### 🟡 For Intermediate: Interview Patterns

#### The Requirements Gathering Framework

When you're in a system design interview for a proximity service, the interviewer is testing whether you can ask smart clarifying questions. Here's your interview script:

**Phase 1: Understand the Scale (2 minutes)**

"Let me start by understanding the scale of the system. Can I confirm a few assumptions?"

- "How many businesses are we indexing globally?" (Answer: ~100M)
- "How many users do we expect?" (Answer: ~500M total, ~100M daily active)
- "What's the peak query rate we need to support?" (Answer: ~50K queries/second)
- "What's our target response time?" (Answer: <100ms for p95)

**Phase 2: Clarify the Features (3 minutes)**

"Now let me understand what features we need to support:"

- "What search radius should we support?" (Answer: 1-50km configurable, most common is 5km)
- "Do we need real-time updates when businesses change hours?" (Answer: Yes, within 5 minutes)
- "Should we support complex filters?" (Answer: Yes - category, rating, price, open now, features)
- "Do we need personalization?" (Answer: Yes for advanced version, but start with basic ranking)

**Phase 3: Understand Geographic Scope (2 minutes)**

```text
"I want to confirm the geographic scope:"

Question: "Are we global or regional?"
└─ Answer: Global (100+ countries)
    └─ Implication: Need multi-region architecture for low latency

Question: "Do we need to handle high-density areas like Manhattan?"
└─ Answer: Yes, up to 10K+ businesses per km²
    └─ Implication: Need special indexing (QuadTree) for dense areas

Question: "What about cross-border searches?"
└─ Answer: Yes, searches near borders should work seamlessly
    └─ Implication: Can't shard strictly by country
```

⚠️ **Common Mistake:** Many candidates jump straight into architecture without understanding requirements. Spend 5-10 minutes here - it shows you're thorough and prevents designing the wrong system!

#### Making Assumptions Explicit

After asking questions, state your assumptions clearly:

```text
"Based on our discussion, I'm going to assume:

✅ Scale: 100M businesses, 500M users, 50K QPS peak
   → Need horizontal scaling and caching

✅ Latency: <100ms p95 response time
   → Must use geospatial indexes (Geohash/QuadTree) and multi-tier caching

✅ Geography: Global with multi-region support
   → Deploy in 4-5 regions (US-East, US-West, EU, Asia, etc.)

✅ Consistency: Eventual consistency acceptable (5-minute delay for updates)
   → Can use aggressive caching, don't need strong consistency

Are these assumptions reasonable?"
```

This approach shows you're thinking strategically and gives the interviewer a chance to course-correct if needed.

### 🔴 For Advanced: Production Considerations

#### Requirements and Business Impact

When you're making requirements decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Consistency vs Performance**

```text
Scenario: A restaurant updates their closing time from 10 PM to 9 PM

Option A: Strong Consistency (Immediate Updates)
├─ Guarantee: Every search immediately shows 9 PM closing time
├─ Implementation: Write-through cache invalidation, sync database updates
├─ Latency: 150ms query time (need to check fresh data)
├─ Business Impact: Users never see stale hours, but slower experience
└─ Use Case: Critical for services where staleness causes major issues

Option B: Eventual Consistency (5-minute delay)
├─ Guarantee: Updates appear within 5 minutes
├─ Implementation: Async cache invalidation, TTL-based expiry
├─ Latency: 50ms query time (serve from cache)
├─ Business Impact: 0.1% of users might see stale hours, but 3x faster
└─ Use Case: Most proximity services (Yelp, Google Maps use this)

💡 Real-world: Yelp uses eventual consistency with 2-5 minute cache TTL because the performance benefit (3x faster queries) outweighs the occasional stale data. Critical updates (business permanently closed) use write-through invalidation.
```

**Trade-off 2: Feature Richness vs Time-to-Market**

```text
Phase 1 (MVP - 3 months):
├─ Basic proximity search with Geohash indexing
├─ Simple filtering (category, rating)
├─ Business listings with basic info
└─ Goal: Prove the concept, get user feedback

Phase 2 (Enhanced - 6 months):
├─ Advanced filters (price, hours, features)
├─ Real-time business status
├─ User reviews and ratings
├─ Photo uploads
└─ Goal: Feature parity with competitors

Phase 3 (Advanced - 12 months):
├─ ML-based personalization
├─ Recommendation engine
├─ Social features (check-ins, friends)
├─ Voice search and NLP
└─ Goal: Differentiation and competitive advantage
```

**Trade-off 3: Build vs Buy for Geospatial**

```text
Option A: Use Managed Service (e.g., AWS Location Service, Google Maps Platform)
├─ Pros: Fast to market, fully managed, proven reliability
├─ Cons: $50K-200K/month at scale, vendor lock-in, less customization
├─ Use Case: Startups, rapid prototyping
└─ Decision: Best for first 1-2 years or if geo search isn't core business

Option B: Build Custom (PostGIS + Elasticsearch + custom indexing)
├─ Pros: Full control, $10K-50K/month cost, can optimize for specific needs
├─ Cons: 6-12 months to build, need specialized expertise
├─ Use Case: Mature companies where geo search is core product
└─ Decision: Best for companies like Yelp where proximity is the primary feature

💡 Real-world: Yelp and Foursquare built custom solutions because location search is their core business. Uber started with Google Maps but later built their own system as they scaled to reduce costs by 80%.
```

#### Advanced Requirements Patterns

**Handling Regulatory Requirements:**

```text
GDPR (Europe):
├─ Data Residency: User data must stay in EU region
├─ Right to Deletion: Delete user data within 30 days
├─ Consent Management: Track user consent for personalization
└─ Implementation: Separate EU database cluster, deletion queues

CCPA (California):
├─ Data Disclosure: Users can request all data collected
├─ Opt-out: Users can opt-out of data selling
└─ Implementation: Data export APIs, opt-out tracking

Accessibility Requirements:
├─ WCAG 2.1 AA compliance for mobile apps
├─ Screen reader support
├─ Voice search for visually impaired users
└─ Implementation: Semantic HTML, ARIA labels, voice APIs
```

**Enterprise Requirements:**

When selling to enterprises (restaurant chains, hotel groups), requirements expand:

- **SSO Integration**: Support SAML, OAuth for corporate logins
- **Audit Logs**: Track all business profile changes for compliance
- **SLA Guarantees**: 99.99% uptime with financial penalties
- **Bulk Operations**: Update 10K+ locations simultaneously
- **White-labeling**: Custom branding for enterprise portals
- **Analytics**: Custom dashboards and data exports

### Real-World Example: How Yelp Evolved Their Requirements

Let's look at how Yelp's requirements changed over time:

**2004-2006 - Early Days:**

```text
Context: Small startup in San Francisco
├─ Businesses: 10K local businesses
├─ Users: 100K users
├─ Features: Basic business listings, user reviews
├─ Technology: Single PostgreSQL database, simple distance calculation
└─ Result: Worked fine for single city
```

**2007-2010 - National Expansion:**

```text
Context: Expanding to 50+ US cities
├─ Businesses: 5M businesses across US
├─ Users: 10M users
├─ Added: Multi-city support, mobile apps, check-ins
├─ Challenge: Simple distance calculation too slow
├─ Solution: Introduced Geohash indexing
└─ Result: Could handle national scale with <500ms response times
```

**2011-2015 - Global Scale:**

```text
Context: International expansion, mobile-first
├─ Businesses: 50M businesses in 30+ countries
├─ Users: 100M monthly active users
├─ Added: Personalized recommendations, real-time business updates
├─ Challenge: High-density cities (NYC, Tokyo) causing timeouts
├─ Solution: Hybrid approach - Geohash + QuadTree for dense areas
└─ Result: Consistent <100ms response globally
```

**2016-Present - AI-Powered:**

```text
Context: Competing with Google Maps, mature platform
├─ Businesses: 100M+ businesses globally
├─ Users: 500M+ users
├─ Added: ML recommendations, voice search, augmented reality
├─ Challenge: Personalization latency, cold start problem
├─ Solution: Precomputed embeddings, feature store, A/B testing
└─ Result: 75% increase in engagement, 99.99% availability
```

📊 **By The Numbers:**
- 2004: 100 QPS, single server
- 2010: 1K QPS, 10 servers
- 2015: 10K QPS, 100 servers  
- 2025: 50K QPS, 500+ servers across 5 regions

**Key Lesson:** Requirements evolve as you scale. Start simple (MVP with basic proximity search), then add complexity (personalization, real-time updates) as you grow. Don't try to build everything day one!

### 🎯 Interview Questions: Requirements & Scope

#### Question 1: How would you gather requirements for designing a proximity service like Yelp?

**What the interviewer wants to know:**
- Can you systematically gather information?
- Do you ask about scale, features, and constraints?
- Can you prioritize requirements?

**Answer Framework:**

```text
1. Understand the Scale
   ├─ "How many businesses are we indexing?"
   ├─ "How many users and what's the query rate?"
   ├─ "What's our target response time?"
   └─ Purpose: Determine if we need simple vs complex architecture

2. Clarify Core Features
   ├─ "What's the primary use case - finding nearby businesses?"
   ├─ "What filters do users need - category, rating, price?"
   ├─ "Do we need real-time updates or is eventual consistency OK?"
   └─ Purpose: Define MVP vs nice-to-have features

3. Understand Geographic Scope
   ├─ "Are we global or regional?"
   ├─ "Do we need to handle high-density areas?"
   ├─ "What about cross-border searches?"
   └─ Purpose: Determine sharding strategy and index complexity

4. Discuss Trade-offs
   ├─ "Should we prioritize consistency or performance?"
   ├─ "Is personalization required for MVP?"
   ├─ "What's more important - fast reads or fresh data?"
   └─ Purpose: Show you understand there are no perfect solutions

Example Numbers:
├─ Scale: 100M businesses, 50K QPS
├─ Latency: <100ms p95
├─ Consistency: 5-minute eventual consistency acceptable
└─ Geography: Global with multi-region deployment
```

**Follow-up: What if the interviewer says "assume Google Maps scale"?**

```text
Google Maps scale means:
├─ Businesses: 200M+ global POIs (points of interest)
├─ Users: 1B+ monthly active users
├─ QPS: 100K+ queries per second global
├─ Latency: <50ms p95 (stricter than Yelp)
└─ Availability: 99.99%+ (financial impact of downtime)

Architecture changes needed:
1. More aggressive sharding (50+ shards vs 10)
2. Edge caching at CDN level (Cloudflare Workers)
3. Precomputed results for popular queries
4. Multi-region active-active deployment (not active-passive)
5. Custom network protocols (not just HTTP REST)
```

#### Question 2: How do functional and non-functional requirements differ for a proximity service?

**What the interviewer wants to know:**
- Do you understand the difference between what the system does vs how well it does it?
- Can you translate business requirements into technical specifications?

**Answer Framework:**

```text
Functional Requirements (WHAT the system does):
├─ Search businesses within radius
│  └─ Technical: Implement geospatial query with distance calculation
├─ Filter by category, rating, price
│  └─ Technical: Multi-criteria filtering on indexed fields
├─ Show business details
│  └─ Technical: Fetch business data from database
└─ Display on map
    └─ Technical: Return GeoJSON for map rendering

Non-Functional Requirements (HOW WELL it does it):
├─ Performance: <100ms p95 latency
│  └─ Technical: Need caching, geospatial indexes, load balancing
├─ Scalability: Support 50K QPS
│  └─ Technical: Horizontal scaling, sharding, read replicas
├─ Availability: 99.99% uptime
│  └─ Technical: Multi-region, failover, health checks
└─ Consistency: Eventual consistency (5min delay OK)
    └─ Technical: Async updates, TTL-based caching

Example Mapping:
Business Requirement: "Users want to find restaurants open right now"
├─ Functional: System must filter by current business hours
└─ Non-Functional: Filter must add <10ms to query time
```

#### Question 3: What questions would you ask about handling high-density areas?

**What the interviewer wants to know:**
- Do you anticipate edge cases and scaling challenges?
- Can you think about performance implications?

**Answer Framework:**

```text
Key Questions to Ask:

1. Definition of "High-Density"
   ├─ "What's the maximum businesses per km² we need to support?"
   ├─ Expected: 10K+ in places like Manhattan, Tokyo
   └─ Implication: Need special data structures (QuadTree)

2. Performance Requirements
   ├─ "Should high-density areas have same <100ms latency?"
   ├─ Expected: Yes, consistent experience globally
   └─ Implication: Can't do linear scan, must use adaptive indexing

3. Search Radius Impact
   ├─ "In Manhattan with 10K businesses/km², a 5km radius means 785K businesses!"
   ├─ Question: "Should we limit results or use clustering?"
   └─ Solution: Return top 500 results max, use map clustering for display

4. Index Strategy
   ├─ "Can we use same index for low and high density areas?"
   ├─ Answer: Hybrid approach - Geohash for normal, QuadTree for high density
   └─ Implementation: Detect density and route to appropriate index

Example Calculation:
Manhattan density: 10,000 businesses/km²
5km radius search: π × 5² = 78.5 km²
Total businesses: 78.5 × 10,000 = 785,000 businesses
Without optimization: 785K × 0.01ms = 7,850ms (TIMEOUT!)
With QuadTree: Reduce to ~1000 candidates, then <100ms
```

### 🤔 Think About It

1. **For Beginners:** Why do you think we need special geospatial indexes instead of just calculating distance to every business? (Hint: Think about how long it would take to check 100 million businesses one by one)

2. **For Intermediate:** If you had to choose between strong consistency (all users see updates immediately) and performance (3x faster queries), which would you prioritize for a proximity service? Why?

3. **For Advanced:** How would your requirements change if you were building a proximity service specifically for:
   - Emergency services (finding nearest hospital)?
   - Drone delivery (real-time dynamic locations)?
   - AR navigation (need 10ms response time for smooth experience)?

### ✅ Key Takeaways

- **Requirements drive architecture**: Spend 5-10 minutes understanding scale, features, and constraints before designing
- **Start with MVP**: Don't try to build everything day one - Yelp started with just listings and reviews
- **Scale determines complexity**: 1K businesses can use simple distance calculation; 100M businesses need Geohash/QuadTree
- **Geography matters**: High-density areas (Manhattan, Tokyo) need special handling with QuadTree or S2
- **Trade-offs are inevitable**: Choose eventual consistency + performance over strong consistency + latency for most proximity services
- **Ask clarifying questions**: In interviews, asking smart questions is more important than jumping into solutions
- **Business context matters**: Different industries (emergency services vs restaurant discovery) have vastly different requirements

### 🎯 Practice Exercise

**Scenario:** You're designing a proximity service for a pet adoption platform. Users should be able to find adoptable pets (dogs, cats, etc.) at shelters within a specified radius.

**Your Task:**
1. List 5 functional requirements specific to pet adoption (not just generic proximity search)
2. List 5 non-functional requirements with specific numbers (latency, scale, etc.)
3. Identify 3 key differences from a restaurant proximity service like Yelp
4. Estimate the scale: How many shelters? How many searches per day? What search radius makes sense?

**Bonus Challenge:** How would your design differ if pets move between shelters frequently (high churn rate) vs restaurants that rarely change location?

---

## Section 2: Planning for Scale (Capacity Estimation)

### What You'll Learn

By the end of this section, you'll be able to:
- Calculate queries per second (QPS) from daily active users
- Estimate storage requirements for business data, photos, and geospatial indexes
- Determine bandwidth needs for API requests and responses
- Perform back-of-the-envelope calculations during interviews
- Understand how geographic density impacts resource requirements

### Why This Matters

Before designing any component, you need to know HOW MUCH you're building for. A system for 1,000 businesses is completely different from one handling 100 million! Real-world example: Foursquare initially designed for urban explorers in NYC (thousands of venues) but when they expanded globally (millions of venues), their simple distance calculation approach collapsed. They had to completely rebuild their geospatial indexing system. Spending 5 minutes on capacity estimation saves months of re-engineering!

### 🟢 For Beginners: The Fundamentals

#### Why Do We Need Capacity Estimation?

Imagine you're opening a restaurant. You wouldn't just guess how many tables you need - you'd estimate how many customers you expect per day, calculate seats needed for peak hours, and plan your kitchen capacity accordingly. System design is exactly the same!

Capacity estimation helps you answer three critical questions:

1. **How many servers do I need?** (Can one server handle all requests, or do I need 100?)
2. **How much storage do I need?** (Will 100GB suffice, or do I need 100TB?)
3. **How much network bandwidth do I need?** (Is 100Mbps enough, or do I need 10Gbps?)

Let's walk through a simple example for a small proximity service:

**Scenario: Local City Business Finder**
- Businesses: 10,000 (one city only)
- Users: 50,000
- Each user searches 2 times per day
- Business data: 2KB per business
- Each search returns 10 businesses

**Step 1: Calculate Traffic**

```text
Daily searches = 50,000 users × 2 searches/user = 100,000 searches/day

Searches per second:
├─ Average: 100,000 searches ÷ 86,400 seconds = 1.16 QPS
└─ Peak (assume 3x average): 1.16 × 3 = 3.5 QPS

Conclusion: One server can easily handle this! 
(Most servers can handle 1000+ QPS)
```

**Step 2: Calculate Storage**

```text
Business data: 10,000 businesses × 2KB = 20MB
Geospatial index: ~10% overhead = 2MB
Total: 22MB

Conclusion: This fits easily in memory! No need for complex caching.
```

**Step 3: Calculate Bandwidth**

```text
Each request: 100 bytes (lat, lng, filters)
Each response: 10 businesses × 200 bytes = 2KB

Bandwidth needed:
├─ Requests: 3.5 QPS × 100 bytes = 350 bytes/second
└─ Responses: 3.5 QPS × 2KB = 7KB/second

Conclusion: Negligible bandwidth - any modern server can handle this.
```

💡 **Pro Tip:** For small systems (< 10K QPS), a single powerful server is often enough. Don't over-engineer!

#### Understanding the Core Metrics

Let's break down the key metrics you need to estimate:

**QPS (Queries Per Second)**
Think of this like customers arriving at your restaurant per second. If 100 customers come per hour, that's 100 ÷ 3600 = 0.028 customers per second. For systems, if you have 100M searches per day:
- Average QPS: 100M ÷ 86,400 seconds = 1,157 QPS
- Peak QPS (lunch/dinner rush): ~5,000-10,000 QPS

**Storage**
This is like figuring out how much space you need in your warehouse:
- Business data: Number of businesses × size per business record
- Geospatial indexes: Usually 10-30% overhead on top of raw data
- User data: Number of users × size per user profile
- Photos/media: Typically stored separately in CDN

**Bandwidth**
Think of this as the size of your restaurant doors - how much data needs to flow in and out:
- Request bandwidth: QPS × average request size
- Response bandwidth: QPS × average response size
- Media bandwidth: Photo views per second × photo size

#### The Magic of Peak vs Average

Here's something crucial that beginners often miss: systems don't run at constant load!

```text
Restaurant analogy:
├─ Average customers: 10 per hour (0.003 per second)
├─ Peak (lunch): 60 per hour (0.017 per second)
└─ Need to staff for peak, not average!

System analogy:
├─ Average QPS: 1,000 QPS
├─ Peak QPS (lunch/dinner): 5,000 QPS
└─ Need servers for 5,000 QPS, not 1,000!

Common peak multipliers:
├─ Social media: 2-3x (consistent usage throughout day)
├─ E-commerce: 5-10x (black Friday, flash sales)
├─ Restaurant search: 5-6x (lunch 11-2, dinner 5-9)
└─ News sites: 50-100x (breaking news spikes)
```

For proximity services, we typically use a **5x peak multiplier** because searches spike during meal times (lunch and dinner).

### 🟡 For Intermediate: Interview Patterns

#### The Capacity Estimation Interview Framework

When an interviewer says "Design Yelp," here's how you tackle capacity estimation:

**Step 1: Establish Numbers (2 minutes)**

"Let me start with some back-of-the-envelope calculations. I'll make some assumptions and you can correct me:"

```text
Assumptions:
├─ Businesses: 100M globally (Yelp scale)
├─ Users: 500M total, 100M daily active
├─ Searches per user per day: 3
└─ Write rate: 1M business updates per day

Calculations:
├─ Daily searches: 100M users × 3 = 300M searches/day
├─ Average QPS: 300M ÷ 86,400 = 3,472 QPS
├─ Peak QPS: 3,472 × 5 = 17,360 QPS (round to 20K QPS)
└─ Write QPS: 1M ÷ 86,400 = 11.6 QPS
```

⚠️ **Common Mistake:** Many candidates forget to calculate peak QPS and only provide average. Always multiply by a peak factor (2-5x depending on use case)!

**Step 2: Storage Estimation (2 minutes)**

"Now let me estimate storage requirements:"

```text
Business Data:
├─ Record structure:
│   ├─ Basic info (name, address, phone): 500 bytes
│   ├─ Coordinates + geohash: 50 bytes
│   ├─ Metadata (rating, hours, features): 200 bytes
│   └─ Total per business: ~750 bytes (round to 1KB)
├─ Total: 100M businesses × 1KB = 100GB
└─ With indexes: 100GB × 1.3 = 130GB

Geospatial Indexes:
├─ Geohash index: 100M × 20 bytes = 2GB
├─ QuadTree (for dense areas): ~3x overhead = 6GB
└─ Total indexes: 8GB

User Data:
├─ User profile: 500M users × 2KB = 1TB
├─ User preferences: 100M active × 500 bytes = 50GB
└─ Total: 1.05TB

Photos:
├─ Average photos per business: 10
├─ Size per photo: 500KB (original)
├─ Total: 100M × 10 × 500KB = 500TB
├─ Compression + CDN: ~200TB
└─ Note: Stored in S3/CDN, not in database

Reviews:
├─ Average reviews per business: 50
├─ Size per review: 1KB (text + metadata)
├─ Total: 100M × 50 × 1KB = 5TB
└─ Stored in Cassandra (write-heavy, time-series data)

Total Storage:
├─ Business data: 130GB
├─ Indexes: 8GB
├─ Users: 1TB
├─ Photos: 200TB (CDN)
├─ Reviews: 5TB
└─ Grand Total: ~207TB
```

**Step 3: Bandwidth Estimation (2 minutes)**

"Let me calculate network bandwidth requirements:"

```text
Request Bandwidth:
├─ Request size: 200 bytes (lat, lng, radius, filters)
├─ Peak QPS: 20K
├─ Bandwidth: 20K × 200 bytes = 4MB/second = 32Mbps
└─ Negligible - not a bottleneck

Response Bandwidth:
├─ Response size: 20 businesses × 500 bytes = 10KB per response
├─ Peak QPS: 20K
├─ Bandwidth: 20K × 10KB = 200MB/second = 1.6Gbps
└─ Significant - need good network capacity

Photo Delivery (CDN):
├─ Assume 60% of searches view business photos
├─ Photo views: 20K QPS × 0.6 = 12K photos/second
├─ Photo size (thumbnail): 50KB
├─ Bandwidth: 12K × 50KB = 600MB/second = 4.8Gbps
└─ Critical - must use CDN to offload

Total Bandwidth:
├─ API responses: 1.6Gbps per region
├─ CDN (photos): 4.8Gbps global
└─ Need: 10Gbps connections per datacenter
```

#### Advanced Calculations: Geographic Distribution

Now let's think about how geography affects our estimates:

```text
Global Distribution (100M businesses):
├─ North America: 40M (40%)
│   ├─ United States: 30M
│   ├─ Canada: 8M
│   └─ Mexico: 2M
├─ Europe: 25M (25%)
│   ├─ EU countries: 20M
│   └─ UK: 5M
├─ Asia: 30M (30%)
│   ├─ China: 15M
│   ├─ India: 10M
│   └─ Others: 5M
└─ Others: 5M (5%)

Traffic Distribution (300M searches/day):
├─ North America: 150M (50%) - highest search rate per business
├─ Europe: 75M (25%)
├─ Asia: 60M (20%)
└─ Others: 15M (5%)

Implication: Need to deploy in multiple regions!
├─ US-East: 100M searches/day = 5.8K QPS (peak: 29K)
├─ US-West: 50M searches/day = 2.9K QPS (peak: 14.5K)
├─ EU-West: 75M searches/day = 4.3K QPS (peak: 21.5K)
└─ Asia-Pacific: 75M searches/day = 4.3K QPS (peak: 21.5K)

Each region needs:
├─ API servers: 30K QPS ÷ 500 QPS per server = 60 servers
├─ With 2x redundancy: 120 servers per region
└─ Total global: 480 servers
```

### 🔴 For Advanced: Production Considerations

#### Deep Capacity Analysis with Business Impact

Let's think about capacity from a Total Cost of Ownership (TCO) perspective:

**Compute Costs:**

```text
API Server Fleet:
├─ Servers needed: 480 globally (120 per region × 4 regions)
├─ Server spec: 16 vCPU, 64GB RAM
├─ Cost per server: $500/month (AWS c5.4xlarge)
├─ Total compute: 480 × $500 = $240K/month
└─ Annual: $2.88M/year

Caching Layer (Redis):
├─ Cache size per region: 170GB
├─ Redis instances: 5 per region (r5.2xlarge, 64GB each)
├─ Cost: 5 × $300 × 4 regions = $6K/month
└─ Annual: $72K/year

Database Servers:
├─ PostgreSQL primaries: 10 shards × 4 regions = 40 servers
├─ Read replicas: 40 primaries × 3 replicas = 120 servers
├─ Cost per DB server: $800/month (db.r5.2xlarge)
├─ Total: 160 × $800 = $128K/month
└─ Annual: $1.536M/year

Elasticsearch Cluster:
├─ Nodes needed: 20 per region × 4 regions = 80 nodes
├─ Cost per node: $600/month (i3.2xlarge)
├─ Total: 80 × $600 = $48K/month
└─ Annual: $576K/year

Storage Costs:
├─ Business/User data: 1.2TB × $0.10/GB = $120/month
├─ Photos (S3): 200TB × $0.023/GB = $4,600/month
├─ Reviews (Cassandra): 5TB on SSD = $500/month
└─ Annual: $62.6K/year

CDN Costs:
├─ Photo delivery: 4.8Gbps × 30 days × 3600 seconds
├─ Data transfer: ~15PB/month
├─ CDN cost: $0.02/GB = 15,000TB × $0.02 = $300K/month
└─ Annual: $3.6M/year

Total Infrastructure Cost:
├─ Compute: $2.88M/year
├─ Caching: $72K/year
├─ Database: $1.536M/year
├─ Search: $576K/year
├─ Storage: $62.6K/year
├─ CDN: $3.6M/year
└─ Total: $8.73M/year ($728K/month)

Cost per Search:
├─ 300M searches/day × 365 days = 109.5B searches/year
├─ Cost per search: $8.73M ÷ 109.5B = $0.00008 per search
└─ Business Model: Charge advertisers $0.50+ per click = 6,250x ROI
```

**Optimization Opportunities:**

```text
Cost Reduction Strategy 1: Reserved Instances
├─ Use 3-year reserved instances for baseline capacity
├─ Savings: 40% on compute = $1.15M/year savings
└─ Risk: Locked in for 3 years, can't easily scale down

Cost Reduction Strategy 2: Spot Instances
├─ Use spot instances for 30% of API servers
├─ Savings: 70% on those servers = $346K/year
└─ Risk: Can be terminated, need graceful degradation

Cost Reduction Strategy 3: CDN Optimization
├─ Increase cache TTL from 1 hour to 24 hours
├─ Reduce origin fetches by 80%
├─ Savings: $2.88M/year on CDN costs
└─ Trade-off: Slightly staler photos

Cost Reduction Strategy 4: Smart Caching
├─ Cache hot locations more aggressively
├─ Reduce database queries by 90%
├─ Savings: Can use smaller database instances = $920K/year
└─ Trade-off: More complex cache invalidation

Total Potential Savings: $5.3M/year (60% reduction!)
New Total Cost: $3.4M/year
```

#### Capacity Planning for Growth

```text
Year 1 (Current: 100M businesses, 50K QPS):
├─ Infrastructure: $8.73M
├─ Cost per business: $0.087/year
└─ Cost per search: $0.00008

Year 2 (2x growth: 200M businesses, 100K QPS):
├─ Infrastructure: $15M (not 2x due to economies of scale)
├─ Why not 2x cost?
│   ├─ Can use larger, more efficient servers (better $/QPS)
│   ├─ CDN costs grow sublinearly (better cache hit rates)
│   ├─ Database costs grow 1.5x (better sharding efficiency)
│   └─ Network costs have volume discounts
├─ Cost per business: $0.075/year (13% reduction!)
└─ Cost per search: $0.00007 (12% reduction!)

Year 5 (10x growth: 1B businesses, 500K QPS):
├─ Infrastructure: $45M (only 5x cost for 10x scale!)
├─ Economies of scale:
│   ├─ Negotiate better rates with cloud providers
│   ├─ Own datacenter for primary regions
│   ├─ Custom CDN with ISP partnerships
│   └─ Optimized protocols reduce bandwidth 50%
├─ Cost per business: $0.045/year (48% reduction from Year 1!)
└─ Cost per search: $0.00004 (50% reduction!)

Key Insight: Infrastructure costs grow sublinearly with scale!
At 10x scale, per-unit costs drop by ~50%
```

#### Advanced Estimation: High-Density Scenarios

Let's analyze the edge case that breaks most proximity services:

```text
Manhattan High-Density Scenario:
├─ Area: 59 km² (Manhattan)
├─ Businesses: 100,000
├─ Density: 1,695 businesses/km²

5km Radius Search in Times Square:
├─ Search area: π × 5² = 78.5 km²
├─ But Manhattan is only 59 km², so cap at 59 km²
├─ Businesses in radius: 59 × 1,695 = 100,000 businesses!
├─ All of Manhattan fits in a 5km radius from Times Square

Query Performance Analysis:
├─ Naive approach: Check all 100K businesses
│   ├─ Distance calc: 0.00001ms per business
│   ├─ Total: 100,000 × 0.00001 = 1 second (TIMEOUT!)
│   └─ Verdict: UNACCEPTABLE
│
├─ Geohash approach (6-char, ±0.61km):
│   ├─ Geohash covers ~3.7 km²
│   ├─ Businesses in geohash: 3.7 × 1,695 = 6,272
│   ├─ Still need to check 6K+ businesses: 62ms
│   └─ Verdict: Better but still slow
│
└─ QuadTree approach:
    ├─ Dynamically subdivide high-density areas
    ├─ Leaf nodes: Max 500 businesses each
    ├─ Tree depth for Manhattan: 8 levels
    ├─ Query complexity: O(log n) = O(log 100K) = 17 node checks
    ├─ Businesses examined: ~1,000 (after tree traversal)
    ├─ Query time: 10ms (tree) + 10ms (distance) = 20ms
    └─ Verdict: ACCEPTABLE!

Resource Impact:
├─ Geohash index: 100K × 20 bytes = 2MB (negligible)
├─ QuadTree index: 100K × 200 bytes = 20MB (significant but manageable)
├─ Memory per server: 20MB × 50 major cities = 1GB
└─ Decision: Use hybrid - Geohash for normal areas, QuadTree for dense cities
```

### Real-World Example: How Google Maps Handles Scale

Let's examine how Google Maps evolved their capacity planning:

**2005 - Launch:**

```text
Context: US-only, desktop web
├─ Points of Interest: 10M businesses in US
├─ Users: 5M monthly active
├─ QPS: ~100 average, ~500 peak
├─ Infrastructure: 50 servers in one datacenter
├─ Cost: ~$100K/year
└─ Strategy: Single region, simple architecture
```

**2010 - Mobile Explosion:**

```text
Context: Global expansion, mobile apps launched
├─ POIs: 50M globally
├─ Users: 100M monthly active (20x growth!)
├─ QPS: 5K average, 25K peak
├─ Infrastructure: 1,000 servers across 3 regions
├─ Cost: ~$5M/year
├─ Challenge: Mobile searches spike during commute times
├─ Solution: Added aggressive caching, multi-region deployment
└─ Result: Maintained <200ms latency globally
```

**2015 - Dominant Platform:**

```text
Context: #1 map app globally, real-time traffic
├─ POIs: 200M globally
├─ Users: 1B monthly active (10x growth!)
├─ QPS: 50K average, 250K peak
├─ Infrastructure: 20,000+ servers across 15 regions
├─ Cost: ~$100M/year
├─ Challenge: Need sub-100ms latency for smooth navigation
├─ Solution: Edge computing with custom network infrastructure
└─ Result: 50ms p95 latency globally
```

**2025 - AI-Powered:**

```text
Context: AR navigation, predictive suggestions
├─ POIs: 300M+ globally (including points of interest)
├─ Users: 2B+ monthly active
├─ QPS: 200K average, 1M peak
├─ Infrastructure: 100,000+ servers, custom hardware
├─ Cost: ~$500M/year (but generates $20B+ revenue)
├─ Innovation: Custom TPUs for ML, predictive preloading
└─ ROI: 40x return on infrastructure investment
```

📊 **By The Numbers:**
- 2005: $0.02 per user per year
- 2010: $0.05 per user per year (2.5x)
- 2015: $0.10 per user per year (2x)
- 2025: $0.25 per user per year (2.5x)

**Key Lesson:** Infrastructure cost per user grows over time as features become more sophisticated (real-time traffic, ML predictions, AR), but revenue per user grows faster (40x ROI)!

### 🎯 Interview Questions: Capacity Estimation

#### Question 1: Walk me through estimating QPS for a proximity service with 100M daily active users.

**What the interviewer wants to know:**
- Can you break down the problem systematically?
- Do you consider peak vs average?
- Do you use reasonable assumptions?

**Answer Framework:**

```text
Step 1: Clarify Usage Pattern
├─ "How many searches does each user perform per day?"
├─ Assumption: 3 searches per user (typical for restaurant/business search)
└─ Reasoning: People search at breakfast, lunch, and dinner

Step 2: Calculate Daily Volume
├─ Total searches: 100M users × 3 searches = 300M searches/day
└─ This is our baseline

Step 3: Calculate Average QPS
├─ QPS = Total searches ÷ seconds per day
├─ QPS = 300M ÷ 86,400 seconds
├─ QPS = 3,472 (round to 3,500)
└─ But we can't design for average!

Step 4: Account for Peak Hours
├─ Proximity searches spike during meal times
├─ Peak hours: 11 AM-2 PM (lunch), 5 PM-9 PM (dinner) = 6 hours
├─ Assume 70% of searches happen during these 6 hours
├─ Peak volume: 300M × 0.7 = 210M searches in 6 hours
├─ Peak QPS: 210M ÷ (6 × 3,600) = 9,722 QPS
└─ Add 2x safety margin: ~20K QPS

Step 5: Consider Geographic Distribution
├─ Not all regions peak simultaneously
├─ US lunch time is Europe evening, Asia late night
├─ Per-region peak: 20K ÷ 4 regions = 5K QPS per region
└─ Each region needs capacity for 5K QPS

Final Answer:
├─ Global peak: 20K QPS
├─ Per-region peak: 5K QPS
├─ Design each region for: 5K QPS × 2 (redundancy) = 10K QPS capacity
└─ Servers needed: 10K ÷ 500 QPS per server = 20 servers per region
```

**Follow-up: What if it's a lunch delivery app instead of restaurant search?**

```text
Key Difference: Much sharper peaks!
├─ Lunch delivery: 90% of orders between 11:30 AM - 1:30 PM (2 hours)
├─ Not evenly distributed like restaurant search
├─ Peak calculation:
│   ├─ Daily orders: 100M users × 0.3 conversion = 30M orders
│   ├─ Peak orders: 30M × 0.9 = 27M in 2 hours
│   ├─ Peak QPS: 27M ÷ (2 × 3,600) = 3,750 QPS
│   └─ With 5x spike (promotional campaigns): 18.7K QPS
├─ Result: Similar overall QPS but much sharper spike
└─ Need: Better autoscaling, queue-based processing
```

#### Question 2: How would you estimate storage for 100M businesses with photos and reviews?

**What the interviewer wants to know:**
- Can you break down storage into components?
- Do you understand different storage tiers (database vs CDN)?
- Do you consider growth and retention?

**Answer Framework:**

```text
1. Business Core Data
   ├─ Fields: Name (100B), address (200B), coordinates (16B), 
   │          phone (20B), category (50B), hours (200B), metadata (200B)
   ├─ Total per business: ~800 bytes (round to 1KB for safety)
   ├─ Total: 100M × 1KB = 100GB
   └─ With database indexes (1.3x overhead): 130GB

2. Geospatial Indexes
   ├─ Geohash index: Business ID + Geohash (8 chars) = 24 bytes per business
   ├─ Total: 100M × 24 bytes = 2.4GB
   ├─ QuadTree (for dense areas): ~3x overhead = 7GB
   └─ Total indexes: ~10GB (fits in memory!)

3. User Data
   ├─ User profiles: 500M users × 2KB = 1TB
   ├─ User preferences: 100M active × 500 bytes = 50GB
   ├─ User sessions/auth: 50GB
   └─ Total: ~1.1TB

4. Photos (Tricky Part!)
   ├─ Average photos per business: 10 photos
   ├─ Original photo size: 2MB
   ├─ Thumbnails: 3 sizes (50KB, 200KB, 500KB)
   ├─ Storage per business:
   │   ├─ Originals: 10 × 2MB = 20MB
   │   ├─ Thumbnails: 10 × (50KB + 200KB + 500KB) = 7.5MB
   │   └─ Total: 27.5MB per business
   ├─ Total: 100M × 27.5MB = 2,750TB = 2.75PB
   ├─ With compression (JPEG → WebP, 30% savings): ~2PB
   └─ Note: Store in S3/CDN, not in database!

5. Reviews
   ├─ Average reviews per business: 50
   ├─ Review size: 1KB (500 chars text + metadata)
   ├─ Total: 100M × 50 × 1KB = 5TB
   ├─ Growth: 10% new reviews per year = 500GB/year growth
   └─ Store in Cassandra (time-series, write-heavy)

6. Analytics & Logs
   ├─ Search logs: 300M searches/day × 1KB = 300GB/day
   ├─ Retention: 90 days = 27TB
   ├─ Business analytics: 5TB
   └─ Total: 32TB

Total Storage Summary:
├─ Business data (PostgreSQL): 130GB
├─ Geospatial indexes (memory): 10GB
├─ User data (PostgreSQL): 1.1TB
├─ Photos (S3/CDN): 2PB
├─ Reviews (Cassandra): 5TB
├─ Analytics (ClickHouse): 32TB
└─ Grand Total: ~2.04PB (mostly photos!)

Storage Costs (at scale):
├─ PostgreSQL: 1.3TB × $0.10/GB = $130/month
├─ S3 (photos): 2PB × $0.023/GB = $47K/month
├─ Cassandra: 5TB × $0.10/GB (SSD) = $500/month
├─ Analytics: 32TB × $0.02/GB (HDD) = $640/month
└─ Total: ~$48K/month = $576K/year
```

#### Question 3: How do you handle capacity estimation for high-density areas like Manhattan?

**What the interviewer wants to know:**
- Do you consider edge cases in your estimates?
- Can you identify performance bottlenecks?
- Do you think about adaptive strategies?

**Answer Framework:**

```text
Challenge: Manhattan has 10K+ businesses/km², 5km radius = 785K businesses!

Step 1: Identify the Problem
├─ Normal area: 100 businesses/km² × 78.5 km² = 7,850 businesses
├─ Manhattan: 10,000 businesses/km² × 59 km² = 100,000 businesses
├─ Impact: 13x more businesses to check!
└─ Query time: 7,850 × 0.01ms = 78ms → 100,000 × 0.01ms = 1000ms (TIMEOUT!)

Step 2: Calculate Resource Impact
├─ Memory for standard Geohash:
│   ├─ 6-char geohash covers ~3.7 km²
│   ├─ Businesses per geohash: 3.7 × 10,000 = 37,000
│   ├─ Still too many to scan quickly!
│   └─ Verdict: Geohash alone insufficient
│
├─ Memory for QuadTree:
│   ├─ Tree structure: 200 bytes per business
│   ├─ Manhattan: 100K × 200 = 20MB
│   ├─ 50 major cities: 50 × 20MB = 1GB
│   └─ Verdict: Fits in memory, acceptable!
│
└─ Query time comparison:
    ├─ Geohash: O(n) linear scan = 370ms
    ├─ QuadTree: O(log n) tree traversal = 15ms
    └─ Speedup: 25x faster!

Step 3: Storage Overhead
├─ Standard index: 100M × 20 bytes = 2GB
├─ QuadTree for dense areas: 5M businesses × 200 bytes = 1GB additional
├─ Total: 3GB (minimal overhead)
└─ Decision: Use hybrid approach

Step 4: Adaptive Strategy
├─ Detect density at query time:
│   ├─ If geohash has < 1,000 businesses → use Geohash (fast path)
│   ├─ If geohash has > 1,000 businesses → use QuadTree (complex path)
│   └─ 95% of queries use fast path, 5% use complex path
│
├─ Resource allocation:
│   ├─ Cache hot dense areas (Manhattan, Tokyo, etc.) in memory
│   ├─ 50 dense areas × 20MB = 1GB per server
│   ├─ Reduces query time from 15ms → 5ms
│   └─ Cost: Negligible (1GB RAM costs < $1/month)
│
└─ Monitoring metrics:
    ├─ Track query time by geohash
    ├─ Identify new dense areas automatically
    ├─ Build QuadTree proactively when area exceeds threshold
    └─ Alert if query time > 50ms

Capacity Impact Summary:
├─ Memory per server: +1GB for dense area caching
├─ CPU per server: +10% for QuadTree traversal
├─ Storage: +1GB for QuadTree indexes
├─ Overall cost impact: < 5% increase
└─ Performance benefit: 96% improvement in dense areas
```

### 🤔 Think About It

1. **For Beginners:** If you double the number of businesses from 100M to 200M, does that mean you need double the servers? Why or why not? (Hint: Think about caching and request/response sizes)

2. **For Intermediate:** You estimate needing 20K QPS capacity, but your average is only 3.5K QPS. Isn't that wasteful? How would you optimize costs while still handling peaks?

3. **For Advanced:** Your CFO says "We're spending $8M/year on infrastructure but only $3M on engineering. Can't we just rent more engineers and use less infrastructure?" How do you respond with data?

### ✅ Key Takeaways

- **Always calculate peak, not average**: Systems must handle peak load - use 3-5x multiplier for proximity services
- **Storage grows with photos**: 100M businesses = 2PB of photo storage, but only 100GB of business data
- **Geographic distribution matters**: Deploy in multiple regions to reduce latency and distribute load
- **High-density areas are edge cases**: Manhattan-style density requires special handling (QuadTree)
- **Costs scale sublinearly**: 10x growth doesn't mean 10x cost - economies of scale kick in
- **Show your math in interviews**: Writing calculations step-by-step demonstrates structured thinking
- **Consider all storage tiers**: Database vs CDN vs cache have vastly different cost profiles

### 🎯 Practice Exercise

**Scenario:** You're designing a proximity service for a food delivery platform (like DoorDash) in a major metropolitan area.

**Your Task:**
1. Estimate daily QPS if you have 5M active users and each user orders 0.3 times per day
2. Calculate storage needed for 100K restaurants with 5 photos each
3. Estimate bandwidth for delivering menu photos (assume 40% of searches view photos)
4. How would your estimates change during a major sporting event (Super Bowl) when orders spike 10x in a 3-hour window?

**Bonus Challenge:** The city has 100K restaurants but 80% are concentrated in downtown (20 km²). How does this affect your capacity planning?

---

