# Contact Tracing App (Arogya Setu) - Quick Interview Reference

**Last Updated:** October 26, 2025

---

## Problem Statement

Design a contact tracing and health monitoring application for a country with 100M+ users during a pandemic. The system must detect proximity contacts via Bluetooth, notify users of potential exposure, track health status, and maintain strict privacy compliance.

---

## Key Numbers (Back-of-Envelope)

```text
Scale:
├─ Users: 100M registered, 50M DAU
├─ Encounters: 10/user/day = 500M encounters/day
├─ Positive Cases: 100,000 active (0.1% of population)
├─ Notifications: 100K exposures/day (20 contacts per positive case)
└─ QPS: 2,400 average, 24,000 peak

Storage:
├─ User Profiles: 100M × 241 bytes = 24 GB
├─ Infected Keys: 100K × 4.5 KB = 450 MB
├─ On-Device: 4.5 KB per user (14 days of encounters)
└─ Total Server: ~25 GB (with replication: 75 GB)

Bandwidth:
├─ Upload: 100 GB/day (health status + positive case uploads)
├─ Download: 56 TB/day (infected keys) → 560 GB/day with CDN optimization
└─ Notifications: 200 MB/day (FCM/APNS)

Cost (AWS):
├─ App Servers: $3,000/month
├─ Databases: $5,000/month
├─ Cache: $2,000/month
├─ CDN: $10,000/month
├─ Push: $5,000/month
└─ Total: ~$25,000/month = $0.00025/user/month
```

---

## Architecture Diagram (Simplified)

```text
┌─────────────────────────────────────────────────────────┐
│                    Mobile Clients                       │
│  ┌──────────────┐              ┌──────────────┐        │
│  │   iOS App    │              │ Android App  │        │
│  │ GAEN API     │              │ GAEN API     │        │
│  └──────┬───────┘              └──────┬───────┘        │
│         │                             │                 │
│         └─────────────┬───────────────┘                 │
└───────────────────────┼─────────────────────────────────┘
                        │
                   [Load Balancer]
                        │
            ┌───────────┴───────────┐
            │    API Gateway        │
            │  Auth + Rate Limiting │
            └───────────┬───────────┘
                        │
    ┌───────────────────┼───────────────────┐
    │                   │                   │
┌───▼────┐      ┌──────▼──────┐    ┌──────▼──────┐
│ User   │      │  Diagnosis  │    │Notification │
│Service │      │  Service    │    │  Service    │
└───┬────┘      └──────┬──────┘    └──────┬──────┘
    │                  │                   │
    │                  │                   │
┌───▼────┐      ┌──────▼──────┐    ┌──────▼──────┐
│  PG    │      │ Cassandra   │    │  FCM/APNS   │
│(Users) │      │(Inf. Keys)  │    │             │
└────────┘      └──────┬──────┘    └─────────────┘
                       │
                  ┌────▼────┐
                  │S3 → CDN │
                  │(Keys)   │
                  └─────────┘
```

---

## Core Components

### 1. Bluetooth Proximity Detection
- **Technology**: Bluetooth Low Energy (BLE 4.0+)
- **Range**: 2 meters (WHO guidelines)
- **Scan Frequency**: Every 5 seconds / 5 minutes (duty cycling)
- **RSSI Threshold**: -70 dBm ≈ 2m distance
- **Battery Drain**: <5% per day (optimized)
- **Platform**: Google/Apple Exposure Notification (GAEN) API

### 2. Privacy-Preserving Contact Matching
- **Protocol**: DP-3T or GAEN
- **Keys**: Daily TEK (Temporary Exposure Key)
- **IDs**: 144 RPIs/day (Rolling Proximity Identifiers, rotate every 10 min)
- **Storage**: Local on-device (14-day retention)
- **Matching**: Client-side (download infected keys, check locally)
- **Privacy**: Zero-knowledge server (doesn't know who met whom)

### 3. Health Status Tracking
- **Self-Assessment**: Symptom checker (14 symptoms)
- **Risk Scoring**: Weighted algorithm (0-100 scale)
- **Test Verification**: OTP from health authority
- **Vaccination**: Digital certificate storage
- **Compliance**: GDPR, HIPAA compliant (AES-256 encryption)

### 4. Location Hotspots
- **Opt-in**: User consent required
- **Geo-hashing**: 5-char geohash (2.4 km cells)
- **Privacy**: Differential privacy (Laplace noise)
- **Update Frequency**: Hourly
- **Visualization**: Heat map (red/orange/yellow/green)

---

## Database Design (Key Tables)

```sql
-- User Profiles (PostgreSQL)
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    device_id VARCHAR(64) UNIQUE,
    platform ENUM('ios', 'android'),
    last_active TIMESTAMP,
    INDEX idx_device (device_id)
);

-- Health Status (PostgreSQL)
CREATE TABLE health_status (
    user_id UUID PRIMARY KEY,
    status ENUM('safe', 'at_risk', 'symptomatic', 'positive'),
    risk_score INTEGER,
    symptoms_encrypted BYTEA,  -- AES-256
    test_date TIMESTAMP,
    INDEX idx_status (status)
);

-- Infected Keys (Cassandra)
CREATE TABLE infected_keys (
    tek BLOB,  -- 16 bytes
    upload_date DATE,
    transmission_risk INT,
    PRIMARY KEY ((upload_date), tek)
) WITH default_time_to_live = 1209600;  -- 14 days

-- Hotspots (PostgreSQL)
CREATE TABLE location_hotspots (
    geohash CHAR(5) PRIMARY KEY,
    positive_count INTEGER,
    risk_level ENUM('low', 'medium', 'high', 'critical'),
    INDEX idx_risk (risk_level)
);
```

---

## API Endpoints (RESTful)

```http
# User Registration
POST /api/v1/users/register
{
  "device_id": "abc123",
  "platform": "android"
}
→ Returns: {user_id, auth_token}

# Upload Infected Keys (Positive Test)
POST /api/v1/diagnosis/upload
Authorization: Bearer {token}
{
  "verification_code": "ABC123",
  "temporary_exposure_keys": [...]  -- 14 days of TEKs
}
→ Returns: {upload_id, keys_accepted}

# Download Infected Keys
GET /api/v1/diagnosis/keys?since=2025-10-12&region=IN
→ Returns: {keys: [...], next_cursor}
→ Caching: CDN, TTL 1 hour

# Health Self-Assessment
POST /api/v1/health/assess
{
  "symptoms": {fever: true, cough: "dry"},
  "age_group": "45-60"
}
→ Returns: {risk_score, risk_level, recommendation}

# Nearby Hotspots
GET /api/v1/hotspots/nearby?lat=28.6&lon=77.2&radius=5000
→ Returns: {hotspots: [{geohash, risk_level, case_count}]}

# Register Device Token (Push)
POST /api/v1/devices/register-token
{
  "token": "fcm_token",
  "platform": "android"
}
→ Returns: {status: "registered"}
```

---

## Critical Design Decisions

### 1. Centralized vs Decentralized?
**Choice: Decentralized (GAEN)**
- ✅ Privacy: Server never knows social graph
- ✅ Trust: Higher user adoption
- ❌ Tradeoff: Less data for public health authorities
- **Interview Tip**: Mention this tradeoff explicitly!

### 2. Bluetooth Only vs BT + GPS?
**Choice: Bluetooth Primary, GPS Opt-in**
- ✅ BT: Works indoors, privacy-friendly, low battery
- ✅ GPS: Enables hotspot mapping (opt-in only)
- ❌ GPS: Privacy concerns, high battery drain
- **Implementation**: BT mandatory, GPS optional with clear consent

### 3. Real-Time vs Batch Notifications?
**Choice: Hybrid**
- Exposure alerts: Real-time (critical, P0)
- Daily reminders: Batch hourly (P1)
- Updates: Batch daily (P3)
- **Optimization**: Adaptive batching based on queue size

---

## Scalability Strategies

### Database Sharding
- **User Data**: Shard by `user_id % 10` (10 shards)
- **Infected Keys**: Shard by `upload_date` (temporal)
- **Hotspots**: Shard by `geohash prefix` (spatial)

### Caching (3-Tier)
- **L1**: Application cache (in-memory, 60% hit rate)
- **L2**: Redis cluster (85% hit rate, 100 GB)
- **L3**: CDN (95% hit rate for infected keys)
- **Overall**: 99.7% cache hit rate

### Auto-Scaling
- **Kubernetes HPA**: Min 10, Max 200 pods
- **Trigger**: CPU >60%, Memory >70%
- **Scale Up**: 50% at a time (every 60s)
- **Scale Down**: 5 pods at a time (every 300s)

---

## Security & Privacy

### Encryption
- **At Rest**: AES-256-GCM (database, S3)
- **In Transit**: TLS 1.3 (all APIs)
- **On Device**: Secure Enclave/Keystore (keys)

### Privacy Techniques
- **Anonymization**: UUID-based, no PII
- **Differential Privacy**: Laplace noise on hotspot counts
- **Local Processing**: Contact matching on-device
- **Data Minimization**: Only collect essentials

### Compliance
- **GDPR**: Right to erasure, data portability
- **HIPAA**: PHI encryption, audit logging
- **Retention**: Auto-delete after 14-21 days

---

## Common Interview Pitfalls

❌ **Don't:**
- Forget to mention battery optimization (critical for adoption!)
- Ignore privacy (design will be rejected in interviews)
- Underestimate FCM/APNS rate limits
- Skip platform differences (iOS vs Android BLE)

✅ **Do:**
- Start with requirements (functional + non-functional)
- Discuss centralized vs decentralized tradeoff
- Calculate capacity with specific numbers
- Mention real-world examples (Arogya Setu, TraceTogether)
- Address failure modes (retry logic, fallbacks)

---

## Interview Talking Points

**Opening** (2 min):
> "Contact tracing apps detect proximity via Bluetooth to notify users of potential exposure. Key challenges are privacy, battery life, and scale. I'll design a privacy-preserving system using Google/Apple Exposure Notification API for 100M users."

**Requirements** (5 min):
- Functional: BT detection, contact matching, health tracking, notifications
- Non-Functional: 99.99% uptime, <100ms latency, <5% battery, GDPR compliant

**Architecture** (10 min):
- Mobile: BLE scanning, local encounter storage, GAEN API
- Backend: Microservices (User, Diagnosis, Notification, Hotspot)
- Data: PostgreSQL (users), Cassandra (keys), Redis (cache)

**Scale** (5 min):
- 100M users, 50M DAU, 2,400 QPS avg, 24,000 QPS peak
- 25 GB storage, 56 TB/day bandwidth (optimized with CDN)
- $25K/month cost

**Tradeoffs** (3 min):
- Privacy vs Public Health: Choose decentralized for user trust
- BT vs GPS: BT primary, GPS opt-in
- Real-time vs Batch: Hybrid approach

---

## Quick Facts

- **Arogya Setu**: 180M users (India), fastest app adoption in history
- **TraceTogether**: 4.5M users (Singapore), 80% penetration
- **Corona-Warn-App**: 30M users (Germany), open-source
- **GAEN Protocol**: Used by 50+ countries, 1B+ users globally

---

## Resources

- Full Design: `arogya_setu_system_design.md` (4,595 lines)
- DP-3T Whitepaper: https://github.com/DP-3T/documents
- GAEN Spec: Apple/Google Exposure Notification API
- Open Source: Corona-Warn-App (Germany), SwissCovid (Switzerland)

---

**Time:** Total ~25 minutes for full interview

**Next:** Practice explaining privacy-preserving cryptography (DP-3T) in simple terms!
