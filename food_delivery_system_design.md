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

## 6. Order Management & State Machine

### What You'll Learn
- Order lifecycle from placement to delivery
- State machine design for complex workflows
- State transitions and valid operations
- Handling cancellations and modifications
- Error recovery and retry mechanisms

### Why This Matters
An order goes through 12+ states involving 3 different parties (customer, restaurant, driver). Missing a state transition could mean a customer pays but never receives food, or a restaurant prepares food that's never picked up. Uber Eats processes 10M orders/day—even 0.1% failure rate means 10,000 unhappy customers daily.

---

### 🟢 Beginner Level: Order Lifecycle

**The Journey of an Order:**

Think of an order like a relay race with multiple runners (customer, restaurant, driver) passing a baton (the order):

```text
CUSTOMER hands off order → RESTAURANT prepares → DRIVER delivers → CUSTOMER receives

1. CUSTOMER PHASE (Runner 1):
   PLACED      Customer clicks "Place Order"
   ↓
   PAYMENT_PROCESSING  Charging credit card
   ↓
   CONFIRMED   Payment successful

2. RESTAURANT PHASE (Runner 2):
   ACCEPTED    Restaurant accepts order
   ↓
   PREPARING   Kitchen is cooking
   ↓
   READY       Food is ready for pickup

3. DRIVER PHASE (Runner 3):
   ASSIGNED    Driver assigned to order
   ↓
   EN_ROUTE_TO_RESTAURANT  Driver heading to restaurant
   ↓
   ARRIVED_AT_RESTAURANT   Driver at pickup location
   ↓
   PICKED_UP   Driver has the food
   ↓
   EN_ROUTE_TO_CUSTOMER    Driver delivering
   ↓
   ARRIVED     Driver at customer location
   ↓
   DELIVERED   Food handed to customer

4. COMPLETION PHASE:
   COMPLETED   Customer confirms receipt
```

**Happy Path Timeline (35 minutes total):**

```text
Time    State                        What's Happening
00:00   PLACED                       Customer clicks "Place Order"
00:05   CONFIRMED                    Payment processed, restaurant notified
00:30   ACCEPTED                     Restaurant accepts (25 sec avg acceptance time)
00:45   ASSIGNED                     Driver assigned (15 sec matching time)
01:00   PREPARING                    Kitchen starts cooking
15:00   READY                        Food is ready (14 min prep time)
15:30   PICKED_UP                    Driver picks up (30 sec at restaurant)
16:00   EN_ROUTE_TO_CUSTOMER         Driver starts delivery
24:00   ARRIVED                      Driver at customer location (8 min drive)
24:30   DELIVERED                    Food handed to customer
25:00   COMPLETED                    Customer confirms
```

**Error Scenarios:**

```text
SCENARIO 1: Restaurant Rejects Order
PLACED → PAYMENT_PROCESSING → CONFIRMED → REJECTED
└─ Trigger refund, notify customer, suggest alternative restaurants

SCENARIO 2: Driver Cancels After Assignment
PLACED → CONFIRMED → ACCEPTED → ASSIGNED → CANCELLED_BY_DRIVER
└─ Reassign to another driver, extend ETA, notify customer

SCENARIO 3: Customer Cancels During Preparation
PLACED → CONFIRMED → ACCEPTED → PREPARING → CANCELLED_BY_CUSTOMER
└─ Stop preparation, partial refund (if food already prepared), compensate restaurant

SCENARIO 4: Food Never Picked Up (Driver No-Show)
READY → (30 min timeout) → ABANDONED
└─ Refund customer, compensate restaurant, deactivate driver
```

---

### 🟡 Intermediate Level: State Machine Implementation

**Understanding State Transitions:**

A state machine is a mathematical model that defines all possible states an order can be in and the valid transitions between those states. In food delivery, this is critical because we have three parties (customer, restaurant, driver) who can all trigger state changes, and we need to ensure orders never get into invalid states (like "DELIVERED" before "PICKED_UP").

**The 12 Core States:**

1. **PLACED** - Customer submitted order, not yet paid
2. **PAYMENT_PROCESSING** - Payment gateway is charging card
3. **CONFIRMED** - Payment successful, waiting for restaurant
4. **ACCEPTED** - Restaurant accepted order
5. **PREPARING** - Kitchen is cooking food
6. **READY** - Food is ready for pickup
7. **ASSIGNED** - Driver assigned to order
8. **PICKED_UP** - Driver has the food
9. **EN_ROUTE** - Driver is delivering
10. **DELIVERED** - Food handed to customer
11. **COMPLETED** - Transaction finished (terminal state)
12. **CANCELLED/REJECTED** - Order cancelled (terminal states)

**State Transition Matrix:**

This diagram shows which state transitions are valid. An order can only move from its current state to one of the allowed next states:

```text
FROM STATE          →  ALLOWED NEXT STATES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PLACED              →  PAYMENT_PROCESSING, CANCELLED
PAYMENT_PROCESSING  →  CONFIRMED, CANCELLED
CONFIRMED           →  ACCEPTED, REJECTED, CANCELLED
ACCEPTED            →  PREPARING, ASSIGNED, CANCELLED
PREPARING           →  READY, ASSIGNED, CANCELLED
READY               →  ASSIGNED, PICKED_UP, CANCELLED
ASSIGNED            →  PICKED_UP, ACCEPTED*, CANCELLED
PICKED_UP           →  EN_ROUTE, DELIVERED, CANCELLED
EN_ROUTE            →  DELIVERED, CANCELLED
DELIVERED           →  COMPLETED
COMPLETED           →  (terminal - no transitions)
CANCELLED           →  (terminal - no transitions)
REJECTED            →  (terminal - no transitions)

*ASSIGNED → ACCEPTED happens when driver cancels, need to reassign
```

**Why This Structure Matters:**

1. **Prevents Invalid Operations:** Can't mark order as delivered before it's picked up
2. **Enables Rollback:** If driver cancels, we can go back to ACCEPTED state
3. **Tracks Progress:** Each state has specific meaning and side effects
4. **Supports Auditing:** Every transition is logged with actor (who made change)
5. **Handles Failures:** Timeouts can trigger automatic state changes

**State Transition Logic (Pseudocode):**

```text
FUNCTION transition_order_state(order_id, new_state, actor, reason):
    // Step 1: Load current order from database
    current_order = database.get_order(order_id)
    current_state = current_order.status
    
    // Step 2: Validate this transition is allowed
    IF new_state NOT IN allowed_transitions[current_state]:
        THROW InvalidTransitionError
        RETURN error response to caller
    
    // Step 3: Execute transition within database transaction
    START database transaction:
        // Update order status
        database.update_order(
            order_id = order_id,
            status = new_state,
            updated_at = current_timestamp
        )
        
        // Create audit trail record
        database.insert_order_history(
            order_id = order_id,
            from_state = current_state,
            to_state = new_state,
            actor = actor,  // "customer", "restaurant", "driver", "system"
            reason = reason,
            timestamp = current_timestamp
        )
        
        // Trigger side effects (notifications, etc.)
        execute_side_effects(order_id, current_state, new_state)
    END transaction
    
    // Step 4: Publish event to message queue for async processing
    kafka.publish(topic="order_state_changed", message={
        order_id: order_id,
        old_state: current_state,
        new_state: new_state,
        actor: actor,
        timestamp: current_timestamp
    })
    
    RETURN success response
```

**Side Effects for Each State Transition:**

When an order changes state, specific actions must happen automatically:

```text
CONFIRMED → (Restaurant notified, driver matching initiated)
├─ Send push notification to restaurant tablet
├─ Start background job to find nearby drivers
└─ Update customer: "Restaurant is reviewing your order"

ASSIGNED → (Driver and customer notified)
├─ Send push notification to driver app with order details
├─ Update customer: "Driver John is heading to restaurant"
└─ Start tracking driver acceptance timeout (30 seconds)

READY → (Driver notified to pick up)
├─ Send urgent notification to driver: "Food is ready!"
├─ Update customer: "Your food is ready for pickup"
└─ Start tracking pickup timeout (30 minutes)

PICKED_UP → (Location tracking begins)
├─ Enable real-time GPS tracking (1-second intervals)
├─ Update customer: "Driver picked up your order"
├─ Send WebSocket updates with driver location
└─ Calculate initial ETA based on distance

DELIVERED → (Payment settlement triggered)
├─ Stop GPS tracking
├─ Process payment settlement (charge customer, pay restaurant/driver)
├─ Request ratings from customer and driver
└─ Update customer: "Enjoy your meal!"

CANCELLED → (Refunds and notifications)
├─ Process refund to customer (full or partial based on state)
├─ Release assigned driver (if any)
├─ Compensate restaurant if food was prepared
└─ Notify all parties of cancellation
```

**Interview Talking Points:**

When discussing this in an interview, emphasize:

1. **Why we need state machine:** "Without explicit state management, orders could end up in invalid states like DELIVERED before PICKED_UP. The state machine acts as a contract that enforces valid transitions."

2. **How we handle concurrency:** "We use database transactions to ensure state changes are atomic. If two services try to change state simultaneously, one will succeed and one will fail with a conflict error."

3. **Audit trail importance:** "Every state change is logged with timestamp and actor. This is critical for customer support (why was my order cancelled?) and analytics (what % of orders get cancelled during PREPARING?)"

4. **Terminal states:** "COMPLETED, CANCELLED, and REJECTED are terminal states - once reached, the order cannot transition further. This prevents accidentally 'uncancelling' an order."

**Timeout Handling:**

When orders get "stuck" in certain states (restaurant doesn't accept, driver doesn't pick up), we need automatic recovery mechanisms. This prevents orders from being forgotten and ensures good customer experience.

**Timeout Configuration by State:**

```text
STATE                TIME LIMIT    ACTION IF EXCEEDED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONFIRMED            5 minutes     Auto-reject order, refund customer
READY                30 minutes    Reassign to different driver
PICKED_UP            1 hour        Alert customer support, contact driver
```

**Why These Timeouts?**

1. **CONFIRMED (5 min):** Restaurant should respond quickly. If they're too busy or tablet is off, customer shouldn't wait indefinitely. After 5 minutes, we auto-reject and suggest alternative restaurants.

2. **READY (30 min):** Food quality degrades. If driver doesn't show up within 30 minutes, food gets cold. We reassign to another driver or cancel if no drivers available.

3. **PICKED_UP (1 hour):** Normal deliveries take 15-30 minutes. If it's taking >1 hour, something is wrong (driver lost, accident, wrong address). Alert human support team to investigate.

**Timeout Monitor Logic (Pseudocode):**

```text
// Background job runs every 1 minute
SCHEDULED_JOB check_order_timeouts() EVERY 1 minute:
    
    current_time = now()
    
    // Check each timeout-sensitive state
    FOR EACH (state, timeout_duration) IN timeout_config:
        
        cutoff_time = current_time - timeout_duration
        
        // Find orders stuck in this state beyond timeout
        stuck_orders = database.query(
            SELECT * FROM orders 
            WHERE status = state 
            AND updated_at < cutoff_time
        )
        
        // Handle each stuck order
        FOR EACH order IN stuck_orders:
            handle_timeout(order, state)

FUNCTION handle_timeout(order, state):
    
    CASE state:
        
        WHEN CONFIRMED:
            // Restaurant didn't accept within 5 minutes
            log_warning("Order " + order.id + " timeout: restaurant unresponsive")
            
            // Auto-reject the order
            transition_order_state(
                order_id = order.id,
                new_state = REJECTED,
                actor = "system",
                reason = "Restaurant timeout - no response within 5 minutes"
            )
            
            // Notify customer with alternative suggestions
            notification_service.send(
                user_id = order.customer_id,
                message = "Sorry, restaurant couldn't accept your order. Here are similar options nearby.",
                alternative_restaurants = find_similar_restaurants(order.restaurant_id)
            )
            
            // Update metrics
            metrics.increment("order.timeout.restaurant_unresponsive")
        
        WHEN READY:
            // Driver didn't pick up within 30 minutes
            log_error("Order " + order.id + " timeout: driver no-show")
            
            // Increment reassignment counter
            order.reassignment_count += 1
            
            IF order.reassignment_count < 3:
                // Try reassigning to different driver
                matching_service.reassign_driver(order.id)
                
                // Extend ETA for customer
                notification_service.send(
                    user_id = order.customer_id,
                    message = "We're finding a new driver for your order. Sorry for the delay!"
                )
            ELSE:
                // Already tried 3 times, give up
                transition_order_state(
                    order_id = order.id,
                    new_state = CANCELLED,
                    actor = "system",
                    reason = "Multiple driver no-shows - unable to find delivery driver"
                )
                
                // Full refund to customer
                payment_service.refund(order.id, amount = order.total)
                
                // Compensate restaurant for prepared food
                payment_service.compensate_restaurant(order.restaurant_id, amount = order.subtotal * 0.5)
            
            // Update metrics
            metrics.increment("order.timeout.driver_no_show")
        
        WHEN PICKED_UP:
            // Delivery taking too long (>1 hour)
            log_error("Order " + order.id + " timeout: delivery delayed")
            
            // Create high-priority support ticket
            support_service.create_ticket(
                order_id = order.id,
                priority = "HIGH",
                issue = "Delivery taking longer than expected (>1 hour)",
                assigned_to = "on_call_support_team"
            )
            
            // Send urgent notification to driver
            notification_service.send_urgent(
                user_id = order.driver_id,
                message = "Are you okay? This delivery is taking longer than expected. Please contact support if you need help.",
                requires_acknowledgment = true
            )
            
            // Keep customer informed
            notification_service.send(
                user_id = order.customer_id,
                message = "Your order is delayed. We've contacted the driver and are working to resolve this. You may be eligible for a refund or credit."
            )
            
            // Update metrics
            metrics.increment("order.timeout.delivery_delayed")
```

**Real-World Example (Uber Eats):**

Uber Eats uses similar timeout mechanisms:

- **Restaurant Response:** If restaurant doesn't respond in 3-5 minutes, order is auto-cancelled with full refund
- **Driver Assignment:** If no driver accepts within 10 minutes, customer is notified and can cancel for full refund
- **Pickup Delay:** If driver doesn't pick up within reasonable time (varies by restaurant prep time), order is reassigned
- **Delivery Delay:** If delivery takes >2x the estimated time, customer gets automatic credit/refund

**Timeout Metrics to Monitor:**

```text
METRIC                                  TARGET        ALERT IF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restaurant timeout rate                 <2%           >5%
Driver no-show rate                     <1%           >3%
Delivery delay rate (>1 hour)           <0.5%         >1%
Average restaurant response time        <30 seconds   >2 minutes
Average driver pickup time              <5 minutes    >10 minutes
```

If timeout rates exceed thresholds, it indicates systemic issues:
- High restaurant timeouts → Need to onboard more restaurants or improve tablet software
- High driver no-shows → Need better driver incentives or improve matching algorithm
- High delivery delays → Need more drivers in area or better route optimization

---

### 🔴 Advanced Level: Distributed State Management

**Eventual Consistency Challenges:**

```text
PROBLEM: Order status updated in multiple places
- PostgreSQL (source of truth)
- Redis (cache for fast reads)
- Cassandra (analytics and history)
- Customer's mobile app (WebSocket update)

SCENARIO: Race Condition
Time    Service             Action
00:00   Restaurant Service  Accepts order → writes to PostgreSQL (status: ACCEPTED)
00:01   Restaurant Service  Publishes event to Kafka: "OrderAccepted"
00:02   Cache Service       Receives event → updates Redis cache
00:01   Customer App        Requests order status from API
00:02   Order Service       Reads from Redis → still shows CONFIRMED (cache not updated yet!)
00:03   Customer App        Shows "Confirmed" (wrong state)
00:04   Cache Service       Redis updated to ACCEPTED
00:05   Customer App        Receives WebSocket update → shows "Accepted" (correct state)

RESULT: Customer sees wrong state for 3 seconds (acceptable for food delivery)

SOLUTION: Eventually consistent reads with version numbers
```

**Version Vectors (Detecting Conflicts):**

```python
class OrderWithVersion:
    def __init__(self, order_id):
        self.order_id = order_id
        self.status = OrderState.PLACED
        self.version = 1  # Monotonically increasing version
        self.vector_clock = {}  # For distributed version tracking
    
    def update_status(self, new_status, service_id):
        """
        Update status with vector clock for conflict detection
        
        service_id: Which service made the update (e.g., "restaurant_service")
        """
        # Increment version
        self.version += 1
        
        # Update vector clock (Lamport timestamp)
        if service_id not in self.vector_clock:
            self.vector_clock[service_id] = 0
        self.vector_clock[service_id] += 1
        
        self.status = new_status
        
        return {
            'order_id': self.order_id,
            'status': self.status,
            'version': self.version,
            'vector_clock': self.vector_clock
        }

# Example: Concurrent updates from restaurant and driver
order = OrderWithVersion(order_id=98765)

# Restaurant service updates
order.update_status(OrderState.ACCEPTED, service_id="restaurant_service")
# version=2, vector_clock={"restaurant_service": 1}

# Driver service updates (concurrently, before hearing about restaurant update)
order.update_status(OrderState.ASSIGNED, service_id="driver_service")
# version=3, vector_clock={"restaurant_service": 1, "driver_service": 1}

# Conflict resolution: Compare vector clocks
# If clocks are incomparable (concurrent updates), use business logic
# In this case: ASSIGNED takes precedence over ACCEPTED (later in state machine)
```

**Saga Pattern for Order Placement:**

```python
class OrderPlacementSaga:
    """
    Distributed transaction for order placement
    Ensures all services are coordinated or rolled back
    """
    
    def execute(self, order_request):
        """
        Execute saga: order placement with compensating transactions
        """
        saga_id = generate_uuid()
        
        try:
            # Step 1: Create order (Order Service)
            order = self.create_order(order_request)
            self.log_saga_step(saga_id, "create_order", order.id)
            
            # Step 2: Reserve inventory (Restaurant Service)
            self.reserve_inventory(order.id, order.items)
            self.log_saga_step(saga_id, "reserve_inventory", order.id)
            
            # Step 3: Charge payment (Payment Service)
            payment = self.charge_payment(order.id, order.total)
            self.log_saga_step(saga_id, "charge_payment", payment.id)
            
            # Step 4: Assign driver (Matching Service)
            driver = self.assign_driver(order.id)
            self.log_saga_step(saga_id, "assign_driver", driver.id)
            
            # Success! Commit saga
            self.commit_saga(saga_id)
            return order
        
        except InventoryNotAvailable as e:
            # Compensation: Cancel order (no payment charged yet)
            self.compensate_create_order(order.id)
            raise OrderCreationFailed("Item not available")
        
        except PaymentFailed as e:
            # Compensation: Unreserve inventory, cancel order
            self.compensate_reserve_inventory(order.id)
            self.compensate_create_order(order.id)
            raise OrderCreationFailed("Payment declined")
        
        except NoDriverAvailable as e:
            # Compensation: Refund payment, unreserve inventory, cancel order
            self.compensate_charge_payment(payment.id)
            self.compensate_reserve_inventory(order.id)
            self.compensate_create_order(order.id)
            raise OrderCreationFailed("No drivers available")
    
    def compensate_create_order(self, order_id):
        """Cancel order (compensating transaction)"""
        transition_order_state(order_id, OrderState.CANCELLED, actor="system")
    
    def compensate_reserve_inventory(self, order_id):
        """Release reserved inventory"""
        kafka.publish('inventory_unreserve', {'order_id': order_id})
    
    def compensate_charge_payment(self, payment_id):
        """Refund payment"""
        kafka.publish('payment_refund', {'payment_id': payment_id})
```

**Order Modification (Complex Case):**

```python
def modify_order(order_id, new_items, customer_id):
    """
    Allow customer to modify order (add/remove items)
    Only allowed within 2 minutes of placement
    """
    # 1. Load order and validate
    order = db.get_order(order_id)
    
    if order.customer_id != customer_id:
        raise UnauthorizedError("Not your order")
    
    # Can only modify if order is still early in lifecycle
    if order.status not in [OrderState.PLACED, OrderState.CONFIRMED]:
        raise InvalidOperationError(
            "Cannot modify order after restaurant starts preparing"
        )
    
    # Check time window (2 minutes)
    if datetime.utcnow() - order.created_at > timedelta(minutes=2):
        raise InvalidOperationError("Modification window expired")
    
    # 2. Calculate price difference
    old_total = order.total
    new_total = calculate_order_total(new_items)
    price_diff = new_total - old_total
    
    # 3. Update order in transaction
    with db.transaction():
        # Update order items
        db.delete_order_items(order_id)
        db.insert_order_items(order_id, new_items)
        
        # Update order total
        db.update_order(order_id, total=new_total)
        
        # Charge additional amount or refund difference
        if price_diff > 0:
            payment_service.charge_additional(order_id, price_diff)
        elif price_diff < 0:
            payment_service.refund_partial(order_id, abs(price_diff))
        
        # Log modification
        db.insert_order_modification_log(
            order_id=order_id,
            old_total=old_total,
            new_total=new_total,
            modified_at=datetime.utcnow()
        )
    
    # 4. Notify restaurant of modification
    kafka.publish('order_modified', {
        'order_id': order_id,
        'old_items': order.items,
        'new_items': new_items
    })
    
    return db.get_order(order_id)
```

---

### Key Takeaways

✅ **State machine with 12+ states** ensures order progresses correctly through lifecycle

✅ **Valid state transitions** prevent invalid operations (can't deliver before pickup)

✅ **Timeout handling** auto-transitions stuck orders (5 min restaurant acceptance, 30 min pickup)

✅ **Event sourcing with Kafka** provides audit trail and enables event replay

✅ **Saga pattern** manages distributed transactions with compensating actions

✅ **Version vectors** detect and resolve concurrent updates from multiple services

✅ **Modification window (2 min)** allows customer changes while maintaining kitchen efficiency

---

**Think About It:**
- What if customer's payment method expires during delivery? When should we charge?
- How do we handle partial deliveries (driver delivers to wrong address, only half the items)?
- Should we allow restaurants to modify order (add free items as apology)?
- What happens if order is in DELIVERED state but customer claims food never arrived?

---

## 7. Real-Time Driver Matching

### What You'll Learn
- Geospatial algorithms for driver discovery
- Scoring and ranking drivers for assignment
- Batch delivery optimization
- Handling driver rejections and reassignments
- Real-time availability tracking

### Why This Matters
Matching the right driver to an order affects delivery time, cost, and customer satisfaction. Uber Eats must find and assign a driver within 30 seconds while considering 50+ factors: distance, driver rating, vehicle type, current load, acceptance rate, earnings today, etc. Poor matching means long delivery times (cold food) or inefficient routes (unhappy drivers).

---

### 🟢 Beginner Level: Finding Drivers Nearby

**The Basic Concept:**

When an order is ready, find all drivers within a radius and pick the closest one:

```text
Order placed at restaurant: Joe's Pizza (40.7500°N, -73.9900°W)

Step 1: Find drivers within 5 km radius
Driver A: 0.8 km away (2 min drive)
Driver B: 1.5 km away (4 min drive)
Driver C: 3.2 km away (8 min drive)
Driver D: 7.0 km away (outside radius, ignored)

Step 2: Filter by availability
Driver A: Online, no current delivery ✅
Driver B: Online, but already delivering another order ❌
Driver C: Offline ❌

Step 3: Assign to closest available driver
RESULT: Assign to Driver A (0.8 km away)
```

**Geospatial Distance Calculation (Haversine Formula):**

To find nearby drivers, we need to calculate the distance between two GPS coordinates (latitude/longitude). The Haversine formula calculates the great-circle distance between two points on a sphere.

**Haversine Formula:**

```text
Distance Formula for Earth's Surface:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Given: Point A (lat₁, lon₁) and Point B (lat₂, lon₂)
Earth radius: R = 6,371 km

Step 1: Convert degrees to radians
    φ₁ = lat₁ × π/180
    φ₂ = lat₂ × π/180
    Δφ = (lat₂ - lat₁) × π/180
    Δλ = (lon₂ - lon₁) × π/180

Step 2: Apply Haversine formula
    a = sin²(Δφ/2) + cos(φ₁) × cos(φ₂) × sin²(Δλ/2)
    c = 2 × arcsin(√a)
    distance = R × c

Example Calculation:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restaurant: (40.7500°N, -73.9900°W)
Driver:     (40.7484°N, -73.9857°W)

Δlat = 0.0016°, Δlon = 0.0043°
Result: distance = 0.472 km (472 meters)
```

**Why Haversine, Not Pythagorean Distance?**

You can't use simple Pythagorean theorem (√(Δx² + Δy²)) because:
1. Earth is a sphere, not flat
2. Longitude lines converge at poles (1° longitude at equator ≠ 1° longitude at North Pole)
3. At New York latitude, Pythagorean would be off by 20-30%

**Optimization:** For very small distances (<10 km), you can use faster approximations, but Haversine is standard.

---

### 🟡 Intermediate Level: Intelligent Matching Algorithms

**Multi-Factor Scoring:**

Simply choosing the closest driver isn't optimal. We need to balance multiple factors to ensure good outcomes for customers, drivers, and the platform.

**The Six Scoring Factors:**

```text
FACTOR              WEIGHT   FORMULA                                      RATIONALE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Distance         40%      score = 1 / (1 + distance_km)               Faster delivery = happier customers
2. Driver Rating    20%      score = rating / 5.0                        High-rated drivers give better service
3. Acceptance Rate  15%      score = acceptance_rate                      Reliable drivers who don't reject orders
4. Vehicle Type     10%      score = 1.0 if match, 0.7 if mismatch      Bike for small, car for large orders
5. Earnings Fairness 10%     score = 1.0 if below avg, 0.5 if above     Spread orders fairly among drivers
6. Idle Time        5%       score = min(1.0, idle_minutes / 30)        Reward drivers who've been waiting

TOTAL SCORE = Σ (factor_score × weight)
```

**Detailed Scoring Examples:**

```text
Example 1: Driver A vs Driver B
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

DRIVER A:
├─ Distance: 0.8 km       → score = 1/(1+0.8) = 0.556 → weighted = 0.556 × 0.40 = 0.222
├─ Rating: 4.8/5.0        → score = 0.960           → weighted = 0.960 × 0.20 = 0.192
├─ Acceptance: 95%        → score = 0.950           → weighted = 0.950 × 0.15 = 0.143
├─ Vehicle: bike (match)  → score = 1.000           → weighted = 1.000 × 0.10 = 0.100
├─ Earnings: $80 today    → score = 1.000           → weighted = 1.000 × 0.10 = 0.100
└─ Idle: 45 min           → score = 1.000           → weighted = 1.000 × 0.05 = 0.050
TOTAL SCORE = 0.807

DRIVER B:
├─ Distance: 1.5 km       → score = 1/(1+1.5) = 0.400 → weighted = 0.400 × 0.40 = 0.160
├─ Rating: 4.9/5.0        → score = 0.980           → weighted = 0.980 × 0.20 = 0.196
├─ Acceptance: 92%        → score = 0.920           → weighted = 0.920 × 0.15 = 0.138
├─ Vehicle: car (mismatch)→ score = 0.700           → weighted = 0.700 × 0.10 = 0.070
├─ Earnings: $180 today   → score = 0.500           → weighted = 0.500 × 0.10 = 0.050
└─ Idle: 10 min           → score = 0.333           → weighted = 0.333 × 0.05 = 0.017
TOTAL SCORE = 0.631

RESULT: Assign to Driver A (0.807 > 0.631)
```

**Why These Specific Weights?**

1. **Distance (40%):** Most important because it directly affects delivery time. Customer ordered food because they're hungry NOW.

2. **Rating (20%):** High-rated drivers provide better customer experience (polite, handle food carefully, navigate efficiently).

3. **Acceptance Rate (15%):** Drivers who frequently reject cause delays. If we assign to driver with 50% acceptance rate, we'll likely need to reassign.

4. **Vehicle Type (10%):** Bikes are faster in traffic for small orders. Cars needed for large catering orders.

5. **Earnings Fairness (10%):** Prevent situation where one driver gets all orders while others sit idle. Improves driver retention.

6. **Idle Time (5%):** Small factor to prevent starvation (driver waiting 2 hours should get next order).

**Interview Discussion Point:**

"These weights are tunable via A/B testing. Uber Eats might test 35% distance vs 45% distance and measure impact on customer satisfaction, delivery time, and driver retention. The optimal weights vary by city (NYC traffic favors bikes more than rural areas)."

**Geospatial Indexing with Redis:**

Finding drivers within a radius needs to be FAST (<100ms). Scanning entire driver database (1M drivers) is too slow. We use Redis geospatial indexing.

**Redis GEORADIUS Command:**

```text
DATA STRUCTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Redis Sorted Set: "driver_locations"
Key Format: "driver:{driver_id}"
Geohash Encoding: Automatically calculated by Redis

Example Data:
  driver:12345 → (lat: 40.7500, lon: -73.9900, geohash: dr5regw)
  driver:67890 → (lat: 40.7484, lon: -73.9857, geohash: dr5regu)
  driver:11111 → (lat: 40.7600, lon: -74.0000, geohash: dr5ru3w)

QUERY PSEUDOCODE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
FUNCTION find_nearby_drivers(restaurant_lat, restaurant_lon, radius_km):
    
    // Redis command: GEORADIUS key longitude latitude radius unit [options]
    // Note: Redis expects LONGITUDE first, then LATITUDE!
    results = redis.execute(
        command = "GEORADIUS",
        key = "driver_locations",
        longitude = restaurant_lon,
        latitude = restaurant_lat,
        radius = radius_km,
        unit = "km",
        options = ["WITHDIST", "WITHCOORD", "ASC"]
        // WITHDIST: include distance
        // WITHCOORD: include coordinates
        // ASC: sort by distance (closest first)
    )
    
    available_drivers = []
    
    FOR EACH result IN results:
        driver_key = result.key           // "driver:12345"
        distance_km = result.distance     // 0.8
        coordinates = result.coords       // [lon, lat]
        
        // Extract driver ID from key
        driver_id = extract_id(driver_key)  // 12345
        
        // Load full driver details from PostgreSQL
        driver = database.get_driver(driver_id)
        
        // Filter: only include online drivers without current order
        IF driver.is_online AND driver.current_order_id IS NULL:
            available_drivers.append({
                driver_id: driver_id,
                distance_km: distance_km,
                driver_info: driver
            })
    
    RETURN available_drivers

PERFORMANCE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Query Time: O(N + log(M)) where N = results returned, M = total drivers
Typical: <10ms to find drivers within 5km radius (even with 1M drivers)
Why Fast: Redis uses geohash for spatial indexing (similar to QuadTree)
```

**How Geohash Works:**

```text
Geohash divides the world into a grid and assigns each cell a hash string.
Nearby locations have similar hash prefixes.

Example:
  Location A: dr5regw3p6h9  (Manhattan, NY)
  Location B: dr5regu3nxyz  (close to A, shares prefix "dr5reg")
  Location C: ezs42vhm8dn7  (London, UK - totally different prefix)

To find nearby drivers:
1. Calculate geohash for restaurant location
2. Query drivers with similar geohash prefixes
3. Much faster than checking distance to every driver
```

**Update Driver Location (Pseudocode):**

```text
FUNCTION update_driver_location(driver_id, latitude, longitude):
    // Called every 1 second while driver is online
    
    // Update Redis geospatial index
    redis.execute(
        command = "GEOADD",
        key = "driver_locations",
        longitude = longitude,  // longitude first!
        latitude = latitude,
        member = "driver:" + driver_id
    )
    
    // Also update PostgreSQL for persistence
    database.update(
        table = "drivers",
        where = {id: driver_id},
        set = {
            current_latitude: latitude,
            current_longitude: longitude,
            location_updated_at: current_timestamp
        }
    )
    
    // Set TTL to auto-remove offline drivers after 60 seconds
    redis.execute(
        command = "EXPIRE",
        key = "driver:" + driver_id,
        seconds = 60
    )
```

**Real-World Usage (Uber):**

Uber's driver matching runs at ~100 QPS during peak hours in NYC:
- GEORADIUS query: 5-10ms
- Score 20 candidates: 5ms
- Total matching time: <20ms (well under 30-second requirement)
```

**Assignment Algorithm:**

Once we have scored drivers, we need to actually assign the order. This involves offering the order to drivers in score order and handling rejections.

**Assignment Flow (Pseudocode):**

```text
FUNCTION assign_driver_to_order(order_id, max_attempts=3):
    
    // Step 1: Load order and restaurant details
    order = database.get_order(order_id)
    restaurant = database.get_restaurant(order.restaurant_id)
    
    // Step 2: Find nearby drivers (start with 5km radius)
    nearby_drivers = find_nearby_drivers(
        lat = restaurant.latitude,
        lon = restaurant.longitude,
        radius_km = 5
    )
    
    // Step 3: Handle no drivers found
    IF nearby_drivers.length == 0:
        // Expand search radius to 10km
        nearby_drivers = find_nearby_drivers(
            lat = restaurant.latitude,
            lon = restaurant.longitude,
            radius_km = 10
        )
        
        IF nearby_drivers.length == 0:
            // Still no drivers - notify customer and retry
            notification_service.send(
                user_id = order.customer_id,
                message = "Finding a driver for you... This may take a few minutes."
            )
            
            // Schedule retry in 60 seconds
            job_queue.schedule(
                function = assign_driver_to_order,
                params = {order_id: order_id},
                delay_seconds = 60
            )
            
            RETURN null  // No driver assigned yet
    
    // Step 4: Score all available drivers
    scored_drivers = []
    FOR EACH driver IN nearby_drivers:
        score = calculate_driver_score(driver, order, restaurant)
        scored_drivers.append({
            driver_id: driver.id,
            score: score,
            distance_km: driver.distance_km
        })
    
    // Step 5: Sort by score (highest first)
    scored_drivers.sort_descending_by(score)
    
    // Step 6: Offer order to top-ranked drivers
    FOR attempt = 1 TO min(max_attempts, scored_drivers.length):
        
        current_driver = scored_drivers[attempt]
        driver_id = current_driver.driver_id
        
        // Send offer to driver (30-second timeout)
        offer_response = offer_order_to_driver(
            order_id = order_id,
            driver_id = driver_id,
            timeout_seconds = 30
        )
        
        IF offer_response == ACCEPTED:
            // SUCCESS! Driver accepted the order
            
            // Update order status in database
            database.update_order(
                order_id = order_id,
                driver_id = driver_id,
                status = "ASSIGNED",
                assigned_at = current_timestamp
            )
            
            // Publish event for other services
            kafka.publish(
                topic = "order_assigned",
                message = {
                    order_id: order_id,
                    driver_id: driver_id,
                    assignment_score: current_driver.score,
                    assignment_attempt: attempt,
                    timestamp: current_timestamp
                }
            )
            
            // Notify customer
            notification_service.send(
                user_id = order.customer_id,
                message = "Driver found! John is heading to the restaurant."
            )
            
            RETURN driver_id
        
        ELSE IF offer_response == REJECTED:
            // Driver manually rejected, try next driver
            log_info("Driver " + driver_id + " rejected order " + order_id)
            metrics.increment("driver.rejection.manual")
            CONTINUE to next driver
        
        ELSE IF offer_response == TIMEOUT:
            // Driver didn't respond in 30 seconds, try next driver  
            log_warning("Driver " + driver_id + " timed out on order " + order_id)
            metrics.increment("driver.rejection.timeout")
            CONTINUE to next driver
    
    // Step 7: All attempts failed - retry with larger radius
    log_warning("Order " + order_id + ": All " + max_attempts + " drivers rejected")
    
    // Schedule retry with expanded radius (15km) after 2 minutes
    job_queue.schedule(
        function = assign_driver_to_order,
        params = {order_id: order_id, radius_km: 15},
        delay_seconds = 120
    )
    
    // Notify customer of delay
    notification_service.send(
        user_id = order.customer_id,
        message = "We're still finding the best driver for you. Thanks for your patience!"
    )
    
    RETURN null  // Assignment failed, will retry

FUNCTION offer_order_to_driver(order_id, driver_id, timeout_seconds):
    
    // Send push notification to driver app
    notification_service.send_to_driver(
        driver_id = driver_id,
        notification_type = "NEW_ORDER_OFFER",
        order_details = {
            order_id: order_id,
            restaurant_name: "Joe's Pizza",
            pickup_address: "123 Main St",
            delivery_address: "456 Oak Ave",
            estimated_payout: "$8.50",
            estimated_distance: "3.2 km",
            estimated_time: "25 minutes"
        },
        action_buttons = ["ACCEPT", "REJECT"]
    )
    
    // Wait for driver response (with timeout)
    response = wait_for_driver_response(
        driver_id = driver_id,
        order_id = order_id,
        timeout_seconds = timeout_seconds
    )
    
    RETURN response  // ACCEPTED, REJECTED, or TIMEOUT
```

**Assignment Strategy Decision Tree:**

```text
START: New order needs driver
    ↓
Find drivers within 5km radius
    ↓
┌──────────────────────┐
│ Drivers found?       │
└──────────────────────┘
    ↓ NO                      ↓ YES
Expand to 10km radius      Score all drivers
    ↓                         ↓
┌──────────────────────┐  Sort by score (best first)
│ Drivers found?       │     ↓
└──────────────────────┘  Offer to #1 driver (30 sec timeout)
    ↓ NO                      ↓
Notify customer          ┌──────────────────────┐
"Finding driver..."      │ Driver response?     │
Retry in 60 sec          └──────────────────────┘
    ↓                      ↓ ACCEPT   ↓ REJECT/TIMEOUT
Return NULL             SUCCESS!     Offer to #2 driver (30 sec)
                           ↓             ↓
                     Update order    ┌──────────────────────┐
                     Notify all      │ Driver response?     │
                     Publish event   └──────────────────────┘
                                       ↓ ACCEPT   ↓ REJECT/TIMEOUT
                                     SUCCESS!     Offer to #3 driver
                                                     ↓
                                                  (if still rejected)
                                                     ↓
                                                  Expand to 15km
                                                  Retry in 2 minutes
```

**Metrics to Track:**

```text
METRIC                              TARGET      ALERT IF
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Average time to assign driver       <15 seconds  >30 seconds
Driver rejection rate (manual)      <10%         >20%
Driver timeout rate (no response)   <5%          >15%
Orders needing radius expansion     <3%          >10%
Orders failing assignment (3 tries) <1%          >3%
Average driver score at assignment  >0.75        <0.60
```

**Real-World Examples:**

**Uber Eats:**
- Tries top 3 drivers sequentially (not parallel to avoid double-assignment)
- 15-second timeout per driver (shorter than our 30-second)
- Shows driver estimate earnings prominently ($6-$9 for this trip)
- Dynamic radius: starts at 3km in dense cities, 10km in suburbs

**DoorDash:**
- Uses "batch matching" - sometimes offers 2-3 orders to one driver
- Willing to wait up to 5 minutes before expanding radius
- Prioritizes "Dashers" (their brand for drivers) who are closer to customer's area for faster delivery

**Interview Discussion:**

"Why sequential offers instead of parallel? If we offer to 3 drivers simultaneously and all accept, we have a conflict. Sequential is safer but slower. Some platforms use a hybrid: offer to top driver, wait 15 seconds, then simultaneously offer to #2 and #3 if #1 times out."

---

### 🔴 Advanced Level: Batch Optimization & ML-Powered Matching

**Batch Delivery (Multiple Orders, One Driver):**

```python
def optimize_batch_delivery(driver_id, orders):
    """
    Optimize route for driver to pick up and deliver multiple orders
    Using Traveling Salesman Problem (TSP) approximation
    """
    # Example: Driver has 3 orders to deliver
    # Order A: Restaurant R1 → Customer C1
    # Order B: Restaurant R2 → Customer C2
    # Order C: Restaurant R3 → Customer C3
    
    # Goal: Find optimal route through R1, R2, R3, C1, C2, C3
    # Constraint: Must pick up from restaurant before delivering to customer
    
    locations = []
    for order in orders:
        restaurant = db.get_restaurant(order.restaurant_id)
        customer_address = order.delivery_address
        
        locations.append({
            'type': 'pickup',
            'order_id': order.id,
            'location': (restaurant.latitude, restaurant.longitude),
            'earliest_time': order.estimated_ready_time
        })
        locations.append({
            'type': 'delivery',
            'order_id': order.id,
            'location': (customer_address.latitude, customer_address.longitude),
            'depends_on': f'pickup_{order.id}'  # Must happen after pickup
        })
    
    # Use Google Maps Directions API or internal routing service
    route = routing_service.optimize_route(
        start_location=driver.current_location,
        locations=locations,
        constraints={
            'max_delivery_time_per_order': 45,  # Each order < 45 min
            'max_total_time': 90,                # Total route < 90 min
            'temperature_sensitive': True        # Prioritize hot food
        }
    )
    
    return route

# Example optimal route:
# Driver → R1 (pickup A) → R2 (pickup B) → C1 (deliver A) → C2 (deliver B) → R3 (pickup C) → C3 (deliver C)
```

**Machine Learning for Demand Prediction:**

```python
class DeliveryDemandPredictor:
    """
    Predict delivery demand to pre-position drivers
    Uses historical data + real-time features
    """
    
    def __init__(self):
        self.model = self.load_trained_model()
    
    def predict_demand(self, zone_id, timestamp):
        """
        Predict number of orders in next 30 minutes for a zone
        
        Features:
        - Day of week (Friday > Monday)
        - Hour of day (lunch/dinner rush)
        - Weather (rain increases orders)
        - Local events (concerts, sports games)
        - Historical average for this zone/time
        """
        features = self.extract_features(zone_id, timestamp)
        
        predicted_orders = self.model.predict(features)
        
        return predicted_orders
    
    def extract_features(self, zone_id, timestamp):
        """Extract features for prediction"""
        # Time features
        day_of_week = timestamp.weekday()  # 0=Monday, 6=Sunday
        hour = timestamp.hour
        is_weekend = day_of_week >= 5
        is_lunch_rush = 11 <= hour <= 14
        is_dinner_rush = 17 <= hour <= 21
        
        # Weather features
        weather = weather_api.get_current(zone_id)
        is_raining = weather.precipitation > 0
        temperature_f = weather.temperature
        
        # Historical features
        historical_avg = db.get_avg_orders(zone_id, day_of_week, hour)
        
        # Event features
        events = events_api.get_events(zone_id, timestamp)
        has_major_event = len(events) > 0
        
        return {
            'day_of_week': day_of_week,
            'hour': hour,
            'is_weekend': is_weekend,
            'is_lunch_rush': is_lunch_rush,
            'is_dinner_rush': is_dinner_rush,
            'is_raining': is_raining,
            'temperature': temperature_f,
            'historical_avg': historical_avg,
            'has_major_event': has_major_event
        }
    
    def reposition_drivers(self):
        """
        Suggest driver repositioning based on predicted demand
        Run every 15 minutes
        """
        zones = db.get_all_zones()
        timestamp = datetime.utcnow() + timedelta(minutes=30)
        
        demand_predictions = {}
        for zone in zones:
            demand = self.predict_demand(zone.id, timestamp)
            current_drivers = db.count_available_drivers(zone.id)
            
            demand_predictions[zone.id] = {
                'predicted_demand': demand,
                'current_supply': current_drivers,
                'imbalance': demand - current_drivers
            }
        
        # Find zones with excess demand (need more drivers)
        excess_demand_zones = [
            (zone_id, data['imbalance'])
            for zone_id, data in demand_predictions.items()
            if data['imbalance'] > 5  # Need 5+ more drivers
        ]
        
        # Find zones with excess supply (can spare drivers)
        excess_supply_zones = [
            (zone_id, abs(data['imbalance']))
            for zone_id, data in demand_predictions.items()
            if data['imbalance'] < -5  # Have 5+ extra drivers
        ]
        
        # Suggest repositioning
        for demand_zone_id, deficit in excess_demand_zones:
            for supply_zone_id, surplus in excess_supply_zones:
                distance = calculate_zone_distance(supply_zone_id, demand_zone_id)
                
                if distance < 5:  # Only reposition within 5 km
                    notify_drivers_to_reposition(
                        from_zone=supply_zone_id,
                        to_zone=demand_zone_id,
                        incentive=calculate_incentive(deficit)
                    )
        
        return demand_predictions

# Example: During Friday dinner rush in Manhattan
predictor = DeliveryDemandPredictor()
predictions = predictor.reposition_drivers()
# Result: Notify drivers in quiet Brooklyn to move to busy Manhattan (with $5 incentive)
```

---

### Key Takeaways

✅ **Geospatial indexing (Redis GEORADIUS)** enables sub-100ms driver lookups within radius

✅ **Multi-factor scoring** considers distance, rating, acceptance rate, vehicle type, earnings fairness

✅ **Assignment retries with backoff:** Try 3 best drivers, expand radius if all reject

✅ **Batch optimization** allows one driver to handle multiple orders (Traveling Salesman Problem)

✅ **ML demand prediction** enables proactive driver repositioning (reduce wait times by 20%)

✅ **Driver marketplace dynamics:** Balance driver earnings (fairness) with customer wait times (speed)

---

## 8. Location Tracking & ETA Calculation

### What You'll Learn
- Real-time GPS tracking architecture
- ETA calculation with traffic awareness
- WebSocket implementation for live updates
- Handling unreliable GPS signals
- Privacy considerations for location data

### Why This Matters
Customers refresh the order tracking screen 10+ times during delivery. Accurate ETAs build trust—if ETA says "5 minutes" but driver arrives in 15, customer is frustrated. Uber Eats tracks 200K concurrent drivers with 1-second GPS updates (7.2M location updates per hour), requiring specialized time-series infrastructure.

---

### 🟢 Beginner Level: GPS Tracking Basics

**How It Works:**

```text
Driver's Phone (GPS Receiver)
   ↓ Every 1 second
   Sends: {latitude: 40.7500, longitude: -73.9900, timestamp: "2025-11-04T19:20:00Z"}
   ↓ HTTPS POST
Location Service (Backend)
   ↓ Stores in Cassandra (time-series DB)
   ↓ Updates Redis cache (latest location)
   ↓ Broadcasts via WebSocket
Customer's Phone
   ↓ Receives location update
   Updates map (shows driver moving)
```

**ETA Calculation (Simple):**

```python
def calculate_simple_eta(driver_location, customer_location, avg_speed_kmh=30):
    """
    Calculate ETA assuming constant speed (no traffic)
    """
    # Distance in km
    distance = haversine_distance(
        driver_location.latitude,
        driver_location.longitude,
        customer_location.latitude,
        customer_location.longitude
    )
    
    # Time in hours
    time_hours = distance / avg_speed_kmh
    
    # Convert to minutes
    eta_minutes = time_hours * 60
    
    return int(eta_minutes)

# Example:
driver_loc = (40.7500, -73.9900)
customer_loc = (40.7484, -73.9857)
eta = calculate_simple_eta(driver_loc, customer_loc)
# Result: 1 minute (very close)
```

---

### 🟡 Intermediate Level: Production-Grade Tracking

**WebSocket Implementation:**

```python
import asyncio
import websockets
import json

class LocationTracker:
    def __init__(self):
        self.active_connections = {}  # {order_id: [websocket1, websocket2, ...]}
    
    async def handle_customer_connection(self, websocket, order_id):
        """
        Handle WebSocket connection from customer app
        Customer subscribes to location updates for their order
        """
        # Register connection
        if order_id not in self.active_connections:
            self.active_connections[order_id] = []
        self.active_connections[order_id].append(websocket)
        
        try:
            # Send initial location
            order = db.get_order(order_id)
            driver_id = order.driver_id
            latest_location = redis_client.hgetall(f'driver_location:{driver_id}')
            
            await websocket.send(json.dumps({
                'type': 'location_update',
                'driver_id': driver_id,
                'latitude': float(latest_location['latitude']),
                'longitude': float(latest_location['longitude']),
                'eta_minutes': calculate_eta(order_id),
                'timestamp': latest_location['timestamp']
            }))
            
            # Keep connection alive
            while True:
                # Wait for ping from client
                message = await websocket.recv()
                
                if message == 'ping':
                    await websocket.send('pong')
        
        except websockets.ConnectionClosed:
            # Remove connection when customer closes app
            self.active_connections[order_id].remove(websocket)
    
    async def broadcast_location_update(self, driver_id, latitude, longitude):
        """
        Broadcast driver location to all customers tracking this driver
        Called every 1 second when driver location is updated
        """
        # Find all orders assigned to this driver
        orders = db.get_active_orders_for_driver(driver_id)
        
        for order in orders:
            order_id = order.id
            
            if order_id in self.active_connections:
                # Calculate ETA for this specific order
                eta_minutes = calculate_eta(order_id)
                
                # Prepare message
                message = json.dumps({
                    'type': 'location_update',
                    'driver_id': driver_id,
                    'latitude': latitude,
                    'longitude': longitude,
                    'eta_minutes': eta_minutes,
                    'timestamp': datetime.utcnow().isoformat()
                })
                
                # Send to all connected customers for this order
                for websocket in self.active_connections[order_id]:
                    try:
                        await websocket.send(message)
                    except websockets.ConnectionClosed:
                        # Connection closed, remove it
                        self.active_connections[order_id].remove(websocket)

# Start WebSocket server
tracker = LocationTracker()

async def main():
    async with websockets.serve(tracker.handle_customer_connection, "0.0.0.0", 8765):
        await asyncio.Future()  # Run forever

asyncio.run(main())
```

**Traffic-Aware ETA:**

**Traffic-Aware ETA Calculation:**

Simple distance-based ETA doesn't account for real-world factors like traffic, road closures, or construction. Uber Eats integrates with mapping services (Google Maps, Mapbox) to get accurate travel times.

**ETA Calculation Approach:**

```text
FUNCTION calculate_traffic_aware_eta(order_id):
    
    // Step 1: Get current positions
    order = database.get_order(order_id)
    driver_id = order.driver_id
    
    // Get driver's latest GPS location from Redis cache
    driver_location = redis.get("driver_location:" + driver_id)
    // Returns: {latitude: 40.7500, longitude: -73.9900, timestamp: "..."}
    
    // Get customer's delivery address from order
    customer_location = {
        latitude: order.delivery_latitude,
        longitude: order.delivery_longitude
    }
    
    // Step 2: Call Google Maps Directions API
    api_request = {
        origin: driver_location,
        destination: customer_location,
        mode: "driving",
        departure_time: "now",  // Use current traffic conditions
        traffic_model: "best_guess",  // Consider typical traffic patterns
        alternatives: false  // Just need one route
    }
    
    directions_response = google_maps_api.directions(api_request)
    
    // Step 3: Parse response
    IF directions_response.status == "OK":
        // Extract duration accounting for current traffic
        route = directions_response.routes[0]
        leg = route.legs[0]
        
        duration_in_traffic_seconds = leg.duration_in_traffic.value
        eta_minutes = duration_in_traffic_seconds / 60
        
        // Step 4: Cache result (valid for 1 minute)
        redis.setex(
            key = "eta:" + order_id,
            value = eta_minutes,
            ttl_seconds = 60  // Cache expires after 1 minute
        )
        
        // Step 5: Return ETA
        RETURN eta_minutes
    
    ELSE:
        // Fallback if API fails
        RETURN calculate_simple_eta(driver_location, customer_location)

FUNCTION calculate_simple_eta(driver_location, customer_location):
    // Fallback calculation using average speed
    
    distance_km = haversine_distance(
        driver_location.latitude,
        driver_location.longitude,
        customer_location.latitude,
        customer_location.longitude
    )
    
    average_speed_kmh = 30  // Assume 30 km/h in city
    time_hours = distance_km / average_speed_kmh
    eta_minutes = time_hours * 60
    
    RETURN eta_minutes
```

**Google Maps API Response Structure:**

```json
{
  "status": "OK",
  "routes": [{
    "legs": [{
      "distance": {
        "value": 5200,  // meters
        "text": "5.2 km"
      },
      "duration": {
        "value": 600,  // seconds (10 min without traffic)
        "text": "10 mins"
      },
      "duration_in_traffic": {
        "value": 720,  // seconds (12 min with current traffic)
        "text": "12 mins"
      },
      "start_address": "123 Main St, New York, NY",
      "end_address": "456 Oak Ave, New York, NY"
    }],
    "overview_polyline": {
      "points": "encoded_polyline_string_here"
    }
  }]
}
```

**Why Traffic-Aware ETA is Critical:**

```text
SCENARIO: Rush Hour Delivery
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Simple ETA (distance/speed):
  Distance: 5 km
  Average speed: 30 km/h
  Calculated ETA: 10 minutes

Traffic-Aware ETA (Google Maps):
  Distance: 5 km (same)
  Current traffic: heavy congestion
  Actual travel time: 18 minutes
  
IMPACT:
  ❌ Simple ETA: Customer expects food in 10 min, gets it in 18 min → unhappy
  ✅ Traffic ETA: Customer expects food in 18 min, gets it in 18 min → satisfied
```

**ETA Update Frequency:**

- **Recalculate every 30 seconds** while driver is en route
- **Why not more frequent?** Google Maps API costs $5 per 1000 requests
  - 200K concurrent deliveries × 120 requests per delivery (30-min delivery ÷ 30-sec interval) = 24M API calls/hour
  - Cost: 24M / 1000 × $5 = $120,000/hour = $2.9M/day (!!)
  
**Cost Optimization:**

```text
STRATEGY 1: Reduce API call frequency
- Recalculate every 2 minutes instead of 30 seconds → 96% cost reduction
- Only update when driver's location changes significantly (>200 meters)

STRATEGY 2: Use self-hosted routing engine
- OpenStreetMap + OSRM (Open Source Routing Machine) 
- $0 per request (just infrastructure cost)
- Trade-off: Less accurate traffic data than Google

STRATEGY 3: Hybrid approach (Uber's strategy)
- Use Google Maps for initial route calculation
- Use simple dead reckoning for updates (driver is traveling at X speed toward destination)
- Recalculate with Google Maps every 5 minutes or if driver deviates from route
- Cost: 90% reduction while maintaining accuracy
```

**Real Example (Uber Eats in NYC):**

```text
Order placed: 7:00 PM (peak dinner hour)
Initial ETA: 35 minutes

7:05 PM - Driver picks up food
  → Google Maps ETA: 12 minutes (accounting for Times Square traffic)
  → Customer sees: "Arriving at 7:17 PM"

7:10 PM - Traffic accident on route
  → Driver automatically rerouted by Google Maps
  → New ETA: 15 minutes
  → Customer sees: "Arriving at 7:25 PM (delayed due to traffic)"

7:23 PM - Driver arrives
  → Actual time: 18 minutes from pickup
  → Close enough to ETA that customer is satisfied
```

---

### 🔴 Advanced Level: Optimizations & Edge Cases

**Location Data Storage (Cassandra):**

```cql
-- Time-series table for driver locations
CREATE TABLE driver_locations (
    driver_id BIGINT,
    timestamp TIMESTAMP,
    latitude DECIMAL,
    longitude DECIMAL,
    accuracy DECIMAL,      -- GPS accuracy in meters
    speed DECIMAL,         -- Speed in km/h
    bearing DECIMAL,       -- Direction (0-360 degrees)
    PRIMARY KEY (driver_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC)
  AND compaction = {'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS'}
  AND default_time_to_live = 2592000;  -- 30 days TTL

-- Query: Get driver's location history for last 10 minutes
SELECT * FROM driver_locations
WHERE driver_id = 54321
  AND timestamp > NOW() - INTERVAL '10 minutes'
ORDER BY timestamp DESC;

-- Efficient storage: 50 bytes/location × 3600 locations/hour × 200K drivers = 36 GB/hour
-- With 30-day retention: 36 GB × 24 × 30 = 25.9 TB (matches our capacity planning!)
```

**Handling Unreliable GPS:**

**Handling Unreliable GPS:**

GPS signals can be inaccurate due to tall buildings (urban canyons), tunnels, bad weather, or device issues. We need to validate and smooth GPS data before using it.

**GPS Validation Logic (Pseudocode):**

```text
FUNCTION validate_and_smooth_gps(driver_id, new_lat, new_lon, timestamp):
    
    // Step 1: Get recent location history (last 5 points, within 30 seconds)
    previous_locations = cassandra.query(
        "SELECT * FROM driver_locations 
         WHERE driver_id = ? AND timestamp > ? 
         ORDER BY timestamp DESC LIMIT 5",
        params = [driver_id, timestamp - 30_seconds]
    )
    
    // Step 2: If first location, accept it
    IF previous_locations.length == 0:
        RETURN {latitude: new_lat, longitude: new_lon}
    
    last_location = previous_locations[0]
    
    // Step 3: Calculate distance moved since last update
    distance_km = haversine_distance(
        last_location.latitude,
        last_location.longitude,
        new_lat,
        new_lon
    )
    
    // Step 4: Calculate time elapsed
    time_diff_seconds = (timestamp - last_location.timestamp).total_seconds()
    
    // Step 5: Calculate implied speed
    implied_speed_kmh = (distance_km / time_diff_seconds) * 3600
    
    // Step 6: Sanity check - reject impossible speeds
    IF implied_speed_kmh > 120:  // 120 km/h = 75 mph (highway speed limit)
        log_warning("Driver " + driver_id + ": GPS anomaly detected - " +
                   "speed " + implied_speed_kmh + " km/h is impossible")
        
        // Use last known good location instead
        RETURN {
            latitude: last_location.latitude,
            longitude: last_location.longitude
        }
    
    // Step 7: Smooth out GPS jitter using Exponential Moving Average
    smoothing_factor = 0.7  // Weight of new reading
    
    smoothed_lat = (smoothing_factor × new_lat) + 
                   ((1 - smoothing_factor) × last_location.latitude)
    
    smoothed_lon = (smoothing_factor × new_lon) + 
                   ((1 - smoothing_factor) × last_location.longitude)
    
    RETURN {latitude: smoothed_lat, longitude: smoothed_lon}
```

**Why Each Validation Step Matters:**

```text
VALIDATION                  PURPOSE                          EXAMPLE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Speed check (<120 km/h)     Detect GPS glitches          Driver appears to teleport 5km
                            or spoofing attempts          in 2 seconds → reject

Exponential smoothing       Reduce GPS jitter            GPS bounces ±20m due to
                            for smoother customer UX      building reflections → smooth

Time-based filtering        Ignore stale data            GPS update delayed by 30 sec
                                                          due to network → use last known

Distance threshold          Debounce tiny movements      GPS drifts 2m while parked
                            to reduce API calls           → don't update map
```

**Real-World GPS Issues:**

```text
ISSUE 1: Urban Canyon (Manhattan)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tall buildings reflect GPS signals → appears to jump between locations
Solution: Use last 3 locations to detect pattern, smooth aggressively

ISSUE 2: Tunnel (Lincoln Tunnel NYC)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
No GPS signal for 5 minutes while underground
Solution: Extrapolate position based on last known speed and direction
         Show customer "Driver is in tunnel, ETA unchanged"

ISSUE 3: GPS Spoofing (Driver Fraud)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Driver uses fake GPS app to appear at delivery location without going there
Detection: Check speed (>120 km/h impossible), compare with cellular triangulation,
          verify with landmarks (driver marked "arrived" but cellular shows 2km away)
```

**Privacy Considerations:**

After delivery is complete, we need to comply with GDPR and data privacy laws by reducing location precision.

**Privacy-Preserving Location Anonymization:**

```text
BACKGROUND JOB run_daily():
    // Run once per day to anonymize old location data
    
    FOR EACH driver IN all_drivers:
        
        // Get completed deliveries older than 30 days
        old_deliveries = database.query(
            "SELECT order_id, delivered_at FROM orders
             WHERE driver_id = ? AND status = 'COMPLETED'
             AND delivered_at < NOW() - INTERVAL '30 days'",
            params = [driver.id]
        )
        
        FOR EACH delivery IN old_deliveries:
            
            // Reduce GPS precision from 10m accuracy to 100m accuracy
            // This protects driver privacy while maintaining analytics value
            
            cassandra.execute(
                "UPDATE driver_locations
                 SET latitude = ?, longitude = ?
                 WHERE driver_id = ? AND timestamp BETWEEN ? AND ?",
                params = [
                    round_to_100m_precision(latitude),
                    round_to_100m_precision(longitude),
                    driver.id,
                    delivery.delivered_at - 1_hour,
                    delivery.delivered_at + 15_minutes
                ]
            )
        
        log_info("Anonymized location history for driver " + driver.id)

FUNCTION round_to_100m_precision(coordinate):
    // Round to 3 decimal places = ~100m precision
    // Example: 40.7484123 → 40.748
    //          -73.9857456 → -73.986
    
    RETURN round(coordinate * 1000) / 1000
```

**Precision Levels & Privacy:**

```text
DECIMAL PLACES    PRECISION     USE CASE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
6 places          10 cm         Active delivery tracking (real-time)
5 places          1 meter       Recent deliveries (<24 hours)
4 places          10 meters     Recent deliveries (<7 days)
3 places          100 meters    Old deliveries (>30 days) ← GDPR compliant
2 places          1 km          Aggregated analytics only
1 place           10 km         City-level statistics
```

**Why This Balance?**

- **Real-time tracking (6 decimals):** Customers need accurate driver location to meet them at door
- **Analytics (3 decimals):** Platform needs historical data for route optimization, but 100m precision is enough
- **Privacy protection:** Driver's exact home address not stored (only approximate neighborhood)

**GDPR Compliance:**

European regulation requires:
1. **Purpose limitation:** Only collect data needed for legitimate business purpose
2. **Storage limitation:** Don't keep precise data longer than necessary
3. **Right to erasure:** Driver can request complete deletion

**Implementation:**

```text
Data Retention Policy:
├─ 0-24 hours:    Full precision (6 decimals) - real-time tracking
├─ 1-30 days:     Medium precision (4 decimals) - support/dispute resolution
├─ 30+ days:      Low precision (3 decimals) - analytics only
└─ 1+ year:       Delete or further anonymize (city-level only)
```

---

## 9. Dynamic Pricing & Delivery Fees

### What You'll Learn
- Dynamic pricing algorithms (surge pricing)
- Delivery fee calculation factors
- Supply/demand balancing
- Price transparency and communication
- A/B testing pricing strategies

### Why This Matters
Pricing affects marketplace balance: too low → drivers don't accept orders, too high → customers don't order. Uber Eats uses dynamic pricing to balance supply (available drivers) and demand (incoming orders) in real-time, adjusting every 5 minutes based on local conditions.

---

### 🟢 Beginner Level: Basic Delivery Fee Calculation

**Understanding Delivery Fee Components:**

Delivery fees must cover the driver's costs (gas, time, vehicle wear) while remaining affordable for customers. The fee structure balances several factors.

**Fee Calculation Formula:**

```text
DELIVERY FEE COMPONENTS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Base Fee          $2.99   Fixed cost to cover driver acceptance
2. Distance Fee      $0.50/km Variable cost based on travel distance  
3. Small Order Fee   $2.00   Only if order subtotal < $15
4. Maximum Cap       $9.99   Never exceed (to keep competitive)

TOTAL FEE = Base + (Distance × Rate) + Small Order Fee
            BUT NOT EXCEEDING $9.99
```

**Example Calculations:**

```text
EXAMPLE 1: Standard Order
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Order subtotal: $25
Distance: 3 km

Base fee:        $2.99
Distance fee:    3 km × $0.50 = $1.50
Small order fee: $0 (order >$15)
TOTAL:           $4.49 ✓

EXAMPLE 2: Small Order
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Order subtotal: $12
Distance: 3 km

Base fee:        $2.99
Distance fee:    3 km × $0.50 = $1.50
Small order fee: $2.00 (order <$15)
TOTAL:           $6.49 ✓

EXAMPLE 3: Far Distance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Order subtotal: $30
Distance: 15 km

Base fee:        $2.99
Distance fee:    15 km × $0.50 = $7.50
Small order fee: $0
SUB-TOTAL:       $10.49
CAPPED:          $9.99 ✓ (hits maximum)
```

**Why Each Component?**

1. **Base Fee ($2.99):**  Covers minimum driver compensation even for very short deliveries. Without this, a 1km delivery only pays $0.50 to driver - not worth their time.

2. **Distance Fee ($0.50/km):** Compensates driver for longer trips (gas, time). Linear pricing is simple and fair - twice the distance = twice the fee.

3. **Small Order Fee ($2.00):** Discourages very small orders ($5 coffee delivery). Platform still pays driver $5-7 total, so small orders aren't profitable without this surcharge.

4. **Maximum Cap ($9.99):** Keeps competitive with other platforms. If fee goes to $15, customer will use competitor or just pick up food themselves.

**Interview Point:**

"These numbers are market-specific. NYC might have higher base fee ($4.99) due to higher driver costs, while small cities might have lower ($1.99). We'd A/B test different pricing to find optimal balance between order volume and driver satisfaction."

---

### 🟡 Intermediate Level: Dynamic Surge Pricing

**Understanding Supply & Demand:**

When demand (orders) exceeds supply (drivers), delivery times increase and drivers cherry-pick orders. Surge pricing fixes this by:
1. Increasing fees → incentivizes more drivers to go online
2. Decreasing demand → price-sensitive customers wait or cancel
3. Reaching equilibrium faster

**Surge Multiplier Algorithm:**

```text
FUNCTION calculate_surge_multiplier(zone_id, current_time):
    
    // Step 1: Count active demand and supply
    active_orders = database.count(
        WHERE zone = zone_id 
        AND status IN ['CONFIRMED', 'PREPARING', 'READY']
        AND created_at > current_time - 15_minutes
    )
    
    available_drivers = database.count(
        WHERE zone = zone_id
        AND is_online = true
        AND current_order_id IS NULL
    )
    
    // Step 2: Calculate demand/supply ratio
    IF available_drivers == 0:
        demand_supply_ratio = 10.0  // No drivers = maximum surge
    ELSE:
        demand_supply_ratio = active_orders / available_drivers
    
    // Step 3: Map ratio to surge multiplier
    base_multiplier = CASE demand_supply_ratio:
        WHEN < 0.5:  RETURN 1.0   // Excess supply (10 drivers, 3 orders)
        WHEN < 1.0:  RETURN 1.0   // Balanced (10 drivers, 8 orders)
        WHEN < 2.0:  RETURN 1.2   // Moderate demand (10 drivers, 15 orders)
        WHEN < 3.0:  RETURN 1.5   // High demand (10 drivers, 25 orders)
        WHEN < 5.0:  RETURN 2.0   // Very high (10 drivers, 40 orders)
        ELSE:        RETURN 3.0   // Extreme (10 drivers, 60+ orders)
    
    // Step 4: Apply time-of-day modifier
    hour = current_time.hour
    IF (11 <= hour <= 14) OR (17 <= hour <= 21):
        base_multiplier = base_multiplier × 1.1  // +10% during lunch/dinner
    
    // Step 5: Apply weather modifier
    weather = weather_api.get_current(zone_id)
    IF weather.is_raining:
        base_multiplier = base_multiplier × 1.2  // +20% in rain
    IF weather.is_snowing:
        base_multiplier = base_multiplier × 1.5  // +50% in snow
    
    // Step 6: Cap at maximum 3x
    final_multiplier = min(base_multiplier, 3.0)
    
    RETURN final_multiplier

EXAMPLE CALCULATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Zone: Manhattan Midtown
Time: Tuesday 7:00 PM (dinner rush)
Active orders: 150
Available drivers: 40
Weather: Raining

Step 1: Count
  active_orders = 150
  available_drivers = 40

Step 2: Calculate ratio
  ratio = 150 / 40 = 3.75

Step 3: Base multiplier
  ratio 3.75 falls in "WHEN < 5.0" → base_multiplier = 2.0

Step 4: Time modifier
  7 PM is in peak hours (17-21) → 2.0 × 1.1 = 2.2

Step 5: Weather modifier
  Raining → 2.2 × 1.2 = 2.64

Step 6: Cap check
  2.64 < 3.0 → no capping needed

RESULT: 2.64x surge multiplier

Applied to order:
  Normal delivery fee: $5.00
  Surge delivery fee: $5.00 × 2.64 = $13.20
```

**Visual Representation of Surge Levels:**

```text
SURGE ZONES (Manhattan Example, 7 PM Friday)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Zone                    Drivers  Orders  Ratio  Surge   Fee
────────────────────────────────────────────────────────────────
Midtown (Times Square)     15      75    5.0    3.0x   $15.00 🔴
Financial District         25      45    1.8    1.2x    $6.00 🟡
Upper East Side            40      35    0.88   1.0x    $5.00 🟢
Brooklyn Heights           50      30    0.60   1.0x    $5.00 🟢
Queens (Astoria)           30      80    2.67   1.5x    $7.50 🟠

🔴 = Extreme demand  🟠 = High demand  🟡 = Moderate  🟢 = Normal
```

**Why Cap at 3x?**

- **Customer Psychology:** 4x or 5x surge feels like price gouging, damages brand
- **Competitive Pressure:** DoorDash might not surge, customers switch platforms
- **Regulatory Risk:** Some cities banned surge pricing during emergencies
- **Empirical Data:** Uber found 3x brings enough drivers online without killing demand

**Surge Update Frequency:**

```text
Update Every 5 Minutes:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
7:00 PM - Calculate surge for all zones → publish to cache
7:05 PM - Recalculate (demand changed) → update cache
7:10 PM - Recalculate again...

Why 5 minutes?
✓ Frequent enough to respond to demand spikes
✓ Infrequent enough to avoid confusing customers
✗ 1-minute updates: too volatile, customers frustrated
✗ 15-minute updates: too slow, miss demand spikes
```

# Apply surge to delivery fee
base_fee = calculate_delivery_fee(order)  # $6.49
final_fee = base_fee * surge  # $6.49 × 1.8 = $11.68
```

---

## 10. Restaurant Catalog & Menu Management

### What You'll Learn
- Restaurant onboarding and verification process
- Real-time menu synchronization strategies
- Inventory management and sold-out item handling
- Search and discovery optimization
- Menu versioning and change management

### Why This Matters
With 500K restaurants each having 50+ menu items, the catalog contains 25M+ items. Menu data changes frequently (items sold out, prices updated, new items added), requiring real-time synchronization. Uber Eats must balance fresh data (avoid showing sold-out items) with performance (can't query 500K databases every search). Poor menu management leads to failed orders when customers order unavailable items.

---

### 🟢 Beginner Level: Restaurant Catalog Basics

#### What is the Restaurant Catalog?

Think of the restaurant catalog as a giant digital directory, like a phonebook but for restaurants. It contains:

```text
CATALOG CONTENTS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restaurant Profile:
├─ Name, cuisine type, address, phone
├─ Operating hours (Monday-Sunday, breakfast/lunch/dinner)
├─ Delivery radius (5 km default)
├─ Average rating (4.2/5.0)
├─ Photos (storefront, popular dishes)
└─ Tags (Vegan, Gluten-free, Fast Food)

Menu:
├─ Categories (Appetizers, Mains, Desserts, Drinks)
├─ Items (name, description, price, photo)
├─ Availability (in-stock vs sold-out)
├─ Customization options (size, toppings, spice level)
└─ Dietary info (vegetarian, calories, allergens)
```

#### Restaurant Onboarding Process

When a new restaurant joins Uber Eats:

```text
ONBOARDING STEPS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 1: Registration
  → Restaurant provides business info, license, tax ID
  → Platform verifies legitimacy (not a fake/scam restaurant)
  → Approval takes 2-3 business days

Step 2: Menu Creation
  → Restaurant uploads menu (or platform digitizes from PDF)
  → Add photos for each dish
  → Set prices, categories, descriptions
  → Platform quality checks (no offensive names, reasonable prices)

Step 3: Integration
  → Install tablet in restaurant kitchen for receiving orders
  → Configure payment/settlement details
  → Set delivery radius and hours
  → Train staff on using the system

Step 4: Launch
  → Soft launch (limited visibility, test orders)
  → Monitor first 10-20 orders for quality
  → Full launch (restaurant appears in search)
```

#### Menu Update Frequency

Menus aren't static - they change throughout the day:

```text
MENU CHANGES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Morning (9 AM):
  Breakfast menu active → Pancakes, eggs, coffee

Afternoon (2 PM):  
  Switch to lunch menu → Burgers, salads, sandwiches

Evening (5 PM):
  Switch to dinner menu → Steaks, pasta, full bar

Sold Out Events:
  Popular item runs out → Mark as unavailable immediately
  Ingredient shortage → Disable all dishes using that ingredient

Special Events:
  Valentine's Day → Special prix-fixe menu
  Super Bowl Sunday → Party platters and bulk orders
```

---

### 🟡 Intermediate Level: Real-Time Synchronization

#### The Challenge of Stale Data

**Problem:**
Customer sees "Truffle Burger - $15" on their app at 7:00 PM. They order it. Restaurant receives order at 7:01 PM but burger sold out at 6:55 PM. Order must be cancelled. Customer is frustrated.

**Solution: Real-Time Menu Sync**

```text
MENU SYNC ARCHITECTURE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restaurant Tablet                Platform Backend          Customer Apps
      ↓                                 ↓                         ↓
  [UPDATE BUTTON]              [Menu Service]            [Browse Menu]
  "Mark Truffle                      ↓                          ↓
   Burger Sold Out"            Write to PostgreSQL        Read from Redis
      ↓                              ↓                          ↓
  Send to API              Update Redis cache            Get updated menu
   (HTTP POST)           Invalidate CDN cache          See "SOLD OUT" badge
      ↓                              ↓                          ↓
   <200 OK>              Publish to Kafka            WebSocket update
  "Item marked            "menu_updated"              (optional real-time)
   unavailable"          event published

TIME: <2 seconds end-to-end
```

#### Change Data Capture (CDC)

**How It Works:**

```text
CDC PIPELINE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PostgreSQL (Source of Truth)
      ↓ (Database transaction log)
  Debezium CDC Connector
      ↓ (Captures INSERT/UPDATE/DELETE)
  Kafka Topic: "menu-changes"
      ↓ (Event stream)
  ┌─────────────┬─────────────┬─────────────┐
  ↓             ↓             ↓             ↓
Redis       Elasticsearch  Analytics    Notification
Cache       Search Index   Warehouse    Service
(Update)    (Reindex)      (Log)        (Alert team)
```

**Example CDC Event:**

```json
{
  "event_type": "UPDATE",
  "table": "menu_items",
  "timestamp": "2025-11-04T19:05:23Z",
  "before": {
    "item_id": 67890,
    "name": "Truffle Burger",
    "price": 15.00,
    "available": true
  },
  "after": {
    "item_id": 67890,
    "name": "Truffle Burger",
    "price": 15.00,
    "available": false
  },
  "restaurant_id": 12345
}
```

#### Elasticsearch for Menu Search

**Why Elasticsearch?**

PostgreSQL is slow for complex text searches. Finding "vegan gluten-free pasta near Times Square" across 500K restaurants would take 10+ seconds. Elasticsearch returns results in <100ms.

**Search Index Structure:**

```json
{
  "restaurant_id": 12345,
  "name": "Green Garden Cafe",
  "cuisine_types": ["Vegan", "Healthy", "Mediterranean"],
  "location": {
    "lat": 40.7580,
    "lon": -73.9855
  },
  "menu_items": [
    {
      "name": "Quinoa Buddha Bowl",
      "description": "Organic quinoa with roasted vegetables, tahini dressing",
      "tags": ["vegan", "gluten-free", "high-protein"],
      "price": 14.99
    }
  ],
  "rating": 4.7,
  "delivery_time_min": 25
}
```

**Search Query Example:**

```text
USER SEARCHES: "vegan pasta"

Elasticsearch Query:
  Match "vegan" in: cuisine_types, menu_items.tags, menu_items.description
  Match "pasta" in: menu_items.name, menu_items.description
  Filter by: location within 5km, currently open, rating > 3.5
  Sort by: relevance score, then rating, then delivery time

Results in <100ms:
  1. Green Garden Cafe - "Vegan Penne Arrabiata" (4.7★, 25 min)
  2. Plant Power - "Cashew Alfredo Pasta" (4.6★, 30 min)
  3. Veggie House - "Gluten-free Pasta Primavera" (4.5★, 35 min)
```

---

### 🔴 Advanced Level: Menu Versioning & Optimization

#### Menu Versioning Strategy

**Problem:** Restaurant updates prices during peak hours. Some customers saw old price, some saw new price. Who pays which amount?

**Solution: Menu Versioning**

```text
VERSIONED MENU STORAGE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
menu_versions table:
  version_id    restaurant_id  effective_from        items_json
  ──────────────────────────────────────────────────────────────
  v1001        12345          2025-11-01 00:00     {"burger": $12}
  v1002        12345          2025-11-04 18:00     {"burger": $15}  ← Price increase

Order placement logic:
  WHEN customer places order:
    1. Snapshot current menu version → v1002
    2. Store version_id with order
    3. Calculate total using v1002 prices
    4. Even if menu changes later, order uses v1002 prices

BENEFIT: Price consistency - customer pays what they saw
```

#### Dynamic Menu Recommendations

**Personalized Sorting:**

Instead of showing all restaurants in same order to everyone, personalize based on:

```text
RANKING FACTORS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
User History (40%):
  - Frequently orders Italian → boost Italian restaurants
  - Always orders vegetarian → boost veg options
  - Usually orders 7-9 PM → show dinner menus

Location Context (25%):
  - Closer restaurants ranked higher
  - Account for traffic (30 min away in traffic vs 10 min away empty roads)

Time of Day (15%):
  - Breakfast time → boost breakfast places
  - Late night → boost 24-hour restaurants

Popularity (10%):
  - Trending restaurants this week
  - Highly rated by similar users

Promotions (10%):
  - Free delivery this restaurant
  - 20% off first order
```

#### Inventory Prediction

**Prevent Sold-Out Issues:**

```text
ML MODEL: Predict when items will sell out

Features:
  - Current inventory level
  - Historical sales rate (sells 50 burgers/hour on Friday nights)
  - Time of day / day of week
  - Special events (concert nearby → increased demand)

Prediction:
  At 6:00 PM, model predicts:
    "Only 30 burgers left, selling at 50/hour → will sell out by 6:36 PM"

Action:
  6:30 PM → Automatically mark as "Last Few Left!" on app
  6:36 PM → Automatically mark as sold out
  Prevents orders for items that will be gone by preparation time
```

### Real-World Examples

**Uber Eats Menu System:**
- 800K+ restaurants globally
- Menu updates propagate in <5 seconds average
- Elasticsearch powers 95% of searches
- Menu versioning prevents price disputes (99.9% of orders charged correct amount)

**DoorDash Merchant Portal:**
- Real-time inventory sync via POS integration
- Automatic item disabling when prep time exceeds 30 minutes
- Bulk menu updates during off-hours (2-4 AM)

### 🤔 Think About It

- What if restaurant forgets to re-enable sold-out item? (Lost sales)
- How do we handle menu photos? (25M items × 200KB = 5TB of images!)
- Should we allow restaurants to change prices mid-day? (Surge pricing for restaurants?)
- What about menu A/B testing? (Show higher prices to some users?)

### ✅ Key Takeaways

✅ **500K restaurants = 25M menu items** requiring fast search (Elasticsearch <100ms)

✅ **Real-time sync via CDC** (Change Data Capture) ensures menu freshness (<5 sec lag)

✅ **Menu versioning** prevents price disputes (customer pays what they saw)

✅ **Elasticsearch powers search** with complex filters (cuisine, dietary, location, rating)

✅ **ML predictions** reduce sold-out orders (predict inventory depletion)

✅ **Personalized recommendations** boost conversion (40% factor: user history)

---

## 11. Payment Processing & Settlement

### What You'll Learn
- Multi-party payment split logic
- Payment gateway integration
- Idempotency and retry mechanisms
- Fraud detection strategies
- Settlement timing (instant vs T+1 vs T+7)
- Refund processing

### Why This Matters
Food delivery involves complex money flows: customer pays $49, platform keeps $12 commission, restaurant gets $28, driver gets $9. A single failed payment can cascade into multiple failures (driver worked but doesn't get paid, restaurant prepared food but isn't compensated). Uber Eats processes $50M+ daily ($1.5B monthly) requiring 99.99% payment accuracy and PCI DSS Level 1 compliance.

---

### 🟢 Beginner Level: Understanding Payment Flows

#### The Three-Party Settlement

Unlike Amazon (two parties: customer → platform), food delivery has three payees:

```text
PAYMENT BREAKDOWN EXAMPLE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer Order Total: $49.12
├─ Food subtotal:      $35.98
├─ Delivery fee:       $5.00
├─ Service fee:        $2.50 (platform fee)
├─ Tax:                $3.14
└─ Tip:                $5.00

MONEY SPLITS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Platform receives:     $49.12 (from customer)
  ↓
Platform keeps:        $2.50 (service fee = 25% of delivery fee**)
  ↓
Restaurant gets:       $39.12 ($35.98 food + $3.14 tax)
  ↓
Driver gets:           $10.00 ($5.00 delivery fee + $5.00 tip)

**Commission varies by restaurant tier (15-30%)
```

#### When Money Moves

```text
TIMELINE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
7:00 PM - Customer places order
  → Credit card AUTHORIZED (not charged yet)
  → Hold $49.12 on customer's card

7:25 PM - Food delivered
  → Credit card CAPTURED (actually charged)
  → Money debited from customer

7:26 PM - Platform processes settlements
  → Driver paid INSTANTLY (via instant payout to debit card)
  → Restaurant queued for T+1 settlement

Next Day 9:00 AM - Restaurant paid
  → Batch transfer to restaurant bank account
  → $39.12 deposited (minus platform commission already deducted)
```

#### Why Different Timing?

- **Drivers get instant pay:** Keeps drivers happy, reduces churn, competitive advantage
- **Restaurants get T+1:** Lower transaction fees (batch vs individual), fraud review window
- **Platform holds money overnight:** Earns interest (millions of dollars daily), cash flow buffer

---

### 🟡 Intermediate Level: Payment Gateway Integration

#### Payment Service Provider (Stripe/Braintree)

**Why not build our own?**

Building payment processing requires:
- PCI DSS Level 1 certification ($500K+ annual compliance cost)
- Integration with 200+ banks globally
- Fraud detection ML models
- 99.99% uptime SLA
- Support for 100+ currencies

**Better:** Pay Stripe 2.9% + $0.30 per transaction and focus on core business.

**Integration Architecture:**

```text
PAYMENT FLOW:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer App                 Platform Backend           Stripe API
     ↓                              ↓                        ↓
[Checkout Button]         [Order Service]           [Payment Gateway]
     ↓                              ↓                        ↓
Enter card details    →    Tokenize card           Create payment intent
(4111-1111-1111-1111)     (send to Stripe)         (return client_secret)
     ↓                              ↓                        ↓
Submit payment        →    Call Stripe API    →     Process payment
     ↓                    (confirm payment)          (charge card)
     ↓                              ↓                        ↓
Loading...                  Wait for webhook    ←    Webhook: "succeeded"
     ↓                              ↓                        
Success message!       →    Update order status
                           (PAYMENT_COMPLETE)
```

**Key Integration Points:**

```json
// Payment Intent Creation
POST /api/orders/{order_id}/payment
{
  "payment_method": "card",
  "amount": 4912,  // cents
  "currency": "USD",
  "customer_id": "cus_abc123",
  "metadata": {
    "order_id": "order_xyz789",
    "restaurant_id": 12345
  }
}

// Stripe Response
{
  "id": "pi_abc123",
  "status": "requires_confirmation",
  "client_secret": "pi_abc123_secret_xyz",
  "amount": 4912
}

// Stripe Webhook (async notification)
{
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_abc123",
      "status": "succeeded",
      "amount_received": 4912
    }
  }
}
```

#### Idempotency Keys

**Problem:** Network timeout → customer retries → double charge!

**Solution: Idempotency**

```text
IDEMPOTENCY MECHANISM:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Request 1 (7:00:00 PM):
  POST /api/payments
  Idempotency-Key: order_xyz789_payment_v1
  → Payment processed, stored in cache

Request 2 (7:00:05 PM) - Customer clicks "Pay" again:
  POST /api/payments  
  Idempotency-Key: order_xyz789_payment_v1
  → Key found in cache → Return cached result
  → No duplicate charge!

Pseudocode:
  FUNCTION process_payment(order_id, amount, idempotency_key):
    
    // Check cache first
    cached_result = redis.get("idem:" + idempotency_key)
    IF cached_result EXISTS:
      RETURN cached_result  // Duplicate request
    
    // Process new payment
    payment_result = stripe.charge(amount)
    
    // Cache result for 24 hours
    redis.setex("idem:" + idempotency_key, 86400, payment_result)
    
    RETURN payment_result
```

---

### 🔴 Advanced Level: Fraud Detection & Settlement

#### Multi-Layered Fraud Detection

**Layer 1: Pre-Authorization Checks (Blocking)**

```text
INSTANT REJECTION CRITERIA:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Stolen card (on global blacklist)
2. Velocity limit exceeded (>5 orders in 1 hour from same card)
3. BIN mismatch (card type doesn't match expected)
4. Shipping address in high-fraud country
5. Device fingerprint matches known fraudster

ACTION: Immediately decline, prompt different payment method
```

**Layer 2: ML Risk Scoring (Review Queue)**

```text
FRAUD SCORE CALCULATION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Features (30+ signals):
  ├─ Order value ($300 order from new user → suspicious)
  ├─ Delivery address changes frequently
  ├─ Using VPN/proxy
  ├─ Multiple payment methods tried
  ├─ Device fingerprint (new device vs known device)
  ├─ Time of order (3 AM orders higher fraud rate)
  └─ Historical behavior (100 successful orders → trustworthy)

ML Model Output:
  Score 0.0-0.3: Low risk → Auto-approve
  Score 0.3-0.7: Medium risk → Manual review
  Score 0.7-1.0: High risk → Decline or require verification

ACTION:
  IF score > 0.7:
    Require additional verification (CVV, 3D Secure, phone call)
  ELSE IF score > 0.3:
    Flag for post-delivery review
  ELSE:
    Proceed normally
```

**Layer 3: Post-Delivery Analysis (Chargeback Prevention)**

```text
CHARGEBACK PATTERNS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer claims: "I never received food"

Platform evidence:
  ✓ GPS shows driver at delivery address for 2 minutes
  ✓ Photo uploaded of food at door
  ✓ Customer marked as "delivered" in app
  ✓ No customer support ticket within 30 minutes

DECISION: Reject chargeback, provide evidence to bank
SUCCESS RATE: 85% of chargebacks won
```

#### Settlement Batching & Optimization

**Why Batch Payments?**

Individual bank transfers cost $0.25-0.50 each. Processing 10M orders = $2.5M-5M daily just in transfer fees!

**Batching Strategy:**

```text
DAILY SETTLEMENT BATCHES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Restaurant payouts (T+1):
  Aggregate all orders per restaurant per day
  
  Example - Restaurant #12345:
    Order 1: $28.50
    Order 2: $31.20  
    Order 3: $45.00
    Total: $104.70
  
  ONE bank transfer: $104.70
  Fee: $0.30 (vs $0.90 for 3 separate transfers)
  
  Batch transfer at 9 AM daily
  ACH transfer (cheap but slow)

Driver payouts (Instant):
  Individual transfers immediately after delivery
  Use instant transfer API (higher fee $1.00 vs $0.30)
  Driver satisfaction > cost savings
  
COST ANALYSIS:
  10M orders/day → 500K restaurant payouts (20 orders/restaurant avg)
  Batched: 500K × $0.30 = $150K/day
  Individual: 10M × $0.30 = $3M/day
  SAVINGS: $2.85M/day = $1.04B/year
```

#### Refund Processing

```text
REFUND SCENARIOS & TIMING:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer cancels before restaurant accepts (5 min window):
  → Full refund (100%)
  → Instant (within 1 minute)

Restaurant cancels (out of ingredients):
  → Full refund (100%)
  → Platform compensates customer with $5 credit
  → Refund within 5 minutes

Driver issue (couldn't deliver):
  → Full refund + delivery fee
  → Restaurant keeps 50% (food was prepared)
  → Driver penalized

Quality issue (cold food, wrong order):
  → Partial refund (20-50% judgment call)
  → Customer support review
  → Restaurant may be charged (if at fault)

REFUND TIMING:
  Credit card: 5-7 business days (bank processing)
  Platform credit: Instant (just database update)
  
TIP: Offer platform credit first (faster + cheaper for platform)
```

### Real-World Examples

**Uber Eats Payment Scale:**
- $1.5B monthly transaction volume
- 99.97% payment success rate
- Average fraud rate: 0.08% (industry average 1.5%)
- Chargeback win rate: 82%
- Settlement cost: 0.15% of GMV (batching savings)

**DoorDash Instant Pay:**
- Drivers can cash out daily earnings instantly
- $1.99 fee for instant transfer (vs free weekly)
- 60% of drivers use instant pay at least once/month
- Reduces driver churn by 15%

### 🤔 Think About It

- What if driver delivers but customer claims "never received"? (Photo proof, GPS tracking)
- How do you prevent restaurants from inflating prices on platform vs in-store? (Price audits)
- Should platform eat the cost of payment failures? (No - require backup payment method)
- What about cryptocurrency payments? (High volatility, regulatory uncertainty)

### ✅ Key Takeaways

✅ **Three-party settlement:** Customer → Platform → Restaurant + Driver (complex splits)

✅ **Instant driver pay** vs **T+1 restaurant pay** (driver satisfaction vs cost optimization)

✅ **Idempotency keys** prevent duplicate charges (24-hour cache)

✅ **Multi-layer fraud detection:** Pre-auth checks + ML scoring + post-delivery analysis

✅ **Settlement batching** saves $1B+ annually (aggregate per restaurant per day)

✅ **99.99% payment accuracy** required (10M orders × 0.01% failure = 1,000 angry customers daily)

---

## 12. Scalability & Performance

### What You'll Learn
- Horizontal vs vertical scaling strategies
- Database sharding and partitioning
- Caching layers and invalidation
- Load balancing techniques
- Auto-scaling policies
- Performance optimization tactics

### Why This Matters
Uber Eats must scale from 50 orders/sec at 3 AM to 1,150 orders/sec during dinner rush (23x spike). Poor scaling means slow page loads (customers leave), failed orders (revenue loss), and crashed servers (complete outage). With proper scaling, infrastructure costs stay at 0.02% of revenue while handling 10M daily orders.

---

### 🟢 Beginner Level: Understanding Scalability

#### What Does "Scale" Mean?

Imagine a restaurant with 10 tables. During lunch rush, 50 customers arrive. Options:

```text
RESTAURANT ANALOGY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Vertical Scaling (Scale Up):
  Buy a bigger restaurant (10 tables → 100 tables)
  ✓ Simple (same restaurant, just bigger)
  ✗ Expensive (commercial real estate costs)
  ✗ Limited (can't have 1,000 table restaurant)
  
Horizontal Scaling (Scale Out):
  Open 5 smaller restaurants (10 tables each = 50 total)
  ✓ Flexible (add more as needed)
  ✓ Cost-effective (smaller spaces cheaper)
  ✗ Complex (coordinate across locations)
```

**In Software Terms:**

```text
VERTICAL SCALING:
  1 server: 8 CPU cores, 32 GB RAM
     ↓
  1 bigger server: 64 CPU cores, 512 GB RAM
  
  Cost: $500/month → $8,000/month
  Limit: Can't buy infinitely large servers

HORIZONTAL SCALING:
  1 server: 8 CPU cores, 32 GB RAM
     ↓
  10 servers: 8 CPU cores, 32 GB RAM each
  
  Cost: $500/month → $5,000/month
  Limit: Can add thousands of servers
```

#### Traffic Patterns

```text
DAILY TRAFFIC CURVE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Orders/sec
   1200 |                    ╱╲
   1000 |                  ╱    ╲
    800 |                ╱        ╲
    600 |         ╱╲   ╱            ╲
    400 |       ╱    ╲╱              ╲╱╲
    200 |     ╱                            ╲
     50 |____╱________________________________╲____
        6AM  9AM  12PM  3PM  6PM  9PM  12AM  3AM

        Breakfast  Lunch      Dinner      Late
        ────────────────────────────────────────
Peak:   9 AM       12-1 PM    6-8 PM      N/A
QPS:    400        800        1,150       100
```

**Scaling Strategy:**

- **3 AM (50 QPS):** Run 5 servers (10 QPS each)
- **12 PM (800 QPS):** Auto-scale to 80 servers (10 QPS each)
- **7 PM (1,150 QPS):** Auto-scale to 115 servers (10 QPS each)
- **11 PM (200 QPS):** Scale down to 20 servers

**Cost Savings:** Pay for what you use (not peak capacity 24/7)

---

### 🟡 Intermediate Level: Database Sharding

#### Why Shard?

**Problem:** Single PostgreSQL database hits limits at ~10,000 QPS

```text
BOTTLENECK:
  10M orders/day = 115 avg QPS
  Peak 10x = 1,150 QPS
  Each order = 5 DB queries (read menu, check inventory, write order, update driver, log history)
  Total: 1,150 × 5 = 5,750 QPS
  
  Single DB limit: ~10,000 QPS
  We're okay now, but:
    - 2x growth = 11,500 QPS (exceeds limit)
    - Need solution before hitting wall
```

**Solution: Sharding (Split Data Across Multiple DBs)**

```text
SHARDING STRATEGY - BY CITY:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Shard 1 (NYC):
  ├─ Orders: order_id, customer_id, restaurant_id
  ├─ Restaurants: 50,000 in NYC
  └─ Drivers: 100,000 in NYC
  Load: 30% of total (3M orders/day)

Shard 2 (LA):
  ├─ Orders, Restaurants, Drivers for LA
  └─ Load: 15% (1.5M orders/day)

Shard 3 (Chicago):
  └─ Load: 8% (800K orders/day)

...100 city shards total

ROUTING:
  Order comes in → Extract city_id from address
  city_id="NYC" → Route to Shard 1
  city_id="LA"  → Route to Shard 2
```

**Shard Mapping:**

```text
SHARD LOOKUP TABLE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
city_id    shard_id    db_host
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NYC        shard_01    db-nyc-01.internal
LA         shard_02    db-la-01.internal
CHI        shard_03    db-chi-01.internal

Pseudocode:
  FUNCTION get_db_connection(order):
    city_id = geocode(order.delivery_address)
    shard = SHARD_MAP[city_id]
    connection = connect_to_db(shard.db_host)
    RETURN connection
```

#### Multi-Level Caching

**Cache Hierarchy:**

```text
LAYERED CACHE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Layer 1: Application Cache (In-Memory)
  ├─ Location: Server RAM
  ├─ Size: 100 MB per server
  ├─ Contents: Hot data (current user session, active orders)
  ├─ Hit Rate: 40%
  └─ Latency: <1 ms

Layer 2: Redis (Shared Cache)
  ├─ Location: Separate Redis cluster
  ├─ Size: 100 GB total
  ├─ Contents: Menu data, driver locations, session tokens
  ├─ Hit Rate: 85% (of misses from L1)
  └─ Latency: <5 ms

Layer 3: Database (Source of Truth)
  ├─ Location: PostgreSQL shards
  ├─ Size: 80 TB
  ├─ Contents: All data (orders, users, transactions)
  ├─ Hit Rate: 15% (only cache misses hit DB)
  └─ Latency: 50-100 ms

TOTAL CACHE HIT RATE:
  L1: 40%
  L2: 85% × 60% = 51%
  Combined: 91% requests served from cache
  Only 9% hit database
```

**Example Flow:**

```text
USER REQUEST: Load restaurant menu for "Joe's Pizza"

Step 1: Check L1 cache (server RAM)
  key = "menu:restaurant:12345"
  IF found: RETURN immediately (1 ms)

Step 2: Check L2 cache (Redis)
  IF found: 
    Store in L1 cache
    RETURN (5 ms total)

Step 3: Query database
  IF found:
    Store in L1 and L2 cache
    RETURN (100 ms total)
  
99% of requests: <5 ms (cache hit)
1% of requests: ~100 ms (cache miss)
Average: 5.95 ms
```

---

### 🔴 Advanced Level: Auto-Scaling & Optimization

#### Auto-Scaling Policies

```text
SCALING TRIGGERS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Metric-Based Scaling:
  IF avg_cpu_usage > 70% for 5 minutes:
    Add 20% more servers (round up)
    Example: 50 servers → 60 servers
  
  IF avg_cpu_usage < 30% for 15 minutes:
    Remove 20% of servers (round down)
    Example: 50 servers → 40 servers
  
  Min servers: 10 (always available)
  Max servers: 500 (cost limit)

Time-Based Scaling (Predictive):
  5:00 PM → Scale up 30% (dinner rush incoming)
  6:00 PM → Scale up another 20%
  9:00 PM → Scale down 20%
  11:00 PM → Scale down to baseline

Event-Based Scaling:
  Super Bowl Sunday → Pre-scale 2x capacity
  Valentine's Day → Pre-scale 3x capacity
  City-wide power outage → Scale down affected region
```

**Cool-Down Periods:**

```text
WHY NEEDED: Prevent flapping

BAD (No cool-down):
  6:00 PM - CPU 71% → Add 10 servers
  6:02 PM - CPU 65% (new servers helped) → Remove 10 servers
  6:04 PM - CPU 72% (removed too early) → Add 10 servers
  6:06 PM - Repeat...

GOOD (With 5-min cool-down):
  6:00 PM - CPU 71% → Add 10 servers
  6:02 PM - CPU 65% → Wait (cool-down active)
  6:05 PM - Cool-down ended, CPU still 65% → Keep servers
  6:20 PM - CPU 30% for 15 min → Remove servers
```

#### Performance Optimization Techniques

**Database Query Optimization:**

```text
SLOW QUERY (Before):
  SELECT * FROM orders 
  WHERE customer_id = 12345 
  ORDER BY created_at DESC;
  
  Time: 2,500 ms (no index on customer_id)
  Rows scanned: 10M (full table scan)

FAST QUERY (After):
  CREATE INDEX idx_customer_orders ON orders(customer_id, created_at DESC);
  
  SELECT order_id, status, total, created_at 
  FROM orders 
  WHERE customer_id = 12345 
  ORDER BY created_at DESC 
  LIMIT 10;
  
  Time: 15 ms (index used, only fetch needed columns)
  Rows scanned: 10 (index seek)
  
IMPROVEMENT: 166x faster
```

**API Response Compression:**

```text
UNCOMPRESSED JSON RESPONSE:
  {
    "restaurants": [ /* 50 restaurants */ ],
    "menus": [ /* 2,500 menu items */ ]
  }
  
  Size: 1.2 MB
  Transfer time (4G LTE): 1.2 MB ÷ 10 Mbps = 960 ms

COMPRESSED (gzip):
  Same JSON, gzipped
  
  Size: 180 KB (85% reduction)
  Transfer time: 180 KB ÷ 10 Mbps = 144 ms
  
IMPROVEMENT: 6.7x faster, saves bandwidth
```

### Real-World Examples

**Uber Eats Infrastructure:**
- 5,000+ application servers globally
- Auto-scales 10x between off-peak and peak hours
- Database: 100 PostgreSQL shards across 10 regions
- Cache hit rate: 94% (Redis + CDN)
- Average API response time: 85 ms (p50), 250 ms (p99)

**DoorDash Scaling:**
- Started with monolith (2013)
- Migrated to microservices (2016-2018)
- Current: 200+ microservices
- Handles 5x traffic spikes during Super Bowl

### 🤔 Think About It

- What if all 115 servers crash simultaneously? (Multi-region redundancy)
- Should we cache user location? (Privacy concern - cache only city-level)
- How do we test auto-scaling before production? (Chaos engineering, load testing)
- What's the cost of running at peak capacity 24/7? (5x current cost)

### ✅ Key Takeaways

✅ **Horizontal scaling** preferred over vertical (add servers vs bigger servers)

✅ **Auto-scaling** saves 60% infrastructure cost (pay for what you use)

✅ **Database sharding by city** distributes load across 100 shards

✅ **Multi-level caching** (L1 + L2) achieves 91%+ hit rate

✅ **Performance optimization** = Indexing + Compression + Query tuning

✅ **23x traffic spike** (50 QPS → 1,150 QPS) handled transparently

---

## 13. Security & Fraud Prevention

### What You'll Learn
- Authentication and authorization mechanisms
- Data encryption (in-transit and at-rest)
- PCI DSS compliance requirements
- Fraud detection patterns and ML models
- DDoS protection strategies
- Privacy compliance (GDPR, CCPA)

### Why This Matters
Food delivery platforms are prime targets for fraud: stolen credit cards ($5M annual losses), promo code abuse ($2M), fake driver accounts, GPS spoofing, and data breaches. A single security breach can cost $50M+ in fines, lawsuits, and lost trust. Uber Eats must balance security (protect user data) with usability (don't make login too difficult) while processing $1.5B monthly transactions under PCI DSS Level 1 compliance.

---

### 🟢 Beginner Level: Authentication & Authorization

#### Understanding the Difference

```text
AUTHENTICATION: Who are you?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
"Prove you're John Smith"
Methods:
  ├─ Password (something you know)
  ├─ SMS code (something you have - phone)
  ├─ Fingerprint (something you are - biometric)
  └─ Social login (Google/Facebook OAuth)

AUTHORIZATION: What can you do?
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
"You're John Smith, but are you allowed to cancel THIS order?"
Rules:
  ├─ Customer can cancel THEIR OWN orders
  ├─ Driver can view ASSIGNED orders only
  ├─ Restaurant can update THEIR OWN menu
  └─ Admin can view ALL data
```

#### JWT (JSON Web Token) Authentication

**How It Works:**

```text
LOGIN FLOW:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Step 1: Customer logs in
  POST /api/auth/login
  {
    "email": "john@example.com",
    "password": "SecurePass123!"
  }

Step 2: Server validates credentials
  IF password matches database:
    Generate JWT token
    
Step 3: Return JWT to customer
  {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600  // 1 hour
  }

Step 4: Customer stores token (local storage/cookie)

Step 5: All future requests include token
  GET /api/orders
  Header: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Step 6: Server validates token
  IF token valid AND not expired:
    Process request
  ELSE:
    Return 401 Unauthorized
```

**JWT Token Structure:**

```json
// Decoded JWT
{
  "header": {
    "alg": "HS256",  // Encryption algorithm
    "typ": "JWT"
  },
  "payload": {
    "user_id": 12345,
    "email": "john@example.com",
    "role": "customer",
    "exp": 1730668800  // Expiration timestamp
  },
  "signature": "encrypted_hash_here"
}
```

**Why JWT?**
- **Stateless:** Server doesn't need to store sessions (scales better)
- **Self-contained:** Token has all info (user_id, role) - no database lookup needed
- **Secure:** Cryptographically signed (can't be tampered with)

#### Role-Based Access Control (RBAC)

```text
USER ROLES & PERMISSIONS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CUSTOMER:
  ✓ Place orders
  ✓ View own order history
  ✓ Update own profile
  ✗ View other customers' orders
  ✗ Modify menu prices

DRIVER:
  ✓ Accept/reject order assignments
  ✓ Update location
  ✓ Mark orders as delivered
  ✗ See customer payment info
  ✗ Cancel completed orders

RESTAURANT:
  ✓ Update own menu
  ✓ Accept/reject orders
  ✓ Mark items as sold out
  ✗ See driver earnings
  ✗ Access other restaurants' data

ADMIN:
  ✓ View all data (for support)
  ✓ Issue refunds
  ✓ Suspend accounts
  ✓ View analytics dashboards
```

**Authorization Check Logic:**

```text
FUNCTION authorize_action(user, action, resource):
  
  // Extract user role from JWT
  role = user.role  // "customer", "driver", "restaurant", "admin"
  
  CASE action:
    
    WHEN "view_order":
      IF role == "customer":
        RETURN resource.customer_id == user.id
      ELSE IF role == "driver":
        RETURN resource.driver_id == user.id
      ELSE IF role == "restaurant":
        RETURN resource.restaurant_id == user.id
      ELSE IF role == "admin":
        RETURN true
      ELSE:
        RETURN false
    
    WHEN "cancel_order":
      IF role == "customer" AND resource.status == "PLACED":
        RETURN resource.customer_id == user.id
      ELSE IF role == "admin":
        RETURN true
      ELSE:
        RETURN false
    
    // More cases...
```

---

### 🟡 Intermediate Level: Data Encryption & Compliance

#### Encryption at Rest

**What Gets Encrypted:**

```text
SENSITIVE DATA REQUIRING ENCRYPTION:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✓ Credit card numbers (PCI DSS requirement)
✓ Social security numbers (for driver background checks)
✓ Bank account numbers (for settlements)
✓ Passwords (hashed, not encrypted - different)
✓ Driver's license photos

✗ Order history (not sensitive enough)
✗ Restaurant names (public data)
✗ Delivery addresses (needed for queries)
```

**Encryption Method (AES-256):**

```text
ENCRYPTION PROCESS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Plain text: "4111-1111-1111-1111" (credit card)
     ↓
Encryption key: 256-bit random key (stored in AWS KMS)
     ↓
AES-256 algorithm
     ↓
Cipher text: "U2FsdGVkX1+vupppZksvRf5pq5g5XjFRIipRkwB0K1Y="
     ↓
Store in database: Only encrypted version saved

DECRYPTION (when needed):
Retrieve cipher text from database
     ↓
Fetch encryption key from AWS KMS (requires permission)
     ↓
Decrypt with AES-256
     ↓
Plain text: "4111-1111-1111-1111"
     ↓
Process payment
     ↓
Discard plain text from memory
```

#### Encryption in Transit (TLS/SSL)

**All API calls use HTTPS:**

```text
WITHOUT HTTPS (HTTP):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer App  →  Internet  →  Server
    ↓                             
Sends: {"email": "john@example.com", "password": "SecurePass123!"}
    ↓
ANYONE on same WiFi can read this! (Man-in-the-middle attack)

WITH HTTPS (TLS 1.3):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer App  →  Internet  →  Server
    ↓                             
Sends: "8f32j4k23h4k2j3h4k23j4h23k4j" (encrypted gibberish)
    ↓
Only server can decrypt (has private key)
```

#### PCI DSS Compliance

**Payment Card Industry Data Security Standard:**

```text
12 REQUIREMENTS (Simplified):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Install firewalls
2. Don't use default passwords
3. Encrypt stored card data (AES-256)
4. Encrypt transmitted card data (TLS 1.2+)
5. Use anti-virus software
6. Develop secure systems
7. Restrict data access (need-to-know basis)
8. Assign unique ID to each person with access
9. Restrict physical access to card data
10. Track all access to card data
11. Test security systems regularly
12. Maintain security policy

AUDIT: Annual audit by Qualified Security Assessor (QSA)
COST: $50K-$500K annually
PENALTY: $5K-$100K per month for non-compliance
```

**Tokenization (Reduce PCI Scope):**

```text
INSTEAD OF STORING CARDS:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Customer enters: 4111-1111-1111-1111
      ↓
Send to Stripe API (they store it)
      ↓
Stripe returns: "tok_1234abcd"
      ↓
WE store: "tok_1234abcd" (not the actual card!)
      ↓
For future charges: Send token to Stripe
      ↓
Stripe charges the real card

BENEFIT: 
✓ We never store real card numbers
✓ Reduces PCI DSS compliance scope
✓ If our database is hacked, tokens are useless
```

---

### 🔴 Advanced Level: Fraud Detection & DDoS Protection

#### Machine Learning Fraud Detection

**Training Data (Historical Labels):**

```text
LABELED FRAUD EXAMPLES:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Order ID   Features                              Fraud?
──────────────────────────────────────────────────────────────
001        New user, $300 order, VPN, 3 AM       YES
002        Regular user, $25 order, normal       NO
003        10 orders/hour, same card             YES
004        First order, $15, verified phone      NO
```

**ML Model (Random Forest):**

```text
INPUT FEATURES (40+):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
User History:
  ├─ Account age (new account = higher risk)
  ├─ Previous order count
  ├─ Previous fraud/dispute rate
  └─ Email domain (gmail.com vs random-site.xyz)

Order Characteristics:
  ├─ Order value ($500 = suspicious for first order)
  ├─ Number of items (100 items = suspicious)
  ├─ Delivery address (hotel/airport = higher risk)
  └─ Time of order (3 AM = higher risk)

Payment Info:
  ├─ Card BIN (first 6 digits reveal bank)
  ├─ Card country vs delivery country mismatch
  ├─ Multiple payment methods tried
  └─ Payment method age

Device/Network:
  ├─ Device fingerprint (known device vs new)
  ├─ IP address (VPN/proxy detection)
  ├─ User agent (mobile app vs desktop browser)
  └─ GPS location vs delivery address mismatch

OUTPUT:
Fraud probability: 0.0-1.0

ACTIONS:
  IF prob > 0.8: Auto-decline + block card
  IF prob > 0.5: Require 3D Secure verification
  IF prob > 0.3: Flag for manual review
  IF prob < 0.3: Auto-approve
```

**Model Performance:**

```text
CONFUSION MATRIX:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
                  Predicted Fraud    Predicted Legit
Actual Fraud           980              20 (False Negative)
Actual Legit            80            9,920 (True Negative)

Accuracy: 99.1%
Precision: 92.5% (of flagged orders, 92.5% were actually fraud)
Recall: 98.0% (caught 98% of actual fraud)
False Positive Rate: 0.8% (80 legit orders declined - bad UX)

COST-BENEFIT:
  Caught fraud: 980 orders × $45 avg = $44,100 saved
  Lost legit orders: 80 × $30 avg = $2,400 lost
  NET BENEFIT: $41,700/day = $15M/year
```

#### DDoS Protection

**Attack Scenario:**

```text
DISTRIBUTED DENIAL OF SERVICE:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Normal traffic: 1,000 requests/second
     ↓
Attacker botnet: 100,000 requests/second
     ↓
Server overwhelmed → Crashes → Legitimate users can't access site
     ↓
Revenue loss: $50K/hour during dinner rush
```

**Defense Layers:**

```text
LAYER 1: CloudFlare/Akamai (Edge Protection)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Filters:
  ├─ Block known bad IPs (botnet databases)
  ├─ Rate limiting (max 10 req/sec per IP)
  ├─ Challenge suspicious traffic (CAPTCHA)
  └─ GeoIP filtering (block countries not in service area)

Result: 99% of DDoS traffic blocked before reaching servers

LAYER 2: API Gateway (Application-Level)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Throttling:
  ├─ 100 req/min per user (authenticated)
  ├─ 10 req/min per IP (unauthenticated)
  ├─ 1,000 req/min per API key (restaurants/drivers)
  └─ Exponential backoff for violators

LAYER 3: Auto-Scaling
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
IF traffic spike detected:
  Add 100 servers within 2 minutes
  Absorb attack while filtering continues
  Cost: $500/hour (cheaper than downtime)
```

### Real-World Examples

**Uber Breach (2016):**
- 57M user records stolen (names, emails, phone numbers)
- Hackers accessed AWS S3 bucket with hardcoded credentials
- Uber paid hackers $100K ransom to delete data (later fined $148M)
- Lesson: Never hardcode credentials, use AWS IAM roles

**DoorDash Promo Code Abuse (2019):**
- Users discovered glitch: unlimited $10 promo codes
- Lost $2M in 48 hours before fix
- Solution: Rate limit promo code usage (1 per user per day)

### 🤔 Think About It

- Should we store delivery addresses encrypted? (Impacts search performance)
- What if ML model flags a VIP customer as fraud? (Manual override process)
- How do we balance security vs convenience? (2FA annoys users but prevents account takeover)
- What about insider threats? (Employee stealing customer data)

### ✅ Key Takeaways

✅ **JWT authentication** for stateless, scalable auth (1-hour expiration)

✅ **RBAC** ensures users only access their own data (customer/driver/restaurant/admin roles)

✅ **AES-256 encryption** for sensitive data at rest, **TLS 1.3** for data in transit

✅ **PCI DSS compliance** via tokenization (store tokens, not real cards)

✅ **ML fraud detection** catches 98% of fraud with 0.8% false positive rate ($15M saved annually)

✅ **Multi-layer DDoS protection** blocks 99%+ of attack traffic (CloudFlare + API throttling + auto-scaling)

---

## 14. Monitoring & Observability

**Key Metrics:**
- Order success rate: 98%+
- Average delivery time: 35 minutes
- Driver utilization: 70%
- Customer satisfaction (NPS): 60+

**Alerting:**
- PagerDuty for critical alerts
- Slack for warnings
- Grafana dashboards

---

## 15. Design Trade-Offs & Decisions

**Consistency vs Availability:**
- Orders: Strong consistency (CP)
- Locations: Eventual consistency (AP)

**SQL vs NoSQL:**
- PostgreSQL for transactions
- Cassandra for time-series
- Redis for caching

**Microservices vs Monolith:**
- Chose microservices for independent scaling
- Trade-off: Increased complexity

---

## 16. Interview Preparation Guide

**Key Questions to Expect:**

1. "How do you handle a driver going offline mid-delivery?"
2. "How do you prevent restaurants from being overwhelmed with orders?"
3. "How do you calculate surge pricing fairly?"
4. "How do you handle payment failures during delivery?"

**Framework for Answering:**

1. Clarify requirements
2. Estimate scale
3. High-level design
4. Deep-dive critical components
5. Discuss trade-offs

---

## Putting It All Together

**System Summary:**

Uber Eats is a complex three-sided marketplace coordinating 10M daily orders across 500K restaurants and 1M drivers. Key architectural decisions:

✅ **Event-driven microservices** for decoupling and scalability
✅ **Multi-database strategy** optimized for different data types
✅ **Real-time location tracking** with WebSocket and Cassandra
✅ **Intelligent driver matching** using geospatial algorithms and ML
✅ **Dynamic pricing** balancing supply and demand
✅ **State machine** ensuring correct order lifecycle
✅ **Multi-region deployment** for low latency globally

**Cost:** $4.9M/year infrastructure (0.02% of revenue)  
**Scale:** 115 orders/second average, 1,150/second peak  
**Uptime:** 99.9% (43 minutes downtime/month)

---

## Next Steps & Resources

**Practice Questions:**
- Design Doordash
- Design Grubhub
- Design restaurant reservation system

**Read More:**
- Uber Engineering Blog
- "Designing Data-Intensive Applications" by Martin Kleppmann

---

**Congratulations!** You've completed the Food Delivery System Design course!

