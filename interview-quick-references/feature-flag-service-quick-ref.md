# FEATURE FLAG & EXPERIMENTATION PLATFORM — Interview Quick Reference

**File Purpose:** Condensed day-of-interview reference for a LaunchDarkly-style feature flag and experimentation platform (control plane + data plane + analytics). Review in 5–10 minutes.

**Last Updated:** April 14, 2026

---

## Core Problem Statement

- **What:** Remote configuration for code paths (flags), deterministic targeting, real-time propagation to SDKs, optional A/B testing with exposure analytics.
- **Key challenge:** **100B+ evaluations/day** without one RPC per flag; **&lt;1s propagation** for kill switches; **enterprise** security (keys, audit, isolation).
- **Three planes:** Control (Postgres, IAM), Data (snapshots, Redis, object store, stream gateways), Analytics (Kafka → ClickHouse).

---

## Numbers That Matter

| Metric | Value | Notes |
|--------|-------|--------|
| Enterprise customers | 50,000 | Noisy-neighbor isolation |
| End users | 500M | Mostly client-side eval |
| Evaluations/day | 100B | ~1.16M/s avg, ~3.5M/s peak |
| Propagation SLO | &lt;1s to 99% SDKs | Kill switches drive this |
| Server eval p99 | &lt;50ms | Network + cache + rules |
| Local eval p99 | sub-ms | In-process after snapshot |
| Availability | 99.99% | ~52.6 min/year |

---

## High-Level Architecture

```text
[Admin UI / CI] --> [Config API] --> [PostgreSQL] (source of truth)
                         |
                    [Compiler] --> [Redis] + [Object store snapshots]
                         |
[Mobile/Web SDK] <---> [Stream gateways SSE/WS]
[Server SDK] ------> [Evaluation API] --> Redis

[SDKs] ---> [Kafka: exposure / change] ---> [ClickHouse]
```

### Components (talking points)

- **Compiler:** `flag_versions` JSON → compiled artifact; invalidates cache; writes immutable snapshot.
- **Stream gateway:** Long-lived connections; patch fan-out; scoped tokens.
- **Evaluation API:** Server-side-only paths; bulk evaluate; hides rules.
- **Kafka:** Decouple telemetry from user latency; partition by `tenant_id`.

---

## Flag Evaluation (Order)

```text
1. Archived / off? → default
2. Kill switch? → forced variation
3. Prerequisites OK?
4. Rules top-down → first match
5. Fallthrough / default
6. Percentage rollout → MurmurHash(userKey, flagKey, salt) → basis points
```

- **Sticky:** stable `user.key`, not ephemeral session (unless intentional).
- **Basis points:** 0–9999 for hundredths of a percent; avoid float bugs.

---

## Real-Time Distribution

| Transport | Use when |
|-----------|----------|
| SSE | Browser-friendly one-way push |
| WebSocket | Bidirectional, mobile efficiency |
| Polling | Fallback; restrictive proxies |

**Pattern:** Immutable snapshot URL (CDN) + incremental **patches** + **ETag**; jittered reconnect; debounce bulk edits (exception path for kill switch, bounded).

---

## SDK Modes

| Mode | Pros | Cons |
|------|------|------|
| Local eval | Sub-ms, scales with traffic | Limited rules on client |
| Server eval | Hides rules + PII resolution | Latency + service scale |

**Never** put **server SDK secret** in browser/mobile repos.

---

## Data Stores (Sound Bite)

- **PostgreSQL:** Flags, versions, segments metadata, RBAC, audit.
- **Redis:** Hot compiled snapshots, metadata, rate limits.
- **Object store:** Large versioned bundles; CDN-friendly URLs.
- **Kafka:** `flag.change`, `exposure.events`.
- **ClickHouse:** OLAP for experiments; dedupe by `nonce`.

---

## Security Checklist

- Per-**environment** keys; **rotate** with grace period.
- Audit: actor, diff, ticket, IP hash, timestamp.
- SOC 2 / GDPR: minimize PII in segments and logs; residency options.
- **Streaming:** short-lived tokens bound to SDK identity.

---

## Experiments (Minimum Viable Story)

- **Exposure** event: user saw treatment (`flag_key`, `variation`, `snapshotVersion`, `nonce`).
- **SRM:** chi-square on assignments vs expected split — auto-stop if broken.
- **Guardrails:** latency, errors, revenue bands — auto-pause / page humans.

---

## APIs to Name (15)

`POST/PATCH flags`, `GET flags`, `POST evaluate`, `POST evaluate/bulk`, `GET snapshots/latest`, `GET stream/sse`, `POST segments`, `POST segments/.../import`, `POST events/exposure`, `GET audit`, `POST flags/...:simulate`, `POST sdkKeys:rotate`, `POST flags/...:archive`, `GET healthz`

---

## Trade-Offs (One-Liners)

- **SSE vs WebSocket:** simplicity vs bidirectional efficiency.
- **Local vs server eval:** speed vs secrecy.
- **Sampling exposures:** cost vs statistical power (watch bias).
- **Strong consistency:** control plane yes; data plane eventually bounded.

---

## 45-Minute Interview Pacing

```text
0–5:  Scope (experiments? client eval? regions?)
5–10: NFR + numbers on whiteboard
10–20: Three-plane diagram + stores
20–32: Evaluation + streaming deep dive
32–40: Postgres/Redis/Kafka/ClickHouse
40–45: Security (keys, audit) + monitoring + trade-offs
```

---

## Lightning Questions (Cheat Sheet)

1. **Why not RPC per flag?** Bundled snapshot + local eval.
2. **Murmur vs SHA?** Speed for bucketing; HMAC for signing bundles.
3. **Redis down?** Object store snapshot + circuit breaker + stale mode.
4. **Exactly-once exposures?** At-least-once + dedupe `nonce`.
5. **Noisy tenant?** Dedicated Kafka partitions / rate limits / shards.

---

**Full design:** [feature_flag_service_system_design.md](../feature_flag_service_system_design.md) (~2,500+ lines — same educational style as `food_delivery_system_design.md`: collapsible interview Q&A per major section, worked capacity tables, request-path narratives, RBAC/STRIDE, SRM/guardrails, autoscaling/load-shedding ladders.)
