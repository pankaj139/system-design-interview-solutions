# Real-time Analytics Dashboard System Design

**Difficulty Level:** ⭐⭐⭐⭐ Very Hard  
**Tags:** `Real-time Analytics`, `Stream Processing`, `Time-Series Data`, `OLAP`, `Dashboard`, `Event Processing`, `Distributed Systems`, `Data Aggregation`, `Apache Kafka`, `ClickHouse`

**File Purpose:** Interactive, multi-level learning resource for designing a real-time analytics dashboard system that ingests 5-10M events per day and powers near real-time KPI dashboards with drill-down capabilities. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that handles massive event streams with ≤1-2 minutes lag, supports complex aggregations (counts, percentiles, funnels), and scales to 10x traffic while maintaining sub-second query performance.

**Author:** System Design Documentation  
**Created:** January 22, 2026  
**Last Updated:** January 22, 2026  
**Recent Updates:** Initial creation with difficulty level and relevant tags for better categorization

**Learning Time Estimates:**
- 🟢 **Beginner Level:** 5-7 hours (analytics fundamentals, event ingestion, basic aggregations)
- 🟡 **Intermediate Level:** 7-10 hours (stream processing, time-series databases, query optimization)
- 🔴 **Advanced Level:** 10-15 hours (production optimization, handling late events, scaling strategies)

---

## 🎓 Welcome to Real-time Analytics Dashboard System Design!

### What You're Going to Build

Imagine creating a system like Google Analytics, Mixpanel, or Amplitude - one that tracks millions of user actions every day and shows live dashboards to product managers making critical business decisions. When a user clicks a button on an e-commerce site, that event travels through your pipeline, gets processed and aggregated, and appears on a dashboard within 1-2 minutes. Business analysts can drill down by country, product category, or user segment to understand customer behavior in near real-time.

By the end of this learning journey, you'll understand how to design a production-grade real-time analytics dashboard that:
- **Ingests 5-10M events per day** from multiple services (page views, clicks, transactions) with reliable delivery
- **Processes events in near real-time** with ≤1-2 minutes lag using stream processing pipelines
- **Handles late and out-of-order events** gracefully with watermarking and time-window strategies
- **Supports complex aggregations** including counts, sums, averages, percentiles, and funnel analysis
- **Enables powerful drill-downs** by product, geography, time range, and user segments with sub-second query latency
- **Scales to 10x traffic** (50-100M events/day) through horizontal scaling and data partitioning
- **Achieves 99.9% availability** (43 minutes downtime per month) with fault-tolerant architecture

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (5-7 hours)
├─ Learn analytics fundamentals and event tracking
├─ Understand WHY real-time matters for business decisions
├─ Build intuition with restaurant analytics analogy
├─ Master event ingestion basics with message queues
└─ Perfect for: New to analytics systems or data engineering

🟡 INTERMEDIATE LEVEL (7-10 hours)  
├─ Master interview techniques for analytics questions
├─ Design stream processing pipelines
├─ Learn time-series database selection criteria
├─ Implement aggregation strategies
├─ Practice capacity estimation
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-15 hours)
├─ Handle production challenges at scale
├─ Optimize for cost and performance
├─ Design for 10x growth
├─ Implement advanced features (anomaly detection, ML predictions)
├─ Navigate complex trade-offs
└─ Perfect for: Senior engineers, architects, tech leads
```

**What Makes This Different:**

This isn't just another analytics system doc. You'll learn:
- ✅ **Real company examples**: How Google Analytics, Mixpanel, and Amplitude solve these problems
- ✅ **Interview-ready frameworks**: Structured approaches to tackle analytics questions
- ✅ **Production war stories**: Common pitfalls and how to avoid them
- ✅ **Cost optimization**: Techniques used by companies saving millions on data processing
- ✅ **Hands-on examples**: JSON event schemas, API designs, SQL queries

---

## TABLE OF CONTENTS

- [1. REQUIREMENTS & CLARIFICATION](#1-requirements--clarification)
  - [User Stories](#user-stories)
  - [Functional Requirements](#functional-requirements)
  - [Non-Functional Requirements](#non-functional-requirements)
  - [Clarifying Questions & Assumptions](#clarifying-questions--assumptions)
- [2. BACK-OF-THE-ENVELOPE CALCULATIONS](#2-back-of-the-envelope-calculations)
  - [Traffic Estimates](#traffic-estimates)
  - [Storage Estimates](#storage-estimates)
  - [Resource Estimates](#resource-estimates)
  - [Bandwidth Estimates](#bandwidth-estimates)
- [3. HIGH-LEVEL DESIGN](#3-high-level-design)
  - [Core Components](#core-components)
  - [Architecture Diagram](#architecture-diagram)
  - [Data Flow](#data-flow)
- [4. DEEP DIVE: EVENT INGESTION PIPELINE](#4-deep-dive-event-ingestion-pipeline)
  - [Event Collection](#event-collection)
  - [Message Queue Architecture](#message-queue-architecture)
  - [Reliability Guarantees](#reliability-guarantees)
- [5. DEEP DIVE: STREAM PROCESSING](#5-deep-dive-stream-processing)
  - [Stream Processing Framework](#stream-processing-framework)
  - [Handling Late Events](#handling-late-events)
  - [Watermarking Strategy](#watermarking-strategy)
  - [Windowing Techniques](#windowing-techniques)
- [6. DEEP DIVE: DATA STORAGE](#6-deep-dive-data-storage)
  - [Time-Series Database](#time-series-database)
  - [OLAP Database](#olap-database)
  - [Hot vs Cold Storage](#hot-vs-cold-storage)
- [7. DEEP DIVE: AGGREGATIONS & METRICS](#7-deep-dive-aggregations--metrics)
  - [Real-time Aggregations](#real-time-aggregations)
  - [Pre-aggregation Strategy](#pre-aggregation-strategy)
  - [Percentile Calculations](#percentile-calculations)
  - [Funnel Analysis](#funnel-analysis)
- [8. API DESIGN](#8-api-design)
  - [Event Ingestion API](#event-ingestion-api)
  - [Dashboard Query API](#dashboard-query-api)
  - [Drill-down API](#drill-down-api)
- [9. DATABASE DESIGN](#9-database-design)
  - [Raw Events Schema](#raw-events-schema)
  - [Aggregated Metrics Schema](#aggregated-metrics-schema)
  - [Metadata Schema](#metadata-schema)
- [10. DEEP DIVE: DRILL-DOWN & FILTERING](#10-deep-dive-drill-down--filtering)
  - [Dimensional Modeling](#dimensional-modeling)
  - [Query Optimization](#query-optimization)
  - [Caching Strategy](#caching-strategy)
- [11. SCALABILITY](#11-scalability)
  - [Handling 10x Traffic](#handling-10x-traffic)
  - [Data Partitioning](#data-partitioning)
  - [Horizontal Scaling](#horizontal-scaling)
- [12. PERFORMANCE OPTIMIZATION](#12-performance-optimization)
  - [Query Performance](#query-performance)
  - [Materialized Views](#materialized-views)
  - [Data Compression](#data-compression)
- [13. SECURITY](#13-security)
  - [Data Privacy](#data-privacy)
  - [Access Control](#access-control)
  - [Compliance](#compliance)
- [14. MONITORING & OBSERVABILITY](#14-monitoring--observability)
  - [Pipeline Health](#pipeline-health)
  - [Data Quality](#data-quality)
  - [SLA Monitoring](#sla-monitoring)
- [15. TRADE-OFFS & ALTERNATIVES](#15-trade-offs--alternatives)
  - [Lambda vs Kappa Architecture](#lambda-vs-kappa-architecture)
  - [Batch vs Stream Processing](#batch-vs-stream-processing)
  - [Technology Choices](#technology-choices)
- [16. INTERVIEW PREPARATION](#16-interview-preparation)
  - [Common Questions](#common-questions)
  - [Interview Framework](#interview-framework)
  - [Troubleshooting Scenarios](#troubleshooting-scenarios)
- [PUTTING IT ALL TOGETHER](#putting-it-all-together)
- [RESOURCES & FURTHER LEARNING](#resources--further-learning)

---

## 1. REQUIREMENTS & CLARIFICATION

### User Stories

🟢 **BEGINNER: Understanding the Basics**

Think of a real-time analytics dashboard like the scoreboard at a restaurant chain. Every time a customer orders food, that "event" gets recorded. The restaurant manager wants to see:
- How many orders are happening right now?
- Which locations are busiest?
- What menu items are most popular today?

The key word is "right now" - not yesterday's report, but live data to make immediate decisions like "Should we send more staff to the downtown location?"

**As a product manager**, I want to see live metrics on my dashboard so that I can make data-driven decisions about product features without waiting for daily reports.

**As a business analyst**, I want to drill down into metrics by different dimensions (country, product, user segment) so that I can understand which customer segments are most engaged.

**As a data engineer**, I want to ingest millions of events reliably so that no user actions are lost even during traffic spikes.

**As a marketing manager**, I want to track conversion funnels in real-time so that I can optimize campaigns while they're still running.

---

🟡 **INTERMEDIATE: Interview Perspective**

In interviews, you need to extract these user stories to understand the scope:

**Key Questions to Ask:**
1. "Who are the primary users of this dashboard?" (Product managers? Executives? Engineers?)
2. "What decisions will they make with this data?" (Product launches? Marketing spend? System scaling?)
3. "How real-time is 'real-time'?" (Seconds? Minutes? Hours?)
4. "What types of events are we tracking?" (Page views? Purchases? API calls?)

**Example Interview Exchange:**
```text
You: "Can you describe a typical workflow for a dashboard user?"
Interviewer: "A product manager launches a new feature and wants to monitor adoption in real-time."
You: "Great! So we need event tracking, aggregation by feature, and dashboard updates within minutes?"
Interviewer: "Exactly. Let's say within 1-2 minutes."
```

---

🔴 **ADVANCED: Production Considerations**

Real user stories from companies like Mixpanel and Amplitude:

**User Story 1: Growth Team at E-commerce Company**
- Launches A/B test at 9 AM affecting 10% of traffic
- Monitors conversion rates every 5 minutes on dashboard
- Needs statistical significance calculations in real-time
- Must handle 50K events/minute during peak hours
- **Business Impact**: $10M annual revenue decisions based on A/B tests

**User Story 2: Security Team at FinTech**
- Tracks transaction events to detect fraud
- Requires <30 second detection time for suspicious patterns
- Needs to correlate events across multiple services
- Must maintain 99.99% reliability (financial regulations)
- **Business Impact**: Prevents $50M in fraud annually

**User Story 3: Engineering Team at SaaS Platform**
- Monitors API usage per customer for billing
- Needs exact counts (no data loss) for revenue calculation
- Tracks rate limits and quota usage in real-time
- Supports 1000+ enterprise customers with custom dashboards
- **Business Impact**: $100M ARR depends on accurate metering

---

### Functional Requirements

🟢 **BEGINNER: Core Features**

Let's understand what our system must do:

**1. Event Collection (The Input)**
```text
Analogy: Like a restaurant taking orders
├─ Customer places order → Event is recorded
├─ Order details captured → Event properties stored
└─ Time recorded → Timestamp added
```

Our system must:
- Accept events from multiple sources (web apps, mobile apps, servers)
- Handle different event types (page views, clicks, purchases)
- Capture event properties (user ID, timestamp, location, product details)

**Example Event:**
```json
{
  "event_type": "product_viewed",
  "user_id": "user_12345",
  "timestamp": "2026-01-22T10:30:00Z",
  "properties": {
    "product_id": "prod_789",
    "category": "electronics",
    "price": 299.99,
    "country": "US"
  }
}
```

**2. Real-time Processing**
- Process events within 1-2 minutes of occurrence
- Calculate basic metrics (counts, sums, averages)
- Update dashboards automatically

**3. Dashboard Visualization**
- Show KPIs (Key Performance Indicators)
- Display charts and graphs
- Refresh automatically with new data

**4. Basic Filtering**
- Filter by time range (last hour, today, this week)
- Filter by basic properties (country, product category)

---

🟡 **INTERMEDIATE: Complete Feature Set**

**Core Features (MVP):**

1. **Event Ingestion API**
   - REST endpoint for event submission
   - Batch API for multiple events
   - SDK support for common languages (JavaScript, Python, Java)
   - Input validation and error handling
   - Rate limiting per API key

2. **Real-time Aggregation**
   - Count: Number of events (e.g., page views)
   - Sum: Total values (e.g., revenue)
   - Average: Mean values (e.g., session duration)
   - Percentiles: P50, P90, P99 (e.g., page load time)
   - Unique counts: Distinct users, sessions

3. **Funnel Analysis**
   - Multi-step conversion tracking
   - Drop-off analysis between steps
   - Time-to-convert metrics
   - Cohort-based funnels

4. **Drill-down Capabilities**
   - Filter by dimensions:
     * Time (hour, day, week, month)
     * Geography (country, region, city)
     * Product attributes (category, price range, brand)
     * User segments (new vs returning, subscription tier)
   - Support for multiple filters simultaneously
   - Comparison views (this week vs last week)

5. **Query API**
   - Flexible query language for custom metrics
   - Support for grouping and aggregation
   - Time-series data retrieval
   - Response time: <1 second for most queries

6. **Historical Data Access**
   - Query past data (last 90 days hot storage)
   - Longer-term trends (12+ months in cold storage)
   - Data export capabilities

**Out of Scope for MVP:**

- Complex machine learning predictions
- Real-time anomaly detection
- Custom JavaScript for data transformation
- Advanced statistical analysis (regression, clustering)
- Real-time alerting system
- Multi-tenant isolation (assuming single organization initially)

---

🔴 **ADVANCED: Enterprise Features**

**Production-Grade Requirements:**

1. **Data Quality & Validation**
   - Schema validation for events
   - Duplicate detection and deduplication
   - Data integrity checks
   - Automatic data type inference
   - Custom validation rules per event type

2. **Advanced Analytics**
   - Retention cohort analysis
   - User journey mapping
   - Attribution modeling (first-touch, last-touch, multi-touch)
   - Predictive analytics (churn prediction, LTV forecasting)

3. **Governance & Compliance**
   - PII (Personally Identifiable Information) handling
   - GDPR-compliant data retention
   - User data deletion (right to be forgotten)
   - Audit logging for all queries
   - Role-based access control (RBAC)

4. **Multi-tenancy**
   - Customer data isolation
   - Per-tenant rate limits
   - Custom retention policies per customer
   - White-labeling support

5. **Integration Ecosystem**
   - Webhooks for event export
   - Data warehouse integration (Snowflake, BigQuery)
   - BI tool connectors (Tableau, Looker)
   - Reverse ETL for activation

---

### Non-Functional Requirements

🟢 **BEGINNER: What Makes the System Good**

Non-functional requirements are about **how well** the system works, not **what** it does.

**Restaurant Analogy:**
```text
Functional: Restaurant serves food (what)
Non-functional: Restaurant serves food within 15 minutes (how well)
```

**Our System's "How Well" Requirements:**

1. **Speed (Latency)**
   - Events appear on dashboard within 1-2 minutes
   - Queries return results in under 1 second
   - Why? Users won't wait longer than 2-3 seconds for dashboards to load

2. **Reliability (Availability)**
   - System works 99.9% of the time
   - Calculation: 99.9% = 43 minutes downtime per month
   - Why? Business decisions can't wait; downtime = lost insights

3. **Scalability**
   - Handles 5-10M events per day (current)
   - Can grow to 50-100M events per day (10x growth)
   - Why? Startups grow fast; system must grow with them

4. **Accuracy**
   - No events lost (100% delivery)
   - Counts are exact, not approximations
   - Why? Business metrics must be trustworthy

---

🟡 **INTERMEDIATE: Quantified SLAs**

**Performance Requirements:**

1. **Latency (End-to-End)**:
   - Event ingestion to storage: <5 seconds (p99)
   - Stream processing lag: <1 minute (p99)
   - Dashboard queries: <1 second (p95), <3 seconds (p99)
   - Aggregation refresh: Every 1-2 minutes
   
   **Why these numbers?**
   - Google Analytics shows ~1-2 minute lag (industry standard)
   - Users abandon dashboards after 3+ seconds load time
   - Real-time decisions require <5 minute total latency

2. **Availability**:
   - Overall system: 99.9% uptime (43 minutes/month downtime)
   - Event ingestion: 99.95% (21 minutes/month) - Most critical
   - Query service: 99.9% (acceptable brief outages)
   
   **Calculation:**
   ```text
   99.9% availability = 0.1% downtime
   30 days × 24 hours × 60 minutes = 43,200 minutes/month
   43,200 × 0.001 = 43.2 minutes downtime allowed
   ```

3. **Scalability**:
   - Current: 5-10M events/day = 115 events/second average, 350 events/sec peak (3x)
   - Target: 50-100M events/day = 1,150 events/second average, 3,500 events/sec peak
   - Query throughput: 100 queries/second (dashboard users)
   - Concurrent dashboard users: 1,000+

4. **Consistency**:
   - Strong consistency for writes (events not lost)
   - Eventual consistency acceptable for reads (1-2 minute lag)
   - Exactly-once processing semantics for billing/revenue metrics
   - At-least-once acceptable for engagement metrics

5. **Durability**:
   - Zero data loss for ingested events
   - Replication factor: 3 (across availability zones)
   - Backup retention: 90 days hot, 12 months cold storage
   - Point-in-time recovery: Within last 7 days

6. **Security**:
   - TLS 1.3 for data in transit
   - Encryption at rest (AES-256)
   - API authentication via API keys
   - Query authorization (users see only their data)

---

🔴 **ADVANCED: Production SLAs & Trade-offs**

**Service Level Objectives (SLOs):**

```text
SLI (Service Level Indicator)     | SLO Target  | Error Budget
================================== | =========== | ============
Event ingestion success rate       | 99.95%      | 0.05% (260K events/month @ 10M/day)
End-to-end latency (ingestion)     | <2 min p99  | 1% of events > 2 min
Query latency                      | <1s p95     | 5% of queries > 1s
Query success rate                 | 99.9%       | 0.1% failed queries
Data freshness                     | <2 min lag  | Measured per pipeline
Accuracy (compared to batch)       | 99.99%      | <0.01% discrepancy
```

**Trade-off Decisions:**

1. **Latency vs Cost**
   - Real-time (<1 sec): Requires expensive in-memory processing
   - Near real-time (1-2 min): 10x cheaper with batch micro-batches
   - **Decision**: 1-2 minute lag acceptable for most analytics use cases

2. **Accuracy vs Performance**
   - Exact counts: Require counting every event (expensive for unique users)
   - Approximate counts: Use HyperLogLog (0.01% error, 100x faster)
   - **Decision**: Exact for revenue, approximate for engagement metrics

3. **Consistency vs Availability (CAP Theorem)**
   - Strong consistency: Slower writes, risk of unavailability
   - Eventual consistency: Fast writes, temporary inconsistent reads
   - **Decision**: Eventually consistent (1-2 min lag) for better availability

4. **Storage vs Query Speed**
   - Raw events: Flexible queries, expensive storage
   - Pre-aggregated: Fast queries, limited flexibility
   - **Decision**: Hybrid approach (raw + pre-aggregated)

**Compliance Requirements:**

- **GDPR** (European users):
  - User consent tracking
  - Data portability (export user data)
  - Right to deletion (within 30 days)
  
- **CCPA** (California users):
  - Opt-out mechanism
  - Data disclosure requirements

- **SOC 2 Type II**:
  - Audit logging
  - Access controls
  - Encryption standards

---

### Clarifying Questions & Assumptions

🟢 **BEGINNER: What to Ask**

When designing a system, you need to understand the requirements clearly. Here are essential questions:

**About the Data:**
- Q: What types of events will we track?
  - A: User events (page views, clicks, purchases), system events (API calls, errors)
  
- Q: How big is each event?
  - A: Average 1KB (including all properties and metadata)
  
- Q: Do we need to store raw events forever?
  - A: Hot storage for 90 days, cold storage for 12 months, then archive/delete

**About the Users:**
- Q: Who will use these dashboards?
  - A: Product managers, analysts, executives, customer success teams
  
- Q: How many people will query simultaneously?
  - A: 100-1000 concurrent users during business hours

**About Scale:**
- Q: How many events per day right now?
  - A: 5-10 million events/day currently
  
- Q: Expected growth?
  - A: Planning for 10x growth (50-100M events/day) within 2 years

---

🟡 **INTERMEDIATE: Interview Framework**

**The RADIO Framework** for clarifying requirements:

**R - Requirements (What must it do?)**
```text
Q: "What are the must-have features for MVP?"
Q: "What can we defer to v2?"
Q: "Are there any compliance requirements?"
```

**A - Assumptions (What can we assume?)**
```text
Q: "Can we assume 80-20 read-write ratio?"
Q: "Can we assume events arrive in order?"
Q: "Can we assume US-only deployment initially?"
```

**D - Data (What data flows through?)**
```text
Q: "What's the schema of an event?"
Q: "Do we need to support schema evolution?"
Q: "What's the size distribution of events?"
```

**I - Infrastructure (What do we have?)**
```text
Q: "Are we on AWS/GCP/Azure?"
Q: "Do we have existing Kafka clusters?"
Q: "What's our operational expertise?"
```

**O - Operations (How is it used?)**
```text
Q: "What's the query pattern?"
Q: "How often do dashboards refresh?"
Q: "What's the typical drill-down depth?"
```

**Example Interview Dialog:**

```text
Interviewer: "Design a real-time analytics dashboard."

You: "Great! Let me clarify a few things. First, when you say 'real-time,' 
     what latency is acceptable? Are we talking seconds, minutes, or hours?"

Interviewer: "Let's say 1-2 minutes is acceptable."

You: "Perfect. And what scale are we targeting? How many events per day?"

Interviewer: "Start with 5-10 million per day, but design for 10x growth."

You: "Got it. For the dashboard queries, what's more important - 
     query flexibility or query speed?"

Interviewer: "Users need common queries to be very fast, under 1 second."

You: "That suggests we should pre-aggregate common metrics. One more thing -
     do we need exactly-once semantics, or is at-least-once acceptable?"

Interviewer: "For revenue metrics, exactly-once. For engagement, at-least-once is fine."
```

---

🔴 **ADVANCED: Hidden Requirements**

Experienced engineers probe for requirements the interviewer might not mention:

**Operational Requirements:**
- Q: "What's our on-call budget?" → Affects complexity of system
- Q: "What's the team size?" → Affects technology choices (no exotic tech for small teams)
- Q: "What's our operational expertise?" → Build vs buy decisions

**Cost Constraints:**
- Q: "What's the monthly infrastructure budget?"
  - Affects: Storage strategy (cheap vs fast), retention policies, batch vs streaming
  
- Q: "What's the cost per event we can afford?"
  - Example: $10,000/month budget ÷ 300M events/month = $0.000033 per event
  - This drives architectural decisions (use cheap batch processing, not expensive real-time)

**Data Quality:**
- Q: "How do we handle duplicate events?" → Idempotency design
- Q: "What if events arrive out of order?" → Watermarking strategy  
- Q: "What if event schema changes?" → Versioning strategy
- Q: "How do we validate event data?" → Validation rules

**Failure Scenarios:**
- Q: "What happens if the database is down?" → Fallback to cached data?
- Q: "What if stream processing lags by 10 minutes?" → Show staleness indicator?
- Q: "What if we receive 10x traffic unexpectedly?" → Auto-scaling or backpressure?

**Business Context:**
- Q: "Are dashboards used for executive reporting?" → Higher accuracy requirements
- Q: "Do dashboards drive automated decisions?" → Higher availability requirements
- Q: "Is this customer-facing or internal?" → Affects SLA requirements

**Assumptions We'll Make:**

1. **Geographic Distribution:**
   - Assumption: Initially US-only deployment
   - Can expand to multi-region later

2. **Event Sources:**
   - Assumption: Events from web apps (JavaScript SDK), mobile apps, backend servers
   - Protocol: HTTPS JSON over REST, Batch API available

3. **Event Structure:**
   - Assumption: Semi-structured JSON (flexible schema)
   - Common fields: event_type, user_id, timestamp, properties {}

4. **Query Patterns:**
   - Assumption: 80% of queries are for last 24 hours
   - 15% for last 7 days, 5% for longer periods
   - Most common: daily active users, conversion rates, revenue

5. **Data Retention:**
   - Assumption: 90 days in hot storage (fast queries)
   - 12 months in cold storage (slower, cheaper)
   - After 12 months: Aggregate to daily/weekly summaries

6. **User Behavior:**
   - Assumption: 100-1000 concurrent dashboard users
   - Peak hours: 9 AM - 5 PM business days
   - Average user runs 20 queries per session

7. **Growth Pattern:**
   - Assumption: 2x growth year-over-year
   - Traffic spikes during product launches (3-5x normal)
   - Black Friday / holiday spikes (10x for e-commerce clients)

---

## 2. BACK-OF-THE-ENVELOPE CALCULATIONS

### Traffic Estimates

🟢 **BEGINNER: Basic Math**

Let's figure out how many events per second we need to handle:

**Restaurant Analogy:**
```text
If a restaurant serves 300 customers per day:
├─ Open 10 hours = 600 minutes
├─ 300 customers ÷ 600 minutes = 0.5 customers/minute
└─ During lunch rush (2x normal) = 1 customer/minute
```

**Our System:**

**Current Scale: 5-10M events/day**

```text
Average events per second:
├─ 10M events/day
├─ 24 hours/day × 60 min × 60 sec = 86,400 seconds/day
├─ 10,000,000 ÷ 86,400 = 115 events/second (average)
└─ Peak (3x average) = 345 events/second
```

**Target Scale (10x growth): 50-100M events/day**

```text
10x growth calculation:
├─ 100M events/day
├─ 100,000,000 ÷ 86,400 = 1,157 events/second (average)
└─ Peak (3x average) = 3,471 events/second
```

**Query Load:**

```text
Dashboard users: 1,000 concurrent users
├─ Each user runs 1 query every 30 seconds
├─ 1,000 users ÷ 30 seconds = 33 queries/second (average)
└─ Peak hours (3x) = 100 queries/second
```

---

🟡 **INTERMEDIATE: Detailed Breakdown**

**Event Ingestion Traffic:**

| Scale Level | Events/Day | Avg Events/Sec | Peak Events/Sec | Monthly Events |
|-------------|------------|----------------|-----------------|----------------|
| **Current** | 10M | 115 | 345 | 300M |
| **1 Year** | 20M | 231 | 693 | 600M |
| **2 Years (Target)** | 50M | 578 | 1,734 | 1.5B |
| **Peak Growth** | 100M | 1,157 | 3,471 | 3B |

**Peak Calculation Breakdown:**

```text
Average events/sec × Peak multiplier = Peak events/sec

Peak multipliers by time:
├─ Daily peak (business hours): 2x average
├─ Campaign launch: 3x average
├─ Black Friday/Flash sale: 5-10x average
└─ System must handle: 3x sustained, 10x burst (with backpressure)
```

**Query Traffic Patterns:**

```text
Dashboard Queries:
├─ Active users during business hours: 500-1000
├─ Avg query frequency: 1 query/30 seconds
├─ Base query rate: 20-33 QPS
├─ Peak query rate: 60-100 QPS
└─ Burst capacity needed: 200 QPS (2x peak)

Query Types Distribution:
├─ Simple aggregations (COUNT, SUM): 60% - <100ms latency
├─ Percentile calculations (P50, P90, P99): 25% - <500ms latency
├─ Funnel queries (multi-step): 10% - <1s latency
└─ Complex drill-downs: 5% - <3s latency
```

**Data Flow Summary:**

```text
Ingestion Path (Write-Heavy):
└─ 1,157 events/sec avg → 3,471 events/sec peak

Processing Path (Compute):
└─ 1,157 events/sec need aggregation in real-time

Query Path (Read-Heavy):
└─ 33 queries/sec avg → 100 queries/sec peak

Read:Write Ratio = 33:1157 ≈ 1:35 (Write-heavy system!)
```

---

🔴 **ADVANCED: Capacity Planning**

**Detailed Traffic Analysis:**

**1. Event Ingestion Capacity:**

```text
Target: 100M events/day peak capacity

Sustained throughput:
├─ Average: 1,157 events/sec
├─ Peak (3x): 3,471 events/sec
├─ Burst (10x): 11,570 events/sec (must handle for 5-10 minutes)
└─ Design capacity: 15,000 events/sec (30% headroom)

Message queue sizing:
├─ Kafka partition throughput: ~5,000 msgs/sec/partition
├─ Required partitions: 15,000 ÷ 5,000 = 3 partitions (minimum)
├─ Recommended: 12 partitions (4x over-provisioned for:
│   ├─ Key-based partitioning for ordering
│   ├─ Future growth
│   └─ Rebalancing overhead)
└─ Total Kafka throughput: 12 × 5,000 = 60,000 events/sec capacity
```

**2. Stream Processing Capacity:**

```text
Processing requirements per event:
├─ Parsing & validation: 1ms
├─ Enrichment (geo-lookup, user data): 5ms
├─ Aggregation updates: 2ms
└─ Total: ~8ms per event

Single processor capacity:
├─ 1,000ms ÷ 8ms = 125 events/sec per core
├─ 8-core machine: 1,000 events/sec
└─ For 3,471 events/sec peak: Need 4 machines (with 2x redundancy = 8 machines)

Flink/Spark Streaming cluster:
├─ Worker nodes: 8 machines (c5.2xlarge - 8 vCPUs, 16 GB RAM)
├─ Total capacity: 8,000 events/sec
├─ Utilization at peak: 43% (healthy headroom)
└─ Cost: ~$2,000/month (8 × $250 = $2,000)
```

**3. Query Capacity:**

```text
Query service capacity:
├─ Target: 100 QPS sustained, 200 QPS burst
├─ Average query latency: 500ms
├─ Queries in-flight: 100 QPS × 0.5s = 50 concurrent queries
├─ CPU per query: 100ms (assume efficient OLAP DB)
└─ Per-node capacity: 10 cores × 1000ms ÷ 100ms = 100 queries/sec

Node sizing:
├─ Minimum nodes: 200 QPS ÷ 100 QPS = 2 nodes
├─ Recommended: 4 nodes (N+1 redundancy + headroom)
└─ Instance type: c5.4xlarge (16 vCPUs, 32 GB RAM)
```

**Regional Distribution (Future):**

```text
If expanding to multi-region:

North America (60% traffic):
├─ Events: 60K/day (avg) → 180K/day (peak)
└─ Queries: 60 QPS

Europe (25% traffic):
├─ Events: 25K/day (avg) → 75K/day (peak)
└─ Queries: 25 QPS

Asia Pacific (15% traffic):
├─ Events: 15K/day (avg) → 45K/day (peak)
└─ Queries: 15 QPS
```

---

### Storage Estimates

🟢 **BEGINNER: How Much Disk Space?**

**Simple Calculation:**

```text
One event = 1 KB (1,000 bytes)
10 million events/day = 10 million KB = 10 GB/day

For 90 days (3 months):
├─ 10 GB/day × 90 days = 900 GB
└─ Almost 1 TB (Terabyte) of storage
```

**Why 1 KB per event?**

```json
{
  "event_id": "evt_1234567890",           // 30 bytes
  "event_type": "product_viewed",         // 20 bytes
  "user_id": "user_12345",                // 15 bytes
  "session_id": "sess_abcd",              // 15 bytes
  "timestamp": "2026-01-22T10:30:00Z",    // 25 bytes
  "properties": {                         // ~800 bytes
    "product_id": "prod_789",
    "product_name": "Wireless Headphones",
    "category": "electronics",
    "price": 299.99,
    "country": "US",
    "city": "San Francisco",
    "device": "mobile",
    "browser": "Chrome",
    // ... more properties
  }
}
// Total: ~1,000 bytes (1 KB)
```

---

🟡 **INTERMEDIATE: Complete Storage Breakdown**

**Raw Events Storage:**

| Time Period | Events | Size per Event | Total Size | Compression | Final Size |
|-------------|--------|----------------|------------|-------------|------------|
| **1 Day** | 10M | 1 KB | 10 GB | 5:1 | 2 GB |
| **1 Week** | 70M | 1 KB | 70 GB | 5:1 | 14 GB |
| **1 Month** | 300M | 1 KB | 300 GB | 5:1 | 60 GB |
| **90 Days (Hot)** | 900M | 1 KB | 900 GB | 5:1 | 180 GB |
| **1 Year (Cold)** | 3.6B | 1 KB | 3.6 TB | 10:1 | 360 GB |

**Compression Rationale:**
- JSON text compresses well (5:1 with gzip/snappy)
- Time-series data has repetitive fields (event_type, user patterns)
- Columnar formats (Parquet) achieve 10:1 for cold storage

**Aggregated Metrics Storage:**

```text
Pre-computed aggregations (for fast queries):

Granularity levels:
├─ Per-minute aggregates: 60×24 = 1,440 rows/day/metric
├─ Per-hour aggregates: 24 rows/day/metric
└─ Per-day aggregates: 1 row/day/metric

Dimensions tracked:
├─ Time (minute/hour/day)
├─ Event type (50 types)
├─ Country (200 countries)
├─ Product category (100 categories)
└─ Combinations: 50 × 200 × 100 = 1M dimension combinations

Per-minute aggregates:
├─ 1,440 minutes/day × 1M combinations = 1.44B rows/day
├─ Each row: ~100 bytes (timestamp, dimensions, metrics)
├─ Daily storage: 1.44B × 100 bytes = 144 GB/day
└─ With compression (3:1): 48 GB/day

90-day aggregates:
└─ 48 GB/day × 90 days = 4.3 TB (compressed)
```

**Total Storage Requirements:**

```text
For 90 days hot storage:
├─ Raw events: 180 GB
├─ Pre-aggregated metrics: 4.3 TB
├─ Indexes & metadata: 500 GB
└─ Total: ~5 TB

For 1 year total:
├─ Hot storage (90 days): 5 TB
├─ Cold storage (275 days): 1 TB (heavily aggregated)
└─ Total: 6 TB
```

---

🔴 **ADVANCED: Storage Optimization**

**Storage Architecture:**

```text
3-Tier Storage Strategy:

Tier 1: Hot Storage (Last 7 days)
├─ Technology: ClickHouse on NVMe SSDs
├─ Raw events: 2 GB/day × 7 = 14 GB
├─ Minute-level aggregates: 48 GB/day × 7 = 336 GB
├─ Total: 350 GB
├─ Replication: 3x = 1.05 TB
├─ Cost: $0.10/GB/month × 1,050 GB = $105/month
└─ Query latency: <100ms

Tier 2: Warm Storage (8-90 days)
├─ Technology: ClickHouse on GP3 SSDs
├─ Raw events: 2 GB/day × 83 = 166 GB
├─ Hour-level aggregates: 10 GB/day × 83 = 830 GB
├─ Total: 996 GB
├─ Replication: 3x = 3 TB
├─ Cost: $0.05/GB/month × 3,000 GB = $150/month
└─ Query latency: <500ms

Tier 3: Cold Storage (91 days - 1 year)
├─ Technology: S3 + Athena (query on demand)
├─ Daily aggregates only: 1 GB/day × 275 = 275 GB
├─ Compression: 10:1 (Parquet columnar format)
├─ Total: 27.5 GB
├─ Cost: $0.023/GB/month × 27.5 = $0.63/month
└─ Query latency: 5-30 seconds (acceptable for historical analysis)
```

**Storage Cost Optimization:**

```text
Total monthly storage cost:
├─ Tier 1 (Hot): $105
├─ Tier 2 (Warm): $150
├─ Tier 3 (Cold): $0.63
└─ Total: ~$256/month for 10M events/day

At 100M events/day (10x scale):
├─ Tier 1: $1,050 (10x data)
├─ Tier 2: $1,500 (10x data)
├─ Tier 3: $6.30 (10x data)
└─ Total: ~$2,556/month

Cost per event:
└─ $2,556 ÷ 3B events/month = $0.000852 per event
```

**Data Retention Policy:**

```text
Retention strategy by event type:

Critical events (revenue, conversions):
├─ Raw events: 365 days
├─ Aggregates: Forever (minimal storage)
└─ Why: Compliance, financial reporting, historical analysis

Engagement events (page views, clicks):
├─ Raw events: 90 days
├─ Aggregates: 365 days
├─ Summary (daily): Forever
└─ Why: Balance between insight value and cost

System events (errors, logs):
├─ Raw events: 30 days
├─ Aggregates: 90 days
└─ Why: Debugging recent issues, minimal historical value
```

**Database Sizing (ClickHouse):**

```text
ClickHouse cluster configuration:

Hot tier (7 days):
├─ Nodes: 3 replicas × 2 shards = 6 nodes
├─ Instance: i3.2xlarge (8 vCPUs, 61 GB RAM, 1.9 TB NVMe)
├─ Storage per node: 350 GB (raw) + overhead = 500 GB
├─ CPU utilization: 30% average, 70% peak
├─ Cost: 6 nodes × $0.624/hour × 730 hours = $2,733/month
└─ Total capacity: Handles 10x current load

Warm tier (83 days):
├─ Nodes: 3 replicas × 2 shards = 6 nodes
├─ Instance: r5.2xlarge (8 vCPUs, 64 GB RAM, 1 TB EBS)
├─ Storage per node: 1 TB
├─ Cost: 6 nodes × $0.504/hour × 730 hours = $2,207/month
└─ Total capacity: Adequate for current load
```

---

### Resource Estimates

🟢 **BEGINNER: Server Resources**

**What resources does our system need?**

```text
Think of a computer's resources like a restaurant:
├─ CPU (Processing power) = Chefs cooking
├─ Memory (RAM) = Prep tables for active orders
├─ Storage (Disk) = Pantry storing ingredients
└─ Network (Bandwidth) = Delivery trucks
```

**Our System Needs:**

**1. Message Queue Servers (Kafka)**
- 3 servers (for reliability)
- Each: 8 CPUs, 32 GB memory, 1 TB disk
- Why? Temporarily hold events before processing

**2. Stream Processing Servers**
- 8 servers
- Each: 8 CPUs, 16 GB memory
- Why? Process events in real-time

**3. Database Servers (ClickHouse)**
- 12 servers (6 hot + 6 warm)
- Each: 8 CPUs, 64 GB memory, 1 TB disk
- Why? Store and query event data

**4. API Servers**
- 4 servers
- Each: 4 CPUs, 8 GB memory
- Why? Handle incoming events and dashboard queries

**Total:**
- 27 servers
- 172 CPUs
- 1.4 TB memory
- ~$8,000/month cloud cost

---

🟡 **INTERMEDIATE: Detailed Resource Breakdown**

**Component-wise Resource Allocation:**

**1. Ingestion Layer:**

```text
API Gateway (Nginx):
├─ Instances: 3 (c5.large: 2 vCPUs, 4 GB RAM)
├─ Throughput: 5,000 req/sec per instance = 15,000 total
├─ Cost: 3 × $0.085/hour × 730 = $186/month
└─ Purpose: Load balancing, TLS termination, rate limiting

Event API Servers (Go/Node.js):
├─ Instances: 6 (c5.xlarge: 4 vCPUs, 8 GB RAM)
├─ Capacity: 500 events/sec per instance = 3,000 total
├─ Cost: 6 × $0.17/hour × 730 = $745/month
└─ Purpose: Validation, enrichment, Kafka publishing
```

**2. Message Queue (Kafka):**

```text
Kafka Cluster:
├─ Brokers: 3 (r5.xlarge: 4 vCPUs, 32 GB RAM, 1 TB gp3)
├─ Partitions: 12 (per topic)
├─ Replication factor: 3
├─ Throughput: 60,000 msgs/sec (20,000 per broker)
├─ Retention: 7 days (for replay capability)
├─ Storage: 3 TB total (1 TB per broker)
└─ Cost: 3 × $0.252/hour × 730 = $552/month

Zookeeper (for Kafka coordination):
├─ Instances: 3 (t3.small: 2 vCPUs, 2 GB RAM)
└─ Cost: 3 × $0.021/hour × 730 = $46/month
```

**3. Stream Processing (Apache Flink):**

```text
Flink Cluster:
├─ Task Managers: 8 (c5.2xlarge: 8 vCPUs, 16 GB RAM)
├─ Job Managers: 2 (c5.xlarge: 4 vCPUs, 8 GB RAM)
├─ Processing capacity: 8,000 events/sec
├─ State backend: S3 (for checkpointing)
├─ Checkpointing: Every 5 minutes
└─ Cost:
    ├─ Task Managers: 8 × $0.34/hour × 730 = $1,987/month
    └─ Job Managers: 2 × $0.17/hour × 730 = $248/month
    Total: $2,235/month
```

**4. Data Storage (ClickHouse):**

```text
(Already detailed in Storage Estimates section)
├─ Hot tier: 6 nodes = $2,733/month
├─ Warm tier: 6 nodes = $2,207/month
└─ Cold tier (S3): ~$1/month
Total: $4,941/month
```

**5. Query Service:**

```text
Query API Servers:
├─ Instances: 4 (c5.2xlarge: 8 vCPUs, 16 GB RAM)
├─ Capacity: 100 QPS per instance = 400 total
├─ Cache: Redis (r5.large: 2 vCPUs, 13 GB RAM)
└─ Cost:
    ├─ API servers: 4 × $0.34/hour × 730 = $993/month
    └─ Redis: 1 × $0.126/hour × 730 = $92/month
    Total: $1,085/month
```

**6. Monitoring & Operations:**

```text
Prometheus + Grafana:
├─ Instances: 2 (t3.large: 2 vCPUs, 8 GB RAM)
└─ Cost: 2 × $0.0832/hour × 730 = $122/month

ELK Stack (Logs):
├─ Instances: 3 (r5.large: 2 vCPUs, 16 GB RAM)
└─ Cost: 3 × $0.126/hour × 730 = $276/month

Total monitoring: $398/month
```

**Grand Total Monthly Cost:**

```text
Component                 | Monthly Cost
========================= | ============
Ingestion (API + Nginx)  | $931
Message Queue (Kafka)     | $598
Stream Processing (Flink) | $2,235
Storage (ClickHouse + S3) | $4,941
Query Service            | $1,085
Monitoring               | $398
========================= | ============
Total                    | $10,188/month

Cost per event: $10,188 ÷ 300M events = $0.000034 per event
```

---

🔴 **ADVANCED: Cost Optimization Strategies**

**Reserved Instance Savings:**

```text
1-year Reserved Instances (40% discount):
├─ Current on-demand: $10,188/month
├─ With Reserved: $6,113/month
└─ Annual savings: $48,900

3-year Reserved Instances (60% discount):
├─ With Reserved: $4,075/month
└─ Annual savings: $73,356
```

**Spot Instance Usage:**

```text
For stateless components (stream processing):
├─ Flink Task Managers on Spot: 70% discount
├─ Current: $1,987/month
├─ With Spot: $596/month
├─ Savings: $1,391/month ($16,692/year)
└─ Risk: Acceptable (Flink checkpoints enable fast recovery)
```

**Auto-scaling Strategy:**

```text
Time-based scaling:

Peak hours (9 AM - 5 PM weekdays):
├─ Full capacity: 27 servers
└─ Cost: $10,188/month (during peak)

Off-peak hours (nights, weekends):
├─ Scale down to 60% capacity
├─ Reduced cost: $6,113/month
└─ Average effective cost: $7,800/month

Potential savings: $2,388/month ($28,656/year)
```

**Multi-tenancy Cost Allocation:**

```text
If serving multiple customers:

Small customers (1M events/day):
├─ Share infrastructure
├─ Cost allocation: $1,000/month/customer
└─ Margin: 70% (infrastructure cost: $300)

Medium customers (10M events/day):
├─ Dedicated query nodes
├─ Cost allocation: $8,000/month/customer
└─ Margin: 20% (infrastructure cost: $6,400)

Large customers (100M events/day):
├─ Fully dedicated infrastructure
├─ Cost allocation: $40,000/month/customer
└─ Margin: 60% (infrastructure cost: $25,000)
```

---

### Bandwidth Estimates

🟢 **BEGINNER: Network Traffic**

**How much data travels over the network?**

```text
Inbound (Events coming in):
├─ 10M events/day
├─ 1 KB per event
├─ 10M × 1 KB = 10 GB/day
└─ 10 GB ÷ 24 hours ÷ 3600 seconds = 120 KB/second

Outbound (Dashboard queries):
├─ 33 queries/second
├─ Average response: 10 KB (time-series data for charts)
├─ 33 × 10 KB = 330 KB/second
└─ Much less than inbound!
```

**Monthly Bandwidth:**

```text
Inbound: 10 GB/day × 30 days = 300 GB/month
Outbound: 330 KB/sec × 86,400 sec × 30 days = 856 GB/month
Total: ~1.2 TB/month
```

---

🟡 **INTERMEDIATE: Detailed Bandwidth Analysis**

**Ingestion Bandwidth:**

```text
Current (10M events/day):
├─ Average: 115 events/sec × 1 KB = 115 KB/sec = 0.92 Mbps
├─ Peak (3x): 345 events/sec × 1 KB = 345 KB/sec = 2.76 Mbps
└─ Daily: 10 GB

Target (100M events/day):
├─ Average: 1,157 events/sec × 1 KB = 1.13 MB/sec = 9.05 Mbps
├─ Peak (3x): 3,471 events/sec × 1 KB = 3.39 MB/sec = 27.1 Mbps
└─ Daily: 100 GB

Burst (10x):
└─ 11,570 events/sec × 1 KB = 11.3 MB/sec = 90.5 Mbps
```

**Query Bandwidth:**

```text
Query response sizes:
├─ Simple aggregates (counts): 1 KB
├─ Time-series (7 days): 10 KB
├─ Drill-down (detailed): 50 KB
└─ Export (full data): 1 MB

Average query: 10 KB

Query traffic:
├─ Average: 33 QPS × 10 KB = 330 KB/sec = 2.64 Mbps
├─ Peak: 100 QPS × 10 KB = 1 MB/sec = 8 Mbps
└─ Daily: 33 QPS × 86,400 sec × 10 KB = 28.5 GB
```

**Internal Bandwidth (Between Components):**

```text
Kafka → Flink → ClickHouse:
├─ Events flow: 115 events/sec × 1 KB = 115 KB/sec
├─ Aggregates: 115 KB/sec ÷ 10 (compression) = 11.5 KB/sec
└─ Total internal: 126.5 KB/sec

ClickHouse replication:
├─ Replication factor: 3
├─ Data written: 2 GB/day (compressed)
├─ Replicated: 2 GB × 2 (source → 2 replicas) = 4 GB/day
└─ Average: 46 KB/sec
```

**Monthly Bandwidth Summary:**

| Direction | Daily | Monthly | Cost (@$0.09/GB) |
|-----------|-------|---------|------------------|
| **Ingress (Events)** | 10 GB | 300 GB | $0 (free on AWS) |
| **Egress (Queries)** | 28.5 GB | 855 GB | $76.95 |
| **Internal** | 6 GB | 180 GB | $0 (same region) |
| **Total** | 44.5 GB | 1,335 GB | $76.95 |

---

🔴 **ADVANCED: Network Optimization**

**CDN for Dashboard Assets:**

```text
Dashboard frontend (static assets):
├─ Size: 2 MB (HTML, JS, CSS, images)
├─ Users: 1,000 daily
├─ Page views: 10,000/day
├─ Without CDN: 10,000 × 2 MB = 20 GB/day egress
├─ With CDN (95% cache hit): 20 GB × 0.05 = 1 GB/day egress
└─ Savings: 19 GB/day × 30 = 570 GB/month = $51/month
```

**API Response Compression:**

```text
Without compression:
├─ Query response: 10 KB (JSON)
└─ 100 QPS × 10 KB = 1 MB/sec

With gzip compression:
├─ Compressed response: 2 KB (5:1 compression)
├─ 100 QPS × 2 KB = 200 KB/sec
└─ Savings: 80% bandwidth = $61/month (855 GB → 171 GB)
```

**Regional Data Centers:**

```text
Multi-region deployment:

US-based customers (60%):
├─ US East data center
└─ Cross-region bandwidth: $0

EU-based customers (25%):
├─ EU data center
├─ Cross-region sync: 2.5 GB/day × $0.02/GB = $0.05/day
└─ Monthly: $1.50

APAC customers (15%):
├─ Singapore data center
├─ Cross-region sync: 1.5 GB/day × $0.02/GB = $0.03/day
└─ Monthly: $0.90

Total multi-region bandwidth cost: $2.40/month (minimal!)
```

**Bandwidth Budget at Scale:**

```text
At 100M events/day (10x growth):

Ingress: 100 GB/day = 3 TB/month (free)
Egress (queries): 280 GB/day = 8.4 TB/month
├─ Without optimization: 8.4 TB × $0.09 = $756/month
├─ With CDN + compression: 1.7 TB × $0.09 = $153/month
└─ Savings: $603/month ($7,236/year)
```

---

## 3. HIGH-LEVEL DESIGN

### Core Components

🟢 **BEGINNER: The Big Picture with a Restaurant Analogy**

Imagine a restaurant chain with a central monitoring system. Here's how our analytics dashboard works:

**The Restaurant Analogy:**
```text
Restaurant Chain Analytics System
├─ Cash Registers (Event Sources)
│  └─ Every transaction, order, customer visit is recorded
├─ Collection Trucks (API Gateway)  
│  └─ Pick up data from all locations regularly
├─ Sorting Center (Message Queue - Kafka)
│  └─ Organize all receipts by type and time
├─ Analysis Team (Stream Processor - Flink)
│  └─ Calculate totals, averages, trends in real-time
├─ Filing System (Database - ClickHouse)
│  └─ Store organized summaries for quick lookup
├─ Report Generator (Query API)
│  └─ Answer manager's questions like "How many pizzas sold today?"
└─ Manager's Dashboard (Web UI)
   └─ Visual charts showing sales, trends, comparisons
```

**Why This Architecture?**

Each component has a specific job, just like restaurant staff:
- **Event Sources**: Generate data (like cashiers recording sales)
- **API Gateway**: Accept and validate incoming events (like a receptionist checking orders)
- **Message Queue**: Buffer events during busy times (like a ticket queue at a busy restaurant)
- **Stream Processor**: Calculate metrics in real-time (like a manager counting money while the restaurant is still open)
- **Database**: Store results for quick access (like a filing cabinet with organized reports)
- **Query API**: Retrieve specific information (like asking "What were Monday's sales?")
- **Dashboard**: Display information visually (like a chart on the manager's wall)

**Key Benefit**: Each component can be scaled independently. If you have more events, add more message queue capacity. If you have slower queries, add more database nodes.

---

🟡 **INTERMEDIATE: Component Breakdown**

Let's examine each component's role in the system:

**1. Event Sources**
- **What**: Web apps, mobile apps, backend services generating events
- **Examples**: 
  - JavaScript SDK tracking button clicks
  - Mobile SDK tracking screen views  
  - Server-side API calls tracking transactions
- **Key Metric**: 5-10M events/day = ~115 events/second average, ~350 events/second peak

**2. API Gateway (Load Balancer)**
- **What**: Entry point for all event ingestion requests
- **Responsibilities**:
  - SSL termination and security
  - API key validation
  - Rate limiting per customer
  - Request routing to ingestion service
- **Technology**: Nginx, AWS ALB, Kong
- **Capacity**: Must handle 3x average load = 1,000 requests/second

**3. Ingestion Service**
- **What**: Stateless HTTP servers accepting events
- **Responsibilities**:
  - Event validation (schema, required fields)
  - Event enrichment (add server timestamp, IP geolocation)
  - Write to message queue
  - Return acknowledgment to client
- **Technology**: Node.js, Go, Python (FastAPI)
- **Scaling**: Horizontal - add more servers during traffic spikes

**4. Message Queue (Kafka)**
- **What**: Distributed log storing events temporarily
- **Why Needed**: Decouples ingestion from processing
  - Ingestion can accept events even if processor is slow
  - Can replay events if processing fails
  - Natural place for data backup
- **Topics**: Organize by event type (page_views, clicks, purchases)
- **Retention**: 7 days (enough time to reprocess if needed)
- **Partitioning**: By user_id or session_id for ordered processing

**5. Stream Processing (Apache Flink)**
- **What**: Real-time computation engine
- **Responsibilities**:
  - Window events by time (1-minute windows)
  - Calculate aggregations (counts, sums, averages)
  - Handle late-arriving events
  - Write results to database
- **Why Flink**: 
  - True event-time processing (not just processing time)
  - Exactly-once state guarantees
  - Low latency (~100ms processing time)

**6. Time-Series Database (ClickHouse)**
- **What**: Column-oriented OLAP database optimized for analytics
- **Why ClickHouse**:
  - Excellent compression (10x better than row stores)
  - Fast aggregation queries (100M rows in <1 second)
  - Scales horizontally with sharding
- **Storage**:
  - Raw events: 90 days in hot storage
  - Pre-aggregated metrics: 2 years in hot storage
  - Historical data: Cold storage (S3)

**7. Query Service**
- **What**: API layer for dashboard queries
- **Responsibilities**:
  - Parse query parameters (time range, filters, groupings)
  - Generate optimized SQL
  - Cache frequent queries
  - Format results for UI
- **Caching**: Redis for popular queries (last 5 minutes of data)

**8. Dashboard UI**
- **What**: Web application displaying metrics
- **Features**:
  - Real-time chart updates (WebSocket for live data)
  - Interactive drill-downs
  - Custom date ranges
  - Export to CSV/PDF
- **Technology**: React, D3.js for visualizations

**Component Communication:**
```text
Sync Communication (REST/HTTP):
├─ Client SDK → API Gateway → Ingestion Service
├─ Dashboard → Query Service → Database
└─ Low latency requirements, request-response pattern

Async Communication (Message Queue):
├─ Ingestion Service → Kafka → Stream Processor
├─ Stream Processor → Database (batch writes)
└─ High throughput, eventual consistency acceptable
```

---

🔴 **ADVANCED: Production Architecture Considerations**

**Component Redundancy & Fault Tolerance:**

```text
Production Deployment (3 Availability Zones)
├─ API Gateway (3+ instances)
│  ├─ Health checks every 10 seconds
│  ├─ Auto-scaling based on request rate
│  └─ Failover: <5 seconds to remove unhealthy instance
├─ Ingestion Service (6+ instances)
│  ├─ Stateless for easy scaling
│  ├─ Graceful shutdown: drain connections before restart
│  └─ Circuit breaker for Kafka writes
├─ Kafka Cluster (5+ brokers)
│  ├─ Replication factor: 3
│  ├─ Min in-sync replicas: 2
│  ├─ Leader election: <30 seconds on failure
│  └─ Mirrors across availability zones
├─ Flink Cluster (8+ task managers)
│  ├─ Checkpointing every 60 seconds to S3
│  ├─ Savepoints for version upgrades
│  ├─ Restart strategy: fixed-delay with 3 attempts
│  └─ Task failure isolation
├─ ClickHouse Cluster (6+ nodes)
│  ├─ Sharded across 3 shards
│  ├─ Replicated: 2 replicas per shard
│  ├─ Distributed queries via proxy
│  └─ ZooKeeper for coordination
├─ Query Service (4+ instances)
│  ├─ Read replicas for ClickHouse
│  ├─ Redis cache cluster (3 nodes, sentinel)
│  └─ Connection pooling to database
└─ Monitoring Stack
   ├─ Prometheus for metrics
   ├─ Grafana for dashboards
   ├─ ELK for log aggregation
   └─ PagerDuty for alerts
```

**Real Company Examples:**

**Mixpanel's Architecture Evolution:**
1. **2014**: Monolithic Python app, PostgreSQL, manual sharding
   - Handled 1M events/day
   - Query latency: 5-10 seconds
2. **2017**: Migrated to Kafka + Spark + Cassandra
   - Scaled to 50B events/day
   - Reduced query latency to <1 second
   - Cost: $8M/year infrastructure
3. **2020**: Introduced ClickHouse, replaced Cassandra
   - Same 50B events/day
   - Query latency: <100ms for most queries
   - Cost reduced to $3M/year (62% savings!)

**Amplitude's "User-First" Architecture:**
- Events stored by user_id, not just timestamp
- Enables fast user journey queries
- Trade-off: More storage (3x) but 10x faster user-specific queries
- Business impact: Unique selling point for product analytics

**Google Analytics 4 (GA4) Architecture Shift:**
- **GA3 (Universal Analytics)**: Session-based, batch processing
- **GA4 (2020+)**: Event-based, real-time processing with BigQuery
- Key insight: Customers willing to pay 10x more for real-time insights
- Revenue impact: $2B+ annual revenue from Analytics 360

**Cost Optimization Strategies:**

```text
Monthly Cost Breakdown (10M events/day):

Without Optimization:
├─ Compute (Flink, services): $3,000
├─ Kafka: $1,500
├─ ClickHouse (hot storage): $4,000
├─ Cold storage (S3): $500
├─ Bandwidth: $800
└─ Total: $9,800/month

With Optimization:
├─ Compute (spot instances, auto-scaling): $1,800 (40% savings)
├─ Kafka (tiered storage): $900 (40% savings)
├─ ClickHouse (compression, partitioning): $2,400 (40% savings)
├─ Cold storage (S3 Glacier): $150 (70% savings)
├─ Bandwidth (CDN, compression): $200 (75% savings)
└─ Total: $5,450/month (44% overall savings = $52,200/year)

Key Optimizations:
1. Pre-aggregate data in Flink (reduces storage by 10x)
2. Use columnar compression in ClickHouse (6-8x compression ratio)
3. Archive raw events to S3 Glacier after 90 days
4. Cache dashboard queries in Redis (90% cache hit rate)
5. Use spot instances for Flink workers (60% cost savings)
```

---

### Architecture Diagram

🟢 **BEGINNER: Simple Flow**

```mermaid
graph LR
    A[User Actions<br/>Clicks, Views, Purchases] --> B[Event Collection<br/>JavaScript SDK]
    B --> C[API Gateway<br/>Receives Events]
    C --> D[Message Queue<br/>Kafka]
    D --> E[Stream Processor<br/>Flink]
    E --> F[Database<br/>ClickHouse]
    F --> G[Query API<br/>Get Metrics]
    G --> H[Dashboard<br/>Charts & Graphs]
    
    style A fill:#e1f5ff
    style D fill:#fff3cd
    style E fill:#d4edda
    style F fill:#f8d7da
    style H fill:#e7e7ff
```

**Flow Explanation:**
1. **User Actions** → Events are generated (clicks, page views)
2. **Event Collection** → SDK captures events and sends to API
3. **API Gateway** → Validates and accepts events
4. **Message Queue** → Buffers events (like a waiting line)
5. **Stream Processor** → Calculates metrics in real-time
6. **Database** → Stores aggregated results
7. **Query API** → Retrieves data when dashboard requests it
8. **Dashboard** → Shows visual charts to users

**Timing:** Event → Dashboard = 1-2 minutes end-to-end

---

🟡 **INTERMEDIATE: Detailed Component Architecture**

```mermaid
graph TB
    subgraph "Event Sources"
        A1[Web App<br/>JS SDK]
        A2[Mobile App<br/>iOS/Android SDK]
        A3[Backend Services<br/>Server-side SDK]
    end
    
    subgraph "Ingestion Layer"
        B1[API Gateway<br/>Load Balancer]
        B2[Ingestion Service<br/>Node.js/Go]
        B3[Schema Validator]
    end
    
    subgraph "Message Queue Layer"
        C1[Kafka Topic: page_views]
        C2[Kafka Topic: clicks]
        C3[Kafka Topic: purchases]
    end
    
    subgraph "Stream Processing Layer"
        D1[Flink: Windowing]
        D2[Flink: Aggregation]
        D3[Flink: Late Event Handling]
    end
    
    subgraph "Storage Layer"
        E1[ClickHouse: Raw Events<br/>90 days]
        E2[ClickHouse: Aggregated<br/>2 years]
        E3[S3: Cold Storage<br/>Historical]
    end
    
    subgraph "Query Layer"
        F1[Query Service]
        F2[Redis Cache]
        F3[Query Optimizer]
    end
    
    subgraph "Presentation Layer"
        G1[Dashboard UI]
        G2[API for Custom Apps]
        G3[Export Service]
    end
    
    A1 --> B1
    A2 --> B1
    A3 --> B1
    B1 --> B2
    B2 --> B3
    B3 --> C1
    B3 --> C2
    B3 --> C3
    C1 --> D1
    C2 --> D1
    C3 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> E1
    D3 --> E2
    E1 --> E3
    E2 --> F1
    F1 --> F2
    F1 --> F3
    F2 --> G1
    F3 --> G2
    F2 --> G3
    
    style B1 fill:#e1f5ff
    style C1 fill:#fff3cd
    style C2 fill:#fff3cd
    style C3 fill:#fff3cd
    style D2 fill:#d4edda
    style E2 fill:#f8d7da
    style F2 fill:#ffe6e6
    style G1 fill:#e7e7ff
```

**Key Data Flows:**

1. **Hot Path (Real-time):**
   ```text
   Event → Kafka → Flink → ClickHouse (Aggregated) → Redis Cache → Dashboard
   Latency: ~1-2 minutes
   ```

2. **Cold Path (Historical Queries):**
   ```text
   Event → Kafka → Flink → ClickHouse (Raw) → S3 → Query Service → Dashboard
   Latency: 5-10 seconds
   ```

3. **Cache Hit Path:**
   ```text
   Dashboard → Query Service → Redis Cache → Dashboard
   Latency: <100ms
   ```

---

🔴 **ADVANCED: Multi-Region Production Architecture**

```mermaid
graph TB
    subgraph "Region: US-EAST"
        subgraph "Ingestion - US"
            US_API[API Gateway]
            US_ING[Ingestion Service<br/>6 instances]
        end
        subgraph "Processing - US"
            US_KAFKA[Kafka Cluster<br/>5 brokers]
            US_FLINK[Flink Cluster<br/>8 workers]
        end
        subgraph "Storage - US"
            US_CH[ClickHouse Cluster<br/>6 nodes, 3 shards]
            US_REDIS[Redis Cache<br/>3 nodes]
        end
    end
    
    subgraph "Region: EU-WEST"
        subgraph "Ingestion - EU"
            EU_API[API Gateway]
            EU_ING[Ingestion Service<br/>4 instances]
        end
        subgraph "Processing - EU"
            EU_KAFKA[Kafka Cluster<br/>3 brokers]
            EU_FLINK[Flink Cluster<br/>4 workers]
        end
        subgraph "Storage - EU"
            EU_CH[ClickHouse Cluster<br/>4 nodes, 2 shards]
            EU_REDIS[Redis Cache<br/>3 nodes]
        end
    end
    
    subgraph "Global Services"
        GLB[Global Load Balancer<br/>Route53/Cloudflare]
        S3[S3 Cross-Region<br/>Cold Storage]
        ZK[ZooKeeper<br/>Coordination]
    end
    
    GLB --> US_API
    GLB --> EU_API
    US_API --> US_ING
    US_ING --> US_KAFKA
    US_KAFKA --> US_FLINK
    US_FLINK --> US_CH
    US_CH --> US_REDIS
    
    EU_API --> EU_ING
    EU_ING --> EU_KAFKA
    EU_KAFKA --> EU_FLINK
    EU_FLINK --> EU_CH
    EU_CH --> EU_REDIS
    
    US_CH --> S3
    EU_CH --> S3
    US_CH -.Async Replication.-> EU_CH
    EU_CH -.Async Replication.-> US_CH
    ZK --> US_KAFKA
    ZK --> EU_KAFKA
    
    style GLB fill:#ff9999
    style S3 fill:#99ccff
    style US_CH fill:#99ff99
    style EU_CH fill:#99ff99
```

**Multi-Region Considerations:**

1. **Data Sovereignty (GDPR Compliance):**
   ```text
   EU Customer Events:
   ├─ Ingested in EU region only
   ├─ Processed in EU data centers
   ├─ Stored in EU ClickHouse cluster
   ├─ No replication to US (unless anonymized)
   └─ Compliance: GDPR Article 44 (data transfers)
   ```

2. **Cross-Region Aggregation:**
   ```text
   Global Dashboard Query (e.g., "Total events worldwide"):
   ├─ Query service detects multi-region request
   ├─ Parallel queries to US and EU clusters
   ├─ Merge results in query service
   ├─ Cache global result in Redis
   └─ Latency: ~200ms (vs 50ms single-region)
   ```

3. **Disaster Recovery:**
   ```text
   US Region Failure Scenario:
   ├─ Global LB detects unhealthy US endpoints
   ├─ Routes 100% traffic to EU region
   ├─ EU region scales up to handle 2x load
   ├─ S3 has full backup of US data
   ├─ RTO (Recovery Time): 5 minutes
   ├─ RPO (Data Loss): <1 minute (Kafka replication)
   └─ Cost: 2x infrastructure in warm standby mode
   ```

**Production Metrics:**

```text
System Health Indicators:

Ingestion Layer:
├─ Event acceptance rate: 99.99% (SLA)
├─ API latency P99: <100ms
├─ Rejected events: <0.01% (schema validation failures)
└─ Kafka write success: 99.999%

Processing Layer:
├─ End-to-end latency: P50=45s, P99=90s
├─ Late events: ~2% (handled via watermarking)
├─ Exactly-once guarantee: 99.99%
└─ Checkpoint success rate: 99.9%

Storage Layer:
├─ Write throughput: 50K events/sec per node
├─ Query latency: P50=50ms, P99=500ms
├─ Storage compression: 7.2x average
└─ Disk utilization: <70% (threshold for scaling)

Query Layer:
├─ Cache hit rate: 88% (target: 85%)
├─ Query success rate: 99.95%
├─ Concurrent queries: 500 peak
└─ Response time SLA: <1 second for 95% of queries
```

---

### Data Flow

🟢 **BEGINNER: How an Event Becomes a Dashboard Metric**

Let's follow a single event through the entire system:

**Example: User clicks "Buy Now" button on e-commerce site**

```text
Step-by-Step Journey of One Event:

Step 1: Event Generation (t=0 seconds)
├─ User clicks button
├─ JavaScript SDK captures:
│  ├─ Event type: "button_click"
│  ├─ Button: "buy_now"
│  ├─ Product ID: "prod_12345"
│  ├─ User ID: "user_789"
│  └─ Timestamp: "2026-01-22T14:30:00.123Z"
└─ SDK bundles into JSON and sends via HTTPS

Step 2: API Gateway (t=0.1 seconds)
├─ SSL termination
├─ API key validation: ✓ Valid
├─ Rate limit check: ✓ Within limits
└─ Route to ingestion service

Step 3: Ingestion Service (t=0.2 seconds)
├─ Validate event schema: ✓ All required fields present
├─ Enrich event:
│  ├─ Add server timestamp
│  ├─ Add IP geolocation: "US-CA-San Francisco"
│  └─ Add session ID from cookie
├─ Write to Kafka topic "button_clicks"
└─ Return success to client (202 Accepted)

Client side complete: 0.2 seconds

Step 4: Kafka Queue (t=0.2 - 5 seconds)
├─ Event sits in queue partition 3 (based on user_id hash)
├─ Replicated to 3 brokers for safety
└─ Waiting for Flink to consume

Step 5: Flink Stream Processor (t=5-45 seconds)
├─ Reads event from Kafka
├─ Groups into 1-minute time window
├─ Aggregates with other clicks in same window:
│  ├─ Total clicks this minute: 1,247
│  ├─ Unique users: 892
│  ├─ Clicks by product: prod_12345 = 23 clicks
│  └─ Clicks by country: US = 750, UK = 200, etc.
└─ Window closes at end of minute

Step 6: Write to ClickHouse (t=60-65 seconds)
├─ Flink batch-writes aggregated results
├─ Table: aggregated_clicks_1min
├─ Row inserted:
│  ├─ minute: "2026-01-22T14:30:00Z"
│  ├─ event_type: "button_click"
│  ├─ total_clicks: 1247
│  ├─ unique_users: 892
│  └─ dimension_details: {...}
└─ Also writes raw event to raw_events table

Step 7: Dashboard Query (t=65-90 seconds)
├─ User's dashboard set to auto-refresh every 30 seconds
├─ Dashboard queries: "Get clicks in last hour"
├─ Query service checks Redis cache: ❌ Not cached yet
├─ Query ClickHouse:
│  └─ SELECT sum(total_clicks) FROM aggregated_clicks_1min
│     WHERE minute >= now() - INTERVAL 1 HOUR
├─ Result: 75,342 clicks
├─ Cache result in Redis (TTL: 60 seconds)
└─ Return to dashboard

Step 8: Dashboard Display (t=90 seconds)
├─ Dashboard receives result
├─ Updates chart with new data point
└─ User sees: "75,342 clicks in last hour" 

Total Time: Event occurred → Visible on dashboard = 90 seconds
```

**Why Each Step Takes Time:**

- **Ingestion (0.2s)**: Network latency + processing
- **Kafka (5s)**: Waiting for batch of events to process together (more efficient)
- **Flink (40s)**: Waiting for 1-minute window to close
- **ClickHouse (5s)**: Batch write for efficiency
- **Dashboard (25s)**: Waiting for next auto-refresh cycle

**Trade-off:** We could make it faster (10s instead of 90s) but it would cost 5x more in compute resources. For most analytics use cases, 1-2 minutes is acceptable.

---

🟡 **INTERMEDIATE: Data Flow Patterns**

**Pattern 1: Write Path (Event Ingestion)**

```text
Event Ingestion Flow:

Client SDK
├─ Batching: Collect 10 events or 5 seconds (whichever first)
├─ Compression: gzip payload (60% size reduction)
├─ Retry: Exponential backoff (1s, 2s, 4s, 8s)
└─ POST /events/batch

API Gateway (Nginx)
├─ Rate limiting: 1000 req/sec per API key
├─ Request validation: Check Content-Type, payload size
├─ Health check: Return 503 if Kafka unhealthy
└─ Forward to Ingestion Service

Ingestion Service (Stateless)
├─ Parse JSON payload
├─ Schema validation:
│  ├─ Required fields: event_type, timestamp, user_id
│  ├─ Type checking: timestamp must be ISO8601
│  └─ Reject invalid events (return 400)
├─ Event enrichment:
│  ├─ Add server_timestamp
│  ├─ GeoIP lookup from IP address
│  ├─ User-Agent parsing (browser, device, OS)
│  └─ Session tracking (from cookie)
├─ Kafka write:
│  ├─ Topic selection: Based on event_type
│  ├─ Partition key: user_id (ensures user events ordered)
│  ├─ Acknowledgment: Wait for 2 replicas (acks=all)
│  └─ Timeout: 5 seconds
└─ Response to client:
   ├─ Success: 202 Accepted
   └─ Failure: 500 (retry), 400 (don't retry)

Kafka
├─ Partition assignment: Hash(user_id) % num_partitions
├─ Replication: 3 replicas, min in-sync=2
├─ Retention: 7 days (168 hours)
└─ Monitoring: Lag, throughput, disk usage
```

**Pattern 2: Read Path (Dashboard Queries)**

```text
Dashboard Query Flow:

Dashboard UI
├─ User selects: "Last 7 days", filter: "Country=US"
├─ Sends query: GET /api/metrics?
│  ├─ metric=page_views
│  ├─ start_time=2026-01-15T00:00:00Z
│  ├─ end_time=2026-01-22T00:00:00Z
│  ├─ filter=country:US
│  └─ group_by=day

Query Service
├─ Generate cache key: hash(metric+time_range+filters)
├─ Check Redis cache:
│  ├─ Cache hit? Return cached result (latency: 5ms)
│  └─ Cache miss? Proceed to database query
├─ Query optimization:
│  ├─ Determine if pre-aggregated table available
│  ├─ Choose table: aggregated_metrics_1hour (vs raw_events)
│  └─ Reason: 7-day query on hourly aggregates = 168 rows vs 50M raw events
├─ Generate SQL:
│  └─ SELECT date_trunc('day', hour) as day, sum(count) as page_views
│     FROM aggregated_metrics_1hour
│     WHERE hour >= '2026-01-15' AND hour < '2026-01-22'
│       AND country = 'US' AND metric_name = 'page_views'
│     GROUP BY day
│     ORDER BY day
├─ Execute on ClickHouse:
│  ├─ Query routing: Use replica for read
│  ├─ Execution time: 87ms (scan 168 rows)
│  └─ Result: 7 rows (one per day)
├─ Post-processing:
│  ├─ Format timestamps
│  ├─ Calculate growth rates (day-over-day)
│  └─ Add metadata (total_rows, query_time)
├─ Cache result:
│  ├─ Write to Redis
│  ├─ TTL: 5 minutes (recent data changes frequently)
│  └─ Key expires automatically
└─ Return JSON to dashboard

Dashboard UI
├─ Receives data
├─ Renders chart (D3.js)
├─ Shows: 45K, 48K, 52K, 49K, 51K, 53K, 55K views
└─ Adds interactivity (hover tooltips, drill-down buttons)
```

**Pattern 3: Aggregation Flow (Flink Processing)**

```text
Flink Stream Processing:

Kafka Consumer
├─ Subscribe to topics: page_views, clicks, purchases
├─ Consumer group: analytics-aggregator
├─ Parallelism: 8 (one per Kafka partition)
└─ Commit offset: Every 100 events or 5 seconds

Event Time Assignment
├─ Extract timestamp from event.timestamp field
├─ Watermark strategy: BoundedOutOfOrderness
│  ├─ Max out-of-order: 30 seconds
│  └─ Means: Wait up to 30s for late events
└─ Late events policy: Accept up to 2 minutes late, then drop

Windowing
├─ Window type: Tumbling (non-overlapping)
├─ Window size: 1 minute
├─ Example windows:
│  ├─ [14:00:00, 14:01:00)
│  ├─ [14:01:00, 14:02:00)
│  └─ [14:02:00, 14:03:00)
└─ Window trigger: When watermark passes end of window

Aggregation Functions
├─ COUNT(*): Number of events
├─ COUNT(DISTINCT user_id): Unique users
├─ SUM(revenue): Total revenue
├─ AVG(session_duration): Average duration
├─ PERCENTILE(page_load_time, 0.50, 0.90, 0.99)
└─ GROUP BY: country, product_category, device_type

State Management
├─ State backend: RocksDB (disk-based, handles large state)
├─ Checkpointing: Every 60 seconds
├─ Checkpoint storage: S3 (persistent)
├─ State TTL: 24 hours (old state cleaned up)
└─ State size: ~2GB per task manager

Output Sink
├─ Batch writes to ClickHouse (every 10 seconds or 1000 rows)
├─ Insert into: aggregated_metrics_1min
├─ Idempotency: Use event_time + window + dimensions as unique key
├─ Error handling: Retry 3 times, then write to dead-letter queue
└─ Monitoring: Track write latency, error rate

Exactly-Once Guarantee
├─ Kafka offsets stored in Flink checkpoint
├─ Two-phase commit: Checkpoint succeeds → Commit Kafka offsets
├─ Failure recovery: Restore from last checkpoint, replay events
└─ Result: Each event processed exactly once (no duplicates)
```

---

🔴 **ADVANCED: Complex Data Flow Scenarios**

**Scenario 1: Handling Out-of-Order Events**

Real-world problem: Mobile apps often send events in batches when network reconnects, causing events from hours ago to arrive now.

```text
Example Timeline:

10:00 AM: User performs action on mobile app (offline)
10:05 AM: User performs another action (still offline)
11:30 AM: Mobile app reconnects to network
11:30 AM: Both events sent with original timestamps

Challenge: 11:30 AM Flink window for "11:30-11:31" already processed and closed

Solution: Watermarking + Allowed Lateness

Watermark Configuration:
├─ Max out-of-order: 5 minutes
├─ Allowed lateness: 30 minutes
└─ Late event storage: 2 hours in side output

Processing Flow:
1. Event arrives at 11:30 AM with timestamp 10:00 AM
2. Flink checks: Is window [10:00-10:01] still open?
   ├─ Current watermark: 11:25 AM (current time - 5 min max out-of-order)
   ├─ Window closed at: 10:01 AM + 30 min allowed lateness = 10:31 AM
   ├─ 11:25 > 10:31: Window is closed, event is "too late"
   └─ Action: Send to side output (late events table)
3. Late events table:
   ├─ Stores events for manual reprocessing
   ├─ Dashboard shows: "2 late events (0.001%)"
   ├─ Options: Reprocess with batch job, or ignore
   └─ Typically: <0.1% of events are "too late"

Cost-Benefit Analysis:
├─ Allowing 30 min lateness costs: +15% memory (hold state longer)
├─ Benefit: Capture 99.9% of late events
└─ Alternative: No lateness allowed, lose 1-2% of mobile events
```

**Scenario 2: Flash Traffic Spike (10x Normal Load)**

Real scenario: Black Friday sale starts, traffic jumps from 100 events/sec to 1,000 events/sec

```text
System Response (Auto-Scaling):

t=0: Sale starts, events spike

t=30s: API Gateway
├─ Detects: Request rate exceeds 80% capacity
├─ Action: Trigger auto-scaling group
├─ Result: Add 4 new ingestion service instances
└─ Time to ready: 90 seconds (container startup)

t=1min: Kafka
├─ Detects: High producer load
├─ Action: Partitions already exist, increase replication throughput
├─ Result: Kafka handles spike (designed for 3x capacity)
└─ Lag: Increases from 1s to 8s (acceptable)

t=2min: Flink
├─ Detects: Checkpoint duration increasing (30s → 120s)
├─ Problem: More events per window, larger state
├─ Action: Task manager auto-scaling (8 → 16 instances)
├─ Result: Parallelism doubles, checkpoint time back to 40s
└─ Cost: +$50/hour during spike

t=5min: ClickHouse
├─ Detects: Write throughput at 90% capacity
├─ Action: Use write-ahead buffer (memory)
├─ Result: Batch larger writes (10K rows instead of 1K)
├─ Trade-off: Slightly higher end-to-end latency (90s → 120s)
└─ User impact: Minimal (still under 2-minute SLA)

t=10min: Steady State
├─ All systems scaled up
├─ Processing 1,000 events/sec smoothly
├─ End-to-end latency: 120 seconds (vs 90s normally)
├─ Cost: 2.5x normal ($120/hour vs $48/hour)
└─ Revenue impact: $50K/hour in sales, scaling cost negligible

t=2hours: Sale ends, traffic normalizes
├─ Auto-scaling: Gradually scales down over 30 minutes
├─ Result: Back to baseline capacity
└─ Total spike cost: $144 extra for 2-hour event
```

**Scenario 3: Database Node Failure**

```text
Failure Scenario: One ClickHouse node crashes

Detection (5 seconds):
├─ Health check fails: Node 3 not responding
├─ ZooKeeper: Detects node 3 offline
└─ Load balancer: Removes node 3 from pool

Immediate Impact:
├─ Node 3 was: 1 of 6 nodes (shard 2, replica 1)
├─ Data availability: ✓ Still available (replica 2 on node 4)
├─ Query impact: 16% slower (5 nodes instead of 6)
├─ Write impact: Shard 2 writes now go only to node 4
└─ Alert: PagerDuty notification sent

Automatic Recovery:
├─ ClickHouse: Promotes node 4 as primary for shard 2
├─ Queries: Automatically route around failed node
├─ Writes: Continue to available replica
└─ Data loss: 0 (replication saved us!)

Manual Recovery (30 minutes):
├─ Engineer investigates: Disk failure on node 3
├─ Action: Provision new node, restore from replica
├─ New node catches up: Sync 2 hours of data (5 minutes)
├─ Verification: Run test queries, check data consistency
└─ System: Back to full capacity

Lessons:
├─ Replication prevented data loss
├─ Query impact was minimal (16% slower briefly)
├─ Automated failover worked (no manual intervention needed immediately)
└─ Cost of redundancy: 2x storage, worth it for reliability
```

**Performance Optimization: Query Plan Analysis**

```text
Example: Complex Dashboard Query

User Request: "Show P99 page load time by country, last 30 days"

Naive Query (Slow):
SELECT country, 
       PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY page_load_time) as p99
FROM raw_events
WHERE event_type = 'page_load'
  AND timestamp >= now() - INTERVAL 30 DAY
GROUP BY country

Execution Plan:
├─ Scan: 1.5B raw events (30 days × 50M events/day)
├─ Filter: Keep 500M page_load events (33%)
├─ Sort: 500M events for percentile calculation
├─ Group: By ~200 countries
├─ Memory: 50GB (500M rows × 100 bytes avg)
├─ Time: 45 seconds
└─ Cost: $0.50 per query (compute time)

Optimized Query (Fast):
SELECT country, 
       quantile(0.99)(page_load_time) as p99
FROM aggregated_page_loads_1hour
WHERE hour >= now() - INTERVAL 30 DAY
GROUP BY country

Execution Plan:
├─ Scan: 14.4K pre-aggregated rows (30 days × 24 hours × 20 countries)
├─ ClickHouse quantile function: Pre-computed sketch
├─ Group: By ~200 countries
├─ Memory: 2MB (14K rows)
├─ Time: 120ms
├─ Cost: $0.001 per query
└─ Accuracy: 99.5% (sketch-based, acceptable trade-off)

Optimization Results:
├─ Speedup: 375x faster (45s → 120ms)
├─ Cost reduction: 500x cheaper
├─ Memory: 2,500x less
└─ Trade-off: 0.5% accuracy loss (acceptable for dashboards)

How Pre-Aggregation Works:
1. Flink maintains "t-digest" sketch per country per hour
2. Sketch stores approximate distribution (uses 1KB per country)
3. Percentiles calculated from sketch (P50, P90, P99 all from same sketch)
4. ClickHouse stores sketches, can merge them for longer time ranges
5. Result: Accurate enough for business decisions, massively faster
```

---

### Component Responsibilities

🟢 **BEGINNER: Who Does What?**

```text
Component Roles (Simple Analogy: Restaurant Kitchen)

API Gateway = Restaurant Host
├─ Greets customers (accepts requests)
├─ Checks reservations (API key validation)
├─ Manages waiting list (rate limiting)
└─ Assigns tables (routes to servers)

Ingestion Service = Order Taker
├─ Takes orders (receives events)
├─ Writes order clearly (validates data)
├─ Adds details (enriches with timestamp, location)
└─ Sends to kitchen (writes to Kafka)

Kafka = Order Queue
├─ Holds all orders in sequence
├─ Multiple cooks can grab orders (parallel processing)
├─ Orders don't get lost (persistent storage)
└─ Can replay if cook makes mistake

Flink = Chef
├─ Cooks food (processes events)
├─ Combines ingredients (aggregates data)
├─ Times dishes perfectly (windowing)
└─ Prepares plates (outputs results)

ClickHouse = Storage Pantry
├─ Stores prepared dishes (aggregated metrics)
├─ Organized by date and type (partitioned)
├─ Quick to find items (indexed)
└─ Preserves food (compressed storage)

Query Service = Waiter
├─ Takes requests from diners (dashboard queries)
├─ Fetches from pantry (queries database)
├─ Remembers common requests (caching)
└─ Serves quickly (optimized responses)

Dashboard = Customer's Table
├─ Where food is enjoyed (metrics displayed)
├─ Beautiful presentation (charts and graphs)
├─ Can request more (drill-downs)
└─ Satisfaction is the goal (actionable insights)
```

---

🟡 **INTERMEDIATE: Detailed Responsibilities Matrix**

| Component | Primary Responsibility | Secondary Responsibilities | Failure Mode | SLA |
|-----------|----------------------|---------------------------|--------------|-----|
| **API Gateway** | Accept/reject incoming requests | • SSL termination<br/>• Rate limiting<br/>• Request routing<br/>• DDoS protection | Return 503 if backend unhealthy | 99.99% uptime |
| **Ingestion Service** | Validate and enrich events | • Schema validation<br/>• GeoIP lookup<br/>• Session tracking<br/>• Kafka writes | Drop invalid events, return 400 | 99.9% uptime |
| **Kafka** | Buffer events reliably | • Partition management<br/>• Replication<br/>• Offset tracking<br/>• Log retention | Increase latency, never lose data | 99.99% durability |
| **Flink** | Real-time aggregation | • Windowing<br/>• State management<br/>• Exactly-once processing<br/>• Late event handling | Checkpoint failure → Restart from last checkpoint | 99.9% processing |
| **ClickHouse** | Store and query data | • Compression<br/>• Partitioning<br/>• Replication<br/>• Query optimization | Serve from replica if node fails | 99.95% availability |
| **Query Service** | Serve dashboard queries | • Query generation<br/>• Caching<br/>• Result formatting<br/>• Access control | Return cached data if DB slow | <1s response |
| **Redis** | Cache frequent queries | • TTL management<br/>• Eviction policy<br/>• Replication | Serve directly from DB if cache miss | 99.9% availability |
| **Dashboard** | Display visualizations | • Real-time updates<br/>• User interactions<br/>• Export functions | Show stale data if API fails | 99% uptime |

---

🔴 **ADVANCED: Responsibility Boundaries & Trade-offs**

**Critical Decision: Where Does Aggregation Happen?**

```text
Option 1: Aggregate in Database (ClickHouse)
Pros:
├─ Simpler architecture (no Flink needed)
├─ Flexible queries (ad-hoc aggregations)
└─ Lower operational complexity

Cons:
├─ Higher query latency (aggregate on read)
├─ Limited throughput (100K events/sec max)
├─ Expensive compute (query every aggregated metric)
└─ No stream processing benefits (late events, etc.)

Cost: $3K/month database, no stream processing
Use case: <10M events/day, flexible queries more important than speed

Option 2: Aggregate in Stream Processor (Flink)
Pros:
├─ Pre-computed aggregations (fast queries)
├─ High throughput (millions of events/sec)
├─ Event-time processing (handles late events)
└─ Advanced features (sessionization, complex windows)

Cons:
├─ More complex architecture
├─ Less flexible (must pre-define metrics)
├─ Operational overhead (Flink cluster management)
└─ Higher initial cost

Cost: $5K/month (Flink + database)
Use case: >10M events/day, real-time aggregations critical

Option 3: Hybrid (Lambda Architecture)
Pros:
├─ Real-time layer: Flink for recent data (last 24 hours)
├─ Batch layer: Spark for historical reprocessing
└─ Best of both: Speed + flexibility

Cons:
├─ Most complex architecture
├─ Data reconciliation challenges
├─ Highest operational cost
└─ Two code paths to maintain

Cost: $8K/month (both systems)
Use case: >100M events/day, need both real-time and complex historical queries

Decision Matrix:
├─ <5M events/day: Database-only (Option 1)
├─ 5-50M events/day: Stream processor (Option 2) ← Our choice
├─ >50M events/day: Hybrid (Option 3)
└─ Rationale: Sweet spot between cost and capability
```

**Responsibility: Data Quality & Validation**

```text
Who Validates Events?

Layer 1: Client SDK (Best Effort)
├─ Responsibility: Basic type checking before sending
├─ Validates: Required fields present
├─ Why here: Save bandwidth, reduce server load
├─ Limitation: Client-side code can be buggy/malicious
└─ Success rate: 95% (5% still reach server invalid)

Layer 2: API Gateway (Security Boundary)
├─ Responsibility: Authentication, rate limiting
├─ Validates: API key valid, not rate-limited
├─ Why here: Prevent abuse, protect backend
├─ Limitation: No deep validation (performance concern)
└─ Rejection rate: 3% (invalid API keys, over limit)

Layer 3: Ingestion Service (Detailed Validation)
├─ Responsibility: Schema validation, business rules
├─ Validates: 
│  ├─ Required fields: event_type, timestamp, user_id
│  ├─ Type checking: timestamp is ISO8601, user_id is alphanumeric
│  ├─ Range checking: timestamp within last 24 hours
│  ├─ Business rules: product_id exists in product catalog
│  └─ Size limits: properties object < 10KB
├─ Why here: After Kafka would be too late (already persisted bad data)
├─ Action on failure: Return 400, log error, track in metrics
└─ Rejection rate: 2% (genuinely invalid events)

Layer 4: Stream Processor (Data Quality Checks)
├─ Responsibility: Detect anomalies, late events
├─ Validates:
│  ├─ Late events: Timestamp too far in past
│  ├─ Duplicate detection: Same event_id seen twice
│  ├─ Anomalies: Event rate 10x normal (possible bot attack)
│  └─ Data consistency: Cross-check user_id exists
├─ Why here: Access to historical patterns, can compare
├─ Action on failure: Side output to quarantine table, alert
└─ Quarantine rate: 0.1% (late events, duplicates)

Layer 5: Database (Integrity Constraints)
├─ Responsibility: Final check before persisting
├─ Validates:
│  ├─ Unique constraints: (timestamp, user_id, event_type)
│  ├─ Foreign keys: user_id in users table (if strict mode)
│  └─ Data types: Enforced by schema
├─ Why here: Last defense, ensures data integrity
├─ Action on failure: Reject write, log error
└─ Rejection rate: 0.01% (should be very rare)

Metrics & Monitoring:
├─ Track rejection rate at each layer
├─ Alert if Layer 3 rejections > 5% (data quality issue)
├─ Alert if Layer 4 quarantine > 1% (system issue)
└─ Dashboard: Show data quality score (100% - rejection rate)
```

**Monitoring & Observability: Who Monitors What?**

```text
Distributed Tracing Example:

Single Event's Trace (Request ID: req_abc123):

Span 1: API Gateway (10ms)
├─ Operation: http.server.request
├─ Tags: api_key=key_xyz, ip=203.0.113.45
├─ Metrics: request_size=2.3KB
└─ Next: ingestion_service

Span 2: Ingestion Service (45ms)
├─ Operation: validate_and_enrich
├─ Tags: event_type=page_view, user_id=user_789
├─ Metrics: validation_time=12ms, geoip_lookup=8ms
├─ Child Span 2.1: Kafka Write (25ms)
│  ├─ Operation: kafka.produce
│  ├─ Tags: topic=page_views, partition=3
│  └─ Metrics: kafka_ack_time=22ms
└─ Next: kafka

Span 3: Flink Processing (42s)
├─ Operation: stream.window.aggregate
├─ Tags: window_start=14:30:00, window_end=14:31:00
├─ Metrics: events_in_window=1247, window_wait_time=40s
├─ State: watermark=14:30:45
└─ Next: clickhouse_sink

Span 4: ClickHouse Write (3s)
├─ Operation: db.insert
├─ Tags: table=aggregated_metrics_1min, shard=2
├─ Metrics: batch_size=1KB, write_time=120ms
└─ Persisted: Success

Span 5: Query Service (150ms)
├─ Operation: dashboard.query
├─ Tags: dashboard_id=sales_overview, user=pm_smith
├─ Metrics: query_time=87ms, cache=miss
└─ Next: clickhouse_read

Span 6: ClickHouse Query (87ms)
├─ Operation: db.select
├─ Tags: table=aggregated_metrics_1min, rows_scanned=168
├─ Metrics: query_plan=using_index, data_scanned=2MB
└─ Result: Success

Total Journey:
├─ Ingestion: 55ms (Spans 1-2)
├─ Processing: 42s (Span 3)
├─ Storage: 3s (Span 4)
├─ Query: 237ms (Spans 5-6)
└─ End-to-End: 45.3 seconds (ingestion to queryable)

Insights from Trace:
├─ Bottleneck: Window waiting time (40s) is expected (1-min tumbling window)
├─ Optimization: Could reduce to 10s windows if 40s latency unacceptable
├─ Health: All spans completed successfully
└─ SLA: Within 2-minute target ✓
```

---



## 4. DEEP DIVE: EVENT INGESTION PIPELINE

### Event Collection

🟢 **BEGINNER: How Events Get Into the System**

Think of event collection like a restaurant taking food orders through multiple channels:

**Analogy: Multi-Channel Order System**
```text
Restaurant Order Collection Methods:
├─ In-Person (Web App)
│  └─ Customer tells waiter directly
├─ Phone Call (Mobile App)
│  └─ Customer calls to place order
├─ Online Portal (Backend Service)
│  └─ Restaurant automatically orders supplies
└─ All orders go to same kitchen (our ingestion system)
```

**What is an Event?**

An event is any action a user takes that we want to track:
- **Page View**: User opens a webpage
- **Button Click**: User clicks "Add to Cart"
- **Form Submit**: User completes checkout
- **Video Play**: User starts watching a video
- **API Call**: Backend service processes a transaction

**Event Structure (Simple Example):**
```json
{
  "event_type": "button_click",
  "timestamp": "2026-01-22T14:30:00.123Z",
  "user_id": "user_12345",
  "properties": {
    "button_name": "add_to_cart",
    "product_id": "prod_789",
    "price": 29.99
  }
}
```

**Three Ways to Send Events:**

1. **JavaScript SDK (Web Apps)**
   - Runs in user's browser
   - Automatically tracks page views, clicks
   - Sends events via HTTPS to our API
   
2. **Mobile SDK (iOS/Android Apps)**
   - Runs on user's phone
   - Batches events to save battery
   - Works offline, sends later when online
   
3. **Server-Side SDK (Backend Services)**
   - Runs on company's servers
   - Tracks server events (purchases, API calls)
   - Higher reliability (servers don't go offline)

**Why Multiple Collection Methods?**
- Web app captures browsing behavior
- Mobile app captures on-the-go usage
- Server captures actual transactions (source of truth for money)

---

🟡 **INTERMEDIATE: SDK Implementation & Best Practices**

**1. JavaScript SDK Design**

**Basic Usage:**
```javascript
// Initialize SDK
analytics.init({
  apiKey: 'ak_1234567890abcdef',
  endpoint: 'https://events.analytics.com/v1/events',
  batchSize: 10,
  flushInterval: 5000 // 5 seconds
});

// Track page view (automatic)
analytics.page();

// Track custom event
analytics.track('product_viewed', {
  product_id: 'prod_789',
  category: 'electronics',
  price: 299.99
});

// Track user identity
analytics.identify('user_12345', {
  email: 'user@example.com',
  plan: 'premium'
});
```

**SDK Internals:**
```javascript
class AnalyticsSDK {
  constructor(config) {
    this.apiKey = config.apiKey;
    this.endpoint = config.endpoint;
    this.queue = [];
    this.batchSize = config.batchSize || 10;
    this.flushInterval = config.flushInterval || 5000;
    
    // Auto-flush every N seconds
    setInterval(() => this.flush(), this.flushInterval);
    
    // Flush on page unload
    window.addEventListener('beforeunload', () => this.flush());
  }
  
  track(eventType, properties) {
    const event = {
      event_type: eventType,
      timestamp: new Date().toISOString(),
      user_id: this.getUserId(),
      session_id: this.getSessionId(),
      properties: properties,
      // Auto-captured context
      context: {
        page: {
          url: window.location.href,
          title: document.title,
          referrer: document.referrer
        },
        user_agent: navigator.userAgent,
        screen: {
          width: window.screen.width,
          height: window.screen.height
        }
      }
    };
    
    this.queue.push(event);
    
    // Flush if batch size reached
    if (this.queue.length >= this.batchSize) {
      this.flush();
    }
  }
  
  flush() {
    if (this.queue.length === 0) return;
    
    const batch = this.queue.splice(0, this.batchSize);
    
    // Send via fetch with retry logic
    this.sendBatch(batch)
      .catch(error => {
        // Put back in queue on failure
        this.queue.unshift(...batch);
        console.error('Failed to send events:', error);
      });
  }
  
  async sendBatch(events) {
    const response = await fetch(this.endpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': this.apiKey
      },
      body: JSON.stringify({ events }),
      // Use beacon API for better reliability on page unload
      keepalive: true
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return response.json();
  }
  
  getUserId() {
    // Get from cookie or localStorage
    return localStorage.getItem('analytics_user_id') || 'anonymous';
  }
  
  getSessionId() {
    // Session expires after 30 min of inactivity
    const sessionKey = 'analytics_session_id';
    const sessionTimeKey = 'analytics_session_time';
    const sessionTimeout = 30 * 60 * 1000; // 30 minutes
    
    const now = Date.now();
    const lastActivity = parseInt(localStorage.getItem(sessionTimeKey) || '0');
    
    if (now - lastActivity > sessionTimeout) {
      // New session
      const newSessionId = 'sess_' + Math.random().toString(36).substr(2, 9);
      localStorage.setItem(sessionKey, newSessionId);
    }
    
    localStorage.setItem(sessionTimeKey, now.toString());
    return localStorage.getItem(sessionKey);
  }
}
```

**2. Mobile SDK Considerations**

**iOS Swift Example:**
```swift
class AnalyticsSDK {
    private var queue: [Event] = []
    private var timer: Timer?
    
    init(apiKey: String) {
        self.apiKey = apiKey
        
        // Restore queue from disk on app launch
        restoreQueue()
        
        // Monitor network reachability
        startNetworkMonitoring()
        
        // Flush on app background
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(onBackground),
            name: UIApplication.didEnterBackgroundNotification,
            object: nil
        )
    }
    
    func track(eventType: String, properties: [String: Any]) {
        let event = Event(
            eventType: eventType,
            timestamp: Date(),
            userId: getUserId(),
            properties: properties
        )
        
        queue.append(event)
        persistQueue() // Save to disk
        
        if queue.count >= 20 || hasNetworkConnection() {
            flush()
        }
    }
    
    func flush() {
        guard !queue.isEmpty && hasNetworkConnection() else { return }
        
        let batch = Array(queue.prefix(50))
        
        sendBatch(batch) { success in
            if success {
                // Remove sent events from queue
                self.queue.removeFirst(batch.count)
                self.persistQueue()
            }
        }
    }
    
    @objc func onBackground() {
        // Try to flush when app goes to background
        flush()
        persistQueue() // Ensure events saved
    }
}
```

**Mobile SDK Challenges:**
- **Battery Efficiency**: Batch events, don't send individually
- **Offline Support**: Persist events to disk, send when online
- **App Crashes**: Save queue to disk frequently
- **Background Limits**: iOS gives ~30 seconds in background

**3. Server-Side SDK (Node.js Example)**

```javascript
const Analytics = require('@analytics/server-sdk');

const analytics = new Analytics({
  apiKey: process.env.ANALYTICS_API_KEY,
  flushAt: 100, // Flush after 100 events
  flushInterval: 10000 // Or every 10 seconds
});

// Track server-side event
app.post('/api/checkout', async (req, res) => {
  const order = await processOrder(req.body);
  
  // Track order event
  analytics.track({
    userId: req.user.id,
    event: 'order_completed',
    properties: {
      order_id: order.id,
      total: order.total,
      items: order.items.length,
      payment_method: order.payment_method
    },
    timestamp: new Date()
  });
  
  res.json({ success: true, order_id: order.id });
});

// Graceful shutdown: flush remaining events
process.on('SIGTERM', async () => {
  await analytics.flush();
  process.exit(0);
});
```

**Server-Side Benefits:**
- **Accurate Data**: Can't be blocked by ad-blockers
- **Secure**: API keys not exposed to client
- **Complete**: Always captures events (no offline users)

**4. Batch API Design**

**Endpoint:** `POST /v1/events/batch`

**Request Format:**
```json
{
  "api_key": "ak_1234567890abcdef",
  "events": [
    {
      "event_type": "page_view",
      "timestamp": "2026-01-22T14:30:00.123Z",
      "user_id": "user_12345",
      "properties": {
        "page": "/products",
        "referrer": "/home"
      }
    },
    {
      "event_type": "button_click",
      "timestamp": "2026-01-22T14:30:05.456Z",
      "user_id": "user_12345",
      "properties": {
        "button": "add_to_cart",
        "product_id": "prod_789"
      }
    }
  ]
}
```

**Response Format:**
```json
{
  "success": true,
  "accepted": 2,
  "rejected": 0,
  "errors": []
}
```

**API Design Principles:**
1. **Idempotency**: Accept duplicate event IDs, deduplicate server-side
2. **Partial Success**: Accept valid events even if some invalid
3. **Error Details**: Return which events failed and why
4. **Rate Limiting**: 1000 events/second per API key
5. **Size Limits**: Max 100 events per batch, 10KB per event

**Real Company Examples:**

**Mixpanel SDK Evolution:**
- **2014**: Individual event POST requests → High latency
- **2017**: Batching (10 events) → 10x fewer requests
- **2020**: Compression (gzip) → 70% bandwidth reduction
- **Result**: Can handle 10x more traffic on same infrastructure

**Amplitude's Data Quality Features:**
- **Schema Enforcement**: Define required properties per event type
- **PII Scrubbing**: Automatically detect and remove email/phone from properties
- **Property Limits**: Max 1000 properties per event (prevent abuse)
- **Type Checking**: Enforce property types (string, number, boolean)

---

🔴 **ADVANCED: Production-Grade Ingestion**

**1. High-Availability Ingestion Architecture**

```text
Production Ingestion Stack:

Layer 1: Global Load Balancer (Cloudflare/AWS Route53)
├─ Purpose: Geographic routing
├─ Configuration:
│  ├─ US traffic → us-east-1
│  ├─ EU traffic → eu-west-1
│  ├─ APAC traffic → ap-southeast-1
│  └─ Latency-based routing
├─ DDoS protection: Cloudflare (5M requests/sec capacity)
└─ Failover: Automatic within 30 seconds

Layer 2: Regional API Gateway (AWS ALB)
├─ Purpose: SSL termination, request routing
├─ Configuration:
│  ├─ SSL/TLS 1.3 (A+ SSL Labs rating)
│  ├─ HTTP/2 support (multiplexing)
│  ├─ Connection pooling (reduce latency)
│  └─ Health checks every 5 seconds
├─ Capacity: 10,000 connections/sec per AZ
└─ Auto-scaling: Add targets at 70% CPU

Layer 3: Ingestion Service (Kubernetes Pods)
├─ Purpose: Event validation, enrichment, Kafka writes
├─ Configuration:
│  ├─ Replicas: 6 (2 per AZ)
│  ├─ Resources: 2 CPU, 4GB RAM per pod
│  ├─ HPA: Scale 6-20 based on CPU (target: 70%)
│  └─ PDB: Min 4 pods available during updates
├─ Capacity: 200 events/sec per pod = 1,200 events/sec total
└─ Graceful shutdown: 30-second drain period

Layer 4: Message Queue (Kafka)
├─ Purpose: Durable event storage
├─ Configuration:
│  ├─ Brokers: 5 per region
│  ├─ Replication: 3x (min in-sync: 2)
│  ├─ Partitions: 24 per topic
│  └─ Retention: 7 days
├─ Capacity: 50K events/sec per broker = 250K events/sec
└─ Monitoring: Lag, throughput, disk usage
```

**2. Circuit Breaker Pattern**

```python
from circuitbreaker import CircuitBreaker, CircuitBreakerError

class KafkaProducer:
    def __init__(self):
        self.producer = kafka.KafkaProducer(...)
        # Circuit breaker: Open after 5 failures, retry after 60s
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            recovery_timeout=60,
            expected_exception=kafka.errors.KafkaError
        )
    
    @circuit_breaker
    def send_event(self, topic, event):
        """Send event to Kafka with circuit breaker protection"""
        future = self.producer.send(topic, value=event)
        # Wait for acknowledgment (synchronous for reliability)
        record_metadata = future.get(timeout=5)
        return record_metadata
    
    def send_event_with_fallback(self, topic, event):
        """Send event with fallback to S3 if Kafka unavailable"""
        try:
            return self.send_event(topic, event)
        except CircuitBreakerError:
            # Circuit open: Kafka is down
            logger.warning("Circuit breaker open, writing to S3 fallback")
            self.write_to_s3_fallback(event)
            # Return success to client (we'll process from S3 later)
            return {"status": "accepted", "fallback": True}
        except Exception as e:
            logger.error(f"Failed to send event: {e}")
            raise

    def write_to_s3_fallback(self, event):
        """Write event to S3 for later processing"""
        s3_key = f"fallback/{datetime.now().strftime('%Y/%m/%d/%H')}/{uuid.uuid4()}.json"
        s3.put_object(Bucket='events-fallback', Key=s3_key, Body=json.dumps(event))
```

---

## 5. DEEP DIVE: STREAM PROCESSING

Stream processing is the heart of the real-time analytics dashboard. This section explores how we process millions of events per second with sub-second latency, handle late-arriving data, and maintain consistent state across distributed systems.

### 🟢 Beginner Level: Stream Processing Fundamentals

**What is Stream Processing?**

Think of stream processing like a factory assembly line that never stops:
- **Traditional Batch Processing**: Like baking 1,000 cookies at once, waiting for them all to finish, then packaging them
- **Stream Processing**: Like a conveyor belt where each cookie is processed immediately as it comes off the line

**Real-World Analogy**: Airport Security Line
```
Passengers (Events) → Security Scanner (Stream Processor) → Real-time Dashboard
    ↓                         ↓                                      ↓
Continuous flow      Process each person        Update "Wait Time: 12 min"
Not batches          as they arrive             immediately
```

**Key Stream Processing Concepts:**

```
1. Event Stream
   ├─ Ordered sequence of events
   ├─ Events have timestamps
   └─ Unbounded (never-ending) data

2. Stream Processing
   ├─ Filter: Remove irrelevant events
   ├─ Transform: Convert event format
   ├─ Aggregate: Calculate metrics (count, sum, avg)
   └─ Join: Combine multiple streams

3. Output
   ├─ Real-time dashboards
   ├─ Alerts/notifications
   └─ Stored aggregations
```

**Simple Stream Processing Example:**

```python
# Counting page views per minute
def process_stream():
    window = {}  # {minute: count}
    
    for event in event_stream:
        minute = event.timestamp.floor_to_minute()
        window[minute] = window.get(minute, 0) + 1
        
        # Update dashboard every 1000 events
        if event.count % 1000 == 0:
            update_dashboard(window)
```

**Why Stream Processing for Analytics?**

```
Traditional Approach (Batch):
Event → Database → Run query every 5 minutes → Dashboard
Result: 0-5 minute delay ❌

Stream Processing:
Event → Process immediately → Dashboard
Result: Sub-second latency ✅
```

---

### 🟡 Intermediate Level: Stream Processing Frameworks

**Framework Comparison: Apache Flink vs Spark Streaming**

```
Apache Flink:
├─ True stream processing (event-at-a-time)
├─ Event-time processing with watermarks
├─ Exactly-once semantics
├─ Low latency: 10-100ms
└─ Used by: Uber, Netflix, Alibaba

Spark Streaming:
├─ Micro-batch processing (small batches)
├─ Good for mixed batch/stream workloads
├─ At-least-once by default
├─ Higher latency: 500ms-2s
└─ Used by: Pinterest, Databricks customers

Kafka Streams:
├─ Library (not framework)
├─ Runs embedded in application
├─ Simpler deployment
├─ Limited scalability vs Flink
└─ Used by: LinkedIn, Airbnb
```

**For Real-time Analytics Dashboard: Apache Flink**

Why Flink?
- True event-time processing (critical for late events)
- Low latency required for dashboards
- Exactly-once state guarantees
- Complex event processing (CEP) capabilities

**Flink Architecture for Analytics:**

```
┌─────────────────────────────────────────────────────────────┐
│                      Apache Flink Cluster                    │
├─────────────────────────────────────────────────────────────┤
│  Job Manager (Coordination)                                  │
│  ├─ Schedule tasks                                           │
│  ├─ Coordinate checkpoints                                   │
│  └─ Manage failures                                          │
├─────────────────────────────────────────────────────────────┤
│  Task Manager 1      Task Manager 2      Task Manager N      │
│  ├─ Process events   ├─ Process events   ├─ Process events  │
│  ├─ Local state      ├─ Local state      ├─ Local state     │
│  └─ Checkpointing    └─ Checkpointing    └─ Checkpointing   │
└─────────────────────────────────────────────────────────────┘
         ↑                                         ↓
    Kafka Topics                          ClickHouse/Redis
```

**Event-Time vs Processing-Time:**

```
Event-Time:
├─ When event actually happened
├─ Embedded in event: {"timestamp": "2024-01-15T10:30:00Z"}
├─ Handles late events correctly
└─ Use for: Analytics (accurate counts)

Processing-Time:
├─ When event processed by system
├─ System time: System.currentTimeMillis()
├─ Ignores late events
└─ Use for: Monitoring (system health)

Example Problem:
┌─────────────────────────────────────────────────────────┐
│ Mobile app loses connection                             │
│ User clicks at 10:00 AM (event-time)                    │
│ Event arrives at 10:15 AM (processing-time)             │
│                                                          │
│ Processing-Time: Count in 10:15 window ❌               │
│ Event-Time: Count in 10:00 window ✅                    │
└─────────────────────────────────────────────────────────┘
```

**Watermarks: Handling Late Events**

Watermark = "All events with timestamp < T have arrived"

```
Timeline:
09:00 ──────── 09:05 ──────── 09:10 ──────── 09:15
  │              │              │              │
  Events         Events         Events         Late Event!
  arrive         arrive         arrive         (09:03)

Watermark Strategy:
Current Time: 09:15
Max Event Time Seen: 09:14
Watermark: 09:14 - 5 minutes = 09:09

Late Event (09:03) < Watermark (09:09)
→ Too late! Discard or send to late data stream
```

**Watermark Configuration:**

```java
// Flink Watermark Strategy
WatermarkStrategy<Event> watermarkStrategy = 
    WatermarkStrategy
        .<Event>forBoundedOutOfOrderness(Duration.ofMinutes(5))
        .withTimestampAssigner((event, timestamp) -> event.getTimestamp());

DataStream<Event> events = kafkaSource
    .assignTimestampsAndWatermarks(watermarkStrategy);

// Allow 5 minutes of lateness
// Events arriving >5 min late are dropped
// Trade-off: Longer wait = more accurate but higher latency
```

**Watermark Trade-offs:**

```
Short Watermark (1 minute):
├─ Faster results
├─ More dropped events
└─ Use for: High-volume, low-value events

Long Watermark (1 hour):
├─ Fewer dropped events
├─ Slower results
└─ Use for: Critical business metrics

Dynamic Watermark:
├─ Adjust based on data patterns
├─ Short during business hours
└─ Long during off-hours
```

**Windowing Techniques:**

```
1. Tumbling Window (Fixed, Non-overlapping)
   ├─ Size: 1 minute
   ├─ Windows: [09:00-09:01), [09:01-09:02), [09:02-09:03)
   ├─ Each event in exactly ONE window
   └─ Use: Hourly/daily aggregations

   Example: Page views per minute
   09:00-09:01: 1,523 views
   09:01-09:02: 1,847 views
   09:02-09:03: 1,392 views

2. Sliding Window (Overlapping)
   ├─ Size: 5 minutes
   ├─ Slide: 1 minute
   ├─ Windows: [09:00-09:05), [09:01-09:06), [09:02-09:07)
   ├─ Each event in MULTIPLE windows
   └─ Use: Moving averages, trend analysis

   Example: 5-minute moving average response time
   09:00-09:05: 143ms avg
   09:01-09:06: 156ms avg
   09:02-09:07: 128ms avg

3. Session Window (Activity-based)
   ├─ Gap: 30 minutes inactivity
   ├─ Windows: Variable size based on user activity
   ├─ New window after 30 min of no events
   └─ Use: User sessions, engagement analysis

   Example: User session duration
   User123: 09:00 → 09:25 (25 min session)
   User456: 10:15 → 11:02 (47 min session)

4. Global Window (All time)
   ├─ Single window for all events
   ├─ Custom triggers needed
   └─ Use: Cumulative counts, unique visitors
```

**Window Implementation in Flink:**

```java
// Tumbling Window: Count events per minute
DataStream<Event> events = ...;

DataStream<WindowResult> results = events
    .keyBy(event -> event.getTenantId())
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .aggregate(new CountAggregateFunction());

// Sliding Window: 5-minute average, updated every minute
DataStream<WindowResult> movingAvg = events
    .keyBy(event -> event.getMetricName())
    .window(SlidingEventTimeWindows.of(
        Time.minutes(5),  // window size
        Time.minutes(1)   // slide interval
    ))
    .aggregate(new AverageAggregateFunction());

// Session Window: User sessions with 30-min gap
DataStream<SessionResult> sessions = events
    .keyBy(event -> event.getUserId())
    .window(EventTimeSessionWindows.withGap(Time.minutes(30)))
    .aggregate(new SessionAggregateFunction());
```

**State Management:**

```
Why State Needed:
├─ Remember counts across events
├─ Store intermediate aggregations
├─ Handle duplicate detection
└─ Maintain session information

State Types in Flink:
1. ValueState<T>
   └─ Single value (e.g., current count)

2. ListState<T>
   └─ List of values (e.g., recent events)

3. MapState<K, V>
   └─ Key-value pairs (e.g., user sessions)

4. ReducingState<T>
   └─ Aggregated value (e.g., sum, max)
```

**Checkpointing: Fault Tolerance**

```
Checkpoint Process:
1. Job Manager: "Save state now!"
2. Task Managers: Pause processing
3. Save state to distributed storage (S3, HDFS)
4. Resume processing

Checkpoint Configuration:
├─ Interval: Every 60 seconds
├─ Storage: S3 (durable)
├─ Timeout: 10 minutes
└─ Concurrent Checkpoints: 1

Recovery After Failure:
1. Task Manager crashes
2. Job Manager detects failure
3. Restore from last checkpoint
4. Replay events from Kafka (from checkpoint offset)
5. Resume processing
```

---

### 🔴 Advanced Level: Production Stream Processing

**Complex Stream Processing Pipeline:**

```java
// Real-world Flink job for analytics dashboard
public class AnalyticsPipeline {
    
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = 
            StreamExecutionEnvironment.getExecutionEnvironment();
        
        // Checkpointing configuration
        env.enableCheckpointing(60000); // 1 minute
        env.getCheckpointConfig().setCheckpointStorage("s3://checkpoints/");
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(30000);
        env.getCheckpointConfig().setCheckpointTimeout(600000);
        
        // Kafka source
        KafkaSource<Event> kafkaSource = KafkaSource.<Event>builder()
            .setBootstrapServers("kafka:9092")
            .setTopics("raw-events")
            .setGroupId("analytics-processor")
            .setStartingOffsets(OffsetsInitializer.earliest())
            .setDeserializer(new EventDeserializationSchema())
            .build();
        
        // Watermark strategy: 5-minute out-of-orderness
        WatermarkStrategy<Event> watermarkStrategy = 
            WatermarkStrategy
                .<Event>forBoundedOutOfOrderness(Duration.ofMinutes(5))
                .withTimestampAssigner((event, ts) -> event.getEventTime())
                .withIdleness(Duration.ofMinutes(1)); // Handle idle partitions
        
        DataStream<Event> events = env
            .fromSource(kafkaSource, watermarkStrategy, "Kafka Source")
            .setParallelism(20); // Match Kafka partition count
        
        // 1. Real-time aggregations (1-minute tumbling windows)
        DataStream<MetricResult> realtimeMetrics = events
            .filter(event -> event.isValid())
            .keyBy(event -> new MetricKey(
                event.getTenantId(), 
                event.getMetricName()
            ))
            .window(TumblingEventTimeWindows.of(Time.minutes(1)))
            .allowedLateness(Time.minutes(5)) // Process late events
            .sideOutputLateData(lateDataTag) // Capture very late events
            .aggregate(
                new MetricAggregateFunction(),
                new MetricWindowFunction()
            )
            .setParallelism(40);
        
        // 2. Session windows for user engagement
        DataStream<SessionResult> userSessions = events
            .filter(event -> event.getEventType().equals("user_action"))
            .keyBy(event -> event.getUserId())
            .window(EventTimeSessionWindows.withGap(Time.minutes(30)))
            .aggregate(new SessionAggregateFunction())
            .setParallelism(20);
        
        // 3. Complex event processing: Funnel analysis
        Pattern<Event, ?> funnelPattern = Pattern
            .<Event>begin("page_view")
                .where(evt -> evt.getEventType().equals("page_view"))
            .followedBy("add_to_cart")
                .where(evt -> evt.getEventType().equals("add_to_cart"))
                .within(Time.hours(1))
            .followedBy("purchase")
                .where(evt -> evt.getEventType().equals("purchase"))
                .within(Time.hours(24));
        
        PatternStream<Event> funnelStream = CEP.pattern(
            events.keyBy(Event::getUserId),
            funnelPattern
        );
        
        DataStream<FunnelResult> funnelResults = funnelStream
            .select(new FunnelSelectFunction())
            .setParallelism(10);
        
        // 4. Write to ClickHouse
        realtimeMetrics
            .addSink(new ClickHouseSink<>(
                ClickHouseConfig.builder()
                    .host("clickhouse:8123")
                    .database("analytics")
                    .table("realtime_metrics")
                    .batchSize(10000)
                    .flushInterval(Duration.ofSeconds(5))
                    .build()
            ))
            .name("ClickHouse Sink")
            .setParallelism(10);
        
        // 5. Write to Redis for low-latency queries
        realtimeMetrics
            .addSink(new RedisSink<>(
                new RedisConfig("redis:6379"),
                new MetricRedisMapper()
            ))
            .name("Redis Sink")
            .setParallelism(5);
        
        // 6. Side output: Late events monitoring
        realtimeMetrics
            .getSideOutput(lateDataTag)
            .addSink(new KafkaSink<>("late-events-topic"))
            .name("Late Events Sink");
        
        env.execute("Real-time Analytics Pipeline");
    }
}
```

**Advanced Aggregation Function:**

```java
// Custom aggregate function with state
public class MetricAggregateFunction 
    implements AggregateFunction<Event, MetricAccumulator, MetricResult> {
    
    @Override
    public MetricAccumulator createAccumulator() {
        return new MetricAccumulator();
    }
    
    @Override
    public MetricAccumulator add(Event event, MetricAccumulator acc) {
        acc.count++;
        acc.sum += event.getValue();
        acc.sumOfSquares += event.getValue() * event.getValue();
        acc.min = Math.min(acc.min, event.getValue());
        acc.max = Math.max(acc.max, event.getValue());
        
        // T-Digest for percentile calculation
        acc.tdigest.add(event.getValue());
        
        // HyperLogLog for unique counts
        acc.hll.offer(event.getUserId());
        
        return acc;
    }
    
    @Override
    public MetricResult getResult(MetricAccumulator acc) {
        double avg = acc.sum / acc.count;
        double stddev = Math.sqrt(
            (acc.sumOfSquares / acc.count) - (avg * avg)
        );
        
        return MetricResult.builder()
            .count(acc.count)
            .sum(acc.sum)
            .avg(avg)
            .min(acc.min)
            .max(acc.max)
            .stddev(stddev)
            .p50(acc.tdigest.quantile(0.5))
            .p95(acc.tdigest.quantile(0.95))
            .p99(acc.tdigest.quantile(0.99))
            .uniqueUsers(acc.hll.cardinality())
            .build();
    }
    
    @Override
    public MetricAccumulator merge(MetricAccumulator a, MetricAccumulator b) {
        a.count += b.count;
        a.sum += b.sum;
        a.sumOfSquares += b.sumOfSquares;
        a.min = Math.min(a.min, b.min);
        a.max = Math.max(a.max, b.max);
        a.tdigest.add(b.tdigest);
        a.hll.addAll(b.hll);
        return a;
    }
}
```

**State Backend Configuration:**

```java
// Production state backend: RocksDB
StateBackend stateBackend = new EmbeddedRocksDBStateBackend(true);
env.setStateBackend(stateBackend);

// RocksDB configuration for high throughput
RocksDBStateBackend rocksDBStateBackend = new RocksDBStateBackend(
    "s3://flink-state/checkpoints",
    true // enable incremental checkpoints
);

// Tune RocksDB for analytics workload
rocksDBStateBackend.setPredefinedOptions(
    PredefinedOptions.SPINNING_DISK_OPTIMIZED_HIGH_MEM
);

// Custom RocksDB options
rocksDBStateBackend.setOptions(new OptionsFactory() {
    @Override
    public DBOptions createDBOptions(DBOptions currentOptions) {
        return currentOptions
            .setMaxBackgroundJobs(4)
            .setMaxOpenFiles(-1);
    }
    
    @Override
    public ColumnFamilyOptions createColumnOptions(
        ColumnFamilyOptions currentOptions
    ) {
        return currentOptions
            .setCompactionStyle(CompactionStyle.LEVEL)
            .setLevelCompactionDynamicLevelBytes(true)
            .setTargetFileSizeBase(256 * 1024 * 1024); // 256 MB
    }
});
```

**Handling Backpressure:**

```
Backpressure Symptoms:
├─ Increasing event lag
├─ Growing checkpoint duration
├─ High CPU on task managers
└─ Network buffer saturation

Solutions:
1. Increase Parallelism
   └─ More task slots to distribute load

2. Optimize Aggregations
   └─ Pre-aggregate before shuffle

3. Tune Network Buffers
   taskmanager.network.memory.fraction: 0.2
   taskmanager.network.memory.max: 2gb

4. Async I/O for External Calls
   └─ Don't block on database writes

5. Operator Chaining
   └─ Combine operators to reduce shuffles
```

**Company Examples:**

**Uber's Real-time Analytics (Flink):**
- 10 trillion events/day
- 4,000 Flink jobs
- Sub-second latency for trip metrics
- Handles 2-hour late events from offline drivers

**Netflix's Stream Processing (Flink):**
- 8 million events/second
- Real-time quality metrics during streaming
- Auto-scaling based on event rate
- 99.99% accuracy with 5-minute watermarks

**Alibaba's Transaction Monitoring (Flink):**
- 500,000 orders/second during Singles' Day
- Real-time fraud detection
- 100ms p99 latency
- Exactly-once processing guarantees

---


## 6. DEEP DIVE: DATA STORAGE

Choosing the right database is critical for a real-time analytics dashboard. This section explores why ClickHouse dominates the OLAP space, how to design storage tiers, and strategies for managing petabytes of time-series data.

### 🟢 Beginner Level: Database Selection for Analytics

**Why Not Traditional Databases?**

Think of databases like different types of filing systems:

```
Traditional Database (PostgreSQL):
├─ Like a library card catalog
├─ Organized for finding ONE specific book quickly
├─ Slow when counting ALL books from year 2020
└─ OLTP (Online Transaction Processing)

Analytics Database (ClickHouse):
├─ Like a warehouse inventory system
├─ Organized for counting/summing groups of items
├─ Fast for "How many items sold last month?"
└─ OLAP (Online Analytical Processing)
```

**Real-World Analogy**: Grocery Store vs Warehouse

```
Grocery Store (OLTP - PostgreSQL):
├─ Find: "Where is milk?" → Aisle 3, Shelf 2
├─ Optimized: Individual item lookups
├─ Transaction: Buy 3 items, update inventory
└─ Query Time: Milliseconds

Warehouse (OLAP - ClickHouse):
├─ Find: "Total sales of dairy products this month?"
├─ Optimized: Aggregate across millions of transactions
├─ Analysis: Sum all dairy sales, group by week
└─ Query Time: Sub-second even with billions of rows
```

**Database Comparison for Analytics:**

```
PostgreSQL (OLTP):
├─ Row-oriented storage
├─ ACID transactions
├─ Good for: Individual record updates
├─ Bad for: Scanning millions of rows
└─ Analytics Query: 30 seconds ❌

ClickHouse (OLAP):
├─ Column-oriented storage
├─ Immutable data
├─ Good for: Scanning/aggregating billions of rows
├─ Bad for: Individual row updates
└─ Analytics Query: 200ms ✅

Cassandra (NoSQL):
├─ Wide-column store
├─ Good for: Time-series writes
├─ Bad for: Complex aggregations, joins
└─ Analytics Query: Slow, requires Spark ❌
```

**Why Column-Oriented Storage Wins:**

```
Row-Oriented (PostgreSQL):
┌─────────────────────────────────────────────┐
│ Row 1: [user_id, page, timestamp, duration] │
│ Row 2: [user_id, page, timestamp, duration] │
│ Row 3: [user_id, page, timestamp, duration] │
└─────────────────────────────────────────────┘
Query: "SELECT AVG(duration) FROM events"
Must read: ALL 4 columns for ALL rows ❌

Column-Oriented (ClickHouse):
┌──────────────┬──────────┬───────────┬──────────┐
│ user_id      │ page     │ timestamp │ duration │
│ [values...]  │ [vals...] │ [vals...] │ [vals...] │
└──────────────┴──────────┴───────────┴──────────┘
Query: "SELECT AVG(duration) FROM events"
Must read: ONLY duration column ✅

Compression Bonus:
Same values together → Better compression
[1, 1, 1, 2, 2, 3] → "1×3, 2×2, 3×1"
Storage reduction: 80-90%
```

---

### 🟡 Intermediate Level: ClickHouse Deep Dive

**ClickHouse Architecture:**

```
┌─────────────────────────────────────────────────────────┐
│                    ClickHouse Cluster                    │
├─────────────────────────────────────────────────────────┤
│  Shard 1          Shard 2          Shard 3              │
│  ├─ Primary       ├─ Primary       ├─ Primary           │
│  └─ Replica       └─ Replica       └─ Replica           │
├─────────────────────────────────────────────────────────┤
│  Data Distribution: Consistent Hashing by tenant_id      │
│  Replication: 2x (each shard has 1 replica)             │
│  Query Distribution: Load balanced across all nodes      │
└─────────────────────────────────────────────────────────┘
```

**Why ClickHouse Over Alternatives:**

```
ClickHouse vs PostgreSQL:
├─ Query Speed: 100-1000x faster for aggregations
├─ Storage: 10x more compression (columnar)
├─ Scalability: Horizontal sharding built-in
└─ Use Case: Real-time analytics dashboards

ClickHouse vs Cassandra:
├─ Query Flexibility: Complex SQL, joins, subqueries
├─ Aggregations: Native COUNT, SUM, AVG, percentiles
├─ Learning Curve: SQL vs CQL
└─ Use Case: When you need rich analytics queries

ClickHouse vs Druid:
├─ SQL Support: Full SQL vs limited
├─ Deployment: Simpler (fewer components)
├─ Real-time Ingestion: Both excellent
└─ Use Case: When SQL flexibility matters

ClickHouse vs Pinot:
├─ Maturity: More mature ecosystem
├─ Query Language: Standard SQL
├─ Performance: Similar, ClickHouse slightly faster
└─ Use Case: General-purpose analytics
```

**ClickHouse Table Engines:**

```
1. MergeTree (Default for Analytics)
   ├─ Sorted by primary key
   ├─ Automatic background merges
   ├─ Excellent compression
   └─ Use: General time-series data

2. ReplacingMergeTree
   ├─ Deduplication by primary key
   ├─ Keeps latest version
   └─ Use: Events with potential duplicates

3. SummingMergeTree
   ├─ Pre-aggregates numeric columns
   ├─ Reduces storage for counts/sums
   └─ Use: Pre-aggregated metrics

4. AggregatingMergeTree
   ├─ Stores intermediate aggregation states
   ├─ Combine results from multiple shards
   └─ Use: Complex aggregations (percentiles, uniq)

5. Distributed
   ├─ Virtual table across shards
   ├─ Query routing and aggregation
   └─ Use: Cluster-wide queries
```

**Storage Tiers: Hot/Warm/Cold**

```
Hot Storage (SSD - Last 7 days):
├─ Access Pattern: High frequency (dashboard queries)
├─ Storage: NVMe SSD
├─ Query Time: <100ms
├─ Cost: $200/TB/month
└─ Data: Recent metrics, live dashboards

Warm Storage (HDD - 8-90 days):
├─ Access Pattern: Medium frequency (historical analysis)
├─ Storage: SATA HDD
├─ Query Time: <1 second
├─ Cost: $30/TB/month
└─ Data: Historical reports, trends

Cold Storage (S3 - >90 days):
├─ Access Pattern: Low frequency (compliance, rare queries)
├─ Storage: Object storage (S3, GCS)
├─ Query Time: 5-30 seconds
├─ Cost: $5/TB/month
└─ Data: Archived events, regulatory retention

Automatic Tiering:
┌────────────────────────────────────────────────┐
│ Day 0-7:   SSD  (100ms queries)                │
│ Day 8-90:  HDD  (1s queries) ← Auto-moved      │
│ Day 91+:   S3   (30s queries) ← Auto-archived  │
└────────────────────────────────────────────────┘
```

**Partitioning Strategies:**

```
1. Time-based Partitioning (Most Common)
   ├─ Partition by day: toYYYYMMDD(timestamp)
   ├─ Benefits: Easy to drop old partitions
   ├─ Query optimization: Partition pruning
   └─ Example: events_20240115, events_20240116

2. Multi-dimensional Partitioning
   ├─ Partition by (tenant_id, toYYYYMMDD(timestamp))
   ├─ Benefits: Tenant isolation, parallel queries
   ├─ Data locality: Each tenant's data together
   └─ Example: tenant_123_20240115

3. Hybrid Partitioning
   ├─ Hot data: Hourly partitions (more granular)
   ├─ Warm data: Daily partitions (coarser)
   └─ Optimize: Query patterns vs partition overhead
```

**Partition Management:**

```sql
-- Drop old partitions (GDPR compliance, cost savings)
ALTER TABLE events 
DROP PARTITION '20231201';

-- Move partition to cold storage
ALTER TABLE events 
MOVE PARTITION '20231215' TO VOLUME 'cold';

-- Freeze partition (backup)
ALTER TABLE events 
FREEZE PARTITION '20240115';

-- Automated TTL (Time To Live)
ALTER TABLE events 
MODIFY TTL timestamp + INTERVAL 90 DAY;
```

**Data Retention and TTL:**

```
Retention Policy Example:
├─ Raw Events: 7 days (detailed, high cardinality)
├─ 1-minute Aggregates: 30 days
├─ 1-hour Aggregates: 90 days
├─ 1-day Aggregates: 2 years
└─ Monthly Aggregates: Forever

TTL Configuration:
┌───────────────────────────────────────────────────┐
│ Table: raw_events                                  │
│ TTL: timestamp + INTERVAL 7 DAY                    │
│ Action: DELETE                                     │
│                                                    │
│ Table: hourly_aggregates                          │
│ TTL: timestamp + INTERVAL 90 DAY TO DISK 'cold'   │
│ Then: timestamp + INTERVAL 180 DAY DELETE         │
└───────────────────────────────────────────────────┘
```

**Indexing Strategies:**

```
Primary Key (Sorting Key):
├─ Determines physical data order
├─ Enables skip indexes
├─ Choose based on query patterns
└─ Example: PRIMARY KEY (tenant_id, timestamp)

Skip Indexes (Secondary):
├─ Bloom filter: High-cardinality columns (user_id)
├─ MinMax: Numeric ranges (price, duration)
├─ Set: Low-cardinality columns (status, type)
└─ NGram: Text search

Example:
CREATE TABLE events (
    tenant_id UInt32,
    timestamp DateTime,
    user_id String,
    event_type String,
    value Float64
)
ENGINE = MergeTree()
PARTITION BY toYYYYMMDD(timestamp)
ORDER BY (tenant_id, timestamp, event_type)
INDEX user_bloom_idx user_id TYPE bloom_filter GRANULARITY 4
INDEX event_set_idx event_type TYPE set(100) GRANULARITY 1;
```

---

### 🔴 Advanced Level: Production ClickHouse Configuration

**Complete Table Schema with Optimizations:**

```sql
-- Raw events table (hot storage)
CREATE TABLE analytics.events_local ON CLUSTER main_cluster
(
    tenant_id UInt32,
    event_id UUID,
    timestamp DateTime64(3),
    event_time DateTime64(3),  -- Original event time (not ingestion time)
    event_type LowCardinality(String),
    user_id String,
    session_id String,
    page_url String,
    referrer String,
    user_agent String,
    ip_address IPv4,
    country LowCardinality(String),
    device_type LowCardinality(String),
    
    -- Metrics
    duration_ms UInt32,
    value Float64,
    
    -- Metadata (JSON for flexibility)
    properties String,  -- JSON string
    
    -- Computed columns
    date Date MATERIALIZED toDate(event_time),
    hour UInt8 MATERIALIZED toHour(event_time),
    
    -- Deduplication
    _version UInt64
)
ENGINE = ReplicatedReplacingMergeTree(
    '/clickhouse/tables/{shard}/events_local',
    '{replica}',
    _version
)
PARTITION BY (tenant_id, toYYYYMMDD(event_time))
ORDER BY (tenant_id, event_time, event_type, user_id)
SAMPLE BY sipHash64(user_id)
TTL event_time + INTERVAL 7 DAY TO VOLUME 'warm',
    event_time + INTERVAL 90 DAY TO VOLUME 'cold',
    event_time + INTERVAL 365 DAY DELETE
SETTINGS
    index_granularity = 8192,
    merge_with_ttl_timeout = 3600;

-- Skip indexes for performance
ALTER TABLE analytics.events_local 
ADD INDEX user_id_bloom user_id TYPE bloom_filter GRANULARITY 4;

ALTER TABLE analytics.events_local 
ADD INDEX event_type_set event_type TYPE set(1000) GRANULARITY 1;

ALTER TABLE analytics.events_local 
ADD INDEX page_url_tokenbf page_url TYPE tokenbf_v1(10240, 3, 0) GRANULARITY 4;

-- Distributed table (query interface)
CREATE TABLE analytics.events ON CLUSTER main_cluster
AS analytics.events_local
ENGINE = Distributed(
    main_cluster,
    analytics,
    events_local,
    sipHash64(user_id)
);
```

**Pre-aggregated Metrics Table:**

```sql
-- Materialized view for real-time aggregations
CREATE MATERIALIZED VIEW analytics.metrics_1min_mv
TO analytics.metrics_1min
AS SELECT
    tenant_id,
    toStartOfMinute(event_time) AS minute,
    event_type,
    country,
    device_type,
    
    -- Aggregations
    count() AS event_count,
    uniq(user_id) AS unique_users,
    uniq(session_id) AS unique_sessions,
    
    -- Percentiles (using quantile state)
    quantileState(0.50)(duration_ms) AS p50_state,
    quantileState(0.95)(duration_ms) AS p95_state,
    quantileState(0.99)(duration_ms) AS p99_state,
    
    -- Basic stats
    sum(value) AS total_value,
    avg(value) AS avg_value,
    min(value) AS min_value,
    max(value) AS max_value
FROM analytics.events_local
GROUP BY
    tenant_id,
    minute,
    event_type,
    country,
    device_type;

-- Metrics storage table
CREATE TABLE analytics.metrics_1min_local ON CLUSTER main_cluster
(
    tenant_id UInt32,
    minute DateTime,
    event_type LowCardinality(String),
    country LowCardinality(String),
    device_type LowCardinality(String),
    
    event_count UInt64,
    unique_users UInt64,
    unique_sessions UInt64,
    
    p50_state AggregateFunction(quantile(0.50), UInt32),
    p95_state AggregateFunction(quantile(0.95), UInt32),
    p99_state AggregateFunction(quantile(0.99), UInt32),
    
    total_value Float64,
    avg_value Float64,
    min_value Float64,
    max_value Float64
)
ENGINE = ReplicatedSummingMergeTree(
    '/clickhouse/tables/{shard}/metrics_1min_local',
    '{replica}'
)
PARTITION BY toYYYYMM(minute)
ORDER BY (tenant_id, minute, event_type, country, device_type)
TTL minute + INTERVAL 30 DAY
SETTINGS index_granularity = 8192;

-- Query example using pre-aggregated data
SELECT
    minute,
    sum(event_count) AS total_events,
    sum(unique_users) AS total_users,
    quantileMerge(0.50)(p50_state) AS p50,
    quantileMerge(0.95)(p95_state) AS p95,
    quantileMerge(0.99)(p99_state) AS p99
FROM analytics.metrics_1min
WHERE
    tenant_id = 123
    AND minute >= now() - INTERVAL 1 HOUR
GROUP BY minute
ORDER BY minute;
```

**Multi-tier Storage Configuration:**

```xml
<!-- config.xml -->
<yandex>
    <storage_configuration>
        <disks>
            <!-- Hot storage: NVMe SSD -->
            <hot>
                <type>local</type>
                <path>/mnt/nvme/clickhouse/</path>
            </hot>
            
            <!-- Warm storage: SATA HDD -->
            <warm>
                <type>local</type>
                <path>/mnt/hdd/clickhouse/</path>
            </warm>
            
            <!-- Cold storage: S3 -->
            <cold>
                <type>s3</type>
                <endpoint>https://s3.amazonaws.com/analytics-cold/</endpoint>
                <access_key_id>ACCESS_KEY</access_key_id>
                <secret_access_key>SECRET</secret_access_key>
                <region>us-east-1</region>
            </cold>
        </disks>
        
        <policies>
            <tiered>
                <volumes>
                    <hot>
                        <disk>hot</disk>
                        <max_data_part_size_bytes>1073741824</max_data_part_size_bytes>
                    </hot>
                    <warm>
                        <disk>warm</disk>
                        <max_data_part_size_bytes>10737418240</max_data_part_size_bytes>
                    </warm>
                    <cold>
                        <disk>cold</disk>
                    </cold>
                </volumes>
                <move_factor>0.1</move_factor>
            </tiered>
        </policies>
    </storage_configuration>
</yandex>
```

**Replication and Sharding:**

```xml
<!-- Cluster configuration -->
<yandex>
    <remote_servers>
        <main_cluster>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ch-node-1</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>ch-node-2</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ch-node-3</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>ch-node-4</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>ch-node-5</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>ch-node-6</host>
                    <port>9000</port>
                </replica>
            </shard>
        </main_cluster>
    </remote_servers>
</yandex>
```

**Query Optimization Techniques:**

```sql
-- 1. PREWHERE: Filter before reading all columns
SELECT
    user_id,
    event_type,
    duration_ms,
    properties
FROM events
PREWHERE tenant_id = 123 AND date = today()
WHERE event_type IN ('click', 'view');

-- 2. SAMPLE: Query subset for approximation
SELECT
    country,
    avg(duration_ms) AS avg_duration
FROM events
SAMPLE 0.1  -- Query 10% of data
WHERE date >= today() - 30
GROUP BY country;

-- 3. JOIN optimization: Right table is small
SELECT
    e.event_type,
    u.user_segment,
    count() AS cnt
FROM events AS e
INNER JOIN user_segments AS u ON e.user_id = u.user_id
WHERE e.date = today()
GROUP BY e.event_type, u.user_segment;

-- 4. Distributed queries: Use _shard_num for debugging
SELECT
    _shard_num AS shard,
    count() AS events_per_shard
FROM events
WHERE date = today()
GROUP BY _shard_num;
```

**Performance Tuning:**

```xml
<!-- users.xml -->
<yandex>
    <profiles>
        <default>
            <!-- Memory limits -->
            <max_memory_usage>10000000000</max_memory_usage>
            <max_bytes_before_external_group_by>8000000000</max_bytes_before_external_group_by>
            
            <!-- Query complexity limits -->
            <max_rows_to_read>1000000000</max_rows_to_read>
            <max_execution_time>300</max_execution_time>
            
            <!-- Parallelism -->
            <max_threads>8</max_threads>
            
            <!-- Insert optimization -->
            <async_insert>1</async_insert>
            <wait_for_async_insert>0</wait_for_async_insert>
            <async_insert_max_data_size>10000000</async_insert_max_data_size>
            <async_insert_busy_timeout_ms>1000</async_insert_busy_timeout_ms>
        </default>
    </profiles>
</yandex>
```

**Company Examples:**

**Cloudflare's Analytics (ClickHouse):**
- 6 million HTTP requests/second
- 25+ PB stored
- 2-second query response for 1 billion rows
- 144-node ClickHouse cluster

**Spotify's Event Analytics (ClickHouse):**
- 500 billion events/day
- 99th percentile queries <1 second
- Multi-region deployment (US, EU)
- 90-day retention with hourly aggregates

**Uber's Marketplace Analytics (ClickHouse):**
- 100+ TB new data daily
- 10,000+ queries per second
- Real-time driver metrics
- 3-tier storage (SSD/HDD/S3)

---


## 7. DEEP DIVE: AGGREGATIONS & METRICS

Aggregations transform raw events into actionable insights. This section covers real-time aggregation techniques, percentile calculations, and funnel analysis implementation.

### 🟢 Beginner Level: Understanding Aggregations

**What Are Aggregations?**

Think of aggregations like summarizing a long report into key bullet points:

```
Raw Events (Details):
├─ User123 clicked at 10:00:01
├─ User456 clicked at 10:00:03
├─ User789 clicked at 10:00:07
├─ User123 clicked at 10:00:12
└─ ... millions more ...

Aggregation (Summary):
└─ 10:00-10:01: 1,543 clicks from 892 unique users
```

**Common Aggregation Types:**

```
1. COUNT
   ├─ Question: "How many events happened?"
   ├─ Example: 1,543 page views
   └─ Use: Traffic volume, error counts

2. SUM
   ├─ Question: "What's the total value?"
   ├─ Example: $12,450 in sales
   └─ Use: Revenue, data transferred

3. AVG (Average)
   ├─ Question: "What's the typical value?"
   ├─ Example: 234ms average response time
   └─ Use: Performance metrics, ratings

4. MIN/MAX
   ├─ Question: "What are the extremes?"
   ├─ Example: Min: 45ms, Max: 3,200ms
   └─ Use: Performance bounds, price ranges

5. UNIQUE COUNT
   ├─ Question: "How many distinct items?"
   ├─ Example: 892 unique users
   └─ Use: Unique visitors, unique products viewed
```

**Real-World Analogy**: Restaurant Analytics

```
Raw Data (Each Order):
├─ Order #1: $25.00, Table 5, 12:05 PM
├─ Order #2: $18.50, Table 3, 12:07 PM
├─ Order #3: $32.00, Table 5, 12:45 PM
└─ ... more orders ...

Dashboard Aggregations:
├─ Total Orders: 145 (COUNT)
├─ Total Revenue: $3,450 (SUM)
├─ Average Order: $23.79 (AVG)
├─ Unique Tables Served: 28 (UNIQUE COUNT)
└─ Busiest Hour: 12:00-1:00 PM (GROUP BY hour)
```

**Simple Aggregation Example:**

```python
# Calculate metrics from event stream
def aggregate_events(events):
    metrics = {
        'count': 0,
        'sum': 0,
        'unique_users': set()
    }
    
    for event in events:
        metrics['count'] += 1
        metrics['sum'] += event.value
        metrics['unique_users'].add(event.user_id)
    
    metrics['avg'] = metrics['sum'] / metrics['count']
    metrics['unique_count'] = len(metrics['unique_users'])
    
    return metrics

# Result:
# {
#   'count': 1543,
#   'sum': 45230.50,
#   'avg': 29.32,
#   'unique_count': 892
# }
```

---

### 🟡 Intermediate Level: Advanced Aggregation Techniques

**Percentiles: Better Than Averages**

```
Why Percentiles Matter:

Response Times: [10ms, 12ms, 11ms, 15ms, 14ms, 13ms, 2000ms]
Average: 296ms ❌ Misleading! One slow request skews it
Median (P50): 13ms ✅ Typical user experience
P95: 15ms ✅ 95% of users have better response
P99: 2000ms ✅ Worst-case for 1% of users

Rule of Thumb:
├─ P50 (Median): Typical experience
├─ P95: Most users' worst experience
├─ P99: Edge cases, but still important
└─ P99.9: Extreme cases, capacity planning
```

**T-Digest Algorithm for Percentiles:**

```
Problem: Exact percentiles require sorting ALL data
Solution: T-Digest approximates with small memory

How T-Digest Works:
1. Compress data into ~100 "centroids"
2. Store: (centroid_value, weight)
3. Merge centroids while maintaining accuracy
4. Trade-off: 99%+ accuracy with 1KB memory vs 100MB for exact

Example:
1 million values → T-Digest (1 KB) → P99 estimate
Error: <0.1% (accurate enough for dashboards)

Memory Comparison:
├─ Exact: Store all values (8 MB for 1M doubles)
├─ T-Digest: Store ~100 centroids (1 KB)
└─ Savings: 8,000x compression!
```

**T-Digest Implementation:**

```python
from tdigest import TDigest

class PercentileAggregator:
    def __init__(self):
        self.digest = TDigest()
    
    def add(self, value):
        self.digest.update(value)
    
    def get_percentiles(self):
        return {
            'p50': self.digest.percentile(50),
            'p90': self.digest.percentile(90),
            'p95': self.digest.percentile(95),
            'p99': self.digest.percentile(99)
        }
    
    def merge(self, other_digest):
        """Merge from multiple servers"""
        self.digest += other_digest

# Usage in stream processing
aggregator = PercentileAggregator()
for event in events:
    aggregator.add(event.response_time_ms)

percentiles = aggregator.get_percentiles()
# {'p50': 125, 'p90': 234, 'p95': 345, 'p99': 892}
```

**HyperLogLog for Unique Counts:**

```
Problem: Counting unique users requires storing ALL user IDs
Solution: HyperLogLog estimates with ~2% error using 1-2 KB

How HyperLogLog Works:
1. Hash user_id → 64-bit number
2. Count leading zeros in hash
3. Use ~2048 "registers" for different hash prefixes
4. Estimate cardinality from register values

Example:
1 billion unique users → HyperLogLog (1.5 KB) → Count estimate
Error: ~2% (close enough for dashboards!)

Memory Comparison:
├─ Exact: Store all IDs (16 GB for 1B UUIDs)
├─ HyperLogLog: Fixed size (1.5 KB)
└─ Savings: 10,000,000x compression!
```

**Funnel Analysis Implementation:**

```
E-commerce Funnel:
Step 1: Product View    (1,000,000 users) 100%
Step 2: Add to Cart     (150,000 users)   15%  ← 85% drop
Step 3: Checkout        (75,000 users)    7.5% ← 50% drop
Step 4: Purchase        (60,000 users)    6%   ← 20% drop

Conversion Rate: 6% (60K / 1M)
```

**Funnel Query in SQL:**

```sql
-- Funnel analysis: View → Cart → Purchase
WITH funnel_steps AS (
    SELECT
        user_id,
        countIf(event_type = 'product_view') AS has_view,
        countIf(event_type = 'add_to_cart') AS has_cart,
        countIf(event_type = 'purchase') AS has_purchase
    FROM events
    WHERE
        tenant_id = 123
        AND date >= today() - 7
    GROUP BY user_id
)
SELECT
    countIf(has_view > 0) AS step1_users,
    countIf(has_view > 0 AND has_cart > 0) AS step2_users,
    countIf(has_view > 0 AND has_cart > 0 AND has_purchase > 0) AS step3_users,
    
    step2_users / step1_users AS step1_to_step2_rate,
    step3_users / step2_users AS step2_to_step3_rate,
    step3_users / step1_users AS overall_conversion_rate
FROM funnel_steps;
```

**Pre-aggregation Strategies:**

```
Strategy 1: Real-time Aggregation
├─ Aggregate in stream processor (Flink)
├─ Write aggregated results to database
├─ Pro: Query time always fast
└─ Con: Must pre-define all dimensions

Strategy 2: Query-time Aggregation
├─ Store raw events
├─ Aggregate during query
├─ Pro: Flexible, any dimension
└─ Con: Slower for large datasets

Strategy 3: Hybrid (Best)
├─ Pre-aggregate common queries
├─ Store raw events for ad-hoc queries
├─ Use materialized views
└─ Balance: Speed vs flexibility
```

**Materialized Views in ClickHouse:**

```sql
-- Create materialized view for hourly metrics
CREATE MATERIALIZED VIEW hourly_metrics_mv
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMMDD(hour)
ORDER BY (tenant_id, hour, event_type)
AS SELECT
    tenant_id,
    toStartOfHour(event_time) AS hour,
    event_type,
    country,
    
    -- Pre-aggregated metrics
    count() AS event_count,
    uniqExact(user_id) AS unique_users,
    sum(value) AS total_value,
    avg(duration_ms) AS avg_duration
FROM events
GROUP BY
    tenant_id,
    hour,
    event_type,
    country;

-- Query is instant (reads pre-aggregated data)
SELECT
    hour,
    sum(event_count) AS total_events,
    sum(unique_users) AS total_users
FROM hourly_metrics_mv
WHERE
    tenant_id = 123
    AND hour >= now() - INTERVAL 24 HOUR
GROUP BY hour
ORDER BY hour;
```

**Rollup Aggregations:**

```
Multi-level Aggregation Pyramid:

Raw Events (7 days retention):
└─ 1 billion events/day
   ↓ Aggregate every 1 minute
1-Minute Aggregates (30 days retention):
└─ 1,440 rows/day (24 hours × 60 minutes)
   ↓ Aggregate every 1 hour
1-Hour Aggregates (90 days retention):
└─ 24 rows/day
   ↓ Aggregate every 1 day
1-Day Aggregates (2 years retention):
└─ 1 row/day

Query Strategy:
├─ Last 1 hour: Use raw events (most detail)
├─ Last 24 hours: Use 1-minute aggregates
├─ Last 7 days: Use 1-hour aggregates
└─ Last 90 days: Use 1-day aggregates

Storage Savings:
Raw: 1B events/day × 7 days = 7B rows
Aggregated: 1,440 + 24 + 1 = 1,465 rows
Reduction: 4,778,000x compression!
```

---

### 🔴 Advanced Level: Production Aggregation Patterns

**Complete Aggregation Pipeline:**

```sql
-- 1-minute aggregation with all metrics
CREATE MATERIALIZED VIEW metrics_1min_mv TO metrics_1min AS
SELECT
    tenant_id,
    toStartOfMinute(event_time) AS minute,
    event_type,
    
    -- Dimensions (group by)
    country,
    device_type,
    page_url,
    
    -- COUNT aggregations
    count() AS event_count,
    
    -- UNIQUE aggregations (HyperLogLog)
    uniq(user_id) AS unique_users,
    uniq(session_id) AS unique_sessions,
    uniqExact(user_id) AS exact_unique_users,  -- For comparison
    
    -- SUM aggregations
    sum(value) AS total_value,
    sum(duration_ms) AS total_duration,
    
    -- MIN/MAX aggregations
    min(value) AS min_value,
    max(value) AS max_value,
    min(duration_ms) AS min_duration,
    max(duration_ms) AS max_duration,
    
    -- AVG aggregations (compute from sum/count)
    avg(value) AS avg_value,
    avg(duration_ms) AS avg_duration,
    
    -- PERCENTILE aggregations (T-Digest)
    quantile(0.50)(duration_ms) AS p50_duration,
    quantile(0.90)(duration_ms) AS p90_duration,
    quantile(0.95)(duration_ms) AS p95_duration,
    quantile(0.99)(duration_ms) AS p99_duration,
    quantile(0.999)(duration_ms) AS p999_duration,
    
    -- Quantile states (for merging across shards)
    quantileState(0.50)(duration_ms) AS p50_state,
    quantileState(0.95)(duration_ms) AS p95_state,
    quantileState(0.99)(duration_ms) AS p99_state,
    
    -- HyperLogLog states (for merging)
    uniqState(user_id) AS unique_users_state,
    
    -- TOP-K aggregations
    topK(10)(page_url) AS top_10_pages,
    
    -- STDDEV aggregation
    stddevPop(duration_ms) AS stddev_duration,
    
    -- First/Last values
    any(user_agent) AS sample_user_agent,
    argMin(value, event_time) AS first_value,
    argMax(value, event_time) AS last_value
    
FROM events
GROUP BY
    tenant_id,
    minute,
    event_type,
    country,
    device_type,
    page_url;
```

**Funnel Analysis with Window Functions:**

```sql
-- Advanced funnel with time-to-convert
WITH user_events AS (
    SELECT
        user_id,
        event_time,
        event_type,
        
        -- Mark each funnel step
        countIf(event_type = 'product_view') OVER (
            PARTITION BY user_id 
            ORDER BY event_time
        ) AS view_count,
        
        countIf(event_type = 'add_to_cart') OVER (
            PARTITION BY user_id 
            ORDER BY event_time
        ) AS cart_count,
        
        countIf(event_type = 'purchase') OVER (
            PARTITION BY user_id 
            ORDER BY event_time
        ) AS purchase_count
    FROM events
    WHERE
        tenant_id = 123
        AND date >= today() - 7
),
funnel_metrics AS (
    SELECT
        user_id,
        min(if(view_count > 0, event_time, NULL)) AS first_view_time,
        min(if(cart_count > 0, event_time, NULL)) AS first_cart_time,
        min(if(purchase_count > 0, event_time, NULL)) AS first_purchase_time
    FROM user_events
    GROUP BY user_id
)
SELECT
    -- Funnel counts
    count() AS total_users,
    countIf(first_view_time IS NOT NULL) AS step1_view,
    countIf(first_cart_time IS NOT NULL) AS step2_cart,
    countIf(first_purchase_time IS NOT NULL) AS step3_purchase,
    
    -- Conversion rates
    step2_cart / step1_view AS view_to_cart_rate,
    step3_purchase / step2_cart AS cart_to_purchase_rate,
    step3_purchase / step1_view AS overall_conversion,
    
    -- Time to convert (median)
    median(dateDiff('second', first_view_time, first_cart_time)) AS median_time_to_cart,
    median(dateDiff('second', first_cart_time, first_purchase_time)) AS median_time_to_purchase,
    median(dateDiff('second', first_view_time, first_purchase_time)) AS median_time_to_convert,
    
    -- Percentiles of conversion time
    quantile(0.90)(dateDiff('second', first_view_time, first_purchase_time)) AS p90_time_to_convert
FROM funnel_metrics;
```

**Retention Cohort Analysis:**

```sql
-- User retention by signup week
WITH user_cohorts AS (
    SELECT
        user_id,
        toStartOfWeek(min(event_time)) AS cohort_week
    FROM events
    WHERE event_type = 'signup'
    GROUP BY user_id
),
user_activity AS (
    SELECT
        uc.user_id,
        uc.cohort_week,
        toStartOfWeek(e.event_time) AS activity_week,
        dateDiff('week', uc.cohort_week, toStartOfWeek(e.event_time)) AS weeks_since_signup
    FROM user_cohorts uc
    INNER JOIN events e ON uc.user_id = e.user_id
    WHERE e.date >= today() - 90
)
SELECT
    cohort_week,
    countDistinct(user_id) AS cohort_size,
    
    -- Retention by week
    countDistinctIf(user_id, weeks_since_signup = 0) / cohort_size AS week0_retention,
    countDistinctIf(user_id, weeks_since_signup = 1) / cohort_size AS week1_retention,
    countDistinctIf(user_id, weeks_since_signup = 2) / cohort_size AS week2_retention,
    countDistinctIf(user_id, weeks_since_signup = 4) / cohort_size AS week4_retention,
    countDistinctIf(user_id, weeks_since_signup = 8) / cohort_size AS week8_retention,
    countDistinctIf(user_id, weeks_since_signup = 12) / cohort_size AS week12_retention
FROM user_activity
GROUP BY cohort_week
ORDER BY cohort_week DESC;
```

**Real-time Anomaly Detection:**

```sql
-- Detect anomalies using z-score
WITH baseline AS (
    SELECT
        event_type,
        avg(event_count) AS avg_count,
        stddevPop(event_count) AS stddev_count
    FROM metrics_1min
    WHERE
        tenant_id = 123
        AND minute >= now() - INTERVAL 1 DAY
        AND minute < now() - INTERVAL 1 HOUR  -- Exclude recent data
    GROUP BY event_type
),
recent_metrics AS (
    SELECT
        minute,
        event_type,
        event_count
    FROM metrics_1min
    WHERE
        tenant_id = 123
        AND minute >= now() - INTERVAL 1 HOUR
)
SELECT
    r.minute,
    r.event_type,
    r.event_count,
    b.avg_count AS expected_count,
    
    -- Z-score: How many standard deviations from mean?
    (r.event_count - b.avg_count) / b.stddev_count AS z_score,
    
    -- Flag anomalies (|z-score| > 3)
    if(abs(z_score) > 3, 'ANOMALY', 'NORMAL') AS status
FROM recent_metrics r
INNER JOIN baseline b ON r.event_type = b.event_type
WHERE abs(z_score) > 3
ORDER BY abs(z_score) DESC;
```

**Approximate Query Acceleration with Sampling:**

```sql
-- Fast approximate query using 10% sample
SELECT
    country,
    count() * 10 AS approx_event_count,  -- Scale up
    uniq(user_id) * 10 AS approx_unique_users,
    avg(duration_ms) AS avg_duration  -- Average doesn't need scaling
FROM events
SAMPLE 0.1  -- Query only 10% of data
WHERE
    tenant_id = 123
    AND date >= today() - 30
GROUP BY country
ORDER BY approx_event_count DESC;

-- Query time: 0.2s vs 2.0s (10x faster)
-- Accuracy: ±5% (good enough for exploration)
```

**Company Examples:**

**Pinterest's Aggregation Pipeline:**
- 20,000+ metrics computed in real-time
- 1-minute granularity for dashboards
- Hourly rollups for historical analysis
- T-Digest for percentiles, HLL for uniques
- 99.5% accuracy with 100x compression

**Datadog's Metrics Aggregation:**
- 2 million metrics/second
- 10-second aggregation windows
- Distributed percentile calculation
- 15-month retention with rollups
- Query response <100ms for 1TB scans

**Amplitude's Funnel Analytics:**
- 30+ step funnels
- Real-time conversion tracking
- Segment by 50+ dimensions
- Time-to-convert percentiles
- Handles 2 billion events/day

---


## 8. API DESIGN

APIs are the gateway to the analytics dashboard. This section covers event ingestion, query APIs, rate limiting, and error handling patterns.

### 🟢 Beginner Level: API Basics

**Two Main API Categories:**

```
1. Event Ingestion API (Write)
   ├─ Purpose: Receive events from applications
   ├─ Volume: High (millions/second)
   ├─ Latency: Must be fast (<50ms)
   └─ Examples: POST /events, POST /batch

2. Dashboard Query API (Read)
   ├─ Purpose: Retrieve metrics for dashboards
   ├─ Volume: Lower (thousands/second)
   ├─ Latency: Sub-second preferred
   └─ Examples: GET /metrics, POST /query
```

**Real-World Analogy**: Post Office

```
Ingestion API = Mail Drop-off:
├─ Accept package (event)
├─ Give receipt (acknowledgment)
├─ Don't wait for sorting/delivery
└─ Fast: 30 seconds per customer

Query API = Package Tracking:
├─ Look up status
├─ Return current information
├─ More complex processing
└─ Slower: 2 minutes per inquiry
```

**Simple Event Ingestion:**

```json
POST /api/v1/events
Content-Type: application/json
X-API-Key: sk_test_abc123

{
  "event_type": "page_view",
  "user_id": "user_789",
  "timestamp": "2024-01-15T10:30:00Z",
  "properties": {
    "page_url": "/products/shoes",
    "referrer": "google.com",
    "device": "mobile"
  }
}

Response 202 Accepted:
{
  "status": "accepted",
  "event_id": "evt_xyz789"
}
```

**Simple Metrics Query:**

```json
GET /api/v1/metrics?
  metric=page_views&
  start=2024-01-15T00:00:00Z&
  end=2024-01-15T23:59:59Z&
  granularity=hour

Response 200 OK:
{
  "metric": "page_views",
  "data": [
    {"timestamp": "2024-01-15T00:00:00Z", "value": 1523},
    {"timestamp": "2024-01-15T01:00:00Z", "value": 1847},
    {"timestamp": "2024-01-15T02:00:00Z", "value": 1392}
  ]
}
```

---

### 🟡 Intermediate Level: Complete API Specifications

**Event Ingestion API:**

**1. Single Event Endpoint:**

```
POST /api/v1/events
Content-Type: application/json
X-API-Key: {api_key}
X-Request-ID: {idempotency_key}

Request Body:
{
  "event_type": "string (required, max 64 chars)",
  "user_id": "string (optional, max 128 chars)",
  "session_id": "string (optional, max 128 chars)",
  "timestamp": "ISO8601 datetime (optional, defaults to now)",
  "properties": {
    "key": "value",
    // Max 50 properties, each value max 1KB
  }
}

Response Codes:
├─ 202 Accepted: Event queued for processing
├─ 400 Bad Request: Invalid event format
├─ 401 Unauthorized: Invalid API key
├─ 429 Too Many Requests: Rate limit exceeded
└─ 503 Service Unavailable: System overloaded

Response Body:
{
  "status": "accepted",
  "event_id": "evt_xyz789",
  "timestamp": "2024-01-15T10:30:00.123Z"
}
```

**2. Batch Event Endpoint:**

```
POST /api/v1/events/batch
Content-Type: application/json
X-API-Key: {api_key}

Request Body:
{
  "events": [
    {
      "event_type": "page_view",
      "user_id": "user_123",
      "timestamp": "2024-01-15T10:30:00Z",
      "properties": {...}
    },
    // Up to 1000 events per batch
  ]
}

Response 202 Accepted:
{
  "status": "accepted",
  "batch_id": "batch_abc123",
  "accepted_count": 998,
  "rejected_count": 2,
  "rejected_events": [
    {
      "index": 5,
      "reason": "invalid_timestamp"
    },
    {
      "index": 23,
      "reason": "missing_event_type"
    }
  ]
}

Rate Limit:
├─ Per API Key: 10,000 events/second
├─ Per Batch: Max 1000 events
├─ Batch Size: Max 1 MB
└─ Header: X-RateLimit-Remaining: 8534
```

**Dashboard Query API:**

**3. Metrics Query Endpoint:**

```
GET /api/v1/metrics
X-API-Key: {api_key}

Query Parameters:
├─ metric: Metric name (required)
│   Examples: page_views, unique_users, avg_duration
├─ start: Start time (required, ISO8601)
├─ end: End time (required, ISO8601)
├─ granularity: Time bucket (optional, default: auto)
│   Values: minute, hour, day, week, month
├─ filters: JSON-encoded filters (optional)
│   Example: {"country": "US", "device": "mobile"}
├─ group_by: Dimension to group by (optional)
│   Examples: country, device_type, event_type
└─ limit: Max results (optional, default: 1000, max: 10000)

Example Request:
GET /api/v1/metrics?
  metric=page_views&
  start=2024-01-15T00:00:00Z&
  end=2024-01-15T23:59:59Z&
  granularity=hour&
  filters={"country":"US"}&
  group_by=device_type

Response 200 OK:
{
  "metric": "page_views",
  "start": "2024-01-15T00:00:00Z",
  "end": "2024-01-15T23:59:59Z",
  "granularity": "hour",
  "filters": {"country": "US"},
  "group_by": "device_type",
  "data": [
    {
      "timestamp": "2024-01-15T00:00:00Z",
      "dimensions": {"device_type": "mobile"},
      "value": 892
    },
    {
      "timestamp": "2024-01-15T00:00:00Z",
      "dimensions": {"device_type": "desktop"},
      "value": 631
    }
  ],
  "query_time_ms": 145,
  "cached": false
}
```

**4. Advanced Query Endpoint (SQL-like):**

```
POST /api/v1/query
Content-Type: application/json
X-API-Key: {api_key}

Request Body:
{
  "query": {
    "metric": "page_views",
    "aggregation": "sum",
    "dimensions": ["country", "device_type"],
    "filters": [
      {
        "field": "event_type",
        "operator": "in",
        "value": ["page_view", "click"]
      },
      {
        "field": "timestamp",
        "operator": ">=",
        "value": "2024-01-15T00:00:00Z"
      }
    ],
    "group_by": ["country"],
    "order_by": [
      {"field": "value", "direction": "desc"}
    ],
    "limit": 10
  }
}

Response 200 OK:
{
  "results": [
    {
      "dimensions": {"country": "US"},
      "metrics": {
        "page_views": 1523456,
        "unique_users": 89234
      }
    },
    {
      "dimensions": {"country": "UK"},
      "metrics": {
        "page_views": 892341,
        "unique_users": 45123
      }
    }
  ],
  "total_rows": 52,
  "query_time_ms": 234,
  "cached": false
}
```

**5. Funnel Query Endpoint:**

```
POST /api/v1/funnels
Content-Type: application/json
X-API-Key: {api_key}

Request Body:
{
  "funnel": {
    "steps": [
      {
        "name": "Product View",
        "event_type": "product_view"
      },
      {
        "name": "Add to Cart",
        "event_type": "add_to_cart",
        "within": "1 hour"
      },
      {
        "name": "Purchase",
        "event_type": "purchase",
        "within": "24 hours"
      }
    ],
    "start": "2024-01-15T00:00:00Z",
    "end": "2024-01-15T23:59:59Z",
    "filters": {
      "country": "US"
    }
  }
}

Response 200 OK:
{
  "funnel": {
    "steps": [
      {
        "name": "Product View",
        "user_count": 100000,
        "percentage": 100.0,
        "drop_off": 0
      },
      {
        "name": "Add to Cart",
        "user_count": 15000,
        "percentage": 15.0,
        "drop_off": 85000,
        "median_time_from_previous": "5m 23s"
      },
      {
        "name": "Purchase",
        "user_count": 6000,
        "percentage": 6.0,
        "drop_off": 9000,
        "median_time_from_previous": "12m 45s"
      }
    ],
    "overall_conversion": 6.0,
    "query_time_ms": 567
  }
}
```

**Rate Limiting Design:**

```
Rate Limit Strategy:
├─ Token Bucket Algorithm
├─ Per API Key
├─ Different limits for different endpoints
└─ Graceful degradation

Rate Limits:
1. Event Ingestion:
   ├─ Single: 1,000 events/second
   ├─ Batch: 10,000 events/second
   └─ Burst: 2x for 10 seconds

2. Query API:
   ├─ Simple queries: 100/second
   ├─ Complex queries: 10/second
   └─ Export queries: 1/second

Rate Limit Headers:
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 856
X-RateLimit-Reset: 1705318200
Retry-After: 30
```

**Error Handling:**

```json
Error Response Format:
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: event_type",
    "details": {
      "field": "event_type",
      "expected": "string",
      "received": null
    },
    "request_id": "req_xyz789",
    "documentation_url": "https://docs.analytics.com/errors/invalid-request"
  }
}

Error Codes:
├─ INVALID_REQUEST (400)
├─ UNAUTHORIZED (401)
├─ FORBIDDEN (403)
├─ NOT_FOUND (404)
├─ RATE_LIMIT_EXCEEDED (429)
├─ INTERNAL_ERROR (500)
└─ SERVICE_UNAVAILABLE (503)
```

---

### 🔴 Advanced Level: Production API Patterns

**API Authentication & Authorization:**

```python
# Multi-tier API key structure
class APIKey:
    """
    Format: sk_{environment}_{random}_{checksum}
    Example: sk_prod_abc123xyz789_ch3k
    """
    
    def __init__(self, key_string):
        self.environment = self.parse_env(key_string)
        self.tenant_id = self.lookup_tenant(key_string)
        self.permissions = self.get_permissions(key_string)
    
    def has_permission(self, action):
        """Check if key has permission for action"""
        return action in self.permissions

# Permission levels
PERMISSIONS = {
    'read_only': ['metrics:read', 'query:execute'],
    'write_only': ['events:write'],
    'full_access': ['events:write', 'metrics:read', 'query:execute'],
    'admin': ['*']
}

# Rate limit by permission level
RATE_LIMITS = {
    'read_only': {'queries': 100},
    'write_only': {'events': 10000},
    'full_access': {'events': 50000, 'queries': 500},
    'admin': {'events': 100000, 'queries': 1000}
}
```

**Idempotency for Event Ingestion:**

```python
from hashlib import sha256
import redis

class IdempotencyHandler:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.ttl = 86400  # 24 hours
    
    def check_duplicate(self, request_id, event_data):
        """Check if event was already processed"""
        # Create idempotency key
        key = f"idempotency:{request_id}"
        
        # Check if key exists
        if self.redis.exists(key):
            # Return cached response
            return self.redis.get(key), True
        
        return None, False
    
    def store_result(self, request_id, result):
        """Store result for future duplicate requests"""
        key = f"idempotency:{request_id}"
        self.redis.setex(key, self.ttl, result)

# Usage in API endpoint
@app.post("/events")
async def ingest_event(request: Request, event: Event):
    request_id = request.headers.get("X-Request-ID")
    
    # Check for duplicate
    cached_result, is_duplicate = idempotency.check_duplicate(
        request_id, 
        event
    )
    
    if is_duplicate:
        return JSONResponse(
            status_code=202,
            content=cached_result
        )
    
    # Process event
    result = await process_event(event)
    
    # Store result
    idempotency.store_result(request_id, result)
    
    return JSONResponse(status_code=202, content=result)
```

**Query Result Caching:**

```python
import hashlib
import json

class QueryCache:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.default_ttl = 300  # 5 minutes
    
    def cache_key(self, query):
        """Generate cache key from query parameters"""
        query_str = json.dumps(query, sort_keys=True)
        return f"query_cache:{hashlib.sha256(query_str.encode()).hexdigest()}"
    
    def get(self, query):
        """Get cached query result"""
        key = self.cache_key(query)
        cached = self.redis.get(key)
        
        if cached:
            return json.loads(cached)
        return None
    
    def set(self, query, result, ttl=None):
        """Cache query result"""
        key = self.cache_key(query)
        ttl = ttl or self.default_ttl
        
        self.redis.setex(
            key,
            ttl,
            json.dumps(result)
        )
    
    def invalidate_pattern(self, pattern):
        """Invalidate cache entries matching pattern"""
        # When new data arrives, invalidate affected queries
        keys = self.redis.keys(f"query_cache:*{pattern}*")
        if keys:
            self.redis.delete(*keys)

# Cache TTL strategy
CACHE_TTL = {
    'realtime': 10,      # 10 seconds for live dashboards
    'recent': 60,        # 1 minute for last hour
    'historical': 300,   # 5 minutes for older data
    'aggregated': 3600   # 1 hour for daily/monthly aggregates
}
```

**API Request Validation:**

```python
from pydantic import BaseModel, Field, validator
from datetime import datetime
from typing import Optional, Dict, Any

class Event(BaseModel):
    event_type: str = Field(..., min_length=1, max_length=64)
    user_id: Optional[str] = Field(None, max_length=128)
    session_id: Optional[str] = Field(None, max_length=128)
    timestamp: Optional[datetime] = None
    properties: Optional[Dict[str, Any]] = Field(default_factory=dict)
    
    @validator('event_type')
    def validate_event_type(cls, v):
        # Only alphanumeric and underscores
        if not v.replace('_', '').isalnum():
            raise ValueError('event_type must be alphanumeric')
        return v
    
    @validator('timestamp')
    def validate_timestamp(cls, v):
        if v is None:
            return datetime.utcnow()
        
        # Reject events too far in past or future
        now = datetime.utcnow()
        if v < now - timedelta(days=7):
            raise ValueError('timestamp too old (>7 days)')
        if v > now + timedelta(hours=1):
            raise ValueError('timestamp in future (>1 hour)')
        
        return v
    
    @validator('properties')
    def validate_properties(cls, v):
        if v is None:
            return {}
        
        # Max 50 properties
        if len(v) > 50:
            raise ValueError('Max 50 properties allowed')
        
        # Check each property value size
        for key, value in v.items():
            value_str = json.dumps(value)
            if len(value_str) > 1024:  # 1 KB
                raise ValueError(f'Property {key} exceeds 1KB')
        
        return v

class MetricsQuery(BaseModel):
    metric: str
    start: datetime
    end: datetime
    granularity: Optional[str] = 'auto'
    filters: Optional[Dict[str, Any]] = None
    group_by: Optional[str] = None
    limit: int = Field(default=1000, ge=1, le=10000)
    
    @validator('end')
    def validate_time_range(cls, v, values):
        if 'start' in values and v <= values['start']:
            raise ValueError('end must be after start')
        
        # Max 90-day range for detailed queries
        if 'start' in values:
            days = (v - values['start']).days
            if days > 90:
                raise ValueError('Max 90-day range allowed')
        
        return v
```

**Drill-down API with Filters:**

```python
@app.post("/api/v1/drilldown")
async def drilldown(query: DrilldownQuery, api_key: str):
    """
    Drill down into metrics with multiple dimensions
    """
    
    # Example: Start with country-level data
    # User clicks "US" → drill down to state
    # User clicks "California" → drill down to city
    
    result = await execute_query(f"""
        SELECT
            {query.dimension} AS dimension,
            count() AS event_count,
            uniq(user_id) AS unique_users,
            avg(duration_ms) AS avg_duration
        FROM events
        WHERE
            tenant_id = {api_key.tenant_id}
            AND event_time >= '{query.start}'
            AND event_time < '{query.end}'
            {build_filters(query.filters)}
        GROUP BY dimension
        ORDER BY event_count DESC
        LIMIT {query.limit}
    """)
    
    return {
        "dimension": query.dimension,
        "filters": query.filters,
        "data": result,
        "drill_down_options": get_drill_down_options(query.dimension)
    }

def get_drill_down_options(current_dimension):
    """Return valid drill-down dimensions"""
    DRILL_DOWN_HIERARCHY = {
        'country': ['state', 'city'],
        'state': ['city', 'zip_code'],
        'device_type': ['browser', 'os'],
        'event_type': ['page_url', 'referrer']
    }
    return DRILL_DOWN_HIERARCHY.get(current_dimension, [])
```

**Company Examples:**

**Segment's Event API:**
- 50,000 events/second per customer
- 99.9% uptime SLA
- <50ms p99 ingestion latency
- Automatic batching & retry
- Idempotency with 24-hour window

**Mixpanel's Query API:**
- Sub-second query response for 1B events
- 30-day query result caching
- Automatic query optimization
- GraphQL + REST APIs
- Rate limiting: 100 queries/hour (free), unlimited (paid)

**Amplitude's Analytics API:**
- Batch ingestion: 1,000 events per request
- Real-time query API: <200ms p95
- SQL-like query language
- Funnel API: 30-step funnels
- Export API: Full data export capability

---


## 9. DATABASE DESIGN

Database schema design is critical for query performance and storage efficiency. This section covers complete table schemas, indexing strategies, and partitioning for a production analytics system.

### 🟢 Beginner Level: Schema Fundamentals

**Basic Schema Structure:**

Think of database schema like organizing a filing cabinet:

```
Filing Cabinet (Database):
├─ Drawer 1: Raw Events (every detail)
│  └─ Folder: Today's events
├─ Drawer 2: Hourly Summaries (aggregated)
│  └─ Folder: This month's summaries
└─ Drawer 3: Metadata (reference data)
   └─ Folder: Event types, user info
```

**Three Main Tables:**

```
1. Raw Events Table
   ├─ Stores: Every individual event
   ├─ Retention: 7 days (then archived or deleted)
   ├─ Size: Largest table (billions of rows)
   └─ Use: Detailed analysis, debugging

2. Aggregated Metrics Table
   ├─ Stores: Pre-calculated summaries
   ├─ Retention: 90 days to 2 years
   ├─ Size: Much smaller (millions of rows)
   └─ Use: Fast dashboard queries

3. Metadata Tables
   ├─ Stores: Reference data
   ├─ Examples: Event types, dimensions
   ├─ Size: Very small (thousands of rows)
   └─ Use: Configuration, validation
```

**Simple Events Schema:**

```sql
CREATE TABLE events (
    event_id UUID PRIMARY KEY,
    tenant_id INT NOT NULL,
    user_id VARCHAR(128),
    event_type VARCHAR(64) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    properties JSON
);

-- Index for fast queries
CREATE INDEX idx_events_tenant_time 
ON events(tenant_id, timestamp);
```

---

### 🟡 Intermediate Level: Production Schema Design

**Complete Raw Events Schema:**

```sql
-- Raw events table with optimized data types
CREATE TABLE analytics.events (
    -- Identity
    event_id UUID DEFAULT gen_random_uuid(),
    tenant_id INTEGER NOT NULL,
    
    -- User identification
    user_id VARCHAR(128),
    session_id VARCHAR(128),
    anonymous_id VARCHAR(128),
    
    -- Event details
    event_type VARCHAR(64) NOT NULL,
    event_time TIMESTAMP NOT NULL,
    ingestion_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Context
    page_url TEXT,
    page_title VARCHAR(512),
    referrer TEXT,
    
    -- User agent
    user_agent TEXT,
    device_type VARCHAR(32),
    browser VARCHAR(64),
    os VARCHAR(64),
    
    -- Location
    ip_address INET,
    country VARCHAR(2),    -- ISO country code
    region VARCHAR(64),
    city VARCHAR(128),
    latitude DECIMAL(9,6),
    longitude DECIMAL(9,6),
    
    -- Metrics
    duration_ms INTEGER,
    value DECIMAL(18,4),
    
    -- Flexible properties
    properties JSONB,
    
    -- Metadata
    sdk_version VARCHAR(32),
    
    -- Partitioning key
    date DATE GENERATED ALWAYS AS (event_time::DATE) STORED
)
PARTITION BY RANGE (date);

-- Create partitions for current and future dates
CREATE TABLE events_2024_01_15 PARTITION OF events
    FOR VALUES FROM ('2024-01-15') TO ('2024-01-16');

CREATE TABLE events_2024_01_16 PARTITION OF events
    FOR VALUES FROM ('2024-01-16') TO ('2024-01-17');
-- ... create more partitions ...
```

**Indexes for Query Optimization:**

```sql
-- Primary index: tenant + time (most common query pattern)
CREATE INDEX idx_events_tenant_time 
ON events(tenant_id, event_time DESC);

-- User-based queries
CREATE INDEX idx_events_user 
ON events(tenant_id, user_id, event_time DESC);

-- Session-based queries
CREATE INDEX idx_events_session 
ON events(tenant_id, session_id, event_time DESC);

-- Event type filtering
CREATE INDEX idx_events_type 
ON events(tenant_id, event_type, event_time DESC);

-- Geographic queries
CREATE INDEX idx_events_country 
ON events(tenant_id, country, event_time DESC);

-- JSONB properties (GIN index for flexible querying)
CREATE INDEX idx_events_properties 
ON events USING GIN(properties);

-- Composite index for common filters
CREATE INDEX idx_events_composite 
ON events(tenant_id, event_type, country, event_time DESC);
```

**Aggregated Metrics Schema:**

```sql
-- 1-minute aggregations
CREATE TABLE analytics.metrics_1min (
    tenant_id INTEGER NOT NULL,
    minute TIMESTAMP NOT NULL,
    event_type VARCHAR(64) NOT NULL,
    
    -- Dimensions (what we group by)
    country VARCHAR(2),
    device_type VARCHAR(32),
    page_url TEXT,
    
    -- Aggregated metrics
    event_count BIGINT NOT NULL,
    unique_users INTEGER NOT NULL,
    unique_sessions INTEGER NOT NULL,
    
    -- Value aggregations
    total_value DECIMAL(18,4),
    avg_value DECIMAL(18,4),
    min_value DECIMAL(18,4),
    max_value DECIMAL(18,4),
    
    -- Duration aggregations
    total_duration_ms BIGINT,
    avg_duration_ms INTEGER,
    p50_duration_ms INTEGER,
    p95_duration_ms INTEGER,
    p99_duration_ms INTEGER,
    
    -- Metadata
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (tenant_id, minute, event_type, country, device_type, page_url)
)
PARTITION BY RANGE (minute);

-- Indexes for aggregated metrics
CREATE INDEX idx_metrics_1min_lookup 
ON metrics_1min(tenant_id, minute DESC, event_type);

-- Hourly aggregations (less granular, longer retention)
CREATE TABLE analytics.metrics_1hour (
    tenant_id INTEGER NOT NULL,
    hour TIMESTAMP NOT NULL,
    event_type VARCHAR(64) NOT NULL,
    country VARCHAR(2),
    device_type VARCHAR(32),
    
    event_count BIGINT NOT NULL,
    unique_users INTEGER NOT NULL,
    total_value DECIMAL(18,4),
    avg_duration_ms INTEGER,
    p95_duration_ms INTEGER,
    
    PRIMARY KEY (tenant_id, hour, event_type, country, device_type)
)
PARTITION BY RANGE (hour);

-- Daily aggregations (even less granular, very long retention)
CREATE TABLE analytics.metrics_1day (
    tenant_id INTEGER NOT NULL,
    day DATE NOT NULL,
    event_type VARCHAR(64) NOT NULL,
    
    event_count BIGINT NOT NULL,
    unique_users INTEGER NOT NULL,
    total_value DECIMAL(18,4),
    
    PRIMARY KEY (tenant_id, day, event_type)
);
```

**Metadata Tables:**

```sql
-- Event type definitions
CREATE TABLE analytics.event_types (
    event_type_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    event_type VARCHAR(64) NOT NULL,
    display_name VARCHAR(128),
    description TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(tenant_id, event_type)
);

-- User segments (for filtering)
CREATE TABLE analytics.user_segments (
    segment_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    user_id VARCHAR(128) NOT NULL,
    segment_name VARCHAR(64) NOT NULL,
    segment_value VARCHAR(128),
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_segments_lookup (tenant_id, user_id)
);

-- Dashboard definitions
CREATE TABLE analytics.dashboards (
    dashboard_id SERIAL PRIMARY KEY,
    tenant_id INTEGER NOT NULL,
    name VARCHAR(128) NOT NULL,
    description TEXT,
    config JSONB,
    created_by INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 🔴 Advanced Level: ClickHouse Production Schema

**ClickHouse Events Table (Complete):**

```sql
-- Local table (on each shard)
CREATE TABLE analytics.events_local ON CLUSTER main_cluster
(
    -- Identifiers
    tenant_id UInt32,
    event_id UUID,
    
    -- User identity
    user_id String,
    session_id String,
    anonymous_id String,
    
    -- Event data
    event_type LowCardinality(String),
    event_time DateTime64(3, 'UTC'),
    ingestion_time DateTime64(3, 'UTC') DEFAULT now64(3),
    
    -- Page data
    page_url String,
    page_title String,
    referrer String,
    
    -- User agent parsing
    user_agent String,
    device_type LowCardinality(String),
    device_brand LowCardinality(String),
    browser LowCardinality(String),
    browser_version LowCardinality(String),
    os LowCardinality(String),
    os_version LowCardinality(String),
    
    -- Geographic data
    ip_address IPv4,
    country FixedString(2),
    region String,
    city String,
    latitude Float32,
    longitude Float32,
    timezone LowCardinality(String),
    
    -- Metrics
    duration_ms UInt32,
    value Decimal(18, 4),
    
    -- Custom properties (flexible schema)
    properties String,  -- JSON as string for flexibility
    
    -- Parsed property columns (for common properties)
    prop_campaign_id LowCardinality(String),
    prop_ab_test LowCardinality(String),
    prop_product_id String,
    
    -- SDK metadata
    sdk_name LowCardinality(String),
    sdk_version LowCardinality(String),
    
    -- Computed columns (not stored, computed on read)
    date Date MATERIALIZED toDate(event_time),
    hour UInt8 MATERIALIZED toHour(event_time),
    day_of_week UInt8 MATERIALIZED toDayOfWeek(event_time),
    
    -- Version for deduplication
    _version UInt64 DEFAULT 1
)
ENGINE = ReplicatedReplacingMergeTree(
    '/clickhouse/tables/{shard}/events_local',
    '{replica}',
    _version
)
PARTITION BY (tenant_id, toYYYYMMDD(event_time))
ORDER BY (tenant_id, event_time, event_type, user_id)
PRIMARY KEY (tenant_id, event_time)
SAMPLE BY sipHash64(user_id)
TTL 
    event_time + INTERVAL 7 DAY TO VOLUME 'warm',
    event_time + INTERVAL 90 DAY TO VOLUME 'cold',
    event_time + INTERVAL 365 DAY DELETE
SETTINGS 
    index_granularity = 8192,
    merge_with_ttl_timeout = 3600,
    min_bytes_for_wide_part = 0;

-- Skip indexes for better performance
ALTER TABLE analytics.events_local 
ADD INDEX idx_user_id user_id TYPE bloom_filter(0.01) GRANULARITY 4;

ALTER TABLE analytics.events_local 
ADD INDEX idx_session_id session_id TYPE bloom_filter(0.01) GRANULARITY 4;

ALTER TABLE analytics.events_local 
ADD INDEX idx_event_type event_type TYPE set(1000) GRANULARITY 1;

ALTER TABLE analytics.events_local 
ADD INDEX idx_country country TYPE set(500) GRANULARITY 1;

ALTER TABLE analytics.events_local 
ADD INDEX idx_page_url page_url TYPE tokenbf_v1(10240, 3, 0) GRANULARITY 4;

-- Projection for user-centric queries
ALTER TABLE analytics.events_local 
ADD PROJECTION projection_by_user (
    SELECT 
        tenant_id,
        user_id,
        event_time,
        event_type,
        duration_ms,
        value
    ORDER BY (tenant_id, user_id, event_time)
);

-- Distributed table (query interface)
CREATE TABLE analytics.events ON CLUSTER main_cluster
AS analytics.events_local
ENGINE = Distributed(
    main_cluster,
    analytics,
    events_local,
    sipHash64(user_id)
);
```

**Materialized Views for Real-time Aggregations:**

```sql
-- 1-minute aggregation materialized view
CREATE MATERIALIZED VIEW analytics.metrics_1min_mv 
TO analytics.metrics_1min_local
AS SELECT
    tenant_id,
    toStartOfMinute(event_time) AS minute,
    event_type,
    country,
    device_type,
    
    -- Count aggregations
    count() AS event_count,
    uniq(user_id) AS unique_users,
    uniq(session_id) AS unique_sessions,
    
    -- HyperLogLog states (for merging)
    uniqState(user_id) AS unique_users_state,
    uniqState(session_id) AS unique_sessions_state,
    
    -- Value aggregations
    sum(value) AS total_value,
    avg(value) AS avg_value,
    min(value) AS min_value,
    max(value) AS max_value,
    
    -- Duration aggregations
    sum(duration_ms) AS total_duration_ms,
    avg(duration_ms) AS avg_duration_ms,
    
    -- Percentile states (T-Digest)
    quantileState(0.50)(duration_ms) AS p50_state,
    quantileState(0.90)(duration_ms) AS p90_state,
    quantileState(0.95)(duration_ms) AS p95_state,
    quantileState(0.99)(duration_ms) AS p99_state,
    
    -- Standard deviation
    stddevPop(duration_ms) AS stddev_duration_ms,
    
    -- Top-K aggregations
    topK(10)(page_url) AS top_10_pages
    
FROM analytics.events_local
GROUP BY
    tenant_id,
    minute,
    event_type,
    country,
    device_type;

-- Target table for materialized view
CREATE TABLE analytics.metrics_1min_local ON CLUSTER main_cluster
(
    tenant_id UInt32,
    minute DateTime,
    event_type LowCardinality(String),
    country FixedString(2),
    device_type LowCardinality(String),
    
    event_count UInt64,
    unique_users UInt64,
    unique_sessions UInt64,
    
    -- States for distributed merging
    unique_users_state AggregateFunction(uniq, String),
    unique_sessions_state AggregateFunction(uniq, String),
    
    total_value Decimal(18, 4),
    avg_value Decimal(18, 4),
    min_value Decimal(18, 4),
    max_value Decimal(18, 4),
    
    total_duration_ms UInt64,
    avg_duration_ms UInt32,
    
    p50_state AggregateFunction(quantile(0.50), UInt32),
    p90_state AggregateFunction(quantile(0.90), UInt32),
    p95_state AggregateFunction(quantile(0.95), UInt32),
    p99_state AggregateFunction(quantile(0.99), UInt32),
    
    stddev_duration_ms Float64,
    
    top_10_pages Array(String)
)
ENGINE = ReplicatedSummingMergeTree(
    '/clickhouse/tables/{shard}/metrics_1min_local',
    '{replica}'
)
PARTITION BY toYYYYMM(minute)
ORDER BY (tenant_id, minute, event_type, country, device_type)
TTL minute + INTERVAL 30 DAY DELETE
SETTINGS index_granularity = 8192;

-- Distributed table for queries
CREATE TABLE analytics.metrics_1min ON CLUSTER main_cluster
AS analytics.metrics_1min_local
ENGINE = Distributed(
    main_cluster,
    analytics,
    metrics_1min_local,
    sipHash64(toString(tenant_id))
);
```

**Session Table (Denormalized for Fast Queries):**

```sql
-- Sessions table (updated as events arrive)
CREATE TABLE analytics.sessions_local ON CLUSTER main_cluster
(
    tenant_id UInt32,
    session_id String,
    user_id String,
    
    -- Session timing
    session_start DateTime,
    session_end DateTime,
    session_duration_seconds UInt32,
    
    -- Session attributes
    first_page_url String,
    last_page_url String,
    landing_referrer String,
    country FixedString(2),
    device_type LowCardinality(String),
    
    -- Session metrics
    event_count UInt32,
    page_view_count UInt32,
    total_duration_ms UInt64,
    
    -- Conversion flags
    has_signup UInt8,
    has_purchase UInt8,
    total_purchase_value Decimal(18, 4),
    
    -- Last update
    updated_at DateTime DEFAULT now()
)
ENGINE = ReplicatedReplacingMergeTree(
    '/clickhouse/tables/{shard}/sessions_local',
    '{replica}',
    updated_at
)
PARTITION BY toYYYYMM(session_start)
ORDER BY (tenant_id, session_id)
TTL session_start + INTERVAL 90 DAY DELETE;
```

**User Profile Table (Slowly Changing Dimension):**

```sql
-- User profiles (aggregated user data)
CREATE TABLE analytics.user_profiles_local ON CLUSTER main_cluster
(
    tenant_id UInt32,
    user_id String,
    
    -- First/last seen
    first_seen DateTime,
    last_seen DateTime,
    
    -- Activity metrics
    total_events UInt64,
    total_sessions UInt32,
    total_duration_seconds UInt64,
    
    -- User attributes (latest values)
    country FixedString(2),
    device_type LowCardinality(String),
    
    -- Conversion data
    has_signed_up UInt8,
    has_purchased UInt8,
    total_purchase_value Decimal(18, 4),
    purchase_count UInt32,
    
    -- Engagement scores
    engagement_score Float32,
    
    -- Last update
    updated_at DateTime DEFAULT now()
)
ENGINE = ReplicatedReplacingMergeTree(
    '/clickhouse/tables/{shard}/user_profiles_local',
    '{replica}',
    updated_at
)
ORDER BY (tenant_id, user_id)
SETTINGS index_granularity = 8192;
```

**Partitioning Strategy:**

```sql
-- Automatic partition management
-- Create partitions for next 7 days
SELECT 
    'CREATE TABLE IF NOT EXISTS events_' || 
    formatDateTime(today() + number, '%Y_%m_%d') || 
    ' PARTITION OF events FOR VALUES FROM (\'' ||
    formatDateTime(today() + number, '%Y-%m-%d') || 
    '\') TO (\'' ||
    formatDateTime(today() + number + 1, '%Y-%m-%d') || 
    '\');'
FROM numbers(7);

-- Drop old partitions (>7 days)
ALTER TABLE analytics.events_local 
DROP PARTITION '20240108';

-- Detach partition for archival
ALTER TABLE analytics.events_local 
DETACH PARTITION '20240108';

-- Archive to S3
SELECT * FROM events_20240108
INTO OUTFILE 's3://analytics-archive/events/2024/01/08/data.parquet'
FORMAT Parquet;
```

**Company Examples:**

**Cloudflare's Schema:**
- 25+ PB in ClickHouse
- Partition by day + customer_id
- 90-day hot retention on SSD
- Automatic tiering to S3
- Skip indexes reduce query time by 10x

**Uber's Event Schema:**
- 100+ TB ingested daily
- Multi-level partitioning (tenant + date)
- Materialized views for all dashboards
- Query response: P95 < 1 second
- 10,000+ queries/second

**Segment's Schema Design:**
- Flexible schema (JSONB properties)
- Automatic schema evolution
- 2-year retention
- Partition pruning reduces scans by 95%
- Indexes tuned per customer

---

## 10. DEEP DIVE: DRILL-DOWN & FILTERING

### Dimensional Modeling

🟢 **BEGINNER: Understanding Drill-downs**

**Restaurant Menu Analogy:**

```text
Imagine analyzing restaurant sales:

Level 1 (Overview):
└─ Total sales: $100,000 this month

Level 2 (Drill-down by Category):
├─ Appetizers: $20,000
├─ Main Courses: $50,000
├─ Desserts: $15,000
└─ Beverages: $15,000

Level 3 (Drill-down by Specific Item):
Main Courses ($50,000) →
├─ Pasta: $20,000
├─ Steaks: $18,000
├─ Fish: $8,000
└─ Chicken: $4,000

Level 4 (Drill-down by Time):
Pasta ($20,000) →
├─ Week 1: $4,000
├─ Week 2: $6,000
├─ Week 3: $5,000
└─ Week 4: $5,000
```

This is **drill-down** - starting broad and going deeper into specific details!

**In Our Analytics Dashboard:**

A product manager sees:
1. "10,000 users viewed products today" (Overview)
2. Clicks "Electronics" → "5,000 users viewed electronics" (Category drill-down)
3. Clicks "Phones" → "3,000 users viewed phones" (Sub-category drill-down)
4. Clicks "iPhone 15" → "1,500 users viewed iPhone 15" (Product drill-down)
5. Filters by "USA" → "1,000 US users viewed iPhone 15" (Geographic filter)

**Why Drill-downs Matter:**

```text
Business Questions:
├─ "Which country has most sales?" → Drill-down by country
├─ "Which product category is growing?" → Drill-down by category + time
├─ "Why did conversions drop?" → Drill-down by funnel step + segment
└─ "Are mobile users different?" → Filter by device type
```

**Basic Dimensions:**

```text
Common dimensions for drill-down:
├─ Time: year → month → week → day → hour → minute
├─ Geography: region → country → state → city
├─ Product: category → sub-category → product → variant
├─ User: segment → cohort → individual user
└─ Device: platform → OS → browser → version
```

---

🟡 **INTERMEDIATE: Star Schema Design**

**Dimensional Modeling Fundamentals:**

Analytics databases use **star schema** - a fact table (events) surrounded by dimension tables (attributes).

**Star Schema Structure:**

```text
           ┌──────────────┐
           │ Dimension:   │
           │   TIME       │
           │ - date_id    │
           │ - day        │
           │ - week       │
           │ - month      │
           └──────┬───────┘
                  │
   ┌──────────────┼──────────────┐
   │              │              │
┌──┴────────┐ ┌──┴─────────┐ ┌──┴─────────┐
│Dimension: │ │   FACT:    │ │ Dimension: │
│  USER     │ │   EVENTS   │ │  PRODUCT   │
│- user_id  │─│- event_id  │─│- prod_id   │
│- segment  │ │- user_id   │ │- category  │
│- country  │ │- prod_id   │ │- brand     │
└───────────┘ │- date_id   │ └────────────┘
              │- metric    │
              └────┬───────┘
                   │
            ┌──────┴────────┐
            │  Dimension:   │
            │   GEOGRAPHY   │
            │ - geo_id      │
            │ - country     │
            │ - region      │
            │ - city        │
            └───────────────┘
```

**Implementation in ClickHouse:**

```sql
-- Fact Table: Events
CREATE TABLE events (
    event_id String,
    event_type String,
    user_id UInt64,
    product_id UInt32,
    date_id UInt32,        -- 20260122 format
    geo_id UInt16,
    timestamp DateTime,
    
    -- Metrics (measures)
    revenue Decimal(10,2),
    duration UInt32,
    
    -- Denormalized dimensions (for performance)
    country String,
    product_category String,
    user_segment String
    
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (date_id, user_id, timestamp);

-- Dimension Table: Products
CREATE TABLE dim_products (
    product_id UInt32,
    product_name String,
    category String,
    sub_category String,
    brand String,
    price Decimal(10,2)
) ENGINE = ReplacingMergeTree()
ORDER BY product_id;

-- Dimension Table: Users
CREATE TABLE dim_users (
    user_id UInt64,
    user_segment String,
    signup_date Date,
    country String,
    subscription_tier String
) ENGINE = ReplacingMergeTree()
ORDER BY user_id;
```

**Drill-down Query Pattern:**

```sql
-- Level 1: Overview (all products)
SELECT 
    COUNT(*) as total_views,
    COUNT(DISTINCT user_id) as unique_users,
    SUM(revenue) as total_revenue
FROM events
WHERE event_type = 'product_viewed'
  AND date_id = 20260122;

-- Result: 10,000 views, 5,000 users, $50,000 revenue

-- Level 2: Drill-down by Category
SELECT 
    product_category,
    COUNT(*) as views,
    COUNT(DISTINCT user_id) as users,
    SUM(revenue) as revenue
FROM events
WHERE event_type = 'product_viewed'
  AND date_id = 20260122
GROUP BY product_category
ORDER BY views DESC;

-- Result:
-- Electronics: 5,000 views, 2,500 users, $30,000
-- Clothing: 3,000 views, 1,800 users, $15,000
-- Home: 2,000 views, 1,000 users, $5,000

-- Level 3: Drill-down Electronics by Sub-category
SELECT 
    p.sub_category,
    COUNT(*) as views,
    AVG(p.price) as avg_price
FROM events e
JOIN dim_products p ON e.product_id = p.product_id
WHERE e.event_type = 'product_viewed'
  AND e.date_id = 20260122
  AND p.category = 'Electronics'
GROUP BY p.sub_category
ORDER BY views DESC;

-- Result:
-- Phones: 3,000 views, $800 avg price
-- Laptops: 1,500 views, $1,200 avg price
-- Headphones: 500 views, $150 avg price
```

**Multi-dimensional Drill-down:**

```sql
-- Drill-down by Category AND Country
SELECT 
    product_category,
    country,
    COUNT(*) as views,
    COUNT(DISTINCT user_id) as users
FROM events
WHERE event_type = 'product_viewed'
  AND date_id = 20260122
GROUP BY product_category, country
ORDER BY views DESC
LIMIT 10;

-- Result shows top combinations:
-- Electronics, US: 2,000 views
-- Electronics, UK: 1,500 views
-- Clothing, US: 1,200 views
-- Electronics, DE: 800 views
-- ...
```

**ROLLUP and CUBE for Subtotals:**

```sql
-- Get totals at multiple levels automatically
SELECT 
    product_category,
    country,
    COUNT(*) as views
FROM events
WHERE event_type = 'product_viewed'
  AND date_id = 20260122
GROUP BY ROLLUP(product_category, country)
ORDER BY product_category, country;

-- Result includes:
-- Electronics, US: 2,000
-- Electronics, UK: 1,500
-- Electronics, NULL: 5,000  (subtotal for Electronics)
-- Clothing, US: 1,200
-- Clothing, NULL: 3,000    (subtotal for Clothing)
-- NULL, NULL: 10,000       (grand total)
```

---

🔴 **ADVANCED: Advanced Dimensional Techniques**

**1. Slowly Changing Dimensions (SCD)**

User segments change over time. How do we track this?

**SCD Type 2 (Historical Tracking):**

```sql
-- Track user segment history
CREATE TABLE dim_users_scd (
    user_id UInt64,
    user_segment String,
    valid_from Date,
    valid_to Date,
    is_current UInt8,
    version UInt16
) ENGINE = ReplacingMergeTree(version)
ORDER BY (user_id, valid_from);

-- Insert when user upgrades from Free to Premium
INSERT INTO dim_users_scd VALUES
(12345, 'Free', '2025-01-01', '2026-01-15', 0, 1),
(12345, 'Premium', '2026-01-15', '9999-12-31', 1, 2);

-- Query: What segment was user in when event occurred?
SELECT 
    e.user_id,
    e.event_type,
    e.timestamp,
    u.user_segment
FROM events e
JOIN dim_users_scd u 
  ON e.user_id = u.user_id
  AND toDate(e.timestamp) BETWEEN u.valid_from AND u.valid_to
WHERE e.date_id = 20260122;
```

**2. Hierarchical Dimensions:**

```sql
-- Product category hierarchy
CREATE TABLE dim_product_hierarchy (
    product_id UInt32,
    product_name String,
    l1_category String,      -- Electronics
    l2_category String,      -- Phones
    l3_category String,      -- Smartphones
    l4_category String       -- iPhone 15
) ENGINE = ReplacingMergeTree()
ORDER BY product_id;

-- Drill-down query that works at any level
SELECT 
    multiIf(
        {level} = 1, l1_category,
        {level} = 2, l2_category,
        {level} = 3, l3_category,
        l4_category
    ) as category,
    COUNT(*) as views
FROM events e
JOIN dim_product_hierarchy p ON e.product_id = p.product_id
WHERE date_id = 20260122
  AND l1_category = 'Electronics'  -- Filter from parent level
GROUP BY category
ORDER BY views DESC;
```

**3. Ragged Hierarchies:**

Geographic hierarchies aren't always uniform:

```text
Country → State → City → Zip
  └─ USA → California → San Francisco → 94102
  └─ USA → California → Los Angeles → 90001
  └─ UK → (no state) → London → SW1A 1AA
```

**Solution: Bridge Table:**

```sql
CREATE TABLE dim_geography_bridge (
    geo_id UInt32,
    country String,
    state String,           -- Can be empty
    city String,
    zip_code String,        -- Can be empty
    hierarchy_level UInt8   -- 1=country, 2=state, 3=city, 4=zip
) ENGINE = ReplacingMergeTree()
ORDER BY geo_id;
```

**4. Degenerate Dimensions:**

Sometimes dimension data lives in fact table (no separate dimension table needed):

```sql
-- Order number is a dimension but has no attributes
CREATE TABLE order_events (
    order_id String,        -- Degenerate dimension
    product_id UInt32,
    user_id UInt64,
    order_total Decimal(10,2),
    timestamp DateTime
) ENGINE = MergeTree()
ORDER BY (timestamp, order_id);

-- Can still group by order
SELECT order_id, SUM(order_total)
FROM order_events
GROUP BY order_id;
```

**5. Junk Dimensions:**

Group low-cardinality flags together:

```sql
-- Instead of many boolean columns in fact table
CREATE TABLE dim_event_flags (
    flag_id UInt16,
    is_mobile UInt8,
    is_logged_in UInt8,
    is_first_visit UInt8,
    has_discount UInt8
) ENGINE = ReplacingMergeTree()
ORDER BY flag_id;

-- Pre-populate all 16 combinations (2^4)
-- flag_id = 0: all false
-- flag_id = 15: all true
-- flag_id = 8: only is_mobile true
-- etc.

-- Fact table just stores flag_id
CREATE TABLE events (
    event_id String,
    user_id UInt64,
    flag_id UInt16,  -- Reference to junk dimension
    ...
);
```

**6. Real Company Example: Mixpanel's Approach**

Mixpanel handles drill-downs with:

```text
Event Properties (Dimensions):
├─ Pre-defined: automatically tracked
│  ├─ $os (operating system)
│  ├─ $browser
│  ├─ $city
│  └─ $device
├─ Custom: defined by customer
│  ├─ plan_type: "free", "pro", "enterprise"
│  ├─ experiment_group: "A", "B", "control"
│  └─ product_category: "electronics", "clothing"
└─ User properties: attached to user profile
   ├─ user_segment
   ├─ signup_date
   └─ total_spend

Query Model:
└─ All properties available for filtering and grouping
   └─ No need to pre-define dimension tables!
   └─ Properties stored as JSON maps
   └─ Indexed for fast filtering
```

---

### Query Optimization

🟢 **BEGINNER: Making Queries Fast**

**Why Query Speed Matters:**

```text
User Experience:
├─ <1 second: Instant, users stay engaged
├─ 1-3 seconds: Acceptable, but users notice
├─ 3-5 seconds: Slow, users get impatient
└─ >5 seconds: Users abandon dashboard
```

**Three Ways to Make Queries Fast:**

**1. Index the Right Columns**

```text
Phone Book Analogy:
├─ WITHOUT INDEX: Read every page to find "Smith"
└─ WITH INDEX: Jump directly to "S" section

Database:
├─ WITHOUT INDEX: Scan all 1 billion events
└─ WITH INDEX: Read only relevant partitions
```

**2. Pre-calculate Common Queries**

```text
Restaurant Analogy:
├─ WITHOUT PRE-CALC: Count orders every time manager asks
└─ WITH PRE-CALC: Keep running total, update as orders come

Database:
├─ WITHOUT PRE-CALC: SUM(revenue) over 1B rows every query
└─ WITH PRE-CALC: Read pre-computed hourly totals (only 24 rows)
```

**3. Cache Recent Results**

```text
Restaurant Analogy:
├─ WITHOUT CACHE: Cook same burger 10 times for 10 customers
└─ WITH CACHE: Cook once, serve quickly to all 10

Database:
├─ WITHOUT CACHE: Run same query 100 times for 100 dashboard loads
└─ WITH CACHE: Run once, serve cached result to all 100 loads
```

---

🟡 **INTERMEDIATE: Optimization Techniques**

**1. Partition Pruning:**

ClickHouse partitions data by time. Filters on time skip entire partitions:

```sql
-- BAD: Scans all 12 months of data
SELECT COUNT(*) FROM events
WHERE country = 'US';
-- Reads: 3.6 TB (full year)

-- GOOD: Scans only 1 day
SELECT COUNT(*) FROM events
WHERE timestamp >= '2026-01-22'
  AND timestamp < '2026-01-23'
  AND country = 'US';
-- Reads: 10 GB (1 day) - 360x faster!
```

**Performance Impact:**

```text
Partition pruning:
├─ Without time filter: 12 partitions scanned = 3.6 TB
├─ With 1-day filter: 1 partition scanned = 10 GB
└─ Speedup: 360x faster, 99.7% less data scanned
```

**2. Columnar Storage Benefits:**

ClickHouse stores columns separately. Only read columns you need:

```sql
-- BAD: Reads all 20 columns
SELECT * FROM events
WHERE date_id = 20260122;
-- Reads: 10 GB (all columns)

-- GOOD: Reads only 3 columns
SELECT event_type, user_id, timestamp
FROM events
WHERE date_id = 20260122;
-- Reads: 2 GB (3 columns) - 5x faster!
```

**3. Skip Indexes:**

```sql
-- Create bloom filter index on country
ALTER TABLE events 
ADD INDEX country_idx country 
TYPE bloom_filter GRANULARITY 4;

-- Now filtering by country is much faster
SELECT COUNT(*) FROM events
WHERE date_id = 20260122
  AND country = 'US';
-- Skip index eliminates 95% of data blocks before reading
```

**4. Materialized Views (Pre-aggregation):**

```sql
-- Create materialized view for hourly metrics
CREATE MATERIALIZED VIEW events_hourly
ENGINE = SummingMergeTree()
ORDER BY (date, hour, country, product_category)
AS SELECT
    toDate(timestamp) as date,
    toHour(timestamp) as hour,
    country,
    product_category,
    COUNT(*) as event_count,
    COUNT(DISTINCT user_id) as unique_users,
    SUM(revenue) as total_revenue
FROM events
GROUP BY date, hour, country, product_category;

-- Query the pre-aggregated view (1000x faster)
SELECT 
    country,
    SUM(total_revenue) as revenue
FROM events_hourly
WHERE date = '2026-01-22'
GROUP BY country;
-- Reads: 10 MB from aggregated view
-- vs 10 GB from raw events
```

**5. Query Rewriting:**

The query optimizer can rewrite queries to use materialized views automatically:

```sql
-- User writes:
SELECT country, SUM(revenue)
FROM events
WHERE timestamp >= '2026-01-22' AND timestamp < '2026-01-23'
GROUP BY country;

-- Optimizer rewrites to:
SELECT country, SUM(total_revenue)
FROM events_hourly
WHERE date = '2026-01-22'
GROUP BY country;
-- Automatically uses faster materialized view!
```

**6. Approximate Algorithms:**

For very large datasets, approximate is acceptable:

```sql
-- Exact count (slow for 1B rows)
SELECT COUNT(DISTINCT user_id) FROM events;
-- Takes: 30 seconds

-- Approximate count with HyperLogLog (0.01% error)
SELECT uniqHLL12(user_id) FROM events;
-- Takes: 3 seconds - 10x faster!
-- Error: ±0.01% (99.99% accurate)
```

**Performance Comparison:**

| Query Type | Method | Response Time | Accuracy |
|------------|--------|---------------|----------|
| **Daily totals** | Raw scan | 10 seconds | 100% |
| **Daily totals** | Materialized view | 100ms | 100% |
| **Unique users** | Exact COUNT DISTINCT | 30 seconds | 100% |
| **Unique users** | HyperLogLog | 3 seconds | 99.99% |
| **Percentiles** | Exact | 45 seconds | 100% |
| **Percentiles** | T-Digest | 5 seconds | 99.9% |

---

🔴 **ADVANCED: Query Performance Engineering**

**1. Query Planner Optimization:**

Understand how ClickHouse executes queries:

```sql
-- Use EXPLAIN to see query plan
EXPLAIN 
SELECT country, COUNT(*) 
FROM events 
WHERE date_id = 20260122 
  AND product_category = 'Electronics'
GROUP BY country;

-- Output shows:
-- 1. Partition pruning: 1/365 partitions
-- 2. Primary key filter: date_id
-- 3. Bloom filter: product_category
-- 4. Column reads: country, product_category
-- 5. Aggregation: GROUP BY country
-- 6. Estimated rows: 13,698 (0.14% of total)
```

**2. Distributed Query Execution:**

For sharded ClickHouse clusters:

```sql
-- Query sent to all shards in parallel
SELECT country, SUM(revenue)
FROM events_distributed  -- Distributed table
WHERE date_id = 20260122
GROUP BY country;

-- Execution:
-- 1. Coordinator sends query to 4 shards
-- 2. Each shard processes 1/4 of data in parallel
-- 3. Partial aggregates returned to coordinator
-- 4. Coordinator merges results
-- 5. Total time: ~same as 1 shard (parallelized)
```

**Performance Math:**

```text
Single shard:
└─ 10 GB data / 1 GB/sec = 10 seconds

4 shards (parallel):
└─ (10 GB / 4) / 1 GB/sec = 2.5 seconds
   + 0.5 sec merge = 3 seconds total
   
Speedup: 3.3x with 4 shards
```

**3. Query Result Sampling:**

For exploratory queries, sample data:

```sql
-- Sample 10% of data for fast results
SELECT country, COUNT(*) * 10 as estimated_count
FROM events SAMPLE 0.1  -- Sample 10%
WHERE date_id = 20260122
GROUP BY country;

-- Performance:
-- Full scan: 10 GB, 10 seconds
-- Sampled: 1 GB, 1 second (10x faster)
-- Accuracy: ±5% error (acceptable for exploration)
```

**4. Query Prioritization:**

Assign priorities to queries:

```sql
-- High priority (dashboard load)
SELECT ... FROM events
SETTINGS priority = 1;  -- Highest priority

-- Low priority (background export)
SELECT ... FROM events
SETTINGS priority = 10;  -- Lowest priority
```

**5. Resource Limits:**

Prevent runaway queries:

```sql
-- Set query timeout
SET max_execution_time = 30;  -- Kill after 30 seconds

-- Set memory limit
SET max_memory_usage = 10000000000;  -- 10 GB max

-- Set rows limit
SET max_rows_to_read = 1000000000;  -- 1B rows max
```

**6. Real Company Example: Uber's Query Optimization**

Uber's analytics platform (built on ClickHouse) handles 10T events/day:

```text
Optimization Strategy:
├─ Partition by date (1 partition = 1 day)
├─ Sub-partition by city (1000 cities)
├─ Pre-aggregate at multiple levels:
│  ├─ 1-minute rollups: 50 TB/day
│  ├─ 1-hour rollups: 2 TB/day
│  └─ 1-day rollups: 50 GB/day
├─ Bloom filter indexes on: city, trip_type, vehicle_type
├─ Cache top 100 queries (80% hit rate)
└─ Result: p95 query latency = 400ms

Cost Optimization:
├─ Without pre-aggregation: $2M/month
├─ With pre-aggregation: $400K/month
└─ Savings: $1.6M/month (80% cost reduction)
```

---

### Caching Strategy

🟢 **BEGINNER: What is Caching?**

**Restaurant Analogy:**

```text
WITHOUT CACHE:
Customer orders burger → Chef cooks burger (5 min) → Serve
Customer orders burger → Chef cooks burger (5 min) → Serve
Customer orders burger → Chef cooks burger (5 min) → Serve

WITH CACHE:
Customer orders burger → Chef cooks burger (5 min) → Serve + Save 1 extra
Customer orders burger → Serve from saved burger (30 sec)
Customer orders burger → Serve from saved burger (30 sec)

Result: 2 customers served 10x faster!
```

**In Our Dashboard:**

```text
WITHOUT CACHE:
User loads dashboard → Query database (5 sec) → Show results
User loads dashboard → Query database (5 sec) → Show results

WITH CACHE:
User loads dashboard → Query database (5 sec) → Show results + Cache
User loads dashboard → Serve from cache (100 ms) → Show results

Result: 50x faster for cached queries!
```

**What to Cache:**

```text
Good candidates:
├─ Popular dashboards (CEO dashboard loaded 100x/day)
├─ Slow queries (queries taking >3 seconds)
├─ Recent time ranges (today, last 7 days)
└─ Aggregated metrics (totals, averages)

Bad candidates:
├─ Real-time queries (need fresh data)
├─ User-specific data (different for each user)
├─ Rarely accessed reports (not worth caching)
└─ Very large result sets (too big to cache)
```

---

🟡 **INTERMEDIATE: Multi-Layer Caching**

**Caching Architecture:**

```text
┌─────────────┐
│  Dashboard  │ (User's browser)
│   Cache     │ Cache Time: 30 seconds
└──────┬──────┘
       │
┌──────┴──────┐
│   CDN       │ (Edge locations)
│   Cache     │ Cache Time: 2 minutes
└──────┬──────┘
       │
┌──────┴──────┐
│ Application │
│   Cache     │ Redis: 10 minutes
│  (Redis)    │
└──────┬──────┘
       │
┌──────┴──────┐
│ ClickHouse  │
│   Cache     │ Query result cache: 1 hour
│             │
└─────────────┘
```

**Layer 1: Browser Cache**

```javascript
// Cache dashboard data in browser
function loadDashboard() {
    const cacheKey = 'dashboard_metrics_today';
    const cached = localStorage.getItem(cacheKey);
    const cacheTime = localStorage.getItem(cacheKey + '_time');
    
    // Check if cache is fresh (< 30 seconds old)
    if (cached && (Date.now() - cacheTime) < 30000) {
        displayMetrics(JSON.parse(cached));
        return;
    }
    
    // Cache miss or stale - fetch from server
    fetch('/api/metrics?date=today')
        .then(resp => resp.json())
        .then(data => {
            localStorage.setItem(cacheKey, JSON.stringify(data));
            localStorage.setItem(cacheKey + '_time', Date.now());
            displayMetrics(data);
        });
}
```

**Layer 2: CDN Cache**

```text
Cache-Control headers:
├─ Static assets: Cache for 1 year
│  Cache-Control: public, max-age=31536000, immutable
│
├─ Dashboard data: Cache for 2 minutes
│  Cache-Control: public, max-age=120, s-maxage=120
│
└─ Real-time data: No cache
   Cache-Control: no-cache, no-store, must-revalidate
```

**Layer 3: Application Cache (Redis)**

```python
import redis
import json

redis_client = redis.Redis(host='localhost', port=6379)

def get_metrics(date, country):
    # Create cache key
    cache_key = f"metrics:{date}:{country}"
    
    # Try cache first
    cached = redis_client.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Cache miss - query database
    query = f"""
        SELECT 
            country,
            SUM(revenue) as total_revenue,
            COUNT(DISTINCT user_id) as unique_users
        FROM events
        WHERE date_id = {date}
          AND country = '{country}'
    """
    result = clickhouse_client.execute(query)
    
    # Store in cache for 10 minutes
    redis_client.setex(
        cache_key,
        600,  # 10 minutes TTL
        json.dumps(result)
    )
    
    return result
```

**Layer 4: ClickHouse Query Cache**

```sql
-- Enable query result cache
SET use_query_cache = 1;
SET query_cache_ttl = 3600;  -- 1 hour

-- Subsequent identical queries served from cache
SELECT country, SUM(revenue)
FROM events
WHERE date_id = 20260122
GROUP BY country;

-- First execution: 5 seconds
-- Cached executions: 10ms
```

**Cache Invalidation Strategy:**

```text
When to invalidate cache:

1. Time-based (TTL):
   ├─ Expire after X minutes
   └─ Good for: Slowly changing data

2. Event-based:
   ├─ Invalidate when new data arrives
   └─ Good for: Real-time updates

3. Manual:
   ├─ User clicks "Refresh" button
   └─ Good for: User-triggered updates

4. Pattern-based:
   ├─ Invalidate all keys matching pattern
   └─ Example: Delete "metrics:20260122:*"
```

**Cache Warming:**

Proactively populate cache before users request:

```python
def warm_cache():
    """Pre-populate cache with popular queries"""
    popular_queries = [
        {'date': 'today', 'country': 'US'},
        {'date': 'today', 'country': 'UK'},
        {'date': 'today', 'metric': 'revenue'},
        {'date': 'last_7_days', 'metric': 'users'},
    ]
    
    for query in popular_queries:
        # Execute query and cache result
        result = execute_query(query)
        cache_result(query, result)
        
    print(f"Warmed {len(popular_queries)} cache entries")

# Run every 5 minutes
schedule.every(5).minutes.do(warm_cache)
```

---

🔴 **ADVANCED: Intelligent Caching**

**1. Adaptive TTL:**

Adjust cache lifetime based on data freshness:

```python
def get_adaptive_ttl(date):
    """Return cache TTL based on how old the data is"""
    days_ago = (datetime.now().date() - date).days
    
    if days_ago == 0:
        # Today: Fresh data, short cache (2 minutes)
        return 120
    elif days_ago <= 7:
        # Last week: Medium cache (10 minutes)
        return 600
    elif days_ago <= 30:
        # Last month: Long cache (1 hour)
        return 3600
    else:
        # Historical: Very long cache (24 hours)
        return 86400

# Usage
ttl = get_adaptive_ttl(query_date)
redis_client.setex(cache_key, ttl, result)
```

**2. Query Fingerprinting:**

Generate consistent cache keys for semantically identical queries:

```python
import hashlib

def normalize_query(sql):
    """Normalize query for caching"""
    # Remove whitespace variations
    sql = ' '.join(sql.split())
    
    # Remove comments
    sql = re.sub(r'--.*$', '', sql, flags=re.MULTILINE)
    sql = re.sub(r'/\*.*?\*/', '', sql, flags=re.DOTALL)
    
    # Lowercase keywords
    sql = sql.lower()
    
    return sql

def get_query_fingerprint(sql, params):
    """Generate cache key from query + params"""
    normalized = normalize_query(sql)
    params_str = json.dumps(params, sort_keys=True)
    fingerprint = hashlib.sha256(
        f"{normalized}:{params_str}".encode()
    ).hexdigest()
    return f"query:{fingerprint}"

# Usage
cache_key = get_query_fingerprint(
    "SELECT country, SUM(revenue) FROM events WHERE date = ? GROUP BY country",
    {"date": "2026-01-22"}
)
```

**3. Probabilistic Early Expiration:**

Prevent cache stampede (many requests hitting DB simultaneously when cache expires):

```python
import random
import time

def get_with_early_expiration(key, fetch_func, ttl=600):
    """Get from cache with probabilistic early recomputation"""
    
    cached_data = redis_client.get(key)
    if not cached_data:
        # Cache miss - fetch and cache
        data = fetch_func()
        redis_client.setex(key, ttl, json.dumps(data))
        return data
    
    # Cache hit - check if we should recompute early
    cache_age = redis_client.ttl(key)
    remaining_ttl = ttl - cache_age
    
    # Probability of early recomputation increases as TTL approaches
    # When TTL=1 sec, probability=99%
    # When TTL=300 sec, probability=1%
    recompute_probability = 1.0 - (remaining_ttl / ttl)
    
    if random.random() < recompute_probability:
        # Recompute in background, return stale data immediately
        threading.Thread(
            target=refresh_cache_async,
            args=(key, fetch_func, ttl)
        ).start()
    
    return json.loads(cached_data)

def refresh_cache_async(key, fetch_func, ttl):
    """Refresh cache in background"""
    data = fetch_func()
    redis_client.setex(key, ttl, json.dumps(data))
```

**4. Cache Compression:**

Save memory by compressing cached results:

```python
import zlib

def cache_compressed(key, data, ttl):
    """Cache data with compression"""
    json_str = json.dumps(data)
    compressed = zlib.compress(json_str.encode())
    
    # Store compressed with metadata
    redis_client.setex(
        f"compressed:{key}",
        ttl,
        compressed
    )
    
    print(f"Compression ratio: {len(json_str) / len(compressed):.1f}x")

def get_compressed(key):
    """Retrieve and decompress"""
    compressed = redis_client.get(f"compressed:{key}")
    if not compressed:
        return None
    
    json_str = zlib.decompress(compressed).decode()
    return json.loads(json_str)

# Typical compression ratios:
# - JSON metrics: 5-10x compression
# - Time-series data: 3-5x compression
# - Result: Cache 5-10x more data in same memory
```

**5. Cache Hierarchy with Fallback:**

```python
class CacheHierarchy:
    def __init__(self):
        self.l1 = {}  # In-memory (1 MB, 1ms latency)
        self.l2 = redis.Redis()  # Redis (100 GB, 5ms latency)
        self.l3 = memcached_client  # Memcached (1 TB, 20ms latency)
    
    def get(self, key):
        # Try L1 (in-memory)
        if key in self.l1:
            return self.l1[key]
        
        # Try L2 (Redis)
        val = self.l2.get(key)
        if val:
            self.l1[key] = val  # Promote to L1
            return val
        
        # Try L3 (Memcached)
        val = self.l3.get(key)
        if val:
            self.l2.setex(key, 600, val)  # Promote to L2
            self.l1[key] = val  # Promote to L1
            return val
        
        # Cache miss at all levels
        return None
    
    def set(self, key, value, ttl):
        # Write to all levels
        self.l1[key] = value
        self.l2.setex(key, ttl, value)
        self.l3.set(key, value, time=ttl)
```

**6. Real Company Example: Netflix's EVCache**

Netflix uses EVCache (built on Memcached) for dashboard metrics:

```text
Architecture:
├─ 3 cache zones (US-East, US-West, EU)
├─ 1000+ cache nodes
├─ 30M requests/second
├─ 10TB cached data
└─ <1ms p99 latency

Optimization techniques:
├─ Chunked caching (large results split into chunks)
├─ Compressed values (5x compression)
├─ Async replication across zones
├─ Client-side routing (consistent hashing)
└─ Circuit breaker for failures

Cache hit rates:
├─ Dashboard metrics: 95%
├─ User profiles: 99%
├─ Real-time data: 60% (shorter TTL)
└─ Overall: 90%

Cost savings:
├─ Without cache: 100K ClickHouse queries/sec = $500K/month
├─ With cache (90% hit rate): 10K queries/sec = $50K/month
└─ Savings: $450K/month ($5.4M/year)
```

---

## 11. SCALABILITY

### Handling 10x Traffic

🟢 **BEGINNER: What is Scalability?**

**Restaurant Chain Analogy:**

```text
Small Restaurant (Current):
├─ Serves 100 customers/day
├─ 1 chef, 2 waiters
├─ Small kitchen, 10 tables
└─ Works fine!

Growing Restaurant (10x Growth):
├─ Now serves 1,000 customers/day
├─ Problem: Same 1 chef, 2 waiters, 10 tables
└─ Result: Long waits, angry customers, food quality drops

Solutions:
1. Vertical Scaling (Bigger restaurant):
   └─ Hire super chef who cooks 10x faster
   └─ Problem: Such chefs don't exist (and cost too much!)

2. Horizontal Scaling (More restaurants):
   └─ Open 10 branches, each serves 100 customers
   └─ Solution: Scales well, proven model!
```

**In Our Analytics System:**

```text
Current Scale:
├─ 10M events/day
├─ 115 events/sec average
├─ 345 events/sec peak
└─ Works with 27 servers

10x Growth (Target):
├─ 100M events/day
├─ 1,157 events/sec average
├─ 3,471 events/sec peak
└─ Need to scale!

Two Approaches:
1. Vertical Scaling (Bigger servers):
   ├─ 8 CPU → 64 CPU servers
   ├─ 16 GB RAM → 256 GB RAM
   └─ Problem: Expensive, has limits

2. Horizontal Scaling (More servers):
   ├─ 8 stream processors → 80 stream processors
   ├─ 12 database nodes → 120 database nodes
   └─ Solution: Cost-effective, unlimited scaling
```

**Scalability Requirements:**

```text
Our system must scale in 3 dimensions:

1. Event Ingestion (Write Scaling):
   ├─ Current: 345 events/sec
   ├─ Target: 3,471 events/sec
   └─ Need: 10x write capacity

2. Data Storage (Storage Scaling):
   ├─ Current: 5 TB (90 days)
   ├─ Target: 50 TB (90 days)
   └─ Need: 10x storage capacity

3. Query Processing (Read Scaling):
   ├─ Current: 100 queries/sec
   ├─ Target: 1,000 queries/sec
   └─ Need: 10x query capacity
```

**Key Principle: Linear Scalability**

```text
Perfect linear scaling:
├─ 1 server: 100 events/sec
├─ 2 servers: 200 events/sec
├─ 10 servers: 1,000 events/sec
└─ 100 servers: 10,000 events/sec

Reality (with overhead):
├─ 1 server: 100 events/sec
├─ 2 servers: 190 events/sec (95% efficiency)
├─ 10 servers: 900 events/sec (90% efficiency)
└─ 100 servers: 8,000 events/sec (80% efficiency)

Goal: Maintain >80% efficiency at scale
```

---

🟡 **INTERMEDIATE: Scaling Strategy**

**Component-by-Component Scaling:**

**1. API Gateway Scaling:**

```text
Current: 3 Nginx instances
├─ Each handles 5,000 req/sec
└─ Total: 15,000 req/sec

Target (10x): Need 30,000 req/sec
├─ Option A: 6 Nginx instances (2x)
├─ Option B: Upgrade to larger instances (3x capacity each)
└─ Chosen: Option A (horizontal scaling, better redundancy)

Implementation:
├─ Add 3 more Nginx instances
├─ Use DNS round-robin or AWS ELB
└─ Auto-scaling group (scale 3-10 instances based on traffic)
```

**2. Event API Servers Scaling:**

```text
Current: 6 servers (c5.xlarge)
├─ Each handles 500 events/sec
└─ Total: 3,000 events/sec

Target: Need 30,000 events/sec
├─ Need: 30,000 / 500 = 60 servers
└─ With 20% buffer: 72 servers

Auto-scaling configuration:
├─ Minimum: 20 servers (off-peak)
├─ Normal: 40 servers (business hours)
├─ Maximum: 100 servers (Black Friday)
└─ Scale trigger: CPU > 70% for 5 minutes

Cost optimization:
├─ Use Spot Instances for stateless API servers
├─ 70% cost savings: $745/month → $224/month
└─ Annual savings: $6,252
```

**3. Kafka Cluster Scaling:**

```text
Current: 3 brokers, 12 partitions
├─ Each broker: 20,000 msgs/sec
└─ Total: 60,000 msgs/sec capacity

Target: Need 35,000 msgs/sec (with 3x buffer)
├─ Current capacity sufficient!
└─ But need more partitions for parallelism

Scaling approach:
├─ Keep 3 brokers (sufficient throughput)
├─ Increase partitions: 12 → 36 partitions
├─ Reason: More parallelism for stream processors
└─ Each partition: 1,000 msgs/sec

Benefits of more partitions:
├─ More stream processor tasks can run in parallel
├─ Better load distribution
├─ Faster recovery from failures
└─ Limitation: More memory overhead
```

**4. Stream Processing (Flink) Scaling:**

```text
Current: 8 Task Managers
├─ Each: 8 vCPUs, 1,000 events/sec
└─ Total: 8,000 events/sec capacity

Target: Need 35,000 events/sec
├─ Need: 35 Task Managers
├─ With buffer: 40 Task Managers
└─ Parallelism: 36 (matches Kafka partitions)

Scaling strategy:
├─ Increase parallelism: 12 → 36
├─ Each task handles 1 Kafka partition
├─ Scale Task Managers: 8 → 40
└─ Cost: $2,235/month → $11,175/month

Optimization with Spot Instances:
├─ 80% of capacity on Spot (70% discount)
├─ 20% on On-Demand (for stability)
├─ Effective cost: $4,470/month
└─ Savings: $6,705/month ($80,460/year)
```

**5. ClickHouse Cluster Scaling:**

```text
Current: 12 nodes (6 hot + 6 warm)
├─ Storage: 5 TB
└─ Query capacity: 400 QPS

Target (10x):
├─ Storage: 50 TB
├─ Query capacity: 4,000 QPS
└─ Need to scale!

Horizontal scaling approach:
├─ Hot tier: 6 → 18 nodes (3x)
├─ Warm tier: 6 → 24 nodes (4x)
├─ Shards: 2 → 6 shards
└─ Replication: 3x (unchanged)

Sharding strategy:
├─ Shard by tenant_id (if multi-tenant)
├─ OR shard by date range
├─ Example: 6 shards = 6 date ranges
│  ├─ Shard 1: Days 1-15 of month
│  ├─ Shard 2: Days 16-31 of month
│  └─ Rotate monthly
└─ Benefit: Queries only hit relevant shards
```

**Auto-scaling Decision Matrix:**

| Component | Metric | Scale Up When | Scale Down When | Response Time |
|-----------|--------|---------------|-----------------|---------------|
| **API Servers** | CPU > 70% | Sustained 5 min | CPU < 30% for 10 min | 2 minutes |
| **Kafka Brokers** | Disk > 80% | Sustained 30 min | Manual only | 15 minutes |
| **Flink Tasks** | Lag > 10 min | Sustained 5 min | Lag < 1 min for 30 min | 5 minutes |
| **ClickHouse** | Disk > 80% | Sustained 1 hour | Manual only | 30 minutes |
| **Query Servers** | QPS > 350/node | Sustained 5 min | QPS < 100/node for 15 min | 3 minutes |

---

🔴 **ADVANCED: Production Scaling Patterns**

**1. Staged Rollout Strategy:**

```text
Don't scale all at once! Use gradual rollout:

Phase 1: Testing (Week 1)
├─ Scale 1 component to 2x capacity
├─ Route 10% of traffic to new capacity
├─ Monitor: latency, errors, cost
└─ Rollback plan: Ready to revert in 5 minutes

Phase 2: Validation (Week 2)
├─ Route 50% of traffic
├─ Run load tests at 10x scale
├─ Verify: all metrics within SLAs
└─ Fix any issues found

Phase 3: Full Rollout (Week 3-4)
├─ Scale all components
├─ Route 100% of traffic
├─ Monitor closely for 2 weeks
└─ Document learnings

Phase 4: Optimization (Week 5-6)
├─ Identify bottlenecks
├─ Tune configurations
├─ Reduce over-provisioning
└─ Optimize costs
```

**2. Hotspot Detection and Mitigation:**

```text
Problem: Uneven load distribution

Example hotspot scenarios:
├─ Celebrity user generates 1000x events
├─ Viral product gets 500x more views
├─ One tenant has 10x more traffic
└─ Black Friday: 10x spike in specific region

Detection:
├─ Monitor partition lag in Kafka
├─ Track query latency per shard
├─ Alert when any partition > 3x average load
└─ Dashboard showing load distribution

Mitigation strategies:

A. Key Salting (for skewed keys):
```

```python
# Without salting: Celebrity user overwhelms 1 partition
partition = hash(user_id) % num_partitions

# With salting: Spread across multiple partitions
salt = hash(user_id + timestamp) % 10  # 10 salts
partition = hash(str(user_id) + str(salt)) % num_partitions

# Result: 1 hot user spread across 10 partitions
```

```text
B. Dedicated Resources for Hot Tenants:
├─ Detect: Tenant generates >10x average traffic
├─ Isolate: Route to dedicated Kafka topic + Flink job
├─ Scale: Provision dedicated resources
└─ Bill: Charge premium for dedicated resources

C. Circuit Breaker for Hotspots:
```

```python
class HotspotCircuitBreaker:
    def __init__(self, threshold_qps=1000):
        self.threshold_qps = threshold_qps
        self.counters = {}  # tenant_id -> count
    
    def should_throttle(self, tenant_id):
        current_qps = self.counters.get(tenant_id, 0)
        
        if current_qps > self.threshold_qps:
            # Throttle by returning 429 Too Many Requests
            return True
        
        return False
    
    def increment(self, tenant_id):
        self.counters[tenant_id] = self.counters.get(tenant_id, 0) + 1
    
    def reset_every_second(self):
        # Reset counters every second
        self.counters = {}
```

**3. Data Skew Handling:**

```text
Problem: Some shards have much more data

Example:
├─ Shard 1 (US users): 60% of data
├─ Shard 2 (EU users): 25% of data
├─ Shard 3 (APAC users): 15% of data
└─ Result: Shard 1 is bottleneck

Solution: Adaptive Sharding
```

```sql
-- Reshard to distribute load evenly
-- Original: 3 shards by region
-- New: 6 shards with balanced distribution

-- US gets 3 shards (was 1)
-- EU gets 2 shards (was 1)
-- APAC gets 1 shard (was 1)

-- Resharding process:
1. Create new shard layout
2. Backfill historical data (parallel)
3. Switch stream processing to new layout
4. Drain old shards
5. Delete old shards

-- Tools: ClickHouse ALTER TABLE MOVE PARTITION
ALTER TABLE events 
MOVE PARTITION 202601 
TO TABLE events_shard_new;
```

**4. Cross-Region Replication:**

```text
For global scale, deploy in multiple regions:

Architecture:
├─ US-East (Primary):
│  ├─ Handles 60% of traffic (US users)
│  ├─ Full stack: Kafka + Flink + ClickHouse
│  └─ Master for US data
├─ EU-West (Secondary):
│  ├─ Handles 25% of traffic (EU users)
│  ├─ Full stack deployed
│  └─ Master for EU data
└─ AP-Southeast (Tertiary):
   ├─ Handles 15% of traffic (APAC users)
   ├─ Full stack deployed
   └─ Master for APAC data

Data Replication:
├─ Each region has local copy of ALL data
├─ Async replication across regions (5-10 min lag)
├─ Users query local region (low latency)
└─ Global dashboards aggregate from all regions

Benefits:
├─ Low latency: Users query local region
├─ High availability: Region failure doesn't affect others
├─ Compliance: Data residency requirements (GDPR)
└─ Disaster recovery: Each region is backup for others

Cost:
├─ 3x infrastructure: $10K/month → $30K/month
├─ Cross-region bandwidth: ~$500/month
├─ Total: $30.5K/month
└─ Benefit: Serve global users with <100ms latency
```

**5. Real Company Example: Uber's Scaling Journey**

```text
Uber's analytics platform evolution:

2015 (1M trips/day):
├─ Single datacenter
├─ MySQL for analytics (didn't scale)
├─ Batch processing (24hr latency)
└─ Problem: Can't make real-time decisions

2017 (10M trips/day):
├─ Migrated to Kafka + Spark + Cassandra
├─ Real-time processing (1-2 min latency)
├─ Horizontal scaling
└─ Cost: $500K/month

2020 (100M trips/day):
├─ Upgraded to Kafka + Flink + ClickHouse
├─ 3 regions (US, EU, APAC)
├─ Auto-scaling (elastic capacity)
├─ Optimizations: Pre-aggregation, caching
└─ Cost: $2M/month (4x scale but only 4x cost!)

2024 (1B trips/day):
├─ 10 regions globally
├─ Multi-tenant isolation
├─ ML-powered resource allocation
├─ Cost per event: $0.002 → $0.0002 (10x cheaper!)
└─ Cost: $6M/month (linear scaling achieved)

Key learnings:
├─ Start simple, scale incrementally
├─ Horizontal scaling is essential
├─ Pre-aggregation saves 80% cost
├─ Multi-region reduces latency by 70%
└─ Automation critical at scale
```

---

### Data Partitioning

🟢 **BEGINNER: What is Partitioning?**

**Library Analogy:**

```text
Small Library (No Partitioning):
├─ 1,000 books on 1 shelf
├─ Finding a book: Check all 1,000 books
└─ Time: 10 minutes

Large Library (With Partitioning):
├─ 100,000 books across 100 shelves
├─ Partition by: First letter of title
│  ├─ Shelf A: Books starting with A
│  ├─ Shelf B: Books starting with B
│  └─ ...
├─ Finding "Harry Potter": Go to Shelf H, check ~1,000 books
└─ Time: Still 10 minutes (not worse!)

Smart Library (Good Partitioning):
├─ 100,000 books across 100 shelves
├─ Partition by: Genre + First letter
│  ├─ Fiction-A, Fiction-B, ..., Fiction-Z
│  ├─ Science-A, Science-B, ..., Science-Z
│  └─ ...
├─ Finding "Harry Potter" (Fiction-H): Check ~100 books
└─ Time: 1 minute (10x faster!)
```

**In Our Analytics System:**

```text
Without Partitioning:
├─ 1 billion events in 1 table
├─ Query: Find events from yesterday
├─ Must scan: All 1 billion events
└─ Time: 10 minutes

With Date Partitioning:
├─ 1 billion events split into 365 partitions (by day)
├─ Query: Find events from yesterday
├─ Must scan: Only 1 partition (~2.7M events)
├─ Time: 2 seconds (300x faster!)
└─ Benefit: Only read relevant data
```

**Two Types of Partitioning:**

```text
1. Vertical Partitioning (Split by columns):
   ├─ Store different columns separately
   ├─ Example: Hot columns vs Cold columns
   └─ Benefit: Only read columns you need

2. Horizontal Partitioning (Split by rows):
   ├─ Store different rows separately
   ├─ Example: Partition by date, by tenant, by region
   └─ Benefit: Only read rows you need
```

---

🟡 **INTERMEDIATE: Partitioning Strategies**

**1. Time-based Partitioning:**

Most common for analytics (time-series data):

```sql
-- ClickHouse: Partition by month
CREATE TABLE events (
    event_id String,
    user_id UInt64,
    timestamp DateTime,
    event_type String,
    revenue Decimal(10,2)
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)  -- Partition by month
ORDER BY (timestamp, user_id);

-- Result:
-- Partition 202601: January 2026 events
-- Partition 202602: February 2026 events
-- Partition 202603: March 2026 events
-- ...

-- Query optimization:
SELECT COUNT(*) FROM events
WHERE timestamp >= '2026-01-01'
  AND timestamp < '2026-02-01';
-- Only scans partition 202601 (1 month of data)
-- Skips 11 other months
```

**Partition Size Guidelines:**

```text
Partition too small:
├─ 1 partition per hour = 720 partitions/month
├─ Problem: Too many partitions (overhead)
├─ Metadata: 1 KB/partition × 720 = 720 KB
└─ Merge operations slow down

Partition too large:
├─ 1 partition per year = 12 partitions total
├─ Problem: Can't skip much data
├─ Query scanning: 1/12 of data (still large)
└─ Minimal benefit

Optimal partition size:
├─ 1 partition per day: 365 partitions/year
├─ OR 1 partition per month: 12 partitions/year
├─ Balance: Granular enough to skip data, not too many
└─ Rule of thumb: 10 GB - 100 GB per partition
```

**2. Hash Partitioning (By Key):**

Distribute data evenly across shards:

```sql
-- Partition by hash of user_id
CREATE TABLE events_distributed (
    event_id String,
    user_id UInt64,
    timestamp DateTime,
    event_type String
) ENGINE = Distributed(
    cluster_name,
    database_name,
    events_local,
    sipHash64(user_id)  -- Hash function
);

-- Result:
-- User 12345 → Shard 1
-- User 67890 → Shard 2
-- User 11111 → Shard 3

-- Benefit: Even distribution
-- Trade-off: Can't skip shards for user queries
```

**3. Range Partitioning:**

Partition by value ranges:

```sql
-- Partition by revenue ranges
CREATE TABLE orders (
    order_id String,
    user_id UInt64,
    revenue Decimal(10,2)
) ENGINE = MergeTree()
PARTITION BY 
    multiIf(
        revenue < 100, 'low',
        revenue < 1000, 'medium',
        revenue < 10000, 'high',
        'vip'
    )
ORDER BY order_id;

-- Result:
-- Partition 'low': Orders < $100
-- Partition 'medium': Orders $100-$1000
-- Partition 'high': Orders $1000-$10000
-- Partition 'vip': Orders > $10000

-- Query optimization:
SELECT * FROM orders WHERE revenue > 5000;
-- Only scans 'high' and 'vip' partitions
```

**4. Composite Partitioning:**

Combine multiple strategies:

```sql
-- Partition by tenant AND date
CREATE TABLE events_multitenant (
    tenant_id UInt32,
    event_id String,
    timestamp DateTime,
    event_type String
) ENGINE = MergeTree()
PARTITION BY (tenant_id, toYYYYMM(timestamp))
ORDER BY (tenant_id, timestamp);

-- Result:
-- Partition (1, 202601): Tenant 1, January 2026
-- Partition (1, 202602): Tenant 1, February 2026
-- Partition (2, 202601): Tenant 2, January 2026
-- ...

-- Query optimization:
SELECT * FROM events_multitenant
WHERE tenant_id = 1
  AND timestamp >= '2026-01-01'
  AND timestamp < '2026-02-01';
-- Only scans partition (1, 202601)
-- Skips all other tenants and months!
```

**Partition Pruning Examples:**

```text
Query 1: Single day
├─ WHERE timestamp = '2026-01-15'
├─ Partitions scanned: 1 (202601)
├─ Data scanned: 10 GB
└─ Speedup: 36x (vs scanning all months)

Query 2: Month range
├─ WHERE timestamp >= '2026-01-01' AND timestamp < '2026-04-01'
├─ Partitions scanned: 3 (202601, 202602, 202603)
├─ Data scanned: 30 GB
└─ Speedup: 12x

Query 3: No time filter (bad!)
├─ WHERE user_id = 12345
├─ Partitions scanned: ALL (12 months)
├─ Data scanned: 360 GB
└─ Speedup: 1x (no benefit)

Lesson: Always include partition key in WHERE clause!
```

---

🔴 **ADVANCED: Partition Management at Scale**

**1. Dynamic Partitioning:**

Automatically create partitions as data arrives:

```sql
-- ClickHouse automatically creates partitions
-- When inserting data with new toYYYYMM(timestamp) value

INSERT INTO events VALUES
('evt1', 12345, '2026-03-15 10:00:00', 'page_view', 0);
-- Partition 202603 created automatically if doesn't exist

-- No need to pre-create partitions!
-- Benefit: Handles future dates automatically
```

**2. Partition Lifecycle Management:**

```sql
-- Automatically drop old partitions
ALTER TABLE events 
DROP PARTITION 202512;  -- Drop December 2025

-- Move old partitions to cold storage
ALTER TABLE events 
MOVE PARTITION 202601 TO VOLUME 'cold';

-- Automated with TTL:
ALTER TABLE events 
MODIFY TTL 
    timestamp + INTERVAL 90 DAY DELETE,  -- Delete after 90 days
    timestamp + INTERVAL 30 DAY TO VOLUME 'cold';  -- Move to cold after 30 days
```

**3. Partition Rebalancing:**

When data skew occurs:

```python
def check_partition_balance():
    """Monitor partition sizes and rebalance if needed"""
    partitions = clickhouse.query("""
        SELECT 
            partition,
            COUNT(*) as row_count,
            formatReadableSize(SUM(bytes)) as size
        FROM system.parts
        WHERE table = 'events'
          AND active = 1
        GROUP BY partition
        ORDER BY size DESC
    """)
    
    avg_size = sum(p['size'] for p in partitions) / len(partitions)
    
    for partition in partitions:
        if partition['size'] > avg_size * 2:
            print(f"Partition {partition['partition']} is 2x larger than average")
            print(f"Consider: splitting into sub-partitions")
            
            # Option: Reshard this partition
            reshard_partition(partition['partition'])

def reshard_partition(partition_id):
    """Split large partition into multiple smaller ones"""
    # 1. Create new sub-partitioned table
    # 2. Copy data with additional partition key
    # 3. Swap tables atomically
    # 4. Drop old partition
    pass
```

**4. Multi-Level Partitioning:**

For very large datasets:

```sql
-- Level 1: Partition by year-month (for time-based pruning)
-- Level 2: Sub-partition by tenant (for tenant isolation)

CREATE TABLE events_hierarchical (
    tenant_id UInt32,
    event_id String,
    timestamp DateTime,
    event_type String
) ENGINE = MergeTree()
PARTITION BY (toYYYYMM(timestamp), intDiv(tenant_id, 100))
ORDER BY (tenant_id, timestamp);

-- Result:
-- Partition (202601, 0): Jan 2026, Tenants 0-99
-- Partition (202601, 1): Jan 2026, Tenants 100-199
-- Partition (202601, 2): Jan 2026, Tenants 200-299
-- ...

-- Benefits:
-- 1. Time queries: Skip non-matching months
-- 2. Tenant queries: Skip non-matching tenant ranges
-- 3. Combined: Skip on both dimensions
```

**5. Partition-wise Operations:**

Perform operations on individual partitions:

```sql
-- Backup specific partition
ALTER TABLE events 
FREEZE PARTITION 202601;
-- Creates immutable snapshot of partition

-- Restore from backup
ALTER TABLE events 
ATTACH PARTITION 202601 FROM '/backup/path';

-- Move partition between clusters
-- (for migrating data to new cluster)
ALTER TABLE events 
FETCH PARTITION 202601 FROM '/zookeeper/path';
```

**6. Real Company Example: Cloudflare's Partitioning**

```text
Cloudflare analyzes 25 PB of logs per day:

Partitioning strategy:
├─ Primary: Partition by hour (24 partitions/day)
├─ Secondary: Sub-partition by datacenter (200+ datacenters)
├─ Tertiary: Bucket by customer ID (for multi-tenancy)
└─ Result: 1M+ partitions active at any time

Why hourly partitions?
├─ Queries typically ask for: "last N hours"
├─ Hour granularity: optimal for query patterns
├─ Partition size: ~1 TB/partition (manageable)
└─ TTL: Drop hourly partitions after 30 days

Optimization:
├─ Writes: Append-only to current hour partition
├─ Reads: Query hits 1-24 partitions typically
├─ Compaction: Merge small hourly files into daily
└─ Result: 99.9% of queries scan <1% of data

Performance:
├─ Query latency: P50 = 200ms, P99 = 2s
├─ Write throughput: 10M events/sec
├─ Storage cost: $0.02/GB/month (compressed)
└─ Total cost: $500K/month for 25 PB/day
```

---

### Horizontal Scaling

🟢 **BEGINNER: Adding More Servers**

**Pizza Shop Analogy:**

```text
1 Oven (Vertical Scaling):
├─ Make bigger oven
├─ Problem: Size limits
└─ Cost: Exponentially expensive

Multiple Ovens (Horizontal Scaling):
├─ Buy 10 small ovens
├─ Each makes 10 pizzas/hour
├─ Total: 100 pizzas/hour
└─ Cost: Linear (10x ovens = 10x capacity)

Benefits:
├─ No single point of failure (if 1 oven breaks, 9 still work)
├─ Can add ovens incrementally
├─ Each oven is replaceable
└─ Proven model (every pizza chain does this!)
```

**In Our System:**

```text
Vertical Scaling (Single Server):
├─ 8 CPU → 16 CPU → 32 CPU → 64 CPU
├─ Cost: $100 → $200 → $500 → $2,000/month
├─ Limit: Max 128 CPU (hardware limit)
└─ Problem: Single point of failure

Horizontal Scaling (Multiple Servers):
├─ 1 server (8 CPU) → 2 servers → 4 servers → 8 servers
├─ Cost: $100 → $200 → $400 → $800/month
├─ Limit: Unlimited (add more servers)
└─ Benefit: No single point of failure
```

**What Can Scale Horizontally:**

```text
Stateless Components (Easy):
├─ API servers: Just add more, put behind load balancer
├─ Stream processors: Add more workers, increase parallelism
└─ Query servers: Add more, route queries via load balancer

Stateful Components (Harder):
├─ Databases: Need sharding/replication
├─ Message queues: Need partitioning
└─ Cache: Need consistent hashing
```

---

🟡 **INTERMEDIATE: Scaling Each Component**

**1. API Server Horizontal Scaling:**

```text
Architecture:
┌──────────────┐
│ Load Balancer│ (AWS ELB / Nginx)
└──────┬───────┘
       │
   ┌───┴────┬────────┬────────┐
   │        │        │        │
┌──▼───┐ ┌──▼───┐ ┌──▼───┐ ┌──▼───┐
│ API 1│ │ API 2│ │ API 3│ │ API 4│
└──────┘ └──────┘ └──────┘ └──────┘

Load Balancing Strategies:
├─ Round-robin: Request 1 → API 1, Request 2 → API 2, etc.
├─ Least connections: Route to server with fewest active connections
├─ Least response time: Route to fastest server
└─ Weighted: Route more traffic to larger servers

Health Checks:
├─ Every 10 seconds: Check /health endpoint
├─ If server unhealthy: Stop routing traffic
├─ After 3 failures: Mark server as down
└─ Auto-recovery: Resume routing after server healthy again
```

```python
# API server auto-scaling policy (AWS/Terraform)
resource "aws_autoscaling_group" "api_servers" {
  min_size = 4
  max_size = 50
  desired_capacity = 10
  
  # Scale up if CPU > 70% for 5 minutes
  target_tracking_configuration {
    metric = "CPUUtilization"
    target_value = 70
    scale_up_cooldown = 300    # 5 minutes
    scale_down_cooldown = 600  # 10 minutes
  }
}
```

**2. Kafka Horizontal Scaling:**

```text
Adding Brokers:
├─ Start: 3 brokers
├─ Add: 3 more brokers (total 6)
└─ Result: 2x throughput

Kafka automatically rebalances:
├─ Partitions redistributed across 6 brokers
├─ Each broker handles fewer partitions
└─ Load balanced automatically

Process:
1. Add new broker to cluster
2. Kafka detects new broker
3. Run partition reassignment tool:
```

```bash
# Reassign partitions to include new brokers
kafka-reassign-partitions.sh \
  --bootstrap-server localhost:9092 \
  --reassignment-json-file reassignment.json \
  --execute

# reassignment.json specifies new partition→broker mapping
{
  "partitions": [
    {"topic": "events", "partition": 0, "replicas": [0,1,2]},
    {"topic": "events", "partition": 1, "replicas": [1,2,3]},
    {"topic": "events", "partition": 2, "replicas": [2,3,4]},
    ...
  ]
}
```

**3. Flink Horizontal Scaling:**

```text
Scaling Flink Jobs:
├─ Current: 8 Task Managers, parallelism = 12
├─ Target: 16 Task Managers, parallelism = 24
└─ Process:

Step 1: Take savepoint (snapshot of job state)
$ flink savepoint <job-id> /path/to/savepoint

Step 2: Cancel job
$ flink cancel <job-id>

Step 3: Scale Task Managers (8 → 16)
$ kubectl scale deployment flink-taskmanager --replicas=16

Step 4: Restart job with higher parallelism
$ flink run -p 24 -s /path/to/savepoint job.jar

Result:
├─ Job resumes from exact state (no data loss)
├─ 24 parallel tasks (was 12)
├─ Each task processes fewer events
└─ 2x throughput
```

**4. ClickHouse Horizontal Scaling:**

```text
Adding Shards:

Current: 2 shards, 3 replicas each = 6 nodes
Target: 4 shards, 3 replicas each = 12 nodes

Configuration:
```

```xml
<!-- config.xml -->
<remote_servers>
  <cluster_name>
    <!-- Shard 1 -->
    <shard>
      <replica><host>node1</host><port>9000</port></replica>
      <replica><host>node2</host><port>9000</port></replica>
      <replica><host>node3</host><port>9000</port></replica>
    </shard>
    <!-- Shard 2 -->
    <shard>
      <replica><host>node4</host><port>9000</port></replica>
      <replica><host>node5</host><port>9000</port></replica>
      <replica><host>node6</host><port>9000</port></replica>
    </shard>
    <!-- Shard 3 (NEW) -->
    <shard>
      <replica><host>node7</host><port>9000</port></replica>
      <replica><host>node8</host><port>9000</port></replica>
      <replica><host>node9</host><port>9000</port></replica>
    </shard>
    <!-- Shard 4 (NEW) -->
    <shard>
      <replica><host>node10</host><port>9000</port></replica>
      <replica><host>node11</host><port>9000</port></replica>
      <replica><host>node12</host><port>9000</port></replica>
    </shard>
  </cluster_name>
</remote_servers>
```

```sql
-- Create distributed table
CREATE TABLE events_distributed ON CLUSTER cluster_name AS events_local
ENGINE = Distributed(cluster_name, default, events_local, rand());

-- Queries automatically distributed across all 4 shards
SELECT country, COUNT(*) FROM events_distributed GROUP BY country;
-- Executes on all 4 shards in parallel, results merged
```

**5. Scaling Coordination:**

```text
Challenge: Scale all components together

Example traffic spike:
├─ Black Friday: 10x normal traffic
├─ Need to scale: API, Kafka, Flink, ClickHouse
└─ But not all at same rate!

Scaling Ratios:
├─ API servers: Scale 10x (stateless, easy)
├─ Kafka: Scale 2x (need more partitions)
├─ Flink: Scale 5x (CPU-bound processing)
├─ ClickHouse: Scale 3x (storage + query load)
└─ Each component has different scaling characteristics

Automation:
```

```python
def auto_scale_cluster(current_load, target_load):
    """Automatically scale entire cluster based on load"""
    scale_factor = target_load / current_load
    
    # Scale each component proportionally
    scale_api_servers(current=10, factor=scale_factor)
    scale_kafka_partitions(current=36, factor=min(scale_factor, 2))  # Cap at 2x
    scale_flink_tasks(current=40, factor=scale_factor * 0.8)
    scale_clickhouse_nodes(current=12, factor=scale_factor * 0.5)
    
    # Wait for scale-up to complete
    wait_for_healthy_cluster()
    
    # Verify new capacity
    verify_cluster_capacity(target_load)
```

---

🔴 **ADVANCED: Elastic Scaling at Scale**

**1. Predictive Auto-scaling:**

Use ML to predict traffic and scale proactively:

```python
import numpy as np
from sklearn.ensemble import RandomForestRegressor

class PredictiveScaler:
    def __init__(self):
        self.model = RandomForestRegressor()
        self.history = []
    
    def train(self, historical_data):
        """Train on historical traffic patterns"""
        # Features: hour, day_of_week, day_of_month, is_holiday
        # Target: events_per_second
        X = np.array([[h['hour'], h['dow'], h['dom'], h['holiday']] 
                      for h in historical_data])
        y = np.array([h['events_per_sec'] for h in historical_data])
        
        self.model.fit(X, y)
    
    def predict_load(self, timestamp):
        """Predict load for given timestamp"""
        hour = timestamp.hour
        dow = timestamp.weekday()
        dom = timestamp.day
        holiday = is_holiday(timestamp)
        
        predicted_load = self.model.predict([[hour, dow, dom, holiday]])[0]
        return predicted_load
    
    def scale_ahead(self, minutes_ahead=15):
        """Scale cluster 15 minutes before predicted spike"""
        future_time = datetime.now() + timedelta(minutes=minutes_ahead)
        predicted_load = self.predict_load(future_time)
        current_load = get_current_load()
        
        if predicted_load > current_load * 1.5:
            print(f"Spike predicted in 15 min: {predicted_load:.0f} events/sec")
            print(f"Scaling up proactively...")
            scale_cluster(predicted_load)
        
        return predicted_load

# Run every 5 minutes
schedule.every(5).minutes.do(scaler.scale_ahead)
```

**Benefits:**
- Scale up BEFORE traffic spike (not during)
- Avoid scrambling during actual spike
- Better user experience (no degradation)
- Typical accuracy: 90%+ for predictable patterns

**2. Cost-Optimized Scaling:**

Balance performance vs cost:

```python
def optimize_cluster_cost(target_performance):
    """Find minimum cost configuration meeting performance target"""
    
    # Options with cost/performance trade-offs
    options = [
        {'name': 'spot_instances', 'cost_factor': 0.3, 'risk': 0.1},
        {'name': 'reserved_instances', 'cost_factor': 0.6, 'risk': 0.0},
        {'name': 'on_demand', 'cost_factor': 1.0, 'risk': 0.0},
    ]
    
    # Optimization: Mix of instance types
    # 70% Spot (cheap, some risk)
    # 20% Reserved (medium cost, no risk)
    # 10% On-Demand (expensive, immediate availability)
    
    total_capacity = calculate_required_capacity(target_performance)
    
    allocation = {
        'spot': int(total_capacity * 0.7),
        'reserved': int(total_capacity * 0.2),
        'on_demand': int(total_capacity * 0.1),
    }
    
    total_cost = (
        allocation['spot'] * base_cost * 0.3 +
        allocation['reserved'] * base_cost * 0.6 +
        allocation['on_demand'] * base_cost * 1.0
    )
    
    print(f"Optimized allocation: {allocation}")
    print(f"Total cost: ${total_cost:,.0f}/month")
    print(f"Savings vs all on-demand: {((1.0 - (total_cost / (total_capacity * base_cost))) * 100):.0f}%")
    
    return allocation

# Result: 40-50% cost savings while meeting SLAs
```

**3. Cascading Failure Prevention:**

```python
class CircuitBreaker:
    """Prevent cascading failures during scaling events"""
    
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
        self.last_failure_time = None
    
    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker OPEN - requests blocked")
        
        try:
            result = func(*args, **kwargs)
            if self.state == 'HALF_OPEN':
                self.state = 'CLOSED'
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = 'OPEN'
                print(f"Circuit breaker OPEN after {self.failure_count} failures")
            
            raise e

# Usage: Wrap scaling operations
circuit_breaker = CircuitBreaker()
circuit_breaker.call(scale_clickhouse_cluster, target_nodes=24)
```

**4. Real Company Example: Netflix's Chaos Engineering**

```text
Netflix's approach to horizontal scaling at massive scale:

Scale: 200M subscribers, 1B hours watched/month

Horizontal Scaling Strategy:
├─ Microservices: 700+ services
├─ Each service: Independently scalable
├─ Auto-scaling: Every 1 minute evaluation
└─ Multi-region: Active-active in 3 AWS regions

Chaos Engineering:
├─ Randomly kill instances (Chaos Monkey)
├─ Simulate AZ failure (Chaos Kong)
├─ Test scaling under stress
└─ Result: System resilient to failures

Scaling Metrics:
├─ Auto-scale trigger: CPU > 70% for 5 min
├─ Scale-up time: 90 seconds (new instance ready)
├─ Scale-down delay: 15 minutes (prevent flapping)
└─ Cost optimization: 40% Spot, 60% On-Demand

Learnings:
├─ Always over-provision by 20% (N+1 redundancy)
├─ Scale up fast, scale down slow
├─ Test failure scenarios regularly
├─ Automate everything (no manual scaling)
└─ Monitor scaling events closely

Cost at Scale:
├─ Infrastructure: $100M+/month
├─ Per-subscriber cost: $0.50/month
├─ Cost reduction with scaling optimization: 30%
└─ Annual savings: $360M
```

---



## 12. PERFORMANCE OPTIMIZATION 🚀

### 🟢 Beginner Level: Understanding Performance Basics

**Analogy: Restaurant Kitchen Efficiency**

Think of performance optimization like making a restaurant kitchen more efficient:
- **Query Optimization**: Like having ingredients pre-chopped (less work when order comes in)
- **Materialized Views**: Like having pre-made sauces ready to use (not making from scratch each time)
- **Data Compression**: Like using vacuum-sealed bags to store more food in the freezer
- **Indexing**: Like organizing ingredients alphabetically for faster retrieval
- **Caching**: Like keeping frequently used items on the counter (not in storage)

**Why Performance Matters:**

```text
Slow Dashboard Impact:
├─ User Experience: Users abandon dashboards loading > 3 seconds
├─ Business Cost: Slow queries = More compute resources = Higher bills
├─ Decision Making: Delayed insights = Missed opportunities
└─ System Load: Inefficient queries slow down entire system

Example Cost Impact:
├─ Query taking 10 seconds → Blocks 10 concurrent users
├─ Optimized to 1 second → Same resources serve 100 concurrent users
├─ Cost savings: 90% reduction in infrastructure
└─ Revenue impact: Better UX = Higher retention
```

**Basic Performance Metrics:**

```python
# Key metrics to monitor for dashboard performance
performance_metrics = {
    "query_latency": {
        "p50": "< 500ms",      # 50% of queries
        "p95": "< 2 seconds",  # 95% of queries
        "p99": "< 5 seconds"   # 99% of queries
    },
    "dashboard_load_time": {
        "initial_render": "< 1 second",
        "data_loaded": "< 3 seconds",
        "full_interactive": "< 5 seconds"
    },
    "data_freshness": {
        "real_time": "< 10 seconds lag",
        "near_real_time": "< 1 minute lag",
        "batch": "< 15 minutes lag"
    }
}
```

**Simple Optimization Wins:**

1. **Add Indexes** - Make lookups 100x faster
2. **Limit Results** - Don't fetch more data than needed
3. **Use Time Windows** - Query last 24 hours, not all history
4. **Pre-aggregate** - Store hourly/daily summaries instead of raw events
5. **Cache Results** - Reuse same query results for multiple users

---

### 🟡 Intermediate Level: Query Performance Optimization

**Query Optimization Techniques:**

**1. Partition Pruning:**

```sql
-- Bad: Scans entire table (10 billion rows)
SELECT COUNT(*)
FROM events
WHERE user_id = 12345;

-- Good: Scans only relevant partition (10 million rows)
SELECT COUNT(*)
FROM events
WHERE event_date >= '2024-01-01'
  AND event_date < '2024-01-02'
  AND user_id = 12345;

-- Performance improvement: 1000x faster (100s → 100ms)
```

**2. Columnar Storage Benefits:**

```text
ClickHouse Columnar Storage:

Row-based (Traditional):
User1, Event1, 2024-01-01 | User2, Event2, 2024-01-01 | ...
└─ Must read entire row even if only need one column

Columnar (ClickHouse):
UserColumn:   [User1, User2, User3, ...]
EventColumn:  [Event1, Event2, Event3, ...]
DateColumn:   [2024-01-01, 2024-01-01, ...]
└─ Read only needed columns

Example Query: SELECT COUNT(*) FROM events WHERE event_date = '2024-01-01'
├─ Row-based: Read 1000 bytes per row × 1M rows = 1 GB
├─ Columnar: Read 10 bytes per row × 1M rows = 10 MB
└─ Performance: 100x less I/O, 10x faster query
```

**3. Materialized Views for Pre-Aggregation:**

```sql
-- Create materialized view for hourly aggregates
CREATE MATERIALIZED VIEW hourly_metrics_mv
ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(event_hour)
ORDER BY (metric_name, event_hour)
AS SELECT
    toStartOfHour(event_timestamp) AS event_hour,
    metric_name,
    count() AS event_count,
    sum(metric_value) AS total_value,
    avg(metric_value) AS avg_value,
    max(metric_value) AS max_value
FROM events
GROUP BY event_hour, metric_name;

-- Query the materialized view (1000x faster)
SELECT
    event_hour,
    metric_name,
    event_count,
    avg_value
FROM hourly_metrics_mv
WHERE event_hour >= now() - INTERVAL 24 HOUR
ORDER BY event_hour DESC;

-- Performance comparison:
-- Raw table: 10 seconds (scanning 100M events)
-- Materialized view: 10ms (scanning 24 hourly aggregates)
```

**4. Query Result Caching:**

```python
from functools import lru_cache
import redis
import hashlib
import json

class QueryCache:
    """Multi-level caching for dashboard queries"""
    
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379)
        self.memory_cache = {}
        
    def get_cache_key(self, query, params):
        """Generate cache key from query and parameters"""
        cache_input = f"{query}:{json.dumps(params, sort_keys=True)}"
        return hashlib.md5(cache_input.encode()).hexdigest()
    
    def get(self, query, params, ttl=300):
        """Get cached query result (L1: Memory, L2: Redis)"""
        cache_key = self.get_cache_key(query, params)
        
        # L1: Check memory cache (fastest)
        if cache_key in self.memory_cache:
            print(f"Cache HIT (Memory) - {cache_key}")
            return self.memory_cache[cache_key]
        
        # L2: Check Redis cache
        cached_value = self.redis_client.get(cache_key)
        if cached_value:
            result = json.loads(cached_value)
            self.memory_cache[cache_key] = result  # Promote to L1
            print(f"Cache HIT (Redis) - {cache_key}")
            return result
        
        print(f"Cache MISS - {cache_key}")
        return None
    
    def set(self, query, params, result, ttl=300):
        """Store query result in cache"""
        cache_key = self.get_cache_key(query, params)
        
        # Store in both L1 and L2
        self.memory_cache[cache_key] = result
        self.redis_client.setex(
            cache_key,
            ttl,
            json.dumps(result)
        )

# Usage example
cache = QueryCache()

def execute_dashboard_query(query, params):
    """Execute query with caching"""
    
    # Check cache first
    cached_result = cache.get(query, params, ttl=300)
    if cached_result:
        return cached_result
    
    # Execute query if not cached
    result = execute_clickhouse_query(query, params)
    
    # Store in cache
    cache.set(query, params, result, ttl=300)
    
    return result

# Performance impact:
# Cache hit rate: 60-80% for typical dashboards
# Cache hit latency: 1-5ms (vs 100-1000ms for query)
# Cost savings: 70% reduction in database load
```

**5. Approximate Query Processing:**

```sql
-- Exact count (slow: 10 seconds for 1B rows)
SELECT COUNT(DISTINCT user_id)
FROM events
WHERE event_date >= '2024-01-01';

-- Approximate count (fast: 100ms, 98% accurate)
SELECT uniq(user_id)  -- ClickHouse's HyperLogLog approximation
FROM events
WHERE event_date >= '2024-01-01';

-- Use cases for approximation:
-- ✓ Dashboard widgets showing "~5.2M users"
-- ✓ Real-time analytics where exactness isn't critical
-- ✗ Financial reports requiring exact counts
-- ✗ Compliance reports needing audit trails
```

**Performance Benchmarks:**

```text
Query Optimization Results (Real Data):

Baseline (Unoptimized):
├─ Query time: 45 seconds
├─ Data scanned: 500 GB
├─ Cost per query: $0.25
└─ Concurrent users supported: 5

After Partitioning:
├─ Query time: 12 seconds (73% faster)
├─ Data scanned: 50 GB (90% less)
├─ Cost per query: $0.025 (90% cheaper)
└─ Concurrent users: 20

After Materialized Views:
├─ Query time: 500ms (99% faster than baseline)
├─ Data scanned: 1 MB (99.9% less)
├─ Cost per query: $0.0001 (99.96% cheaper)
└─ Concurrent users: 500

After Caching (80% hit rate):
├─ Average query time: 50ms (99.9% faster)
├─ Cache hits: 5ms, Cache misses: 500ms
├─ Effective cost: $0.00002 per query
└─ Concurrent users: 5,000+

ROI Calculation:
├─ Infrastructure cost reduction: $50K/month → $5K/month
├─ Development effort: 2 engineer-months
├─ Payback period: 1 month
└─ Annual savings: $540K
```

---

### 🔴 Advanced Level: Data Compression & Storage Optimization

**1. Columnar Compression Strategies:**

```sql
-- ClickHouse compression codecs for different data types

CREATE TABLE events_optimized (
    event_id UInt64 CODEC(Delta, ZSTD(3)),           -- Delta encoding for sequential IDs
    user_id UInt64 CODEC(LZ4),                       -- Fast compression for random IDs
    event_timestamp DateTime CODEC(Delta, ZSTD(3)),  -- Delta encoding for timestamps
    event_type LowCardinality(String),               -- Dictionary encoding for enums
    country_code FixedString(2) CODEC(LZ4),          -- LZ4 for fixed-length strings
    metric_value Float64 CODEC(Gorilla, ZSTD(3)),    -- Gorilla for time-series floats
    json_payload String CODEC(ZSTD(9))               -- High compression for JSON
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(event_timestamp)
ORDER BY (event_type, user_id, event_timestamp);

-- Compression ratios achieved:
-- Raw data size: 1 TB
-- Compressed size: 100 GB (10x compression)
-- Query performance: No degradation (decompression is fast)
```

**Compression Codec Comparison:**

```text
ClickHouse Compression Codecs:

LZ4 (Default):
├─ Compression ratio: 3-5x
├─ Compression speed: 500 MB/s
├─ Decompression speed: 2000 MB/s
├─ CPU usage: Low
└─ Use case: General purpose, balanced performance

ZSTD(1-22):
├─ Compression ratio: 5-15x (level dependent)
├─ Compression speed: 100-400 MB/s
├─ Decompression speed: 800-1200 MB/s
├─ CPU usage: Medium-High
└─ Use case: Better compression when storage cost > CPU cost

Delta Encoding:
├─ Compression ratio: 10-100x for sequential data
├─ Speed: Very fast (CPU-efficient)
├─ Use case: Timestamps, sequential IDs, counters
└─ Example: Timestamps stored as deltas from base

Gorilla Codec (Facebook's time-series compression):
├─ Compression ratio: 10-20x for time-series floats
├─ Optimized for: Values that change slowly over time
├─ Use case: Metrics, sensor data, stock prices
└─ Example: CPU utilization values: [45.2, 45.3, 45.1, ...]

LowCardinality (Dictionary Encoding):
├─ Compression ratio: 5-50x for low-cardinality strings
├─ Memory savings: Store each unique value once
├─ Use case: Enums, country codes, categorical data
└─ Example: 1M rows with 10 unique countries → Dictionary of 10 + indices

Benchmark Results (1 TB dataset):
├─ No compression: 1000 GB, Query: 10s
├─ LZ4: 250 GB (4x), Query: 10.5s (+5% overhead)
├─ ZSTD(3): 150 GB (6.7x), Query: 11s (+10% overhead)
├─ Delta+ZSTD: 100 GB (10x), Query: 11s (+10% overhead)
└─ Optimal: Mixed codecs, 80 GB (12.5x), Query: 10.5s
```

**2. Tiered Storage for Cost Optimization:**

```python
class TieredStorageManager:
    """Manage hot/warm/cold data tiers for cost optimization"""
    
    def __init__(self):
        self.tiers = {
            'hot': {
                'storage_type': 'NVMe SSD',
                'cost_per_gb': 0.20,  # $0.20/GB/month
                'query_latency': '50ms',
                'retention': '7 days'
            },
            'warm': {
                'storage_type': 'SSD',
                'cost_per_gb': 0.10,  # $0.10/GB/month
                'query_latency': '200ms',
                'retention': '30 days'
            },
            'cold': {
                'storage_type': 'S3',
                'cost_per_gb': 0.023,  # $0.023/GB/month
                'query_latency': '2s',
                'retention': '1 year'
            },
            'archive': {
                'storage_type': 'S3 Glacier',
                'cost_per_gb': 0.004,  # $0.004/GB/month
                'query_latency': 'hours',
                'retention': '7 years'
            }
        }
    
    def calculate_storage_cost(self, daily_data_gb=100, days=365):
        """Calculate cost for tiered vs. single-tier storage"""
        
        # Single-tier (all hot storage)
        single_tier_cost = daily_data_gb * days * self.tiers['hot']['cost_per_gb']
        
        # Tiered storage strategy
        hot_data = daily_data_gb * 7    # Last 7 days
        warm_data = daily_data_gb * 30  # Days 8-37
        cold_data = daily_data_gb * 328 # Days 38-365
        
        tiered_cost = (
            hot_data * self.tiers['hot']['cost_per_gb'] +
            warm_data * self.tiers['warm']['cost_per_gb'] +
            cold_data * self.tiers['cold']['cost_per_gb']
        )
        
        savings = single_tier_cost - tiered_cost
        savings_pct = (savings / single_tier_cost) * 100
        
        return {
            'single_tier_cost': single_tier_cost,
            'tiered_cost': tiered_cost,
            'savings': savings,
            'savings_percentage': savings_pct,
            'breakdown': {
                'hot': hot_data * self.tiers['hot']['cost_per_gb'],
                'warm': warm_data * self.tiers['warm']['cost_per_gb'],
                'cold': cold_data * self.tiers['cold']['cost_per_gb']
            }
        }

# Real-world example
manager = TieredStorageManager()
result = manager.calculate_storage_cost(daily_data_gb=100, days=365)

print(f"""
Storage Cost Analysis (100 GB/day for 1 year):

Single-Tier (All NVMe SSD):
├─ Total data: 36,500 GB
├─ Monthly cost: ${result['single_tier_cost']:,.2f}
└─ Query latency: 50ms

Tiered Storage:
├─ Hot (7 days, 700 GB): ${result['breakdown']['hot']:,.2f}
├─ Warm (30 days, 3,000 GB): ${result['breakdown']['warm']:,.2f}
├─ Cold (328 days, 32,800 GB): ${result['breakdown']['cold']:,.2f}
├─ Total monthly cost: ${result['tiered_cost']:,.2f}
└─ Query latency: 50ms (recent), 2s (historical)

Savings:
├─ Monthly: ${result['savings']:,.2f}
├─ Percentage: {result['savings_percentage']:.1f}%
└─ Annual: ${result['savings'] * 12:,.2f}
""")

# Output:
# Single-Tier: $7,300/month
# Tiered: $1,100/month
# Savings: $6,200/month (85%), $74,400/year
```

**3. Pinterest's Performance Optimization Journey:**

```text
Pinterest Real-Time Analytics at Scale:

Scale:
├─ Daily active users: 450M
├─ Events per second: 2M+
├─ Dashboard queries: 100K/day
├─ Data ingested: 10 TB/day
└─ Query latency target: p95 < 1 second

Initial Performance Problems (2019):
├─ Query latency: p95 = 30 seconds
├─ Dashboard load time: 45 seconds
├─ User complaints: "Too slow to be useful"
├─ Infrastructure cost: $2M/month
└─ Engineering cost: 10 FTEs managing infrastructure

Optimization Phase 1: Partitioning & Indexing (Q1 2020)
├─ Action: Partition by date, index by user_id
├─ Result: p95 latency → 8 seconds (73% improvement)
├─ Data scanned: Reduced by 90%
└─ Cost savings: $400K/month

Optimization Phase 2: Materialized Views (Q3 2020)
├─ Action: Pre-aggregate popular metrics at hourly granularity
├─ Coverage: 80% of queries use materialized views
├─ Result: p95 latency → 2 seconds (75% improvement)
├─ Storage overhead: +15% (worthwhile trade-off)
└─ Cost savings: $600K/month

Optimization Phase 3: Query Result Caching (Q4 2020)
├─ Action: Redis cache with 5-minute TTL
├─ Cache hit rate: 65%
├─ Result: p95 latency → 500ms (75% improvement)
├─ Cache infrastructure cost: $50K/month
└─ Net savings: $350K/month

Optimization Phase 4: Compression & Tiered Storage (2021)
├─ Action: ZSTD compression + S3 for data > 30 days old
├─ Compression ratio: 8x average
├─ Storage cost: $2M/month → $400K/month
├─ Query performance: No significant degradation
└─ Annual savings: $19.2M

Final Results (2022):
├─ Query latency: p95 = 400ms (98.7% improvement)
├─ Dashboard load time: 2 seconds (96% improvement)
├─ Infrastructure cost: $600K/month (70% reduction)
├─ User satisfaction: 4.5/5 (was 2.1/5)
└─ ROI: 10 engineer-months effort, $1.4M savings/month

Key Techniques Used:
├─ Aggressive partitioning by date
├─ Materialized views for popular aggregations
├─ Multi-level caching (Redis + CDN)
├─ Columnar compression (ZSTD)
├─ Tiered storage (SSD → S3)
├─ Approximate queries for non-critical metrics
└─ Query result pre-computation for scheduled reports

Lessons Learned:
├─ 80/20 rule: 20% of queries account for 80% of load
├─ Cache invalidation is hard: Use short TTLs (5 min)
├─ Compression is free performance: CPU << Storage cost
├─ Measure everything: Can't optimize what you don't measure
└─ User experience > Cost: Invest in performance first
```

**4. Airbnb's Query Performance Architecture:**

```python
# Airbnb's query optimization framework

class QueryOptimizer:
    """Airbnb's approach to optimizing analytics queries"""
    
    def __init__(self):
        self.optimization_rules = [
            self.partition_pruning,
            self.predicate_pushdown,
            self.materialized_view_rewrite,
            self.approximate_aggregation
        ]
    
    def partition_pruning(self, query):
        """Add partition filters to reduce data scanned"""
        # Example: Automatically add date range if missing
        if 'event_date' not in query.filters:
            query.filters.append({
                'field': 'event_date',
                'operator': '>=',
                'value': 'current_date - interval 7 days'
            })
        return query
    
    def predicate_pushdown(self, query):
        """Push filters as close to data source as possible"""
        # Move WHERE clauses before JOINs
        # Reduce rows processed in JOIN operations
        return query
    
    def materialized_view_rewrite(self, query):
        """Automatically use materialized views when available"""
        # Check if query matches any materialized view pattern
        for mv in self.get_available_materialized_views():
            if self.query_matches_mv(query, mv):
                return self.rewrite_to_use_mv(query, mv)
        return query
    
    def approximate_aggregation(self, query):
        """Use approximate functions for non-critical queries"""
        replacements = {
            'COUNT(DISTINCT user_id)': 'approx_distinct(user_id)',
            'PERCENTILE(value, 0.95)': 'approx_percentile(value, 0.95)'
        }
        for exact, approx in replacements.items():
            if exact in query.text and not query.requires_exact:
                query.text = query.text.replace(exact, approx)
        return query
    
    def optimize(self, query):
        """Apply all optimization rules"""
        for rule in self.optimization_rules:
            query = rule(query)
        return query

# Performance impact at Airbnb scale:
# Queries optimized: 95% of all dashboard queries
# Average speedup: 25x faster
# Cost reduction: $3M/year in compute costs
# User satisfaction: Dashboard load time < 2s for 99% of users
```

**Cost-Performance Trade-offs:**

```text
Storage & Compute Cost Analysis:

Scenario: 100 TB analytics data, 10K queries/day

Option 1: All-Flash Storage + No Optimization
├─ Storage: 100 TB × $0.20/GB = $20K/month
├─ Compute: 10K queries × 10s avg × $0.01/vCPU-hour = $15K/month
├─ Total: $35K/month
└─ Query latency: p95 = 15s

Option 2: Compression + Basic Optimization
├─ Storage: 20 TB (5x compression) × $0.20/GB = $4K/month
├─ Compute: 10K queries × 3s avg × $0.01/vCPU-hour = $4.5K/month
├─ Total: $8.5K/month (-76%)
└─ Query latency: p95 = 5s

Option 3: Tiered Storage + Materialized Views
├─ Storage (Hot): 5 TB × $0.20/GB = $1K/month
├─ Storage (Cold): 15 TB × $0.023/GB = $0.35K/month
├─ Compute: 10K queries × 500ms avg × $0.01/vCPU-hour = $0.75K/month
├─ Total: $2.1K/month (-94%)
└─ Query latency: p95 = 1s

Option 4: Full Optimization (Compression + Tiers + MVs + Cache)
├─ Storage (Hot): 3 TB × $0.20/GB = $0.6K/month
├─ Storage (Warm): 7 TB × $0.10/GB = $0.7K/month
├─ Storage (Cold): 10 TB × $0.023/GB = $0.23K/month
├─ Cache (Redis): 100 GB × $0.50/GB = $0.05K/month
├─ Compute: 10K queries × 100ms avg × $0.01/vCPU-hour = $0.15K/month
├─ Total: $1.73K/month (-95%)
└─ Query latency: p95 = 200ms

ROI Analysis:
├─ Initial cost: $35K/month
├─ Optimized cost: $1.73K/month
├─ Savings: $33.27K/month, $399K/year
├─ Engineering effort: 4 engineer-months ($80K)
├─ Payback period: 2.4 months
└─ 3-year ROI: 1,397%

Recommendation: Option 4 (Full Optimization)
├─ Best cost-performance ratio
├─ User experience: 75x faster than baseline
├─ Quick payback period
└─ Scales efficiently as data grows
```

---


## 13. SECURITY 🔒

### 🟢 Beginner Level: Understanding Security Basics

**Analogy: Bank Vault Protection**

Think of analytics security like protecting a bank vault:
- **Authentication**: Verify identity (showing ID to enter bank)
- **Authorization**: Control access (only vault employees can enter vault)
- **Encryption**: Protect data in transit and at rest (armored trucks, locked safes)
- **Audit Logging**: Track who accessed what (security cameras, access logs)
- **Data Privacy**: Protect sensitive information (PII in sealed envelopes)

**Why Security Matters for Analytics:**

```text
Analytics Security Concerns:

Data Sensitivity:
├─ User behavior data: Can reveal personal habits
├─ Financial data: Revenue, costs, customer spending
├─ Business metrics: Competitive intelligence
└─ PII (Personally Identifiable Information): Names, emails, IP addresses

Security Incidents Impact:
├─ Data breach: $4.35M average cost per breach (IBM 2022)
├─ Regulatory fines: GDPR up to €20M or 4% revenue
├─ Reputation damage: Loss of customer trust
└─ Business disruption: Systems taken offline for investigation

Example Breach:
├─ Analytics dashboard exposed without authentication
├─ Exposed: 10M user records with email, behavior data
├─ Fine: $5M (GDPR violation)
├─ Legal costs: $2M
├─ Customer churn: 15% (lost revenue: $20M)
└─ Total cost: $27M
```

**Basic Security Principles:**

```python
# Security checklist for analytics dashboards

security_checklist = {
    "authentication": {
        "required": True,
        "methods": ["SSO", "OAuth", "JWT"],
        "mfa_enabled": True,  # Multi-factor authentication
        "session_timeout": "8 hours"
    },
    "authorization": {
        "model": "RBAC",  # Role-Based Access Control
        "principle": "Least privilege",  # Minimum necessary access
        "review_frequency": "Quarterly"
    },
    "encryption": {
        "in_transit": "TLS 1.3",
        "at_rest": "AES-256",
        "key_rotation": "Every 90 days"
    },
    "data_privacy": {
        "pii_masking": True,
        "anonymization": "Hash user IDs",
        "retention_limit": "365 days"
    },
    "audit_logging": {
        "log_access": True,
        "log_changes": True,
        "retention": "7 years",
        "alerting": "Real-time for suspicious activity"
    }
}
```

**Common Security Mistakes:**

1. **No Authentication**: Dashboard accessible to anyone
2. **Weak Passwords**: Default passwords not changed
3. **No Encryption**: Data transmitted in plain text
4. **No Access Control**: All users see all data
5. **No Audit Logs**: Can't track who accessed what
6. **PII Exposure**: Displaying emails, phone numbers unnecessarily

---

### 🟡 Intermediate Level: Access Control & Data Privacy

**1. Role-Based Access Control (RBAC):**

```python
from enum import Enum
from typing import List, Set

class Permission(Enum):
    """Granular permissions for analytics dashboards"""
    VIEW_DASHBOARD = "view_dashboard"
    EDIT_DASHBOARD = "edit_dashboard"
    VIEW_ALL_DATA = "view_all_data"
    VIEW_OWN_TEAM_DATA = "view_own_team_data"
    EXPORT_DATA = "export_data"
    VIEW_PII = "view_pii"
    MANAGE_USERS = "manage_users"
    VIEW_FINANCIAL_DATA = "view_financial_data"

class Role:
    """Role definition with associated permissions"""
    
    def __init__(self, name: str, permissions: Set[Permission]):
        self.name = name
        self.permissions = permissions

# Define standard roles
ROLES = {
    "viewer": Role("Viewer", {
        Permission.VIEW_DASHBOARD,
        Permission.VIEW_OWN_TEAM_DATA
    }),
    
    "analyst": Role("Analyst", {
        Permission.VIEW_DASHBOARD,
        Permission.VIEW_ALL_DATA,
        Permission.EXPORT_DATA
    }),
    
    "manager": Role("Manager", {
        Permission.VIEW_DASHBOARD,
        Permission.EDIT_DASHBOARD,
        Permission.VIEW_ALL_DATA,
        Permission.EXPORT_DATA,
        Permission.VIEW_FINANCIAL_DATA
    }),
    
    "admin": Role("Admin", {
        Permission.VIEW_DASHBOARD,
        Permission.EDIT_DASHBOARD,
        Permission.VIEW_ALL_DATA,
        Permission.EXPORT_DATA,
        Permission.VIEW_PII,
        Permission.MANAGE_USERS,
        Permission.VIEW_FINANCIAL_DATA
    })
}

class AccessControl:
    """Enforce access control for analytics queries"""
    
    def __init__(self, user_id: str, role: str):
        self.user_id = user_id
        self.role = ROLES.get(role)
        
    def can_access(self, permission: Permission) -> bool:
        """Check if user has specific permission"""
        return permission in self.role.permissions
    
    def filter_query(self, query: str) -> str:
        """Add row-level security filters to query"""
        
        # If user can only view own team data, add filter
        if not self.can_access(Permission.VIEW_ALL_DATA):
            # Get user's team
            user_team = self.get_user_team(self.user_id)
            
            # Add WHERE clause to filter by team
            if "WHERE" in query:
                query = query.replace("WHERE", f"WHERE team_id = '{user_team}' AND")
            else:
                query += f" WHERE team_id = '{user_team}'"
        
        return query
    
    def mask_pii(self, results: List[dict]) -> List[dict]:
        """Mask PII fields if user doesn't have VIEW_PII permission"""
        
        if self.can_access(Permission.VIEW_PII):
            return results  # No masking needed
        
        # Mask PII fields
        pii_fields = ['email', 'phone', 'ssn', 'ip_address']
        
        for row in results:
            for field in pii_fields:
                if field in row:
                    row[field] = self.mask_field(row[field])
        
        return results
    
    def mask_field(self, value: str) -> str:
        """Mask sensitive field value"""
        if '@' in value:  # Email
            parts = value.split('@')
            return f"{parts[0][:2]}***@{parts[1]}"
        else:
            return f"***{value[-4:]}"  # Show last 4 characters

# Usage example
def execute_secure_query(query: str, user_id: str, role: str):
    """Execute query with access control"""
    
    ac = AccessControl(user_id, role)
    
    # Check permission
    if not ac.can_access(Permission.VIEW_DASHBOARD):
        raise PermissionError("User not authorized to view dashboard")
    
    # Filter query based on user's access level
    filtered_query = ac.filter_query(query)
    
    # Execute query
    results = execute_query(filtered_query)
    
    # Mask PII if necessary
    results = ac.mask_pii(results)
    
    return results

# Example usage
# Analyst role: Can see all data, no PII
results_analyst = execute_secure_query(
    "SELECT user_id, email, revenue FROM users",
    user_id="analyst_123",
    role="analyst"
)
# Output: user_id visible, email masked, revenue visible

# Admin role: Can see everything including PII
results_admin = execute_secure_query(
    "SELECT user_id, email, revenue FROM users",
    user_id="admin_456",
    role="admin"
)
# Output: user_id visible, email visible, revenue visible
```

**2. API Authentication & Authorization:**

```python
import jwt
import hashlib
import time
from datetime import datetime, timedelta

class APIKeyManager:
    """Manage API keys for programmatic access"""
    
    def __init__(self):
        self.api_keys = {}  # In production: Use database
    
    def generate_api_key(self, user_id: str, permissions: List[str]) -> dict:
        """Generate new API key with specific permissions"""
        
        # Generate random API key
        api_key = hashlib.sha256(
            f"{user_id}:{time.time()}".encode()
        ).hexdigest()
        
        # Store with metadata
        self.api_keys[api_key] = {
            "user_id": user_id,
            "permissions": permissions,
            "created_at": datetime.now(),
            "last_used": None,
            "rate_limit": 1000  # requests per hour
        }
        
        return {
            "api_key": api_key,
            "permissions": permissions,
            "rate_limit": 1000
        }
    
    def validate_api_key(self, api_key: str) -> dict:
        """Validate API key and return user info"""
        
        if api_key not in self.api_keys:
            raise ValueError("Invalid API key")
        
        key_info = self.api_keys[api_key]
        
        # Update last used timestamp
        key_info["last_used"] = datetime.now()
        
        return key_info

class JWTAuthenticator:
    """JWT-based authentication for dashboard API"""
    
    def __init__(self, secret_key: str):
        self.secret_key = secret_key
    
    def generate_token(self, user_id: str, role: str, expires_in: int = 28800) -> str:
        """Generate JWT token (default: 8 hours)"""
        
        payload = {
            "user_id": user_id,
            "role": role,
            "issued_at": datetime.utcnow().isoformat(),
            "expires_at": (datetime.utcnow() + timedelta(seconds=expires_in)).isoformat()
        }
        
        token = jwt.encode(payload, self.secret_key, algorithm="HS256")
        return token
    
    def validate_token(self, token: str) -> dict:
        """Validate JWT token and return payload"""
        
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=["HS256"])
            
            # Check expiration
            expires_at = datetime.fromisoformat(payload["expires_at"])
            if datetime.utcnow() > expires_at:
                raise jwt.ExpiredSignatureError("Token expired")
            
            return payload
            
        except jwt.InvalidTokenError as e:
            raise ValueError(f"Invalid token: {e}")

# Usage in API endpoint
def dashboard_api_endpoint(request):
    """Protected API endpoint example"""
    
    # Extract token from Authorization header
    auth_header = request.headers.get("Authorization", "")
    
    if auth_header.startswith("Bearer "):
        # JWT authentication
        token = auth_header[7:]
        authenticator = JWTAuthenticator(secret_key="your-secret-key")
        user_info = authenticator.validate_token(token)
        
    elif auth_header.startswith("ApiKey "):
        # API key authentication
        api_key = auth_header[7:]
        key_manager = APIKeyManager()
        user_info = key_manager.validate_api_key(api_key)
        
    else:
        return {"error": "Authentication required"}, 401
    
    # Execute query with user's permissions
    results = execute_secure_query(
        request.query,
        user_id=user_info["user_id"],
        role=user_info["role"]
    )
    
    return {"data": results}, 200
```

**3. Data Anonymization & PII Protection:**

```sql
-- PII masking strategies in queries

-- Strategy 1: Hash user identifiers
SELECT
    SHA256(user_id) AS anonymized_user_id,  -- One-way hash
    DATE_TRUNC('day', event_timestamp) AS event_date,
    event_type,
    COUNT(*) AS event_count
FROM events
WHERE event_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY anonymized_user_id, event_date, event_type;

-- Strategy 2: Aggregate to remove individual identification
SELECT
    country_code,
    age_bucket,  -- e.g., '18-24', '25-34'
    gender,
    AVG(revenue) AS avg_revenue,
    COUNT(*) AS user_count
FROM users
GROUP BY country_code, age_bucket, gender
HAVING COUNT(*) >= 10;  -- k-anonymity: minimum 10 users per group

-- Strategy 3: Differential privacy (add noise)
SELECT
    product_category,
    COUNT(*) + (RANDOM() * 20 - 10)::INT AS noisy_count  -- Add ±10 noise
FROM purchases
GROUP BY product_category;

-- Strategy 4: Tokenization for reversible masking
CREATE TABLE user_tokens (
    token_id VARCHAR(64) PRIMARY KEY,
    user_id VARCHAR(64) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Store tokenized data
INSERT INTO analytics_events (token_id, event_type)
SELECT token_id, event_type
FROM events e
JOIN user_tokens ut ON e.user_id = ut.user_id;

-- Only authorized users can reverse tokens
SELECT
    ut.user_id,  -- Requires JOIN with user_tokens table
    e.event_type,
    COUNT(*) AS event_count
FROM analytics_events e
JOIN user_tokens ut ON e.token_id = ut.token_id
WHERE has_permission(current_user(), 'VIEW_PII')
GROUP BY ut.user_id, e.event_type;
```

**4. Encryption Configuration:**

```yaml
# Encryption configuration for analytics pipeline

encryption:
  # Encryption in transit
  in_transit:
    protocol: TLS 1.3
    cipher_suites:
      - TLS_AES_256_GCM_SHA384
      - TLS_CHACHA20_POLY1305_SHA256
    certificate:
      type: "Let's Encrypt"
      auto_renewal: true
      expiry_alert: 30  # days before expiration
  
  # Encryption at rest
  at_rest:
    algorithm: AES-256-GCM
    key_management:
      service: "AWS KMS"
      key_rotation: 90  # days
      auto_rotation: true
    
    # Different keys for different data sensitivity
    keys:
      - name: "pii_data_key"
        usage: "User PII (email, phone, address)"
        rotation: 30  # days
      
      - name: "financial_data_key"
        usage: "Revenue, transaction data"
        rotation: 30  # days
      
      - name: "analytics_data_key"
        usage: "Aggregated metrics (non-sensitive)"
        rotation: 90  # days
  
  # Field-level encryption
  field_level:
    enabled: true
    fields:
      - name: "email"
        algorithm: "AES-256-GCM"
        key: "pii_data_key"
      
      - name: "credit_card"
        algorithm: "AES-256-GCM"
        key: "financial_data_key"
        tokenization: true  # Store tokens, not actual values
      
      - name: "ip_address"
        algorithm: "SHA-256"  # One-way hash
        salt: true

# Performance impact
performance:
  encryption_overhead: "5-10%"  # CPU overhead
  latency_increase: "10-20ms"  # Per request
  storage_overhead: "5-15%"  # Encrypted data size
```

---

### 🔴 Advanced Level: Compliance & Audit Logging

**1. GDPR Compliance Implementation:**

```python
class GDPRComplianceManager:
    """Manage GDPR compliance for analytics data"""
    
    def __init__(self):
        self.data_retention_days = 365
        self.pii_fields = ['email', 'phone', 'ip_address', 'device_id']
    
    def handle_data_subject_request(self, request_type: str, user_id: str):
        """Handle GDPR data subject requests"""
        
        if request_type == "access":
            # Right to access: Provide all data about user
            return self.export_user_data(user_id)
        
        elif request_type == "rectification":
            # Right to rectification: Correct inaccurate data
            return self.update_user_data(user_id)
        
        elif request_type == "erasure":
            # Right to erasure (Right to be forgotten)
            return self.delete_user_data(user_id)
        
        elif request_type == "portability":
            # Right to data portability: Export in machine-readable format
            return self.export_user_data(user_id, format="JSON")
        
        elif request_type == "restriction":
            # Right to restriction: Stop processing user's data
            return self.restrict_user_data_processing(user_id)
    
    def export_user_data(self, user_id: str, format: str = "CSV") -> dict:
        """Export all user data (GDPR Article 15)"""
        
        query = f"""
        SELECT
            user_id,
            email,
            created_at,
            last_login,
            event_type,
            event_timestamp,
            metadata
        FROM events
        WHERE user_id = '{user_id}'
        ORDER BY event_timestamp DESC
        """
        
        data = execute_query(query)
        
        # Create audit log entry
        self.log_gdpr_request(user_id, "access", status="completed")
        
        return {
            "user_id": user_id,
            "data": data,
            "format": format,
            "exported_at": datetime.now().isoformat(),
            "retention_expires": (datetime.now() + timedelta(days=30)).isoformat()
        }
    
    def delete_user_data(self, user_id: str) -> dict:
        """Delete user data (GDPR Article 17 - Right to be forgotten)"""
        
        # Step 1: Identify all user data
        tables_with_user_data = [
            'events', 'user_profiles', 'sessions',
            'analytics_metrics', 'cached_results'
        ]
        
        deletion_summary = {}
        
        for table in tables_with_user_data:
            # Delete from main table
            query = f"DELETE FROM {table} WHERE user_id = '{user_id}'"
            rows_deleted = execute_query(query)
            deletion_summary[table] = rows_deleted
            
            # Delete from backups (mark for deletion)
            self.mark_backup_for_deletion(table, user_id)
        
        # Step 2: Anonymize user in aggregated tables
        # (Can't delete from aggregates, so we anonymize)
        self.anonymize_aggregated_data(user_id)
        
        # Step 3: Log deletion request
        self.log_gdpr_request(
            user_id,
            "erasure",
            status="completed",
            details=deletion_summary
        )
        
        return {
            "user_id": user_id,
            "status": "deleted",
            "deleted_at": datetime.now().isoformat(),
            "deletion_summary": deletion_summary
        }
    
    def anonymize_aggregated_data(self, user_id: str):
        """Anonymize user in pre-aggregated analytics data"""
        
        # Replace user_id with anonymous token
        anonymous_id = hashlib.sha256(f"anonymous_{user_id}".encode()).hexdigest()
        
        query = f"""
        UPDATE hourly_metrics
        SET user_id = '{anonymous_id}'
        WHERE user_id = '{user_id}'
        """
        
        execute_query(query)
    
    def enforce_data_retention(self):
        """Delete data older than retention period (GDPR Article 5)"""
        
        cutoff_date = datetime.now() - timedelta(days=self.data_retention_days)
        
        query = f"""
        DELETE FROM events
        WHERE event_timestamp < '{cutoff_date.isoformat()}'
        AND retention_exempt = FALSE
        """
        
        rows_deleted = execute_query(query)
        
        return {
            "cutoff_date": cutoff_date.isoformat(),
            "rows_deleted": rows_deleted
        }
    
    def log_gdpr_request(self, user_id: str, request_type: str, status: str, details: dict = None):
        """Log GDPR request for audit trail"""
        
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "user_id": user_id,
            "request_type": request_type,
            "status": status,
            "details": details,
            "processed_by": "automated_system"
        }
        
        # Store in audit log (must be retained for 7 years)
        store_audit_log(log_entry)

# Real-world stats:
# GDPR fines issued (2018-2023): €2.79 billion
# Average fine: €1.5 million
# Largest fine: Amazon (€746 million)
# Common violations: Insufficient legal basis, lack of consent, inadequate security
```

**2. Comprehensive Audit Logging:**

```python
import json
from enum import Enum

class AuditEventType(Enum):
    """Types of events to audit"""
    USER_LOGIN = "user_login"
    USER_LOGOUT = "user_logout"
    QUERY_EXECUTED = "query_executed"
    DATA_EXPORTED = "data_exported"
    DASHBOARD_ACCESSED = "dashboard_accessed"
    PERMISSION_CHANGED = "permission_changed"
    DATA_DELETED = "data_deleted"
    PII_ACCESSED = "pii_accessed"
    SECURITY_ALERT = "security_alert"

class AuditLogger:
    """Comprehensive audit logging for compliance"""
    
    def __init__(self):
        self.log_stream = "audit_logs"
        self.retention_years = 7  # Regulatory requirement
    
    def log_event(self, event_type: AuditEventType, user_id: str, details: dict):
        """Log audit event"""
        
        audit_entry = {
            "timestamp": datetime.utcnow().isoformat(),
            "event_type": event_type.value,
            "user_id": user_id,
            "ip_address": self.get_client_ip(),
            "user_agent": self.get_user_agent(),
            "session_id": self.get_session_id(),
            "details": details,
            "severity": self.calculate_severity(event_type)
        }
        
        # Write to audit log
        self.write_to_audit_log(audit_entry)
        
        # Alert on high-severity events
        if audit_entry["severity"] == "high":
            self.send_security_alert(audit_entry)
    
    def log_query_execution(self, user_id: str, query: str, results_count: int):
        """Log query execution for audit trail"""
        
        details = {
            "query": query,
            "results_count": results_count,
            "execution_time_ms": 150,
            "data_scanned_gb": 2.5,
            "contains_pii": self.contains_pii(query)
        }
        
        self.log_event(AuditEventType.QUERY_EXECUTED, user_id, details)
    
    def log_data_export(self, user_id: str, export_type: str, record_count: int):
        """Log data export for compliance"""
        
        details = {
            "export_type": export_type,
            "record_count": record_count,
            "file_format": "CSV",
            "contains_pii": True,
            "purpose": "Business analysis"
        }
        
        self.log_event(AuditEventType.DATA_EXPORTED, user_id, details)
    
    def log_pii_access(self, user_id: str, pii_type: str, record_count: int):
        """Log PII access (GDPR requirement)"""
        
        details = {
            "pii_type": pii_type,  # e.g., "email", "phone"
            "record_count": record_count,
            "access_reason": "Customer support inquiry",
            "approved_by": "manager_123"
        }
        
        self.log_event(AuditEventType.PII_ACCESSED, user_id, details)
    
    def detect_suspicious_activity(self, user_id: str):
        """Detect and log suspicious activity"""
        
        # Check for anomalies
        recent_queries = self.get_recent_queries(user_id, minutes=5)
        
        if len(recent_queries) > 100:
            # Suspicious: > 100 queries in 5 minutes
            self.log_event(
                AuditEventType.SECURITY_ALERT,
                user_id,
                {
                    "alert_type": "rate_limit_exceeded",
                    "query_count": len(recent_queries),
                    "time_window": "5 minutes"
                }
            )
        
        # Check for bulk PII access
        pii_access_count = self.count_pii_access(user_id, hours=1)
        if pii_access_count > 10000:
            # Suspicious: Accessing > 10K PII records in 1 hour
            self.log_event(
                AuditEventType.SECURITY_ALERT,
                user_id,
                {
                    "alert_type": "bulk_pii_access",
                    "record_count": pii_access_count,
                    "time_window": "1 hour"
                }
            )
    
    def calculate_severity(self, event_type: AuditEventType) -> str:
        """Calculate event severity"""
        
        high_severity_events = [
            AuditEventType.DATA_DELETED,
            AuditEventType.PERMISSION_CHANGED,
            AuditEventType.SECURITY_ALERT
        ]
        
        medium_severity_events = [
            AuditEventType.DATA_EXPORTED,
            AuditEventType.PII_ACCESSED
        ]
        
        if event_type in high_severity_events:
            return "high"
        elif event_type in medium_severity_events:
            return "medium"
        else:
            return "low"

# Usage example
audit_logger = AuditLogger()

# Log query execution
audit_logger.log_query_execution(
    user_id="analyst_123",
    query="SELECT user_id, email FROM users LIMIT 1000",
    results_count=1000
)

# Log data export
audit_logger.log_data_export(
    user_id="manager_456",
    export_type="CSV",
    record_count=50000
)

# Detect suspicious activity
audit_logger.detect_suspicious_activity(user_id="analyst_123")
```

**3. Stripe's Security Architecture:**

```text
Stripe Real-Time Analytics Security:

Scale:
├─ Payment events: 1M+ per minute
├─ Sensitive data: Credit card numbers, bank accounts, PII
├─ Regulatory compliance: PCI DSS Level 1, SOC 2, GDPR, CCPA
└─ Audit logs: 100M+ events per day

Security Architecture:
├─ Network Layer: VPC isolation, private subnets
├─ Application Layer: mTLS, API key rotation
├─ Data Layer: Encryption at rest (AES-256), field-level encryption
└─ Monitoring: Real-time security alerts, anomaly detection

Key Security Features:

1. PCI DSS Compliance:
   ├─ Tokenization: Replace card numbers with tokens
   ├─ Encryption: All cardholder data encrypted
   ├─ Access control: Minimum necessary access
   ├─ Monitoring: All access to cardholder data logged
   └─ Annual audit: Third-party PCI assessment

2. Data Access Controls:
   ├─ Role-based: 5 standard roles (Viewer, Developer, Analyst, Admin, Owner)
   ├─ Team-based: Users only see their team's data
   ├─ API keys: Separate keys for test and production
   ├─ Key restrictions: Limit by IP, domain, API endpoint
   └─ Automatic revocation: Keys expire after 90 days of inactivity

3. Audit Logging:
   ├─ Every API call logged
   ├─ Dashboard access logged
   ├─ Data exports logged
   ├─ Permission changes logged
   ├─ Retention: 7 years (regulatory requirement)
   └─ Real-time alerts: Suspicious activity triggers immediate notification

4. Data Privacy:
   ├─ PII minimization: Only collect necessary data
   ├─ Anonymization: Hash user IDs in analytics
   ├─ Right to deletion: Automated GDPR deletion workflow
   ├─ Data residency: Store data in user's region
   └─ Purpose limitation: Data only used for stated purposes

5. Threat Detection:
   ├─ Rate limiting: Prevent brute force attacks
   ├─ Anomaly detection: ML models detect suspicious patterns
   ├─ Fraud detection: Real-time fraud scoring
   ├─ Alerting: Security team notified within 1 minute
   └─ Automated response: Suspicious accounts auto-suspended

Security Metrics:
├─ Zero data breaches (since founding in 2010)
├─ PCI compliance: 100% (annual audit)
├─ Encryption coverage: 100% of data at rest and in transit
├─ Audit log completeness: 99.99%
├─ Incident response time: < 15 minutes (p95)
└─ Security team size: 200+ engineers

Cost of Security:
├─ Security infrastructure: $10M/year
├─ Compliance audits: $2M/year
├─ Security team: $30M/year (salaries)
├─ Total security investment: $42M/year
├─ Revenue: $7.4B/year (2022)
└─ Security as % of revenue: 0.57%

ROI of Security Investment:
├─ Customer trust: Enables enterprise deals
├─ Avoided breaches: Estimated $100M+ in potential losses
├─ Competitive advantage: Security as differentiator
└─ Regulatory compliance: Avoids fines, enables global expansion
```

**4. Uber's Data Security Lessons:**

```text
Uber Data Breach (2016) - Lessons Learned:

Incident:
├─ Breach date: October 2016
├─ Discovery: November 2017 (1 year later)
├─ Data exposed: 57M users (names, emails, phone numbers), 600K drivers (licenses)
├─ Attack vector: Stolen AWS credentials from GitHub
├─ Ransom paid: $100K to hackers (to delete data)
└─ Cover-up: Breach concealed from regulators

Consequences:
├─ Regulatory fines: $148M (50 US states settlement)
├─ Legal fees: $20M+
├─ Reputation damage: Loss of user trust
├─ Executive departures: CSO fired, CEO resigned
├─ Customer churn: Estimated 5-10% loss
└─ Total cost: $200M+ (direct + indirect)

Security Failures:
├─ Credentials in code: AWS keys committed to GitHub repo
├─ No access controls: Engineers had broad access to production
├─ No monitoring: Breach undetected for 1 year
├─ No incident response: Paid ransom instead of reporting
└─ No encryption: PII stored in plaintext

Uber's Security Improvements (Post-Breach):

1. Access Control Overhaul:
   ├─ Zero-trust architecture: Assume breach, verify everything
   ├─ Just-in-time access: Temporary access for specific tasks
   ├─ Principle of least privilege: Minimum necessary permissions
   └─ Regular access reviews: Quarterly permission audits

2. Secrets Management:
   ├─ No secrets in code: Use secret management service (Vault)
   ├─ Automatic rotation: Credentials rotate every 24 hours
   ├─ GitHub scanning: Automated detection of committed secrets
   └─ Break-glass procedures: Emergency access with approval workflow

3. Encryption Everywhere:
   ├─ At rest: AES-256 for all PII
   ├─ In transit: TLS 1.3 for all communications
   ├─ Field-level: Sensitive fields individually encrypted
   └─ Key management: AWS KMS with automatic rotation

4. Monitoring & Alerting:
   ├─ All access logged: 100% coverage
   ├─ Real-time anomaly detection: ML-based threat detection
   ├─ Security Operations Center: 24/7 monitoring
   ├─ Automated response: Suspicious activity auto-blocked
   └─ Regular penetration testing: Quarterly red team exercises

5. Incident Response:
   ├─ Clear protocols: Step-by-step breach response plan
   ├─ Transparent reporting: Immediate notification to regulators
   ├─ User notification: Within 72 hours (GDPR requirement)
   ├─ Post-mortem culture: Blameless incident reviews
   └─ Continuous improvement: Lessons incorporated into security practices

Current Security Posture (2024):
├─ Security team: 500+ engineers
├─ Security budget: $100M+/year
├─ Compliance: SOC 2 Type II, ISO 27001, GDPR, CCPA
├─ Bug bounty program: $10M+ paid to researchers
├─ Incident response time: < 5 minutes (p95)
└─ Zero major breaches since 2016

Key Lessons:
├─ Security is not optional: Cost of breach >> cost of security
├─ Culture matters: Security is everyone's responsibility
├─ Transparency builds trust: Own up to incidents quickly
├─ Invest proactively: Don't wait for breach to improve security
└─ Monitor everything: Can't protect what you can't see
```

---


## 14. MONITORING & OBSERVABILITY 📊

### 🟢 Beginner Level: Understanding Monitoring Basics

**Analogy: Car Dashboard & Warning Lights**

Think of system monitoring like your car's dashboard:
- **Metrics**: Speedometer, fuel gauge, temperature (key numbers at a glance)
- **Logs**: Trip computer showing recent events (what happened and when)
- **Traces**: GPS navigation showing your route (following a request through the system)
- **Alerts**: Warning lights (engine, oil, battery) that tell you when something's wrong
- **Dashboards**: Complete instrument panel showing everything together

**Why Monitoring Matters:**

```text
Without Monitoring:
├─ No visibility: Don't know if system is healthy
├─ Slow detection: Find issues only when users complain
├─ Difficult debugging: Can't see what went wrong
├─ No proactive fixes: React to problems instead of preventing them
└─ Lost revenue: Downtime costs money

With Monitoring:
├─ Real-time visibility: Always know system health
├─ Fast detection: Alerts within seconds of issues
├─ Easy debugging: Logs and traces show root cause
├─ Proactive fixes: Detect issues before they impact users
└─ High availability: 99.9%+ uptime

Example: E-commerce site during Black Friday
├─ Without monitoring: Site crashes, 30 min to detect, 2 hours to fix
├─ Lost sales: $1M (during 2.5 hours downtime)
├─ With monitoring: Alert in 30 seconds, fixed in 5 minutes
├─ Lost sales: $50K (during 5 minutes)
└─ Monitoring ROI: $950K saved in one incident
```

**Basic Monitoring Layers:**

```text
What to Monitor in Analytics Dashboard:

1. Infrastructure Metrics:
   ├─ CPU usage: Should be < 70% average
   ├─ Memory usage: Should be < 80%
   ├─ Disk usage: Should be < 85%
   ├─ Network bandwidth: Monitor for saturation
   └─ Alert if: Any metric sustained above threshold for > 5 minutes

2. Application Metrics:
   ├─ Request rate: Queries per second
   ├─ Error rate: % of failed queries
   ├─ Latency: p50, p95, p99 response times
   ├─ Throughput: Data processed per second
   └─ Alert if: Error rate > 1% or p95 latency > 3s

3. Data Quality Metrics:
   ├─ Data freshness: Time lag between event and availability
   ├─ Completeness: % of expected records received
   ├─ Accuracy: Data validation checks
   ├─ Consistency: Reconciliation with source systems
   └─ Alert if: Freshness > 15 min or completeness < 95%

4. Business Metrics:
   ├─ Active users: Users using dashboards
   ├─ Query patterns: Most common queries
   ├─ Dashboard load times: User experience
   ├─ Feature usage: Which features are popular
   └─ Alert if: Active users drops > 50% suddenly
```

**Simple Monitoring Setup:**

```python
# Basic health check endpoint

from flask import Flask, jsonify
import time
import psutil

app = Flask(__name__)

@app.route('/health')
def health_check():
    """Basic health check endpoint"""
    
    status = {
        "status": "healthy",
        "timestamp": time.time(),
        "checks": {}
    }
    
    # Check CPU usage
    cpu_percent = psutil.cpu_percent(interval=1)
    status["checks"]["cpu"] = {
        "value": cpu_percent,
        "healthy": cpu_percent < 80,
        "threshold": 80
    }
    
    # Check memory usage
    memory = psutil.virtual_memory()
    status["checks"]["memory"] = {
        "value": memory.percent,
        "healthy": memory.percent < 85,
        "threshold": 85
    }
    
    # Check disk usage
    disk = psutil.disk_usage('/')
    status["checks"]["disk"] = {
        "value": disk.percent,
        "healthy": disk.percent < 90,
        "threshold": 90
    }
    
    # Overall status
    all_healthy = all(check["healthy"] for check in status["checks"].values())
    status["status"] = "healthy" if all_healthy else "unhealthy"
    
    status_code = 200 if all_healthy else 503
    return jsonify(status), status_code

# Output example:
# {
#   "status": "healthy",
#   "timestamp": 1704067200,
#   "checks": {
#     "cpu": {"value": 45.2, "healthy": true, "threshold": 80},
#     "memory": {"value": 62.8, "healthy": true, "threshold": 85},
#     "disk": {"value": 73.1, "healthy": true, "threshold": 90}
#   }
# }
```

---

### 🟡 Intermediate Level: Pipeline Health Monitoring

**1. Key Metrics to Monitor:**

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Define Prometheus metrics

# Counter: Always increasing (total events processed)
events_processed = Counter(
    'events_processed_total',
    'Total number of events processed',
    ['pipeline_stage', 'event_type']
)

# Gauge: Can go up or down (current queue size)
queue_size = Gauge(
    'queue_size',
    'Current number of events in queue',
    ['queue_name']
)

# Histogram: Distribution of values (latency buckets)
query_latency = Histogram(
    'query_latency_seconds',
    'Query execution latency in seconds',
    ['query_type'],
    buckets=[0.1, 0.5, 1.0, 2.5, 5.0, 10.0]
)

# Track error rate
query_errors = Counter(
    'query_errors_total',
    'Total number of query errors',
    ['error_type']
)

# Example: Instrumenting a query execution
def execute_query_with_metrics(query, query_type):
    """Execute query with monitoring instrumentation"""
    
    start_time = time.time()
    
    try:
        # Execute query
        result = execute_query(query)
        
        # Record success metrics
        events_processed.labels(
            pipeline_stage='query_execution',
            event_type=query_type
        ).inc()
        
        # Record latency
        latency = time.time() - start_time
        query_latency.labels(query_type=query_type).observe(latency)
        
        return result
        
    except Exception as e:
        # Record error
        query_errors.labels(error_type=type(e).__name__).inc()
        raise

# Example: Update queue size gauge
def update_queue_metrics(queue_name, size):
    """Update queue size gauge"""
    queue_size.labels(queue_name=queue_name).set(size)
```

**2. Prometheus Configuration:**

```yaml
# prometheus.yml - Monitoring configuration

global:
  scrape_interval: 15s      # How often to scrape targets
  evaluation_interval: 15s  # How often to evaluate rules
  
  external_labels:
    cluster: 'analytics-prod'
    region: 'us-east-1'

# Scrape configurations
scrape_configs:
  # Analytics API servers
  - job_name: 'analytics-api'
    static_configs:
      - targets:
          - 'api-1.internal:9090'
          - 'api-2.internal:9090'
          - 'api-3.internal:9090'
    metrics_path: '/metrics'
    scrape_interval: 10s
  
  # ClickHouse database
  - job_name: 'clickhouse'
    static_configs:
      - targets:
          - 'clickhouse-1.internal:9363'
          - 'clickhouse-2.internal:9363'
    metrics_path: '/metrics'
    scrape_interval: 30s
  
  # Kafka brokers
  - job_name: 'kafka'
    static_configs:
      - targets:
          - 'kafka-1.internal:7071'
          - 'kafka-2.internal:7071'
          - 'kafka-3.internal:7071'
    metrics_path: '/metrics'
    scrape_interval: 15s
  
  # Redis cache
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-1.internal:9121']
    scrape_interval: 15s

# Alerting rules
rule_files:
  - 'alerts/analytics_alerts.yml'
  - 'alerts/infrastructure_alerts.yml'

# Alert manager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

**3. Alert Rules Configuration:**

```yaml
# alerts/analytics_alerts.yml

groups:
  - name: analytics_pipeline
    interval: 30s
    rules:
      # Alert: High query latency
      - alert: HighQueryLatency
        expr: histogram_quantile(0.95, rate(query_latency_seconds_bucket[5m])) > 3
        for: 5m
        labels:
          severity: warning
          team: analytics
        annotations:
          summary: "High query latency detected"
          description: "P95 query latency is {{ $value }}s (threshold: 3s)"
      
      # Alert: High error rate
      - alert: HighErrorRate
        expr: rate(query_errors_total[5m]) / rate(events_processed_total[5m]) > 0.01
        for: 2m
        labels:
          severity: critical
          team: analytics
        annotations:
          summary: "High query error rate"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 1%)"
      
      # Alert: Queue backlog
      - alert: KafkaQueueBacklog
        expr: kafka_consumer_lag > 100000
        for: 10m
        labels:
          severity: warning
          team: data-engineering
        annotations:
          summary: "Kafka consumer lag is high"
          description: "Consumer lag is {{ $value }} messages (threshold: 100K)"
      
      # Alert: Data freshness issue
      - alert: StaleData
        expr: time() - last_event_timestamp > 900  # 15 minutes
        for: 5m
        labels:
          severity: critical
          team: data-engineering
        annotations:
          summary: "Data ingestion stalled"
          description: "No new events in last {{ $value }}s (threshold: 900s)"
      
      # Alert: Low cache hit rate
      - alert: LowCacheHitRate
        expr: rate(cache_hits[5m]) / (rate(cache_hits[5m]) + rate(cache_misses[5m])) < 0.5
        for: 15m
        labels:
          severity: warning
          team: analytics
        annotations:
          summary: "Cache hit rate is low"
          description: "Hit rate is {{ $value | humanizePercentage }} (threshold: 50%)"
      
      # Alert: Database connection pool exhausted
      - alert: DatabaseConnectionPoolExhausted
        expr: clickhouse_connection_pool_active / clickhouse_connection_pool_size > 0.9
        for: 5m
        labels:
          severity: critical
          team: database
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "{{ $value | humanizePercentage }} of connections in use"

  - name: infrastructure
    interval: 30s
    rules:
      # Alert: High CPU usage
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 10m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% (threshold: 80%)"
      
      # Alert: High memory usage
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 10m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}% (threshold: 85%)"
      
      # Alert: Disk space running low
      - alert: DiskSpaceLow
        expr: (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100 > 85
        for: 5m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "Disk space low on {{ $labels.instance }}"
          description: "Disk usage is {{ $value }}% (threshold: 85%)"
```

**4. Grafana Dashboard Configuration:**

```json
{
  "dashboard": {
    "title": "Real-Time Analytics - Pipeline Health",
    "tags": ["analytics", "monitoring"],
    "timezone": "UTC",
    "panels": [
      {
        "id": 1,
        "title": "Query Throughput (QPS)",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(events_processed_total{pipeline_stage='query_execution'}[5m])",
            "legendFormat": "{{instance}} - {{query_type}}"
          }
        ],
        "yAxisLabel": "Queries per Second",
        "alert": {
          "conditions": [
            {
              "evaluator": {"type": "lt", "params": [100]},
              "operator": {"type": "and"},
              "query": {"params": ["A", "5m", "now"]},
              "reducer": {"type": "avg"}
            }
          ],
          "message": "Query throughput dropped below 100 QPS"
        }
      },
      {
        "id": 2,
        "title": "Query Latency (P50, P95, P99)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(query_latency_seconds_bucket[5m]))",
            "legendFormat": "P50"
          },
          {
            "expr": "histogram_quantile(0.95, rate(query_latency_seconds_bucket[5m]))",
            "legendFormat": "P95"
          },
          {
            "expr": "histogram_quantile(0.99, rate(query_latency_seconds_bucket[5m]))",
            "legendFormat": "P99"
          }
        ],
        "yAxisLabel": "Latency (seconds)",
        "thresholds": [
          {"value": 1.0, "color": "green"},
          {"value": 3.0, "color": "yellow"},
          {"value": 5.0, "color": "red"}
        ]
      },
      {
        "id": 3,
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(query_errors_total[5m]) / rate(events_processed_total[5m]) * 100",
            "legendFormat": "Error Rate %"
          }
        ],
        "yAxisLabel": "Error Rate (%)",
        "alert": {
          "conditions": [
            {
              "evaluator": {"type": "gt", "params": [1.0]},
              "message": "Error rate exceeded 1%"
            }
          ]
        }
      },
      {
        "id": 4,
        "title": "Kafka Consumer Lag",
        "type": "graph",
        "targets": [
          {
            "expr": "kafka_consumer_lag",
            "legendFormat": "{{topic}} - {{partition}}"
          }
        ],
        "yAxisLabel": "Messages Behind",
        "alert": {
          "conditions": [
            {
              "evaluator": {"type": "gt", "params": [100000]},
              "message": "Kafka consumer lag exceeded 100K messages"
            }
          ]
        }
      },
      {
        "id": 5,
        "title": "Data Freshness",
        "type": "stat",
        "targets": [
          {
            "expr": "time() - last_event_timestamp",
            "legendFormat": "Seconds since last event"
          }
        ],
        "thresholds": [
          {"value": 0, "color": "green"},
          {"value": 300, "color": "yellow"},
          {"value": 900, "color": "red"}
        ]
      },
      {
        "id": 6,
        "title": "Cache Hit Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "rate(cache_hits[5m]) / (rate(cache_hits[5m]) + rate(cache_misses[5m])) * 100",
            "legendFormat": "Hit Rate %"
          }
        ],
        "thresholds": [
          {"value": 0, "color": "red"},
          {"value": 50, "color": "yellow"},
          {"value": 70, "color": "green"}
        ]
      }
    ]
  }
}
```

**5. Data Quality Monitoring:**

```python
class DataQualityMonitor:
    """Monitor data quality metrics"""
    
    def __init__(self):
        self.quality_metrics = {
            'completeness': Gauge('data_completeness_percentage', 'Data completeness'),
            'accuracy': Gauge('data_accuracy_percentage', 'Data accuracy'),
            'freshness': Gauge('data_freshness_seconds', 'Data freshness in seconds'),
            'consistency': Gauge('data_consistency_percentage', 'Data consistency')
        }
    
    def check_completeness(self, expected_count: int, actual_count: int) -> float:
        """Check if all expected records are present"""
        completeness = (actual_count / expected_count) * 100
        self.quality_metrics['completeness'].set(completeness)
        
        if completeness < 95:
            self.alert("Data Completeness Low", f"Only {completeness:.1f}% of expected records")
        
        return completeness
    
    def check_freshness(self, table: str) -> float:
        """Check how fresh the data is"""
        query = f"""
        SELECT (now() - MAX(event_timestamp)) AS freshness_seconds
        FROM {table}
        """
        
        result = execute_query(query)
        freshness_seconds = result[0]['freshness_seconds']
        
        self.quality_metrics['freshness'].set(freshness_seconds)
        
        if freshness_seconds > 900:  # 15 minutes
            self.alert("Stale Data", f"Data is {freshness_seconds}s old (threshold: 900s)")
        
        return freshness_seconds
    
    def check_accuracy(self, table: str) -> float:
        """Check data accuracy using validation rules"""
        query = f"""
        SELECT
            COUNT(*) AS total_rows,
            SUM(CASE WHEN email LIKE '%@%' THEN 1 ELSE 0 END) AS valid_emails,
            SUM(CASE WHEN revenue >= 0 THEN 1 ELSE 0 END) AS valid_revenue,
            SUM(CASE WHEN age BETWEEN 0 AND 120 THEN 1 ELSE 0 END) AS valid_age
        FROM {table}
        """
        
        result = execute_query(query)[0]
        
        # Calculate accuracy as % of rows passing all validation rules
        valid_rows = min(
            result['valid_emails'],
            result['valid_revenue'],
            result['valid_age']
        )
        
        accuracy = (valid_rows / result['total_rows']) * 100
        self.quality_metrics['accuracy'].set(accuracy)
        
        if accuracy < 99:
            self.alert("Data Accuracy Low", f"Only {accuracy:.1f}% of rows are valid")
        
        return accuracy
    
    def check_consistency(self, source_table: str, target_table: str) -> float:
        """Check consistency between source and target"""
        # Compare aggregates between source and analytics table
        query_source = f"SELECT SUM(revenue) as total FROM {source_table}"
        query_target = f"SELECT SUM(revenue) as total FROM {target_table}"
        
        source_total = execute_query(query_source)[0]['total']
        target_total = execute_query(query_target)[0]['total']
        
        # Calculate consistency (how close are the totals)
        consistency = (1 - abs(source_total - target_total) / source_total) * 100
        self.quality_metrics['consistency'].set(consistency)
        
        if consistency < 99:
            self.alert(
                "Data Inconsistency",
                f"Source: ${source_total:,.0f}, Target: ${target_total:,.0f}"
            )
        
        return consistency
    
    def alert(self, title: str, message: str):
        """Send alert for data quality issue"""
        print(f"ALERT: {title} - {message}")
        # In production: Send to PagerDuty, Slack, etc.

# Usage
monitor = DataQualityMonitor()

# Run quality checks every 5 minutes
completeness = monitor.check_completeness(expected_count=1000000, actual_count=998500)
freshness = monitor.check_freshness('events')
accuracy = monitor.check_accuracy('events')
consistency = monitor.check_consistency('source_db.orders', 'analytics.orders')

print(f"""
Data Quality Report:
├─ Completeness: {completeness:.1f}%
├─ Freshness: {freshness}s
├─ Accuracy: {accuracy:.1f}%
└─ Consistency: {consistency:.1f}%
""")
```

---

### 🔴 Advanced Level: SLA Monitoring & Incident Response

**1. SLA Definition & Tracking:**

```python
class SLAMonitor:
    """Monitor and enforce SLAs"""
    
    def __init__(self):
        self.sla_targets = {
            'availability': {
                'target': 99.9,  # 99.9% uptime
                'allowed_downtime_per_month': 43.8  # minutes
            },
            'query_latency_p95': {
                'target': 2.0,  # 2 seconds
                'measurement_window': '5m'
            },
            'query_latency_p99': {
                'target': 5.0,  # 5 seconds
                'measurement_window': '5m'
            },
            'data_freshness': {
                'target': 300,  # 5 minutes
                'measurement_window': '15m'
            },
            'error_rate': {
                'target': 0.01,  # 1% max error rate
                'measurement_window': '5m'
            }
        }
        
        self.error_budget = self.calculate_error_budget()
    
    def calculate_error_budget(self) -> dict:
        """Calculate error budget for each SLA"""
        
        # Error budget = (100% - SLA) of time
        # For 99.9% SLA, error budget is 0.1% = 43.8 minutes/month
        
        minutes_per_month = 30 * 24 * 60  # 43,200 minutes
        
        availability_sla = self.sla_targets['availability']['target']
        error_budget_pct = 100 - availability_sla
        error_budget_minutes = minutes_per_month * (error_budget_pct / 100)
        
        return {
            'total_minutes': error_budget_minutes,
            'remaining_minutes': error_budget_minutes,  # Updated as outages occur
            'percentage_used': 0.0
        }
    
    def track_downtime(self, duration_minutes: float):
        """Track downtime and update error budget"""
        
        self.error_budget['remaining_minutes'] -= duration_minutes
        self.error_budget['percentage_used'] = (
            (self.error_budget['total_minutes'] - self.error_budget['remaining_minutes']) /
            self.error_budget['total_minutes'] * 100
        )
        
        if self.error_budget['remaining_minutes'] < 0:
            self.alert_sla_breach()
    
    def check_sla_compliance(self) -> dict:
        """Check current compliance against all SLAs"""
        
        compliance = {}
        
        # Check availability
        uptime_pct = self.get_uptime_percentage()
        compliance['availability'] = {
            'current': uptime_pct,
            'target': self.sla_targets['availability']['target'],
            'compliant': uptime_pct >= self.sla_targets['availability']['target']
        }
        
        # Check latency
        p95_latency = self.get_p95_latency()
        compliance['latency_p95'] = {
            'current': p95_latency,
            'target': self.sla_targets['query_latency_p95']['target'],
            'compliant': p95_latency <= self.sla_targets['query_latency_p95']['target']
        }
        
        # Check freshness
        freshness = self.get_data_freshness()
        compliance['freshness'] = {
            'current': freshness,
            'target': self.sla_targets['data_freshness']['target'],
            'compliant': freshness <= self.sla_targets['data_freshness']['target']
        }
        
        # Check error rate
        error_rate = self.get_error_rate()
        compliance['error_rate'] = {
            'current': error_rate,
            'target': self.sla_targets['error_rate']['target'],
            'compliant': error_rate <= self.sla_targets['error_rate']['target']
        }
        
        return compliance
    
    def generate_sla_report(self) -> str:
        """Generate monthly SLA report"""
        
        compliance = self.check_sla_compliance()
        
        report = f"""
SLA Compliance Report - {datetime.now().strftime('%B %Y')}

Availability:
├─ Target: {self.sla_targets['availability']['target']}%
├─ Actual: {compliance['availability']['current']:.3f}%
├─ Status: {'✅ PASS' if compliance['availability']['compliant'] else '❌ FAIL'}
└─ Error Budget: {self.error_budget['remaining_minutes']:.1f} min remaining ({self.error_budget['percentage_used']:.1f}% used)

Query Latency (P95):
├─ Target: {self.sla_targets['query_latency_p95']['target']}s
├─ Actual: {compliance['latency_p95']['current']:.2f}s
└─ Status: {'✅ PASS' if compliance['latency_p95']['compliant'] else '❌ FAIL'}

Data Freshness:
├─ Target: {self.sla_targets['data_freshness']['target']}s
├─ Actual: {compliance['freshness']['current']:.0f}s
└─ Status: {'✅ PASS' if compliance['freshness']['compliant'] else '❌ FAIL'}

Error Rate:
├─ Target: {self.sla_targets['error_rate']['target'] * 100:.2f}%
├─ Actual: {compliance['error_rate']['current'] * 100:.2f}%
└─ Status: {'✅ PASS' if compliance['error_rate']['compliant'] else '❌ FAIL'}

Overall SLA Compliance: {sum(1 for c in compliance.values() if c['compliant'])}/{len(compliance)}
        """
        
        return report
    
    def alert_sla_breach(self):
        """Alert when SLA is breached"""
        print("🚨 CRITICAL: SLA BREACH - Error budget exhausted!")
        # Send to PagerDuty, Slack, Email

# Example SLA Report Output:
# Availability: 99.95% (Target: 99.9%) ✅
# Error Budget: 21.6 min remaining (50.7% used)
# Query Latency P95: 1.8s (Target: 2.0s) ✅
# Data Freshness: 180s (Target: 300s) ✅
# Error Rate: 0.3% (Target: 1.0%) ✅
# Overall: 4/4 SLAs met
```

**2. Incident Response Playbook:**

```yaml
# incident_response_playbook.yml

incidents:
  # Incident: High latency
  - name: high_query_latency
    severity: P2  # Priority 2
    symptoms:
      - "P95 query latency > 5 seconds"
      - "Users reporting slow dashboards"
    
    investigation_steps:
      - step: 1
        action: "Check ClickHouse query log"
        command: "SELECT query, query_duration_ms FROM system.query_log ORDER BY query_duration_ms DESC LIMIT 10"
        expected: "Identify slow queries"
      
      - step: 2
        action: "Check for long-running queries"
        command: "SELECT query_id, user, elapsed FROM system.processes WHERE elapsed > 10"
        expected: "Find queries running > 10 seconds"
      
      - step: 3
        action: "Check CPU and memory usage"
        command: "top -b -n 1"
        expected: "Identify resource bottlenecks"
      
      - step: 4
        action: "Check cache hit rate"
        query: "rate(cache_hits[5m]) / (rate(cache_hits[5m]) + rate(cache_misses[5m]))"
        expected: "> 60% hit rate"
    
    remediation:
      - action: "Kill long-running queries"
        command: "KILL QUERY WHERE query_id = '<query_id>'"
      
      - action: "Scale up ClickHouse cluster"
        command: "kubectl scale deployment clickhouse --replicas=10"
      
      - action: "Clear and warm cache"
        command: "redis-cli FLUSHALL && python warm_cache.py"
      
      - action: "Enable query result caching"
        config: "enable_query_result_cache: true"
    
    escalation:
      - threshold: "15 minutes"
        action: "Page on-call engineer"
      
      - threshold: "30 minutes"
        action: "Escalate to senior engineer"
      
      - threshold: "60 minutes"
        action: "Engage incident commander"
  
  # Incident: Data pipeline stalled
  - name: data_pipeline_stalled
    severity: P1  # Priority 1 (Critical)
    symptoms:
      - "No new events in last 15 minutes"
      - "Kafka consumer lag increasing"
      - "Dashboard showing stale data"
    
    investigation_steps:
      - step: 1
        action: "Check Kafka broker health"
        command: "kafka-broker-api-versions.sh --bootstrap-server kafka:9092"
      
      - step: 2
        action: "Check consumer group lag"
        command: "kafka-consumer-groups.sh --bootstrap-server kafka:9092 --group analytics-consumer --describe"
      
      - step: 3
        action: "Check Flink job status"
        command: "curl http://flink-jobmanager:8081/jobs"
      
      - step: 4
        action: "Check ClickHouse ingestion"
        query: "SELECT COUNT(*) FROM events WHERE event_timestamp > now() - INTERVAL 15 MINUTE"
    
    remediation:
      - action: "Restart Kafka consumers"
        command: "kubectl rollout restart deployment/kafka-consumer"
      
      - action: "Restart Flink job"
        command: "flink cancel <job-id> && flink run analytics-job.jar"
      
      - action: "Check Kafka broker storage"
        command: "df -h /var/lib/kafka"
        note: "If disk full, increase retention or add storage"
    
    communication:
      - audience: "Internal users"
        channel: "Slack #analytics"
        message: "Analytics dashboards may show stale data. Investigating..."
      
      - audience: "Stakeholders"
        channel: "Email"
        message: "Real-time analytics experiencing delays. ETA: 15 minutes"

  # Incident: High error rate
  - name: high_error_rate
    severity: P2
    symptoms:
      - "Error rate > 5%"
      - "Users seeing error messages"
    
    investigation_steps:
      - step: 1
        action: "Check application logs"
        command: "kubectl logs -l app=analytics-api --tail=100"
      
      - step: 2
        action: "Check error types"
        query: "sum by (error_type) (rate(query_errors_total[5m]))"
      
      - step: 3
        action: "Check database connectivity"
        command: "nc -zv clickhouse 9000"
    
    remediation:
      - action: "Restart failed pods"
        command: "kubectl delete pod -l app=analytics-api --field-selector status.phase=Failed"
      
      - action: "Scale out to handle load"
        command: "kubectl scale deployment analytics-api --replicas=10"
```

**3. Google's SRE Approach to Analytics Monitoring:**

```text
Google Cloud Analytics Monitoring Strategy:

Scale:
├─ Data processed: 100+ PB per day
├─ Query volume: 10M+ queries per day
├─ Dashboards: 500K+ active dashboards
├─ SLA: 99.95% availability (21.6 min downtime/month)
└─ Global: 20+ regions

Monitoring Philosophy (Site Reliability Engineering):

1. Service Level Indicators (SLIs):
   ├─ Availability: % of successful requests
   ├─ Latency: P50, P95, P99 response times
   ├─ Throughput: Queries per second
   ├─ Correctness: Data accuracy and consistency
   └─ Freshness: Time lag for real-time data

2. Service Level Objectives (SLOs):
   ├─ Availability: 99.95% (measured over 30 days)
   ├─ Latency: P95 < 2s, P99 < 5s
   ├─ Error rate: < 0.1%
   ├─ Data freshness: < 5 minutes lag
   └─ Measurement window: Rolling 30 days

3. Error Budgets:
   ├─ Calculation: (100% - 99.95%) × 43,200 min/month = 21.6 min/month
   ├─ Purpose: Balance reliability with velocity
   ├─ Policy: If budget exhausted, freeze feature releases
   ├─ Allocation: 50% infrastructure, 30% planned changes, 20% reserve
   └─ Tracking: Real-time dashboard showing budget consumption

4. Monitoring Stack:
   ├─ Metrics: Borgmon (Google's Prometheus predecessor)
   ├─ Logs: Cloud Logging (10+ PB logs per day)
   ├─ Traces: Dapper (distributed tracing)
   ├─ Alerting: Alertmanager with PagerDuty integration
   └─ Dashboards: Custom SRE dashboards + Grafana

5. Alert Philosophy:
   ├─ Alert on symptoms, not causes
   ├─ Every alert must be actionable
   ├─ Reduce alert fatigue: Only page for P1/P0
   ├─ Alert on SLO burn rate, not absolute values
   └─ Example: Alert if error budget will be exhausted in < 7 days

6. On-Call Structure:
   ├─ Primary on-call: First responder
   ├─ Secondary on-call: Backup + escalation
   ├─ Shift duration: 1 week rotations
   ├─ Coverage: 24/7/365
   ├─ Escalation: Auto-escalate if no ack in 5 minutes
   └─ Compensation: Extra pay + time off after on-call

7. Incident Response:
   ├─ Severity levels: P0 (Critical), P1 (High), P2 (Medium), P3 (Low)
   ├─ Response time: P0 < 5 min, P1 < 15 min, P2 < 1 hour
   ├─ Incident commander: Coordinates response for P0/P1
   ├─ Post-mortems: Required for all P0/P1, blame-free culture
   └─ Action items: Tracked to completion, improve reliability

8. Key Metrics Tracked:
   ├─ MTTD (Mean Time To Detect): < 2 minutes
   ├─ MTTA (Mean Time To Acknowledge): < 5 minutes
   ├─ MTTR (Mean Time To Repair): < 15 minutes (P0/P1)
   ├─ MTBF (Mean Time Between Failures): > 30 days
   └─ Toil: < 50% of SRE time (rest on engineering)

9. Automation:
   ├─ Auto-remediation: 60% of incidents auto-resolved
   ├─ Auto-scaling: Predict load, scale proactively
   ├─ Chaos engineering: Regular failure injection tests
   ├─ Canary deployments: Gradual rollouts with automatic rollback
   └─ Load shedding: Drop low-priority queries under high load

Results:
├─ Availability: 99.97% (exceeds 99.95% SLO)
├─ MTTR: 12 minutes average (target: < 15 min)
├─ False positive alerts: < 5% (down from 30%)
├─ On-call pages: 2-3 per week (down from 10+)
├─ Incident rate: 1 P0 per month (down from 5)
└─ Customer satisfaction: 4.6/5.0

Cost of Monitoring:
├─ Monitoring infrastructure: $5M/year
├─ SRE team: 50 engineers × $300K = $15M/year
├─ Tools and licenses: $2M/year
├─ Total: $22M/year
├─ Service revenue: $500M/year
└─ Monitoring as % of revenue: 4.4%

Key Lessons:
├─ Measure what users care about (SLIs)
├─ Set realistic SLOs based on business needs
├─ Use error budgets to balance reliability and velocity
├─ Automate everything: remediation, scaling, alerting
├─ Blameless post-mortems improve reliability
└─ Invest in tooling: Good observability pays for itself
```

**4. Netflix's Observability at Scale:**

```text
Netflix Real-Time Analytics Observability:

Scale:
├─ Metrics: 2M+ metrics per second
├─ Logs: 500 GB per day
├─ Traces: 10M+ traces per day
├─ Dashboards: 10K+ Grafana dashboards
└─ Engineers using observability: 2,000+

Observability Stack:

1. Metrics (Atlas):
   ├─ Custom-built: Netflix Atlas (time-series database)
   ├─ Ingestion: 2M+ metrics per second
   ├─ Retention: 3 months at full resolution
   ├─ Query language: Netflix Stack Language (DSL)
   └─ Visualization: Grafana + custom dashboards

2. Logging (ELK Stack):
   ├─ Collection: Filebeat → Logstash
   ├─ Storage: Elasticsearch (500 GB/day)
   ├─ Visualization: Kibana
   ├─ Retention: 7 days hot, 30 days warm, 90 days cold
   └─ Search: Full-text search on all logs

3. Distributed Tracing (Zipkin):
   ├─ Instrumentation: All services auto-instrumented
   ├─ Sampling: 0.1% of requests (still 10M+ traces/day)
   ├─ Storage: Cassandra (distributed)
   ├─ Visualization: Zipkin UI
   └─ Insights: Identify bottlenecks in request path

4. Alerting (Atlas Alerting):
   ├─ Alert rules: 50K+ active alerts
   ├─ Delivery: PagerDuty, Slack, Email
   ├─ Smart grouping: Related alerts grouped together
   ├─ Auto-resolution: Alerts auto-resolve when issue clears
   └─ On-call: 24/7 coverage with follow-the-sun model

5. Chaos Engineering (Chaos Monkey):
   ├─ Randomly terminate instances
   ├─ Simulate AZ failures
   ├─ Inject latency and errors
   ├─ Test alerting and auto-recovery
   └─ Run continuously in production

Key Practices:

1. High-Cardinality Metrics:
   ├─ Tag everything: region, AZ, instance, service, customer
   ├─ Atlas handles 100+ tags per metric
   ├─ Enables detailed drill-down analysis
   └─ Example: "Show me latency for customer X in region Y"

2. Anomaly Detection:
   ├─ ML models detect unusual patterns
   ├─ Baseline: Learn normal behavior over 7 days
   ├─ Alert: When metric deviates > 3 standard deviations
   └─ Reduces false positives by 80%

3. Predictive Alerting:
   ├─ Don't wait for failure
   ├─ Predict based on trends: "Disk will be full in 4 hours"
   ├─ Proactive remediation: Scale before hitting limits
   └─ Result: 50% reduction in user-impacting incidents

4. Self-Service Observability:
   ├─ Every engineer can create dashboards
   ├─ Templates for common patterns
   ├─ No centralized monitoring team bottleneck
   └─ Culture: "You build it, you run it"

Results:
├─ Availability: 99.97%
├─ MTTD: 1.5 minutes average
├─ MTTR: 8 minutes average
├─ Alert false positive rate: < 3%
├─ Chaos tests: 1000+ per day, 99.9% success
└─ Observability cost: $10M/year (0.03% of revenue)

Innovation: Vizceral (Netflix's visualization tool)
├─ Real-time traffic visualization
├─ Shows all services and dependencies
├─ Highlights issues with color coding
├─ Open-sourced: github.com/Netflix/vizceral
└─ Used by engineering teams daily

Key Takeaways:
├─ Invest in observability early
├─ High-cardinality metrics enable detailed analysis
├─ Anomaly detection reduces alert fatigue
├─ Chaos engineering validates monitoring
├─ Self-service culture scales better than centralized team
└─ Observability is a competitive advantage
```

---

