# GitHub Copilot Instructions for System Design Interview Solutions

## Repository Overview

This repository contains comprehensive system design solutions for technical interview preparation, targeting FAANG-level companies. Each design follows a structured educational framework with multi-level explanations (Beginner 🟢, Intermediate 🟡, Advanced 🔴) and includes production-ready considerations.

## Key Architecture Patterns

### Educational Content Structure

Every system design document follows this **12-15 section template**:

1. **Requirements & Planning** (Sections 1-2): Requirements gathering, capacity planning with back-of-envelope calculations
2. **Core System Components** (Sections 3-8): System-specific technical deep-dives (e.g., transcoding, caching, URL generation)
3. **Production Concerns** (Sections 9-13): Scalability, security, monitoring, trade-offs analysis
4. **Interview Preparation** (Section 14-15): Interview frameworks, troubleshooting scenarios, architecture evolution

### Content Organization Patterns

- **Main Designs**: `*_system_design.md` - Comprehensive 12,000+ line educational documents
- **Quick References**: `interview-quick-references/*-quick-ref.md` - Concise 400-line interview prep
- **Progress Tracking**: `TODO.md` - Detailed completion tracker with 23/52 designs complete

### Documentation Standards

All designs must include:
- Mermaid diagrams for architecture visualization
- Back-of-envelope calculations with specific numbers (QPS, storage, bandwidth)
- Database schemas with indexing strategies  
- Complete API designs (10+ endpoints per system)
- Real-world company examples (Netflix, Google, Amazon practices)
- Security considerations and monitoring strategies

## Development Workflows

### Creating New System Designs

1. **Follow the Template**: Use `EDUCATIONAL_TEMPLATE_GUIDE.md` (2,400+ lines) as the authoritative structure guide
2. **Update Progress**: Mark design as 🚧 in-progress in `TODO.md`, update completion status when done
3. **Target Metrics**: Aim for 12,000+ lines for comprehensive coverage
4. **Multi-Level Content**: Every section needs Beginner/Intermediate/Advanced explanations

### Key Writing Conventions

- **Analogies for Beginners**: Use everyday analogies (restaurants, parking garages, libraries)
- **Interview Scripts for Intermediate**: Provide frameworks with exact questions to ask
- **Production Concerns for Advanced**: Business impact, compliance (GDPR, PCI DSS), enterprise features
- **Code Usage (HLD-Focused)**:
  - ✅ **INCLUDE**: Data structures (JSON/dicts), API signatures, config examples, DB schemas
  - ❌ **AVOID**: Full class implementations, ML training code, complex algorithms, detailed business logic
  - **Reasoning**: HLD interviews test system architecture, not coding implementation
  - **Prefer**: Text diagrams, formulas, pseudocode over working code
- **Real Numbers**: Use specific metrics (99.99% uptime = 52 minutes downtime/year)

### Quality Standards

- **Markdown Compliance**: All code blocks must specify language (MD040 rule)
- **Consistent Formatting**: Use headings not bold for structure (MD036 rule) 
- **Tree Diagrams**: Use consistent `├─ └─` format for hierarchical information
- **Company Examples**: Reference real implementations with evolution timelines

## System-Specific Patterns

### Scale Benchmarks by Category

- **Storage Systems**: 100+ PB data, 10M+ QPS
- **Social Media**: 500M+ DAU, 10B+ daily operations  
- **Streaming**: 100M+ concurrent users, 300+ Tbps bandwidth
- **E-commerce**: 1M+ orders/day, $1B+ transaction volume
- **Real-time Systems**: <100ms latency, 99.9%+ delivery guarantees

### Technology Stack Patterns

**Consistent Choices Across Designs**:
- **Load Balancing**: Nginx, AWS ELB, HAProxy
- **Databases**: PostgreSQL (transactional), Cassandra (time-series), Redis (caching)
- **Message Queues**: Kafka (high-throughput), RabbitMQ (reliability)
- **Search**: Elasticsearch with CDC sync patterns
- **Storage**: S3 with multi-tier strategies (hot/warm/cold)
- **Monitoring**: Prometheus + Grafana + ELK stack

### Deep-Dive Topics

Advanced sections (400-800 lines each) cover:
- **ML Pipelines**: Feature engineering, training, serving, A/B testing with TensorFlow/PyTorch
- **Edge Computing**: CDN optimization, serverless at edge, geographic distribution
- **Distributed Systems**: Consistent hashing, consensus algorithms, conflict resolution

## File Navigation

### High-Priority Designs (Interview Focus)
- `url_shortener_system_design.md` - Foundation pattern example
- `distributed_cache_system_design.md` - Core infrastructure
- `social_media_platform_system_design.md` - Complex user-facing system
- `video_streaming_system_design.md` - High-scale media processing

### Interview Preparation
- `interview-quick-references/` - Day-of-interview condensed guides
- Quick references follow consistent format: Problem statement, key numbers, architecture, decisions

### Progress Management
- `TODO.md` - Comprehensive tracker with completion status, priorities, time estimates
- Organized by categories: Storage, Messaging, Search, E-commerce, etc.

## Best Practices for AI Agents

### When Creating New Content

1. **Reference the Template**: Always consult `EDUCATIONAL_TEMPLATE_GUIDE.md` for structure
2. **Calculate Scale**: Include specific back-of-envelope calculations for storage, QPS, bandwidth
3. **Multi-Perspective**: Address all three levels (Beginner/Intermediate/Advanced) per section
4. **Real Examples**: Include how major tech companies (Netflix, Google, Meta) solve similar problems
5. **Interview Focus**: Structure content to help users succeed in technical interviews

### When Reviewing/Editing

1. **Verify Completeness**: Check against 12-15 section template structure
2. **Validate Numbers**: Ensure calculations are realistic and well-explained
3. **Check Consistency**: Verify technology choices align with established patterns
4. **Test Explanations**: Ensure analogies work for beginners, frameworks help intermediates

### Common Pitfalls to Avoid

- **Skipping Levels**: Don't assume knowledge - explain fundamentals for beginners
- **Vague Metrics**: Use specific numbers (99.99% not "high availability")  
- **Missing Company Examples**: Always include real-world validation
- **Incomplete APIs**: Design complete endpoint sets, not just core operations
- **Ignoring Edge Cases**: Address failure modes and scaling bottlenecks

This repository represents production-level system design knowledge distilled into educational format, optimized for technical interview success while maintaining architectural depth for learning.