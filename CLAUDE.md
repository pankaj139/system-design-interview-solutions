# CLAUDE.md — AI Assistant Guide for System Design Interview Solutions

This file provides guidance for AI assistants (Claude, Copilot, etc.) working in this repository. Read this before creating or editing any content.

---

## Repository Overview

This is a **documentation-only** educational repository containing comprehensive system design solutions for FAANG-level technical interviews. There is **no production code** — only Markdown files with architecture diagrams, schemas, API designs, and educational explanations.

**Primary Goal:** Help software engineers master system design interviews through multi-level (Beginner → Intermediate → Advanced) content that teaches architectural thinking.

**Current Status:** 16/52 designs complete (30.8%), with 9 in-progress and 27 planned.

---

## Repository Structure

```
/
├── *_system_design.md              (27 comprehensive design documents, ~12,000+ lines each)
├── interview-quick-references/     (25 condensed quick-reference guides, ~400-600 lines each)
│   └── *-quick-ref.md
├── EDUCATIONAL_TEMPLATE_GUIDE.md   (Authoritative style guide — 2,400+ lines, READ THIS FIRST)
├── README.md                       (Repository overview and completed designs index)
├── TODO.md                         (Progress tracker — 16/52 complete as of Feb 2026)
├── .github/
│   └── copilot-instructions.md     (AI agent development guidelines)
└── .gitignore
```

**File naming conventions:**
- `snake_case_system_design.md` — main design documents
- `kebab-case-quick-ref.md` — quick reference guides (in `interview-quick-references/`)

---

## Development Workflows

### Git Setup

- **Primary branch:** `main`
- **Active development branch:** `claude/add-claude-documentation-GpJRn`
- All work should be committed to the development branch and pushed with:
  ```bash
  git push -u origin claude/add-claude-documentation-GpJRn
  ```

### Creating a New System Design Document

1. **Consult `EDUCATIONAL_TEMPLATE_GUIDE.md`** — it is the authoritative reference for all structure and formatting decisions
2. **Assign difficulty and tags** at the top of the document (see conventions below)
3. **Follow the 12-15 section template** (described in the next section)
4. **Update `TODO.md`** — mark the design as 🚧 in-progress immediately; mark ✅ complete when done
5. **Update `README.md`** — add an entry in the completed designs list when finished
6. **Create a quick-reference** in `interview-quick-references/` following the existing pattern
7. **Target 12,000+ lines** for a full design; quick references are 400-600 lines

### Editing Existing Documents

- Preserve the existing section structure; don't reorder sections
- Maintain all three difficulty levels (🟢/🟡/🔴) in every section you touch
- Keep numbers specific and realistic (e.g., "99.99% uptime = 52 minutes downtime/year", not "high availability")
- After editing a design, verify it still matches the 12-15 section template

---

## Document Structure (12-15 Section Template)

Every `*_system_design.md` file must follow this structure:

```
Section 0:  Header (difficulty stars, tags, metadata)
Section 1:  Welcome / Learning Path introduction
Section 2:  Table of Contents
Section 3:  Understanding Requirements (functional + non-functional)
Section 4:  Capacity Planning (back-of-envelope calculations)
Section 5:  High-Level Architecture (Mermaid diagram)
Section 6-10: System-Specific Deep Dives (5 topics unique to each system)
Section 11: Scalability & Performance
Section 12: Security Considerations
Section 13: Monitoring & Observability
Section 14: Trade-Offs & Design Decisions
Section 15: Interview Preparation (frameworks, troubleshooting, evolution)
Section 16: Putting It All Together (synthesis + resources)
```

Each section uses this internal pattern:

```markdown
### Section N: [Title]

#### What You'll Learn
#### Why This Matters

##### 🟢 For Beginners: [Topic]
[Everyday analogy + fundamentals]

##### 🟡 For Intermediate: [Topic]
[Interview framework + trade-off analysis]

##### 🔴 For Advanced: [Topic]
[Production concerns + edge cases]

#### Think About It
#### Key Takeaways
#### Practice Exercise
```

---

## Content Conventions

### Difficulty Levels

Every document header must include a difficulty rating:

| Rating | Level | Examples |
|--------|-------|---------|
| ⭐⭐ | Medium | URL Shortener, Text Storage |
| ⭐⭐⭐ | Medium-Hard | CDN, Distributed Cache, Autocomplete |
| ⭐⭐⭐⭐ | Hard | File Storage, KV Store, Web Crawler, Video Streaming |
| ⭐⭐⭐⭐⭐ | Expert | Google Photos, E-commerce, Payment Gateway |

### Tags (6-10 per document)

Select from these categories:

- **System Type:** `Storage`, `Messaging`, `Media`, `Social Network`, `E-commerce`, `Search`, `Real-time`
- **Technologies:** `Distributed Systems`, `Caching`, `ML/AI`, `WebSocket`, `Encryption`, `CDN`
- **Algorithms:** `Consistent Hashing`, `Fan-out`, `Geospatial`, `Ranking`, `Vector Clocks`
- **Scale:** `High Throughput`, `Low Latency`, `Global Scale`, `High Availability`
- **Features:** `Privacy Engineering`, `Analytics`, `Recommendations`, `Fraud Detection`

### Multi-Level Writing Guidelines

- **🟢 Beginner:** Use everyday analogies (restaurants, parking garages, libraries). Explain all jargon. Assume no prior distributed systems knowledge.
- **🟡 Intermediate:** Provide interview frameworks with exact questions to ask. Include decision trees and trade-off comparisons.
- **🔴 Advanced:** Cover production concerns — compliance (GDPR, PCI DSS), cost optimization, failure modes, enterprise features, company architecture evolution.

### Code and Diagrams

**Include (HLD-appropriate):**
- JSON/YAML API request/response examples
- Database schemas with field types and indexes
- Configuration snippets
- Pseudocode for key algorithms
- Mermaid architecture diagrams
- `├─ └─` tree diagrams for hierarchies

**Do NOT include:**
- Full class/function implementations
- ML model training code
- Business logic implementations
- Working executable code

**Always specify language on code blocks** (MD040 rule):
````
```python   ✅
```json     ✅
```         ❌ (no language = linting error)
````

### Callout Patterns

Use these consistently throughout documents:
- `💡 Pro Tip:` — Interview-specific hints
- `⚠️ Watch Out:` — Common mistakes to avoid
- `📊 Example:` — Real-world references
- `🎯 Key Point:` — Critical concepts to memorize

### Numbers and Metrics

Always use specific numbers, never vague qualifiers:

| Use this | Not this |
|----------|----------|
| 99.99% uptime (52 min/year downtime) | "high availability" |
| <100ms p99 latency | "low latency" |
| 1M requests/day, ~12 QPS | "high traffic" |
| 500 PB storage across 100+ edge nodes | "large scale" |

### Scale Benchmarks by System Category

| Category | Typical Scale |
|----------|---------------|
| Storage Systems | 100+ PB data, 10M+ QPS |
| Social Media | 500M+ DAU, 10B+ daily operations |
| Streaming | 100M+ concurrent users, 300+ Tbps |
| E-commerce | 1M+ orders/day, $1B+ transaction volume |
| Real-time Systems | <100ms latency, 99.9%+ delivery guarantee |

---

## Technology Stack Conventions

When multiple options exist, prefer the **established stack** used consistently across this repository. Introducing one-off tech choices creates inconsistency; always note trade-offs if deviating.

| Role | Preferred Choice | Alternatives Mentioned |
|------|-----------------|----------------------|
| Load Balancing | Nginx, AWS ELB | HAProxy |
| OLTP Database | PostgreSQL | MySQL |
| Time-Series / Wide Column | Apache Cassandra | DynamoDB |
| Caching | Redis | Memcached |
| Search | Elasticsearch | Solr |
| Message Queue (throughput) | Apache Kafka | AWS SQS |
| Message Queue (reliability) | RabbitMQ | — |
| Object Storage | AWS S3 | GCS, MinIO |
| Analytics DB | ClickHouse | BigQuery |
| Monitoring | Prometheus + Grafana | — |
| Log Aggregation | ELK Stack (Elasticsearch, Logstash, Kibana) | — |
| CDN | CloudFront, Cloudflare, Akamai | — |

---

## Quality Checklist

Before committing a new or updated design, verify:

- [ ] Document header has difficulty stars (⭐⭐–⭐⭐⭐⭐⭐) and 6-10 tags
- [ ] All 12-15 sections are present and in order
- [ ] Every section has 🟢, 🟡, and 🔴 subsections
- [ ] Back-of-envelope calculations include QPS, storage, and bandwidth
- [ ] At least one Mermaid architecture diagram is present
- [ ] API design covers 10+ endpoints
- [ ] All code blocks specify a language
- [ ] Uses specific numbers everywhere (no vague qualifiers)
- [ ] Includes real company examples (Netflix, Google, Amazon, Uber, etc.)
- [ ] Security and monitoring sections are complete
- [ ] `TODO.md` updated to reflect current status
- [ ] `README.md` updated if the design is newly completed

---

## Common Pitfalls

- **Skipping difficulty levels** — every section needs all three levels. Don't assume the reader has background knowledge.
- **Vague metrics** — always use exact numbers.
- **Missing company examples** — real-world validation makes content authoritative.
- **Incomplete APIs** — design full endpoint sets (10+ endpoints), not just the core 2-3.
- **Ignoring edge cases** — failure modes and scaling bottlenecks belong in every design.
- **Adding executable code** — this is HLD, not LLD. Pseudocode and schemas only.
- **Forgetting to update TODO.md/README.md** — always keep progress tracking accurate.

---

## Key Reference Files

| File | Purpose |
|------|---------|
| `EDUCATIONAL_TEMPLATE_GUIDE.md` | **Primary reference** — full template with examples (read before creating content) |
| `TODO.md` | Progress tracker — update when starting or completing any design |
| `README.md` | Public-facing index — update when a design reaches ✅ Complete |
| `.github/copilot-instructions.md` | Additional AI-agent guidelines (complements this file) |
| `url_shortener_system_design.md` | Best example of a fully compliant design to reference |

---

## No Build or Test Infrastructure

This is a pure documentation repository. There are no:
- Build scripts, Makefiles, or CI/CD pipelines
- Test suites or linters (beyond Markdown style conventions)
- Docker/Kubernetes configurations
- Package managers (npm, pip, etc.)

Content quality is enforced through the template structure and peer review, not automation.
