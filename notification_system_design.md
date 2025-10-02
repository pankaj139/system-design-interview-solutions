# NOTIFICATION SYSTEM DESIGN

## Multi-Channel Notification Platform

**Document Purpose:** This document describes the architecture, components, and operational strategies for a large-scale, multi-channel notification system for a social platform. It focuses on fan-out at scale, priority-driven delivery, user preferences, quiet hours, rate limiting, batching, analytics, and robust integration with push providers (FCM/APNs), SMS, Email, and In-app channels.

**Last Updated:** October 2, 2025

---

## TABLE OF CONTENTS

- [REQUIREMENTS & CLARIFICATION](#requirements--clarification)
  - [User Stories](#user-stories)
  - [Functional Requirements (MVP)](#functional-requirements-mvp)
  - [Non-Functional Requirements](#non-functional-requirements)
  - [Clarifying Questions & Assumptions](#clarifying-questions--assumptions)
  
- [BACK-OF-THE-ENVELOPE CALCULATIONS](#back-of-the-envelope-calculations)
  - [Traffic Estimates](#traffic-estimates)
  - [Storage Estimates](#storage-estimates)
  - [Bandwidth & Cost Estimates](#bandwidth--cost-estimates)
  
- [HIGH-LEVEL DESIGN](#high-level-design)
  - [Core Components](#core-components)
  - [Architecture Diagram](#architecture-diagram)
  - [Data Flow](#data-flow)
  
- [API DESIGN](#api-design)
  - [Producer API](#producer-api)
  - [Preference API](#preference-api)
  - [Token Management API](#token-management-api)
  - [Webhook Endpoints](#webhook-endpoints)
  
- [DATA MODEL & SCHEMA](#data-model--schema)
  - [Core Tables/Collections](#core-tablescollections)
  - [Indexes](#indexes)
  
- [DEEP DIVES](#deep-dives)
  - [Fan-out Strategy](#fan-out-strategy)
  - [Priority Queues & Scheduling](#priority-queues--scheduling)
  - [Retry Logic & Backoff](#retry-logic--backoff)
  - [Rate Limiting & Frequency Capping](#rate-limiting--frequency-capping)
  - [Template Management & Personalization](#template-management--personalization)
  - [Token Management (FCM/APNs)](#token-management-fcmapns)
  - [Analytics & Observability](#analytics--observability)
  
- [SECURITY & COMPLIANCE](#security--compliance)
- [TRADE-OFFS & DECISIONS](#trade-offs--decisions)
- [FUTURE ENHANCEMENTS](#future-enhancements)
- [SUMMARY](#summary)

---

## SECURITY & COMPLIANCE

- PII minimization; store only required data; encrypt at rest (KMS) and TLS in transit.
- Access controls: service-to-service auth (mTLS/JWT), RBAC for admin tools.
- Consent management and audit logs for opt-in/opt-out; comply with GDPR/CCPA/CAN-SPAM.
- Link tracking with privacy-safe parameters; allow user-level tracking opt-out.

## TRADE-OFFS & DECISIONS

- Fan-out on write vs read: choose write for timely push; in-app can combine with read-time aggregation.
- One global queue vs per-channel queues: per-channel to isolate failures and tune throughput.
- Central template rendering vs worker-side rendering: worker-side to apply latest prefs and reduce central hot spot.
- Exactly-once vs at-least-once: at-least-once with idempotency keys for practicality and scale.

## FUTURE ENHANCEMENTS

- ML-based send-time optimization within quiet hour boundaries.
- Adaptive channel selection based on historical engagement.
- Multi-tenant throttling policies and self-serve template editor with validation.
- Cross-region routing with latency-aware provider selection.

## SUMMARY

This design enables multi-channel notifications at 1B/day scale with priority-aware queues, robust fan-out, user-centric preferences and quiet hours, template-driven personalization, strong delivery guarantees with retries/backoff, comprehensive analytics, and anti-spam controls, integrating cleanly with FCM/APNs, SMS, Email, and In-app channels.

---

## DEEP DIVES

### Fan-out Strategy

- Hybrid fan-out: precompute recipient sets for large segments; stream-based expansion for dynamic audiences.
- Use partitioned workers keyed by `user_id` hash to preserve per-user ordering per channel.
- Batch enqueue to queues in chunks (e.g., 5K–20K) to avoid broker hot partitions.
- For extremely large sets (10M+), stage recipient IDs in a blob store and stream them through workers.

### Priority Queues & Scheduling

- Separate queues per `channel × priority` enable differentiated SLAs.
- Weighted fair scheduling across priorities with minimum bandwidth guarantees for urgent/high.
- Per-channel concurrency controls and circuit breakers to protect providers.

### Retry Logic & Backoff

- Exponential backoff with jitter: base 2^n seconds capped (e.g., 5m push/in-app, 30m email, 60m SMS).
- Classify errors: retryable (429/5xx/timeouts), non-retryable (4xx invalid tokens, spam complaints).
- Dead-letter queues (DLQ) after max attempts with reason code and diagnostics.

### Rate Limiting & Frequency Capping

- Sliding window counters per `user × channel × category` with configurable caps.
- Global and per-tenant quotas; spillover to lower priority or defer.
- Quiet hours enforcement: queue with `not_before` timestamps; urgent can bypass with audit.

### Template Management & Personalization

- Versioned templates with locale variants; control flags for A/B testing.
- Mustache/Handlebars-like variables; conditional sections; link tracking parameters.
- Rendering happens in channel workers to leverage latest preferences at send time.

### Token Management (FCM/APNs)

- Handle token rotation and invalidation; dedupe multiple devices per user.
- Feedback processing: remove invalid/expired tokens; backoff on provider throttling.
- For APNs: maintain HTTP/2 connections with connection pooling; for FCM: topic messaging when suitable.

### Analytics & Observability

- Emit events at key points: enqueued, queued, sent, delivered, opened, clicked, failed, suppressed.
- Funnel metrics by channel, priority, template, segment, region.
- Dashboards with percentiles for latency; alerts on delivery dips or provider error spikes.

---

## DATA MODEL & SCHEMA

### Core Tables/Collections

```text
users
  - user_id (PK)

device_tokens
  - user_id (PK part)
  - device_id (PK part)
  - platform (ios|android)
  - token (unique, indexed)
  - last_seen_at, expires_at, is_valid

notification_preferences
  - user_id (PK)
  - channels: jsonb
  - categories: jsonb
  - quiet_hours: start, end, tz
  - updated_at

templates
  - template_id (PK)
  - channel (push|email|sms|in_app)
  - locale (en-US, ...)
  - version
  - body, subject (for email), variables: jsonb
  - updated_at

notifications
  - notification_id (PK)
  - event_type, priority, channels: jsonb
  - audience_ref (segment or list id)
  - created_at

notification_jobs
  - job_id (PK)
  - notification_id (FK)
  - user_id
  - channel
  - status (queued|sent|delivered|failed|suppressed|bounced)
  - provider_message_id
  - attempts, next_retry_at
  - created_at, updated_at

analytics_events (time-series/OLAP)
  - event_id
  - job_id, user_id, channel, event_type (sent|delivered|opened|clicked|bounce|complaint)
  - ts
```

### Indexes

- `device_tokens(token)` unique, `device_tokens(user_id, is_valid)`
- `notification_jobs(user_id, channel, status)` for per-user draining
- `notification_jobs(status, next_retry_at)` for retry sweeps
- `analytics_events(channel, event_type, ts)` for dashboard queries

---

## API DESIGN

### Producer API

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
```

```http
202 Accepted
{
  "enqueue_id": "enq_abc123",
  "estimated_recipients": 2000000
}
```

### Preference API

```http
GET /v1/users/{user_id}/notification-preferences
```

```http
PUT /v1/users/{user_id}/notification-preferences
Content-Type: application/json

{
  "quiet_hours": {"start": "22:00", "end": "07:00", "tz": "America/Los_Angeles"},
  "channels": {
    "push": {"enabled": true},
    "email": {"enabled": false},
    "sms": {"enabled": false},
    "in_app": {"enabled": true}
  },
  "categories": {
    "marketing": {"enabled": false, "max_per_day": 1},
    "social": {"enabled": true, "max_per_hour": 5}
  }
}
```

### Token Management API

```http
POST /v1/devices/register
Content-Type: application/json

{
  "user_id": "u1",
  "platform": "ios", // or android
  "token": "apns_or_fcm_token",
  "device_id": "d123",
  "app_version": "9.2.1"
}
```

```http
POST /v1/devices/unregister
{
  "user_id": "u1",
  "device_id": "d123"
}
```

### Webhook Endpoints

```http
POST /v1/webhooks/push/receipt  // FCM/APNs delivery receipts
POST /v1/webhooks/email/events  // delivered/opened/clicked/spam/complaint
POST /v1/webhooks/sms/events    // delivered/failed
```

---

## HIGH-LEVEL DESIGN

### Core Components

- Notification API (Producer-facing): Ingests logical events (e.g., "new_comment"). Performs schema validation, idempotency, and lightweight enrichment.
- Preference Service: Stores per-user per-channel/category preferences, quiet hours, locale. Cached aggressively (Redis) with write-through to DB.
- Template Service: Manages templates by channel/locale/variant with variables and conditional blocks. Returns compiled content.
- Orchestrator: Determines recipients, channels, priority, batching windows; emits channel-specific jobs to priority queues.
- Fan-out Service: Expands to millions of recipients efficiently using batched queue writes and partitioned fan-out workers.
- Priority Queues: Separate queues per channel × priority (urgent/high/normal/low) with quotas and scheduling.
- Channel Workers: Push (FCM/APNs), SMS, Email, In-app workers with retry, backoff, and provider failover.
- Token Service: Manages device tokens (registration, expiry, invalidation, dedupe per user/device).
- Rate Limiter & Anti-Spam: Multi-dimensional limits (user, channel, category) and frequency capping with sliding window.
- Event Bus: Internal bus (e.g., Kafka) for events: enqueued, sent, delivered, opened, clicked, failed.
- Analytics Pipeline: Stream processing to aggregate metrics, build funnels, and power dashboards (OLAP/ClickHouse/BigQuery).
- Webhook Ingestion: Handles FCM/APNs feedback, Email/SMS provider webhooks; updates delivery/open/click status.
- Storage: Relational for user prefs/templates; NoSQL/time-series for events; Redis for hot caches; S3/GCS for archives.

### Architecture Diagram

```text
[Producers]
   |
   v
[Notification API] --(read)-> [Preference Cache/DB]
   |--(read)-> [Template Service]
   v
[Orchestrator] --(fan-out)-> [Priority Queues (per channel×priority)]
                         |-> [Rate Limiter]
                         |-> [Batcher]
   v
[Channel Workers]
   |--> Push: [FCM/APNs]
   |--> SMS: [SMS Aggregators]
   |--> Email: [SMTP/Email API]
   |--> In-app: [Notification Store]
   v
[Webhook Ingestion] -> [Event Bus] -> [Analytics/OLAP]
```

### Data Flow

1. Producer posts event to Notification API with event type and audience.
2. Orchestrator loads preferences, applies quiet hours, maps to channels and priority.
3. Fan-out expands recipients using partitioned workers; dedupes and applies per-user limits.
4. Jobs are placed into channel×priority queues; scheduler drains with quotas.
5. Channel workers personalize via Template Service and send to providers with retry/backoff.
6. Webhooks update status; events flow to analytics for delivery/open/click funnels.

---

## BACK-OF-THE-ENVELOPE CALCULATIONS

### Traffic Estimates

```text
Daily notifications: 1,000,000,000/day
Average RPS: ~11,574 req/s (1e9 / 86,400)
Peak RPS (10× bursts): ~115,740 req/s
Channel mix (example): Push 60%, In-app 20%, Email 10%, SMS 10%
Priority mix (example): Urgent 1%, High 9%, Normal 60%, Low 30%
```

### Storage Estimates

```text
Event log (enqueue): 1B/day, 200 bytes/event avg → ~200 GB/day (~6 TB/mo)
Notification payload archive (personalized body + metadata):
  - Push/In-app avg 500 B, Email 2 KB, SMS 160 B
  - Weighted avg ≈ ~700 B → ~700 GB/day (~21 TB/mo)
Delivery logs + analytics events (sent, delivered, open, click):
  - 2–3 events per notification avg, 150 B/event → ~300–450 GB/day
Token registry (active devices):
  - Assume 200M tokens (multiple devices per user), 200 B/token → ~40 GB
```

### Bandwidth & Cost Estimates

```text
Outbound provider calls:
  - Push: small JSON payloads via FCM/APNs; ~600 MB–1.2 TB/day
  - Email: SMTP/API with body + assets; depends on template; assume ~200–400 GB/day
  - SMS: API JSON; payload negligible, cost dominant
Provider costs (very rough):
  - Push: mostly free (infra costs dominate)
  - Email: $0.10–$1.00 per 1K emails
  - SMS: $0.005–$0.05 per SMS (region-dependent) → batching/frequency capping critical
```

---

## REQUIREMENTS & CLARIFICATION

### User Stories

- As a user, I want to receive notifications via Push, SMS, Email, and In-app based on my preferences.
- As a user, I want quiet hours so notifications are suppressed or delayed during certain times.
- As a user, I want to opt out per channel and per category (e.g., marketing vs transactional).
- As a system, I want to deliver urgent notifications immediately, ahead of normal and low priority items.
- As a system, I want to batch low-priority notifications to reduce noise and cost.
- As a system, I want reliable delivery with retries and backoff when providers fail.
- As a system, I want analytics on sent, delivered, opened, and clicked events.

### Functional Requirements (MVP)

- Channels: Push (FCM/APNs), SMS, Email, In-app.
- Scale: 100M users; 1B notifications/day.
  - Average ≈ 11.6K notifications/sec; plan for 10× peak.
- Priority levels: urgent, high, normal, low with separate queues and SLAs.
- User preferences: per-channel, per-category, per-frequency; quiet hours.
- Rate limiting: per-user, per-channel, per-category; anti-spam and frequency capping.
- Batching: aggregate low-priority notifications by user/channel when applicable.
- Personalization & templates: per-locale, per-variant with variable substitution.
- Token management: device token registration/rotation/expiry handling.
- Delivery tracking: provider response status; webhook ingestion for delivery/open/click.
- Retry & backoff: exponential backoff with jitter; max attempts per channel.
- Idempotency: dedupe to avoid duplicates on retries or fan-out races.

### Non-Functional Requirements

- Reliability: ≥99% delivery rate for push notifications measured end-to-end.
- Availability: 99.99% for API and enqueue path; channel workers can degrade.
- Latency Targets:
  - Urgent: P95 < 1s end-to-end for push/in-app; SMS/Email best-effort provider-dependent.
  - High/Normal: P95 < 5s; Low: best-effort with batching windows (e.g., 5–30 min).
- Cost efficiency: minimize SMS/Email costs via batching and channel fallback policies.
- Security & Privacy: data minimization, encryption in transit and at rest.
- Observability: per-channel and per-priority metrics, traces, and dashboards.

### Clarifying Questions & Assumptions

- Assume multi-region active-active with regional fan-out; cross-region spillover on failover.
- Assume push providers: FCM for Android, APNs for iOS; SMTP for Email; multiple SMS aggregators.
- Assume in-app notifications stored server-side and synced per session.
- Assume default quiet hours user-local time with overrides; urgent bypasses with audit.
- Assume consent and legal compliance (CAN-SPAM/GDPR/DSAR) with audit trails.
- Assume event producers publish logical events; Notification Orchestrator handles fan-out and personalization.


