# Ticket Booking System Design (BookMyShow/Ticketmaster-like)

**Difficulty Level:** ⭐⭐⭐ Hard  
**Tags:** `Distributed Systems`, `Concurrency`, `Database Design`, `Caching`, `High Availability`, `Real-time`, `Payments`, `API Design`

**File Purpose:** Interactive, multi-level learning resource for designing a ticket booking system. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that handles millions of bookings for concerts, movies, sports events, and live shows — solving the hardest challenge in e-commerce: selling a finite, non-fungible inventory to millions of simultaneous users without double-selling a single seat.

**Author:** System Design Documentation  
**Created:** April 4, 2026  
**Last Updated:** April 4, 2026  

---

## 🎓 Welcome to Ticket Booking System Design!

### What You're Going to Build

Imagine designing the backend for BookMyShow or Ticketmaster — the infrastructure that lets 2 million fans simultaneously rush to buy tickets the moment Taylor Swift announces a new tour. In those first 90 seconds, your system must:

- Show accurate, real-time seat availability maps
- Lock seats temporarily while users complete payment
- Process payment for hundreds of thousands of transactions per minute
- Guarantee zero double-bookings, even under extreme concurrent load
- Generate unique, fraud-proof tickets (QR codes, barcodes)
- Handle failures gracefully — if payment fails, release the seat immediately

By the end of this learning journey, you'll understand how to design a production-grade ticket booking system that:

- Serves **50M+ users** browsing events at the same time
- Handles **500K+ concurrent booking attempts** during peak (Taylor Swift on-sale)
- Processes **1M+ bookings/day** with **zero double-sells**
- Responds to seat availability queries in **<50ms**
- Achieves **99.99% uptime** (52 minutes downtime/year)
- Generates and validates fraud-proof digital tickets at venue entry

### 📚 Your Learning Path

```text
🟢 BEGINNER LEVEL (4-6 hours)
├─ Understand the seat reservation problem
├─ Learn why tickets are harder than shopping carts
├─ Build intuition with concert/movie theatre analogies
└─ Perfect for: New to system design

🟡 INTERMEDIATE LEVEL (6-8 hours)
├─ Master optimistic vs pessimistic locking strategies
├─ Learn distributed lock design with Redis
├─ Practice concurrency and idempotency patterns
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (8-12 hours)
├─ Production seat locking at scale (Ticketmaster's approach)
├─ Distributed saga for payment atomicity
├─ Queue-based fairness during flash sales
├─ Fraud detection, resale prevention, and GDPR compliance
└─ Perfect for: Senior engineers and architects
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of web requests (HTTP)
- Familiarity with databases (storing and retrieving data)
- No prior distributed systems experience needed!

**For Intermediate:**
- Comfortable with SQL databases and transactions (ACID)
- Familiar with Redis or caching concepts
- Understanding of REST APIs and HTTP status codes

**For Advanced:**
- Experience with distributed systems, CAP theorem
- Knowledge of message queues (Kafka, RabbitMQ)
- Familiarity with distributed locking and saga patterns

### 📊 What Makes This System Uniquely Challenging

Most e-commerce systems sell fungible items — if you buy the last red t-shirt in size M, any warehouse can ship it. Ticket systems are different:

| Challenge | Why It's Hard |
|-----------|---------------|
| **Non-fungible inventory** | Seat A7 in Row 3 is not the same as Seat A8. Each seat is unique. |
| **Time-bounded scarcity** | Once an event starts, unsold tickets have zero value. |
| **Flash sale load** | 10,000x normal traffic in seconds when Taylor Swift goes on sale |
| **Zero double-selling tolerance** | Selling the same seat twice causes a real-world conflict at the venue |
| **Payment atomicity** | Seat must be released if payment fails — but payment services can be slow |
| **Fairness expectations** | Users expect a queue, not first-come-first-served on luck of network latency |

💡 **Pro Tip:** In your interview, the moment you say "ticket systems have non-fungible inventory and require zero double-sells," you immediately demonstrate deep understanding of what makes this problem unique.

---

### 📖 Beginner's Glossary

Before diving in, here are the key terms you'll encounter throughout this document:

| Term | Plain English Explanation |
|------|--------------------------|
| **Seat Lock** | Temporarily reserving a seat for a user while they complete payment (like a 10-minute hold) |
| **Idempotency** | Making an operation safe to retry — booking the same seat twice should result in one booking, not two |
| **Optimistic Locking** | Assume no conflict, check at commit time — "I'll try, and rollback if someone beat me" |
| **Pessimistic Locking** | Assume conflict, lock upfront — "I'll grab the lock before I even start" |
| **Distributed Lock** | A lock that works across multiple servers (not just one process) |
| **ACID Transaction** | Database guarantees: Atomicity, Consistency, Isolation, Durability |
| **Saga Pattern** | Breaking a long transaction (book seat + charge card) into steps with compensating actions if a step fails |
| **QR Code / Barcode** | The scannable code on your ticket that venues use to verify authenticity |
| **Flash Sale** | Sudden spike in demand (e.g., Taylor Swift tickets go on sale) — can be 10,000x normal traffic |
| **Waitlist** | A queue of users who want a ticket if a held seat is released (timeout or payment failure) |
| **Concurrency** | Many users doing the same thing at the same time — the core challenge of ticket systems |
| **Race Condition** | When two users try to book the same seat simultaneously and both think they succeeded |
| **Inventory** | The pool of available seats/tickets for an event |
| **Reservation Window** | The time limit a user has to complete payment before their seat hold expires (typically 10-15 minutes) |
| **Fan-out** | Sending the same event/notification to many subscribers simultaneously |
| **CDC (Change Data Capture)** | Streaming database changes to other systems in real-time (e.g., availability updates to cache) |
| **Two-Phase Commit (2PC)** | A protocol to ensure all parts of a distributed transaction either all succeed or all fail |
| **Idempotency Key** | A unique ID you attach to a payment request so it's safe to retry without double-charging |
| **Chargebacks** | When a cardholder disputes a payment — common in fraud scenarios |
| **PCI DSS** | Payment Card Industry Data Security Standard — rules for handling credit card data |

---
