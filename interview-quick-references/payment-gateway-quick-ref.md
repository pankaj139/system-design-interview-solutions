# Payment Gateway - Principal Engineer Level - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a payment gateway processing 10M transactions/day with PCI DSS compliance and multi-currency support
- **Key Challenge:** 99.999% availability, <2s processing time, zero data loss, fraud detection, reconciliation
- **Scale:** 10M transactions/day ($1B volume), 100K merchants, 100+ currencies, 99.999% uptime

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Transactions | 10M | Given |
| Daily Volume | $1B | 10M × $100 average |
| Transactions/Second | 116 TPS (348 peak) | 10M / 86,400 (3x peak) |
| Storage (5 years) | 36.5 TB | 10M × 2 KB × 365 × 5 |
| Audit Logs (7 years) | 64 TB | 10M × 5 logs × 500 bytes × 365 × 7 |
| API Servers | 35 | 640 peak TPS ÷ 100 × 5x redundancy |
| Database Servers | 33 | Transaction (20) + Audit (12) + Token (9) |

## 🏗️ High-Level Architecture

```text
[Merchant/Customer] -> [Load Balancer (5x redundancy)] -> [API Gateway]
         |                        |                            |
         v                        v                            v
[Payment Service] -> [Tokenization] -> [Fraud Detection (ML)] -> [Auth Service]
         |                |                    |                       |
         v                v                    v                       v
[Card Processor] -> [PostgreSQL (Sharded)] -> [Vault (Encrypted)] -> [Redis Cache]
         |                |                    |                       |
         v                v                    v                       v
[Kafka Events] -> [Webhook Workers] -> [Reconciliation] -> [Settlement]
```

**Components:**
- Payment Service: Go, idempotency design, <2s processing
- Authorization: Separate auth and capture, 7-day expiry
- Tokenization: Vault encrypted storage, never store raw cards
- Fraud Detection: ML risk scoring (98% accuracy), 50ms latency
- Double-Entry Ledger: Immutable bookkeeping, daily reconciliation
- Multi-Currency: 100+ currencies, real-time forex, 0.5% spread

## 💾 Data Model (Essentials)

### Core Tables
- **transactions**: transaction_id (PK), idempotency_key (UNIQUE), merchant_id, amount, currency, status, auth_code
- **audit_logs**: log_id (PK), transaction_id, event_type, event_data (JSONB), created_at (WORM)
- **refunds**: refund_id (PK), transaction_id (FK), amount, status, reason
- **settlements**: settlement_id (PK), merchant_id (FK), total_amount, net_amount, settlement_date

### Advanced Patterns
- **Sharding**: By merchant_id (10 shards, 1M transactions each)
- **Partitioning**: Audit logs by date (monthly), transactions by merchant_id
- **Idempotency**: Redis SETNX for atomic check, 24h TTL, PostgreSQL backup (30d)
- **Double-Entry**: Every transaction = 2+ ledger entries (debit + credit), always balanced

### Security
- **Tokenization**: Vault encrypted, no raw card storage, PCI DSS compliant
- **Encryption**: TLS 1.3 (transit), AES-256 (rest), key rotation (90 days)
- **Audit**: Immutable logs, 7-year retention, WORM storage

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| POST | /payments/authorize | Authorize payment with fraud check |
| POST | /payments/{id}/capture | Capture authorized payment |
| POST | /payments/{id}/refund | Process full or partial refund |
| POST | /webhooks | Configure webhook endpoints |

## 🚀 Critical Talking Points (Principal Engineer Level)

### Idempotency Design
- **What:** Client-generated key prevents duplicate transactions, 24h TTL, Redis + PostgreSQL
- **Why:** Network failures, retries, timeouts - must handle safely without duplicate charges
- **Detail:** Redis SETNX for atomic check, return existing result if duplicate, <2ms check
- **Tradeoff:** Added complexity vs safety - critical for financial transactions
- **Alternative:** No idempotency (simple but dangerous, duplicate charges)

### Double-Entry Bookkeeping
- **What:** Every transaction creates debit + credit entries, sum debits = sum credits
- **Why:** Financial accuracy, reconciliation, auditability, compliance
- **Detail:** Immutable ledger, daily balance verification, automated reconciliation
- **Tradeoff:** Storage overhead vs accuracy - worth it for financial integrity
- **Alternative:** Single-entry (simpler but no balance verification, reconciliation issues)

### Fraud Detection with ML
- **What:** Multi-layer detection (rules + ML + network analysis), 98% accuracy, 0.08% false positives
- **Why:** Prevent fraud ($5M saved annually), protect merchants and customers
- **Detail:** XGBoost model, 200 features, 50ms latency, daily training
- **Tradeoff:** 50ms latency vs fraud prevention - worth $5M savings
- **Alternative:** Rules only (faster but 90% accuracy, higher fraud rate)

### Authorization vs Capture
- **What:** Separate auth (reserve funds) and capture (charge customer) steps, 7-day auth validity
- **Why:** Flexibility for order changes, delayed fulfillment, partial captures
- **Detail:** Auth holds funds, capture processes charge, auto-void after 7 days
- **Tradeoff:** Complexity vs flexibility - worth it for e-commerce use cases
- **Alternative:** Combined auth-capture (simpler but no flexibility)

### PCI DSS Compliance
- **What:** Never store raw cards, tokenization, encryption, network segmentation, audits
- **Why:** Legal requirement, customer trust, avoid penalties ($500K+ fines)
- **Detail:** Vault tokenization, AES-256 encryption, quarterly scans, annual pen tests
- **Tradeoff:** High compliance cost vs legal requirement - mandatory
- **Alternative:** Non-compliant (illegal, massive fines, business shutdown)

### Multi-Currency with Real-Time Forex
- **What:** Support 100+ currencies, real-time rates (30s updates), 0.5% spread
- **Why:** Global reach, DCC (Dynamic Currency Conversion), spread revenue
- **Detail:** Multiple forex providers, median rate calculation, cache 60s TTL
- **Tradeoff:** Forex risk vs global reach - hedging mitigates risk
- **Alternative:** Single currency (simpler but limited market)

### Reconciliation & Settlement
- **What:** Daily matching with processor reports, automated resolution, T+1 settlement
- **Why:** Ensure accuracy, detect discrepancies, timely merchant payouts
- **Detail:** 99.5% auto-match rate, 99% auto-resolution, batch settlement by currency
- **Tradeoff:** Complexity vs accuracy - 99.5% match rate justifies automation
- **Alternative:** Manual reconciliation (cheaper but error-prone, slow)

## ⚖️ Key Trade-Offs (Principal Engineer Analysis)

| Decision | Choice | Alternative | Principal Engineer Analysis |
|----------|--------|-------------|----------------------------|
| Idempotency | Mandatory with 24h TTL | Optional | Financial safety requires idempotency, prevent duplicate charges |
| Bookkeeping | Double-entry | Single-entry | Financial accuracy and reconciliation require double-entry |
| Fraud Detection | ML + Rules | Rules only | $5M annual savings justify 50ms latency overhead |
| Auth vs Capture | Separate | Combined | E-commerce flexibility justifies complexity |
| Tokenization | Vault (encrypted) | Database | PCI DSS Level 1 requires secure token storage |
| Currency Support | 100+ currencies | USD only | Global market access justifies forex complexity |
| Settlement | T+1 automated | Manual | Merchant satisfaction requires fast, accurate settlement |

## 🔥 Bottlenecks & Solutions (Principal Engineer Level)

### Payment Processor Timeouts
- **Root Cause:** Card networks timeout 1-2% of transactions, unclear status
- **Impact:** Customer charged but merchant not credited, manual reconciliation
- **Solutions:**
  - Async processing: Return "processing", poll every 2s, max 30s
  - Circuit breaker: Trip if >5% errors, route to backup processor
  - Idempotent retry: Safe retry with same key, exponential backoff
- **Monitoring:** Timeout rate, resolution time, circuit breaker trips

### Webhook Delivery Failures
- **Root Cause:** Merchant endpoint down 5% of time, webhooks lost
- **Impact:** Merchant unaware of payment status, manual follow-up
- **Solutions:**
  - Exponential backoff: 1m, 5m, 15m, 1h, 6h, 24h (6 retries)
  - Dead letter queue: Manual review and re-delivery
  - Webhook dashboard: Real-time status, manual retry
- **Monitoring:** Delivery success rate, retry count, DLQ depth

### Database Write Contention
- **Root Cause:** 10M transactions/day, single database bottleneck
- **Impact:** Slow inserts (100ms), potential timeouts during peak
- **Solutions:**
  - Sharding by merchant_id: 10 shards, 1M transactions each
  - Async audit logging: Kafka → DB, reduce write load
  - Read replicas: 1 primary + 3 replicas, 75% read reduction
  - Connection pooling: HikariCP 200 connections
- **Monitoring:** Write latency, CPU usage, connection pool utilization

### Reconciliation Discrepancies
- **Root Cause:** Timing differences, processor errors, network issues
- **Impact:** 0.5% mismatches (50K/day), manual investigation required
- **Solutions:**
  - Auto-resolution: Timing differences resolved automatically (99%)
  - Manual review queue: Complex cases flagged for team
  - Adjustment entries: Correct mismatches in ledger
  - Daily reports: Automated discrepancy detection
- **Monitoring:** Match rate, discrepancy count, resolution time

## 💡 Interview Tips (Principal Engineer Level)

- **Start Here:** "Let me clarify - we need 99.999% availability (5.26 min/year downtime) with PCI DSS Level 1 compliance"
- **Emphasize:** Idempotency design, double-entry bookkeeping, fraud detection, PCI DSS compliance, reconciliation
- **Be Ready For:** "How do you prevent duplicate charges?", "What if processor times out?", "How do you handle refunds?"
- **Don't Forget:** Security (PCI DSS, encryption, tokenization), compliance (SOC 2, ISO 27001), monitoring (SLA/SLO/SLI)

---

**Scale:** 10M transactions/day, $1B volume, 100K merchants, 100+ currencies, 99.999% uptime
**Key Tech:** PostgreSQL (sharded), Vault (tokenization), Redis, Kafka, XGBoost (fraud ML)
**Complexity:** ⭐⭐⭐⭐⭐ Principal Engineer Level
**Interview Time:** 60-90 minutes
