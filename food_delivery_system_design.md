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

### 🎯 Interview Questions - Requirements & Planning

#### Beginner Level

**Q1:** What are the core functional requirements for a food delivery system like Uber Eats?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Start with the three main user types and their needs:

**Customer Requirements:**
- Browse restaurants by location (within 5 km delivery radius)
- Search menu items (by name, cuisine, dietary preferences)
- Place orders (select items, customize, add to cart, checkout)
- Track order status in real-time (confirmed → preparing → picked up → delivered)
- Rate and review orders (restaurant quality, delivery speed)

**Restaurant Requirements:**
- Receive incoming orders via tablet/app
- Accept or reject orders (based on capacity, inventory)
- Update menu items (add/remove items, mark as sold out)
- Manage operating hours (breakfast/lunch/dinner schedules)
- View earnings and analytics

**Driver Requirements:**
- Go online/offline (mark availability)
- Receive order assignment notifications
- Accept or reject deliveries
- Navigate to pickup and delivery locations
- Mark order stages (picked up, delivered)
- Track earnings

**Platform Requirements:**
- Match available drivers to orders (<30 sec)
- Calculate delivery fees dynamically
- Process payments (customer → platform → restaurant + driver)
- Handle disputes and refunds
- Monitor system health

**Interview Tip:** Always mention all three user types - showing you understand the marketplace complexity.

</details>

**Q2:** How would you distinguish functional vs non-functional requirements for this system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Functional Requirements (WHAT the system does):**
- Customer can place an order
- Driver can accept/reject assignments
- Restaurant can update menu availability
- System calculates delivery fee based on distance
- Platform processes three-way payment splits

**Non-Functional Requirements (HOW WELL the system performs):**

**Performance:**
- Order placement: <200ms response time
- Driver matching: <30 seconds
- Location updates: 1-second intervals
- Menu search: <500ms

**Scalability:**
- Handle 10M orders/day
- Support 500K concurrent users
- Scale 10x during peak hours

**Availability:**
- 99.9% uptime (43 min downtime/month)
- Multi-region deployment for disaster recovery

**Consistency:**
- Strong consistency for orders and payments (no double-charging)
- Eventual consistency for driver locations (1-sec lag acceptable)

**Security:**
- PCI DSS Level 1 compliance for payments
- Encrypted data at rest (AES-256)
- Encrypted data in transit (TLS 1.3)

**Interview Tip:** Use the "-ilities" framework: scalability, availability, reliability, maintainability, security.

</details>

**Q3:** What clarifying questions would you ask the interviewer about scale?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Geographic Scale:**
- "How many cities/countries do we serve?" (1 city vs global)
- "What's our geographic distribution?" (concentrated vs spread out)
- "Do we need multi-region deployment?" (latency requirements)

**User Scale:**
- "How many daily active users?" (defines infrastructure size)
- "How many restaurants on the platform?" (catalog size)
- "How many active drivers?" (matching pool size)
- "What's the ratio of users to restaurants to drivers?" (10,000:1:100?)

**Order Scale:**
- "How many orders per day?" (throughput requirements)
- "What's the average order value?" (payment processing volume)
- "Peak vs average traffic?" (capacity planning)
- "Order distribution (breakfast/lunch/dinner)?" (auto-scaling needs)

**Growth Scale:**
- "What's the expected growth rate?" (6 months, 1 year, 5 years)
- "Are we starting from scratch or have existing users?" (migration strategy)

**Sample Interview Exchange:**
```
You: "How many orders per day should the system handle?"
Interviewer: "Let's assume 10 million daily orders"
You: "And what's the peak to average ratio during dinner rush?"
Interviewer: "Good question - assume 10x spike, so 1,150 orders/second at peak"
You: "Got it. I'll design for that peak load with auto-scaling..."
```

</details>

#### Intermediate Level

**Q4:** How would you handle the chicken-and-egg problem of launching in a new city?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

The problem: Need restaurants to attract customers, need customers to attract restaurants, need both to attract drivers.

**Phased Launch Strategy:**

**Phase 1: Restaurant Partnerships (Month 1)**
- Partner with 20-30 popular restaurants
- Offer zero commission for first 3 months
- Provide free tablet hardware and setup
- Focus on diverse cuisines (Italian, Chinese, Mexican, etc.)

**Phase 2: Driver Recruitment (Month 2)**
- Guarantee minimum earnings ($15/hour even if no orders)
- Offer sign-up bonuses ($500 for first 50 deliveries)
- Start with part-time drivers (lower commitment)
- Target existing delivery drivers from competitors

**Phase 3: Soft Launch (Month 3)**
- Limited area (5 km radius around downtown)
- Invite-only customers (beta testers)
- Heavy promotion ($20 off first 3 orders)
- Monitor metrics: order fulfillment rate, delivery time, customer satisfaction

**Phase 4: Public Launch (Month 4+)**
- Expand to full city
- Reduce promotions gradually
- Add more restaurants as demand grows
- Remove driver earning guarantees

**Key Metrics to Track:**
- Supply/Demand ratio (orders per available driver)
- Restaurant acceptance rate (>80% goal)
- Driver utilization (>60% goal)
- Customer retention (>50% order again within 30 days)

**Interview Tip:** Show understanding of marketplace dynamics and cold-start problem.

</details>

**Q5:** What trade-offs would you make between consistency and availability for different data types?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Use CAP theorem to make informed decisions:

**Strong Consistency (CP - Prioritize Consistency over Availability):**

**Orders & Payments:**
- Must be strongly consistent
- Can't have duplicate charges or lost orders
- Acceptable to fail order if database unavailable (better than double-charging)
- Use: PostgreSQL with ACID transactions

**Menu Prices:**
- Must be consistent
- Customer must pay the price they saw
- Use menu versioning (snapshot price at order time)

**Restaurant Inventory:**
- Should be consistent
- Prevent overselling (accepting order for sold-out item)
- Brief unavailability acceptable (restaurant waits 10 sec to accept order)

**Eventual Consistency (AP - Prioritize Availability over Consistency):**

**Driver Locations:**
- Can tolerate 1-2 second lag
- Showing driver 100m away when actually 150m away is acceptable
- Never want location tracking to fail (availability critical)
- Use: Cassandra for location time-series

**Restaurant Ratings:**
- 4.7★ vs 4.8★ doesn't matter in real-time
- Can update ratings every 5 minutes
- Availability more important than exact precision

**Search Results:**
- Showing slightly stale menu (item added 30 sec ago not yet indexed) is acceptable
- Better than search being unavailable
- Use: Elasticsearch with CDC sync

**Decision Matrix:**
```
Data Type         Consistency  Why?
────────────────────────────────────────────────────────
Orders            Strong       Money involved
Payments          Strong       Money involved  
Inventory         Strong       Customer trust
Driver Locations  Eventual     Real-time UX
Ratings/Reviews   Eventual     Not time-critical
Search Index      Eventual     Performance > freshness
```

**Interview Tip:** Always explain the "why" behind consistency choices, not just the "what".

</details>

#### Advanced Level

**Q6:** How would you design the system to handle a Super Bowl delivery surge (3x normal dinner rush)?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Predictive Scaling (1 Week Before):**
- Analyze historical data from previous events
- Notify restaurants to pre-stock inventory
- Recruit temporary drivers (gig workers for 1-day contract)
- Pre-scale infrastructure 2x (add servers, database replicas)

**Day-Of Preparation:**
- **Morning (8 AM):** Scale to 3x capacity
- **Pre-event (2 PM):** All systems on high alert, engineers on-call
- **Surge pricing enabled:** 1.5x-2x multiplier to balance supply/demand
- **Restaurant grouping:** Batch orders to nearby restaurants (reduce driver travel)

**Real-Time Adjustments:**
- **Auto-scaling triggers:** 
  - CPU > 60% → add 20% more servers
  - Orders/driver > 0.5 → surge pricing +10%
- **Circuit breakers:** 
  - If payment gateway >2% error rate → switch to backup
  - If database shard >80% full → reject new orders in that zone (graceful degradation)

**Capacity Buffers:**
```
Normal dinner rush:  1,150 orders/sec → 120 servers
Super Bowl surge:    3,450 orders/sec → 360 servers (3x)
Safety buffer:       +20% → 430 servers total

Cost: 430 servers × $2/hour × 6 hours = $5,160
Revenue: 3.45M orders × $3 avg commission = $10.35M
ROI: 1,987x (easily justified)
```

**Post-Event:**
- **Immediate:** Scale down to 2x (gradual decrease over 2 hours)
- **Next day:** Return to normal capacity
- **Post-mortem:** Analyze what worked, what failed, how to improve

**Monitoring Alerts:**
- Error rate >1% → Page on-call engineer
- Response time >500ms → Trigger auto-scaling
- Database CPU >80% → Add read replicas
- Order failure rate >0.1% → Activate backup systems

**Interview Tip:** Show you can think about both technical solutions (auto-scaling) and business solutions (surge pricing, temporary drivers).

</details>

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

### 🎯 Interview Questions - Capacity Planning

#### Beginner Level

**Q1:** How would you estimate the daily orders for a food delivery system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Use population-based approach:

**Step 1: Define geography**
- Let's assume United States (330M people)

**Step 2: Calculate addressable market**
- Urban/suburban population: 80% × 330M = 264M
- Age 18-65 (ordering demographic): 60% × 264M = 158M
- Smartphone users: 85% × 158M = 134M addressable users

**Step 3: Estimate penetration and frequency**
- Food delivery app users: 30% × 134M = 40M users
- Active users (order monthly): 50% × 40M = 20M monthly active users
- Order frequency: 15 orders/month on average
- Daily orders: (20M × 15) / 30 days = **10M orders/day**

**Sanity Check:**
- DoorDash actual: ~4M orders/day (2023)
- Uber Eats actual: ~6M orders/day (2023)
- Combined market: ~10M orders/day ✓

**Interview Tip:** Always do a sanity check with known data points. Shows you can validate assumptions.

</details>

**Q2:** Calculate the storage needed for 1 year of order data.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Order Record Size:**
```
Order table:
  - order_id: 8 bytes (BIGINT)
  - customer_id, restaurant_id, driver_id: 8 bytes each = 24 bytes
  - timestamps (placed, confirmed, delivered): 8 bytes each = 24 bytes
  - amounts (subtotal, fees, tax, tip): 8 bytes each = 32 bytes
  - status, notes: 200 bytes
  Total per order: ~300 bytes

Order Items (avg 3 items per order):
  - item_id, quantity, price: 24 bytes each
  - customizations: 100 bytes
  Total per item: 124 bytes
  Total for 3 items: 372 bytes

Total per order: 300 + 372 = 672 bytes ≈ 700 bytes (round up for indexes)
```

**Annual Calculation:**
```
Orders per day: 10M
Orders per year: 10M × 365 = 3.65 billion orders
Storage: 3.65B × 700 bytes = 2.56 TB

Add 30% for database indexes and overhead:
Total: 2.56 TB × 1.3 = 3.3 TB for orders
```

**Plus Other Data:**
- Location data (30-day retention): ~47 TB
- User data (40M users × 5 KB): 200 GB
- Restaurant data (500K × 100 KB): 50 GB
- Photos/images: 100 TB

**Total Year 1: ~150 TB**

**Interview Tip:** Always add overhead for indexes (20-30%) and round up for safety.

</details>

**Q3:** How many servers would you need to handle 1,200 orders/second at peak?

<details>
<parameter name="new_str">### Key Takeaways

✅ **Design for peak load, not average:** 10x traffic difference between off-peak and peak hours

✅ **Caching is critical:** 99% cache hit rate reduces database load from 2.6M to 26K reads/second

✅ **Storage grows predictably:** ~80 TB first year, ~210 TB after 5 years (mostly location data)

✅ **Bandwidth is expensive:** $160K/month, but CDN reduces server bandwidth by 90%

✅ **Infrastructure cost is tiny vs revenue:** 0.02% of revenue ($410K vs $1.875B monthly)

✅ **Multi-region deployment is necessary:** For low latency globally and disaster recovery

✅ **Auto-scaling saves money:** Scale up during peak hours, scale down during off-peak (saves $439K/month)

---

### 🎯 Interview Questions - Capacity Planning

#### Beginner Level

**Q1:** How would you estimate the daily orders for a food delivery system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Use population-based approach:

**Step 1: Define geography**
- Let's assume United States (330M people)

**Step 2: Calculate addressable market**
- Urban/suburban population: 80% × 330M = 264M
- Age 18-65 (ordering demographic): 60% × 264M = 158M
- Smartphone users: 85% × 158M = 134M addressable users

**Step 3: Estimate penetration and frequency**
- Food delivery app users: 30% × 134M = 40M users
- Active users (order monthly): 50% × 40M = 20M monthly active users
- Order frequency: 15 orders/month on average
- Daily orders: (20M × 15) / 30 days = **10M orders/day**

**Sanity Check:**
- DoorDash actual: ~4M orders/day (2023)
- Uber Eats actual: ~6M orders/day (2023)
- Combined market: ~10M orders/day ✓

**Interview Tip:** Always do a sanity check with known data points. Shows you can validate assumptions.

</details>

**Q2:** Calculate the storage needed for 1 year of order data.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Order Record Size:**
```
Order table:
  - order_id: 8 bytes (BIGINT)
  - customer_id, restaurant_id, driver_id: 8 bytes each = 24 bytes
  - timestamps (placed, confirmed, delivered): 8 bytes each = 24 bytes
  - amounts (subtotal, fees, tax, tip): 8 bytes each = 32 bytes
  - status, notes: 200 bytes
  Total per order: ~300 bytes

Order Items (avg 3 items per order):
  - item_id, quantity, price: 24 bytes each
  - customizations: 100 bytes
  Total per item: 124 bytes
  Total for 3 items: 372 bytes

Total per order: 300 + 372 = 672 bytes ≈ 700 bytes (round up for indexes)
```

**Annual Calculation:**
```
Orders per day: 10M
Orders per year: 10M × 365 = 3.65 billion orders
Storage: 3.65B × 700 bytes = 2.56 TB

Add 30% for database indexes and overhead:
Total: 2.56 TB × 1.3 = 3.3 TB for orders
```

**Plus Other Data:**
- Location data (30-day retention): ~47 TB
- User data (40M users × 5 KB): 200 GB
- Restaurant data (500K × 100 KB): 50 GB
- Photos/images: 100 TB

**Total Year 1: ~150 TB**

**Interview Tip:** Always add overhead for indexes (20-30%) and round up for safety.

</details>

**Q3:** How many servers would you need to handle 1,200 orders/second at peak?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Step 1: Determine server capacity**
- Assume 1 application server handles 10 requests/second (conservative)
- With optimizations (caching, connection pooling): 20 requests/second

**Step 2: Calculate API calls per order**
```
1 order requires:
  - 1 POST /orders (create order)
  - 2 GET /restaurants (browse, verify)
  - 1 GET /menu (fetch items)
  - 1 POST /payments (process)
  - 3 WebSocket updates (status notifications)
  Total: ~8 API calls per order
```

**Step 3: Calculate total requests**
```
Peak orders: 1,200/second
Requests: 1,200 × 8 = 9,600 requests/second

Add background tasks (location updates, analytics):
Total: 9,600 × 1.3 = 12,480 requests/second
```

**Step 4: Calculate server count**
```
Servers needed: 12,480 / 20 = 624 servers

Add 20% buffer for failures: 624 × 1.2 = 749 servers
Round up: 750 servers at peak
```

**Cost Analysis:**
```
Peak hours: 4 hours/day (lunch + dinner)
Normal hours: 20 hours/day

Peak: 750 servers × 4 hours = 3,000 server-hours
Normal: 75 servers × 20 hours = 1,500 server-hours
Daily: 4,500 server-hours

Monthly: 4,500 × 30 = 135,000 server-hours
Cost: 135,000 × $0.10/hour = $13,500/month (just compute)
```

**Interview Tip:** Always show buffer capacity (10-20%) for unexpected spikes and failures.

</details>

#### Intermediate Level

**Q4:** How would you calculate bandwidth requirements for real-time driver location tracking?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Assumptions:**
- 200K drivers online concurrently (during peak)
- Location update every 1 second
- Each location: lat/lon (16 bytes) + timestamp (8 bytes) + metadata (26 bytes) = 50 bytes

**Upload Bandwidth (Drivers → Platform):**
```
Updates per second: 200,000
Bytes per second: 200,000 × 50 bytes = 10 MB/sec
Bandwidth: 10 MB/sec × 8 bits/byte = 80 Mbps

Daily volume:
  10 MB/sec × 86,400 seconds = 864 GB/day
  Per month: 864 GB × 30 = 25.9 TB/month
```

**Download Bandwidth (Platform → Customers):**
```
Customers tracking orders: 200K (1 customer per active delivery)
Updates per customer: 1 per second
Bytes per update: 50 bytes (location) + 200 bytes (ETA, map data) = 250 bytes

Bandwidth: 200,000 × 250 bytes = 50 MB/sec = 400 Mbps
Daily: 50 MB/sec × 86,400 = 4.3 TB/day
Monthly: 130 TB/month
```

**Total Bandwidth:**
```
Upload: 26 TB/month
Download: 130 TB/month
Total: 156 TB/month

Cost (AWS data transfer):
  156 TB × $0.05/GB = $7,800/month
```

**Optimization - WebSocket vs Polling:**
```
WebSocket (bidirectional connection):
  - Overhead: 2 bytes per message
  - Efficient for real-time

HTTP Polling (customer requests every second):
  - Overhead: 500 bytes (HTTP headers)
  - Inefficient: 250× more bandwidth

Savings: Use WebSocket, save ~$1.9M/month on bandwidth!
```

**Interview Tip:** Compare alternatives (WebSocket vs polling) to show you understand optimization trade-offs.

</details>

**Q5:** How would you size your Redis cache cluster?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Data to Cache:**

**1. Driver Locations (Hot Data):**
```
Active drivers: 200,000
Data per driver: 100 bytes (location, status, current_order)
Size: 200,000 × 100 bytes = 20 MB
```

**2. Menu Data:**
```
Popular restaurants: 50,000 (10% of 500K)
Menu per restaurant: 50 items × 500 bytes = 25 KB
Size: 50,000 × 25 KB = 1.25 GB
```

**3. Session Tokens:**
```
Active users: 500,000 concurrent
Token size: 500 bytes (JWT + metadata)
Size: 500,000 × 500 bytes = 250 MB
```

**4. Order Status (Recent):**
```
Active orders: 200,000 (in-flight)
Data per order: 2 KB (status, ETA, driver info)
Size: 200,000 × 2 KB = 400 MB
```

**Total Cache Data: ~2 GB**

**Sizing Calculation:**
```
Active data: 2 GB
Add 50% for overhead (connection objects, temp data): 3 GB
Add 2x for growth: 6 GB
Add buffer for peak (3x normal): 18 GB

Recommendation: 20 GB Redis cluster
```

**Cluster Configuration:**
```
Option 1: Single large instance
  - 1× cache.r6g.xlarge (26 GB RAM)
  - Cost: $0.252/hour = $184/month
  - Risk: Single point of failure

Option 2: Clustered (recommended)
  - 3× cache.r6g.large (13 GB RAM each)
  - Total: 39 GB RAM (replication factor 2)
  - Cost: 3 × $0.126/hour = $276/month
  - Benefit: High availability, automatic failover
```

**Cache Eviction Policy:**
```
LRU (Least Recently Used) with TTL:
  - Driver locations: 30 seconds TTL
  - Menu data: 5 minutes TTL
  - Session tokens: 1 hour TTL
  - Order status: 2 hours TTL

When cache full:
  - Remove least recently accessed items
  - Prefer evicting menu data over session tokens
```

**Interview Tip:** Always plan for 2-3x headroom in cache size for growth and peak load.

</details>

#### Advanced Level

**Q6:** Design a cost optimization strategy for a food delivery platform spending $5M/year on infrastructure.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Current Cost Breakdown ($410K/month = $4.9M/year):**
```
Compute: $218K (53%)
Storage: $22K (5%)
Bandwidth: $160K (39%)
Other: $10K (2%)
```

**Optimization Strategies:**

**1. Compute Optimization (Save $87K/month):**

**Spot Instances (40% savings):**
```
Current: 100% On-Demand instances
Optimized: 70% Spot + 30% On-Demand

Stateless services (order placement, search): Spot instances
Stateful services (databases): On-Demand

Savings: $218K × 0.70 × 0.40 = $61K/month
```

**Right-Sizing (20% savings):**
```
Analysis: 30% of servers running at <20% CPU
Action: Downgrade from c5.2xlarge → c5.xlarge

Savings: $218K × 0.30 × 0.20 = $13K/month
```

**Reserved Instances for baseline (30% savings):**
```
Buy 1-year RIs for minimum capacity (50 servers always needed)
Savings: 50 servers × $0.10/hr × 730 hrs × 0.30 = $1,095/month
```

**Total Compute Savings: $75K/month**

**2. Storage Optimization (Save $8K/month):**

**Tiered Storage:**
```
Hot data (0-30 days): S3 Standard ($0.023/GB)
Warm data (31-90 days): S3 Infrequent Access ($0.0125/GB)
Cold data (90+ days): S3 Glacier ($0.004/GB)

Current: 500 TB × $0.023 = $11,500
Optimized:
  - 100 TB hot × $0.023 = $2,300
  - 200 TB warm × $0.0125 = $2,500
  - 200 TB cold × $0.004 = $800
Total: $5,600

Savings: $11,500 - $5,600 = $5,900/month
```

**Data Compression:**
```
Location data compresses 80% (lat/lon are repetitive)
500 TB × 0.80 reduction = save 400 TB

Savings: 400 TB × $0.023 = $9,200/month
```

**Total Storage Savings: $15K/month**

**3. Bandwidth Optimization (Save $80K/month):**

**Image Optimization:**
```
Restaurant photos: 200 KB → 50 KB (WebP format, lazy loading)
Bandwidth reduction: 75%

Savings: $160K × 0.50 × 0.75 = $60K/month
```

**CDN Edge Caching:**
```
Increase cache hit rate: 85% → 95%
Reduce origin requests by 67%

Savings: $160K × 0.10 = $16K/month
```

**Geographic Costing:**
```
Most traffic from US, but some from Asia/Europe
Move static assets to regional CDN POPs

Savings: $4K/month
```

**Total Bandwidth Savings: $80K/month**

**TOTAL SAVINGS: $170K/month = $2.04M/year (42% reduction)**

**Final Cost: $410K - $170K = $240K/month = $2.88M/year**

**ROI Table:**
```
Optimization          Investment  Monthly Savings  Payback Period
────────────────────────────────────────────────────────────────
Spot Instances        $0          $61K            Immediate
Right-Sizing          $10K        $13K            0.8 months
Reserved Instances    $0          $1K             Immediate
Tiered Storage        $5K         $15K            0.3 months
Image Optimization    $20K        $60K            0.3 months
CDN Improvements      $15K        $20K            0.75 months

Total                 $50K        $170K           0.3 months
```

**Interview Tip:** Always quantify savings and show ROI. Cloud optimization is 20-40% savings with minimal effort.

</details>

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

### 🎯 Interview Questions - System Architecture Overview

#### Beginner Level

**Q1:** What are the main advantages of using a microservices architecture for a food delivery platform?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Microservices architecture provides several critical benefits for food delivery systems:

**Independent Scaling:**
- **Order Service** handles 115 orders/sec average but spikes to 1,150/sec at dinner rush (6-8 PM)
- **Location Service** processes 200K GPS updates/sec continuously
- Can scale Order Service 10x during peak while Location Service stays constant
- Cost savings: Only scale what you need, not entire monolith

**Fault Isolation:**
- If **Recommendation Engine** crashes, customers can still place orders
- If **Analytics Service** goes down, real-time operations continue
- Example: Uber Eats in 2019 - promotion service crashed but orders still processed
- 99.9% uptime maintained even when non-critical services fail

**Technology Flexibility:**
- **Matching Service:** Uses Go for high-performance geospatial calculations
- **Payment Service:** Uses Java for enterprise integrations
- **Location Service:** Uses Erlang for real-time concurrency
- Choose best tool for each job, not one-size-fits-all

**Team Autonomy:**
- Order team can deploy 5x/day without coordinating with Driver team
- Faster feature delivery (weekly releases vs quarterly monolith releases)
- Smaller codebase per team (50K lines vs 2M line monolith)

**Deployment Safety:**
- Rolling deployments: Update 10% of Order Service, test, rollout to 100%
- Blue-green deployments: Run old and new versions simultaneously
- If bug found: Rollback in 30 seconds (vs hours for monolith)

**Example Comparison:**
```
Monolith Problem:
- Update restaurant menu feature
- Requires redeploying entire application
- Risk: Payment processing might break
- Downtime: 15 minutes maintenance window

Microservices Solution:
- Update only Restaurant Service
- Other services unaffected
- Zero downtime deployment
- Rollback takes 30 seconds if needed
```

**Interview Tip:** Always mention the trade-offs—microservices add operational complexity (more services to monitor, distributed tracing needed) but provide critical benefits at scale.

</details>

**Q2:** How does event-driven architecture with Kafka help decouple services in a food delivery system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Event-driven architecture with Kafka enables asynchronous communication and temporal decoupling:

**How It Works:**
When a customer places an order, the Order Service publishes an `order.created` event to Kafka. Multiple services consume this event independently:
- **Matching Service** starts searching for available drivers
- **Restaurant Service** notifies the restaurant to prepare food
- **Analytics Service** logs order for business intelligence
- **Notification Service** sends confirmation email/SMS to customer

**Key Benefits:**

**1. Temporal Decoupling:**
- Order Service doesn't wait for all consumers to process event
- Order placement returns in <200ms (vs 2+ seconds if calling all services)
- If Analytics Service is slow, doesn't block order confirmation

**2. Service Independence:**
- Adding new consumer (e.g., Fraud Detection Service) doesn't require changing Order Service
- Can deploy new analytics features without touching order placement code
- Services can be offline during deployment—events queue up in Kafka

**3. Reliability Through Persistence:**
- Kafka persists events for 7 days (configurable retention)
- If Matching Service crashes, events aren't lost
- On restart, processes missed events from Kafka
- Example: Kafka stores 50M events/day × 7 days = 350M events (~70 TB)

**4. Replay Capability:**
- Bug in Analytics Service? Replay last 24 hours of orders
- Testing new feature? Replay production events in staging
- Debugging: Replay specific order events to reproduce issue

**Real-World Example:**
```
DoorDash Order Flow (Synchronous - BAD):
Customer → Order Service → calls Restaurant Service → calls Matching Service → calls Notification Service
Total latency: 200ms + 150ms + 300ms + 100ms = 750ms
Problem: If Notification Service times out (10 sec), order placement fails

Uber Eats Order Flow (Event-Driven - GOOD):
Customer → Order Service (publishes event) → returns immediately (200ms)
           ↓ Kafka
           ├→ Restaurant Service (consumes async)
           ├→ Matching Service (consumes async)
           └→ Notification Service (consumes async)
Total latency to customer: 200ms (4x faster!)
Resilience: If any consumer fails, order still confirmed
```

**Message Format:**
```json
{
  "event_type": "order.created",
  "order_id": "ORD-12345",
  "timestamp": "2025-01-15T18:30:00Z",
  "customer_id": "USR-789",
  "restaurant_id": "RST-456",
  "items": [...],
  "total_amount": 42.50,
  "delivery_address": {...}
}
```

**Interview Tip:** Mention that Kafka's partitioning ensures ordered processing within a partition (all events for order_id "ORD-12345" go to same partition), which is critical for maintaining state machine consistency.

</details>

**Q3:** Why would you use both REST APIs and WebSockets in the same system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

REST APIs and WebSockets serve different communication patterns:

**REST APIs (Request-Response):**
Use for operations that need immediate response and happen infrequently:
- **Place order:** Customer initiates, expects confirmation in <200ms
- **Search restaurants:** Customer searches, expects results immediately
- **Update driver availability:** Driver toggles online/offline a few times per shift
- **Characteristics:** Client-initiated, stateless, one request = one response

**WebSockets (Bidirectional Streaming):**
Use for continuous data streams that update frequently:
- **Driver location:** Updates every 1 second (200K active drivers = 200K updates/sec)
- **Order status:** Real-time updates (preparing → ready → picked up → delivered)
- **Live ETA updates:** Customer sees "Driver is 5 min away... 4 min... 3 min..."
- **Characteristics:** Server-initiated, stateful connection, continuous updates

**Why Both?**

**1. Efficiency:**
```
REST Approach (Polling - BAD):
Customer app polls every 5 seconds: "Is my order ready?"
- 200K active customers × 12 polls/min = 2.4M requests/min (40K req/sec)
- Most responses: "No, still preparing" (wasted bandwidth)
- 5-second latency between status changes

WebSocket Approach (Push - GOOD):
Server pushes updates only when status changes
- 200K customers, 4 status changes per order = 800K pushes per minute (13K/sec)
- 3x less traffic than polling
- <1 second latency (instant notification)
```

**2. Resource Usage:**
- REST: Each request opens/closes TCP connection (overhead)
- WebSocket: Single persistent connection per client (efficient)
- For 200K concurrent users: 200K persistent connections vs 40K new connections/sec

**3. Real-World Example:**
Uber Eats driver app:
- **REST:** `/api/v1/drivers/accept-order` (POST) - Driver accepts delivery
- **WebSocket:** `ws://api.ubereats.com/drivers/{driver_id}/stream` - Receives new order notifications in real-time

Customer app:
- **REST:** `/api/v1/orders` (POST) - Place order
- **WebSocket:** `ws://api.ubereats.com/orders/{order_id}/status` - Watch order progress live

**Trade-offs:**
- REST: Simpler to implement, easier to cache, works through corporate firewalls
- WebSocket: More complex (need state management), harder to load balance, requires WebSocket-aware proxies
- Solution: Use REST as default, WebSocket only for high-frequency updates

**Interview Tip:** Mention that you'd use WebSocket for <1% of traffic (status updates) and REST for 99% of traffic (CRUD operations), showing you understand when each is appropriate.

</details>

#### Intermediate Level

**Q4:** How would you implement circuit breakers to handle service failures between microservices?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Circuit breakers prevent cascading failures when downstream services become unhealthy:

**How Circuit Breaker Works:**

**1. Three States:**
```
CLOSED (Normal Operation):
- All requests pass through to downstream service
- Monitor error rate and response time
- If 10 consecutive failures OR 50% error rate → OPEN

OPEN (Service Failing):
- Immediately reject requests without calling service
- Return fallback response or cached data
- After 30 seconds → HALF-OPEN (test if service recovered)

HALF-OPEN (Testing Recovery):
- Allow 5 test requests through
- If all succeed → CLOSED (service recovered)
- If any fail → OPEN (service still failing)
```

**2. Implementation Example:**

When Order Service calls Payment Service:
```
Circuit Breaker Monitoring:
- Last 100 requests: 15 failures (15% error rate)
- Threshold: 20% error rate
- Status: CLOSED (still working)

Then Payment Service database crashes:
- Next 10 requests: All fail (100% error rate)
- Threshold exceeded → Circuit OPEN
- Order Service stops calling Payment Service
- Returns to customer: "Payment temporarily unavailable, try again in 1 minute"
```

**3. Real-World Configuration:**
```
Order Service → Restaurant Service:
- Timeout: 2 seconds
- Error threshold: 50% over 10 requests
- Open duration: 30 seconds
- Fallback: Return cached menu

Order Service → Payment Service:
- Timeout: 5 seconds (critical path, more time)
- Error threshold: 20% over 10 requests (strict, involves money)
- Open duration: 60 seconds
- Fallback: Queue order, process payment async

Driver Service → Location Service:
- Timeout: 1 second (real-time)
- Error threshold: 80% over 20 requests (high tolerance)
- Open duration: 10 seconds
- Fallback: Use last known location
```

**4. Benefits:**

**Prevents Cascading Failures:**
```
Without Circuit Breaker:
Order Service → Restaurant Service (slow, 10s timeout)
1,000 concurrent orders × 10s timeout = 10,000 threads waiting
Order Service runs out of threads → crashes
Now Customer Service also crashes (calls Order Service)
Entire platform down!

With Circuit Breaker:
Order Service → Restaurant Service (detects failure after 10 requests)
Circuit opens → fast-fail in <10ms
Order Service remains healthy with 1,000 concurrent orders
Uses fallback: cached restaurant data
Only Restaurant Service affected, rest of platform works
```

**5. Monitoring Metrics:**
- Circuit state changes per minute (CLOSED → OPEN should be rare)
- Fallback invocation rate (how often are we serving stale data?)
- Service recovery time (how long in OPEN state?)

**6. Production Example from Uber Eats:**
During Black Friday 2022, Uber Eats Recommendation Service became overloaded:
- Circuit breaker detected 60% error rate
- Opened circuit after 15 failed requests
- Fallback: Show "popular restaurants" instead of personalized recommendations
- Result: Order placement still worked, personalization degraded gracefully
- Recommendation Service recovered in 4 minutes
- Circuit closed automatically, personalization resumed

**Interview Tip:** Explain that circuit breakers are about "failing fast and gracefully"—better to show cached menu than wait 10 seconds for timeout, then crash the whole service.

</details>

**Q5:** Design a strategy for handling partial failures in a distributed order placement flow involving Order Service, Restaurant Service, Matching Service, and Payment Service.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Handling partial failures requires idempotency, retry logic, and saga patterns:

**Failure Scenarios:**

**Scenario 1: Payment Succeeds, Restaurant Notification Fails**
```
Timeline:
1. Order created (order_id: ORD-123)
2. Payment charged: $42.50 ✅
3. Kafka publishes order.created event ✅
4. Restaurant Service crashes before consuming event ❌

Problem:
- Customer charged but restaurant never notified
- Food not prepared, customer waiting
- Money taken, no service provided!

Solution (Saga with Compensation):
1. Order Service detects Restaurant Service not acknowledged after 30 sec
2. Triggers compensating transaction:
   - Refund payment: $42.50
   - Update order status: FAILED_RESTAURANT_UNREACHABLE
   - Send notification: "Order cancelled, refund in 3-5 days"
3. Log incident for manual review
```

**Scenario 2: Matching Service Finds Driver, But Driver Already Offline**
```
Problem:
- Matching Service assigns order to Driver A at 6:00:00 PM
- Driver A went offline at 5:59:58 PM (2 seconds before)
- Race condition due to eventual consistency

Solution (Optimistic Locking):
1. Matching Service assigns order with version check:
   PUT /drivers/{driver_id}/assignment
   If-Match: driver_version=v123
   
2. Driver Service validates:
   - Check if driver still online
   - Check if driver hasn't exceeded max concurrent orders (3)
   - If validation fails → return 409 Conflict
   
3. Matching Service receives 409 → retries with different driver
4. After 3 retries (3 seconds), escalate:
   - Increase delivery fee to attract more drivers
   - Expand search radius from 2km to 5km
```

**Scenario 3: Network Partition During Order Placement**
```
Problem:
- Customer clicks "Place Order"
- Request sent to Order Service
- Network timeout (15 seconds)
- Customer clicks "Place Order" again (duplicate)

Solution (Idempotency Keys):
1. Customer app generates idempotency key on first attempt:
   POST /orders
   X-Idempotency-Key: 7c9e8a2f-4b3d-11eb-b378-0242ac130002
   
2. Order Service checks Redis cache:
   - Key exists? Return existing order (201 Created)
   - Key doesn't exist? Process new order, cache result for 24 hours
   
3. Second click with same key → returns same order (no duplicate charge)
```

**Comprehensive Strategy:**

**1. Use Saga Pattern (Choreography-Based):**
```
Order Placement Saga:
1. Order Service: Create order → publish order.created
2. Payment Service: Charge customer → publish payment.completed
3. Restaurant Service: Notify restaurant → publish restaurant.notified
4. Matching Service: Assign driver → publish driver.assigned

If any step fails:
- Publish compensating event (order.cancelled, payment.refunded)
- Each service listens for compensating events
- Rollback happens automatically
```

**2. Retry with Exponential Backoff:**
```
Attempt 1: Immediate (0ms delay)
Attempt 2: 100ms delay
Attempt 3: 400ms delay  (100 × 2²)
Attempt 4: 1,600ms delay (100 × 4²)
Attempt 5: Give up, trigger compensation

Configuration:
- Max retries: 5
- Timeout per attempt: 2 seconds
- Total timeout: 10 seconds (user waits max 10 sec)
```

**3. Dead Letter Queues (DLQ):**
```
Normal Flow:
Order Service → Kafka → Restaurant Service ✅

Failure After 5 Retries:
Order Service → Kafka → Restaurant Service (failed 5x)
                   ↓
              Dead Letter Queue (DLQ)
                   ↓
         Manual Review Dashboard
         (Customer Support investigates)
```

**4. Health Checks and Graceful Degradation:**
```
Before attempting operation:
1. Check service health endpoint: GET /health
2. If unhealthy → skip operation, use fallback
3. Example: If Restaurant Service down:
   - Don't attempt notification
   - Log order to manual review queue
   - Customer Support calls restaurant by phone
```

**Production Metrics:**
```
Target SLOs:
- Order success rate: 99.5% (1 in 200 orders may fail)
- Partial failure recovery: <30 seconds (saga compensation)
- Duplicate prevention: 100% (idempotency)
- Failed orders in DLQ: <0.1% (manual review)

DoorDash Production Numbers:
- 10M orders/day
- 50K partial failures/day (0.5%)
- 45K recovered via retry (90% success rate)
- 5K sent to DLQ for manual resolution
- Average compensation time: 18 seconds
```

**Interview Tip:** Emphasize that "eventual success" is acceptable in many cases—if restaurant notification fails initially, retry for 30 seconds before giving up. Most transient failures resolve within seconds.

</details>

#### Advanced Level

**Q6:** Compare microservices vs monolith architecture for a food delivery startup with 100 daily orders vs an established platform like Uber Eats with 10M daily orders. When would you recommend migrating?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

The microservices vs monolith decision depends on scale, team size, and operational maturity:

**Startup Phase (100 orders/day):**

**Recommend: Monolith Architecture**

**Reasoning:**
- **Single server handles load:** 100 orders/day = 0.001 orders/sec (trivial load)
- **Faster development:** No distributed system complexity, shared codebase
- **Lower operational cost:** 1 server ($200/month) vs 10 microservices ($2,000/month)
- **Easier debugging:** Single log file, no distributed tracing needed
- **Smaller team:** 3-5 engineers manage entire codebase

**Architecture:**
```
Django Monolith:
- Single PostgreSQL database
- Single Redis cache
- Deploy to AWS EC2 t3.medium ($50/month)
- Nginx reverse proxy
- All features in one codebase (customers, restaurants, drivers)

Pros:
- Time to market: 3 months (vs 9 months for microservices)
- Cost: $500/month total
- Can reach 10,000 orders/day before scaling issues

Cons:
- All features deployed together (risky)
- Entire app goes down if one feature crashes
- Harder to scale horizontally (must scale entire app)
```

**Growth Phase (10K orders/day):**

**Recommend: Modular Monolith**
- Keep single deployment unit
- But organize code into modules with clear boundaries
- Prepare for future microservices migration
- Example: Separate Python packages for orders, restaurants, drivers

**Scale-Up Phase (100K orders/day):**

**Trigger Migration to Microservices**

**Migration Indicators:**
1. **Team Growth:** 20+ engineers (multiple teams needed)
2. **Deployment Frequency:** Teams blocked waiting for releases (want 5x/day deploys)
3. **Performance Bottlenecks:** Different components need independent scaling
4. **Availability Requirements:** 99.9% uptime needed (42 min downtime/month)

**Migration Strategy (Strangler Pattern):**
```
Month 1-3: Extract Location Service
- Highest traffic (200K GPS updates/sec)
- Independent scaling needed
- Clear API boundary
- Build new Location microservice
- Route 10% traffic → 50% → 100% over 6 weeks
- Old monolith code stays as backup

Month 4-6: Extract Payment Service
- Requires PCI DSS compliance isolation
- Different security requirements
- Deploy to separate PCI-compliant infrastructure

Month 7-9: Extract Matching Service
- Complex algorithm (Go for performance)
- Independent release cycle
- Heavy compute requirements

Month 10-12: Extract remaining services
- Order, Restaurant, Driver, Notification services
- Core monolith shrinks to API Gateway
```

**Established Platform (10M orders/day - Uber Eats Scale):**

**Microservices Architecture (Required)**

**Configuration:**
```
15+ Microservices:
├─ Order Service (120 instances, Go)
├─ Payment Service (80 instances, Java)
├─ Restaurant Service (60 instances, Python)
├─ Driver Service (100 instances, Go)
├─ Matching Service (200 instances, Go) ← highest load
├─ Location Service (150 instances, Erlang)
├─ Notification Service (40 instances, Node.js)
├─ Analytics Service (30 instances, Scala)
├─ Search Service (50 instances, Elasticsearch)
└─ ... 6 more services

Total Infrastructure:
- 1,000+ servers
- 100+ database instances
- 50+ Redis clusters
- 20+ Kafka clusters
- Cost: $2M/month (but revenue: $30M/month commission)
```

**Cost-Benefit Analysis:**

**Monolith vs Microservices at Different Scales:**
```
100 orders/day:
- Monolith: $500/month, 2 engineers
- Microservices: $2,000/month, 5 engineers (4x cost, no benefit)
- Decision: Monolith wins

10,000 orders/day:
- Monolith: $2,000/month, 10 engineers (starting to struggle)
- Microservices: $5,000/month, 15 engineers (clearer ownership)
- Decision: Borderline, consider modular monolith

100,000 orders/day:
- Monolith: Infeasible (deploys take 30 min, frequent outages)
- Microservices: $50,000/month, 50 engineers across 10 teams
- Decision: Microservices required

10,000,000 orders/day:
- Monolith: Impossible
- Microservices: $2M/month, 200 engineers, essential for scale
- Decision: Microservices only option
```

**Migration Triggers Checklist:**
```
Migrate to microservices when ANY of these are true:
☑ Daily orders > 100K
☑ Engineering team > 20 people
☑ Deployment frequency needs > 2x/day per team
☑ Different components need different scaling (10x location traffic vs payment)
☑ Regulatory compliance requires isolation (PCI DSS for payments)
☑ Response time degradation (p95 latency > 1 second)
☑ Incidents cause full platform downtime (no fault isolation)
```

**Real-World Example:**
DoorDash started as a monolith in 2013:
- 0-10K orders/day (2013-2014): Monolith in Python
- 10K-100K orders/day (2014-2016): Modular monolith
- 100K-1M orders/day (2016-2018): Migrated to microservices
- 1M-10M orders/day (2018-2020): Added 30+ microservices
- Today (2025): 100+ microservices, 5,000+ engineers

**Migration took 2 years with 50 engineers** (expensive but necessary).

**Interview Tip:** Show you understand "premature optimization"—don't build for scale you don't have. Start simple (monolith), migrate when complexity justifies cost. Use metrics to justify migration decision.

</details>

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

### 🎯 Interview Questions - Database Design

#### Beginner Level

**Q1:** Why would you use PostgreSQL for orders and payments but Cassandra for driver locations in a food delivery system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Different data types have different requirements—choose databases based on access patterns and consistency needs:

**PostgreSQL for Orders and Payments:**

**Why PostgreSQL?**
- **ACID Transactions:** Orders involve money—must guarantee all-or-nothing execution
- **Strong Consistency:** Customer sees correct order status immediately after payment
- **Complex Queries:** Join orders with customers, payments, and restaurant data
- **Mature Ecosystem:** 20+ years of production use, well-understood failure modes

**Example Order Transaction:**
```sql
BEGIN TRANSACTION;
  -- 1. Create order record
  INSERT INTO orders (customer_id, restaurant_id, total_amount)
  VALUES (123, 456, 42.50);
  
  -- 2. Charge payment
  INSERT INTO payments (order_id, amount, status)
  VALUES (789, 42.50, 'CHARGED');
  
  -- 3. Update customer loyalty points
  UPDATE customers SET loyalty_points = loyalty_points + 42
  WHERE customer_id = 123;
COMMIT;

If ANY step fails (e.g., payment declined):
- Entire transaction rolls back
- Order not created, customer not charged
- Data stays consistent
```

**Cassandra for Driver Locations:**

**Why Cassandra?**
- **High Write Throughput:** 200K drivers × 1 update/sec = 200K writes/sec
- **Time-Series Optimized:** Driver locations have natural time ordering
- **Eventual Consistency OK:** Showing driver 100m away when actually 120m is acceptable
- **Linear Scalability:** Add more nodes for more throughput (PostgreSQL doesn't scale writes this way)

**Example Location Write:**
```sql
-- Single driver sends location every second
INSERT INTO driver_locations (driver_id, timestamp, lat, lng)
VALUES (12345, '2025-01-15 18:30:00', 37.7749, -122.4194);

-- Cassandra distributes across nodes automatically
-- No cross-node coordination needed (fast!)
-- Data available in 10-50ms globally (eventual consistency)
```

**Performance Comparison:**
```
PostgreSQL:
- Write throughput: 10K writes/sec per node (limited by ACID overhead)
- Read latency: 5-10ms (complex queries with joins)
- Consistency: Strong (always see latest data)
- Best for: Transactional data (orders, payments)

Cassandra:
- Write throughput: 100K writes/sec per node (optimized for writes)
- Read latency: 1-5ms (simple key-value lookups)
- Consistency: Tunable (eventual by default)
- Best for: Time-series data (locations, events, logs)
```

**Real-World Scale:**
```
Uber Eats - 10M daily orders:
- PostgreSQL cluster: 20 servers (master + replicas)
- Handles: 115 orders/sec average, 1,150 peak
- Storage: 5 TB (orders, payments, customers)

- Cassandra cluster: 50 servers (distributed)
- Handles: 200K location updates/sec
- Storage: 200 TB (driver locations, historical tracking)
```

**Cost-Benefit:**
- PostgreSQL: Higher per-server cost ($2K/month) but fewer servers needed
- Cassandra: Lower per-server cost ($1K/month) but more servers needed for redundancy
- Total cost similar, but serves different purposes

**Interview Tip:** Explain that it's not PostgreSQL vs Cassandra—use BOTH based on data characteristics. Orders need ACID (PostgreSQL), locations need throughput (Cassandra).

</details>

**Q2:** What indexes would you create on the orders table to optimize common queries?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Indexes dramatically improve query performance but have trade-offs:

**Common Query Patterns:**

**Query 1: Find customer's recent orders**
```sql
SELECT * FROM orders 
WHERE customer_id = 12345 
ORDER BY created_at DESC 
LIMIT 20;

Index Needed:
CREATE INDEX idx_orders_customer_time 
ON orders(customer_id, created_at DESC);

Why Composite Index?
- Filter by customer_id first (narrow down to 1,000 orders)
- Then sort by created_at (already sorted in index)
- No table scan needed (fast!)

Performance:
- Without index: 50ms (scans 10M orders)
- With index: 2ms (direct lookup of 1,000 orders)
- 25x faster!
```

**Query 2: Find restaurant's active orders**
```sql
SELECT * FROM orders 
WHERE restaurant_id = 456 
  AND status IN ('CONFIRMED', 'PREPARING', 'READY')
ORDER BY created_at ASC;

Index Needed:
CREATE INDEX idx_orders_restaurant_status_time 
ON orders(restaurant_id, status, created_at ASC);

Why Three Columns?
- Filter by restaurant_id (narrow to 100 orders)
- Filter by status (narrow to 20 active orders)
- Sort by created_at (FIFO order for kitchen)

Performance:
- Without index: 30ms (scans all orders)
- With index: 1ms (index-only scan)
- 30x faster!
```

**Query 3: Find orders by delivery address (for fraud detection)**
```sql
SELECT * FROM orders 
WHERE delivery_address = '123 Main St, SF CA 94102'
  AND created_at > NOW() - INTERVAL '24 hours';

Index Needed:
CREATE INDEX idx_orders_address_time 
ON orders(delivery_address, created_at DESC);

Use Case:
- Detect if 10+ orders to same address in 1 hour (potential fraud)
- Track delivery patterns for route optimization
```

**Partial Index for Active Orders:**
```sql
-- Most queries only care about orders from last 30 days
-- Old orders (6 months+) rarely accessed

CREATE INDEX idx_orders_active 
ON orders(status, restaurant_id, created_at)
WHERE created_at > NOW() - INTERVAL '30 days';

Benefits:
- Index size: 500 MB instead of 50 GB (100x smaller!)
- Faster inserts (smaller index to update)
- Covers 95% of queries (recent orders only)
```

**Index Strategy:**
```
DO Create Indexes For:
✅ Foreign keys (customer_id, restaurant_id, driver_id)
✅ Status columns frequently filtered (order status)
✅ Timestamp columns (created_at, updated_at)
✅ Composite indexes for common query combinations

DON'T Create Indexes For:
❌ Low cardinality columns (is_cancelled: only 2 values)
❌ Columns rarely queried (internal notes)
❌ Small tables (<10,000 rows) - table scan is faster
```

**Index Maintenance:**
```
PostgreSQL Automatic Maintenance:
- Indexes updated on every INSERT/UPDATE (cost: +20ms per write)
- Indexes consume disk space (10-30% of table size)
- VACUUM periodically cleans up dead index entries

Production Example:
- Orders table: 10M rows = 5 GB data
- 6 indexes: 1.5 GB total (30% overhead)
- Insert latency: 5ms without indexes → 25ms with 6 indexes
- Trade-off: Slower writes for 100x faster reads (acceptable!)
```

**Real-World Uber Eats Configuration:**
```sql
-- Customer order history (most common query)
CREATE INDEX idx_orders_customer_time 
ON orders(customer_id, created_at DESC);

-- Restaurant active orders (kitchen dashboard)
CREATE INDEX idx_orders_restaurant_active 
ON orders(restaurant_id, status)
WHERE status != 'DELIVERED' AND status != 'CANCELLED';

-- Driver delivery history (earnings calculation)
CREATE INDEX idx_orders_driver_completed 
ON orders(driver_id, created_at)
WHERE status = 'DELIVERED';

-- Fraud detection (same address multiple orders)
CREATE INDEX idx_orders_address_recent 
ON orders(delivery_address, created_at)
WHERE created_at > NOW() - INTERVAL '7 days';

-- Analytics (order value trends)
CREATE INDEX idx_orders_time_value 
ON orders(created_at, total_amount);

Total: 5 indexes covering 98% of queries
```

**Interview Tip:** Mention the trade-off—indexes slow down writes (each index adds 5ms to INSERT) but speed up reads (10-100x). For read-heavy systems like order history, this is acceptable. For write-heavy systems like location tracking, use fewer indexes.

</details>

**Q3:** How would you handle database schema migrations for the orders table with 100M rows without downtime?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Database migrations on large tables require careful planning to avoid locking and downtime:

**Scenario: Adding a "tip_amount" column to orders table**

**❌ Naive Approach (Causes 30-minute downtime):**
```sql
-- This locks the entire table during migration!
ALTER TABLE orders ADD COLUMN tip_amount DECIMAL(10,2) DEFAULT 0.0;

Problem:
- 100M rows × 0.5ms per row = 50,000 seconds (14 hours!)
- Table locked during entire migration
- All order placements blocked
- Revenue loss: 14 hours × $50K/hour = $700K!
```

**✅ Zero-Downtime Migration Strategy:**

**Phase 1: Add Column (Nullable, No Default) - 1 minute**
```sql
-- No table rewrite needed, only schema change
ALTER TABLE orders ADD COLUMN tip_amount DECIMAL(10,2) DEFAULT NULL;

-- No rewrite because:
-- - NULL values don't consume space
-- - No row scanning needed
-- Downtime: <5 seconds (just schema lock)
```

**Phase 2: Backfill Data Gradually - 6 hours**
```sql
-- Update 1 million rows at a time (small batches)
-- Run during low-traffic hours (2 AM - 8 AM)

DO $$
DECLARE
  batch_size INT := 1000000;
  min_id BIGINT := 1;
  max_id BIGINT;
BEGIN
  SELECT MAX(order_id) INTO max_id FROM orders;
  
  WHILE min_id <= max_id LOOP
    -- Update one batch
    UPDATE orders
    SET tip_amount = 0.0
    WHERE order_id BETWEEN min_id AND min_id + batch_size
      AND tip_amount IS NULL;
    
    -- Release locks between batches (crucial!)
    COMMIT;
    
    -- Sleep 100ms to avoid overloading database
    PERFORM pg_sleep(0.1);
    
    min_id := min_id + batch_size;
  END LOOP;
END $$;

Benefits:
- Each batch takes 2 seconds (short lock duration)
- 100M rows ÷ 1M per batch = 100 batches × 2 sec = 200 sec (~3 min query time)
- With 100ms sleep between batches = 10 sec sleep time total
- Total time: 3 min + pauses = backfill over 6 hours
- Application continues running (NULL is handled gracefully)
```

**Phase 3: Application Code Update - Rolling Deployment**
```python
# Old code (before migration) - handles NULL
order_data = {
    'order_id': 12345,
    'total_amount': 42.50,
    'tip_amount': row.tip_amount or 0.0  # Handles NULL gracefully
}

# New code (after backfill) - writes tip_amount
order_data = {
    'order_id': 12345,
    'total_amount': 42.50,
    'tip_amount': 5.00  # Now always populated
}

Deployment Strategy:
- Deploy new code to 10% of servers (canary)
- Monitor for 1 hour
- If no errors, deploy to 100% over 6 hours (rolling)
```

**Phase 4: Add NOT NULL Constraint - 1 minute**
```sql
-- After backfill complete and new code deployed
ALTER TABLE orders ALTER COLUMN tip_amount SET DEFAULT 0.0;
ALTER TABLE orders ALTER COLUMN tip_amount SET NOT NULL;

-- This is fast because all rows already have values
-- No table rewrite needed
```

**Advanced Techniques:**

**1. Use pg_repack for Major Schema Changes:**
```sql
-- For complex migrations (changing column type, adding multiple columns)
-- pg_repack rebuilds table in background without locking

-- Install extension
CREATE EXTENSION pg_repack;

-- Rebuild orders table with new schema
pg_repack --table orders --jobs 4

How it works:
- Creates new table with new schema
- Copies rows incrementally (batched)
- Tracks changes via triggers
- Swaps old table with new table (1 second lock)
- Total time: 4 hours for 100M rows
- Downtime: <5 seconds (just the swap)
```

**2. Shadow Tables for Large Migrations:**
```sql
-- Create new table with desired schema
CREATE TABLE orders_new (
  order_id BIGINT PRIMARY KEY,
  customer_id BIGINT NOT NULL,
  tip_amount DECIMAL(10,2) NOT NULL DEFAULT 0.0,
  -- ... other columns
);

-- Dual-write: Application writes to both tables
-- (Code change deployed first)

-- Backfill old data from orders → orders_new
-- (Background job over 24 hours)

-- Switch read traffic: orders_new → orders
-- (Atomic rename)
ALTER TABLE orders RENAME TO orders_old;
ALTER TABLE orders_new RENAME TO orders;

-- Drop old table after 7 days (safety buffer)
DROP TABLE orders_old;
```

**Production Checklist:**
```
Before Migration:
☑ Backup database (pg_dump)
☑ Test migration on staging with production-size data
☑ Schedule during low-traffic hours (2 AM - 6 AM)
☑ Have rollback plan (how to undo migration)
☑ Notify engineering team (on-call ready)

During Migration:
☑ Monitor query performance (slow query log)
☑ Monitor replication lag (replicas behind master?)
☑ Monitor application error rate (500 errors spiking?)
☑ Run migrations in batches (1M rows at a time)

After Migration:
☑ Verify data correctness (compare row counts)
☑ Run ANALYZE to update query planner statistics
☑ Monitor for 24 hours (watch for anomalies)
```

**Real-World Example from DoorDash:**
In 2019, DoorDash migrated orders table from INT to BIGINT for order_id:
- Table size: 500M rows
- Migration approach: Shadow table + dual writes
- Duration: 2 weeks (gradual backfill)
- Downtime: 0 seconds
- Engineer cost: 3 engineers × 2 weeks = 6 engineer-weeks
- Worth it to avoid running out of order IDs!

**Interview Tip:** Emphasize that large table migrations are about minimizing risk—do it gradually in phases, with each phase reversible, rather than one big-bang change that could crash the database.

</details>

#### Intermediate Level

**Q4:** Design a database sharding strategy for a food delivery platform that handles 10M orders/day across 100 cities. What would you use as the shard key and why?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Sharding distributes data across multiple databases for horizontal scalability:

**Sharding Requirements:**

**Scale Targets:**
- 10M orders/day = 115 orders/sec average, 1,150 orders/sec peak
- Single PostgreSQL server handles ~10K writes/sec (sufficient for now)
- But: 10M orders × 365 days × 3 years = 10.9B orders = 5 TB (too large for one server)
- Need sharding for storage capacity, not write throughput (yet)

**Shard Key Options:**

**Option 1: Shard by order_id (❌ Poor Choice)**
```
Shard Assignment:
- order_id % num_shards = shard_number
- Example: order_id 12345 → 12345 % 10 = shard 5

Problems:
- Customer orders distributed across all shards
- Query "get customer's recent orders" requires querying all 10 shards (scatter-gather)
- Latency: 10 queries × 50ms = 500ms (too slow!)
- Can't route to single shard based on common queries
```

**Option 2: Shard by customer_id (❌ Uneven Distribution)**
```
Shard Assignment:
- customer_id % num_shards = shard_number

Problems:
- Power users create hot shards
- Example: Corporate customer (Google) orders 1,000 lunches/day → all on shard 3
- Shard 3 gets 10x traffic of other shards
- Can't scale evenly

Data Distribution:
Shard 0: 900K orders
Shard 1: 950K orders
Shard 2: 1.1M orders
Shard 3: 4.2M orders ← Corporate customers!
Shard 4: 850K orders
...
Uneven load = poor resource utilization
```

**Option 3: Shard by city_id (✅ Good, but not optimal)**
```
Shard Assignment:
- Each city gets assigned to a shard
- city_id → shard_mapping table

Example:
City          Orders/Day    Shard
─────────────────────────────────
San Francisco  500K         Shard 1
New York       800K         Shard 2
Chicago        300K         Shard 3
Los Angeles    600K         Shard 4

Benefits:
- Geographic locality (city data co-located)
- Easy to route: city known from delivery address
- Most queries within single city (95% of queries)

Problems:
- Uneven distribution (NYC 2.5x Chicago)
- Hard to rebalance (move entire city to new shard)
```

**Option 4: Shard by (city_id + consistent hashing) (✅ Optimal)**
```
Shard Assignment:
shard = consistent_hash(city_id, order_date_week) % num_shards

Example:
San Francisco, Week 1 → Shard 1
San Francisco, Week 2 → Shard 3
San Francisco, Week 3 → Shard 7
(Distributes SF orders across multiple shards)

Benefits:
- Even distribution (each week hashes differently)
- Geographic locality maintained (same week → same shard)
- Common queries still work:
  - "SF orders this week" → query 1 shard
  - "SF orders last 4 weeks" → query 4 shards (acceptable)
  
Performance:
- 95% of queries hit 1 shard (this week's orders)
- 4% of queries hit 2-4 shards (recent history)
- 1% of queries hit all shards (analytics, acceptable)
```

**Routing Logic:**
```python
def get_shard(city_id: int, order_date: datetime) -> int:
    # Calculate week of year (1-52)
    week = order_date.isocalendar()[1]
    
    # Combine city_id and week for shard key
    shard_key = f"{city_id}_{week}"
    
    # Consistent hash to shard number
    hash_value = hashlib.md5(shard_key.encode()).digest()
    shard_num = int.from_bytes(hash_value[:4], 'big') % NUM_SHARDS
    
    return shard_num

# Query: "Get San Francisco orders from this week"
shard = get_shard(city_id=5, order_date=datetime.now())
query_single_shard(shard, "SELECT * FROM orders WHERE city_id = 5 AND ...")

# Query: "Get San Francisco orders from last month"
# Need to query 4 shards (4 weeks)
shards = [get_shard(5, week) for week in last_4_weeks]
results = [query_shard(shard, "SELECT ...") for shard in shards]
merged_results = merge_and_sort(results)
```

**Shard Configuration:**
```
10M orders/day, 100 cities:
- Average: 100K orders/city/day
- Storage per city per week: 100K × 7 days × 2 KB = 1.4 GB
- Target shard size: 100 GB (manageable for single PostgreSQL server)
- Shards needed: 10M orders/day × 7 days × 2 KB / 100 GB = 16 shards
- Use 32 shards for 2x safety buffer

Shard Distribution Example:
Shard 1: SF-Week1, LA-Week3, Chicago-Week2     → 98 GB
Shard 2: NYC-Week1, Boston-Week4, Seattle-Week2 → 102 GB
Shard 3: ...
(Even distribution across shards)
```

**Handling Cross-Shard Queries:**
```python
# Most common: Customer order history (single customer, any city/week)
# Problem: Customer orders across multiple cities/weeks → multiple shards

Solution 1: Maintain secondary index in Elasticsearch
- All orders indexed by customer_id
- Query Elasticsearch for order IDs
- Fetch full details from shards

Solution 2: Separate customer_orders mapping table
CREATE TABLE customer_orders (
  customer_id BIGINT,
  order_id BIGINT,
  shard_id INT
);
- Query mapping table first
- Then query specific shards

Performance:
- Elasticsearch approach: 50ms (single ES query + parallel shard queries)
- Mapping table approach: 80ms (2 queries: mapping + shards)
- Both acceptable for customer order history page
```

**Rebalancing Strategy:**
```
When to add shards:
- Storage: Any shard >80% full (80 GB)
- Performance: Any shard >5,000 queries/sec

How to add shards:
1. Add new empty shards (32 → 64 shards)
2. New orders route to new shard distribution
3. Old data stays on old shards (no migration)
4. Queries check both old and new shards
5. After 1 year, migrate old data gradually

Cost:
- Avoid moving 10B old orders (expensive, risky)
- Accept querying more shards for historical data
- Only active orders (last 30 days) need fast access
```

**Interview Tip:** Explain that sharding is a last resort—scale vertically first (bigger servers), then add read replicas, and only shard when you have no other option. Sharding adds significant complexity (cross-shard queries, rebalancing, failure handling).

</details>

**Q5:** Compare ACID vs BASE consistency models. When would you choose strong consistency vs eventual consistency for different parts of a food delivery system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

ACID and BASE represent different consistency-availability trade-offs:

**ACID (Strong Consistency):**
- **Atomicity:** All or nothing (transaction succeeds completely or rolls back)
- **Consistency:** Data always in valid state (constraints enforced)
- **Isolation:** Concurrent transactions don't interfere
- **Durability:** Committed data never lost (survives crashes)

**BASE (Eventual Consistency):**
- **Basically Available:** System remains operational even during failures
- **Soft state:** Data may be in flux, not always consistent
- **Eventually consistent:** Data becomes consistent over time (seconds/minutes)

**Decision Matrix for Food Delivery:**

**1. Orders & Payments (ACID - Strong Consistency Required):**

**Why Strong Consistency?**
- Money involved—double-charging customer is unacceptable
- Customer expects order to appear immediately after payment
- Refunds must be accurate (can't refund order that doesn't exist)

**Example with PostgreSQL:**
```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
  -- 1. Create order
  INSERT INTO orders (customer_id, total_amount)
  VALUES (123, 42.50) RETURNING order_id;
  
  -- 2. Charge payment
  INSERT INTO payments (order_id, amount, status)
  VALUES (789, 42.50, 'CHARGED');
  
  -- If payment fails (card declined):
  ROLLBACK; -- Order never created, customer never charged
  
  -- If payment succeeds:
  COMMIT; -- Order and payment atomically committed
END TRANSACTION;

Result:
- Either both order and payment exist, or neither exists
- No partial state (order exists but payment failed)
- Customer immediately sees order in app after payment
```

**Cost of ACID:**
- Write latency: 50ms (vs 5ms for BASE)
- Write throughput: 10K/sec per server (vs 100K/sec for BASE)
- Can't partition data across regions (requires cross-datacenter coordination)

**Acceptable because:**
- Orders are 115/sec average (well within 10K/sec capacity)
- 50ms latency acceptable for order placement (one-time operation)
- Revenue protection worth the performance cost

**2. Driver Locations (BASE - Eventual Consistency Acceptable):**

**Why Eventual Consistency?**
- Location updates are continuous (200K updates/sec)
- Showing driver 100m away when actually 120m is acceptable
- Availability more important than perfect accuracy (map must always load)

**Example with Cassandra:**
```sql
-- Driver sends location every second
INSERT INTO driver_locations (driver_id, timestamp, lat, lng)
VALUES (12345, '2025-01-15 18:30:00', 37.7749, -122.4194);

-- Write succeeds in 5ms (no coordination across replicas)
-- Replicates to 2 other datacenters in background
-- San Francisco datacenter: 0ms delay
-- New York datacenter: 50ms delay (coast-to-coast)
-- London datacenter: 100ms delay (transatlantic)

Result:
- SF customers see location immediately
- NY customers see location 50ms old (driver moved 1 meter)
- London customers see location 100ms old (driver moved 2 meters)
- All acceptable for real-time tracking!
```

**Benefits of BASE:**
- Write throughput: 200K updates/sec (20x higher than ACID)
- Write latency: 5ms (10x faster than ACID)
- Geographic distribution: Can write to closest datacenter
- Availability: 99.99% (vs 99.9% for ACID)

**3. Restaurant Inventory (Strong Consistency with Optimistic Locking):**

**Why Strong Consistency?**
- Prevent overselling (accepting order for sold-out item)
- Customer trust issue (ordered burger, restaurant doesn't have it)

**But:**
- Don't need full ACID (menu changes infrequent)
- Can tolerate brief unavailability (restaurant waits 5 seconds)

**Solution: Optimistic Locking with PostgreSQL**
```sql
-- Menu item has version number
UPDATE menu_items
SET quantity = quantity - 1,
    version = version + 1
WHERE item_id = 789
  AND quantity >= 1
  AND version = 5;  -- Optimistic lock

-- If no rows updated (version changed by another request):
-- Retry transaction (someone else bought last item)

-- If row updated successfully:
-- Inventory decremented, order proceeds
```

**Performance:**
- 99% of time: Single query, 5ms latency (optimistic case)
- 1% of time: Retry needed, 20ms latency (contention on popular items)
- Average: 5.15ms (acceptable for order placement)

**4. Search Index (Eventual Consistency with CDC):**

**Why Eventual Consistency?**
- Search results don't need to be real-time (5-minute lag acceptable)
- Availability critical (search always available, even if slightly stale)
- Performance important (100ms search query target)

**Example with Elasticsearch + CDC:**
```
PostgreSQL (Source of Truth):
1. Restaurant adds menu item "Vegan Burger" at 6:00:00 PM
   INSERT INTO menu_items (...) VALUES (...);

2. Change Data Capture (Debezium) detects change
   Reads PostgreSQL write-ahead log (WAL)
   
3. Publishes event to Kafka at 6:00:01 PM (1 sec delay)
   { "event": "menu_item.created", "item_id": 789, ... }

4. Elasticsearch consumer processes event at 6:00:05 PM (5 sec delay)
   Indexes item in Elasticsearch

5. Customer searches "vegan burger" at 6:00:10 PM
   Sees new item in results (10 sec lag total)

Acceptable:
- 10-second lag is invisible to customers
- Search always available (Elasticsearch never blocks)
- PostgreSQL not slowed down by search indexing
```

**5. Analytics Data (Eventual Consistency with Batch Processing):**

**Why Eventual Consistency?**
- Analytics queries are not time-critical (30-minute lag acceptable)
- Don't impact real-time operations (run separate infrastructure)

**Example:**
```
Real-time Database (PostgreSQL):
- Orders, payments, driver locations

Analytics Database (Snowflake):
- Copy data via batch job every 15 minutes
- Run complex aggregations without impacting production

Business Dashboard Query:
"Total orders per city in last 7 days"
- Runs on Snowflake (not production database)
- Data 15 minutes old (acceptable for business metrics)
- Query takes 30 seconds (complex aggregation)
- Doesn't slow down order placement
```

**Decision Framework:**

```
Choose ACID (Strong Consistency) when:
✅ Money involved (orders, payments, refunds)
✅ Immediate accuracy required (inventory, pricing)
✅ Rare writes (100s per second, not 100Ks)
✅ Acceptable latency: 50-100ms

Choose BASE (Eventual Consistency) when:
✅ No money directly involved (locations, ratings, search)
✅ Slight staleness acceptable (5-60 second lag)
✅ High write volume (10,000s per second)
✅ Need low latency: <10ms
✅ Geographic distribution required (multi-region)
```

**Consistency Guarantees:**
```
Component          Consistency  Latency  Throughput  Trade-off
─────────────────────────────────────────────────────────────────
Orders             ACID         50ms     10K/sec     Correctness
Payments           ACID         50ms     10K/sec     No double-charge
Inventory          ACID+OCC     5-20ms   50K/sec     No overselling
Driver Locations   BASE         5ms      200K/sec    Availability
Ratings/Reviews    BASE         10ms     10K/sec     Not critical
Search Index       BASE         2ms      50K/sec     Performance
Analytics          BASE         N/A      Batch       Async processing
```

**Interview Tip:** Explain that you don't choose ACID vs BASE for entire system—use ACID where money is involved, BASE everywhere else. This hybrid approach balances correctness with performance.

</details>

#### Advanced Level

**Q6:** Design a multi-region database architecture for a global food delivery platform with strict data residency requirements (EU customer data must stay in EU, US data in US). How do you handle cross-region orders (EU customer orders while traveling in US)?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Multi-region data residency requires careful architecture to balance compliance, performance, and user experience:

**Regulatory Requirements:**

**GDPR (EU):**
- EU citizen data must be stored in EU datacenters
- Cannot transfer to US without explicit consent or adequacy decision
- Fines: 4% of global revenue or €20M (whichever is higher)

**CCPA (California):**
- California resident data has specific access/deletion requirements
- Less strict than GDPR on data location

**Other Regulations:**
- China: All data must stay within China borders
- Russia: Similar data localization laws

**Architecture Design:**

**1. User Home Region (Data Residency):**

Every user has a "home region" where their personal data lives:
```
User Registration:
- User signs up in Paris → home_region = "EU"
- All PII stored in EU datacenters:
  - Name, email, phone, address
  - Payment methods (credit cards)
  - Order history
  - Preferences

Database Configuration:
EU Region (Frankfurt):
  - PostgreSQL primary for EU users
  - Cassandra cluster for EU driver locations
  - Redis cache for EU session data

US Region (Oregon):
  - PostgreSQL primary for US users
  - Cassandra cluster for US driver locations
  - Redis cache for US session data

Asia Region (Singapore):
  - PostgreSQL primary for Asia users
  - Similar setup
```

**2. Cross-Region Access (Traveling Scenario):**

**Scenario:** EU customer travels to San Francisco, orders food

**Challenge:**
- Customer data in EU (Frankfurt datacenter)
- Restaurant data in US (Oregon datacenter)
- Driver location in US (Oregon datacenter)
- Order must reference all three

**Solution: Multi-Region Order with Data Minimization:**

```
Step 1: Customer Authentication
- Customer logs in from San Francisco
- Request routed to US region (geographically closest)
- US API Gateway checks Redis cache (miss)
- Makes authenticated request to EU region for user profile
- EU region returns minimal data:
  ✅ user_id, name (needed for order)
  ✅ payment_token (tokenized, not actual card number)
  ❌ Full address history (not needed)
  ❌ EU order history (not needed for US order)
- Cache result in US region for 1 hour

Latency: 150ms (cross-Atlantic round trip)

Step 2: Order Placement
- Create order in US region (local PostgreSQL)
orders_us:
  order_id: 789
  customer_id: 123 (reference to EU user)
  customer_home_region: "EU"
  restaurant_id: 456 (US restaurant)
  driver_id: 789 (US driver)
  order_region: "US"
  created_at: 2025-01-15 18:30:00
  
- Store minimal customer info in US order:
  delivery_name: "John D." (first name + initial)
  delivery_phone: "+1-415-555-XXXX"
  delivery_address: "123 Main St" (no EU address)

Latency: 50ms (local write)

Step 3: Cross-Region Sync (Asynchronous)
- After order completes, sync summary to EU region:
orders_summary_eu:
  order_id: 789
  customer_id: 123
  order_region: "US"
  order_date: 2025-01-15
  total_amount: 42.50
  status: "DELIVERED"
  
- EU user sees "1 order in United States" in order history
- Click for details → fetches from US region (on-demand)

Latency: 5 seconds (async sync, user doesn't wait)
```

**3. Compliance & Data Minimization:**

**Data Classification:**
```
PII (Personally Identifiable Information) - STRICT RESIDENCY:
- Full name, email, phone number
- Home address (EU addresses stay in EU)
- Payment information (credit card numbers)
- Biometric data (facial recognition for fraud)
→ NEVER replicated cross-region without consent

Non-PII Order Data - RELAXED RESIDENCY:
- Order items (what food was ordered)
- Restaurant details
- Delivery address (temporary, not home address)
- Order status and timestamps
→ Can be stored in order_region for operational needs

Aggregated Analytics - NO RESIDENCY REQUIREMENTS:
- City-level statistics ("San Francisco had 50K orders")
- Menu popularity (no customer linkage)
→ Can be processed globally
```

**4. User Experience Optimization:**

**Pre-Travel Preparation:**
```
Mobile App Detects Location Change:
1. User opens app in San Francisco (GPS coordinates)
2. App detects: home_region=EU, current_region=US
3. Background sync:
   - Download popular restaurants in SF (pre-cache)
   - Download delivery addresses used in US previously
   - Sync payment methods (tokenized only)
4. User ready to order (no cross-region latency)

Implementation:
if (user.home_region != detected_region):
    background_sync({
        'popular_restaurants': fetch_local(),
        'past_delivery_addresses': fetch_from_home_region(),
        'payment_tokens': fetch_from_home_region()
    })
```

**5. Failure Scenarios:**

**Scenario: EU region unavailable, EU customer in US wants to order**
```
Graceful Degradation:
1. Attempt to fetch user profile from EU (timeout after 5 seconds)
2. Fallback: Allow guest checkout in US
   - Collect delivery details locally
   - Create temporary user profile in US
   - Process order normally
3. After EU region recovers:
   - Merge temporary profile with main profile
   - Sync order history to EU
   
User Experience:
- Order still succeeds (availability)
- Slight inconvenience (re-enter address)
- Better than "service unavailable"
```

**6. Cross-Region Query Patterns:**

**Customer Support: "Show me all orders for user 123"**
```
Query Flow:
1. Identify user's home region: EU
2. Query EU region: Get order_ids in EU
3. Query all regions: Get order_ids referencing customer 123
4. Merge results:
   - 150 orders in EU (detailed data)
   - 3 orders in US (summary data)
   - 1 order in Asia (summary data)
5. Display consolidated view

Performance:
- Parallel queries to all regions: 200ms
- Acceptable for customer support tool (not customer-facing)
```

**7. Data Deletion (GDPR Right to be Forgotten):**
```
Customer Request: "Delete my data"

Deletion Process:
1. Mark user as deleted in home region (EU)
2. Propagate deletion event to all regions via Kafka
3. Each region:
   - Anonymize orders (replace name with "User123")
   - Remove PII (phone, email)
   - Keep order data for business records (legal requirement)
4. Verify deletion across all regions (audit trail)

Timeline:
- EU region: Deleted in 1 hour
- US region: Deleted in 24 hours (async propagation)
- All regions: Deleted within 30 days (GDPR requirement)
```

**8. Cost Optimization:**

```
Multi-Region Infrastructure Costs:
EU Region:
- PostgreSQL: 10 servers × $2K/month = $20K
- Cassandra: 20 servers × $1K/month = $20K
- Redis: 5 servers × $500/month = $2.5K
- Subtotal: $42.5K/month

US Region: $42.5K/month (similar scale)
Asia Region: $20K/month (smaller scale)

Total: $105K/month

Single Global Region Alternative:
- $50K/month (50% savings)
- But: GDPR non-compliance → $20M fine risk!
- Decision: Multi-region cost is insurance against fines
```

**Interview Tip:** Emphasize that data residency is about minimizing data transfer, not eliminating it—you can transfer minimal data (user_id, payment token) for operational needs, but full PII (addresses, credit cards) must stay in home region. Always design for "data minimization" principle.

</details>

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

### 🎯 Interview Questions - API Design

#### Beginner Level

**Q1:** What HTTP methods would you use for a food delivery API and what are the best practices for RESTful design?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

REST APIs use HTTP methods to represent CRUD operations with specific semantics:

**HTTP Methods & Use Cases:**

**GET (Read - Idempotent, Safe):**
- Retrieve data without modifying server state
- Can be cached
- Safe to retry (no side effects)

```
Examples:
GET /api/v1/restaurants?city=SF&cuisine=italian
→ Returns list of Italian restaurants in San Francisco
→ 200 OK with JSON array

GET /api/v1/restaurants/12345
→ Returns single restaurant details
→ 200 OK with JSON object
→ 404 Not Found if restaurant doesn't exist

GET /api/v1/orders?customer_id=789&status=active
→ Returns customer's active orders
→ 200 OK with paginated results

Best Practices:
✅ Use query parameters for filtering (?status=active)
✅ Use path parameters for resource IDs (/restaurants/12345)
✅ Return 200 OK for success, 404 for not found
✅ Cache-Control headers for performance (Cache-Control: max-age=300)
```

**POST (Create - Not Idempotent):**
- Create new resources
- Returns created resource with location header
- Not safe to retry (may create duplicates)

```
Example:
POST /api/v1/orders
Content-Type: application/json
X-Idempotency-Key: 7c9e8a2f-4b3d-11eb

Request Body:
{
  "restaurant_id": 12345,
  "items": [
    {"menu_item_id": 678, "quantity": 2},
    {"menu_item_id": 679, "quantity": 1}
  ],
  "delivery_address": "123 Main St",
  "payment_method_id": "pm_xyz"
}

Response: 201 Created
Location: /api/v1/orders/98765
{
  "order_id": 98765,
  "status": "CONFIRMED",
  "estimated_delivery": "2025-01-15T19:30:00Z",
  "total_amount": 42.50
}

Best Practices:
✅ Return 201 Created (not 200 OK)
✅ Include Location header with new resource URL
✅ Use idempotency keys to prevent duplicate creates
✅ Validate input (return 400 Bad Request for invalid data)
```

**PUT (Replace - Idempotent):**
- Replace entire resource
- Client provides complete resource representation
- Idempotent (multiple identical requests have same effect)

```
Example:
PUT /api/v1/restaurants/12345/menu-items/678
{
  "name": "Vegan Burger",
  "description": "Plant-based burger with avocado",
  "price": 12.99,
  "category": "Entrees",
  "available": true,
  "preparation_time": 15
}

→ Replaces entire menu item (all fields required)
→ 200 OK if updated, 404 if not found

Use Case:
- Complete updates (restaurant updates entire menu item)
- Client has full resource representation
```

**PATCH (Partial Update - Idempotent):**
- Update specific fields
- Client provides only changed fields
- More efficient than PUT

```
Example:
PATCH /api/v1/restaurants/12345/operating-hours
{
  "monday": {
    "open": "11:00",
    "close": "22:00"
  }
}

→ Updates only Monday hours (other days unchanged)
→ 200 OK with updated resource

Use Case:
- Partial updates (change one field without sending all fields)
- Mobile apps with limited bandwidth
```

**DELETE (Remove - Idempotent):**
- Delete resource
- Idempotent (deleting twice has same effect as deleting once)

```
Example:
DELETE /api/v1/orders/98765

→ 204 No Content (successful deletion, no response body)
→ 404 Not Found if already deleted (or never existed)
→ 409 Conflict if order can't be deleted (already delivered)

Use Case:
- Cancel order (if not yet preparing)
- Remove saved payment method
- Delete account
```

**Common Mistakes to Avoid:**

**❌ Using GET for state changes:**
```
BAD: GET /api/v1/orders/12345/cancel
→ GET should be read-only, use POST instead

GOOD: POST /api/v1/orders/12345/cancel
→ POST indicates state change
```

**❌ Returning wrong status codes:**
```
BAD: 200 OK when resource not found
GOOD: 404 Not Found

BAD: 200 OK for create operation
GOOD: 201 Created

BAD: 500 Internal Server Error for invalid input
GOOD: 400 Bad Request
```

**❌ Non-RESTful URLs:**
```
BAD: GET /api/v1/getRestaurantById?id=12345
GOOD: GET /api/v1/restaurants/12345

BAD: POST /api/v1/create-order
GOOD: POST /api/v1/orders

BAD: GET /api/v1/restaurants-in-city-SF
GOOD: GET /api/v1/restaurants?city=SF
```

**Status Code Cheat Sheet:**
```
2xx Success:
200 OK - Successful GET, PUT, PATCH, DELETE
201 Created - Successful POST (resource created)
204 No Content - Successful DELETE (no response body)

4xx Client Errors:
400 Bad Request - Invalid input (validation failed)
401 Unauthorized - Missing or invalid authentication
403 Forbidden - Authenticated but not authorized
404 Not Found - Resource doesn't exist
409 Conflict - Resource already exists or conflict
429 Too Many Requests - Rate limit exceeded

5xx Server Errors:
500 Internal Server Error - Unexpected server error
502 Bad Gateway - Upstream service failure
503 Service Unavailable - Temporary outage
504 Gateway Timeout - Upstream service timeout
```

**RESTful Design Principles:**
```
1. Use nouns, not verbs in URLs:
   ✅ /orders, /restaurants, /drivers
   ❌ /getOrder, /createRestaurant, /updateDriver

2. Use plural nouns for collections:
   ✅ /orders, /restaurants
   ❌ /order, /restaurant

3. Use hierarchy for relationships:
   ✅ /restaurants/12345/menu-items
   ❌ /menu-items?restaurant_id=12345 (acceptable for filtering)

4. Use query parameters for filtering, sorting, pagination:
   ✅ /orders?status=active&sort=created_at&page=2
   ❌ /orders/active/sorted-by-date/page-2

5. Version your API:
   ✅ /api/v1/orders, /api/v2/orders
   ❌ /api/orders (will break clients when you change it)
```

**Interview Tip:** Mention that REST is about predictability—developers should be able to guess API endpoints based on naming conventions. `/orders` obviously returns orders, `POST /orders` creates an order. Consistency matters more than perfect adherence to REST principles.

</details>

**Q2:** How would you design pagination for an API endpoint that returns a customer's order history (potentially 10,000+ orders)?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Pagination prevents sending massive payloads and enables efficient data retrieval:

**Pagination Strategies:**

**1. Offset-Based Pagination (Simple, but flawed at scale):**

```
Request:
GET /api/v1/orders?customer_id=789&limit=20&offset=0

Response:
{
  "orders": [...20 orders...],
  "pagination": {
    "total": 10500,
    "limit": 20,
    "offset": 0,
    "has_next": true
  }
}

Next Page:
GET /api/v1/orders?customer_id=789&limit=20&offset=20

How it works:
SQL: SELECT * FROM orders 
     WHERE customer_id = 789 
     ORDER BY created_at DESC 
     LIMIT 20 OFFSET 20;

Pros:
✅ Simple to implement
✅ Can jump to any page (page 1, page 50, page 100)
✅ Shows total count (useful for UI: "Showing 1-20 of 10,500")

Cons:
❌ Slow for large offsets (OFFSET 10000 scans 10,000 rows)
❌ Inconsistent results (new order inserted, pagination shifts)
❌ Database must count all rows for total (expensive)

Performance:
- Page 1 (offset=0): 50ms
- Page 50 (offset=1000): 200ms
- Page 500 (offset=10000): 2 seconds! (unusable)
```

**2. Cursor-Based Pagination (Recommended for scale):**

```
Request:
GET /api/v1/orders?customer_id=789&limit=20

Response:
{
  "orders": [
    {
      "order_id": 98765,
      "created_at": "2025-01-15T18:30:00Z",
      ...
    },
    ... 19 more orders ...
  ],
  "pagination": {
    "next_cursor": "eyJvcmRlcl9pZCI6OTg3NDUsImNyZWF0ZWRfYXQiOjE2NDc2ODI2MDB9",
    "has_next": true
  }
}

Next Page (use cursor from previous response):
GET /api/v1/orders?customer_id=789&limit=20&cursor=eyJvcmRlcl9pZCI6OTg3NDUsImNyZWF0ZWRfYXQiOjE2NDc2ODI2MDB9

How it works:
1. Cursor encodes last row's values (order_id, created_at)
2. Decode cursor: {"order_id": 98745, "created_at": 1647682600}
3. SQL: SELECT * FROM orders 
         WHERE customer_id = 789
           AND (created_at, order_id) < (1647682600, 98745)
         ORDER BY created_at DESC, order_id DESC
         LIMIT 20;

Cursor Encoding (Base64 JSON):
cursor = base64.encode(json.dumps({
    "order_id": 98745,
    "created_at": 1647682600
}))

Pros:
✅ Consistent performance (always ~50ms, regardless of page)
✅ Works with real-time data (new orders don't break pagination)
✅ No need to count total rows (faster)
✅ Database uses index efficiently

Cons:
❌ Can't jump to arbitrary page (no "go to page 50")
❌ Can't show total count easily
❌ Slightly more complex to implement

Performance:
- Every page: 50ms (constant time!)
- Index used: idx_orders_customer_time (customer_id, created_at, order_id)
```

**3. Hybrid Approach (Best of Both Worlds):**

**Use Offset for First 100 Orders (Recent Orders):**
```
GET /api/v1/orders?customer_id=789&limit=20&offset=0

→ Most customers view only first 2-3 pages (last 60 orders)
→ Offset pagination works fine for small offsets (offset < 100)
→ Shows total count: "You have 150 orders"
```

**Use Cursor for Older Orders (Archive):**
```
After page 5 (offset=100):
GET /api/v1/orders?customer_id=789&limit=20&cursor=abc123

→ Switch to cursor-based pagination
→ Hide total count (not needed for old orders)
→ "Load More" button instead of page numbers
```

**4. Optimizations for Large Result Sets:**

**Index-Only Scans:**
```sql
-- Query only needs columns in index (no table lookup)
CREATE INDEX idx_orders_customer_time 
ON orders(customer_id, created_at DESC, order_id, total_amount);

SELECT order_id, created_at, total_amount
FROM orders
WHERE customer_id = 789
ORDER BY created_at DESC
LIMIT 20;

→ PostgreSQL reads only index (no table scan)
→ 10x faster (5ms instead of 50ms)
```

**Caching Recent Orders:**
```python
# Cache customer's first page (most common query)
cache_key = f"orders:customer:{customer_id}:page:1"
cached_result = redis.get(cache_key)

if cached_result:
    return json.loads(cached_result)  # 1ms (cache hit)

# Cache miss - query database
orders = db.query("SELECT * FROM orders WHERE customer_id = ?", customer_id)
redis.setex(cache_key, 300, json.dumps(orders))  # Cache for 5 minutes
return orders

Cache Hit Rate:
- First page: 95% cache hit (customers check order status frequently)
- Other pages: 20% cache hit (rarely accessed)
```

**5. Real-World Example (Uber Eats):**

```
Customer Order History:
- Average customer: 50 orders total
- Use offset pagination (simple, fast for small result sets)

Restaurant Order Dashboard (1000+ orders/day):
- Use cursor pagination (large result sets)
- Show last 100 orders with offset (today's orders)
- Older orders use cursor (historical data)

Driver Delivery History (10,000+ deliveries):
- Pure cursor pagination (large result sets)
- No page numbers (infinite scroll)
- No total count (not needed)
```

**6. Mobile App Considerations:**

```
Mobile App Best Practices:
1. Smaller page sizes (limit=10 instead of 20)
   - Less data over cellular network
   - Faster response times

2. Infinite scroll instead of pagination
   - Better mobile UX (no clicking page numbers)
   - Use cursor-based pagination behind the scenes

3. Prefetch next page
   - When user scrolls to item 7/10, fetch next page
   - Seamless UX (no loading spinner)

Implementation:
GET /api/v1/orders?limit=10
→ Returns 10 orders + next_cursor

User scrolls to order 7:
GET /api/v1/orders?limit=10&cursor={next_cursor}
→ Prefetch next 10 orders

User reaches order 10:
→ Next page already loaded (instant!)
```

**Interview Tip:** Mention that most systems use offset pagination for simplicity, but cursor-based pagination is necessary at scale. Show you understand the trade-offs—offset is easier to implement and better for small result sets, cursor is more complex but essential for large datasets like customer order history.

</details>

**Q3:** Design an idempotency mechanism to prevent duplicate orders when a customer clicks "Place Order" twice due to network issues.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Idempotency ensures that multiple identical requests have the same effect as a single request:

**Problem Scenario:**
```
Timeline:
18:30:00 - Customer clicks "Place Order"
18:30:00 - Mobile app sends POST /api/v1/orders (Request 1)
18:30:03 - Network timeout (request still processing)
18:30:03 - Customer clicks "Place Order" again (frustrated)
18:30:03 - Mobile app sends POST /api/v1/orders (Request 2)
18:30:05 - Request 1 completes → Order created, customer charged $42.50
18:30:05 - Request 2 completes → Order created, customer charged $42.50
Result: Customer charged twice, received two orders! 😱
```

**Solution: Idempotency Keys**

**1. Client-Side Key Generation:**

```javascript
// Mobile app generates unique key per operation
const idempotencyKey = generateUUID(); // "7c9e8a2f-4b3d-11eb-b378-0242ac130002"

// Store key locally (prevents re-generating on retry)
localStorage.setItem('order_idempotency_key', idempotencyKey);

// Include key in request header
fetch('/api/v1/orders', {
  method: 'POST',
  headers: {
    'X-Idempotency-Key': idempotencyKey,
    'Authorization': 'Bearer ...'
  },
  body: JSON.stringify({
    restaurant_id: 12345,
    items: [...],
    total_amount: 42.50
  })
});
```

**2. Server-Side Idempotency Check:**

```python
from fastapi import FastAPI, Header
import redis
import json

app = FastAPI()
redis_client = redis.Redis()

@app.post("/api/v1/orders")
async def create_order(
    order_data: dict,
    idempotency_key: str = Header(None, alias="X-Idempotency-Key")
):
    # 1. Validate idempotency key present
    if not idempotency_key:
        return {"error": "X-Idempotency-Key header required"}, 400
    
    # 2. Check if request already processed
    cache_key = f"idempotency:{idempotency_key}"
    cached_result = redis_client.get(cache_key)
    
    if cached_result:
        # Request already processed - return cached result
        return json.loads(cached_result), 201
    
    # 3. Check if request currently processing (race condition)
    lock_key = f"idempotency_lock:{idempotency_key}"
    lock_acquired = redis_client.set(lock_key, "1", nx=True, ex=30)
    
    if not lock_acquired:
        # Another request with same key is processing
        # Wait and retry (or return 409 Conflict)
        time.sleep(0.5)
        cached_result = redis_client.get(cache_key)
        if cached_result:
            return json.loads(cached_result), 201
        return {"error": "Request already processing"}, 409
    
    try:
        # 4. Process order (first time with this key)
        order = create_order_in_database(order_data)
        charge_payment(order)
        notify_restaurant(order)
        
        # 5. Cache result for 24 hours
        result = {
            "order_id": order.id,
            "status": "CONFIRMED",
            "total_amount": order.total_amount
        }
        redis_client.setex(cache_key, 86400, json.dumps(result))
        
        return result, 201
    
    finally:
        # 6. Release lock
        redis_client.delete(lock_key)
```

**3. Request Flow with Idempotency:**

```
Request 1 (18:30:00):
1. Client generates key: "7c9e8a2f..."
2. POST /api/v1/orders with X-Idempotency-Key: "7c9e8a2f..."
3. Server checks Redis: Key not found
4. Server acquires lock: "idempotency_lock:7c9e8a2f..." = "1"
5. Server processes order: Creates order, charges payment
6. Server caches result: "idempotency:7c9e8a2f..." = {"order_id": 98765, ...}
7. Server releases lock
8. Server returns: 201 Created with order details

Request 2 (18:30:03, duplicate):
1. Client reuses same key: "7c9e8a2f..." (from localStorage)
2. POST /api/v1/orders with X-Idempotency-Key: "7c9e8a2f..."
3. Server checks Redis: Key found! (cached result)
4. Server returns cached result: 201 Created with same order details
5. No database write, no payment charge (idempotent!)

Result:
- Customer charged once: $42.50 ✅
- One order created: order_id 98765 ✅
- Both requests return same response ✅
```

**4. Cache TTL Strategy:**

```
Idempotency Cache Configuration:
- TTL: 24 hours (86400 seconds)
- Why 24 hours?
  - Long enough to handle any reasonable retry scenario
  - Short enough to not bloat Redis (old keys expire)
  - Matches typical payment authorization window

- Storage per key: ~500 bytes (order response JSON)
- Expected keys: 10M orders/day × 1.1 (10% duplicates) = 11M keys
- Storage: 11M × 500 bytes = 5.5 GB (fits in single Redis instance)

- After 24 hours:
  - Key expires from Redis
  - Same idempotency key can be reused (unlikely, new UUID generated)
  - Duplicate check no longer needed (order visible in UI)
```

**5. Edge Cases:**

**Partial Failure (Order created, payment failed):**
```python
try:
    order = create_order_in_database(order_data)  # Succeeds
    charge_payment(order)  # Fails (card declined)
except PaymentError:
    # Store failure in cache (prevent retry with different payment)
    error_result = {
        "error": "Payment declined",
        "order_id": order.id,
        "status": "PAYMENT_FAILED"
    }
    redis_client.setex(cache_key, 86400, json.dumps(error_result))
    return error_result, 402  # Payment Required
    
# Retry with same idempotency key → returns cached error
# Client must use NEW idempotency key with different payment method
```

**Distributed Systems (Multiple API Servers):**
```
Challenge:
- Request 1 hits Server A
- Request 2 hits Server B (load balancer distributes)
- Both servers check Redis simultaneously (race condition!)

Solution: Redis-based distributed lock
1. Server A acquires lock: SET lock_key "1" NX EX 30
2. Server B tries to acquire: SET returns null (lock exists)
3. Server B waits 500ms, checks cache, returns cached result
4. Server A completes, releases lock

Lock timeout: 30 seconds
- If server crashes, lock auto-expires
- Prevents deadlock
```

**6. Alternative Approaches:**

**Database Unique Constraint:**
```sql
CREATE TABLE orders (
    order_id BIGSERIAL PRIMARY KEY,
    idempotency_key UUID UNIQUE,  -- Enforces uniqueness
    customer_id BIGINT NOT NULL,
    ...
);

INSERT INTO orders (idempotency_key, customer_id, ...)
VALUES ('7c9e8a2f...', 789, ...);

-- Second insert with same key fails:
ERROR: duplicate key value violates unique constraint "orders_idempotency_key_key"

Pros:
✅ Database enforces idempotency (no race conditions)
✅ No Redis dependency

Cons:
❌ Creates order record even for duplicates (bloats database)
❌ Can't return cached response (need to query database)
❌ Transaction rollback on duplicate (expensive)
```

**7. Real-World Production Stats:**

```
Uber Eats Idempotency Metrics:
- 10M orders/day
- 1M duplicate requests (10% due to network retries)
- 500K prevented duplicates (5% true duplicates)
- 500K intentional retries (5% user clicking multiple times)

Redis Cache:
- 11M keys stored daily
- 5.5 GB memory usage
- 99.9% cache hit rate for duplicates
- <1ms cache lookup latency

Cost Savings:
- Prevented duplicate charges: 500K × $42.50 avg = $21.25M/day
- Support tickets avoided: 500K × 50% (would contact support) = 250K tickets
- Support cost saved: 250K × $5/ticket = $1.25M/day
- Redis cost: $500/month (0.001% of savings!)
```

**Interview Tip:** Explain that idempotency is not just about preventing duplicate orders—it's about providing a predictable API. Clients should be able to retry any request safely without worrying about side effects. This is critical for mobile apps with unreliable networks.

</details>

#### Intermediate Level

**Q4:** How would you implement rate limiting for a food delivery API to prevent abuse? What rate limits would you set for different types of clients (customer app, restaurant dashboard, driver app)?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Rate limiting protects APIs from abuse and ensures fair resource allocation:

**Rate Limiting Strategies:**

**1. Token Bucket Algorithm (Recommended):**

```
Concept:
- Bucket holds tokens (e.g., 100 tokens)
- Each request consumes 1 token
- Tokens refill at constant rate (e.g., 10 tokens/second)
- Request allowed if bucket has tokens, denied if empty

Example:
Bucket capacity: 100 tokens
Refill rate: 10 tokens/second

Timeline:
00:00 - Bucket starts with 100 tokens
00:01 - Customer makes 50 requests → 50 tokens consumed
00:02 - Customer makes 60 requests → 50 remaining + 10 refilled = 60 tokens consumed
00:03 - Customer makes 20 requests → 60 remaining + 10 refilled = 70 tokens, request allowed
00:04 - Customer makes 100 requests → Only 80 tokens available (70 + 10), 80 allowed, 20 denied

Benefits:
✅ Allows bursts (100 requests instantly if bucket full)
✅ Smooth rate limiting (gradual token refill)
✅ Simple to implement with Redis
```

**2. Redis-Based Implementation:**

```python
import redis
import time

redis_client = redis.Redis()

def check_rate_limit(client_id: str, max_tokens: int, refill_rate: int) -> bool:
    """
    client_id: Unique identifier (user_id, API key, IP address)
    max_tokens: Bucket capacity
    refill_rate: Tokens added per second
    """
    bucket_key = f"rate_limit:{client_id}"
    now = time.time()
    
    # Get current bucket state
    bucket = redis_client.hgetall(bucket_key)
    
    if not bucket:
        # First request - initialize bucket
        redis_client.hset(bucket_key, mapping={
            "tokens": max_tokens - 1,  # Consume 1 token
            "last_refill": now
        })
        redis_client.expire(bucket_key, 3600)  # Expire after 1 hour of inactivity
        return True
    
    # Calculate tokens to add based on time elapsed
    last_refill = float(bucket[b"last_refill"])
    tokens = float(bucket[b"tokens"])
    time_passed = now - last_refill
    tokens_to_add = time_passed * refill_rate
    
    # Refill bucket (capped at max_tokens)
    tokens = min(max_tokens, tokens + tokens_to_add)
    
    if tokens >= 1:
        # Allow request, consume token
        redis_client.hset(bucket_key, mapping={
            "tokens": tokens - 1,
            "last_refill": now
        })
        return True
    else:
        # Deny request, no tokens available
        return False

# Usage in API endpoint
@app.get("/api/v1/restaurants")
async def list_restaurants(user_id: str):
    if not check_rate_limit(user_id, max_tokens=100, refill_rate=10):
        return {"error": "Rate limit exceeded. Try again in 10 seconds."}, 429
    
    # Process request normally
    restaurants = query_restaurants()
    return restaurants
```

**3. Rate Limits by Client Type:**

**Customer App (Mobile/Web):**
```
Endpoint                    Rate Limit        Reasoning
──────────────────────────────────────────────────────────
GET /restaurants            100 req/min       Browsing restaurants
GET /restaurants/:id        200 req/min       Viewing menus (frequent)
POST /orders                10 req/min        Placing orders (rare)
GET /orders                 50 req/min        Checking order status
GET /orders/:id/status      100 req/min       Real-time tracking
WebSocket /orders/:id       1 connection/user Live updates

Implementation:
- Rate limit per user_id (authenticated users)
- Rate limit per IP (guest users browsing restaurants)
- Burst allowed for browsing (support fast scrolling)
- Strict limit for order placement (prevent spam)

Why these limits?
- 100 restaurants/min = 1.6 req/sec (fast scrolling through list)
- 10 orders/min = prevent accidental duplicate orders
- 200 menu views/min = customer comparing multiple restaurants
```

**Restaurant Dashboard (Web App):**
```
Endpoint                    Rate Limit        Reasoning
──────────────────────────────────────────────────────────
GET /orders/incoming        500 req/min       Dashboard polls every 3 sec
POST /orders/:id/accept     30 req/min        Accepting orders
POST /menu-items            20 req/min        Adding menu items
PATCH /menu-items/:id       50 req/min        Updating prices
WebSocket /orders/stream    1 connection/user Real-time orders

Implementation:
- Rate limit per restaurant_id
- Higher limits than customers (business critical)
- Separate limits for read vs write operations

Why these limits?
- 500 incoming orders/min = dashboard polls every 3 sec (120 polls/min) × 4 tabs
- 30 order accepts/min = busy restaurant during lunch rush
- Auto-refresh every 3 seconds requires higher GET limits
```

**Driver App (Mobile):**
```
Endpoint                    Rate Limit        Reasoning
──────────────────────────────────────────────────────────
POST /location-updates      Unlimited         Real-time tracking critical
GET /orders/available       200 req/min       Checking for new orders
POST /orders/:id/accept     20 req/min        Accepting deliveries
POST /orders/:id/pickup     10 req/min        Marking picked up
POST /orders/:id/deliver    10 req/min        Marking delivered
WebSocket /orders/assigned  1 connection/user Order notifications

Implementation:
- Rate limit per driver_id
- No limit on location updates (business critical!)
- Moderate limits on state changes

Why these limits?
- Location updates: 1/sec × 60 sec = 60/min (unlimited, critical for ETA)
- 200 available orders/min = driver aggressively checking for deliveries
- State changes limited to prevent fraud (fake deliveries)
```

**Public API (Partners/Integrations):**
```
Tier           Rate Limit      Cost        Use Case
─────────────────────────────────────────────────────────
Free           100 req/hour    $0          Testing, small projects
Starter        1,000 req/hour  $50/month   Small businesses
Professional   10,000 req/hour $500/month  Medium businesses
Enterprise     Custom          Custom      Large partners

Implementation:
- Rate limit per API key
- Different tiers based on payment plan
- Stricter limits than internal apps (prevent abuse)

Example API Key:
- Key: "sk_live_Abc123..."
- Tier: Professional
- Rate limit: 10,000 req/hour = 166 req/min = 2.7 req/sec
```

**4. Multi-Level Rate Limiting:**

```python
def check_multi_level_rate_limit(user_id: str, endpoint: str) -> tuple[bool, str]:
    """
    Apply multiple rate limit tiers:
    1. Global (all users): 100K req/sec (prevent DDoS)
    2. Per-user: 100 req/min (prevent abuse)
    3. Per-endpoint: 10 req/min (e.g., order placement)
    """
    
    # Level 1: Global rate limit (circuit breaker)
    if not check_rate_limit("global", max_tokens=100000, refill_rate=1666):
        return False, "System overloaded. Try again later."
    
    # Level 2: Per-user rate limit
    if not check_rate_limit(f"user:{user_id}", max_tokens=100, refill_rate=1.66):
        return False, "User rate limit exceeded. Max 100 req/min."
    
    # Level 3: Per-endpoint rate limit
    if endpoint == "POST /orders":
        if not check_rate_limit(f"user:{user_id}:orders", max_tokens=10, refill_rate=0.16):
            return False, "Order rate limit exceeded. Max 10 orders/min."
    
    return True, ""

# Usage
allowed, error_message = check_multi_level_rate_limit(user_id, "POST /orders")
if not allowed:
    return {"error": error_message}, 429
```

**5. Rate Limit Response Headers:**

```
HTTP Response Headers (Industry Standard):
HTTP/1.1 200 OK
X-RateLimit-Limit: 100         ← Max requests per window
X-RateLimit-Remaining: 73      ← Requests remaining
X-RateLimit-Reset: 1705342200  ← Unix timestamp when limit resets
Retry-After: 60                ← Seconds until retry allowed (if rate limited)

When rate limit exceeded:
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1705342200
Retry-After: 60
Content-Type: application/json

{
  "error": "Rate limit exceeded",
  "message": "You have made 100 requests in the last minute. Please try again in 60 seconds.",
  "retry_after": 60
}
```

**6. Adaptive Rate Limiting:**

```python
def get_adaptive_rate_limit(user_id: str) -> int:
    """
    Adjust rate limits based on user behavior:
    - New users: Lower limits (prevent bot signups)
    - Verified users: Normal limits
    - Premium users: Higher limits
    - Abusive users: Severely restricted
    """
    user = get_user(user_id)
    
    if user.is_premium:
        return 500  # 5x normal limit
    elif user.is_verified and user.account_age_days > 30:
        return 100  # Normal limit
    elif user.is_verified:
        return 50   # New but verified
    elif user.abuse_score > 0.8:
        return 10   # Suspected abuse
    else:
        return 25   # New unverified user
    
# Abuse detection
def calculate_abuse_score(user_id: str) -> float:
    """
    Score 0-1 based on suspicious behavior:
    - Failed payment attempts
    - Multiple account creations from same IP
    - Rapid order placements and cancellations
    - Unusual API usage patterns
    """
    user_behavior = get_user_behavior(user_id)
    
    score = 0.0
    score += user_behavior.failed_payments * 0.2
    score += user_behavior.duplicate_accounts * 0.3
    score += user_behavior.order_cancellations * 0.1
    
    return min(1.0, score)
```

**7. Production Metrics:**

```
Uber Eats Rate Limiting Stats:
- 1M active users
- 10M API requests/hour peak
- 500K rate limit denials/hour (5% of traffic)
- 90% of denials: Legitimate users (retry storms, bugs)
- 10% of denials: Malicious (DDoS, scraping, abuse)

Rate Limit Configuration:
- Redis Cluster: 10 nodes, 100 GB memory
- Rate limit keys: 5M active (1M users × 5 endpoints)
- Memory per key: 100 bytes
- Total memory: 500 MB (0.5% of Redis capacity)

Response Times:
- Rate limit check: 1ms (Redis lookup)
- Cache hit rate: 99.9% (keys cached in application memory)
- Overhead: <1% of request latency
```

**Interview Tip:** Explain that rate limiting is a balancing act—too strict and you frustrate legitimate users, too lenient and you allow abuse. Use tiered limits (browsing vs order placement), allow bursts (token bucket), and provide clear error messages with retry times. Monitor rate limit denials to detect bugs (legitimate traffic being blocked) vs abuse.

</details>

**Q5:** Design an API versioning strategy that allows you to introduce breaking changes (e.g., changing order status values from strings to enums) without impacting existing mobile app users.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

API versioning enables backward-compatible evolution while introducing necessary changes:

**Versioning Strategies:**

**1. URL-Based Versioning (Recommended for REST APIs):**

```
Current API (v1):
GET /api/v1/orders/12345

Response:
{
  "order_id": 12345,
  "status": "confirmed",  ← String value
  "created_at": "2025-01-15T18:30:00Z"
}

New API (v2):
GET /api/v2/orders/12345

Response:
{
  "order_id": 12345,
  "status": "CONFIRMED",  ← Enum value (uppercase)
  "status_code": 1,       ← Numeric code
  "created_at": "2025-01-15T18:30:00.000Z",  ← Millisecond precision
  "created_timestamp": 1705342200  ← Unix timestamp (additional field)
}

Benefits:
✅ Clear version in URL (easy to see which version used)
✅ Can run both versions simultaneously
✅ Easy to route based on URL prefix
✅ Clients explicitly opt into new version

Drawbacks:
❌ URL changes (clients must update endpoint)
❌ Multiple versions to maintain (technical debt)
```

**2. Header-Based Versioning:**

```
Request:
GET /api/orders/12345
Accept: application/vnd.ubereats.v1+json

Response:
{
  "order_id": 12345,
  "status": "confirmed"
}

New Version Request:
GET /api/orders/12345
Accept: application/vnd.ubereats.v2+json

Response:
{
  "order_id": 12345,
  "status": "CONFIRMED",
  "status_code": 1
}

Benefits:
✅ URL stays same (cleaner URLs)
✅ Follows HTTP content negotiation standard

Drawbacks:
❌ Version hidden in headers (harder to debug)
❌ Can't easily test in browser (need to set headers)
❌ Caching more complex (vary by header)
```

**3. Migration Strategy for Breaking Changes:**

**Scenario: Change order status from strings to enums**

**Phase 1: Add New Fields (v1 - Non-Breaking):**
```python
# Month 1: Support both formats in v1
@app.get("/api/v1/orders/{order_id}")
def get_order_v1(order_id: int):
    order = db.get_order(order_id)
    
    return {
        "order_id": order.id,
        "status": order.status.lower(),           # Old format: "confirmed"
        "status_enum": order.status.upper(),      # New format: "CONFIRMED"
        "status_code": order.status_to_code(),    # New format: 1
        # Both old and new fields present!
    }

# No clients break - old clients ignore new fields
# New clients can start using status_enum field
```

**Phase 2: Announce Deprecation (v1 - 3 months):**
```
API Response Headers:
HTTP/1.1 200 OK
Deprecation: true
Sunset: Wed, 15 Apr 2025 00:00:00 GMT  ← v1 shutdown date
Link: <https://api.ubereats.com/docs/v2>; rel="alternate"

Response Body:
{
  "order_id": 12345,
  "status": "confirmed",  ← Deprecated field
  "_deprecated_fields": {
    "status": "This field will be removed on 2025-04-15. Use status_enum instead."
  },
  "status_enum": "CONFIRMED",
  "status_code": 1
}

Email to API Users:
Subject: Action Required: API v1 Deprecation
Body: 
We are deprecating the "status" string field in favor of "status_enum" 
on April 15, 2025. Please update your integration to use the new field.

Migration Guide: https://docs.ubereats.com/migration/v1-to-v2
```

**Phase 3: Launch v2 API (Parallel Operation - 6 months):**
```python
# v1 API (legacy)
@app.get("/api/v1/orders/{order_id}")
def get_order_v1(order_id: int):
    order = db.get_order(order_id)
    return {
        "order_id": order.id,
        "status": order.status.lower(),  # Still supported
        "status_enum": order.status.upper(),
        "status_code": order.status_to_code()
    }

# v2 API (new)
@app.get("/api/v2/orders/{order_id}")
def get_order_v2(order_id: int):
    order = db.get_order(order_id)
    return {
        "order_id": order.id,
        "status": order.status.upper(),  # Only enum format
        "status_code": order.status_to_code(),
        "created_at": order.created_at.isoformat()
        # "status" string field removed
    }

# Both versions coexist:
# - Old mobile apps use /api/v1/
# - New mobile apps use /api/v2/
# - Both work simultaneously
```

**Phase 4: Monitor Adoption (6 months):**
```python
# Track v1 vs v2 usage
def track_api_version(version: str, endpoint: str):
    redis_client.incr(f"api_usage:{version}:{endpoint}")

# Metrics dashboard:
API Version Usage (Last 30 Days):
- v1: 2.5M requests (25%) ← Still significant
- v2: 7.5M requests (75%) ← Most clients migrated

Top v1 Clients (Laggards):
1. iOS app v2.1.3: 1M requests (old version)
2. Android app v2.0.1: 800K requests
3. Partner API key "pk_abc123": 500K requests

Actions:
- Force update iOS/Android apps (push notification)
- Contact partners still using v1
- Send final deprecation warning
```

**Phase 5: Sunset v1 (After 12 months):**
```python
# Month 12: Disable v1 API
@app.get("/api/v1/orders/{order_id}")
def get_order_v1(order_id: int):
    return {
        "error": "API v1 has been sunset",
        "message": "Please upgrade to v2: https://docs.ubereats.com/v2",
        "sunset_date": "2025-04-15"
    }, 410  # 410 Gone (permanent removal)

# Remaining v1 clients (< 1%) must upgrade
# Customer support handles edge cases
```

**4. Backward Compatibility Techniques:**

**Additive Changes (Non-Breaking):**
```
✅ Add new fields (old clients ignore them)
✅ Add new endpoints (old clients don't use them)
✅ Make required fields optional (provide defaults)
✅ Relax validation (accept more input)

Example:
Old: { "order_id": 123, "total_amount": 42.50 }
New: { "order_id": 123, "total_amount": 42.50, "tip_amount": 5.00 }
→ Old clients work (ignore tip_amount)
```

**Breaking Changes (Require New Version):**
```
❌ Remove fields (old clients break)
❌ Rename fields (old clients break)
❌ Change field types (string → enum)
❌ Change URL structure (/orders → /purchases)
❌ Stricter validation (reject previously accepted input)

Example:
Old: { "status": "confirmed" }
New: { "status": "CONFIRMED" }
→ Old clients break (expect lowercase)
→ Requires v2 API
```

**5. Code Organization for Multi-Version Support:**

```python
# Shared business logic (version-independent)
class OrderService:
    def get_order(self, order_id: int) -> Order:
        return db.query(Order).filter_by(id=order_id).first()

# Version-specific serializers
class OrderSerializerV1:
    def serialize(self, order: Order) -> dict:
        return {
            "order_id": order.id,
            "status": order.status.lower(),  # v1 format
            "created_at": order.created_at.isoformat()
        }

class OrderSerializerV2:
    def serialize(self, order: Order) -> dict:
        return {
            "order_id": order.id,
            "status": order.status.upper(),  # v2 format
            "status_code": order.status_to_code(),
            "created_at": order.created_at.isoformat(),
            "created_timestamp": int(order.created_at.timestamp())
        }

# Version-specific endpoints
@app.get("/api/v1/orders/{order_id}")
def get_order_v1(order_id: int):
    order = order_service.get_order(order_id)
    return OrderSerializerV1().serialize(order)

@app.get("/api/v2/orders/{order_id}")
def get_order_v2(order_id: int):
    order = order_service.get_order(order_id)  # Same business logic
    return OrderSerializerV2().serialize(order)  # Different format
```

**6. Mobile App Version Enforcement:**

```python
# Enforce minimum app version for API access
@app.before_request
def check_app_version():
    app_version = request.headers.get("X-App-Version")  # e.g., "3.2.1"
    
    if not app_version:
        return {"error": "X-App-Version header required"}, 400
    
    min_version = "3.0.0"  # Minimum supported version
    if version_compare(app_version, min_version) < 0:
        return {
            "error": "App version too old",
            "message": "Please update to the latest version",
            "min_version": min_version,
            "download_url": "https://apps.apple.com/ubereats"
        }, 426  # 426 Upgrade Required

# Force update for critical security fixes
if is_critical_security_fix_needed(app_version):
    return {
        "error": "Critical update required",
        "message": "This version has a security vulnerability",
        "force_update": true
    }, 426
```

**7. Versioning Best Practices:**

```
DO:
✅ Version from day 1 (/api/v1/ not /api/)
✅ Support versions for 12+ months
✅ Communicate deprecation 6+ months in advance
✅ Provide migration guides and code samples
✅ Monitor version adoption (which clients using v1 vs v2)
✅ Use semantic versioning for breaking changes (v1 → v2)

DON'T:
❌ Break v1 without warning (customer trust issue)
❌ Support too many versions (technical debt)
❌ Change behavior without version bump (sneaky breaking change)
❌ Deprecate too quickly (<6 months notice)
❌ Ignore clients stuck on old versions (support them)
```

**Interview Tip:** Emphasize that API versioning is about managing technical debt—every new version adds maintenance burden (2x codebases), but necessary for evolving the API. Use deprecation timelines (6-12 months), monitor adoption, and communicate clearly with clients. Show you understand the balance between innovation (new features) and stability (don't break existing clients).

</details>

#### Advanced Level

**Q6:** Design a WebSocket protocol for real-time order status updates that handles disconnections, reconnections, and ensures no status updates are missed even if the client is offline for 5 minutes.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Real-time updates via WebSocket require careful handling of network instability and message delivery guarantees:

**WebSocket Architecture:**

**1. Connection Establishment:**

```javascript
// Client (Mobile App)
class OrderStatusSocket {
  constructor(orderId) {
    this.orderId = orderId;
    this.ws = null;
    this.reconnectAttempts = 0;
    this.lastEventId = null;  // Track last received event
    this.connect();
  }
  
  connect() {
    // Include last event ID for resumption
    const resumeToken = this.lastEventId || 'none';
    const wsUrl = `wss://api.ubereats.com/ws/orders/${this.orderId}?resume_from=${resumeToken}`;
    
    this.ws = new WebSocket(wsUrl);
    
    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
      
      // Send authentication
      this.ws.send(JSON.stringify({
        type: 'auth',
        token: localStorage.getItem('jwt_token')
      }));
    };
    
    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.handleMessage(message);
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      this.reconnect();
    };
  }
  
  handleMessage(message) {
    // Update last event ID for resume capability
    if (message.event_id) {
      this.lastEventId = message.event_id;
      localStorage.setItem(`ws_last_event_${this.orderId}`, message.event_id);
    }
    
    switch (message.type) {
      case 'status_update':
        this.updateOrderStatus(message.data);
        break;
      case 'driver_location':
        this.updateDriverLocation(message.data);
        break;
      case 'eta_update':
        this.updateETA(message.data);
        break;
      case 'catchup':
        // Received missed events after reconnection
        message.events.forEach(event => this.handleMessage(event));
        break;
    }
  }
  
  reconnect() {
    this.reconnectAttempts++;
    const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);
    // Exponential backoff: 2s, 4s, 8s, 16s, 30s (max)
    
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
    setTimeout(() => this.connect(), delay);
  }
}

// Usage
const orderSocket = new OrderStatusSocket(12345);
```

**2. Server-Side WebSocket Handler:**

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
import redis
import json

app = FastAPI()
redis_client = redis.Redis()

class ConnectionManager:
    def __init__(self):
        self.active_connections: dict[int, list[WebSocket]] = {}
    
    async def connect(self, order_id: int, websocket: WebSocket, resume_from: str):
        await websocket.accept()
        
        # Add to active connections
        if order_id not in self.active_connections:
            self.active_connections[order_id] = []
        self.active_connections[order_id].append(websocket)
        
        # Send catchup events if resuming
        if resume_from != 'none':
            await self.send_catchup(websocket, order_id, resume_from)
    
    async def send_catchup(self, websocket: WebSocket, order_id: int, last_event_id: str):
        """Send all events since last_event_id (client was offline)"""
        # Fetch missed events from Redis (stored for 1 hour)
        events_key = f"order_events:{order_id}"
        stored_events = redis_client.lrange(events_key, 0, -1)
        
        # Filter events after last_event_id
        catchup_events = []
        found_last_event = False
        
        for event_json in stored_events:
            event = json.loads(event_json)
            if found_last_event:
                catchup_events.append(event)
            elif event['event_id'] == last_event_id:
                found_last_event = True
        
        # Send catchup message with all missed events
        if catchup_events:
            await websocket.send_json({
                'type': 'catchup',
                'events': catchup_events,
                'message': f'Sending {len(catchup_events)} missed events'
            })
    
    async def broadcast(self, order_id: int, message: dict):
        """Send message to all clients watching this order"""
        # Generate unique event ID
        event_id = f"{order_id}_{int(time.time() * 1000)}"
        message['event_id'] = event_id
        
        # Store event in Redis for catchup (expire after 1 hour)
        events_key = f"order_events:{order_id}"
        redis_client.rpush(events_key, json.dumps(message))
        redis_client.expire(events_key, 3600)
        
        # Send to all connected clients
        if order_id in self.active_connections:
            disconnected = []
            for websocket in self.active_connections[order_id]:
                try:
                    await websocket.send_json(message)
                except:
                    disconnected.append(websocket)
            
            # Remove disconnected clients
            for ws in disconnected:
                self.active_connections[order_id].remove(ws)

manager = ConnectionManager()

@app.websocket("/ws/orders/{order_id}")
async def order_status_websocket(
    websocket: WebSocket,
    order_id: int,
    resume_from: str = 'none'
):
    # Authenticate
    auth_message = await websocket.receive_json()
    if not verify_token(auth_message.get('token')):
        await websocket.close(code=1008, reason="Unauthorized")
        return
    
    # Connect
    await manager.connect(order_id, websocket, resume_from)
    
    try:
        # Keep connection alive
        while True:
            # Receive heartbeat/ping from client
            data = await websocket.receive_json()
            
            if data.get('type') == 'ping':
                await websocket.send_json({'type': 'pong'})
    
    except WebSocketDisconnect:
        # Client disconnected
        manager.active_connections[order_id].remove(websocket)
```

**3. Event Publishing (Order Status Changes):**

```python
# When order status changes (e.g., restaurant confirms order)
async def update_order_status(order_id: int, new_status: str):
    # Update database
    db.execute(
        "UPDATE orders SET status = ? WHERE order_id = ?",
        (new_status, order_id)
    )
    
    # Broadcast to all WebSocket clients watching this order
    await manager.broadcast(order_id, {
        'type': 'status_update',
        'data': {
            'order_id': order_id,
            'status': new_status,
            'status_code': status_to_code(new_status),
            'timestamp': datetime.utcnow().isoformat()
        }
    })
    
    # Fallback: Send push notification (if WebSocket disconnected)
    send_push_notification(order_id, f"Order {new_status}")
```

**4. Handling Disconnections:**

**Client Offline for 30 Seconds (Short Disconnection):**
```
Timeline:
18:30:00 - Client connected, watching order 12345
18:30:15 - Order status: CONFIRMED → PREPARING
           WebSocket sends event (event_id: "12345_1705342215000")
           Client receives event ✅
18:30:20 - Client loses network (elevator, tunnel)
           WebSocket disconnected
18:30:25 - Order status: PREPARING → READY
           WebSocket tries to send event (client offline)
           Event stored in Redis for catchup
18:30:30 - Client regains network
           WebSocket reconnects with resume_from=12345_1705342215000
18:30:31 - Server sends catchup event (PREPARING → READY)
           Client receives missed event ✅

Result: No events missed, seamless experience
```

**Client Offline for 5 Minutes (Extended Disconnection):**
```
Timeline:
18:30:00 - Client connected
18:30:15 - Order status: CONFIRMED → PREPARING (event sent)
18:31:00 - Client loses network (phone call, poor signal)
           WebSocket disconnected
18:31:30 - Order status: PREPARING → READY (event stored)
18:32:00 - Order status: READY → PICKED_UP (event stored)
18:32:30 - Driver location updates (50 events stored)
18:35:00 - Order status: PICKED_UP → DELIVERED (event stored)
18:36:00 - Client regains network (5 minutes offline)
           WebSocket reconnects with resume_from=last_event_id
18:36:01 - Server sends catchup: 53 events
           Client processes all events (fast-forward UI)

Redis Event Storage:
order_events:12345 = [
  {"event_id": "...", "type": "status_update", "status": "PREPARING"},
  {"event_id": "...", "type": "status_update", "status": "READY"},
  {"event_id": "...", "type": "status_update", "status": "PICKED_UP"},
  {"event_id": "...", "type": "driver_location", "lat": 37.7749, ...},
  ... 50 more driver location events ...
  {"event_id": "...", "type": "status_update", "status": "DELIVERED"}
]

TTL: 1 hour (events expire after 1 hour)
```

**5. Delivery Guarantees:**

**At-Least-Once Delivery:**
```
Problem: Network packet loss, client crash
Solution: Client tracks last_event_id, requests catchup on reconnect

Possible Issue: Client receives duplicate event
Example:
- Server sends event (event_id: "abc123")
- Client receives event but network drops before ACK
- Server retries, sends same event again
- Client receives duplicate

Mitigation: Client deduplicates by event_id
if (this.receivedEventIds.has(message.event_id)) {
    console.log('Duplicate event ignored');
    return;
}
this.receivedEventIds.add(message.event_id);
this.handleMessage(message);
```

**Ordered Delivery:**
```
Problem: Events arrive out of order (network reordering, multiple servers)
Solution: Include sequence numbers, client reorders

Example:
Event 1: status=CONFIRMED, seq=1
Event 3: status=READY, seq=3 (arrives first due to network)
Event 2: status=PREPARING, seq=2 (arrives second)

Client reorders:
1. Buffer events by sequence number
2. Process in order (seq=1, 2, 3)
3. Display correct status progression
```

**6. Scalability (100K Concurrent Connections):**

**WebSocket Server Farm:**
```
Load Balancer (Sticky Sessions):
- Hash by order_id → WebSocket server
- Same order always routed to same server
- 10 WebSocket servers, 10K connections each

Architecture:
Client → Load Balancer → WebSocket Server 1 (10K connections)
                      → WebSocket Server 2 (10K connections)
                      → WebSocket Server 3 (10K connections)
                      ... 10 servers total

Event Broadcasting:
- Order service publishes event to Redis Pub/Sub
- All WebSocket servers subscribe to Redis
- Each server broadcasts to its connected clients

Redis Pub/Sub:
PUBLISH order_events_12345 '{"status": "READY", ...}'
→ All 10 WebSocket servers receive event
→ Server 3 has clients for order 12345
→ Broadcasts to those clients
→ Other servers ignore (no clients for that order)
```

**7. Fallback Mechanisms:**

```
Priority 1: WebSocket (Real-Time)
- Instant delivery (<100ms latency)
- Best user experience

Priority 2: Push Notifications (Backup)
- If WebSocket disconnected >1 minute
- Delivery in 2-5 seconds
- Wakes app if backgrounded

Priority 3: Polling (Last Resort)
- App polls GET /orders/{id}/status every 30 seconds
- If WebSocket and push notifications fail
- Degrades gracefully

Implementation:
if (websocket_connected):
    # Use WebSocket
elif (push_notification_available):
    # Use push notification
else:
    # Fall back to polling
    setInterval(() => {
        fetch(`/api/v1/orders/${orderId}/status`)
    }, 30000);
```

**Interview Tip:** Emphasize that WebSocket is about managing real-world network conditions—connections drop constantly (elevators, tunnels, poor signal). Design for resilience with event storage (Redis), reconnection logic (exponential backoff), and delivery guarantees (at-least-once with deduplication). Always have fallbacks (push notifications, polling) for when WebSocket fails.

</details>

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

### 🎯 Interview Questions - Order Management & State Machine

#### Beginner Level

**Q1:** Why do we need a state machine for order management? Why not just update order status directly?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

A state machine enforces **business rules and prevents invalid operations**. Without it, you could accidentally mark an order as DELIVERED before it was PICKED_UP, or allow customers to cancel after driver already arrived.

**Problems Without State Machine:**

```text
Without State Machine (Direct Updates):
❌ Order marked DELIVERED without being PICKED_UP first
❌ Customer cancels after food is delivered
❌ Restaurant accepts order that was already cancelled
❌ Driver picks up order from wrong restaurant (order was reassigned)
❌ Payment charged twice (no idempotency)
```

**Benefits With State Machine:**

1. **Valid Transitions Only:** Order can only move from PREPARING → READY → PICKED_UP (not skip steps)
2. **Actor Validation:** Only restaurant can transition to PREPARING, only driver to PICKED_UP
3. **Business Logic Enforcement:** Can't cancel order after it's delivered (too late)
4. **Audit Trail:** Every transition logged with who made change and why
5. **Timeout Handling:** Automatic transitions if stuck (restaurant hasn't accepted in 5 min → auto-reject)

**Real-World Example (Uber Eats):**

```text
Order Timeline with State Machine:
00:00 - PLACED (customer)
00:05 - CONFIRMED (payment service)
00:30 - ACCEPTED (restaurant)
15:00 - READY (restaurant)
15:30 - PICKED_UP (driver)
24:00 - DELIVERED (driver)

If customer tries to cancel at 20:00 (during delivery):
→ State machine checks: current state = EN_ROUTE
→ Policy: Can't cancel after PICKED_UP
→ Response: "Sorry, your order is already on the way"
```

**Interview Tip:** Emphasize that state machines are about **enforcing business invariants**, not just tracking status. They prevent bugs that cause money loss (double refunds) or bad customer experience (claiming food was delivered when it wasn't).

</details>

**Q2:** What are the "terminal states" in the order state machine and why do they matter?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Terminal states** are states where an order's lifecycle ends—no further transitions are allowed. Once an order reaches a terminal state, it's "done" and immutable.

**The Three Terminal States:**

```text
1. COMPLETED (successful delivery)
   - Customer received food
   - Payment settled
   - All parties can rate each other
   - No further actions possible

2. CANCELLED (order cancelled before delivery)
   - Cancelled by customer, restaurant, or system
   - Refund processed (full or partial)
   - No food was delivered
   - Cannot be "un-cancelled"

3. REJECTED (restaurant refused order)
   - Restaurant declined due to capacity/inventory
   - Full refund to customer
   - No driver was ever assigned
   - Similar to CANCELLED but different actor
```

**Why Terminal States Matter:**

1. **Finality:** Once COMPLETED, can't transition back to PREPARING (prevents fraud—driver can't "un-deliver" food)
2. **Idempotency:** Multiple "complete order" API calls don't create duplicate state changes
3. **Payment Settlement:** Terminal states trigger final payment processing (charge customer, pay restaurant/driver)
4. **Data Archival:** Orders in terminal states can be moved to cold storage after 90 days
5. **Audit Compliance:** Terminal state means all actions logged and unchangeable

**State Transition Diagram:**

```text
PLACED → PAYMENT_PROCESSING → CONFIRMED → ACCEPTED → PREPARING → READY → ASSIGNED → PICKED_UP → EN_ROUTE → DELIVERED → COMPLETED ✓
   ↓                ↓              ↓          ↓          ↓         ↓        ↓         ↓          ↓
   └────────────────┴──────────────┴──────────┴──────────┴─────────┴────────┴──────────┴────→ CANCELLED ✓
                                                                                               
CONFIRMED ──────────────────────────────────────────────────────────────────────────────────→ REJECTED ✓

✓ = Terminal State (no further transitions)
```

**Real-World Impact:**

At Uber Eats scale (10M orders/day):
- **COMPLETED:** 8.5M orders/day (85% success rate)
- **CANCELLED:** 1.4M orders/day (14% cancel rate)
- **REJECTED:** 100K orders/day (1% rejection rate)

Without terminal state enforcement, you'd have chaos:
- Customers trying to cancel completed orders for free food
- Drivers trying to mark cancelled orders as delivered for payment
- Support agents accidentally reopening old disputes

**Interview Tip:** Mention that terminal states enable **database partitioning**—active orders in hot storage (SSD), completed orders moved to cold storage (cheaper S3/Glacier) after 30 days.

</details>

**Q3:** How do timeouts work in the order state machine? Give an example.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Timeouts** automatically transition orders when they're "stuck" in a state for too long, preventing orders from being forgotten and ensuring good customer experience.

**Timeout Configuration Table:**

```text
STATE               TIMEOUT      ACTION IF EXCEEDED
──────────────────────────────────────────────────────────────────────
CONFIRMED           5 minutes    Auto-reject, refund customer
READY               30 minutes   Reassign to different driver
PICKED_UP           1 hour       Alert support team, contact driver
```

**Detailed Example: Restaurant Timeout (CONFIRMED → REJECTED)**

```text
Scenario: Restaurant's tablet is off or restaurant is overwhelmed

Timeline:
00:00 - Order placed, payment successful → CONFIRMED
00:01 - System sends notification to restaurant (no response)
00:02 - System sends 2nd notification (still no response)
00:05 - ⏰ TIMEOUT! 5 minutes elapsed in CONFIRMED state

Automatic Actions:
1. Transition order to REJECTED state
2. Process full refund to customer ($35.00)
3. Notify customer: "Sorry, restaurant couldn't accept your order"
4. Suggest 3 alternative restaurants nearby
5. Log timeout incident for restaurant (affects their acceptance rate metric)
6. If restaurant has >20% timeout rate → send warning email

Customer Impact:
- Found out in 5 minutes instead of waiting 30+ minutes
- Got refund immediately
- Can order from another restaurant
- Total delay: 5 minutes vs 30+ minutes (much better!)
```

**Implementation (Background Job):**

```python
# Runs every 1 minute
def check_order_timeouts():
    current_time = datetime.utcnow()
    
    # Find orders stuck in CONFIRMED for >5 minutes
    stuck_orders = db.query("""
        SELECT * FROM orders 
        WHERE status = 'CONFIRMED' 
        AND updated_at < NOW() - INTERVAL '5 minutes'
    """)
    
    for order in stuck_orders:
        # Auto-reject the order
        transition_order_state(
            order_id=order.id,
            new_state='REJECTED',
            actor='system',
            reason='Restaurant timeout - no response within 5 minutes'
        )
        
        # Process refund
        refund_service.process_refund(order.id, amount=order.total)
        
        # Notify customer with alternatives
        notification_service.send(
            user_id=order.customer_id,
            message=f"Sorry, {order.restaurant_name} couldn't accept your order.",
            alternatives=find_similar_restaurants(order.restaurant_id)
        )
        
        # Track metrics
        metrics.increment('order.timeout.restaurant_unresponsive')
```

**Why These Specific Timeouts?**

- **5 minutes (CONFIRMED):** Customers are hungry NOW. If restaurant can't respond in 5 min, they're too busy or having issues. Better to fail fast.
  
- **30 minutes (READY):** Food quality degrades after sitting 30 min. Cold pizza/soggy fries = bad review. Reassign to another driver or cancel.

- **1 hour (PICKED_UP):** Normal delivery is 15-30 min. If taking >1 hour, something is seriously wrong (driver accident, wrong address). Need human intervention.

**Real-World Stats (DoorDash):**

- Restaurant timeout rate: 2% of orders (200K/day at 10M order scale)
- Driver no-show rate: 1% of orders (100K/day)
- Delayed delivery rate: 0.5% (50K/day)

**Interview Tip:** Explain that timeouts are a **trade-off between customer patience and system efficiency**. Too short (2 min) → many false positives (restaurant was about to accept). Too long (15 min) → terrible customer experience.

</details>

#### Intermediate Level

**Q4:** Explain the Saga pattern for distributed order placement. Why is it needed?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

The **Saga pattern** manages distributed transactions across multiple microservices when placing an order. Since we can't use a single database transaction spanning Order Service, Payment Service, Inventory Service, and Matching Service, we use sagas to ensure **all-or-nothing** completion with compensating transactions for rollback.

**The Problem Without Sagas:**

```text
Order Placement Involves 4 Services:
1. Order Service → Create order in database
2. Inventory Service → Reserve menu items at restaurant
3. Payment Service → Charge customer's credit card
4. Matching Service → Assign driver to order

What if Payment succeeds but Matching fails (no drivers available)?
→ Customer charged but food never delivered = BAD!
→ Need to undo previous steps (cancel order, unreserve inventory, refund payment)
```

**Saga Pattern Implementation:**

```text
Order Placement Saga (Choreography Style):

STEP 1: Create Order
  ├─ Action: Order Service creates order (status: PLACED)
  ├─ Success: Publish "OrderCreated" event → Kafka
  └─ Failure: Return error to customer, end saga

STEP 2: Reserve Inventory
  ├─ Action: Inventory Service reserves items (burger, fries marked as pending)
  ├─ Success: Publish "InventoryReserved" event → Kafka
  └─ Failure: Compensate → Cancel order, refund not needed (not charged yet)

STEP 3: Charge Payment
  ├─ Action: Payment Service charges customer's card ($35.00)
  ├─ Success: Publish "PaymentSuccessful" event → Kafka
  └─ Failure: Compensate → Unreserve inventory, cancel order

STEP 4: Assign Driver
  ├─ Action: Matching Service finds and assigns driver
  ├─ Success: Publish "DriverAssigned" event → Order complete!
  └─ Failure: Compensate → Refund payment, unreserve inventory, cancel order
```

**Compensating Transactions (Rollback):**

When a step fails, we execute compensating transactions to undo previous steps:

```python
class OrderPlacementSaga:
    def execute(self, order_request):
        saga_id = generate_uuid()
        completed_steps = []
        
        try:
            # Step 1: Create order
            order = order_service.create_order(order_request)
            completed_steps.append('create_order')
            
            # Step 2: Reserve inventory
            inventory_service.reserve_items(order.id, order.items)
            completed_steps.append('reserve_inventory')
            
            # Step 3: Charge payment
            payment = payment_service.charge(
                customer_id=order.customer_id,
                amount=order.total,
                idempotency_key=f"order_{order.id}"
            )
            completed_steps.append('charge_payment')
            
            # Step 4: Assign driver
            driver = matching_service.assign_driver(order.id)
            completed_steps.append('assign_driver')
            
            # Success! All steps completed
            return order
        
        except InventoryNotAvailable:
            # Compensate: Just cancel order (no payment charged yet)
            self.compensate_create_order(order.id)
            raise OrderPlacementFailed("Item not available")
        
        except PaymentDeclined:
            # Compensate: Unreserve inventory, cancel order
            self.compensate_reserve_inventory(order.id)
            self.compensate_create_order(order.id)
            raise OrderPlacementFailed("Payment declined")
        
        except NoDriverAvailable:
            # Compensate: Refund payment, unreserve inventory, cancel order
            self.compensate_charge_payment(payment.id)
            self.compensate_reserve_inventory(order.id)
            self.compensate_create_order(order.id)
            raise OrderPlacementFailed("No drivers available")
    
    def compensate_create_order(self, order_id):
        """Cancel order"""
        order_service.cancel_order(order_id, reason="saga_rollback")
    
    def compensate_reserve_inventory(self, order_id):
        """Release reserved inventory"""
        inventory_service.unreserve_items(order_id)
    
    def compensate_charge_payment(self, payment_id):
        """Refund payment"""
        payment_service.refund(payment_id, reason="order_placement_failed")
```

**Why Sagas vs 2-Phase Commit (2PC)?**

```text
2-Phase Commit (ACID):
✅ Strong consistency (all-or-nothing)
❌ Blocks resources during commit (low throughput)
❌ Single point of failure (coordinator)
❌ Not suitable for microservices

Saga Pattern (BASE):
✅ High throughput (async, non-blocking)
✅ No single point of failure
✅ Works well with microservices
❌ Eventual consistency (brief inconsistency during rollback)
❌ More complex to implement
```

**Real-World Example (Uber Eats):**

At 10M orders/day:
- **99% success rate:** 9.9M orders complete successfully
- **1% rollback rate:** 100K orders fail and need compensation
  - 40K fail at inventory check (40%)
  - 40K fail at payment (40%)
  - 20K fail at driver matching (20%)

**Saga Execution Time:**
- **Happy path:** 200ms (all steps succeed)
- **Rollback:** 500ms (need to undo 2-3 steps)

**Interview Tip:** Mention that sagas provide **semantic atomicity** (business-level all-or-nothing) rather than **technical atomicity** (database ACID). There's a brief window where partial state exists, but compensating transactions ensure eventual consistency.

</details>

**Q5:** How do you ensure idempotency in order state transitions?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Idempotency** ensures that calling the same state transition multiple times has the same effect as calling it once. This prevents duplicate charges, double cancellations, and other bugs caused by retries or network failures.

**The Problem Without Idempotency:**

```text
Scenario: Network timeout during order placement

Client calls: POST /orders/create
→ Server receives request
→ Server creates order, charges payment
→ Network hiccup: response gets lost
→ Client thinks request failed, retries
→ Server creates ANOTHER order, charges payment AGAIN
❌ Customer charged twice! ($35 × 2 = $70)
```

**Solution: Idempotency Keys**

Every critical operation includes a unique idempotency key (usually UUID) that prevents duplicate processing:

```python
def create_order(order_request, idempotency_key):
    """
    Create order with idempotency key to prevent duplicates
    """
    # Check if we've already processed this request
    existing_order = redis_client.get(f"idempotency:{idempotency_key}")
    
    if existing_order:
        # Already processed this request, return cached result
        return json.loads(existing_order)
    
    # Lock to prevent concurrent processing of same key
    lock = redis_client.lock(f"lock:{idempotency_key}", timeout=10)
    
    if not lock.acquire(blocking=False):
        # Another thread is processing this request, wait and retry
        time.sleep(0.1)
        return create_order(order_request, idempotency_key)
    
    try:
        # Process order creation
        order = db.create_order(
            customer_id=order_request.customer_id,
            restaurant_id=order_request.restaurant_id,
            items=order_request.items,
            total=order_request.total
        )
        
        # Charge payment with same idempotency key
        payment = payment_service.charge(
            customer_id=order_request.customer_id,
            amount=order.total,
            idempotency_key=idempotency_key  # Same key propagates to payment
        )
        
        # Cache result for 24 hours
        redis_client.setex(
            f"idempotency:{idempotency_key}",
            86400,  # 24 hours TTL
            json.dumps(order.to_dict())
        )
        
        return order
    
    finally:
        lock.release()
```

**State Transition Idempotency:**

```python
def transition_order_state(order_id, new_state, actor, idempotency_key):
    """
    Transition order state idempotently
    """
    # Generate deterministic idempotency key if not provided
    if not idempotency_key:
        idempotency_key = f"{order_id}:{new_state}:{actor}:{int(time.time() / 60)}"
        # Same minute = same key (prevents duplicate transitions in same minute)
    
    # Check if transition already completed
    transition_record = db.query(
        "SELECT * FROM order_state_transitions WHERE idempotency_key = ?",
        [idempotency_key]
    )
    
    if transition_record:
        # Already transitioned, return success (idempotent!)
        return {"status": "success", "already_processed": True}
    
    # Validate transition is allowed
    current_order = db.get_order(order_id)
    
    if current_order.status == new_state:
        # Already in target state, return success (idempotent!)
        return {"status": "success", "already_in_state": True}
    
    if new_state not in allowed_transitions[current_order.status]:
        # Invalid transition
        raise InvalidTransitionError(
            f"Cannot transition from {current_order.status} to {new_state}"
        )
    
    # Execute transition atomically
    with db.transaction():
        # Update order status
        db.update_order(order_id, status=new_state, updated_at=datetime.utcnow())
        
        # Record transition (idempotency key ensures uniqueness)
        db.insert_transition(
            order_id=order_id,
            from_state=current_order.status,
            to_state=new_state,
            actor=actor,
            idempotency_key=idempotency_key,
            timestamp=datetime.utcnow()
        )
    
    return {"status": "success", "transitioned": True}
```

**Database Schema for Idempotency:**

```sql
CREATE TABLE order_state_transitions (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT NOT NULL,
    from_state VARCHAR(50),
    to_state VARCHAR(50),
    actor VARCHAR(50),  -- 'customer', 'restaurant', 'driver', 'system'
    idempotency_key VARCHAR(255) UNIQUE,  -- Prevents duplicate transitions
    timestamp TIMESTAMP DEFAULT NOW(),
    
    INDEX idx_order_id (order_id),
    UNIQUE INDEX idx_idempotency (idempotency_key)  -- Enforces idempotency at DB level
);
```

**Idempotency Key Generation Strategies:**

```text
OPERATION            IDEMPOTENCY KEY FORMAT                    WHY?
─────────────────────────────────────────────────────────────────────────────────────
Order creation       {client_generated_uuid}                  Client generates UUID, sends with request
State transition     {order_id}:{new_state}:{timestamp_min}  Same minute = same transition
Payment              {order_id}:payment:{amount}              Prevents duplicate charges
Refund               {order_id}:refund:{reason}               Prevents duplicate refunds
Driver assignment    {order_id}:assign:{attempt_num}          Prevents double-assignment
```

**Real-World Impact (Stripe Example):**

Stripe (payment processor used by Uber Eats) enforces idempotency:
- Every API call accepts optional `Idempotency-Key` header
- If same key sent twice, Stripe returns cached response from first call
- Prevents duplicate charges even if client retries

```bash
# First request
curl -X POST https://api.stripe.com/v1/charges \
  -H "Idempotency-Key: abc123" \
  -d amount=3500 \
  -d currency=usd
# Response: {charge_id: "ch_xyz"}

# Retry (network timeout)
curl -X POST https://api.stripe.com/v1/charges \
  -H "Idempotency-Key: abc123" \
  -d amount=3500 \
  -d currency=usd
# Response: SAME charge {charge_id: "ch_xyz"} (not a new charge!)
```

**Interview Tip:** Emphasize that idempotency is critical for **money operations** (payments, refunds) and **state machines** (can't transition twice). It's the difference between charging a customer once vs twice ($35 vs $70).

</details>

#### Advanced Level

**Q6:** How would you handle concurrent state transitions from multiple services? Give a specific example with resolution.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Concurrent state transitions** occur when multiple services try to change an order's state simultaneously. This is common in distributed systems and requires careful locking and conflict resolution.

**Real-World Scenario: Race Condition**

```text
Timeline: Restaurant accepts order at same moment customer cancels

TIME    SERVICE              ACTION                           ORDER STATE
────────────────────────────────────────────────────────────────────────────
00:00   Order created                                         CONFIRMED
        
00:10   Restaurant Service   Read order (status: CONFIRMED)   CONFIRMED
00:10   Customer Service     Read order (status: CONFIRMED)   CONFIRMED
        
00:11   Restaurant Service   Validate transition OK           CONFIRMED
00:11   Customer Service     Validate transition OK           CONFIRMED
        
00:12   Restaurant Service   Write: status = ACCEPTED         ACCEPTED ✓
00:13   Customer Service     Write: status = CANCELLED        CANCELLED ✓
        
RESULT: Customer cancelled but restaurant started preparing food!
        Restaurant wasted time/ingredients, customer got charged.
        ❌ Lost $20 in food cost + bad experience
```

**Solution 1: Optimistic Locking with Version Numbers**

```python
class Order(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    status = db.Column(db.String(50))
    version = db.Column(db.Integer, default=1)  # Optimistic lock
    updated_at = db.Column(db.DateTime, default=datetime.utcnow)

def transition_order_state_safe(order_id, new_state, actor):
    """
    Transition with optimistic locking to prevent race conditions
    """
    max_retries = 3
    
    for attempt in range(max_retries):
        # Step 1: Read order with current version
        order = db.session.query(Order).filter_by(id=order_id).first()
        current_version = order.version
        current_state = order.status
        
        # Step 2: Validate transition is allowed
        if new_state not in allowed_transitions[current_state]:
            raise InvalidTransitionError(
                f"Cannot transition from {current_state} to {new_state}"
            )
        
        # Step 3: Try to update with version check (atomic!)
        result = db.session.execute(
            """
            UPDATE orders 
            SET status = :new_state,
                version = :new_version,
                updated_at = NOW()
            WHERE id = :order_id 
              AND version = :current_version
            """,
            {
                'new_state': new_state,
                'new_version': current_version + 1,
                'order_id': order_id,
                'current_version': current_version
            }
        )
        
        if result.rowcount == 1:
            # Success! Version matched, update applied
            db.session.commit()
            
            # Log transition
            log_state_transition(order_id, current_state, new_state, actor)
            
            return {"success": True, "new_version": current_version + 1}
        else:
            # Version mismatch = another service updated order
            db.session.rollback()
            
            if attempt < max_retries - 1:
                # Retry with exponential backoff
                time.sleep(0.1 * (2 ** attempt))  # 100ms, 200ms, 400ms
                continue
            else:
                # All retries failed
                raise ConcurrentModificationError(
                    f"Order {order_id} was modified by another service"
                )
```

**How Optimistic Locking Prevents Race Condition:**

```text
Corrected Timeline with Version Check:

TIME    SERVICE              ACTION                                    VERSION
────────────────────────────────────────────────────────────────────────────────
00:00   Order created                                                  v1
        
00:10   Restaurant Service   Read order (v1, status: CONFIRMED)       v1
00:10   Customer Service     Read order (v1, status: CONFIRMED)       v1
        
00:12   Restaurant Service   UPDATE orders SET status=ACCEPTED,        
                            version=2 WHERE id=123 AND version=1
                            → Success! (1 row updated)                 v2
        
00:13   Customer Service     UPDATE orders SET status=CANCELLED,
                            version=2 WHERE id=123 AND version=1
                            → Failed! (0 rows updated, version mismatch)
        
00:14   Customer Service     Retry: Read order (v2, status: ACCEPTED)
                            → Validate: Can't cancel ACCEPTED order 
                              within 2 minutes
                            → Return error to customer:
                              "Restaurant is already preparing your order"
                            
RESULT: ✅ Restaurant successfully accepted, customer can't cancel
        Conflict detected and resolved correctly!
```

**Solution 2: Distributed Lock with Redis**

```python
import redis
from contextlib import contextmanager

redis_client = redis.Redis(host='localhost', port=6379)

@contextmanager
def order_lock(order_id, timeout=5):
    """
    Distributed lock to prevent concurrent state transitions
    """
    lock_key = f"order_lock:{order_id}"
    lock_acquired = False
    
    try:
        # Try to acquire lock (non-blocking)
        lock_acquired = redis_client.set(
            lock_key,
            "locked",
            nx=True,  # Only set if doesn't exist
            ex=timeout  # Expire after 5 seconds (prevents deadlock)
        )
        
        if not lock_acquired:
            raise OrderLockedException(
                f"Order {order_id} is being modified by another service"
            )
        
        yield  # Execute protected code
        
    finally:
        if lock_acquired:
            # Release lock
            redis_client.delete(lock_key)

def transition_order_state_locked(order_id, new_state, actor):
    """
    Transition with distributed lock
    """
    with order_lock(order_id):
        # Only one service can execute this block at a time
        
        order = db.get_order(order_id)
        
        if new_state not in allowed_transitions[order.status]:
            raise InvalidTransitionError(
                f"Cannot transition from {order.status} to {new_state}"
            )
        
        # Update order (safe from concurrent modifications)
        db.update_order(
            order_id=order_id,
            status=new_state,
            updated_at=datetime.utcnow()
        )
        
        # Log transition
        log_state_transition(order_id, order.status, new_state, actor)
        
        return {"success": True}
```

**Comparison: Optimistic vs Pessimistic Locking**

```text
OPTIMISTIC LOCKING (Version Numbers):
✅ High throughput (no locks held)
✅ Better for read-heavy workloads
✅ No deadlock risk
❌ Requires retry logic
❌ Wasted work if conflict occurs

PESSIMISTIC LOCKING (Redis Distributed Lock):
✅ Guaranteed no conflicts
✅ Simpler logic (no retries)
❌ Lower throughput (lock contention)
❌ Deadlock risk if lock not released
❌ Single point of failure (Redis)

RECOMMENDATION: Use optimistic locking for order transitions
- Conflicts are rare (<1% of cases)
- High throughput needed (10M orders/day)
- Retries are cheap (just re-read and re-validate)
```

**Real-World Stats (Uber Eats):**

At 10M orders/day with optimistic locking:
- **99.7% transactions succeed on first try** (9.97M)
- **0.3% need retry** (30K conflicts detected and retried)
  - 90% succeed on 2nd try (27K)
  - 9% succeed on 3rd try (2.7K)
  - 1% fail after 3 retries (300 orders → escalate to support)

**Performance Impact:**
- **Optimistic locking:** 5ms per transition (no lock overhead)
- **Pessimistic locking:** 15ms per transition (Redis RTT + lock acquisition)
- **Scale impact:** 10ms × 10M orders = 28 hours of extra CPU time/day (!!)

**Interview Tip:** Explain that optimistic locking is preferred for **low-conflict scenarios** (order state transitions), while pessimistic locking is better for **high-conflict scenarios** (limited inventory, concert ticket sales). Food delivery has low conflict because each order is independent.

</details>

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

### 🎯 Interview Questions - Real-Time Driver Matching

#### Beginner Level

**Q1:** How do you find drivers within a 5km radius of a restaurant? Why not just scan all drivers?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

We use **geospatial indexing** (specifically Redis GEORADIUS) to find nearby drivers in under 100ms. Scanning all drivers would be impossibly slow at scale.

**The Naive Approach (Why It Doesn't Work):**

```text
For each driver in database (1 million drivers):
  1. Calculate distance from driver to restaurant (Haversine formula)
  2. If distance < 5km, add to results
  
Time Complexity: O(N) where N = total drivers
Calculation time per driver: 1ms (Haversine + DB lookup)
Total time: 1,000,000 drivers × 1ms = 1,000 seconds = 16 minutes!
❌ Way too slow (need <100ms for good UX)
```

**The Optimized Approach (Redis GEORADIUS):**

Redis uses geohashing to organize drivers by location in a sorted set. Nearby drivers have similar geohash prefixes, enabling fast range queries.

```text
Step 1: Store driver locations in Redis
─────────────────────────────────────────────────
Command: GEOADD driver_locations -73.9900 40.7500 driver:12345
         GEOADD driver_locations -73.9857 40.7484 driver:67890
         
Redis stores:
- Key: "driver_locations" (geospatial index)
- Members: driver:12345, driver:67890, ...
- Scores: Geohash-encoded coordinates

Step 2: Query for drivers within 5km radius
─────────────────────────────────────────────────
Command: GEORADIUS driver_locations -73.9900 40.7500 5 km WITHDIST ASC
         
Redis returns (in <10ms):
- driver:12345 (0.8 km away)
- driver:67890 (1.2 km away)
- driver:33333 (3.5 km away)
- driver:44444 (4.7 km away)

Step 3: Filter by availability
─────────────────────────────────────────────────
For each returned driver:
  Load from PostgreSQL (is_online=true, current_order_id=null)
  
Final result: 2-3 available drivers within 5km
Total time: 10ms (Redis) + 20ms (PostgreSQL) = 30ms ✅
```

**How Geohashing Works (Simplified):**

```text
Geohash divides the world into a grid and assigns each cell a unique string:

World
├─ North America (prefix: "d")
│  ├─ United States (prefix: "dr")
│  │  ├─ New York (prefix: "dr5")
│  │  │  ├─ Manhattan (prefix: "dr5r")
│  │  │  │  ├─ Times Square area (prefix: "dr5reg")
│  │  │  │  │  ├─ Exact location: "dr5regw3p6h9"

Nearby locations share prefixes:
- Times Square:  "dr5regw3p6h9"
- 2 blocks away: "dr5regw3p6xx" (shares "dr5regw3p6")
- London, UK:    "gcpvj0d" (completely different)

To find nearby drivers:
1. Hash restaurant location → "dr5regw"
2. Find all drivers with similar prefix → O(log N) lookup
3. Filter by exact distance → Only check ~20 candidates vs 1M
```

**Real-World Performance (Uber Eats NYC):**

```text
METRIC                          NAIVE SCAN    REDIS GEORADIUS
──────────────────────────────────────────────────────────────────
Drivers in system               1,000,000     1,000,000
Drivers within 5km (avg)        50            50
Query time                      16 minutes    8ms
Candidates checked              1,000,000     50
Memory usage                    0             ~200 MB (all drivers)
Scalability                     ❌            ✅
```

**Why Redis for Geospatial?**

1. **In-memory speed:** All data in RAM → <10ms queries
2. **Built-in geohashing:** Don't need to implement complex spatial indexing
3. **Atomic updates:** Driver location updates are thread-safe
4. **Persistence:** Optional AOF/RDB for crash recovery
5. **Replication:** Redis Cluster for high availability

**Interview Tip:** Mention that PostGIS (PostgreSQL extension) is an alternative with geospatial support, but Redis is faster for real-time queries because it's in-memory. PostGIS is better for complex spatial queries (polygons, routing) while Redis excels at simple radius searches.

</details>

**Q2:** What factors should you consider when scoring drivers for assignment beyond just distance?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Choosing the **closest** driver isn't always optimal. We need a **multi-factor scoring algorithm** that balances customer experience, driver fairness, and platform efficiency.

**The Six Key Scoring Factors:**

```text
1. DISTANCE (Weight: 40%)
   ├─ Why: Faster pickup = fresher food, happier customer
   ├─ Formula: score = 1 / (1 + distance_km)
   └─ Example: 0.8km → score = 0.556, 3km → score = 0.250

2. DRIVER RATING (Weight: 20%)
   ├─ Why: High-rated drivers provide better service
   ├─ Formula: score = rating / 5.0
   └─ Example: 4.8★ → score = 0.96, 4.0★ → score = 0.80

3. ACCEPTANCE RATE (Weight: 15%)
   ├─ Why: Drivers who often reject waste time
   ├─ Formula: score = acceptance_rate
   └─ Example: 95% → score = 0.95, 70% → score = 0.70

4. VEHICLE TYPE (Weight: 10%)
   ├─ Why: Bikes faster in traffic, cars needed for bulk
   ├─ Formula: score = 1.0 if match, 0.7 if mismatch
   └─ Example: Small order + bike = 1.0, Large order + bike = 0.7

5. EARNINGS FAIRNESS (Weight: 10%)
   ├─ Why: Spread orders evenly among drivers
   ├─ Formula: score = 1.0 if below average, 0.5 if above
   └─ Example: Earned $80 (avg $100) → 1.0, Earned $180 → 0.5

6. IDLE TIME (Weight: 5%)
   ├─ Why: Reward drivers waiting long, prevent starvation
   ├─ Formula: score = min(1.0, idle_minutes / 30)
   └─ Example: Idle 45 min → 1.0, Idle 10 min → 0.33
```

**Concrete Example: Driver A vs Driver B**

```text
Restaurant Location: Joe's Pizza (40.7500°N, -73.9900°W)
Order: 2 burgers, 1 fries (small order, fits in bike bag)

DRIVER A:
├─ Distance: 0.8 km (2 min away)
│  → 1/(1+0.8) = 0.556 × 40% = 0.222
├─ Rating: 4.8★ / 5.0
│  → 0.960 × 20% = 0.192
├─ Acceptance: 95% (reliable)
│  → 0.950 × 15% = 0.143
├─ Vehicle: Bike (matches small order)
│  → 1.000 × 10% = 0.100
├─ Earnings: $80 today (below $100 average)
│  → 1.000 × 10% = 0.100
└─ Idle: 45 minutes (waiting long time)
   → 1.000 × 5% = 0.050
TOTAL SCORE: 0.807

DRIVER B:
├─ Distance: 1.5 km (4 min away)
│  → 1/(1+1.5) = 0.400 × 40% = 0.160
├─ Rating: 4.9★ / 5.0 (excellent!)
│  → 0.980 × 20% = 0.196
├─ Acceptance: 92%
│  → 0.920 × 15% = 0.138
├─ Vehicle: Car (overkill for small order)
│  → 0.700 × 10% = 0.070
├─ Earnings: $180 today (well above average)
│  → 0.500 × 10% = 0.050
└─ Idle: 10 minutes (recently active)
   → 0.333 × 5% = 0.017
TOTAL SCORE: 0.631

RESULT: Assign to Driver A (0.807 > 0.631)
Reasoning: Even though Driver B has better rating, Driver A is 
closer, has matching vehicle type, and hasn't earned much today.
```

**Why These Specific Weights?**

```text
WEIGHT   FACTOR            REASONING
────────────────────────────────────────────────────────────────────
40%      Distance          Most important for food freshness & ETA
                           Customer ordered because hungry NOW
                           
20%      Rating            High-rated drivers → better reviews
                           4.9★ driver worth going bit farther for
                           
15%      Acceptance        Driver rejecting 50% of time wastes 30 sec
                           Better to skip them initially
                           
10%      Vehicle Type      Bike vs car makes 3-5 min difference in city
                           But not critical if other factors strong
                           
10%      Earnings Fairness Driver retention strategy
                           Prevent "rich get richer" effect
                           
5%       Idle Time         Tie-breaker for otherwise equal drivers
                           Small weight prevents starvation
```

**A/B Testing Different Weights:**

Uber Eats constantly tests weight configurations:

```text
EXPERIMENT: Does higher weight on rating improve satisfaction?

Control Group (original weights):
- Distance: 40%, Rating: 20%
- Customer satisfaction: 4.6/5.0
- Average delivery time: 28 minutes

Test Group (rating emphasis):
- Distance: 30%, Rating: 30%
- Customer satisfaction: 4.65/5.0 (+1.1% ✅)
- Average delivery time: 32 minutes (+4 min ❌)

RESULT: Keep original weights (time more important than rating)
Customers prefer fast delivery over slightly better-rated driver
```

**Interview Tip:** Emphasize that these weights are **city-specific and dynamic**. Dense cities (Manhattan) might weight distance at 50% because traffic matters more. Suburban areas might weight acceptance rate higher because fewer drivers means rejections are costly.

</details>

**Q3:** What happens if all nearby drivers reject an order? How do you handle this gracefully?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

When drivers reject orders, we use a **multi-stage escalation strategy** with expanding radius, increased incentives, and eventually graceful failure with customer communication.

**The Assignment Retry Flow:**

```text
ATTEMPT 1: Try top 3 drivers within 5km radius (sequential, 30 sec timeout each)
├─ Driver #1 (0.8km, score 0.807) → REJECTED (in 10 sec)
├─ Driver #2 (1.5km, score 0.631) → TIMEOUT (no response in 30 sec)
└─ Driver #3 (2.1km, score 0.590) → REJECTED (in 5 sec)
Total time: 45 seconds
❌ No driver assigned

ATTEMPT 2: Expand radius to 10km, increase surge pricing +15%
├─ Find 5 more drivers within 10km
├─ Notify customer: "Finding the best driver for you..."
├─ Try top 3 drivers (30 sec timeout each)
└─ Driver #4 (6.2km, score 0.450) → ACCEPTED ✅
Total time: 45 sec (attempt 1) + 30 sec (attempt 2) = 75 seconds
✅ Driver assigned (acceptable, under 2 min target)

ATTEMPT 3 (if needed): Expand to 15km, increase surge +30%
├─ Find drivers up to 15km away
├─ Notify customer: "Still looking... May take a few extra minutes"
├─ Try all available drivers
└─ If still no acceptance → ESCALATE

ESCALATION: After 3 minutes, no drivers
├─ Offer customer choice:
│  ├─ Option A: Wait 10 more minutes with $5 credit
│  ├─ Option B: Cancel for full refund
│  └─ Option C: Pick up order yourself (no delivery fee)
└─ Alert operations team to investigate driver shortage
```

**Detailed Implementation:**

```python
def assign_driver_with_retries(order_id, max_attempts=3):
    """
    Try to assign driver with expanding radius and increasing incentives
    """
    radii = [5, 10, 15]  # km
    surge_multipliers = [1.0, 1.15, 1.30]  # 0%, 15%, 30% increase
    
    for attempt in range(max_attempts):
        radius_km = radii[attempt]
        surge = surge_multipliers[attempt]
        
        # Find nearby drivers within current radius
        nearby_drivers = find_nearby_drivers(
            order_id=order_id,
            radius_km=radius_km
        )
        
        if len(nearby_drivers) == 0:
            # No drivers in radius
            if attempt < max_attempts - 1:
                # Try next attempt with larger radius
                notify_customer(
                    order_id=order_id,
                    message=f"Expanding search radius to {radii[attempt+1]}km..."
                )
                continue
            else:
                # All attempts exhausted
                return handle_no_drivers_available(order_id)
        
        # Score and sort drivers
        scored_drivers = score_and_sort_drivers(nearby_drivers, order_id)
        
        # Update delivery fee with surge pricing
        if surge > 1.0:
            update_delivery_fee(order_id, multiplier=surge)
            notify_customer(
                order_id=order_id,
                message=f"High demand - delivery fee adjusted to ${calculate_fee(order_id)}"
            )
        
        # Try offering to top 3 drivers
        for i, driver in enumerate(scored_drivers[:3]):
            response = offer_order_to_driver(
                order_id=order_id,
                driver_id=driver['id'],
                timeout_seconds=30,
                incentive=surge  # Show higher payout to driver
            )
            
            if response == 'ACCEPTED':
                # Success!
                log_assignment_success(
                    order_id=order_id,
                    driver_id=driver['id'],
                    attempt_number=attempt + 1,
                    time_to_assign=(datetime.utcnow() - order.created_at).seconds
                )
                return driver['id']
            
            elif response == 'REJECTED':
                # Driver manually rejected, try next
                log_driver_rejection(
                    order_id=order_id,
                    driver_id=driver['id'],
                    rejection_type='manual'
                )
                # Penalize driver's acceptance rate
                update_driver_acceptance_rate(driver['id'], accepted=False)
                continue
            
            elif response == 'TIMEOUT':
                # Driver didn't respond, try next
                log_driver_rejection(
                    order_id=order_id,
                    driver_id=driver['id'],
                    rejection_type='timeout'
                )
                continue
        
        # All drivers in this attempt rejected
        if attempt < max_attempts - 1:
            # Try next attempt with larger radius
            notify_customer(
                order_id=order_id,
                message="Still finding the best driver for you..."
            )
            time.sleep(5)  # Brief pause before escalation
    
    # All attempts failed
    return handle_no_drivers_available(order_id)

def handle_no_drivers_available(order_id):
    """
    Graceful failure when no drivers available after all attempts
    """
    order = db.get_order(order_id)
    
    # Option 1: Offer to wait with incentive
    offer_wait_with_credit(
        order_id=order_id,
        credit_amount=5.00,
        estimated_wait=10  # minutes
    )
    
    # Option 2: Allow cancellation with full refund
    allow_cancellation(
        order_id=order_id,
        refund_amount=order.total
    )
    
    # Option 3: Suggest pickup
    suggest_pickup(
        order_id=order_id,
        new_total=order.subtotal  # Remove delivery fee
    )
    
    # Alert operations team
    alert_ops_team(
        severity='HIGH',
        issue='No drivers available',
        order_id=order_id,
        restaurant_location=order.restaurant.location,
        time=datetime.utcnow()
    )
    
    # Update metrics
    metrics.increment('order.assignment.failed')
    
    return None
```

**Why Sequential (Not Parallel) Offers?**

```text
PARALLEL (Offer to 3 drivers at once):
✅ Faster assignment (30 sec instead of 90 sec)
❌ CONFLICT: What if 2 drivers both accept?
   → Need conflict resolution logic
   → One driver gets cancelled (bad experience)
   → Wasted time for rejected driver

SEQUENTIAL (Offer one at a time):
✅ No conflicts (only 1 driver can accept)
✅ Simpler logic
❌ Slower (30 sec × 3 drivers = 90 sec worst case)

HYBRID (Uber's Approach):
├─ Try #1 driver, wait 15 seconds
├─ If timeout, simultaneously offer to #2 and #3
└─ First to accept wins (low conflict probability)
```

**Real-World Stats (DoorDash):**

At 10M orders/day:
- **85% assigned on attempt 1** (8.5M orders, <30 sec)
- **12% assigned on attempt 2** (1.2M orders, 30-90 sec)
- **2% assigned on attempt 3** (200K orders, 90+ sec)
- **1% failed after 3 attempts** (100K orders, escalated)

**Failure Causes:**
- 40% (40K): High demand, not enough drivers online
- 30% (30K): Restaurant in remote area (no nearby drivers)
- 20% (20K): Bad weather (drivers going offline)
- 10% (10K): Technical issues (app crashes, network problems)

**Interview Tip:** Explain that the key is **transparent customer communication**. Don't leave customer wondering—tell them "We're expanding our search" or "High demand in your area, thanks for patience." Providing ETA for assignment (vs just ETA for delivery) builds trust.

</details>

#### Intermediate Level

**Q4:** How does Redis GEORADIUS work under the hood? What's the time complexity?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

Redis GEORADIUS uses **geohashing** to convert 2D coordinates (lat/lon) into a 1D sorted set, enabling fast O(log N + M) range queries where N is total drivers and M is results returned.

**Geohash Encoding (How It Works):**

```text
Step 1: Convert latitude/longitude to binary grid
─────────────────────────────────────────────────────

World map divided into grid recursively:
- First split: Latitude < 0° (south) vs >= 0° (north)
- Second split: Longitude < 0° (west) vs >= 0° (east)
- Continue splitting until desired precision

Example: Times Square, NYC (40.7580°N, -73.9855°W)

Latitude: 40.7580° (range -90° to +90°)
├─ Is it >= 0°? YES → bit = 1 (northern hemisphere)
├─ Is it >= 45°? NO → bit = 0 (below 45°)
├─ Is it >= 22.5°? YES → bit = 1 (above 22.5°)
├─ Is it >= 33.75°? YES → bit = 1
└─ ... continue for 26 bits → 11010111001001...

Longitude: -73.9855° (range -180° to +180°)
├─ Is it >= 0°? NO → bit = 0 (western hemisphere)
├─ Is it >= -90°? YES → bit = 1
├─ Is it >= -45°? NO → bit = 0
└─ ... continue for 26 bits → 01001101110101...

Step 2: Interleave latitude and longitude bits
─────────────────────────────────────────────────────
Latitude bits:  1 1 0 1 0 1 1 1 0 0 1 ...
Longitude bits: 0 1 0 0 1 1 0 1 1 1 0 ...
Interleaved:    01 11 00 01 01 11 01 11 01 01 01 ...
                (lon bit, lat bit, lon bit, lat bit, ...)

Step 3: Convert binary to base-32 string (geohash)
─────────────────────────────────────────────────────
Binary: 01110001011101011101...
Split into 5-bit chunks: 01110 | 00101 | 11010 | 11101 | ...
Convert to base-32 (0-9, a-z): "dr5regw3pb" ← geohash

Precision levels:
├─ "dr5regw3pb" (10 chars) → ~20cm accuracy
├─ "dr5regw3p" (9 chars) → ~5m accuracy
├─ "dr5regw" (7 chars) → ~150m accuracy
└─ "dr5" (3 chars) → ~150km accuracy
```

**Why Geohashing Enables Fast Queries:**

```text
Key Insight: Nearby locations have similar geohash prefixes!

Times Square:      "dr5regw3pb"
2 blocks away:     "dr5regw3p9" (shares "dr5regw3p")
1 mile away:       "dr5regu7xy" (shares "dr5reg")
10 miles away:     "dr5r5abc12" (shares "dr5r")
London, UK:        "gcpvj0d..." (completely different)

To find nearby drivers:
1. Calculate geohash of restaurant: "dr5regw"
2. Find all drivers with prefix "dr5regw" → Fast! O(log N) sorted set lookup
3. Check exact distance for candidates → O(M) where M = candidates
4. Total: O(log N + M) instead of O(N) for naive scan
```

**Redis Implementation (Sorted Set):**

```text
Data Structure:
─────────────────────────────────────────────────────
Redis Sorted Set: "driver_locations"

Member (driver ID)    Score (52-bit geohash integer)
──────────────────────────────────────────────────────
driver:12345          3750620197      ← "dr5regw3pb" encoded
driver:67890          3750620185      ← "dr5regw3p9" (nearby!)
driver:11111          3750510042      ← "dr5regu7xy" (farther)
driver:99999          2188428541      ← "gcpvj0d..." (London)

Sorted by score → enables range queries!
```

**GEORADIUS Query Execution:**

```python
# What happens when you run:
GEORADIUS driver_locations -73.9855 40.7580 5 km WITHDIST ASC

# Redis internally does:
def georadius(key, lon, lat, radius_km):
    # Step 1: Convert center point to geohash
    center_geohash = encode_geohash(lon, lat, precision=26)
    center_score = geohash_to_integer(center_geohash)
    # Example: center_score = 3750620197
    
    # Step 2: Calculate geohash range for radius
    # For 5km radius, geohash precision ~6 chars ("dr5reg")
    min_score = center_score - calculate_radius_offset(radius_km)
    max_score = center_score + calculate_radius_offset(radius_km)
    # Example: min_score = 3750610000, max_score = 3750630000
    
    # Step 3: Range query on sorted set (O(log N) to find start, O(M) to scan)
    candidates = zrangebyscore(key, min_score, max_score)
    # Returns ~50 drivers with similar geohash
    
    # Step 4: Filter by exact distance (Haversine formula)
    results = []
    for member in candidates:
        driver_location = get_driver_location(member)
        distance = haversine(lon, lat, 
                            driver_location.lon, driver_location.lat)
        
        if distance <= radius_km:
            results.append({
                'driver_id': member,
                'distance_km': distance
            })
    
    # Step 5: Sort by distance ascending
    results.sort(key=lambda x: x['distance_km'])
    
    return results
```

**Time Complexity Analysis:**

```text
OPERATION                         TIME COMPLEXITY
───────────────────────────────────────────────────────────────
Encode center point to geohash    O(1) - fixed 26 bits
Calculate geohash range           O(1) - simple arithmetic
Range query on sorted set         O(log N + M)
  - Find start position           O(log N) - binary search
  - Scan matching entries         O(M) - linear scan
Haversine distance calculation    O(M) - check each candidate
Sort results by distance          O(M log M)
───────────────────────────────────────────────────────────────
TOTAL:                            O(log N + M log M)

Where:
- N = total drivers in system (1,000,000)
- M = drivers within radius (typically 50-100)

Real performance:
- log(1,000,000) ≈ 20 comparisons to find starting point
- 50-100 candidates to check distance
- 50 log(50) ≈ 300 comparisons to sort
- Total: <10ms on modern hardware
```

**Comparison to Alternatives:**

```text
APPROACH              TIME COMPLEXITY    QUERY TIME (1M drivers)
──────────────────────────────────────────────────────────────────
Naive scan            O(N)               16 minutes (1ms per driver)
R-tree (PostGIS)      O(log N + M)       50-100ms (disk I/O)
Quadtree              O(log N + M)       20-50ms (memory)
Geohash (Redis)       O(log N + M)       5-10ms (in-memory)
──────────────────────────────────────────────────────────────────

Redis wins due to:
✅ In-memory (no disk I/O)
✅ Highly optimized sorted set implementation
✅ Simple API (no complex geospatial query language)
```

**Real-World Performance (Uber):**

```text
Production Metrics (NYC peak hour):
├─ Queries per second: 100 QPS
├─ Average query time: 8ms
├─ 95th percentile: 15ms
├─ 99th percentile: 25ms
└─ Drivers in system: 800K
```

**Interview Tip:** Emphasize that geohashing trades **precision for speed**. At 5km radius, we might include drivers 5.1km away (false positives) or miss drivers at exactly 5.0km (false negatives) due to grid boundaries. But this trade-off is acceptable—better to check 10 extra drivers than scan 1M drivers.

</details>

**Q5:** How would you ensure fairness in driver assignment? What if one driver keeps getting all the orders?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Fairness in driver assignment** prevents the "rich get richer" problem where high-rated drivers near popular restaurants monopolize orders while other drivers sit idle. We use **earnings-based scoring**, **rotation algorithms**, and **zone-based quotas** to distribute orders fairly.

**The Fairness Problem:**

```text
Without Fairness Mechanism:

Driver A (High Score):
├─ Location: 0.5km from popular restaurant
├─ Rating: 4.9★ (excellent)
├─ Acceptance rate: 98%
└─ Score: 0.85 (consistently highest)

Driver B (Lower Score):
├─ Location: 2.0km from popular restaurant
├─ Rating: 4.6★ (good)
├─ Acceptance rate: 90%
└─ Score: 0.65 (consistently lower)

Result over 1 hour:
Driver A: Gets 10 orders → Earns $80
Driver B: Gets 1 order → Earns $8

Driver B's Perspective:
- "I've been online 2 hours, only got 1 order"
- "I'm paying for gas and parking for nothing"
- "Driver A is hogging all the orders!"
- Eventually: Driver B goes offline (platform loses driver)

Impact: 20% of drivers get 80% of orders → low driver retention
```

**Solution 1: Earnings-Based Score Adjustment**

```python
def calculate_fairness_score(driver_id, current_hour):
    """
    Boost score for drivers who haven't earned much today
    """
    # Get driver's earnings for current day
    today_earnings = db.get_driver_earnings(
        driver_id=driver_id,
        date=datetime.today()
    )
    
    # Get average earnings for all active drivers
    avg_earnings = db.get_average_earnings(date=datetime.today())
    
    # Calculate fairness multiplier
    if today_earnings < avg_earnings * 0.5:
        # Earned less than 50% of average → boost by 50%
        fairness_score = 1.5
    elif today_earnings < avg_earnings * 0.8:
        # Earned 50-80% of average → boost by 25%
        fairness_score = 1.25
    elif today_earnings < avg_earnings * 1.2:
        # Earned 80-120% of average → normal
        fairness_score = 1.0
    elif today_earnings < avg_earnings * 1.5:
        # Earned 120-150% of average → reduce by 20%
        fairness_score = 0.8
    else:
        # Earned >150% of average → reduce by 40%
        fairness_score = 0.6
    
    return fairness_score

# Example usage in driver scoring:
base_score = calculate_base_score(driver)  # Distance, rating, etc.
fairness_multiplier = calculate_fairness_score(driver.id, current_hour)
final_score = base_score * fairness_multiplier

# Driver A: base_score=0.85, earned $80 (above avg) → 0.85 × 0.8 = 0.68
# Driver B: base_score=0.65, earned $8 (below avg) → 0.65 × 1.5 = 0.97
# Result: Driver B now ranks higher! Gets next order
```

**Solution 2: Round-Robin Within Score Bands**

```text
Instead of always picking the absolute highest score, use score bands:

SCORE BAND         DRIVERS                    ASSIGNMENT
────────────────────────────────────────────────────────────────
0.80-1.00 (A)     Driver A (0.85), C (0.82)  Round-robin
0.60-0.80 (B)     Driver B (0.65), D (0.72)  Round-robin
0.40-0.60 (C)     Driver E (0.55), F (0.48)  Round-robin

Algorithm:
1. Find all drivers in top score band (0.80-1.00)
2. Among those drivers, pick the one who received an order longest ago
3. If all drivers in top band received order recently, move to next band

Example Timeline:
00:00 - Order 1 → Driver A (score 0.85, band A, last order: never)
00:05 - Order 2 → Driver C (score 0.82, band A, last order: never)
00:10 - Order 3 → Driver A (score 0.85, band A, last order: 00:00) ← Would be picked
                  BUT Driver C's last order was more recent (00:05)
                  SO pick Driver A again (he's been waiting longer)
00:15 - Order 4 → Driver C (score 0.82, band A, last order: 00:05)
                  Driver A received order at 00:10 (5 min ago)
                  Driver C received order at 00:05 (10 min ago)
                  → Driver C gets order (waiting longer)
```

**Solution 3: Zone-Based Quotas**

```python
class ZoneQuotaManager:
    """
    Ensure each zone has fair access to orders
    Prevents dense zones from monopolizing all deliveries
    """
    
    def __init__(self):
        # Track orders delivered per zone per hour
        self.zone_orders = defaultdict(int)
        self.zone_capacity = {}  # Max orders per zone per hour
    
    def calculate_zone_capacity(self, zone_id, current_hour):
        """
        Dynamic capacity based on demand and driver availability
        """
        # Get number of active drivers in zone
        active_drivers = db.count_active_drivers(zone_id=zone_id)
        
        # Estimate: each driver can handle 3 orders/hour
        capacity = active_drivers * 3
        
        self.zone_capacity[zone_id] = capacity
        return capacity
    
    def can_assign_to_zone(self, zone_id):
        """
        Check if zone is under quota
        """
        current_orders = self.zone_orders[zone_id]
        capacity = self.zone_capacity.get(zone_id, float('inf'))
        
        return current_orders < capacity
    
    def filter_drivers_by_quota(self, drivers):
        """
        Remove drivers from zones that exceeded quota
        """
        eligible_drivers = []
        
        for driver in drivers:
            zone_id = driver.current_zone_id
            
            if self.can_assign_to_zone(zone_id):
                eligible_drivers.append(driver)
            else:
                # Zone is at capacity, skip this driver
                logger.info(f"Zone {zone_id} at capacity, skipping driver {driver.id}")
        
        return eligible_drivers
    
    def record_assignment(self, driver_id, zone_id):
        """
        Increment zone counter after assignment
        """
        self.zone_orders[zone_id] += 1

# Usage:
quota_manager = ZoneQuotaManager()
nearby_drivers = find_nearby_drivers(order_id, radius_km=5)

# Filter out drivers from over-quota zones
eligible_drivers = quota_manager.filter_drivers_by_quota(nearby_drivers)

# Score and assign
best_driver = score_and_select(eligible_drivers)
quota_manager.record_assignment(best_driver.id, best_driver.current_zone_id)
```

**Real-World Metrics (DoorDash):**

```text
WITHOUT Fairness Mechanisms:
├─ Top 20% drivers: Get 75% of orders
├─ Bottom 50% drivers: Get 10% of orders
├─ Driver churn rate: 35% per month (drivers quit)
└─ Customer complaints: "Why is it always the same driver?"

WITH Fairness Mechanisms:
├─ Top 20% drivers: Get 35% of orders (still more, but not monopoly)
├─ Bottom 50% drivers: Get 35% of orders (much improved)
├─ Driver churn rate: 18% per month (47% reduction!)
└─ Driver satisfaction: +23%

Business Impact:
- Reduced driver acquisition cost: $50/driver × 10K drivers/month × 17% = $85K saved/month
- Increased driver retention → better service quality
- More diverse driver pool → better coverage
```

**Trade-offs:**

```text
AGGRESSIVE Fairness (earnings weight 30%):
✅ Very balanced earnings across drivers
❌ Slightly longer delivery times (may assign farther driver)
❌ Customer gets lower-rated drivers more often

MODERATE Fairness (earnings weight 10%):
✅ Balanced earnings but still optimizes for speed
✅ Minimal impact on delivery time
❌ Some inequality still exists

NO Fairness (earnings weight 0%):
✅ Fastest deliveries (always closest driver)
✅ Highest-rated drivers (best customer experience)
❌ 50% of drivers earn very little → quit platform
❌ Long-term: fewer drivers → worse service
```

**Interview Tip:** Frame fairness as a **marketplace health** issue, not just altruism. If drivers aren't earning fairly, they'll leave the platform, reducing supply, increasing wait times, and ultimately harming customers. It's in the platform's economic interest to keep drivers happy and active.

</details>

#### Advanced Level

**Q6:** Design a batch delivery optimization system where one driver picks up multiple orders and delivers them efficiently.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Batch delivery** allows one driver to handle 2-3 orders simultaneously, improving driver earnings (+40%) and platform efficiency while maintaining acceptable delivery times (<45 min per order). This requires solving a **constrained Vehicle Routing Problem (VRP)**.

**The Business Case for Batching:**

```text
Single-Order Model:
Driver: Idle → Restaurant A (5 min) → Customer A (10 min) → Idle
Time: 15 minutes total
Earnings: $7 per delivery
Efficiency: 40 minutes idle per hour (wasted)

Batch Model (3 orders):
Driver: Idle → Restaurant A (5 min) → Restaurant B (3 min) → 
        Customer A (8 min) → Restaurant C (4 min) → 
        Customer B (7 min) → Customer C (10 min) → Idle
Time: 37 minutes total
Earnings: $7 × 3 = $21
Efficiency: 23 minutes idle per hour (much better)

Driver Perspective:
- Single: $28/hour (4 deliveries)
- Batch: $34/hour (5-6 deliveries)
- +21% earnings with batching!
```

**Constraints for Batch Optimization:**

```text
HARD CONSTRAINTS (Must satisfy):
1. Pickup before delivery: Must pick up Order A from Restaurant A before delivering to Customer A
2. Max delivery time: Each order < 45 minutes total (placement → delivery)
3. Food temperature: Hot food shouldn't wait >20 minutes after pickup
4. Vehicle capacity: Max 3 orders per driver (bag size limit)
5. Dietary restrictions: Don't mix vegan/non-vegan in same bag (cross-contamination)

SOFT CONSTRAINTS (Optimize):
1. Minimize total driving time
2. Minimize per-order delivery time
3. Maximize driver earnings
4. Prefer similar restaurant locations (easier pickup)
```

**Batch Matching Algorithm:**

```python
import googlemaps
from ortools.constraint_solver import routing_enums_pb2
from ortools.constraint_solver import pywrapcp

class BatchDeliveryOptimizer:
    """
    Optimize route for driver to pick up and deliver multiple orders
    Uses Google OR-Tools for Vehicle Routing Problem
    """
    
    def __init__(self):
        self.gmaps = googlemaps.Client(key='YOUR_API_KEY')
    
    def find_batchable_orders(self, driver_location, max_radius_km=5):
        """
        Find orders that can be batched together
        """
        # Get all unassigned orders near driver
        nearby_orders = db.query("""
            SELECT * FROM orders 
            WHERE status = 'CONFIRMED'
            AND assigned_driver_id IS NULL
            AND ST_Distance_Sphere(
                restaurant_location,
                POINT(:driver_lon, :driver_lat)
            ) < :radius_meters
            ORDER BY created_at ASC
            LIMIT 10
        """, {
            'driver_lat': driver_location.latitude,
            'driver_lon': driver_location.longitude,
            'radius_meters': max_radius_km * 1000
        })
        
        # Filter by time constraints
        batchable_orders = []
        for order in nearby_orders:
            # Order must be ready for pickup within 15 minutes
            time_until_ready = order.estimated_ready_time - datetime.utcnow()
            
            if timedelta(0) <= time_until_ready <= timedelta(minutes=15):
                batchable_orders.append(order)
        
        return batchable_orders
    
    def optimize_batch_route(self, driver, orders):
        """
        Find optimal route through restaurants and customers
        Using Vehicle Routing Problem with Time Windows (VRPTW)
        """
        if len(orders) == 0:
            return None
        
        # Build location list
        locations = [driver.current_location]  # Start point
        location_map = {0: {'type': 'driver', 'id': driver.id}}
        
        idx = 1
        for order in orders:
            # Add restaurant pickup
            locations.append(order.restaurant.location)
            location_map[idx] = {
                'type': 'pickup',
                'order_id': order.id,
                'delivery_idx': idx + 1  # Link to corresponding delivery
            }
            idx += 1
            
            # Add customer delivery
            locations.append(order.customer.location)
            location_map[idx] = {
                'type': 'delivery',
                'order_id': order.id,
                'pickup_idx': idx - 1  # Link to corresponding pickup
            }
            idx += 1
        
        # Create distance matrix (driving time in minutes)
        distance_matrix = self.create_distance_matrix(locations)
        
        # Set up OR-Tools routing model
        manager = pywrapcp.RoutingIndexManager(
            len(locations),  # Number of locations
            1,               # Number of vehicles (1 driver)
            0                # Depot (driver starting location)
        )
        
        routing = pywrapcp.RoutingModel(manager)
        
        # Define distance callback
        def distance_callback(from_index, to_index):
            from_node = manager.IndexToNode(from_index)
            to_node = manager.IndexToNode(to_index)
            return distance_matrix[from_node][to_node]
        
        transit_callback_index = routing.RegisterTransitCallback(distance_callback)
        routing.SetArcCostEvaluatorOfAllVehicles(transit_callback_index)
        
        # Add pickup-delivery constraints
        for order in orders:
            pickup_idx = None
            delivery_idx = None
            
            # Find pickup and delivery indices for this order
            for idx, loc_info in location_map.items():
                if loc_info.get('order_id') == order.id:
                    if loc_info['type'] == 'pickup':
                        pickup_idx = manager.NodeToIndex(idx)
                    elif loc_info['type'] == 'delivery':
                        delivery_idx = manager.NodeToIndex(idx)
            
            # Constraint: Must visit pickup before delivery
            routing.solver().Add(
                routing.VehicleVar(pickup_idx) == routing.VehicleVar(delivery_idx)
            )
            routing.AddPickupAndDelivery(pickup_idx, delivery_idx)
        
        # Add time window constraints (each order < 45 minutes)
        time_dimension = routing.GetDimensionOrDie('Time')
        for order_id in [o.id for o in orders]:
            # Find delivery node for this order
            for idx, loc_info in location_map.items():
                if loc_info.get('order_id') == order_id and loc_info['type'] == 'delivery':
                    delivery_node = idx
                    routing.solver().Add(
                        time_dimension.CumulVar(manager.NodeToIndex(delivery_node)) <= 45
                    )
        
        # Set search parameters
        search_parameters = pywrapcp.DefaultRoutingSearchParameters()
        search_parameters.first_solution_strategy = (
            routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC
        )
        search_parameters.local_search_metaheuristic = (
            routing_enums_pb2.LocalSearchMetaheuristic.GUIDED_LOCAL_SEARCH
        )
        search_parameters.time_limit.seconds = 5  # Max 5 seconds to solve
        
        # Solve
        solution = routing.SolveWithParameters(search_parameters)
        
        if solution:
            return self.extract_route(solution, routing, manager, location_map)
        else:
            # No valid route found (constraints too tight)
            return None
    
    def create_distance_matrix(self, locations):
        """
        Create matrix of driving times between all location pairs
        Uses Google Maps Directions API
        """
        n = len(locations)
        matrix = [[0] * n for _ in range(n)]
        
        # Batch request to Google Maps API
        origins = [f"{loc.lat},{loc.lon}" for loc in locations]
        destinations = origins
        
        result = self.gmaps.distance_matrix(
            origins=origins,
            destinations=destinations,
            mode='driving',
            departure_time='now'  # Use current traffic
        )
        
        for i in range(n):
            for j in range(n):
                if i == j:
                    matrix[i][j] = 0
                else:
                    # Extract duration in minutes
                    duration_sec = result['rows'][i]['elements'][j]['duration_in_traffic']['value']
                    matrix[i][j] = duration_sec // 60
        
        return matrix
    
    def extract_route(self, solution, routing, manager, location_map):
        """
        Extract optimized route from OR-Tools solution
        """
        route = []
        index = routing.Start(0)
        
        while not routing.IsEnd(index):
            node = manager.IndexToNode(index)
            location_info = location_map[node]
            route.append(location_info)
            index = solution.Value(routing.NextVar(index))
        
        return route

# Example Usage:
optimizer = BatchDeliveryOptimizer()

# Driver available in Manhattan
driver = db.get_driver(driver_id=12345)

# Find nearby orders that can be batched
batchable_orders = optimizer.find_batchable_orders(
    driver_location=driver.current_location,
    max_radius_km=3
)

# Optimize route for up to 3 orders
if len(batchable_orders) >= 2:
    best_batch = optimizer.optimize_batch_route(
        driver=driver,
        orders=batchable_orders[:3]  # Max 3 orders
    )
    
    if best_batch:
        # Assign all orders to driver
        for order_id in [o.id for o in batchable_orders[:3]]:
            assign_order_to_driver(order_id, driver.id)
        
        # Send route to driver app
        send_batch_route(driver.id, best_batch)
```

**Example Optimized Route:**

```text
INPUT: Driver with 3 orders
- Order A: Restaurant R1 → Customer C1 (created 5 min ago)
- Order B: Restaurant R2 → Customer C2 (created 3 min ago)
- Order C: Restaurant R3 → Customer C3 (created 1 min ago)

NAIVE ROUTE (FIFO):
Driver → R1 (5min) → C1 (8min) → R2 (6min) → C2 (7min) → R3 (9min) → C3 (10min)
Total time: 45 minutes
Order A delivery time: 13 minutes ✅
Order B delivery time: 26 minutes ✅
Order C delivery time: 45 minutes ⚠️ (just under limit)

OPTIMIZED ROUTE (Clustered pickups):
Driver → R1 (5min) → R2 (2min) → R3 (3min) → C1 (6min) → C2 (4min) → C3 (5min)
Total time: 25 minutes
Order A delivery time: 16 minutes ✅
Order B delivery time: 21 minutes ✅
Order C delivery time: 25 minutes ✅
Driver saved: 20 minutes (44% faster!)
```

**Real-World Results (Uber Eats Batch Deliveries):**

```text
METRICS                         SINGLE ORDER    BATCH (2-3 ORDERS)
────────────────────────────────────────────────────────────────────
Driver earnings per hour        $22             $31 (+41%)
Platform revenue per hour       $15             $24 (+60%)
Average delivery time           28 minutes      32 minutes (+14%)
Customer satisfaction           4.7/5.0         4.5/5.0 (-4%)
Driver acceptance rate          85%             92% (+8%)

Trade-off Analysis:
✅ Significantly higher driver earnings (happier drivers)
✅ More efficient use of drivers (can serve more customers)
⚠️ Slightly longer delivery times (but still under 45 min)
⚠️ Slightly lower satisfaction (cold food if route is inefficient)

Recommendation: Use batching for 30% of orders (non-peak times)
Peak dinner rush: Disable batching (speed is critical)
```

**Interview Tip:** Emphasize that batch optimization is a **constrained optimization problem** similar to the Traveling Salesman Problem (TSP) but with additional constraints (pickup before delivery, time windows, capacity limits). The key is finding the right balance between driver efficiency and customer experience—too much batching leads to cold food and bad reviews.

</details>

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

### 🎯 Interview Questions - Location Tracking & ETA Calculation

#### Beginner Level

**Q1:** Why use WebSockets for real-time location updates instead of HTTP polling?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**WebSockets** provide a persistent, bidirectional connection that's far more efficient than HTTP polling for real-time location updates. At scale, WebSockets reduce bandwidth by 99% and server load by 95%.

**HTTP Polling (The Inefficient Way):**

```text
Customer app needs to show driver's real-time location:

Method 1: Short Polling
─────────────────────────────────────────────────────
Customer app → Server: "Where is driver?" (HTTP GET)
Server → Customer app: "{lat: 40.7500, lon: -73.9900}"
[Wait 1 second]
Customer app → Server: "Where is driver?" (HTTP GET)
Server → Customer app: "{lat: 40.7501, lon: -73.9901}"
[Wait 1 second]
Customer app → Server: "Where is driver?" (HTTP GET)
...repeat every 1 second...

Problems:
❌ New HTTP connection every second (TCP handshake overhead)
❌ Full HTTP headers sent each time (~500 bytes)
❌ Server must handle 200K customers × 1 req/sec = 200K QPS
❌ 90% of requests return "no change" (driver hasn't moved)
❌ Latency: 500-1000ms per request (see update 1 sec after it happens)

Bandwidth Usage (per customer):
- Request headers: 300 bytes
- Response headers: 200 bytes
- Response body: 100 bytes
- Total: 600 bytes per second × 200K customers = 120 MB/sec
- Daily: 120 MB/sec × 86400 sec = 10.4 TB/day (!!)

Method 2: Long Polling (Better, but still wasteful)
─────────────────────────────────────────────────────
Customer app → Server: "Tell me when driver moves"
[Server holds connection open, waiting for change]
[Driver moves 100m]
Server → Customer app: "{lat: 40.7501, lon: -73.9901}"
Customer app → Server: "Tell me when driver moves again"
[Repeat...]

Problems:
❌ Server holds 200K open connections (memory intensive)
❌ Still requires new HTTP request after each update
❌ Doesn't scale well with 200K concurrent users
```

**WebSocket (The Efficient Way):**

```text
WebSocket: Persistent bidirectional connection

Initial Handshake (Once per customer):
─────────────────────────────────────────────────────
Customer app → Server: HTTP GET /track?order_id=123 (Upgrade: websocket)
Server → Customer app: HTTP 101 Switching Protocols
[Connection upgraded to WebSocket - stays open]

Ongoing Updates (Every 1 second, 25-minute delivery):
─────────────────────────────────────────────────────
Driver moves → Server sends: {lat: 40.7500, lon: -73.9900}
Driver moves → Server sends: {lat: 40.7501, lon: -73.9901}
Driver moves → Server sends: {lat: 40.7502, lon: -73.9902}
...1,500 updates over 25 minutes...
[No reconnection needed!]

Benefits:
✅ Single connection for entire delivery (no reconnection overhead)
✅ No HTTP headers after initial handshake (just raw data)
✅ Server can push updates immediately (no polling delay)
✅ Bidirectional: Customer can send messages too (e.g., "ping")
✅ Low latency: <50ms from driver location change to customer sees it

Bandwidth Usage (per customer):
- Initial handshake: 600 bytes (one-time)
- Per update: 50 bytes (just {lat, lon, timestamp})
- Total: 600 + (50 bytes × 1500 updates) = 75 KB per delivery
- vs HTTP polling: 600 bytes × 1500 = 900 KB per delivery
- Savings: 92% less bandwidth! ✅
```

**Scalability Comparison:**

```text
SCENARIO: 200K concurrent deliveries (Uber Eats NYC at dinner rush)

HTTP Short Polling:
├─ Requests per second: 200K customers × 1 req/sec = 200K QPS
├─ Bandwidth: 120 MB/sec = 10.4 TB/day
├─ Server cost: 500 servers × $0.10/hour = $50/hour = $1,200/day
└─ Total cost: $1,200 + $2,000 (bandwidth) = $3,200/day

WebSocket:
├─ Concurrent connections: 200K (persistent)
├─ Bandwidth: 10 MB/sec = 864 GB/day (only sending when driver moves)
├─ Server cost: 50 servers × $0.10/hour = $5/hour = $120/day
└─ Total cost: $120 + $170 (bandwidth) = $290/day

SAVINGS: $3,200 - $290 = $2,910/day = $87K/month = $1.06M/year! 💰
```

**Real-World Implementation (Uber Eats):**

```python
import asyncio
import websockets
import json

class LocationWebSocketServer:
    def __init__(self):
        self.connections = {}  # {order_id: websocket}
    
    async def handle_customer_connection(self, websocket, order_id):
        """
        Handle WebSocket connection from customer app
        """
        # Register connection
        self.connections[order_id] = websocket
        
        try:
            # Send initial location
            driver_location = get_driver_location(order_id)
            await websocket.send(json.dumps({
                'type': 'location_update',
                'latitude': driver_location.lat,
                'longitude': driver_location.lon,
                'eta_minutes': calculate_eta(order_id),
                'timestamp': datetime.utcnow().isoformat()
            }))
            
            # Keep connection alive (ping/pong)
            while True:
                try:
                    # Wait for ping from client (heartbeat)
                    message = await asyncio.wait_for(
                        websocket.recv(),
                        timeout=30  # 30 sec timeout
                    )
                    
                    if message == 'ping':
                        await websocket.send('pong')
                
                except asyncio.TimeoutError:
                    # Client didn't send heartbeat, close connection
                    break
        
        except websockets.ConnectionClosed:
            # Client disconnected
            pass
        
        finally:
            # Clean up
            del self.connections[order_id]
    
    async def broadcast_location_update(self, order_id, driver_location):
        """
        Push location update to customer (if connected)
        """
        if order_id in self.connections:
            websocket = self.connections[order_id]
            
            try:
                await websocket.send(json.dumps({
                    'type': 'location_update',
                    'latitude': driver_location.lat,
                    'longitude': driver_location.lon,
                    'eta_minutes': calculate_eta(order_id),
                    'timestamp': datetime.utcnow().isoformat()
                }))
            except websockets.ConnectionClosed:
                # Connection closed, remove it
                del self.connections[order_id]

# Start WebSocket server
server = LocationWebSocketServer()
asyncio.run(websockets.serve(server.handle_customer_connection, "0.0.0.0", 8765))
```

**When to Use What:**

```text
USE WEBSOCKETS:
✅ Real-time location tracking (frequent updates)
✅ Live chat/messaging
✅ Stock tickers, sports scores
✅ Collaborative editing
✅ Gaming

USE HTTP POLLING:
✅ Infrequent updates (check email every 5 minutes)
✅ Background sync (upload photos when available)
✅ Simple status checks
✅ One-time queries
```

**Interview Tip:** Mention that WebSockets have a trade-off—they require **stateful servers** (connection stays on same server), making horizontal scaling more complex. Solutions include sticky sessions (load balancer pins customer to server) or Redis pub/sub (servers share messages).

</details>

**Q2:** How do you calculate ETA for food delivery? What factors affect accuracy?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**ETA (Estimated Time of Arrival)** must account for multiple factors beyond just distance: traffic conditions, driver route efficiency, restaurant preparation time, and historical patterns. Accurate ETAs build customer trust.

**Naive ETA Calculation (Distance ÷ Speed):**

```text
Simple formula:
ETA = Distance / Average_Speed

Example:
- Driver to customer: 5 km
- Average city speed: 30 km/h
- ETA = 5 km ÷ 30 km/h = 0.167 hours = 10 minutes

Problems:
❌ Assumes constant speed (ignores traffic lights, traffic jams)
❌ Doesn't account for route (straight line vs actual roads)
❌ Ignores time of day (rush hour vs midnight)
❌ No restaurant preparation time included
❌ Accuracy: ~60% (frequently wrong by 5-10 minutes)
```

**Production ETA Calculation (Multi-Factor):**

```text
ETA = Restaurant_Prep_Time + Driver_Travel_Time + Contingency_Buffer

STEP 1: Restaurant Preparation Time
────────────────────────────────────────────────────
- Historical average for this restaurant
- Varies by cuisine type (pizza: 12 min, sushi: 8 min, burger: 15 min)
- Adjusted for current order queue (if 5 orders ahead, add 3 min)
- Time of day factor (lunch rush → slower)

Example Calculation:
Base prep time: 12 minutes (pizza)
Current queue: 2 orders × 1.5 min = +3 minutes
Rush hour penalty: +2 minutes
Total prep time: 17 minutes

STEP 2: Driver Travel Time (Traffic-Aware)
────────────────────────────────────────────────────
Use Google Maps Directions API with current traffic:

Request:
{
  "origin": "driver_current_location",
  "destination": "customer_delivery_address",
  "mode": "driving",
  "departure_time": "now",  // Use live traffic data
  "traffic_model": "best_guess"
}

Response:
{
  "duration": 600,  // 10 minutes without traffic
  "duration_in_traffic": 840  // 14 minutes with current traffic
}

Use duration_in_traffic (14 min) for accuracy

STEP 3: Contingency Buffer
────────────────────────────────────────────────────
Add buffer based on confidence level:
- High confidence (short distance, light traffic): +2 minutes
- Medium confidence (medium distance/traffic): +3 minutes
- Low confidence (long distance, heavy traffic): +5 minutes

Buffer calculation:
IF distance < 2km AND traffic_level = "light": buffer = 2 min
ELSE IF distance < 5km AND traffic_level = "moderate": buffer = 3 min
ELSE: buffer = 5 min

Final ETA:
────────────────────────────────────────────────────
ETA = 17 min (prep) + 14 min (travel) + 3 min (buffer) = 34 minutes

Tell customer: "Arrives in 30-35 minutes"
(Range accounts for uncertainty)
```

**Factors Affecting ETA Accuracy:**

```text
FACTOR                      IMPACT          WHY?
──────────────────────────────────────────────────────────────────────
1. Traffic Conditions       ±5-10 min       Rush hour doubles travel time
2. Weather                  ±3-5 min        Rain/snow slows drivers 30%
3. Restaurant Speed         ±5 min          Some restaurants consistently late
4. Driver Experience        ±2 min          New drivers navigate slower
5. Order Complexity         ±3 min          10-item order vs 2-item order
6. Building Type            ±2 min          Apartment (elevator) vs house
7. Time of Day              ±5 min          Lunch/dinner rush vs off-peak
8. Day of Week              ±3 min          Friday/Saturday busier

Example Scenarios:
─────────────────────────────────────────────────────
BEST CASE (Tuesday 3 PM, clear weather, experienced driver):
- Base ETA: 25 minutes
- Adjustments: -2 min (off-peak) + 0 (good weather)
- Final ETA: 23 minutes ✅ (high accuracy)

WORST CASE (Friday 7 PM, heavy rain, new driver):
- Base ETA: 25 minutes
- Adjustments: +8 min (rush hour) + 4 min (rain) + 2 min (new driver)
- Final ETA: 39 minutes ⚠️ (lower accuracy)
```

**Dynamic ETA Updates:**

```text
ETA should update as driver travels:

00:00 - Order placed
├─ Initial ETA: 35 minutes (based on prep time + distance)
├─ Tell customer: "Arrives by 6:35 PM"

00:12 - Restaurant marks order ready (faster than expected!)
├─ Updated ETA: 30 minutes (reduced prep time)
├─ Notify customer: "Your order is ready earlier! Now arriving by 6:30 PM"

00:15 - Driver picks up order
├─ Recalculate ETA based on current location + traffic
├─ Google Maps API: 15 minutes to customer
├─ Updated ETA: 15 minutes (6:15 PM + 15 min = 6:30 PM)
├─ Tell customer: "Driver picked up your order. Arriving in 15 minutes"

00:20 - Driver stuck in traffic (5 min into delivery, still 5 km away)
├─ Google Maps API: 18 minutes remaining (traffic jam detected)
├─ Updated ETA: 18 minutes
├─ Notify customer: "Delayed due to traffic. New arrival time: 6:38 PM"

00:30 - Driver arrives
├─ Actual delivery time: 30 minutes (vs initial estimate 35 min)
├─ Customer satisfied (arrived earlier than promised!)
```

**ETA Accuracy Metrics (Real-World):**

```text
PLATFORM        ACCURACY (±5 min)    AVERAGE ERROR    CUSTOMER SATISFACTION
────────────────────────────────────────────────────────────────────────────
Uber Eats       82%                  4.2 minutes      4.3/5.0
DoorDash        79%                  4.8 minutes      4.2/5.0
Grubhub         74%                  5.6 minutes      4.0/5.0

Key Insight: +1% ETA accuracy → +0.05 points satisfaction
Better to slightly over-estimate (arrive 2 min early) than under-estimate (arrive 5 min late)
```

**Interview Talking Points:**

1. **Use traffic APIs:** Don't calculate yourself—Google Maps/Mapbox have real-time traffic data from millions of users

2. **Update frequently:** Recalculate ETA every 2-3 minutes during delivery, not just once at order placement

3. **Historical data:** Track restaurant average prep times and driver average speeds by time of day

4. **Conservative estimates:** Better to tell customer "35 minutes" and deliver in 30 (happy!) than say "25 minutes" and take 35 (angry!)

5. **Transparent communication:** If ETA changes significantly (>5 min), proactively notify customer with reason ("traffic delay")

**Interview Tip:** Mention that ETA accuracy is a **key business metric**—it directly impacts customer satisfaction, repeat orders, and platform reputation. Uber Eats tracks "ETA beat rate" (% of orders delivered before estimated time) as a KPI, targeting 55-60% (slightly conservative estimates).

</details>

**Q3:** How do you store and query location history for millions of drivers? What database is best?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Location history** is a **time-series workload**: write-heavy (1 location/sec per driver), time-based queries (show last 10 minutes), and massive volume (25.9 TB/month). **Cassandra** is optimal for this use case.

**The Scale of Location Data:**

```text
Scale Calculation:
├─ 200,000 active drivers (concurrent during peak)
├─ Location update every 1 second
├─ Data per update: ~50 bytes (driver_id, lat, lon, timestamp, speed, bearing)
├─ Writes per second: 200K drivers × 1 update/sec = 200K writes/sec
├─ Data per hour: 200K writes/sec × 50 bytes × 3600 sec = 36 GB/hour
├─ Data per day: 36 GB/hour × 24 hours = 864 GB/day
└─ Data per month (30-day retention): 864 GB × 30 = 25.9 TB

Query Patterns:
├─ Get driver's current location (by driver_id) → 100K QPS
├─ Get driver's last 10 minutes of locations → 10K QPS
├─ Replay driver's route for dispute resolution → 100 QPM
└─ Analytics: aggregate driver behavior patterns → batch jobs
```

**Why Cassandra for Location Data:**

```text
REQUIREMENT              CASSANDRA SOLUTION
────────────────────────────────────────────────────────────────────
1. High write throughput  ✅ Optimized for writes (append-only LSM tree)
   (200K writes/sec)        No update-in-place overhead
                            
2. Time-series queries    ✅ Clustering key = timestamp (sorted by time)
   (get last N minutes)     Efficient range scans

3. Horizontal scalability ✅ Partition by driver_id (evenly distributed)
   (grow to 1M drivers)     Add nodes without downtime

4. High availability      ✅ Replication factor 3 (no single point of failure)
   (99.9% uptime)           Survives node failures

5. Time-based expiration  ✅ TTL (Time To Live) auto-deletes old data
   (30-day retention)       No manual cleanup needed

6. Large data volumes     ✅ Designed for TB-scale datasets
   (25 TB/month)            Efficient compression

7. Predictable latency    ✅ O(1) writes, O(log N) reads
   (p95 < 50ms)             No hot spots with proper partitioning
```

**Cassandra Schema Design:**

```sql
-- Time-series table for driver locations
CREATE TABLE driver_locations (
    driver_id BIGINT,
    timestamp TIMESTAMP,
    latitude DECIMAL(10, 8),     -- 8 decimal places = ~1mm precision
    longitude DECIMAL(11, 8),
    accuracy DECIMAL(5, 2),      -- GPS accuracy in meters (e.g., 5.50m)
    speed DECIMAL(5, 2),         -- Speed in km/h (e.g., 45.30)
    bearing DECIMAL(5, 2),       -- Direction 0-360 degrees (e.g., 135.50)
    battery_level INT,           -- Driver's phone battery % (for support)
    
    PRIMARY KEY (driver_id, timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC)
  AND compaction = {
      'class': 'TimeWindowCompactionStrategy',
      'compaction_window_size': '1',
      'compaction_window_unit': 'DAYS'
  }
  AND default_time_to_live = 2592000;  -- 30 days TTL (auto-delete)

-- Explanation of design decisions:
-- 1. Partition key = driver_id
--    - All locations for same driver stored together
--    - Evenly distributed (200K drivers = 200K partitions)
--    - Query by driver_id is O(1)
--
-- 2. Clustering key = timestamp (DESC)
--    - Locations sorted newest-first within partition
--    - Range queries efficient: "get last 10 minutes"
--    - No need to sort results
--
-- 3. Time-series compaction strategy
--    - Groups data by time window (1 day)
--    - Old data compacted separately from new data
--    - Efficient for time-based queries
--
-- 4. TTL = 30 days
--    - Automatically deletes data older than 30 days
--    - No manual cleanup needed
--    - Saves storage costs
```

**Write Pattern (High Throughput):**

```python
from cassandra.cluster import Cluster
from cassandra.query import BatchStatement
import datetime

cluster = Cluster(['cassandra-node-1', 'cassandra-node-2', 'cassandra-node-3'])
session = cluster.connect('food_delivery')

def write_location_update(driver_id, location_data):
    """
    Write driver location to Cassandra
    Called every 1 second from driver app
    """
    query = """
        INSERT INTO driver_locations 
        (driver_id, timestamp, latitude, longitude, accuracy, speed, bearing, battery_level)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
    """
    
    prepared_stmt = session.prepare(query)
    
    session.execute(prepared_stmt, (
        driver_id,
        datetime.datetime.utcnow(),
        location_data['latitude'],
        location_data['longitude'],
        location_data['accuracy'],
        location_data['speed'],
        location_data['bearing'],
        location_data['battery_level']
    ))
    
    # Note: Cassandra write is O(1) and takes ~2-5ms
    # No indexes to update, just append to commit log

# Batch writes for efficiency (from backend service)
def batch_write_locations(location_updates):
    """
    Write multiple locations in a batch (up to 50 per batch)
    """
    batch = BatchStatement()
    prepared_stmt = session.prepare("""
        INSERT INTO driver_locations 
        (driver_id, timestamp, latitude, longitude, accuracy, speed, bearing, battery_level)
        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
    """)
    
    for update in location_updates[:50]:  # Max 50 statements per batch
        batch.add(prepared_stmt, (
            update['driver_id'],
            update['timestamp'],
            update['latitude'],
            update['longitude'],
            update['accuracy'],
            update['speed'],
            update['bearing'],
            update['battery_level']
        ))
    
    session.execute(batch)
```

**Read Pattern (Time-Range Queries):**

```python
def get_recent_locations(driver_id, minutes=10):
    """
    Get driver's locations from last N minutes
    Used for showing route history or debugging
    """
    cutoff_time = datetime.datetime.utcnow() - datetime.timedelta(minutes=minutes)
    
    query = """
        SELECT timestamp, latitude, longitude, speed, bearing
        FROM driver_locations
        WHERE driver_id = ?
          AND timestamp > ?
        ORDER BY timestamp DESC
    """
    
    rows = session.execute(query, (driver_id, cutoff_time))
    
    locations = []
    for row in rows:
        locations.append({
            'timestamp': row.timestamp.isoformat(),
            'latitude': float(row.latitude),
            'longitude': float(row.longitude),
            'speed': float(row.speed),
            'bearing': float(row.bearing)
        })
    
    return locations

# Example result (driver's last 10 minutes):
# [
#   {'timestamp': '2025-01-04T19:25:30Z', 'latitude': 40.7500, 'longitude': -73.9900, ...},
#   {'timestamp': '2025-01-04T19:25:29Z', 'latitude': 40.7499, 'longitude': -73.9901, ...},
#   ...600 rows (10 min × 60 sec/min = 600 location points)
# ]
```

**Why NOT Use These Alternatives:**

```text
ALTERNATIVE          WHY NOT?
─────────────────────────────────────────────────────────────────────
PostgreSQL + TimescaleDB
├─ Write throughput   ❌ ~50K writes/sec (not enough for 200K)
├─ Scalability        ❌ Vertical scaling only (limited to single node)
└─ Cost               ❌ Expensive high-memory servers needed

MongoDB
├─ Write throughput   ⚠️ ~80K writes/sec (borderline)
├─ Time-series        ❌ No native time-series optimization until v5.0
└─ Sharding           ⚠️ Complex shard key management

InfluxDB (Time-Series DB)
├─ Write throughput   ✅ Excellent (300K+ writes/sec)
├─ Time-series        ✅ Optimized for time-series
├─ Scalability        ⚠️ Clustering is enterprise-only ($$)
└─ Query language     ❌ Non-standard (Flux/InfluxQL, not SQL)

Cassandra
├─ Write throughput   ✅ 500K+ writes/sec (plenty of headroom)
├─ Time-series        ✅ Native support with TWCS compaction
├─ Scalability        ✅ Linear horizontal scaling
├─ Cost               ✅ Open-source, commodity hardware
└─ Industry proven    ✅ Used by Apple, Netflix, Uber
```

**Storage Optimization:**

```text
Compression: Cassandra's LZ4 compression
├─ Uncompressed: 50 bytes per location × 200K drivers × 86400 sec/day = 864 GB/day
├─ Compressed: ~15 bytes per location (70% compression)
└─ Actual storage: 260 GB/day = 7.8 TB/month (vs 25.9 TB uncompressed)

Cost Savings:
├─ AWS EBS SSD: $0.10/GB-month
├─ Uncompressed: 25.9 TB × $0.10 = $2,590/month
├─ Compressed: 7.8 TB × $0.10 = $780/month
└─ Savings: $1,810/month = $21,720/year 💰
```

**Interview Tip:** Emphasize that the choice of Cassandra is driven by **workload characteristics**: write-heavy, time-series, high availability, and massive scale. Always justify database choice with concrete numbers (200K writes/sec, 25 TB/month) rather than saying "Cassandra is good for time-series."

</details>

#### Intermediate Level

**Q4:** How do you validate GPS data to detect anomalies like spoofing or signal errors?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**GPS validation** prevents fraud (drivers faking location), detects technical issues (signal loss), and ensures data quality for accurate ETAs. We use multi-layered validation: speed checks, geofencing, cellular triangulation cross-checks, and anomaly detection.

**Why GPS Validation Matters:**

```text
PROBLEM: Unreliable GPS Data

Scenario 1: Driver Fraud (GPS Spoofing)
├─ Driver uses fake GPS app
├─ Appears to be at customer location without actually going there
├─ Marks order "delivered" and keeps food
├─ Impact: Customer charged $35, no food, angry customer

Scenario 2: GPS Signal Loss (Tunnel)
├─ Driver enters tunnel, GPS signal lost
├─ Location freezes at tunnel entrance
├─ Customer sees: "Driver hasn't moved in 5 minutes"
├─ Impact: Worried customer, support tickets, bad experience

Scenario 3: GPS Jitter (Urban Canyon)
├─ Tall buildings reflect GPS signals
├─ Location bounces wildly: 40.7500 → 40.7520 → 40.7495
├─ Customer sees driver "teleporting" on map
├─ Impact: Confusing UX, inaccurate ETA

Solution: Multi-layer GPS validation
```

**Validation Layer 1: Speed Check (Impossible Movement)**

```python
def validate_speed(driver_id, new_location, timestamp):
    """
    Check if implied speed is physically possible
    Reject if speed > 120 km/h (highway speed limit)
    """
    # Get last known location
    last_location = cassandra.query(
        "SELECT latitude, longitude, timestamp FROM driver_locations "
        "WHERE driver_id = ? ORDER BY timestamp DESC LIMIT 1",
        [driver_id]
    )[0]
    
    # Calculate distance moved (Haversine formula)
    distance_km = haversine_distance(
        last_location.latitude,
        last_location.longitude,
        new_location.latitude,
        new_location.longitude
    )
    
    # Calculate time elapsed
    time_diff_sec = (timestamp - last_location.timestamp).total_seconds()
    
    # Calculate implied speed
    if time_diff_sec > 0:
        speed_kmh = (distance_km / time_diff_sec) * 3600
    else:
        speed_kmh = 0
    
    # Validation thresholds
    if speed_kmh > 120:
        # REJECT: Impossible speed (driver can't teleport 10 km in 1 second)
        log_warning(
            f"GPS anomaly: Driver {driver_id} speed {speed_kmh:.1f} km/h "
            f"(moved {distance_km:.2f} km in {time_diff_sec:.1f} sec)"
        )
        metrics.increment('gps.validation.speed_rejected')
        
        # Use last known good location instead
        return {
            'valid': False,
            'reason': 'impossible_speed',
            'fallback_location': last_location,
            'rejected_speed_kmh': speed_kmh
        }
    
    elif speed_kmh > 80:
        # WARNING: High speed (highway driving, but acceptable)
        log_info(f"Driver {driver_id} traveling at {speed_kmh:.1f} km/h (highway)")
        return {'valid': True, 'warning': 'high_speed'}
    
    else:
        # VALID: Normal city driving speed
        return {'valid': True}

# Example: Detect GPS spoofing
# Last location: Central Park (40.7829°N, -73.9654°W) at 19:00:00
# New location:  Times Square (40.7580°N, -73.9855°W) at 19:00:02
# Distance: 3.2 km in 2 seconds = 5,760 km/h (faster than airplane!)
# → REJECTED (GPS spoofing detected)
```

**Validation Layer 2: Geofencing (Service Area Check)**

```python
def validate_service_area(driver_id, location):
    """
    Check if driver is within service area
    Reject locations far outside city boundaries
    """
    # Define service area (NYC example)
    NYC_BOUNDS = {
        'min_lat': 40.4774,  # Southern boundary (Staten Island)
        'max_lat': 40.9176,  # Northern boundary (Bronx)
        'min_lon': -74.2591, # Western boundary (Staten Island)
        'max_lon': -73.7004  # Eastern boundary (Queens)
    }
    
    # Check if location is within bounds
    in_bounds = (
        NYC_BOUNDS['min_lat'] <= location.latitude <= NYC_BOUNDS['max_lat'] and
        NYC_BOUNDS['min_lon'] <= location.longitude <= NYC_BOUNDS['max_lon']
    )
    
    if not in_bounds:
        # REJECT: Driver location outside NYC
        log_error(
            f"GPS anomaly: Driver {driver_id} location ({location.latitude}, "
            f"{location.longitude}) outside service area"
        )
        metrics.increment('gps.validation.out_of_bounds')
        
        return {
            'valid': False,
            'reason': 'outside_service_area',
            'location': (location.latitude, location.longitude)
        }
    
    return {'valid': True}

# Example: Detect spoofing to different city
# Driver registered in NYC but GPS shows Los Angeles coordinates
# → REJECTED (driver can't be 4,000 km away)
```

**Validation Layer 3: Cellular Triangulation Cross-Check**

```python
def validate_with_cellular(driver_id, gps_location, cellular_towers):
    """
    Cross-check GPS with cellular tower triangulation
    Cellular is harder to spoof than GPS
    """
    # Get driver's connected cellular towers (from telco API)
    tower_locations = []
    for tower_id in cellular_towers:
        tower_data = cellular_api.get_tower_location(tower_id)
        tower_locations.append({
            'latitude': tower_data.latitude,
            'longitude': tower_data.longitude,
            'signal_strength': tower_data.signal_strength
        })
    
    # Triangulate approximate location from cellular towers
    cellular_location = triangulate_location(tower_locations)
    
    # Calculate distance between GPS and cellular estimate
    discrepancy_km = haversine_distance(
        gps_location.latitude,
        gps_location.longitude,
        cellular_location.latitude,
        cellular_location.longitude
    )
    
    # Validation threshold
    if discrepancy_km > 2.0:
        # WARNING: GPS and cellular don't match
        # GPS says driver at Times Square, cellular says driver in Brooklyn
        log_warning(
            f"GPS/cellular mismatch for driver {driver_id}: "
            f"{discrepancy_km:.2f} km discrepancy"
        )
        metrics.increment('gps.validation.cellular_mismatch')
        
        return {
            'valid': True,  # Don't reject, but flag for review
            'warning': 'cellular_mismatch',
            'gps_location': gps_location,
            'cellular_location': cellular_location,
            'discrepancy_km': discrepancy_km
        }
    
    return {'valid': True, 'verified_by_cellular': True}

# Example: Detect GPS spoofing
# GPS says: Times Square (40.7580°N, -73.9855°W)
# Cellular towers say: Brooklyn (40.6782°N, -73.9442°W)
# Discrepancy: 10 km
# → FLAG for manual review (possible spoofing)
```

**Validation Layer 4: Pattern Analysis (Anomaly Detection)**

```python
class GPSAnomalyDetector:
    """
    Machine learning model to detect unusual GPS patterns
    Trained on historical data to identify normal vs abnormal behavior
    """
    
    def __init__(self):
        self.model = load_trained_model()  # Pre-trained ML model
    
    def detect_anomalies(self, driver_id, location_history):
        """
        Analyze recent location history for anomalies
        """
        # Extract features from last 10 minutes
        features = self.extract_features(driver_id, location_history)
        
        # Predict if behavior is anomalous
        anomaly_score = self.model.predict_proba([features])[0][1]
        
        # Threshold: >0.8 probability = likely anomaly
        if anomaly_score > 0.8:
            # Investigate further
            anomaly_type = self.classify_anomaly(location_history)
            
            log_warning(
                f"GPS anomaly detected for driver {driver_id}: "
                f"score={anomaly_score:.2f}, type={anomaly_type}"
            )
            
            return {
                'valid': False,
                'reason': 'anomaly_detected',
                'anomaly_type': anomaly_type,
                'anomaly_score': anomaly_score
            }
        
        return {'valid': True}
    
    def extract_features(self, driver_id, location_history):
        """
        Extract features for ML model
        """
        # Feature 1: Average speed
        speeds = [loc.speed for loc in location_history]
        avg_speed = sum(speeds) / len(speeds)
        
        # Feature 2: Speed variance (how much speed changes)
        speed_variance = variance(speeds)
        
        # Feature 3: Direction changes (how often driver turns)
        direction_changes = count_direction_changes(location_history)
        
        # Feature 4: Stop frequency (how often driver stops)
        stops = count_stops(location_history, threshold_kmh=5)
        
        # Feature 5: Route efficiency (actual distance vs optimal)
        actual_distance = calculate_path_distance(location_history)
        optimal_distance = haversine_distance(
            location_history[0].latitude,
            location_history[0].longitude,
            location_history[-1].latitude,
            location_history[-1].longitude
        )
        route_efficiency = optimal_distance / actual_distance
        
        return {
            'avg_speed': avg_speed,
            'speed_variance': speed_variance,
            'direction_changes': direction_changes,
            'stops': stops,
            'route_efficiency': route_efficiency
        }
    
    def classify_anomaly(self, location_history):
        """
        Classify type of anomaly
        """
        # Anomaly Pattern 1: Teleportation
        # Large jumps in location (GPS spoofing)
        max_jump = max(calculate_distances(location_history))
        if max_jump > 1.0:  # 1 km jump
            return 'teleportation'
        
        # Anomaly Pattern 2: Stationary while supposedly driving
        # Speed reported as 0 for extended period
        stationary_duration = calculate_stationary_duration(location_history)
        if stationary_duration > 300:  # 5 minutes
            return 'stationary_too_long'
        
        # Anomaly Pattern 3: Unrealistic route
        # Driver going in circles or wrong direction
        route_efficiency = calculate_route_efficiency(location_history)
        if route_efficiency < 0.5:  # Taking 2x longer route than necessary
            return 'inefficient_route'
        
        return 'unknown'

# Usage
detector = GPSAnomalyDetector()
recent_locations = get_recent_locations(driver_id=12345, minutes=10)
result = detector.detect_anomalies(driver_id=12345, location_history=recent_locations)

if not result['valid']:
    # Flag driver for manual review
    flag_driver_for_review(driver_id=12345, reason=result['anomaly_type'])
    # Temporarily pause driver account pending investigation
    suspend_driver(driver_id=12345, duration_hours=24)
```

**Real-World Fraud Detection Stats:**

```text
Uber Eats GPS Fraud Detection (2024):
├─ GPS spoofing attempts: 0.3% of deliveries (30K/day at 10M orders)
├─ Detected by speed check: 85% (25.5K/day)
├─ Detected by cellular mismatch: 10% (3K/day)
├─ Detected by ML anomaly detection: 5% (1.5K/day)
└─ Total prevented fraud: $35 × 30K = $1.05M/day = $383M/year!

False Positive Rate:
├─ Legitimate drivers flagged: 0.05% (5K/day)
├─ Manual review time: 2 min per case
├─ Support cost: 5K × 2 min = 167 hours/day = $5,000/day
└─ Still profitable: $1.05M saved - $5K cost = $1.045M net benefit/day
```

**Interview Tip:** Emphasize **defense in depth**—use multiple validation layers because no single check is perfect. Speed checks catch obvious spoofing, cellular cross-checks catch sophisticated spoofing, and ML catches novel attack patterns. Also mention that you need to balance **security vs user experience**—too aggressive validation blocks legitimate drivers with poor GPS signal.

</details>

**Q5:** How do you handle location privacy and GDPR compliance for driver data?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**Location privacy** is critical for GDPR compliance and driver trust. We use **data minimization** (only collect what's needed), **time-based degradation** (reduce precision over time), **anonymization** (remove PII), and **right to erasure** (allow drivers to delete data).

**GDPR Requirements for Location Data:**

```text
GDPR Article 5 Principles:
─────────────────────────────────────────────────────────────────
1. Purpose Limitation
   ✅ Only collect location data for legitimate business purpose
   ❌ Can't use delivery location data for advertising targeting

2. Data Minimization
   ✅ Only collect necessary precision (10m accuracy, not 10cm)
   ❌ Don't store GPS data when driver is offline

3. Storage Limitation
   ✅ Don't keep precise data longer than necessary
   ❌ Can't store exact home address indefinitely

4. Accuracy
   ✅ Allow drivers to correct incorrect location data
   
5. Integrity & Confidentiality
   ✅ Encrypt data at rest and in transit
   ✅ Access controls (only authorized employees can view)

6. Accountability
   ✅ Document why location data is collected and how it's used
   ✅ Regular audits of data retention and deletion
```

**Privacy-Preserving Architecture:**

```text
DATA LIFECYCLE           PRECISION LEVEL         PURPOSE
─────────────────────────────────────────────────────────────────────
0-24 hours (active)      6 decimals (~10cm)     Real-time tracking
├─ Use case: Live delivery tracking for customer
├─ Retention: Memory cache (Redis) + Cassandra
└─ Access: Customer (for their orders), support (for disputes)

1-7 days (recent)        5 decimals (~1m)       Dispute resolution
├─ Use case: Customer claims "food never delivered"
├─ Retention: Cassandra (hot storage)
└─ Access: Support team only (with audit log)

7-30 days (historical)   4 decimals (~10m)      Analytics, route optimization
├─ Use case: Improve ETA algorithms, traffic patterns
├─ Retention: Cassandra (with reduced precision)
└─ Access: Data science team (aggregated queries only)

30+ days (archived)      3 decimals (~100m)     City-level analytics
├─ Use case: "How many deliveries in Manhattan last month?"
├─ Retention: S3 cold storage
└─ Access: Business intelligence (no individual driver data)

1+ year                  Deleted or 2 decimals  Aggregate statistics only
├─ Use case: "Average delivery speed in NYC 2024 vs 2023"
├─ Retention: Data warehouse (aggregated, anonymized)
└─ Access: Executives, not tied to individual drivers
```

**Implementation: Time-Based Precision Degradation**

```python
class LocationPrivacyManager:
    """
    Automatically reduce location precision over time for privacy
    """
    
    def __init__(self):
        self.cassandra_session = get_cassandra_session()
    
    def degrade_old_locations(self):
        """
        Daily job to reduce precision of old location data
        """
        # Stage 1: 7-30 days old → Reduce to 4 decimals (~10m)
        self.degrade_locations(
            min_age_days=7,
            max_age_days=30,
            target_precision=4  # 4 decimal places
        )
        
        # Stage 2: 30+ days old → Reduce to 3 decimals (~100m)
        self.degrade_locations(
            min_age_days=30,
            max_age_days=365,
            target_precision=3
        )
        
        # Stage 3: 1+ year old → Delete completely
        self.delete_old_locations(min_age_days=365)
    
    def degrade_locations(self, min_age_days, max_age_days, target_precision):
        """
        Reduce precision of location coordinates
        """
        # Calculate date range
        max_date = datetime.utcnow() - timedelta(days=min_age_days)
        min_date = datetime.utcnow() - timedelta(days=max_age_days)
        
        # Query old locations
        locations = self.cassandra_session.execute(
            """
            SELECT driver_id, timestamp, latitude, longitude
            FROM driver_locations
            WHERE timestamp < ? AND timestamp > ?
            ALLOW FILTERING
            """,
            (max_date, min_date)
        )
        
        # Reduce precision
        for location in locations:
            degraded_lat = round_to_precision(location.latitude, target_precision)
            degraded_lon = round_to_precision(location.longitude, target_precision)
            
            # Update with reduced precision
            self.cassandra_session.execute(
                """
                UPDATE driver_locations
                SET latitude = ?, longitude = ?
                WHERE driver_id = ? AND timestamp = ?
                """,
                (degraded_lat, degraded_lon, location.driver_id, location.timestamp)
            )
        
        log_info(
            f"Degraded {len(locations)} locations to {target_precision} decimals"
        )
    
    def delete_old_locations(self, min_age_days):
        """
        Delete locations older than specified age
        """
        cutoff_date = datetime.utcnow() - timedelta(days=min_age_days)
        
        # Cassandra has TTL for automatic deletion, but manual cleanup for certainty
        deleted_count = self.cassandra_session.execute(
            """
            DELETE FROM driver_locations
            WHERE timestamp < ?
            ALLOW FILTERING
            """,
            (cutoff_date,)
        )
        
        log_info(f"Deleted {deleted_count} locations older than {min_age_days} days")
        metrics.increment('privacy.locations_deleted', value=deleted_count)

def round_to_precision(coordinate, decimal_places):
    """
    Round coordinate to specified precision
    
    Examples:
    - 40.7580123 → 40.758 (3 decimals = ~100m)
    - 40.7580123 → 40.7580 (4 decimals = ~10m)
    - 40.7580123 → 40.75801 (5 decimals = ~1m)
    """
    multiplier = 10 ** decimal_places
    return round(coordinate * multiplier) / multiplier

# Example:
# Original: 40.7580123, -73.9855456 (driver's exact location at home)
# After 30 days: 40.758, -73.986 (approximate neighborhood)
# After 1 year: DELETED (no longer stored)
```

**GDPR "Right to Erasure" (Right to be Forgotten):**

```python
def process_erasure_request(driver_id, request_id):
    """
    Handle GDPR Article 17 "Right to Erasure" request
    Driver requests deletion of all personal data
    """
    log_info(f"Processing erasure request {request_id} for driver {driver_id}")
    
    # Step 1: Delete all location history
    cassandra_session.execute(
        "DELETE FROM driver_locations WHERE driver_id = ?",
        (driver_id,)
    )
    
    # Step 2: Delete from backup archives
    s3_client.delete_objects(
        Bucket='food-delivery-backups',
        Delete={
            'Objects': [
                {'Key': f'locations/driver_{driver_id}/*.parquet'}
            ]
        }
    )
    
    # Step 3: Remove from analytics data warehouse
    # Replace driver_id with anonymous hash for historical reports
    anonymous_id = hashlib.sha256(str(driver_id).encode()).hexdigest()[:16]
    redshift_client.execute(
        """
        UPDATE delivery_analytics
        SET driver_id = ?, is_anonymized = true
        WHERE driver_id = ?
        """,
        (anonymous_id, driver_id)
    )
    
    # Step 4: Log erasure for compliance audit
    audit_log_db.insert({
        'request_id': request_id,
        'driver_id': driver_id,
        'request_date': datetime.utcnow(),
        'completed_date': datetime.utcnow(),
        'data_deleted': [
            'driver_locations (all records)',
            'backup_archives (S3)',
            'analytics_data (anonymized)'
        ]
    })
    
    # Step 5: Notify driver of completion
    notification_service.send_email(
        to=driver_email,
        subject="Your data deletion request is complete",
        body="All your personal location data has been deleted from our systems."
    )
    
    metrics.increment('gdpr.erasure_requests_completed')
    log_info(f"Erasure request {request_id} completed for driver {driver_id}")

# Real-world stats:
# - Erasure requests: ~0.5% of drivers per year (5K drivers at 1M total)
# - Processing time: 24-48 hours (due to backup archive deletion)
# - Cost per request: ~$2 (mostly manual verification labor)
```

**Access Controls & Audit Logging:**

```python
class LocationAccessControl:
    """
    Control who can access sensitive location data
    Log all access for GDPR accountability
    """
    
    ROLES = {
        'customer': ['view_own_order_locations'],
        'support_agent': ['view_dispute_locations'],
        'data_scientist': ['view_aggregated_only'],
        'operations_manager': ['view_real_time_dashboard'],
        'compliance_officer': ['view_audit_logs']
    }
    
    def can_access_location(self, user_id, role, driver_id, timestamp):
        """
        Check if user is authorized to access location data
        """
        # Customer can only see their own orders
        if role == 'customer':
            order = db.get_order_for_customer(user_id)
            if order.driver_id != driver_id:
                return False  # Not their driver
            if datetime.utcnow() - order.delivered_at > timedelta(hours=24):
                return False  # Order completed >24h ago
            return True
        
        # Support agent can see recent data for disputes
        elif role == 'support_agent':
            if datetime.utcnow() - timestamp > timedelta(days=7):
                return False  # Too old, no longer accessible
            return True
        
        # Data scientist can only see aggregated data
        elif role == 'data_scientist':
            return False  # No access to individual locations
        
        return False
    
    def log_access(self, user_id, role, driver_id, timestamp, purpose):
        """
        Log every access to location data for audit trail
        """
        audit_log_db.insert({
            'access_time': datetime.utcnow(),
            'user_id': user_id,
            'user_role': role,
            'driver_id': driver_id,
            'timestamp_accessed': timestamp,
            'purpose': purpose,
            'ip_address': get_client_ip()
        })
    
    def access_location(self, user_id, role, driver_id, timestamp, purpose):
        """
        Controlled access to location data with authorization and logging
        """
        # Check authorization
        if not self.can_access_location(user_id, role, driver_id, timestamp):
            log_warning(
                f"Unauthorized location access attempt: "
                f"user={user_id}, role={role}, driver={driver_id}"
            )
            raise PermissionError("Access denied to location data")
        
        # Log access for audit
        self.log_access(user_id, role, driver_id, timestamp, purpose)
        
        # Retrieve location data
        location = cassandra_session.execute(
            """
            SELECT latitude, longitude, timestamp
            FROM driver_locations
            WHERE driver_id = ? AND timestamp = ?
            """,
            (driver_id, timestamp)
        )
        
        return location

# Usage:
access_control = LocationAccessControl()

# Customer viewing their delivery
location = access_control.access_location(
    user_id=customer_id,
    role='customer',
    driver_id=12345,
    timestamp=datetime.utcnow(),
    purpose='track_delivery'
)

# Support agent investigating dispute
location = access_control.access_location(
    user_id=support_agent_id,
    role='support_agent',
    driver_id=12345,
    timestamp=datetime.utcnow() - timedelta(days=3),
    purpose='dispute_resolution_ticket_7890'
)
```

**Privacy Impact Assessment:**

```text
DATA BREACH SCENARIO: Cassandra database exposed

WITHOUT Privacy Controls:
├─ Exposed: 200K drivers × 86400 locations/day × 365 days = 6.3 billion locations
├─ Precision: 6 decimals (~10cm) - exact home addresses visible
├─ Impact: SEVERE - drivers' home addresses, travel patterns, work schedules exposed
├─ GDPR fine: Up to 4% of global revenue (~$400M for Uber Eats)
└─ Reputational damage: Drivers quit platform, media backlash

WITH Privacy Controls:
├─ Exposed: Only last 24 hours at full precision = 17.3 million locations
├─ Older data: Degraded to 3 decimals (~100m) - approximate neighborhoods only
├─ Impact: LIMITED - recent routes visible, but not home addresses
├─ GDPR fine: Likely avoided (demonstrated reasonable safeguards)
└─ Reputational damage: Minimal (proactive privacy measures respected)
```

**Interview Tip:** Frame location privacy as both **legal compliance** (avoid GDPR fines) and **business advantage** (driver trust and retention). Drivers are more likely to stay on a platform that respects their privacy. Also mention that privacy-by-design is cheaper than retrofitting—build degradation and access controls from day one.

</details>

#### Advanced Level

**Q6:** Design a system to detect and handle when a driver goes through a tunnel or loses GPS signal during delivery. How do you maintain customer UX?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:**

**GPS signal loss** (tunnels, parking garages, dense urban areas) requires **dead reckoning** (extrapolate position), **route prediction** (estimate tunnel exit), **transparent UX** (tell customer what's happening), and **fallback tracking** (cellular towers, WiFi positioning).

**The Problem: GPS Blackout**

```text
Real-World Scenario: Lincoln Tunnel (NYC ↔ NJ)

Timeline:
19:15:00 - Driver enters tunnel (Manhattan side)
├─ Last GPS: 40.7693°N, -73.9995°W
├─ Speed: 45 km/h
├─ Direction: 270° (west)
└─ Customer sees: Driver approaching delivery

19:15:01 - GPS signal lost (underground)
├─ No GPS updates for next 5 minutes
├─ Customer sees: Driver frozen at tunnel entrance
└─ Customer thinks: "Why hasn't driver moved? Is something wrong?"

19:15:30 - Customer refreshes map frantically
├─ Still no update (driver still in tunnel)
├─ Customer anxiety increasing
└─ Customer considers: Calling support, canceling order

19:20:00 - Driver exits tunnel (New Jersey side)
├─ GPS signal restored: 40.7587°N, -74.0249°W
├─ Customer sees: Driver suddenly "teleported" 3 km
└─ Customer confused: "How did driver get there so fast?"

PROBLEMS:
❌ 5-minute gap in location tracking
❌ Customer anxiety (thinks delivery is delayed or lost)
❌ Sudden position jump (poor UX, looks like a bug)
❌ ETA completely wrong during tunnel (still based on last known position)
```

**Solution: Dead Reckoning + Route Prediction**

```python
class GPSSignalLossHandler:
    """
    Handle GPS signal loss using dead reckoning and route prediction
    """
    
    def __init__(self):
        self.gmaps_client = googlemaps.Client(key='YOUR_API_KEY')
        self.redis_client = redis.Redis()
    
    def detect_signal_loss(self, driver_id):
        """
        Detect when GPS signal is lost (no updates for >30 seconds)
        """
        last_update = self.redis_client.get(f'last_gps_update:{driver_id}')
        
        if last_update:
            last_update_time = datetime.fromisoformat(last_update.decode())
            time_since_update = (datetime.utcnow() - last_update_time).total_seconds()
            
            if time_since_update > 30:
                # Signal lost!
                return True, time_since_update
        
        return False, 0
    
    def dead_reckoning(self, driver_id, last_known_location, route):
        """
        Estimate current position based on last known location, speed, and route
        """
        # Get last known state
        last_speed_kmh = last_known_location.speed
        last_bearing = last_known_location.bearing
        time_elapsed_sec = (datetime.utcnow() - last_known_location.timestamp).total_seconds()
        
        # Calculate distance traveled (assuming constant speed)
        distance_traveled_km = (last_speed_kmh / 3600) * time_elapsed_sec
        
        # Estimate position along route
        estimated_position = self.extrapolate_position(
            start_location=(last_known_location.latitude, last_known_location.longitude),
            bearing=last_bearing,
            distance_km=distance_traveled_km,
            route_polyline=route.overview_polyline
        )
        
        return estimated_position
    
    def extrapolate_position(self, start_location, bearing, distance_km, route_polyline):
        """
        Calculate estimated position along route polyline
        """
        # Decode route polyline (list of lat/lon points along route)
        route_points = googlemaps.convert.decode_polyline(route_polyline)
        
        # Find closest point on route to start_location
        closest_idx = self.find_closest_point_on_route(start_location, route_points)
        
        # Travel along route from closest point
        distance_remaining = distance_km
        current_idx = closest_idx
        
        while distance_remaining > 0 and current_idx < len(route_points) - 1:
            # Distance to next point
            segment_distance = haversine_distance(
                route_points[current_idx][0],
                route_points[current_idx][1],
                route_points[current_idx + 1][0],
                route_points[current_idx + 1][1]
            )
            
            if distance_remaining <= segment_distance:
                # Interpolate position within this segment
                fraction = distance_remaining / segment_distance
                estimated_lat = (
                    route_points[current_idx][0] +
                    (route_points[current_idx + 1][0] - route_points[current_idx][0]) * fraction
                )
                estimated_lon = (
                    route_points[current_idx][1] +
                    (route_points[current_idx + 1][1] - route_points[current_idx][1]) * fraction
                )
                return (estimated_lat, estimated_lon)
            
            distance_remaining -= segment_distance
            current_idx += 1
        
        # Reached end of route
        return route_points[-1]
    
    def handle_signal_loss(self, driver_id, order_id):
        """
        Main handler for GPS signal loss
        """
        # Get last known location
        last_location = cassandra.query(
            """
            SELECT * FROM driver_locations
            WHERE driver_id = ?
            ORDER BY timestamp DESC
            LIMIT 1
            """,
            [driver_id]
        )[0]
        
        # Get planned route
        order = db.get_order(order_id)
        route = self.get_route(last_location, order.delivery_address)
        
        # Check if driver is likely in a known tunnel
        tunnel_info = self.check_if_in_tunnel(last_location, route)
        
        if tunnel_info:
            # Driver is in a tunnel, predict exit
            estimated_exit_time = tunnel_info['avg_transit_time']  # e.g., 5 minutes
            exit_location = tunnel_info['exit_location']
            
            # Notify customer with transparent message
            notification_service.send_to_customer(
                user_id=order.customer_id,
                message={
                    'type': 'driver_in_tunnel',
                    'tunnel_name': tunnel_info['name'],
                    'estimated_exit_time': estimated_exit_time,
                    'message': (
                        f"Your driver is in {tunnel_info['name']}. "
                        f"Location will update when they exit in ~{estimated_exit_time} minutes."
                    )
                }
            )
            
            # Show estimated position on customer's map (move along tunnel route)
            self.animate_tunnel_transit(
                order_id=order_id,
                entry_location=last_location,
                exit_location=exit_location,
                transit_time_sec=estimated_exit_time * 60
            )
        
        else:
            # Not in a known tunnel, use dead reckoning
            estimated_position = self.dead_reckoning(driver_id, last_location, route)
            
            # Show estimated position with reduced confidence
            notification_service.send_to_customer(
                user_id=order.customer_id,
                message={
                    'type': 'gps_signal_weak',
                    'message': (
                        "Driver's GPS signal is temporarily weak. "
                        "Location is estimated and will update shortly."
                    ),
                    'estimated_location': estimated_position,
                    'confidence': 'low'
                }
            )
        
        # Try fallback positioning methods
        self.try_fallback_positioning(driver_id, order_id)
    
    def check_if_in_tunnel(self, location, route):
        """
        Check if location is near a known tunnel entrance
        """
        KNOWN_TUNNELS = [
            {
                'name': 'Lincoln Tunnel',
                'entry': (40.7693, -73.9995),  # Manhattan entrance
                'exit': (40.7587, -74.0249),   # New Jersey exit
                'avg_transit_time': 5  # minutes
            },
            {
                'name': 'Holland Tunnel',
                'entry': (40.7264, -74.0106),
                'exit': (40.7290, -74.0332),
                'avg_transit_time': 6
            },
            # ... more tunnels
        ]
        
        for tunnel in KNOWN_TUNNELS:
            distance_to_entry = haversine_distance(
                location.latitude,
                location.longitude,
                tunnel['entry'][0],
                tunnel['entry'][1]
            )
            
            if distance_to_entry < 0.2:  # Within 200m of tunnel entrance
                return tunnel
        
        return None
    
    def animate_tunnel_transit(self, order_id, entry_location, exit_location, transit_time_sec):
        """
        Smoothly animate driver icon through tunnel on customer's map
        Even though we have no real GPS data
        """
        start_time = datetime.utcnow()
        
        # Calculate incremental positions
        steps = 60  # Update every second for 60 seconds (1 minute)
        
        for step in range(steps):
            # Linear interpolation between entry and exit
            fraction = step / steps
            
            interpolated_lat = (
                entry_location.latitude +
                (exit_location[0] - entry_location.latitude) * fraction
            )
            interpolated_lon = (
                entry_location.longitude +
                (exit_location[1] - entry_location.longitude) * fraction
            )
            
            # Send estimated position to customer via WebSocket
            websocket_service.broadcast_location_update(
                order_id=order_id,
                location={
                    'latitude': interpolated_lat,
                    'longitude': interpolated_lon,
                    'estimated': True,  # Flag as estimated
                    'confidence': 'low',
                    'reason': 'tunnel_transit'
                }
            )
            
            # Wait 1 second before next update
            time.sleep(1)
    
    def try_fallback_positioning(self, driver_id, order_id):
        """
        Use alternative positioning methods when GPS unavailable
        """
        # Method 1: Cellular tower triangulation
        cellular_location = self.get_cellular_location(driver_id)
        if cellular_location:
            return cellular_location
        
        # Method 2: WiFi positioning
        wifi_location = self.get_wifi_location(driver_id)
        if wifi_location:
            return wifi_location
        
        # Method 3: Last known location + dead reckoning
        return self.dead_reckoning(driver_id, last_location, route)
```

**Customer UX During Signal Loss:**

```text
GOOD UX (Transparent Communication):

Customer sees on map:
┌─────────────────────────────────────────┐
│  🚗 (driver icon, slightly faded)       │
│                                         │
│  🔵 ━━━━━━━━━━━━━━━━━━━━━━━━━> 🏠      │
│  (tunnel route shown as dashed line)   │
│                                         │
│  ℹ️  Driver is in Lincoln Tunnel        │
│     Location will update when they     │
│     exit in ~4 minutes                 │
│                                         │
│  ETA: 15 minutes                       │
└─────────────────────────────────────────┘

Driver icon slowly moves along tunnel route (animated dead reckoning)
Customer understands situation, not anxious

BAD UX (No Communication):

Customer sees:
┌─────────────────────────────────────────┐
│  🚗 (driver icon, stuck at one point)   │
│                                         │
│  🔵─┬─────────────────────────────> 🏠  │
│     ↑                                   │
│  (driver hasn't moved for 5 minutes)   │
│                                         │
│  😰 "Is my driver lost?"                │
│  😰 "Should I call support?"            │
│                                         │
│  ETA: ⏳ Calculating...                 │
└─────────────────────────────────────────┘

Customer is confused and anxious, may complain or cancel
```

**Fallback Positioning Accuracy:**

```text
POSITIONING METHOD        ACCURACY    AVAILABILITY    LATENCY
────────────────────────────────────────────────────────────────
GPS (normal)              5-10m       Outdoors        <1 sec
Cellular triangulation    50-500m     Anywhere        2-5 sec
WiFi positioning          20-100m     Urban areas     3-10 sec
Dead reckoning            100-1000m   Always          <1 sec
Route prediction          500-2000m   With route      <1 sec
```

**Real-World Stats (Uber Eats):**

```text
GPS Signal Loss Frequency:
├─ Tunnels: 2% of deliveries (200K/day at 10M orders)
├─ Parking garages: 5% of deliveries (500K/day)
├─ Dense urban areas: 3% of deliveries (300K/day)
└─ Total affected: 10% of deliveries (1M/day)

Impact of Signal Loss Handling:
├─ WITHOUT dead reckoning: 15% increase in "driver lost?" support tickets
├─ WITH dead reckoning: 3% increase (80% reduction!)
├─ Cost savings: 12% × 1M orders × $5/ticket = $600K/day = $219M/year
```

**Interview Tip:** Emphasize that the key is **transparent communication**—don't hide the problem from the customer. Tell them "Driver is in tunnel, location will update in 4 minutes" rather than pretending everything is fine while the driver icon freezes. Customers appreciate honesty and clarity.

</details>

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

