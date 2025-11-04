# Food Delivery System (Uber Eats/DoorDash) - Quick Reference

**Interview Time: 45 minutes** | **Difficulty: ⭐⭐⭐⭐⭐ Expert** | **Category: Multi-sided Marketplace**

---

## Problem Statement

Design a food delivery platform like Uber Eats that connects customers, restaurants, and delivery drivers. The system should handle 10M daily orders across 500K restaurants and 1M drivers in 100+ cities, with <30 second driver matching and <45 minute delivery times.

---

## 1. Requirements (5 minutes)

### Functional Requirements
- **Customer:** Browse restaurants, order food, track delivery, pay
- **Restaurant:** Manage menu, accept/reject orders, track preparation
- **Driver:** Accept orders, navigate to pickup/delivery, update location
- **Platform:** Match drivers, calculate fees, process payments, monitor performance

### Non-Functional Requirements
- **Scale:** 10M orders/day (115/sec avg, 1,150/sec peak)
- **Performance:** <200ms order placement, <30s driver matching, <1s location updates
- **Availability:** 99.9% uptime (43 min downtime/month)
- **Consistency:** Strong for payments/orders, eventual for locations

---

## 2. Back-of-Envelope Calculations (5 minutes)

### Traffic
- **Daily orders:** 10M
- **Peak QPS:** 1,200 orders/second (10x average during lunch/dinner)
- **Concurrent deliveries:** 200K

### Storage
- **Orders:** 10M/day × 3 KB = 30 GB/day → 10.8 TB/year
- **Location data:** 200K drivers × 2,100 updates/delivery × 50 bytes = 1.05 TB/day
- **Retention:** Orders forever, locations 30 days
- **Total (Year 1):** ~80 TB

### Bandwidth
- **Inbound:** 350 Gbps (location updates, orders)
- **Outbound:** 83 Gbps (mostly CDN for menus/photos)

### Cost
- **Monthly:** ~$410K infrastructure
- **ROI:** 0.02% of revenue ($410K vs $1.875B monthly)

---

## 3. High-Level Design (10 minutes)

```text
┌─────────────────────────────────────────────────────────────┐
│                    API GATEWAY (Kong)                        │
│          Auth, Rate Limiting, Request Routing                │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────────────┐   ┌───────────────┐   ┌──────────────┐
│ Order Service │   │Restaurant Svc │   │ Driver Svc   │
└───────────────┘   └───────────────┘   └──────────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────────────┐
                    │  Kafka Queue  │
                    └───────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────────────┐   ┌──────────────┐   ┌──────────────┐
│ Matching Svc  │   │Location Svc  │   │ Payment Svc  │
│(Driver assign)│   │(GPS tracking)│   │ (Settlement) │
└───────────────┘   └──────────────┘   └──────────────┘
```

### Key Services
1. **Order Service:** CRUD orders, state machine (12 states)
2. **Matching Service:** Geospatial driver assignment (<30s)
3. **Location Service:** Real-time GPS tracking (1-second updates)
4. **Payment Service:** Multi-party settlement (customer, restaurant, driver)
5. **Restaurant Service:** Menu management, inventory
6. **Notification Service:** Push notifications, SMS

---

## 4. Database Design (5 minutes)

### Multi-Database Strategy
```sql
-- PostgreSQL (Transactional Data - Strong Consistency)
orders: id, customer_id, restaurant_id, driver_id, status, total, created_at
order_items: id, order_id, menu_item_id, quantity, price
customers: id, name, email, phone
restaurants: id, name, address, latitude, longitude, rating
drivers: id, name, phone, vehicle_type, rating, is_online

Indexes:
- orders(customer_id, created_at DESC)
- orders(restaurant_id, status)
- orders(driver_id, status)
- restaurants GIST(latitude, longitude)  -- Geospatial index
```

```cql
-- Cassandra (Time-Series Data - High Write Throughput)
driver_locations:
  PRIMARY KEY (driver_id, timestamp)
  latitude, longitude, accuracy, speed
  TTL: 30 days

order_status_history:
  PRIMARY KEY (order_id, timestamp)
  status, actor, reason
```

```text
-- Redis (Caching - Sub-millisecond Reads)
order:{order_id}  → Hash (status, driver_id, restaurant_id)
driver_locations  → Geospatial (GEORADIUS for nearby drivers)
menu:restaurant:{id} → JSON (10-min TTL)
```

### Sharding Strategy
- **Shard by (city_id, order_id % shards_per_city)**
- NYC: 10 shards, Boise: 1 shard
- Enables geo-locality + even distribution

---

## 5. API Design (5 minutes)

### Customer APIs
```http
POST /v1/orders                  # Place order
GET  /v1/orders/{id}             # Track order
PATCH /v1/orders/{id}/cancel     # Cancel order
GET  /v1/restaurants?lat={}&lon={}&radius={}  # Browse

# WebSocket for real-time updates
ws://api.ubereats.com/orders/{id}
  → Receives: order_status, driver_location updates every 1 second
```

### Restaurant APIs
```http
GET  /v1/restaurant/orders?status=PENDING
POST /v1/restaurant/orders/{id}/accept
PATCH /v1/restaurant/menu/items/{id}  # Update availability
```

### Driver APIs
```http
POST /v1/driver/status            # Go online/offline
GET  /v1/driver/orders/available  # Get available orders
POST /v1/driver/orders/{id}/accept
POST /v1/driver/location          # Update GPS (every 1 second)
POST /v1/driver/orders/{id}/pickup
POST /v1/driver/orders/{id}/deliver
```

---

## 6. Deep-Dive: Order State Machine (5 minutes)

```text
PLACED → PAYMENT_PROCESSING → CONFIRMED → ACCEPTED → PREPARING
  → READY → ASSIGNED → PICKED_UP → EN_ROUTE → DELIVERED → COMPLETED

Side paths:
CONFIRMED → REJECTED (restaurant rejects)
PREPARING → CANCELLED (customer cancels, partial refund)
ASSIGNED → ACCEPTED (driver cancels, reassign)
```

**Timeout Handling:**
- CONFIRMED → 5 min timeout → REJECTED (restaurant didn't accept)
- READY → 30 min timeout → Reassign driver
- PICKED_UP → 1 hour timeout → Alert support

**Saga Pattern for Distributed Transaction:**
1. Create order → 2. Reserve inventory → 3. Charge payment → 4. Assign driver
- If any step fails, run compensating transactions (refund, unreserve, cancel)

---

## 7. Deep-Dive: Driver Matching Algorithm (5 minutes)

### Geospatial Search (Redis GEORADIUS)
```python
# Find drivers within 5 km of restaurant
GEORADIUS driver_locations -73.9857 40.7484 5 km WITHDIST
# Result: [(driver:123, 0.8km), (driver:456, 1.5km), ...]
```

### Multi-Factor Scoring
```python
score = (
    distance_score * 0.40 +      # Closer is better
    rating_score * 0.20 +         # Higher rating is better
    acceptance_rate * 0.15 +      # Prefer drivers who accept
    vehicle_suitability * 0.10 +  # Bike for small, car for large
    earnings_fairness * 0.10 +    # Prefer drivers who earned less today
    idle_time * 0.05              # Prefer drivers waiting longer
)
```

### Assignment Flow
1. Find drivers within 5 km
2. Filter by online & available
3. Score all candidates
4. Offer to top 3 drivers (30-second timeout)
5. If all reject → expand radius to 10 km, retry

### Batch Optimization
- Assign multiple orders to one driver
- Solve Traveling Salesman Problem (TSP) for optimal route
- Constraint: Each order delivered within 45 minutes

---

## 8. Deep-Dive: Real-Time Location Tracking (3 minutes)

### Architecture
- **Driver app:** Send GPS every 1 second via HTTPS POST
- **Location Service:** Store in Cassandra, update Redis cache
- **WebSocket Gateway:** Broadcast to customers in real-time
- **Storage:** 200K drivers × 2,100 updates/delivery × 50 bytes = 1.05 TB/day

### ETA Calculation
```python
# Simple: distance / avg_speed
# Advanced: Google Maps Directions API (traffic-aware)
eta_minutes = google_maps.directions(
    origin=driver_location,
    destination=customer_location,
    departure_time="now"
)['duration_in_traffic'] / 60
```

### GPS Validation
- Reject speeds >120 km/h (likely GPS error)
- Smooth jitter with exponential moving average
- Handle tunnels (use last known location)

---

## 9. Trade-Offs & Decisions (2 minutes)

| Decision | Choice | Trade-off |
|----------|--------|-----------|
| **Consistency** | CP for orders, AP for locations | Orders must be accurate, location delay ok |
| **Database** | PostgreSQL + Cassandra + Redis | Complexity vs performance |
| **Architecture** | Microservices | Flexibility vs debugging complexity |
| **Driver Matching** | 30-second timeout | Speed vs finding best driver |
| **Surge Pricing** | Max 3x | Driver incentive vs customer frustration |
| **Location Updates** | 1-second interval | Accuracy vs bandwidth cost |

---

## 10. Bottlenecks & Solutions (3 minutes)

### Bottleneck 1: Database Writes During Peak
- **Problem:** 1,200 orders/sec → 4,800 writes/sec (approaching single DB limit)
- **Solution:** Shard by city (100 shards), async writes via Kafka

### Bottleneck 2: Driver Location Updates
- **Problem:** 200K drivers × 1 update/sec = 200K writes/sec
- **Solution:** Cassandra (optimized for time-series), Redis cache for reads

### Bottleneck 3: Driver Matching Latency
- **Problem:** Scoring all 500 nearby drivers takes >1 second
- **Solution:** Pre-filter to top 20 by distance, score only those

### Bottleneck 4: WebSocket Connections
- **Problem:** 200K concurrent WebSocket connections
- **Solution:** Dedicated WebSocket servers (10K connections per server = 20 servers)

---

## 11. Key Metrics

- **Order Success Rate:** 98%+ (2% cancelled/rejected)
- **Average Delivery Time:** 35 minutes
- **Driver Matching Time:** <30 seconds (95th percentile)
- **Driver Utilization:** 70% (drivers busy 70% of online time)
- **Customer Satisfaction (NPS):** 60+
- **Infrastructure Cost:** $410K/month (0.02% of revenue)

---

## 12. Interview Tips

### Do:
✅ **Ask clarifying questions** (scale, geography, MVP vs full product)
✅ **Start with high-level design** before diving deep
✅ **Use specific numbers** (1,200 QPS, 80 TB storage)
✅ **Discuss trade-offs** (strong vs eventual consistency)
✅ **Consider failure scenarios** (driver cancels, payment fails)

### Don't:
❌ Jump straight into implementation details
❌ Ignore non-functional requirements
❌ Forget to discuss monitoring/alerting
❌ Over-engineer (don't build for 100x scale on day 1)

### Common Follow-Up Questions:
1. "How do you handle a driver going offline mid-delivery?"
   → Reassign to nearby driver, notify customer, compensate original driver
   
2. "How do you prevent restaurants from being overwhelmed?"
   → Rate limiting, max concurrent orders, preparation time estimates
   
3. "How do you detect and prevent fake orders?"
   → ML fraud detection, device fingerprinting, velocity checks

4. "How would you add ghost kitchens (cloud kitchens)?"
   → Virtual restaurant entities, shared physical location, separate menus

---

## 13. Real-World Examples

**Uber Eats (2024):**
- 10M+ daily orders globally
- 800K+ restaurants
- Operating in 6,000+ cities, 45+ countries
- $31B annual GMV (Gross Merchandise Value)

**DoorDash (Market Leader in US):**
- 60%+ US market share
- 550K+ delivery drivers
- 450K+ restaurants

**Technical Innovations:**
- ML demand prediction (20% reduction in delivery times)
- Batched deliveries (30% increase in driver efficiency)
- Dynamic pricing (balances supply/demand within 5 minutes)

---

## References

- Uber Engineering Blog: uber.com/blog/engineering
- "Designing Data-Intensive Applications" - Martin Kleppmann
- DoorDash Engineering Blog: doordash.engineering

---

**Total Prep Time:** 45-60 minutes to master this quick reference  
**Next Practice:** Design Instacart (grocery delivery) or Grubhub
