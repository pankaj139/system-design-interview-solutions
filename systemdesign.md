<!-- Formatted from PDF export: soft hyphens removed, page numbers stripped; Front matter + Sections 1–8 rebuilt from systemdesign.md.bak -->
# 200+ System Design Interview Questions & Answers

_A complete, modern, FAANG-level preparation guide_

_From Fundamentals to Distributed Systems Mastery_

## Contents

- [Section 1 — System Design Fundamentals](#section-1-system-design-fundamentals)
- [Section 2 — System Design Questions](#section-2-system-design-questions)
- [Section 3 — Interview Theory Questions](#section-3-interview-theory-questions)
- [Section 4 — Cheat Sheets](#section-4-cheat-sheets)
- [Section 5 — ASCII Diagram Library](#section-5-ascii-diagram-library)
- [Section 6 — Interview Checklists](#section-6-interview-checklists)
- [Section 7 — Flashcards](#section-7-flashcards)
- [Section 8 — Multiple Choice Questions](#section-8-multiple-choice-questions)


## Preface

System design has become the single most important — and most feared — round of the modern software-engineering interview. Coding rounds test whether you can write correct code; system design rounds test whether you can build companies. They ask: given ambiguous requirements, unpredictable scale, unreliable networks, and finite money, can you design a system that stays up, stays fast, and stays affordable?

This book was written for engineers at every stage of that journey. Whether you are a student preparing for your first internship, a backend developer preparing for a senior promotion, a full-stack engineer stepping into architecture, or a staff engineer coaching juniors, you will find something in these pages that sharpens your thinking.

**What this book gives you:**

- Section 1 — Fundamentals. Every core concept a system designer must own, explained from first principles with pros, cons, real-world examples, and interview-ready summaries.
- Section 2 — Fully worked design questions. Each question follows the same battle-tested framework FAANG interviewers use: requirements, scale estimation, API, database, high-level architecture, deep dives, trade-offs, and failure handling.
- Section 3 — Theory Q&A. Rapid-fire conceptual comparisons — Kafka vs RabbitMQ, SQL vs NoSQL, JWT vs Session — the “gotcha” questions that reveal depth of understanding.
- Section 4 — Cheat sheets. Distilled one-page references for last-minute revision.
- Section 5 — ASCII diagram library. Reusable architectures you can adapt in whiteboard or virtual interviews.
- Section 6 — Checklists. The mental checklists senior engineers run through unconsciously.
- Section 7 — Flashcards. Spaced-repetition friendly Q/A pairs.
- Section 8 — MCQs. Timed self-assessment with detailed explanations.

**How to get the most out of this book:**

Do not read it front-to-back like a novel. Read Section 1 slowly. Then, when preparing for an interview, pick five questions from Section 2 and whiteboard them yourself before reading the solution. Compare, refine, repeat. Use Sections 7 and 8 for daily maintenance.

Above all, remember: system design is a conversation, not a monologue. The frameworks in this book give you scaffolding. Your judgment, curiosity, and communication turn scaffolding into architecture.

Good luck.

## How to Use This Book

- For students and freshers: Start with Section 1 (Fundamentals). Do not skip. Every advanced question later assumes you own these concepts. Then attempt easy questions in Section 2.
- For working backend / full-stack engineers: Skim Section 1 for gaps. Work through medium and hard questions in Section 2. Use Section 3 to shore up rusty comparisons.
- For senior engineers, tech leads, and architects: Focus on Section 2 hard/expert questions and the trade-offs subsection. Interviewers at senior levels care less about your first design and more about how you evolve it under pressure.
- For interviewers: Sections 2 and 3 double as an interview question bank. Section 6 checklists help calibrate evaluations.

**Study rhythm we recommend:**

| Week | Focus |
| --- | --- |
| 1–2 | Section 1 (Fundamentals), flashcards daily |
| 3–4 | Section 2 easy + medium questions |
| 5–6 | Section 2 hard + expert questions |
| 7 | Section 3 theory + Section 4 cheat sheets |
| 8 | MCQ marathon + mock interviews |

## Roadmap for Learning System Design

```text
┌───────────────────────────────────┐
│ Networking Basics │
│ HTTP · TCP · DNS · TLS · OSI │
└────────────────┬──────────────────┘
│
┌────────────────┴──────────────────┐
│ Databases & Storage │
│ SQL · NoSQL · Indexes · Sharding │
└────────────────┬──────────────────┘
│
┌────────────────┴──────────────────┐
│ Caching, Queues & Load Balance │
│ Redis · Kafka · L4/L7 · CDN │
└────────────────┬──────────────────┘
│
┌────────────────┴──────────────────┐
│ Distributed Systems Theory │
│ CAP · PACELC · Consensus · Locks │
└────────────────┬──────────────────┘
│
┌────────────────┴──────────────────┐
│ Microservices & Cloud │
│ Docker · K8s · Service Mesh │
└────────────────┬──────────────────┘
│
┌────────────────┴──────────────────┐
│ Practice Design Questions │
│ Whiteboard · Trade-off · Iterate │
└───────────────────────────────────┘
```

**Suggested learning order:**

1. Foundations — Networking, HTTP, TCP/UDP, DNS, TLS.
2. Data — Relational vs non-relational, indexes, transactions, ACID vs BASE.
3. Scale primitives — Load balancing, caching, CDN, queues, replication, sharding.
4. Distributed theory — CAP, PACELC, consistency models, consensus (Raft/Paxos in outline), leader election.
5. Operational concerns — Observability, tracing, circuit breakers, retries, idempotency.
6. Architectural styles — Monolith, microservices, event-driven, CQRS, event sourcing.
7. Practice — Solve design questions daily, iterate on trade-offs.

## System Design Interview Strategy

A typical 45–60 minute system-design interview follows a rhythm. Missing the rhythm is the #1 reason strong engineers fail.

- Minute 0–5 — Clarify. Never start drawing. Ask about scope, users, features, geography, and constraints. Explicitly separate must-have from nice-to-have.
- Minute 5–10 — Estimate scale. Reads/sec, writes/sec, storage/year, bandwidth. Numbers anchor every downstream decision. Round aggressively — the interviewer wants Fermi reasoning, not spreadsheets.
- Minute 10–20 — API and data model. Sketch a handful of endpoints (or gRPC methods). Pick a data model. State whether it fits SQL or NoSQL and why. If NoSQL, name the access patterns.
- Minute 20–35 — High-level architecture. Client → CDN → LB → API → Cache → DB. Add queues, workers, storage. Draw boxes and arrows. Label protocols.
- Minute 35–50 — Deep dive. The interviewer will pick 1–2 components (usually the hardest: feed ranking, real-time sync, exactly-once, geo-sharding). Go deep. Discuss data structures, algorithms, failure modes.
- Minute 50–55 — Bottlenecks, scale, trade-offs. “What breaks first? How do we fix it? What did we trade away?”
- Minute 55–60 — Wrap. Summarize the design, name the trade-offs, mention alternatives you considered.

The four evaluation axes interviewers score you on:

| Axis | What they look for |
| --- | --- |
| Requirements gathering | Clarifying questions, prioritization |
| Design breadth | Do you know the palette of building blocks? |
| Design depth | Can you go 5 layers deep on one component? |
| Communication | Structured, collaborative, humble under pushback |

## Common Mistakes

1. Jumping to a solution before clarifying requirements. Slow down.
2. Ignoring non-functional requirements (latency, availability, cost). These drive architecture more than features do.
3. Over-engineering. A URL shortener does not need Kafka + microservices on day one.
4. Under-engineering. A global chat app cannot live on a single Postgres instance.
5. Hand-waving away consistency. Say which consistency model you want and why.
6. Forgetting failure modes. Every dependency is a future outage. Discuss retries, timeouts, circuit breakers.
7. Not estimating scale. Without numbers, choices are arbitrary.
8. Talking about tools instead of trade-offs. “I would use Kafka” is not an answer. “I need durable, ordered, replayable messages with backpressure — Kafka fits; the trade-off is operational complexity” is.
9. Being defensive under pushback. The interviewer poking your design is part of the interview. Engage.
10. Running out of time on the intro. Budget minutes explicitly.

## Section 1 — System Design Fundamentals

This section defines every foundational concept a system designer must master. Each entry is compact but complete: definition, why it matters, trade-offs, real-world use, and interview questions.

### Section 1 fundamentals index

- [1.1 What is System Design?](#11-what-is-system-design)
- [1.2 Functional Requirements](#12-functional-requirements)
- [1.3 Nonfunctional Requirements (NFRs)](#13-non-functional-requirements-nfrs)
- [1.4 Scalability](#14-scalability)
- [1.5 Availability](#15-availability)
- [1.6 Reliability](#16-reliability)
- [1.7 Durability](#17-durability)
- [1.8 Fault Tolerance](#18-fault-tolerance)
- [1.9 Consistency](#19-consistency)
- [1.10 Latency](#110-latency)
- [1.11 Throughput](#111-throughput)
- [1.12 CAP Theorem](#112-cap-theorem)
- [1.13 PACELC](#113-pacelc)
- [1.14 Distributed Systems](#114-distributed-systems)
- [1.15 Monolith vs Microservices](#115-monolith-vs-microservices)
- [1.16 Horizontal Scaling](#116-horizontal-scaling)
- [1.17 Vertical Scaling](#117-vertical-scaling)
- [1.18 Replication](#118-replication)
- [1.19 Partitioning](#119-partitioning)
- [1.20 Sharding](#120-sharding)
- [1.21 Caching](#121-caching)
- [1.22 Load Balancer](#122-load-balancer)
- [1.23 Database Indexes](#123-database-indexes)
- [1.24 Database Normalization](#124-database-normalization)
- [1.25 SQL vs NoSQL](#125-sql-vs-nosql)
- [1.26 CDN (Content Delivery Network)](#126-cdn-content-delivery-network)
- [1.27 Message Queue](#127-message-queue)
- [1.28 EventDriven Architecture](#128-eventdriven-architecture)
- [1.29 REST](#129-rest)
- [1.30 GraphQL](#130-graphql)
- [1.31 gRPC](#131-grpc)
- [1.32 WebSockets](#132-websockets)
- [1.33 API Gateway](#133-api-gateway)
- [1.34 Authentication](#134-authentication)
- [1.35 Authorization](#135-authorization)
- [1.36 OAuth 2.0](#136-oauth-20)
- [1.37 JWT (JSON Web Token)](#137-jwt-json-web-token)
- [1.38 Session Management](#138-session-management)
- [1.39 Rate Limiting](#139-rate-limiting)
- [1.40 Reverse Proxy](#140-reverse-proxy)
- [1.41 Forward Proxy](#141-forward-proxy)
- [1.42 DNS (Domain Name System)](#142-dns-domain-name-system)
- [1.43 HTTP](#143-http)
- [1.44 HTTPS](#144-https)
- [1.45 TCP](#145-tcp)
- [1.46 UDP](#146-udp)
- [1.47 SSL/TLS](#147-ssltls)
- [1.48 CIDR](#148-cidr)
- [1.49 Network Basics](#149-network-basics)
- [1.50 OSI Model](#150-osi-model)
- [1.51 Cloud Basics](#151-cloud-basics)
- [1.52 Containers](#152-containers)
- [1.53 Docker](#153-docker)
- [1.54 Kubernetes](#154-kubernetes)
- [1.55 Service Discovery](#155-service-discovery)
- [1.56 Monitoring](#156-monitoring)
- [1.57 Logging](#157-logging)
- [1.58 Observability](#158-observability)
- [1.59 Tracing](#159-tracing)
- [1.60 Circuit Breaker](#160-circuit-breaker)
- [1.61 Retry Pattern](#161-retry-pattern)
- [1.62 Saga Pattern](#162-saga-pattern)
- [1.63 CQRS (Command Query Responsibility Segregation)](#163-cqrs-command-query-responsibility-segregation)
- [1.64 Event Sourcing](#164-event-sourcing)
- [1.65 Bloom Filter](#165-bloom-filter)
- [1.66 Consistent Hashing](#166-consistent-hashing)
- [1.67 Leader Election](#167-leader-election)
- [1.68 Distributed Lock](#168-distributed-lock)
- [1.69 ExactlyOnce Delivery](#169-exactlyonce-delivery)
- [1.70 AtLeastOnce Delivery](#170-atleastonce-delivery)
- [1.71 AtMostOnce Delivery](#171-atmostonce-delivery)

### 1.1 What is System Design?

#### Definition

The discipline of defining the architecture, components, interfaces, and data flow of a software system to satisfy specified requirements at scale.

#### Why it matters

It bridges business requirements and engineering execution, determining whether a product can grow, stay reliable, and evolve.

#### Advantages

- Aligns engineering with business goals
- Prevents costly re-architecture later
- Improves reliability and cost-efficiency

#### Disadvantages / Trade-offs

- Time-consuming up-front
- Requires broad expertise
- Wrong choices lock in tech debt

#### Real-world example

Designing WhatsApp’s messaging pipeline to handle 100B+ msgs/day.

#### Interview questions

- How do you approach a new system design?
- What separates a good design from a great design?

#### Summary

What is System Design? — The discipline of defining the architecture, components, interfaces, and data flow of a software system to satisfy specified requirements at scale.

### 1.2 Functional Requirements

#### Definition

Explicit features the system must provide — what the system does.

#### Why it matters

They define the product surface: APIs, workflows, user actions. Missing them means the system fails its purpose.

#### Advantages

- Concrete and testable
- Drive API and data model

#### Disadvantages / Trade-offs

- Can explode in scope
- Often change during development

#### Real-world example

“Users can post photos, follow others, and view a feed” (Instagram).

#### Interview questions

- Give three functional requirements for Uber.
- How do you prioritize FRs?

#### Summary

Functional Requirements — Explicit features the system must provide — what the system does.

### 1.3 Nonfunctional Requirements (NFRs)

#### Definition

Quality attributes: latency, availability, throughput, security, cost, maintainability — how well the system behaves.

#### Why it matters

NFRs shape architecture more than FRs. A chat app that is functional but takes 10s to deliver a message is broken.

#### Advantages

- Drive architecture and infra choices
- Measurable via SLOs

#### Disadvantages / Trade-offs

- Often vague if unquantified
- Trade off against each other

#### Real-world example

“p99 message delivery < 500ms globally; 99.99% availability.”

#### Interview questions

- Name five NFRs for a payments system.
- How do NFRs conflict with each other?

#### Summary

Non-functional Requirements (NFRs) — Quality attributes: latency, availability, throughput, security, cost, maintainability — how well the system behaves.

### 1.4 Scalability

#### Definition

The ability of a system to handle growing load — more users, more data, more requests — by adding resources.

#### Why it matters

Every consumer-scale product hits scaling walls. Systems that cannot scale die or get rewritten.

#### Advantages

- Handles growth without redesign
- Elastic cost model

#### Disadvantages / Trade-offs

- Adds coordination overhead
- Harder to reason about

#### Real-world example

Netflix scaling to 260M+ subscribers across 190 countries.

#### Interview questions

- Vertical vs horizontal scaling — when to pick which?
- Name three bottlenecks that break scalability.

#### Summary

Scalability — The ability of a system to handle growing load — more users, more data, more requests — by adding resources.

### 1.5 Availability

#### Definition

The percentage of time the system is operational and reachable. Commonly expressed as “nines” (99.9%, 99.99%).

#### Why it matters

Downtime = lost revenue, lost trust. SLAs are contractual.

#### Advantages

- Directly measurable
- Aligns with SLAs

#### Disadvantages / Trade-offs

- Higher availability = exponentially higher cost
- Requires redundancy at every layer

#### Real-world example

AWS S3 targets 99.99% availability, 99.999999999% durability.

#### Interview questions

- What does 99.99% availability mean in minutes/year?
- How do you design for 5-nines?

#### Summary

Availability — The percentage of time the system is operational and reachable.

### 1.6 Reliability

#### Definition

Probability that the system performs correctly over a period of time. Broader than availability — includes correctness.

#### Why it matters

An available-but-wrong system is worse than a down system. Reliability is trust.

#### Advantages

- Captures correctness + uptime
- Drives fault-tolerant design

#### Disadvantages / Trade-offs

- Hard to measure holistically
- Requires end-to-end testing

#### Real-world example

Banking ledger: never lose or double-count a transaction.

#### Interview questions

- Difference between availability and reliability?

#### Summary

Reliability — Probability that the system performs correctly over a period of time.

### 1.7 Durability

#### Definition

Once data is acknowledged as written, it will not be lost — even under hardware failure.

#### Why it matters

Loss of user data is often unrecoverable business damage.

#### Advantages

- Protects against data loss

#### Disadvantages / Trade-offs

- Requires replication, adds cost and latency

#### Real-world example

S3′s 11 nines of durability via cross-AZ replication.

#### Interview questions

- How does S3 achieve 11 nines?
- Durability vs availability?

#### Summary

Durability — Once data is acknowledged as written, it will not be lost — even under hardware failure.

### 1.8 Fault Tolerance

#### Definition

The ability of a system to continue operating correctly despite the failure of some of its components.

#### Why it matters

In distributed systems, failures are the norm, not the exception.

#### Advantages

- Prevents cascading outages
- Improves user experience

#### Disadvantages / Trade-offs

- Adds complexity
- Requires redundancy

#### Real-world example

Kafka replicating partitions across brokers; if one dies, another serves.

#### Interview questions

- How do circuit breakers help fault tolerance?

#### Summary

Fault Tolerance — The ability of a system to continue operating correctly despite the failure of some of its components.

### 1.9 Consistency

#### Definition

All nodes see the same data at the same time (strong consistency), or eventually converge (eventual consistency).

#### Why it matters

Wrong consistency choice = corrupted state or bad UX.

#### Advantages

- Simplifies reasoning (when strong)
- Improves availability (when eventual)

#### Disadvantages / Trade-offs

- Strong consistency limits availability (CAP)
- Eventual consistency confuses users

#### Real-world example

DynamoDB offers both eventually-consistent (cheap, fast) and strongly-consistent reads.

#### Interview questions

- Explain linearizability vs sequential consistency.

#### Summary

Consistency — All nodes see the same data at the same time (strong consistency), or eventually converge (eventual consistency).

### 1.10 Latency

#### Definition

Time taken for a single request to be processed end-to-end.

#### Why it matters

Amazon found +100ms latency = −1% revenue. Google: +400ms = −0.6% searches.

#### Advantages

- User-perceived quality metric

#### Disadvantages / Trade-offs

- Hard to reduce below physics (speed of light)

#### Real-world example

Google search targets p50 < 200ms.

#### Interview questions

- Latency vs response time?
- How to reduce tail latency?

#### Summary

Latency — Time taken for a single request to be processed end-to-end.

### 1.11 Throughput

#### Definition

Number of requests or operations processed per unit time.

#### Why it matters

Determines capacity planning and cost.

#### Advantages

- Easy to measure and optimize

#### Disadvantages / Trade-offs

- Can be increased at the cost of latency (batching)

#### Real-world example

Kafka clusters routinely process millions of messages/sec.

#### Interview questions

- Latency vs throughput trade-off?

#### Summary

Throughput — Number of requests or operations processed per unit time.

### 1.12 CAP Theorem

#### Definition

In the presence of a network Partition, a distributed system must choose between Consistency and Availability.

#### Why it matters

Frames the fundamental trade-off in every distributed data store.

#### Advantages

- Simple mental model

#### Disadvantages / Trade-offs

- Oversimplifies — reality is a spectrum (see PACELC)

#### Real-world example

MongoDB (CP by default), Cassandra (AP), traditional RDBMS single-node (CA).

#### Interview questions

- Give an example of AP and CP databases.
- Is CA possible in a distributed system?

#### Summary

CAP Theorem — In the presence of a network Partition, a distributed system must choose between Consistency and Availability.

### 1.13 PACELC

#### Definition

Extension of CAP: if Partitioned, choose A or C; Else, choose Latency or Consistency.

#### Why it matters

Captures the everyday trade-off (latency vs consistency) that CAP ignores.

#### Advantages

- Realistic model of database behavior

#### Disadvantages / Trade-offs

- Less well-known

#### Real-world example

DynamoDB is PA/EL; Spanner is PC/EC.

#### Interview questions

- Classify Cassandra and Spanner in PACELC.

#### Summary

PACELC — Extension of CAP: if Partitioned, choose A or C; Else, choose Latency or Consistency.

### 1.14 Distributed Systems

#### Definition

A system whose components run on multiple networked computers coordinating via messages.

#### Why it matters

Every large-scale system is distributed. Fundamentals: partial failures, network delays, no global clock.

#### Advantages

- Scale and fault tolerance

#### Disadvantages / Trade-offs

- Complexity: consensus, ordering, split-brain

#### Real-world example

Google Spanner spans continents with globally consistent transactions.

#### Interview questions

- Name three fallacies of distributed computing.

#### Summary

Distributed Systems — A system whose components run on multiple networked computers coordinating via messages.

### 1.15 Monolith vs Microservices

#### Definition

Monolith: single deployable unit. Microservices: many small, independently deployable services.

#### Why it matters

Choice affects team velocity, deployment cadence, and blast radius of failures.

#### Advantages

- Microservices: independent scaling, tech diversity
- Monolith: simpler ops, easier transactions

#### Disadvantages / Trade-offs

- Microservices: distributed complexity, network calls
- Monolith: harder to scale teams

#### Real-world example

Amazon and Netflix famously migrated monolith → microservices.

#### Interview questions

- When is a monolith the right choice?
- Distributed monolith anti-pattern?

#### Summary

Monolith vs Microservices — Monolith: single deployable unit.

### 1.16 Horizontal Scaling

#### Definition

Adding more machines (scale out).

#### Why it matters

The only way to scale beyond a single machine’s limits.

#### Advantages

- Near-infinite scale
- Fault tolerance via redundancy

#### Disadvantages / Trade-offs

- Requires stateless design or careful data partitioning

#### Real-world example

Web tier of every large service.

#### Interview questions

- Why is horizontal scaling harder for stateful systems?

#### Summary

Horizontal Scaling — Adding more machines (scale out).

### 1.17 Vertical Scaling

#### Definition

Adding more CPU/RAM/disk to a single machine (scale up).

#### Why it matters

Simple, effective — until you hit hardware ceilings.

#### Advantages

- No code changes needed
- No distributed complexity

#### Disadvantages / Trade-offs

- Hardware limits
- Single point of failure

#### Real-world example

Small Postgres deployments scale vertically first.

#### Interview questions

- Trade-offs between vertical and horizontal scaling?

#### Summary

Vertical Scaling — Adding more CPU/RAM/disk to a single machine (scale up).

### 1.18 Replication

#### Definition

Keeping copies of data on multiple nodes.

#### Why it matters

Foundation of availability and read scaling.

#### Advantages

- Fault tolerance
- Read throughput

#### Disadvantages / Trade-offs

- Write amplification
- Consistency challenges

#### Real-world example

Postgres streaming replication; Kafka ISR.

#### Interview questions

- Sync vs async replication?

#### Summary

Replication — Keeping copies of data on multiple nodes.

### 1.19 Partitioning

#### Definition

Splitting data horizontally across nodes so each holds only a subset.

#### Why it matters

Required beyond single-node capacity.

#### Advantages

- Scales writes and storage

#### Disadvantages / Trade-offs

- Cross-partition queries expensive
- Rebalancing hard

#### Real-world example

Cassandra token ranges.

#### Interview questions

- What is a partition key?

#### Summary

Partitioning — Splitting data horizontally across nodes so each holds only a subset.

### 1.20 Sharding

#### Definition

Application-level partitioning of data across multiple databases.

#### Why it matters

Enables scaling beyond a single DB engine’s limits.

#### Advantages

- Massive scale

#### Disadvantages / Trade-offs

- Cross-shard joins complex
- Resharding painful

#### Real-world example

Instagram sharding photos by user ID.

#### Interview questions

- Sharding strategies: range, hash, geo?

#### Summary

Sharding — Application-level partitioning of data across multiple databases.

### 1.21 Caching

#### Definition

Storing hot data in fast memory to reduce load on slower stores.

#### Why it matters

Single biggest lever for latency and cost.

#### Advantages

- Order-of-magnitude latency wins

#### Disadvantages / Trade-offs

- Cache invalidation is hard
- Stale data risk

#### Real-world example

Redis in front of Postgres for session and object caches.

#### Interview questions

- Cache-aside vs write-through vs write-back?

#### Summary

Caching — Storing hot data in fast memory to reduce load on slower stores.

### 1.22 Load Balancer

#### Definition

Distributes incoming traffic across backend servers.

#### Why it matters

Enables horizontal scale and high availability.

#### Advantages

- Removes SPOF
- Enables rolling deploys

#### Disadvantages / Trade-offs

- Itself must be HA
- Adds a hop

#### Real-world example

AWS ALB, NGINX, HAProxy, Envoy.

#### Interview questions

- L4 vs L7 load balancing?
- Sticky sessions — pros and cons?

#### Summary

Load Balancer — Distributes incoming traffic across backend servers.

### 1.23 Database Indexes

#### Definition

Auxiliary data structures (usually B-tree or hash) that speed up lookups by trading write cost and disk.

#### Why it matters

The difference between a 1ms and 10s query.

#### Advantages

- Massive read speedups

#### Disadvantages / Trade-offs

- Slow writes
- Extra storage

#### Real-world example

B-tree indexes on Postgres primary and foreign keys.

#### Interview questions

- When should you NOT add an index?
- Covering index?

#### Summary

Database Indexes — Auxiliary data structures (usually B-tree or hash) that speed up lookups by trading write cost and disk.

### 1.24 Database Normalization

#### Definition

Organizing schema to eliminate redundancy (1NF → 3NF → BCNF).

#### Why it matters

Consistency and storage efficiency in OLTP.

#### Advantages

- No update anomalies
- Smaller storage

#### Disadvantages / Trade-offs

- More joins
- Slower reads

#### Real-world example

Users, orders, order_items separated in an e-commerce schema.

#### Interview questions

- When to denormalize?

#### Summary

Database Normalization — Organizing schema to eliminate redundancy (1NF → 3NF → BCNF).

### 1.25 SQL vs NoSQL

#### Definition

SQL: relational, ACID, strong schema. NoSQL: flexible schema, horizontal scale, various models (KV, document, column, graph).

#### Why it matters

Foundational storage choice.

#### Advantages

- SQL: transactions, joins, mature ecosystem
- NoSQL: scale, flexibility

#### Disadvantages / Trade-offs

- SQL: harder horizontal scale
- NoSQL: weaker consistency, no joins

#### Real-world example

Stripe on Postgres; Netflix on Cassandra.

#### Interview questions

- When would you pick NoSQL over SQL?

#### Summary

SQL vs NoSQL — SQL: relational, ACID, strong schema.

### 1.26 CDN (Content Delivery Network)

#### Definition

Geographically distributed cache of static (and increasingly dynamic) content close to users.

#### Why it matters

Cuts latency and origin load dramatically.

#### Advantages

- Global low latency
- DDoS absorption

#### Disadvantages / Trade-offs

- Cost
- Cache invalidation

#### Real-world example

Cloudflare, Akamai, CloudFront.

#### Interview questions

- Pull vs push CDN?

#### Summary

CDN (Content Delivery Network) — Geographically distributed cache of static (and increasingly dynamic) content close to users.

### 1.27 Message Queue

#### Definition

Asynchronous, decoupled communication channel between services.

#### Why it matters

Enables async workflows, back-pressure, and reliability.

#### Advantages

- Decoupling
- Load smoothing

#### Disadvantages / Trade-offs

- Extra infrastructure
- Ordering / duplication challenges

#### Real-world example

RabbitMQ, SQS, ActiveMQ.

#### Interview questions

- When to use a queue vs a stream?

#### Summary

Message Queue — Asynchronous, decoupled communication channel between services.

### 1.28 EventDriven Architecture

#### Definition

Services communicate via events rather than direct calls.

#### Why it matters

Enables loose coupling and evolvability.

#### Advantages

- Loose coupling
- Auditability

#### Disadvantages / Trade-offs

- Harder to reason about flow
- Eventual consistency

#### Real-world example

Uber trip lifecycle emitted as events consumed by billing, fraud, analytics.

#### Interview questions

- Event notification vs event-carried state transfer?

#### Summary

Event-Driven Architecture — Services communicate via events rather than direct calls.

### 1.29 REST

#### Definition

Architectural style for HTTP APIs based on resources, verbs (GET/POST/ PUT/DELETE), and stateless requests.

#### Why it matters

Ubiquitous default for web APIs.

#### Advantages

- Simple, cacheable, universal tooling

#### Disadvantages / Trade-offs

- Over/under-fetching
- Chatty for graph data

#### Real-world example

Stripe API.

#### Interview questions

- REST maturity levels (Richardson)?

#### Summary

REST — Architectural style for HTTP APIs based on resources, verbs (GET/ POST/PUT/DELETE), and stateless requests.

### 1.30 GraphQL

#### Definition

Query language for APIs; clients ask for exactly the fields they need.

#### Why it matters

Solves over/under-fetching in complex UIs.

#### Advantages

- Client-driven queries
- Single endpoint

#### Disadvantages / Trade-offs

- N+1 problem
- Harder caching

#### Real-world example

GitHub, Shopify.

#### Interview questions

- GraphQL vs REST trade-offs?

#### Summary

GraphQL — Query language for APIs; clients ask for exactly the fields they need.

### 1.31 gRPC

#### Definition

HTTP/2-based RPC framework using Protocol Buffers.

#### Why it matters

Low-latency, strongly-typed inter-service communication.

#### Advantages

- Fast, streaming, typed

#### Disadvantages / Trade-offs

- Browser support limited
- Binary — harder to debug

#### Real-world example

Google, Netflix internal services.

#### Interview questions

- gRPC vs REST for internal APIs?

#### Summary

gRPC — HTTP/2-based RPC framework using Protocol Buffers.

### 1.32 WebSockets

#### Definition

Full-duplex TCP channel over a single HTTP upgrade.

#### Why it matters

Enables real-time bidirectional communication.

#### Advantages

- Real-time push
- Low overhead

#### Disadvantages / Trade-offs

- Stateful — harder to scale
- Firewall issues

#### Real-world example

Slack, chat apps, live trading.

#### Interview questions

- WebSockets vs SSE vs long polling?

#### Summary

WebSockets — Full-duplex TCP channel over a single HTTP upgrade.

### 1.33 API Gateway

#### Definition

Single entry point that routes, authenticates, rate-limits, and observes requests to backend services.

#### Why it matters

Cross-cutting concerns in one place.

#### Advantages

- Centralized auth, rate limiting, logging

#### Disadvantages / Trade-offs

- SPOF if not HA
- Latency hop

#### Real-world example

Kong, AWS API Gateway, Envoy.

#### Interview questions

- Responsibilities of an API gateway?

#### Summary

API Gateway — Single entry point that routes, authenticates, rate-limits, and observes requests to backend services.

### 1.34 Authentication

#### Definition

Verifying who the user is.

#### Why it matters

First line of defense.

#### Advantages

- Prevents impersonation

#### Disadvantages / Trade-offs

- UX friction (2FA)

#### Real-world example

Password + TOTP; passkeys; OAuth login.

#### Interview questions

- Password vs passkey vs OAuth?

#### Summary

Authentication — Verifying who the user is.

### 1.35 Authorization

#### Definition

Verifying what the user is allowed to do.

#### Why it matters

Prevents privilege escalation.

#### Advantages

- Fine-grained control (RBAC/ABAC)

#### Disadvantages / Trade-offs

- Complex policy management

#### Real-world example

AWS IAM.

#### Interview questions

- RBAC vs ABAC vs ReBAC?

#### Summary

Authorization — Verifying what the user is allowed to do.

### 1.36 OAuth 2.0

#### Definition

Delegated authorization protocol allowing third-party apps limited access without sharing credentials.

#### Why it matters

Standard for “Login with Google/Facebook/GitHub”.

#### Advantages

- No password sharing
- Scoped tokens

#### Disadvantages / Trade-offs

- Complex spec, many flows

#### Real-world example

“Sign in with Google”.

#### Interview questions

- Authorization code vs implicit vs PKCE?

#### Summary

OAuth 2.0 — Delegated authorization protocol allowing third-party apps limited access without sharing credentials.

### 1.37 JWT (JSON Web Token)

#### Definition

Signed, self-contained token carrying claims about the user.

#### Why it matters

Enables stateless authentication.

#### Advantages

- Stateless — no session store lookup

#### Disadvantages / Trade-offs

- Revocation hard
- Grows in size

#### Real-world example

Bearer tokens in Authorization: Bearer ....

#### Interview questions

- JWT vs opaque session tokens?

#### Summary

JWT (JSON Web Token) — Signed, self-contained token carrying claims about the user.

### 1.38 Session Management

#### Definition

Server tracks user session state, referenced by session ID cookie.

#### Why it matters

Traditional web auth model.

#### Advantages

- Easy revocation
- Small cookie

#### Disadvantages / Trade-offs

- Server state — requires sticky sessions or shared store

#### Real-world example

Rails, Django default sessions in Redis.

#### Interview questions

- Where to store sessions at scale?

#### Summary

Session Management — Server tracks user session state, referenced by session ID cookie.

### 1.39 Rate Limiting

#### Definition

Restricting number of requests per unit time per client.

#### Why it matters

Protects backends from abuse and overload.

#### Advantages

- Fairness
- Cost control

#### Disadvantages / Trade-offs

- False positives
- State overhead

#### Real-world example

GitHub API: 5000 req/hr authenticated.

#### Interview questions

- Token bucket vs leaky bucket vs sliding window?

#### Summary

Rate Limiting — Restricting number of requests per unit time per client.

### 1.40 Reverse Proxy

#### Definition

Server sitting in front of backends, forwarding client requests.

#### Why it matters

Enables TLS termination, load balancing, caching, and security.

#### Advantages

- Centralized cross-cutting concerns

#### Disadvantages / Trade-offs

- Extra hop

#### Real-world example

NGINX in front of app servers.

#### Interview questions

- Reverse vs forward proxy?

#### Summary

Reverse Proxy — Server sitting in front of backends, forwarding client requests.

### 1.41 Forward Proxy

#### Definition

Server sitting in front of clients, forwarding their requests to the internet.

#### Why it matters

Used for caching, filtering, anonymization.

#### Advantages

- Client-side control

#### Disadvantages / Trade-offs

- Requires client configuration

#### Real-world example

Corporate web proxy; Squid.

#### Interview questions

- Give a use case for a forward proxy.

#### Summary

Forward Proxy — Server sitting in front of clients, forwarding their requests to the internet.

### 1.42 DNS (Domain Name System)

#### Definition

Distributed system that translates human-readable names to IP addresses.

#### Why it matters

First hop in every internet request.

#### Advantages

- Hierarchical, cacheable

#### Disadvantages / Trade-offs

- Propagation delay
- Attack vector (DNS spoofing)

#### Real-world example

Route 53, Cloudflare DNS.

#### Interview questions

- DNS resolution steps?
- TTL trade-offs?

#### Summary

DNS (Domain Name System) — Distributed system that translates humanreadable names to IP addresses.

### 1.43 HTTP

#### Definition

Stateless application-layer protocol for the web.

#### Why it matters

The lingua franca of APIs.

#### Advantages

- Ubiquitous, well-tooled

#### Disadvantages / Trade-offs

- Head-of-line blocking in HTTP/1.1

#### Real-world example

Every web request.

#### Interview questions

- HTTP/1.1 vs HTTP/2 vs HTTP/3?

#### Summary

HTTP — Stateless application-layer protocol for the web.

### 1.44 HTTPS

#### Definition

HTTP over TLS — encrypted and authenticated.

#### Why it matters

Required for security and modern browser features.

#### Advantages

- Confidentiality, integrity, authenticity

#### Disadvantages / Trade-offs

- Handshake latency (mitigated by TLS 1.3, 0-RTT)

#### Real-world example

Every modern site.

#### Interview questions

- What does the TLS handshake do?

#### Summary

HTTPS — HTTP over TLS — encrypted and authenticated.

### 1.45 TCP

#### Definition

Reliable, ordered, connection-oriented transport.

#### Why it matters

Backbone of HTTP, SSH, most application protocols.

#### Advantages

- Reliability built in

#### Disadvantages / Trade-offs

- Head-of-line blocking
- Handshake latency

#### Real-world example

Web, email, SSH.

#### Interview questions

- TCP handshake steps?

#### Summary

TCP — Reliable, ordered, connection-oriented transport.

### 1.46 UDP

#### Definition

Connectionless, unreliable, low-overhead transport.

#### Why it matters

Real-time media, DNS, gaming, QUIC.

#### Advantages

- Low latency
- No head-of-line blocking

#### Disadvantages / Trade-offs

- No reliability — app must handle

#### Real-world example

DNS, video calls, QUIC/HTTP-3.

#### Interview questions

- When to prefer UDP over TCP?

#### Summary

UDP — Connectionless, unreliable, low-overhead transport.

### 1.47 SSL/TLS

#### Definition

Cryptographic protocols providing secure communication.

#### Why it matters

Foundation of HTTPS and secure APIs.

#### Advantages

- Encryption + auth

#### Disadvantages / Trade-offs

- Certificate management overhead

#### Real-world example

Let’s Encrypt for free TLS certs.

#### Interview questions

- Symmetric vs asymmetric crypto in TLS?

#### Summary

SSL/TLS — Cryptographic protocols providing secure communication.

### 1.48 CIDR

#### Definition

Notation for IP address ranges: 10.0.0.0/16 = 65,536 addresses.

#### Why it matters

Fundamental for VPC and subnet design.

#### Advantages

- Compact subnet representation

#### Disadvantages / Trade-offs

- Requires binary intuition

#### Real-world example

AWS VPC subnets.

#### Interview questions

- How many hosts in /24?

#### Summary

CIDR — Notation for IP address ranges: `10.

### 1.49 Network Basics

#### Definition

Understanding of IP, MAC, subnets, routing, NAT, firewalls.

#### Why it matters

Bedrock for any cloud/system design.

#### Advantages

- Enables VPC design and debugging

#### Disadvantages / Trade-offs

- Deep topic

#### Real-world example

AWS VPC, security groups, route tables.

#### Interview questions

- What is NAT and why is it used?

#### Summary

Network Basics — Understanding of IP, MAC, subnets, routing, NAT, firewalls.

### 1.50 OSI Model

#### Definition

Seven-layer conceptual model of networking: Physical, Data Link, Network, Transport, Session, Presentation, Application.

#### Why it matters

Mental map for debugging network issues.

#### Advantages

- Clear separation of concerns

#### Disadvantages / Trade-offs

- TCP/IP doesn’t strictly follow it

#### Real-world example

“Layer 7 load balancer” = application layer.

#### Interview questions

- Which layer is TLS?

#### Summary

OSI Model — Seven-layer conceptual model of networking: Physical, Data Link, Network, Transport, Session, Presentation, Application.

### 1.51 Cloud Basics

#### Definition

Renting compute, storage, network, and managed services from providers like AWS, GCP, Azure.

#### Why it matters

Default deployment target for modern systems.

#### Advantages

- Elastic capacity
- Managed services accelerate delivery

#### Disadvantages / Trade-offs

- Vendor lock-in
- Cost surprises

#### Real-world example

AWS EC2, S3, RDS.

#### Interview questions

- IaaS vs PaaS vs SaaS?

#### Summary

Cloud Basics — Renting compute, storage, network, and managed services from providers like AWS, GCP, Azure.

### 1.52 Containers

#### Definition

Isolated user-space environments sharing the host kernel.

#### Why it matters

Standard packaging for cloud-native apps.

#### Advantages

- Consistent envs
- Fast startup

#### Disadvantages / Trade-offs

- Kernel-level isolation only

#### Real-world example

Docker containers.

#### Interview questions

- Containers vs VMs?

#### Summary

Containers — Isolated user-space environments sharing the host kernel.

### 1.53 Docker

#### Definition

Popular container runtime and image format.

#### Why it matters

De facto standard for packaging services.

#### Advantages

- Reproducible builds
- Ecosystem

#### Disadvantages / Trade-offs

- Image sprawl
- Security surface

#### Real-world example

Dockerfile → image → run anywhere.

#### Interview questions

- What is a multi-stage Dockerfile?

#### Summary

Docker — Popular container runtime and image format.

### 1.54 Kubernetes

#### Definition

Orchestrator for containerized workloads: scheduling, scaling, self-healing, service discovery.

#### Why it matters

Standard for running microservices at scale.

#### Advantages

- Auto-scaling, rolling deploys, self-healing

#### Disadvantages / Trade-offs

- Steep learning curve
- Operational overhead

#### Real-world example

EKS, GKE, AKS.

#### Interview questions

- Pod vs Deployment vs Service?

#### Summary

Kubernetes — Orchestrator for containerized workloads: scheduling, scaling, self-healing, service discovery.

### 1.55 Service Discovery

#### Definition

Mechanism for services to find each other’s network locations dynamically.

#### Why it matters

Required in dynamic environments (auto-scaling, K8s).

#### Advantages

- No hard-coded IPs
- Enables dynamic topology

#### Disadvantages / Trade-offs

- Extra dependency

#### Real-world example

Consul, etcd, K8s DNS.

#### Interview questions

- Client-side vs server-side discovery?

#### Summary

Service Discovery — Mechanism for services to find each other’s network locations dynamically.

### 1.56 Monitoring

#### Definition

Collecting metrics on system health, latency, errors.

#### Why it matters

You cannot fix what you cannot see.

#### Advantages

- Early warning of issues

#### Disadvantages / Trade-offs

- Alert fatigue if noisy

#### Real-world example

Prometheus + Grafana; Datadog.

#### Interview questions

- What is a golden signal?

#### Summary

Monitoring — Collecting metrics on system health, latency, errors.

### 1.57 Logging

#### Definition

Emitting structured records of events for later inspection.

#### Why it matters

Post-mortem and debugging fuel.

#### Advantages

- Debugging power

#### Disadvantages / Trade-offs

- Storage cost
- PII risk

#### Real-world example

ELK stack; CloudWatch Logs.

#### Interview questions

- Structured vs unstructured logs?

#### Summary

Logging — Emitting structured records of events for later inspection.

### 1.58 Observability

#### Definition

Ability to infer internal state from external signals: metrics + logs + traces.

#### Why it matters

Broader than monitoring — enables debugging unknown-unknowns.

#### Advantages

- Debug distributed systems

#### Disadvantages / Trade-offs

- Requires instrumentation everywhere

#### Real-world example

OpenTelemetry; Honeycomb.

#### Interview questions

- Three pillars of observability?

#### Summary

Observability — Ability to infer internal state from external signals: metrics + logs + traces.

### 1.59 Tracing

#### Definition

Following a single request across services with correlated spans.

#### Why it matters

Essential for debugging microservices.

#### Advantages

- End-to-end latency breakdown

#### Disadvantages / Trade-offs

- Sampling trade-offs

#### Real-world example

Jaeger, Zipkin, OpenTelemetry.

#### Interview questions

- Sampling strategies for tracing?

#### Summary

Tracing — Following a single request across services with correlated spans.

### 1.60 Circuit Breaker

#### Definition

Wraps calls to a dependency; opens when failures exceed threshold to prevent cascading failure.

#### Why it matters

Isolates failures in distributed systems.

#### Advantages

- Prevents cascading outages

#### Disadvantages / Trade-offs

- Requires tuning

#### Real-world example

Netflix Hystrix, Resilience4j.

#### Interview questions

- Closed / Open / Half-open states?

#### Summary

Circuit Breaker — Wraps calls to a dependency; opens when failures exceed threshold to prevent cascading failure.

### 1.61 Retry Pattern

#### Definition

Automatically retrying a failed operation, ideally with exponential backoff and jitter.

#### Why it matters

Handles transient failures gracefully.

#### Advantages

- Improves reliability

#### Disadvantages / Trade-offs

- Can amplify load — retry storms

#### Real-world example

AWS SDK default retry policies.

#### Interview questions

- Why jitter?
- When NOT to retry?

#### Summary

Retry Pattern — Automatically retrying a failed operation, ideally with exponential backoff and jitter.

### 1.62 Saga Pattern

#### Definition

Manages distributed transactions as a sequence of local transactions with compensating actions on failure.

#### Why it matters

Replaces 2PC in microservices.

#### Advantages

- No distributed lock
- Scalable

#### Disadvantages / Trade-offs

- Compensating logic can be complex

#### Real-world example

Order → Payment → Inventory → Shipping (with rollbacks).

#### Interview questions

- Choreography vs orchestration saga?

#### Summary

Saga Pattern — Manages distributed transactions as a sequence of local transactions with compensating actions on failure.

### 1.63 CQRS (Command Query Responsibility Segregation)

#### Definition

Separate models for writes (commands) and reads (queries).

#### Why it matters

Enables independent optimization of read and write paths.

#### Advantages

- Independent scaling
- Different data stores

#### Disadvantages / Trade-offs

- More complexity

#### Real-world example

Write to relational DB, read from Elasticsearch.

#### Interview questions

- When to use CQRS?

#### Summary

CQRS (Command Query Responsibility Segregation) — Separate models for writes (commands) and reads (queries).

### 1.64 Event Sourcing

#### Definition

Persist state as an append-only log of events; current state derived by replay.

#### Why it matters

Full audit history, temporal queries.

#### Advantages

- Audit trail
- Rebuild state

#### Disadvantages / Trade-offs

- Schema evolution hard
- Storage growth

#### Real-world example

Banking ledgers.

#### Interview questions

- Event sourcing vs CRUD?

#### Summary

Event Sourcing — Persist state as an append-only log of events; current state derived by replay.

### 1.65 Bloom Filter

#### Definition

Probabilistic data structure telling you if an element is definitely not in a set or possibly in it.

#### Why it matters

Cheap membership check that avoids expensive disk reads.

#### Advantages

- Tiny memory footprint

#### Disadvantages / Trade-offs

- False positives
- No deletions (standard version)

#### Real-world example

Cassandra uses Bloom filters to skip SSTables.

#### Interview questions

- False positive rate vs size?

#### Summary

Bloom Filter — Probabilistic data structure telling you if an element is definitely not in a set or possibly in it.

### 1.66 Consistent Hashing

#### Definition

Hashing scheme where adding/removing nodes only remaps a small fraction of keys.

#### Why it matters

Enables elastic distributed caches and databases.

#### Advantages

- Minimal remap on topology change

#### Disadvantages / Trade-offs

- Uneven load without virtual nodes

#### Real-world example

DynamoDB, Cassandra, Memcached clients.

#### Interview questions

- Why virtual nodes?

#### Summary

Consistent Hashing — Hashing scheme where adding/removing nodes only remaps a small fraction of keys.

### 1.67 Leader Election

#### Definition

Distributed algorithm to select a single coordinator among peers.

#### Why it matters

Required for consistent coordination.

#### Advantages

- Simplifies coordination

#### Disadvantages / Trade-offs

- Leader is bottleneck / SPOF

#### Real-world example

Raft, Paxos, ZooKeeper.

#### Interview questions

- Split-brain — what and how to prevent?

#### Summary

Leader Election — Distributed algorithm to select a single coordinator among peers.

### 1.68 Distributed Lock

#### Definition

Mutual exclusion primitive across processes on different machines.

#### Why it matters

Prevents concurrent modification.

#### Advantages

- Enables cross-node coordination

#### Disadvantages / Trade-offs

- Fencing tokens needed for correctness

#### Real-world example

Redis Redlock, ZooKeeper locks.

#### Interview questions

- Why is Redlock controversial?

#### Summary

Distributed Lock — Mutual exclusion primitive across processes on different machines.

### 1.69 ExactlyOnce Delivery

#### Definition

Each message is processed exactly once — no loss, no duplicates.

#### Why it matters

Impossible in strict sense over unreliable networks; achievable via idempotency + dedup.

#### Advantages

- Correct business semantics

#### Disadvantages / Trade-offs

- Expensive; often not truly needed

#### Real-world example

Kafka exactly-once semantics with transactions.

#### Interview questions

- How to approximate exactly-once?

#### Summary

Exactly-Once Delivery — Each message is processed exactly once — no loss, no duplicates.

### 1.70 AtLeastOnce Delivery

#### Definition

Every message is delivered at least once; duplicates possible.

#### Why it matters

Default reliability level for most queues.

#### Advantages

- Simple
- No loss

#### Disadvantages / Trade-offs

- Consumers must be idempotent

#### Real-world example

SQS standard queues.

#### Interview questions

- Design an idempotent consumer.

#### Summary

At-Least-Once Delivery — Every message is delivered at least once; duplicates possible.

### 1.71 AtMostOnce Delivery

#### Definition

Every message is delivered at most once; loss possible, no duplicates.

#### Why it matters

Best-effort / fire-and-forget.

#### Advantages

- Cheap, simple

#### Disadvantages / Trade-offs

- Data loss

#### Real-world example

UDP-based telemetry.

#### Interview questions

- When is at-most-once acceptable?

#### Summary

At-Most-Once Delivery — Every message is delivered at most once; loss possible, no duplicates.

## Section 2 — System Design Questions

Each question below follows the same complete framework: requirements, scale estimation, API, database, architecture, deep components, data flow, caching, DB choice, message queues, storage, security, scaling, bottlenecks, failure handling, trade-offs, alternatives, optimization, interview tips, common mistakes, ASCII architecture, complexity, and example request/response.

### Section 2 question index

- [Q1. Design a URL Shortener (TinyURL/Bit.ly) — Easy](#q1-design-a-url-shortener-tinyurlbitly-easy)
- [Q2. Design a Rate Limiter — Easy](#q2-design-a-rate-limiter-easy)
- [Q3. Design Pastebin — Easy](#q3-design-pastebin-easy)
- [Q4. Design a Key-Value Store (Redis-like) — Easy](#q4-design-a-key-value-store-redis-like-easy)
- [Q5. Design a Notification Service — Easy](#q5-design-a-notification-service-easy)
- [Q6. Design a Distributed Session Store — Easy](#q6-design-a-distributed-session-store-easy)
- [Q7. Design a Parking Lot System — Easy](#q7-design-a-parking-lot-system-easy)
- [Q8. Design a Distributed Unique ID Generator (Snowflake) — Easy](#q8-design-a-distributed-unique-id-generator-snowflake-easy)
- [Q9. Design Autocomplete / Typeahead — Easy](#q9-design-autocomplete-typeahead-easy)
- [Q10. Design a Web Crawler — Easy](#q10-design-a-web-crawler-easy)
- [Q11. Design WhatsApp — Medium](#q11-design-whatsapp-medium)
- [Q12. Design Instagram — Medium](#q12-design-instagram-medium)
- [Q13. Design Twitter — Medium](#q13-design-twitter-medium)
- [Q14. Design a News Feed System — Medium](#q14-design-a-news-feed-system-medium)
- [Q15. Design Reddit — Medium](#q15-design-reddit-medium)
- [Q16. Design Pinterest — Medium](#q16-design-pinterest-medium)
- [Q17. (Medium) Design YouTube — Medium](#q17-medium-design-youtube-medium)
- [Q18. (Medium) Design Spotify — Medium](#q18-medium-design-spotify-medium)
- [Q19. (Medium) Design Netflix — Medium](#q19-medium-design-netflix-medium)
- [Q20. (Medium) Design Discord — Medium](#q20-medium-design-discord-medium)
- [Q21. Design Slack — Medium](#q21-design-slack-medium)
- [Q22. Design Zoom — Medium](#q22-design-zoom-medium)
- [Q23. Design Gmail — Medium](#q23-design-gmail-medium)
- [Q24. Design Google Drive / Dropbox — Medium](#q24-design-google-drive-dropbox-medium)
- [Q25. Design a Search Engine — Medium](#q25-design-a-search-engine-medium)
- [Q26. Design a Recommendation Engine (YouTube/Netflix style) — Medium](#q26-design-a-recommendation-engine-youtubenetflix-style-medium)
- [Q27. Design a CDN — Medium](#q27-design-a-cdn-medium)
- [Q28. Design DNS — Medium](#q28-design-dns-medium)
- [Q29. Design a Hotel Booking System (Booking.com) — Medium](#q29-design-a-hotel-booking-system-bookingcom-medium)
- [Q30. Design a Flight Booking System — Medium](#q30-design-a-flight-booking-system-medium)
- [Q31. Design Uber / Lyft — Hard](#q31-design-uber-lyft-hard)
- [Q32. Design a Food Delivery System (Swiggy/DoorDash) — Hard](#q32-design-a-food-delivery-system-swiggydoordash-hard)
- [Q33. Design Airbnb — Hard](#q33-design-airbnb-hard)
- [Q34. Design a Payment Gateway (Stripe-like) — Hard](#q34-design-a-payment-gateway-stripe-like-hard)
- [Q35. Design UPI / Realtime Payments — Hard](#q35-design-upi-realtime-payments-hard)
- [Q36. Design a Stock Exchange Matching Engine — Hard](#q36-design-a-stock-exchange-matching-engine-hard)
- [Q37. Design a Cryptocurrency Exchange — Hard](#q37-design-a-cryptocurrency-exchange-hard)
- [Q38. Design Kafka — Hard](#q38-design-kafka-hard)
- [Q39. Design Elasticsearch — Hard](#q39-design-elasticsearch-hard)
- [Q40. Design Collaborative Document Editing — Hard](#q40-design-collaborative-document-editing-hard)
- [Q41. Design a Massively Multiplayer Online Game (MMOG) backend — Hard](#q41-design-a-massively-multiplayer-online-game-mmog-backend-hard)
- [Q42. Design an Online Judge (LeetCode-like) — Hard](#q42-design-an-online-judge-leetcode-like-hard)
- [Q43. Design a Ticket Booking System (BookMyShow) — Hard](#q43-design-a-ticket-booking-system-bookmyshow-hard)
- [Q44. Design an IoT Platform — Hard](#q44-design-an-iot-platform-hard)
- [Q45. Design Google Maps — Expert](#q45-design-google-maps-expert)
- [Q46. Design an LLMserving system (ChatGPT-like) — Expert](#q46-design-an-llmserving-system-chatgpt-like-expert)
- [Q47. Design a Global Distributed SQL Database (Spanner-like) — Expert](#q47-design-a-global-distributed-sql-database-spanner-like-expert)
- [Q48. Design an AdServing / RealTime Bidding System — Expert](#q48-design-an-adserving-realtime-bidding-system-expert)
- [Q49. Design a Realtime Collaborative Whiteboard (Miro-like) — Expert](#q49-design-a-realtime-collaborative-whiteboard-miro-like-expert)
- [Q50. Design a Face Recognition Service at Scale — Expert](#q50-design-a-face-recognition-service-at-scale-expert)

### Q1. Design a URL Shortener (TinyURL/Bit.ly) — Easy

#### Problem Statement

Create a service that takes a long URL and generates a short, unique alias. When a user accesses the short link, they are redirected to the original long URL with minimum latency.

#### Functional Requirements

- Generate a unique short alias for a given long URL.
- Redirect users from the short URL to the original long URL (HTTP 302).
- Allow users to specify a custom alias (optional).
- Support link expiration and deletion.
- Provide basic analytics (click counts).

#### Non-functional Requirements

- Low Latency: Redirection should happen in < 10ms.
- High Availability: The system must be available 99.99% of the time.
- Scalability: Handle billions of URLs and millions of requests per second.
- Read-heavy: The read-to-write ratio is roughly 100:1.

#### Scale Estimation

- DAU: 100M users.
- Write QPS: 100M / 86400 ≈ 1.2k/s.
- Read QPS: 1.2k * 100 = 120k/s.
- Storage: 500 bytes per record. 5 years * 365 days * 100M URLs/day = 182.5B URLs. Total storage: ~91 TB.

#### API Design

- POST /v1/urls: {long_url, custom_alias?, expire_at?} -> {short_url}
- GET / {short_url}: Redirects to long_url (302 Found)
- DELETE /v1/urls/{short_url}: Deletes the mapping.
- GET /v1/analytics/{short_url}: Returns click metrics.

#### Database Design

- URL_Mapping (SQL/NoSQL): short_url_key (PK), original_url, created_at, expires_at, user_id.
- Index on short_url_key.

#### High-Level Architecture

A Load Balancer distributes traffic to a fleet of Stateless Web Servers. The servers interact with a distributed Key-Value store for lookups and a Relational/NoSQL DB for persistence. A specialized Token Service handles unique key generation.

#### Detailed Components

- Key Generation Service (KGS): Pre-generates unique 7-8 character strings (base62) to prevent collisions during high-concurrency writes.
- Redirection Service: A high-throughput service that checks the cache first, then the DB, and returns a 302.
- Analytics Worker: Consumes logs via Kafka to update click counts asynchronously without slowing down the redirect path.
- Cleanup Service: A background job that removes expired URLs to free up storage.

#### Data Flow

**Write**

1. Client sends long URL
2. Service gets unique key from KGS
3. Service saves to DB and Cache
4. Returns short URL

**Read**

1. Client hits short URL
2. Service checks Redis cache
3. (Miss) Check DB
4. Update Cache
5. Emit event to Kafka
6. Respond with 302 status and Location header

#### Caching Strategy

Use Redis with LFU (Least Frequently Used) eviction. Cache the top 20% of daily active URLs (Pareto Principle). TTL: 24 hours or until expiration.

#### Database Choice

Cassandra or MongoDB is preferred due to the massive scale and simple key-value lookups. Cassandra handles writes and high-volume reads across multiple data centers excellently, and the data model is simple enough that ACID is not strictly required for this scale.

#### Message Queue Usage

Kafka/RabbitMQ is used for asynchronous analytics processing. This decoupling ensures that the redirect speed isn’t affected by database writes for metrics.

#### Storage Strategy

Relational metadata stays in the DB. Historical logs are moved to cold storage (S3) for long-term reporting.

#### Security

Rate limiting by IP/API Key to prevent scraping. HTTPS for all traffic. Validation of long URLs to prevent phishing/malware links.

#### Scaling Strategy

Horizontal scaling of web servers. Database sharding based on the Hash(short_url). Regional deployments (Edge Locations) to reduce global latency.

#### Bottlenecks

- KGS Single Point of Failure: Mitigate by keeping two KGS instances with separate key ranges in memory.
- DB Throughput: Use massive read-replicas or DynamoDB to scale linearly.
- Cache Exhaustion: Use a cluster of Redis nodes (Redis Cluster).

#### Failure Handling

- DB Down: Read from Cache; new writes fail or queue.
- Redis Down: Fall back to DB (latency increases).
- KGS Down: Use a local buffer of keys on the web server until KGS recovers.

#### Trade-offs

- 301 vs 302 Redirects: 302 chosen to ensure every click hits our server for analytics, though 301 is better for SEO.
- Custom Aliases: Allows user flexibility but requires a collision check against the existing DB before assignment.
- Strong vs Eventual Consistency: Eventual consistency is accepted for analytics, but URL creation must be immediate (read-after-write).

#### Alternative Approaches

- Hashing long_url: Rejected due to potential collisions (MD5/SHA) and difficulty handling two different users wanting different short links for the same long URL.
- Generating keys on the fly via DB Auto-increment: Rejected as it’s hard to scale across multiple database shards without collisions.

#### Optimization Tips

- Use Bloom Filters to quickly check if a short URL exists before hitting the database.
- Use Base62 (a-z, A-Z, 0-9) to keep URLs short and URL-safe.
- Place the cache at the Edge (CDN) for even lower latency.

#### Interview Tips

- Start with the math (QPS/Storage) to show you understand scale.
- Focus on how to handle the “Unique Key Generation” problem.
- Explain the difference between 301 (Permanent) and 302 (Found) redirects.

#### Final Architecture Diagram

```text
Client -> Load Balancer -> Web/API Cluster -> Redis (Cache)
| |
v v
KGS (Keys) Cassandra (DB) -> Kafka -> Analytics
```

#### Complexity Discussion

- Time: O(1) for lookups from Cache/DB.
- Space: O(N) where N is the number of URLs.
- Network: 1 round trip for cache hit; 2 if DB fetch is needed.

#### Example API Request

```json
{
  "long_url": "https://www.google.com/search?q=system+design+interview",
  "expire_at": "2025-12-31T23:59:59Z"
}
```

#### Example API Response

```json
{
  "short_url": "https://tiny.url/xyz789",
  "created_at": "2023-10-01T12:00:00Z"
}
```

### Q2. Design a Rate Limiter — Easy

#### Problem Statement

Develop a mechanism to control the rate of traffic sent or received by a network interface or service. It prevents resource starvation, protects against DoS attacks, and manages costs for API usage.

#### Functional Requirements

- Limit requests based on a key (IP, UserID, or API Key).
- Support configurable thresholds (e.g., 5 requests per second).
- Provide clear feedback when a limit is exceeded (HTTP 429).
- Support multiple time windows (second, minute, hour).

#### Non-functional Requirements

- Low Latency: Must not add more than 1-2ms to the request.
- Accuracy: Should be as precise as possible across a distributed system.
- High Availability: The rate limiter itself cannot be a single point of failure.
- Scalability: Must handle millions of requests per second.

#### Scale Estimation

- 1M RPS across the fleet.
* 100M Daily Active Users.
- Storage: 20-50 bytes per user in Redis. 100M users = ~5GB RAM, well within a single large Redis node or small cluster.

#### API Design

- Internal Check: isAllowed(key, limit, window) -> {allowed: boolean, remaining: int, resetTime: long}
- HTTP Headers: X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset.

#### Database Design

- No traditional DB. Use Redis for high-speed atomic increments.
- Key: rate_limit:{user_id}:{window_timestamp}
- Value: count (integer)

#### High-Level Architecture

The Rate Limiter acts as a middleware between the Load Balancer and the Service. It queries a distributed cache (Redis) to track request counts against defined policies.

#### Detailed Components

- Middleware/Sidecar: The code module that intercepts the request.
- Configuration Service: Stores rules (e.g., /login is 5/min, /search is 100/min).
- Redis Cluster: Stores the current counters using atomic operations (INCR, EXPIRE).
- Policy Engine: Matches incoming requests to specific rules based on path or headers.

#### Data Flow

1. Request arrives at API Gateway
2. Gateway extracts the key (e.g., UserID)
3. Gateway calls Redis INCR(key)
4. If value > limit, return
5. 429
6. If value == 1, set EXPIRE(key, window)
7. Else, allow request to proceed to the backend server

#### Caching Strategy

Redis is the primary store. Use Sliding Window Log or Sliding Window Counter to prevent “bursting” at the edge of fixed windows.

#### Database Choice

Redis is chosen for its in-memory speed and support for atomic operations. Consistency is favored over absolute persistence, though Redis AOF can be used.

#### Message Queue Usage

Log events of “Rate Limit Exceeded” to Kafka for asynchronous security analysis and to detect potential bot attacks.

#### Storage Strategy

In-memory (Redis). Policies can be stored in a relational DB (Postgres) and cached locally by the middleware.

#### Security

Prevent “Rate Limiter Bypass” by placing it behind a secure Firewall. Use cryptographic hashes for keys if UserIDs are sensitive.

#### Scaling Strategy

Redis Cluster for horizontal scaling of the counter store. Locally cache policies on the API Gateway to avoid DB lookups. Cross-region replication if the service is global.

#### Bottlenecks

- Redis Latency: Mitigate by using a local in-memory cache for very high-frequency keys (though this sacrifices some accuracy).
- Race Conditions: Use Lua scripts in Redis to ensure INCR and EXPIRE are atomic.
- Global Lock Contention: Shard Redis by UserID.

#### Failure Handling

- Fail-open vs Fail-closed: Usually, rate limiters “Fail-open” (allow traffic) to ensure availability if Redis dies, unless security is the primary concern.

#### Trade-offs

- Accuracy vs Latency: Fixed window is faster/simpler but less accurate than Sliding Window.
- Centralized vs Distributed: Centralized (Redis) is accurate but adds network hop; Local is faster but doesn’t track across servers.

#### Alternative Approaches

- Token Bucket: Great for bursts but slightly more complex to implement in Redis.
- Leaky Bucket: Smooths traffic out but can delay requests.

#### Optimization Tips

- Use Lua scripts to reduce network round trips to Redis.
- Batch write metrics to the analytics engine.
- Use a multi-tier approach: L1 (local cache) for obvious floods, L2 (Redis) for global limits.

#### Interview Tips

- Discuss the different algorithms (Token Bucket, Leaking Bucket, Fixed Window, Sliding Window).
- Mention how to handle distributed environments (race conditions).
- Explain 429 status codes and headers.

#### Final Architecture Diagram

```text
Client -> Load Balancer -> API Gateway [Rate Limit Middleware] -> Backend
|
Redis Cluster (Counters)
```

#### Complexity Discussion

- Time: O(1) for Redis atomic ops.
- Space: O(U) where U is the number of active users in the current window.
- Network: 1-2 ms overhead per request.

#### Example API Request

```http
GET /api/v1/resource

HTTP/1.1
Host: api.example.com
Authorization: Bearer my-token
```

#### Example API Response

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1672531200
Content-Type: application/json

{
  "error": "Rate limit exceeded. Try again in 30 seconds."
}
```

### Q3. Design Pastebin — Easy

#### Problem Statement

Create a web service where users can store and share plain text (snippets) online. Each “paste” is identified by a unique URL and can have an expiration date.

#### Functional Requirements

- Users can upload a “paste” of text.
- Users receive a unique URL for the paste.
- Pastes can have an expiration (min, hour, day, never).
- Users can optionally register to manage their pastes.
- Support syntax highlighting/ language detection (metadata).

#### Non-functional Requirements

- Availability: High availability for reading pastes.
- Durability: Pastes must not be lost once “saved” (until expiry).
- Scalability: Handle large volumes of text data.
- Low Latency: Quick retrieval of text.

#### Scale Estimation

- 1M new pastes/day.
- Avg paste size: 10KB.
* 1M * 10KB = 10GB/day. 3.6TB/year.
- Read/Write ratio: 10:1. 10M reads/day ≈ 115 QPS.

#### API Design

- POST /v1/pastes: {content, expiration, title, folder_id?} -> {short_url}
- GET /v1/pastes/{paste_id}: Returns paste content and metadata.
- DELETE /v1/pastes/ {paste_id}: Removes the paste.

#### Database Design

- Metadata (SQL): Paste (paste_id, user_id, title, expiration_date, created_at, storage_path).
- Content (Object Store): S3 or a NoSQL Blob store.

#### High-Level Architecture

Users upload text via a Web Server. Metadata is stored in a Relational DB for easy querying/indexing, while the actual text content is stored in a cost-effective Object Store (like S3).

#### Detailed Components

- Object Storage (S3): Stores the raw text files to keep the database slim.
- Metadata DB (Postgres): Handles user info and paste locations.
- Cache (Redis): Stores frequently accessed pastes in memory.
- Cleaning Service: Background worker that deletes expired objects from S3 and rows from DB.

#### Data Flow

**Write**

1. User submits text
2. Generate unique ID
3. Write text to S3
4. Write metadata + S3 path to Postgres
5. Return URL

**Read**

1. Hit URL
2. Check Redis for content
3. (Miss) Query DB for S3 path
4. Fetch content from S3
5. Cache in Redis
6. Respond

#### Caching Strategy

Keep the top 5% of most popular pastes in Redis. Since pastes are immutable (usually), cache invalidation is only needed on deletion.

#### Database Choice

Postgres for metadata (ACID, relational features for users/ folders). S3 for content because it is cheaper for large blobs than storing text in a DB or NoSQL column.

#### Message Queue Usage

Use a queue for “Cleanup” tasks. When a paste expires, add it to a queue to ensure both DB and S3 entries are deleted eventually.

#### Storage Strategy

S3 with Lifecycle Policies. If a paste hasn’t been accessed in 30 days and has no “never” expiry, move to Infrequent Access or Glacier (or just delete).

#### Security

Content scrubbing for malicious scripts (XSS). Private vs Public pastes (simple AuthZ check). Rate limiting on creation.

#### Scaling Strategy

S3 scales automatically. Shard Postgres by user_id or paste_id. Use a CDN (CloudFront) for serving popular pastes globally.

#### Bottlenecks

- DB Metadata growing too large: Shard the DB.
- High Read Latency for S3: Mitigate with Aggressive Caching and CDN.
- Duplicate short IDs: Use a Snowflake ID generator or a KGS (similar to TinyURL).

#### Failure Handling

- S3 Unavailable: Pastes cannot be read. Use Cross-Region Replication (CRR) for high durability.
- DB Down: Read-only mode possible if using a cache with long TTL.

#### Trade-offs

- SQL vs NoSQL: SQL used for metadata for better relational queries, even though NoSQL is easier to scale.
- Storage: Blobs in DB vs S3. S3 is cheaper and keeps the DB small/fast.

#### Alternative Approaches

- Storing text in Cassandra: Good for high writes, but S3 is easier to manage for larger blobs.
- Using a File System: Hard to scale horizontally compared to S3.

#### Optimization Tips

- Gzip/Compress text before storing to S3 to save space.
- Use a Pre-signed URL for direct S3 downloads for very large pastes.
- Unique IDs should be URL-safe (Base62).

#### Interview Tips

- Clear separation between metadata and content is the key to this design.
- Discuss storage cost optimizations.
- Mention handling of “Never Expire” pastes.

#### Final Architecture Diagram

```text
Client -> LB -> Web Service -> Redis
| |
v v
Postgres (Meta) Amazon S3 (Text Blobs)
```

#### Complexity Discussion

- Time: O(1) for retrieval.
- Space: O(N) where N is total paste volume.
- Network: Two hops (DB + S3) for a cache miss.

#### Example API Request

```json
{
  "content": "SELECT * FROM users WHERE id = 1;",
  "title": "SQL Query",
  "expiry": "1h"
}
```

#### Example API Response

```json
{
  "url": "https://pastebin.com/aB3c4D",
  "paste_id": "aB3c4D",
  "expires_at": "2023-11-01T10:00:00Z"
}
```

### Q4. Design a Key-Value Store (Redis-like) — Easy

#### Problem Statement

Design a distributed in-memory key-value store. It must provide fast access to data, support simple data types (strings, lists), and offer durability options.

#### Functional Requirements

- put(key, value) and get(key).
- Support for Time-toLive (TTL).
- Support for basic data structures (Strings, Sets).
- Atomic operations on keys.

#### Non-functional Requirements

- Extremely Low Latency: Sub-millisecond responses.
- High Availability: Replicated data across nodes.
- Scalability: Handle TBs of data across a cluster.
- Tunable Consistency (Eventual vs Strong).

#### Scale Estimation

- 1M+ QPS per cluster.
* 100GB to many TBs of data.
- Read/ Write varies (often 50/50 for session stores, 90/10 for caches).

#### API Design

- SET key value [EX seconds]
- GET key
- DEL key
- EXPIRE key seconds

#### Database Design

- Data is stored in an InMemory Hash Map. For durability, use:
- WAL (Write Ahead Log): Append-only file of every mutation.
- Snapshotting: Periodic dump of memory to disk (RDB).

#### High-Level Architecture

A distributed cluster where keys are partitioned across nodes using Consistent Hashing. Each partition has a Primary node for writes and Replicas for high availability/reads.

#### Detailed Components

- Storage Engine: In-memory Hash Table with an LRU eviction policy.
- Replication Manager: Handles master-slave synchronization.
- Coordinator Node/Gossip Protocol: Maintains cluster state and membership.
- Persistence Engine: Manages WAL and Snapshotting to disk.

#### Data Flow

1. Client sends SET(K, V)
2. Hashing determines the target Node
3. Node writes to WAL (disk)
4. Node updates In-Memory Hash Map
5. Asynchronously replicates to Slaves
6. Acknowledges to Client

#### Caching Strategy

The entire system is a cache. Use LRU (Least Recently Used) or LFU for eviction when memory limit is reached.

#### Database Choice

N/A (We are building the DB). Memory is the primary store (RAM). Disk (SSD) is used for persistence.

#### Message Queue Usage

Not used in the core path, but replication can be seen as a log-based stream (similar to Kafka’s internals).

#### Storage Strategy

Data resides in RAM. Persist to local SSD via AOF (Append-only file) for durability. Periodically compact AOF to keep it small.

#### Security

Mutual TLS between nodes. Password/Token-based client auth. VPC isolation.

#### Scaling Strategy

Consistent Hashing to add nodes without massive data reshuffling. Virtual nodes to prevent “hotspots.”

#### Bottlenecks

- Network Bandwidth: For very large values, the NIC becomes the bottleneck.
- RAM Limit: Vertical scaling (bigger RAM) or Horizontal (more shards).
- Garbage Collection (if in Java/Go): Causes pauses; C++ or Rust is preferred for subms latency.

#### Failure Handling

- Master Failure: Sentinel or Orchestrator promotes a Slave.
- Network Partition: Quorum-based writes (Paxos/Raft) to prevent split-brain.
- Node Reboot: Reconstruct state from WAL and Last Snapshot.

#### Trade-offs

- Consistency vs Availability: Choose Eventual Consistency for speed or Strong for correctness.
- Latency vs Durability: Syncing to disk on every write (fsync) is slow; async is fast but risks losing data.

#### Alternative Approaches

- Single-threaded (Redis): Simple, no locks, but can’t use all CPU cores for one process.
- Multi-threaded (Memcached): Better CPU utilization, but requires complex locking/concurrency control.

#### Optimization Tips

- Use Zerocopy for network transfers.
- Implement Pipeline API to batch commands and reduce round trips.
- Use Compact Data Structures (Ziplist) for small collections to save RAM.

#### Interview Tips

- Focus on how Consistent Hashing works.
- Discuss Replication (Async vs Sync).
- Explain Persistence trade-offs (AOF vs RDB).

#### Final Architecture Diagram

```text
Client -> [Consistent Hashing] -> Node 1 (Master) -> [Sync] -> Node 1 (Slave)
-> Node 2 (Master) -> [Sync] -> Node 2 (Slave)
...
```

#### Complexity Discussion

- Time: O(1) for GET/SET.
- Space: O(N) where N is the number of keys.
- Network: Minimal (usually TCP/RESP protocol).

#### Example API Request

```json
SET user:101 "{\"name\":\"John\"}" EX 3600
```

#### Example API Response

```json
OK
```

### Q5. Design a Notification Service — Easy

#### Problem Statement

System to send notifications to users via different channels: Email, SMS, and Push (iOS/Android). It must be reliable, scalable, and handle millions of alerts daily.

#### Functional Requirements

- Support multiple platforms (Push, SMS, Email).
- Send real-time and scheduled notifications.
- Allow users to opt-in/out of specific categories.
- Track delivery status (Sent, Delivered, Failed).

#### Non-functional Requirements

- High Reliability: Messages should not be lost.
- Low Latency: Deliver within seconds.
- Scalability: Handle sudden bursts (e.g., breaking news).
- Extensibility: Easy to add new providers (Twilio, SendGrid, FCM).

#### Scale Estimation

- 10M users.
* 100M notifications/day.
- Peaks of 50k notifications/sec.
- Retention: Notification logs for 30 days.

#### API Design

- POST /v1/notifications: {user_id, message, type, priority} -> {request_id}
- GET /v1/notifications/status/{request_id}: Returns delivery status.
- PUT /v1/users/ settings: Updates user preferences.

#### Database Design

- User Preferences (Postgres): user_id, channel_type, enabled.
- Notification Logs (Cassandra/NoSQL): notification_id, user_id, status, provider_response, timestamp.

#### High-Level Architecture

An API Gateway receives requests, validates them, and pushes them into a Message Queue. Different Workers consume from the queue to call third-party providers (APNS, Twilio, etc.).

#### Detailed Components

- Notification Servers: Validate inputs, perform Auth, and enqueue tasks.
- Message Queues (Kafka): Buffer to handle spikes and decouple components.
- Workers: Specialized code for each channel (EmailWorker, SMSWorker).
- Delivery Tracker: Updates the database with status codes from providers.
- User Setting Cache: Redis store for quick lookup of user preferences.

#### Data Flow

1. Service A calls Notification API
2. API validates data and checks Redis for User Preferences
3. API writes to DB (Status: Pending) and enqueues to Kafka
4. Workers pick up from Kafka
5. Workers call Provider (e.g., SendGrid)
6. Workers update DB status (Status: Sent/Failed)

#### Caching Strategy

Cache UserPreferences and DeviceTokens in Redis with a long TTL (invalidated on update). This prevents hitting the main DB for every single notification.

#### Database Choice

Postgres for relational user data (settings). Cassandra for logs because it handles high write-throughput and time-series data (notification history) very effectively across clusters.

#### Message Queue Usage

Kafka is critical here. It allows retries on failure (exponential backoff) and acts as a buffer so that a slow third-party provider doesn’t block the entire system.

#### Storage Strategy

Recent logs in Cassandra. Older logs (90 days+) archived in S3/ BigQuery for analytics and compliance.

#### Security

API Key Auth for internal services. Rate limiting to prevent accidental spamming. Privacy: No PII in logs, only IDs.

#### Scaling Strategy

Scale workers horizontally based on Queue length. Use Kafka partitions to parallelize processing. Use multiple providers (Primary/Secondary) for each channel to ensure delivery.

#### Bottlenecks

- Third-party Throttling: Mitigated by worker-level rate limiting.
- Queue Backlog: Scale workers or increase Kafka partitions.
- Database Write Saturation: Shard Cassandra by user_id.

#### Failure Handling

- Provider Down: Retry with exponential backoff; if persistent, switch to a fallback provider.
- Worker Death: Kafka will reassign partitions to another worker.
- User Opt-out: Ensure the worker checks the setting again before calling the provider to avoid complaints.

#### Trade-offs

- At-least-once vs Exactly-once: We aim for at-least-once; duplicate notifications are annoying but better than missing a critical alert.
- Latency vs cost: Batching emails saves money but increases latency.

#### Alternative Approaches

- Synchronous calls to providers: Rejected because it blocks the caller and won’t handle spikes.
- Single Queue for all types: Rejected; one slow provider (e.g., Email) shouldn’t delay critical Push alerts.

#### Optimization Tips

- Use Templating Service to generate HTML/Text messages outside the main path.
- Deduplication Logic: Use a hash of the content and user_id in Redis for a 5-minute window to avoid sending the same alert twice.
- Batch status updates to the database.

#### Interview Tips

- Show the “Fan-out” pattern from the Queue to different workers.
- Emphasize the importance of Retries and Idempotency.
- Discuss the “Priority” queue concept (Emergency alerts vs Marketing).

#### Final Architecture Diagram

```text
Service -> API LB -> Notification Engine -> Redis (Prefs)
|
v
Kafka (Topic per Type)
|
----------------------------------------------------
| | |
SMS Worker Email Worker Push Worker
| | |
(Twilio) (SendGrid) (FCM/APNS)
```

#### Complexity Discussion

- Time: O(1) for enqueueing.
- Space: O(N) for logs.
- Network: Multiple hops (Service -> API -> Queue -> Worker -> Provider).

#### Example API Request

```json
{
  "user_id": "u123",
  "type": "push",
  "content": "Your package has arrived!",
  "priority": "high"
}
```

#### Example API Response

```json
{
  "status": "enqueued",
  "request_id": "req-9999"
}
```

### Q6. Design a Distributed Session Store — Easy

#### Problem Statement

As a system scales horizontally, a single server can no longer store user session data locally. The goal is to design a high-performance, globally accessible store that maintains user state (login status, shopping carts, preferences) across multiple web servers.

#### Functional Requirements

- Create a session upon successful authentication.
- Retrieve session data by session ID for every incoming request.
- Update session data (e.g., last accessed time, cart items).
- Remove/expire sessions after inactivity or explicit logout.
- Support multiple device logins for the same user.

#### Non-functional Requirements

- Latency: Ultra-low latency (<5ms) for retrievals as it is on the critical path of every request.
- Availability: Extremely high availability; if the session store is down, users cannot log in.
- Consistency: Eventual consistency is usually acceptable, but session creation requires strong consistency.
- Scale: Must handle millions of concurrent active sessions.
- Durability: Medium; session loss is annoying (logs user out) but not catastrophic like financial data loss.

#### Scale Estimation

- DAU: 100 Million.
- Write QPS: 10k/s (logins/updates).
- Read QPS: 500k/s (every API request reads session).
- Storage: 100M sessions * 2KB/ session = 200 GB.
- Bandwidth: 500k * 2KB = 1 GB/s.

#### API Design

- POST /v1/sessions - Returns session_id.
- GET /v1/sessions/{id} - Returns Session Object.
- PUT /v1/sessions/{id} - Updates session data.
- DELETE /v1/sessions/{id} - Invalidates session.

#### Database Design

- Redis (Key-Value): Key: sess:{session_id}, Value: Hash (user_id, creation_time, last_access, data_blob).
- TTL: Native Redis TTL set to 30 minutes, sliding window update on every read.

#### High-Level Architecture

The client sends a request with a cookie/token. The Load Balancer forwards to a Web Server, which queries the Distributed Cache (Redis Cluster) to validate the session before hitting downstream microservices.

#### Detailed Components

- Web Tier: Stateless servers that extract Session ID from headers.
- Session Service: An internal abstraction layer to handle session logic (validation, extensions).
- Redis Cluster: Sharded key-value store using consistent hashing.
- Config Service: Manages cluster topology and node discovery.

#### Data Flow

1. Write: User logs in → Auth Service validates → Session Service generates UUID → Writes to Redis with TTL → Returns cookie to client
2. Read: Client sends request → API Gateway/Service calls Session Service → Service queries Redis → If hit, update TTL and return data; if miss, redirect to login

#### Caching Strategy

Redis is the primary store, not just a cache. A local L1 cache (In-memory on web server) can be used for very hot sessions but risks inconsistency. Use “Lazy Expiration” (Redis handles this).

#### Database Choice

Redis is the industry standard here due to its in-memory speed, native data structures (Hashes), and built-in TTL mechanism. It outperforms RDBMS for high-throughput, low-latency key-value lookups.

#### Message Queue Usage

Optional: Use Kafka to stream “Session Activity” events for analytics/fraud detection without blocking the main request path.

#### Storage Strategy

Primarily RAM-based. Use AOF (Append Only File) with everysec sync to disk for a balance of performance and durability in case of cluster restart.

#### Security

Session IDs should be cryptographically secure random strings (UUID v4). Set HttpOnly and Secure flags on cookies. Implement rate limiting on session creation to prevent brute-force attacks.

#### Scaling Strategy

Scale Redis via Sharding (Cluster mode). Distribute keys by session_id % number_of_shards. Use a Global Load Balancer to route users to the nearest regional session cluster.

#### Bottlenecks

- Redis Memory Limit: Mitigated by aggressive TTLs and LRU eviction.
- Network Bandwidth: Mitigated by compressing large session blobs (e.g., Protobuf/Gzip).
- Hot Keys: Rarely an issue for sessions since they are user-specific, but high-traffic bot accounts might need rate limiting.

#### Failure Handling

- Redis Node Down: Redis Cluster promotes a slave to master automatically.
- Complete Cache Wipe: Failover to a persistent DB (Postgres) or force all users to re-login (standard industry compromise).

#### Trade-offs

- Latency vs. Durability: Choosing Redis AOF over RDB for better durability at a slight IO cost.
- Centralized vs. Sticky Sessions: Centralized store is chosen over Sticky Sessions to allow seamless server scaling and deployments.
- Clientside (JWT) vs. Serverside (Redis): Server-side allows instant revocation, whereas JWTs require a blacklist.

#### Alternative Approaches

- JWT (JSON Web Tokens): Stateless, no DB lookup needed. Rejected because sessions cannot be easily invalidated before expiry.
- Rela tional DB (Postgres): High durability. Rejected because latencies are too high for per-request lookups at scale.

#### Optimization Tips

- Use MGET to fetch multiple sessions if a dashboard aggregates data.
- Use Protobuf instead of JSON to reduce payload size by 40-60%.
- Implement “Sliding Window” updates asynchronously to avoid write-latency on every read.

#### Interview Tips

- Mention “Session Hijacking” and how to prevent it.
- Discuss the difference between a “Stateful” and “Stateless” architecture.
- Address how to handle cross-region session replication if requested.

#### Final Architecture Diagram

```text
Client -> Load Balancer -> [Web Server/API Gateway]
|
[Session Service]
|
[Redis Cluster Shards]
(Node 1) (Node 2) (Node 3)
```

#### Complexity Discussion

- Time: O(1) for session lookup/insert via Hash Map.
- Space: O(N) where N is the number of active sessions.
- Network: 1 RTT between Web Server and Redis.

#### Example API Request

```http
GET /v1/sessions/abc-123-xyz
Host: session-service.internal
Authorization: Bearer <internal_token>
```

#### Example API Response

```json
{
  "session_id": "abc-123-xyz",
  "user_id": "user_88",
  "created_at": 1715432000,
  "data": {
    "role": "admin",
    "cart_count": 5
  }
}
```

### Q7. Design a Parking Lot System — Easy

#### Problem Statement

Design a system to manage a parking lot with multiple levels, varying spot sizes (small, medium, large), and automated payment processing.

#### Functional Requirements

- Support multiple entrances and exits.
- Real-time tracking of available spots by type.
- Ability to assign a spot to a vehicle based on its size.
- Calculate fees based on duration and vehicle type at exit.
- Generate a ticket upon entry.

#### Non-functional Requirements

- Availability: High availability for entry/exit gates to prevent traffic jams.
- Consistency: Strong consistency for spot allocation (avoid double-booking the same spot).
- Latency: Fast processing (<2s) for gate opening.
- Scalability: Capable of handling hundreds of parking lots for a corporation.

#### Scale Estimation

- Lot Capacity: 5,000 spots.
- Daily Transactions: 10,000 entries/exits.
- Writes: Very low (0.1 QPS).
- Reads: Moderate (for status displays).
- Storage: Negligible (MBs/year).

#### API Design

- POST /v1/tickets/entry - Params: vehicle_type, gate_id. Returns ticket_id.
- POST /v1/tickets/exit - Params: ticket_id, payment_info. Returns receipt.
- GET /v1/spots/ availability - Returns count of free spots per type.

#### Database Design

- SQL (Postgres): Necessary for ACID compliance during spot allocation.
- ParkingSpots: id, level, type, is_occupied, current_ticket_id.
- Tickets: id, spot_id, entry_time, exit_time, status (active/paid).

#### High-Level Architecture

Entry Gate hardware calls the Parking Service, which queries the DB for an empty spot, marks it occupied, and generates a ticket. Exit Gate terminal calculates the fee and updates the DB upon payment.

#### Detailed Components

- Gate Controller: IoT interface for physical barriers.
- Parking Service: Core logic for spot selection and fee calculation.
- Payment Service: Integration with Stripe/Square.
- Display Service: Updates electronic signs showing “Full” or “X spots left.”

#### Data Flow

1. Entry: Driver presses button → Gate Controller calls Service → Service finds spot SELECT ... FOR UPDATE → Create Ticket → Gate opens
2. Exit: Driver scans ticket → Service calculates (now - entry_time)

1. rate → Payment processed → Spot marked free → Gate opens.

#### Caching Strategy

Cache availability counts in Redis (e.g., lot_123_small_count) for the display signs. Use periodic sync from DB to keep counters accurate.

#### Database Choice

Relational Database (Postgres) is ideal. Transactional integrity is critical to ensure two cars entering at different gates don’t get assigned the same spot simultaneously.

#### Message Queue Usage

Use a queue (RabbitMQ) to send “Ticket Created” or “Payment Succeeded” events to a secondary Analytics/Reporting service.

#### Storage Strategy

Relational tables for operational data. S3 or cold storage for logs and security camera snapshots linked to tickets.

#### Security

Encrypt payment data (PCI-DSS). Use HMAC for ticket IDs to prevent users from guessing IDs to lower their parking fees.

#### Scaling Strategy

Shard the database by parking_lot_id. A single Postgres instance can handle thousands of small lots; sharding allows global scale.

#### Bottlenecks

- Race Conditions at Entry: Solved by DB transactions.
- Payment Gateway Downtime: Implement “Offline Mode” where gates open and charge later if possible.
- Single Point of Failure: API Gateway redundancy across availability zones.

#### Failure Handling

- DB Down: Entry gates switch to “Manual Mode” (paper tickets).
- Payment Service Down: Store payment intent locally and retry later (grace period for drivers).

#### Trade-offs

- Optimistic vs Pessimistic Locking: Using Pessimistic locking (SELECT FOR UPDATE) to guarantee no double-booking, despite slightly lower throughput.
- Centralized vs Local Processing: Centralized for easier management, but increases dependency on internet connectivity.

#### Alternative Approaches

- NoSQL (Cassandra): Rejected because atomic “find and update” operations are harder to implement than in SQL.
- Clientside Logic: Rejected because hardware controllers should remain thin/dumb for easy replacement.

#### Optimization Tips

- Use a bitmask or specialized index for finding available spots quickly.
- Pre-calculate fees at the exit gate UI to reduce “wait time” for users.
- Implement ANPR (Automatic Number Plate Recognition) to remove the need for physical tickets.

#### Interview Tips

- Start with the Object-Oriented Design (Classes: Vehicle, Spot, Level).
- Transition quickly to the System Design (Concurrency, DB choice).
- Clarify how you handle different vehicle sizes fitting into larger spots (Small car in Large spot).

#### Final Architecture Diagram

```text
[Entry Gate] <--> [LB] <--> [Parking Service] <--> [Postgres DB]
[Exit Gate] <--> [LB] |
[Signage] <--- [Redis Cache] <--- (Polls DB)
```

#### Complexity Discussion

- Time: O(1) for ticket generation (index-backed lookup).
- Space: O(Spots) in DB.
- Network: Minimal.

#### Example API Request

```http
POST /v1/tickets/entry

{
  "vehicle_type": "LARGE",
  "gate_id": "GATE_A"
}
```

#### Example API Response

```json
{
  "ticket_id": "TKT-99821",
  "spot_id": "SPOT-L-402",
  "entry_time": "2024-05-12T10:00:00Z"
}
```

### Q8. Design a Distributed Unique ID Generator (Snowflake) — Easy

#### Problem Statement

Create a system that generates 64-bit unique, roughly timeordered IDs at high scale across multiple data centers without a single point of failure or central coordination.

#### Functional Requirements

- IDs must be unique across the global system.
- IDs must be 64-bit integers (for database efficiency).
- IDs should be roughly sorted by time.
- System must handle 10,000+ requests per second per node.

#### Non-functional Requirements

- Availability: 99.999% availability (ID generation is a core dependency).
- Latency: Ultra-low latency (<1ms).
- Scalability: Must work across thousands of worker nodes.
- No Coordination: Nodes should not talk to each other for every ID.

#### Scale Estimation

- QPS: 100k+ per second.
- ID Length: 64-bit.
- Lifespan: Support for 50+ years before overflow.

#### API Design

- GET /v1/next-id - Returns a 64-bit long.
* (Usually implemented as a Library, not an API, but can be a gRPC service).

#### Database Design

No DB is needed for the hot path. A configuration provider like Zookeeper or Etcd is used to manage and assign Worker IDs to nodes.

#### High-Level Architecture

Each generator node (or application instance) constructs an ID by concatenating a timestamp, a worker ID, and a local sequence number.

#### Detailed Components

- Timestamp (41 bits): Milliseconds since a custom epoch (e.g., Jan 1, 2024).
- Worker ID (10 bits): Assigned by Zookeeper, allows for 1024 unique nodes.
- Sequence (12 bits): Incremented for every ID generated in the same millisecond.
- Sign Bit (1 bit): Always 0 to keep the number positive.

#### Data Flow

1. Startup: Worker node contacts Zookeeper → Obtains unique Worker ID
2. Runtime: Node receives request → Gets current timestamp → If diff from last ID, reset sequence to 0 → Else increment sequence → Shift bits and OR them together → Return ID

#### Caching Strategy

Not applicable for the ID generation itself, but Zookeeper info is cached locally on the worker node.

#### Database Choice

N/A for generation. Zookeeper is used for distributed coordination (Worker ID assignment) because it handles consensus and node heartbeats reliably.

#### Message Queue Usage

None. MQ would add too much latency to a primitive operation like ID generation.

#### Storage Strategy

No persistent storage for IDs; they are generated on the fly. Metrics and logs should be exported to Prometheus/ELK.

#### Security

IDs are predictable (chronological). If security/obfuscation is needed, this design is inappropriate (use UUIDs or encrypted IDs).

#### Scaling Strategy

Scale by adding more worker nodes. With 10 bits for Worker ID, you can have 1,024 nodes. Each node can generate 4,096 IDs per millisecond (2^12), totaling 4.1 million IDs/sec.

#### Bottlenecks

- Clock Drift: If a node’s clock moves backward, it might generate duplicate IDs. Mitigation: Refuse to generate IDs until the clock catches up.
- Sequence Exhaustion: If >4096 IDs requested in 1ms. Mitigation: The thread waits for the next millisecond.

#### Failure Handling

- Zookeeper Down: Node uses its cached Worker ID; if it restarts while Zookeeper is down, it fails to start (safe failure).
- Clock Rewind: System enters a “Wait Mode” or throws an error to maintain uniqueness.

#### Trade-offs

- 64bit vs 128bit: 64-bit fits in standard BigInt, whereas UUIDs (128bit) take more index space.
- Centralized vs Decentralized: Decentralized reduces latency but introduces complexity (Worker ID management).

#### Alternative Approaches

- UUID v4: 128-bit, random. Rejected because it’s not time-ordered and large for DB indexes.
- DB Autoincrement with Step: Multiple DBs incrementing by n. Rejected because it’s hard to scale horizontally and maintain.

#### Optimization Tips

- Use bitwise operations for maximum speed.
- Set a custom epoch to extend the 41-bit timestamp life (up to 69 years).
- For very high throughput, pre-generate batches of IDs in a buffer.

#### Interview Tips

- Write the bit breakdown on the board clearly (41 + 10 + 12 = 63 + 1 sign).
- Explain why 41 bits for time (2^41 ms ≈ 69 years).
- Address the “Clock Drift” scenario—it’s the most common follow-up.

#### Final Architecture Diagram

```text
[Client] -> [Worker Node A] -> (Uses its assigned WorkerID 1)
[Client] -> [Worker Node B] -> (Uses its assigned WorkerID 2)
^ ^ |_________|
|
[Zookeeper] (Maintains WorkerID assignments)
```

#### Complexity Discussion

- Time: O(1) - Constant time bitwise ops.
- Space: O(1) - No storage needed on node.

#### Example API Request

```http
GET /v1/next-id
```

#### Example API Response

```json
181512345678901234
```

### Q9. Design Autocomplete / Typeahead — Easy

#### Problem Statement

Build a search suggestion service that provides the top 5-10 most relevant completions as a user types into a search bar, with extremely low latency.

#### Functional Requirements

- Provide suggestions based on a prefix.
- Suggestions should be ranked by popularity (frequency).
- Update suggestions as new popular queries emerge.
- Support for multi-word phrases.

#### Non-functional Requirements

- Latency: Real-time feel (<100ms response).
- Availability: High availability; search should work even if suggestions fail.
- Scala bility: Handle huge spikes (e.g., breaking news).
- Freshness: New trending topics should appear within minutes/hours.

#### Scale Estimation

- DAU: 100 Million.
- Queries: 500 Million/day.
- Keystrokes: Assuming 5 letters/query, 2.5 Billion requests/day.
- QPS: ~30k average, 60k peak.

#### API Design

- GET /v1/suggest?q={prefix}&limit=5 - Returns a list of strings.

#### Database Design

- Trie (Data Structure): Stored in memory for fast prefix lookups.
- Redis/Key-Value: Stores the Trie nodes or a pre-calculated mapping (Prefix -> Top Suggestions).

#### High-Level Architecture

A client sends keystrokes to a Suggestion Service. This service queries a Trie (residing in memory/Redis) to find the node matching the prefix and returns the top-weighted children.

#### Detailed Components

- Data Collector: Logs search queries to Kafka.
- Aggre gator: Processes logs (Spark/Flink) to calculate query frequencies.
- Trie Builder: Builds the Trie offline and pushes to servers.
- Query Service: Handles incoming HTTP requests, queries the in-memory Trie.

#### Data Flow

1. Read Path: User types “ap” → API Gateway → Query Service → Looks up “ap” in Trie → Returns [“apple”, “amazon”, “applied”] → Frontend displays dropdown
2. Write Path: User completes search “apple” → Log to Kafka → Aggregator updates frequency count → Worker rebuilds Trie branch → Updates Query Service cache

#### Caching Strategy

Use a Browser-side cache (local storage) for the same session. Edge caching (CDN) for extremely common prefixes like “a”, “b”, “s”.

#### Database Choice

NoSQL/In-memory. A Trie is the specialized structure, but for persistence, use a Key-Value store where the Key is the prefix and the Value is a serialized list of suggestions.

#### Message Queue Usage

Kafka is used to decouple the search input path from the suggestion-ranking logic. This prevents analysis from slowing down user searches.

#### Storage Strategy

Raw logs in S3/HDFS. Processed Trie snapshots in Redis or specialized Trie-servers (like Lucene/Elasticsearch suggest-mode).

#### Security

Filter out profane or sensitive words using a “Blacklist” service. Rate limit requests per IP to prevent scraping of the suggestion database.

#### Scaling Strategy

Shard the Trie by the first 1-2 characters of the prefix (e.g., Server 1 handles ‘a’-‘m’, Server 2 handles ‘n’-‘z’). Replicate shards for high availability.

#### Bottlenecks

- Trie Size: If the Trie is too big for RAM. Mitigation: Store only prefixes up to length 10 and limit branch depth.
- Update Frequency: Rebuilding the whole Trie is expensive. Mitigation: Update the Trie incrementally or swap snapshots every hour.

#### Failure Handling

- Suggestion Service Down: The search bar simply shows nothing (fails gracefully); the user can still type and press enter.
- Trie Inconsistency: Use versioned snapshots to allow rollbacks if a bad batch is deployed.

#### Trade-offs

- Realtime vs. Batch Aggregation: Batch provides more stable/ accurate rankings but misses “Breaking News” for a few hours.
- Memory vs. Disk: Keeping Trie in memory for speed vs disk for cost/scale.

#### Alternative Approaches

- Elasticsearch (Completion Suggester): Highly effective but heavier than a custom Trie.
- SQL LIKE 'prefix%': Rejected immediately due to performance (O(N) vs O(Length of prefix)).

#### Optimization Tips

- Store the “Top 10” suggestions directly at each node in the Trie to avoid traversing all children at query time.
- Use “Top-K” algorithms (like Heavy Hitters) for frequency estimation.
- Client-side debouncing (wait 100ms before sending request) to reduce QPS.

#### Interview Tips

- Discuss the Trie structure first, then move to how to handle 100M users.
- Mention “Trie Sharding” and “Browser Caching.”
- Explain how you handle “Trending” topics (weighting by time decay).

#### Final Architecture Diagram

```text
Client --(Debounced)--> LB --> Query Service
|
[Trie in Redis/Local RAM] <--- [Trie Builder]
^
|
[Aggregator] <--- [Kafka] <--- [Log Collector]
```

#### Complexity Discussion

- Time: O(L) where L is the length of the search prefix.
- Space: O(Nodes
- Alphabet Size).

#### Example API Request

```http
GET /v1/suggest?q=fast&limit=3
```

#### Example API Response

```json
{
  "suggestions": [
    "fast and furious",
    "fasting",
    "fastly share price"
  ]
}
```

### Q10. Design a Web Crawler — Easy

#### Problem Statement

Design a system that crawls the World Wide Web, fetches content from pages, and stores the processed data for a search engine index.

#### Functional Requirements

- Given a set of seed URLs, visit all reachable pages.
- Extract text and metadata from HTML.
- Detect and extract links to discover new URLs.
- Handle various content types (HTML, PDF).

#### Non-functional Requirements

- Scalability: Must handle billions of pages.
- Politeness: Do not DOS websites; respect robots.txt.
- Extensibility: Easy to add new parsers (e.g., for images).
- Robustness: Handle broken links, slow servers, and “spider traps.”

#### Scale Estimation

- Volume: 1 Billion pages/month.
- QPS: ~400 pages/sec.
- Storage: 1B * 100KB/page = 100 TB/month.

#### API Design

- Crawlers typically don’t have public APIs. Internal management:
- POST /admin/seeds - Add new starting URLs.
- GET /admin/status - Current crawl throughput and error rates.

#### Database Design

- URL Frontier (Queue): Persistent queue of URLs to visit.
- URL Selection DB (Redis + HBase): Store all visited URLs to avoid duplicates (Bloom Filter for efficiency).
- Content Storage (S3/HDFS): Store raw HTML and extracted text.

#### High-Level Architecture

The system starts with a URL Frontier. “Fetchers” pull URLs from the frontier, download content, pass it to “Extractors” which find new links, and “Dedupers” ensure those links aren’t already visited before adding them back to the Frontier.

#### Detailed Components

- Seed URLs: The starting point (e.g., dmoz directory).
- URL Frontier: Prioritizes URLs (PageRank-based) and manages politeness.
- HTML Fetcher: Downloads pages using HTTP.
- DNS Resolver: Caches DNS lookups to speed up fetching.
- Content Deduplicator: Uses Checksums/SimHash to avoid storing duplicate content.

#### Data Flow

1. Pick URL from Frontier → Check robots.txt → Fetch page via HTTP
2. Store raw HTML in Blob store → Extract links
3. For each link: Check if visited (Bloom Filter) → If new, add to Frontier with a priority score

#### Caching Strategy

DNS Cache (local to fetchers). Robots.txt cache (Redis) to avoid re-fetching the same site’s rules. URL seen-set (Redis/Bloom) to prevent cycles.

#### Database Choice

HBase (or Cassandra) for the main URL metadata store because it scales horizontally to billions of rows. S3 for raw content because it’s cheap and durable.

#### Message Queue Usage

Kafka is used as the link between Fetchers and Extractors to allow them to scale independently. The Frontier itself is essentially a complex, prioritized distributed queue.

#### Storage Strategy

BigTable/HBase for the “Crawl DB.” S3 for the “Document Store.” Use compression (Zstd/Snappy) to reduce storage costs.

#### Security

Implement strict timeouts to prevent “slow loris” attacks from malicious sites. Run fetchers in sandboxed environments if executing JS. Follow robots.txt strictly.

#### Scaling Strategy

Distribute the Frontier by domain name (Host) across different worker nodes. This ensures all requests to wikipedia.org come from one node, making politeness easier to manage.

#### Bottlenecks

- DNS Resolution: Solved by local DNS caching and using a custom resolver.
- Frontier Size: Keeping billions of URLs in RAM. Mitigation: Use disk-backed queues (Kafka) or hybrid RAM/Disk models.
- Politeness Delay: Waiting between requests to the same IP. Mitigation: High parallelism across different domains.

#### Failure Handling

- Fetcher Node Crash: URL is timed out in the Frontier and reassigned (Acknowledgement pattern).
- Bad/Infinite Content: Limit maximum download size (e.g., 2MB) and maximum links per page.

#### Trade-offs

- DFS vs. BFS: BFS is usually preferred to find high-quality pages near the surface, though DFS might be faster for a single site.
- Freshness vs. Coverage: Do you re-crawl old pages or find new ones? (Solved by Priority Scoring).

#### Alternative Approaches

- Single Machine Crawler: Easy to build, but cannot scale to the web.
- Serverless (Lambda) Fetchers: Easy scaling, but DNS overhead and cost make it expensive for 1B pages.

#### Optimization Tips

- Use a Bloom Filter to check if a URL was already seen before hitting the primary DB.
- Use a Checkpointing mechanism to resume a crawl from the last known state.
- Implement Custom DNS to bypass OS-level bottleneck.

#### Interview Tips

- Emphasize Politeness and robots.txt; it shows real-world awareness.
- Explain Deduplication (SimHash/MinHash) — it’s a common deep-dive.
- Distinguish between the “Frontier” (the orchestrator) and the “Fetcher” (the worker).

#### Final Architecture Diagram

```text
[Seeds] -> [Frontier] <-------------------------
| |
[Fetcher Pool] (+ DNS Cache) |
| |
[Content Deduper] -> [S3 Storage] | (New Links)
| |
[Link Extractor] -------------------------
|
[Seen URL Filter (Bloom/Redis)]
```

#### Complexity Discussion

- Time: O(N) where N is number of pages.
- Space: O(U) where U is number of unique URLs (stored in Seen-set and Frontier).

#### Example API Request

```json
{
  "url": "https://example.com/page1",
  "priority": 0.85,
  "depth": 2,
  "last_crawled": null
}
```

#### Example API Response

```json
{
  "url": "https://example.com/page1",
  "content_hash": "a1b2c3d4",
  "links": [
    "https://example.com/page2",
    "https://google.com"
  ],
  "text_preview": "Welcome to example..."
}
```

### Q11. Design WhatsApp — Medium

#### Problem Statement

Design a real-time, highly scalable instant messaging application providing one-on-one and group chats. The system must support message delivery tracking (sent, delivered, read) and media sharing while maintaining low latency.

#### Functional Requirements

- One-on-one real-time messaging with delivery status.
- Group chats (up to 500 members).
- Online/Offline presence status.
- Media sharing (images, videos, documents).
- Persistent message history/synchronization across multiple devices.

#### Non-functional Requirements

- Low Latency: Real-time delivery (under 200ms).
- High Availability: 99.99% uptime for sending/receiving.
- High Consistency: Message ordering must be strictly preserved.
- Durability: Once a message is acknowledged, it must not be lost.
- Scale: Support billions of users and trillions of messages.

#### Scale Estimation

- DAU: 2 Billion users.
- QPS (Write): Assuming 50 messages/ user/day ≈ 1.1M messages per second.
- Storage: 100 bytes/msg → 100TB/day (text only).
- Bandwidth: 1.1M msg/s * 100 bytes ≈ 110MB/s (write) / 220MB/s (read/ delivery).

#### API Design

- POST /v1/messages: Sends a message (payload: recipient_id, body, type).
- GET /v1/conversations: Returns lists of recent chats.
- GET /v1/messages/{conv_id}: Paginated history.
- PUT /v1/presence: Updates user heartbeat/status.
- WebSocket /connect: Persistent connection for real-time duplex communication.

#### Database Design

- User Table (SQL): user_id, phone, name, last_seen.
- Message Table (NoSQL/Wide Column): msg_id (K), conv_id (Partition), sender_id, body, timestamp, status.
- Group Table (SQL): group_id, owner_id, members_list, created_at.

#### High-Level Architecture

Users connect to a Gateway Service via WebSockets (load-balanced by a TCP LB). The Gateway manages active sessions and routes messages to a Chat Service. The Chat Service persists data and interacts with a Presence Service and Push Notification Service for offline users.

#### Detailed Components

- WebSocket Gateway: Maintains millions of long-lived TCP connections for real-time delivery.
- Presence Service: A Redis-backed service tracking last_active timestamps via heartbeats.
- Message Store: A distributed NoSQL DB (Cassandra/HBase) optimized for heavy writes and sequential reads.
- Media Service/S3: Handles binary uploads, generating thumbnails and storing files in object storage.
- Notification Engine: Handles FCM/APNS for users not currently connected via WebSocket.

#### Data Flow

1. Write: Client A sends message via WebSocket → Gateway → Chat Service → Save to DB → Ack to Client A
2. Read: Chat Service checks if Client B is online → If yes, push via Client B’s Gateway → If no, send Push Notification

#### Caching Strategy

- Redis: Cache the most recent 50-100 messages for every active conversation (LRU).
- Edge Caching: CDNs for frequently accessed media files.
- Presence Cache: Key-value store for user status to avoid DB hits on every heartbeat.

#### Database Choice

Cassandra is the primary choice for messages because it handles massive write volumes and allows for efficient range queries on timestamp within a conversation_id partition. SQL (Postgres) is used for User profiles and Metadata due to ACID requirements for account management.

#### Message Queue Usage

Kafka acts as a buffer between the Chat Service and the persistent storage/analytics, ensuring that high-traffic spikes don’t overwhelm the database.

#### Storage Strategy

Text messages in Cassandra; Media files (images/videos) in Amazon S3 with CloudFront CDN; Metadata and session info in Redis.

#### Security

End-to-end encryption (Signal Protocol). TLS for all transit. OAuth 2.0/ JWT for session auth. Rate limiting at the Gateway to prevent spamming.

#### Scaling Strategy

Horizontal scaling of Gateway servers using a consistent hashing ring to map user_id to a specific server instance. Database sharding by conversation_id.

#### Bottlenecks

- Hot Partitions: Massively active group chats (mitigate by subsharding or in-memory fan-out).
- WebSocket Connections: OS limits on open file descriptors (mitigate by tuning kernel parameters and horizontal scaling).
- Presence Storms: Millions of status updates (mitigate by batching and increasing heartbeat intervals).

#### Failure Handling

- Gateway Down: Client reconnects to a different instance; messages are queued in the DB until retrieval.
- DB Down: Local write-ahead logging (WAL) and multi-region replication.
- Cache Miss: Fallback to Cassandra; background process repopulates Redis.

#### Trade-offs

- Consistency vs Availability: Optimized for Availability/Partition Tolerance (AP) in CAP; messages might arrive slightly out of order if clocks skew, fixed by client-side sequencing.
- Stateful vs Stateless: Gateways are stateful to maintain TCP sockets, increasing deployment complexity.
- Polling vs WebSockets: WebSockets chosen for lower latency despite higher server resource overhead.

#### Alternative Approaches

1. HTTP Long Polling: Rejected due to high header overhead and latency. 2. Firebase: Rejected for a custom enterprise solution due to lack of control over data sharding and cost at scale.

#### Optimization Tips

- Use Protocol Buffers instead of JSON for the WebSocket payload to reduce bandwidth.
- Implement “Sequence Numbers” generated by the client to handle deduplication.
- Use a “Gossip Protocol” for managing server-to-server health checks.

#### Interview Tips

- Focus on how to handle the “Last Seen” feature efficiently without killing the DB.
- Explain the difference between a “Pull” and “Push” model for message delivery.
- Discuss how to handle group chats with 100k+ members (fan-out challenges).

#### Final Architecture Diagram

```text
Client A --- (WS) ---> Load Balancer ---+
|
Client B --- (WS) ---> Gateway Service <--> Redis (Presence)
|
Chat Service <------> Kafka ----> Cassandra (Messages)
|
Notification Service ---> APNS/FCM
```

#### Complexity Discussion

- Time: Writing a message is O(1) in the DB (append-only logic).
- Space: O(N) where N is the total number of messages across the platform.
- Network: O(1) for 1-on-1; O(M) for a group chat of size M.

#### Example API Request

```json
{
  "to": "user_8821",
  "type": "text",
  "content": "Hello, world!",
  "client_msg_id": "uuid-999-123"
}
```

#### Example API Response

```json
{
  "status": "sent",
  "msg_id": "db-row-777",
  "timestamp": 1672531200
}
```

### Q12. Design Instagram — Medium

#### Problem Statement

Design a photo-sharing social media service where users can upload photos, follow others, and view a personalized feed of the top photos from people they follow.

#### Functional Requirements

- Photo uploading and storage.
- Feed generation (images from followers).
- Follow/Unfollow system.
- Search for users/hashtags.
- Likes and Comments on photos.

#### Non-functional Requirements

- High Availability: Read-heavy system (Feed) must always be available.
- Latency: Feed generation should be < 200ms.
- Eventual Consistency: Okay for follows/likes to show up with slight delays.
- Durability: Photos must never be lost.
- Scale: Support 500M+ DAU.

#### Scale Estimation

- DAU: 500M.
- Read/Write Ratio: 100:1.
- Uploads: 5M photos/day ≈ 60/sec.
- Storage: 5M * 2MB/photo ≈ 10TB/day ≈ 3.6PB/year.

#### API Design

- POST /v1/media/upload: Upload image, returns media_url.
- POST /v1/posts: Create post with media_url and caption.
- GET /v1/feeds/user/{id}: Retrieve the current user’s feed.
- POST /v1/friendships/create/{id}: Follow a user.

#### Database Design

- User (Postgres): user_id (PK), username, email.
- Photo (Postgres): photo_id (PK), user_id, path, lat, long, created_at.
- Follows (Post gres): follower_id, followee_id. Composite PK.
- Feed Cache (Redis): user_id -> List[photo_ids].

#### High-Level Architecture

Users upload photos via a Media Service to S3. Metadata is stored in a Postgres Cluster. A Feed Service pre-calculates feeds and stores them in Redis. A Follow Service manages the graph.

#### Detailed Components

- Media Service: Resizes images into multiple resolutions and stores them in S3.
- Feed Publisher: A background worker that pushes new post IDs to the Redis feeds of all followers (Fan-out).
- Search Service: Uses Elasticsearch to index captions and usernames.
- CDN: Amazon CloudFront to serve images globally with low latency.

#### Data Flow

1. Upload Path: Client → Media Service → S3 → DB (metadata) → Trigger Fan-out
2. Read Path: Client → Feed Service → Fetch pre-computed IDs from Redis → Hydrate metadata from DB/Cache → Return

#### Caching Strategy

- Redis (Feed): Stores the top 200-500 post IDs for each user’s feed.
- Redis (Metadata): Caches hot photo metadata and user profiles.
- CDN: Caches images at edge locations based on popularity.

#### Database Choice

Postgres with sharding (by user_id) for basic metadata because of relational integrity for flows like likes/follows. S3 for blob storage.

#### Message Queue Usage

Kafka is used for the asynchronous fan-out process. When a celebrity posts, we don’t update millions of feeds synchronously; we queue the task.

#### Storage Strategy

S3 for original and resized photos. Postgres for relational data. Elasticsearch for text-based search.

#### Security

Pre-signed URLs for S3 uploads. OAuth2 for session management. Rate limiting for post creation and follow actions.

#### Scaling Strategy

Sharding the Postgres DB by user_id. Implementing a “Pull” model for celebrities (Active users with millions of followers) so we don’t blow up Redis with fan-out.

#### Bottlenecks

- Celebrity Fanout: Millions of followers lead to massive writes (Mitigate: Hybrid model - Pull for celebs, Push for others).
- Image Delivery: Slow global loading (Mitigate: Multi-region S3 and aggressive CDN usage).
- Feed Refresh: Cache misses (Mitigate: Pre-warm cache for users who log in daily).

#### Failure Handling

- S3 Failure: Multi-region replication (Cross-Region Replication).
- Redis Cache Failure: Re-compute from DB (slow but functional).
- Database Slave Lag: Force read from master for critical self-updates.

#### Trade-offs

- Storage vs Computation: Storing multiple image sizes (storage) vs resizing on the fly (compute). Chose storage for latency.
- Consistency vs Avail ability: Eventual consistency for feed updates to maintain high availability.

#### Alternative Approaches

1. NoSQL for Metadata: Rejected because relational queries (follows/likes) are more natural in SQL. 2. Strict Push Model: Rejected because if a user has 50M followers, one post causes 50M Redis writes.

#### Optimization Tips

- Use “WebP” format for images to save 30% bandwidth.
- Implement lazy loading on the client side.
- Use consistent hashing for the Redis cluster.

#### Interview Tips

- Prioritize discussing the “Fan-out” problem; it’s the heart of social media design.
- Distinguish between “High-volume” users and “Normal” users.
- Mention CDNs early-on as images are the primary content.

#### Final Architecture Diagram

```text
Client ---> LB ---> Media Service ---> S3 & DB
|
---> Feed Service <--- Redis (Feed Cache)
|
---> Post Service ---> Kafka ---> Fan-out Workers
```

#### Complexity Discussion

- Feed Generation (Pull): O(F
- log P) where F is followers, P is posts per follower.
- Fanout (Push): O(N) where N is number of followers.

#### Example API Request

```json
{
  "caption": "Sunset at the beach",
  "media_id": "img_99821x",
  "location": {
    "lat": 34.0,
    "long": -118.0
  }
}
```

#### Example API Response

```json
{
  "post_id": "post_5544",
  "status": "published",
  "created_at": "2023-10-01T12:00:00Z"
}
```

### Q13. Design Twitter — Medium

#### Problem Statement

Design a social networking service where users can post short messages (tweets), follow other users, and view a timeline of tweets from followees.

#### Functional Requirements

- Post tweets (text, images, videos).
- Follow/Unfollow system.
- User Timeline (own tweets).
- Home Timeline (tweets from follows).
- Retweets and Likes.

#### Non-functional Requirements

- High Availability: Read availability is critical.
- Latency: Feed browsing must be snappy (< 100ms).
- Scalability: Support 300M+ active users.
- Eventual Consistency: Okay for tweets to take seconds to appear in all feeds.

#### Scale Estimation

- DAU: 300M.
- Tweets: 500M/day ≈ 6k/sec.
- Read Volume: 300B views/day → 3.5M/sec.
- Storage: 500M * 200 bytes ≈ 100GB/day (text).

#### API Design

- POST /v1/tweet: Body text and media IDs.
- GET /v1/timeline/home: Returns user’s home feed.
- GET /v1/timeline/user/{id}: Returns specific user’s posts.
- POST /v1/ tweet/{id}/like: Increments like count.

#### Database Design

- Tweet Table (NoSQL/Wide Column): tweet_id (K), author_id, content, created_at.
- User Table (SQL): user_id, name, bio, followers_count.
- Social Graph (Graph DB or SQL): follower_id, followee_id.

#### High-Level Architecture

Users interact via a Tweet Service. For the Home Timeline, a Fanout Service asynchronously injects tweet IDs into the Timeline Cache (Redis) of followers. A Search Service handles hashtags/trends.

#### Detailed Components

- Timeline Service: Serves the pre-computed Redis list of tweet IDs to the user.
- Fanout Worker: A fleet of consumers that update billions of Redis entries per day.
- Media Service: Handles S3 uploads for images/videos.
- Metrics Service: Tracks trending hashtags using a sliding window algorithm (Apache Flink/Storm).

#### Data Flow

1. Writing: User tweets → Tweet Service saves to DB → Push to Kafka → Fan-out workers → Update Redis lists of all followers
2. Reading: User requests timeline → Timeline Service fetches IDs from Redis → Hydrates tweet content from Tweet Cache → Returns JSON

#### Caching Strategy

- Redis: Crucial for Home Timelines (in-memory lists of tweet IDs).
- Tweet Content Cache: Redis/Memcached to store individual tweet JSON objects.
- User Cache: Store user profiles and follow relationships.

#### Database Choice

Cassandra for storing tweets because it supports high write throughput and scales linearly. Redis is the primary “database” for the live timeline to meet latency requirements.

#### Message Queue Usage

Kafka is mandatory to decouple the tweet creation from the expensive fan-out process, handling surges like the “Justin Bieber” effect.

#### Storage Strategy

Tweets in Cassandra, media in S3, timelines in Redis (limited to recent 1000 items per user).

#### Security

Rate limiting by API key/User ID to prevent bot spam. Content moderation filters using ML. TLS for privacy.

#### Scaling Strategy

Implementing a Hybrid Fanout. For normal users, use “Push” (update followers’ timelines on post). For celebrities (5M+ followers), use “Pull” (followers merge celebrity tweets at read-time) to avoid the “Thundering Herd” problem.

#### Bottlenecks

- Fanout for Celebs: Too many Redis writes (Mitigate: Hybrid model).
- Hot Tweets: High-volume likes/retweets on one ID (Mitigate: Distributed counters in Redis).
- Cold Users: Users who haven’t logged in for months (Mitigate: Don’t keep their timeline in Redis; recompute on login).

#### Failure Handling

- Redis Node Down: Rehash and rebuild from Cassandra.
- Kafka Lag: Fan-out delays will occur; users will see “eventual consistency” in action.

#### Trade-offs

- Memory vs Latency: Using massive amounts of RAM (Redis) to ensure millisecond latency for reads.
- Complexity vs Performance: Hybrid fan-out adds code complexity but is necessary for system stability.

#### Alternative Approaches

1. Fully Relational SQL: Rejected due to the inability to handle the write-heavy fan-out at scale. 2. Pure Pull Model: Rejected because calculating the timeline by querying all followees at read-time is too slow for 3.5M QPS.

#### Optimization Tips

- Only store Tweet IDs in the timeline cache, not the full tweet object.
- Use “Snowflake” ID generation for time-sortable unique IDs.
- Pre-fetch timelines for users who are currently active (WebSocket status).

#### Interview Tips

- Deep dive into the Snowflake ID generation algorithm.
- Focus on how you handle “Trending Topics” (count-min sketch or sliding windows).
- Understand the Hybrid Fan-out inside out.

#### Final Architecture Diagram

```text
Client ---> LB ---> Tweet Service ---> Cassandra
| |
| ---> Kafka ---> Fan-out Workers ---> Redis | (Timeline)
---> Timeline Service <-----------------------------/
```

#### Complexity Discussion

- Tweet Post: O(1) write; O(N) fan-out where N is followers.
- Timeline Read: O(1) lookup in Redis list.

#### Example API Request

```json
{
  "text": "Hello Twitter #firsttweet",
  "media_ids": [
    "m1",
    "m2"
  ]
}
```

#### Example API Response

```json
{
  "tweet_id": "1234567890",
  "created_at": "2023-10-01T12:00:00Z"
}
```

### Q14. Design a News Feed System — Medium

#### Problem Statement

Design the backend that generates a list of constantly updating status updates, photos, and activities from a user’s social graph (friends, followed pages, and groups).

#### Functional Requirements

- Aggregate content from friends, pages, and groups.
- Support text, images, and videos.
- Rank feed items based on relevance or time.
- Infinite scroll/pagination.

#### Non-functional Requirements

- Latency: Fast loading (< 300ms).
- Scalability: Handle millions of concurrent users.
- Availability: System must survive partial failures.
- Freshness: New posts should appear quickly.

#### Scale Estimation

- DAU: 100M.
- Avg Feed Refresh: 10 times per user/day.
- Total Feed Requests: 1B/day ≈ 12k/sec.
- Fanout: 1 post reaching 500 friends → 6M updates/sec (if using push).

#### API Design

- GET /v1/me/feed?page_token=...&limit=20: Main feed endpoint.
- POST / v1/feed/post: Create new entry.

#### Database Design

- Feed Metadata (NoSQL/SQL): post_id, user_id, content_ref, timestamp.
- User Graph: user_id, friend_id.
- Feed Cache (Redis): List of post IDs mapped to user_id.

#### High-Level Architecture

The Feed Generation Service pulls data from the Social Graph and Post Service. It ranks items using a Ranking Service and stores the result in a Feed Cache. The News Feed API serves this to the client.

#### Detailed Components

- Aggregator: Collects recent posts from all followed entities.
- Ranking Service: Uses ML models (weights for affinity, recency, type) to score posts.
- Post Cache: Stores the actual content of the posts (JSON).
- Fanout Service: Distributes new posts to the caches of active friends.

#### Data Flow

1. Post Creation: User A posts → Post Service → Fan-out Service → Find active friends → Update their Redis Feed Caches
2. Feed Retrieval: User B requests feed → Feed Service → Get IDs from Redis Cache → Hydrate with post content → Return

#### Caching Strategy

- Redis: For the pre-computed feeds of “Active Users” (users who logged in within the last 3 days).
- Content Cache: High-concurrency access to common post data.

#### Database Choice

Cassandra for storing the post metadata due to high availability and partitionability by user_id. Redis for the high-speed feed consumption layer.

#### Message Queue Usage

Kafka to handle the fan-out task. Since a single post can trigger 1000s of cache updates, an async queue is vital to prevent blocking the “Post” API.

#### Storage Strategy

Metadata in Cassandra; Binary assets (images) in S3. Redis for the “Hot Feed.”

#### Security

Privacy settings check (AuthZ) at the aggregation layer to ensure User A doesn’t see User B’s post if they aren’t friends.

#### Scaling Strategy

Separate the ranking into two phases: Retrieval (get 1000 candidate posts) and Ranking (score the top 50). This reduces computational overhead.

#### Bottlenecks

- Fanout explosion: For users with many friends.
- Ranking Latency: Complex ML models take too long (Mitigate: Use simple heuristic ranking for the first 10 items, deep rank the rest).
- Memory: Keeping feeds for all users is expensive (Mitigate: Only cache active users).

#### Failure Handling

- Cache Miss: Fallback to the “Pull” model (query DB for friends’ posts).
- Ranking Service Down: Fallback to simple “Chronological” sort.

#### Trade-offs

- Push vs Pull: Push (Fan-out) offers better read performance but uses more storage. Pull (Fan-in) saves storage but is slow on read.
- Freshness vs Latency: Using cached feeds might be 5 seconds behind but is much faster than live aggregation.

#### Alternative Approaches

1. Live Aggregation (Pull): Too slow for users with 1000+ friends. 2. Global Chronological Feed: Doesn’t provide personalized value (the “TikTok/Facebook” effect).

#### Optimization Tips

- Use a “Viewed” set in Redis to avoid showing the same posts twice to a user.
- Compress post metadata in Redis using MessagePack.

#### Interview Tips

- Talk about “Feed Ranking” (Recency, Affinity, Content Type).
- Discuss how you handle “Inactive Users” (don’t waste cache space).

#### Final Architecture Diagram

```text
Client ---> Feed API ---> Feed Cache | ^
| |
Post Service ---> Fan-out Worker
| |
Graph Service <------/
```

#### Complexity Discussion

- Read: O(1) from Redis list + O(K) hydration where K is page size.
- Write: O(N) fan-out.

#### Example API Request

```http
GET /v1/me/feed?limit=10

HTTP/1.1
Authorization: Bearer <token>
```

#### Example API Response

```json
{
  "items": [
    {
      "post_id": "123",
      "author": "John",
      "text": "Hi!"
    }
  ],
  "next_page_token": "abc_55"
}
```

### Q15. Design Reddit — Medium

#### Problem Statement

Design a massive social news aggregation and discussion platform where users can submit content, vote (up/down), and participate in deeply nested comment threads.

#### Functional Requirements

- Post submission (links, text, media).
- Subreddits (Communities).
- Voting system (Upvote/Downvote) and Karma calculation.
- Nested comment threads.
- Front page (Top/Hot ranking).

#### Non-functional Requirements

- High Availability: Readers should always see content.
- Consistency: Eventual consistency for vote counts and karma.
- Scala bility: Handle massive sudden spikes (e.g., “Game of Thrones” finale).
- Readheavy: 90% lurkers, 10% contributors.

#### Scale Estimation

- DAU: 50M.
- Votes: 100M+ per day.
- Comments: 10M/day.
- Read QPS: 50k+ QPS on the front page.

#### API Design

- POST /r/{subreddit}/submit: Create post.
- POST /v1/vote: (post_id, direction).
- GET /r/{subreddit}/comments/{post_id}: Fetch comment tree.
- GET /r/{subreddit}/ hot: Fetch ranked posts.

#### Database Design

- Posts (SQL/NoSQL): post_id, sub_id, author_id, title, url, score, created_at.
- Comments (SQL): comment_id, parent_id, post_id, body, path/ltree.
- Votes (NoSQL/KV): user_id, post_id, vote_type.

#### High-Level Architecture

Users access subreddits via a Subreddit Service. A Vote Service handles high-throughput voting and updates a Ranking Engine. The Comment Service retrieves nested structures, and the Front Page Service aggregates “Hot” posts.

#### Detailed Components

- Ranking Engine: Asynchronously calculates scores based on Reddit’s “Hot” algorithm (𝑆𝑐𝑜𝑟𝑒=𝑓(𝑢𝑝𝑣𝑜𝑡𝑒𝑠,𝑑𝑜𝑤𝑛𝑣𝑜𝑡𝑒𝑠,𝑎𝑔𝑒)).
- Comment Service: Uses a “path” or “closure table” logic to fetch massive trees efficiently.
- Vote Buffer: A Kafka queue that absorbs high-frequency votes to avoid locking the DB.
- Post Service: Manages content and metadata.

#### Data Flow

1. Voting: User votes → Vote Service → Update Redis (for immediate UI feedback) → Kafka → Batch update DB & Recalculate Ranking
2. Reading: User fetches Subreddit → Feed Service → Fetches pre-ranked list from Redis → Subreddit JSON returned

#### Caching Strategy

- Redis: Hot subreddit feeds (top 1000 posts).
- Memcached: Fragment caching for comment HTML/JSON.
- CDN: Assets and static pages.

#### Database Choice

Postgres with the ltree extension is excellent for nested comments. Cassandra for the Vote log because of its write-efficiency.

#### Message Queue Usage

Kafka is used to decouple the ranking logic and karma updates from the voting action.

#### Storage Strategy

S3 for images/videos. Postgres for relational data (Comments, Subs). Redis for rankings.

#### Security

Anti-spam/Anti-manipulation (Sybil attack protection) for voting. Rate limiting on post creation.

#### Scaling Strategy

Read-replicas for Postgres to handle the heavy read load. Sharding by subreddit_id to distribute load across databases.

#### Bottlenecks

- Nested Comments: Very deep trees can be slow (Mitigate: Paginate or collapse branches, use path enumeration).
- Vote Fraud: Bot farms (Mitigate: IPbased rate limiting and ML fraud detection).
- Front Page Contention: Global lock or high contention on “Hot” items (Mitigate: Cache the pre-calculated ranking).

#### Failure Handling

- Vote Service Down: Users see cached counts; votes are queued at the edge.
- Database Master Down: Failover to a standby replica.

#### Trade-offs

- Accuracy vs Performance: Showing “approximate” vote counts (Fuzzing) to prevent exact bot tracking and improve performance.
- Nested vs Flat: Nested comments are harder to fetch but essential for Reddit’s identity.

#### Alternative Approaches

1. Adjacency List for Comments: Rejected because recursive queries are expensive. 2. Synchronous Karma Updates: Rejected because it would bottleneck the voting process.

#### Optimization Tips

- Use “Vote Fuzzing” to confuse bots.
- Pre-render comment trees for the top 100 posts.
- Use a “Bloom Filter” to check if a user has already voted on a post in the local cache.

#### Interview Tips

- Be ready to explain the “Hot” ranking algorithm (logarithmic scale for votes, linear for time).
- Discuss the data structure for nested comments (ltree, path enumeration, or closure tables).

#### Final Architecture Diagram

```text
Client ---> LB ---> Vote Service ---> Kafka ---> Ranking Engine
| |
---> Post/Comment Service ---+---> Redis (Hot Cache)
|
---> Postgres (ltree)
```

#### Complexity Discussion

- Vote: O(1) in Kafka; O(1) update in DB.
- Comment Tree: O(N) where N is the number of nodes in the visible tree branch.

#### Example API Request

```json
{
  "post_id": "p_9982",
  "direction": 1
}
```

#### Example API Response

```json
{
  "status": "success",
  "new_score": 1402,
  "voted": 1
}
```

### Q16. Design Pinterest — Medium

#### Problem Statement

Design a highly scalable visual discovery engine where users can save, organize, and discover images (Pins) on thematic Boards. The system must handle massive image storage and provide a personalized feed of content based on user interests.

#### Functional Requirements

- Users can upload and create “Pins” (images with metadata and URLs).
- Users can create “Boards” and save Pins to them.
- Users can follow other users or specific boards.
- System generates a personalized “Home Feed” for each user.
- Search functionality for Pins based on keywords or tags.

#### Non-functional Requirements

- High Availability: The system must be available for reading even during partial failures.
- Low Latency: Feed loading and image rendering must be near-instant (< 200ms for feed metadata).
- Eventual Consistency: It is acceptable if a follower sees a new Pin a few seconds late.
- High Scalability: Support hundreds of millions of users and billions of Pins.

#### Scale Estimation

- DAU: 500M.
- Write QPS: ~5k/sec (new pins/saves).
- Read QPS: ~500k/sec (feed browsing).
- Storage: 500M users * 20 pins/year * 1MB/image = 10 PB per year.
- Bandwidth: 500k QPS * 2MB per page view = 1 TB/sec egress.

#### API Design

- POST /v1/pins: {media_url, description, board_id} -> returns pin_id.
- GET /v1/feeds/home: {page_token} -> returns list of pin_objects.
- POST /v1/boards: {name, privacy} -> returns board_id.
- POST /v1/pins/{id}/save: {board_id} -> returns success.
- GET /v1/search: {query, filters} -> returns ranked results.

#### Database Design

- Users Table (SQL): user_id, email, profile_data.
- Pins Table (NoSQL/Cassandra): pin_id, creator_id, image_url, metadata, created_at.
- Boards Table (SQL): board_id, owner_id, name.
- Board_Pins Mapping (Cassandra): board_id, pin_id, added_at (partitioned by board_id).
- Follows Table (Cassandra): follower_id, followee_id.

#### High-Level Architecture

The system utilizes a microservices architecture. A Load Balancer distributes requests to an API Gateway. The Pin Service handles uploads, the Feed Service generates timelines, and the Graph Service manages follows. Data is stored in a combination of RDBMS (metadata), Cassandra (high-volume mappings), and S3 (images).

#### Detailed Components

- Image Processor: Asynchronous workers that resize images into multiple thumbnails and run ML tagging.
- Feed Worker: Pre-computes home feeds for active users and stores them in Redis.
- Search Consumer: Syncs new Pins to an Elasticsearch/OpenSearch cluster.
- Graph Service: Dedicated service managing the complex relationships between users, boards, and pins.

#### Data Flow

1. Write Path: Client uploads image to S3 → Pin Service saves metadata to DB → Message Queue triggers Image Processor → Search Indexer updates
2. Read Path: Client requests feed → API Gateway calls Feed Service → Service fetches pre-computed IDs from Redis → Hydrates metadata from Pin Cache → Returns JSON

#### Caching Strategy

- Edge: CDN (CloudFront/Akamai) for all static images/thumbnails.
- Application: Redis for pre-computed user feeds (LRU eviction).
- Database: Lookaside cache for Pin metadata to reduce DB load.

#### Database Choice

We use Cassandra for the Pins and Board-mappings due to its linear scalability and high write throughput. Postgres (sharded) is used for core User/ Board metadata where ACID compliance is needed for account management. S3 is the only choice for cost-effective, durable binary storage.

#### Message Queue Usage

Kafka is used to decouple the Pin creation path from downstream tasks like thumbnail generation, NSFW content filtering, and search indexing.

#### Storage Strategy

Original images are stored in S3 “Standard”. Thumbnails (various sizes) are stored in S3 with “Intelligent Tiering”. Metadata is sharded across DB nodes.

#### Security

- OAuth2/JWT for session management.
- Pre-signed URLs for secure S3 uploads.
- Encryption at rest (AES-256) for PII.
- Rate limiting at API Gateway (NGINX/Kong).

#### Scaling Strategy

- 10x load: Horizontal scaling of Feed Workers.
- Read Replicas for SQL databases.
- Geographic sharding of Cassandra clusters to reduce crossregion latency.

#### Bottlenecks

- Fanout: Popular boards/users can cause write spikes; mitigated by asynchronous background processing.
- Hot Shards: Viral pins can overwhelm a single DB node; mitigated by aggressive caching and CDN.
- Feed Freshness: Balancing pre-computation with real-time updates; mitigated by a hybrid “Pull/Push” model for celebrities.

#### Failure Handling

- S3 down: Use multi-region replication.
- Redis cache miss: Fall back to DB (with circuit breakers to prevent DB collapse).
- Kafka lag: Scale consumers; prioritize real-time feed visibility over search indexing.

#### Trade-offs

- Eventual vs Strong Consistency: Chose eventual consistency for Pins to favor availability.
- Precomputed vs Onthefly Feeds: Pre-compute for speed, despite higher storage cost.
- NoSQL vs SQL: Sacrificed complex joins in NoSQL for massive horizontal write scale.

#### Alternative Approaches

- Pullbased feed calculation: Rejected because it’s too slow for users following 1000+ boards.
- Storing images in DB (BLOB): Rejected due to performance degradation and cost.

#### Optimization Tips

- Use WebP format for images to reduce bandwidth by 30%.
- Implement Bloom Filters to quickly check if a user has already seen a Pin.
- Batch DB writes for “saves” to reduce IOPS.

#### Interview Tips

- Focus on the “Feed Generation” logic; it’s the core of the system.
- Be ready to discuss how you handle “Celebrity” boards (millions of followers).
- Emphasize image optimization and CDN strategies.

#### Final Architecture Diagram

```text
[Client] -> [LB] -> [API Gateway]
|
------------------------------
| | |
[Pin Service] [Feed Service] [Graph Service]
| | |
[Kafka] [Redis Feed] [Cassandra/PG]
|
[Image Workers] -> [S3 / CDN]
```

#### Complexity Discussion

- Time: Feed fetching is O(1) from Redis.
- Space: O(NM) where N is users and M is feed size for Redis storage. Network: High egress for image delivery; minimized by CDN edge caching.

#### Example API Request

```http
POST /v1/pins
Content-Type: application/json

{
  "image_url": "s3://bucket/raw_img_123.jpg",
  "description": "Mid-century modern decor",
  "board_id": "board_456"
}
```

#### Example API Response

```json
{
  "pin_id": "pin_987",
  "status": "processing",
  "created_at": "2023-10-27T10:00:00Z"
}
```

### Q17. (Medium) Design YouTube — Medium

#### Problem Statement

Design a global video-sharing platform where users can upload, view, and comment on videos. The system must handle petabytes of data, provide low-latency playback across various network conditions, and support massive concurrent viewership.

#### Functional Requirements

- Users can upload videos.
- Users can search for videos.
- Users can view videos (streaming).
- System tracks view counts and supports likes/comments.
- Video recommendation/trending list.

#### Non-functional Requirements

- Availability: Extremely high (99.99%).
- Reliability: Uploaded videos must never be lost.
- Latency: Low start-up time (TTFB) for video playback.
- Scalability: Handle 100M+ DAU and millions of concurrent streams.

#### Scale Estimation

- Uploads: 500 hours of video per minute.
- Average Video Size: 500MB (HD).
- Total Storage: 500 hours * 60 min * 500MB = 15 PB/day.
- Views: 5 Billion views/day.
- Egress Bandwidth: Massive (Tbps range).

#### API Design

- POST /v1/videos/upload: (Multipart upload) -> returns video_id.
- GET / v1/videos/{id}: Returns metadata and stream manifest URL (HLS/DASH).
- POST /v1/ videos/{id}/comments: {text} -> returns comment_id.
- GET /v1/search: {q} -> list of videos.

#### Database Design

- Metadata (Vitess/MySQL): video_id, uploader_id, title, description, storage_path, status.
- Comments (Cassandra): video_id, comment_id, user_id, text, timestamp.
- User Data (Postgres): user_id, subscription_list, history.
- View Counts (Redis + Cassandra): Counter-specific sharded store.

#### High-Level Architecture

Uploads flow through a Transcoding Pipeline which creates multiple resolutions. Metadata is stored in a sharded RDBMS. Videos are served via a global Content Delivery Network (CDN). A distributed Search Service (Elasticsearch) handles discovery.

#### Detailed Components

- Transcoder: Converts raw uploads into HLS/DASH formats and fragmented MP4s (360p, 720p, 1080p, 4K).
- Blob Storage: S3 or GCS for raw and processed video chunks.
- CDN: Points of Presence (PoPs) globally that cache video segments closer to users.
- Video Feed/Manager: Generates personalized recommendations via ML model inferences.

#### Data Flow

1. Upload: Client → Upload Service → S3 → Notify Transcoder via Queue → Transcoder writes chunks to S3 → Update Metadata DB
2. Playback: Client → Metadata API → Get Streaming URL → Request segments from CDN → CDN fetches from S3 if miss → Client player assembles chunks

#### Caching Strategy

- Video Chunks: Global CDN (edge caching).
- Metadata: Redis clusters for popular videos.
- Thumbnails: CDN and local browser cache.

#### Database Choice

Vitess (MySQL sharding) for metadata because of its proven scale with YouTube’s own history, providing ACID for uploader stats while scaling horizontally. Cassandra for comments/likes due to high write frequency and timeseries nature.

#### Message Queue Usage

Kafka is critical for the transcoding pipeline (handling retries) and for streaming analytics (view count processing, logs for ads).

#### Storage Strategy

HLS/DASH segments stored in S3. Cold storage (Glacier) for original high-res binaries after a period of inactivity.

#### Security

- DRM (Widevine, FairPlay) for copyright protection.
- Content ID system for fingerprinting and copyright strikes.
- Geofencing for region-locked content.
- TLS for all API traffic.

#### Scaling Strategy

- Horizontal scaling of Transcoding workers.
- Multi-level CDN (ISP-level caches).
- Database sharding by video_id.

#### Bottlenecks

- Transcoding Delay: Solved by priority queues (short videos first).
- Hot Videos: Solved by multi-layer caching and predictive caching in CDNs.
- Storage Cost: Solved by deduplication and deleting low-view old content (if policy allows).

#### Failure Handling

- Transcoder failure: Kafka keeps the message; another worker picks it up.
- CDN failure: Redirect to secondary CDN or Origin S3 (slower but works).
- Database downtime: Use read-replicas; switch to “Read-Only” mode.

#### Trade-offs

- Quality vs Latency: Adaptive Bitrate Streaming (ABR) adjusts quality to prevent buffering.
- Consistency vs Availability: View counts are eventually consistent to handle massive scale.
- Cost vs Durability: Store multiple copies of popular videos, fewer of unpopular ones.

#### Alternative Approaches

- P2P Streaming: Rejected due to high complexity and modern high-speed infrastructure.
- Single Resolution Storage: Rejected because it fails to support low-bandwidth users.

#### Optimization Tips

- Use H.265 or AV1 codecs for better compression.
- Enable “Zero-copy” file transfers in the OS.
- Edge-side includes (ESI) for dynamic UI components.

#### Interview Tips

- Focus on the Transcoding Pipeline.
- Mention Adaptive Bitrate Streaming (ABR).
- Talk about the trade-off of “View Counting” (concurrency vs accuracy).

#### Final Architecture Diagram

```text
[Client] -> [CDN] -> [S3 Storage]
|
[API Gateway] -> [Metadata Service] -> [Vitess Cluster]
|
[Upload Service] -> [Kafka] -> [Transcoder] -> [S3]
```

#### Complexity Discussion

- Time: Transcoding is O(n) where n is video length.
- Space: O(kn) where k is the number of target resolutions. Network: Heavily dominated by O(n) media streaming.

#### Example API Request

```http
GET /v1/videos/vid_789/stream Accept: application/x-mpegURL
```

#### Example API Response

```json
{
  "manifest_url": "https://cdn.youtube.com/vid_789/master.m3u8",
  "thumbnail_url": "https://cdn.youtube.com/thumb/vid_789.jpg",
  "title": "System Design 101"
}
```

### Q18. (Medium) Design Spotify — Medium

#### Problem Statement

Design a music streaming service that allows users to search for songs, create playlists, and stream audio. The system must support millions of concurrent listeners and provide a personalized experience through “Discover Weekly” features.

#### Functional Requirements

- Users can stream audio tracks.
- Users can search by artist, album, or song.
- Users can create and manage playlists.
- Users can follow artists and friends.
- System tracks listening history for recommendations.

#### Non-functional Requirements

- Low latency: Audio playback must start instantly (< 500ms).
- Gapless playback: No interruptions between songs.
- Scalability: Support 500M+ users.
- Offline mode: Support for downloading tracks (client-side mobile requirement).

#### Scale Estimation

- DAU: 200M.
- Songs: 100M tracks.
- Read QPS (Metadata): 100k/sec.
- Streaming Bandwidth: 200M users * 320kbps = Huge (managed by CDN and P2P).
- Storage: 100M songs * 5MB = 500 TB.

#### API Design

- GET /v1/search: {query} -> list of tracks/artists.
- GET /v1/tracks/{id}/ stream: Returns audio stream URL.
- POST /v1/playlists: {name, tracks} -> returns playlist_id.
- PUT /v1/me/player/play: {track_uri} -> starts playback.

#### Database Design

- Music Metadata (Postgres): track_id, artist_id, album_id, duration, genre.
- User/Playlists (Cassandra): playlist_id, user_id, track_ids (list), modified_at.
- Search Index: Elasticsearch for fuzzy matching on names.
- User History (BigTable): Massive time-series of every song played.

#### High-Level Architecture

Audio files are stored in S3 and served via CDN. Metadata is stored in sharded SQL. A Playlist service manages user collections. A dedicated Recommendation Engine (Spark/Flink) processes listening history to generate personalized feeds.

#### Detailed Components

- Audio Storage: Encrypted Ogg Vorbis or AAC files stored in S3.
- Search Service: Elasticsearch cluster for real-time artist/song lookups.
- Playback Service: Manages state of the active player (active device, current song).
- Recommendation Engine: Offline batch processing (Hadoop) for Discover Weekly and online streaming for “Radio”.

#### Data Flow

1. Playback: User clicks Play → Client calls Playback Service → Metadata DB provides S3/CDN URL → Client fetches audio chunks
2. Playlist Update: User adds song → Playlist Service updates Cassandra → Cache invalidates for that playlist

#### Caching Strategy

- Metadata: Redis for popular tracks/artists.
- User Session: Redis for current playback state.
- Audio Chunks: Local device cache (crucial for mobile) and CDN.

#### Database Choice

Postgres for core catalog metadata because relational sanity is needed for Album-Artist-Track hierarchies. Cassandra for playlists and user social graphs due to high write volume and flexibility.

#### Message Queue Usage

Kafka streams every “Play” event to the analytics engine and the recommendation engine.

#### Storage Strategy

Multi-bitrate audio files in S3. High-quality (320kbps) and Lowquality (96kbps) versions stored separately.

#### Security

- Widevine/FairPlay DRM.
- Encrypted audio transport.
- OAuth for thirdparty integrations (e.g., smart speakers).

#### Scaling Strategy

- CDN at the edge for audio content.
- Micro-frontend or microservice architecture to scale teams and features.
- Sharding Cassandra by playlist_id or user_id.

#### Bottlenecks

- Search Latency: Solved by optimizing Elasticsearch and using completion suggesters.
- Data Volume for Recs: Solved by using sampling or focused history windows.
- Concurrent Streams: Enforcing one-stream-per-account limits via Redis locking.

#### Failure Handling

- CDN down: Fallback to alternative CDN provider.
- Metadata DB lag: Use stale cache values; playback shouldn’t stop.
- Search down: Fallback to basic cached “Top Charts”.

#### Trade-offs

- Audio Quality vs Bandwidth: Offer settings to users to balance data usage.
- Batch vs Realtime Recs: Monthly discover lists are batch (cheaper); “Start Radio” is real-time.

#### Alternative Approaches

- P2P Delivery (Legacy Spotify): In early days, Spotify used P2P to reduce server costs; rejected now as mobile/mobile-data grew.
- Storing Audio in DB: Rejected immediately (unscalable).

#### Optimization Tips

- Use HTTP/2 or QUIC for faster chunk delivery.
- Prefetch the next song in the playlist to the client.
- Compress artist images and metadata.

#### Interview Tips

- Highlight the User History/Recommendation loop.
- Discuss Offline Mode synchronization.
- Mention how to handle synchronized playback across devices (Connect).

#### Final Architecture Diagram

```text
[Client] -> [LB] -> [API Gateway]
|
-----------------------------------
| | | |
[Search Svc] [Playlist Svc] [Audio Svc] [Rec Engine]
| | | |
[E-Search] [Cassandra] [CDN/S3] [Kafka/Spark]
```

#### Complexity Discussion

- Time: Metadata retrieval is O(1) with cache.
- Space: Cassandra grows O(UP) where U=users, P=avg playlist size. Network: Audio streaming is the primary cost (bandwidth).

#### Example API Request

```http
GET /v1/me/player/currently-playing
Authorization: Bearer <token>
```

#### Example API Response

```json
{
  "item": {
    "name": "Blinding Lights",
    "artist": "The Weeknd",
    "duration_ms": 200000
  },
  "is_playing": true
}
```

### Q19. (Medium) Design Netflix — Medium

#### Problem Statement

Design a global video-on-demand platform. Focus on highquality video delivery, a sophisticated recommendation engine, and a resilient architecture that operates across multiple AWS regions for extreme availability.

#### Functional Requirements

- Users can browse and search a catalog of movies/ shows.
- Users can stream high-definition video.
- Users can manage profiles and watchlists.
- System supports “Pick up where you left off” (Playback position).
- System supports multiple devices (TV, Mobile, Web).

#### Non-functional Requirements

- Availability: 99.999% (Active-Active multiregion).
- Low Latency: Instant video start.
- Scalability: Support 250M+ subscribers sharing accounts.
- Quality: Support up to 4K/HDR.

#### Scale Estimation

- Subscribers: 250M.
- Concurrent users: 20M+.
- Peak traffic: ~30% of total US internet traffic at night.
- Storage: Petabytes of video, replicated globally.

#### API Design

- GET /v1/titles/{id}: Metadata for a movie.
- GET /v1/play: Returns Playback Context (URL, subtitles, DRM).
- POST /v1/play/progress: Updates timestamp for “Continue Watching”.
- GET /v1/home: Returns categorized rows (Personalized).

#### Database Design

- Title Metadata (Cassandra): title_id, metadata_json, categorization.
- User Profiles (Postgres/CockroachDB): user_id, profiles, billing_info.
- Playback History (Cassandra): profile_id, title_id, offset, last_watched.
- Asset Map: DynamoDB for mapping video ID to S3 file locations across regions.

#### High-Level Architecture

Uses a “Control Plane” (running on AWS) for metadata, search, and auth, and a “Data Plane” (Open Connect) which is a custom CDN of hardware appliances co-located inside ISPs to serve the actual bits.

#### Detailed Components

- Archer (Encoding Pipeline): Massively parallel workers creating thousands of versions of one movie (codec, resolution, language).
- Open Connect (OCA): Netflix’s private CDN.
- Playback Service: Handles the negotiation of which OCA a client should stream from.
- Zuul: The API Gateway (Edge Service) for routing, filtering, and security.

#### Data Flow

1. Metadata Browsing: Client → Zuul → Metadata Service → Cassandra → Client
2. Playback Start: Client → Playback Svc → Redirect to nearest Open Connect Appliance (OCA) → Client fetches bytes via HLS/DASH
3. State Sync: Client periodically sends offset to Playback History Svc → Cassandra

#### Caching Strategy

- EVCache: Netflix’s custom distributed memcached-based wrapper for heavy metadata caching.
- Regional Caching: Keeping data in the user’s specific AWS region.

#### Database Choice

Cassandra is the workhorse for Netflix for its multi-region replication capabilities and high availability. CockroachDB or sharded Postgres for billing/accounts where consistency is non-negotiable.

#### Message Queue Usage

Kafka for real-time telemetry (every button click, pause, play) to power the “Viewing Experience” dashboard and recommendations.

#### Storage Strategy

Originals in S3. Encoded chunks are pushed to Open Connect appliances at the edges of the network.

#### Security

- MSL (Message Security Layer) for secure communication between client and server.
- DRM (PlayReady, Widevine).
- Complex Device Auth for TVs vs Web.

#### Scaling Strategy

- Multi-region deployment (US-East, US-West, EU-West). If one region fails, traffic is failed over in minutes.
- Predictive scaling of microservices based on time-of-day.

#### Bottlenecks

- Cold Starts: Scaling up EC2 instances fast enough for evening peaks.
- Regional Failover: Managing state (like playback markers) during a total region outage.
- ISP Throttling: Mitigated by Open Connect appliances inside ISP networks.

#### Failure Handling

- Chaos Engineering: Hystrix (Circuit Breaking) and Simian Army to test resilience.
- Fallback content: If recommendation fails, show a static “Popular” list.
- Degraded Video: Lower bitrate if network congestion occurs.

#### Trade-offs

- Cost vs Performance: Investing in custom CDN hardware (Open Connect) vs paying for third-party CDNs.
- Consistency: Playback markers are “eventually consistent” across devices.

#### Alternative Approaches

- Public CDN (Akamai/CloudFront): Rejected for the core video delivery due to prohibitive costs at Netflix scale.
- Standard SQL DB for history: Rejected due to high write-volume and the need for global replication.

#### Optimization Tips

- Pershot encoding: Adjusting bitrate based on the complexity of the scene (e.g., an action scene needs more bits than a static wall).
- Parallel encoding: Splitting a movie into 5-minute chunks for faster processing.

#### Interview Tips

- Explain Open Connect; it’s the differentiator between Netflix and a standard YouTube.
- Discuss Multiregion Availability.
- Mention Chaos Engineering.

#### Final Architecture Diagram

```text
[Client] --- (Request) ---> [API Gateway (AWS)]
| |
(Video Stream) [Microservices / Cassandra]
| |
[Open Connect (ISP)] <--- (Push) -- [Control Plane]
```

#### Complexity Discussion

- Time: Metadata retrieval O(1) from local EVCache.
- Space: Massive O(VR) where V is video content and R is resolutions/codecs. Network: The primary challenge; handled by ISP-local distribution.

#### Example API Request

```http
POST /v1/play/progress

{
  "title_id": "movie_123",
  "offset_seconds": 3450,
  "profile_id": "p_abc"
}
```

#### Example API Response

```json
{
  "status": "success",
  "timestamp": "2023-11-01T20:00:00Z"
}
```

### Q20. (Medium) Design Discord — Medium

#### Problem Statement

Design a real-time communication platform for communities. It must support persistent chat rooms (Guilds/Servers), direct messaging, and realtime voice/video. The system must handle high-velocity messaging and thousands of concurrent users in a single channel.

#### Functional Requirements

- Users can create/join Servers (Guilds).
- Text channels for persistent messaging.
- Real-time presence (Online/Idle/DND).
- Voice/Video channels.
- Push notifications for mentions and DMs.

#### Non-functional Requirements

- Low Latency: Message delivery < 100ms.
- High Availability: Critical for community interaction.
- Consistency: Messages must appear in the same order for everyone.
- Scalability: Support millions of servers, some with 500k+ members.

#### Scale Estimation

- MAU: 150M.
- Messages: Billions per day.
- Write QPS: 50k - 100k/sec.
- Presence Updates: Millions per second (requires stateful handling).

#### API Design

- POST /v1/channels/{id}/messages: Sends a text message.
- GET /v1/ channels/{id}/messages: Fetch message history.
- POST /v1/guilds: Create a new server.
- WebSocket /gateway: Persistent connection for real-time events.

#### Database Design

- Guild/User Metadata (Postgres/CockroachDB): Small but critical relational data.
- Messages (ScyllaDB/Cassandra): channel_id, message_id (snowflake), author_id, content, timestamp. (Partitioned by channel_id).
- Presence (Redis): Volatile store for user_id -> status.

#### High-Level Architecture

Discord uses a “Gateway” (WebSocket) layer to maintain persistent connections. Text messages are stored in ScyllaDB for fast writes/reads. Voice is handled via separate Voice Servers (SFUs). Media is stored in S3.

#### Detailed Components

- Gateway Service: Elixir/Erlang-based cluster that manages WebSockets and routes events.
- Guild Service: Manages server membership and permissions.
- Presence Service: Highly optimized for broadcasting status changes to friends/guild-mates.
- Voice Media Relay: Specialized servers for lowlatency UDP audio/video.

#### Data Flow

1. Sending Message: Client A → WebSocket Gateway → Message Service → Write to ScyllaDB → Gateway broadcasts to all online Clients in that channel
2. Presence Update: Client A status change → Gateway → Presence Service → Identify all friends/guild-mates → Push event via their respective Gateway connections

#### Caching Strategy

- Guild Metadata: Redis.
- Permissions: Heavy caching on Gateway nodes to avoid DB lookups for every message.
- Recent Messages: Inmemory cache in the Gateway for hot channels.

#### Database Choice

ScyllaDB (a C++ rewrite of Cassandra) is chosen for its predictable low latency and high throughput for message storage. Postgres for relational data like guild settings.

#### Message Queue Usage

RabbitMQ or Pulsar for internal inter-service communication and task distribution (e.g., generating link previews).

#### Storage Strategy

ScyllaDB for the message text. Amazon S3 for user-uploaded images and files.

#### Security

- WebSockets over TLS (WSS).
- Fine-grained Role-Based Access Control (RBAC).
- DDoS protection (Cloudflare) and aggressive rate-limiting on API/Gateway.

#### Scaling Strategy

- Sharding the Gateway by user_id.
- Using Elixir/Phoenix for high concurrency on a single node.
- Geography-based Voice Relay placement.

#### Bottlenecks

- Presence Fanout: A user with 1000 friends going online triggers 1000 events. Mitigated by limiting updates to active guilds.
- Large Guilds: A message in a 500k-member guild is a massive broadcast challenge. Mitigated by lazyloading the member list.
- Message Sequencing: Solved by using “Snowflake” IDs for chronological ordering without a central counter.

#### Failure Handling

- Gateway node dies: Clients automatically reconnect to a new node.
- ScyllaDB node down: Quorum-based reads/writes maintain availability.
- Voice Server fails: Client switches to another relay in the same region.

#### Trade-offs

- Stateful vs Stateless: Chose stateful Gateways (WebSockets) for performance, despite scaling complexity.
- Eventual Consistency: Presence is eventually consistent to protect the system from O(N^2) storming.

#### Alternative Approaches

- HTTP Long Polling: Rejected due to high overhead and latency.
- MongoDB for Messages: Rejected because it struggles with the specific fragmentation/compaction needs of trillions of small messages.

#### Optimization Tips

- Use Discord’s Snowflake IDs for distributed, time-ordered primary keys.
- Use Protobufs over WebSockets to reduce the size of the frames.
- Compress presence updates by batching.

#### Interview Tips

- Mention Elixir/Erlang specifically for the Gateway’s concurrency.
- Focus on the Fanout problem (how to update 100k people at once).
- Be ready to explain how Voice (WebRTC/UDP) differs from Text (TCP/WS).

#### Final Architecture Diagram

```text
[Client] <---(WSS)---> [Gateway Cluster (Elixir)]
| |
(UDP) [Message Svc] -- [ScyllaDB]
| |
[Voice Relay] [Presence Svc] -- [Redis]
```

#### Complexity Discussion

- Time: Message delivery is O(M) where M is the number of online users in the channel.
- Space: Message storage is O(Total Messages).
- Network: Significant persistent connection overhead; requires many small Gateway nodes.

#### Example API Request

```json
// WebSocket Frame (JSON)
{ "op": 2, // Identify "d": { "token": "my_auth_token", "properties": { "$os": "linux" }
}
}
```

#### Example API Response

```json
{
  "t": "MESSAGE_CREATE",
  "d": {
    "content": "Hello World",
    "author": {
      "username": "John"
    },
    "channel_id": "999"
  }
}
```

### Q21. Design Slack — Medium

#### Problem Statement

Design a real-time team collaboration platform that supports persistent messaging, channels, and direct messages. The system must handle millions of concurrent users with sub-second latency for message delivery across multiple devices.

#### Functional Requirements

- One-to-one and group (channel) instant messaging.
- Persistent chat history searchable by users.
- Real-time presence indicators (online/ offline/away).
- File uploads and sharing within conversions.
- Push notifications for offline users.
- Support for multiple “Workspaces” per user.

#### Non-functional Requirements

- Low Latency: Message delivery < 200ms.
- High Availability: 99.99% uptime; global availability.
- Consistency: Strong consistency for message ordering within a channel.
- Durability: Zero message loss once acknowledged by the server.
- Scalability: Handle 50M+ DAU and 1M+ messages/second.

#### Scale Estimation

- DAU: 50M. Avg messages/user/day: 50.
- Write QPS: (50M * 50) / 86400 ≈ 30k QPS (Peak 60k).
- Read QPS (Fanout): If avg channel size is 50, Read QPS ≈ 1.5M.
- Storage: 2.5B messages/day. At 200 bytes/message ≈ 500GB/ day ≈ 180TB/year.

#### API Design

- POST /v1/messages: Sends a message; body {channel_id, text, mentions}.
- GET /v1/channels/{id}/messages: Fetches paginated history; params {limit, last_msg_id}.
- POST /v1/presence/status: Updates user status; body {status, emoji}.
- GET /v1/workspaces/ {id}/channels: Lists joined channels.

#### Database Design

- Users Table (Postgres): user_id, email, password_hash, status.
- Channels Table (Postgres): channel_id, workspace_id, type (public/private), creator_id.
- Messages Table (Cassandra): Partition Key: channel_id, Clustering Key: message_id (TimeUUID). Fields: sender_id, body, attachments, metadata.
- Presence (Redis): Key: user_id, Value: status_timestamp.

#### High-Level Architecture

Users connect via WebSockets to a Chat Service. The Chat Service persists messages to a distributed NoSQL store and broadcasts them to active participants via a Pub/Sub layer. A separate Presence Service tracks heartbeats, while an Indexing Service pipes messages to Elasticsearch for search.

#### Detailed Components

- Gateway Service: Manages persistent WebSocket connections and session authentication.
- Message Service: Validates messages and writes them to the primary database.
- Fanout Service: Distributes messages to online subscribers via Redis Pub/Sub.
- Presence Service: Tracks user liveness using periodic heartbeats (TTL in Redis).
- Push Service: Handles mobile/desktop notifications for inactive users via APNs/FCM.
- Search Service: Consumer that ingests Kafka events into Elasticsearch.

#### Data Flow

1. Write Path: User A sends message → WebSocket → Gateway → Message Service → Cassandra Write → Message Queue (Kafka)
2. Read Path: Kafka Consumer → Fan-out Service → Gateway (WebSocket) → User B (Live). User C (Offline) pulls history via REST API on login

#### Caching Strategy

- Edge: CDN for static assets (icons, UI).
- Application: Redis cluster caching the last 100-200 messages per active channel to avoid DB hits.
- Presence: Distributed Redis hash maps for volatile user status.

#### Database Choice

Cassandra is the primary message store because it offers high write throughput and scales linearly. The partition key channel_id ensures that all messages for a single channel are stored together, allowing for efficient range scans by message_id (TimeUUID) for history loading. Postgres is used for relational metadata (users, workspace memberships) where ACID is critical.

#### Message Queue Usage

Kafka acts as the central ingestion point for all “Message Sent” events. This decouples storage from downstream tasks like search indexing, push notifications, and analytics.

#### Storage Strategy

Message text is stored in Cassandra. Large files/images are stored in S3 with metadata (URL, size) stored in the messages table.

#### Security

TLS for all traffic. OAuth2/JWT for API auth. Workspace-level isolation; message access checked against channel membership lists stored in a distributed cache.

#### Scaling Strategy

Shard WebSocket gateways by user_id. Use consistent hashing for Fan-out services. Partition Cassandra by channel_id. Deploy services across multiple regions with global LB.

#### Bottlenecks

- Big Channels: A channel with 100k users creates a massive fanout storm (Mitigation: limit active WebSocket broadcasts, use lazy loading).
- Hot Partitions: Extremely active channels (Mitigation: further shard channel_id by time buckets).
- WebSocket State: Sustaining 50M connections (Mitigation: distributed session store, small heartbeat intervals).

#### Failure Handling

- Gateway Fail: Client reconnects to a different node via LB; history gap is filled by polling.
- Cassandra Node Down: Replicated writes (Quorum) ensure no data loss.
- Kafka Lag: Fan-out remains real-time; search/analytics catch up eventually.

#### Trade-offs

- Consistency vs Availability: Chose Eventual Consistency for message ordering across devices to maintain high availability during network partitions.
- WebSocket vs Long Polling: WebSocket for bidirectional lower latency at the cost of higher server memory state.
- NoSQL vs SQL: NoSQL for scale over SQL join capabilities for messages.

#### Alternative Approaches

- XMPP Protocol: Robust but complex to scale and customize for modern Slack features.
- SQL Sharding: Difficult to manage re-balancing as message volume grows compared to Cassandra’s native scaling.

#### Optimization Tips

- Use Protobuf instead of JSON for WebSocket messages to save bandwidth.
- Implement “Read Receipts” as a background batch write to reduce QPS.
- Client-side optimistic UI updates.

#### Interview Tips

- Focus on the “Fan-out” problem; specifically how to handle a channel with 50,000 members.
- Explain the difference between persistent (WebSocket) and stateless (REST) paths.
- Mention how to handle message delivery across multiple devices for the same user.

#### Final Architecture Diagram

```text
[Client] <--> [LB / API Gateway]
|
+-----------+-----------+
| | |
[WebSockets] [Rest API] [Auth/Presence]
| | |
[Pub/Sub] <--> [Services] <--> [Redis Cache]
| | |
[Kafka] ----> [DB: Cassandra] [Search: ES]
```

#### Complexity Discussion

- Time: Writing a message is O(1) after connection setup. Fetching history is O(K) where K is the number of messages.
- Space: O(N) where N is total messages stored.
- Network: O(M) where M is the number of recipients in a channel for fan-out.

#### Example API Request

```http
POST /v1/messages

{
  "channel_id": "CH12345",
  "text": "Hello Team!",
  "device_id": "mobile_001",
  "nonce": "unique_id_for_dedup"
}
```

#### Example API Response

```json
{
  "message_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "sent",
  "timestamp": 1672531200
}
```

### Q22. Design Zoom — Medium

#### Problem Statement

Design a scalable video conferencing system that supports one-on-one and large-group video calls. The system must prioritize low latency (sub-150ms) and handle varying network conditions across global locations.

#### Functional Requirements

- 1:1 and group video/audio calls.
- Screen sharing and real-time chat.
- Meeting scheduling and management.
- Recording and cloud storage of meetings.
- Participant management (mute, kick, hand-raise).

#### Non-functional Requirements

- Ultra-low latency: < 150ms for interactivity.
- Scalability: Support meetings with 1,000+ participants.
- Reliability: 99.9% connection success rate.
- Security: End-to-end encryption (E2EE) support.

#### Scale Estimation

- DAU: 300M. Peak Concurrent Users: 20M.
- Bandwidth: 1:1 call @ 1Mbps. 20M users = ~10-20 Tbps aggregate.
- Metadata Storage: 1B meetings/year ≈ 1TB metadata.

#### API Design

- POST /v1/meetings: Create a meeting; returns meeting_id and token.
- GET /v1/meetings/{id}: Join a meeting; returns Media Server IP.
- POST /v1/meetings/{id}/ record: Start recording.

#### Database Design

- Users Table (MySQL): user_id, name, email, plan_type.
- Meetings Table (MySQL): meeting_id, host_id, start_time, duration, settings_blob.
- Recording Metadata (Postgres): meeting_id, s3_path, size, status.

#### High-Level Architecture

The system uses a Signaling Server (WebSockets) to handle call orchestration and a Media Server (SFU - Selective Forwarding Unit) to route video packets. Clients send their streams to the SFU, which forwards them to other participants without transcoding, minimizing latency. A Global Controller selects the closest SFU to the participants.

#### Detailed Components

- Signaling Service: Manages call setup, SDP exchange, and ICE candidates using WebSockets.
- Media Server (SFU): Receives, replicates, and forwards media packets (UDP/RTP).
- SFU Manager: Monitors SFU load and assigns meetings to specific servers.
- Recording Service: Connects to the SFU as a “silent participant” to transcode and save streams to S3.
- Turn/Stun Servers: Assist in NAT traversal for peer-to-peer or server-mediated connections.

#### Data Flow

1. Call Join: Client → Signaling Service → Retrieve Meeting Info → Assign SFU via SFU Manager
2. Media Stream: Client A → UDP/RTP → Media Server (SFU) → Forwarding to Client B/C/D

#### Caching Strategy

- Meeting Info: Cache meeting settings and participant lists in Redis to speed up join times.
- SFU Health: Real-time state of SFU capacity cached in Redis.

#### Database Choice

MySQL/Postgres for meeting and user metadata due to the relational nature and need for ACID transactions during scheduling. Distributed NoSQL isn’t strictly necessary here as the “hot” data is the media stream, which bypasses the database entirely.

#### Message Queue Usage

Kafka is used to process “Meeting Ended” events to trigger post-processing tasks like generating attendance reports or moving temporary recordings to permanent S3 storage.

#### Storage Strategy

Video recordings are stored as raw segments in S3. After the meeting, a background job stitches segments into a MP4 and moves them to long-term “Glacier” storage for cost efficiency.

#### Security

AES-256 encryption. End-to-End Encryption (E2EE) where the keys are never known to the SFU. Meeting passwords and “Waiting Room” logic via the Signaling Service.

#### Scaling Strategy

Deploy Media Servers (SFUs) in 50+ global POPs (Points of Presence). Use Anycast IP to route users to the nearest Signaling gateway. Horizontal scaling of SFUs is key; if a meeting grows too large, use a “Cascading SFU” tree.

#### Bottlenecks

- High Fanout: 1,000 people in one meeting (Mitigation: Use “active talker” logic where only the top 5 speakers’ video is sent).
- Network Jitter: (Mitigation: Jitter buffers on the client side and Adaptive Bitrate Streaming).
- Signaling Overload: (Mitigation: Shard signaling servers by meeting_id).

#### Failure Handling

- SFU Crashes: The Signaling Service detects the disconnect and forces clients to a new SFU.
- Network Partition: Clients attempt to reconnect; the SFU holds the session state for 30 seconds.

#### Trade-offs

- SFU vs MCU: SFU (Forwarding) is chosen over MCU (Mixing) because SFU has lower latency and is cheaper, though it requires more client-side CPU.
- UDP vs TCP: UDP is prioritized for media to avoid the “Head-of-line blocking” of TCP.

#### Alternative Approaches

- PeertoPeer (WebRTC): Rejected for large groups as client upstream bandwidth is the bottleneck.
- MCU (Multipoint Control Unit): Rejected due to high server-side CPU costs for transcoding every stream.

#### Optimization Tips

- Simulcast: Client sends 3 resolutions (high, med, low) and SFU forwards the best one based on recipient bandwidth.
- Audio-only mode for lowbandwidth users.
- Hardware acceleration (H.264/H.265) on client devices.

#### Interview Tips

- Differentiate between the Control Plane (Signaling) and Data Plane (Media).
- Explain why UDP is better than TCP for video calls.
- Discuss how you handle 1000 users in a single room (The “Active Speaker” pattern).

#### Final Architecture Diagram

```text
[Client A] [Client B]
\ /
[Signaling (WS)] -- [Meeting DB]
|
[SFU Manager] -- [SFU Cluster (UDP)]
| / \
[Recording Svc] --> [S3] [Client C]
```

#### Complexity Discussion

- Time: Media forwarding is O(1) latency per packet.
- Space: Minimal on server (transient buffers); storage O(Duration
- Bitrate).
- Network: O(N^2) potentially, but SFU reduces this to O(N) by selective forwarding.

#### Example API Request

```http
POST /v1/meetings

{
  "user_id": "u123",
  "topic": "System Design Session",
  "type": "scheduled",
  "start_time": "2023-10-27T10:00:00Z"
}
```

#### Example API Response

```json
{
  "meeting_id": "888-999-111",
  "join_url": "https://zoom.us/j/888999111",
  "token": "jwt_access_token"
}
```

### Q23. Design Gmail — Medium

#### Problem Statement

Design a massive-scale email service capable of handling petabytes of data, providing nearly instantaneous search, and ensuring that no email is ever lost.

#### Functional Requirements

- Sending and receiving emails (SMTP/IMAP/POP3).
- Email categorization/folders (Inbox, Sent, Drafts, Spam).
- Full-text search across all emails.
- Large attachments support.
- Spam filtering and malware detection.

#### Non-functional Requirements

- Durability: 99.9999999% (no data loss).
- Availability: 99.99%.
- Consistency: Eventual consistency is okay for search, but strong consistency for folder views.
- Scalability: Support for billions of users and trillions of emails.

#### Scale Estimation

- DAU: 1B. Emails sent/received: 10/day.
- Write QPS: 10B/86400 ≈ 115k QPS.
- Storage: 10B emails * 50KB avg ≈ 500TB/day ≈ 180PB/year.

#### API Design

- GET /v1/folders: List all folders.
- GET /v1/messages?folder=inbox: List emails with pagination.
- POST /v1/messages/send: Send an email (body, recipients, attachments).
- GET /v1/messages/search?q=query: Full-text search.

#### Database Design

- Metadata Store (Spanner/Postgres Sharded): user_id, msg_id, folder_id, is_read, snippet, timestamp.
- Blob Store (Bigtable/Cassandra): msg_id, raw_content, headers.
- Attachment Store (S3/GCS): attachment_id, message_id, blob_url.

#### High-Level Architecture

Incoming emails arrive via SMTP workers and are placed in a Queue. An Orchesrator picks them up, saves the raw body to a NoSQL Blob store, updates the Metadata DB, and triggers the Search Indexer. Outbound emails are sent through an SMTP relay after spam checks.

#### Detailed Components

- SMTP Gateway: Handles the mail transfer protocol for external communication.
- Mail Submission Service: Internal API for composing/ sending emails.
- Metadata Service: Manages folder structures and email statuses (read/unread).
- Search Service: Uses inverted indices (Elasticsearch/Lucene) sharded by user_id.
- Filter Service: Runs ML models for spam and “Priority Inbox” classification.
- Attachment Service: Manages uploads/downloads from object storage.

#### Data Flow

1. Receive: External SMTP → SMTP Gateway → Filter Service (Spam Check) → Queue → Storage Service (Bigtable + Spanner) → Indexer
2. Send: Client → API → Submission Service → Storage Service → SMTP Relay → Destination Server

#### Caching Strategy

- Hot Metadata: Redis cache for the first page of the “Inbox” folder per user.
- User Profiles: Cache user settings and anti-spam preferences.

#### Database Choice

Google Bigtable (or Cassandra/HBase) is ideal for the email body because it handles massive sparse data and high write loads. Spanner (or sharded SQL) is used for metadata because it provides ACID transactions which are crucial for actions like “Move to Folder” or “Mark as Read” across multiple devices.

#### Message Queue Usage

Kafka is used to decouple the receipt of an email from the heavy processing (Spam check, indexing, notification delivery).

#### Storage Strategy

Email bodies are stored in Bigtable. Attachments > 1MB are stored in S3/GCS. Metadata is kept in a geo-distributed relational store.

#### Security

TLS for transport (STARTTLS). S/MIME or PGP support. Data at rest is encrypted per-user. Rate limiting on the SMTP gateway to prevent DDoS/Spam botnets.

#### Scaling Strategy

Shard all data by user_id. This ensures that a single user’s data is co-located, making operations like “Empty Trash” efficient. Use a “Search-per-user” architecture to prevent global index hotspots.

#### Bottlenecks

- Massive Search Indices: (Mitigation: Use a tiered search index where recent emails are in SSD-backed memory and older emails are on disk).
- Spam Waves: (Mitigation: Implement global reputation scores for IP addresses).
- Storage Cost: (Mitigation: Data compression and deduplication of attachments across users).

#### Failure Handling

- SMTP Failure: Senders will retry (standard SMTP behavior).
- Storage Failure: Use 3x replication across regions.
- Indexer Delay: Search results might be slightly stale, but the email is still accessible via the Inbox folder.

#### Trade-offs

- Search Freshness vs Throughput: Indexing is asynchronous to ensure fast email delivery, accepting a few seconds of search lag.
- Storage Cost vs Availability: Multiple replicas increase costs but are necessary for durability.

#### Alternative Approaches

- Pure SQL: Rejected because managing petabytes of text in a traditional SQL DB is a scaling nightmare.
- Single Search Index: Rejected because search performance degrades with trillion-entry indices; user-sharding is superior.

#### Optimization Tips

- Deduplication: Store only one copy of an attachment sent to 100 people in the same company.
- Protocol Buffers: Highly efficient serialization for internal service communication.
- Prefetching: Predict when a user will open an email and pre-fetch the body.

#### Interview Tips

- Focus on how users are sharded and why user_id is the natural partition key.
- Mention the complexity of the “Search” feature across billions of docs.
- Discuss the lifecycle of an attachment.

#### Final Architecture Diagram

```text
[Sender] -> [SMTP GW] -> [Kafka]
|
+----------------+----------------+
| | |
[Filter Svc] [Storage Svc] [Search Indexer]
| / \ |
[Spam DB] [Bigtable] [Spanner] [Elasticsearch]
```

#### Complexity Discussion

- Time: Metadata lookup O(1). Search O(log N) per user shard.
- Space: O(Total Mail Volume).
- Network: Heavy during attachment transfers; minimized via regional edge points.

#### Example API Request

```http
GET /v1/messages?folder=inbox&limit=20
Authorization: Bearer <token>
```

#### Example API Response

```json
{
  "messages": [
    {
      "id": "msg1",
      "subject": "Hello",
      "from": "bob@abc.com",
      "snippet": "Hey, did you see..."
    },
    {
      "id": "msg2",
      "subject": "Receipt",
      "from": "store@shop.com",
      "snippet": "Your order..."
    }
  ],
  "next_page_token": "abc123xyz"
}
```

### Q24. Design Google Drive / Dropbox — Medium

#### Problem Statement

Design a file synchronization and storage service that allows users to upload, download, and share files across multiple devices while ensuring data integrity and versioning.

#### Functional Requirements

- File upload/download/update.
- Automatic file synchronization across devices.
- File versioning and “trash” recovery.
- Sharing files/ folders with specific users or via public links.
- Offline access and delta updates (only sync changed chunks).

#### Non-functional Requirements

- Durability: Extremely high (11 nines).
- Consistency: Strong consistency for a single user (no stale file versions).
- Availability: 99.99%.
- Scalability: Support for billions of files.

#### Scale Estimation

- DAU: 100M. Avg storage/user: 10GB.
- Total Storage: 1EB (Exabyte).
- Upload QPS: 10k QPS (Peak).
- Bandwidth: Heavy egress/ingress.

#### API Design

- POST /v1/files/upload: Upload file (multipart).
- GET /v1/files/{id}: Download file.
- GET /v1/files/{id}/versions: List file history.
- POST /v1/files/share: Create share link.

#### Database Design

- Metadata Store (SQL MySQL/Postgres): file_id, owner_id, parent_folder_id, file_name, is_directory, latest_version_id.
- File Versions (SQL): version_id, file_id, chunk_list_hash, size, timestamp.
- Chunk Store (NoSQL Dy namoDB): chunk_hash, s3_link. This allows for deduplication.

#### High-Level Architecture

The system consists of a Block Service that breaks files into chunks (e.g., 4MB). Only modified chunks are uploaded (delta sync). A Metadata Service tracks the file tree and versions. A Notification Service (Long Polling/WebSockets) notifies other devices when a change occurs.

#### Detailed Components

- Block Service: Handles chunking, hashing (MD5/ SHA256), and deduplication.
- Metadata Service: Manages the namespace (folders/ filenames) and versioning.
- Synchronization Service: Resolves conflicts and determines which chunks the client lacks.
- Storage Service: Interface for S3/Object storage.
- Notification Service: Pushes updates to clients to trigger downloads.

#### Data Flow

1. Upload: Client chunks file → Hashes chunks → Checks if chunks exist (dedup) → Uploads new chunks to S3 → Updates Metadata DB → Notify other devices
2. Sync: Client receives notification → Asks Metadata Svc for latest version → Compares local chunks with remote hashes → Downloads only missing chunks

#### Caching Strategy

- Metadata Cache: Redis for frequently accessed folder structures.
- Chunk Lookup: Cache the existence of common chunk hashes to avoid DB hits during upload.

#### Database Choice

MySQL or Postgres with sharding (by user_id) for Metadata. Relational DBs are preferred because of the hierarchical nature of folders and the need for ACID transactions when renaming/moving files.

#### Message Queue Usage

Kafka is used to trigger asynchronous tasks: generating thumbnails, virus scanning, and updating search indices.

#### Storage Strategy

Use Object Storage (S3/Azure Blob) for chunks. Chunks are immutable. When a file is modified, a new version is created pointing to a new set of chunk hashes.

#### Security

Client-side encryption or server-side encryption (SSE). Signed URLs for direct S3 downloads. Fine-grained Access Control Lists (ACLs) for sharing.

#### Scaling Strategy

Shard metadata by namespace_id or user_id. Use a Global Content Delivery Network (CDN) for frequent downloads. Scale Block Services horizontally to handle high-bandwidth traffic.

#### Bottlenecks

- Heavy Sync Traffic: (Mitigation: Delta sync and chunking).
- Metadata Growth: (Mitigation: Sharding and archiving old versions).
- Conflict Resolution: (Mitigation: Version vectors or “Last write wins” with user prompting).

#### Failure Handling

- Upload Interrupt: Resumable uploads using chunk offsets.
- S3 Regional Failure: Multi-region replication of object data.

#### Trade-offs

- Chunk Size: Small chunks increase metadata overhead; large chunks increase sync bandwidth. 4MB is a common middle ground.
- Strong vs Eventual Consistency: Chose strong consistency for file metadata to avoid “missing file” errors after an upload.

#### Alternative Approaches

- Storing Files in DB: Rejected; DBs are not optimized for large binary blobs.
- Full File Sync: Rejected; too slow for large files with minor changes.

#### Optimization Tips

- Deduplication: Check hashes across all users to save massive storage space.
- Compression: Compress chunks before upload.
- Edge Computing: Move chunk hashing logic to the client to save server CPU.

#### Interview Tips

- Clearly explain the chunking and deduplication process.
- Discuss how you handle file conflicts (two people editing at once).
- Mention the Notification Service for keeping multiple devices in sync.

#### Final Architecture Diagram

```text
[Client] -> [Load Balancer]
/ \
[Block Svc] [Metadata Svc]
| |
[S3 Chunks] [SQL Meta DB]
| |
[Redis Cache] [Kafka Queue] -> [Thumbnails/Search]
```

#### Complexity Discussion

- Time: Upload is O(File size / Chunk size). Metadata lookup O(log N).
- Space: O(Total unique chunks).
- Network: Optimized by Delta Sync O(Changed Chunks).

#### Example API Request

```http
POST /v1/files/upload-session

{
  "file_name": "resume.pdf",
  "folder_id": "root",
  "total_size": 2048000
}
```

#### Example API Response

```json
{
  "session_id": "sess_987",
  "upload_url": "https://s3.region.amazonaws.com/uploads/...",
  "status": "ready"
}
```

### Q25. Design a Search Engine — Medium

#### Problem Statement

Design a high-level system for a search engine like Google that can crawl the web, index trillions of pages, and provide relevant search results in milliseconds.

#### Functional Requirements

- Crawling: Discover and fetch web pages.
- Indexing: Extract and store data for efficient searching.
- Ranking: Provide the most relevant results for a query (e.g., PageRank).
- Search: User interface for querying and displaying results.

#### Non-functional Requirements

- Scalability: Trillions of pages, billions of queries.
- Latence: Sub-200ms for search queries.
- Freshness: New content indexed within minutes/hours.
- Availability: High availability for the search frontend.

#### Scale Estimation

- Web Pages: 100 Billion+.
- Query Volume: 100k QPS.
- Storage: If 100KB per page → 10PB of raw data.
- Index Size: Generally 10-20% of raw data.

#### API Design

- GET /search?q={query}&page={n}: Returns ranked results.
- POST /internal/ crawl/submit: Add a URL to the crawl queue.

#### Database Design

- Crawl DB (HBase/Cassandra): url_hash, url, last_crawled, status, content_hash.
- Inverted Index (Custom Distributed File System): Mapping word_id -> [list of doc_ids + frequency + positions].
- Document Store (Bigtable): doc_id, title, snippet, body_text.

#### High-Level Architecture

A distributed Crawler fetches pages and stores them in a Document Store. An Indexer reads these documents, creates an Inverted Index, and calculates ranking signals (like PageRank). The Search Service takes user queries, retrieves relevant document IDs from the Index, and returns the ranked snippets.

#### Detailed Components

- URL Frontier: A prioritized queue of URLs to be crawled.
- Crawler: Fleet of workers that fetch HTML, parse for new links, and handle robots.txt.
- Content Extractor: Strips HTML tags and extracts text/metadata.
- Indexer Builder: MapReduce jobs that build the inverted index from the document corpus.
- Ranker: Uses machine learning and link analysis to score documents.
- Query Processor: Handles query expansion, spellcheck, and index retrieval.

#### Data Flow

1. Index Path: URL Frontier → Crawler → Document Store → Indexer Builder (MapReduce) → Inverted Index + Ranking DB
2. Query Path: User Query → Query Processor → Inverted Index Lookup → Ranking/Scoring → Document Store (Snippet retrieval) → Result Page

#### Caching Strategy

- Top Queries: Cache results for the top 1M search terms in Redis.
- Document Metadata: Cache popular page snippets to reduce Bigtable hits.

#### Database Choice

Bigtable is the choice for the document store because it scales to petabytes and supports high-throughput writes from the crawlers. For the Inverted Index, a custom distributed file system (like Google’s SSTables) is used for maximum read performance during queries.

#### Message Queue Usage

A massive distributed queue (Kafka or similar) is used for the URL Frontier to manage crawl priorities and ensure the crawlers are never idle.

#### Storage Strategy

Raw HTML is often compressed and stored in GCS/S3. The processed Inverted Index is distributed across thousands of “Index Servers” in many shards.

#### Security

Filtering of malicious sites/malware. Protection against search result manipulation (SEO spam). Rate limiting for the search API to prevent scraping.

#### Scaling Strategy

Shard the inverted index by word_id. Use a “multi-level index” where the most common words’ lists are sharded across multiple machines. Geographically distribute search servers to reduce RTT.

#### Bottlenecks

- The “The” Problem: Extremely common words have huge posting lists (Mitigation: Stop words and sharding).
- Crawl Politeness: Not overwhelming small sites (Mitigation: Domain-based rate limiting in URL Frontier).
- Freshness: (Mitigation: Priority crawling for news/popular sites).

#### Failure Handling

- Crawler Node Fails: URL is put back in the Frontier.
- Index Server Fails: Shards are replicated (3x); the query is routed to a replica.

#### Trade-offs

- Index Freshness vs Accuracy: Background batch indexing is more efficient but results in slightly stale content.
- Depth vs Breadth of Crawl: Choosing which URLs to ignore to save resources.

#### Alternative Approaches

- RDBMS for Indexing: Rejected immediately; SQL cannot handle the scale or the specific inverted-index schema required.
- Simple Grep: Theoretically possible but takes hours for one query; hence the Inverted Index.

#### Optimization Tips

- Inverted Index Compression: Use Variable Byte Encoding for document IDs to save space.
- Snippet Generation: Store a small summary of the page directly in the index servers.
- Query Expansion: Use synonyms to find more relevant results.

#### Interview Tips

- Explain the Inverted Index clearly; it’s the core of search.
- Discuss PageRank or ranking signals to show you understand quality.
- Address the URL Frontier and how it handles priority/politeness.

#### Final Architecture Diagram

```text
[Crawler] -> [Doc Store] -> [MapReduce Indexing]
^ | | | v v
[Frontier] <- [Parser] [Inverted Index]
|
[User Query] -> [Query Processor] -> [Ranker]
```

#### Complexity Discussion

- Time: Crawling is slow (network bound). Search query is O(Word_freq) or O(1) with caching.
- Space: O(Total unique words
- Total docs).
- Network: Massive bandwidth required for crawling the open web.

#### Example API Request

```http
GET /search?q=system+design+interview&lang=en
```

#### Example API Response

```json
{
  "results": [
    {
      "title": "System Design Primer",
      "url": "https://github.com/...",
      "snippet": "Learn how to design large scale systems..."
    }
  ],
  "total_hits": 1500000
}
```

### Q26. Design a Recommendation Engine (YouTube/Netflix style) — Medium

#### Problem Statement

Create a system that suggests relevant content to users in real-time based on their historical behavior, preferences, and similar user patterns. The goal is to maximize user engagement metrics like CTR (Click-Through Rate) and watch time.

#### Functional Requirements

- Provide a list of “top N” recommended items for a user’s homepage.
- Capture real-time user events (clicks, views, likes).
- Incorporate new content (cold start) into recommendations quickly.
- Filter out already watched or restricted content.
- Support diverse recommendation types (Trending, Based on Recently Watched, Personalized).

#### Non-functional Requirements

- High Availability: Users should always see something, even if stale.
- Low Latency: Recommendations must load in < 200ms.
- Scalability: Support 100M+ users and billions of events/day.
- Consistency: Eventual consistency is acceptable for model updates.
- Diversity/Serendipity: Avoid “filter bubbles.”

#### Scale Estimation

- DAU: 100M.
- Events per user/day: 20 (2B writes/day).
- Write QPS: ~25k (avg), 50k (peak).
- Read QPS: ~10k (homepage loads).
- Storage: 2B events * 500 bytes = 1TB/day. 365TB/year.

#### API Design

- GET /v1/recommendations?userId={id}&limit=20: Returns list of item IDs.
- POST /v1/events: Ingests user behavior (click, view, duration).
- GET /v1/trending: Returns globally popular items.

#### Database Design

- User Profiles (NoSQL/Cassandra): user_id, preferences, watch_history (list), demographic_data.
- Metadata (Postgres): item_id, title, tags, upload_date, category.
- Feature Store (Redis/RedisFeature): Pre-computed embeddings for users and items.

#### High-Level Architecture

User actions are ingested via a Gateway to a Message Queue. A Stream Processor (Flink) updates real-time features, while a Batch Job (Spark) trains deep learning models (Collaborative Filtering/Two-Tower). For serving, a Candidate Generation phase narrows millions of items to hundreds, followed by a Ranking phase using a heavy model (DeepFM) to sort them, and a Re-ranking phase for business logic (diversity/ads).

#### Detailed Components

- Ingestion Service: High-throughput API to collect user signals and push to Kafka.
- Candidate Generator: Uses Approximate Nearest Neighbor (ANN) search like Faiss or Milvus on embeddings.
- Ranking Service: Takes candidate set and applies a complex ML model to predict CTR.
- Feature Store: Central repository for low-latency retrieval of state (e.g., “how many videos did user X watch in 10 mins?”).
- Training Pipeline: Offline Spark/Hadoop jobs to recalculate weights based on historical logs.

#### Data Flow

1. Write Path: User clicks video → API Service → Kafka → Flink (Realtime features) & S3 (Batch logs)
2. Read Path: User opens app → Recommendation Service → Candidate Generator (ANN) → Ranker (ML Scoring) → Filter (Business Logic) → Return list

#### Caching Strategy

- L1 Cache (Redis): Pre-computed “Standard” recommendations for generic users.
- L2 Cache (InMemory): Recently fetched item metadata.
- CDN: Thumbnails and video fragments.

#### Database Choice

Cassandra is ideal for storing massive, high-write user activity logs due to its LSM-tree structure. Metadata belongs in Postgres for relational integrity and complex querying by admin tools.

#### Message Queue Usage

Kafka acts as the backbone for decoupling ingestion from both real-time feature engineering (Flink) and long-term storage (S3/HDFS).

#### Storage Strategy

Raw event logs are stored in S3 in Parquet format for costeffective batch processing via Spark. Model weights/artifacts are stored in an Object Store with versioning.

#### Security

OAuth2 for API access; PII masking in the training pipeline; Rate limiting on the event ingestion endpoint to prevent bot-skewed models.

#### Scaling Strategy

Scale the Candidate Generator horizontally; use sharded Redis for the Feature Store; use GPU clusters for the Ranking inference service.

#### Bottlenecks

- Cold Start: Solved by metadata-based (content-based) filtering for new items.
- Feature Staleness: Mitigated by Lambda Architecture (combining Batch + Stream).
- Model Serving Latency: Mitigated by model quantization and caching candidate sets.

#### Failure Handling

If the Ranking Service fails, fall back to the Candidate Generator’s raw list. If ANN fails, fall back to “Trending Now” from Redis.

#### Trade-offs

- Accuracy vs. Latency: Chose two-stage (Generator + Ranker) instead of one big model.
- Batch vs. Online: Chose hybrid to balance resource cost and freshness.
- Exploration vs. Exploitation: Use Epsilon-greedy strategies to show new content.

#### Alternative Approaches

- Pure Batch: (Rejected) Too slow to react to “viral” trends.
- Pure Rulebased: (Rejected) Cannot scale to millions of users or capture subtle patterns.

#### Optimization Tips

- Use Approximate Nearest Neighbors (HNSW algorithm) instead of exact dot product.
- Compress embeddings to reduce Feature Store memory footprint.
- Implement “negative sampling” during training to improve model robustness.

#### Interview Tips

- Mention the “Two-Tower” architecture (User & Item towers).
- Distinguish between Retrieval (Candidate Generation) and Ranking.
- Focus on how you handle the “Feedback Loop” (users clicking what you recommend).

#### Common Mistakes

- Trying to rank all items in the database for every request (impossible latency).
- Ignoring the “Cold Start” problem for new users/items.
- Neglecting business logic (e.g., don’t show NSFW content).

#### Final Architecture Diagram

```text
Client -> Load Balancer -> Reco Service
|--> Candidate Generator (Faiss/ANN)
|--> Feature Store (Redis)
|--> Ranking Service (TF Serving/GPU)
|--> Filtering (Post-processing)
Ingestion -> Kafka -> Flink/Spark -> S3/HDFS
```

#### Complexity Discussion

- Time: Retrieval is O(log N) with ANN; Ranking is O(K) where K is candidate size (~200-500).
- Space: O(U + I) where U is users and I is items (for embeddings).

#### Example API Request

```http
GET /v1/recommendations?userId=u123&context=homepage
```

#### Example API Response

```json
{
  "recommendations": [
    {
      "id": "vid_99",
      "score": 0.98,
      "type": "personalized"
    },
    {
      "id": "vid_42",
      "score": 0.85,
      "type": "trending"
    }
  ]
}
```

### Q27. Design a CDN — Medium

#### Problem Statement

Design a Content Delivery Network (CDN) to serve static and dynamic content (images, JS, CSS, video) to users with ultra-low latency by caching data at geographically distributed “Edge” locations.

#### Functional Requirements

- Cache static content from origin servers.
- Route users to the nearest Edge POP (Point of Presence).
- Handle cache invalidation (Purge).
- Support “Push” and “Pull” caching models.
- Provide analytics/logging for content owners.

#### Non-functional Requirements

- Ultra-low latency (< 30ms for cached hits).
- High Availability (99.999%).
- Massive Throughput (Tbps scale).
- Security (DDoS protection, SSL termination).

#### Scale Estimation

- 10,000 Edge nodes in 500 locations.
* 100M Requests Per Second (RPS) globally.
* 10 PB of total cached data.
- Bandwidth: 100 Tbps.

#### API Design

- PURGE /assets/{path}: Removes content from cache.
- PUT /assets/{path}: Push model; upload content directly to CDN.
- GET /stats?assetId={id}: Fetch usage metrics.

#### Database Design

- Edge Metadata (SQLite/RocksDB at Edge): Stores local file metadata and TTLs.
- Global Config (Postgres/CockroachDB): Customer settings, origin mappings, and billing.
- Logs (ClickHouse): High-velocity request logs for analytics.

#### High-Level Architecture

The system consists of an Origin Server (source of truth), a Global Traffic Manager (DNS-based or Anycast routing), and Edge POPs. Each POP contains an L7 Load Balancer (Nginx/Envoy), a Cache Engine (Varnish/ Custom), and local SSD storage.

#### Detailed Components

- DNS/Anycast Router: Resolves the CDN domain to the IP of the closest POP using BGP Anycast.
- Reverse Proxy (Edge): Handles SSL/TLS and forwards misses to the origin.
- Cache Store: High-performance storage (NVMe SSDs + RAM) using Consistent Hashing for internal distribution.
- Invalidation Service: A tree-based broadcast system to propagate “purge” commands globally in seconds.
- Log Aggregator: Streams logs from Edge to a central cluster for billing.

#### Data Flow

1. Read Path (Miss): User requests cdn.com/f.jpg → Closest POP → Cache Miss → POP fetches from Origin → POP stores locally → POP returns to user
2. Read Path (Hit): User requests cdn.com/f.jpg → Closest POP → Cache Hit → POP returns data directly

#### Caching Strategy

- Hierarchical Caching: Edge POPs fetch from “Regional Shields” instead of the Origin to reduce Origin load.
- TTL: Respect Cache-Control headers.
- Consistent Hashing: Distributes files across multiple servers within a single POP.

#### Database Choice

ClickHouse is used for logs because it handles massive insert rates and complex analytical queries (OLAP) required for customer dashboards and billing.

#### Message Queue Usage

Kafka is used to stream access logs from all Edge nodes to the central analytics and security (WAF) engines.

#### Storage Strategy

Tiered storage within Edge: RAM (hot), NVMe (warm), SATA SSD (cool). Least Recently Used (LRU) is the primary eviction policy.

#### Security

DDoS mitigation via Anycast (spreading the load); WAF (Web Application Firewall) at the Edge; Token-based authentication (Signed URLs) for private content.

#### Scaling Strategy

Use BGP Anycast to add more POPs without changing DNS; scale POPs horizontally by adding more commodity servers.

#### Bottlenecks

- Origin Overload: Mitigated by “Request Collapsing” (only one request to origin for the same asset).
- Purge Latency: Mitigated by a high-priority pub-sub backbone.
- Cold Cache: Solve using “Pre-warming” for known traffic spikes (e.g., game releases).

#### Failure Handling

If an Edge POP fails, BGP automatically reroutes traffic to the next closest POP. If the Origin fails, the CDN serves stale content (if configured).

#### Trade-offs

- Consistency vs. Performance: Opted for eventual consistency for purges to maintain global speed.
- DNS Routing vs. Anycast: Anycast is faster but harder to implement/debug than Geo-DNS.

#### Alternative Approaches

- Centralized Proxy: (Rejected) Too much latency for global users.
- PeertoPeer CDN: (Rejected) Unreliable performance and high complexity.

#### Optimization Tips

- Use TCP BBR for faster congestion control.
- Support HTTP/3 (QUIC) to reduce handshake latency over lossy networks.
- Implement “Byte Range Requests” for large video files.

#### Interview Tips

- Explain the difference between Push vs. Pull models.
- Discuss how DNS resolution works in the context of CDNs.
- Highlight “Request Collapsing” as a pro-level optimization.

#### Common Mistakes

- Forgetting cache invalidation (it’s the hardest part).
- Ignoring SSL/TLS overhead at the Edge.
- Not explaining how the “closest” node is actually found.

#### Final Architecture Diagram

```text
User -> [Anycast IP] -> Edge POP (L7 LB) -> Cache (SSD)
| (Miss)
Regional Shield -> Origin (S3/Server)
```

#### Complexity Discussion

- Time: O(1) for cache hits at Edge.
- Space: O(Total Cached Content) across global nodes.
- Network: O(1) hop to Edge via Anycast.

#### Example API Request

```json
PURGE /static/hero.png HTTP/1.1 Host: api.cdn-provider.com Authorization: Bearer <token>
```

#### Example API Response

```json
{
  "status": "accepted",
  "purge_id": "p123",
  "estimated_completion_ms": 2000
}
```

### Q28. Design DNS — Medium

#### Problem Statement

Design a global Domain Name System (DNS) that translates human-readable domain names (e.g., google.com) into IP addresses. It must be highly resilient, distributed, and fast.

#### Functional Requirements

- Resolve a domain name to an IPv4/IPv6 address.
- Support various record types (A, AAAA, CNAME, MX, TXT).
- Update/Upsert records (for domain owners).
- Propagate updates globally.
- Provide caching at multiple levels.

#### Non-functional Requirements

- Availability: 100% (DNS failing breaks the internet).
- Latency: Sub-10ms for cached queries.
- Scalability: Handle trillions of queries per day.
- Integrity: Prevent DNS spoofing/poisoning.

#### Scale Estimation

- Queries: 10M per second globally.
- Domains: 350M+ registered domains.
- Storage: ~500 bytes per record. 350M * 500 = 175 GB (small enough for memory).

#### API Design

- resolve(domain_name, type): Internal logic for DNS query.
- POST /v1/ records: Add a new record (Authoritative API).
- PUT /v1/records/{id}: Update existing record.

#### Database Design

- Master DB (Relational Postgres): Stores the source of truth for records.
- Edge Store (Key-Value LMDB/Custom): Read-optimized, memory-mapped storage at the nameserver.

#### High-Level Architecture

The hierarchy consists of Recursive Resolvers (the “searchers”), Root Nameservers, TLD Nameservers (.com, .net), and Authori tative Nameservers (the owners). We focus on building the Authoritative Name server and the Recursive Resolver.

#### Detailed Components

- Recursive Resolver: Performs the iterative search for the user.
- Authoritative Server: Hosts the actual records for a zone.
- Zone Trans fer Service: Uses protocols like AXFR/IXFR to sync data from Master to Slaves.
- Central Registry API: Where users manage their domains.
- DDoS Scrubber: Infront of servers to filter malicious traffic.

#### Data Flow

1. Resolution: User → Resolver → Check Cache. If miss: Resolver - > Root (get TLD IP) → TLD (get Authoritative IP) → Authoritative (get A record) → Cache result → Return to User
2. Update: Admin → API Service → Master DB → Trigger Zone Update → Slave Nameservers pull new data

#### Caching Strategy

- TTLbased: Records have a “Time to Live.” After TTL, the resolver must re-query.
- Negative Caching: Cache “Not Found” responses to prevent repeated NXDOMAIN lookups.

#### Database Choice

Postgres for the Management API due to ACID requirements for domain ownership. For the actual serving path, a specialized memory-mapped KV store (LMDB) is used for maximum read performance.

#### Message Queue Usage

Kafka/RabbitMQ notifies edge nodes that a zone update is available, triggering a pull-based synchronization.

#### Storage Strategy

Records are stored in “Zone Files” in the Master DB and distributed as binary blobs to edge nameservers.

#### Security

DNSSEC (Digital signatures to prevent spoofing); Rate limiting by client IP; Anycast to absorb Volumetric DDoS attacks.

#### Scaling Strategy

BGP Anycast is the primary scaling mechanism, allowing thousands of servers to share one IP address.

#### Bottlenecks

- Propagation Delay: Solved by low TTLs and proactive pushing of updates.
- Large Zone Transfers: Mitigated by Incremental Zone Transfers (IXFR).
- DDoS: Mitigated by geographical distribution and hardware scrubbers.

#### Failure Handling

Authoritative servers are deployed in “Sets” (ns1, ns2). If ns1 is down, the resolver automatically tries ns2.

#### Trade-offs

- Consistency vs. Availability: DNS prioritizes Availability (eventual consistency of records).
- UDP vs. TCP: UDP for speed (small packets), TCP for large responses (DNSSEC) and zone transfers.

#### Alternative Approaches

- Centralized Global DB: (Rejected) Impossible latency and single point of failure.
- Blockchainbased DNS: (Rejected) Current tech is too slow for 10M QPS resolution.

#### Optimization Tips

- Use EDNS0 for larger packet sizes over UDP.
- Implement “Prefetching” for popular domains before TTL expires.
- Keep the entire hot-set of records in RAM.

#### Interview Tips

- Draw the hierarchy clearly: Root -> TLD -> Authoritative.
- Explain “Iterative” vs “Recursive” queries.
- Mention Anycast as it’s the “secret sauce” of DNS scale.

#### Common Mistakes

- Ignoring how updates propagate (Zone Transfers).
- Thinking DNS only uses TCP.
- Not mentioning the TTL concept.

#### Final Architecture Diagram

```text
Client -> [Recursive Resolver]
| (Iterative Search)
|--> Root Server |--> TLD Server (.com)
|--> Authoritative Server (A record)
Management API -> Master DB -> Slaves
```

#### Complexity Discussion

- Time: O(1) for cached; O(Hierarchy Depth) for recursive search (usually 3-4 hops).
- Space: O(Total Records).

#### Example API Request

```json
dig @8.8.8.8 google.com A
```

#### Example API Response

```json
google.com. 300 IN A 142.250.190.46
```

### Q29. Design a Hotel Booking System (Booking.com) — Medium

#### Problem Statement

Design a system to browse hotels, check availability for specific dates, and book rooms. It must handle high concurrency during peak seasons and prevent overbooking.

#### Functional Requirements

- Search for hotels by city/date/guests.
- Show hotel details and room availability.
- Book a room and process payment.
- Manage hotel inventory (for owners).
- Support booking cancellations.

#### Non-functional Requirements

- Strong Consistency for bookings (no overbooking).
- High Availability for searching.
- Low Latency for search results.
- Scalability for seasonal spikes.

#### Scale Estimation

- 1M hotels, 50M rooms.
* 10M DAU.
- Search QPS: 5k.
- Booking QPS: 100.
- Storage: 10M bookings/year (~5GB).

#### API Design

- GET /v1/hotels?city={}&checkin={}&checkout={}: Search hotels.
- GET /v1/ hotels/{id}/rooms: Get available rooms.
- POST /v1/bookings: Reserve a room (Atomic).
- POST /v1/payments/confirm: Finalize booking.

#### Database Design

- Hotel DB (Postgres): hotels, rooms, room_inventory.
- Booking DB (Postgres): bookings (id, user_id, room_id, status, checkin, checkout).
- Search Index (Elasticsearch): Hotel metadata for fast geo-filtering.

#### High-Level Architecture

The system uses a Read/Write separation. Search Service uses Elasticsearch for high-performance filtering. Booking Service uses a RDBMS with ACID transactions to ensure room inventory remains accurate. A Payment Service handles external gateway integration via a state machine.

#### Detailed Components

- Search Service: Queries ES for hotels matching criteria.
- Availability Service: Checks the RDS room_inventory table for specific date ranges.
- Booking Service: Handles the “Reserved -> Paid -> Confirmed” workflow.
- Inven tory Manager: Daily scheduled jobs to initialize inventory for the next 365 days.
- Archive Service: Moves old bookings to cold storage (S3).

#### Data Flow

1. Search: User → Search Service → Elasticsearch → Return Hotel List
2. Booking: User → Booking Service → Start DB Transaction → Check inventory (SELECT FOR UPDATE) → Insert booking → Commit → Call Payment Service

#### Caching Strategy

- Redis: Cache hotel metadata and static descriptions.
- Don’t Cache: Final inventory/availability (consistency risk).
- CDN: Hotel images and maps.

#### Database Choice

Relational DB (Postgres) is mandatory for the booking/inventory logic. We need SERIALIZABLE isolation or pessimistic locking to ensure that if two people click “Book” at the exact same millisecond for the last room, only one succeeds.

#### Message Queue Usage

Kafka is used to sync updates from the Postgres Hotel DB to Elasticsearch. Also used for sending confirmation emails/SMS asynchronously.

#### Storage Strategy

Images are stored in S3 with CloudFront. Booking history > 2 years is moved to BigQuery for analytics.

#### Security

PCI-DSS compliance for payments (use Stripe/Braintree to avoid touching raw cards); HTTPS for all traffic; idempotent booking requests using idempotency_key.

#### Scaling Strategy

Shard the Booking DB by hotel_id or region. Use read-replicas for the Search Service.

#### Bottlenecks

- DB Locking: High contention on popular hotels. Mitigation: Use Redis-based distributed locks for temporary (10-min) holds.
- Search Latency: Complex geo-queries. Mitigation: Use Elasticsearch’s geo-sharding.

#### Failure Handling

Saga Pattern for distributed transactions: if payment fails, the “Compensation” step releases the room reservation in the Booking DB.

#### Trade-offs

- Pessimistic vs Optimistic Locking: Pessimistic is safer for rare resources (luxury suites); Optimistic is better for high-volume standard rooms.
- Search Freshness: ES might be 1s behind Postgres; acceptable for hotel metadata but not final price.

#### Alternative Approaches

- NoSQL (Cassandra): (Rejected) Hard to handle daterange availability queries and ACID transactions.
- Inmemory only inventory: (Rejected) Risk of data loss on crash.

#### Optimization Tips

- Use a “Calendar Table” to store pre-computed room counts per day for the next year.
- Implement “In-progress” booking markers to prevent others from even seeing the room during checkout.

#### Interview Tips

- The “Overbooking” problem is the core of this interview. Focus on DB transactions.
- Mention how to handle time zones for check-in/out.
- Explain the Payment State Machine (Pending, Success, Failed, Refunded).

#### Common Mistakes

- Using NoSQL for inventory (concurrency hell).
- Not handling “Race Conditions” during the booking step.
- Forgetting that a “Room” is actually a “Room Type” with a count.

#### Final Architecture Diagram

```text
User -> Load Balancer -> Search Service -> Elasticsearch
-> Booking Service -> Postgres (Primary)
| -> Kafka -> Analytics
| -> Payment Gateway
```

#### Complexity Discussion

- Time: Search O(log N); Booking O(1) with row-level locks.
- Space: O(Hotels * 365 days) for inventory table.

#### Example API Request

```http
POST /v1/bookings

{
  "hotel_id": "h789",
  "room_type": "deluxe",
  "dates": [
    "2023-12-01",
    "2023-12-05"
  ],
  "idempotency_key": "unique-uuid-123"
}
```

#### Example API Response

```json
{
  "booking_id": "b_555",
  "status": "reserved",
  "expiry": "2023-11-20T10:15:00Z"
}
```

### Q30. Design a Flight Booking System — Medium

#### Problem Statement

Design a global airline reservation system. Users should be able to search for flights (including multi-segment trips), view pricing, and reserve seats. The system must integrate with GDS (Global Distribution Systems) like Amadeus or Sabre.

#### Functional Requirements

- Search flights by origin, destination, and date.
- Support one-way and round-trip.
- Seat selection and reservation.
- Manage flight schedules and pricing (Dynamic Pricing).
- Integration with external GDS for multiairline routes.

#### Non-functional Requirements

- High Consistency: No double-booked seats.
- Latency: Search must aggregate results from multiple airlines efficiently.
- Availability: Search is highly available; Booking is highly consistent.
- Scale: Support massive bursts during holidays.

#### Scale Estimation

- 100k flights/day globally.
* 200 seats/flight = 20M seats/day.
- Search QPS: 10k (many users browse without buying).
- Booking QPS: 500.

#### API Design

- GET /v1/flights/search?from=SFO&to=LHR&date=...: Fetch flights.
- GET /v1/ flights/{id}/seats: Fetch seat map.
- POST /v1/orders: Create a temporary reservation (lock seat).
- POST /v1/orders/{id}/confirm: Finalize after payment.

#### Database Design

- Flight DB (Postgres/CockroachDB): flights, segments, aircrafts.
- Seat Map (Redis + Postgres): Current seat occupancy.
- User/Ticket DB (Cassandra): tickets, passenger_info (high volume, write-heavy).

#### High-Level Architecture

Uses an Aggregator Service to fetch data from local DBs and external GDS APIs. An Inventory Service manages seat locks. Since flight data is often cached, the system uses a Fare Service to calculate dynamic pricing based on demand.

#### Detailed Components

- GDS Adapter: Interfaces with SOAP/REST APIs of Amadeus/Sabre.
- Fare Engine: Rules-based engine for calculating taxes, fuel surcharges, and discounts.
- Seat Map Service: Visual representation of the plane using a JSON-stored layout.
- Booking Orchestrator: Manages the workflow between reserving the seat and issuing the ticket.

#### Data Flow

1. Search: User → Search Aggregator → Query Local DB + Parallel calls to GDS → Merge/Sort results → Cache in Redis → Return
2. Booking: User selects seat → Order Service → DB Transaction: Check seat status → Update seat to “LOCKED” (time-bound) → User pays → DB Update seat to “OCCUPIED” → Generate PNR (Passenger Name Record)

#### Caching Strategy

- Redis: Cache search results for 5-10 minutes (prices change frequently).
- Couchbase: Store session-based seat locks for high-speed availability checks.

#### Database Choice

CockroachDB (Distributed SQL) is excellent here for global scale while maintaining ACID. It handles the “Global” aspect of flights (SFO to LHR) better than a single Postgres instance by keeping data close to the user’s region.

#### Message Queue Usage

Kafka handles the flight status updates (delays, gate changes) which are pushed to the user’s app via WebSockets.

#### Storage Strategy

Transaction logs and PNRs are stored in an immutable ledger or a heavily audited SQL table for legal compliance.

#### Security

TLS 1.3; API Keys for GDS; OAuth2 for users; strict input validation for PNR lookups to prevent “guessing” other people’s trips.

#### Scaling Strategy

Shard flight data by airline_code or flight_number. Use Global Server Load Balancing (GSLB) to route users to the nearest regional data center.

#### Bottlenecks

- External API Latency: GDS calls can be slow. Mitigation: Fan-out pattern with a strict timeout; return partial results.
- Seat Contention: 100 people trying to book the same “cheap” seat. Mitigation: Distributed locking in Redis with a 10-minute TTL.

#### Failure Handling

If the GDS is down, only local flights (our airline) can be booked. If the Payment Service is down, extend the seat-lock TTL to allow the user more time or notify them.

#### Trade-offs

- Stale Search Results: Decided to cache search results to save GDS costs, even if the price might change at checkout.
- Consistency vs. Availability: During a partition, stop accepting bookings for affected flights to prevent overbooking (CP system).

#### Alternative Approaches

- NoSQL for Seats: (Rejected) Impossible to safely guarantee one seat = one passenger without complex application-side logic.
- Syn chronous GDS search: (Rejected) Too slow; user experience suffers.

#### Optimization Tips

- Use Gzipped JSON for large seat map transfers.
- Implement “Long Polling” or WebSockets for the “Searching…” screen to handle slow GDS responses.
- Pre-calculate the most common routes (e.g., NYC to LON) and cache them aggressively.

#### Interview Tips

- Mention PNR (Passenger Name Record) – it shows domain knowledge.
- Explain how to handle “Connecting Flights” (if one segment fails, the whole booking must fail/rollback).
- Discuss how prices change based on how many seats are left (Demand-based pricing).

#### Common Mistakes

- Not considering time-limited seat locks (what if the user never pays?).
- Ignoring the complexity of third-party GDS integrations.
- Failing to handle different time zones for departure/arrival.

#### Final Architecture Diagram

```text
User -> Aggregator -> [Local DB / Redis]
| -> [GDS Adapters] -> [Amadeus/Sabre]
Booking Service -> CockroachDB (Global ACID)
| -> Payment Service
| -> Notification Service (Mail/SMS)
```

#### Complexity Discussion

- Time: Search is O(N) where N is the number of sources; Booking is O(1).
- Space: O(F
- S) where F is flights and S is seats.

#### Example API Request

```http
POST /v1/orders

{
  "flight_id": "AA123",
  "seat": "12A",
  "passengers": [
    {
      "name": "John Doe",
      "passport": "A1234567"
    }
  ]
}
```

#### Example API Response

```json
{
  "order_id": "ord_999",
  "status": "pending_payment",
  "expiry": "2023-11-20T11:00:00Z",
  "total_price": 450.0
}
```

### Q31. Design Uber / Lyft — Hard

#### Problem Statement

Design a real-time ride-hailing marketplace that matches passengers with nearby drivers. The system must track driver locations in real-time, handle trip lifecycle management, and provide ETA calculations under high concurrency.

#### Functional Requirements

- Drivers should be able to broadcast their real-time location and availability.
- Riders should be able to request a ride based on their current location and destination.
- The system must match a rider with an available driver within a 2-5km radius.
- Riders should see real-time location updates of the assigned driver.
- The system must handle trip completion, fare calculation, and rating.

#### Non-functional Requirements

- Low Latency: Location updates and matching must happen in < 500ms.
- High Availability: The system must be available 99.99% of the time; a crash prevents people from moving.
- High Scalability: Support millions of concurrent drivers/riders globally.
- Consistency: Eventual consistency is okay for location, but strict consistency is required for trip matching to avoid double-booking.

#### Scale Estimation

- DAU: 50M.
- Concurrent Drivers: 1M.
- Location Updates: 1 update per 3-5 seconds.
- QPS (Write): ~333k requests/sec (1M / 3s).
- Storage: 50M trips/day * 1KB = 50GB/day. 5-year storage ~90TB.

#### API Design

- POST /v1/locations: Update driver lat/long (Internal Heartbeat).
- POST / v1/rides/request: Rider requests a trip (pickup_lat, pickup_lng, type). returns ride_id.
- GET /v1/rides/{id}/status: Poll or WebSocket for ride status/driver location.
- PATCH /v1/ rides/{id}/accept: Driver accepts a ride request.

#### Database Design

- DriverLocation (Redis/Geohash): driver_id, lat, lng, last_update.
- Trips (Postgres): trip_id, rider_id, driver_id, status (PENDING, ACTIVE, COMPLETED), source_geo, dest_geo, fare.
- Drivers (Postgres): driver_id, status (IDLE, BUSY), vehicle_info.

#### High-Level Architecture

The system uses a Quadtree or Geohash based indexing service to track driver positions. A Match Maker Service listens to ride requests and queries the Location Service for nearby drivers, while a Trip Management Service maintains the state machine of the ride.

#### Detailed Components

- Location Service: Buffers GPS pings in Redis using GEOADD for fast radial queries.
- Matching Engine: Orchestrates the “offer” logic, sending push notifications to drivers in a priority queue.
- Edge Service (WebSockets): Maintains persistent connections for sub-second location streaming to riders.
- Maps/ Routing Engine: Calculates ETA and optimal routes using OSRM or Google Maps API.
- Payment & Billing: Asynchronous worker that processes transactions once the trip state is COMPLETED.

#### Data Flow

1. Write (Location): Driver App → LB → Location Service → Redis (GEOSET) + Kafka (for analytics)
2. Read (Match): Rider App → Request Ride → Match Maker → Query Redis for nearby IDs → Filter drivers by status → Notify Driver via Firebase/APNs

#### Caching Strategy

Use Redis for all “hot” data: driver locations (TTL 30s) and active trip metadata. Geometric tiles for mapping can be cached at the CDN level.

#### Database Choice

Postgres (with PostGIS) for Trip and User records because ACID compliance is vital for financial transactions and ride states. Redis for location indexing because memory-mode GEORADIUS provides the required sub-10ms latency at high scale.

#### Message Queue Usage

Kafka is used to decouple the Trip service from Billing, Analytics, and Notification services. If a driver finishes a trip, a message is published so the ledger is updated without blocking the driver’s next ride.

#### Storage Strategy

Transactional data in Postgres. Historical GPS traces (telemetry) in S3/HDFS/ClickHouse for route optimization and dispute resolution.

#### Security

TLS for all transit. JWT for session management. Rate limiting by IP and UserID. PII masking for phone numbers via Twilio.

#### Scaling Strategy

Shard the Location Service by Geohash regions (e.g., one shard per city or state). Use Read Replicas for the Trip DB.

#### Bottlenecks

1. Hotspots: High demand in a small area (e.g., a stadium). Mitigation: Dynamic geohash precision. 2. Matching Conflicts: Multiple riders matched to one driver. Mitigation: Distributed locking in Redis during the “Accept” phase. 3. DB Write Load: Millions of GPS pings. Mitigation: Batch writes to persistent storage; only store latest in Redis.

#### Failure Handling

If the Matching engine fails, fallback to a greedy regional search. If Redis fails, reload state from the last 30s of Kafka location logs.

#### Trade-offs

- Precision vs. Latency: Updating location every 5s instead of 1s to save battery/bandwidth.
- Consistency vs. Availability: Use Eventual Consistency for location updates to ensure the system remains available.
- Architecture: Quadtree vs. Geohash. Geohash is easier to shard across Redis nodes.

#### Alternative Approaches

1. PeertoPeer: Rejected due to lack of central control over pricing and safety. 2. SQLonly Location Tracking: Rejected because SELECT WHERE distance < X is too slow for millions of updates per second.

#### Optimization Tips

- Use Google S2 Geometry library for highly efficient spherical math.
- Implement “Forward Dispatch” (matching a driver before they finish their current trip) to reduce idle time.
- Use Protobuf for location pings to reduce payload size by 60%.

#### Interview Tips

- Focus heavily on the “Matching” logic and how you handle concurrent “Accepts.”
- Explain how you’d handle the “Thundering Herd” problem when thousands of riders open the app after a concert.

#### Common Mistakes

- Forgetting to handle the “Driver is busy” state in the location index.
- Using a standard relational DB for real-time location updates.
- Not addressing how to update the UI when the driver moves.

#### Final Architecture Diagram

```text
[Rider/Driver Apps]
| (WebSocket/HTTPS)
[Load Balancer]
|
[Gateway Service] ---- [Auth]
|
[Location Service] <---> [Redis (Geohash)]
| |
[Match Engine] <--------- [Kafka] ----> [Analytics]
| |
[Trip Service] <---> [Postgres DB]
|
[Notification Svc] -> [Firebase/APNs]
```

#### Complexity Discussion

- Time: Geohash lookup is O(log N) or O(1) with hash maps.
- Space: Redis storage is O(D) where D is number of active drivers.
- Network: O(U) where U is the frequency of GPS updates.

#### Example API Request

```http
POST /v1/rides/request

{
  "rider_id": "usr_99",
  "origin": {
    "lat": 40.7128,
    "lng": -74.006
  },
  "destination": {
    "lat": 40.7306,
    "lng": -73.9352
  },
  "service_type": "UberX"
}
```

#### Example API Response

```json
{
  "ride_id": "trip_df823",
  "status": "searching",
  "estimated_fare": 25.5,
  "currency": "USD"
}
```

### Q32. Design a Food Delivery System (Swiggy/DoorDash) — Hard

#### Problem Statement

Build a three-sided marketplace (Customer, Restaurant, Delivery Partner) that handles menu browsing, order placement, real-time tracking, and dispatching. The system must synchronize kitchen prep time with courier arrival.

#### Functional Requirements

- Customers can search for restaurants and view menus.
- Customers can place orders and track status in real-time.
- Restaurants can manage menus and accept/reject orders.
- Couriers (Dashers) receive delivery assignments based on location.
- Real-time status updates (Order Placed -> Preparing -> Out for Delivery).

#### Non-functional Requirements

- High Availability: Critical during peak hours (lunch/dinner).
- Consistency: Strict consistency for payments and order status.
- Low Latency: Search and menu browsing must be fast (< 200ms).
- Scalability: Handle huge spikes during holidays or sporting events.

#### Scale Estimation

- 10M DAU.
* 1M Orders/day.
- Peak QPS: 5k (Ordering), 50k (Browsing).
- Storage: 1M orders * 2KB = 2GB/day. 5 years = 3.6TB.

#### API Design

- GET /v1/restaurants?lat=x&lng=y: List nearby restaurants.
- POST /v1/ orders: Create an order (cart_items, address_id).
- GET /v1/orders/{id}/tracking: Get status and courier location.
- PATCH /v1/courier/status: Update courier availability/ location.

#### Database Design

- Restaurant/Menu (MongoDB): Flexible schema for varied menu items/options.
- Orders (Postgres): Relational schema for ACID transactions.
- Courier Tracker (Redis): TTL-based location storage.

#### High-Level Architecture

The system consists of a Search Service (Elasticsearch) for discovery, an Order Service for state management, and a Dispatch Service that matches orders to couriers. A Push Service keeps all three parties in sync via WebSockets/Long Polling.

#### Detailed Components

- Search Service: Uses Elasticsearch for geospatial search and text relevance.
- Inventory Service: Manages real-time item availability (out of stock).
- Fulfillment/Dispatch: Complex logic to minimize delivery time and maximize courier utilization.
- Notification Engine: Orchestrates SMS, Push, and Email updates.
- Payment Gateway: Integration with Stripe/Braintree for transaction processing.

#### Data Flow

1. Order Placement: User → Order Service → Inventory Check → Payment Check → Order DB (Status: PENDING)
2. Restaurant Notification: Order Service → Notification → Restaurant App (Accepts) → Order DB (Status: PREPARING)
3. Dispatch: Order Service → Dispatcher → Find nearby Courier (Redis) → Notify Courier (Accepts) → Order DB (Status: COURIER_ASSIGNED)

#### Caching Strategy

Cache Restaurant metadata and Menus in Redis. Use CDNs for menu images. Order status is cached to prevent DB hits during frequent app-polling.

#### Database Choice

Postgres for Orders and Payments because of multi-table joins and transaction integrity. MongoDB for menus because restaurant offerings vary wildly (toppings, sizes, combos) and require a document structure.

#### Message Queue Usage

RabbitMQ/Kafka for task decoupling. Example: Once an order is placed, trigger the “Send Invoice,” “Update Loyalty Points,” and “Alert Kitchen” tasks asynchronously.

#### Storage Strategy

Structured data in RDS/NoSQL. Images (food/receipts) in S3. Logs and analytics in BigQuery.

#### Security

PCI-DSS compliance for payments. AuthZ to ensure restaurants only see their orders. Rate limiting to prevent scrapers from stealing menu data.

#### Scaling Strategy

Partition the Dispatch Service by city/zone. Use Read Replicas for the restaurant catalog to handle heavy browse traffic.

#### Bottlenecks

1. Peak Hour Spikes: 10x traffic at 7 PM. Mitigation: Auto-scaling groups and queue-based leveling. 2. Inventory Lag: User orders something just as it sells out. Mitigation: Distributed lock or inventory reservation flag. 3. Courier Supply: Not enough drivers for orders. Mitigation: Dynamic pricing (surge) and batching multiple orders for one driver.

#### Failure Handling

If the Dispatch service is down, orders are queued. If the Payment gateway is down, allow “Cash on Delivery” or show a “Payments Unavailable” banner.

#### Trade-offs

- Search latency vs. Freshness: Syncing Postgres to Elasticsearch might have a 1-second lag.
- Batching Orders: Better for profit but slows down individual deliveries.

#### Alternative Approaches

1. Single Monolith: Easier to start, but impossible for different teams (Restaurant vs. Logistics) to scale independently. 2. SQL for Menus: Too many joins for complex modifiers; NoSQL is more performant here.

#### Optimization Tips

- Use Geofencing to trigger “Courier is nearby” alerts to the customer.
- Pre-calculate popular menu items in the cache.
- Use a “Sidecar” pattern for logging and monitoring.

#### Interview Tips

- Mention the “Three-Legged” nature: Customer, Restaurant, and Courier.
- Discuss how to handle the “Order Cancellation” flow (who gets refunded? what happens to the food?).

#### Common Mistakes

- Ignoring the Restaurant’s side of the UI/UX.
- Not explaining how couriers are efficiently assigned (Greedy vs. Global Optimization).

#### Final Architecture Diagram

```text
[User] [Restaurant] [Courier]
\ | /
\---- [LB/Gateway] --/
/ | \
[Search] [Order Svc] [Dispatch Svc]
| | |
[ES] [Postgres] [Redis/Geo]
\ | /
-----> [Kafka] <-----
|
[Payment / Notification]
```

#### Complexity Discussion

- Matching: O(N) where N is couriers in the cell.
- Search: O(log N) via Elasticsearch indices.

#### Example API Request

```http
POST /v1/orders

{
  "restaurant_id": "rest_456",
  "items": [
    {
      "id": "burger_1",
      "qty": 2
    }
  ],
  "total": 24.99,
  "payment_method": "card_987"
}
```

#### Example API Response

```json
{
  "order_id": "ord_555",
  "estimated_delivery": "2023-10-27T18:45:00Z",
  "status": "ACCEPTED"
}
```

### Q33. Design Airbnb — Hard

#### Problem Statement

Design a global marketplace for lodging and tourism. The system must support property listings, complex search filters (price, date, amenities), and a booking system that prevents double-booking of dates.

#### Functional Requirements

- Host can list properties with photos and descriptions.
- Guest can search for properties by location, date range, and price.
- Guest can book a stay (check-in/check-out).
- System must handle booking status and payments.
- Reviews and ratings for both guests and hosts.

#### Non-functional Requirements

- High Availability for search/browsing.
- Strict Consistency for bookings (avoid overbooking a room).
- Low Latency for global search results.
- Scalability to handle millions of listings and images.

#### Scale Estimation

- 100M Users, 5M Listings.
- Search QPS: 10k.
- Booking QPS: 100.
- Storage: 5M listings * 5 images (0.5MB each) = 12.5 TB for photos.

#### API Design

- GET /v1/search?lat=x&lng=y&checkin=date&checkout=date: Find listings.
- GET /v1/listings/{id}: Detailed view.
- POST /v1/bookings: Request a reservation.
- POST / v1/listings: (Host) Create listing.

#### Database Design

- Listings (Postgres/PostGIS): Spatial index for search, metadata.
- Calendar/Availability (Cassandra): listing_id, date, status (AVAILABLE/ BOOKED). High write volume for calendar updates.
- Bookings (Postgres): Relational for ACID transactions.

#### High-Level Architecture

The system uses a Search Service powered by Elasticsearch for rich filtering. A Booking Service manages the availability calendar using a distributed lock or row-level locking in Postgres to ensure atomicity. Images are served via CDN from S3.

#### Detailed Components

- Search Indexer: Listens for listing updates via CDC (Change Data Capture) and updates Elasticsearch.
- Availability Service: A highly optimized service to check if a listing is free for N consecutive days.
- Payment Service: Handles escrow-like payments (guest pays, host receives after check-in).
- Review Service: Aggregates reviews and calculates average ratings.

#### Data Flow

1. Search: User → Search Svc → Elasticsearch (filter by lat/lng, price, amenities) → Filter results against Availability Svc (is it free on these dates?) → Result
2. Booking: User → Booking Svc → DB Transaction (Check dates in Availability Table → Insert Booking → Mark dates as Booked) → Payment → Success

#### Caching Strategy

Redis for listing details and user sessions. CDN (Cloudfront/ Akamai) globally for high-resolution property images.

#### Database Choice

Postgres for Listings and Bookings because we need transactional integrity for money and room occupancy. Elasticsearch is non-negotiable for the “Search” experience (fuzzy matching, range filters).

#### Message Queue Usage

Kafka to sync Listing changes to Search Index, and to trigger asynchronous tasks like “Send Confirmation Email” or “Notify Host via SMS.”

#### Storage Strategy

AWS S3 for all high-resolution images. Use a post-upload lambda to generate thumbnails.

#### Security

Identity verification (KYC) for hosts/guests. OAuth2 for social logins. Encryption at rest for PII.

#### Scaling Strategy

Sharding the Availability and Booking tables by listing_id. Using Georeplication for the Search Index to serve users from the nearest data center.

#### Bottlenecks

1. Calendar Concurrency: Millions of people checking dates. Mitigation: Distribute calendar into weekly/monthly buckets for faster queries. 2. Search Relevance: Too many results. Mitigation: Machine learning models for ranking based on user history. 3. Image Loading: High latency for global users. Mitigation: Aggressive CDN caching and WebP compression.

#### Failure Handling

If the Search service fails, fallback to a simplified DB query. If the Payment service is down, queue the booking request as “PENDING_PAYMENT.”

#### Trade-offs

- Consistency vs. Performance: Checking availability during search makes search slower but UX better (prevents clicking on booked rooms).
- Normal ization: Denormalizing host info into the Listing table to avoid joins.

#### Alternative Approaches

1. NoSQL for Bookings: Rejected because the risk of double-booking (race conditions) is too high without ACID. 2. Clientside Search: Only works for small datasets; rejected for 5M listings.

#### Optimization Tips

- Use Geohashes for clustering listings on a map.
- Implement Saga pattern for distributed transactions between Booking and Payment services.
- Use HyperLogLog for counting unique listing views.

#### Interview Tips

- The “Double Booking” problem is the core of this interview. Explain row-level locking or SELECT ... FOR UPDATE.
- Discuss how to handle different time zones for check-in/out.

#### Common Mistakes

- Failing to explain how Elasticsearch stays in sync with the main DB.
- Ignoring the photo upload/processing pipeline.

#### Final Architecture Diagram

```text
[Guest App] [Host App]
| |
[ Load Balancer/Gateway ]
/ | \
[Search] [Booking] [Listing Manage]
| | |
[ES] [Postgres] [Postgres]
\ | /
--[Kafka]-----------
| |
[Payment] [S3 Bucket]
```

#### Complexity Discussion

- Search: O(log N) where N is property count.
- Booking: O(K) where K is number of nights (checking each day’s availability).

#### Example API Request

```http
POST /v1/bookings

{
  "listing_id": "lst_123",
  "check_in": "2023-12-01",
  "check_out": "2023-12-05",
  "guest_count": 2
}
```

#### Example API Response

```json
{
  "booking_id": "bk_789",
  "status": "confirmed",
  "total_price": 500.0
}
```

### Q34. Design a Payment Gateway (Stripe-like) — Hard

#### Problem Statement

Design a robust, secure payment gateway that acts as an interface between merchants and financial institutions. The system must process transactions, handle refunds, and ensure exactly-once processing in a highly regulated environment.

#### Functional Requirements

- Merchants can integrate via API to accept payments.
- Support multiple payment methods (Card, Bank Transfer, Wallets).
- Handle the full transaction lifecycle (Authorize, Capture, Refund, Dispute).
- Provide a dashboard for merchants to view transaction history.
- Webhooks to notify merchant systems of status changes.

#### Non-functional Requirements

- Extreme Consistency: Financial data must never be lost or duplicated (Atomicity).
- Security: PCI-DSS compliance, hardware security modules (HSM).
- Durability: Zero data loss for transaction logs.
- Avail ability: 99.999% uptime; payment failure means lost revenue for merchants.

#### Scale Estimation

- 10,000 Transactions Per Second (TPS) at peak.
* 1 Billion transactions per month.
- Storage: 1 KB per transaction = 1 TB/month.

#### API Design

- POST /v1/charges: Create a payment (amount, currency, source, idempotency_key).
- GET /v1/charges/{id}: Retrieve charge status.
- POST /v1/refunds: Initiate a refund.
- GET /v1/balance: Check merchant account balance.

#### Database Design

- Transactions (Postgres): tx_id, merchant_id, amount, status, idempotency_key, external_ref.
- Ledger (Postgres/Immutable Log): Double-entry bookkeeping records (debits/credits).
- Merchant Metadata (Postgres/Cassan dra): API keys, business info.

#### High-Level Architecture

An API Gateway handles ingestion. An Idempotency Layer prevents duplicates. The Core Payment Engine talks to external Bank Adapters. An asynchronous Ledger Service records financial movements, and a Webhook Service notifies the merchants.

#### Detailed Components

- Idempotency Manager: Uses Redis to store unique keys for 24 hours.
- Bank/Processor Adapters: Translates internal requests to ISO 8583 or JSON for Stripe/Adyen/Banks.
- Risk/Fraud Engine: Analyzes transactions in realtime (velocity checks, IP blacklists).
- Ledger: An append-only database to ensure an audit trail.
- Vault Service: Tokenizes sensitive card data (PCI-DSS requirement).

#### Data Flow

1. Request: Merchant → API Gateway (Check Idempotency) → Risk Engine → Vault (Tokenize Card)
2. Execution: Payment Engine → Bank Adapter → External Bank (Authorize) → DB Update (Status: AUTHORIZED)
3. Settlement: Background job → External Bank (Capture) → Update Ledger (Status: SUCCESS) → Trigger Webhook

#### Caching Strategy

Redis for API key validation, rate limiting, and idempotency keys. Transaction data is NOT cached for reading to avoid staleness; use read-replicas instead.

#### Database Choice

Postgres (Aurora) with high-availability configuration. Relational databases are essential for the ACID properties required by the double-entry bookkeeping system to ensure money isn’t created or destroyed out of thin air.

#### Message Queue Usage

Kafka is used for downstream processing: updating analytics, generating receipts, and retrying webhook deliveries. Reliable queuing is needed for “At-least-once” delivery of notifications.

#### Storage Strategy

Relational DB for active transactions. Cold Storage (S3/Glac ier) for logs and transaction archives required for 7-year legal compliance.

#### Security

PCIDSS Level 1. Use an isolated Vault for PANs (Primary Account Numbers). Mutual TLS (mTLS) for bank communications.

#### Scaling Strategy

Shard the transaction database by merchant_id. Use a distributed lock for sensitive ledger updates. Global edge points to reduce latency for non-sensitive API calls.

#### Bottlenecks

1. Bank Latency: External banks take 2-5 seconds. Mitigation: Asynchronous processing with callbacks/webhooks. 2. Consistency vs. Speed: Locking the ledger records. Mitigation: Use a high-performance distributed ledger or sharded Postgres. 3. Duplicate Requests: Network retries causing double charges. Mitigation: Mandatory idempotency keys.

#### Failure Handling

If a bank is down, the adapter triggers a circuit breaker and potentially fails over to a secondary provider. If the Webhook service fails, implement an exponential backoff retry mechanism.

#### Trade-offs

- Latency vs. Security: Extra hops for fraud checks add latency but save money on chargebacks.
- Synchronous vs. Asynchronous: Authorize is sync (to tell the user if it worked); Capture/Settlement is async.

#### Alternative Approaches

1. NoSQL for Ledger: Rejected because it’s hard to maintain strict double-entry balance constraints (Sum of Debits == Sum of Credits). 2. Public Cloud for Vaulting: Possible, but often required to be physically or logically isolated.

#### Optimization Tips

- Use “Optimistic Locking” for merchant balance updates.
- Batch bank settlement files to reduce processing fees.
- Implement a “Reconciliation Service” to compare internal logs with daily bank statements.

#### Interview Tips

- Emphasize Idempotency—it is the most important word in this interview.
- Mention DoubleEntry Bookkeeping.
- Talk about handling the “Partial Failure” case (e.g., Bank charged, but DB update failed).

#### Common Mistakes

- Storing raw credit card numbers in the main database.
- Not explaining how to handle time-outs from the bank’s side.
- Forgetting to mention how to handle refunds/chargebacks.

#### Final Architecture Diagram

```text
[Merchant] -> [LB] -> [API Gateway (Idempotency)]
|
[Risk/Fraud Engine]
|
[Vault Service] <--- [Payment Engine] ---> [Bank Adapter]
| | |
[Token DB] [Postgres DB] [External Bank]
|
[Kafka / Ledger]
/ | \
[Webhooks] [Receipts] [Analytics]
```

#### Complexity Discussion

- Time: O(1) for lookups; external API calls are the bottleneck.
- Space: O(N) where N is the transaction volume.

#### Example API Request

```http
POST /v1/charges

{
  "amount": 5000,
  "currency": "usd",
  "source": "tok_visa_123",
  "idempotency_key": "unique_str_789"
}
```

#### Example API Response

```json
{
  "id": "ch_aa11",
  "amount": 5000,
  "status": "succeeded",
  "created": 1698345600
}
```

### Q35. Design UPI / Realtime Payments — Hard

#### Problem Statement

Design a real-time, mobile-first payment system that allows instant fund transfers between two different bank accounts via a Virtual Payment Address (VPA) or QR code. The system must act as a central switch (like NPCI) connecting multiple PSPs (Payment Service Providers) and Banks.

#### Functional Requirements

- Users can link multiple bank accounts to a single VPA (e.g., user@bank).
- Instant P2P and P2M (merchant) fund transfers.
- Standardized QR code scanning for payments.
- Transaction PIN verification for security.
- Balance inquiry and transaction history.

#### Non-functional Requirements

- Subsecond Latency: The “Golden Path” (Request to Success) must be < 2 seconds.
- UltraHigh Availability: Systemic failure can stop a nation’s economy.
- Strict Consistency: Correctness of balances is paramount.
- High Throughput: Capable of handling 50k+ TPS.

#### Scale Estimation

- 300M Monthly Active Users.
* 10 Billion transactions per month.
- Peak TPS: 100,000.
- Daily Storage: 10B / 30 * 1KB ≈ 333 GB/day.

#### API Design

- POST /v1/txn/pay: Initiate payment (senderVPA, receiverVPA, amount, pin_hash).
- GET /v1/account/balance: Check linked bank balance.
- POST /v1/account/ link: Mobile number to Bank Account binding.
- POST /v1/txn/verify: Status check for a transaction.

#### Database Design

- VPA Registry (Cassandra/DynamoDB): Mapping VPA -> {AccountNo, IFSC, BankID}. High read volume.
- Transaction Switch Log (Postgres): txn_id, sender_info, receiver_info, status, amount, timestamp.
- Device Binding (Redis/ SQL): DeviceID, MobileNumber, PublicKey.

#### High-Level Architecture

The UPI Switch sits in the middle. The PSP (App) sends a request to the Switch. The Switch looks up the VPA in a central directory, communicates with the Remitter Bank (Sender) to debit funds, and concurrently or sequentially communicates with the Beneficiary Bank (Receiver) to credit funds.

#### Detailed Components

- VPA Resolver: A high-performance distributed hash table mapping aliases to bank details.
- Authentication Service: Validates device fingerprints and encrypted MPIN.
- The Switch: Orchestrator that manages the 2-phase or 3-phase commit across different bank APIs.
- Settlement Engine: Background process for net settlement between banks (interbank ledger).
- Common Library (SDK): Provided to banks/PSPs for encryption and standardizing communication.

#### Data Flow

1. Initiation: User App → PSP Server → UPI Switch
2. Resolution: Switch queries VPA Registry to find Beneficiary Bank
3. Debit: Switch → Remitter Bank (Verify PIN + Debit)
4. Credit: Switch → Beneficiary Bank (Credit)
5. Finalize: Switch sends “Success” to both PSPs and logs the transaction

#### Caching Strategy

Redis for VPA resolution (TTL 1 hour) to reduce DB load. Cache device metadata and active sessions. No caching for transaction status; must be fresh.

#### Database Choice

Postgres (with Citus or Sharding) for the main transaction log to ensure ACID. Cassandra for the VPA Registry because it provides high availability and scales linearly for simple key-value lookups.

#### Message Queue Usage

Kafka for logging every state change for auditing. Also used to decouple the “Reconciliation” and “Notification” services from the main transaction flow.

#### Storage Strategy

Hot Storage: Sharded SQL for last 3 months of transactions. Cold Storage: Data warehouse (Hadoop/Snowflake) for long-term audit logs.

#### Security

Hardware Security Modules (HSM) for PIN processing. Device Bind ing (SIM binding) to prevent remote hijacking. End-to-end encryption between PSP and Switch.

#### Scaling Strategy

Regional Partitioning: Shard the Switch and DBs based on BankID or User Geolocation. Use Anycast IP to direct traffic to the nearest data center.

#### Bottlenecks

1. Bank Response Time: Banks have legacy systems. Mitigation: Aggressive timeouts and a “Pending” state management. 2. VPA Hotspots: Popular merchants receiving 1000s of hits. Mitigation: Read replicas for the VPA service. 3. Network Jitter: Mobile networks are unstable. Mitigation: Idempotent retries from the mobile app.

#### Failure Handling

If the Beneficiary bank is down, the Switch may put the transaction in “PENDING” and retry or trigger a “Deemed Success” (if debit was successful and credit is guaranteed later).

#### Trade-offs

- Synchronous vs. Asynchronous Credit: Synchronous credit is safer but slower. Asynchronous (Debit then acknowledge) is faster for the user.
- Central ized vs. Decentralized VPA: Centralized is easier to manage but is a single point of failure (mitigated by massive redundancy).

#### Alternative Approaches

1. Blockchain: Too slow and lacks central regulatory control. 2. Pure P2P (No Switch): Complexity for banks to connect to 100+ other banks; n^2 connections vs. n connections to the Switch.

#### Optimization Tips

- Use UDP/gRPC for internal communication between Switch components to shave off milliseconds.
- Implement “Pre-check” for balance (if cached/ available) to fail fast.
- Use Connection Pooling to banks to avoid handshake overhead.

#### Interview Tips

- Explain the difference between Payer PSP and Payee PSP.
- Discuss how the system handles a “Debit successful but Credit failed” scenario (Reversal).
- Highlight the security aspect: “Something you have (SIM/Device)” and “Something you know (PIN).”

#### Common Mistakes

- Neglecting the inter-bank settlement process (the actual movement of “bulk” money).
- Assuming the bank APIs are always fast and reliable.

#### Final Architecture Diagram

```text
[User App] -> [PSP Server]
|
[ UPI SWITCH ]
/ | \
[VPA Registry] [Auth Svc] [Txn Log (SQL)]
| | |
[Remitter Bank] <--|--> [Beneficiary Bank]
| | |
\-----[ Kafka / Reconciliation ]-----/
```

#### Complexity Discussion

- Time: O(1) for VPA lookup; O(B) for bank roundtrips.
- Space: O(T) where T is the total transaction volume.

#### Example API Request

```http
POST /v1/txn/pay

{
  "remitter_vpa": "alice@okbank",
  "beneficiary_vpa": "bob@okaxis",
  "amount": "100.00",
  "txn_note": "Dinner",
  "encrypted_pin": "base64_blob"
}
```

#### Example API Response

```json
{
  "txn_id": "UPI123456789",
  "status": "SUCCESS",
  "timestamp": "2023-10-27T10:00:00Z"
}
```

### Q36. Design a Stock Exchange Matching Engine — Hard

#### Problem Statement

Design a high-performance electronic trading system that matches buy and sell orders for financial instruments. The system must process millions of orders per second with microsecond latency and guarantee strict FIFO (First-In-First-Out) prize-time priority.

#### Functional Requirements

- Place Limit, Market, and Stop orders.
- Real-time Order Book updates (Level 2 data).
- Order cancellation and modification.
- Execution reports (fills/partial fills) delivered to participants.
- Market data broadcasting (UDP Multicast or WebSockets).
- Account balance and margin validation before matching.

#### Non-functional Requirements

- Latency: Ultra-low (sub-millisecond P99).
- Availability: High (99.999%), though correctness/consistency is prioritized over availability during partitions.
- Consistency: Strict ACID transactions for order matching.
- Scale: Support 100k+ symbols and peak bursts of 1M orders/sec.

#### Scale Estimation

- DAU: 500k active traders.
- QPS: 50k avg, 1M peak.
- Storage: 10B trades/year (approx. 2TB raw data).
- Bandwidth: 10Gbps+ for market data feeds.
- Math: 1M orders/sec * 100 bytes = 100MB/s throughput for the matching core.

#### API Design

- POST /v1/orders (gRPC/FIX): Place order {symbol, side, qty, price, type}. Returns OrderID.
- DELETE /v1/orders/{id}: Cancel order. Returns status.
- GET /v1/ orderbook/{symbol}: Get current L2 snapshots.
- GET /v1/executions: Stream of executions via WebSocket/FIX.

#### Database Design

- Orders (SQL Postgres): id, user_id, symbol, price, qty, status, timestamp. Primary store for history.
- Order Book (InMemory/C++): Doubly linked lists with HashMaps for 𝑂(1) lookups and 𝑂(1) cancels.
- Account Ledger (SQL): account_id, balance, reserved_margin. Must be strictly consistent.

#### High-Level Architecture

The system uses a Gateway for protocol translation (FIX/Binary), a Risk Engine for pre-trade checks, and a Sequencer to provide a global monotonic ID. The Matching Engine (ME) is a deterministic state machine running entirely in RAM. Matched trades are persisted via a Journaler and indexed in a Query Service.

#### Detailed Components

- Sequencer: Uses Raft or Zab to assign a sequence number to every incoming event to ensure deterministic replay.
- Matching Engine (ME): A single-threaded (per partition/symbol set) process using an in-memory Btree or Price-Point Map.
- Risk Engine: Validates that the user has enough collateral before forwarding to the ME.
- Market Data Service: Aggregates order book changes and broadcasts them via UDP Multicast (LAN) or WebSockets (WAN).
- Journaler: Low-latency disk logger (AOF) for recovery.

#### Data Flow

1. Write: Client → Gateway → Sequencer (Assign ID) → Risk Engine → Matching Engine → Journaler → Async DB Update
2. Read: Client → Market Data Service (Cache) → Subscriber

#### Caching Strategy

No traditional caching (Redis) in the matching path; it’s too slow. Instead, the “Cache” is the Matching Engine’s internal RAM. Snapshots are stored every 𝑁 sequences to speed up recovery.

#### Database Choice

Use PostgreSQL for trade history and user accounts because of ACID requirements. However, the active state lives in memory (C++ or Java with LMAX Disruptor). SQL is used only for “cold” data and archival.

#### Message Queue Usage

Use LMAX Disruptor (Inter-process) or Kafka (Interservice) for decoupling the ME from the reporting and billing services.

#### Storage Strategy

High-speed NVMe for the Transaction Journal. S3 for long-term historical trade logs and compliance (WORM - Write Once Read Many).

#### Security

TLS 1.3 for external traffic, dedicated leased lines (Direct Connect) for HFT firms, KYC validation at the Gateway, and HMAC signatures for API requests.

#### Scaling Strategy

Partition the Matching Engine by Symbol (e.g., BTC/USD on Node 1, ETH/USD on Node 2). This allows horizontal scaling of the engine without cross-node synchronization.

#### Bottlenecks

1. Network I/O: Mitigated by using Kernel Bypass (DPDK/Solarflare). 2. Garbage Collection: Mitigated by using C++ or Zero-GC Java (Object Pooling). 3. The Sequencer: Mitigated by using high-speed hardware clocks (PTP).

#### Failure Handling

If an ME node dies, a secondary follower replays the Journal from the last snapshot to rebuild the state. Since the sequencer is deterministic, the result is identical.

#### Trade-offs

- Latency vs. Durability: Chose synchronous journaling for safety over maximum speed.
- Singlethreaded vs Multithreaded ME: Chose single-threaded per symbol to avoid locking overhead.
- Consistency vs. Availability: Favored Strict Consistency (CP in CAP).

#### Alternative Approaches

1. Distributed Transactions (2PC): Rejected because it adds milliseconds of latency. 2. NoSQL (Cassandra): Rejected because it lacks the ACID guarantees needed for financial ledgers.

#### Optimization Tips

- Use Memory Mapping (mmap) for the Journaler.
- CPU Pinning to avoid context switches.
- Data structure alignment to utilize CPU L1/L2 caches.

#### Interview Tips

- Focus on Determinism.
- Mention the significance of the LMAX Disruptor.
- Explain that the Matching Engine never touches a database in the hot path.

#### Common Mistakes

- Suggesting a REST API for the matching core (too slow).
- Using a global lock on the order book.
- Forgetting pre-trade risk checks.

#### Final Architecture Diagram

```text
Client -> [Gateway] -> [Sequencer] -> [Matching Engine] -> [Journaler]
| |
[Risk Engine] [Market Data] -> [Websocket]
| |
[PostgreSQL] [Historical API]
```

#### Complexity Discussion

- Time: Matching is 𝑂(1) or 𝑂(log𝑃) where 𝑃 is the number of price levels.
- Space: 𝑂(𝑁) where 𝑁 is the number of open orders.
- Network: 𝑂(1) hops between sequencer and engine.

#### Example API Request

```json
{
  "symbol": "AAPL",
  "side": "BUY",
  "price": 150.25,
  "quantity": 100,
  "type": "LIMIT"
}
```

#### Example API Response

```json
{
  "orderId": "ox-12345",
  "status": "ACCEPTED",
  "timestamp": "2023-10-27T10:00:00.001Z"
}
```

### Q37. Design a Cryptocurrency Exchange — Hard

#### Problem Statement

Design a platform for trading digital assets. Unlike a traditional exchange, this must handle hot/cold wallet management, blockchain integrations, and 24/7/365 global operations.

#### Functional Requirements

- User registration and KYC.
- Fiat on-ramp/off-ramp (Deposits/Withdrawals).
- Crypto wallet generation.
- Spot trading (Matching Engine).
- Real-time price tickers.

#### Non-functional Requirements

- Security: Most critical; protection against private key theft.
- Availability: 24/7 uptime.
- Scalability: Support rapid spikes in volatility.
- Auditability: Every satoshi must be accounted for.

#### Scale Estimation

- 10M users.
* 1M orders/day.
* 100k requests/sec (price feeds).
- Storage: 50TB+ for historical trade data and audit logs.

#### API Design

- POST /v1/wallet/deposit/address: Generate deposit address.
- POST /v1/ orders: Same as Stock Exchange.
- GET /v1/balance: Returns all asset holdings.
- POST / v1/withdraw: Initiate crypto transfer to external address.

#### Database Design

- User Table: id, email, kyc_status, 2fa_secret.
- Wallets: user_id, currency, public_key, encrypted_private_key_shard.
- Transactions: id, user_id, type (DEPOSIT/TRADE), amount, tx_hash.

#### High-Level Architecture

Uses a Matching Engine similar to Q36, but wrapped in a Wallet Infrastructure. A Custodian Service manages private keys. Blockchain Listeners (Nodes) monitor the chain for deposits, and Withdrawal Workers sign outgoing transactions.

#### Detailed Components

- HMS (Hardware Security Module): Stores master keys for signing transactions.
- Hot Wallet: Fast, connected to the internet, contains small funds.
- Cold Wallet: Air-gapped, requires manual intervention for large transfers.
- Event Sourcing Service: Records every balance change to allow full reconstruction.

#### Data Flow

1. Deposit: User sends BTC → Blockchain Node detects transfer → Event Service notifies Wallet Service → User balance updated
2. Trade: User places order → Matching Engine (RAM) → Trade Executed → Settlement Service updates internal ledgers

#### Caching Strategy

Redis for user sessions, exchange rates, and recent order history. CDN for front-end assets and market data snapshots.

#### Database Choice

CockroachDB or Spanner for the internal ledger to ensure global consistency and horizontal scalability across regions, preventing double-spending.

#### Message Queue Usage

Kafka is used for the “Clearing and Settlement” pipeline. Once the Matching Engine produces a fill, Kafka ensures the ledger service and notification service process it exactly once.

#### Storage Strategy

Ledger data in NewSQL. Large blob data (IDs, KYC documents) in S3 with encryption at rest.

#### Security

Multi-sig wallets, 2FA, IP whitelisting for withdrawals, cold storage for 95% of assets, and rate-limiting at the Cloudflare edge.

#### Scaling Strategy

Horizontal scaling of the API and WebSocket layers. Microservices architecture allowing independent scaling of the “Withdrawal Worker” and “Trade History” services.

#### Bottlenecks

1. Blockchain Sync Speed: Mitigated by running multiple full nodes. 2. Ledger Contention: Mitigated by sharding the ledger by user_id. 3. Key Signing: Mitigated by using MPC (Multi-Party Computation) for faster signing.

#### Failure Handling

Wallet services use redundant nodes. If the Matching Engine fails, it recovers from the journal. For blockchain forks, the system pauses withdrawals automatically.

#### Trade-offs

- Speed vs. Security: Withdrawals have a delay for manual/risk review.
- Hot vs. Cold: More in hot wallet increases speed but increases risk.

#### Alternative Approaches

1. Decentralized Exchange (DEX): Rejected due to high latency and gas costs. 2. Standard PostgreSQL: Rejected in favor of Spanner for better cross-region availability.

#### Optimization Tips

- Batch blockchain withdrawals to save on network fees.
- Use WebSockets (WSS) instead of polling for price updates.
- Pre-calculate candle data (OHLC) in a Time-Series DB (InfluxDB).

#### Interview Tips

- Differentiate between “on-chain” (deposits) and “off-chain” (trading) operations.
- Explain the concepts of Hot/Cold storage clearly.

#### Common Mistakes

- Storing private keys in plain text or standard DBs.
- Treating crypto deposits as instantaneous (ignoring confirmations).

#### Final Architecture Diagram

```text
Client -> [API Gateway] -> [Matching Engine] -> [Ledger Service]
| | |
[Wallet Service] <--- [Kafka] -----------> [DB/Audit]
|
[Blockchain Nodes]
```

#### Complexity Discussion

- Time: Matching is 𝑂(1). Ledger updates are 𝑂(1) with indexed PKs.
- Space: 𝑂(𝑈) where 𝑈 is the number of users.

#### Example API Request

```json
{
  "asset": "ETH",
  "amount": "1.5",
  "destination": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e"
}
```

#### Example API Response

```json
{
  "withdrawalId": "w-999",
  "status": "PENDING_APPROVAL"
}
```

### Q38. Design Kafka — Hard

#### Problem Statement

Design a distributed, partitioned, replicated commit log service for high-throughput message streaming. It must provide low-latency writes and allow multiple consumers to read data at their own pace.

#### Functional Requirements

- Publish messages to topics.
- Subscribe to topics and read messages.
- Message persistence for a retention period.
- Consumer groups for load balancing.
- Order guarantees within a partition.

#### Non-functional Requirements

- High Throughput: Millions of messages/sec.
- Durability: Data must not be lost once acknowledged.
- Scalability: Add brokers/ partitions on the fly.
- Availability: Tolerant of broker failures.

#### Scale Estimation

- 1 Trillion messages/day.
* 10 PB storage.
- Write QPS: 10M/s.
- Read QPS: 50M/s (multiple fan-outs).

#### API Design

- produce(topic, partition, key, value)
- fetch(topic, partition, offset)
- listOffsets(topic, partition)
- joinGroup(group_id)

#### Database Design

No traditional DB. Data is stored in Appendonly Segment Files on disk. An Index file maps message offsets to byte positions.

#### High-Level Architecture

The cluster consists of Brokers. Each Topic is split into Partitions. Each partition has one Leader and multiple Followers (ISR - In-Sync Replicas). A Metadata Store (Zookeeper or KRaft) manages the cluster state.

#### Detailed Components

- Producer: Buffers messages, batches them, and sends them to the leader broker.
- Broker: Handles I/O, writing batches to the OS Page Cache.
- Consumer: Pull-based model; tracks its own progress via Offsets.
- Log Manager: Handles segment rolling and retention (deletion/compaction).
- Con troller: A broker elected to manage partition leadership.

#### Data Flow

1. Write: Producer → Leader Broker → Write to Page Cache → Replicate to Followers → Ack to Producer
2. Read: Consumer → Leader Broker → Zero-copy read from Page Cache (or disk) → Send to Consumer

#### Caching Strategy

Relies heavily on the Operating System Page Cache. Since Kafka reads are sequential, the OS pre-fetches data, making reads nearly as fast as RAM.

#### Database Choice

Local File System. Kafka bypasses the overhead of a database by using the filesystem as a structured log.

#### Message Queue Usage

Kafka is the message queue, but internally it uses a request queue to handle incoming network requests.

#### Storage Strategy

Logstructured storage. Active segments are written to; closed segments are immutable. Support for Log Compaction (keep only the latest value for a key).

#### Security

SASL/OAUTHBEARER for AuthN, ACLs for AuthZ, and TLS for encryption in transit.

#### Scaling Strategy

Increase the number of Partitions for a topic to allow more parallel consumers and spread the load across more brokers.

#### Bottlenecks

1. Disk I/O: Mitigated by sequential writes and batching. 2. Network Bandwidth: Mitigated by end-to-end compression (Gzip/Snappy). 3. CPU (Encryp tion/Compression): Mitigated by offloading to specialized hardware or efficient libraries.

#### Failure Handling

If a Leader fails, the Controller elects a new leader from the ISR. If the Controller fails, a new one is elected via Zookeeper/KRaft.

#### Trade-offs

- Pull vs Push: Chose Pull to let consumers handle their own backpressure.
- Sync vs Async Replication: Sync for durability, Async for throughput.
- Filesystem vs DB: Filesystem for raw performance, sacrificing complex queries.

#### Alternative Approaches

1. RabbitMQ: Rejected because it doesn’t store history (not a commit log). 2. Pulsar: Similar, but rejected for this design to focus on Kafka’s simpler architecture.

#### Optimization Tips

- Zerocopy (sendfile): Transfers data from page cache directly to network socket.
- Batching: Reduces the number of network round trips and disk IOPS.
- Page Cache tuning: Allocate most RAM to the OS for caching.

#### Interview Tips

- Explain Zerocopy clearly.
- Understand the role of the ISR (InSync Replicas).
- Know the difference between “At-least-once” and “Exactlyonce” (EOS).

#### Common Mistakes

- Thinking Kafka pushes data to consumers.
- Confusing partitions for replication.

#### Final Architecture Diagram

```text
Producers -> [Broker (Leader)] --(Replication)--> [Broker (Follower)]
| Page Cache |
v v
[Segment Files] [Segment Files]
|
v
Consumers <- [Broker (Leader)]
```

#### Complexity Discussion

- Time: 𝑂(1) for both append and fetch (by offset).
- Space: 𝑂(𝑁) where 𝑁 is message volume within retention.

#### Example API Request

```json
{
  "apiKey": "PRODUCE",
  "topic": "user-clicks",
  "partition": 2,
  "records": [
    {
      "key": "u1",
      "value": "click-home"
    }
  ]
}
```

#### Example API Response

```json
{
  "topic": "user-clicks",
  "partition": 2,
  "baseOffset": 50512,
  "errorCode": 0
}
```

### Q39. Design Elasticsearch — Hard

#### Problem Statement

Design a distributed, RESTful search and analytics engine capable of solving a growing number of use cases like full-text search, logs, and metrics with near real-time latency.

#### Functional Requirements

- Index JSON documents.
- Full-text search with relevance scoring (BM25).
- Aggregations (Min, Max, Avg, Histograms).
- Auto-suggest / Type-ahead.
- Multi-tenancy via multiple indices.

#### Non-functional Requirements

- Near RealTime (NRT): Changes visible within 1 second.
- Scalability: Scale to petabytes of data.
- Availability: No data loss on node failure.
- Consistency: Eventual consistency for reads.

#### Scale Estimation

- 100 TB of logs/day.
* 10k Search Queries per second (SQPS).
* 1M Indexing OPS.
- Retention: 30 days (3 PB total).

#### API Design

- PUT /{index}/_doc/{id}: Index a document.
- POST /{index}/_search: Query with DSL (JSON).
- GET /{index}/_stats: Monitoring indices.
- DELETE /{index}: Remove index.

#### Database Design

Data is stored in Lucene Segments. Key structures: Inverted Index (for search) and Doc Values (columnar for aggregations).

#### High-Level Architecture

A cluster of Nodes. Each node can be a Master (cluster state), Data (stores shards), or Coordinating node (routes requests). Indices are split into Shards (primary and replica).

#### Detailed Components

- Inverted Index: Mapping of words to document IDs.
- Translog: Write-ahead log for durability.
- Segment Merging: Background process to combine Lucene segments.
- Circuit Breakers: Prevent nodes from OOM by tracking memory usage.
- Buffer: In-memory buffer for incoming documents.

#### Data Flow

1. Write: Request → Coordinating Node → Primary Shard (Write to Buffer + Translog) → Replica Shards → Ack. Every 1s, buffer is flushed to a new Segment (Refresh)
2. Read: Request → Coordinating Node → Scatter to all Shards → Gather results → Sort/Merge → Return

#### Caching Strategy

- Node Query Cache: Caches filters.
- Shard Request Cache: Caches local search results.
- Field Data Cache: Used for aggregations (mostly superseded by Doc Values).

#### Database Choice

Lucene is the core engine. It’s chosen because of its highly optimized data structures (FSTs, Skip Lists) for text retrieval.

#### Message Queue Usage

Not built-in, but usually preceded by Kafka or Logstash to buffer high-volume logs before indexing to prevent overwhelming ES.

#### Storage Strategy

Local SSDs for active shards. S3/GCS for Searchable Snap shots (cold storage) to reduce costs.

#### Security

RBAC (Role-Based Access Control), Field/Document level security, TLS, and Audit logging.

#### Scaling Strategy

Increase shards for parallelism. Add nodes for capacity. Use Hot WarmCold architecture to move older data to cheaper hardware.

#### Bottlenecks

1. Garbage Collection: Java heap management (mitigated by G1GC or ZGC). 2. Too many shards: Increases master node overhead (mitigated by shard shrinking). 3. Heavy Aggregations: Mitigated by using Doc Values and memory limits.

#### Failure Handling

If a primary shard fails, a replica is promoted. The Cluster State is replicated across all master-eligible nodes to ensure no single point of failure.

#### Trade-offs

- NRT vs Realtime: 1s refresh interval for better throughput.
- Memory vs Performance: Columnar storage (Doc Values) on disk vs in RAM.

#### Alternative Approaches

1. Solr: Similar, but harder to scale dynamically. 2. Postgres (GIN Index): Rejected for large scale/aggregations.

#### Optimization Tips

- Disable _all field to save space.
- Use bulk API for indexing.
- Set refresh_interval to −1 during large bulk loads.

#### Interview Tips

- Explain the Inverted Index concept clearly.
- Discuss the ScatterGather pattern for distributed search.

#### Common Mistakes

- Thinking ES is a primary database (it’s a search index).
- Ignoring the “split-brain” problem (mitigated by minimum_master_nodes or the new cluster coordination in 7.x+).

#### Final Architecture Diagram

```text
Client -> [LB] -> [Coordinating Node]
/ | \
[Primary Shard] [Replica] [Primary Shard]
| | |
[Lucene Seg] [Lucene Seg] [Lucene Seg]
```

#### Complexity Discussion

- Time: Search is 𝑂(𝑄⋅log𝐷) where 𝐷 is doc count.
- Space: 𝑂(𝑇𝑒𝑟𝑚𝑠+𝐷𝑜𝑐𝑠).

#### Example API Request

```json
{
  "query": {
    "match": {
      "title": "architect"
    }
  },
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

#### Example API Response

```json
{
  "hits": {
    "total": 1,
    "hits": [
      {
        "_source": {
          "title": "Software Architect"
        }
      }
    ]
  },
  "aggregations": {
    "avg_price": {
      "value": 150.0
    }
  }
}
```

### Q40. Design Collaborative Document Editing — Hard

#### Problem Statement

Design a real-time collaborative editor like Google Docs where multiple users can edit the same document simultaneously without conflicts, seeing each other’s changes in real-time.

#### Functional Requirements

- Real-time multi-user editing.
- Presence indicators (who is online).
- Cursor tracking.
- Document history/versioning.
- Comments and suggestions.

#### Non-functional Requirements

- Latency: Low (under 100ms for local feedback).
- Consistency: Eventual consistency (Convergence).
- Availability: High availability.
- Scalability: Millions of concurrent documents.

#### Scale Estimation

- 100M active docs.
* 10M DAU.
- Avg 3 users/doc.
* 10-20 edits/ sec per active doc.

#### API Design

- GET /v1/doc/{id}: Fetch initial state.
- WS /v1/doc/{id}/stream: WebSocket for sending/receiving operations.
- POST /v1/doc/{id}/snapshot: Save a manual version.

#### Database Design

- Documents (NoSQL MongoDB/CouchDB): id, content, version.
- Operations Log (Cassandra): doc_id, seq_num, user_id, operation_data.
- Presence (Redis): doc_id, user_id, last_seen.

#### High-Level Architecture

Uses WebSockets for real-time bi-directional communication. Conflict resolution is handled via OT (Operational Transformation) or CRDT (Conflictfree Replicated Data Types). A Session Server manages active doc state in memory.

#### Detailed Components

1. OT/CRDT Engine: The logic that merges concurrent changes. 2. WebSocket Server: Maintains persistent connections. 3. Pub/Sub (Re dis): Broadcasts changes from one user to others on the same doc. 4. Snapshot Service: Periodically saves the full doc state to avoid replaying millions of ops.

#### Data Flow

1. Write: User types → Local UI updates → Op sent via WS → Server validates/transforms → Server broadcasts to others → Saved to Op Log
2. Read: User opens doc → Load latest Snapshot → Apply pending operations since snapshot → Join WS stream

#### Caching Strategy

Redis for the current “Hot” document state and active user cursors. LRU cache on the Session Server for the most recent document segments.

#### Database Choice

Cassandra for the Operations Log because of its high write throughput and ability to handle time-ordered data. S3 for storing document snapshots.

#### Message Queue Usage

Redis Pub/Sub for instant broadcast of small delta updates to connected clients.

#### Storage Strategy

Store deltas (operations) in Cassandra. Store full snapshots (PDF/JSON) in S3. This allows “Time Travel” through the document history.

#### Security

JWT for session auth, TLS, and fine-grained permissions (Viewer vs Editor).

#### Scaling Strategy

Sticky load balancing to ensure all users of Doc A are routed to the same Session Server or use a distributed state (CRDT) that works across servers.

#### Bottlenecks

1. Operational Complexity: OT is hard to implement for complex formatting. 2. WebSocket Connections: OS limits on open files (mitigated by horizontal scaling). 3. Large Doc history: Mitigated by periodic snapshots.

#### Failure Handling

If a client disconnects, they buffer local changes and resync upon reconnection. If a server dies, clients reconnect to a new one and resend unacknowledged ops.

#### Trade-offs

- OT vs CRDT: OT (Centralized, easier for linear text) vs CRDT (Decentralized, better for offline, higher memory overhead).
- Consistency vs Performance: Optimistic UI updates (Local first) for perceived zero-latency.

#### Alternative Approaches

1. Diff/Patch (Gitlike): Rejected because it doesn’t work well for real-time sub-second conflicts. 2. Locking: Rejected because only one person could type at a time.

#### Optimization Tips

- Gzip operation payloads.
- Debounce cursor position updates.
- Use Immutable.js or similar on the front end to manage state efficiently.

#### Interview Tips

- Be ready to explain OT (transforming indexes vs other ops) or CRDT (unique IDs for every character).
- Mention “Optimistic UI”.

#### Common Mistakes

- Implementing a simple “Last Write Wins” (LWW) strategy.
- Neglecting the complexity of offline-to-online transitions.

#### Final Architecture Diagram

```text
Client A \ [Session Server] -> [Redis Pub/Sub]
Client B -- [Load Balancer] -- [Session Server] -> [Cassandra (Ops)]
Client C / [Session Server] -> [S3 (Snapshots)]
```

#### Complexity Discussion

- Time: Transformation is 𝑂(1) per operation.
- Space: 𝑂(𝑁) where 𝑁 is document size + operation history.

#### Example API Request

```json
{
  "type": "insert",
  "index": 12,
  "char": "a",
  "version": 45
}
```

#### Example API Response

```json
{
  "status": "ACK",
  "newVersion": 46
}
```

### Q41. Design a Massively Multiplayer Online Game (MMOG) backend — Hard

#### Problem Statement

Design a persistent, low-latency backend for a world with thousands of concurrent players in a shared space. The system must synchronize state (position, health, inventory) in real-time while ensuring anti-cheat measures and fault tolerance.

#### Functional Requirements

- Real-time movement and state synchronization (<100ms lag).
- Persistent player profiles, inventories, and progress.
- Matchmaking and sharding of game worlds (Rooms/Zones).
- Global chat and social features (Friends/Guilds).
- Anti-cheat verification on the server side (Authoritative Server).
- Combat/Interaction resolution.

#### Non-functional Requirements

- Latency: Ultra-low latency for state updates (UDP/WebSockets).
- Consistency: Eventual consistency for social; Strong consistency for inventory/transactions.
- Availability: High availability for the lobby; session-based availability for game instances.
- Scale: Support 1M+ concurrent users (CCU) globally.
- Durability: Zero data loss for in-game purchases and character progress.

#### Scale Estimation

- 10M DAU, 1M CCU.
- Movement updates: 20Hz (20 ticks/sec).
- Write QPS: 1M CCU * 20 updates = 20M updates/sec (handled in-memory, persisted periodically).
- Storage: 10M players * 100KB/profile = 1TB (Metadata).

#### API Design

- POST /v1/auth/login: Authenticate and get Session Token.
- GET /v1/ matchmaking/join: Queue for a game session; returns Game Server IP + Port.
- GET /v1/ player/profile: Fetch stats/inventory.
- WebSocket/UDP /game/stream: Bidirectional stream for real-time state.

#### Database Design

- Postgres (Global): User accounts, Wallets, Inventories (ACID compliant).
- Redis (Regional): Session data, Leaderboards, World State snapshots.
- DynamoDB (Logging): Game logs, Audit trails, Combat history.

#### High-Level Architecture

A distributed microservices architecture where a “Lobby Service” handles metadata via REST, while a fleet of “Game Servers” handles the heavy lifting of spatial simulation over UDP/WebSockets, coordinated by a “State Manager.”

#### Detailed Components

- Game Server (Spatial Shard): Authoritative engine (Unity/Unreal headless) running the physics loop.
- Matchmaker: Pairs players based on EMR/Rank and Latency/Region.
- Pub/Sub (Redis): Broadcasts world events (e.g., “Boss Spawned”) to all servers in a cluster.
- Gateway Service: Handles connection multiplexing and protocol translation (TCP/UDP).
- Background Worker: Periodically persists transient game state from memory to DB.

#### Data Flow

1. Write (Movement): Client sends UDP packet (Input) → Game Server validates physics → Server updates local memory → Server broadcasts new state to nearby players via Area of Interest (AoI) algorithm
2. Read (Inventory): Client requests /v1/inventory → API Gateway → Inventory Service → Postgres → Result cached in Redis

#### Caching Strategy

- Hot Data: Player stats and location are cached in-memory on the specific Game Server.
- Global Cache: Redis for leaderboards and friend statuses (TTL 5m).
- Local Cache: Client-side prediction for immediate visual feedback.

#### Database Choice

Use Postgres for persistent storage because transactional integrity is non-negotiable for virtual economies. However, the “live” database for world state must be Redis or a custom in-memory store because disk I/O is too slow for 20Hz updates.

#### Message Queue Usage

Kafka is used for asynchronous tasks: processing loot drops, analytics, and syncing data to the search index (Elasticsearch) for player discovery.

#### Storage Strategy

S3 for static assets (textures, models) and game binaries. Cassandra for high-volume logs (client-side telemetry) to debug crashes and detect hacks.

#### Security

- TLS for all API traffic.
- DTLS/Encryption for game traffic.
- Serverside validation: Client only sends “Inputs” (keys pressed), Server calculates “Resulting State” to prevent speed-hacks.
- Rate limiting on API gateways.

#### Scaling Strategy

Horizontal scaling of Game Server nodes. Use Kubernetes with Agones to manage dedicated game server lifecycles, spinning up instances based on CCU demand in specific geographic regions.

#### Bottlenecks

1. Network Jitter: Mitigate by using UDP and Interpolation/Extrapolation on the client. 2. Database Write Contention: Mitigate by batching in-memory state updates and flushing to DB every 30-60 seconds. 3. CrossShard Communi cation: Mitigate by minimizing player movement between shards.

#### Failure Handling

- Game Server Crash: Players in that session are disconnected; matchmaker re-routes (Session state lost unless periodically checkpointed).
- Data base Down: Read-only mode via replicas; queue writes in Kafka.

#### Trade-offs

- UDP vs TCP: Chose UDP for speed despite packet loss (Packet loss > Latency in MMOGs).
- Consistency vs Latency: Chose Eventual Consistency for movement to maintain performance.
- Sharding: Chose Spatial Sharding vs Instancing; complicates cross-shard visuals but improves density.

#### Alternative Approaches

1. PeertoPeer (P2P): Rejected due to security risks (cheating) and NAT traversal complexities. 2. Single Monolithic World: Rejected; doesn’t scale horizontally. Must use “Channels” or “Instances.”

#### Optimization Tips

- Area of Interest (AoI): Only send data to players about objects they can see/hear.
- Delta Compression: Only send what changed since the last tick.
- Protocol Buffers: Use binary serialization to reduce packet size.

#### Interview Tips

- Focus on the “Tick Rate” and its impact on performance.
- Explain the difference between a “Matchmaker” and a “Game Server”.
- Highlight how to handle the “Dead Reckoning” algorithm.

#### Final Architecture Diagram

```text
Client (Player) <--> Load Balancer (UDP/TCP)
|
[Matchmaker] <--> [Lobby/API Service] <--> [Postgres]
|
[Game Server Cluster (Agones/K8s)] <--> [Redis Cache]
|
[Kafka] --> [Analytics/Logs] --> [S3/BigQuery]
```

#### Complexity Discussion

- Time: O(N log N) for spatial partitioning (Quadtrees) to find nearby players.
- Network: O(N*M) where N is players and M is updates per sec; optimized by AoI to O(N).

#### Example API Request

```http
POST /v1/matchmaking/join

{
  "player_id": "u-9982",
  "region": "us-east-1",
  "game_mode": "battle_royale"
}
```

#### Example API Response

```json
{
  "status": "success",
  "server_ip": "13.251.1.45",
  "port": 7777,
  "token": "xkx-992-secret"
}
```

### Q42. Design an Online Judge (LeetCode-like) — Hard

#### Problem Statement

Design a system that accepts user-submitted code in multiple languages, executes it against test cases in a secure, isolated environment, and returns results (Pass/Fail, Time, Memory). It must handle high bursts during contests.

#### Functional Requirements

- Support for multiple languages (Java, Python, C+ +, Go).
- Secure code execution (sandboxing).
- Real-time feedback via WebSockets/ Polling.
- Problem management and test case storage.
- Contest leaderboard (realtime updates).

#### Non-functional Requirements

- Security: Prevent RCE (Remote Code Execution) and network access from sandbox.
- Scalability: Handle 100k+ submissions during a 1-hour contest.
- Isolation: One user’s code shouldn’t affect another’s performance.
- Durability: Submissions must be saved for history.

#### Scale Estimation

- 1M registered users, 50k DAU.
- Contest peak: 500 submissions/sec.
- Execution time: ~2-5s per submission.
- Workers: 500 sub/s * 5s = 2500 concurrent sandbox containers needed.

#### API Design

- GET /problems: List problems.
- POST /submit: Submit code (Language, Code, ProblemID). Returns submission_id.
- GET /submission/{id}: Polling for result.

#### Database Design

- Postgres: Users, Problems, Submissions (Metadata, status, language).
- S3: Large test case files (Input/Output).
- Redis: Leaderboards (Sorted Sets), Submission Queue.

#### High-Level Architecture

Users submit code via an API Gateway. The Submis sion Service saves the entry and pushes a task to a Message Queue. Distant Judge Workers pull tasks, download test cases, execute code in Docker/gVisor, and update the DB/Redis.

#### Detailed Components

- Judge Worker: A stateless node that pulls code, initializes a sandbox, and runs it.
- Sandbox (gVisor/Nsjail): Prevents system call abuse and limits CPU/RAM/Disk.
- Test Case Manager: Caches hot test cases locally on workers to avoid S3 calls.
- WebSocket Server: Pushes execution status (Compiling -> Running -> Finished).

#### Data Flow

1. Write: User hits /submit → App Service writes to Postgres (PENDING) - > Pushes to RabbitMQ
2. Execution: Worker picks message → Downloads test cases → Runs in Sandbox → Validates Output → Updates Postgres (ACCEPTED)
3. Read: User’s browser receives WebSocket update or polls /submission/{id}

#### Caching Strategy

- Redis: Problem metadata and Top 100 leaderboard.
- Worker Local Disk: LRU cache for frequently used Docker images and test cases.

#### Database Choice

Postgres is perfect for relational data (User-Problem mapping). Redis is essential for the “Contest Leaderboard” using ZADD and ZRANGE for O(log N) ranking.

#### Message Queue Usage

RabbitMQ or Kafka to decouple the API from the slow execution process. It acts as a buffer during contest spikes.

#### Storage Strategy

S3 for code files and large test inputs. Metadata stays in DB.

#### Security

- Isolation: Use gVisor (Google’s container sandbox) to intercept syscalls.
- Resource Limits: cgroups for CPU/RAM limits.
- Network: Disable NIC within the sandbox.
- Timeouts: Kill process after 𝑋 seconds to prevent infinite loops.

#### Scaling Strategy

Auto-scale Judge Workers based on Queue Depth (Backlog). If queue > 1000, spin up more EC2/K8s nodes.

#### Bottlenecks

1. I/O Bound Sandboxes: Reading large test cases. (Mitigation: Local SSD caching). 2. Database Write Lock: During contests. (Mitigation: Batch leaderboard updates). 3. Cold Start: Pulling Docker images. (Mitigation: Pre-warm worker nodes).

#### Failure Handling

- Worker Fails: Message is not ACKed in RabbitMQ; another worker picks it up.
- Sandbox Crash: Worker logs the error, marks submission as “Internal Error,” and resets.

#### Trade-offs

- Synchronous vs Asynchronous: Chose Async for better UX and system stability.
- Container vs VM: Containers (Docker) are faster to spin up but less secure than VMs; added gVisor to bridge the gap.

#### Alternative Approaches

1. Serverless (AWS Lambda): Rejected due to 500MB limit, slow cold starts, and cost at scale. 2. Raw Process Execution: Rejected; way too insecure (a user could rm -rf /).

#### Optimization Tips

- Pre-compile code before entering the sandbox to save time.
- Use a “Pipe” to stream test cases instead of loading all into memory.
- Zstd compression for test cases in transit.

#### Interview Tips

- Focus heavily on the Security aspect (Sandboxing).
- Explain how the Leaderboard handles ties (e.g., time-based penalties).
- Mention Idempotency in the Judge Worker.

#### Final Architecture Diagram

```text
Web Client --(WebSocket/HTTPS)--> API Gateway
|
[Postgres] <--- [Submission Service] ---> [Redis Leaderboard]
|
[Message Queue]
|
[S3] <---- [Judge Worker (gVisor)] ----> [Update DB]
```

#### Complexity Discussion

- Time: Code execution is O(T) where T is the problem’s time limit.
- Space: Limited by Cgroups.
- Queue: O(N) where N is the number of pending submissions.

#### Example API Request

```json
{
  "problem_id": 101,
  "language": "python3",
  "code": "print(sum(map(int, input().split())))"
}
```

#### Example API Response

```json
{
  "submission_id": "sub_77821",
  "status": "QUEUED"
}
```

### Q43. Design a Ticket Booking System (BookMyShow) — Hard

#### Problem Statement

Design a system for booking movie/event tickets across multiple theaters. It must handle high concurrency during blockbuster releases, prevent double-booking, and manage seat locks during the checkout process.

#### Functional Requirements

- Search movies by city/date.
- Select theater, showtime, and seats.
- Temporary seat locking (5-10 mins).
- Payment integration and booking confirmation.
- Ticket generation (QR codes).

#### Non-functional Requirements

- Consistency: Strict consistency for seat selection (No double-booking).
- Latency: Fast search and UI updates.
- Scale: Handle 100k requests/sec during peak sales.
- Durability: Bookings must survive system failures.

#### Scale Estimation

- 10M DAU.
* 1000 screenings/day per city * 100 cities = 100k screenings/day.
- Peak: New Marvel movie release (50k bookings/min).

#### API Design

- GET /movies?city=NY: List current movies.
- GET /showtimes? movie_id=1&date=2023-10-10: List theaters.
- POST /bookings/lock: Request seat lock (TheaterID, ShowID, SeatIDs). Returns LockID.
- POST /bookings/confirm: Finalize payment and book.

#### Database Design

- Postgres: Theaters, Movies, Showtimes, Bookings.
- Redis: Seat Map (Bitmaps or Hashes) for real-time availability and locking.

#### High-Level Architecture

Users browse via a Search Service (backed by Elasticsearch). Once a seat is selected, a Booking Service uses Redis to place a distributed lock. If payment succeeds, the Payment Service triggers the Ticket Service to persist the record in Postgres.

#### Detailed Components

- Lock Manager (Redis): Uses SETNX or Redlock to hold seats for 10 minutes.
- Search Service: Elasticsearch for typo-tolerant movie searches.
- Payment Gateway: Third-party integration (Stripe/Paypal).
- Worker (Expiration): Clears expired locks if payment isn’t completed.

#### Data Flow

1. Locking: User selects Seat A1 → App checks Redis → SET seat:A1:show:123 user:55 EX 600 NX → Success
2. Booking: User pays → App calls / confirm → Transaction begins: Update Bookings table, update Seats table, delete Redis lock → Commit

#### Caching Strategy

- CDN: Movie posters and static UI parts.
- Redis: Seat availability (Read-heavy, updated on lock/unlock).
- Local Cache: Popular movie details for the homepage.

#### Database Choice

Postgres with SERIALIZABLE isolation or SELECT FOR UPDATE is crucial for the final booking to prevent race conditions. Redis is used for the high-frequency “Locking” phase to offload the DB.

#### Message Queue Usage

Kafka to send confirmation emails/SMS and to sync Postgres data to Elasticsearch.

#### Storage Strategy

Postgres for all relational data. ActiveMQ for coordinating long-running payment tasks.

#### Security

- HTTPS/TLS.
- Rate limiting by IP/User to prevent bot-scraping of seats.
- Checksum verification for payment callbacks.

#### Scaling Strategy

Database sharding by ShowID or CityID. Read replicas for the Search Service. Redis Cluster for handling seat-map traffic.

#### Bottlenecks

1. Database Deadlocks: Multiple users trying to lock seats in the same transaction. (Mitigation: Lock in Redis first). 2. Payment Latency: Users waiting for 3rd party. (Mitigation: Webhook-based async updates). 3. Thundering Herd: Million users hitting the same ShowID. (Mitigation: Request collapsing at LB level).

#### Failure Handling

- Redis Fails: Backend falls back to Postgres (slower but safe).
- Payment Fails: Booking Service releases the seat lock immediately.

#### Trade-offs

- Availability vs Consistency: Chose Consistency (CP in CAP) for bookings. Better to fail a request than sell one seat twice.
- Redis vs DB for Locks: Redis is faster but volatile; mitigated by long TTL and DB fallback.

#### Alternative Approaches

1. NoSQL (Cassandra): Rejected because it’s hard to handle multi-row ACID transactions for seat locking. 2. Fully Synchronous Payment: Rejected; it would hold DB connections open too long.

#### Optimization Tips

- Use Redis Bitmaps to represent 200 seats in a theater (saves memory).
- Implement a Waiting Room (Virtual Queue) for ultra-high demand events.
- Server-Sent Events (SSE) to update seat colors in real-time for other users.

#### Interview Tips

- The “Double Booking” problem is the core of this interview. Explain your locking mechanism clearly.
- Discuss how to handle the edge case where a user pays but the system crashes before updating the DB.

#### Final Architecture Diagram

```text
Client -> LB -> Booking Service -> Redis (Locks)
|
Postgres (Master) <-> Search (ES)
|
Payment Gateway -> Kafka -> Notification Service
```

#### Complexity Discussion

- Searching: O(log N) with ESR.
- Locking: O(1) in Redis.
- Booking: O(1) row update in ACID DB.

#### Example API Request

```json
{
  "show_id": "SH-990",
  "seats": [
    "A1",
    "A2"
  ],
  "user_id": "U-123"
}
```

#### Example API Response

```json
{
  "lock_id": "lock_abc123",
  "expires_at": "2023-10-10T12:00:00Z",
  "total_price": 25.0
}
```

### Q44. Design an IoT Platform — Hard

#### Problem Statement

Design a platform to ingest, process, and store telemetry data from billions of devices (sensors, smart home, industrial). The system must support real-time monitoring, historical analysis, and remote device management.

#### Functional Requirements

- Device registration and authentication (MTLS).
- Data ingestion (Telemetry) via MQTT/HTTP/CoAP.
- Real-time rule engine (e.g., “If temp > 40C, send alert”).
- Device “Shadows” (State sync when device is offline).
- Firmware-Over-The-Air (FOTA) updates.

#### Non-functional Requirements

- Scale: Millions of messages per second.
- Availability: High (Industrial sensors shouldn’t lose data).
- Latency: Low for rule triggering (<500ms).
- Storage: Efficient long-term storage for petabytes of timeseries data.

#### Scale Estimation

- 100M devices.
* 1 message per 30 seconds per device.
- QPS: ~3.3M msgs/sec.
- Storage: 3.3M * 100 bytes/msg = 330MB/sec = ~10PB/year.

#### API Design

- POST /v1/devices/register: Onboard new device, return certs.
- GET / v1/devices/{id}/shadow: Get last known/desired state.
- PATCH /v1/devices/{id}/shadow: Update desired state (e.g., turn light off).

#### Database Design

- Postgres: Device metadata, ownership, and configuration.
- Cassandra or InfluxDB: High-volume telemetry (Time-series).
- Redis: Device Shadow storage (Latest state).

#### High-Level Architecture

A fleet of MQTT Brokers handles persistent connections. Messages are pushed to Kafka. Stream Processors (Flink) evaluate rules. Timeseries DBs store raw data, while a Device Shadow Service manages state synchronization using Redis.

#### Detailed Components

- MQTT Broker (EMQX/VerneMQ): Light-weight pub/sub protocol for constrained devices.
- Registry Service: Manages device identities and X.509 certificates.
- Rule Engine: Flink or Spark Streaming for detecting anomalies.
- API Gateway: For mobile apps/web to interact with device shadows via REST.

#### Data Flow

1. Ingestion: Device connects via MQTT → Publishes to telemetry/ topic → Broker forwards to Kafka
2. Processing: Flink consumes from Kafka → Checks rules → If threshold met, calls Alert Service
3. Storage: Time-series writer consumes from Kafka → Batches writes to Cassandra

#### Caching Strategy

- Redis: Stores the “Desired” vs “Reported” state (Shadow).
- Edge Cache: Metadata about devices near the ingestion point to speed up AuthN.

#### Database Choice

Cassandra/ScyllaDB for telemetry because it handles highvolume writes across geographic regions. Redis for shadows because state lookups must be O(1) for millions of devices.

#### Message Queue Usage

Kafka is the backbone. It decouples ingestion from slow storage and allows multiple consumers (Rules engine, Analytics, Storage) to process the same stream.

#### Storage Strategy

Hot data in Cassandra (7 days). Cold data moved to S3/ Parquet for cheap archival and BigQuery/Athena analysis.

#### Security

- mTLS: Mutual TLS with per-device certificates.
- TPM: Support for Hardware Security Modules on devices.
- ACLs: MQTT topics restricted so Device A cannot read Device B’s data.

#### Scaling Strategy

Horizontal scaling of MQTT brokers. Use a Distributed Hash Table (DHT) or a load balancer with “Sticky Sessions” based on DeviceID for the MQTT protocol.

#### Bottlenecks

1. MQTT Connection Limit: Individual servers capping at 500k connections. (Mitigation: Cluster brokers). 2. Largescale Certificate Revocation: (Mitigation: Use CRLs or short-lived certs with OCSP). 3. Write Amplification: In TSDB. (Mitigation: Use downsampling/rollups).

#### Failure Handling

- Broker Down: Devices attempt reconnect with exponential backoff.
- Kafka Lag: Backpressure applied; Flink markers for checkpointing.

#### Trade-offs

- MQTT vs HTTP: MQTT is better for power/bandwidth; HTTP is easier for web-integration.
- StoreandForward: Chose to lose some real-time telemetry over blocking the ingestion pipeline.

#### Alternative Approaches

1. Raw AWS IoT Core: Good for startups, but too expensive for billions of messages at scale. 2. Single SQL DB: Rejected immediately due to write-volume constraints.

#### Optimization Tips

- Topic Tries: Optimize MQTT broker topic matching.
- Pro tobuf/CBOR: Use binary formats instead of JSON for payloads.
- Batching: Never write single points to Cassandra; always batch by partition.

#### Interview Tips

- Explain the “Device Shadow” concept clearly (Desired vs. Reported state).
- Discuss how you handle devices with intermittent connectivity.
- Mention the “Registry” and how you prevent unauthorized devices from connecting.

#### Final Architecture Diagram

```text
Devices --(MQTT/mTLS)--> [MQTT Broker Cluster]
|
[Kafka Cluster]
|
----------------------------------------------
| | | |
[Rule Engine] [TSDB Writer] [Analytics] [Shadow Sync]
| | | |
[Send Alert] [Cassandra] [S3] [Redis]
```

#### Complexity Discussion

- Ingestion: O(1) per message.
- State Sync: O(1) via Redis.
- Rule Matching: O(R) where R is the number of active rules per device.

#### Example API Request

```http
PATCH /v1/devices/dev-99/shadow

{
  "state": {
    "desired": {
      "thermostat_c": 22
    }
  }
}
```

#### Example API Response

```json
{
  "status": "accepted",
  "metadata": {
    "version": 452
  }
}
```

### Q45. Design Google Maps — Expert

#### Problem Statement

Design a global geographic information system (GIS) that provides map rendering, point-of-interest (POI) search, and optimal routing (A* / Dijkstra) between two points, considering real-time traffic.

#### Functional Requirements

- Render map tiles at various zoom levels.
- POI search (restaurants, gas stations) near a location.
- Navigation/Routing (Directions) with traffic awareness.
- User location tracking/ETA updates.

#### Non-functional Requirements

- Latency: Navigation updates < 200ms; Search < 100ms.
- Availability: 99.99% (Crucial for driving).
- Scale: 1B+ users; Petabytes of map data.
- Accuracy: High precision for coordinates and traffic.

#### Scale Estimation

- 1B MAU, 100M DAU.
- Map Data: Globally, millions of road segments.
- QPS Search: 50k/sec.
- QPS Tile Fetching: ~1M/sec.

#### API Design

- GET /v1/tile/{z}/{x}/{y}: Fetch raster or vector map tile.
- GET /v1/search?q=pizza&lat=...&long=...: Spatial search.
- GET /v1/directions? origin=...&dest=...&mode=driving: Get polyline and steps.

#### Database Design

- PostgreSQL with PostGIS: Master data for road segments and POIs.
- Graph Database (Custom/Neo4j): For routing algorithms.
- S3/CDN: For pre-rendered map tiles.
- Redis: Real-time traffic speeds for segments.

#### High-Level Architecture

A Tile Service serves static map data via CDN. A Location Service tracks user probes to calculate traffic. A Search Service uses Elasticsearch/Quadtrees for spatial indexing. A Routing Service runs algorithms on a sharded Road Graph loaded in RAM.

#### Detailed Components

- Quadtree/Google S2: Library to index the earth into hierarchy cells for fast spatial lookup.
- Graph Engine: Shards the world into “sectors” and calculates shortest paths using A
- and hub labeling.
- Traffic Engine: Aggregates GPS pings from users to determine road speeds.
- Tile Renderer: Converts raw road/ building data into Vector/Raster formats.

#### Data Flow

1. Search: User types “Cafe” → Search Service queries S2-indexed Elasticsearch → Filters by proximity → Returns POIs
2. Routing: User requests A to B → Routing Service fetches relevant graph segments → Injects live traffic weights from Redis → Runs A

1. → Returns path.

#### Caching Strategy

- CDN: Cache map tiles at the edge.
- Redis: Cache popular route results (TTL 1 min due to traffic).
- Browser Cache: Vector data for smooth zooming.

#### Database Choice

PostGIS for persistence because of excellent support for geographic primitives. S2 Geometry library in-memory for live operations because standard DBs are too slow for 50k searches/sec.

#### Message Queue Usage

Kafka to ingest billions of “Location Probes” (GPS tracks) from users to be processed by the Traffic Engine.

#### Storage Strategy

S3 for static tiles. Cassandra for historical traffic patterns (e.g., “Monday 8 AM average speed”).

#### Security

- Location privacy (Anonymizing GPS probes).
- TLS for transit.
- API keys for 3rd party developers (Rate limited).

#### Scaling Strategy

Geographic Sharding: Divide the world graph by continents/ countries. Users in London only need the UK graph shard. Use “Edges” to connect shards.

#### Bottlenecks

1. Route Calculation: A* is CPU intensive. (Mitigation: Pre-calculate “Contraction Hierarchies” / Hub labels). 2. Traffic Spikes: (Mitigation: Stream processing of GPS pings to update weights in sub-seconds). 3. Map Data Updates: (Mitigation: Incremental tile updates rather than full re-renders).

#### Failure Handling

- Traffic Service Down: Fall back to historical averages.
- Tile Service Down: Client uses locally cached tiles or basic vector lines.

#### Trade-offs

- Vector vs Raster Tiles: Vector (client renders) saves bandwidth and allows rotation; Raster (server renders) is better for low-power devices.
- Accuracy vs Latency: Approximate routing for very long distances (e.g., NY to LA) to save CPU.

#### Alternative Approaches

1. Strict Dijkstra: Rejected; too slow for large graphs. Must use A* with heuristics. 2. Relational DB for Routing: Rejected; Join operations for graph traversal are exponentially slow.

#### Optimization Tips

- Hub Labeling: Identify “highway exits” as hubs to skip local road calculations for long trips.
- Map Matching: Snap raw/noisy GPS points to the nearest road segment.
- Zorder curves: For efficient 2D to 1D mapping in databases.

#### Interview Tips

- Demonstrate knowledge of S2 Geometry or Geohashes.
- Don’t just say “Dijkstra”; talk about how a global graph is too large for one machine.
- Explain how traffic affects the weight of edges in the graph.

#### Final Architecture Diagram

```text
[Client/Mobile]
| |
[CDN (Tiles)] [LB]
|
----------------------------------
| | |
[Search (S2)] [Routing (Graph)] [Traffic (Kafka)]
| | |
[PostGIS] [RAM/Redis] [Flink Aggregator]
```

#### Complexity Discussion

- Search: O(log N) where N is POIs in a cell.
- Routing: O(E log V) optimized by heuristics and hierarchies.
- Space: O(V + E) for the graph storage.

#### Example API Request

```http
GET /v1/directions?origin=40.71,-74.00&dest=40.75,-73.98&traffic=true
```

#### Example API Response

```json
{
  "distance_meters": 5400,
  "duration_seconds": 720,
  "polyline": "a~l_FjkzsO_hc@..."
}
```

### Q46. Design an LLMserving system (ChatGPT-like) — Expert

#### Problem Statement

Design a system capable of serving large language models (LLMs) to millions of users with low-latency streaming responses. The system must handle long-context windows, stateful conversations, and heavy GPU compute requirements while maintaining high availability.

#### Functional Requirements

- User can send prompts and receive real-time streaming responses via Server-Sent Events (SSE).
- System must maintain conversation history for context-aware follow-up questions.
- Support for multiple model versions (e.g., GPT-4, Llama-3) and A/B testing.
- Support for “stop sequences” and hyperparameter tuning (temperature, top-p).
- User can view/delete past conversation history.

#### Non-functional Requirements

- Low Time-to-First-Token (TTFT < 200ms) and high Tokens-Per-Second (TPS > 30).
- High availability (99.99%) to serve global requests.
- Scalability to handle spikes in traffic during viral events.
- Strict multitenancy and data isolation for user privacy.

#### Scale Estimation

- DAU: 10M users.
- Avg Queries: 5 per user = 50M QPS (daily) -> ~600 QPS average.
- Peak QPS: 1,500.
- Storage: 100 tokens/msg * 2 bytes/token * 10 msgs/day * 10M users = ~20GB/day for history (~7.3TB/year).
- Compute: H100 GPU cluster (assuming 1 GPU serves 2-4 concurrent streams).

#### API Design

- POST /v1/chat/completions: {model, messages, stream: bool, temp} -> returns SSE Stream.
- GET /v1/conversations: returns list of past chat threads.
- GET /v1/ conversations/{id}: returns full message history for a thread.
- DELETE /v1/conversations/ {id}: deletes a specific thread.

#### Database Design

- Conversations (PostgreSQL): id, user_id, title, created_at. Index on user_id.
- Messages (Cassandra/S3): id, conv_id, role (user/assistant), content, tokens, timestamp. Cassandra provides low-latency writes/reads for sequential chat data.
- KV Cache (Redis): Stores intermediate KV-tensors for active sessions to speed up multi-turn inference.

#### High-Level Architecture

Users connect via a Global Load Balancer to a Fleet of API Gateways. The Gateway handles Authn and routes requests to the Inference Orchestrator. The Orchestrator manages the “KV Cache” and interacts with the GPU Inference Cluster (vLLM/TGI) to generate tokens, which are streamed back to the client.

#### Detailed Components

- API Gateway: Handles JWT validation, rate limiting, and SSE connection management.
- Inference Orchestrator: Manages the context window, truncates history if needed, and fetches relevant RAG data or KV-caches.
- GPU Inference Engine: Runs optimized kernels (vLLM with PagedAttention) to maximize throughput and minimize memory fragmentation.
- Tokenization Service: Pre-processes text into integers and ensures length constraints are met before hitting GPUs.
- Metadata DB: Tracks model versions, weights locations in S3, and billing quotas.

#### Data Flow

1. Write (Inference): User sends prompt → Gateway validates - > Orchestrator retrieves history from Cassandra → Prompt + History sent to GPU Worker → GPU streams tokens back to Gateway via gRPC → Gateway pushes SSE to Client
2. Read (History): User opens app → Request goes to History Service - > Fetches thread list from Postgres → Fetches message details from Cassandra → Displays UI

#### Caching Strategy

- Semantic Cache (Redis/Milvus): Cache responses for identical prompts (hit-ratio depends on use-case).
- KV Cache (GPU Memory/Remote Redis): PagedAttention allows sharing KV-caches across tokens to avoid redundant re-computation of prompt prefixes.

#### Database Choice

Use PostgreSQL for relational metadata (users, billing, settings) due to ACID requirements. Use Cassandra for the actual message logs because its LSM-tree structure is optimized for high-volume, time-ordered writes and sequential reads within a conversation partition.

#### Message Queue Usage

Kafka is used for asynchronous tasks: logging tokens for billing, updating search indexes for chat history, and feeding human-feedback data (RLHF) into the fine-tuning pipeline.

#### Storage Strategy

Object Storage (S3) stores massive model weights (checkpoints) and long-term cold archives of conversation logs. NVMe SSDs on GPU nodes are used for local model loading.

#### Security

TLS 1.3 for all transit. Prompt injection filtering via a dedicated guardrail model. Data at rest encrypted via AES-256. PII masking in logs.

#### Scaling Strategy

Implement Model Parallelism (Tensor/Pipeline) for huge models (175B+ params) across multiple GPUs. Use Autoscaling groups based on GPU utilization. Deploy in multi-region clusters to reduce latency.

#### Bottlenecks

1. GPU Memory (VRAM): Mitigated by PagedAttention and Quantization (FP8/INT8). 2. KVCache Networking: Mitigated by keeping conversational state local to a specific GPU pod where possible (session affinity). 3. Cold Starts: Mitigated by pre-warming GPU nodes with model weights in a local cache.

#### Failure Handling

- GPU Node Death: Orchestrator detects heart-beat loss and re-routes request to a healthy node.
- Database Down: Read-only mode for history using Redis cache.
- Quota Service Down: Default to “fail-open” for premium users or “fail-closed” for free users based on config.

#### Trade-offs

1. Stateful vs Stateless: Chose stateful affinity for KV-cache performance over pure stateless simplicity. 2. Consistency vs Latency: Chose eventual consistency for chat history to ensure the inference stream starts immediately. 3. Highprecision vs Quantization: Chose 4/8-bit quantization to double throughput at the cost of slight perplexity degradation.

#### Alternative Approaches

1. Stateless Inference: Re-sending full history every time. Rejected because it wastes TFLOPS and increases TTFT as history grows. 2. Single SQL DB: Using Postgres for everything. Rejected because message volume would require heavy sharding and vacuuming overhead at this scale.

#### Optimization Tips

- Use Speculative Decoding: Use a small model to predict tokens and a large model to verify them in parallel.
- Continuous Batching: Don’t wait for a batch to finish; insert new requests as soon as one request generates its <EOS> token.
- Binary Serialization: Use Protobuf for internal service communication to reduce overhead.

#### Interview Tips

- Focus on the Inference lifecycle: prefill phase vs. decoding phase.
- Explain PagedAttention; it’s the “industry standard” for memory management in this domain.
- Discuss cost: GPUs are expensive, so utilization (MBU) is as important as latency.

#### Common Mistakes

- Treating LLM inference like a standard CRUD REST API (it’s long-lived and heavy on memory/compute).
- Ignoring the size of the KV-cache for long contexts (can be GBs per user).
- Forgetting about rate limiting at the token level, not just the request level.

#### Final Architecture Diagram

```text
Client (SSE) <--> LB <--> API Gateway <--> Orchestrator
| |
Redis (KV/Cache) <-----+ +---> GPU Cluster (vLLM)
| |
Cassandra (History) <--+ +---> S3 (Weights)
```

#### Complexity Discussion

- Inference Time: O(Seq_Length) for decoding.
- Space: O(Layers
- Hidden_Dim
- Seq_Length) for KV-cache.
- Network: High throughput required between Orchestrator and GPU for context loading.

#### Example API Request

```json
{
  "model": "gpt-4-turbo",
  "messages": [
    {
      "role": "user",
      "content": "How does Raft work?"
    }
  ],
  "temperature": 0.7,
  "stream": true
}
```

#### Example API Response

```json
data: {"id": "chat-123", "choices": [{"delta": {"content": "Raft"}, "finish_reason": null}]} data: {"id": "chat-123", "choices": [{"delta": {"content": " is"}, "finish_reason": null}]}
data: [DONE]
```

### Q47. Design a Global Distributed SQL Database (Spanner-like) — Expert

#### Problem Statement

Design a globally distributed relational database that provides external consistency (serializability), high availability, and horizontal scalability across continents using synchronous replication and hardware-assisted time synchronization.

#### Functional Requirements

- Standard SQL support (DDL/DML).
- Global ACID transactions.
- Automatic horizontal sharding (splitting/merging ranges).
- Pointin-time recovery (stale reads at specific timestamps).
- Secondary indexing across shards.

#### Non-functional Requirements

- Consistency: External consistency matching wall-clock time.
- Availability: Five-nines (99.999%) via Paxos/Raft.
- Latency: Lowlatency local reads; cross-region writes bound by speed of light.
- Scale: Petabytes of data, millions of QPS.

#### Scale Estimation

- Nodes: 10,000+ nodes globally.
- Data: 10 PB+.
- Through put: 10M+ Queries Per Second (QPS).
- Write Latency: 50ms - 200ms depending on region count (Paxos round trips).

#### API Design

- ExecuteSQL(query, params, timestamp_bound): Primary interface for queries.
- BeginTransaction(): Returns a transaction ID.
- Commit(): Triggers two-phase commit (2PC) over Paxos.
- Read(table, key_range, columns): Low-level KV-style access.

#### Database Design

- Directory/Split Table: Maps key ranges (table, start_key) to Paxos groups.
- Storage Engine: LSM-Tree based (like RocksDB/LevelDB) for versioned data.
- Columns: Every cell has a versioned timestamp (Key, Column, Timestamp) -> Value.

#### High-Level Architecture

The system consists of Universe Masters (metadata), Placement Drivers, and Spanserver nodes. Data is split into “Fragments” or “Tablets.” Each tablet is managed by a Paxos group replicated across zones/regions. A TrueTime API (using Atomic Clocks/GPS) provides synchronized timestamps.

#### Detailed Components

- Spanserver: Responsible for serving data. Houses hundreds of tablets.
- Paxos Group: Formed per tablet to ensure consensus on writes.
- TrueTime API: Provides TT.now() returning an interval [earliest, latest].
- Transac tion Manager: Coordinates 2PC for cross-tablet transactions.
- Placement Driver: Monitors load and moves tablets between servers to balance RU/storage.

#### Data Flow

1. Read (Strong): Client sends request → Locate leader for the Paxos group → Leader ensures it has latest lease → Returns data
2. Write: Client sends request to Leader → Leader acquires local locks → Assigns timestamp s (must be > TT.now().latest of previous tx) → Proposes to Paxos followers → Once majority acknowledges → Leader waits until TT.now().earliest > s (Commit Wait) → Responds to client

#### Caching Strategy

- Block Cache: In-memory caching of SSTable blocks in Spanservers.
- Row Cache: Caches frequently accessed rows.
- Metadata Cache: Client-side caching of key-range to Spanserver mapping.

#### Database Choice

This is a database design. The underlying storage is an LSM Tree because it supports high-throughput versioned writes and efficient range scans required for distributed SQL.

#### Message Queue Usage

Not used in the critical path. Used for background tasks like internal audit logging or replicating changes to external downstream systems (Change Data Capture - CDC).

#### Storage Strategy

Colossus (Distributed File System) or equivalent (HDFS/ S3). Data is stored in sorted files (SSTables). This decouples compute from storage, allowing rapid recovery if a Spanserver fails.

#### Security

Mutual TLS between all nodes. Fine-grained IAM for database users. Transparent Data Encryption (TDE) for files on disk.

#### Scaling Strategy

Automatic Splitting: When a tablet exceeds a size (e.g., 2GB) or load threshold, it splits. Paxos Reconfiguration: Dynamically add replicas to a group to increase read capacity.

#### Bottlenecks

1. Speed of Light: Cross-region Paxos is slow. Mitigation: Follower reads for stale data. 2. Clock Drift: Large TT.now() intervals increase Commit Wait time. Mitigation: High-precision hardware (Atomic clocks). 3. Singlekey Con tention: Hotspots in a single Paxos group. Mitigation: Key salting or automatic load splitting.

#### Failure Handling

- Follower Failure: Paxos continues as long as a majority is up.
- Leader Failure: New leader is elected via Paxos after lease expiry.
- Total Region Outage: Remaining regions form a majority and continue serving.

#### Trade-offs

1. Latency vs Consistency: Chose strict serializability, incurring 2PC/ Paxos latency. 2. Hardware vs Software Time: Used hardware (Atomic clocks) to solve the distributed clock problem instead of pure software logical clocks (which lack global linearity). 3. LSM vs BTree: Chose LSM for better write performance and versioning support.

#### Alternative Approaches

1. CockroachDB approach: Uses HLC (Hybrid Logical Clocks) instead of hardware. Pros: Runs on commodity hardware. Cons: Slightly higher risk of consistency violations if clocks drift too far. 2. Async Replication (MySQL/Postgres): Pros: Ultra-fast writes. Cons: Risk of data loss and lack of global consistency.

#### Optimization Tips

- Snapshot Reads: Allow reads at a past timestamp to avoid locking and Paxos overhead.
- Leader Leases: Leaders don’t need Paxos for every read if they hold a valid time-based lease.
- Locality Groups: Keep related tables in the same Paxos group to avoid 2PC.

#### Interview Tips

- Explain the Commit Wait rule; it’s the core of how Spanner achieves external consistency.
- Know the difference between TwoPhase Commit (for atomicity) and Paxos (for availability).
- Mention that Spanner is “semi-relational” (interleaved tables).

#### Common Mistakes

- Confusing Paxos with 2PC.
- Ignoring the time sync problem and assuming NTP is sufficient.
- Failing to explain how keys are mapped to servers (Sharding).

#### Final Architecture Diagram

```text
Client <--> LB <--> Spanserver (Leader) <--> Paxos Group (Followers)
| |
TrueTime API (GPS/Atomic) |
| |
Colossus (Distributed Storage / SSTables)
```

#### Complexity Discussion

- Read: O(1) network hop to leader + O(log N) LSM-tree lookup.
- Write: O(Paxos Round Trip) + O(Commit Wait) + O(1) LSM-tree append.
- Space: O(Data
- Replication Factor).

#### Example API Request

```json
-- Client Request BEGIN; UPDATE BankAccounts SET balance = balance - 100 WHERE id = 'A'; UPDATE BankAccounts SET balance = balance + 100 WHERE id = 'B'; COMMIT;
```

#### Example API Response

```json
{
  "status": "committed",
  "commit_timestamp": "2023-10-27T10:00:00.000005Z",
  "affected_rows": 2
}
```

### Q48. Design an AdServing / RealTime Bidding System — Expert

#### Problem Statement

Design a high-concurrency, ultra-low latency system that auctions ad space to bidders (DSPs) and serves the winning ad within 100ms. The system must handle millions of bid requests per second and maintain accurate budgets.

#### Functional Requirements

- Receive ad requests from publishers/SDKs.
- Conduct a Real-Time Bidding (RTB) auction with multiple DSPs.
- Select winning ad based on price and relevance.
- Track impressions, clicks, and conversions.
- Manage advertiser budgets and pacing.

#### Non-functional Requirements

- Latency: Ultra-low (< 100ms total round-trip for auctions).
- Throughput: 1M+ QPS.
- Availability: 99.99% (Ads offline = direct revenue loss).
- Consistency: Eventual consistency for reporting, but strict for budget capping.

#### Scale Estimation

- 1M QPS auction requests.
* 10 bidders per auction = 10M outgoing requests/sec.
- Storage: 50TB/day of raw logs (Impressions/Clicks).
- Budget: Real-time updates across distributed nodes.

#### API Design

- GET /ad/request: {slot_id, user_id, device_info, context}.
- POST / bid/response: {auction_id, price, ad_markup, creative_id}.
- GET /ad/pixel/impression: {auction_id, timestamp}.
- GET /ad/pixel/click: {auction_id, redirect_url}.

#### Database Design

- Inventory/Campaigns (Postgres/MySQL): Metadata about ads, targeting rules.
- Budgets (Redis): Atomic counters for fast decrementing.
- User Profiles (Aerospike/DynamoDB): Key-value store for user interests/segments.
- Reporting (ClickHouse/Druid): OLAP for heavy aggregation of impressions/clicks.

#### High-Level Architecture

Ad requests hit an Ad Exchange (Ad Server). The Ad Server fetches user data, identifies relevant campaigns, and sends Bid Requests to external DSPs (Demand Side Platforms). The winner is selected, the creative is served, and tracking pixels trigger asynchronous updates to the budget and reporting systems via Kafka.

#### Detailed Components

- Bidder Proxy: Manages gRPC/HTTP connections to hundreds of external DSPs with strict timeouts.
- Budget Service: High-speed service using Redis LUA scripts to ensure an advertiser doesn’t spend more than their daily limit.
- Fraud Detection: Real-time filter for bot traffic.
- User Store: Low-latency (<2ms) store for user cookie-mapping and segments.
- Pixel Tracker: Lightweight endpoint that absorbs impression pings.

#### Data Flow

1. Auction: User opens page → Ad Server → Fetch User Profile → Send parallel Bid Requests to DSPs (deadline 50ms) → Collect bids → Execute Auction Logic → Return winning Ad Markup
2. Tracking: User views ad → Browser hits Pixel Tracker → Tracker writes event to Kafka → Consumer updates Budget (Redis) → Consumer writes to OLAP (ClickHouse)

#### Caching Strategy

- Campaign Metadata: Cached in-memory on every Ad Server instance (refreshed every 30s).
- User Segments: Cached in Aerospike.
- Negative Cache: Cache “No-Ad-Available” for specific slot/user combos for 1 minute.

#### Database Choice

Aerospike for user profiles due to its sub-millisecond highpercentile latency. ClickHouse for reporting because it can ingest millions of rows per second and perform sub-second analytical queries.

#### Message Queue Usage

Kafka is the backbone. It decouples the critical auction path from the slow billing and reporting paths. Every impression and click is a Kafka message.

#### Storage Strategy

S3 + Spark/Flink for long-term storage and reprocessing of logs. Raw logs are stored in Parquet format for efficient columnar access.

#### Security

OpenRTB protocol standards. HMAC signatures for tracking pixels to prevent “impression stuffing.” Rate limiting per publisher ID.

#### Scaling Strategy

Geosharding: Deploy Ad Servers in AWS/GCP regions closest to users and DSPs to minimize speed-of-light delays. Use Local Redis instances or Sidecars for budget lookups to avoid network hops.

#### Bottlenecks

1. Network Overhead: Calling 20 DSPs in 50ms. Mitigation: Persistent gRPC connections and early-exit on timeout. 2. Budget Overrun: Propagation delay of budget exhaustion. Mitigation: “Pacing” algorithms that throttle requests as budget nears 0. 3. Data Volume: 1M QPS generates massive logs. Mitigation: Probabilistic sampling for non-billing logs.

#### Failure Handling

- DSP Timeout: If a DSP is slow, the auction proceeds without their bid.
- Redis Down: Default to “Ad-Serving-Off” or use a local backup cache of the last known budget state.
- Kafka Lag: Reporting is delayed, but ad-serving (revenue) continues.

#### Trade-offs

1. Accuracy vs Latency: Used Redis for budgets (fast but potentially misses a few cents if Redis fails) vs a slow SQL Tx. 2. Breadth vs Depth: Sending bid requests to many DSPs (more revenue) vs fewer DSPs (lower latency). 3. Realtime vs Batch Reporting: Real-time is needed for budget, Batch/Micro-batch for billing accuracy.

#### Alternative Approaches

1. Clientside Header Bidding: Moves the auction to the browser. Pros: Reduced server costs. Cons: Higher page latency, less control. 2. SQL for Budgets: Rejected because Postgres cannot handle 1M updates/sec on a single “budget” row without locks.

#### Optimization Tips

- Connection Pooling: Maintain warm connections to DSPs.
- UDP for Pixels: For non-critical tracking to save TCP handshake overhead (rarely used now, but interesting).
- Kernel Tuning: Tune the TCP stack (sysctl) on Ad Servers for high connection churn.

#### Interview Tips

- Emphasize the 100ms constraint; it drives every architectural decision.
- Talk about Pacing; advertisers hate spending their $1000 daily budget in the first 5 minutes.
- Discuss OpenRTB; it shows industry knowledge.

#### Common Mistakes

- Trying to use a standard REST framework with high overhead (Spring/Django) for the auction path.
- Not accounting for the “Long Tail” of DSP response times.
- Forgetting that impressions happen after the auction.

#### Final Architecture Diagram

```text
SDK/Web Client --> Ad Exchange (LB)
|
+------------+------------+
| | |
User Store Budget (Redis) DSP 1, 2, 3... (RTB)
| |
+---- Kafka <+
|
ClickHouse / S3
```

#### Complexity Discussion

- Time: Auction is O(N) where N is # of bidders, but parallelized to O(max_timeout).
- Space: O(U) where U is number of active users in the user profile store.
- Network: Massive egress bandwidth to external bidders.

#### Example API Request

```json
{
  "id": "auc_987",
  "imp": [
    {
      "id": "1",
      "banner": {
        "w": 300,
        "h": 250
      }
    }
  ],
  "device": {
    "ua": "Mozilla/5.0...",
    "ip": "1.2.3.4"
  },
  "user": {
    "id": "user_456"
  }
}
```

#### Example API Response

```json
{
  "id": "auc_987",
  "bidid": "bid_111",
  "seatbid": [
    {
      "bid": [
        {
          "price": 2.5,
          "adm": "<script src='...'></script>"
        }
      ]
    }
  ]
}
```

### Q49. Design a Realtime Collaborative Whiteboard (Miro-like) — Expert

#### Problem Statement

Design a system that allows thousands of concurrent users to draw, move shapes, and add text to a shared infinite canvas with real-time synchronization and conflict resolution.

#### Functional Requirements

- Users can draw shapes, lines, and text.
- Real-time cursor tracking for all active users.
- Object persistence (save/load boards).
- Infinite canvas scaling and zooming.
- Undo/Redo functionality across collaborating users.

#### Non-functional Requirements

- Latency: < 50ms for local feedback, < 100ms for remote updates.
- Consistency: Eventual consistency with Conflict-free Replicated Data Types (CRDTs) or Operational Transformation (OT).
- Scalability: Support 10,000+ concurrent users on a single popular board.
- Durability: No lost strokes even if connection drops.

#### Scale Estimation

- DAU: 1M.
- Concurrent per board: 2 to 10,000 (e.g., a public workshop).
- Update Frequency: 20-50 messages/sec per active drawer (cursor moves).
- Payload: Tiny (JSON/Binary coordinates).

#### API Design

- WebSocket /ws/board/{id}: Binary protocol (Protobuf) for all real-time ops.
- GET /api/v1/boards/{id}: Fetch initial state (snapshot).
- POST /api/v1/boards: Create new board.
- GET /api/v1/boards/{id}/history: Fetch previous versions.

#### Database Design

- Board Metadata (Postgres): id, owner_id, permissions, created_at.
- Board State (S3 + Redis): Snapshot of the board (S3) and active operations (Redis).
- Object Store (MongoDB/DynamoDB): To store individual elements: board_id, element_id, type, coords, color, version.

#### High-Level Architecture

Clients connect via WebSockets to a Pub/Sub Cluster (Redis/Nats). Each board is assigned a Room Server (or Topic). Operations are synchronized using CRDTs (e.g., LWW-Element-Set or Automerge) to handle conflicts without a central master.

#### Detailed Components

- WebSocket Gateway: Manages persistent connections and heartbeat.
- Collaboration Engine: Implements CRDT logic to merge incoming patches.
- Snapshot Service: Periodically squashes the operation log into a single JSON/Binary file stored in S3.
- Presence Service: Tracks who is online and their cursor coordinates (ephemeral).
- Asset Service: Handles image uploads to the board (S3).

#### Data Flow

1. Update Path: User moves shape → Client generates CRDT operation → Sent via WebSocket → Server broadcasts to all other users in board room → Clients merge op into local state
2. Persistence Path: Server batches operations → Every 5 seconds, writes to MongoDB → Every 5 minutes, triggers a full Snapshot to S3

#### Caching Strategy

- Current Board State: Kept in-memory on the Room Server and cached in Redis for fast recovery.
- User Sessions: Redis-backed session management.

#### Database Choice

MongoDB is excellent here because whiteboard elements are polymorphic (shapes, text, arrows) and can be represented as flexible BSON documents. Its high-throughput write capabilities support many clients updating many objects.

#### Message Queue Usage

Redis Pub/Sub or Nats for low-latency message broadcasting between WebSocket nodes. Kafka for the snapshotting and analytics pipeline (less time-sensitive).

#### Storage Strategy

S3 for “Checkpoints” (snapshots). If a board has 1 million operations, we don’t want to replay them all; we load the 990,000th op snapshot and apply the last 10,000 ops.

#### Security

WebSocket Auth via JWT. Per-board ACLs. Rate-limiting ops per user to prevent “whiteboard scrubbing” attacks.

#### Scaling Strategy

Sharding by Board ID: All users of Board A go to the same set of WebSocket nodes. Use a Distributed Pub/Sub to allow these nodes to sync. For 10k users on one board, use a “tree-based” broadcast to avoid overloading a single node.

#### Bottlenecks

1. N^2 Broadcast: 10k users moving cursors = 10k messages * 10k recipients = 100M msgs/sec. Mitigation: Throttling cursor updates (10Hz) and spatial interest management. 2. Conflict Resolution: OT is complex to scale. Mitigation: Use CRDTs which are commutative and easier to merge asynchronously. 3. Memory Leakage: Long-lived WS connections. Mitigation: Strict TTLs and client-side reconnection logic.

#### Failure Handling

- WS Node Crash: Client reconnects, hits LB, routed to new node, fetches latest snapshot + missed ops from Redis.
- Network Partition: CRDTs allow users to continue drawing offline and merge once reconnected.

#### Trade-offs

1. CRDT vs OT: CRDTs (Easier scaling, higher memory) vs OT (Centralized, complex, lower memory). Chose CRDT for robust offline support. 2. WebSockets vs HTTP Long Polling: WS is lower overhead for high-frequency cursor moves. 3. Immediate vs Periodic Persistence: Periodic to save DB IOPS.

#### Alternative Approaches

1. Firebase: Great for MVP, but lacks custom merge logic and becomes expensive at 10M DAU. 2. Stateless REST: Too much overhead for 50ms latency requirements.

#### Optimization Tips

- Binary Formatting: Use Protobuf instead of JSON for WebSocket payloads.
- Delta Compression: Only send the changed properties of a shape.
- Canvas Layering: Render the background once; only re-render active/ moving elements.

#### Interview Tips

- Distinguish between State Sync (shapes) and Ephemeral Sync (cursors).
- Know the basics of CRDTs (Commutative, Associative, Idempotent).
- Talk about the rendering loop on the client (requestAnimationFrame).

#### Common Mistakes

- Trying to use a standard RDBMS for every cursor movement.
- Not handling the “10k users on one board” bottleneck.
- Ignoring undo/redo (it’s hard in a collaborative environment).

#### Final Architecture Diagram

```text
Clients <--(WS/Binary)--> WS Gateway Fleet
|
Redis Pub/Sub (Broker)
|
+----------------------+----------------------+
| | |
Snapshot Svc (S3) State DB (MongoDB) Presence (Redis)
```

#### Complexity Discussion

- Merge Time: O(1) for most CRDTs.
- Network: O(Users
- Frequency) per board.
- Storage: O(Elements
- History_Depth).

#### Example API Request

```json
{
  "type": "UPDATE_ELEMENT",
  "boardId": "board_123",
  "data": {
    "id": "shape_456",
    "x": 100,
    "y": 250,
    "version": 42
  }
}
```

#### Example API Response

```json
{
  "type": "ACK",
  "msgId": "msg_789",
  "serverTimestamp": 1698412345
}
```

### Q50. Design a Face Recognition Service at Scale — Expert

#### Problem Statement

Design a system that can identify or verify a person’s face against a database of 1 billion faces in under 500ms. The system must handle highvolume uploads and concurrent searches.

#### Functional Requirements

- Face Detection: Locate faces in an image.
- Face Encoding: Convert face to a vector embedding (e.g., 512-dim).
- Face Search (1:N): Find the person in the DB.
- Face Verification (1:1): Confirm the person matches a specific ID.
- Face Indexing: Add new persons to the searchable index.

#### Non-functional Requirements

- Latency: < 500ms for search.
- Accuracy: High Precision/Recall (99.9%+).
- Scale: 1B+ vectors, 1k QPS for Search.
- Durability: Safe storage of original images and embeddings.

#### Scale Estimation

- 1B Faces.
- Vector Size: 512 floats * 4 bytes = 2KB per face.
- Total Index Size: 1B * 2KB = 2TB (must fit in RAM/SSD for speed).
- Search QPS: 1,000.
- Write QPS: 100.

#### API Design

- POST /v1/detect: {image_blob} -> returns bounding boxes.
- POST / v1/search: {image_blob, top_k} -> returns list of {person_id, confidence}.
- POST / v1/identity: {person_id, name, images[]} -> registers a person.
- POST /v1/verify: {image_blob, person_id} -> returns boolean.

#### Database Design

- Metadata (Postgres): person_id, name, date_of_birth, metadata_json.
- Vector Store (Milvus/Pinecone/FAISS): Specialized DB for Approximate Nearest Neighbor (ANN) search.
- Image Store (S3): Raw images for human auditing.

#### High-Level Architecture

An Inference Gateway receives images, sends them to a GPUbased Detection & Feature Extraction service. The resulting embedding is sent to a Distributed Vector Database for ANN search. Results are joined with metadata from Postgres and returned to the client.

#### Detailed Components

- Preprocessing Svc: Resizes, flattens, and normalizes images.
- Extraction Svc: Runs Deep Learning models (DeepFace, ArcFace) on GPUs.
- Vector DB (Milvus): Clusters 1B vectors and uses HNSW or IVF indexes for sub-100ms search.
- Consistency Manager: Ensures when a face is “deleted,” the vector index is updated.
- Audit Logger: Saves the cropped face and search result for regulatory compliance.

#### Data Flow

1. Search Path: Upload Image → GPU Extractor (returns 512-dim vector) → Vector DB (ANN Search) → Top 5 Candidate IDs → Metadata DB (Get names) → Return
2. Indexing Path: Upload Image + ID → GPU Extractor → Append Vector to Milvus Index → Update Postgres Meta → Confirm

#### Caching Strategy

- Embedding Cache (Redis): If the same image hash is uploaded again, skip GPU extraction.
- Metadata Cache: Store frequently searched identities (e.g., VIPs) in Redis.

#### Database Choice

Milvus or Pinecone because they are purpose-built for vector similarity search at the billion-scale. Traditional SQL/NoSQL cannot perform “Cosine Similarity” or “Euclidean Distance” queries efficiently across 1B rows.

#### Message Queue Usage

Kafka facilitates the indexing pipeline. When a person registers, extraction and indexing happen asynchronously so the user doesn’t wait for the index re-clustering.

#### Storage Strategy

Encrypted S3 buckets for raw images. Local NVMe for the Vector DB to ensure high IOPS during index traversal.

#### Security

Biometric Privacy: Anonymize vectors. Antispoofing: Liveness detection models to prevent “photo of a photo” attacks. Encryption: Vectors are useless without the specific model weights, but should still be encrypted.

#### Scaling Strategy

Vector Sharding: Partition the 1B vectors across 50 nodes (20M each). The query is sent to all shards (scatter-gather), and the top results are merged. GPU Horizontal Scaling: Add more worker nodes for the extraction phase.

#### Bottlenecks

1. GPU Cost: Extraction is expensive. Mitigation: Batching requests. 2. Recall/Accuracy Tradeoff: Speeding up ANN search (e.g., via quantization) reduces accuracy. Mitigation: Reranking (search 100 with ANN, then precisely compare top 10). 3. Index Rebuilding: Adding 1M faces can make the index stale. Mitigation: Use “Streaming Indexes” that support incremental inserts.

#### Failure Handling

- Vector Shard Down: Result might be missing the “true” match. Mitigation: Replicate shards (Master-Replica).
- GPU unavailable: Fallback to CPU-based inference (much slower, but keeps service alive).

#### Trade-offs

1. ANN vs Exact Search: ANN is 1000x faster but can miss the perfect match. 2. Onprem vs Cloud: On-prem for privacy/latency vs Cloud for elastic GPU scaling. 3. Local vs Global Embedding: One large vector vs multiple views of the face.

#### Alternative Approaches

1. Elasticsearch with Vector plugin: Good for 1M faces, lacks performance/efficiency for 1B. 2. Binary Hashing: Convert faces to bitstrings (Hamming distance). Pros: Extremely fast. Cons: Significant loss in accuracy compared to float vectors.

#### Optimization Tips

- Product Quantization (PQ): Compress 512-dim vectors to 64 bytes for memory efficiency.
- HNSW Indexing: Use Hierarchical Navigable Small Worlds for the best speed/accuracy trade-off.
- Batch Inference: Process 16-32 images at once on the GPU to maximize TFLOPS.

#### Interview Tips

- Mention ANN (Approximate Nearest Neighbor); it’s the heart of the design.
- Discuss the ColdStart Indexing problem.
- Touch on Privacy and Ethics (GDPR/BIPA); it shows seniority.

#### Common Mistakes

- Using a standard SQL WHERE clause for face matching.
- Ignoring image preprocessing (alignment, lighting correction).
- Assuming GPU memory is infinite.

#### Final Architecture Diagram

```text
Client --(JSON/Bytes)--> LB --> API Gateway
|
+------------------+-------------------+
| |
GPU Inference (Extraction) Metadata DB (Postgres)
| |
Vector DB (Milvus/HNSW) <---------------------+
|
S3 (Raw Images)
```

#### Complexity Discussion

- Extraction: O(Model_Depth
- Image_Pixels).
- Search: O(log N) for HNSW-style ANN search.
- Space: O(Dim
- N) where N is # of faces.

#### Example API Request

```json
{
  "image": "base64...",
  "gallery_id": "global_blacklist",
  "threshold": 0.85,
  "top_k": 3
}
```

#### Example API Response

```json
{
  "matches": [
    {
      "person_id": "p101",
      "score": 0.98,
      "name": "John Doe"
    },
    {
      "person_id": "p202",
      "score": 0.87,
      "name": "Unknown"
    }
  ]
}
```

## Section 3 — Interview Theory Questions

Rapid-fire conceptual comparisons — the depth-check questions interviewers use to separate strong candidates.

### Section 3 theory index

- [T1. Redis vs Memcached](#t1-redis-vs-memcached)
- [T2. Kafka vs RabbitMQ](#t2-kafka-vs-rabbitmq)
- [T3. CAP Theorem in depth](#t3-cap-theorem-in-depth)
- [T4. PACELC Theorem in depth](#t4-pacelc-theorem-in-depth)
- [T5. SQL vs NoSQL](#t5-sql-vs-nosql)
- [T6. Replication vs Sharding](#t6-replication-vs-sharding)
- [T7. REST vs GraphQL](#t7-rest-vs-graphql)
- [T8. REST vs gRPC](#t8-rest-vs-grpc)
- [T9. JWT vs Session cookies](#t9-jwt-vs-session-cookies)
- [T10. OAuth 2.0 vs OpenID Connect](#t10-oauth-20-vs-openid-connect)
- [T11. Horizontal vs Vertical Scaling](#t11-horizontal-vs-vertical-scaling)
- [T12. Docker vs Kubernetes](#t12-docker-vs-kubernetes)
- [T13. HTTP/1.1 vs HTTP/2 vs HTTP/3](#t13-http11-vs-http2-vs-http3)
- [T14. TCP vs UDP](#t14-tcp-vs-udp)
- [T15. Monolith vs Microservices](#t15-monolith-vs-microservices)
- [T16. Load Balancer Algorithms compared](#t16-load-balancer-algorithms-compared)
- [T17. Database Indexing (B-tree vs Hash vs LSM)](#t17-database-indexing-b-tree-vs-hash-vs-lsm)
- [T18. Caching Strategies (aside vs through vs back)](#t18-caching-strategies-aside-vs-through-vs-back)
- [T19. Event-Driven vs Request/Response](#t19-event-driven-vs-requestresponse)
- [T20. Strong vs Eventual Consistency](#t20-strong-vs-eventual-consistency)
- [T21. Optimistic vs Pessimistic Locking](#t21-optimistic-vs-pessimistic-locking)
- [T22. OLTP vs OLAP](#t22-oltp-vs-olap)
- [T23. Row-store vs Column-store databases](#t23-row-store-vs-column-store-databases)
- [T24. Push vs Pull models for feeds](#t24-push-vs-pull-models-for-feeds)
- [T25. WebSockets vs SSE vs Long Polling](#t25-websockets-vs-sse-vs-long-polling)
- [T26. Sync vs Async replication](#t26-sync-vs-async-replication)
- [T27. Leader-based vs Leader-less replication](#t27-leader-based-vs-leader-less-replication)
- [T28. 2PC vs Saga for distributed transactions](#t28-2pc-vs-saga-for-distributed-transactions)
- [T29. CQRS vs traditional CRUD](#t29-cqrs-vs-traditional-crud)
- [T30. Event Sourcing vs State-Based persistence](#t30-event-sourcing-vs-state-based-persistence)
- [T31. Bloom Filter vs Cuckoo Filter](#t31-bloom-filter-vs-cuckoo-filter)
- [T32. Consistent Hashing vs Modulo Hashing](#t32-consistent-hashing-vs-modulo-hashing)
- [T33. Raft vs Paxos](#t33-raft-vs-paxos)
- [T34. Kubernetes vs Docker Swarm vs Nomad](#t34-kubernetes-vs-docker-swarm-vs-nomad)
- [T35. Reverse Proxy vs API Gateway vs Load Balancer](#t35-reverse-proxy-vs-api-gateway-vs-load-balancer)
- [T36. Idempotency vs Exactly-Once Semantics](#t36-idempotency-vs-exactly-once-semantics)
- [T37. Circuit Breaker vs Retry vs Bulkhead](#t37-circuit-breaker-vs-retry-vs-bulkhead)
- [T38. Blue-Green vs Canary vs Rolling Deployments](#t38-blue-green-vs-canary-vs-rolling-deployments)
- [T39. Sticky Sessions vs Stateless Design](#t39-sticky-sessions-vs-stateless-design)
- [T40. gRPC vs GraphQL for Internal APIs](#t40-grpc-vs-graphql-for-internal-apis)

### T1. Redis vs Memcached

#### Explanation

Both are high-performance, in-memory key-value stores used to reduce database load. Redis is a versatile data structure store that supports persistent storage and complex types, whereas Memcached is a simple, multi-threaded, volatile memory caching system designed for high-concurrency string data.

#### Advantages

- Redis: Supports rich data types (Lists, Hashes, Sets), built-in replication, and snapshots (RDB/AOF) for data persistence.
- Memcached: Better utilization of multi-core CPUs due to its multi-threaded architecture, leading to slightly better performance for very simple key-value lookups at extreme scale.

#### Disadvantages

- Redis: Primarily single-threaded for command execution (though 6.0+ introduced IO threads), which can cause head-of-line blocking on long-running commands.
- Memcached: No data persistence (data is lost on restart), lacks native data structures, and has a 1MB limit on value size.

#### Use Cases

- Pick Redis when you need advanced features like Pub/Sub, Geospatial indexes, or persistent sessions.
- Pick Redis when your data needs to survive a server reboot.
- Pick Memcached when you have a massive, straightforward caching layer with very high concurrent read/write traffic and minimal metadata needs.

#### Interview Tips

- Emphasize that “Redis is usually the default choice” today because its feature set outweighs Memcached’s slight speed edge.
- Mention that Memcached is better at handling large metadata overhead because it manages memory more efficiently for simple strings.

### T2. Kafka vs RabbitMQ

#### Explanation

RabbitMQ is a traditional message broker focusing on delivery guarantees and complex routing via exchanges, while Kafka is a distributed event streaming platform designed as a persistent, append-only log for high-throughput data pipelines.

#### Advantages

- Kafka: Extremely high throughput (millions of msgs/sec), long-term data retention (replayability), and sequential disk I/O.
- RabbitMQ: Flexible routing (wildcards, headers), priority queues, and granular “smart” broker logic that tracks message acknowledgments per consumer.

#### Disadvantages

- Kafka: High operational complexity (requires ZooKeeper or KRaft), lacks granular message-level tracking, and consumers must manage their own offsets.
- RabbitMQ: Lower throughput due to the overhead of routing logic and disk syncs, and “dumb” consumers depend on the broker to manage state.

#### Use Cases

- Pick Kafka for log aggregation, real-time analytics, or Event Sourcing where you need to replay history.
- Pick RabbitMQ for simple task queues, background jobs, or complex request-response patterns between microservices.
- Pick Kafka when you have multiple consumers needing the same stream of data independently.

#### Interview Tips

- Differentiate them by “Push vs. Pull”: RabbitMQ pushes to consumers; Kafka consumers pull from the broker.
- Mention that Kafka is “Parallel by Partition,” meaning its scaling is tied to how you shard your data.

### T3. CAP Theorem in depth

#### Explanation

CAP states that in a distributed system, during a network partition (P), you must choose between Consistency (C) — every read receives the most recent write — or Availability (A) — every request receives a response, even if not the latest version. You cannot have all three in a failing network.

#### Advantages

- CP Systems: Ensure data integrity and “Strong Consistency,” which is vital for financial transactions.
- AP Systems: Ensure high uptime and “Eventual Consistency,” which is vital for social media feeds or shopping carts.

#### Disadvantages

- CP Systems: If a node is disconnected, the system returns an error, leading to a poor user experience during outages.
- AP Systems: Users may see stale data (e.g., a deleted comment reappearing), requiring conflict resolution logic.

#### Use Cases

- Pick CP (HBase, MongoDB, Redis) when data correctness is more important than 100% uptime.
- Pick AP (Cassandra, DynamoDB, CouchDB) when it’s better to show old data than no data at all.
- Pick CA (RDBMS) only if you are running on a single non-partitioned node, though in distributed systems, “P” is inevitable.

#### Interview Tips

- Clarify that “P” (Partition Tolerance) is not optional in distributed systems; the real choice is between C and A when things break.
- Mention that modern systems like Google Spanner use atomic clocks to achieve “Effective” CA, though they are technically CP.

### T4. PACELC Theorem in depth

#### Explanation

PACELC extends CAP by describing system behavior during normal operation. It states: if there is a Partition, choose between Availability or Consistency; Else (under normal operation), choose between Latency or Consistency.

#### Advantages

- Latency-focused (EL): Provides faster response times for users by not waiting for all replicas to sync during healthy periods.
- Consistency-focused (EC): Guarantees that users always see the latest data, even when the network is healthy, at the cost of waiting for replicas.

#### Disadvantages

- EL Systems: Risk “dirty reads” during normal operation because a write might not have reached the node being read.
- EC Systems: Higher latency for every single write/read because of the coordination overhead (e.g., multi-phase commits).

#### Use Cases

- Pick PA/EL (DynamoDB, Cassandra) for global-scale apps where speed is the primary feature (e.g., “Likes” on a post).
- Pick PC/EC (BigTable, MariaDB) for systems where data must be perfectly synchronized at all times.
- Pick PC/EL if you need consistency during partitions but want speed during normal times.

#### Interview Tips

- Use PACELC to show you understand that trade-offs exist even when the system is working perfectly.
- Memorize one “Extreme” example: DynamoDB is PA/EL (prioritizes speed/availability above all).

### T5. SQL vs NoSQL

#### Explanation

SQL databases are relational, schema-based, and favor ACID compliance. NoSQL databases are non-relational, often schema-less, and are designed to scale horizontally by sacrificing some consistency for performance and flexibility.

#### Advantages

- SQL: Structured Query Language (SQL) allows for complex joins, strict data integrity, and normalized data (PostgreSQL, MySQL).
- NoSQL: High write throughput, horizontal scaling (sharding), and the ability to store unstructured or semi-structured data (MongoDB, Cassandra).

#### Disadvantages

- SQL: Difficult to scale horizontally; usually requires vertical scaling (bigger RAM/CPU) or complex manual sharding.
- NoSQL: Lack of standard joins makes complex reporting difficult; “eventual consistency” can lead to application-level bugs.

#### Use Cases

- Pick SQL for ERP, CRM, or financial systems where relationships and transactions are complex.
- Pick NoSQL (Document) for content management or catalogs with evolving schemas.
- Pick NoSQL (Key-Value) for caching or real-time heavy-write ingestion like IoT telemetry.

#### Interview Tips

- Avoid saying “SQL doesn’t scale.” It does, but NoSQL scales differently (horizontal vs vertical).
- Mention that many modern DBs are “Hybrid” (PostgreSQL supports JSONB; MongoDB supports ACID transactions).

### T6. Replication vs Sharding

#### Explanation

Replication is the process of copying the same data across multiple nodes to increase availability and read performance. Sharding is the process of partitioning different data across multiple nodes to handle massive datasets and high write loads.

#### Advantages

- Replication: Provides redundancy (fault tolerance) and allows “Read Scaling” by directing queries to replicas.
- Sharding: Enables “Write Scaling” and allows the database to grow beyond the storage limit of a single machine.

#### Disadvantages

- Replication: Does not help with write bottlenecks (since all writes must go to all nodes eventually) and increases storage costs linearly.
- Sharding: Immense operational complexity; joins across shards are slow/impossible, and “hot shards” (uneven data distribution) can break the system.

#### Use Cases

- Pick Replication when you have a read-heavy app (e.g., a blog or news site).
- Pick Sharding when your dataset is too large for one disk (e.g., TBs of user logs).
- Pick both (Sharded Replicas) for global-scale platforms like Twitter or Facebook.

#### Interview Tips

- Always mention “Resilience” for replication and “Throughput/ Capacity” for sharding.
- Discuss “Shard Keys”—choosing a bad shard key (like a timestamp) is a common failure point in system design.

### T7. REST vs GraphQL

#### Explanation

REST is an architectural style based on standard HTTP verbs and resources identified by URLs. GraphQL is a query language for APIs that allows clients to request exactly the data they need in a single request.

#### Advantages

- REST: Highly cacheable (at the HTTP level), easy to understand, and uses standard status codes.
- GraphQL: Eliminates “over-fetching” (getting too much data) and “under-fetching” (needing multiple calls), and provides a strongly typed schema.

#### Disadvantages

- REST: Often requires multiple “round-trips” to different endpoints to load a single page.
- GraphQL: Complexity of implementation, “n+1” query problems on the backend, and difficult HTTP-level caching.

#### Use Cases

- Pick REST for public APIs where simplicity and caching are prioritized.
- Pick GraphQL for complex mobile apps where bandwidth is limited and UI requirements change frequently.
- Pick GraphQL if you are aggregating data from multiple microservices into a single “Gateway.”

#### Interview Tips

- Mention that GraphQL isn’t “better,” it just shifts complexity from the client to the server.
- Talk about “Dataloader” as the standard solution for GraphQL’s performance issues (n+1 problem).

### T8. REST vs gRPC

#### Explanation

REST typically uses JSON over HTTP/1.1 for human-readable communication. gRPC is a high-performance framework developed by Google that uses Protocol Buffers (binary format) over HTTP/2 for efficient machine-to-machine communication.

#### Advantages

- REST: Ubiquitous, easy to debug with browser tools, and supports a wide range of clients.
- gRPC: Significantly faster and smaller payloads, supports bi-directional streaming, and features “Contract-first” development with .proto files.

#### Disadvantages

- REST: JSON is bulky and slow to parse; HTTP/1.1 suffers from head-of-line blocking.
- gRPC: Harder to debug (binary data), poor browser support (requires gRPC-web), and rigid breaking changes in schemas.

#### Use Cases

- Pick gRPC for internal microservice-to-microservice communication where performance is critical.
- Pick REST for your external-facing API that third-party developers will use.
- Pick gRPC for real-time streaming data services (e.g., stock tickers).

#### Interview Tips

- Highlight that gRPC uses HTTP/2 features like multiplexing and header compression.
- Mention that gRPC saves CPU cycles because binary Protobuf is faster to serialize than string-based JSON.

### T9. JWT vs Session cookies

#### Explanation

Session cookies are stateful identifiers stored in the database/server memory; the browser sends the ID, and the server looks up the user. JWT (JSON Web Tokens) are stateless, self-contained tokens where the user information is encoded and signed directly in the token.

#### Advantages

- Session Cookies: Easy to revoke (just delete the session from the DB) and keeps the payload small.
- JWT: Perfectly scalable for microservices because the “Auth Server” doesn’t need to be hit for every request; the server just validates the signature.

#### Disadvantages

- Session Cookies: Requires a central database (like Redis) to store sessions, creating a potential bottleneck in distributed systems.
- JWT: Impossible to revoke before expiration without complex “blacklist” logic, and vulnerable to being “bloated” with too much data.

#### Use Cases

- Pick Session Cookies for traditional monolithic web apps where security/revocation is the priority.
- Pick JWT for Single Page Applications (SPAs) and microservice architectures.
- Pick JWT when building a mobile app that needs to talk to multiple backend domains.

#### Interview Tips

- If asked about security, mention that JWTs are usually stored in localStorage (XSS risk) versus cookies with HttpOnly flags.
- Discuss the importance of “Refresh Tokens” for JWTs to minimize the window of attack for stolen tokens.

### T10. OAuth 2.0 vs OpenID Connect

#### Explanation

OAuth 2.0 is an authorization framework that lets an app get limited access to a user’s account (via an Access Token). OpenID Connect (OIDC) is an authentication layer built on top of OAuth 2.0 that provides information about “who” the user is (via an ID Token).

#### Advantages

- OAuth 2.0: Industry standard for “delegated access” (e.g., letting an app post to your Twitter).
- OIDC: Standardizes the login process and provides a userinfo endpoint, making “Login with Google” consistent across the web.

#### Disadvantages

- OAuth 2.0: On its own, it doesn’t tell you who the user is, only what they are allowed to do.
- OIDC: Adds complexity to the OAuth flow and requires handling ID Tokens and potentially JWT validation.

#### Use Cases

- Pick OAuth 2.0 when you need to give one app permission to access data in another app (API access).
- Pick OIDC when you want to implement “Social Login” (Single Sign-On).
- Pick both when your app needs to know the user’s identity and needs to perform actions on their behalf.

#### Interview Tips

- Use the “Hotel Key Card” analogy: OAuth is the key card (authorization to enter the room), while OIDC is the ID badge that says who you are.
- Correctly distinguish between the “Access Token” (OAuth) and “ID Token” (OIDC).

### T11. Horizontal vs Vertical Scaling

#### Explanation

Vertical scaling (Scaling Up) involves adding more power (CPU, RAM) to an existing server, whereas Horizontal scaling (Scaling Out) involves adding more machines to your resource pool. Vertical scaling has a hard upper limit defined by hardware (e.g., AWS u-24tb1.112xlarge), while horizontal scaling is theoretically infinite but adds architectural complexity regarding data synchronization and networking.

#### Advantages

- Vertical: Simple to implement; no changes to application code required; low network latency as processes stay on one box.
- Horizontal: High availability (no single point of failure); cheaper to use many commodity instances than one super-computer; extreme resilience.

#### Disadvantages

- Vertical: Hard hardware ceiling; requires downtime to upgrade; single point of failure.
- Horizontal: Requires a Load Balancer; introduces complexity in session management and distributed data consistency; increased network latency between nodes.

#### Use Cases

- Pick Vertical when you are in early-stage development and need simplicity.
- Pick Vertical for small databases that don’t exceed a few terabytes.
- Pick Horizontal for high-traffic web applications with millions of users.
- Pick Horizontal for large-scale distributed systems like Cassandra or Kafka.

#### Interview Tips

- State clearly that modern “Web Scale” almost always implies Horizontal scaling.
- Mention that Vertical scaling is often the first step because it’s easier, but Horizontal is the goal for reliability.
- Discuss the concept of “statelessness” as a prerequisite for Horizontal scaling.

### T12. Docker vs Kubernetes

#### Explanation

Docker is a technology used to create, distribute, and run isolated containers (packaging the code and dependencies). Kubernetes (K8s) is an orchestration platform used to manage those containers across a cluster of machines. You use Docker to build the “ship” and Kubernetes to manage the “fleet” and the “port.”

#### Advantages

- Docker: Ensures “it works on my machine” consistency; fast startup times; lightweight compared to VMs.
- Kubernetes: Automates self-healing (restarts failed containers); handles auto-scaling based on CPU/RAM; manages service discovery and load balancing natively.

#### Disadvantages

- Docker: Managing single containers manually becomes impossible at scale; no built-in way to handle cross-node networking.
- Kubernetes: Extremely steep learning curve; significant operational overhead for small teams; overkill for simple CRUD apps.

#### Use Cases

- Pick Docker when you need to standardize the development environment for a small team.
- Pick Kubernetes when you are running a Microservices architecture with 10+ different services.
- Pick Kubernetes when you need zerodowntime deployments (rolling updates).
- Pick Docker alone (or Docker Compose) for simple side projects or internal tools.

#### Interview Tips

- Clarify that these are not competitors; they work together (though alternatives like Podman or Nomad exist).
- Mention “Pods” as the smallest unit of deployment in K8s.
- Highlight K8s’s ability to handle “StatefulSets” versus “Deployments.”

### T13. HTTP/1.1 vs HTTP/2 vs HTTP/3

#### Explanation

HTTP/1.1 is the classic text-based protocol that suffers from Headof-Line (HoL) blocking. HTTP/2 introduced binary framing and multiplexing (multiple requests over one TCP connection) to solve this. HTTP/3 moves away from TCP entirely, using QUIC (built on UDP) to eliminate HoL blocking at the transport layer, specifically improving performance on unstable mobile networks.

#### Advantages

- HTTP/1.1: Simplest to debug (plain text); universally supported by every legacy system.
- HTTP/2: Header compression (HPACK); server push capabilities; multiplexing reduces latency.
- HTTP/3: Faster connection establishment (0-RTT); better handling of network switches (e.g., switching from Wi-Fi to 5G).

#### Disadvantages

- HTTP/1.1: High overhead; opens many TCP connections; slow due to serial processing.
- HTTP/2: A single TCP packet loss stalls all multiplexed streams (TCP HoL blocking).
- HTTP/3: Massive CPU overhead for encryption; many enterprise firewalls still block UDP on port 443.

#### Use Cases

- Pick HTTP/1.1 for internal microservice communication where simplicity is key.
- Pick HTTP/2 as the standard for modern web browsers to speed up asset loading.
- Pick HTTP/3 for mobile apps where users frequently move between different network signals.

#### Interview Tips

- Focus on “Head-of-Line Blocking”—distinguish between the application layer (H2) and transport layer (H3) versions.
- Mention that H2 and H3 require TLS (HTTPS) by default in most implementations.

### T14. TCP vs UDP

#### Explanation

TCP (Transmission Control Protocol) is a connection-oriented protocol that guarantees delivery and order through a three-way handshake and acknowledgments. UDP (User Datagram Protocol) is a connectionless, “fire-and-forget” protocol that prioritizes speed and low overhead over reliability.

#### Advantages

- TCP: Relied upon for data integrity; handles flow control (prevents overwhelming the receiver); retransmits lost packets automatically.
- UDP: Minimal latency; no handshake delay; tiny packet headers (8 bytes vs TCP’s 20-60 bytes); supports broadcasting.

#### Disadvantages

- TCP: Slower due to overhead; high latency if packets are dropped (Head-of-Line blocking).
- UDP: No guarantee that data arrives; packets may arrive out of order; no built-in congestion control.

#### Use Cases

- Pick TCP for Web (HTTP), Email (SMTP), and File Transfers (FTP) where data accuracy is non-negotiable.
- Pick UDP for VoIP (Zoom, Discord) and Live Streaming where a dropped frame is better than a delayed one.
- Pick UDP for Online Gaming (e.g., FPS games) where real-time position updates are critical.
- Pick UDP for DNS queries to ensure fast resolution.

#### Interview Tips

- Use the analogy: TCP is a phone call (requires connection/ acknowledgment); UDP is a postcard (send it and hope).
- Mention that modern protocols like QUIC use UDP but implement their own reliability layer.

### T15. Monolith vs Microservices

#### Explanation

A Monolith is a single, unified unit where all business logic, UI, and database access are co-located. Microservices break this unit into small, independent services that communicate via APIs (REST, gRPC) or messages. Monoliths prioritize initial speed and simplicity, while Microservices prioritize independent scaling and team autonomy.

#### Advantages

- Monolith: Easier to deploy, test, and debug; low latency (function calls instead of network calls); atomic transactions are simple.
- Microservices: Independent scaling of bottlenecks; technology diversity (use Python for AI, Go for APIs); fault isolation (one crash doesn’t kill the whole system).

#### Disadvantages

- Monolith: Scaling requires duplicating the whole app; “Spaghetti code” risk as it grows; slow build/deploy times.
- Microservices: Network latency; complex distributed transactions (Saga pattern); operational overhead (needs K8s, monitoring, tracing).

#### Use Cases

- Pick Monolith for startups trying to find Product-Market Fit.
- Pick Monolith for small-to-medium teams (under 10-15 engineers).
- Pick Microservices for large organizations where teams need to deploy independently without stepping on toes.
- Pick Microservices when different parts of the app have vastly different resource requirements (e.g., high CPU vs high RAM).

#### Interview Tips

- Don’t bash Monoliths—many large companies (like Shopify or early Instagram) succeeded with them.
- Mention that Microservices is an organizational solution as much as a technical one.

### T16. Load Balancer Algorithms compared

#### Explanation

Load Balancers distribute incoming traffic across multiple servers. The choice of algorithm determines how the “next” server is chosen, balancing factors like server health, current load, and session persistence.

#### Advantages

- Round Robin: Extremely simple; no state required; works well if all servers have identical hardware.
- Least Connections: Smart; prevents sending traffic to a server that is already struggling with long-running requests.
- IP Hash: Ensures a specific client always goes to the same server; useful for local caching/ sessions.

#### Disadvantages

- Round Robin: Can overwhelm a server if it gets stuck with many heavy requests while others are idle.
- Least Connections: Requires the Load Balancer to keep track of every active connection, increasing its CPU usage.
- IP Hash: Can lead to uneven distribution if many users share a single NAT/Gateway IP.

#### Use Cases

- Pick Round Robin for simple, stateless web servers with identical specs.
- Pick Least Connections for database proxies or chat servers where connection length varies wildly.
- Pick Weighted Round Robin when you have a mix of powerful new servers and old, weak hardware.
- Pick IP Hash when your application is “stateful” and you aren’t using a distributed session store like Redis.

#### Interview Tips

- Mention Layer 4 (Transport/IP) vs Layer 7 (Application/URL) load balancing.
- Explain “Sticky Sessions” and the trade-offs regarding horizontal scaling.

### T17. Database Indexing (B-tree vs Hash vs LSM)

#### Explanation

Indexing is a data structure used to speed up data retrieval. B-trees (Balanced Trees) are the standard for general-purpose databases. Hash indexes are for exact matches. LSM-Trees (Log-Structured Merge-Trees) are optimized for high-write throughput by treating the disk as a sequential log.

#### Advantages

- B-tree: Excellent for range queries (age > 20); predictable O(log n) performance; very mature (PostgreSQL, MySQL).
- Hash: O(1) lookups for equality; very fast for simple key-value lookups.
- LSMTree: High write performance; efficient disk space usage via compaction (Cassandra, RocksDB).

#### Disadvantages

- B-tree: Write amplification (every insert requires tree reorganization/disk seeks).
- Hash: Cannot do range scans; collision management; expensive to resize.
- LSMTree: Reads can be slow because you might have to check multiple files (requires Bloom filters).

#### Use Cases

- Pick B-tree for standard RDBMS workloads and range-heavy queries.
- Pick Hash for in-memory caches or simple ID-to-Object lookups.
- Pick LSM-Tree for write-heavy applications like logging, metrics, or messaging platforms.

#### Interview Tips

- Explain that indexes speed up reads but slow down writes because the index must be updated.
- Mention that B-Trees are generally better for HDDs/ SSDs because they minimize random disk I/O.

### T18. Caching Strategies (aside vs through vs back)

#### Explanation

Caching strategies define the relationship between the Application, the Cache, and the Database. In Cache-Aside, the app manages both. In Read-Through/ Write-Through, the application only talks to the cache, which handles the DB. In WriteBack, the app writes to the cache and the cache updates the DB asynchronously later.

#### Advantages

- CacheAside: Resilient; if the cache fails, the app hits the DB directly.
- WriteThrough: Data consistency is high; every write is in both systems before finishing.
- WriteBack: Extreme write performance; reduces load on the DB during traffic spikes.

#### Disadvantages

- CacheAside: Potential for “Stale Data” if the DB is updated without invalidating the cache.
- WriteThrough: Higher write latency because you must write to two places.
- WriteBack: Risk of data loss if the cache crashes before the data is persisted to the DB.

#### Use Cases

- Pick Cache-Aside for most general web applications (e.g., Redis + PostgreSQL).
- Pick Write-Through for banking or systems where data must be immediately consistent.
- Pick Write-Back for logging hits, likes, or real-time leaderboards where losing the last 2 minutes of data is acceptable.

#### Interview Tips

- Always mention “Cache Eviction” policies (LRU, LFU) when discussing these strategies.
- Talk about the “Thundering Herd” problem when a cache key expires and everyone hits the DB at once.

### T19. Event-Driven vs Request/Response

#### Explanation

Request/Response (Synchronous) is a direct interaction where a client waits for a server to process a request (e.g., REST). Event-Driven (Asynchronous) involves a producer emitting an “event” to a broker (like Kafka) and moving on, while consumers process that event whenever they are ready.

#### Advantages

- Req/Res: Simple mental model; immediate feedback/confirmation; easy to debug.
- Event-Driven: Loose coupling; high scalability; handles traffic spikes via buffering; producer doesn’t care if consumer is down.

#### Disadvantages

- Req/Res: Direct coupling (if the server is down, the client fails); “Cascading failures” if one service blocks.
- Event-Driven: Complex to track state/ flow; “Eventual Consistency” issues; hard to implement synchronous user feedback.

#### Use Cases

- Pick Request/Response for UI logins, fetching user profiles, or processing a credit card payment.
- Pick Event-Driven for sending emails, image processing, or updating analytics dashboards.
- Pick Event-Driven for inter-service communication in complex Microservice environments.

#### Interview Tips

- Mention the role of a Message Broker (RabbitMQ, Kafka) as the buffer in Event-Driven systems.
- Discuss “Backpressure”—how event-driven systems prevent consumers from being overwhelmed.

### T20. Strong vs Eventual Consistency

#### Explanation

Strong consistency ensures that any read returns the most recent write (the system looks like one copy of data). Eventual consistency guarantees that if no new updates are made, all reads will eventually return the same value, but there might be a period where different users see different data.

#### Advantages

- Strong: Familiar and easy to program against; no “time-travel” bugs where data disappears.
- Eventual: High availability; low latency; the system stays up even if some nodes are partitioned.

#### Disadvantages

- Strong: High latency (all nodes must agree before finishing a write); system becomes unavailable if a majority of nodes can’t communicate.
- Eventual: Highly complex for developers; requires handling conflicts (e.g., Last Write Wins or CRDTs).

#### Use Cases

- Pick Strong for financial transactions, inventory counts, or password changes.
- Pick Eventual for social media likes, comments, or DNS records.
- Pick Eventual for distributed global caches where speed is prioritized over perfect accuracy.

#### Interview Tips

- Connect this to the CAP Theorem (Consistency vs Availability vs Partition Tolerance).
- Mention “Linearizability” as a formal term for Strong Consistency.

### T21. Optimistic vs Pessimistic Locking

#### Explanation

These are strategies to handle concurrent updates to the same data. Pessimistic locking assumes conflict is likely and locks the record (e.g., SELECT FOR UPDATE) before modification, preventing others from reading or writing. Optimistic locking assumes conflict is rare, using a version number or timestamp to check if the data changed between the read and the write, failing the operation if it has.

#### Advantages

- Pessimistic: Prevents all data integrity issues upfront; no need for application-side retry logic.
- Optimistic: High throughput; doesn’t hold expensive database locks; works well across distributed systems where physical locks are hard to maintain.

#### Disadvantages

- Pessimistic: High performance cost; risk of deadlocks; poor scalability as connection counts grow.
- Optimistic: Transactions fail if competition is high, requiring complex retry loops in the application.

#### Use Cases

- Pick Pessimistic when the cost of a failed transaction is higher than the cost of waiting (e.g., banking transfers).
- Pick Optimistic when contention is low or the UI allows the user to resolve the conflict (e.g., Wiki edits, Shopify inventory).
- Pick Optimistic for long-running workflows where a user might take minutes to complete a form.

#### Interview Tips

- Mention that Optimistic locking is usually implemented via a WHERE version = current_version clause.
- Link this to the “Lost Update” problem in ACID properties.

### T22. OLTP vs OLAP

#### Explanation

OLTP (Online Transactional Processing) is designed for high-frequency, small-scale operations like inserting a single order. OLAP (Online Analytical Processing) is designed for complex, read-heavy queries over massive datasets to find trends, such as calculating the average revenue per user over the last five years.

#### Advantages

- OLTP: Fast response times for end-users; handles thousands of concurrent users; ensures ACID compliance.
- OLAP: Hyper-efficient at scanning millions of rows; supports complex aggregations (SUM, AVG) without locking operational databases.

#### Disadvantages

- OLTP: Becomes extremely slow if you try to run analytical queries on it; storage is not optimized for multidimensional analysis.
- OLAP: Updates are slow and usually batched; not suitable for real-time user interactions; high data latency from the source.

#### Use Cases

- Pick OLTP for the “heart” of your application, like a Postgres or MySQL database for user profiles.
- Pick OLAP when you need a Data Warehouse (e.g., Snowflake, BigQuery) for business intelligence.
- Pick OLAP when you need to run “What-if” scenarios on historical data.

#### Interview Tips

- Distinguish them by workload: “Write-heavy/Point-reads” vs “Scan-heavy/Aggregations.”
- Mention the ETL/ELT process as the bridge that moves data from OLTP to OLAP.

### T23. Row-store vs Column-store databases

#### Explanation

This refers to how data is physically arranged on disk. Row-stores (Postgres, MySQL) keep all data for a single record together, which is great for fetching a specific user. Column-stores (Cassandra, ClickHouse, Redshift) store all values for a single column together, which allows the CPU to skip unnecessary data when calculating averages or sums.

#### Advantages

- Row-store: Fast at inserting a full record; fast at “point lookups” (finding one row by ID).
- Column-store: Incredible compression ratios because similar data types are stored together; drastically faster for analytical queries that only need 2 out of 50 columns.

#### Disadvantages

- Row-store: Slow for analytics because the system must read every piece of data in the row even if you only need the “price” column.
- Column store: Very slow for single-row updates or inserts; “SELECT *” is a performance nightmare.

#### Use Cases

- Pick Row-store for standard CRUD applications and transactional systems.
- Pick Column-store for logging, telemetry, and data warehousing.
- Pick Column-store when you need to aggregate values across billions of rows (e.g., ad-tech analytics).

#### Interview Tips

- Relate this to I/O: “Column-stores minimize disk I/O by only reading the specific columns required for the query.”

### T24. Push vs Pull models for feeds

#### Explanation

In a Push model (Fan-out on Write), a message is delivered to every follower’s inbox at the moment it is posted. In a Pull model (Fan-out on Load), a user’s feed is only computed when they actually log in and refresh their page by querying the posts of everyone they follow.

#### Advantages

- Push: Extremely fast read times for the end user; feed is precomputed.
- Pull: Efficient for the system when users post frequently but have few followers; no storage wasted on inactive users.

#### Disadvantages

- Push: “The Celebrity Problem”—a user with 50 million followers causes a massive write spike that can take minutes to finish.
- Pull: Slower page loads; high “Read” load on the database to aggregate posts from hundreds of followed accounts.

#### Use Cases

- Pick Push for small-to-medium social networks or for users with a “normal” follower count (< 5,000).
- Pick Pull for “Celebrities” or “High-Density” accounts to avoid write-amplification.
- Pick a hybrid approach (like Twitter/X) where regular users push, but celebrities are pulled.

#### Interview Tips

- Always mention the “Celebrity / Hotspot” problem.
- Mention that Push optimizes for “Read Latency” while Pull optimizes for “Write Latency.”

### T25. WebSockets vs SSE vs Long Polling

#### Explanation

These are methods for real-time communication. WebSockets provide a full-duplex (two-way) persistent connection. SSE (Server-Sent Events) is a oneway (server-to-client) persistent connection. Long Polling is a hack where the client requests data, and the server holds the request open until new data is available or a timeout occurs.

#### Advantages

- WebSockets: Lowest latency for two-way communication; efficient for high-frequency updates.
- SSE: Uses standard HTTP; automatic reconnection; lighter on server resources than WebSockets for one-way streams.
- Long Polling: Works on old infrastructure and browsers that don’t support modern protocols.

#### Disadvantages

- WebSockets: Harder to scale (requires sticky sessions or a pub/ sub backbone like Redis); not HTTP-compatible (firewall issues).
- SSE: Only one-way; limited number of concurrent connections per browser in older HTTP/1.1.
- Long Polling: High overhead; server resources are tied up for every “wait” period.

#### Use Cases

- Pick WebSockets for chat apps, multiplayer games, or collaborative editing (Google Docs).
- Pick SSE for live news feeds, stock tickers, or social media notifications.
- Pick Long Polling only as a fallback for legacy system compatibility.

#### Interview Tips

- Mention that WebSockets start as an HTTP request and “Upgrade” to a TCP socket.
- Mention that for modern apps, SSE is often simpler than WebSockets if you don’t need the client to send data back over the same pipe.

### T26. Sync vs Async replication

#### Explanation

Sync replication waits for a follower to acknowledge the write before telling the client “Success.” Async replication returns “Success” as soon as the leader records the write, sending the data to followers in the background.

#### Advantages

- Sync: Strong consistency; no data loss if the leader crashes.
- Async: High performance; the leader can continue working even if followers are slow or the network is jittery.

#### Disadvantages

- Sync: Extremely high latency; if one follower is down, the whole system hangs (availability drops).
- Async: Risk of data loss if the leader fails before the update propagates; “Read-after-write” inconsistency.

#### Use Cases

- Pick Sync for critical financial systems where data loss is unacceptable (often in a “Semi-sync” mode with at least 1 follower).
- Pick Async for social media, logs, and most global-scale web apps where speed is more important than 100% durability.
- Pick Async for cross-region replication to avoid the speed-of-light latency across oceans.

#### Interview Tips

- Discuss the “CAP Theorem” trade-off: Sync prioritizes Consistency, Async prioritizes Availability/Performance.
- Mention “Semi-synchronous” replication as a middle ground (wait for 1 of N followers).

### T27. Leader-based vs Leader-less replication

#### Explanation

Leader-based systems (Postgres, MySQL, MongoDB) have a primary node that handles all writes. Leader-less systems (Cassandra, DynamoDB) allow any node to accept a write, using a quorum (W+R > N) to ensure data consistency across the cluster.

#### Advantages

- Leader-based: Simpler to reason about; no conflict resolution needed for concurrent writes to the same key.
- Leader-less: High availability for writes; no “Single Point of Failure” for the whole cluster; handles multi-datacenter setups naturally.

#### Disadvantages

- Leader-based: The leader is a bottleneck; failover (promoting a new leader) is complex and causes downtime.
- Leader-less: Requires complex conflict resolution (e.g., LWW - Last Write Wins); “stale reads” are common if quorum isn’t configured correctly.

#### Use Cases

- Pick Leader-based for RDBMS applications where ACID and strict ordering are required.
- Pick Leader-less for high-velocity write systems that must never go down (e.g., shopping carts, IoT sensor data).

#### Interview Tips

- Use the term “Quorum” and explain the formula (e.g., for a cluster of 3, you need 2 nodes to agree).
- Mention “Vector Clocks” as a way to resolve conflicts in leaderless systems.

### T28. 2PC vs Saga for distributed transactions

#### Explanation

Both solve the “Atomic Commit” problem across different services. 2PC (Two-Phase Commit) is a synchronous protocol where a coordinator asks all participants to “Prepare” and then “Commit.” A Saga is a sequence of local transactions where each step has a “Compensating Transaction” to undo it if a later step fails.

#### Advantages

- 2PC: Strong consistency; the system behaves like a single database; automatic rollback.
- Saga: Highly scalable; doesn’t lock resources for long periods; works across different types of databases (Polyglot).

#### Disadvantages

- 2PC: “The Coordinator” is a single point of failure; heavy performance penalty; locking can lead to system-wide stalls.
- Saga: No “Isolation” (other transactions can see partial results); complex to implement “Undo” logic (compensation).

#### Use Cases

- Pick 2PC for legacy systems or distributed databases (like Google Spanner) where hardware/network is highly controlled.
- Pick Saga for modern microservices where workflows span multiple services and take seconds or minutes to complete (e.g., booking a trip).

#### Interview Tips

- Explain that Sagas are eventually consistent.
- Mention the two Saga flavors: “Choreography” (event-driven) vs “Orchestration” (central controller).

### T29. CQRS vs traditional CRUD

#### Explanation

CRUD (Create, Read, Update, Delete) uses the same data model for both reads and writes. CQRS (Command Query Responsibility Segregation) splits the application into two parts: Commands (writes) and Queries (reads), often using different databases for each to optimize performance.

#### Advantages

- CRUD: Simple to build, test, and maintain; less infrastructure overhead.
- CQRS: Allows scaling reads and writes independently; reads can be preoptimized (denormalized) for specific UI views.

#### Disadvantages

- CRUD: Complex queries can slow down the write database; difficult to optimize when read and write patterns diverge.
- CQRS: Massive increase in complexity; eventual consistency between the write-store and read-store; “Architectural Overkill” for most apps.

#### Use Cases

- Pick CRUD for 90% of business applications and early-stage MVPs.
- Pick CQRS for high-scale systems where read performance is critical and the read UI is significantly different from the domain model (e.g., complex dashboards).

#### Interview Tips

- Warn that CQRS should not be the “default”—it is a solution for specific scaling and complexity problems.
- Link CQRS with “Materialized Views.”

### T30. Event Sourcing vs State-Based persistence

#### Explanation

State-Based persistence only stores the current version of a record (e.g., Balance = $50). Event Sourcing stores the entire history of changes as an immutable sequence of events (e.g., +$100, -$20, -$30). The current state is derived by “replaying” these events.

#### Advantages

- State-Based: Simple to query; industry standard; uses less storage.
- Event Sourcing: Perfect audit log; ability to reconstruct state at any point in time; “Time Travel” debugging.

#### Disadvantages

- State-Based: History is lost unless you manually build audit tables; no way to know “how” we got to the current state.
- Event Sourcing: Hard to query (needs snapshots to be performant); “Event Schema Evolution” is extremely difficult to manage.

#### Use Cases

- Pick State-Based for most applications where only the “Now” matters.
- Pick Event Sourcing for financial ledgers, version control systems (Git), or complex order management.

#### Interview Tips

- Mention “Snapshots” as a way to optimize Event Sourcing so you don’t have to replay millions of events every time.
- Point out that Event Sourcing and CQRS are often used together but are different concepts.

### T31. Bloom Filter vs Cuckoo Filter

#### Explanation

Both are probabilistic data structures used to test set membership with zero false negatives but possible false positives. Bloom Filters use a bit array and multiple hash functions to flip bits, while Cuckoo Filters use a fingerprint-based hash table that allows for item displacement (moving items between two possible buckets).

#### Advantages

- Bloom Filters: Extremely simple to implement; space efficiency remains constant regardless of the number of hash functions used.
- Cuckoo Filters: Support true deletion of items (Bloom filters usually require a “Counting” variant); higher lookup performance as they are more cache-friendly.

#### Disadvantages

- Bloom Filters: Cannot delete items; performance degrades (false positives increase) as the filter fills up.
- Cuckoo Filters: Insertion can fail (requiring a rebuild) if the table reaches high load factors; more complex logic to handle collisions/ kicking items.

#### Use Cases

- Pick Bloom Filter when you need a simple “is this in the DB” check (e.g., Apache Cassandra, Google Bigtable).
- Pick Bloom Filter when the dataset is static or append-only.
- Pick Cuckoo Filter when you need to delete items frequently from the filter.
- Pick Cuckoo Filter for high-performance networking applications where CPU cache hits matter.

#### Interview Tips

- Mention that neither structure yields “False Negatives”—if it says it’s not there, it’s 100% not there.
- Discuss the “False Positive” trade-off: larger filters reduce errors but consume more RAM.

### T32. Consistent Hashing vs Modulo Hashing

#### Explanation

Modulo Hashing distributes data by calculating hash(key) % N, where N is the number of nodes. Consistent Hashing maps both keys and nodes onto a logical ring (0 to 2^32-1), where a key is assigned to the first node encountered moving clockwise.

#### Advantages

- Modulo Hashing: Extremely simple; requires no metadata storage about node positions.
- Consistent Hashing: Minimizes data movement during rescaling; only Total_Keys / N items need relocation when a node is added/removed.

#### Disadvantages

- Modulo Hashing: Adding or removing a single node forces a massive reshuffle (roughly (N-1)/N of the data moves), causing “cache storms.”
- Consistent Hashing: More complex to implement; requires “Virtual Nodes” to ensure even distribution across physical hardware.

#### Use Cases

- Pick Modulo Hashing when the number of nodes is fixed and never changes (e.g., sharding by ID in a small, static database).
- Pick Consistent Hashing for distributed caches like Memcached or Redis clusters.
- Pick Consistent Hashing for P2P networks or CDNs where nodes join and leave frequently.

#### Interview Tips

- Emphasize the “Virtual Nodes” concept to solve the problem of non-uniform data distribution.
- Explain the “Rehash” impact: Modulo is 𝑂(𝑇𝑜𝑡𝑎𝑙_𝐾𝑒𝑦𝑠) moves, Consistent Hashing is 𝑂(𝐾/𝑁) moves.

### T33. Raft vs Paxos

#### Explanation

These are consensus algorithms used to manage replicated logs across a cluster. Paxos is the original, mathematically proven protocol known for its complexity, while Raft was designed specifically to be more understandable by decomposing consensus into Leader Election, Log Replication, and Safety.

#### Advantages

- Raft: Highly readable and easier to implement correctly; strong leader-based model simplifies state management.
- Paxos: Theoretically more flexible; “Multi-Paxos” can achieve higher throughput by allowing multiple proposers in specific high-latency scenarios.

#### Disadvantages

- Raft: The leader can become a bottleneck; rigid constraints on log matching can slightly limit concurrency compared to advanced Paxos variants.
- Paxos: Notoriously difficult to understand and implement; standard Paxos doesn’t define how to handle membership changes or log compaction.

#### Use Cases

- Pick Raft for most modern distributed systems (e.g., etcd in Kubernetes, HashiCorp Consul/Nomad, TiKV).
- Pick Paxos (or a variant) if you are building massive-scale infrastructure at the level of Google (Spanner, Chubby) or Amazon.
- Pick Raft when developer time and maintainability are priorities over micro-optimizations.

#### Interview Tips

- Start by saying “Raft is designed for understandability.”
- Mention that both require a “Quorum” (𝑁/2+1) to make progress and survive 𝐹 failures with 2𝐹 +1 nodes.

### T34. Kubernetes vs Docker Swarm vs Nomad

#### Explanation

These are container orchestrators that manage the deployment, scaling, and networking of containers. Kubernetes (K8s) is the industry heavyweight with a massive ecosystem; Docker Swarm is a simpler, native Docker tool; Nomad is a flexible, lightweight orchestrator from HashiCorp.

#### Advantages

- Kubernetes: Feature-complete (self-healing, auto-scaling); massive community support; cloud-agnostic standard.
- Docker Swarm: Zero installation (built into Docker); very low learning curve; uses standard Docker Compose files.
- Nomad: Lightweight (single binary); orchestrates both containers and non-containerized apps (Java, binaries); scales to 10k+ nodes easily.

#### Disadvantages

- Kubernetes: Steep learning curve; high resource overhead; requires significant operational effort (“YAML hell”).
- Docker Swarm: Limited feature set; smaller ecosystem; not suitable for highly complex, large-scale microservices.
- Nomad: Smaller ecosystem than K8s; requires other tools (Consul, Vault) to match K8s’s built-in functionality.

#### Use Cases

- Pick Kubernetes for large-scale enterprise microservices requiring complex networking and broad tool support.
- Pick Docker Swarm for small teams or simple internal apps where K8s overhead isn’t justified.
- Pick Nomad for hybrid workloads (containers + legacy binaries) or extremely large clusters.

#### Interview Tips

- Characterize K8s as “The Operating System of the Cloud.”
- Mention that the choice often depends on the team’s “Operational Maturity”—don’t pick K8s if you don’t have a dedicated DevOps person.

### T35. Reverse Proxy vs API Gateway vs Load Balancer

#### Explanation

A Load Balancer (LB) distributes traffic across servers. A Reverse Proxy (RP) sits in front of one or more servers to handle security and caching. An API Gateway is a specialized RP designed for microservices, handling “Layer 7” concerns like authentication, rate limiting, and request transformation.

#### Advantages

- Load Balancer: Increases availability and reliability; simple task (distribute packets).
- Reverse Proxy: Increases security (hides backend IPs); improves performance via caching and SSL termination.
- API Gateway: Centralizes crosscutting concerns (auth, logging); can perform “Request Aggregation” to reduce client round-trips.

#### Disadvantages

- Load Balancer: Does not inspect request content (usually L4); limited intelligence.
- Reverse Proxy: Introduces a single point of failure if not redundant; adds slight latency.
- API Gateway: Can become a “Megolith” (centralized bottleneck); complex to manage routing rules at scale.

#### Use Cases

- Pick a Load Balancer (like AWS NLB) to distribute TCP/UDP traffic at high scale.
- Pick a Reverse Proxy (like NGINX or HAProxy) for SSL termination and static content caching.
- Pick an API Gateway (like Kong, Tyk, or AWS AppSync) when you need to enforce API keys and rate limits for mobile/web clients.

#### Interview Tips

- Clarify that these roles overlap: NGINX can be all three.
- Focus on the OSI layers: LBs often operate at Layer 4 (Transport), while API Gateways operate at Layer 7 (Application).

### T36. Idempotency vs Exactly-Once Semantics

#### Explanation

Idempotency is a property where an operation can be performed multiple times with the same result (e.g., x = 10 is idempotent, x++ is not). Exactly-Once (EO) semantics is a guarantee in messaging systems (like Kafka) that even if a producer retries, the final state reflects the message being processed exactly once.

#### Advantages

- Idempotency: Simplifies system design; allows safe retries at the application level without complex distributed transactions.
- Exactly-Once: Provides the strongest consistency for financial or counting systems; abstracts away retry logic from the developer.

#### Disadvantages

- Idempotency: Requires unique request IDs and “de-duplication” logic in the database (e.g., unique constraints).
- Exactly-Once: Significant performance overhead; relies on heavy-duty coordination and atomic commits between the producer, broker, and consumer.

#### Use Cases

- Pick Idempotency for REST APIs (PUT/DELETE methods should be idempotent by design).
- Pick Exactly-Once for payment processing or billing pipelines where double-counting is catastrophic.
- Pick Idempotency when you want to build resilient systems on top of “At-Least-Once” delivery (the most common industry standard).

#### Interview Tips

- Explain that “Exactly-Once” is often achieved by combining “AtLeast-Once” delivery with “Idempotent” processing.
- Mention the “Idempotency Key” pattern used by Stripe.

### T37. Circuit Breaker vs Retry vs Bulkhead

#### Explanation

These are resilience patterns. Retries attempt the same operation again after a failure. A Circuit Breaker (CB) stops requests to a failing service for a timeout period to allow it to recover. A Bulkhead isolates resources (like thread pools) so that one failing service doesn’t exhaust all system resources.

#### Advantages

- Retry: Handles transient network blips; simple to implement.
- Circuit Breaker: Prevents “cascading failures”; gives downstream services breathing room to recover.
- Bulkhead: Provides fault isolation; ensures service A remains healthy even if service B’s pool is saturated.

#### Disadvantages

- Retry: Can worsen congestion (the “retry storm” effect) if the downstream service is already overloaded.
- Circuit Breaker: Complexity in tuning thresholds (e.g., when to open/close); users may perceive “hard” errors.
- Bulkhead: Increased complexity in resource management and configuration for every service.

#### Use Cases

- Use Retries with “Exponential Backoff” for intermittent network issues.
- Use a Circuit Breaker (e.g., Resilience4j, Hystrix) when calling external 3rd party APIs (like Twilio or Stripe).
- Use Bulkheads in multi-tenant environments to prevent a single busy user from hogging all worker threads.

#### Interview Tips

- Mention the three states of a Circuit Breaker: Closed (Normal), Open (Failing fast), and Half-Open (Testing recovery).
- Emphasizing that retries without backoff are a recipe for a self-inflicted DDoS attack.

### T38. Blue-Green vs Canary vs Rolling Deployments

#### Explanation

Deployment strategies to minimize downtime. Blue-Green switches 100% of traffic from an old version (Blue) to a new one (Green) via a load balancer. Canary releases the new version to a small subset of users (e.g., 5%) before a full rollout. Rolling updates replace old instances with new ones one by one.

#### Advantages

- Blue-Green: Near-instant rollback (just flip the switch back); simplifies state management (no version mix).
- Canary: Safest way to test new code on production traffic; minimizes “blast radius” of bugs.
- Rolling: Zero downtime; requires no extra hardware capacity (unlike Blue-Green).

#### Disadvantages

- Blue-Green: Expensive (requires 2x infrastructure); hard to manage if database schemas change significantly.
- Canary: Complex monitoring and routing logic required to direct specific users.
- Rolling: Slow rollout; for a period, two different versions of the app are running simultaneously (N-1 compatibility issues).

#### Use Cases

- Pick Blue-Green for mission-critical apps where immediate rollback is the top priority.
- Pick Canary for high-traffic apps (Facebook, Netflix) where a bug in a new feature could affect millions.
- Pick Rolling for standard Kubernetes deployments where cost-efficiency and simplicity are preferred.

#### Interview Tips

- Mention that the biggest challenge for all strategies is “Database Migrations”—schema changes must be backward compatible.
- Explain the “Blast Radius”—Blue-Green affects 100%, Canary affects 1-5%.

### T39. Sticky Sessions vs Stateless Design

#### Explanation

Sticky Sessions (Session Affinity) ensure that a specific client always reaches the same server instance. Stateless Design treats every request as independent; any session data (like login state) is stored in a shared external store (e.g., Redis) or a client-side token (JWT).

#### Advantages

- Sticky Sessions: Simple for legacy apps; allows in-memory caching for faster local access.
- Stateless: Horizontal scaling is effortless; failures are handled gracefully (any server can take over); easier to implement with Load Balancers.

#### Disadvantages

- Sticky Sessions: Uneven load distribution; if a server dies, all users attached to it lose their session data.
- Stateless: Requires an extra hop to an external store (latency); state management moves to the client or DB.

#### Use Cases

- Pick Sticky Sessions for legacy monoliths that store session state in HttpSession or local variables.
- Pick Stateless for modern microservices and autoscaling environments (standard practice).
- Pick Stateless when using JWTs for authentication.

#### Interview Tips

- State that “Stateless is the industry preference for cloud-native apps.”
- Mention that Sticky Sessions are often implemented via a “Cookie” inserted by the Load Balancer.

### T40. gRPC vs GraphQL for Internal APIs

#### Explanation

gRPC is a high-performance RPC framework using Protocol Buffers (Binary) and HTTP/2. GraphQL is an application-layer query language that allows clients to request exactly the data they need from a flexible schema.

#### Advantages

- gRPC: Maximum performance (binary format); strict typing; built-in support for streaming; lower CPU/bandwidth usage.
- GraphQL: Eliminates overfetching/under-fetching; simplifies frontend development; provides a “Self-documenting” schema.

#### Disadvantages

- gRPC: Human-unreadable (binary); requires special tools for testing/debugging; limited browser support (requires gRPC-web).
- GraphQL: Can lead to “N+1 query” performance issues on the backend; complexity in caching (no native HTTP caching).

#### Use Cases

- Pick gRPC for low-latency service-to-service communication within a microservices cluster.
- Pick GraphQL for an “API Gateway” or “BFF” (Backend for Frontend) that serves multiple mobile/web clients.
- Pick gRPC when polyglot teams need strictly generated code stubs in different languages (Go, Java, Python).

#### Interview Tips

- Compare them on performance: gRPC is 5-10x faster than REST/ GraphQL due to Protobuf.
- Mention that GraphQL is about “Data Flexibility,” while gRPC is about “Contract-first efficiency.”

## Section 4 — Cheat Sheets

Rapid-fire, single-page references for last-minute revision.

### Section 4 cheat sheet index

- [4.1 Caching Cheat Sheet](#41-caching-cheat-sheet)
- [4.2 Databases Cheat Sheet](#42-databases-cheat-sheet)
- [4.3 Networking Cheat Sheet](#43-networking-cheat-sheet)
- [4.4 Load Balancers Cheat Sheet](#44-load-balancers-cheat-sheet)
- [4.5 Microservices Cheat Sheet](#45-microservices-cheat-sheet)
- [4.6 Queues Cheat Sheet](#46-queues-cheat-sheet)
- [4.7 Design Patterns Cheat Sheet](#47-design-patterns-cheat-sheet)
- [4.8 Cloud Cheat Sheet](#48-cloud-cheat-sheet)
- [4.9 Security Cheat Sheet](#49-security-cheat-sheet)
- [4.10 CAP Cheat Sheet](#410-cap-cheat-sheet)
- [4.11 PACELC Cheat Sheet](#411-pacelc-cheat-sheet)
- [4.12 Scaling Cheat Sheet](#412-scaling-cheat-sheet)

### 4.1 Caching Cheat Sheet

| Concept | Key Idea |
| --- | --- |
| Cache-aside | App reads cache first, DB on miss, writes back to cache. Simple, tolerant of cache failures. |
| Read-through | Cache library reads DB on miss transparently. |
| Write-through | Write goes to cache and DB synchronously. Consistent, slower writes. |
| Write-back | Write goes to cache; async flush to DB. Fast writes, risk of loss. |
| TTL | Time-to-live — auto-expiry to bound staleness. |
| LRU / LFU / FIFO | Eviction policies. |
| Thundering herd | Many misses stampede origin. Fix: request coalescing, jittered TTL. |
| Cache stampede | Similar; use stale-while-revalidate. |

**Tools:** Redis, Memcached, Caffeine, Varnish, CDN edge caches.

### 4.2 Databases Cheat Sheet

| Choice | Pick when |
| --- | --- |
| Postgres / MySQL | Transactions, complex queries, moderate scale. |
| Cassandra / ScyllaDB | Wide-column, write-heavy, multi-DC. |
| DynamoDB | Managed KV, predictable p99, serverless scale. |
| MongoDB | Flexible schema, document model. |
| Redis | Cache, queues, ephemeral state. |
| Elasticsearch | Full-text search, log analytics. |
| ClickHouse / BigQuery | OLAP, analytics, billions of rows. |
| Neo4j / DGraph | Graph traversals |

Read replicas scale reads; sharding scales writes.

### 4.3 Networking Cheat Sheet

- HTTP/1.1: text, HOL blocking, keep-alive.
- HTTP/2: binary, multiplexed streams, header compression (HPACK).
- HTTP/3: QUIC over UDP, no HOL at transport.
- TLS 1.3: 1-RTT handshake, 0-RTT resume.
- TCP: reliable, ordered, connection-oriented.
- UDP: connectionless, low-latency.
- DNS: recursive resolver → root → TLD → authoritative.
- CIDR: /16 = 65,536 addresses; /24 = 256.

### 4.4 Load Balancers Cheat Sheet

| Algorithm | Behavior |
| --- | --- |
| Round Robin | Rotate through backends. |
| Least Connections | Send to least-loaded. |
| IP Hash | Sticky by client IP. |
| Consistent Hash | Sticky, minimal remap. |
| Weighted | Backends have weights (capacity). |
| Random with Two Choices | Pick two, choose less loaded |

Layer 4 (TCP): fast, opaque. Layer 7 (HTTP): smart routing, TLS termination.

### 4.5 Microservices Cheat Sheet

- Boundaries: bounded contexts (DDD).
- Comms: sync (gRPC/REST) + async (queues/events).
- Data: DB per service; no shared schema.
- Discovery: DNS, Consul, K8s services.
- Config: centralized (Consul, ConfigMap).
- Resilience: retries + backoff + circuit breaker + bulkhead + timeout.
- Observability: metrics + logs + traces + correlation ID.

### 4.6 Queues Cheat Sheet

| Tool | Model | Sweet Spot |
| --- | --- | --- |
| Kafka | Distributed log, partitions, consumer groups | Event streaming, replay, high throughput |
| RabbitMQ | AMQP broker, exchanges, routing | Complex routing, RPC, moderate throughput |
| SQS | Managed queue | Simple, serverless |
| NATS | Lightweight pub/sub | Low-latency messaging |
| Redis Streams | Log built on Redis | Lightweight streams |

Delivery: at-most-once, at-least-once, effectively-once (with idempotency).

### 4.7 Design Patterns Cheat Sheet

- Circuit Breaker — fail fast on unhealthy deps.
- Bulkhead — isolate resource pools.
- Retry with jitter — smooth transient failures.
- Idempotency key — safe retries.
- Saga — distributed transactions.
- CQRS — split read/write models.
- Event Sourcing — append-only event log.
- Outbox — transactional publish.
- Sidecar — auxiliary process (Envoy, log shipper).
- Strangler Fig — incremental legacy migration.

### 4.8 Cloud Cheat Sheet

| Category | AWS | GCP | Azure |
| --- | --- | --- | --- |
| Compute | EC2, Lambda, ECS | GCE, Cloud Run, GKE | VM, Functions, AKS |
| Storage | S3, EBS | GCS, Persistent Disk | Blob, Managed Disks |
| SQL | RDS, Aurora | Cloud SQL, Spanner | SQL DB |
| NoSQL | DynamoDB | Firestore, Bigtable | Cosmos DB |
| Queue | SQS, SNS, Kinesis | Pub/Sub | Service Bus |
| CDN | CloudFront | Cloud CDN | Front Door |

### 4.9 Security Cheat Sheet

- Encrypt at rest (KMS) and in transit (TLS 1.2+).
- Rotate secrets; never commit to git.
- Principle of least privilege (IAM).
- Rate limit public APIs.
- Validate all input; parametrize queries.
- Hash + salt passwords (Argon2/bcrypt).
- CSRF tokens for state-changing forms.
- Content Security Policy for XSS defense.
- Signed URLs for temporary asset access.
- Audit logs for sensitive actions.

### 4.10 CAP Cheat Sheet

- CP — MongoDB, HBase, Zookeeper, etcd.
- AP — Cassandra, Dynamo, Riak, CouchDB.
- CA — single-node RDBMS (no partition).

### 4.11 PACELC Cheat Sheet

- PC/EC: Spanner, BigTable — always prefer consistency.
- PA/EL: DynamoDB, Cassandra — always prefer availability/latency.
- PC/EL: MongoDB (tunable) — CP under partition, latency-optimized otherwise.

### 4.12 Scaling Cheat Sheet

1. Measure. Never scale blind.
2. Cache aggressively (client, CDN, edge, app, DB).
3. Read replicas for read-heavy workloads.
4. Shard when a single write leader saturates.
5. Async everything not user-blocking.
6. Precompute (materialized views, fan-out on write).
7. Backpressure via queues.
8. Autoscale on the right signal (CPU, queue depth, custom).

## Section 5 — ASCII Diagram Library

Reusable architectures for whiteboards and virtual interviews.

### Section 5 diagram index

- [5.1 API Gateway](#51-api-gateway)
- [5.2 CDN](#52-cdn)
- [5.3 Microservices](#53-microservices)
- [5.4 Redis Cache](#54-redis-cache)
- [5.5 Kafka](#55-kafka)
- [5.6 RabbitMQ](#56-rabbitmq)
- [5.7 Load Balancer](#57-load-balancer)
- [5.8 Sharding](#58-sharding)
- [5.9 Replication (Primary–Replica)](#59-replication-primaryreplica)
- [5.10 Database Cluster](#510-database-cluster)
- [5.11 Service Discovery](#511-service-discovery)
- [5.12 EventDriven Architecture](#512-eventdriven-architecture)

### 5.1 API Gateway

```text
Clients ──► [ API Gateway ] ──► ┌─► Auth Service
├─► User Service
├─► Order Service
└─► Payment Service
```

Cross-cutting: authN, authZ, rate limit, quotas, logging, tracing, TLS.

### 5.2 CDN

```text
User (Tokyo) ──► Edge PoP (Tokyo) ──miss──► Regional cache ──miss──► Origin (us-east-1)
│ │
└────────────── static/dynamic assets ◄─────────────────┘
```

### 5.3 Microservices

```text
┌───────────────┐
Client ──► LB ──►│ API Gateway │
└───────┬───────┘
│
┌────────────┬───┴────┬────────────┐
▼ ▼ ▼ ▼
[User Svc] [Order Svc][Pay Svc] [Notif Svc]
│ │ │ │
[User DB] [Order DB][Pay DB] [Kafka]
│
[Email/SMS Workers]
```

### 5.4 Redis Cache

```text
App ──► Redis (cache-aside)
│ │ miss
│ ▼
└──► Primary DB ──replicate──► Redis (populate)
```

### 5.5 Kafka

```text
Producers ──► [Broker 1 Broker 2 Broker 3] (partitions replicated)
│ │ │
└───ISR: leader + N followers ───┘
│
```

Consumer Group A: partitions assigned to consumers

Consumer Group B: independent offsets

### 5.6 RabbitMQ

```text
Producer ──► [Exchange] ──routing key──► [Queue] ──► Consumer(s)
```

(direct/topic/fanout/headers)

### 5.7 Load Balancer

```text
┌───────────────┐
Clients ──DNS──► │ L4/L7 LB │─┬─► App Server 1
│ (HA pair,VIP) │ ├─► App Server 2
└───────────────┘ └─► App Server N
```

Health checks remove unhealthy nodes automatically.

### 5.8 Sharding

```text
Shard Router (hash(user_id) % N)
│ │ │ │
```

[Shard 1][Shard 2][Shard 3][Shard 4]

each shard = independent DB with its own replicas

### 5.9 Replication (Primary–Replica)

```text
Writes ──► [ Primary ] ──async/sync──► [ Replica 1 ]
└──► [ Replica 2 ]
Reads ──► round-robin over replicas (eventually consistent)
```

### 5.10 Database Cluster

```text
┌─────────────── Coordinator / Proxy ───────────────┐
│ │ │
```

Shard A Shard B Shard C

[P][R1][R2] [P][R1][R2] [P][R1][R2]

(Raft group) (Raft group) (Raft group)

### 5.11 Service Discovery

```text
Service instance ──register──► [Registry (Consul/etcd)]
Consumer ──lookup──► Registry ──► instance list ──► call
```

Health checks evict dead instances.

### 5.12 EventDriven Architecture

```text
[Order Svc] ──emit "order.created"──► [Kafka Topic]
│
┌─────────────────────┼─────────────────────┐
▼ ▼ ▼
```

[Billing] [Inventory] [Notification]

## Section 6 — Interview Checklists

Use these as the mental scaffolding senior engineers run through automatically.

### Section 6 checklist index

- [6.1 Requirement Gathering Checklist](#61-requirement-gathering-checklist)
- [6.2 Scaling Checklist](#62-scaling-checklist)
- [6.3 Security Checklist](#63-security-checklist)
- [6.4 Caching Checklist](#64-caching-checklist)
- [6.5 Databases Checklist](#65-databases-checklist)
- [6.6 Availability Checklist](#66-availability-checklist)
- [6.7 Monitoring Checklist](#67-monitoring-checklist)
- [6.8 Trade-offs Checklist](#68-trade-offs-checklist)

### 6.1 Requirement Gathering Checklist

- [ ] Who are the users? Consumer, enterprise, internal?
- [ ] What are the top 3 must-have features?
- [ ] What is explicitly out of scope?
- [ ] Geographic distribution?
- [ ] Read-heavy or write-heavy?
- [ ] Consistency requirements? (strong / eventual / bounded)
- [ ] Latency budget? (p50, p99)
- [ ] Availability target? (99.9% / 99.99%)
- [ ] Peak QPS, average QPS?
- [ ] Data size today and 3 years from now?
- [ ] Multi-tenant? Multi-region?

### 6.2 Scaling Checklist

- [ ] Where is the bottleneck? (CPU / IO / DB / network)
- [ ] Can we cache it?
- [ ] Can we replicate reads?
- [ ] Can we shard writes?
- [ ] Can we make it async?
- [ ] Can we precompute?
- [ ] Do we have back-pressure?
- [ ] Do we autoscale on the right signal?

### 6.3 Security Checklist

- [ ] TLS everywhere (in transit).
- [ ] Encryption at rest.
- [ ] AuthN + AuthZ on every endpoint.
- [ ] Rate limiting on public APIs.
- [ ] Input validation and parameterized queries.
- [ ] Secrets in a vault, rotated.
- [ ] Least-privilege IAM.
- [ ] Audit logs for sensitive actions.
- [ ] DDoS protection (CDN/WAF).
- [ ] PII minimization.

### 6.4 Caching Checklist

- [ ] What is hot vs cold data?
- [ ] Cache-aside vs write-through vs write-back?
- [ ] TTL strategy? Jitter?
- [ ] Invalidation on write?
- [ ] Cache stampede protection? (SWR, coalescing)
- [ ] Multi-layer cache? (browser, CDN, app, DB)
- [ ] Memory budget and eviction policy?

### 6.5 Databases Checklist

- [ ] Access patterns identified before schema.
- [ ] SQL vs NoSQL justified.
- [ ] Indexes on frequent lookups (not everything).
- [ ] Partition/sort keys chosen deliberately.
- [ ] Read replicas for read scaling.
- [ ] Sharding plan for write scaling.
- [ ] Backups + PITR.
- [ ] Migration and schema evolution strategy.

### 6.6 Availability Checklist

- [ ] No single points of failure (LB, DB, cache all HA).
- [ ] Multi-AZ deployment.
- [ ] Multi-region for DR (if SLO demands).
- [ ] Health checks + automatic failover.
- [ ] Graceful degradation modes.
- [ ] Chaos testing.
- [ ] Runbooks and on-call.

### 6.7 Monitoring Checklist

- [ ] Golden signals: latency, traffic, errors, saturation.
- [ ] Business metrics (orders/sec, revenue).
- [ ] Alerts tied to SLOs, not raw metrics.
- [ ] Structured logs with correlation IDs.
- [ ] Distributed tracing with sampling.
- [ ] Dashboards per service.
- [ ] Alert routing and rotation.

### 6.8 Trade-offs Checklist

For every choice, state what you give up:

- [ ] Strong consistency ↔ availability, latency, throughput.
- [ ] Normalization ↔ read performance.
- [ ] Microservices ↔ operational complexity.
- [ ] Precomputation ↔ freshness.
- [ ] Sync replication ↔ write latency.
- [ ] Retries ↔ amplified load.
- [ ] Caching ↔ staleness.

## Section 7 — Flashcards

Use these for daily spaced-repetition review. Cover the answer, quiz yourself.

### Section 7 flashcard index

- [Card 1](#card-1)
- [Card 2](#card-2)
- [Card 3](#card-3)
- [Card 4](#card-4)
- [Card 5](#card-5)
- [Card 6](#card-6)
- [Card 7](#card-7)
- [Card 8](#card-8)
- [Card 9](#card-9)
- [Card 10](#card-10)
- [Card 11](#card-11)
- [Card 12](#card-12)
- [Card 13](#card-13)
- [Card 14](#card-14)
- [Card 15](#card-15)
- [Card 16](#card-16)
- [Card 17](#card-17)
- [Card 18](#card-18)
- [Card 19](#card-19)
- [Card 20](#card-20)
- [Card 21](#card-21)
- [Card 22](#card-22)
- [Card 23](#card-23)
- [Card 24](#card-24)
- [Card 25](#card-25)
- [Card 26](#card-26)
- [Card 27](#card-27)
- [Card 28](#card-28)
- [Card 29](#card-29)
- [Card 30](#card-30)
- [Card 31](#card-31)
- [Card 32](#card-32)
- [Card 33](#card-33)
- [Card 34](#card-34)
- [Card 35](#card-35)
- [Card 36](#card-36)
- [Card 37](#card-37)
- [Card 38](#card-38)
- [Card 39](#card-39)
- [Card 40](#card-40)
- [Card 41](#card-41)
- [Card 42](#card-42)
- [Card 43](#card-43)
- [Card 44](#card-44)
- [Card 45](#card-45)
- [Card 46](#card-46)
- [Card 47](#card-47)
- [Card 48](#card-48)
- [Card 49](#card-49)
- [Card 50](#card-50)
- [Card 51](#card-51)
- [Card 52](#card-52)
- [Card 53](#card-53)
- [Card 54](#card-54)
- [Card 55](#card-55)
- [Card 56](#card-56)
- [Card 57](#card-57)
- [Card 58](#card-58)
- [Card 59](#card-59)
- [Card 60](#card-60)
- [Card 61](#card-61)
- [Card 62](#card-62)
- [Card 63](#card-63)
- [Card 64](#card-64)
- [Card 65](#card-65)
- [Card 66](#card-66)
- [Card 67](#card-67)
- [Card 68](#card-68)
- [Card 69](#card-69)
- [Card 70](#card-70)
- [Card 71](#card-71)
- [Card 72](#card-72)
- [Card 73](#card-73)
- [Card 74](#card-74)
- [Card 75](#card-75)
- [Card 76](#card-76)
- [Card 77](#card-77)
- [Card 78](#card-78)
- [Card 79](#card-79)
- [Card 80](#card-80)

### Card 1

**Q:** What is the primary difference between horizontal and vertical scaling?

**A:** Vertical scaling involves adding more power (CPU, RAM) to an existing machine, which has an upper hardware limit and introduces a single point of failure. Horizontal scaling adds more machines to the pool, allowing for near-infinite scale and better fault tolerance through redundancy.

### Card 2

**Q:** What does the CAP theorem state regarding distributed data stores?

**A:** It states that in the event of a network partition, a system can maintain either Consistency (all nodes see the same data) or Availability (every request receives a response), but not both. All systems must choose how to handle the “P” (Partition) by sacrificing either “C” or “A”.

### Card 3

**Q:** Why is “Strong Consistency” difficult to achieve in globally distributed systems?

**A:** Achieving strong consistency requires synchronous communication and consensus among all nodes, which introduces significant latency (speed of light constraints) and reduces availability. Most global systems opt for Eventual Consistency to maintain high performance across long distances.

### Card 4

**Q:** How does a Load Balancer handle “Sticky Sessions”?

**A:** The load balancer uses a cookie or source IP hashing to ensure that all requests from a specific client are routed to the same backend server. This is useful for stateful applications, but it can lead to uneven load distribution and complicates scaling.

### Card 5

**Q:** What is the “Thundering Herd” problem in caching?

**A:** It occurs when a specific cache key expires and a massive surge of concurrent requests all miss the cache and hit the database simultaneously. This can be mitigated using mutex locks (only one process fetches the data) or by adding random jitter to expiration times.

### Card 6

**Q:** What is the purpose of a Content Delivery Network (CDN)?

**A:** A CDN is a geographically distributed network of proxy servers that caches static assets (images, JS, CSS) closer to the end-users. This reduces latency, saves origin bandwidth, and improves page load speeds by serving content from the “edge.”

### Card 7

**Q:** How does Consistent Hashing minimize data movement during reshuffling?

**A:** It maps both nodes and data keys onto a circular hash ring, where each key is assigned to the first node found moving clockwise. When a node is added or removed, only the keys adjacent to that node need to be remapped, rather than re-hashing the entire dataset.

### Card 8

**Q:** What is the difference between a Forward Proxy and a Reverse Proxy?

**A:** A forward proxy sits in front of clients to hide their identities and filter outgoing traffic, whereas a reverse proxy sits in front of servers to handle load balancing, SSL termination, and caching. Users interact with the reverse proxy as if it were the application server itself.

### Card 9

**Q:** Why would you choose a NoSQL database over a Relational Database (RDBMS)?

**A:** NoSQL is preferred when you need a flexible schema, high write throughput, or massive horizontal scalability for unstructured/semi-structured data. RDBMS is better suited for applications requiring complex joins, ACID compliance, and strictly defined data relationships.

### Card 10

**Q:** What is the “Write-Ahead Log” (WAL) in database systems?

**A:** A WAL is a family of techniques for providing atomicity and durability by writing changes to a persistent log file before applying them to the actual database files. In the event of a crash, the system can replay the log to restore the database to a consistent state.

### Card 11

**Q:** Why is “Redlock” a controversial algorithm for distributed locking?

**A:** Critics like Martin Kleppmann argue that Redlock relies on synchronized system clocks to manage timeouts, which are unreliable in distributed systems. If a clock jumps or a process undergoes a long GC pause, the lock’s safety properties can be violated.

### Card 12

**Q:** What is the “Database Sharding” strategy?

**A:** Sharding is a horizontal partitioning technique where a large dataset is split into smaller chunks (shards) and distributed across multiple database instances. Each shard acts as an independent database, reducing the load on a single machine and increasing total storage capacity.

### Card 13

**Q:** What is the role of a “Zookeeper” in a distributed system?

**A:** Zookeeper acts as a centralized coordination service for distributed applications, managing configuration information, naming, and providing distributed synchronization. It uses the Zab protocol to maintain a consistent state across a cluster of nodes.

### Card 14

**Q:** How does the “Gossip Protocol” work in peer-to-peer networks?

**A:** Nodes periodically share their known state with a few randomly selected neighbors, who then pass that information to other neighbors. This ensures that information eventually propagates through the entire network in an exponential fashion, similar to how a virus or rumor spreads.

### Card 15

**Q:** What is the difference between Optimistic and Pessimistic Locking?

**A:** Pessimistic locking prevents conflicts by locking a resource the moment it is accessed, assuming a collision is likely. Optimistic locking allows multiple users to read and modify data simultaneously but checks for a version conflict at the time of the write, failing if the data was changed by another process.

### Card 16

**Q:** When should you use a Message Queue instead of a direct API call?

**A:** Use a message queue to decouple services, handle spikes in traffic (load leveling), and perform long-running background tasks asynchronously. This allows the calling service to remain responsive while the worker processes the message at its own pace.

### Card 17

**Q:** What are the benefits of Microservices over a Monolith?

**A:** Microservices allow teams to develop, deploy, and scale individual components independently using different technology stacks. This architecture improves fault isolation, as a failure in one service doesn’t necessarily take down the whole system, but it adds significant operational complexity.

### Card 18

**Q:** What is “Database Normalization” and why is it used?

**A:** Normalization is the process of organizing data to reduce redundancy and improve data integrity by splitting large tables into smaller ones and defining relationships. While it saves space and prevents anomalies, it often requires complex joins that can slow down read performance.

### Card 19

**Q:** What is a “Dead Letter Queue” (DLQ)?

**A:** A DLQ is a specialized queue where messages are sent if they cannot be processed successfully after a certain number of retries. This prevents faulty messages from blocking the main processing pipeline and allows developers to inspect and debug them later.

### Card 20

**Q:** How does a “Read Replica” improve system performance?

**A:** A read replica is a copy of the primary database that handles read-only queries, offloading traffic from the master node. While this improves read throughput, it introduces the risk of “replication lag,” where the replica may briefly serve stale data.

### Card 21

**Q:** What is the “Saga Pattern” in distributed transactions?

**A:** The Saga pattern manages long-lived transactions across multiple microservices by breaking them into a series of local transactions. Each local transaction has a corresponding “compensating transaction” that rolls back the changes if a subsequent step in the sequence fails.

### Card 22

**Q:** What is the difference between “Push” and “Pull” models in monitoring?

**A:** In a Push model (e.g., StatsD), the application sends metrics to the monitoring server, which is better for short-lived jobs. In a Pull model (e.g., Prometheus), the monitoring server scrapes metrics from a known endpoint, providing better control over traffic and health checking.

### Card 23

**Q:** How do “Vector Clocks” help in distributed systems?

**A:** Vector clocks are used to track the causal ordering of events and detect conflicts in systems with no global clock. Each node maintains a list of counters for all nodes, incrementing its own counter upon an update, allowing the system to determine if one event happened before, after, or concurrently with another.

### Card 24

**Q:** What is “Service Discovery” and why is it needed?

**A:** Service discovery is a mechanism that allows microservices to find the network locations (IP and port) of other services dynamically. As instances spin up or down in a cloud environment, a service registry (like Consul or Eureka) tracks their availability to ensure traffic is routed correctly.

### Card 25

**Q:** What is an “Idempotent” API operation?

**A:** An operation is idempotent if making the same request multiple times has the same effect as making it once. For example, a DELETE request is usually idempotent, while a POST request (creating a new record) usually is not, unless a unique idempotency key is provided.

### Card 26

**Q:** What is “Backpressure” in a streaming system?

**A:** Backpressure is a signal sent from a downstream consumer to an upstream producer indicating that it is being overwhelmed with data. This forces the producer to slow down or buffer data, preventing the consumer from crashing due to resource exhaustion.

### Card 27

**Q:** Why would you use “Bloom Filters” in a database?

**A:** A Bloom filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set. Databases use them to quickly check if a row might exist in a file; if the filter says “no,” the database avoids a costly disk I/O operation.

### Card 28

**Q:** What is “Event Sourcing”?

**A:** Event sourcing is a pattern where state changes are stored as a sequence of immutable events rather than just the current state. The current state can be reconstructed by replaying these events, providing a perfect audit log and the ability to travel back to any point in time.

### Card 29

**Q:** What is the “Two-Phase Commit” (2PC) protocol?

**A:** 2PC is a consensus protocol used to ensure a transaction commits across multiple nodes. It involves a prepare phase, where all nodes agree they are ready to commit, and a commit phase, where they actually execute the change; however, it is slow and prone to blocking if the coordinator fails.

### Card 30

**Q:** What is a “Circuit Breaker” in microservices?

**A:** A circuit breaker is a design pattern that prevents an application from repeatedly trying to execute an operation that’s likely to fail (e.g., calling a down service). After a threshold of failures, the “circuit opens,” and all subsequent calls fail immediately until a timeout period passes and the service is tested again.

### Card 31

**Q:** What is the purpose of “Database Indexing”?

**A:** Indexing creates a data structure (typicaly a B-Tree or Hash Map) that allows the database to find rows much faster than scanning every row in a table. While it significantly speeds up reads, it slows down writes and consumes additional disk space.

### Card 32

**Q:** What is “SSL Termination” at the load balancer?

**A:** SSL termination is the process of decrypting incoming HTTPS traffic at the load balancer before passing it to backend servers as unencrypted HTTP. This offloads the CPU-intensive decryption task from the application servers and simplifies certificate management.

### Card 33

**Q:** How does “Rate Limiting” protect a system?

**A:** Rate limiting restricts the number of requests a user or IP can make within a specific timeframe (e.g., 100 requests per minute). This prevents API abuse, protects against DoS attacks, and ensures fair resource usage among all clients.

### Card 34

**Q:** What is a “Sidecar Pattern”?

**A:** The sidecar pattern involves deploying a helper container alongside a main application container to handle cross-cutting concerns like logging, monitoring, or service proxying (e.g., Envoy). The sidecar shares the same lifecycle and network namespace as the main app.

### Card 35

**Q:** What is “Blue-Green Deployment”?

**A:** It is a deployment strategy that uses two identical environments: “Blue” (current version) and “Green” (new version). Once the Green environment is tested and ready, traffic is switched via the load balancer, allowing for near-zero downtime and instant rollbacks.

### Card 36

**Q:** What is “Data Denormalization”?

**A:** Denormalization is the intentional addition of redundant data to a database to optimize read performance by reducing the need for joins. It is commonly used in data warehousing and NoSQL architectures where read speed is prioritized over write integrity.

### Card 37

**Q:** What is the “Long Polling” technique?

**A:** Long polling is a communication method where the client requests data from the server, and the server holds the request open until new data is available or a timeout occurs. Once the client receives a response, it immediately sends a new request, creating a near-real-time update loop.

### Card 38

**Q:** What is a “Message Broker”?

**A:** A message broker (like RabbitMQ or Kafka) is an intermediary software that facilitates communication between services by translating, routing, and storing messages. It enables asynchronous processing and ensures messages are delivered even if the consumer is temporarily offline.

### Card 39

**Q:** What is “Distributed Tracing”?

**A:** Distributed tracing is a method used to profile and monitor applications, especially those built on microservices, by tracking the path of a request as it travels across various services. Tools like Jaeger or Zipkin use “Trace IDs” to stitch together the entire journey of a single user action.

### Card 40

**Q:** How does “Multi-Version Concurrency Control” (MVCC) work?

**A:** MVCC allows a database to provide concurrent access to data by maintaining multiple versions of a record simultaneously. Readers see a consistent snapshot of the data from when their transaction began, preventing them from being blocked by writers.

### Card 41

**Q:** What is “Hotspotting” in a distributed database?

**A:** Hotspotting occurs when a disproportionate amount of traffic is directed to a single shard or node, often due to a poorly chosen partition key (e.g., using a timestamp). This causes that node to become a bottleneck while others remain idle.

### Card 42

**Q:** What is “Change Data Capture” (CDC)?

**A:** CDC is a process that identifies and captures changes made to a database (Inserts, Updates, Deletes) and delivers those changes in real-time to other systems. It is often implemented by tailing the database’s transaction log (WAL).

### Card 43

**Q:** What is the “API Gateway” pattern?

**A:** An API Gateway acts as a single entry point for all clients, handling tasks like authentication, request routing, rate limiting, and protocol translation. It hides the complexity of the microservices architecture from the consumer.

### Card 44

**Q:** What is the “Paxos” algorithm?

**A:** Paxos is a family of protocols for reaching consensus among a group of unreliable nodes in a distributed system. It ensures that a single value is agreed upon even if some nodes fail or messages are delayed, forming the basis for many consistent data stores.

### Card 45

**Q:** What is “Chaos Engineering”?

**A:** Chaos engineering is the practice of intentionally introducing failures (like shutting down nodes or injecting latency) into a production system to build confidence in its resilience. The goal is to identify and fix weaknesses before they cause unplanned outages.

### Card 46

**Q:** What is a “Stateful” vs. “Stateless” service?

**A:** A stateless service does not store any client data between requests, making it easy to scale and replace. A stateful service (like a database or a socket server) maintains information about the client session, requiring more complex handling for scaling and failover.

### Card 47

**Q:** What is “Pre-signing URLs” in cloud storage?

**A:** A pre-signed URL is a time-limited link generated by a server that grants a client temporary permission to upload or download a specific object directly from cloud storage (like S3). This avoids proxying large files through the application server.

### Card 48

**Q:** What is the difference between “Vertical” and “Horizontal” partitioning?

**A:** Horizontal partitioning (sharding) splits a table into rows across multiple databases. Vertical partitioning splits a table into columns, putting frequently accessed columns in one table and rarely used columns in another to optimize I/O.

### Card 49

**Q:** What is a “Quorum” in distributed systems?

**A:** A quorum is the minimum number of nodes that must agree on an operation (read or write) for it to be considered successful. In a system with 𝑁 nodes, a typical setup requires 𝑊 +𝑅 >𝑁 to ensure that a read always sees the latest write.

### Card 50

**Q:** What is “Object-Relational Mapping” (ORM) and its trade-off?

**A:** ORM allows developers to interact with a database using their programming language’s objects instead of writing SQL. While it speeds up development, it often leads to inefficient queries (like the N+1 problem) and hides the underlying performance characteristics of the DB.

### Card 51

**Q:** What is the “N+1 Query Problem”?

**A:** It occurs when an application makes one query to fetch a list of items and then makes 𝑁 additional queries to fetch related data for each item. This can be resolved by using Eager Loading (SQL Joins) to fetch all necessary data in a single request.

### Card 52

**Q:** What is “ETL” vs “ELT”?

**A:** ETL (Extract, Transform, Load) transforms data on a separate server before loading it into a data warehouse. ELT (Extract, Load, Transform) loads raw data directly into the warehouse and uses the warehouse’s own processing power to perform transformations, which is more common in modern bigdata pipelines.

### Card 53

**Q:** What is a “Reverse Index”?

**A:** A reverse index is a data structure used primarily in full-text search (e.g., Elasticsearch) that maps words to the documents or locations where they appear. This allows for extremely fast searches across large volumes of text compared to standard relational queries.

### Card 54

**Q:** What is the “Command Query Responsibility Segregation” (CQRS) pattern?

**A:** CQRS separates the models for reading data (Queries) from the models for updating data (Commands). This allows each model to be scaled and optimized independently, often using a relational DB for commands and a NoSQL DB or cache for reads.

### Card 55

**Q:** What is “Serverless Computing” (FaaS)?

**A:** Serverless allows developers to write and deploy code as individual functions that trigger on events, without managing the underlying server infrastructure. The cloud provider handles all scaling, but it can suffer from “cold starts” (latency when a function is first called).

### Card 56

**Q:** What is the difference between “Batch Processing” and “Stream Processing”?

**A:** Batch processing (e.g., Hadoop) processes large volumes of data all at once at scheduled intervals. Stream processing (e.g., Flink, Kafka Streams) processes data continuously as it arrives, providing real-time insights with much lower latency.

### Card 57

**Q:** What is “Infrastructure as Code” (IaC)?

**A:** IaC is the practice of managing and provisioning computing infrastructure through machine-readable definition files (like Terraform or CloudFormation) rather than manual configuration. This ensures environment consistency, version control, and repeatable deployments.

### Card 58

**Q:** What is “Heartbeating” in cluster management?

**A:** A heartbeat is a periodic signal sent by a node to a central coordinator or its peers to indicate that it is still alive and functioning. If heartbeats stop being received, the system assumes the node has failed and initiates a failover process.

### Card 59

**Q:** What is a “LSM Tree” (Log-Structured Merge-Tree)?

**A:** An LSM Tree is a data structure optimized for high write throughput by buffering writes in memory and then flushing them to disk as immutable sorted files. Background processes periodically merge these files to keep the number of files low and optimize reads.

### Card 60

**Q:** What is an “Anycast” IP address?

**A:** Anycast is a routing methodology where the same IP address is advertised from multiple locations. The network (BGP) routes the user’s request to the “closest” node (in terms of network hops), commonly used in DNS and CDN nodes for low latency.

### Card 61

**Q:** What is “Graceful Degradation”?

**A:** Graceful degradation is a design philosophy where a system maintains limited functionality even when some components fail, rather than crashing entirely. For example, if a recommendation engine is down, an e-commerce site might show popular items instead of personalized ones.

### Card 62

**Q:** What is the “Fan-out” pattern?

**A:** Fan-out occurs when a single message or request triggers multiple concurrent tasks across different services or workers. In social media, this often refers to delivering a single post to the home feeds of millions of followers simultaneously.

### Card 63

**Q:** What is “SQL Injection” and how is it prevented?

**A:** SQL Injection is a vulnerability where an attacker inserts malicious SQL code into a query through user input. It is prevented by using Parameterized Queries (Prepared Statements), which treat user input as data rather than executable code.

### Card 64

**Q:** What is a “Distributed Hash Table” (DHT)?

**A:** A DHT is a decentralized system that provides a lookup service similar to a hash table, where (key, value) pairs are stored across a network of nodes. It allows for efficient discovery of data without a central coordinator, used in systems like BitTorrent and IPFS.

### Card 65

**Q:** What is “Kubernetes” (K8s)?

**A:** Kubernetes is an open-source platform for automating the deployment, scaling, and management of containerized applications. It provides features like self-healing (restarting failed containers), load balancing, and automated rollouts/rollbacks.

### Card 66

**Q:** What is the “Outbox Pattern”?

**A:** The Outbox pattern ensures atomicity between a database update and sending a message to a queue by writing the message to a special “outbox” table in the same transaction as the business logic. A separate process then polls that table and publishes the messages.

### Card 67

**Q:** What is “Data Gravity”?

**A:** Data gravity is the idea that as data accumulates in one location, it becomes more difficult and expensive to move, drawing applications and services to be located near it. This influences decisions about multicloud strategies and data localization.

### Card 68

**Q:** What is the “RAFT” consensus algorithm?

**A:** RAFT is a consensus algorithm designed to be more understandable than Paxos while being equally safe. It decomposes the problem into leader election, log replication, and safety, ensuring that a cluster of nodes can agree on a shared state.

### Card 69

**Q:** What is the difference between “Latency” and “Throughput”?

**A:** Latency is the time it takes for a single request to be processed (measured in ms), whereas throughput is the total number of requests a system can handle in a given time period (e.g., requests per second). A system can have high throughput but still suffer from high latency.

### Card 70

**Q:** Why are “WebSockets” preferred for real-time chat apps?

**A:** WebSockets provide a persistent, full-duplex communication channel over a single TCP connection, allowing the server to push data to the client instantly. This is much more efficient than HTTP polling, which requires creating a new connection for every message.

### Card 71

**Q:** What is “Role-Based Access Control” (RBAC)?

**A:** RBAC is a security method that restricts system access based on the roles assigned to individual users within an organization. Permissions are attached to roles (e.g., “Editor,” “Admin”) rather than specific users, simplifying the management of security policies.

### Card 72

**Q:** What is the “Long Tail” in latency (P99)?

**A:** The long tail refers to the small percentage of requests that take significantly longer than the average (outliers). Measuring P99 (the 99th percentile) is crucial because it represents the worst experience for users and often reveals deep-seated system bottlenecks.

### Card 73

**Q:** What is “Zero-Copy” I/O?

**A:** Zero-copy is a technique where the CPU does not have to copy data from one memory buffer to another (e.g., from kernel space to user space) during network transfers. This significantly increases efficiency by allowing the NIC to read data directly from the disk buffer.

### Card 74

**Q:** What is a “Service Mesh”?

**A:** A service mesh (like Istio or Linkerd) is a dedicated infrastructure layer for handling service-to-service communication, providing features like traffic management, security (mTLS), and observability. It typically operates using sidecar proxies deployed next to each service.

### Card 75

**Q:** What is “Database Federation”?

**A:** Federation is a way to aggregate data from multiple physically separate databases so they appear as a single logical entity to the application. It allows for querying across different data sources without moving all the data into a single warehouse.

### Card 76

**Q:** What are “Soft Deletes”?

**A:** Soft deleting is the practice of marking a record as “deleted” using a flag (e.g., is_deleted = true) rather than physically removing the row from the database. This allows for easy data recovery and maintains referential integrity, but it can clutter indexes.

### Card 77

**Q:** What is “Clock Skew” in distributed systems?

**A:** Clock skew is the difference in time between the internal clocks of different nodes in a network. Because no two clocks are perfectly synchronized, relying on timestamps for ordering events in a distributed system is dangerous and requires algorithms like Vector Clocks.

### Card 78

**Q:** What is “Negative Caching”?

**A:** Negative caching is the practice of caching the absence of a result (e.g., a “404 Not Found” response). This prevents repeated requests for non-existent resources from constantly hitting the database or origin server and wasting resources.

### Card 79

**Q:** What is “Write-Behind” (Write-Back) caching?

**A:** In a write-behind cache, data is written only to the cache first, and the write to the permanent database happens asynchronously later. This provides extremely low write latency but carries a high risk of data loss if the cache fails before the data is persisted.

### Card 80

**Q:** What is a “Single Point of Failure” (SPOF)?

**A:** A SPOF is any component of a system that, if it fails, stops the entire system from functioning. Building reliable systems involves identifying these points and eliminating them through redundancy, failover mechanisms, and decentralization.

## Section 8 — Multiple Choice Questions

Timed self-assessment. Attempt each question before reading the answer.

### Section 8 MCQ index

- [MCQ Q1](#mcq-q1)
- [MCQ Q2](#mcq-q2)
- [MCQ Q3](#mcq-q3)
- [MCQ Q4](#mcq-q4)
- [MCQ Q5](#mcq-q5)
- [MCQ Q6](#mcq-q6)
- [MCQ Q7](#mcq-q7)
- [MCQ Q8](#mcq-q8)
- [MCQ Q9](#mcq-q9)
- [MCQ Q10](#mcq-q10)
- [MCQ Q11](#mcq-q11)
- [MCQ Q12](#mcq-q12)
- [MCQ Q13](#mcq-q13)
- [MCQ Q14](#mcq-q14)
- [MCQ Q15](#mcq-q15)
- [MCQ Q16](#mcq-q16)
- [MCQ Q17](#mcq-q17)
- [MCQ Q18](#mcq-q18)
- [MCQ Q19](#mcq-q19)
- [MCQ Q20](#mcq-q20)
- [MCQ Q21](#mcq-q21)
- [MCQ Q22](#mcq-q22)
- [MCQ Q23](#mcq-q23)
- [MCQ Q24](#mcq-q24)
- [MCQ Q25](#mcq-q25)
- [MCQ Q26](#mcq-q26)
- [MCQ Q27](#mcq-q27)
- [MCQ Q28](#mcq-q28)
- [MCQ Q29](#mcq-q29)
- [MCQ Q30](#mcq-q30)
- [MCQ Q31](#mcq-q31)
- [MCQ Q32](#mcq-q32)
- [MCQ Q33](#mcq-q33)
- [MCQ Q34](#mcq-q34)
- [MCQ Q35](#mcq-q35)
- [MCQ Q36](#mcq-q36)
- [MCQ Q37](#mcq-q37)
- [MCQ Q38](#mcq-q38)
- [MCQ Q39](#mcq-q39)
- [MCQ Q40](#mcq-q40)
- [MCQ Q41](#mcq-q41)
- [MCQ Q42](#mcq-q42)
- [MCQ Q43](#mcq-q43)
- [MCQ Q44](#mcq-q44)
- [MCQ Q45](#mcq-q45)
- [MCQ Q46](#mcq-q46)
- [MCQ Q47](#mcq-q47)
- [MCQ Q48](#mcq-q48)
- [MCQ Q49](#mcq-q49)
- [MCQ Q50](#mcq-q50)
- [MCQ Q51](#mcq-q51)
- [MCQ Q52](#mcq-q52)
- [MCQ Q53](#mcq-q53)
- [MCQ Q54](#mcq-q54)
- [MCQ Q55](#mcq-q55)
- [MCQ Q56](#mcq-q56)
- [MCQ Q57](#mcq-q57)
- [MCQ Q58](#mcq-q58)
- [MCQ Q59](#mcq-q59)
- [MCQ Q60](#mcq-q60)
- [MCQ Q61](#mcq-q61)
- [MCQ Q62](#mcq-q62)
- [MCQ Q63](#mcq-q63)
- [MCQ Q64](#mcq-q64)
- [MCQ Q65](#mcq-q65)
- [MCQ Q66](#mcq-q66)
- [MCQ Q67](#mcq-q67)
- [MCQ Q68](#mcq-q68)
- [MCQ Q69](#mcq-q69)
- [MCQ Q70](#mcq-q70)
- [MCQ Q71](#mcq-q71)
- [MCQ Q72](#mcq-q72)
- [MCQ Q73](#mcq-q73)
- [MCQ Q74](#mcq-q74)
- [MCQ Q75](#mcq-q75)
- [MCQ Q76](#mcq-q76)
- [MCQ Q77](#mcq-q77)
- [MCQ Q78](#mcq-q78)
- [MCQ Q79](#mcq-q79)
- [MCQ Q80](#mcq-q80)
- [MCQ Q81](#mcq-q81)
- [MCQ Q82](#mcq-q82)
- [MCQ Q83](#mcq-q83)
- [MCQ Q84](#mcq-q84)
- [MCQ Q85](#mcq-q85)
- [MCQ Q86](#mcq-q86)
- [MCQ Q87](#mcq-q87)
- [MCQ Q88](#mcq-q88)
- [MCQ Q89](#mcq-q89)
- [MCQ Q90](#mcq-q90)
- [MCQ Q91](#mcq-q91)
- [MCQ Q92](#mcq-q92)
- [MCQ Q93](#mcq-q93)
- [MCQ Q94](#mcq-q94)
- [MCQ Q95](#mcq-q95)
- [MCQ Q96](#mcq-q96)
- [MCQ Q97](#mcq-q97)
- [MCQ Q98](#mcq-q98)
- [MCQ Q99](#mcq-q99)
- [MCQ Q100](#mcq-q100)

### MCQ Q1

In the context of the CAP theorem, which property is typically sacrificed in a distributed relational database that uses synchronous replication across multiple regions?

- **A)** Consistency
- **B)** Availability
- **C)** Partition Tolerance
- **D)** Durability

> **Answer: B** — In the event of a network partition, a system favoring consistency must stop accepting writes to avoid stale data, thereby sacrificing availability.

### MCQ Q2

Which load balancing algorithm is most effective at handling requests when backend servers have varying hardware specifications?

- **A)** Round Robin
- **B)** Least Connections
- **C)** Weighted Least Connections
- **D)** IP Hash

> **Answer: C** — This algorithm accounts for both the current load and the relative capacity of each server to ensure more powerful nodes handle more traffic.

### MCQ Q3

What is the primary purpose of a “Dead Letter Queue” (DLQ) in a messagedriven architecture?

- **A)** To store messages that have been successfully processed for auditing.
- **B)** To handle messages that cannot be processed after a maximum number of retries.
- **C)** To act as a buffer during high-traffic spikes.
- **D)** To route messages to multiple subscribers simultaneously.

> **Answer: B** — A DLQ prevents problematic messages from clogging the main pipeline by isolating them for manual inspection or later troubleshooting.

### MCQ Q4

In a PACELC-based system, if a partition (P) occurs, the system chooses between Availability (A) and Consistency (C); Else (E), the system chooses between:

- **A)** Latency (L) and Consistency (C)
- **B)** Latency (L) and Durability (D)
- **C)** Scalability (S) and Efficiency (E)
- **D)** Maintainability (M) and Reliability (R)

> **Answer: A** — PACELC extends CAP by stating that during normal operation, a system must trade off between low latency and high consistency.

### MCQ Q5

Which mechanism is commonly used to maintain membership lists and detect node failures in a peer-to-peer distributed system like Cassandra?

- **A)** Paxos Consensus
- **B)** Gossip Protocol
- **C)** Two-Phase Commit
- **D)** Heartbeat via Centralized Coordinator

> **Answer: B** — Gossip protocol is a decentralized, eventual consistency-based communication method that allows nodes to share state and health information efficiently.

### MCQ Q6

When implementing a write-through cache strategy, where is the data written?

- **A)** To the cache first, and then asynchronously to the database.
- **B)** To the database only, and the cache is invalidated.
- **C)** To both the cache and the database simultaneously.
- **D)** To the cache only, and only written to the database when the cache item is evicted.

> **Answer: C** — Write-through ensures data consistency between the cache and the database but introduces higher write latency because both stores must be updated.

### MCQ Q7

Which of the following is an example of “Vertical Scaling”?

- **A)** Adding more instances of a microservice behind a load balancer.
- **B)** Increasing the RAM and CPU of an existing database server.
- **C)** Sharding a database across multiple physical machines.
- **D)** Using a Content Delivery Network (CDN) to serve static assets.

> **Answer: B** — Vertical scaling (scaling up) involves adding resources to a single node, whereas horizontal scaling (scaling out) involves adding more nodes to the system.

### MCQ Q8

In a microservices architecture, what is the primary role of a “Circuit Breaker”?

- **A)** To encrypt traffic between services.
- **B)** To prevent a failing service from causing a cascading failure across the system.
- **C)** To load balance requests between different versions of a service.
- **D)** To provide a centralized log of all service-to-service communication.

> **Answer: B** — It monitors for failures and “trips” to stop calls to an unresponsive service, allowing it time to recover while providing a fallback response to the caller.

### MCQ Q9

Which database indexing structure is most commonly used in RDBMS like MySQL (InnoDB) to support efficient range queries?

- **A)** Hash Table
- **B)** LSM-Tree
- **C)** B+ Tree
- **D)** Inverted Index

> **Answer: C** — B+ Trees maintain sorted data and have linked leaf nodes, making them highly efficient for both point lookups and range scans.

### MCQ Q10

What problem does “Consistent Hashing” primarily solve in a distributed caching system?

- **A)** It ensures that all cache keys are encrypted.
- **B)** It minimizes the number of keys that need to be remapped when a node is added or removed.
- **C)** It guarantees that data is replicated to all nodes in the cluster.
- **D)** It reduces the latency of cross-region data transfers.

> **Answer: B** — By mapping both keys and nodes to a circular hash space, it prevents the massive “cache miss” storm that occurs with traditional modulo-based hashing.

### MCQ Q11

Which of the following is a limitation of the “Two-Phase Commit” (2PC) protocol?

- **A)** It cannot guarantee atomicity.
- **B)** It is a blocking protocol that can lead to resource contention and reduced throughput.
- **C)** It does not work with relational databases.
- **D)** It requires the use of a NoSQL database as a coordinator.

> **Answer: B** — If the coordinator fails, participants may remain locked indefinitely, making it brittle and difficult to scale in high-availability environments.

### MCQ Q12

What is the main advantage of using an “Idempotency Key” in API design?

- **A)** It compresses the request payload.
- **B)** It allows a client to safely retry a request without performing the same operation twice.
- **C)** It speeds up the database query execution time.
- **D)** It prevents Cross-Site Scripting (XSS) attacks.

> **Answer: B** — It ensures that duplicate requests (e.g., due to network timeouts) result in the same state, which is critical for financial transactions.

### MCQ Q13

In the context of HTTP, what is a “Sticky Session”?

- **A)** A session that never expires.
- **B)** The practice of mapping a user’s session to a specific server for the duration of the visit.
- **C)** A session stored exclusively in the browser’s local storage.
- **D)** A method of keeping a TCP connection open indefinitely.

> **Answer: B** — This ensures that stateful applications function correctly by routing all requests from a single user to the same backend instance.

### MCQ Q14

Which protocol is typically used for real-time, bidirectional, persistent communication between a client and a server?

- **A)** HTTP/1.1
- **B)** WebSockets
- **C)** REST
- **D)** SMTP

> **Answer: B** — WebSockets provide a full-duplex communication channel over a single TCP connection, ideal for chat apps and live updates.

### MCQ Q15

What is the purpose of a “Read Replica” in a database architecture?

- **A)** To provide a live backup in case the primary database fails.
- **B)** To offload read traffic from the primary database to improve performance.
- **C)** To handle all write operations for a specific geographic region.
- **D)** To store historical data that is no longer needed in the primary database.

> **Answer: B** — By distributing read queries among replicas, the primary database can focus on write operations, increasing overall system throughput.

### MCQ Q16

In a Log-Structured Merge-Tree (LSM-Tree), why are fragmented “SSTables” periodically merged via “Compaction”?

- **A)** To synchronize data with the cache.
- **B)** To remove deleted/overwritten keys and reduce the number of files to search during reads.
- **C)** To verify the checksums of the stored data.
- **D)** To move data from memory to disk.

> **Answer: B** — Compaction improves read performance by reducing the number of files that must be checked and reclaiming disk space from obsolete data.

### MCQ Q17

Which architectural pattern involves capturing all changes to an application state as a sequence of events?

- **A)** Microkernel
- **B)** Event Sourcing
- **C)** Layered Architecture
- **D)** Monolithic Architecture

> **Answer: B** — Event sourcing ensures that the current state can be reconstructed by replaying the immutable log of events, providing a perfect audit trail.

### MCQ Q18

What does “TLS Termination” refer to in the context of a Load Balancer?

- **A)** Revoking an SSL certificate.
- **B)** Decrypting encrypted traffic at the load balancer before sending it to the backend servers.
- **C)** Blocking all non-HTTPS traffic.
- **D)** Generating new private keys for every request.

> **Answer: B** — This reduces the CPU overhead on backend servers by offloading the intensive decryption process to the load balancer.

### MCQ Q19

Which of the following is a “Pull” based monitoring system?

- **A)** Prometheus
- **B)** Graphite
- **C)** StatsD
- **D)** InfluxDB (Standard Telegraf setup)

> **Answer: A** — Prometheus scrapes metrics from targets by hitting an HTTP endpoint, rather than waiting for services to push data to it.

### MCQ Q20

What is the “Thundering Herd” problem in caching?

- **A)** When many requests for the same expired cache key hit the database simultaneously.
- **B)** When a database becomes too large to fit in memory.
- **C)** When a load balancer fails and all traffic shifts to a single server.
- **D)** When a cache is filled with irrelevant data.

> **Answer: A** — This happens when a hot cache key expires, causing all concurrent requests to fall back to the origin database, potentially crashing it.

### MCQ Q21

In a microservices environment, what is the “Saga Pattern” used for?

- **A)** Aggregating logs from different services.
- **B)** Managing distributed transactions and maintaining data consistency across services.
- **C)** Implementing OAuth2 authentication.
- **D)** Auto-scaling services based on CPU usage.

> **Answer: B** — A Saga manages transactions via a sequence of local transactions and compensation logic to handle failures in a distributed system.

### MCQ Q22

Which type of database is most suitable for representing highly interconnected data, such as a social network’s friend graph?

- **A)** Key-Value Store
- **B)** Document Store
- **C)** Graph Database
- **D)** Column-Family Store

> **Answer: C** — Graph databases like Neo4j are optimized for traversing relationships between entities with constant-time complexity for many hop queries.

### MCQ Q23

What is the primary benefit of using a “Content Delivery Network” (CDN)?

- **A)** It improves the security of the origin server.
- **B)** It reduces latency by serving content from a location geographically closer to the user.
- **C)** It provides a relational interface for NoSQL data.
- **D)** It automatically shards the application database.

> **Answer: B** — By caching content at “Edge” locations, CDNs significantly reduce the physical distance data must travel.

### MCQ Q24

Under the PACELC theorem, what is a system like DynamoDB configured for “Eventual Consistency” choosing during a non-partitioned state?

- **A)** Consistency (C) over Latency (L)
- **B)** Latency (L) over Consistency (C)
- **C)** Availability (A) over Partition Tolerance (P)
- **D)** Durability (D) over Efficiency (E)

> **Answer: B** — Even when the network is healthy, eventual consistency allows for faster responses by not waiting for all replicas to acknowledge a write.

### MCQ Q25

What is “Database Sharding”?

- **A)** Creating an exact copy of the database.
- **B)** Horizontal partitioning of a database to store different rows of data on different servers.
- **C)** Converting a relational database into a JSON format.
- **D)** Encrypting specific columns within a database table.

> **Answer: B** — Sharding divides a large dataset into smaller pieces, allowing the system to scale horizontally and handle more traffic than a single server could.

### MCQ Q26

In a distributed system, which principle states that in the case of a network partition (P), a system must choose between availability (A) and consistency (C)?

- **A)** Amdahl’s Law
- **B)** CAP Theorem
- **C)** PACELC Theorem
- **D)** Moore’s Law

> **Answer: B** — The CAP Theorem asserts that a distributed data store can only provide two out of three guarantees: Consistency, Availability, and Partition Tolerance.

### MCQ Q27

According to the PACELC theorem, if a system is operating normally (no partition), what is the primary trade-off?

- **A)** Availability vs. Consistency
- **B)** Throughput vs. Latency
- **C)** Latency vs. Consistency
- **D)** Durability vs. Scalability

> **Answer: C** — PACELC extends CAP by stating that “Else” (no partition), the system must trade off between Latency and Consistency.

### MCQ Q28

Which load balancing algorithm is most effective for handling requests when backend servers have varying hardware specifications?

- **A)** Round Robin
- **B)** Weighted Round Robin
- **C)** Least Connections
- **D)** IP Hash

> **Answer: B** — Weighted Round Robin allows administrators to assign higher capacities to more powerful servers, ensuring they receive a proportional share of traffic.

### MCQ Q29

What is the primary purpose of a “Gossip Protocol” in a distributed system like Cassandra or DynamoDB?

- **A)** To synchronize data clocks
- **B)** To achieve strict serializability
- **C)** To propagate cluster state and member health
- **D)** To encrypt data in transit

> **Answer: C** — Gossip protocols are peer-to-peer communication mechanisms used to share decentralized information about node membership and health status.

### MCQ Q30

Which consistency model ensures that if process A has communicated to process B that it has updated a data item, a subsequent access by process B will return the updated value?

- **A)** Eventual Consistency
- **B)** Causal Consistency
- **C)** Weak Consistency
- **D)** Monotonic Read Consistency

> **Answer: B** — Causal consistency ensures that operations that are potentially related by cause are seen by all processes in the same order.

### MCQ Q31

In a Write-Ahead Log (WAL) system, when is data considered committed and durable?

- **A)** After it is written to the in-memory cache
- **B)** After it is successfully appended to the log on non-volatile storage
- **C)** After the index is updated
- **D)** After the client receives an HTTP 200 OK

> **Answer: B** — The WAL ensures atomicity and durability by forcing changes to be recorded on permanent storage before they are applied to the database.

### MCQ Q32

Which of the following is an example of a “Pull-based” monitoring system?

- **A)** Graphite
- **B)** StatsD
- **C)** Prometheus
- **D)** Splunk

> **Answer: C** — Prometheus scrapes (pulls) metrics from targets at designated intervals, whereas systems like StatsD rely on applications pushing data to them.

### MCQ Q33

What is the main disadvantage of using “Sticky Sessions” (Session Affinity) in a load balancer?

- **A)** It increases the encryption overhead
- **B)** It can lead to uneven load distribution if a few sessions are very heavy
- **C)** It requires the use of UDP
- **D)** It prevents the use of HTTPS

> **Answer: B** — Sticky sessions can cause “hotspots” where one server becomes overloaded because it is tied to several resource-intensive users while others remain idle.

### MCQ Q34

Using a Bloom Filter in a database (like BigTable or RocksDB) helps to:

- **A)** Sort the data alphabetically
- **B)** Guarantee that a key exists in a SSTable
- **C)** Reduce unnecessary disk seeks for non-existent keys
- **D)** Encrypt sensitive columns

> **Answer: C** — Bloom filters are probabilistic data structures that can quickly tell you if a key “definitely does not exist” in a file, saving expensive I/O.

### MCQ Q35

In a microservices architecture, what is a “Sidecar” pattern?

- **A)** A secondary database for backups
- **B)** A helper container that runs alongside the application container to provide peripheral tasks
- **C)** A load balancer located in a different region
- **D)** A synchronization lock for distributed transactions

> **Answer: B** — The Sidecar pattern attaches a separate process/container (like a service mesh proxy) to an application to handle concerns like logging, monitoring, or security.

### MCQ Q36

Which database partitioning strategy involves hashing a key to determine the target shard?

- **A)** Range-based Partitioning
- **B)** Directory-based Partitioning
- **C)** Consistent Hashing
- **D)** Vertical Partitioning

> **Answer: C** — Consistent hashing maps keys to shards using a hash function, minimizing data reorganization when nodes are added or removed.

### MCQ Q37

What is the “Thundering Herd” problem in caching?

- **A)** When many requests for the same expired cache key all hit the database simultaneously
- **B)** When a hard drive fails in a RAID array
- **C)** When the cache grows larger than the available RAM
- **D)** When the database exceeds its connection limit

> **Answer: A** — This occurs when a popular cached item expires, and the spike in concurrent requests overwhelms the backend data source before the cache can be repopulated.

### MCQ Q38

When implementing a distributed lock, which of the following is required to ensure safety against process pauses (like GC)?

- **A)** A very long TTL
- **B)** A fencing token (monotonically increasing version)
- **C)** A sticky session
- **D)** Symmetric encryption

> **Answer: B** — Fencing tokens ensure that if a process resumes after a long pause, its write request will be rejected by the storage layer if a newer lock has already been issued.

### MCQ Q39

Which protocol is typically used for low-latency, full-duplex communication between a client and a server?

- **A)** HTTP/1.1
- **B)** gRPC (over HTTP/2)
- **C)** WebSockets
- **D)** Both B and C

> **Answer: D** — Both gRPC and WebSockets support bi-directional, persistent communication, which reduces the overhead of repeatedly opening connections.

### MCQ Q40

What is the primary benefit of using a Content Delivery Network (CDN)?

- **A)** To reduce database query complexity
- **B)** To reduce latency by serving content from edge locations closer to the user
- **C)** To provide ACID transactions at the edge
- **D)** To encrypt user passwords

> **Answer: B** — CDNs store copies of static or dynamic content in geographically distributed “edge” servers to minimize the physical distance data travels.

### MCQ Q41

In the context of message queues, what does “At-least-once delivery” imply?

- **A)** Messages are never lost, but may be delivered multiple times
- **B)** Each message is guaranteed to be delivered exactly one time
- **C)** Messages may be lost, but are never duplicated
- **D)** Messages are delivered in the exact order they were sent

> **Answer: A** — At-least-once delivery ensures the consumer receives the message, but due to potential acknowledgement failures, the consumer must be idempotent to handle duplicates.

### MCQ Q42

Which of the following describes a “Read-Through” cache strategy?

- **A)** The application updates the DB, then invalidates the cache
- **B)** The application always writes to the cache first, and the cache synchronously updates the DB
- **C)** The application asks the cache for data; if missing, the cache fetches it from the DB and returns it
- **D)** The application writes to the DB and the DB pushes data to the cache

> **Answer: C** — In Read-Through, the cache library or provider is responsible for fetching missing data from the database, simplifying the application logic.

### MCQ Q43

What is the purpose of an “API Gateway” in microservices?

- **A)** To store the application’s primary state
- **B)** To act as a single entry point for routing, authentication, and rate limiting
- **C)** To compile the source code of various services
- **D)** To manage the hardware layer of a private cloud

> **Answer: B** — An API Gateway centralizes cross-cutting concerns like security and request routing, shielding the internal microservice structure from the client.

### MCQ Q44

Which of the following is a symptom of “Write Amplification” in SSDs or LSMtrees?

- **A)** Reads are faster than writes
- **B)** More data is written to the physical storage than the logical amount requested by the application
- **C)** The cache hit ratio increases over time
- **D)** The network bandwidth is saturated by heartbeat signals

> **Answer: B** — Write amplification occurs when background operations (like compaction or garbage collection) cause the system to rewrite data multiple times.

### MCQ Q45

Quorum-based voting (N/2 + 1) in a distributed system is primarily used to:

- **A)** Increase the speed of light
- **B)** Ensure agreement and prevent “split-brain” scenarios
- **C)** Compress large binary objects
- **D)** Encrypt data using public keys

> **Answer: B** — Quorums ensure that only one set of nodes can make progress or claim to be the leader, maintaining consistency during network partitions.

### MCQ Q46

Which HTTP method is considered “Idempotent”?

- **A)** POST
- **B)** PATCH
- **C)** PUT
- **D)** CONNECT

> **Answer: C** — Idempotent operations (like PUT or DELETE) can be called multiple times with the same result as a single call, whereas POST usually creates a new resource each time.

### MCQ Q47

What is the “Saga Pattern” used for?

- **A)** Managing long-lived distributed transactions across multiple microservices
- **B)** Speeding up CSS rendering in browsers
- **C)** Storing historical log data in cold storage
- **D)** Round-robin DNS resolution

> **Answer: A** — Sagas manage data consistency in microservices by using a sequence of local transactions and compensating transactions in case of failure.

### MCQ Q48

In a “Fan-out” architecture using a Message Broker, how are messages delivered?

- **A)** One message is sent to exactly one consumer in a round-robin fashion
- **B)** A single message is broadcast to all queues or subscribers interested in that topic
- **C)** All messages are stored in a single global table
- **D)** Messages are sent only to the consumer with the lowest latency

> **Answer: B** — Fan-out allows a single event (like a new order) to trigger multiple independent processes (like billing, shipping, and notifications) simultaneously.

### MCQ Q49

What does “Pre-signed URLs” in cloud storage (like S3) allow you to do?

- **A)** Encrypt the entire bucket with a physical key
- **B)** Give temporary, time-limited access to a specific object without sharing credentials
- **C)** Permanently make a private bucket public
- **D)** Speed up the upload process using UDP

> **Answer: B** — Pre-signed URLs grant specific permissions to a user to upload or download an object using a URL that expires after a set duration.

### MCQ Q50

Which mechanism is used to prevent one user from overwhelming a service with too many requests?

- **A)** Service Discovery
- **B)** Rate Limiting
- **C)** Horizontal Scaling
- **D)** Data Normalization

> **Answer: B** — Rate limiting restricts the number of requests a client can make within a specific time window to ensure system stability and fair usage.

### MCQ Q51

Which protocol provides a full-duplex communication channel over a single TCP connection, reducing overhead compared to HTTP polling?

- **A)** Long Polling
- **B)** WebSockets
- **C)** Server-Sent Events (SSE)
- **D)** HTTP Keep-Alive

> **Answer: B** — WebSockets allow for persistent, bidirectional communication, making them more efficient than polling for real-time applications like chat or live dashboards.

### MCQ Q52

In a distributed system designed according to the PACELC theorem, if there is a network partition (P), the system must choose between:

- **A)** Latency and Consistency
- **B)** Availability and Consistency
- **C)** Partition Tolerance and Durability
- **D)** Scalability and Reliability

> **Answer: B** — PACELC extends CAP by stating that in the presence of a partition (P), one must choose between availability (A) and consistency (C).

### MCQ Q53

Which caching strategy involves the application updating the database first and then invalidating the cache entry?

- **A)** Write-through
- **B)** Write-behind
- **C)** Cacheaside
- **D)** Refresh-ahead

> **Answer: C** — In the Cache-aside pattern, the application is responsible for reading from/writing to the database and ensuring the cache is invalidated to prevent stale data.

### MCQ Q54

What is the primary purpose of a Bloom Filter in a database system like Apache Cassandra?

- **A)** To store the actual data in a compressed format
- **B)** To ensure strong consistency across replicas
- **C)** To quickly check if a key might exist in an SSTable, avoiding unnecessary disk I/O
- **D)** To sort keys in alphabetical order for range queries

> **Answer: C** — A Bloom Filter is a space-efficient probabilistic data structure used to reduce expensive disk lookups for non-existent keys.

### MCQ Q55

Which of the following consistency models guarantees that if Process A communicates a value to Process B, any subsequent access by Process B will return that value?

- **A)** Eventual Consistency
- **B)** Causal Consistency
- **C)** Monotonic Reads
- **D)** Strictly Serializable

> **Answer: B** — Causal consistency ensures that operations that are physically or logically related are seen in the same order by all nodes.

### MCQ Q56

In the context of microservices, what is the primary role of the “Circuit Breaker” pattern?

- **A)** To distribute traffic evenly across different service instances
- **B)** To prevent a failure in one service from cascading to other services
- **C)** To encrypt data in transit between internal services
- **D)** To provide a centralized log for debugging service failures

> **Answer: B** — A Circuit Breaker monitors for failures and “trips” to stop requests to a failing service, allowing it time to recover and protecting the rest of the system.

### MCQ Q57

Which consensus algorithm uses the concepts of “Lease” and “Term” and is often used by systems like etcd?

- **A)** Paxos
- **B)** Gossip Protocol
- **C)** Raft
- **D)** Two-Phase Commit (2PC)

> **Answer: C** — Raft is a consensus algorithm designed for understandability that manages a replicated log through leader election and terms.

### MCQ Q58

What is “Write Amplification” in the context of SSDs and certain Database engines?

- **A)** The process of increasing the volume of a database to improve performance
- **B)** The phenomenon where the actual amount of physical data written is a multiple of the logical data intended to be written
- **C)** A technique used to replicate data across multiple availability zones simultaneously
- **D)** The ability to write to multiple master nodes at once

> **Answer: B** — Write amplification occurs when background operations like garbage collection or log-structured updates cause more physical writes than requested, potentially shortening drive lifespan.

### MCQ Q59

Which of these is a characteristic of a “Gossip Protocol”?

- **A)** It relies on a central master node to disseminate information
- **B)** It is a deterministic way to reach consensus in under 10ms
- **C)** It is decentralized and utilizes random peer-to-peer communication to spread information
- **D)** It requires a global lock on all data before an update can occur

> **Answer: C** — Gossip protocols are epidemic-like communication methods used for cluster membership and failure detection in highly distributed systems.

### MCQ Q60

When using “Consistent Hashing,” what happens when a new node is added to a cluster of 𝑁 nodes?

- **A)** All keys must be rehashed and moved to new locations
- **B)** On average, only 1/𝑁 fraction of the keys need to be remapped
- **C)** The hash ring is wiped and rebuilt from scratch
- **D)** Only the keys on the node physically adjacent to the new node are deleted

> **Answer: B** — Consistent hashing minimizes reorganization by ensuring that only a small portion of keys move when the number of slots changes.

### MCQ Q61

Which HTTP status code should a server return when a client exceeds its rate limit?

- **A)** 401 Unauthorized
- **B)** 403 Forbidden
- **C)** 429 Too Many Requests
- **D)** 503 Service Unavailable

> **Answer: C** — The 429 status code is specifically reserved for rate limiting (throttling) to inform the client they have sent too many requests in a given time.

### MCQ Q62

In a “Read-heavy” system with a high cache hit ratio, what is the most likely bottleneck?

- **A)** Disk I/O
- **B)** Database write locks
- **C)** Network bandwidth or Memory
- **D)** CPU cycles for data compression

> **Answer: C** — For read-heavy systems where most data comes from memory (cache), the limits are typically the speed of the network or the amount of available RAM.

### MCQ Q63

What is the “Saga Pattern” primarily used for in microservices?

- **A)** Managing distributed transactions without using a two-phase commit
- **B)** Generating unique IDs across a distributed cluster
- **C)** Storing large binary objects (BLOBs)
- **D)** Encrypting sensitive user data at rest

> **Answer: A** — The Saga pattern manages local transactions and compensation logic across multiple services to maintain data consistency in distributed systems.

### MCQ Q64

Which technology is best suited for implementing a “Message Queue” with strictly ordered, persistent, and replayable event streams?

- **A)** Redis Pub/Sub
- **B)** RabbitMQ
- **C)** Apache Kafka
- **D)** AWS SNS

> **Answer: C** — Kafka is designed as a distributed commit log that allows for partitioned, ordered, and durable message storage that can be replayed by consumers.

### MCQ Q65

In database sharding, what is a “Hot Key” problem?

- **A)** When a key is accessed so frequently that the shard containing it becomes a performance bottleneck
- **B)** When a key is deleted but still exists in the cache
- **C)** When the encryption key for a shard is lost
- **D)** When two keys hash to the same value in a hash map

> **Answer: A** — A hot key occurs when disproportionate traffic is directed to a single shard, negating the benefits of horizontal scaling.

### MCQ Q66

What does the “L” in PACELC stand for?

- **A)** Load-balancing
- **B)** Latency
- **C)** Logging
- **D)** Linearizability

> **Answer: B** — PACELC stands for: if Partition, choose Availability or Consistency; Else (no partition), choose Latency or Consistency.

### MCQ Q67

Which of the following is an “Idempotent” operation?

- **A)** POST /orders (creates a new order each time)
- **B)** PUT /user/123 (updates user 123 with the same data)
- **C)** PATCH /balance?inc=10 (increments balance by 10)
- **D)** DELETE /last_log_entry (deletes the most recent entry)

> **Answer: B** — Idempotency means that performing the same operation multiple times has the same effect as performing it once, which is a standard property of the PUT method.

### MCQ Q68

A “Reverse Proxy” is most commonly used for:

- **A)** Storing user passwords securely
- **B)** Load balancing, SSL termination, and caching
- **C)** Querying the database directly for the client
- **D)** Managing the application’s local state

> **Answer: B** — Reverse proxies sit in front of web servers to distribute traffic, handle security protocols, and improve performance through caching.

### MCQ Q69

What is the primary advantage of using “Protocol Buffers” (Protobuf) over JSON for internal service communication?

- **A)** Protobuf is human-readable and easier to debug
- **B)** Protobuf is binary, resulting in smaller payloads and faster serialization
- **C)** Protobuf does not require a schema
- **D)** Protobuf is natively supported by all web browsers without libraries

> **Answer: B** — Protobuf is a compact binary format that is significantly more efficient than text-based formats like JSON or XML for high-performance systems.

### MCQ Q70

Which database isolation level prevents “Dirty Reads” but allows “Non-repeatable Reads”?

- **A)** Read Uncommitted
- **B)** Read Committed
- **C)** Repeatable Read
- **D)** Serializable

> **Answer: B** — Read Committed ensures that any data read has been committed at the moment it is read, but the data can change between two reads in the same transaction.

### MCQ Q71

What is the “Sidecar Pattern” in a Kubernetes-based microservices architecture?

- **A)** Running two instances of the same service for high availability
- **B)** A container that runs alongside the main application container to provide helper features like logging or a service mesh proxy
- **C)** A method of sharding the database based on geographical location
- **D)** A disaster recovery strategy where a “side” data center is kept on standby

> **Answer: B** — The Sidecar pattern allows for the separation of concerns by placing supporting tasks in a separate container within the same Pod.

### MCQ Q72

In a Content Delivery Network (CDN), what is “Edge Computing”?

- **A)** Building data centers at the edge of a cliff for cooling
- **B)** Moving the entire database to the client’s browser
- **C)** Executing logic (like authentication or image manipulation) at the CDN node closest to the user
- **D)** Calculating the maximum capacity of a network link

> **Answer: C** — Edge computing reduces latency by processing data and requests at the network edge, closer to the end user, rather than at a central origin server.

### MCQ Q73

Which of the following is a “shared-nothing” architecture benefit?

- **A)** It allows all nodes to access a single centralized disk for simplicity
- **B)** It eliminates single points of contention and allows for high horizontal scalability
- **C)** It makes it easier to manage global state and locks
- **D)** It reduces the need for data replication because all nodes see the same data

> **Answer: B** — Shared-nothing architectures involve independent nodes, which makes scaling easier because there is no central resource that everyone must wait for.

### MCQ Q74

What is the purpose of a “Nonce” in web security?

- **A)** To store the user’s session ID permanently
- **B)** To encrypt the database password
- **C)** A “number used once” to prevent replay attacks
- **D)** To identify the geographic location of a client

> **Answer: C** — A nonce is a unique, one-time-use value that ensures a request cannot be intercepted and maliciously reused by an attacker.

### MCQ Q75

Which load balancing algorithm is best if you want to ensure that a specific user always hits the same backend server (Session Stickiness)?

- **A)** Round Robin
- **B)** Least Connections
- **C)** IP Hash
- **D)** Random Selection

> **Answer: C** — IP Hash uses the client’s IP address to consistently map them to the same backend server, maintaining session state without a centralized session store.

### MCQ Q76

In a distributed system, if a partition occurs (the “P” in CAP), which trade-off does the PACELC theorem emphasize for the system’s behavior when it is operating normally (Else)?

- **A)** Latency vs. Consistency
- **B)** Availability vs. Durability
- **C)** Throughput vs. Scalability
- **D)** Security vs. Performance

> **Answer: A** — PACELC extends CAP by stating that in the absence of partitions, a system must trade off between latency and consistency.

### MCQ Q77

Which of the following best describes the “Thundering Herd” problem?

- **A)** A sudden surge in user registrations crashing the authentication service.
- **B)** Many processes simultaneously requesting a specific data piece from a database when a cache entry expires.
- **C)** A DDoS attack targeting the network layer of a load balancer.
- **D)** Microservices failing in a chain due to a lack of circuit breakers.

> **Answer: B** — This occurs when a hot cache key expires and multiple workers all attempt to regenerate the data from the database at once, causing a spike in load.

### MCQ Q78

Which HTTP status code should a server return to implement “Backpressure” when it is overwhelmed with requests?

- **A)** 404 Not Found
- **B)** 500 Internal Server Error
- **C)** 429 Too Many Requests
- **D)** 502 Bad Gateway

> **Answer: C** — The 429 status code explicitly tells the client they have exceeded their rate limit and should slow down or retry later.

### MCQ Q79

What is the primary purpose of a “Gossip Protocol” in a distributed system like Apache Cassandra?

- **A)** To synchronize clocks across nodes using NTP.
- **B)** To achieve consensus on a single value using Paxos.
- **C)** To distribute state and detect node failures in a decentralized manner.
- **D)** To encrypt data transmission between the client and the cluster.

> **Answer: C** — Gossip protocols allow nodes to share membership and status information peer-to-peer, ensuring the cluster achieves eventual convergence on the state of all nodes.

### MCQ Q80

In a microservices architecture, which pattern is most effective for ensuring data consistency across multiple services without using distributed transactions (2PC)?

- **A)** Sidecar Pattern
- **B)** Saga Pattern
- **C)** Ambassador Pattern
- **D)** Strangler Fig Pattern

> **Answer: B** — The Saga pattern manages distributed transactions through a sequence of local transactions and compensating actions to undo changes if a failure occurs.

### MCQ Q81

When using Consistent Hashing, what is the main benefit of introducing “Virtual Nodes”?

- **A)** It increases the security of the hash function against collisions.
- **B)** It reduces the memory overhead of the hash ring.
- **C)** It ensures a more uniform distribution of data across physical nodes with different capacities.
- **D)** It eliminates the need for a load balancer.

> **Answer: C** — Virtual nodes map a single physical node to multiple points on the ring, which smooths out the distribution of keys and prevents “hot spots.”

### MCQ Q82

Which of the following database isolation levels offers the highest protection against anomalies but results in the lowest concurrency?

- **A)** Read Committed
- **B)** Repeatable Read
- **C)** Serializable
- **D)** Snapshot Isolation

> **Answer: C** — Serializable isolation ensures that the result of concurrent transactions is the same as if they were executed one after another, preventing all race conditions.

### MCQ Q83

What is the primary difference between a Forward Proxy and a Reverse Proxy?

- **A)** A forward proxy handles outgoing requests for clients, while a reverse proxy handles incoming requests for servers.
- **B)** A reverse proxy is used for caching, while a forward proxy is only for security.
- **C)** A forward proxy resides in the cloud, while a reverse proxy resides on-premises.
- **D)** A reverse proxy modifies the IP header, while a forward proxy modifies the application payload.

> **Answer: A** — Forward proxies act on behalf of clients to access the internet, whereas reverse proxies sit in front of backend servers to provide load balancing and security.

### MCQ Q84

Which technology is most appropriate for establishing a low-latency, full-duplex communication channel between a client and a server?

- **A)** HTTP Long Polling
- **B)** Server-Sent Events (SSE)
- **C)** WebSockets
- **D)** REST over HTTP/1.1

> **Answer: C** — WebSockets provide a persistent, two-way connection that allows both the client and server to send data at any time without the overhead of HTTP headers for every message.

### MCQ Q85

In the context of database indexing, what is the main advantage of a B+ Tree over a standard B-Tree?

- **A)** B+ Trees are faster for single-point lookups.
- **B)** B+ Trees store data only in leaf nodes, making range scans more efficient via linked leaves.
- **C)** B+ Trees require less disk space.
- **D)** B+ Trees do not require rebalancing.

> **Answer: B** — Because all data/pointers are in the leaves and leaves are linked, range queries only need to find the start point and then traverse the linked list.

### MCQ Q86

Which of the following is a classic trade-off when choosing a Wide-Column store like HBase or Cassandra?

- **A)** High write throughput at the cost of complex join queries.
- **B)** Low storage costs at the cost of slow writes.
- **C)** Strong ACID compliance at the cost of vertical scalability.
- **D)** Simple schema design at the cost of high latency.

> **Answer: A** — Wide-column stores are optimized for massive write volumes and horizontal scaling but do not support relational joins, requiring data to be denormalized.

### MCQ Q87

A “Dead Letter Queue” (DLQ) in a message broker system is used to:

- **A)** Store messages that are too large for the main queue.
- **B)** Archive messages for historical auditing.
- **C)** Hold messages that could not be processed successfully after a certain number of attempts.
- **D)** Speed up the processing of high-priority messages.

> **Answer: C** — DLQs isolate problematic messages so they don’t block the main processing flow, allowing developers to inspect and debug the cause of failure.

### MCQ Q88

Which caching strategy involves writing data to the cache and the permanent data store simultaneously?

- **A)** Write-around
- **B)** Write-through
- **C)** Write-back
- **D)** Refresh-ahead

> **Answer: B** — Write-through ensures data consistency between the cache and the database by updating both at the same time, though it introduces higher write latency.

### MCQ Q89

In a System Design interview, when designing a system like Twitter, what is the “Fan-out” approach for timelines?

- **A)** Compressing old tweets to save space.
- **B)** Delivering a tweet to all of a user’s followers’ caches at write time.
- **C)** Fetching tweets from all followed users only when a user opens the app.
- **D)** Using a CDN to serve media content.

> **Answer: B** — Write-time fan-out (push model) pre-computes the timeline so that read requests are extremely fast, which is critical for high-read social platforms.

### MCQ Q90

What is the purpose of “Heartbeats” in a leader-based distributed system?

- **A)** To synchronize the system clock.
- **B)** To verify that the leader is still alive and prevent a new election.
- **C)** To transfer data snapshots between nodes.
- **D)** To encrypt the communication channel.

> **Answer: B** — If followers stop receiving heartbeats from the leader, they assume the leader has failed and initiate a new election to maintain availability.

### MCQ Q91

Which of these is a benefit of using a Content Delivery Network (CDN)?

- **A)** It reduces the load on the origin server by caching static and dynamic assets closer to the user.
- **B)** It provides a relational schema for unstructured data.
- **C)** It replaces the need for an Application Load Balancer.
- **D)** It automatically encrypts the database at rest.

> **Answer: A** — CDNs utilize edge locations to serve content to users with lower latency while shielding the origin server from high traffic volumes.

### MCQ Q92

When implementing a rate limiter, which algorithm allows for a sudden burst of traffic while maintaining a strict average rate?

- **A)** Fixed Window Counter
- **B)** Sliding Window Log
- **C)** Token Bucket
- **D)** Leaky Bucket

> **Answer: C** — The Token Bucket algorithm allows tokens to accumulate up to a certain burst size, which can be spent all at once, unlike the Leaky Bucket which enforces a constant output rate.

### MCQ Q93

In the Raft consensus algorithm, what happens if a follower receives a request from a candidate with an outdated term number?

- **A)** The follower updates its term and votes for the candidate.
- **B)** The follower ignores the request or rejects the vote.
- **C)** The follower initiates a new election.
- **D)** The follower shuts down to prevent data corruption.

> **Answer: B** — Raft uses monotonic term numbers to ensure that candidates with stale information cannot become leaders and overwrite committed log entries.

### MCQ Q94

What is the primary drawback of using a Multi-Master database replication setup?

- **A)** It creates a single point of failure.
- **B)** It does not support write operations.
- **C)** It requires complex conflict resolution logic when the same data is modified on different masters.
- **D)** It is slower for read operations than a single-master setup.

> **Answer: C** — Since writes can occur anywhere, the system must handle “write-write” conflicts, which often requires complex versioning or “last-write-wins” logic.

### MCQ Q95

Which DNS record type is used to map a domain name to an IPv4 address?

- **A)** AAAA Record
- **B)** CNAME Record
- **C)** MX Record
- **D)** A Record

> **Answer: D** — The “A” (Address) record maps a hostname to a 32-bit IPv4 address, whereas AAAA is for IPv6.

### MCQ Q96

What problem does the “Circuit Breaker” pattern solve in microservices?

- **A)** It prevents a failing service from causing a cascading failure across the entire system.
- **B)** It balances the load between different versions of a service.
- **C)** It encrypts traffic between internal services.
- **D)** It manages user sessions across multiple pods.

> **Answer: A** — By “tripping” the circuit when a service fails, the caller stops making requests to the failing service, allowing it time to recover and protecting other services.

### MCQ Q97

Which of the following is a characteristic of “Idempotency” in API design?

- **A)** The request always returns a 200 OK status.
- **B)** Making the same request multiple times has the same effect as making it once.
- **C)** The request is processed in less than 100 milliseconds.
- **D)** The request is only valid for a single use and expires immediately.

> **Answer: B** — Idempotent operations (like PUT or DELETE) ensure that repeated calls do not result in unintended side effects, such as double-charging a credit card.

### MCQ Q98

In Cloud Computing, “Auto-scaling” primarily addresses which concern?

- **A)** Security
- **B)** Elasticity
- **C)** Data Persistence
- **D)** Code Versioning

> **Answer: B** — Elasticity is the ability of a system to grow or shrink its resource allocation dynamically in response to changing demand.

### MCQ Q99

What is the main use case for a Bloom Filter in a database system like BigTable?

- **A)** To store the actual data values in a compressed format.
- **B)** To perform fast, exact joins between two tables.
- **C)** To quickly check if a key might exist in a large dataset, reducing unnecessary disk I/O.
- **D)** To generate unique UUIDs for new records.

> **Answer: C** — Bloom filters are space-efficient probabilistic data structures that can tell you if a key is “definitely not” in a set or “possibly” in a set, avoiding slow disk lookups.

### MCQ Q100

Which architectural style is characterized by “Everything is a Resource” and uses standard HTTP verbs?

- **A)** GraphQL
- **B)** gRPC
- **C)** SOAP
- **D)** REST

> **Answer: D** — Representational State Transfer (REST) is built around resources identified by URIs and manipulated using standard methods like GET, POST, PUT, and DELETE.

End of Book 
