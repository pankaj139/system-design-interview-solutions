# Google Photos - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a photo storage and sharing service (Google Photos/Flickr) supporting 500M users.

---

## 🎯 Core Problem Statement

- **What:** Design a cloud photo storage service with sharing, search, and automatic backup
- **Key Challenge:** Store and serve petabytes of media efficiently with fast search and sharing
- **Scale:** 500M users, 100M photos/day, 10PB storage, <200ms thumbnail load, 99.9% availability

## 📊 Numbers That Matter

| Metric | Value |
|--------|-------|
| Daily Active Users | 500M |
| Photos Uploaded/Day | 100M |
| Avg Photo Size | 3MB (original) |
| Daily Upload | 300TB |
| Storage (5 years) | 500PB |
| Thumbnail Size | 50KB |
| Read:Write Ratio | 10:1 |

## 🏗️ High-Level Architecture

```text
[Mobile App] → [CDN] → [API Gateway] → [Photo Service]
                                            ↓
                              [Upload: S3] [Metadata: PostgreSQL]
                                            ↓
                              [Processing Pipeline: Resize, ML]
                                            ↓
                              [Search: Elasticsearch] [Recommendations]
```

## 🚀 Critical Talking Points

### Point 1: Tiered Storage

- **What:** Hot (S3 Standard) → Warm (S3 IA) → Cold (Glacier) based on access
- **Why:** Optimize costs - 80% photos not accessed after 30 days
- **Detail:** Original in S3, thumbnails in SSD/CDN, move to Glacier after 6 months
- **Alternative:** Single tier (expensive) or only cold storage (slow access)

### Point 2: ML-Based Search

- **What:** Computer vision to tag photos automatically
- **Why:** Enable search by content without manual tagging
- **Detail:** Face recognition, object detection, scene classification on upload
- **Alternative:** Manual tagging only (poor UX) or text search only (limited)

### Point 3: Deduplication

- **What:** Store identical photos once using content hash (SHA-256)
- **Why:** Save storage - 20-30% duplicate reduction
- **Detail:** Hash on upload, reference count for sharing
- **Alternative:** No dedup (waste storage) or byte-level dedup (complex)

### Point 4: Progressive Loading

- **What:** Serve thumbnail → compressed → full quality progressively
- **Why:** Fast perceived load time on mobile networks
- **Detail:** Generate 5 sizes on upload (thumb, small, medium, large, original)
- **Alternative:** Only original (slow) or only compressed (quality loss)

## ⚖️ Key Trade-Offs

| Decision | Choice | Why |
|----------|--------|-----|
| Storage | S3 + Tiering | Cost optimization for rarely accessed photos |
| Thumbnails | Pre-generated | Fast load vs storage cost |
| Deduplication | Hash-based | Significant storage savings |
| Search | ML + Elasticsearch | Better UX, worth complexity |

## 💡 Interview Tips

**Emphasize:**
- Tiered storage for cost optimization
- ML-based auto-tagging for search
- CDN for fast global delivery
- Deduplication for storage efficiency

**Be Ready For:**
- "How to handle uploads?" → Direct to S3, async processing pipeline
- "How to search photos?" → ML tagging + Elasticsearch full-text search
- "How to share albums?" → Permission table, shareable links with tokens
- "How to optimize costs?" → Tiered storage, deduplication, compression

