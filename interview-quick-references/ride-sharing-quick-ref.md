# Ride-Sharing Service (Uber) - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a ride-sharing platform connecting riders with drivers in real-time
- **Key Challenge:** Efficient driver-rider matching with <5 second response time and real-time location tracking
- **Scale:** 10M daily rides, 500K active drivers, 100+ cities globally

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Active Users | 50M riders | Given |
| Active Drivers | 500K | Given |
| Daily Rides | 10M | Given |
| Peak QPS | 278 rides/sec | 1M rides/hour ÷ 3600 |
| Location Updates/sec | 500K | 500K drivers × 60 updates/min ÷ 60 |
| Storage (5 years) | 1.8PB | 10M rides × 100KB × 365 × 5 |
| API Servers Needed | 50 | 50K QPS ÷ 1000 QPS/server |

## 🏗️ High-Level Architecture

```text
[Rider App] -> [Load Balancer] -> [API Gateway] -> [Matching Service] -> [Redis Cache]
     |              |                    |              |
     v              v                    v              v
[Driver App] -> [WebSocket] -> [Location Service] -> [Cassandra]
     |              |                    |              |
     v              v                    v              v
[Web Dashboard] -> [Auth Service] -> [Trip Service] -> [PostgreSQL]
```

**Components:**
- Load Balancer: Nginx, geographic routing
- API Gateway: Kong, rate limiting, authentication
- Matching Service: Go, geospatial algorithms
- Location Service: Go, real-time tracking
- Redis: Driver locations, geospatial indexing
- PostgreSQL: Trip data, ACID compliance
- Cassandra: Location history, time-series data
- Kafka: Event streaming, async processing

## 💾 Data Model (Essentials)

### Core Tables
- **users**: user_id (PK), email, phone, user_type, rating
- **drivers**: driver_id (PK), user_id (FK), status, current_lat/lng
- **trips**: trip_id (PK), rider_id (FK), driver_id (FK), status, fare
- **payments**: payment_id (PK), trip_id (FK), amount, status

### Key Indexes
- drivers: (status, current_lat, current_lng) - for geospatial queries
- trips: (rider_id, requested_at) - for user history
- payments: (trip_id, payment_status) - for transaction tracking

### Partitioning Strategy
- PostgreSQL: Shard by city_id for trips table
- Cassandra: Partition by driver_id for location data
- Redis: Cluster by geographic regions

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| POST | /trips/request | Request a ride with pickup/destination |
| GET | /trips/{trip_id} | Get real-time trip status and driver location |
| POST | /trips/{trip_id}/cancel | Cancel trip with reason |
| POST | /drivers/location | Update driver location (real-time) |
| POST | /drivers/status | Update driver availability status |
| POST | /trips/{trip_id}/accept | Driver accepts ride request |

## 🚀 Critical Talking Points

### Driver Matching Algorithm
- **What:** Geospatial matching using Redis GeoHash with scoring algorithm
- **Why:** Sub-5 second matching requirement needs fast spatial queries
- **Detail:** Query 2-mile radius, score by distance + rating + acceptance rate, notify top 3 drivers
- **Alternative:** PostgreSQL PostGIS (more complex queries, higher latency)

### Real-time Location Tracking
- **What:** WebSocket connections with 1-second location updates stored in Redis + Cassandra
- **Why:** Riders need real-time driver location, historical data for analytics
- **Detail:** Redis for current location (hot data), Cassandra for time-series history
- **Alternative:** Server-Sent Events (simpler but less efficient for bidirectional)

### Dynamic Pricing Engine
- **What:** Surge pricing algorithm based on demand/supply ratio with ML predictions
- **Why:** Balance supply and demand, incentivize drivers during peak times
- **Detail:** Monitor requests/minute by area, calculate 1.2x-3.0x multiplier
- **Alternative:** Fixed pricing (simpler but less efficient market balancing)

### Payment Processing
- **What:** Stripe integration with idempotent payment processing and commission calculation
- **Why:** Reliable payments with platform fee (15%) and driver earnings (85%)
- **Detail:** Two-phase commit for trip completion and payment processing
- **Alternative:** PayPal (different fee structure, longer processing time)

### Trip State Management
- **What:** State machine managing trip lifecycle with event-driven architecture
- **Why:** Complex business logic with multiple stakeholders and failure scenarios
- **Detail:** States: requested → matched → in-progress → completed/cancelled
- **Alternative:** Simple status field (less robust error handling)

### Geospatial Indexing
- **What:** Redis GeoHash for efficient radius queries on driver locations
- **Why:** Need to find nearby drivers quickly for matching algorithm
- **Detail:** Partition drivers by geographic regions, maintain availability sets
- **Alternative:** Elasticsearch geo queries (more features, higher latency)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Database | PostgreSQL + Cassandra + Redis | Single database | Each optimized for specific use case |
| Matching | Redis GeoHash | PostgreSQL PostGIS | Faster spatial queries, lower latency |
| Location Updates | WebSocket | Polling | Real-time updates, better UX |
| Pricing | Dynamic surge | Fixed pricing | Better market efficiency |
| Payments | Synchronous | Asynchronous | Immediate feedback required |
| Caching | Multi-tier | Single cache | Different TTLs for different data types |

## 🔥 Bottlenecks & Solutions

### Database Write Contention
- **Problem:** High volume location updates causing locks
- **Solution:** Write-behind caching, connection pooling, geographic partitioning

### Driver Matching Latency
- **Problem:** Algorithm too slow during peak hours
- **Solution:** Pre-compute availability zones, Redis GeoHash, parallel notifications

### WebSocket Connection Overload
- **Problem:** Too many connections overwhelming servers
- **Solution:** Connection pooling, Redis Pub/Sub, client-side throttling

### Payment Processing Failures
- **Problem:** Network issues causing payment failures
- **Solution:** Idempotent operations, retry logic, circuit breaker pattern

## 💡 Interview Tips

- **Start Here:** "Let me clarify the requirements - we need to support 10M daily rides with <5 second matching"
- **Emphasize:** Geospatial algorithms, real-time systems, event-driven architecture
- **Be Ready For:** "How do you handle driver cancellations?", "What about surge pricing fairness?"
- **Don't Forget:** Security (PCI compliance), monitoring (real-time metrics), scalability (geographic distribution)

---

**Scale:** 10M daily rides, 500K drivers, 100+ cities
**Key Tech:** Redis GeoHash, WebSocket, PostgreSQL, Cassandra, Kafka
**Complexity:** ⭐⭐⭐⭐ Very Hard
**Interview Time:** 45-60 minutes
