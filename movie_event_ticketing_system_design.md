# Movie & Event Ticketing System Design (BookMyShow-like)

**Difficulty Level:** ⭐⭐⭐⭐ Very Hard  
**Tags:** `E-commerce`, `Real-time`, `Distributed Systems`, `High Concurrency`, `Low Latency`, `Caching`, `Search`, `Messaging`, `Geospatial`, `High Availability`, `Idempotency`

**File Purpose:** Interactive, multi-level learning resource for designing an online movie and live-event ticketing platform. This guide teaches you to handle peak flash traffic during blockbuster releases, seat-level inventory without double-booking, payment-integrated booking flows, and multi-city discovery at 99.95%+ availability with seat-selection latency under 200ms at p99.

**Author:** System Design Documentation  
**Created:** April 14, 2026  
**Last Updated:** April 14, 2026  
**Recent Updates:** Aligned depth with `food_delivery_system_design.md`: ecosystem diagram, long-form user stories, CAP table, step-by-step capacity math, consolidated DB/API chapter in §5, **🎯 Interview Questions** with `<details>` answers per major section, Section 15 question bank + mock rubric, cross-refs and next steps in §16

---

## 🎓 Welcome to Movie & Event Ticketing System Design!

### What You're Going to Build

Imagine building **BookMyShow** or **Fandango**: users browse movies and events by city, pick a showtime, tap seats on an interactive hall map, pay in seconds, and receive a QR ticket on their phone — while thousands of others compete for the same seats during a blockbuster opening night.

By the end of this learning journey, you'll understand how to design a production-grade ticketing platform that:

- Serves **50M+ monthly active users** browsing and **5–20M bookings per month** (scale tuned for interview depth)
- Handles **10,000–50,000 concurrent seat operations** during peak drops without selling one seat twice
- Completes **browse → seat map → pay** flows in **&lt;200ms p99** for cached reads and **&lt;3s p99** for checkout (payment network bound)
- Operates at **99.95% availability** (**~4.38 hours downtime per year**) for discovery and **strong correctness guarantees** for inventory and payments
- Covers **movies, plays, sports, and concerts** with unified inventory, scheduling, and partner (cinema/exhibitor) integrations

### 📚 Your Learning Path

This course is designed for three learning levels:

```text
🟢 BEGINNER LEVEL (5-7 hours)
├─ Learn why seat booking is a concurrency nightmare
├─ Understand showtimes, venues, and digital tickets
└─ Perfect for: First system design project

🟡 INTERMEDIATE LEVEL (7-9 hours)
├─ Interview frameworks: locking, idempotency, search
├─ Capacity estimation and API design under load
└─ Perfect for: FAANG-style interviews

🔴 ADVANCED LEVEL (10-14 hours)
├─ Production patterns: anti-abuse, PCI scope, multi-tenant exhibitors
├─ Failure modes: payment hangs, stale caches, clock skew
└─ Perfect for: Senior/staff-level discussions
```

### 🎯 Prerequisites

**For Beginners:** Basic web and mobile app concepts; what a database row is; no distributed systems required.

**For Intermediate:** HTTP, REST, relational databases, Redis-style caching, basic message queues.

**For Advanced:** Distributed transactions, sagas, consistency models, payment webhooks, observability stacks.

### 📊 What Makes This Learning Experience Unique

Each section follows the same **proven learning pattern** used in `food_delivery_system_design.md` (see also `EDUCATIONAL_TEMPLATE_GUIDE.md`):

1. **What You'll Learn** — Clear learning objectives for the section  
2. **Why This Matters** — Why interviewers and production care  
3. **Multi-level content** — 🟢 Beginner analogies, 🟡 Intermediate frameworks, 🔴 Advanced production angles  
4. **Extended detail** — Schemas, numbers, API/error contracts where helpful  
5. **Real-world examples** — BookMyShow, Fandango, Ticketmaster-scale patterns  
6. **Interview Questions** (selected sections) — `<details>` collapsible answers, same spirit as the food-delivery doc  
7. **Think About It / Key Takeaways / Practice Exercise** — Reinforcement  

**Estimated reading time (rough):** 🟢 **8–10 h** full depth | 🟡 **5–7 h** interview-focused | 🔴 **4–6 h** advanced-only skimming.

Real-world anchors include **BookMyShow, Fandango, Ticketmaster, Atom Tickets**, and regional apps.

💡 **Pro Tip:** In interviews, always separate **inventory correctness** (no double booking) from **read latency** (fast browse). The hardest part is the former under spike load.

---

## Table of Contents

Design pattern follows the **depth and pacing** of `food_delivery_system_design.md`: long-form 🟢/🟡/🔴 levels, worked math, **Interview Questions** with collapsible answers in selected sections, and extended tables. Sections below map to the **educational template** (Section 3 onward).

- [Section 3: Understanding Requirements](#section-3-understanding-requirements)
- [Section 4: Capacity Planning](#section-4-capacity-planning)
- [Section 5: High-Level Architecture](#section-5-high-level-architecture)
- [Section 6: Seat Inventory, Holds & Concurrency Control](#section-6-seat-inventory-holds--concurrency-control)
- [Section 7: Booking Flow, Payments & Order Lifecycle](#section-7-booking-flow-payments--order-lifecycle)
- [Section 8: Discovery, Search & Recommendations](#section-8-discovery-search--recommendations)
- [Section 9: Venues, Geo, Show Scheduling & Partner Integration](#section-9-venues-geo-show-scheduling--partner-integration)
- [Section 10: Digital Tickets, QR Validation & Anti-Fraud](#section-10-digital-tickets-qr-validation--anti-fraud)
- [Section 11: Scalability & Performance](#section-11-scalability--performance)
- [Section 12: Security Considerations](#section-12-security-considerations)
- [Section 13: Monitoring & Observability](#section-13-monitoring--observability)
- [Section 14: Trade-Offs & Design Decisions](#section-14-trade-offs--design-decisions)
- [Section 15: Interview Preparation](#section-15-interview-preparation)
- [Section 16: Putting It All Together](#section-16-putting-it-all-together)
- [Concept primer (terms explained)](#concept-primer-subsections-for-terms-used-in-this-document)

---

## Concept primer: subsections for terms used in this document

Whenever later sections use a specialized idea **without** stopping for a full tutorial, you can jump here for a short definition, then return to the section. Each **###** below is one concept.

**Convention:** Under **Why This Matters** in Sections **3–16**, a **#### Concepts in this section (primer index)** block lists terms used there and points back here. Sections may still say “see **Concept primer**” inline instead of repeating a full definition. For a term that appears only once, we sometimes add a local **#### Concept:** note in that section.

**Tip:** In GitHub-style viewers, open the **Table of Contents** and use **Find in page** (`Ctrl/Cmd+F`) on the concept name — headings here match the bold lead-in (e.g., `Redis SET NX`).

### Concept: MAU, DAU, and traffic units

- **MAU (Monthly Active Users):** distinct users who opened the app/site at least once in a month — a scale headline, not a QPS number.  
- **DAU (Daily Active Users):** same idea for one day.  
- **QPS / RPS:** queries or requests **per second** hitting your APIs — always ask “average vs peak vs hot-key spike.”

### Concept: Latency percentiles (p50, p99)

- **p99 latency:** 99% of requests complete faster than this value; the slowest 1% are worse.  
- **Why p99:** averages hide bad tail; users remember the slow checkout, not the mean.

### Concept: Availability (e.g., 99.95%)

- **99.95%** availability means roughly **4.38 hours downtime per year** (365.25 × 24 × 0.0005).  
- **Important:** “available” must be defined per surface (browse vs pay) — you may degrade browse differently than booking.

### Concept: B2B2C marketplace

- **Business–to–business–to–consumer:** your company sells **through** partners (cinemas) to end users.  
- Implication: **two inventories of truth** can exist (your DB vs partner POS) unless you contract otherwise.

### Concept: Showtime, venue, hall layout

- **Venue:** physical cinema or stadium.  
- **Hall / screen:** one room with a fixed seat map.  
- **Showtime:** a specific **movie (or event) + venue + hall + start time** — the unit customers book against.  
- **Hall layout version:** immutable snapshot of seat labels and geometry; showtimes **point** to a version so past sales stay valid if the map changes later.

### Concept: Hold vs sold (inventory states)

- **Hold:** temporary reservation (usually **Redis**, short TTL) — not money committed.  
- **Sold:** durable record (usually **PostgreSQL**) after successful payment + commit — legally and operationally “yours.”

### Concept: Idempotency and Idempotency-Key

- **Idempotent operation:** repeating it with the same logical intent does **not** change the outcome beyond the first success (e.g., no second order).  
- **Idempotency-Key:** client-supplied unique token (UUID) on **POST**; server stores `key → result` for a TTL so **retries** are safe after network timeouts.

### Concept: Redis SET NX and EX (or PX)

- **SET key value NX:** set only if **not** present — two clients racing: one wins, one gets “not set.”  
- **EX seconds (or PX ms):** expiry — key disappears automatically (used for **hold TTL**).  
- Together: **cheap atomic “claim this seat”** without long DB locks.

### Concept: TTL (time to live)

- A key or cache entry **auto-deletes** after a duration.  
- Used so **abandoned holds** free seats without a cleanup cron being perfectly timely.

### Concept: Redis Lua scripts

- **Lua** runs **atomically** on the Redis server for all operations inside one script against keys in the **same slot** (Cluster).  
- Used to **hold multiple seats at once** without another client sneaking between your per-seat SETs.

### Concept: Redis Cluster hash tags

- Keys like `hold:{st:4412}:F12` use `{st:4412}` so all seats for one showtime land in **one hash slot** — required so **Lua** can touch them in one atomic script.

### Concept: Hot key / hot partition

- **Hot key:** one logical key (e.g., one `showtime_id`) receives a huge fraction of traffic — becomes a bottleneck on a single CPU/thread or shard.  
- **Mitigation:** queues, admission control, splitting design (rare for seat maps), or accepting regional throttling.

### Concept: PostgreSQL UNIQUE constraint

- Declares **at most one row** for a combination of columns, e.g. `(showtime_id, seat_label)`.  
- **INSERT** that duplicates fails — database-enforced **no double booking** for sold seats.

### Concept: ACID transactions

- **Atomicity:** all writes in a transaction commit or none.  
- **Consistency:** constraints still hold after commit.  
- **Isolation:** concurrent transactions don’t see half-written states (depending on isolation level).  
- **Durability:** committed data survives crashes.  
- Used for **orders + seat_sales** together on payment confirmation.

### Concept: Read replica lag

- **Replicas** copy data from the **primary** asynchronously — usually milliseconds to seconds behind.  
- **Risk:** reading “is seat free?” from a replica might be **stale**; **writes** that decide correctness should read **primary** or a dedicated authoritative path.

### Concept: CAP theorem (practical interview use)

- When a **partition** (network split) happens, you cannot maximize **C**onsistency, **A**vailability, and **P**artition tolerance all three in the naive sense.  
- **Ticketing:** money + seats → bias **CP** on booking path; search can be **AP**-ish (eventually fresh).

### Concept: Eventual consistency

- The system **converges** to the correct state if you stop writes; temporary staleness is OK.  
- Fine for **search index**; **not** for “seat sold twice.”

### Concept: Saga and compensation

- **Saga:** multi-step business process with **local transactions** and **compensating actions** (e.g., refund if ticket commit fails).  
- Unlike **2PC**, no global lock across services — fits external PSP and long timeouts.

### Concept: Two-phase commit (2PC) — why we avoid it with PSPs

- **2PC:** all participants vote commit, then coordinator commits — tight coupling.  
- **PSP** is not under your control; holding locks across HTTP calls is fragile — prefer **saga + idempotency + reconciliation**.

### Concept: PSP (payment service provider)

- Third party (Stripe, Razorpay, etc.) that **tokenizes cards**, runs **3DS**, and settles money.  
- You integrate via **API + webhooks**; **PCI scope** shrinks if PAN never touches your servers.

### Concept: Webhook

- **HTTP callback** from PSP to your backend when an event happens (`charge.succeeded`).  
- Usually **at-least-once** delivery — you must **dedupe** by `event_id`.

### Concept: Payment intent vs charge

- Varies by PSP naming; generally **intent** = customer payment session; **charge** = captured money movement record used in **reconciliation** and disputes.

### Concept: Outbox pattern

- In the **same DB transaction** as your business commit, insert a row into **outbox_events**; a **publisher** reads and sends to Kafka — avoids “committed order but never emitted event.”

### Concept: Apache Kafka (topics, partitions, lag)

- **Topic:** stream of messages (e.g., `order_events`).  
- **Partition:** ordered log shard; **key** chooses partition (e.g., `order_id` keeps ordering per order).  
- **Consumer lag:** how far behind consumers are — high lag means delayed emails/tickets.

### Concept: Dead-letter queue (DLQ)

- A **side stream** for messages that failed processing too many times — needs **manual or automated replay** after fixing bugs.

### Concept: OpenSearch / Elasticsearch

- **Search engine** with inverted indexes for **full-text** and facets — great for titles, cast, genres.  
- **Not** the system of record for seat inventory.

### Concept: CDC (change data capture)

- Captures **row changes** from OLTP DB (Postgres) and feeds **downstream** (search index, warehouse).  
- Tools: **Debezium**, logical replication, etc.

### Concept: CDN (content delivery network)

- **Edge caches** for static assets (posters, trailers) close to users — reduces origin load and latency.

### Concept: Cache stampede / thundering herd

- Many clients miss cache **at once** and hammer origin.  
- **Mitigations:** single-flight, request coalescing, **jitter** on TTL, early refresh.

### Concept: API Gateway

- **Edge** authentication, rate limits, routing, TLS termination — one front door before microservices.

### Concept: BFF (backend for frontend)

- **API tailored** to one client (mobile vs web) — aggregates calls so the app makes fewer round trips.

### Concept: Rate limiting and 429 Retry-After

- **Rate limit:** cap requests per user/IP to stop abuse.  
- **HTTP 429:** “too many requests”; **Retry-After** hints backoff — use especially on **hold** endpoints.

### Concept: Bulkhead pattern

- **Isolate** thread pools or connection pools (e.g., browse vs checkout) so one overload doesn’t take down the other.

### Concept: Backpressure

- When downstream is slow, **slow down upstream** (queue, drop non-critical work, return errors) instead of unbounded memory growth.

### Concept: JWT and HMAC signing (tickets)

- **JWT:** header.payload.signature — payload is readable unless encrypted (usually **signed** for integrity).  
- **HMAC:** symmetric signature with a **secret** from KMS; **kid** in header picks which key for rotation.

### Concept: mTLS and OAuth2 client credentials

- **mTLS:** mutual TLS — client and server both present certs — common for **partner** and **gate scanner** APIs.  
- **Client credentials:** machine-to-machine OAuth for **partner sync** jobs.

### Concept: PCI DSS and SAQ-A

- **PCI DSS:** card industry security standard.  
- **SAQ-A:** lighter self-assessment when **card data** stays on PSP-hosted fields — typical for app + Stripe.js style flows.

### Concept: GDPR and data subject requests (DSR)

- EU privacy rules: access, deletion, portability.  
- **Tickets/orders** may be **legal/financial records** — deletion may mean **anonymize** vs hard delete.

### Concept: OpenTelemetry and trace_id

- **Distributed tracing** ties one user request across services — propagate `trace_id` from gateway through inventory, checkout, and async workers.

### Concept: SLO, SLI, error budget

- **SLI:** measured metric (e.g., successful holds).  
- **SLO:** target (e.g., 99.95% success).  
- **Error budget:** allowed failure rate before freezing risky launches.

### Concept: IANA time zone and DST

- Store **UTC** + venue’s **IANA** zone (e.g., `America/New_York`) for correct local display and **DST** jumps.

### Concept: PostGIS / geospatial queries

- **PostGIS** adds geometry types and indexes to Postgres — “cinemas within 5 km” — optional if **city_id** filtering is enough for MVP.

### Concept: WebSocket vs SSE vs polling

- **WebSocket:** bidirectional real-time — good for **live seat map** updates.  
- **SSE:** server push one-way — simpler for read streams.  
- **Polling:** client repeatedly GETs — simplest MVP, more load.

### Concept: Exactly-once user experience vs at-least-once messaging

- **Messaging** is often **at-least-once** — duplicates possible.  
- **UX exactly-once** comes from **idempotent consumers** + dedupe keys, not from pretending the network is reliable.

### Concept: Reconciliation (finance ops)

- **Reconciliation:** matching **PSP** charges/settlement files to internal `orders` and `seat_sales` to find **orphans** (paid but no ticket, or ticket row without matching money).  
- Critical for **trust** and **audit** — run batch jobs + alerts on backlog size.

### Concept: Chargeback

- Cardholder disputes a charge with their bank; merchant may lose revenue plus fees unless proof of delivery exists.  
- Drives **velocity limits**, **3DS**, and **clear** order/ticket audit trails.

### Concept: Manifest (venue operations)

- **Per-showtime list** of sold seats and sometimes buyer metadata — exported for **gate staff** and **fire-code** alignment with partner systems.

### Concept: Onsale and virtual waiting room

- **Onsale:** moment inventory first opens — often causes **flash traffic**.  
- **Virtual waiting room / queue:** admission control layer (e.g., Ticketmaster-style) that **throttles** who may attempt holds during extreme demand.

### Concept: 3-D Secure (3DS) and SCA

- **3DS:** extra card authentication step (often redirect or in-app challenge) that shifts **fraud liability** and satisfies **SCA** (strong customer authentication) rules in many regions.  
- **Interview angle:** PSP runs the UX; you observe success/failure via API + webhooks and may **step up** to 3DS on risky transactions.

### Concept: KMS (key management service)

- **Managed service** (AWS KMS, GCP Cloud KMS, etc.) that **creates, stores, and rotates** cryptographic keys with **audit logs** and **IAM**-scoped access.  
- **Here:** QR/JWT signing secrets and webhook verification keys — **never** checked into git; apps fetch or decrypt-at-use via SDK with short-lived credentials.

### Concept: PgBouncer / RDS Proxy (database connection pooling)

- **Problem:** thousands of app instances × many threads each can **exhaust** Postgres `max_connections`.  
- **PgBouncer / RDS Proxy:** a **pooler** multiplexes many client connections onto fewer server connections — essential at peak for OLTP under flash traffic.

### Concept: FCM (Firebase Cloud Messaging)

- **Google’s** push notification transport to **Android** (and often used with Apple via APNs in hybrid SDKs).  
- **Pattern:** after `CONFIRMED`, emit event → worker sends **data** push (“payment received”) so the user stops staring at a spinner — complements in-app polling.

---

## Section 3: Understanding Requirements

### What You'll Learn

How to scope a ticketing platform: user journeys, partner (cinema) needs, consistency vs latency, and a crisp MVP vs future feature split.

### Why This Matters

Ticketing is **read-heavy** for discovery but **write-contended** for a tiny fraction of seats per show — exactly where naive databases fail. Interviewers want explicit trade-offs (e.g., holds in Redis vs DB locks).

#### Concepts in this section (primer index)

Terms you will see: **B2B2C**, **showtime / venue / hall layout**, **hold vs sold**, **CAP**, **strong vs eventual consistency**, **MAU/DAU**, **availability**, **p99** — each has a matching **### Concept:** subsection under **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: What Are We Building?

**Analogy:** A **numbered parking garage** for each showtime. Thousands of drivers (users) circle the same 200 spots (seats). Only one car per spot — but everyone tries to “reserve” at the same second when a blockbuster goes live.

**Core journeys:**

1. Pick city → browse movies/events → pick venue/showtime  
2. Open **seat map** → tap seats → **hold** temporarily (e.g., **8 minutes**)  
3. Pay → receive **QR ticket** → scan at gate  
4. Optional: cancel/refund per policy; **waitlist** if sold out (nice-to-have)

**Who are the users?**

- **Consumers:** browse, book, manage orders  
- **Exhibitors/partners:** cinemas, event organizers — **they** own seats and pricing rules; our system is often a **B2B2C** marketplace

#### Real-world ecosystem (food-delivery doc style)

Think of ticketing like a **three-sided marketplace** (customer ↔ platform ↔ venue) plus a **fourth dependency** — **payments (PSP)** — that is not “your inventory” but gates conversion:

```text
  MOVIEGOER                      PLATFORM                         EXHIBITOR / CINEMA
      ↓                              ↓                                    ↓
 Browse titles         ←──→   Catalog + Discovery        ←──→    Supplies showtimes, halls,
 Pick showtime                + Search (OpenSearch)                 base prices, blackout rules
 Open seat map                + Inventory (Redis holds)             May also sync inventory
 Pay & get QR                 + Orders + Tickets                  Needs sold-seat MANIFEST
      ↓                              ↓
  PSP (Stripe / Razorpay / …)  Webhooks + reconciliation          Partner ops / gate scanners
```

**Core flows (mirror the “three journeys” depth in food delivery):**

1. **Customer journey:** city select → list → movie detail → showtimes → **seat map** → hold → PSP checkout → ticket/QR → inbox.  
2. **Exhibitor journey:** onboard venues/screens → publish schedule → (optional) pull our sales export → reconcile with POS.  
3. **Platform journey:** rate limits + anti-bot on holds, idempotent payments, notifications, chargeback tooling.

---

### 🟡 For Intermediate: Functional & Non-Functional Requirements

#### Functional requirements (prioritized)

| Priority | Capability | Notes |
|----------|------------|--------|
| P0 | Browse by city, language, format (2D/3D/IMAX), date | Heavy read; CDN + cache |
| P0 | Showtime list + **live seat map** with availability | Per-showtime cache; invalidation on hold/release |
| P0 | **Hold seats** with TTL (e.g., 480s) | Prevents griefing; bounded reservation |
| P0 | Checkout + **payment** + confirmed booking | Idempotent `order_id` / `idempotency-key` |
| P0 | Order history + ticket (QR/PDF) retrieval | Strong consistency on order details |
| P1 | Search (title, artist, genre) | Elasticsearch or managed search |
| P1 | Offers, rewards, gift cards | Adds complexity; segment in interview |
| P2 | Resale/waitlist/partner APIs | Often separate services later |

#### Non-functional requirements

```text
Availability:
├─ Discovery & browsing: 99.95% (~4.38h/year)
├─ Seat holds + checkout: 99.95% with graceful degradation (queue UI if needed)
└─ Correctness: Zero double-booked seats (invariant)

Latency (targets for design discussion):
├─ Listings / detail pages (cached): p99 < 150ms
├─ Seat map read: p99 < 200ms
├─ Hold attempt: p99 < 300ms (regional)
└─ Payment confirmation: dominated by PSP (~1–3s); our goal is not add much beyond 100–200ms

Scale (assumptions for Section 4):
├─ 50M MAU, 8M bookings/month
├─ ~3k cities, 15k screens/venues (illustrative)
└─ Peak: flash drop 20k concurrent seat ops in 30s on a single hot showtime
```

#### Clarifying questions (interview script)

- Single country vs multi-region? Currency?  
- Do we sell **only** our inventory or **aggregator** model (partner APIs)?  
- Refund policy: fixed window? Organizer-specific?  
- Payment methods: cards, UPI, wallets — **PCI** scope implications?  
- Must seat map be **real-time WebSocket** or **polling** acceptable?

#### Detailed user stories (aligned with `food_delivery_system_design.md` depth)

**As a moviegoer, I want to:**

- Browse **now showing / coming soon** for **my city** with correct language and format (2D/3D/IMAX).  
- Search by title, cast, or genre with autocomplete and typo tolerance.  
- See **showtimes** filtered by date, venue distance, and ticket **price range** (inclusive of fees where required by law).  
- Open an **interactive seat map** and know whether a seat is **available, held, or sold**.  
- Hold seats for a **bounded time** (e.g., 8 minutes) while I complete payment.  
- Pay with cards, UPI, wallets, etc. (region-dependent) without exposing PAN to the merchant app.  
- Receive **QR / digital ticket** reliably after payment, plus email/SMS backup.  
- View order history, resend ticket, and request refund **per policy**.  
- Get **honest urgency** (“selling fast”) without bait-and-switch vs real-time inventory.

**As an exhibitor / cinema operator, I want to:**

- Publish and update showtimes; block seats for **house staff / comps** without breaking public map rules.  
- Export a **per-showtime manifest** of sold seats for gate and fire-code compliance.  
- Receive **predictable payouts** and dispute data tied to `charge_id` / settlement batches.  
- Optionally integrate via **partner API** or file drops — with idempotent upserts.

**As a platform operator, I want to:**

- Enforce **zero double-booking** and measurable **checkout success** (excluding user card declines).  
- Detect **fraud, bots, and card testing** without blocking legitimate NAT users entirely.  
- Run **reconciliation** between PSP money movement and internal orders.  
- Observe **per-showtime** hot keys, webhook health, and consumer lag on ticket/email pipelines.

#### Consistency model (CAP-style, interview table)

| Data | Strong vs eventual | Rationale |
| --- | --- | --- |
| `seat_sales` (sold seats) | **Strong** (transactional) | Money + legal seating capacity |
| Redis holds | **Strong per key** (single-threaded + Lua) | Prevent partial multi-seat holds |
| Seat map *display* snapshot | **Slight lag OK** (sub-second to few s) | UX vs load; hold path is authoritative |
| OpenSearch movie index | **Eventual** (seconds to minutes) | Search freshness vs throughput |
| “Trending” lists | **Eventual** | Batch/analytics |
| Ticket email delivery | **Eventual** | At-least-once email pipeline |

---

### 🔴 For Advanced: Business & Compliance Context

- **Chargeback/friendly fraud:** high-value target for card testing; velocity rules and 3DS where required.  
- **Partner SLAs:** cinemas demand **accurate** seat manifests for door entry; reconciliation jobs compare our sold seats vs partner systems if integrated.  
- **Regulatory:** ticket pricing caps in some jurisdictions; **accessibility** seating rules; event-specific minors policies.  
- **Anti-scalping:** limits per user, device fingerprinting, captcha on hot drops — product/legal balance.

#### SLAs you can cite in interviews (examples)

| SLA topic | Example target | Notes |
| --- | --- | --- |
| Discovery availability | 99.95% | Exclude planned maintenance windows if stated |
| Booking API success | 99.95% excluding user card declines | Declines are “successful” operationally but business-negative |
| Manifest export freshness | &lt;15 min after sale batch | Partner cinemas for gate reconciliation |
| Support lookup by order | p95 &lt;2 s | Read replica OK |

#### Multi-sided marketplace dynamics

- **Supply** (showtimes) can disappear when partner **pulls listings** — need `showtime.status = cancelled` propagation and automatic refund offers unless replaced.  
- **Demand spikes** are **non-stationary** (one trailer drop) — capacity tests must script **single-object** hot paths, not uniform random.

#### Think About It

> Why is “eventual consistency” often acceptable for **search ranking** but **not** for “Seat 12 sold twice”?

#### Key Takeaways

- Separate **browse** (scale reads) from **book** (strong invariants + payments).  
- State assumptions on **hold TTL**, **idempotency**, and **partner model** early.

#### Practice Exercise

List **five** ways a malicious user could abuse seat holds if there is no per-user rate limit or bot protection.

### Extended detail: user stories, acceptance criteria, and explicit out-of-scope

#### User stories (interview-ready)

| ID | As a… | I want to… | So that… |
| --- | --- | --- | --- |
| US-1 | Moviegoer | Change city and see correct listings | I don’t book the wrong town |
| US-2 | Moviegoer | See which seats are taken before I pay | I don’t guess availability |
| US-3 | Moviegoer | Pay once and get tickets I can show at the door | Entry is smooth |
| US-4 | Partner ops | Export sold-seat manifest per showtime | Gate staff match our system |
| US-5 | Support | Look up order by PSP `charge_id` or phone | Disputes resolve in one call |

#### Acceptance criteria examples (testable)

- **AC-HOLD-1:** If seat **S** is already in `seat_sales` for showtime **T**, a hold request for **S** returns **409** with code `SEAT_SOLD` in **&lt;300ms p99** in the same region as the user.
- **AC-HOLD-2:** Successful hold returns `hold_id`, `expires_at_ms`, and seat list; client countdown uses **server** `expires_at_ms`, not only local timer.
- **AC-PAY-1:** Duplicate `POST /checkout/sessions` with same `Idempotency-Key` returns **same** `order_id` and HTTP **200** (or **201** once) for **72h**.

#### Explicit non-goals for a first design pass (say this aloud)

- **No** full resale marketplace legal engine  
- **No** dynamic “surge” pricing ML (unless interviewer steers)  
- **No** custom seat map editor UX — only **data model** for hall JSON  
- **No** accounting general ledger — link to orders + PSP only  

#### Assumptions log (copy into interview whiteboard)

| # | Assumption | If wrong |
| --- | --- | --- |
| A1 | One inventory authority per marketplace region | Must split inventory plane |
| A2 | PSP owns 3DS / SCA UX | We scope PCI to redirect/hosted fields |
| A3 | Showtimes are **not** cross-sold in two regions at once | Adds global inventory split-brain risk |

### 🎯 Interview Questions — Requirements & planning (food-delivery style)

#### Beginner

**Q1:** What are the **core functional** requirements for a BookMyShow-like system?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer (outline):**

- **Discovery:** city-aware listings, search, movie detail, showtime filters.  
- **Inventory:** seat-level holds + sold state; **no double booking**.  
- **Checkout:** create order, integrate PSP, confirm, issue tickets.  
- **Post-purchase:** order history, refunds per policy, QR retrieval.  
- **Partner/ops (if in scope):** manifests, reconciliation hooks.

**Tip:** Separate “browse” from “book” in your first sentence — interviewers listen for it.

</details>

**Q2:** How do you separate **functional** vs **non-functional** requirements here?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Functional:** what features exist (holds, pay, ticket).  
**Non-functional:** how well it works — p99 latencies, availability **99.95%**, correctness guarantees, security (PCI scope), abuse resistance.

**Tip:** Tie NFRs to **measurement** (SLOs), not adjectives.

</details>

#### Intermediate

**Q3:** What clarifying questions do you ask about **hot showtimes** and **inventory authority**?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Questions:**

- Single region inventory or multi-region marketplace?  
- Aggregator model: who is **source of truth** for a seat — us vs cinema POS?  
- Must seats be **pick-your-seat**, or are **best available** / general admission acceptable?  
- Cancellation/refund windows and chargeback rates by category (movies vs sports)?

**Tip:** Hot showtime = **single-key contention** — say it explicitly.

</details>

**Q4:** Which parts want **strong consistency** vs **eventual** consistency?

<details>
<summary>💭 Think first, then reveal answer</summary>

Use the table in this section: **seat_sales + payments** strong; **search index** eventual; **marketing badges** eventual with honest labeling.

**Tip:** Name one invariant: **`UNIQUE(showtime_id, seat_label)`**.

</details>

---

## Section 4: Capacity Planning

### What You'll Learn

Back-of-envelope QPS, storage, bandwidth, and hot-spot reasoning for ticketing.

### Why This Matters

Interviewers reward **numbers**: peak RPS on seat APIs, Kafka throughput for order events, and DB shard counts for order tables.

#### Concepts in this section (primer index)

**QPS/RPS**, **p99**, **Redis memory / holds**, **PostgreSQL connections**, **PgBouncer / RDS Proxy**, **Kafka partitions/lag**, **storage + index overhead** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: Traffic in Plain Terms

**Reads:** Like a newspaper stand — many people browse titles; only a few buy a specific copy **right now**.

**Writes:** Buying is rarer than browsing, but **many buyers collide on one show** (opening night).

#### Step-by-step: booking attempts per second (food-delivery doc style)

First anchor **bookings per day**, then derive **average** and **peak** seat *attempts* (not all convert to sales):

```text
Bookings/day:           267,000
Seconds/day:            86,400
Avg bookings/s:         267,000 / 86,400 ≈ 3.1 bookings/s
```

Evening peak often concentrates **40–60%** of bookings into **~4 hours** (illustrative):

```text
Evening bookings (50% of daily):  133,500 in 4 hours = 14,400 seconds
Peak bookings/s:                  133,500 / 14,400 ≈ 9.3 bookings/s
```

A blockbuster **onsale** is sharper still: many **hold attempts** on one showtime in a short window — model **seat op RPS** separately from **booking RPS** (this is why averages lie).

---

### 🟡 For Intermediate: Calculations

#### Assumptions

```text
MAU:                    50,000,000
Daily active users:     5% of MAU = 2.5M DAU
Bookings/month:         8,000,000  →  ~267k bookings/day
Avg tickets/booking:    2.2  →  ~587k tickets/day
Peak factor (evening):  3x vs daily average for booking writes
Read/write ratio:       ~100:1 for overall API traffic (browse vs buy)
Hot showtime peak:      20,000 seat-hold attempts in 30s  →  ~667 holds/s (single hot partition problem!)
```

#### API QPS (rough)

```text
Total API requests/day (assumed 40 reads per booking funnel, mixed): 10M MAU * small fraction…
Simpler anchor:  100M read-like API calls/day →  ~1,160 RPS average
Peak browse:     10x average →  ~12,000 RPS (edge-cached)
Booking path:    267k/day →  3 RPS average; peak night 30 RPS average cluster-wide,
                  BUT single showtime can hit 500–1000 RPS on seat service (hot key).
```

#### Storage (orders + tickets)

```text
Order row (metadata + JSON): ~2 KB
Tickets/day: 587k  →  ~1.2 GB/day raw orders
5-year retention (compressed/archived):  orders tiered to cold storage after 90 days active
PostgreSQL primary:  expect 2–5 TB orders domain (indexes, replicas) at maturity
Seat map static definitions:  small (KB per hall); availability state per showtime in Redis/cache layer
```

#### Bandwidth

```text
Seat map JSON (200 seats): ~20–40 KB gzip per request
10k concurrent map loads at peak:  400 MB/s at origin if uncached — must cache per showtime key at edge + Redis
Trailers/posters:  offload to CDN (S3 + CloudFront); not on application servers
```

---

### 🔴 For Advanced: Hot Partitions & Mitigation

```text
Problem:  Showtime_id = S-999  becomes a "hot key" in any single-partition design.

Mitigations:
├─ Redis per-showtime key sharded by slot range (hash tag per showtime still one key — use internal slot partitions)
├─ Application-layer sharding: seat_id ranges (A1–J10 → shard 1) with orchestration
├─ Admission control: queue + lottery for ultra-hot (Taylor Swift pattern)
└─ Precompute available COUNT for UI; authoritative seat set still single writer path
```

📊 **Example:** Ticketmaster documented **queue** systems for massive onsales; same idea applies at smaller scale for regional blockbusters.

#### Think About It

Estimate Redis memory if **1M concurrent held seats** exist globally at 150 bytes per hold entry.

#### Key Takeaways

- Average QPS misleads; **design for hot showtime** and **fan-out cache invalidation** cost.  
- **CDN** for assets; **Redis** for hot ephemeral holds; **PostgreSQL** for durable orders.

#### Practice Exercise

Given **5,000 RPS** seat-map reads and **95% hit** on CDN+Redis, compute origin RPS.

### Extended detail: worked math and connection budget

#### Seat-map origin RPS (worked)

- Reads: **5,000 RPS** × (1 − 0.95) = **250 RPS** at origin (app + Redis miss path).
- Add **5–10%** for cache stampede recompute → plan **~275–280 RPS** sustained on the seat-map builder for that hotspot.

#### Redis memory for holds (worked)

- Assume **900,000** active hold keys globally at peak evening, **180 bytes**/key (key + small JSON + overhead).
- **900,000 × 180 B ≈ 162 MB** — modest; **hot showtimes** add jitter via key cardinality, not only bytes.
- Plan **2× headroom** for fragmentation and Lua argv → **~350 MB** slice on cluster dedicated to holds.

#### PostgreSQL connections (order-of-magnitude)

- **App servers:** 200 pods × 20 pool connections max = **4,000** client connections if poorly tuned — cap pool (e.g., **5–10** per pod), use **RDS Proxy** or **PgBouncer**, target **&lt;800** live connections to primary under peak.

#### Kafka throughput (orders domain)

- **267k** orders/day × **6** events (create, pay_ok, ticket, email_req, analytics, audit) ≈ **1.6M msgs/day** ≈ **18.5 msg/s** average; peak hour **×6** → **~110 msg/s** — still modest; partition count driven by **parallel consumers**, not raw rate.

#### Storage math with indexes (worked, interview style)

```text
Assume:
  seat_sales row (payload + indexes):     ~500 bytes
  Tickets/day:                            587,000
  Daily seat_sales growth:                587,000 × 500 B ≈ 293 MB/day

Year-1 seat_sales (ignore compaction):
  293 MB × 365 ≈ 107 GB/yr (plus HOT indexes, replicas, WAL overhead)

Orders table (2 KB/order × 267k/day) ≈ 534 MB/day before archival policy
```

**Tip:** Add **20–30%** overhead for Postgres index/page bloat in interviews — same pattern as food delivery storage walkthroughs.

### 🎯 Interview Questions — Capacity planning

**Q1:** Why is average RPS misleading for ticketing?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Browsing averages hide **flash contention** on a single `showtime_id`. Design for **per-showtime seat operation peaks**, queue admission, and Redis hot-key plans — not global averages.

</details>

**Q2:** How do you estimate Redis memory for holds?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** `E[active holds] × bytes per hold key` + fragmentation headroom. Justify **E[active holds]** from hold TTL × arrival rate × completion probability; show **2×** headroom.

</details>

---

## Section 5: High-Level Architecture

### What You'll Learn

Major services, data stores, sync/async boundaries, and a reference diagram for BookMyShow-style flows.

### Why This Matters

A clear **BFF/mobile API**, isolated **inventory service**, and **async order pipeline** demonstrate senior-level decomposition.

#### Concepts in this section (primer index)

**API Gateway**, **BFF**, **CDN**, **Kafka**, **outbox**, **PSP** — definitions under **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: Big Boxes

```text
Client apps ─► API Gateway ─► Services ─► Databases / Redis / Kafka
                                │
                         Payment provider (Stripe/Razorpay)
```

Users talk to **our** APIs; payments go to a **PSP** (we rarely store PAN).

---

### 🟡 For Intermediate: Service boundaries

| Service | Responsibility |
|---------|----------------|
| **User & profile** | Accounts, sessions, devices |
| **Catalog** | Movies, events, creatives, age ratings |
| **Discovery** | City feed, trends, filters; talks to search index |
| **Search index** | Elasticsearch/OpenSearch for titles/casts |
| **Venue & hall** | Hall JSON layouts, seat numbering, screen metadata |
| **Schedule** | Showtimes, blackout rules, partner overrides |
| **Inventory & holds** | Authoritative seat availability + short TTL holds |
| **Pricing & fees** | Base price, tax, convenience fee, offers engine (P1) |
| **Checkout & orders** | Order creation, idempotency, state machine |
| **Payments** | PSP adapters, webhooks, refunds |
| **Tickets** | QR payload, signing keys, rotation |
| **Notifications** | Email/SMS/push via Kafka consumers |
| **Anti-abuse / risk** | Rate limits, device reputation, captcha orchestration |
| **Analytics** | Clickstream to data warehouse (Snowflake/BigQuery/ClickHouse) |

#### REST API surface (v1) — 12+ core endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| `GET` | `/v1/cities` | List cities where we operate |
| `GET` | `/v1/cities/{city_id}/movies` | Now showing / coming soon for city |
| `GET` | `/v1/movies/{movie_id}` | Detail page + creatives |
| `GET` | `/v1/movies/{movie_id}/showtimes` | Filter by date, venue, format |
| `GET` | `/v1/showtimes/{showtime_id}/seat-map` | Hall layout + seat statuses (cached) |
| `POST` | `/v1/showtimes/{showtime_id}/holds` | Acquire seat holds (body: seat labels[]) |
| `DELETE` | `/v1/holds/{hold_id}` | Release holds before payment |
| `POST` | `/v1/checkout/sessions` | Create order + line items; returns client secret for PSP |
| `POST` | `/v1/orders/{order_id}/confirm` | Finalize after PSP success (idempotent) |
| `GET` | `/v1/orders/{order_id}` | Order + ticket IDs |
| `GET` | `/v1/tickets/{ticket_id}` | Ticket detail + QR metadata reference |
| `POST` | `/v1/entry/validate` | Partner gate scanner validates QR (mTLS for venue devices) |
| `GET` | `/v1/users/me/orders` | Order history (paginated) |
| `POST` | `/v1/webhooks/payments` | Inbound PSP events (signature verified; not called by mobile) |

Example hold request:

```http
POST /v1/showtimes/st_9f3c221a/holds HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
Idempotency-Key: 7c1e2b3a-4d5e-6f7a-8b9c-0d1e2f3a4b5c

{
  "seat_labels": ["F12", "F13"],
  "session_id": "sess_mobile_abc123"
}
```

Example session creation response (snippet):

```json
{
  "checkout_session_id": "cs_a1b2c3",
  "order_id": "ord_778899",
  "amount_cents": 85000,
  "currency": "INR",
  "payment_client_secret": "psp_secret_xxx",
  "hold_expires_at_ms": 1713123456789
}
```

#### Mermaid: high-level architecture

```mermaid
flowchart TB
    subgraph clients [Clients]
        MOB[Mobile Apps]
        WEB[Web]
    end

    subgraph edge [Edge]
        CDN[CDN Static Media]
        LB[Load Balancer]
        GW[API Gateway]
    end

    subgraph sync [Core Services]
        CAT[Catalog Service]
        DISC[Discovery Service]
        VEN[Venue Service]
        INV[Inventory Service]
        CHK[Checkout Service]
        PAY[Payment Adapter]
        TIX[Ticket Service]
        NOTIF[Notification Service]
    end

    subgraph data [Data Layer]
        PG[(PostgreSQL)]
        RD[(Redis)]
        OS[(OpenSearch)]
        KFK[Kafka]
        OB[(Object Store)]
    end

    subgraph third [Third Party]
        PSP[Payment PSP]
    end

    MOB --> CDN
    WEB --> CDN
    MOB --> LB
    WEB --> LB
    LB --> GW
    GW --> CAT
    GW --> DISC
    GW --> VEN
    GW --> INV
    GW --> CHK
    CHK --> PAY
    CHK --> TIX
    PAY --> PSP
    CAT --> PG
    VEN --> PG
    INV --> RD
    INV --> PG
    CHK --> PG
    CHK --> KFK
    TIX --> OB
    NOTIF --> KFK
    DISC --> OS
```

---

### 🔴 For Advanced: Deployment topology

- **Multi-AZ** minimum; **multi-region** active/passive or partitioned by country for GDPR/data residency.  
- **Inventory service** colocated with Redis cluster in same AZ as DB writer for conflict-free orchestration (trade latency for correctness).  
- **Outbox pattern** for `order_created` → Kafka → notifications and analytics (never double-send without idempotent consumers).

#### Think About It

Where would you **not** use a distributed transaction across Inventory + Payments?

#### Key Takeaways

- **Inventory** and **checkout** are separate contexts; integrate via **Saga** + **idempotent** PSP calls.  
- **Kafka** decuples confirmation side-effects from request path.

#### Practice Exercise

Add **one** component to the diagram for **experiments** (A/B price display) without impacting inventory correctness.

### Extended detail: synchronous paths, errors, and experiment flag placement

#### Path A — Browse only (read-heavy)

```text
Client → CDN (posters)  
Client → GW → Discovery → OpenSearch / PG replica  
No inventory call unless user opens seat map.
```

#### Path B — Seat map

```text
Client → GW → Inventory-read → Redis snapshot (per showtime) OR assemble PG sold + Redis holds
```

Respond with staleness header optional: `X-Seat-Snapshot-Age-Ms: 42` so client can soft-refresh.

#### Path C — Hold + checkout (write path)

```text
Client → GW → Inventory (Lua/NX) → Checkout (create order) → PSP token  
Async: webhook → PG commit → Outbox → Kafka
```

#### Example error bodies (stable API contract)

```json
{
  "error": {
    "code": "SEAT_NOT_AVAILABLE",
    "message": "Seat F12 is no longer available for this showtime.",
    "showtime_id": "st_9f3c221a",
    "seat_labels": ["F12"],
    "conflict": "SOLD"
  }
}
```

```json
{
  "error": {
    "code": "SEAT_HELD_BY_OTHER",
    "message": "Seat F12 is temporarily held by another shopper.",
    "retry_after_ms": 400
  }
}
```

#### Experiments (A/B) without touching inventory

- Run **Pricing display** or **hero image** experiments in **Discovery** only (`experiment_sdk` at GW).
- **Never** branch inventory or uniqueness rules on experiment flags — same code path for holds/sales.

### Database design overview (maps to food delivery “Database Design” chapter)

This mirrors how `food_delivery_system_design.md` pulls **tables + keys** into one place; here the **critical** OLTP tables are:

| Table / store | Role | Primary access pattern |
| --- | --- | --- |
| `movies`, `venues`, `showtimes` | Schedule + catalog linkage | Read-heavy; cache + OpenSearch |
| `hall_layout_versions`, `seats_static` | Immutable layout | Rare write; read for map |
| `seat_sales` | **Sold** truth | Insert on commit; `UNIQUE(showtime_id, seat_label)` |
| `orders`, `order_line_items` | Money + legal snapshot | By `user_id`, `idempotency_key` |
| `payment_events` | Webhook idempotency | Dedupe by `psp_event_id` |
| `outbox_events` | Reliable Kafka | Poll + publish |
| Redis keys `hold:{st}:{seat}` | **Ephemeral** holds | `SET NX EX` / Lua |

**Indexing reminders:** `(showtime_id)` on `seat_sales` for manifests; `(created_at)` BRIN or monthly partitions for large time scans; `(idempotency_key) UNIQUE` on orders.

#### Extra API examples (partner + admin, interview depth)

```http
GET /v1/partners/v1/showtimes?updated_since=2026-04-14T00:00:00Z HTTP/1.1
Authorization: Bearer m2m_partner_token
```

```http
GET /v1/admin/showtimes/st_4412/manifest.csv HTTP/1.1
Authorization: Bearer admin_token
```

```http
POST /v1/risk/hold_precheck HTTP/1.1
Content-Type: application/json

{ "user_id": "u_1", "showtime_id": "st_4412", "seat_count": 4 }
```

### 🎯 Interview Questions — Architecture & API surface

**Q1:** Where would you **not** use a 2PC across Inventory and Payment?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** PSP is an external system with its own timeouts and retries. Use **Saga**: reserve holds locally, call PSP with idempotency key, commit on webhook; compensate with **release holds + refund** on failure. Avoid holding DB locks across network calls to PSP.

</details>

**Q2:** Why separate Discovery service from Inventory service?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Different scaling profiles and consistency: discovery is **read-heavy, eventually fresh**; inventory is **write-contended and must stay correct**. Separation prevents slow search deploys from risking booking paths.

</details>

---

## Section 6: Seat Inventory, Holds & Concurrency Control

### What You'll Learn

Why row-level locks are not enough at scale, how Redis holds work, and failure modes (TTL, GC, double-click).

### Why This Matters

**This is the classic system design deep dive** for BookMyShow/Fandango — analogous to inventory in e-commerce but with **fine-grained contention**.

#### Concepts in this section (primer index)

**Hold vs sold**, **Redis `SET NX` + TTL**, **Lua**, **hash tags**, **hot keys**, **PostgreSQL `UNIQUE`**, **read replica lag**, **ACID** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: The Problem in One Picture

**Two users tap the same seat** at the same time. Only one can win. The other must see **“Seat taken”** instantly.

We solve it with:

1. A **single authority** for each showtime’s seat set (often one Redis key per seat or a structure per showtime).  
2. A **short hold** (soft reservation) before money moves.  
3. **TTL** so abandoned carts free seats automatically.

---

### 🟡 For Intermediate: Data model & algorithms

#### PostgreSQL (durable source for *sold* state)

```text
Table: showtimes (id PK, movie_id, venue_id, hall_id, starts_at, …)

Table: seat_sales (
  id BIGSERIAL PK,
  showtime_id BIGINT NOT NULL,
  seat_label TEXT NOT NULL,       -- e.g. 'F12'
  order_id BIGINT NOT NULL,
  status TEXT NOT NULL,            -- 'sold', 'refunded'
  UNIQUE (showtime_id, seat_label)
)

Index: (showtime_id) for manifests
```

**Holds** may live in **Redis** for speed:

```text
Key: hold:{showtime_id}:{seat_label} → JSON { user_session_id, expires_at_ms }
OR
Hash: showtime:{id}:holds → field seat_label → expiry
TTL: 480 seconds per field (Redis hash TTL is per-key — often use per-seat keys for simpler TTL)
```

#### Conflict resolution flow

```text
1. Client requests hold(seat F12)
2. Inventory checks: already SOLD in PG? reject
3. SET hold key NX (set if not exists) with EX 480
4. If NX fails → seat held by other → try another seat
5. On payment success → transactional INSERT seat_sales + DELETE hold
6. On expiry → key vanishes; periodic reconciliation cleans stragglers
```

#### Pseudocode (hold acquire)

```text
function tryHold(showtimeId, seat, sessionId):
  if existsSeatSale(showtimeId, seat):
      return CONFLICT_SOLD
  key = "hold:" + showtimeId + ":" + seat
  ok = redis.SET(key, sessionPayload, NX, EX=480)
  if not ok:
      return CONFLICT_HELD
  return OK
```

#### Durable tracking for holds (optional but useful for support)

```text
Table: seat_holds_audit (
  hold_id UUID PRIMARY KEY,
  showtime_id BIGINT NOT NULL,
  seat_label TEXT NOT NULL,
  user_id BIGINT,
  idempotency_key TEXT,
  created_at TIMESTAMPTZ NOT NULL,
  released_at TIMESTAMPTZ,
  outcome TEXT  -- 'converted_to_sale', 'expired', 'user_cancelled'
)
```

Insert **after** Redis succeeds if you need CS traceability; can be async batched to avoid latency on hot path.

#### Seat map read model (cache-aside)

1. **Cold:** build JSON from `hall_layout_version` + query `seat_sales` + scan Redis `hold:{st}:{seat}` keys (expensive) — avoid in request path.  
2. **Warm:** maintain `snapshot:showtime:{id}` in Redis **updated incrementally** on each hold/sell with **version** integer; clients send `If-None-Match: W/"ver123"`.  
3. Invalidate only affected showtime on state change — never global flush for one sale.

---

### 🔴 For Advanced: Edge cases

- **Clock skew:** rely on Redis TTL, not client clocks.  
- **Fail between payment OK and DB commit:** **reconciliation job** compares PSP `charge_id` vs `seat_sales`; saga compensates (refund) if orphan.  
- **Redis failure:** fallback read-only mode **or** degrade to **queued** booking — product choice.  
- **Lua script** for atomic multi-seat hold:

```lua
-- KEYS = hold keys for each seat in same showtime (same hash tag in cluster)
-- ARGV[1] = TTL seconds, ARGV[2] = payload string
local ttl = tonumber(ARGV[1])
local payload = ARGV[2]
for i = 1, #KEYS do
  if redis.call('EXISTS', KEYS[i]) == 1 then
    return redis.error_reply('HELD:' .. KEYS[i])
  end
end
for i = 1, #KEYS do
  redis.call('SET', KEYS[i], payload, 'EX', ttl)
end
return 'OK'
```

**Cluster note:** use a **hash tag** `{st:showtimeId}` in all keys so Lua keys land in one slot — e.g. `hold:{st:4412}:F12`.

Production teams use **Redis Lua** or single-threaded partition to avoid partial holds.

#### Anti-patterns

- Long DB row locks on `seats` table during browsing — kills throughput.  
- Storing only **count** of free seats without seat IDs — users cannot pick exact seats.

#### Think About It

How do you safely **extend** a user’s hold from 8 → 10 minutes without unbounded abuse?

#### Key Takeaways

- **NX + TTL** pattern is standard; pair with **PostgreSQL uniqueness** for sold state.  
- **Hot showtimes** need **admission control** and **observability** per showtime ID.

#### Practice Exercise

Design metrics and alerts for **hold conflict rate** and **post-payment insert failure rate**.

### Extended detail: failure windows and recovery

| Window | What breaks | Detection | Mitigation |
| --- | --- | --- | --- |
| T1 | Client loses network after hold, never pays | Idle holds expire | TTL + optional reminder push before expiry |
| T2 | PSP succeeds; app crashes before DB commit | Orphan `charge_id` | Reconciliation + auto-refund or complete order |
| T3 | DB commits `seat_sales`; Redis hold delete fails | Seat sold + hold key lingers rare | Reconcile job deletes stale holds where sale exists |
| T4 | Kafka emit fails after commit | User has ticket row; no email | Outbox retries; DLQ alert |

**Metric bundle for Section 6 on-call:** `rate(hold_attempt_total{result=conflict})`, `rate(hold_attempt_total{result=ok})`, `histogram_hold_latency_ms`, `inventory_lua_errors_total`.

### 🎯 Interview Questions — Seat inventory & holds

**Q1:** Why not use **only** PostgreSQL row-level locks for holds?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** RLM on hot rows serializes everyone on one showtime’s seats at high QPS; long transactions risk **lock pile-ups** and **connection pool** starvation. Redis `SET NX` + short TTL moves contention to a **purpose-built** store with O(1) primitives; Postgres remains **source of truth for sold**.

</details>

**Q2:** How does **Lua** help vs a loop of `SET NX` in application code?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** A loop can leave **partial** holds if the client crashes mid-loop; another user could grab middle seats. Lua is **atomic** on the Redis side for all keys in one slot (hash tag).

</details>

**Q3:** What happens if **read replica** shows a seat free but it is sold?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** **Mutations** (hold) must check **primary** or a service that reads **authoritative** sold state; replicas are OK for **display** with “may be stale” semantics.

</details>

---

## Section 7: Booking Flow, Payments & Order Lifecycle

### What You'll Learn

End-to-end state machine: created → payment_pending → confirmed/failed; idempotency; webhooks; refunds.

### Why This Matters

Payments duplicate on retries — **idempotency keys** are non-negotiable, same as payment gateway design.

#### Concepts in this section (primer index)

**Idempotency**, **PSP**, **webhooks**, **saga / compensation**, **2PC**, **reconciliation**, **charge vs intent**, **exactly-once UX vs at-least-once**, **FCM** (push after pay) — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: States Are Like Pizza Order Tracking

```text
CREATED → PAYMENT_PENDING → CONFIRMED
                       ↘ FAILED
CONFIRMED → REFUND_PENDING → REFUNDED
```

If payment takes time, UI shows spinner — seats stay **held** until timeout.

---

### 🟡 For Intermediate: Order state machine

```text
States:
- created: order row exists, total computed
- payment_pending: client invoked PSP SDK
- confirmed: webhook/charge success + seat_sales committed
- failed: deny or inventory released
- refunded / partial_refund: finance policies
```

**Idempotency:**

```http
POST /v1/checkout/sessions
Idempotency-Key: 8f3c2c1a-...
```

Server stores `idempotency_key → order_id` mapping for **24–72 hours**.

#### Webhook handling

```text
PSP sends charge.captured → verify signature → find order by metadata order_id
→ BEGIN TRANSACTION → insert seat_sales → update orders SET status='confirmed' → COMMIT
→ emit OrderConfirmed event to Kafka
```

#### Core tables (minimal DDL sketch)

```text
Table: orders (
  id BIGSERIAL PRIMARY KEY,
  idempotency_key TEXT UNIQUE,
  user_id BIGINT NOT NULL,
  showtime_id BIGINT NOT NULL,
  status TEXT NOT NULL,  -- created, payment_pending, confirmed, failed, refunded
  amount_cents BIGINT NOT NULL,
  currency CHAR(3) NOT NULL,
  psp_intent_id TEXT,
  psp_charge_id TEXT,
  price_snapshot JSONB NOT NULL,  -- line items and fees at pay time
  created_at, updated_at TIMESTAMPTZ
)

Table: order_line_items (
  id BIGSERIAL PRIMARY KEY,
  order_id BIGINT REFERENCES orders(id),
  seat_label TEXT NOT NULL,
  unit_price_cents BIGINT NOT NULL
)
```

#### Inbound webhook (sketch)

```json
{
  "id": "evt_8a2b1c",
  "type": "charge.succeeded",
  "data": {
    "object": {
      "id": "ch_9z9z9z",
      "metadata": { "order_id": "778899" }
    }
  }
}
```

Processing: validate signature → upsert `payment_events(evt_8a2b1c)` → transaction → mark `orders` + insert `seat_sales` + delete holds.

---

### 🔴 For Advanced: Saga & compensation

```text
If charge succeeded but DB commit failed:
1. Alert SEVERITY page
2. Reconciliation job charges orphaned captures → auto-refund if no matching seat_sales within 15 minutes
3. DLQ for failed Kafka emits with retry + poison handling
```

📊 **Example:** Stripe-style **idempotent** Charges API + `metadata.order_id` linking.

#### Think About It

Why store **PSP charge_id** on the order row even if you keep `payment_intent_id`?

#### Key Takeaways

- **Exactly-once user experience** = idempotent APIs + at-least-once webhooks + dedupe.  
- **Metadata** links PSP objects to internal orders immutably.

#### Practice Exercise

Write the **retry policy** for webhook handler when PostgreSQL is temporarily unavailable (return 500 to PSP vs queue internally).

### Extended detail: refunds, partial cancel, and mobile polling

#### Refund API (partner + user initiated)

```http
POST /v1/orders/{order_id}/refunds HTTP/1.1
Content-Type: application/json
Idempotency-Key: refund-uuid-...

{
  "line_item_ids": ["li_101", "li_102"],
  "reason": "user_request"
}
```

Return **202** with `refund_job_id` if PSP async; client polls `GET /v1/refunds/{refund_job_id}`.

#### Why store `psp_charge_id`

Disputes and reconciliation files reference **charge** rows — `intent_id` alone is not always what finance exports from PSP.

#### Mobile polling after pay

- Poll `GET /v1/orders/{id}` with **backoff** (200ms → 500ms → 1s, max **30s**) until `status=confirmed` or timeout → then support path.  
- Prefer **push** + poll hybrid: FCM “payment received” reduces rage quits.

### 🎯 Interview Questions — Booking & payments

**Q1:** Walk through **idempotency** for `POST /checkout/sessions`.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Same `Idempotency-Key` + auth context → return the **same** `order_id` and pricing snapshot within TTL (e.g., 72h). Store mapping in Redis or Postgres. Prevents duplicate orders when mobile retries after timeouts.

</details>

**Q2:** PSP webhook arrives **twice** — what do you do?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Dedupe on `psp_event_id` (unique). Second delivery returns **200** after no-op; **do not** double-insert `seat_sales` or double-emit ticket email (use outbox idempotency or `event_id` guard).

</details>

**Q3:** Charge succeeded but DB down — return 500 to PSP or acknowledge?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** If you acknowledge without durable processing, you risk **lost booking**. Typical pattern: **return non-2xx** so PSP retries **or** enqueue to durable internal queue with **at-least-once** processing and reconcile orphans. Never silently drop.

</details>

---

## Section 8: Discovery, Search & Recommendations

### What You'll Learn

Search index design, city feeds, personalization hooks, and cache layering for movie metadata.

### Why This Matters

Slow search loses users before they reach inventory — but **stale** search is tolerable for minutes.

#### Concepts in this section (primer index)

**OpenSearch / Elasticsearch**, **CDC**, **eventual consistency** (search vs seat truth), **CDN**, **cache stampede** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

**City** is the primary filter: users rarely care about Mumbai showtimes when they are in Bengaluru.

**Analogy (food-delivery parallel):** In food delivery, “nearby restaurants” depends on GPS; in ticketing, **“movies playing near me”** depends on **city** (and sometimes radius) mapped to **venues**. Wrong city = wrong inventory pool — same class of bug as showing LA restaurants to a New York user.

**Three layers of “search” to keep straight:**

1. **Metadata search** — “Dune 3” title / cast (OpenSearch).  
2. **Schedule filter** — “tonight after 8pm” (Postgres or cached schedule service).  
3. **Seat truth** — only **Inventory + seat_sales** count for “can I actually buy F12?” — search must never be the only source.

---

### 🟡 For Intermediate

**OpenSearch/Elasticsearch** documents:

```json
{
  "movie_id": "m_123",
  "title": "Example Part II",
  "language": ["en", "hi"],
  "genre": ["action", "drama"],
  "release_date": "2026-05-01",
  "rating": "PG-13"
}
```

**Queries:** multi-match on title, fuzzy, boosting recent releases.

**Personalization (P1):** feature store + collaborative filtering batch jobs; **not** on critical path for MVP.

#### OpenSearch query example (multi-match + city filter)

```json
{
  "size": 20,
  "query": {
    "bool": {
      "must": {
        "multi_match": {
          "query": "dune",
          "fields": ["title^3", "title.kw", "cast"],
          "fuzziness": "AUTO"
        }
      },
      "filter": {
        "term": { "city_ids": "blr" }
      }
    }
  }
}
```

#### “Now showing” ranking signals (MVP → advanced)

| Signal | Weight idea | Source |
| --- | --- | --- |
| Geo proximity to user | High | User city |
| Release recency | Medium | `release_date` |
| Popularity (7d tickets sold) | Medium | batch aggregate job → Redis |
| Editor picks / paid placements | Business override | Admin flags **must** be labeled in API for fairness |
| Diversity (language) | Low | avoid all one language in top 5 |

#### Index update path (CDC)

```text
PostgreSQL (movies) → Debezium → Kafka → OpenSearch consumer (idempotent by movie_id version)
```

Stale search **2–5 min** acceptable for browse; **never** rely on search index for seat counts.

---

### 🔴 For Advanced

- **Per-city availability facets:** maintained by periodic jobs counting future showtimes per movie × city — avoids scanning inventory service per search hit.  
- **Search-index stale consistency:** 1–5 minutes behind is fine; show **“Few seats left”** from cached counts with disclaimer.

#### Think About It

How would **A/B test** two ranking formulas without corrupting **booking** correctness?

#### Key Takeaways

- **Search** scales horizontally; **inventory** does not — keep separation crisp.  
- **Facets** updated asynchronously from schedule + availability aggregates.

#### Practice Exercise

Propose **three** signals for ranking “Now showing” beyond popularity.

### Extended detail: city feeds and “trending” integrity

- **Trending** must use **denormalized counts** from an analytics job — not live inventory scans — or a viral trailer could spike reads and starve booking.  
- **Editorial ordering:** expose `is_promoted` boolean in API so clients show subtle “Ad” where legally required.

### 🎯 Interview Questions — Discovery & search

**Q1:** Can OpenSearch be **strongly consistent** with seat inventory?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Not for seat-level truth. Use search/discovery for **candidates**; authoritative availability lives in **inventory + Postgres**. Badge “few seats left” from **async aggregates** with disclaimer if needed.

</details>

**Q2:** How do you update the search index when a movie’s metadata changes?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** CDC (Debezium) or app-level outbox → Kafka → indexer with **version** or `updated_at` ordering; handle **idempotent** upserts by `movie_id`.

</details>

---

## Section 9: Venues, Geo, Show Scheduling & Partner Integration

### What You'll Learn

Hall layouts as versioned JSON, timezone correctness, blackout windows, and partner sync options.

### Why This Matters

Wrong hall JSON = wrong seats sold — **version** hall definitions per showtime snapshot.

#### Concepts in this section (primer index)

**Showtime, venue, hall layout**, **IANA time zone and DST**, **PostGIS / geospatial** (city/venue), **mTLS and OAuth2 client credentials** (partners), **rate limiting** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

Each **screen** has rows and seat numbers; **VIP** vs **regular** blocks; **wheelchair** positions — modeled as seat metadata.

**Analogy:** A hall layout is like a **restaurant floor plan** in food delivery (tables vs booths) — static structure changes rarely, but **inventory** (who sits where) changes every second during an onsale. Version layouts like **menu versioning**: once a showtime references **layout v3**, you do not mutate v3 in place.

**Timezone trap (say in interview):** “Midnight show Friday” after DST ends — store `starts_at` as **UTC** + carry `venue_timezone`; render locally. Same discipline as scheduling delivery windows across regions.

---

### 🟡 For Intermediate

```text
hall_layout_version_id FK on showtimes
Seat static table: (hall_layout_version_id, seat_label, section, coords_json)
```

Bulk import via **admin portal** + CSV validation; dry-run against template.

**Partner API (aggregator mode):**

```http
POST /partners/v1/showtimes/sync
Authorization: Bearer m2m_token
```

Rate limits per partner; **HMAC** signatures on callbacks.

#### Schema sketch: venues and halls

```text
Table: venues (
  id BIGSERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  address_line TEXT,
  city_id BIGINT NOT NULL,
  lat DOUBLE PRECISION,
  lon DOUBLE PRECISION,
  timezone TEXT NOT NULL  -- IANA, e.g. Asia/Kolkata
)

Table: hall_layout_versions (
  id BIGSERIAL PRIMARY KEY,
  venue_id BIGINT NOT NULL,
  label TEXT NOT NULL,        -- e.g. "Screen 4 v3"
  layout_json JSONB NOT NULL, -- seats, rows, categories
  created_at TIMESTAMPTZ NOT NULL,
  UNIQUE (venue_id, label)
)

Table: showtimes (
  id BIGSERIAL PRIMARY KEY,
  movie_id BIGINT NOT NULL,
  venue_id BIGINT NOT NULL,
  hall_layout_version_id BIGINT NOT NULL REFERENCES hall_layout_versions(id),
  starts_at TIMESTAMPTZ NOT NULL,
  partner_showtime_ref TEXT,
  UNIQUE (partner_showtime_ref)  -- if partner supplies stable id
)
```

#### Geo queries

- **MVP:** filter by `city_id` (indexed).  
- **Nearby cinemas:** PostGIS `ST_DWithin` on venue point + index, or pre-bucket by city polygon.

---

### 🔴 For Advanced

- **DST transitions:** store `starts_at` in UTC; convert with `venue_timezone`.  
- **Split inventory:** partner holds back X% offline — expose **max_sellable** to our inventory allocator.

#### Think About It

What breaks if you **edit** a hall layout** after** some showtimes were already scheduled?

#### Key Takeaways

- **Immutable snapshots** for scheduled shows reference `hall_layout_version_id`.  
- **Partner** sync is **eventually consistent** with reconciliation reports.

#### Practice Exercise

Design a **nightly reconciliation** job comparing partner sold counts vs our `seat_sales`.

### Extended detail: partner sync and idempotent upsert

**Partner batch payload (illustrative):**

```json
{
  "partner_job_id": "pj_20260414_01",
  "showtimes": [
    {
      "partner_ref": "exo_998877",
      "movie_partner_ref": "m_5544",
      "starts_at": "2026-04-16T18:30:00+05:30",
      "hall_label": "Screen 4 v3",
      "currency": "INR",
      "base_price_cents": 35000
    }
  ]
}
```

Processing: upsert by `partner_showtime_ref` = hash(partner + partner_ref); persist job id in `partner_sync_runs` for replay.

### 🎯 Interview Questions — Venues & partner sync

**Q1:** Why is `hall_layout_version_id` on `showtimes` immutable-by-reference?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Changing seat labels or counts after sales started would desynchronize **sold seat labels**, manifests, and customer tickets. New layout → **new version row**; new showtimes only point forward.

</details>

**Q2:** How do you make partner **showtime sync** idempotent?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Partner supplies stable `partner_ref` + your system stores `partner_job_id` / batch dedupe. Retries of same batch produce **same** internal ids or no-op.

</details>

---

## Section 10: Digital Tickets, QR Validation & Anti-Fraud

### What You'll Learn

QR payload design (signed JWT or encrypted blob), rotation, offline gate scanners, and scalping controls.

### Why This Matters

Tickets are bearer instruments — **cryptographic signing** deters forgery; rate limits deter bots.

#### Concepts in this section (primer index)

**JWT and HMAC signing (tickets)**, **rate limiting**, **API Gateway**, **idempotency** (scan dedupe) — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

**QR** encodes ticket_id + show + nonce. Gate app scans → server marks **used** — prevents duplicate entry.

---

### 🟡 For Intermediate

```text
QR payload = base64url( HMAC-SHA256( ticket_id + showtime_id + nonce, rotating_secret ) )
TTL on secret: 90-day rotation; old secrets verify during overlap window
```

**Entry scan API:**

```http
POST /v1/entry/validate
Content-Type: application/json

{
  "ticket_id": "tkt_abc",
  "venue_device_id": "gate_7",
  "payload": "...."
}
```

**Response:** `{ "status": "valid_first_use" }` or `{ "status": "already_used", "used_at": "..." }`

Implement **idempotent** validate for poor connectivity (same device retries).

#### Signed envelope (JWT-style) sketch for QR payload

```json
{
  "header": { "alg": "HS256", "kid": "k2026Q2", "typ": "JWT" },
  "payload": {
    "ticket_id": "tkt_abc",
    "showtime_id": "st_4412",
    "venue_id": "ven_77",
    "seat_label": "F12",
    "iat": 1713120000,
    "exp": 1713141600
  }
}
```

Gate validator: verify signature with `kid` → fetch secret from KMS cache → check `exp` → check `ticket_id` not refunded → apply first-use rule.

#### Table: ticket_scans (first entry)

```text
Table: ticket_scans (
  ticket_id UUID PRIMARY KEY,  -- one row per ticket for first-use semantics
  first_scanned_at TIMESTAMPTZ NOT NULL,
  first_device_id TEXT NOT NULL,
  venue_id BIGINT NOT NULL
)
```

Use `INSERT ... ON CONFLICT DO NOTHING RETURNING *` — if no row returned, fetch existing and return `already_used`.

---

### 🔴 For Advanced

- **Offline mode:** venue devices cache **allowlists** of ticket_ids for next N hours — **security trade-off** vs availability during ISP outages.  
- **Screenshot sharing:** short-lived animated QR (drains battery) — product-heavy, optional.  
- **Scalping:** purchase limits, device graphs, **step-up** verification on high-risk purchases.

📊 **Example:** Large sports leagues issue **rotating barcodes**; cinema often simpler signed static until showtime.

#### Think About It

What happens if two gate scanners **double-tap** the same QR within 1 second?

#### Key Takeaways

- **HMAC/JWT** with key rotation; **first-use** semantics in DB.  
- **Risk** controls live in separate service to iterate policies fast.

#### Practice Exercise

Model `ticket_scans` table with uniqueness on `ticket_id` for first entry.

### Extended detail: scalper controls (product + engineering)

| Control | Implementation hint | Caveat |
| --- | --- | --- |
| Max tickets / user / show | `COUNT(*)` from `seat_sales` + user + showtime in same txn as checkout | Needs login |
| Max holds / IP / min | Gateway rate limit + softer captcha step-up | Shared NAT false positives |
| Payment velocity | PSP radar + internal velocity on `user_id` | May block legitimate corporates |
| Device binding | High-risk flows require app install + device_id | Privacy disclosure |

### 🎯 Interview Questions — Tickets & entry

**Q1:** Why not put **PII** in the QR payload?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** QR may be screenshotted, leaked, or logged by scanners. Prefer **opaque ticket_id** + signed JWT with **minimal claims**; load particulars server-side after auth/validation.

</details>

**Q2:** How do you handle **first-use** under double scan?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** `ticket_scans` primary key on `ticket_id`; first insert wins; subsequent returns `already_used` with same **idempotency key** for device retries.

</details>

---

## Section 11: Scalability & Performance

### What You'll Learn

Caching tiers, connection pooling, read replicas, autoscaling policies, and queue-backed spikes.

### Why This Matters

Scaling **web tier** is easy; scaling **correct inventory** is hard — don’t pretend all layers scale the same.

#### Concepts in this section (primer index)

**CDN**, **hot key / hot partition**, **Redis** primitives (see hold concepts), **read replica lag**, **Kafka** partitions/lag, **bulkhead**, **backpressure**, **SLO, SLI, error budget** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

**CDN** for posters; **Redis** for hot showtimes; **more app servers** behind LB for Diwali weekend traffic.

**Food-delivery parallel:** In food delivery you scale **matching** and **location** pipelines differently; here you scale **Discovery** (like restaurant browse) **horizontally**, but **Inventory** (like real-time stock for a hot SKU) needs **careful key design** — you cannot just “add servers” to Redis **writes** the way you do for stateless APIs.

**Rule of thumb:** If a diagram box handles **seat holds**, draw it **thinner** (fewer hops) and **hotter** (metrics) than browse boxes.

---

### 🟡 For Intermediate

```text
Cache hierarchy:
1. CDN: static assets, movie detail JSON (5–15 min TTL)
2. Redis: showtime metadata, seat holds, idempotency keys
3. PostgreSQL: orders + seat_sales (read replicas for reporting + analytics)

Autoscaling:
- Stateless APIs on CPU + p95 latency SLO
- Redis Cluster fixed shards; scale vertically + shard plan
- Kafka partitions = parallelism for notifications
```

#### Stampede control (popular trailer drop)

- **Probabilistic early refresh:** jitter CDN `Cache-Control` for JSON detail by **±15s** so retries don’t align.  
- **Request coalescing:** single-flight in app process for `movie:{id}` detail key.  
- **Seat map:** never stampede origin — always snapshot key + short TTL with **soft** degradation message if snapshot building.

#### Backpressure

If inventory CPU &gt; 80% sustained, return **429** with `Retry-After` on **hold** only — do not throttle browse APIs on same limiter (split rate limit namespaces).

---

### 🔴 For Advanced

- **Global:** geo-route reads; **inventory stays regional** per marketplace (India vs US catalog).  
- **Load test:** simulate **power users** hitting one showtime — not average QPS.

#### Think About It

When does adding **more Redis replicas** **not** increase write throughput for holds?

#### Key Takeaways

- Scale **reads** horizontally; **writes** need key design + sharding + admission control.

#### Practice Exercise

Define **SLOs** for inventory service separate from discovery API.

### Extended detail: SLO draft (usable in interviews)

| Service | SLI | Target | Error budget / month |
| --- | --- | --- | --- |
| Inventory hold API | p99 latency | &lt;300 ms | — |
| Inventory hold API | success (non-5xx) | 99.95% | ~21.6 min |
| Discovery API | p99 | &lt;150 ms | — |
| Checkout confirm path | e2e post-webhook to ticket row | &lt;5 s p99 | tied to PSP |

### 🎯 Interview Questions — Scalability & performance

**Q1:** Why don’t Redis **replicas** fix hot showtime writes?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Replicas serve **reads**; hold writes go to **primary**/shard master. Hot key needs **partitioning**, **queue**, or **admission control** — not more replicas.

</details>

**Q2:** Name one **load test** scenario that must exist for ticketing.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Thousands of concurrent clients targeting **one showtime_id** for holds — validates Lua, network to Redis, and GC pauses on inventory pods.

</details>

---

## Section 12: Security Considerations

### What You'll Learn

OAuth/OIDC for users, mTLS for partners, PCI scope minimization, secrets, and GDPR/CCPA flows.

### Why This Matters

You process payments via PSP — **SAQ-A** scope is the goal; don’t log card numbers.

#### Concepts in this section (primer index)

**PSP**, **PCI DSS and SAQ-A**, **3DS / SCA**, **KMS**, **GDPR and DSR**, **JWT**, **mTLS**, **webhook** security — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

We **don’t** store credit card numbers; PSP tokenizes. Users login with phone OTP or email.

---

### 🟡 For Intermediate

- **JWT access tokens** (15 min) + refresh rotation  
- **RBAC** for partner admin vs internal ops  
- **Rate limiting** at API Gateway (per IP + per user + per device)  
- **WAF** rules for OWASP Top 10  
- **PII** minimization in ticket payload logs

#### Threat model (condensed)

| Threat | Example | Control |
| --- | --- | --- |
| A01 Broken auth | stolen JWT | short TTL, refresh rotation, bind refresh to device for mobile |
| A03 Injection | SQL in search | parameterize + search escape |
| A10 SSRF | malicious trailer URL | URL allowlist domains, no redirect follower against private IPs |
| Abuse | hold bombing | rate limit + progressive penalty |
| Fraud | card testing small amounts | min amount, velocity with PSP |

#### Secrets

- **QR signing keys** in KMS; **90-day** rotation with overlap; **never** in git.  
- **Webhook signing secret** per environment; rotate via dual-secret window.

---

### 🔴 For Advanced

- **KMS-backed** signing keys for QR; annual penetration tests  
- **Audit** trail for operator actions on manual refunds  
- **Data residency** for EU users if expanded  
- **Bot mitigation:** TLS fingerprinting + **proof-of-work** CAPTCHA only on hot endpoints

#### Think About It

List **three** items that accidentally push you from **SAQ-A** to **SAQ-D**.

#### Key Takeaways

- **Defense in depth:** gateway + service + data layer controls.  
- **Least privilege** IAM for every service account.

#### Practice Exercise

Draft a **data deletion** request flow for a user who booked 20 historical orders.

### Extended detail: data subject request (DSR) outline

1. Verify identity (elevated channel).  
2. **Pause marketing** immediately; tickets may be **legal record** — legal review before hard delete.  
3. If delete approved: anonymize `users` PII; retain `orders` with `user_id` nulled where law allows aggregate-only retention.  
4. Propagate to search analytics with **delayed** batch job.

**Interview line:** “We separate **marketing profile** deletion from **financial record** retention.”

### 🎯 Interview Questions — Security & compliance

**Q1:** What pushes you from **SAQ-A** toward heavier PCI scope?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** Storing, processing, or transmitting **card data** in your systems (PAN in logs, custom vault, non-compliant iframe). Keep PAN entirely on PSP-hosted fields.

</details>

**Q2:** How do you protect **partner m2m** APIs?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** OAuth2 client credentials or mTLS, per-partner credentials, strict IP allowlists optional, signed webhooks, rate limits — same discipline as food-delivery restaurant integrations.

</details>

---

## Section 13: Monitoring & Observability

### What You'll Learn

Golden signals, business KPIs, distributed tracing, SLO/error budgets, and on-call playbooks.

### Why This Matters

Production incidents surface as **hold timeout spikes** or **webhook lag**, not generic CPU graphs.

#### Concepts in this section (primer index)

**OpenTelemetry and trace_id**, **SLO, SLI, error budget**, **Kafka** lag, **reconciliation**, **DLQ** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

Dashboards answer: **Is checkout healthy?** **Are seats stuck?**

---

### 🟡 For Intermediate

**Metrics (examples):**

```text
inventory_hold_conflict_rate (counter)
checkout_success_rate (ratio)
psp_webhook_latency_ms histogram
order_to_ticket_emit_lag_ms
seat_sales_db_insert_failures (must be ~0)
kafka_consumer_lag_seconds
```

**Alerts:**

- p99 `checkout` &gt; 5s for 5 minutes  
- Webhook error rate &gt; 1%  
- **Reconciliation backlog** &gt; 100 orphan charges  
- Redis memory usage &gt; 85%

**Tracing:** OpenTelemetry `trace_id` from API gateway through inventory → checkout → PSP adapter.

#### Structured log fields (minimum)

```json
{
  "trace_id": "a1b2c3",
  "order_id": "778899",
  "showtime_id": "st_4412",
  "user_id": "u_***masked",
  "psp_charge_id": "ch_9z9z",
  "event": "webhook_charge_succeeded_processed",
  "duration_ms": 47
}
```

#### Dashboard panels (one glance)

1. **Conversion funnel:** browse → seat map → hold_ok → pay_started → confirmed (rates per hour).  
2. **Integrity:** `seat_sales_insert_errors`, `duplicate_webhook_suppressed`.  
3. **Money:** orphan captures count, refund SLA breach count.  
4. **Inventory:** hot `showtime_id` by hold RPS.

---

### 🔴 For Advanced

- **SLO:** 99.95% successful checkout attempts excluding user card declines.  
- **Error budget** triggers freeze on risky releases near Diwali.  
- **Synthetic probes:** every 1 min simulate hold in staging mirror of prod config.

#### Think About It

Which metric is a **lagging** indicator vs **leading** indicator for inventory bugs?

#### Key Takeaways

- Pair **technical** and **business** metrics (GMV/hour, conversion).  
- **Correlation IDs** across PSP metadata and internal orders.

#### Practice Exercise

Write a **runbook** title list (5 entries) for PagerDuty scenarios.

### Extended detail: runbook titles + first step

| Runbook | First step |
| --- | --- |
| RB-Inventory Redis primary down | Fail over read-only; disable new holds; banner “booking paused” |
| RB-Webhook backlog | Scale consumers; check PG connections; DLQ depth |
| RB-Orphan PSP charges rising | Pause risky deploys; run reconciliation job in report-only |
| RB-Kafka lag ticket emails | Scale consumer pods; check poison messages |
| RB-Hot showtime abuse | Enable stricter rate limit + captcha for CIDR prefix |

### 🎯 Interview Questions — Monitoring & observability

**Q1:** Name one **leading** and one **lagging** metric for booking integrity.

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** **Leading:** spike in `hold_conflict_rate` or webhook **retry counts** before user-visible failures. **Lagging:** chargebacks per million orders or support tickets about “paid but no ticket” days later.

</details>

**Q2:** What goes on every **structured log line** for checkout?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** `trace_id`, `order_id`, `showtime_id`, `user_id` (masked), `psp` ids where relevant, `event` name, `duration_ms` — same philosophy as food delivery order pipeline logging.

</details>

---

## Section 14: Trade-Offs & Design Decisions

### What You'll Learn

Document major forks: Redis-first vs DB-first holds, monolith vs microservices, sync partner API vs batch feeds, polling vs WebSocket seat maps.

### Why This Matters

Interviewers probe **why not X**. Have crisp answers.

#### Concepts in this section (primer index)

**CAP theorem**, **hold vs sold**, **Redis vs PostgreSQL** split, **WebSocket vs polling**, **outbox**, **saga** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

**Trade-off:** Faster holds (**Redis**) vs simpler ops (**Postgres only**) — we choose Redis + Postgres for speed + durability split.

---

### 🟡 For Intermediate

| Decision | Option A | Option B | Pick |
|----------|----------|----------|------|
| Holds | Redis TTL | PG advisory locks | Redis for scale |
| Seat ID | Natural key (label) | Surrogate UUID | Label per showtime is human-meaningful |
| Real-time map | WebSocket | 2s polling | WebSocket nicer UX; polling simpler MVP |
| Search | Elastic | SQL LIKE | Elastic after 1k titles scaled |

| Monolith vs services | Modular monolith | 12 microservices day one | Monolith until team/ownership pain |
| PG vs Dynamo holds | Postgres conditional writes | Redis TTL + speed | Redis for interview scale path |
| Push notifications | Kafka fanout | Sync email in request | Kafka always for post-pay |
| Seat map transport | WebSocket push deltas | HTTP poll 2s | WS if budget for sock workers |

#### Decision log template (copy into design reviews)

| Decision | Options | Chosen | Revisit when |
| --- | --- | --- | --- |
| Hold store | Redis vs PG | Redis + PG sold | Single-key ops &gt;20k RPS sustained |
| Payment coupling | Sync charge vs async webhook | Webhook confirm + reconcile | PSP SLA changes |

---

### 🔴 For Advanced

- **Strong consistency across regions** for same showtime is **unnecessary** if shows are city-local — single-region inventory authority suffices.  
- **Microservices count** vs team size — start modular monolith if team &lt; 15 engineers; extract inventory when on-call pain proves boundaries.

#### Think About It

When would **DynamoDB** be a better seat store than **Redis + Postgres**?

#### Key Takeaways

- Every **latency** win has an **operational** cost — justify with SLOs.  
- **MVP** bias: ship polling + single region; **phase 2** WebSocket + multi-region DR.

#### Practice Exercise

Create a **decision log** entry for **polling vs WebSocket** with reversibility note.

### 🎯 Interview Questions — Trade-offs

**Q1:** When is **DynamoDB** reasonable for holds?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** AWS-native footprint, need **conditional writes + TTL** at huge scale with `TransactWriteItems`; team already operates Dynamo. Trade-off: different patterns than Redis Lua; still pair with **Postgres** for sold truth in most designs discussed in interviews.

</details>

**Q2:** Monolith first vs microservices first?

<details>
<summary>💭 Think first, then reveal answer</summary>

**Answer:** **Modular monolith** until clear team boundaries/on-call pain — same reasoning as `food_delivery_system_design.md` evolution stories; extract **inventory** first when contention and deploy risk dominate.

</details>

---

## Section 15: Interview Preparation

### What You'll Learn

45-minute flow, drill questions, troubleshooting trees, and evolution roadmap.

### Why This Matters

Ticketing interviews test **concurrency** + **payments** + **UX scale** — practice aloud.

#### Concepts in this section (primer index)

**Hold vs sold**, **idempotency**, **PSP / webhooks**, **hot keys**, **virtual waiting room / onsale** — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners

Memorize the **story**: browse → map → hold → pay → ticket.

---

### 🟡 For Intermediate: 45-minute outline

```text
0–5 min:  Requirements & clarifying questions
5–10 min: Capacity + hot key acknowledgment
10–20 min: Architecture diagram + data stores
20–32 min: Deep dive: inventory holds + idempotent checkout
32–40 min: Failure modes + monitoring
40–45 min: Extensions: search, anti-bot, internationalization
```

**Drill questions:**

1. How prevents double seat sale?  
2. Where idempotency key stored?  
3. What if Redis expires hold mid-payment?  
4. How to scale search but not inventory?  
5. Partner out-of-sync — who source of truth?

#### Full question bank (food-delivery style — practice cover to cover)

| # | Question |
| --- | --- |
| 1 | Draw the **data flow** from tap seat → sold, naming **each** store. |
| 2 | What happens on **double POST** `/checkout/sessions` without idempotency? |
| 3 | How do you **load test** a single showtime without cheating averages? |
| 4 | Explain **exactly-once** ticket email with Kafka. |
| 5 | Why **UNIQUE(showtime_id, seat_label)** and not a counter? |
| 6 | How does **GDPR** interact with invoices? |
| 7 | What metrics page do you open first in an incident? |
| 8 | How would you onboard a **new country** (currency + PSP + catalog)? |
| 9 | Compare **polling vs WebSocket** seat map for MVP. |
| 10 | What is the **business cost** of 1 minute of booking downtime during a blockbuster onsale? |

#### Timeboxed mock rubric (interviewer view)

| Time | You should have produced |
| --- | --- |
| 5 min | Actors, MVP scope, 2 clarifying constraints |
| 15 min | Box diagram + three datastores (CDN, Redis, PG) + PSP |
| 30 min | Hold path + webhook + idempotency + one failure mode |
| 45 min | Search freshness, abuse, monitoring — at least **mentions** |

---

### 🔴 For Advanced

- **CAP** positioning: **CP** for seat sales; **AP** for discovery with compensating UX.  
- **Evolution:** v1 single-city → multi-city → event vertical (sports) → resale marketplace.

#### Practice Exercise

Do one **full** mock explaining only **Section 6** for 12 minutes with diagram.

### Extended detail: troubleshooting tree (spoken)

```text
User says “payment went through but no ticket”
├─ Check order status in admin (`confirmed`?)
├─ If confirmed: ticket row exists? → email/SMS pipeline lag vs Kafka consumer
├─ If not confirmed: PSP dashboard shows charge? → reconciliation path
└─ If duplicate charge: idempotency key misuse → CS refund playbook

User says “seat taken but I was first”
├─ Compare client `requested_at` vs server hold logs (UTC)
├─ Check hold_conflict metrics for bot spike
└─ Explain race: only one NX wins — expected behavior
```

---

## Section 16: Putting It All Together

### What You'll Learn

Synthesized architecture checklist, mental model, and curated resources.

### Why This Matters

Solid narratives win “hire” — connect **business** (partners), **UX**, and **engineering** invariants.

#### Concepts in this section (primer index)

Full glossary: **B2B2C**, inventory (**hold vs sold**), **payments** (**PSP**, **reconciliation**), **tickets** (**JWT/HMAC**), **observability** (**SLO**, **OTel**) — see **[Concept primer](#concept-primer-subsections-for-terms-used-in-this-document)**.

---

### 🟢 For Beginners: The Story in Five Bullets

1. **Browse** is cacheable and eventually fresh.  
2. **Seat map** is read-heavy; **hold** is write-contended — design for the hotspot.  
3. **Pay** uses a trusted PSP; **our job** is idempotent orders.  
4. **Tickets** are signed credentials for entry.  
5. **Observe** business + system metrics — tickets are revenue.

---

### 🟡 For Intermediate: Pre-Submission Checklist

```text
□ Clarified MVP vs P1 features
□ Drew API gateway + inventory + checkout + PSP
□ Wrote hold acquire pseudocode (NX + TTL)
□ Explained webhook → DB transaction ordering
□ Listed 8+ metrics and 3 alerts
□ Mentioned reconciliation for orphan payments
```

---

### 🔴 For Advanced: Real-World Anchors

📊 **Examples:**

| Company | Note |
|---------|------|
| **BookMyShow** | India-scale multi-city aggregator + payments diversity (UPI, wallets) |
| **Fandango** | US studio integrations, heavy mobile |
| **Ticketmaster** | Extreme queueing for stadium events — study **Verified Fan** patterns |

---

### Resources

```text
- Stripe: Idempotency keys documentation
- Redis: SET NX EX patterns; Redis Cluster documentation
- OWASP API Security Top 10
- Google SRE: SLO chapter (free workbook)
```

#### Additional reading (ticketing-adjacent)

- **Ticketmaster / Live Nation engineering** public posts on **queue** and onsale load (conceptual, not proprietary).  
- **PCI SSC:** SAQ-A vs SAQ-D decision trees (merchant docs).  
- **OpenTelemetry** semantic conventions for HTTP + messaging — align `trace_id` across GW, services, Kafka consumers.

#### Cross-reference: same repository (compare systems)

- **`food_delivery_system_design.md`** — Three-sided marketplace, state machines, payments, geospatial, WebSocket; **compare** “driver matching hot path” ↔ “single showtime hold hot path” in mocks.  
- **`payment_gateway_system_design.md`** — Idempotency, webhooks, reconciliation, PCI depth — **reuse** those narratives for PSP sections here.

#### Next steps (food delivery “Next Steps & Resources” style)

1. Re-read **Section 6–7** aloud twice — inventory + money.  
2. Whiteboard **one** failure: orphaned charge — write the **reconciliation** pseudocode.  
3. Speed-run **Section 15** question bank with a timer (**45 min** max).  
4. Optionally sketch **queue-it** pre-waiting room as Phase-2 feature without blocking MVP narrative.

---

### Final Key Takeaways

- **Correctness first** on seats; **speed** second, achieved via caching where safe.  
- **Payments** are never “fire and forget” — **webhooks + reconciliation**.  
- **Product** complexity (events, dynamic pricing) grows in phases — keep the **core inventory story** crisp in interviews.

---

### Practice Exercise (Capstone)

Design **queue-it** style virtual waiting room for **one** 80,000-seat stadium onsale — list **three** system components and **two** fairness pitfalls (bots, geography).

---

## Expanded supplements (inline — same document)

Much of the **deepening content now lives inside Sections 3–16** under **“Extended detail”** subsections so it sits next to the original teaching flow. The short notes below remain as a **quick cross-reference**; prefer the main sections for reading order.

### Supplement aligned to Section 3: Personas and policy matrix

| Persona | Primary outcome | Failure mode | Primary lever |
| --- | --- | --- | --- |
| Casual browser | Fast “tonight near me” | Wrong city defaults | Geo-IP + last city + explicit switch |
| Opening-night fan | Exact seats | Double sale / pay without seats | Redis hold + PG uniqueness + idempotent checkout |
| Family | Adjacent seats | Split parties | Lua multi-seat hold or server-side adjacency solver |
| Exhibitor partner | Accurate gate manifest | Oversell vs hall | `seat_sales` export + nightly reconciliation |
| Risk / fraud | Few chargebacks | Card testing | Velocity + device graphs + step-up 3DS |

**Regulatory snapshot:** persist **price breakdown** (base + tax + convenience fee) on `orders` at pay time — marketing copy may change later; legal truth is the snapshot row.

### Supplement aligned to Section 4: Extra capacity anchors

**Evening peak math (sketch):** If browse is 12,000 RPS average origin-equivalent and Friday 7–10 pm is **8×** that slice’s share, you may see **localized** discovery pools at **~6–10k RPS** still served mostly from edge. Inventory path remains the bottleneck: one hot showtime can still hit **500–1,500 RPS** on seat operations — averages never capture that.

**Partitioning cue:** `seat_sales` by month on `created_at` keeps index bloat bounded; hot queries for “tonight’s shows” stay on schedule + cache layers.

### Supplement aligned to Section 5: BFF and bulkheads

**BFF aggregates** (mobile): `/v1/feed/movie/{id}` might combine catalog + “next showtimes in user’s city” + hero image URLs — fewer round trips, but **don’t** merge inventory into discovery responses unless product accepts staleness of a few seconds.

**Bulkhead thread pools:** Pool A for read-heavy discovery; Pool B for inventory/checkout. If checkout stalls, browsing still works — better than one pool deadlock across the service.

### Supplement aligned to Section 6: Hold invariants (checklist)

```text
□ Sold seat exists in PostgreSQL → never also “available” in cache without invalidation
□ Redis hold TTL always < payment SLA window or aligned with client countdown
□ Multi-seat hold is atomic (Lua) or compensating delete on failure
□ Background job reconciles: orphan holds older than TTL + slack (crash mid-release)
□ Load test: single showtime artifact with k6/Locust at 2× expected peak hold RPS
```

### Supplement aligned to Section 7: PSP event deduplication

Store PSP `event_id` in `payment_events` with `PRIMARY KEY (psp_event_id)` so duplicate webhooks are **no-ops**. Link to `order_id` for support lookups. Return **200** after dedupe so PSP stops retrying endlessly.

### Supplement aligned to Section 8: Search vs inventory truth

**Rule of thumb:** Search may say “playing near you”; only inventory + schedule truth can say “seats available now.” If you show **“selling fast”** from aggregated counts, refresh counts every **60–120 seconds** and label as **estimated** to avoid bait-and-switch complaints.

### Supplement aligned to Section 9: Immutable hall snapshots

When `hall_layout_version_id` is tied to `showtimes`, operations team edits for **next month** create **new** layout version rows — never mutate rows referenced by active showtimes. Migration tool can clone layout with new version label.

### Supplement aligned to Section 10: Double scan at gate

Two scanners **500 ms apart** on same ticket: second response is `already_used` with `first_scan_at` — idempotent response for same device, alert if two **different** device_ids in 1 second (possible screenshot sharing).

### Supplement aligned to Section 11: Autoscaling signal

Combine **CPU**, **p95 latency**, and **request queue depth** — for JVM services CPU alone misleads during GC. For inventory service, add **Redis OP latency** and **hold attempt rejection rate**.

### Supplement aligned to Section 12: SAQ-A boundary

Hosted fields / redirect to PSP keeps most teams in **SAQ-A**. Logging full PAN, storing it, or custom encryption vault in-app pushes scope — say that aloud in interview.

### Supplement aligned to Section 13: Leading vs lagging metrics

- **Leading:** `hold_conflict` derivative up before revenue drops — might mean attack or bug pre-release  
- **Lagging:** `chargebacks per million orders` — tells you weeks later

### Supplement aligned to Section 14: When DynamoDB for holds?

Worth considering **if** you already run AWS-native, want TTL + conditional writes at massive scale, and can model holds as items `(showtime_id, seat_id)` with `TransactWriteItems`. Trade-off: different ops model vs Redis — many teams still pick **Redis + Postgres** for familiarity.

---

## Appendix A: Micro-drill index (concurrency and money)

Use these as **spoken drills** (60–90 seconds each). Each references the same architecture: **Redis holds + Postgres sold + PSP + webhook + idempotency.**

#### Drill M-01 — Double tap same seat

**Prompt:** Two phones tap **F12** within 20 ms.  
**Answer path:** Second `SET NX` fails → 409 or conflict payload → client refreshes map; **no** sold row yet.

#### Drill M-02 — Hold expires mid-OTP

**Prompt:** User is entering bank OTP; hold TTL hits 0.  
**Answer path:** Payment may still complete depending on product; **recommended** strict mode: cancel PSP intent if seats released; **lenient** mode: re-hold if still free (race). State which you pick and why.

#### Drill M-03 — Webhook duplicate

**Prompt:** PSP sends identical `charge.succeeded` twice.  
**Answer path:** Dedupe table on `psp_event_id`; second insert conflict → exit 200 without re-emitting Kafka ticket twice (or use outbox idempotent producer keys).

#### Drill M-04 — PG commit failed after PSP success

**Prompt:** Money captured, DB transaction rolled back.  
**Answer path:** Alert + reconciliation job finds orphan capture → refund OR complete order if idempotent replay safe — never silent loss.

#### Drill M-05 — Read replica lag on “is seat free?”

**Prompt:** User polls map fed from replica 2 s behind; seat sold.  
**Answer path:** Authoritative path for **hold attempt** hits primary or inventory service that does **not** trust stale read for mutation — replica OK for gray UI until user acts.

#### Drill M-06 — Hot showtime Redis

**Prompt:** Single `showtime_id` saturates one Redis shard.  
**Answer path:** Admission control / queue; optional shard by seat segments with orchestration; never “just add replicas” for writes.

#### Drill M-07 — Partner count mismatch

**Prompt:** Partner CSV says 120 sold; we have 118 `seat_sales`.  
**Answer path:** Breakglass reconciliation CSV; clock on export boundaries; timezone; refund window — triage with support playbook.

#### Drill M-08 — QR secret rotation

**Prompt:** Rotate signing key every 90 days with overlap window.  
**Answer path:** `kid` header in JWT / QR envelope; validators try `{kid: current, previous}`; old tickets still scan until expiry.

---

## Appendix B: Stretch interview questions (short form)

For each, use **🟢 outcome → 🟡 mechanism → 🔴 failure + metric.**

| ID | Question |
| --- | --- |
| Q1 | Why not skip Redis and only use Postgres row locks for holds? |
| Q2 | How does idempotency key interact with changing cart contents? |
| Q3 | What is your split of sync vs async after payment confirmation? |
| Q4 | How would you model “best available” seating without a map UI? |
| Q5 | What caches are you willing to stampede during viral trailer release? |
| Q6 | How do you test correctness of seat allocation under concurrency? |
| Q7 | When would you shard Postgres by `city_id` vs `order_id`? |
| Q8 | How does GDPR erasure work when tickets are fiscal receipts? |
| Q9 | Describe multi-tenant partner isolation in admin APIs. |
| Q10 | How do you rate-limit holds without hurting real users on Wi‑Fi NAT? |

---

## Appendix C: Glossary (expanded)

| Term | Meaning |
| --- | --- |
| Hold | Short-lived reservation in Redis (`SET NX` + TTL) before payment |
| Sold | Durable fact in Postgres `seat_sales` with uniqueness on `(showtime_id, seat_label)` |
| Onsale | Time window when inventory first opens — often causes queue |
| PSP | Payment service provider (Stripe, Razorpay, etc.) |
| SAQ-A | PCI self-assessment for merchants outsourcing card entry |
| Manifest | Per-showtime list of sold seats for venue ops |
| Outbox | DB table + publisher ensuring Kafka emits match commits |

---

**End of document.**
