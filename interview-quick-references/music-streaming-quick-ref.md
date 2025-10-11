# Music Streaming Service (Spotify) - Interview Quick Reference

## 🎯 Core Problem Statement

- **What:** Design a global music streaming platform with personalized recommendations and cross-device sync
- **Key Challenge:** Deliver high-quality audio globally with <2 second startup time and adaptive bitrate streaming
- **Scale:** 500M users, 100M DAU, 100M songs catalog, 50M streams/day

## 📊 Numbers That Matter

| Metric | Value | Calculation |
|--------|-------|-------------|
| Daily Active Users | 100M | Given |
| Total Users | 500M | Given |
| Songs Catalog | 100M | Given |
| Daily Streams | 50M | Given |
| Peak Streams/sec | 2,083 | 7.5M streams/hour ÷ 3600 |
| Audio Requests/sec | 22,913 | 2,083 × 11 requests/stream |
| Storage (5 years) | 1.1PB | 541TB × 2 (growth) |
| CDN Bandwidth | 30Gbps | 2,083 streams × 1.8MB |

## 🏗️ High-Level Architecture

```text
[Mobile App] -> [CDN] -> [Load Balancer] -> [API Gateway] -> [Streaming Service] -> [S3]
     |              |           |                |                |                |
     v              v           v                v                v                v
[Desktop App] -> [Edge] -> [Nginx] -> [Kong] -> [Playlist Service] -> [PostgreSQL]
     |              |           |                |                |                |
     v              v           v                v                v                v
[Web Player] -> [CloudFront] -> [Auth Service] -> [Search Service] -> [Elasticsearch]
```

**Components:**
- CDN: CloudFront, 100+ edge locations, audio delivery
- API Gateway: Kong, OAuth2 authentication, rate limiting
- Streaming Service: Go, adaptive bitrate, chunked streaming
- Playlist Service: Node.js, playlist management, cross-device sync
- Search Service: Elasticsearch, full-text search, faceted search
- Recommendation Service: Python/ML, collaborative filtering, content-based
- PostgreSQL: User data, playlists, ACID compliance
- S3: Audio files, multiple formats, global distribution
- Redis: Playback state, recommendations cache, real-time sync

## 💾 Data Model (Essentials)

### Core Tables
- **users**: user_id (PK), email, username, subscription_type, country
- **tracks**: track_id (PK), name, artist_id (FK), album_id (FK), duration_ms
- **artists**: artist_id (PK), name, genres, popularity, followers_count
- **playlists**: playlist_id (PK), name, owner_id (FK), is_public, tracks_count

### Key Indexes
- tracks: (artist_id, popularity) - for artist pages and trending
- playlists: (owner_id, is_public) - for user playlists and discovery
- listening_history: (user_id, played_at) - for recommendations

### Partitioning Strategy
- PostgreSQL: Shard by user_id for user data, by artist_id for music data
- S3: Partition by artist_id/album_id for audio files
- Redis: Cluster by geographic regions for playback state

## 🔌 API Design (Key Endpoints)

| Method | Path | Purpose |
|--------|------|---------|
| GET | /tracks/{track_id} | Get track metadata and audio features |
| GET | /tracks/{track_id}/stream | Get audio stream URL with adaptive bitrate |
| PUT | /me/player/play | Update playback state across devices |
| POST | /users/{user_id}/playlists | Create new playlist |
| POST | /playlists/{playlist_id}/tracks | Add tracks to playlist |
| GET | /search?q={query}&type=track | Search tracks with faceted results |
| GET | /recommendations | Get personalized recommendations |

## 🚀 Critical Talking Points

### Audio Streaming Pipeline
- **What:** Multi-format transcoding (MP3, AAC, OGG) with adaptive bitrate streaming
- **Why:** Different devices need different formats, network conditions vary
- **Detail:** 6 bitrates (96-320kbps), 10-second chunks, CDN distribution with 99% cache hit
- **Alternative:** Single format (simpler but less efficient, poor user experience)

### Recommendation Engine
- **What:** Hybrid ML system combining collaborative filtering and content-based algorithms
- **Why:** Personalized discovery drives user engagement and retention
- **Detail:** User-user/item-item collaborative filtering + audio feature analysis + matrix factorization
- **Alternative:** Simple popularity-based (easier but less personalized)

### Cross-Device Sync
- **What:** Real-time playback state synchronization using WebSocket and Redis
- **Why:** Users expect seamless experience across mobile, desktop, web
- **Detail:** Operational transformation for playlist conflicts, last-write-wins for playback state
- **Alternative:** Polling-based sync (simpler but higher latency, battery drain)

### CDN Architecture
- **What:** CloudFront with 100+ edge locations for global audio delivery
- **Why:** <2 second startup time requires geographic distribution and caching
- **Detail:** Cache warming for popular tracks, predictive caching, geographic routing
- **Alternative:** Self-hosted CDN (more control but higher operational cost)

### Search and Discovery
- **What:** Elasticsearch-based search with audio feature analysis and faceted filtering
- **Why:** Users need to find music by multiple criteria (artist, genre, mood, etc.)
- **Detail:** Full-text search + audio feature vectors + popularity boosting
- **Alternative:** Database-based search (simpler but limited functionality)

### Playlist Management
- **What:** Collaborative playlists with real-time updates and conflict resolution
- **Why:** Social features increase engagement and user retention
- **Detail:** Operational transformation for concurrent edits, WebSocket for real-time updates
- **Alternative:** Simple CRUD operations (easier but no real-time collaboration)

## ⚖️ Key Trade-Offs

| Decision | Choice | Alternative | Why |
|----------|--------|-------------|-----|
| Audio Formats | Multiple (MP3/AAC/OGG) | Single format | Device compatibility vs storage cost |
| Database | Multi-DB (PostgreSQL/Cassandra/Redis/ES) | Single database | Optimized for each use case vs simplicity |
| Streaming | Adaptive bitrate | Fixed bitrate | Better UX vs implementation complexity |
| Recommendations | ML-based | Rule-based | Better personalization vs simpler logic |
| Sync | Real-time WebSocket | Polling | Better UX vs implementation complexity |
| CDN | CloudFront | Self-hosted | Global scale vs control |

## 🔥 Bottlenecks & Solutions

### CDN Cache Misses
- **Problem:** Popular tracks causing cache misses during peak hours
- **Solution:** Cache warming, predictive caching, more edge locations

### Recommendation Latency
- **Problem:** ML model inference too slow for real-time recommendations
- **Solution:** Pre-compute recommendations, model serving infrastructure, caching

### Database Write Contention
- **Problem:** High volume streaming events causing database locks
- **Solution:** Cassandra for time-series data, async writes, batching

### Cross-Device Sync Conflicts
- **Problem:** Simultaneous playback on multiple devices causing conflicts
- **Solution:** Device priority, conflict resolution algorithms, graceful degradation

## 💡 Interview Tips

- **Start Here:** "Let me clarify the requirements - we need to support 100M DAU with <2 second startup time"
- **Emphasize:** CDN architecture, adaptive streaming, ML recommendations, cross-device sync
- **Be Ready For:** "How do you handle offline playback?", "What about copyright protection?"
- **Don't Forget:** Audio transcoding pipeline, recommendation algorithms, real-time sync protocols

---

**Scale:** 500M users, 100M DAU, 100M songs, 50M streams/day
**Key Tech:** CloudFront CDN, Elasticsearch, Redis, PostgreSQL, Cassandra, ML Pipeline
**Complexity:** ⭐⭐⭐⭐ Very Hard
**Interview Time:** 45-60 minutes
