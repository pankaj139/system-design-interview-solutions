# File Storage System - Interview Quick Reference

**File Purpose:** Concise interview-ready quick reference for designing a distributed file storage system (Dropbox/Google Drive) supporting 100M users.

---

## 🎯 Core Problem Statement

- **What:** Design a cloud file storage with sync, sharing, and version control
- **Key Challenge:** Efficiently sync files across devices while handling conflicts
- **Scale:** 100M users, 10B files, 10PB storage, <1s sync latency, 99.99% availability

## 📊 Numbers That Matter

| Metric | Value |
|--------|-------|
| Active Users | 100M |
| Total Files | 10B |
| Avg File Size | 1MB |
| Storage | 10PB |
| Sync Latency | <1s |
| Daily Uploads | 100M files |

## 🏗️ High-Level Architecture

```text
[Desktop/Mobile Client] → [Sync Service] → [Metadata DB]
                              ↓
                      [Block Storage (S3)]
                              ↓
                      [CDN for Downloads]
```

## 🚀 Critical Talking Points

### Point 1: Block-Level Deduplication

- **What:** Split files into chunks (4MB blocks), store unique blocks once
- **Why:** Save bandwidth and storage - upload only changed blocks
- **Detail:** Rolling hash (Rabin fingerprint) to identify boundaries, SHA-256 for block ID
- **Alternative:** File-level (wastes bandwidth on small changes) or byte-level (too granular)

### Point 2: Delta Sync Algorithm

- **What:** Only transfer modified blocks, not entire file
- **Why:** Fast sync on slow networks - 100KB change in 1GB file = 100KB upload
- **Detail:**
  1. Client computes block hashes
  2. Server compares with stored hashes
  3. Upload only new/modified blocks
  4. Server reconstructs file
- **Alternative:** Full file upload (slow), binary diff (complex)

### Point 3: Conflict Resolution

- **What:** Detect and handle concurrent edits from multiple devices
- **Why:** Prevent data loss when file edited offline on 2 devices
- **Detail:**
  - Version vector per file (device_id → version)
  - Conflict if neither version is ancestor of other
  - Create conflict copies for manual merge
- **Alternative:** Last-write-wins (data loss) or lock-based (requires online)

### Point 4: Metadata Service

- **What:** Separate metadata (file structure) from block storage
- **Why:** Fast operations on file tree without touching blocks
- **Detail:**
  - PostgreSQL: file_id, name, path, owner, version, block_list[]
  - Redis cache for hot file metadata
  - Block pointers with reference counting
- **Alternative:** Filesystem-based (doesn't scale) or metadata in blocks (slow)

### Point 5: Client-Side Caching

- **What:** Keep local copy of files and metadata
- **Why:** Instant access to recent files, offline capability
- **Detail:**
  - LRU cache with size limit (50GB)
  - Metadata always cached, blocks on-demand
  - Periodic sync every 30s or on file change
- **Alternative:** Always fetch from server (slow) or sync all files (storage waste)

## ⚖️ Key Trade-Offs

| Decision | Choice | Why |
|----------|--------|-----|
| Deduplication | Block-level (4MB) | Balance granularity and overhead |
| Sync | Delta (blocks) | Bandwidth optimization |
| Conflicts | Version vector + copies | Prevent data loss, user resolves |
| Metadata | Separate service | Fast tree operations |
| Storage | S3 + CDN | Cost-effective, globally distributed |

## 💡 Interview Tips

**Emphasize:**

- Block-level deduplication for bandwidth savings
- Delta sync for efficiency
- Conflict detection with version vectors
- Metadata separation for performance

**Be Ready For:**

- "How to sync 1GB file change?" → Only upload modified blocks
- "What if edited on 2 devices?" → Detect conflict, create copies
- "How to share files?" → Shared folder with permission table
- "How to version files?" → Keep block history, lazy deletion
