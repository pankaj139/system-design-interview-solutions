# Video Streaming Service (Netflix/YouTube) - Principal Engineer Level - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a global video streaming platform with adaptive bitrate streaming and real-time analytics
- **Key Challenge:** <2 second startup time for 100M concurrent viewers with 99% CDN cache hit ratio
- **Scale:** 100M concurrent viewers, 1M hours content (100 PB), 50M uploads/day, 10B views/day

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Total Users | 1B | Given |
| Concurrent Viewers | 100M | Peak traffic |
| Video Content Storage | 100 PB | 60M hours × 2.25 GB × 5 qualities |
| Daily Uploads | 50M videos | Given |
| Daily Views | 10B | 500M users × 20 views |
| Peak Bandwidth | 300 Tbps | 100M × 3 Mbps average bitrate |
| Daily Bandwidth | 5 PB | 500M × 45 min × 3 Mbps |
| Transcoding Servers | 1000 | 50M uploads ÷ 48 videos/hour |
| API Servers | 3500 | 16.7M QPS ÷ 10K + 2x redundancy |
| CDN Cache Size | 10 PB | 200 edges × 50 TB |

## 🏗️ High-Level Architecture

```text
[Mobile/Web/TV Clients] -> [Global CDN (200+ edges, 10 PB cache)] -> [Geographic LB]
         |                             |                                  |
         v                             v                                  v
[Origin Shield] -> [Regional LB] -> [API Gateway] -> [Playback Service]
         |              |                |                  |
         v              v                v                  v
[Upload Service] -> [Transcoding Pipeline (GPU)] -> [Video Storage (S3, 100 PB)]
         |              |                                    |
         v              v                                    v
[Kafka Stream] -> [Flink Processing] -> [Analytics (ClickHouse)] -> [Recommendation Engine]
```

**Components:**
- Global CDN: CloudFront + Akamai, 200+ edge locations, 99% cache hit ratio
- Transcoding Pipeline: 1000 GPU servers (NVIDIA T4), FFmpeg, H.264/H.265/VP9
- Video Storage: AWS S3/GCS, 100 PB, multi-region replication
- Analytics: Kafka + Flink + ClickHouse, 1B events/minute, <5 second latency
- Recommendation: TensorFlow, collaborative filtering + deep learning, <10ms inference

## 💾 Data Model (Essentials)

### Core Tables
- **videos**: video_id (PK), channel_id (FK), title, duration_seconds, status, view_count, manifest_url
- **channels**: channel_id (PK), user_id (FK), channel_name, subscriber_count, is_verified
- **video_processing**: processing_id (PK), video_id (FK), transcoding_status, thumbnail_status, original_file_url
- **video_segments**: segment_id (PK), video_id (FK), quality, segment_number, segment_url, duration_seconds
- **watch_history**: history_id (PK), user_id (FK), video_id (FK), watch_position_seconds, watched_at

### Analytics Tables (ClickHouse)
- **video_views**: event_id, video_id, user_id, watch_duration_seconds, quality_watched, buffering_events
- **video_analytics_daily**: video_id, date, view_count, total_watch_time_seconds, avg_startup_time_ms

### Advanced Indexes
- videos: (channel_id, published_at), (is_public, view_count), Full-text on (title, description)
- watch_history: (user_id, watched_at) for personalization
- video_segments: (video_id, quality, segment_number) for playback

### Caching Strategy
- L1 (Application): 10K hot videos, <1ms
- L2 (Redis): 1M videos metadata, <5ms
- L3 (CDN): 10 PB video segments, <50ms
- Cache hit ratio: 99.2% overall

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /videos/{id}/manifest | Get HLS/DASH manifest with all quality levels |
| POST | /videos/{id}/events | Report playback events (view, buffering, quality change) |
| POST | /videos/upload/initiate | Initiate multipart upload with chunk size |
| GET | /videos/{id}/analytics | Get comprehensive video analytics |
| GET | /search | Search videos with filters and pagination |

## 🚀 Critical Talking Points (Principal Engineer Level)

### Adaptive Bitrate Streaming with ML
- **What:** Multi-bitrate transcoding (240p-4K) with ML-powered quality selection
- **Why:** Minimize buffering while maximizing quality, 95% rebuffer reduction
- **Detail:** HLS/DASH protocol, 2-6 second segments, RL model (Deep Q-Network) for quality selection
- **Tradeoff:** Segmentation overhead vs adaptive quality - worth 20% bandwidth savings
- **Alternative:** Fixed bitrate (simple but 3x buffering rate, poor UX)

### Distributed Transcoding with GPU Acceleration
- **What:** 1000 GPU servers (NVIDIA T4) with FFmpeg, parallel multi-quality encoding
- **Why:** Process 50M uploads/day, 10x faster than CPU, 50% cost savings
- **Detail:** Kafka job queue, auto-scaling 50-200%, priority-based processing
- **Tradeoff:** Infrastructure complexity vs cost - GPU saves $2M/day vs CPU
- **Alternative:** AWS MediaConvert (simple but 3.5x cost = $4M/day extra)

### Global CDN with 99% Cache Hit Ratio
- **What:** Multi-tier CDN (CloudFront + Akamai, 200+ edges, 10 PB cache)
- **Why:** <2 second startup, 95% cost savings, 99.99% availability
- **Detail:** Origin shield, predictive cache warming with ML, ISP-level caching
- **Tradeoff:** Multi-CDN complexity vs reliability - 99.9% → 99.99% availability
- **Alternative:** Single CDN (simple but single point of failure, 99.5% availability)

### Real-Time Analytics Pipeline
- **What:** Kafka + Flink + ClickHouse, 1B events/minute, <5 second latency
- **Why:** Real-time view counts, trending detection, creator analytics
- **Detail:** Lambda architecture (speed + batch layer), HyperLogLog for counting
- **Tradeoff:** Complexity vs real-time - 98% accuracy in <5s vs 100% in 5 minutes
- **Alternative:** Batch processing (simple but 5-10 minute delay, poor creator experience)

### Progressive Transcoding Strategy
- **What:** Phase 1: 720p (2 min), Phase 2: 1080p (background), Phase 3: 4K (background)
- **Why:** Video playable in 2 minutes vs 10 minutes, better creator experience
- **Detail:** Priority-based processing, SLA tiers (verified creators <10 min)
- **Tradeoff:** Delayed full quality vs perceived instant availability
- **Alternative:** Process all qualities before publish (simple but 10-minute wait)

### Multi-Origin Geographic Distribution
- **What:** Primary origin (US), secondary (EU, Asia), cross-region replication
- **Why:** 40% bandwidth reduction per origin, better latency for global users
- **Detail:** 5-minute replication lag, CDN routes to nearest origin
- **Tradeoff:** Replication cost vs performance - worth 40% bandwidth savings
- **Alternative:** Single origin (simple but 2x latency for far users, higher bandwidth)

## ⚖️ Key Trade-Offs (Principal Engineer Analysis)

| Decision | Choice | Alternative | Principal Engineer Analysis |
|----------|--------|-------------|----------------------------|
| Video Codec | H.264 primary + H.265 for 4K | H.265 only | Universal compatibility + 40% 4K bandwidth savings |
| Transcoding | GPU (NVIDIA T4) | AWS MediaConvert | $2M/day savings justifies infrastructure complexity |
| CDN Strategy | Multi-CDN (CloudFront + Akamai) | Single CDN | 99.99% vs 99.5% availability, cost optimization |
| Analytics | Real-time (Kafka + Flink) | Batch (Spark) | Creator experience demands <5 second latency |
| Quality Selection | ML-powered adaptive | Fixed ladder | 95% rebuffer reduction, 20% bandwidth savings |
| Storage | Multi-region S3 | Single region | Geographic redundancy for 99.999% durability |

## 🔥 Bottlenecks & Solutions (Principal Engineer Level)

### Transcoding Queue Backup During Viral Events
- **Root Cause:** Viral videos cause 3x spike in uploads, overwhelming 1000 GPU servers
- **Impact:** Processing delay 10 min → 6 hours, creator complaints, revenue loss
- **Solutions:**
  - Predictive auto-scaling: ML predicts viral events, pre-scale +50% capacity
  - Hybrid cloud bursting: Auto-burst to AWS MediaConvert when queue >1 hour
  - Progressive transcoding: 720p in 2 min (instant playability), full quality background
  - Priority-based SLA: Verified creators <10 min, free users <2 hours
- **Monitoring:** Queue depth by tier, GPU utilization, processing time percentiles

### CDN Cache Miss Storm During Popular Launches
- **Root Cause:** 1M+ simultaneous requests for new popular video, 80% cache miss rate
- **Impact:** Origin bandwidth 50x spike (150 Tbps), 8-second startup time
- **Solutions:**
  - Predictive cache warming: ML predicts popularity, pre-warm 200+ edges
  - Origin shield: Collapse 1000s of misses into single origin request (95% reduction)
  - Gradual rollout: Notify subscribers in 10% waves every 5 minutes
  - Multi-origin: Geographic distribution reduces per-origin bandwidth 40%
- **Monitoring:** Origin bandwidth, cache hit ratio, startup time during launches

### Database Hotspots for Viral Video Metadata
- **Root Cause:** Viral video receives 1M+ QPS to single video_id row
- **Impact:** 500ms query latency vs 10ms normal, cascading timeouts
- **Solutions:**
  - Multi-tier caching: L1 (app, <1ms), L2 (Redis, <5ms), 99.6% hit rate
  - Hot key detection: Auto-promote to L1 cache when >1000 QPS
  - Read replica auto-scaling: Dedicated replica for hot videos
  - Metadata denormalization: Store in Redis with write-behind
- **Monitoring:** Per-video QPS, database latency, cache hit ratio by tier

### Real-Time View Count Aggregation at Scale
- **Root Cause:** 1B view events/minute overwhelming aggregation pipeline
- **Impact:** View counts delayed 5-10 minutes, inaccurate trending
- **Solutions:**
  - Lambda architecture: Real-time (Flink, <5s, 98% accurate) + Batch (Spark, 5min, 100%)
  - HyperLogLog: 99% accuracy with 0.1% memory (12 GB vs 1.2 TB)
  - Hierarchical aggregation: Edge → Region → Global (99% event reduction)
  - Adaptive caching: Popular videos update every 1s, old videos every 60s
- **Monitoring:** Aggregation latency, accuracy vs batch, memory usage, write throughput

## 💡 Interview Tips (Principal Engineer Level)

- **Start Here:** "Let me clarify - we need to handle 100M concurrent viewers with <2 second startup time globally"
- **Emphasize:** Adaptive bitrate streaming, GPU transcoding, CDN architecture, real-time analytics, ML-powered optimization
- **Be Ready For:** "How do you handle live streaming?", "What about DRM and content protection?", "How do you detect copyrighted content?"
- **Don't Forget:** Cost optimization (GPU vs CPU, CDN savings), monitoring (startup time, buffering rate), disaster recovery

---

**Scale:** 100M concurrent viewers, 100 PB storage, 50M uploads/day, 10B views/day, 300 Tbps peak bandwidth
**Key Tech:** HLS/DASH, FFmpeg + NVIDIA T4, Multi-CDN, Kafka + Flink + ClickHouse, TensorFlow
**Complexity:** ⭐⭐⭐⭐⭐ Principal Engineer Level
**Interview Time:** 60-90 minutes
