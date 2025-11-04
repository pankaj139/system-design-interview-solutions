# Food Delivery System Design (Uber Eats/DoorDash)

**Difficulty Level:** ⭐⭐⭐⭐⭐ Expert  
**Tags:** `Food Delivery`, `Real-time Matching`, `Geospatial`, `Order Management`, `Multi-party System`, `Route Optimization`, `Dynamic Pricing`, `WebSocket`, `State Machine`, `Distributed Transactions`

**File Purpose:** Interactive, multi-level learning resource for designing a large-scale food delivery platform. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that handles 10M daily orders across 500K restaurants and 1M delivery drivers in 100+ cities globally, achieving <30 second order assignment time, <1 hour delivery time, and 99.9% uptime.

**Author:** System Design Documentation  
**Created:** November 4, 2025  
**Last Updated:** November 4, 2025  
**Recent Updates:** Initial creation with educational template format, difficulty level and comprehensive tags

---

## 🎓 Welcome to Food Delivery System Design!

### What You're Going to Build

Imagine creating a platform like Uber Eats that coordinates millions of meals daily across a complex three-sided marketplace. When a customer orders pad thai from their favorite restaurant, your system must instantly match the order with the optimal delivery driver, coordinate real-time preparation and pickup, calculate dynamic delivery fees based on distance and demand, track the driver's location every second, handle payment splits between customer, restaurant, and driver, and deliver hot food within 30-45 minutes—all while managing 10 million such orders per day across hundreds of cities.

By the end of this learning journey, you'll understand how to design a production-grade food delivery platform that:
- **Processes 10M daily orders** across 500K restaurants and 1M active delivery drivers (115 orders/second average, 1,150/sec peak)
- **Matches orders to drivers in <30 seconds** using intelligent geospatial algorithms and real-time availability tracking
- **Tracks real-time location** of 200K concurrent deliveries with 1-second GPS updates via WebSocket
- **Calculates dynamic pricing** based on distance, demand, weather, and traffic in <100ms
- **Achieves 99.9% uptime** (only 43 minutes of downtime per month) with global multi-region deployment
- **Handles complex state machines** managing orders through 12+ states from placement to delivery
- **Optimizes delivery routes** in real-time considering traffic, multiple pickups, and customer wait times
- **Processes $50M+ daily transaction volume** with fraud detection and automated commission settlements

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (6-8 hours)
├─ Learn fundamental three-sided marketplace concepts
├─ Understand order lifecycle and state management
├─ Build intuition with restaurant/delivery analogies
└─ Perfect for: New to system design or food delivery platforms

🟡 INTERMEDIATE LEVEL (8-10 hours)  
├─ Master interview techniques for multi-party systems
├─ Learn geospatial matching and routing algorithms
├─ Practice trade-off analysis (consistency vs availability)
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-14 hours)
├─ Production considerations at Uber Eats scale
├─ Real-time optimization and ML-powered dispatch
├─ Handle edge cases (driver cancellations, restaurant delays)
└─ Perfect for: Senior engineers and system architects
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of databases (what is a table, what is an index)
- Familiarity with REST APIs
- No prior system design experience needed!

**For Intermediate:**
- Understanding of distributed systems basics
- Knowledge of caching and message queues
- Familiarity with geospatial concepts (latitude, longitude)
- Basic understanding of WebSockets for real-time communication

**For Advanced:**
- Experience with microservices architecture
- Understanding of eventual consistency and distributed transactions
- Knowledge of optimization algorithms (Dijkstra, Hungarian algorithm)
- Familiarity with ML model serving infrastructure

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:
1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context with Uber Eats examples
3. **Multi-Level Content** - Tailored explanations for your level
4. **Real-World Examples** - How Uber Eats, DoorDash, Grubhub actually do it
5. **Think About It** - Questions to deepen understanding
6. **Key Takeaways** - Summary of main points

---

## Table of Contents

1. [Requirements & Planning](#1-requirements--planning)
2. [Capacity Planning & Calculations](#2-capacity-planning--calculations)
3. [System Architecture Overview](#3-system-architecture-overview)
4. [Database Design](#4-database-design)
5. [API Design](#5-api-design)
6. [Order Management & State Machine](#6-order-management--state-machine)
7. [Real-Time Driver Matching](#7-real-time-driver-matching)
8. [Location Tracking & ETA Calculation](#8-location-tracking--eta-calculation)
9. [Dynamic Pricing & Delivery Fees](#9-dynamic-pricing--delivery-fees)
10. [Restaurant Catalog & Menu Management](#10-restaurant-catalog--menu-management)
11. [Payment Processing & Settlement](#11-payment-processing--settlement)
12. [Scalability & Performance](#12-scalability--performance)
13. [Security & Fraud Prevention](#13-security--fraud-prevention)
14. [Monitoring & Observability](#14-monitoring--observability)
15. [Design Trade-Offs & Decisions](#15-design-trade-offs--decisions)
16. [Interview Preparation Guide](#16-interview-preparation-guide)
17. [Putting It All Together](#putting-it-all-together)
18. [Next Steps & Resources](#next-steps--resources)

---

## 1. Requirements & Planning

### What You'll Learn
- How to gather requirements for a three-sided marketplace
- Functional vs non-functional requirements
- How to identify core MVP features vs nice-to-haves
- How to ask the right clarifying questions in interviews

### Why This Matters
Unlike simple two-sided systems (like ride-sharing with just drivers and riders), food delivery involves three parties: customers, restaurants, and drivers. Each party has different needs, incentives, and technical requirements. Uber Eats processes 10M+ orders daily, meaning requirements must account for this complexity at massive scale. Missing a key requirement could lead to system failure when a restaurant is slow, a driver cancels, or demand surges 10x during lunch rush.

---

### 🟢 Beginner Level: Understanding the Food Delivery Ecosystem

**Real-World Analogy:**

Think of a food delivery platform like a sophisticated restaurant coordination center combined with a taxi dispatch system:

1. **The Menu Board (Restaurant Catalog):** Like a giant digital menu board showing all available restaurants, their dishes, prices, and preparation times
2. **The Order Ticket (Order Management):** Like traditional restaurant order tickets, but digital and tracking multiple states (received, preparing, ready, picked up, delivered)
3. **The Dispatcher (Matching System):** Like a restaurant manager assigning delivery drivers, but automated and considering 50+ factors (distance, driver rating, traffic, order value)
4. **The Delivery Fleet (Driver Network):** Like a fleet of delivery vehicles, but dynamic—drivers come online/offline throughout the day
5. **The Cash Register (Payment System):** Like a POS system, but splitting payment three ways (customer pays, platform takes commission, restaurant and driver get paid)

**The Three Sides of the Marketplace:**

```text
CUSTOMER                    PLATFORM                    RESTAURANT
   ↓                           ↓                            ↓
Browse menus    ←──────→  Catalog service  ←──────→  Menu management
Place order     ←──────→  Order service    ←──────→  Accept/prepare order
Track delivery  ←──────→  Tracking service ←──────→  Update status
Pay for food    ←──────→  Payment service  ←──────→  Receive payment
                              ↓
                          DELIVERY DRIVER
                              ↓
                      Get assigned orders
                      Navigate to pickup
                      Deliver to customer
                      Receive payment
```

**Core User Flows:**

1. **Customer Journey:**
   - Open app → See nearby restaurants (based on GPS location)
   - Browse menu → Add items to cart
   - Checkout → Pay with credit card
   - Track order → See driver location on map
   - Receive food → Rate restaurant and driver

2. **Restaurant Journey:**
   - Receive order notification → Accept order
   - Prepare food → Mark as ready for pickup
   - Hand food to driver → Confirm handoff
   - Receive payment (next day settlement)

3. **Driver Journey:**
   - Go online → Receive order assignment
   - Navigate to restaurant → Confirm arrival
   - Pickup food → Confirm pickup
   - Navigate to customer → Confirm delivery
   - Receive payment (instant or weekly)

---

### 🟡 Intermediate Level: Functional & Non-Functional Requirements

**User Stories:**

**As a customer, I want to:**
- Browse restaurants within 5 km radius with <500ms load time
- Search for specific cuisines, dishes, or restaurants with autocomplete
- See accurate delivery time estimates (±5 minutes accuracy)
- Track my order and driver's location in real-time
- Modify or cancel my order within 2 minutes of placement
- Schedule orders for later delivery (up to 7 days advance)
- Apply promotions and discounts automatically
- Save favorite restaurants and reorder past meals
- Rate and review restaurants and drivers
- Contact support via chat if issues arise

**As a restaurant, I want to:**
- Manage my menu, prices, and availability in real-time
- Accept or reject orders based on kitchen capacity
- Update estimated preparation time dynamically
- Pause accepting new orders during rush or technical issues
- View order history and analytics (popular items, peak hours)
- Manage multiple locations from single dashboard
- Receive payments with clear commission breakdowns
- Set delivery radius and minimum order amounts

**As a delivery driver, I want to:**
- Receive order assignments based on my location
- See route to restaurant and then to customer
- Accept or reject assignments based on distance/payout
- Handle multiple deliveries in one trip (batching)
- Update my online/offline status anytime
- View earnings in real-time and withdraw instantly
- Navigate with in-app GPS or export to Google Maps
- Contact customer or restaurant if needed
- Report issues (customer not home, wrong address, etc.)

**As a platform operator, I want to:**
- Handle 10M orders per day with 99.9% uptime
- Match orders to drivers in <30 seconds
- Ensure food is delivered within promised time (typically 30-45 min)
- Dynamically price delivery fees based on supply/demand
- Detect and prevent fraud (fake orders, payment fraud)
- Scale to 100+ cities across multiple countries
- Monitor system health and performance in real-time
- A/B test new features (matching algorithms, pricing strategies)

---

**Functional Requirements (MVP):**

**1. Restaurant Management:**
- Restaurant onboarding and verification
- Menu CRUD operations (create, read, update, delete)
- Item availability toggle (mark items as sold out)
- Restaurant hours and delivery radius configuration
- Order acceptance/rejection
- Preparation time estimation and updates

**2. Order Management:**
- Order placement with real-time cart calculations
- Order state machine (placed → confirmed → preparing → ready → picked_up → in_transit → delivered)
- Order modification (within time window)
- Order cancellation (customer, restaurant, or driver initiated)
- Order history and receipts
- Scheduled orders (order now, deliver later)

**3. Driver Matching & Assignment:**
- Real-time driver availability tracking
- Intelligent order-to-driver matching (geospatial + heuristics)
- Order reassignment if driver cancels
- Batch delivery optimization (multiple orders in one trip)
- Driver earnings calculation

**4. Real-Time Tracking:**
- Live driver location updates (1-second intervals)
- ETA calculation and updates
- Push notifications for order status changes
- Customer-driver communication (chat, call)

**5. Payment Processing:**
- Customer payment (credit card, digital wallets, cash on delivery)
- Commission calculation (typically 20-30% to platform)
- Restaurant settlement (daily or weekly)
- Driver payout (instant or weekly)
- Refund processing

**6. Search & Discovery:**
- Restaurant search by name, cuisine, dish
- Filter by rating, delivery time, price range
- Sort by popularity, rating, delivery fee
- Personalized recommendations

---

**Non-Functional Requirements:**

**Performance:**
- Order placement: <200ms response time
- Driver matching: <30 seconds
- Location updates: <1 second latency
- Menu loading: <500ms
- Search results: <300ms
- Support 115 orders/second average, 1,150 orders/second peak (10x during lunch/dinner)

**Scalability:**
- 10M orders per day
- 500K active restaurants
- 1M active delivery drivers
- 50M active customers
- Support 200K concurrent deliveries
- Scale to 100+ cities globally

**Availability:**
- 99.9% uptime for core services (only 43 minutes downtime per month)
- 99.99% uptime for payment processing
- Graceful degradation when components fail
- No data loss for orders or transactions

**Reliability:**
- Zero lost orders (all orders must be tracked)
- Guaranteed payment settlement
- Automatic retry for failed operations
- Disaster recovery with <1 hour RTO (Recovery Time Objective)

**Consistency:**
- Strong consistency for inventory (prevent double-ordering sold-out items)
- Strong consistency for payments (no double charges or lost payments)
- Eventual consistency acceptable for driver locations (1-2 second delay ok)
- Eventual consistency acceptable for restaurant ratings/reviews

**Latency:**
- P50 (50th percentile): <100ms for API calls
- P95: <300ms for API calls
- P99: <500ms for API calls
- Real-time updates: <1 second end-to-end

---

### 🔴 Advanced Level: Clarifying Questions & Assumptions

**In an interview, ask these questions to scope the problem:**

**Scale & Geography:**
- What's our target scale? (Daily orders, number of restaurants, drivers, cities)
- Are we launching in one city first or multiple cities simultaneously?
- Do we need to support international operations (multiple currencies, languages)?
- What's our expected growth rate? (2x per year? 10x in 3 years?)

**User Experience:**
- What's our target delivery time? (30 min? 45 min? 1 hour?)
- Do we need to support scheduled deliveries (order now, deliver later)?
- Should we allow order modifications after placement?
- Do we need to support group orders (multiple people ordering together)?

**Marketplace Dynamics:**
- What's our commission model? (Percentage-based? Fixed fee?)
- Do we guarantee minimum wage for drivers?
- Can restaurants set their own delivery radius?
- Do we allow exclusive restaurant partnerships (only on our platform)?

**Technical Constraints:**
- Do we need to integrate with existing POS systems?
- Are there any compliance requirements (PCI DSS for payments, GDPR for data)?
- What's our budget for infrastructure? (Will inform technology choices)
- Do we need offline support (for drivers in areas with poor connectivity)?

**Feature Priorities:**
- Is this an MVP or full-featured product?
- What's the priority: speed (time to market) vs scale (handling massive traffic)?
- Should we focus on customer experience or driver efficiency first?

---

**Assumptions (For This Design):**

**Scale:**
- 10M orders per day (115 orders/second average, 1,150/sec peak)
- 500K restaurants (20% active at any time = 100K)
- 1M drivers (20% online at any time = 200K)
- 50M total users (10M DAU)
- 100 cities across 10 countries

**Usage Patterns:**
- 60% of orders during peak hours (lunch: 11AM-2PM, dinner: 5PM-9PM)
- Average order value: $25
- Average delivery time: 35 minutes
- Average delivery distance: 5 km
- 10% order cancellation rate (customer, restaurant, or driver)
- 5% order modification rate

**Geo-Distribution:**
- North America: 50% of traffic
- Europe: 30% of traffic
- Asia: 20% of traffic
- Need multi-region deployment for low latency

**Business Model:**
- Platform commission: 25% from restaurant
- Delivery fee: $2-$8 based on distance and demand
- Service fee: 15% from customer
- Driver earns 100% of delivery fee + tips

**Feature Scope:**
- MVP includes: order placement, driver matching, tracking, payments
- Phase 2: scheduled deliveries, group orders, subscriptions
- Phase 3: ghost kitchens, dine-in ordering, grocery delivery

---

### Key Takeaways

✅ Food delivery is a **three-sided marketplace** (customer, restaurant, driver) requiring careful balance

✅ Core user flows involve **complex state machines** (orders transition through 12+ states)

✅ **Real-time requirements** dominate: driver locations (1-sec updates), order assignments (<30 sec), ETAs (continuous recalculation)

✅ **Non-functional requirements are critical:** 99.9% uptime, strong consistency for payments/inventory, eventual consistency for locations/reviews

✅ **Ask clarifying questions** in interviews: scale, geography, commission model, feature priorities

✅ **Make reasonable assumptions** but state them clearly (10M daily orders, 35 min avg delivery time, 25% platform commission)

---

**Think About It:**
- How would you handle a flash sale at a popular restaurant (100 orders in 1 minute)?
- What happens if a driver's phone dies mid-delivery?
- How do you prevent restaurants from accepting more orders than they can handle?
- Should order assignment prioritize driver proximity or customer wait time?

---

## 2. Capacity Planning & Calculations

### What You'll Learn
- How to perform back-of-the-envelope calculations for a food delivery system
- How to estimate traffic, storage, bandwidth, and compute resources
- How to calculate costs for running a large-scale platform
- How to plan for peak load (10x traffic during lunch/dinner rush)

### Why This Matters
Uber Eats processes 10M+ orders daily across millions of restaurants and drivers. Without accurate capacity planning, you could over-provision (wasting millions in infrastructure costs) or under-provision (causing outages during peak hours and losing customer trust). A single hour of downtime during dinner rush could cost $1M+ in lost revenue. These calculations help you right-size infrastructure, estimate budgets, and plan for growth.

---

### 🟢 Beginner Level: Basic Traffic Estimation

**Understanding Orders Per Second (QPS/TPS):**

Let's start simple. If we have 10 million orders per day, how many orders per second is that?

```text
Daily Orders: 10,000,000
Seconds per day: 24 hours × 60 minutes × 60 seconds = 86,400 seconds

Average Orders Per Second (OPS): 10,000,000 / 86,400 = 115.7 ≈ 116 orders/second
```

But wait! Orders don't come in evenly throughout the day. People order food during lunch and dinner, not at 3 AM.

**Peak Hours (Lunch & Dinner):**
- Lunch: 11 AM - 2 PM (3 hours)
- Dinner: 5 PM - 9 PM (4 hours)
- Total peak hours: 7 hours = 25,200 seconds
- Orders during peak hours: 60% of daily orders = 6,000,000 orders

```text
Peak Orders Per Second: 6,000,000 / 25,200 = 238 orders/second

But even within peak hours, there are spikes (everyone orders at 12 PM and 7 PM):
Peak Spike: 238 × 5 = 1,190 ≈ 1,200 orders/second
```

**Why This Matters:**
Your system must handle 1,200 orders/second during peak, even though average is only 116/sec. This is a **10x difference**—designing for average would mean crashes during dinner rush!

---

**Estimating Database Operations:**

For each order, how many database operations occur?

```text
Order Placement (Write Operations):
1. Create order record → 1 write
2. Update restaurant inventory → 1 write
3. Create payment transaction → 1 write
4. Create delivery task → 1 write
Total: 4 writes per order

Order Tracking (Read Operations):
1. Customer checks order status every 30 seconds for 35 minutes → 70 reads
2. Driver app updates location every second → 2,100 reads (35 min × 60 sec)
3. Restaurant checks order queue every minute → 35 reads
Total: ~2,200 reads per order

At 1,200 orders/second peak:
Write Operations: 1,200 × 4 = 4,800 writes/second
Read Operations: 1,200 × 2,200 = 2,640,000 reads/second (this is high!)
```

**Reality Check:** 2.6M reads/second seems too high. In practice, we use **caching** to reduce database reads by 95-99%. With 99% cache hit rate:
- Cached reads (served from Redis): 2,640,000 × 0.99 = 2,613,600 reads/sec
- Database reads: 2,640,000 × 0.01 = 26,400 reads/sec

Much more reasonable!

---

### 🟡 Intermediate Level: Storage, Bandwidth & Cost Calculations

**Storage Estimation:**

Let's calculate how much data we store for 10M daily orders over 1 year:

```text
ORDER DATA:
Per Order:
- Order details (JSON): 2 KB (items, quantities, prices, addresses)
- Customer info (reference): 0.1 KB (just user_id)
- Restaurant info (reference): 0.1 KB (just restaurant_id)
- Driver info (reference): 0.1 KB (just driver_id)
- Payment info: 0.5 KB (transaction details)
- Total per order: ~3 KB

Daily: 10M orders × 3 KB = 30 GB
Monthly: 30 GB × 30 = 900 GB
Yearly: 900 GB × 12 = 10.8 TB

With 3x replication (for high availability): 10.8 TB × 3 = 32.4 TB per year
```

```text
LOCATION DATA (GPS Coordinates):
Per delivery:
- 35 minutes delivery time × 60 seconds = 2,100 GPS updates
- Each update: 50 bytes (lat, lon, timestamp, accuracy)
- Per delivery: 2,100 × 50 bytes = 105 KB

Daily: 10M deliveries × 105 KB = 1.05 TB
Monthly: 1.05 TB × 30 = 31.5 TB
Yearly: 31.5 TB × 12 = 378 TB

Location data retention: 30 days (for dispute resolution)
Storage needed: 31.5 TB × 1.5 (indexes + overhead) = 47 TB (hot storage)
```

```text
RESTAURANT CATALOG:
- 500K restaurants × 50 items per menu × 2 KB per item = 50 GB
- Photos: 500K restaurants × 10 photos × 100 KB = 500 GB
- Total: ~550 GB (relatively small, heavily cached)
```

```text
USER DATA:
- 50M users × 1 KB (profile, preferences) = 50 GB
- Order history references: included in order data above
```

**Total Storage (1st Year):**
- Orders: 32.4 TB
- Location data (30-day rolling): 47 TB
- Restaurant catalog: 0.55 TB
- User data: 0.05 TB
- **Total: ~80 TB**

**Storage Over 5 Years:**
- Orders (cumulative): 32.4 TB × 5 = 162 TB
- Location data (30-day rolling): 47 TB (constant)
- Other data: negligible growth
- **Total: ~210 TB**

---

**Bandwidth Estimation:**

```text
INBOUND BANDWIDTH (Data Coming Into System):

Order Placement:
- 1,200 orders/sec × 10 KB (request payload) = 12 MB/sec

Location Updates:
- 200K concurrent drivers × 1 update/sec × 100 bytes = 20 MB/sec

Menu Updates:
- 100K active restaurants × 1 update/minute × 5 KB / 60 = 8.3 MB/sec

Total Inbound: 12 + 20 + 8.3 = 40.3 MB/sec ≈ 350 Gbps (peak)
```

```text
OUTBOUND BANDWIDTH (Data Going Out From System):

Order Status Updates:
- 200K concurrent orders × 2 updates/sec × 1 KB = 400 MB/sec

Driver Location Updates (to customers):
- 200K concurrent orders × 1 update/sec × 100 bytes = 20 MB/sec

Menu Browsing:
- 1M concurrent users × 0.1 requests/sec × 50 KB = 5 GB/sec

Restaurant Photos (CDN):
- 1M users × 0.05 photos/sec × 100 KB = 5 GB/sec

Total Outbound: 0.4 + 0.02 + 5 + 5 = 10.42 GB/sec ≈ 83 Gbps (peak)
```

**Note:** Most outbound traffic (photos, menus) served from CDN, so actual server bandwidth is much lower (~10 Gbps).

---

**Infrastructure Cost Estimation (Monthly AWS Pricing):**

```text
COMPUTE (Application Servers):
- Order Service: 200 instances × c5.2xlarge ($0.34/hr) × 730 hrs = $49,640
- Driver Matching: 100 instances × c5.4xlarge ($0.68/hr) × 730 hrs = $49,640
- Location Service: 150 instances × c5.2xlarge × 730 hrs = $37,230
- Restaurant Service: 50 instances × c5.xlarge ($0.17/hr) × 730 hrs = $6,205
- API Gateway: 100 instances × c5.xlarge × 730 hrs = $12,410
Total Compute: ~$155,000/month
```

```text
DATABASES:
- PostgreSQL (RDS): 20 instances × db.r5.4xlarge ($2.08/hr) × 730 hrs = $30,368
- Cassandra (for location data): 50 nodes × i3.2xlarge ($0.624/hr) × 730 hrs = $22,776
- Redis (caching): 30 instances × r5.2xlarge ($0.504/hr) × 730 hrs = $11,038
Total Databases: ~$64,000/month
```

```text
STORAGE:
- S3 (photos, backups): 500 TB × $0.023/GB = $11,500
- EBS (database volumes): 100 TB × $0.10/GB = $10,000
Total Storage: ~$21,500/month
```

```text
BANDWIDTH:
- Outbound transfer: 2 PB × $0.05/GB = $100,000
- CDN (CloudFront): 3 PB × $0.02/GB = $60,000
Total Bandwidth: ~$160,000/month
```

```text
OTHER SERVICES:
- Kafka (MSK): 20 brokers × $0.21/hr × 730 hrs = $3,066
- Elasticsearch: 10 nodes × $0.50/hr × 730 hrs = $3,650
- Load Balancers (ALB): 50 × $0.0225/hr × 730 hrs = $821
- Lambda (notifications): 10B invocations × $0.20/1M = $2,000
Total Other: ~$10,000/month
```

**TOTAL MONTHLY COST: ~$410,000**  
**YEARLY COST: ~$4.9M**

**Revenue Context:**
- 10M orders/day × $25 avg order × 25% commission = $62.5M daily revenue
- Monthly revenue: $62.5M × 30 = $1.875 billion
- Infrastructure cost: $410K / $1.875B = **0.02% of revenue** (very efficient!)

---

### 🔴 Advanced Level: Capacity Planning for Growth & Peak Load

**Growth Projections:**

```text
Year 1: 10M orders/day, $410K/month infrastructure cost
Year 2: 20M orders/day (2x growth), $820K/month (linear scaling with better optimization)
Year 3: 40M orders/day (2x growth), $1.5M/month (sublinear due to economies of scale)
Year 4: 70M orders/day (1.75x growth), $2.5M/month
Year 5: 100M orders/day (1.43x growth), $3.5M/month

Actual Uber Eats: ~10M orders/day globally (2023 data)
```

**Peak Load Handling (10x Traffic During Lunch/Dinner Rush):**

```text
Normal Load: 116 orders/second
Peak Load: 1,200 orders/second (10x)

Strategies:
1. Auto-Scaling: Scale compute instances 5x during peak (2 hr ramp-up)
2. Caching: Increase cache TTL during peak (reduce DB load)
3. Request Throttling: Queue orders if system is overloaded (better than crash)
4. Async Processing: Decouple order placement from driver assignment
5. CDN: Serve 99% of static content (menus, photos) from edge locations

Cost Impact:
- Peak hour compute: $155K × 5 = $775K/month (only during 7 peak hours)
- Adjusted cost: $155K + ($775K - $155K) × 7/24 = $336K/month
- Savings vs always running at peak: $775K - $336K = $439K/month
```

**Disaster Recovery Planning:**

```text
RTO (Recovery Time Objective): 1 hour
RPO (Recovery Point Objective): 5 minutes

Multi-Region Deployment:
- Primary: US-East (50% traffic)
- Secondary: EU-West (30% traffic)
- Tertiary: AP-Southeast (20% traffic)

Failover Strategy:
- Active-Active: All regions serve traffic simultaneously
- Cross-region replication: Cassandra (eventual consistency), PostgreSQL (async replication)
- Traffic shift: Route53 health checks + weighted routing
- Data loss: <5 minutes (last 5 min of orders might need manual reconciliation)

Cost: 3x infrastructure = $410K × 3 = $1.23M/month
Reality: ~2.5x due to shared services (DNS, CDN, databases) = $1M/month
```

---

**Database Sharding Strategy:**

```text
SHARDING BY CITY:
- 100 cities → 100 shards
- Each shard: 100K orders/day → 1.2 orders/second average
- Allows geo-colocated data (low latency)
- Challenge: Uneven distribution (NYC has 10x more orders than Topeka)

SHARDING BY ORDER_ID:
- Consistent hashing: order_id % 256 shards
- Even distribution across shards
- Each shard: 39K orders/day → 0.45 orders/second
- Challenge: Cross-shard queries (e.g., "get all orders for restaurant X")

HYBRID APPROACH (Uber Eats Uses This):
- Primary sharding by city (geo-locality)
- Secondary sharding by order_id within city (even distribution)
- Example: New York → 10 shards, Topeka → 1 shard
- Benefits: Locality + even distribution + flexible scaling
```

---

**Cache Sizing:**

```text
REDIS CACHE TIERS:

Tier 1 - Hot Data (1 minute TTL):
- Active orders (200K concurrent × 3 KB) = 600 MB
- Driver locations (200K drivers × 100 bytes) = 20 MB
- Total: ~1 GB (replicated 3x) = 3 GB

Tier 2 - Warm Data (10 minute TTL):
- Restaurant menus (100K active × 50 items × 2 KB) = 10 GB
- User sessions (1M concurrent × 1 KB) = 1 GB
- Total: ~11 GB (replicated 3x) = 33 GB

Tier 3 - Cold Data (1 hour TTL):
- Restaurant catalog (500K × 100 KB) = 50 GB
- User profiles (10M DAU × 1 KB) = 10 GB
- Total: ~60 GB (replicated 3x) = 180 GB

Total Redis: 3 + 33 + 180 = 216 GB
With 50% headroom: 216 × 1.5 = 324 GB
Redis instances: 324 GB / 64 GB per instance = 6 instances (r5.2xlarge)

Cost: 6 × $0.504/hr × 730 hrs = $2,207/month
```

---

### Key Takeaways

✅ **Design for peak load, not average:** 10x traffic difference between off-peak and peak hours

✅ **Caching is critical:** 99% cache hit rate reduces database load from 2.6M to 26K reads/second

✅ **Storage grows predictably:** ~80 TB first year, ~210 TB after 5 years (mostly location data)

✅ **Bandwidth is expensive:** $160K/month, but CDN reduces server bandwidth by 90%

✅ **Infrastructure cost is tiny vs revenue:** 0.02% of revenue ($410K vs $1.875B monthly)

✅ **Multi-region deployment is necessary:** For low latency globally and disaster recovery

✅ **Auto-scaling saves money:** Scale up during peak hours, scale down during off-peak (saves $439K/month)

---

**Think About It:**
- If we grow 10x in 2 years, will our database sharding strategy still work?
- How much would it cost to store location data forever (instead of 30 days)?
- What if we offered free delivery (reducing revenue but increasing orders 3x)?
- Could we reduce infrastructure cost by serving menus from static files instead of database?

---

## 3. System Architecture Overview

### What You'll Learn
- High-level architecture of a food delivery platform
- Key microservices and their responsibilities
- Data flow from order placement to delivery
- How components interact via APIs and message queues

### Why This Matters
A food delivery system involves 10+ microservices coordinating in real-time. Understanding the architecture helps you reason about scalability bottlenecks, failure modes, and latency requirements. Uber Eats uses a microservices architecture where each service is independently deployable, scalable, and fault-tolerant. One service crashing (e.g., recommendation engine) shouldn't bring down order placement.

---

### 🟢 Beginner Level: High-Level Components

**The Big Picture:**

Think of the system like a restaurant with specialized staff:
- **Front Desk (API Gateway):** Greets customers, validates requests, routes to right department
- **Order Takers (Order Service):** Take orders, validate items, send to kitchen
- **Kitchen Manager (Restaurant Service):** Manage menus, accept orders, track preparation
- **Dispatcher (Matching Service):** Assign delivery drivers to orders
- **Delivery Coordinator (Location Service):** Track drivers, update customers
- **Cashier (Payment Service):** Process payments, handle refunds
- **Accountant (Settlement Service):** Pay restaurants and drivers

```text
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT APPS                              │
│  (Customer iOS/Android/Web, Restaurant Dashboard, Driver App)   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       API GATEWAY (Kong)                         │
│          (Authentication, Rate Limiting, Routing)                │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐      ┌──────────────┐
│   Order      │    │  Restaurant  │      │   Driver     │
│   Service    │    │   Service    │      │   Service    │
└──────────────┘    └──────────────┘      └──────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
                    ┌──────────────┐
                    │   Message    │
                    │   Queue      │
                    │   (Kafka)    │
                    └──────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐    ┌──────────────┐      ┌──────────────┐
│   Matching   │    │   Location   │      │   Payment    │
│   Service    │    │   Service    │      │   Service    │
└──────────────┘    └──────────────┘      └──────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
                    ┌──────────────┐
                    │  Databases   │
                    │ (PostgreSQL, │
                    │  Cassandra,  │
                    │   Redis)     │
                    └──────────────┘
```

**Simple Order Flow:**

1. Customer places order → **Order Service** validates and creates order
2. **Order Service** publishes "OrderPlaced" event to **Kafka**
3. **Matching Service** picks up event → finds best driver
4. **Driver Service** notifies driver → driver accepts
5. **Location Service** starts tracking driver → sends updates to customer
6. Driver picks up food → **Location Service** updates status
7. Driver delivers → **Payment Service** charges customer, pays restaurant & driver

---

### 🟡 Intermediate Level: Microservices Architecture

**Core Services:**

```text
┌────────────────────────────────────────────────────────────────────┐
│                         API GATEWAY LAYER                          │
├────────────────────────────────────────────────────────────────────┤
│ • Kong API Gateway (authentication, rate limiting, routing)        │
│ • GraphQL Federation (unified API for mobile apps)                │
│ • WebSocket Gateway (real-time location updates)                  │
└────────────────────────────────────────────────────────────────────┘
                                │
        ┌───────────────────────┴───────────────────────┐
        │                                               │
┌────────────────────┐                    ┌────────────────────┐
│  CUSTOMER DOMAIN   │                    │  RESTAURANT DOMAIN │
├────────────────────┤                    ├────────────────────┤
│ • User Service     │                    │ • Restaurant       │
│ • Search Service   │                    │   Service          │
│ • Cart Service     │                    │ • Menu Service     │
│ • Order Service    │                    │ • Inventory        │
│ • Notification     │                    │   Service          │
│   Service          │                    │ • Analytics        │
└────────────────────┘                    │   Service          │
        │                                 └────────────────────┘
        │                                         │
        │                 ┌───────────────────────┘
        │                 │
        ▼                 ▼
┌────────────────────────────────────────────────────────────────────┐
│                        CORE DOMAIN (Order Flow)                    │
├────────────────────────────────────────────────────────────────────┤
│ • Order Management Service (state machine, lifecycle)             │
│ • Matching Service (driver assignment algorithm)                  │
│ • Routing Service (ETA calculation, route optimization)           │
│ • Location Service (GPS tracking, geospatial queries)             │
│ • Payment Service (charges, refunds, settlements)                 │
└────────────────────────────────────────────────────────────────────┘
        │
        ▼
┌────────────────────┐
│   DRIVER DOMAIN    │
├────────────────────┤
│ • Driver Service   │
│ • Earnings Service │
│ • Fleet Management │
└────────────────────┘
        │
        ▼
┌────────────────────────────────────────────────────────────────────┐
│                      PLATFORM SERVICES                             │
├────────────────────────────────────────────────────────────────────┤
│ • Fraud Detection Service                                         │
│ • Promotion Service (coupons, discounts)                          │
│ • Customer Support Service (ticketing, chat)                      │
│ • Analytics & Reporting Service                                   │
│ • ML Services (recommendations, demand prediction, pricing)       │
└────────────────────────────────────────────────────────────────────┘
```

**Detailed Service Responsibilities:**

**1. Order Service:**
- Create, read, update, delete orders
- Validate order (items available, address valid, payment method valid)
- Calculate order total (items + taxes + fees)
- State machine orchestration (placed → confirmed → preparing → etc.)
- Order modification and cancellation
- Scheduled order management

**Tech Stack:** Node.js, PostgreSQL (for ACID transactions), Redis (order cache)

**2. Matching Service:**
- Listen to "OrderPlaced" events from Kafka
- Query available drivers within radius (using geospatial queries)
- Calculate driver scores (distance, rating, acceptance rate, earnings today)
- Assign order to best driver (or batch multiple orders to one driver)
- Handle driver rejections (reassign to next best driver)
- Optimize for delivery time and cost

**Tech Stack:** Python (scikit-learn for ML models), Redis (driver availability cache), PostgreSQL (driver data)

**3. Location Service:**
- Ingest GPS coordinates from driver apps (1-second intervals)
- Store in time-series database (Cassandra)
- Broadcast location updates to customers via WebSocket
- Calculate ETA based on current location, traffic, route
- Detect anomalies (driver going wrong direction)

**Tech Stack:** Go (for high-throughput ingestion), Cassandra (time-series data), Redis (latest location cache), WebSocket (Socket.IO)

**4. Restaurant Service:**
- Restaurant onboarding and profile management
- Menu CRUD operations
- Inventory management (mark items as sold out)
- Order acceptance/rejection logic
- Business hours and delivery radius configuration
- Restaurant analytics dashboard

**Tech Stack:** Java Spring Boot, PostgreSQL, Elasticsearch (menu search)

**5. Payment Service:**
- Charge customer via Stripe/Braintree
- Calculate commissions (platform takes 25%, restaurant gets 75%)
- Calculate delivery fee (distance-based, demand-based)
- Driver payout (instant via Stripe Connect)
- Refund processing
- Transaction ledger (double-entry bookkeeping)

**Tech Stack:** Java (for financial accuracy), PostgreSQL (with ACID), Stripe API

**6. Notification Service:**
- Send push notifications (Firebase FCM, Apple APNS)
- Send SMS (Twilio)
- Send emails (SendGrid)
- Event-driven: listens to Kafka events (OrderConfirmed, DriverAssigned, OutForDelivery, Delivered)
- User preference management (don't send SMS if user disabled it)

**Tech Stack:** Node.js, Kafka Consumer, Redis (user preferences cache)

---

**Data Flow Example: Customer Orders Burger**

```text
Step 1: ORDER PLACEMENT
Customer → API Gateway → Order Service
├─ Validate customer (exists, has payment method)
├─ Validate restaurant (open, delivers to address)
├─ Validate items (burger is available, price is correct)
├─ Calculate total: $10 (burger) + $2 (tax) + $5 (delivery fee) = $17
├─ Create order in PostgreSQL (status: PLACED)
└─ Publish "OrderPlaced" event to Kafka

Step 2: RESTAURANT CONFIRMATION
Restaurant Service (listening to Kafka)
├─ Receive "OrderPlaced" event
├─ Send push notification to restaurant tablet
├─ Restaurant accepts order (within 30 seconds)
├─ Update order status: PLACED → CONFIRMED
└─ Publish "OrderConfirmed" event to Kafka

Step 3: DRIVER MATCHING
Matching Service (listening to Kafka)
├─ Receive "OrderConfirmed" event
├─ Query Redis: get all drivers within 5 km of restaurant
├─ Filter: only drivers who are online and available
├─ Score each driver: score = (1 / distance) × rating × 0.8 + acceptance_rate × 0.2
├─ Assign order to driver with highest score
├─ Update order status: CONFIRMED → DRIVER_ASSIGNED
└─ Publish "DriverAssigned" event to Kafka

Step 4: DRIVER ACCEPTANCE
Driver Service (listening to Kafka)
├─ Receive "DriverAssigned" event
├─ Send push notification to driver app
├─ Driver accepts (within 30 seconds)
├─ Update order status: DRIVER_ASSIGNED → DRIVER_ACCEPTED
└─ Publish "DriverAccepted" event to Kafka

Step 5: PREPARATION
Restaurant Service
├─ Kitchen prepares burger (20 minutes)
├─ Mark order as ready
├─ Update order status: DRIVER_ACCEPTED → READY_FOR_PICKUP
└─ Publish "ReadyForPickup" event to Kafka

Step 6: PICKUP
Location Service (tracking driver GPS)
├─ Driver arrives at restaurant (geofence trigger)
├─ Driver confirms pickup in app
├─ Update order status: READY_FOR_PICKUP → PICKED_UP
└─ Publish "PickedUp" event to Kafka

Step 7: DELIVERY
Location Service
├─ Track driver location (1-second GPS updates)
├─ Send location updates to customer via WebSocket
├─ Calculate ETA continuously (traffic-aware)
├─ Driver arrives at customer address (geofence trigger)
├─ Driver confirms delivery in app
├─ Update order status: PICKED_UP → DELIVERED
└─ Publish "Delivered" event to Kafka

Step 8: PAYMENT SETTLEMENT
Payment Service (listening to Kafka)
├─ Receive "Delivered" event
├─ Charge customer $17 via Stripe
├─ Calculate splits:
│  ├─ Restaurant: $10 × 0.75 = $7.50 (platform keeps 25% = $2.50)
│  ├─ Driver: $5 delivery fee + $0 tip = $5
│  └─ Platform: $2.50 (restaurant commission) + $2 (tax & service fee) = $4.50
├─ Transfer funds:
│  ├─ Restaurant: $7.50 (next-day settlement)
│  └─ Driver: $5 (instant payout via Stripe Connect)
└─ Store transaction in ledger
```

---

### 🔴 Advanced Level: Architecture Patterns & Design Decisions

**Event-Driven Architecture (Why Kafka?):**

```text
PROBLEM: Tight Coupling
Order Service → (calls) → Matching Service → (calls) → Location Service → (calls) → Payment Service
└─ If Location Service is down, entire order flow breaks
└─ Retry logic becomes complex (what if partial failure?)
└─ Scaling is hard (one slow service slows down everything)

SOLUTION: Event-Driven with Kafka
Order Service → publish("OrderPlaced") → Kafka
Matching Service → subscribe("OrderPlaced")
Location Service → subscribe("DriverAssigned")
Payment Service → subscribe("Delivered")

Benefits:
✅ Services are decoupled (one crash doesn't cascade)
✅ Async processing (order placement is fast, matching happens in background)
✅ Event replay (can reprocess events if service was down)
✅ Audit trail (every event is logged)
```

**CQRS (Command Query Responsibility Segregation):**

```text
PROBLEM: Read/Write Conflicts
Orders table:
- Heavy writes during order placement (1,200 orders/sec)
- Heavy reads for order tracking (200K concurrent orders checking status)
- Same database = lock contention, slow queries

SOLUTION: Separate Read and Write Databases
Write Path (PostgreSQL):
├─ Order placement → writes to PostgreSQL (ACID, strong consistency)
├─ Minimal reads (just to validate data)
└─ Optimized for writes (fewer indexes, batch inserts)

Read Path (Cassandra + Redis):
├─ Order updates → replicated to Cassandra (async, eventually consistent)
├─ Customer queries → served from Cassandra (fast reads, no locks)
├─ Hot data → cached in Redis (order status, driver location)
└─ Optimized for reads (denormalized, heavy indexes)

Consistency:
- Write to PostgreSQL → publish event to Kafka → consumer updates Cassandra
- Eventual consistency acceptable (customer sees "Preparing" 1 second after actual state change)
```

**Saga Pattern (Distributed Transactions):**

```text
PROBLEM: How to ensure atomicity across services?
Order placement involves:
1. Create order (Order Service)
2. Charge customer (Payment Service)
3. Reserve inventory (Restaurant Service)
4. Assign driver (Matching Service)

If any step fails, need to rollback previous steps. But services are distributed!

SOLUTION: Saga Pattern with Compensating Transactions
Choreography-Based Saga (using Kafka events):

Step 1: Order Service → create order
  ├─ Success → publish "OrderCreated"
  └─ Failure → return error to customer

Step 2: Payment Service → charge customer
  ├─ Success → publish "PaymentSuccess"
  └─ Failure → publish "PaymentFailed" → Order Service listens → cancel order

Step 3: Restaurant Service → reserve inventory
  ├─ Success → publish "InventoryReserved"
  └─ Failure → publish "InventoryFailed" → Payment Service listens → refund customer

Step 4: Matching Service → assign driver
  ├─ Success → publish "DriverAssigned"
  └─ Failure → publish "MatchingFailed" → trigger compensating transactions

Compensating Transactions:
- "PaymentFailed" → Order Service cancels order, notifies customer
- "InventoryFailed" → Payment Service refunds, Order Service cancels
- "MatchingFailed" → Payment Service refunds, Restaurant unreserves, Order Service cancels
```

**Multi-Region Deployment:**

```text
ARCHITECTURE: Active-Active Multi-Region

Region 1 (US-East): 50% traffic
Region 2 (EU-West): 30% traffic
Region 3 (AP-Southeast): 20% traffic

Data Partitioning:
- Orders: partitioned by city (US orders in US region, EU orders in EU region)
- Restaurants: stored in local region (reduced latency)
- Drivers: stored in local region (GPS updates have low latency)
- Users: globally replicated (user can order from any region)

Cross-Region Replication:
- PostgreSQL: async replication (5-10 second lag acceptable)
- Cassandra: multi-datacenter replication (eventual consistency)
- Redis: no cross-region sync (local cache only)

Failover:
- Route53 health checks → if US-East fails, route traffic to EU-West
- Orders in-flight → replayed from Kafka (event sourcing)
- RTO: <1 hour, RPO: <5 minutes
```

**API Gateway Pattern:**

```text
WHY API GATEWAY?
Without:
Customer App → calls 5 services directly → Order, Restaurant, User, Payment, Notification
├─ Customer must know 5 different URLs
├─ Customer must handle 5 different auth tokens
└─ No rate limiting (customer could DDoS services)

With Kong API Gateway:
Customer App → Kong → routes to correct service
├─ Single entry point (api.ubereats.com)
├─ Centralized authentication (JWT tokens)
├─ Rate limiting (100 requests/minute per user)
├─ Request/response transformation (add headers, compress)
├─ Caching (frequent requests cached at gateway)
└─ Load balancing (route to healthy instances)

Advanced Features:
- Circuit breaker (if service is down, return cached response)
- Retry with exponential backoff
- Request batching (combine multiple requests)
- GraphQL federation (combine multiple services into single GraphQL endpoint)
```

---

### Key Takeaways

✅ **Microservices architecture** enables independent scaling and deployment (Order Service scales differently than Payment Service)

✅ **Event-driven design** with Kafka decouples services and enables async processing

✅ **CQRS pattern** separates read and write paths (PostgreSQL for writes, Cassandra for reads)

✅ **Saga pattern** handles distributed transactions without 2-phase commit

✅ **API Gateway** provides single entry point with auth, rate limiting, and caching

✅ **Multi-region deployment** reduces latency and enables disaster recovery

---

**Think About It:**
- What if Kafka goes down during peak hours? How do orders get processed?
- Should we use orchestration-based saga (centralized) or choreography-based saga (decentralized)?
- How do we prevent duplicate orders if customer clicks "Place Order" twice?
- Should driver location updates go through Kafka or directly to Location Service?

---

## 4. Database Design

### What You'll Learn
- Database schema for a multi-sided marketplace
- SQL vs NoSQL trade-offs for different data types
- Indexing strategies for high-performance queries
- Data partitioning and sharding strategies
- Consistency requirements for different data types

### Why This Matters
A food delivery platform stores diverse data types: transactional (orders, payments), time-series (GPS locations), documents (restaurant menus), graphs (user relationships). Choosing the wrong database could mean slow queries (bad UX), data loss (lost revenue), or inability to scale. Uber Eats uses 5+ different databases, each optimized for specific access patterns.

---

### 🟢 Beginner Level: Core Tables and Relationships

**Key Entities:**

Think of databases like filing cabinets in a restaurant office:
- **Customer Cabinet:** Customer profiles, addresses, payment methods
- **Restaurant Cabinet:** Restaurant info, menus, hours, locations
- **Order Cabinet:** All orders, their items, status, timestamps
- **Driver Cabinet:** Driver profiles, vehicles, earnings, ratings
- **Delivery Cabinet:** Delivery assignments, routes, locations

```text
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│   CUSTOMER   │         │  RESTAURANT  │         │    DRIVER    │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ id (PK)      │         │ id (PK)      │         │ id (PK)      │
│ name         │         │ name         │         │ name         │
│ email        │         │ address      │         │ phone        │
│ phone        │         │ phone        │         │ vehicle_type │
│ created_at   │         │ cuisine_type │         │ license_no   │
└──────────────┘         │ rating       │         │ rating       │
                         │ is_active    │         │ is_online    │
                         └──────────────┘         └──────────────┘
                                │                         │
                                │                         │
                         ┌──────────────┐                │
                         │  MENU_ITEM   │                │
                         ├──────────────┤                │
                         │ id (PK)      │                │
                         │ restaurant_id│────────────────┘
                         │ name         │
                         │ description  │
                         │ price        │
                         │ is_available │
                         └──────────────┘
                                │
                                │
                         ┌──────────────┐
                         │    ORDER     │
                         ├──────────────┤
                         │ id (PK)      │
                         │ customer_id  │────→ CUSTOMER
                         │ restaurant_id│────→ RESTAURANT
                         │ driver_id    │────→ DRIVER
                         │ status       │
                         │ total_amount │
                         │ created_at   │
                         └──────────────┘
                                │
                                │
                         ┌──────────────┐
                         │  ORDER_ITEM  │
                         ├──────────────┤
                         │ id (PK)      │
                         │ order_id     │────→ ORDER
                         │ menu_item_id │────→ MENU_ITEM
                         │ quantity     │
                         │ price        │
                         └──────────────┘
```

**Simple Schema (PostgreSQL):**

```sql
-- Customers table
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_customers_email ON customers(email);

-- Restaurants table
CREATE TABLE restaurants (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    address TEXT NOT NULL,
    latitude DECIMAL(10, 8),
    longitude DECIMAL(11, 8),
    phone VARCHAR(20),
    cuisine_type VARCHAR(50),
    rating DECIMAL(3, 2) DEFAULT 0.0,
    is_active BOOLEAN DEFAULT true,
    delivery_radius_km INTEGER DEFAULT 5,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_restaurants_location ON restaurants USING GIST (
    ll_to_earth(latitude, longitude)
); -- Geospatial index for "restaurants near me"

-- Menu items table
CREATE TABLE menu_items (
    id BIGSERIAL PRIMARY KEY,
    restaurant_id BIGINT REFERENCES restaurants(id),
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    category VARCHAR(50), -- appetizer, main, dessert
    is_available BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_menu_items_restaurant ON menu_items(restaurant_id);
CREATE INDEX idx_menu_items_available ON menu_items(restaurant_id, is_available);

-- Drivers table
CREATE TABLE drivers (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    email VARCHAR(255),
    vehicle_type VARCHAR(20), -- bike, scooter, car
    license_number VARCHAR(50),
    rating DECIMAL(3, 2) DEFAULT 5.0,
    is_online BOOLEAN DEFAULT false,
    current_latitude DECIMAL(10, 8),
    current_longitude DECIMAL(11, 8),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_drivers_online ON drivers(is_online);
CREATE INDEX idx_drivers_location ON drivers USING GIST (
    ll_to_earth(current_latitude, current_longitude)
) WHERE is_online = true; -- Only index online drivers

-- Orders table
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT REFERENCES customers(id),
    restaurant_id BIGINT REFERENCES restaurants(id),
    driver_id BIGINT REFERENCES drivers(id),
    status VARCHAR(20) NOT NULL, -- PLACED, CONFIRMED, PREPARING, etc.
    subtotal DECIMAL(10, 2) NOT NULL,
    tax DECIMAL(10, 2) NOT NULL,
    delivery_fee DECIMAL(10, 2) NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    delivery_address TEXT NOT NULL,
    delivery_latitude DECIMAL(10, 8),
    delivery_longitude DECIMAL(11, 8),
    estimated_delivery_time TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_customer ON orders(customer_id, created_at DESC);
CREATE INDEX idx_orders_restaurant ON orders(restaurant_id, status);
CREATE INDEX idx_orders_driver ON orders(driver_id, status);
CREATE INDEX idx_orders_status ON orders(status, created_at);

-- Order items table
CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT REFERENCES orders(id),
    menu_item_id BIGINT REFERENCES menu_items(id),
    menu_item_name VARCHAR(200) NOT NULL, -- Snapshot at order time
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    total_price DECIMAL(10, 2) NOT NULL
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
```

---

### 🟡 Intermediate Level: Multi-Database Strategy

**Why Multiple Databases?**

```text
POSTGRESQL (Transactional Data):
✅ ACID properties (orders, payments need strong consistency)
✅ Complex joins (order + items + customer + restaurant)
✅ Foreign keys (data integrity)
❌ Slow for high-volume reads (200K concurrent orders checking status)
❌ Not optimized for geospatial queries at scale

USE FOR: Orders, payments, customers, restaurants (write path)

CASSANDRA (Time-Series Data):
✅ High write throughput (2.6M location updates/second)
✅ Time-series data (GPS coordinates with timestamp)
✅ Horizontal scalability (add nodes to scale)
❌ No joins (data must be denormalized)
❌ Eventual consistency (acceptable for location data)

USE FOR: Driver locations, order status history, analytics events

REDIS (Caching):
✅ Sub-millisecond reads (99% cache hit rate)
✅ Geospatial commands (GEORADIUS for "drivers near me")
✅ Pub/Sub (real-time updates)
❌ Volatile (data lost if crash, unless persistence enabled)
❌ Memory-constrained (expensive to store everything)

USE FOR: Active orders, driver availability, restaurant menus (read path)

ELASTICSEARCH (Search):
✅ Full-text search ("find restaurants with 'sushi' in name or description")
✅ Complex filters (cuisine + rating + delivery time)
✅ Aggregations (top cuisines, price ranges)
❌ Not a primary datastore (data loss risk)
❌ Eventual consistency (data synced from PostgreSQL)

USE FOR: Restaurant search, menu search, analytics queries

POSTGRESQL (Graph - Optional):
- Use for friend recommendations, referral networks
- Neo4j alternative if needed
```

**Cassandra Schema for Location Tracking:**

```cql
-- Driver locations (time-series data)
CREATE TABLE driver_locations (
    driver_id BIGINT,
    timestamp TIMESTAMP,
    latitude DECIMAL,
    longitude DECIMAL,
    accuracy DECIMAL,
    speed DECIMAL,
    PRIMARY KEY (driver_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Query: Get driver's location history for last hour
SELECT * FROM driver_locations 
WHERE driver_id = 12345 
  AND timestamp > NOW() - INTERVAL '1 hour';

-- Order status history (audit trail)
CREATE TABLE order_status_history (
    order_id BIGINT,
    timestamp TIMESTAMP,
    status VARCHAR,
    actor VARCHAR, -- customer, restaurant, driver, system
    metadata TEXT, -- JSON with additional info
    PRIMARY KEY (order_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);
```

**Redis Schema for Real-Time Data:**

```text
# Active orders (hash)
Key: order:{order_id}
Value: {
    "status": "PICKED_UP",
    "driver_id": 12345,
    "restaurant_id": 67890,
    "customer_id": 54321,
    "estimated_delivery": "2025-11-04T19:30:00Z"
}
TTL: 2 hours (auto-expire after delivery)

# Driver availability (sorted set)
Key: drivers:online:{city_id}
Score: timestamp of last update
Member: driver_id
ZREMRANGEBYSCORE drivers:online:nyc 0 (NOW() - 60) # Remove drivers offline >60 sec

# Driver locations (geospatial)
Key: driver_locations
GEOADD driver_locations -73.9857 40.7484 driver:12345
GEORADIUS driver_locations -73.9857 40.7484 5 km WITHDIST # Drivers within 5km

# Restaurant menu cache (string)
Key: menu:restaurant:{restaurant_id}
Value: JSON of entire menu
TTL: 10 minutes
```

---

### 🔴 Advanced Level: Sharding, Indexing, and Consistency

**Sharding Strategy:**

```text
PROBLEM: Single PostgreSQL instance can't handle 10M orders/day
- Write throughput: ~1,200 writes/second (approaching single-server limit)
- Storage: 32 TB/year (larger than single server)
- Read throughput: 26K reads/second (even with caching)

SOLUTION: Horizontal Sharding

Option 1: Shard by City
- 100 cities → 100 shards
- Orders for New York → shard 1, San Francisco → shard 2
- Benefits: Geo-locality (low latency), natural data boundary
- Drawbacks: Uneven distribution (NYC has 100x more orders than Boise)

Option 2: Shard by Order ID
- Hash(order_id) % 256 shards
- Benefits: Even distribution
- Drawbacks: Cross-shard queries ("get all orders for restaurant X")

Option 3: Hybrid (Used by Uber Eats)
- Shard by city_id FIRST (100 cities → 100 logical shards)
- Within each city, shard by order_id (e.g., NYC → 10 physical shards)
- Benefits: Geo-locality + even distribution
- Implementation:
  └─ Shard key: (city_id, order_id % num_shards_per_city)
  └─ NYC (high volume): 10 shards
  └─ Boise (low volume): 1 shard
```

**Shard Routing Logic:**

```python
def get_shard(city_id, order_id):
    """Route order to correct shard based on city and order ID"""
    
    # City-to-shard mapping (in config)
    city_shard_config = {
        "nyc": {"start_shard": 0, "num_shards": 10},
        "sf": {"start_shard": 10, "num_shards": 5},
        "boise": {"start_shard": 15, "num_shards": 1},
        # ... 100 cities total
    }
    
    config = city_shard_config[city_id]
    shard_offset = order_id % config["num_shards"]
    shard_id = config["start_shard"] + shard_offset
    
    return f"orders_shard_{shard_id}"

# Example:
# NYC order 123456 → shard 6 (start:0 + (123456 % 10) = 0 + 6)
# Boise order 789012 → shard 15 (start:15 + (789012 % 1) = 15 + 0)
```

**Advanced Indexing:**

```sql
-- Composite index for common query: "Get recent orders for restaurant"
CREATE INDEX idx_orders_restaurant_status_time 
ON orders(restaurant_id, status, created_at DESC);

-- Partial index: Only index active orders (not DELIVERED/CANCELLED)
CREATE INDEX idx_active_orders 
ON orders(driver_id, created_at DESC) 
WHERE status NOT IN ('DELIVERED', 'CANCELLED');

-- Covering index: Include columns to avoid table lookup
CREATE INDEX idx_orders_customer_covering 
ON orders(customer_id, created_at DESC) 
INCLUDE (restaurant_id, status, total_amount);
-- Query: SELECT restaurant_id, status, total_amount WHERE customer_id = X
--        → Served entirely from index (no table scan)

-- GIN index for JSON search (if storing menu in JSONB)
CREATE INDEX idx_restaurant_metadata ON restaurants USING GIN (metadata);
-- Query: WHERE metadata @> '{"dietary": ["vegan"]}'

-- Full-text search index
CREATE INDEX idx_restaurant_search 
ON restaurants USING GIN (to_tsvector('english', name || ' ' || description));
-- Query: WHERE to_tsvector('english', name || ' ' || description) 
--             @@ to_tsquery('english', 'sushi & delivery');
```

**Consistency Models:**

```text
STRONG CONSISTENCY (Use PostgreSQL):
1. Order creation: Must not create duplicate orders
   └─ Use unique constraint on (customer_id, created_at, restaurant_id)
   
2. Payment processing: Must not double-charge customer
   └─ Use database transactions (BEGIN ... COMMIT)
   └─ Idempotency key (if customer clicks "Pay" twice, only charge once)
   
3. Inventory management: Must not oversell sold-out items
   └─ Use row-level locks: SELECT ... FOR UPDATE
   └─ Pessimistic locking during checkout

EVENTUAL CONSISTENCY (Use Cassandra/Redis):
1. Driver locations: 1-2 second delay is acceptable
   └─ Write to Cassandra (async), read from Redis (cached)
   
2. Restaurant ratings: Can update after 5-10 seconds
   └─ Calculate asynchronously, cache in Redis
   
3. Order status for customers: 1 second delay is ok
   └─ Write to PostgreSQL, replicate to Cassandra, customer reads from Cassandra

CAUSAL CONSISTENCY (Use Vector Clocks):
1. Order status updates must be ordered correctly
   └─ Can't show "DELIVERED" before "PICKED_UP"
   └─ Use Lamport timestamps or version vectors
```

**CAP Theorem Trade-offs:**

```text
During network partition, what do we choose?

SCENARIO 1: Order Placement (CP - Consistency + Partition Tolerance)
- Network partition between order service and payment service
- CHOICE: Sacrifice Availability (reject orders) to maintain Consistency
- WHY: Can't create order without charging customer (money loss risk)
- IMPLEMENTATION: Return error "Service temporarily unavailable, try again"

SCENARIO 2: Driver Location Updates (AP - Availability + Partition Tolerance)
- Network partition between location service and customer app
- CHOICE: Sacrifice Consistency (show stale location) to maintain Availability
- WHY: Showing 5-second-old location is better than showing nothing
- IMPLEMENTATION: Continue serving cached locations until partition heals

SCENARIO 3: Menu Browsing (AP - Availability + Partition Tolerance)
- Network partition between menu service and Elasticsearch
- CHOICE: Sacrifice Consistency (show outdated menu) to maintain Availability
- WHY: Customer can browse menus even if some items are sold out
- IMPLEMENTATION: Serve cached menus with banner "Menu may be outdated"
```

---

### Key Takeaways

✅ **Use PostgreSQL for transactional data** (orders, payments) requiring ACID properties

✅ **Use Cassandra for time-series data** (driver locations) with high write throughput

✅ **Use Redis for caching and geospatial queries** (99% cache hit rate reduces DB load)

✅ **Use Elasticsearch for full-text search** (restaurant/menu search)

✅ **Shard by (city_id, order_id)** for geo-locality and even distribution

✅ **Create composite and partial indexes** to optimize common queries

✅ **Choose consistency model based on use case:** Strong for payments, eventual for locations

---

**Think About It:**
- How do we migrate from 1 database to 100 shards without downtime?
- What happens if a shard goes down during peak hours?
- Should we store order history forever or archive old data to cheaper storage?
- How do we maintain referential integrity across sharded databases?

---

## 5. API Design

### What You'll Learn
- RESTful API design for a three-sided marketplace
- API endpoints for customers, restaurants, and drivers
- Request/response formats with real examples
- Authentication and authorization patterns
- Rate limiting and API versioning strategies

### Why This Matters
Uber Eats has 3 different client apps (customer, restaurant, driver) all consuming the same backend APIs. Well-designed APIs enable rapid mobile app development, third-party integrations, and A/B testing. Poor API design leads to chatty interactions (100 API calls to load one screen), security vulnerabilities, and breaking changes that crash apps.

---

### 🟢 Beginner Level: Core RESTful Endpoints

**API Structure:**

```text
Base URL: https://api.ubereats.com/v1

CUSTOMER APIS:
/customers          (manage profile)
/restaurants        (browse restaurants)
/menus              (view menus)
/orders             (place and track orders)
/payments           (manage payment methods)

RESTAURANT APIS:
/restaurant/orders  (incoming orders)
/restaurant/menu    (manage menu)
/restaurant/profile (business settings)

DRIVER APIS:
/driver/profile     (manage profile)
/driver/orders      (view assigned orders)
/driver/location    (update GPS location)
/driver/earnings    (view earnings)
```

**Customer Endpoints:**

```http
# 1. Browse nearby restaurants
GET /v1/restaurants?lat=40.7484&lon=-73.9857&radius=5

Response 200 OK:
{
  "restaurants": [
    {
      "id": 12345,
      "name": "Joe's Pizza",
      "cuisine": "Italian",
      "rating": 4.5,
      "delivery_time_min": 25,
      "delivery_fee": 2.99,
      "min_order": 15.00,
      "distance_km": 1.2,
      "image_url": "https://cdn.ubereats.com/restaurants/12345.jpg"
    },
    {...}
  ],
  "total": 47,
  "page": 1,
  "page_size": 20
}

# 2. Get restaurant menu
GET /v1/restaurants/12345/menu

Response 200 OK:
{
  "restaurant": {
    "id": 12345,
    "name": "Joe's Pizza"
  },
  "categories": [
    {
      "name": "Pizza",
      "items": [
        {
          "id": 67890,
          "name": "Margherita Pizza",
          "description": "Fresh mozzarella, basil, tomato sauce",
          "price": 12.99,
          "image_url": "https://cdn.ubereats.com/items/67890.jpg",
          "available": true,
          "options": [
            {
              "name": "Size",
              "required": true,
              "choices": ["Small (+$0)", "Medium (+$3)", "Large (+$5)"]
            }
          ]
        }
      ]
    }
  ]
}

# 3. Create order
POST /v1/orders
Authorization: Bearer <jwt_token>
Content-Type: application/json

Request Body:
{
  "restaurant_id": 12345,
  "delivery_address": {
    "street": "123 Main St",
    "city": "New York",
    "zip": "10001",
    "latitude": 40.7484,
    "longitude": -73.9857
  },
  "items": [
    {
      "menu_item_id": 67890,
      "quantity": 2,
      "options": {
        "Size": "Large"
      },
      "special_instructions": "Extra basil"
    }
  ],
  "payment_method_id": "pm_abc123",
  "tip_amount": 5.00
}

Response 201 Created:
{
  "order_id": 98765,
  "status": "PLACED",
  "subtotal": 35.98,
  "tax": 3.15,
  "delivery_fee": 4.99,
  "tip": 5.00,
  "total": 49.12,
  "estimated_delivery_time": "2025-11-04T19:30:00Z",
  "created_at": "2025-11-04T18:45:00Z"
}

# 4. Get order status
GET /v1/orders/98765
Authorization: Bearer <jwt_token>

Response 200 OK:
{
  "order_id": 98765,
  "status": "PICKED_UP",
  "restaurant": {
    "id": 12345,
    "name": "Joe's Pizza",
    "phone": "+1-212-555-0100"
  },
  "driver": {
    "id": 54321,
    "name": "John D.",
    "phone": "+1-212-555-0200",
    "rating": 4.8,
    "vehicle_type": "scooter",
    "current_location": {
      "latitude": 40.7500,
      "longitude": -73.9900
    }
  },
  "timeline": [
    {"status": "PLACED", "timestamp": "2025-11-04T18:45:00Z"},
    {"status": "CONFIRMED", "timestamp": "2025-11-04T18:46:00Z"},
    {"status": "PREPARING", "timestamp": "2025-11-04T18:47:00Z"},
    {"status": "READY", "timestamp": "2025-11-04T19:10:00Z"},
    {"status": "PICKED_UP", "timestamp": "2025-11-04T19:12:00Z"}
  ],
  "estimated_delivery_time": "2025-11-04T19:30:00Z",
  "total": 49.12
}

# 5. Cancel order
POST /v1/orders/98765/cancel
Authorization: Bearer <jwt_token>

Request Body:
{
  "reason": "Changed my mind"
}

Response 200 OK:
{
  "order_id": 98765,
  "status": "CANCELLED",
  "refund_amount": 49.12,
  "refund_status": "PENDING"
}
```

---

### 🟡 Intermediate Level: Restaurant & Driver APIs

**Restaurant Endpoints:**

```http
# 1. Get incoming orders
GET /v1/restaurant/orders?status=PENDING
Authorization: Bearer <restaurant_jwt>

Response 200 OK:
{
  "orders": [
    {
      "order_id": 98765,
      "customer": {
        "name": "Alice Smith",
        "phone": "+1-212-555-0300"
      },
      "items": [
        {
          "name": "Margherita Pizza (Large)",
          "quantity": 2,
          "special_instructions": "Extra basil"
        }
      ],
      "total": 35.98,
      "created_at": "2025-11-04T18:45:00Z",
      "estimated_prep_time_min": 20
    }
  ]
}

# 2. Accept order
POST /v1/restaurant/orders/98765/accept
Authorization: Bearer <restaurant_jwt>

Request Body:
{
  "estimated_prep_time_min": 25
}

Response 200 OK:
{
  "order_id": 98765,
  "status": "CONFIRMED",
  "estimated_ready_time": "2025-11-04T19:10:00Z"
}

# 3. Mark order ready for pickup
POST /v1/restaurant/orders/98765/ready
Authorization: Bearer <restaurant_jwt>

Response 200 OK:
{
  "order_id": 98765,
  "status": "READY",
  "driver_eta_min": 5
}

# 4. Update menu item availability
PATCH /v1/restaurant/menu/items/67890
Authorization: Bearer <restaurant_jwt>

Request Body:
{
  "available": false
}

Response 200 OK:
{
  "item_id": 67890,
  "name": "Margherita Pizza",
  "available": false,
  "updated_at": "2025-11-04T19:15:00Z"
}
```

**Driver Endpoints:**

```http
# 1. Update online status
POST /v1/driver/status
Authorization: Bearer <driver_jwt>

Request Body:
{
  "online": true,
  "location": {
    "latitude": 40.7484,
    "longitude": -73.9857
  }
}

Response 200 OK:
{
  "driver_id": 54321,
  "online": true,
  "orders_available": 3
}

# 2. Get available orders (for driver to accept)
GET /v1/driver/orders/available
Authorization: Bearer <driver_jwt>

Response 200 OK:
{
  "orders": [
    {
      "order_id": 98765,
      "restaurant": {
        "name": "Joe's Pizza",
        "address": "456 Main St",
        "distance_km": 0.8
      },
      "delivery_address": "123 Main St",
      "delivery_distance_km": 1.5,
      "estimated_payout": 8.50,
      "pickup_by": "2025-11-04T19:15:00Z"
    }
  ]
}

# 3. Accept order
POST /v1/driver/orders/98765/accept
Authorization: Bearer <driver_jwt>

Response 200 OK:
{
  "order_id": 98765,
  "status": "DRIVER_ASSIGNED",
  "restaurant_location": {
    "latitude": 40.7500,
    "longitude": -73.9900
  },
  "route_to_restaurant": {
    "distance_km": 0.8,
    "duration_min": 5,
    "polyline": "encoded_polyline_here"
  }
}

# 4. Update location (called every 1 second while active)
POST /v1/driver/location
Authorization: Bearer <driver_jwt>

Request Body:
{
  "latitude": 40.7484,
  "longitude": -73.9857,
  "accuracy": 10,
  "timestamp": "2025-11-04T19:20:00Z"
}

Response 200 OK:
{
  "received": true
}

# 5. Confirm pickup
POST /v1/driver/orders/98765/pickup
Authorization: Bearer <driver_jwt>

Response 200 OK:
{
  "order_id": 98765,
  "status": "PICKED_UP",
  "customer_location": {
    "latitude": 40.7484,
    "longitude": -73.9857
  },
  "route_to_customer": {
    "distance_km": 1.5,
    "duration_min": 8,
    "polyline": "encoded_polyline_here"
  }
}

# 6. Confirm delivery
POST /v1/driver/orders/98765/deliver
Authorization: Bearer <driver_jwt>

Request Body:
{
  "photo_url": "https://cdn.ubereats.com/proofs/98765.jpg",
  "notes": "Left at door as requested"
}

Response 200 OK:
{
  "order_id": 98765,
  "status": "DELIVERED",
  "earnings": 8.50,
  "next_order_available": true
}
```

---

### 🔴 Advanced Level: Authentication, Versioning & WebSockets

**Authentication Strategy (JWT):**

```http
# Login (get JWT token)
POST /v1/auth/login
Content-Type: application/json

Request Body:
{
  "email": "alice@example.com",
  "password": "secret123",
  "client_type": "customer"  // or "restaurant", "driver"
}

Response 200 OK:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl...",
  "expires_in": 3600,
  "user": {
    "id": 11111,
    "name": "Alice Smith",
    "email": "alice@example.com"
  }
}

# Use access token in subsequent requests
GET /v1/orders
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# Refresh token when expired
POST /v1/auth/refresh
Content-Type: application/json

Request Body:
{
  "refresh_token": "dGhpcyBpcyBhIHJlZnJl..."
}

Response 200 OK:
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 3600
}
```

**JWT Token Payload:**

```json
{
  "sub": "11111",              // User ID
  "client_type": "customer",   // customer, restaurant, driver
  "email": "alice@example.com",
  "iat": 1699123200,           // Issued at (Unix timestamp)
  "exp": 1699126800,           // Expires at (Unix timestamp, 1 hour later)
  "permissions": [
    "orders:create",
    "orders:read",
    "orders:cancel"
  ]
}
```

**API Versioning:**

```http
# Version 1 (original)
GET /v1/restaurants?lat=40.7484&lon=-73.9857

# Version 2 (added filters)
GET /v2/restaurants?lat=40.7484&lon=-73.9857&cuisine=italian&max_delivery_time=30

# Version 3 (changed response format, breaking change)
GET /v3/restaurants?lat=40.7484&lon=-73.9857
# Response includes "promoted" flag, "discount" percentage, etc.

VERSIONING STRATEGIES:
1. URL versioning: /v1/orders, /v2/orders (used by Uber Eats)
2. Header versioning: Accept: application/vnd.ubereats.v2+json
3. Query param versioning: /orders?version=2

DEPRECATION POLICY:
- Support N-2 versions (currently v3, support v2 and v1)
- Announce deprecation 6 months in advance
- Return deprecation headers:
  Deprecation: Sun, 11 Nov 2025 23:59:59 GMT
  Sunset: Sun, 11 May 2026 23:59:59 GMT
```

**WebSocket API for Real-Time Updates:**

```javascript
// Connect to WebSocket
const ws = new WebSocket('wss://ws.ubereats.com/v1/orders/98765');

// Authenticate
ws.send(JSON.stringify({
  type: 'auth',
  token: 'eyJhbGciOiJIUzI1NiIs...'
}));

// Subscribe to order updates
ws.send(JSON.stringify({
  type: 'subscribe',
  channels: ['order_status', 'driver_location']
}));

// Receive updates
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'order_status') {
    console.log('Order status changed:', data.payload);
    // {
    //   "order_id": 98765,
    //   "old_status": "PREPARING",
    //   "new_status": "READY",
    //   "timestamp": "2025-11-04T19:10:00Z"
    // }
  }
  
  if (data.type === 'driver_location') {
    console.log('Driver location updated:', data.payload);
    // {
    //   "driver_id": 54321,
    //   "latitude": 40.7500,
    //   "longitude": -73.9900,
    //   "eta_minutes": 5,
    //   "timestamp": "2025-11-04T19:20:00Z"
    // }
  }
};

// Heartbeat (every 30 seconds)
setInterval(() => {
  ws.send(JSON.stringify({ type: 'ping' }));
}, 30000);

ws.onmessage = (event) => {
  if (event.data === '{"type":"pong"}') {
    console.log('Connection alive');
  }
};
```

**Rate Limiting:**

```http
# Rate limit headers (returned with every response)
X-RateLimit-Limit: 100             # Requests allowed per window
X-RateLimit-Remaining: 87          # Requests remaining in current window
X-RateLimit-Reset: 1699126800      # Unix timestamp when limit resets

# When rate limit exceeded
Response 429 Too Many Requests:
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again in 42 seconds.",
    "retry_after": 42
  }
}

RATE LIMITS (per user):
- Customer App: 100 requests/minute
- Restaurant Dashboard: 500 requests/minute
- Driver App: 1000 requests/minute (due to location updates)
- Search API: 20 requests/minute (expensive queries)
```

**Idempotency (Preventing Duplicate Orders):**

```http
# Use idempotency key to prevent duplicate orders if customer clicks "Place Order" twice
POST /v1/orders
Authorization: Bearer <jwt_token>
Idempotency-Key: a1b2c3d4-e5f6-7890-abcd-ef1234567890

Request Body:
{
  "restaurant_id": 12345,
  "items": [...]
}

FIRST REQUEST:
Response 201 Created:
{
  "order_id": 98765,
  "status": "PLACED",
  ...
}

SECOND REQUEST (same idempotency key within 24 hours):
Response 200 OK (returns same response as first request):
{
  "order_id": 98765,
  "status": "PLACED",
  ...
}
```

**Error Handling:**

```http
# Standard error response format
Response 400 Bad Request:
{
  "error": {
    "code": "INVALID_ADDRESS",
    "message": "Delivery address is outside restaurant's delivery radius",
    "field": "delivery_address",
    "details": {
      "restaurant_delivery_radius_km": 5,
      "actual_distance_km": 7.2
    }
  }
}

ERROR CODES:
- 400 Bad Request: Client sent invalid data
  - INVALID_ADDRESS, ITEM_NOT_AVAILABLE, PAYMENT_DECLINED
  
- 401 Unauthorized: Missing or invalid JWT token
  - INVALID_TOKEN, TOKEN_EXPIRED
  
- 403 Forbidden: User not authorized for action
  - INSUFFICIENT_PERMISSIONS, RESTAURANT_INACTIVE
  
- 404 Not Found: Resource doesn't exist
  - ORDER_NOT_FOUND, RESTAURANT_NOT_FOUND
  
- 429 Too Many Requests: Rate limit exceeded
  - RATE_LIMIT_EXCEEDED
  
- 500 Internal Server Error: Server-side error
  - DATABASE_ERROR, SERVICE_UNAVAILABLE
  
- 503 Service Unavailable: Temporary downtime
  - MAINTENANCE_MODE, CIRCUIT_BREAKER_OPEN
```

---

### Key Takeaways

✅ **RESTful design:** Use proper HTTP methods (GET for reads, POST for creates, PATCH for updates)

✅ **JWT authentication:** Stateless tokens with short expiry (1 hour) and refresh tokens

✅ **API versioning:** URL-based (/v1, /v2) with 6-month deprecation policy

✅ **WebSockets for real-time:** Order status and driver location updates

✅ **Idempotency keys:** Prevent duplicate orders from double-clicks

✅ **Rate limiting:** Protect APIs from abuse (100-1000 requests/minute depending on client)

✅ **Standard error formats:** Consistent error codes and detailed error messages

---

**Think About It:**
- Should we use GraphQL instead of REST to reduce API calls?
- How do we version WebSocket protocols?
- What happens if customer's phone goes offline during order placement?
- Should rate limits be per IP, per user, or per API key?

---

