# NOTIFICATION SYSTEM - Interview Quick Reference

**File Purpose:** Concise interview-day reference guide for multi-channel notification system design. Covers all critical talking points, architectural decisions, and key numbers. Review in 5-10 minutes before interviews.

**Last Updated:** October 2, 2025

---

## 🎯 Core Problem Statement

- **What:** Build a scalable multi-channel notification system for a social platform supporting Push, SMS, Email, and In-app notifications
- **Key Challenge:** Handle 1B notifications/day with 99% delivery rate, respect user preferences and quiet hours, manage priority queues, and track full delivery funnel
- **Scale:** 100M users, 1B notifications/day across 4 channels, 10× peak bursts

---

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| **Total Users** | 100M | Given assumption |
| **Notifications/Day** | 1B | Given assumption |
| **Average QPS** | 11,574 | 1B / 86,400 seconds |
| **Peak QPS** | 115,740 | 10× average burst |
| **Channel Mix** | Push 60%, In-app 20%, Email 10%, SMS 10% | Typical distribution |
| **Priority Mix** | Urgent 1%, High 9%, Normal 60%, Low 30% | Typical distribution |
| **Delivery Rate Target** | ≥99% | For push notifications |
| **Urgent Latency P95** | <1 second | End-to-end delivery |
| **Normal Latency P95** | <5 seconds | End-to-end delivery |
| **Active Device Tokens** | 200M | 2 devices per user avg |
| **Token Storage** | 40 GB | 200M × 200 bytes |
| **Event Log Storage/Day** | 200 GB | 1B × 200 bytes |
| **Payload Storage/Day** | 700 GB | 1B × 700 bytes avg |
| **Analytics Events/Day** | 2-3B | 2-3 per notification |
| **Batch Window (Urgent)** | 0ms | Immediate send |
| **Batch Window (Normal)** | 5 min | Reduce noise |
| **Batch Window (Low)** | 30 min | Cost optimization |
| **Redis Cache Hit Rate** | >95% | For preferences |

---

## 🏗️ High-Level Architecture

```text
[Event Producers: Microservices]
      |
      v
[Notification API (REST/gRPC)] <---> [Idempotency Store (Redis)]
      |
      +---> [Preference Service] <---> [Preference Cache (Redis 5min TTL)]
      |
      +---> [Template Service] <---> [Template Cache (Redis/CDN)]
      |
      v
[Orchestrator (Fan-out Logic)]
      |
      +---> [Rate Limiter (Redis Sliding Window)]
      +---> [Quiet Hours Checker]
      +---> [Anti-Spam Filter]
      |
      v
┌─────────────────────────────────────────────┐
│      PRIORITY QUEUES (Kafka/RabbitMQ)       │
│  Push.Urgent | Email.Urgent | SMS.Urgent    │
│  Push.High   | Email.High   | SMS.High      │
│  Push.Normal | Email.Normal | SMS.Normal    │
│  Push.Low    | Email.Low    | SMS.Low       │
│  InApp.Urgent/High/Normal/Low                │
└─────────────────────────────────────────────┘
      |
      v
┌─────────────────────────────────────────────┐
│         CHANNEL WORKERS (Scaled)            │
│  Push | SMS | Email | In-App                │
└─────────────────────────────────────────────┘
      |
      +---> [FCM (Android)]
      +---> [APNs (iOS)]
      +---> [SMS Provider (Twilio/etc)]
      +---> [Email Provider (SMTP/SendGrid)]
      +---> [In-App Store (DB)]
      |
      v
[Webhook Ingestion] <--- Provider callbacks
      |
      v
[Event Bus (Kafka)] ---> [Analytics Pipeline] ---> [ClickHouse/BigQuery]
                    ---> [Metrics] ---> [Prometheus/Grafana]
```

### Key Components

- **Notification API:** Ingests events, validates schema, enforces idempotency (Redis), returns 202 Accepted
- **Preference Service:** Manages user preferences (channels, categories, quiet hours), Redis cache 95%+ hit rate
- **Template Service:** Versioned templates by channel/locale, variable substitution, CDN-backed cache
- **Orchestrator:** Fan-out logic, applies preferences, quiet hours, priority assignment, batching windows
- **Fan-out Workers:** Partitioned by user_id hash, handles millions of recipients, batch enqueue to queues
- **Priority Queues:** Separate per `channel × priority` (16 queues total), Kafka/RabbitMQ, enables SLA differentiation
- **Rate Limiter:** Sliding window counters (user × channel × category), Redis, frequency capping
- **Channel Workers:** Consume from queues, render templates, send to providers, retry with exponential backoff
- **FCM/APNs:** HTTP/2 connections for APNs, OAuth 2.0 for FCM, connection pooling, 4KB payload limit
- **Token Management:** Device token registry, handles rotation/expiry, feedback processing, dedupe per user
- **Webhook Ingestion:** Processes delivery receipts, opens, clicks from providers, buffers via Kafka
- **Analytics Pipeline:** Stream processing, aggregates metrics, builds funnels, writes to OLAP

---

## 🔑 Critical Design Decisions

### Decision 1: Fan-out on Write vs Read

- **Choice:** Fan-out on write for push/SMS/email; hybrid for in-app
- **Why:** Push requires immediate token resolution and provider-specific formatting. In-app can aggregate at read time.
- **Trade-off:** Higher write-path complexity but better delivery latency

### Decision 2: Per-Channel Priority Queues

- **Choice:** 16 separate queues (4 channels × 4 priorities)
- **Why:** Isolates failures, enables differentiated SLAs, simplifies scaling and monitoring
- **Trade-off:** More queues to manage but better isolation and control

### Decision 3: Worker-Side Template Rendering

- **Choice:** Render templates in channel workers, not orchestrator
- **Why:** Reduces central hot spot, applies latest preferences at send time, per-channel optimizations
- **Trade-off:** Distributed rendering complexity but better scalability

### Decision 4: At-Least-Once with Idempotency

- **Choice:** At-least-once delivery with idempotency keys
- **Why:** Simpler than exactly-once, leverages provider dedupe (FCM/APNs), client-side dedupe for in-app
- **Trade-off:** Potential duplicates but practical at scale

### Decision 5: Quiet Hours Enforcement

- **Choice:** Queue with `not_before` timestamp; urgent bypasses with audit
- **Why:** Respects user preferences, maintains compliance, allows critical alerts through
- **Trade-off:** More complex scheduling but better UX

---

## 📝 API Design Cheat Sheet

### Send Notification (Producer API)

```http
POST /v1/notifications
Authorization: Bearer <token>
Content-Type: application/json

{
  "event_type": "new_comment",
  "audience": {"user_ids": ["u1", "u2"], "segment": "followers_of:post123"},
  "context": {"post_id": "post123", "commenter": "u9"},
  "priority": "normal",
  "channels": ["push", "in_app"],
  "idempotency_key": "uuid-1234"
}

Response: 202 Accepted
{
  "enqueue_id": "enq_abc123",
  "estimated_recipients": 2000000
}
```

### User Preferences

```http
GET /v1/users/{user_id}/notification-preferences
PUT /v1/users/{user_id}/notification-preferences

{
  "quiet_hours": {"start": "22:00", "end": "07:00", "tz": "America/Los_Angeles"},
  "channels": {
    "push": {"enabled": true},
    "email": {"enabled": false}
  },
  "categories": {
    "social": {"enabled": true, "max_per_hour": 5}
  }
}
```

### Device Token Registration

```http
POST /v1/devices/register
{
  "user_id": "u1",
  "platform": "ios",
  "token": "apns_token_here",
  "device_id": "d123"
}
```

---

## 🔢 Data Model Essentials

### Core Tables

```text
users
  - user_id (PK)

device_tokens
  - user_id, device_id (PK)
  - platform (ios|android)
  - token (unique, indexed)
  - last_seen_at, expires_at, is_valid

notification_preferences
  - user_id (PK)
  - channels: jsonb
  - categories: jsonb
  - quiet_hours: start, end, tz

templates
  - template_id (PK)
  - channel, locale, version
  - body, subject, variables: jsonb

notification_jobs
  - job_id (PK)
  - notification_id (FK)
  - user_id, channel
  - status (queued|sent|delivered|failed|suppressed)
  - attempts, next_retry_at

analytics_events (time-series/OLAP)
  - event_id, job_id, user_id, channel
  - event_type (sent|delivered|opened|clicked)
  - ts
```

### Key Indexes

- `device_tokens(token)` unique
- `device_tokens(user_id, is_valid)`
- `notification_jobs(user_id, channel, status)`
- `notification_jobs(status, next_retry_at)` for retry sweeps
- `analytics_events(channel, event_type, ts)` for dashboards

---

## 🚀 Scaling Strategy

### Horizontal Scaling

- **API Servers:** Stateless, scale behind load balancer, auto-scale on CPU/RPS
- **Fan-out Workers:** Partition by user_id hash, scale to 100+ instances for large broadcasts
- **Channel Workers:** Scale independently per channel based on queue depth, circuit breakers
- **Preference/Template Services:** Read-heavy, cache aggressively (Redis), read replicas

### Database Sharding

- **Shard notification_jobs by user_id:** Consistent hashing across 10-100 shards
- **Shard device_tokens by user_id:** Co-locate with user data
- **Analytics events:** Time-based partitioning in ClickHouse/BigQuery

### Caching Strategy

- **Preferences:** Redis 5-min TTL, 95%+ hit rate, invalidate on updates
- **Templates:** Redis/CDN, versioned, immutable, no expiry
- **Device tokens:** Batch lookups (100 users per query), cache per user

---

## ⚡ Performance Optimizations

### FCM (Firebase Cloud Messaging)

- Use HTTP v1 API (not legacy)
- OAuth 2.0 with service account credentials
- Connection pooling with keep-alive
- Batch sends (500 per request)
- Max payload: 4 KB

```json
{
  "message": {
    "token": "fcm_device_token",
    "notification": {"title": "...", "body": "..."},
    "data": {"post_id": "123", "action": "view_post"},
    "android": {
      "priority": "high",
      "ttl": "3600s",
      "notification": {"channel_id": "social_updates"}
    }
  }
}
```

### APNs (Apple Push Notification)

- HTTP/2 persistent connections
- Token-based auth (JWT, refresh every 50 min)
- Production: `api.push.apple.com:443`
- Max payload: 4 KB (5 KB VoIP)

```json
{
  "aps": {
    "alert": {"title": "...", "body": "..."},
    "badge": 1,
    "sound": "default",
    "category": "COMMENT_CATEGORY"
  },
  "post_id": "123"
}
```

**Headers:**

```http
apns-topic: com.yourapp.bundle
apns-priority: 10  (10=immediate, 5=throttled)
apns-expiration: 1735689600
apns-collapse-id: post123_comment
```

### Retry Configuration

| Channel | Max Attempts | Base Delay | Max Delay | Strategy |
|---------|--------------|------------|-----------|----------|
| Push | 3 | 2s | 5min | Exponential + jitter |
| Email | 5 | 30s | 30min | Exponential + jitter |
| SMS | 3 | 60s | 60min | Exponential + jitter |
| In-app | 5 | 1s | 10min | Exponential + jitter |

**Exponential Backoff Formula:**

```python
delay = min(base_delay * (2 ** (attempt - 1)), max_delay) + random(0, delay * 0.1)
```

**Error Classification:**

- **Retryable:** 429, 500/502/503/504, timeouts, DNS failures
- **Non-Retryable:** 400, 401/403 (after refresh), 404, 410, spam complaints

---

## 🎯 Batching Strategy

### Batching Windows by Priority

- **Urgent:** No batching (immediate send)
- **High:** Micro-batching (100ms window or 100 items)
- **Normal:** 5-minute batching window (or 1K items)
- **Low:** 30-minute batching window (or 5K items)

### User-Level Batching (Digest)

- Aggregate low-priority events per user
- Example: "5 new likes, 3 new comments"
- Reduces notification fatigue and SMS costs

---

## 📊 Monitoring & Alerts

### Key Metrics

**Throughput:**

- `notifications_enqueued_total` (by event_type, priority)
- `notifications_sent_total` (by channel, priority, status)
- `notifications_delivered_total` (by channel)
- `delivery_latency_seconds` (P50, P95, P99 by channel, priority)

**Errors:**

- `notifications_failed_total` (by channel, error_type)
- `retry_attempts_total` (by channel)
- `dead_letter_queue_size` (by channel)
- `invalid_tokens_total` (by channel)

**Queue Health:**

- `queue_depth` (by channel, priority)
- `consumer_lag_seconds` (by channel, priority)

**Provider Metrics:**

- `fcm_success_rate`, `fcm_latency_ms`
- `apns_success_rate`, `apns_latency_ms`
- `email_bounce_rate`, `email_complaint_rate`

### Critical Alerts

1. `notifications_sent_rate < baseline * 0.5` for 5 min → **Critical**
2. `delivery_latency_p95 > SLA` for 5 min → **Warning**
3. `queue_depth > 1M` for 10 min → **Warning**
4. `channel_error_rate > 5%` for 5 min → **Critical**
5. `invalid_token_rate > 20%` for 15 min → **Warning**

---

## 🛡️ Reliability & Fault Tolerance

### Failure Scenarios

**Scenario 1: FCM/APNs Down**

- Circuit breaker trips after N failures
- Queue jobs with extended retry windows
- Alert on-call, continue other channels

**Scenario 2: Database Primary Failure**

- Read replicas serve preferences (stale OK)
- Jobs buffer in Kafka (durable)
- Automatic failover, no notification loss

**Scenario 3: Queue Overflow**

- Backpressure to orchestrator
- Drop low-priority with logging
- Scale workers horizontally
- Alert on queue depth

**Scenario 4: Invalid Token Storm**

- Batch mark invalid tokens
- Async cleanup hourly
- Investigate registration flow

### Multi-Region Deployment

- **Regions:** US-East, US-West, EU-West
- **Active-active** for API and fan-out
- Regional priority queues
- Cross-region replication for preferences/templates
- GeoDNS routing, regional provider endpoints

---

## 🎤 Interview Talking Points

### Opening

"I'll design a notification system handling 1 billion notifications per day across 100 million users with 4 channels: Push, SMS, Email, and In-app. The key challenges are fan-out at scale, priority-driven delivery with 99% delivery rate, respecting user preferences and quiet hours, and tracking the full delivery funnel."

### Key Highlights

1. **Fan-out Strategy:** "For large broadcasts to millions of users, I'll use partitioned fan-out workers keyed by user_id hash. This allows us to scale horizontally and preserve per-user ordering. For extremely large fan-outs (10M+), we stage recipient IDs in blob storage and stream through workers."

2. **Priority Queues:** "I'm using 16 separate queues (4 channels × 4 priorities) to isolate failures and enable differentiated SLAs. Urgent notifications bypass batching and quiet hours with audit trails for compliance."

3. **Provider Integration:** "For push, I integrate with FCM (HTTP v1 API) and APNs (HTTP/2). Both use connection pooling and OAuth/JWT auth. I handle provider-specific errors: 410 means token invalid, 429 means backoff, 500/503 means retry."

4. **Retry Logic:** "Exponential backoff with jitter per channel. Push retries 3× with 2s base delay capped at 5 min. Email retries 5× with 30s base delay capped at 30 min. This prevents thundering herd and respects provider rate limits."

5. **User Preferences:** "Preferences are cached aggressively in Redis with 5-min TTL and 95%+ hit rate. Quiet hours are user-local timezone aware. Urgent notifications can bypass with audit logs for security/critical alerts."

6. **Batching:** "Batching windows vary by priority: urgent is immediate, normal is 5 min, low is 30 min. This reduces notification fatigue and SMS costs while maintaining urgency for critical notifications."

7. **Analytics:** "Full-funnel tracking: sent → delivered → opened → clicked. Events flow through Kafka to ClickHouse/BigQuery for OLAP. Dashboards show per-channel, per-priority metrics with engagement funnels."

### Closing

"This design scales to 1B notifications/day with 99% delivery rate, sub-second latency for urgent push, and comprehensive user controls. It balances reliability, performance, cost efficiency, and user experience at social media scale."

---

## 🔄 Alternatives Comparison

| System | Strengths | Weaknesses | Best For |
|--------|-----------|------------|----------|
| **Custom (This Design)** | Full control, optimized fan-out, multi-channel | Build/maintain complexity | High scale, custom needs |
| **AWS SNS + SQS** | Managed, reliable, easy integration | Less control, costs at scale | Quick start, moderate scale |
| **Twilio Notify** | Easy multi-channel API, managed | Limited customization, expensive | Small-medium scale |
| **Firebase Cloud Messaging** | Free, reliable push | Push-only, limited analytics | Mobile-only use cases |
| **OneSignal** | Full-featured, good analytics | Vendor lock-in, pricing tiers | Fast MVP, standard requirements |

---

## 🎯 Common Interview Questions

**Q: How do you handle hot users (celebrities with millions of followers)?**

A: Detect fan-out size during orchestration. For very large fan-outs (>1M), stage recipient IDs in blob storage, use dedicated high-capacity workers, rate-limit enqueue to queues, and consider progressive rollout over time windows.

**Q: How do you prevent notification spam?**

A: Multi-dimensional rate limiting (user × channel × category) with sliding window counters in Redis. Frequency capping per category (e.g., max 5 social notifications per hour). Quiet hours enforcement. User opt-out controls.

**Q: What if a user has multiple devices?**

A: Token registry stores multiple tokens per user with device_id. For push, send to all active tokens (dedupe on client). For in-app, store once and sync to all sessions. For SMS/Email, send to primary contact only.

**Q: How do you ensure 99% delivery rate?**

A: Retry with exponential backoff, provider fallbacks for SMS/Email, token validity checks, webhook ingestion for delivery receipts, comprehensive monitoring with alerts on delivery dips, circuit breakers for provider outages.

**Q: How do you handle quiet hours across timezones?**

A: Store user timezone in preferences, compute `not_before` timestamp in orchestrator based on local time, queue jobs with scheduled delivery, urgent notifications bypass with audit trail.

---

## ✅ Design Checklist

Before ending interview, ensure you've covered:

- [ ] Requirements clarification (100M users, 1B/day, 4 channels, priorities)
- [ ] Back-of-envelope calculations (11.6K avg QPS, 115K peak QPS)
- [ ] High-level architecture diagram with all major components
- [ ] API design (send notification, preferences, token registration)
- [ ] Data model (users, tokens, preferences, jobs, analytics)
- [ ] Fan-out strategy (partitioned workers, blob storage for large fan-outs)
- [ ] Priority queues (16 queues: 4 channels × 4 priorities)
- [ ] FCM/APNs integration specifics
- [ ] Retry logic with exponential backoff
- [ ] User preferences, quiet hours, rate limiting
- [ ] Batching strategy by priority
- [ ] Analytics pipeline (Kafka → ClickHouse)
- [ ] Monitoring metrics and alerts
- [ ] Failure scenarios and fault tolerance
- [ ] Multi-region deployment
- [ ] Trade-offs discussion

---

**Remember:** Focus on **priority-driven delivery**, **intelligent fan-out**, **provider-specific optimizations**, **user-centric preferences**, and **full-funnel analytics**. These are the differentiators of a production-grade notification system.


