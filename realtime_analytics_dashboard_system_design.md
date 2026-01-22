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

