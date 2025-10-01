# GOOGLE PHOTOS SYSTEM DESIGN

## Cloud-Based Photo Storage and Management Platform

---

## 1. REQUIREMENTS & CLARIFICATION

### User Stories

**As a user, I want to:**

- Upload photos and videos from multiple devices so that I can access them anywhere
- Automatically backup my photos so that I never lose precious memories
- Search photos by content, location, or date so that I can quickly find specific images
- Create and share albums with friends and family so that we can collaborate on memories
- View photos in high quality with fast loading so that I have a great viewing experience
- Organize photos with automatic grouping and smart suggestions so that my library stays organized

### Functional Requirements (MVP)

**Core Features:**

1. **Upload & Storage**
   - Upload photos and videos (max 100MB per photo, 10GB per video)
   - Support multiple formats (JPEG, PNG, RAW, MP4, MOV)
   - Auto-backup from mobile devices

2. **Viewing & Organization**
   - View photos in timeline view
   - Create albums and collections
   - Basic metadata (date, location, device)

3. **Search**
   - Search by date range
   - Search by location
   - Search by metadata tags

4. **Sharing**
   - Share individual photos via link
   - Share albums with specific users
   - Public/private sharing controls

5. **Processing**
   - Generate multiple resolution thumbnails
   - Extract and store metadata (EXIF data)
   - Basic image optimization
   - Face detection and recognition
   - Automatic face grouping and clustering

6. **People & Face Management**
   - Detect faces in photos automatically
   - Group similar faces together
   - Name people and tag photos
   - Search photos by person
   - Face-based photo suggestions

### Non-Functional Requirements

**Performance:**

- Upload latency: < 5 seconds for photos, < 30 seconds for videos
- Image load time: < 200ms for thumbnails, < 1s for full resolution
- Search results: < 500ms

**Availability:**

- 99.9% uptime (about 8.76 hours downtime per year)
- Multi-region deployment for disaster recovery

**Scalability:**

- Support 500M active users
- Handle 100M photo uploads per day
- Support 1B photo views per day

**Security:**

- End-to-end encryption for uploads
- Access control for shared content
- Secure authentication (OAuth 2.0)

**Consistency:**

- Eventual consistency acceptable for non-critical operations
- Strong consistency for ownership and permissions

### Clarifying Questions & Assumptions

**Assumptions:**

1. **Scale:** 500M DAU, 20% upload daily (100M uploads/day)
2. **Usage Pattern:** Read-heavy (1:100 write-to-read ratio)
3. **Geographic Distribution:** Global users, concentrated in US, Europe, Asia
4. **Storage Duration:** Unlimited storage for all users
5. **Content:** Average photo size: 3MB, average video size: 100MB
6. **Devices:** Support web, iOS, Android clients
7. **Out of Scope (Post-MVP):** Advanced AI features (face recognition, object detection), photo editing, live photos, collaborative editing

---

## 2. BACK-OF-THE-ENVELOPE CALCULATIONS

### Traffic Estimates

```text
Daily Active Users (DAU): 500M
Users uploading photos: 20% of DAU = 100M users
Average uploads per user: 5 photos/day
Total photos uploaded per day: 500M photos

Upload QPS:
500M photos / 86,400 seconds = ~5,800 photos/second
Peak upload QPS (3x): ~17,400 photos/second

Video uploads:
10% of uploads are videos = 50M videos/day
Video upload QPS: 50M / 86,400 = ~580 videos/second

View Operations:
80% of DAU view photos: 400M users
Average views per user: 50 photos/day
Total views per day: 20B views/day
View QPS: 20B / 86,400 = ~231,000 views/second
Peak view QPS (3x): ~693,000 views/second
```text

### Storage Estimates

```text
Photo Storage:
Average photo size: 3MB
Daily photo storage: 500M photos × 3MB = 1.5 PB/day
Annual photo storage: 1.5 PB × 365 = 547.5 PB/year

Thumbnail Storage:
3 thumbnail sizes per photo: 150px (10KB), 400px (50KB), 1080px (200KB)
Total thumbnail size per photo: 260KB
Daily thumbnail storage: 500M × 260KB = 130 TB/day
Annual thumbnail storage: 130 TB × 365 = 47.45 PB/year

Video Storage:
Average video size: 100MB
Daily video uploads: 50M videos
Daily video storage: 50M × 100MB = 5 PB/day
Annual video storage: 5 PB × 365 = 1,825 PB/year

Metadata Storage:
Per photo metadata: ~2KB (EXIF, location, user_id, timestamps)
Daily metadata: 500M × 2KB = 1 TB/day
Annual metadata: 365 TB/year

Total Storage (per year):
Photos: 547.5 PB
Thumbnails: 47.45 PB
Videos: 1,825 PB
Metadata: 0.365 PB
Total: ~2,420 PB/year (~2.4 EB/year)

5-Year Storage: ~12 EB
```text

### Bandwidth Estimates

```text
Upload Bandwidth:
Photos: 500M × 3MB / 86,400s = 17.4 GB/s
Videos: 50M × 100MB / 86,400s = 57.9 GB/s
Peak upload bandwidth (3x): 225.9 GB/s

Download Bandwidth:
Thumbnail views: 20B × 50KB / 86,400s = 11.6 TB/s
Full image views (10% of views): 2B × 3MB / 86,400s = 69.4 GB/s
Peak download bandwidth (3x): ~35 TB/s

Total Peak Bandwidth: ~35 TB/s
```text

### Resource Estimates

```text
API Servers:
Peak QPS: ~700K requests/second
Assuming 1,000 QPS per server: 700 servers
With redundancy (2x): 1,400 API servers

Upload Processing Workers:
Peak uploads: 18,000 photos/s + 1,740 videos/s
Processing time: 5s per photo, 30s per video
Required workers: (18,000 × 5 + 1,740 × 30) / processing capacity
Estimated: 2,000 worker instances

Face Detection/Recognition Workers:
Assume 60% of photos contain faces (300M photos/day)
Average faces per photo: 2 faces
Total faces to process: 600M faces/day = 6,944 faces/second
Face detection time: 2s per photo (GPU accelerated)
Face embedding generation: 0.5s per face
Required workers: (18,000 × 0.6 × 2) + (21,600 × 0.5) = ~32,400 operations/s
With GPU instances (100 ops/s each): 324 GPU workers
Face clustering (batch job): Run nightly on new faces

Database Connections:
Metadata operations: ~700K QPS
Connection pool per DB: 1,000 connections
Sharded databases: 100 shards

Face Embeddings Storage:
512-dimension vector per face: 2KB per face
Daily face embeddings: 600M × 2KB = 1.2 TB/day
Annual storage: 438 TB/year
```text

---

## 3. HIGH-LEVEL DESIGN

### System Architecture Diagram

```mermaid
graph TB
    subgraph Client Layer
        Web[Web Client]
        Mobile[Mobile Apps<br/>iOS/Android]
    end
    
    subgraph CDN & Entry
        CDN[CDN<br/>CloudFront/Cloudflare]
        LB[Load Balancer<br/>AWS ALB]
    end
    
    subgraph API Layer
        API[API Gateway<br/>Kong/NGINX]
        Auth[Auth Service<br/>OAuth 2.0]
    end
    
    subgraph Service Layer
        Upload[Upload Service<br/>Node.js]
        View[View Service<br/>Node.js]
        Search[Search Service<br/>Elasticsearch]
        Share[Sharing Service<br/>Node.js]
        Album[Album Service<br/>Node.js]
    end
    
    subgraph Processing Layer
        Queue[Message Queue<br/>Apache Kafka]
        ImgProc[Image Processor<br/>Python/Go]
        Metadata[Metadata Extractor<br/>Python]
        Thumbnail[Thumbnail Generator<br/>ImageMagick]
        FaceDetect[Face Detection<br/>MTCNN/RetinaFace]
        FaceRecog[Face Recognition<br/>FaceNet/ArcFace]
    end
    
    subgraph Data Layer
        Cache[Distributed Cache<br/>Redis Cluster]
        MetaDB[(Metadata DB<br/>Cassandra)]
        UserDB[(User/Album DB<br/>PostgreSQL)]
        SearchDB[(Search Index<br/>Elasticsearch)]
        FaceDB[(Face/Vector DB<br/>Milvus/Pinecone)]
    end
    
    subgraph Storage Layer
        Hot[Hot Storage<br/>S3 Standard]
        Warm[Warm Storage<br/>S3 IA]
        Cold[Cold Storage<br/>S3 Glacier]
    end
    
    Web -->|1. Request| CDN
    Mobile -->|1. Request| CDN
    CDN -->|2. Route| LB
    LB -->|3. Distribute| API
    API -->|4. Authenticate| Auth
    API -->|5. Route| Upload
    API -->|5. Route| View
    API -->|5. Route| Search
    
    Upload -->|6. Upload Photo| Queue
    Queue -->|7. Process| ImgProc
    ImgProc -->|8. Extract| Metadata
    ImgProc -->|9. Generate| Thumbnail
    ImgProc -->|10. Detect Faces| FaceDetect
    FaceDetect -->|11. Generate Embeddings| FaceRecog
    
    ImgProc -->|12. Store Original| Hot
    Thumbnail -->|13. Store Thumbnails| Hot
    Metadata -->|14. Store Metadata| MetaDB
    Metadata -->|15. Index| SearchDB
    FaceRecog -->|16. Store Vectors| FaceDB
    
    View -->|17. Query| Cache
    Cache -->|18. Cache Miss| MetaDB
    View -->|19. Get URL| Hot
    
    Search -->|20. Query| SearchDB
    Share -->|21. Permissions| UserDB
    Album -->|22. Album Data| UserDB
    
    Hot -->|Lifecycle: 30 days| Warm
    Warm -->|Lifecycle: 1 year| Cold
```text

### Data Flow Explanation

**Upload Flow (Steps 1-16):**

1. User uploads photo via web/mobile client
2. CDN routes request to nearest load balancer
3. Load balancer distributes to API gateway
4. API gateway authenticates user with Auth service
5. Request routed to Upload Service
6. Upload Service publishes event to Kafka queue
7. Image Processor picks up message for processing
8. Metadata Extractor extracts EXIF data
9. Thumbnail Generator creates multiple resolution thumbnails
10. Face Detection service detects faces in photo
11. Face Recognition generates embeddings for each detected face
12. Original photo stored in S3 Hot storage
13. Thumbnails stored in S3 with appropriate keys
14. Metadata written to Cassandra for fast queries
15. Photo indexed in Elasticsearch for search
16. Face embeddings stored in Vector DB (Milvus) for similarity matching

**View Flow (Steps 17-19):**
17. View Service checks Redis cache for metadata
18. On cache miss, queries Cassandra
19. Returns CDN URL for photo retrieval

**Search Flow (Step 20):**
20. Search Service queries Elasticsearch index

**Sharing Flow (Step 21):**
21. Sharing Service validates permissions in PostgreSQL

**Face Search Flow:**

- Query face embeddings in Vector DB using similarity search
- Return photos containing matching faces

---

## 4. DATABASE DESIGN

### User Database (PostgreSQL - Relational)

**Users Table:**

```sql
- user_id (PK, UUID)
- email (VARCHAR(255), UNIQUE, NOT NULL)
- username (VARCHAR(100), UNIQUE)
- password_hash (VARCHAR(255))
- created_at (TIMESTAMP)
- last_login (TIMESTAMP)
- storage_quota_gb (INT, DEFAULT 15)
- storage_used_gb (DECIMAL(10,2))
- subscription_tier (ENUM: 'free', 'premium')
- INDEX: idx_email
- INDEX: idx_username
```text

**Albums Table:**

```sql
- album_id (PK, UUID)
- user_id (FK -> Users.user_id)
- album_name (VARCHAR(255))
- description (TEXT)
- cover_photo_id (FK -> Photos.photo_id, NULL)
- is_shared (BOOLEAN, DEFAULT FALSE)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
- INDEX: idx_user_id
- INDEX: idx_created_at
```text

**Sharing Table:**

```sql
- share_id (PK, UUID)
- resource_type (ENUM: 'photo', 'album')
- resource_id (UUID)
- owner_id (FK -> Users.user_id)
- shared_with_user_id (FK -> Users.user_id, NULL)
- share_link (VARCHAR(255), UNIQUE, NULL)
- permission_level (ENUM: 'view', 'edit')
- is_public (BOOLEAN, DEFAULT FALSE)
- expires_at (TIMESTAMP, NULL)
- created_at (TIMESTAMP)
- INDEX: idx_resource_id
- INDEX: idx_owner_id
- INDEX: idx_share_link
```text

### Metadata Database (Cassandra - NoSQL)

**Photos Metadata Table:**

```text
Partition Key: user_id
Clustering Key: upload_date (DESC), photo_id

Columns:
- photo_id (UUID)
- user_id (UUID)
- file_name (TEXT)
- file_size_bytes (BIGINT)
- original_format (TEXT)
- mime_type (TEXT)
- width (INT)
- height (INT)
- upload_date (TIMESTAMP)
- capture_date (TIMESTAMP)
- storage_location (TEXT) // S3 key
- thumbnail_locations (MAP<TEXT, TEXT>) // size -> S3 key
- processing_status (TEXT) // pending, completed, failed
- device_info (TEXT)
- camera_make (TEXT)
- camera_model (TEXT)
- latitude (DOUBLE)
- longitude (DOUBLE)
- location_name (TEXT)
- album_ids (SET<UUID>)
- tags (SET<TEXT>)
- is_favorite (BOOLEAN)
- is_deleted (BOOLEAN)
- deleted_at (TIMESTAMP)
```text

**Videos Metadata Table:**

```text
Partition Key: user_id
Clustering Key: upload_date (DESC), video_id

Columns:
- video_id (UUID)
- user_id (UUID)
- file_name (TEXT)
- file_size_bytes (BIGINT)
- duration_seconds (INT)
- resolution (TEXT) // 1080p, 4K, etc.
- codec (TEXT)
- bitrate (INT)
- fps (INT)
- upload_date (TIMESTAMP)
- capture_date (TIMESTAMP)
- storage_location (TEXT)
- thumbnail_location (TEXT)
- processing_status (TEXT)
- device_info (TEXT)
- latitude (DOUBLE)
- longitude (DOUBLE)
- location_name (TEXT)
- album_ids (SET<UUID>)
- tags (SET<TEXT>)
- is_favorite (BOOLEAN)
- is_deleted (BOOLEAN)
- deleted_at (TIMESTAMP)
```text

### Search Index (Elasticsearch)

**Photos Index:**

```json
{
  "photo_id": "uuid",
  "user_id": "uuid",
  "file_name": "text",
  "upload_date": "date",
  "capture_date": "date",
  "tags": ["text"],
  "location_name": "text",
  "geo_location": {
    "lat": "double",
    "lon": "double"
  },
  "camera_make": "text",
  "camera_model": "text",
  "device_info": "text",
  "width": "integer",
  "height": "integer",
  "is_favorite": "boolean",
  "album_ids": ["uuid"]
}
```text

### Face and People Database (PostgreSQL)

**People Table:**

```sql
- person_id (PK, UUID)
- user_id (FK -> Users.user_id)
- person_name (VARCHAR(255), NULL) // NULL for unnamed face groups
- is_confirmed (BOOLEAN, DEFAULT FALSE) // User confirmed the grouping
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
- face_count (INT) // Number of faces in this person group
- cover_face_id (FK -> Faces.face_id, NULL)
- INDEX: idx_user_id
- INDEX: idx_person_name
- UNIQUE: (user_id, person_name) WHERE person_name IS NOT NULL
```text

**Faces Table:**

```sql
- face_id (PK, UUID)
- photo_id (FK -> Photos.photo_id)
- user_id (FK -> Users.user_id)
- person_id (FK -> People.person_id, NULL) // NULL if unassigned
- bounding_box (JSONB) // {x, y, width, height}
- confidence_score (DECIMAL(5,4)) // 0.0 to 1.0
- embedding_vector_id (VARCHAR(255)) // Reference to vector in Milvus
- detected_at (TIMESTAMP)
- quality_score (DECIMAL(5,4)) // Face quality for grouping
- is_deleted (BOOLEAN, DEFAULT FALSE)
- INDEX: idx_photo_id
- INDEX: idx_user_id
- INDEX: idx_person_id
- INDEX: idx_detected_at
```text

**Face_Clusters Table (for grouping):**

```sql
- cluster_id (PK, UUID)
- user_id (FK -> Users.user_id)
- representative_face_id (FK -> Faces.face_id)
- cluster_size (INT)
- average_confidence (DECIMAL(5,4))
- created_at (TIMESTAMP)
- merged_into_person_id (FK -> People.person_id, NULL)
- INDEX: idx_user_id
- INDEX: idx_created_at
```text

### Vector Database (Milvus/Pinecone)

**Face Embeddings Collection:**

```python
Collection Schema:
- face_id (VARCHAR, PRIMARY KEY)
- user_id (VARCHAR, PARTITION KEY) // For data isolation
- photo_id (VARCHAR)
- person_id (VARCHAR, NULL)
- embedding (FLOAT_VECTOR, dimension=512) // FaceNet/ArcFace embedding
- quality_score (FLOAT)
- detected_at (INT64) // Unix timestamp

Index Configuration:
- Index type: IVF_FLAT or HNSW
- Metric type: COSINE or L2 (Euclidean)
- Search parameters: nprobe=16, ef=200
- Partitioning: By user_id for isolation and performance
```text

**Vector Search Query Example:**

```python
# Find similar faces
search_params = {
    "metric_type": "COSINE",
    "params": {"nprobe": 16}
}

results = collection.search(
    data=[query_embedding],
    anns_field="embedding",
    param=search_params,
    limit=100,
    expr=f"user_id == '{user_id}' and quality_score > 0.7"
)
```text

### Cache Layer (Redis)

**Cache Keys Structure:**

```text
photo_metadata:{photo_id} -> JSON (TTL: 24h)
user_photos:{user_id}:{page} -> List of photo_ids (TTL: 1h)
album_photos:{album_id}:{page} -> List of photo_ids (TTL: 1h)
user_albums:{user_id} -> List of album_ids (TTL: 6h)
thumbnail_url:{photo_id}:{size} -> CDN URL (TTL: 7d)
share_permissions:{share_link} -> Permissions JSON (TTL: 1h)
person_photos:{person_id}:{page} -> List of photo_ids (TTL: 1h)
user_people:{user_id} -> List of person_ids (TTL: 6h)
face_clusters:{user_id} -> Cluster data (TTL: 12h)
```text

---

## 5. API DESIGN

### Base Configuration

**Base URL:** `https://api.googlephotos.example.com/v1`

**Authentication:**

- OAuth 2.0 with JWT tokens
- All requests require: `Authorization: Bearer <access_token>`
- Token expiry: 1 hour (access), 30 days (refresh)

**Versioning:** URL path versioning (`/v1`, `/v2`)

**Rate Limiting:**

- Standard tier: 1,000 requests/hour per user
- Premium tier: 10,000 requests/hour per user
- Upload endpoints: 100 uploads/hour per user

---

### Authentication Endpoints

#### 1. Register User

```http
POST /auth/register
```text

**Request:**

```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "username": "photouser"
}
```text

**Response (201):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "username": "photouser",
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "dGhpc2lzYXJlZnJlc2h0b2tlbg...",
  "expires_in": 3600
}
```text

#### 2. Login

```http
POST /auth/login
```text

**Request:**

```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```text

**Response (200):**

```json
{
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "dGhpc2lzYXJlZnJlc2h0b2tlbg...",
  "expires_in": 3600
}
```text

#### 3. Refresh Token

```http
POST /auth/refresh
```text

**Request:**

```json
{
  "refresh_token": "dGhpc2lzYXJlZnJlc2h0b2tlbg..."
}
```text

**Response (200):**

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 3600
}
```text

#### 4. Logout

```http
POST /auth/logout
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (204):** No content

---

### Photo Management Endpoints

#### 5. Upload Photo

```http
POST /photos/upload
```text

**Headers:**

- `Authorization: Bearer <access_token>`
- `Content-Type: multipart/form-data`

**Request (Form Data):**

```text
photo: [binary file]
file_name: "vacation.jpg"
capture_date: "2025-09-15T14:30:00Z" (optional)
album_id: "uuid" (optional)
tags: ["vacation", "beach"] (optional)
latitude: 37.7749 (optional)
longitude: -122.4194 (optional)
```text

**Response (201):**

```json
{
  "photo_id": "123e4567-e89b-12d3-a456-426614174000",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "file_name": "vacation.jpg",
  "file_size_bytes": 3145728,
  "upload_date": "2025-10-01T10:15:30Z",
  "capture_date": "2025-09-15T14:30:00Z",
  "processing_status": "pending",
  "thumbnail_urls": {
    "150": "https://cdn.example.com/thumbnails/150/photo_id.jpg",
    "400": "https://cdn.example.com/thumbnails/400/photo_id.jpg",
    "1080": "https://cdn.example.com/thumbnails/1080/photo_id.jpg"
  },
  "original_url": "https://cdn.example.com/photos/photo_id.jpg"
}
```text

**Error Response (413):**

```json
{
  "error": "file_too_large",
  "message": "File size exceeds maximum limit of 100MB",
  "max_size_bytes": 104857600
}
```text

#### 6. Get Photo Details

```http
GET /photos/{photo_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "photo_id": "123e4567-e89b-12d3-a456-426614174000",
  "user_id": "550e8400-e29b-41d4-a716-446655440000",
  "file_name": "vacation.jpg",
  "file_size_bytes": 3145728,
  "width": 4032,
  "height": 3024,
  "upload_date": "2025-10-01T10:15:30Z",
  "capture_date": "2025-09-15T14:30:00Z",
  "location": {
    "latitude": 37.7749,
    "longitude": -122.4194,
    "location_name": "San Francisco, CA"
  },
  "camera_info": {
    "make": "Apple",
    "model": "iPhone 14 Pro"
  },
  "urls": {
    "original": "https://cdn.example.com/photos/photo_id.jpg",
    "thumbnails": {
      "150": "https://cdn.example.com/thumbnails/150/photo_id.jpg",
      "400": "https://cdn.example.com/thumbnails/400/photo_id.jpg",
      "1080": "https://cdn.example.com/thumbnails/1080/photo_id.jpg"
    }
  },
  "album_ids": ["album-uuid-1", "album-uuid-2"],
  "tags": ["vacation", "beach"],
  "is_favorite": false
}
```text

#### 7. Get User Photos

```http
GET /photos
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `page` (integer, default: 1)
- `page_size` (integer, default: 50, max: 100)
- `sort_by` (enum: "upload_date", "capture_date", default: "upload_date")
- `order` (enum: "asc", "desc", default: "desc")
- `album_id` (uuid, optional) - filter by album
- `is_favorite` (boolean, optional) - filter favorites only

**Response (200):**

```json
{
  "photos": [
    {
      "photo_id": "123e4567-e89b-12d3-a456-426614174000",
      "file_name": "vacation.jpg",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo_id.jpg",
      "capture_date": "2025-09-15T14:30:00Z",
      "is_favorite": false
    }
  ],
  "pagination": {
    "current_page": 1,
    "page_size": 50,
    "total_items": 1250,
    "total_pages": 25,
    "has_next": true,
    "has_previous": false
  }
}
```text

#### 8. Update Photo

```http
PATCH /photos/{photo_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "file_name": "updated_vacation.jpg",
  "tags": ["vacation", "beach", "sunset"],
  "is_favorite": true
}
```text

**Response (200):**

```json
{
  "photo_id": "123e4567-e89b-12d3-a456-426614174000",
  "message": "Photo updated successfully",
  "updated_fields": ["file_name", "tags", "is_favorite"]
}
```text

#### 9. Delete Photo

```http
DELETE /photos/{photo_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `permanent` (boolean, default: false) - Soft delete vs permanent delete

**Response (200):**

```json
{
  "message": "Photo moved to trash",
  "photo_id": "123e4567-e89b-12d3-a456-426614174000",
  "deleted_at": "2025-10-01T15:30:00Z",
  "permanent_deletion_date": "2025-10-31T15:30:00Z"
}
```text

#### 10. Batch Upload Photos

```http
POST /photos/batch-upload
```text

**Headers:**

- `Authorization: Bearer <access_token>`
- `Content-Type: multipart/form-data`

**Request (Form Data):**

```text
photos: [array of binary files, max 50 per request]
album_id: "uuid" (optional)
```text

**Response (202):**

```json
{
  "batch_id": "batch-uuid",
  "status": "processing",
  "total_photos": 25,
  "processed": 0,
  "status_url": "/photos/batch-upload/batch-uuid/status"
}
```text

#### 11. Get Batch Upload Status

```http
GET /photos/batch-upload/{batch_id}/status
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "batch_id": "batch-uuid",
  "status": "processing",
  "total_photos": 25,
  "processed": 15,
  "successful": 14,
  "failed": 1,
  "progress_percentage": 60,
  "photos": [
    {
      "file_name": "photo1.jpg",
      "status": "completed",
      "photo_id": "uuid"
    },
    {
      "file_name": "photo2.jpg",
      "status": "failed",
      "error": "Invalid format"
    }
  ]
}
```text

---

### Search Endpoints

#### 12. Search Photos

```http
GET /search/photos
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `query` (string, optional) - Text search query
- `start_date` (ISO 8601, optional) - Filter by capture date
- `end_date` (ISO 8601, optional) - Filter by capture date
- `location` (string, optional) - Location name search
- `tags` (array, optional) - Filter by tags
- `camera_make` (string, optional)
- `camera_model` (string, optional)
- `page` (integer, default: 1)
- `page_size` (integer, default: 50, max: 100)

**Response (200):**

```json
{
  "results": [
    {
      "photo_id": "uuid",
      "file_name": "beach_sunset.jpg",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo_id.jpg",
      "capture_date": "2025-09-15T18:30:00Z",
      "location_name": "Malibu Beach, CA",
      "relevance_score": 0.95
    }
  ],
  "total_results": 127,
  "search_time_ms": 45,
  "pagination": {
    "current_page": 1,
    "page_size": 50,
    "total_pages": 3
  }
}
```text

#### 13. Search by Location

```http
GET /search/location
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `latitude` (double, required)
- `longitude` (double, required)
- `radius_km` (double, default: 10, max: 100)
- `page` (integer, default: 1)
- `page_size` (integer, default: 50)

**Response (200):**

```json
{
  "results": [
    {
      "photo_id": "uuid",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo_id.jpg",
      "capture_date": "2025-09-15T18:30:00Z",
      "location": {
        "latitude": 37.7749,
        "longitude": -122.4194,
        "location_name": "San Francisco, CA"
      },
      "distance_km": 2.5
    }
  ],
  "center": {
    "latitude": 37.7749,
    "longitude": -122.4194
  },
  "radius_km": 10,
  "total_results": 45
}
```text

---

### Album Management Endpoints

#### 14. Create Album

```http
POST /albums
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "album_name": "Summer Vacation 2025",
  "description": "Our amazing summer trip to Hawaii",
  "cover_photo_id": "uuid" (optional)
}
```text

**Response (201):**

```json
{
  "album_id": "album-uuid",
  "user_id": "user-uuid",
  "album_name": "Summer Vacation 2025",
  "description": "Our amazing summer trip to Hawaii",
  "cover_photo_id": "uuid",
  "photo_count": 0,
  "created_at": "2025-10-01T10:00:00Z",
  "is_shared": false
}
```text

#### 15. Get Album Details

```http
GET /albums/{album_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "album_id": "album-uuid",
  "user_id": "user-uuid",
  "album_name": "Summer Vacation 2025",
  "description": "Our amazing summer trip to Hawaii",
  "cover_photo_url": "https://cdn.example.com/thumbnails/400/cover_photo.jpg",
  "photo_count": 125,
  "created_at": "2025-10-01T10:00:00Z",
  "updated_at": "2025-10-01T15:30:00Z",
  "is_shared": true,
  "share_link": "https://photos.example.com/shared/abc123"
}
```text

#### 16. Add Photos to Album

```http
POST /albums/{album_id}/photos
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "photo_ids": [
    "photo-uuid-1",
    "photo-uuid-2",
    "photo-uuid-3"
  ]
}
```text

**Response (200):**

```json
{
  "album_id": "album-uuid",
  "added_count": 3,
  "total_photos": 128,
  "message": "Photos added successfully"
}
```text

#### 17. Remove Photos from Album

```http
DELETE /albums/{album_id}/photos
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "photo_ids": [
    "photo-uuid-1",
    "photo-uuid-2"
  ]
}
```text

**Response (200):**

```json
{
  "album_id": "album-uuid",
  "removed_count": 2,
  "total_photos": 126,
  "message": "Photos removed successfully"
}
```text

#### 18. Get User Albums

```http
GET /albums
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `page` (integer, default: 1)
- `page_size` (integer, default: 20, max: 50)
- `sort_by` (enum: "created_at", "updated_at", "name", default: "updated_at")
- `order` (enum: "asc", "desc", default: "desc")

**Response (200):**

```json
{
  "albums": [
    {
      "album_id": "album-uuid",
      "album_name": "Summer Vacation 2025",
      "cover_photo_url": "https://cdn.example.com/thumbnails/400/cover.jpg",
      "photo_count": 125,
      "created_at": "2025-10-01T10:00:00Z",
      "is_shared": true
    }
  ],
  "pagination": {
    "current_page": 1,
    "page_size": 20,
    "total_items": 15,
    "total_pages": 1
  }
}
```text

#### 19. Update Album

```http
PATCH /albums/{album_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "album_name": "Hawaii Vacation 2025",
  "description": "Updated description",
  "cover_photo_id": "new-cover-uuid"
}
```text

**Response (200):**

```json
{
  "album_id": "album-uuid",
  "message": "Album updated successfully",
  "updated_fields": ["album_name", "description", "cover_photo_id"]
}
```text

#### 20. Delete Album

```http
DELETE /albums/{album_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `delete_photos` (boolean, default: false) - Delete photos or just remove from album

**Response (200):**

```json
{
  "message": "Album deleted successfully",
  "album_id": "album-uuid",
  "photos_deleted": false
}
```text

---

### Sharing Endpoints

#### 21. Create Share Link

```http
POST /sharing/create
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "resource_type": "album",
  "resource_id": "album-uuid",
  "permission_level": "view",
  "is_public": true,
  "expires_at": "2025-12-31T23:59:59Z" (optional)
}
```text

**Response (201):**

```json
{
  "share_id": "share-uuid",
  "share_link": "https://photos.example.com/shared/abc123def456",
  "short_code": "abc123def456",
  "resource_type": "album",
  "resource_id": "album-uuid",
  "permission_level": "view",
  "is_public": true,
  "expires_at": "2025-12-31T23:59:59Z",
  "created_at": "2025-10-01T10:00:00Z"
}
```text

#### 22. Share with Specific User

```http
POST /sharing/invite
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "resource_type": "album",
  "resource_id": "album-uuid",
  "shared_with_email": "friend@example.com",
  "permission_level": "edit"
}
```text

**Response (201):**

```json
{
  "share_id": "share-uuid",
  "resource_type": "album",
  "resource_id": "album-uuid",
  "shared_with_user_id": "friend-user-uuid",
  "shared_with_email": "friend@example.com",
  "permission_level": "edit",
  "message": "Invitation sent successfully"
}
```text

#### 23. Get Shared Resource

```http
GET /sharing/{share_code}
```text

**Headers:** `Authorization: Bearer <access_token>` (optional for public shares)

**Response (200):**

```json
{
  "resource_type": "album",
  "resource_id": "album-uuid",
  "album_name": "Summer Vacation 2025",
  "owner": {
    "user_id": "owner-uuid",
    "username": "photouser"
  },
  "permission_level": "view",
  "photo_count": 125,
  "cover_photo_url": "https://cdn.example.com/thumbnails/400/cover.jpg",
  "expires_at": "2025-12-31T23:59:59Z"
}
```text

#### 24. Revoke Share

```http
DELETE /sharing/{share_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "message": "Share access revoked successfully",
  "share_id": "share-uuid"
}
```text

#### 25. List My Shares

```http
GET /sharing/my-shares
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `type` (enum: "created", "received", default: "created")
- `page` (integer, default: 1)
- `page_size` (integer, default: 20)

**Response (200):**

```json
{
  "shares": [
    {
      "share_id": "share-uuid",
      "resource_type": "album",
      "resource_id": "album-uuid",
      "album_name": "Summer Vacation 2025",
      "share_link": "https://photos.example.com/shared/abc123",
      "permission_level": "view",
      "is_public": true,
      "created_at": "2025-10-01T10:00:00Z",
      "expires_at": null,
      "view_count": 42
    }
  ],
  "pagination": {
    "current_page": 1,
    "page_size": 20,
    "total_items": 8,
    "total_pages": 1
  }
}
```text

---

### User Profile Endpoints

#### 26. Get User Profile

```http
GET /users/me
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "user_id": "user-uuid",
  "email": "user@example.com",
  "username": "photouser",
  "created_at": "2024-01-15T10:00:00Z",
  "subscription_tier": "premium",
  "storage": {
    "quota_gb": 100,
    "used_gb": 45.7,
    "available_gb": 54.3,
    "usage_percentage": 45.7
  },
  "statistics": {
    "total_photos": 3456,
    "total_videos": 234,
    "total_albums": 12,
    "shared_albums": 5
  }
}
```text

#### 27. Update User Profile

```http
PATCH /users/me
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "username": "newusername",
  "email": "newemail@example.com"
}
```text

**Response (200):**

```json
{
  "user_id": "user-uuid",
  "username": "newusername",
  "email": "newemail@example.com",
  "message": "Profile updated successfully"
}
```text

#### 28. Get Storage Statistics

```http
GET /users/me/storage
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "storage": {
    "quota_gb": 100,
    "used_gb": 45.7,
    "breakdown": {
      "photos_gb": 38.2,
      "videos_gb": 7.5,
      "thumbnails_gb": 0.0
    },
    "available_gb": 54.3,
    "usage_percentage": 45.7
  },
  "recent_growth": {
    "last_7_days_gb": 2.3,
    "last_30_days_gb": 8.9
  }
}
```text

---

### People & Face Management Endpoints

#### 29. Get People (Face Groups)

```http
GET /people
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `include_unnamed` (boolean, default: false) - Include unnamed face clusters
- `min_face_count` (integer, default: 3) - Minimum faces to show group
- `page` (integer, default: 1)
- `page_size` (integer, default: 50)

**Response (200):**

```json
{
  "people": [
    {
      "person_id": "person-uuid",
      "person_name": "John Doe",
      "is_confirmed": true,
      "face_count": 127,
      "cover_photo_url": "https://cdn.example.com/faces/person_cover.jpg",
      "created_at": "2025-01-15T10:00:00Z"
    },
    {
      "person_id": "person-uuid-2",
      "person_name": null,
      "is_confirmed": false,
      "face_count": 45,
      "cover_photo_url": "https://cdn.example.com/faces/cluster_cover.jpg",
      "created_at": "2025-09-20T14:30:00Z"
    }
  ],
  "pagination": {
    "current_page": 1,
    "page_size": 50,
    "total_items": 23,
    "total_pages": 1
  }
}
```text

#### 30. Get Person Details

```http
GET /people/{person_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "person_id": "person-uuid",
  "person_name": "John Doe",
  "is_confirmed": true,
  "face_count": 127,
  "cover_photo_url": "https://cdn.example.com/faces/person_cover.jpg",
  "created_at": "2025-01-15T10:00:00Z",
  "updated_at": "2025-09-30T18:20:00Z",
  "sample_photos": [
    {
      "photo_id": "photo-uuid-1",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo1.jpg",
      "face_bounding_box": {"x": 120, "y": 80, "width": 200, "height": 250}
    }
  ]
}
```text

#### 31. Name a Person

```http
PATCH /people/{person_id}/name
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "person_name": "John Doe",
  "is_confirmed": true
}
```text

**Response (200):**

```json
{
  "person_id": "person-uuid",
  "person_name": "John Doe",
  "is_confirmed": true,
  "message": "Person named successfully"
}
```text

#### 32. Get Photos by Person

```http
GET /people/{person_id}/photos
```text

**Headers:** `Authorization: Bearer <access_token>`

**Query Parameters:**

- `page` (integer, default: 1)
- `page_size` (integer, default: 50, max: 100)
- `sort_by` (enum: "capture_date", "upload_date", default: "capture_date")
- `order` (enum: "asc", "desc", default: "desc")

**Response (200):**

```json
{
  "person_id": "person-uuid",
  "person_name": "John Doe",
  "photos": [
    {
      "photo_id": "photo-uuid",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo.jpg",
      "capture_date": "2025-09-15T14:30:00Z",
      "face_bounding_box": {"x": 120, "y": 80, "width": 200, "height": 250},
      "confidence_score": 0.95
    }
  ],
  "pagination": {
    "current_page": 1,
    "page_size": 50,
    "total_items": 127,
    "total_pages": 3
  }
}
```text

#### 33. Merge People

```http
POST /people/merge
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "source_person_ids": ["person-uuid-1", "person-uuid-2"],
  "target_person_id": "person-uuid-3",
  "keep_name_from": "person-uuid-3"
}
```text

**Response (200):**

```json
{
  "message": "People merged successfully",
  "merged_person_id": "person-uuid-3",
  "total_faces": 245,
  "source_persons_deleted": 2
}
```text

#### 34. Remove Face from Person

```http
DELETE /people/{person_id}/faces/{face_id}
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "message": "Face removed from person",
  "person_id": "person-uuid",
  "face_id": "face-uuid",
  "remaining_face_count": 126
}
```text

#### 35. Search Photos by Face

```http
POST /search/by-face
```text

**Headers:**

- `Authorization: Bearer <access_token>`
- `Content-Type: multipart/form-data`

**Request (Form Data):**

```text
reference_photo: [binary file or photo_id]
face_id: "face-uuid" (optional, if using existing face)
threshold: 0.75 (optional, similarity threshold 0.0-1.0, default: 0.75)
```text

**Response (200):**

```json
{
  "query_face_id": "face-uuid",
  "matches": [
    {
      "photo_id": "photo-uuid",
      "face_id": "matched-face-uuid",
      "thumbnail_url": "https://cdn.example.com/thumbnails/400/photo.jpg",
      "similarity_score": 0.93,
      "capture_date": "2025-08-10T16:20:00Z",
      "face_bounding_box": {"x": 100, "y": 60, "width": 180, "height": 220}
    }
  ],
  "total_matches": 45,
  "search_time_ms": 127
}
```text

#### 36. Hide Person

```http
PATCH /people/{person_id}/hide
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "is_hidden": true
}
```text

**Response (200):**

```json
{
  "message": "Person hidden from main view",
  "person_id": "person-uuid"
}
```text

**Note:** Hidden people won't appear in the main people list but faces remain linked.

#### 37. Get Face Detection Status

```http
GET /photos/{photo_id}/faces
```text

**Headers:** `Authorization: Bearer <access_token>`

**Response (200):**

```json
{
  "photo_id": "photo-uuid",
  "face_detection_status": "completed",
  "faces_detected": 3,
  "faces": [
    {
      "face_id": "face-uuid-1",
      "person_id": "person-uuid",
      "person_name": "John Doe",
      "bounding_box": {"x": 120, "y": 80, "width": 200, "height": 250},
      "confidence_score": 0.95,
      "quality_score": 0.88
    },
    {
      "face_id": "face-uuid-2",
      "person_id": null,
      "person_name": null,
      "bounding_box": {"x": 450, "y": 120, "width": 190, "height": 240},
      "confidence_score": 0.89,
      "quality_score": 0.76
    }
  ]
}
```text

#### 38. Tag Person in Photo

```http
POST /photos/{photo_id}/tag-person
```text

**Headers:** `Authorization: Bearer <access_token>`

**Request:**

```json
{
  "face_id": "face-uuid",
  "person_id": "person-uuid"
}
```text

**Response (200):**

```json
{
  "message": "Person tagged successfully",
  "photo_id": "photo-uuid",
  "face_id": "face-uuid",
  "person_id": "person-uuid",
  "person_name": "John Doe"
}
```text

---

### Cross-Cutting API Concerns

**Standard Error Response Format:**

```json
{
  "error": "error_code",
  "message": "Human readable error message",
  "details": {
    "field": "specific field with issue",
    "reason": "detailed reason"
  },
  "request_id": "req-uuid",
  "timestamp": "2025-10-01T10:00:00Z"
}
```text

**Common HTTP Status Codes:**

- `200 OK` - Successful request
- `201 Created` - Resource created successfully
- `202 Accepted` - Request accepted for processing
- `204 No Content` - Successful with no response body
- `400 Bad Request` - Invalid request format
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `413 Payload Too Large` - File size exceeds limit
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Service temporarily unavailable

**Rate Limiting Headers:**

```text
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1696156800
```text

**Pagination Strategy:**

- Offset-based pagination for stable datasets (photos, albums)
- Cursor-based pagination for real-time feeds (future feature)

**Idempotency:**

- Upload endpoints accept `Idempotency-Key` header
- Duplicate requests with same key return original response
- Keys valid for 24 hours

**Compression:**

- Supports gzip compression via `Accept-Encoding: gzip`
- Response includes `Content-Encoding: gzip`

**CORS Policy:**

- Allowed origins: configured domains only
- Supports preflight OPTIONS requests
- Credentials allowed for authenticated requests

---

### API Trade-Offs

**Decision: REST vs GraphQL**

- **Choice:** REST API
- **Pros:** Simpler to implement, better caching, wider client support, predictable performance
- **Cons:** Multiple requests for complex queries, over-fetching data
- **Justification:** Photo management has predictable access patterns with well-defined resources. REST's caching benefits are crucial for serving millions of photo URLs.

**Decision: Synchronous vs Asynchronous Upload Processing**

- **Choice:** Hybrid approach - synchronous upload, asynchronous processing
- **Pros:** Immediate upload confirmation, non-blocking thumbnail generation, better resource utilization
- **Cons:** Delayed availability of thumbnails and metadata
- **Justification:** Users need quick upload feedback, but thumbnail generation and metadata extraction can happen asynchronously without impacting UX.

**Decision: Pagination Strategy**

- **Choice:** Offset-based pagination with page numbers
- **Pros:** Simple to implement, allows jumping to specific pages, familiar to users
- **Cons:** Performance degrades with deep pagination, inconsistent results if data changes
- **Justification:** Most users browse recent photos (first few pages), making offset pagination performant for 95% of use cases.

**Decision: Endpoint Granularity**

- **Choice:** Resource-oriented with separate endpoints for different resources
- **Pros:** Clear responsibility, easier to cache, better rate limiting control
- **Cons:** More network requests for related data
- **Justification:** Separation allows independent scaling and caching strategies for photos vs albums vs search.

---

## 6. DEEP-DIVE COMPONENTS & TRADE-OFFS

### 6.1 Face Recognition Pipeline

**Purpose:** Automatically detect faces, generate embeddings, cluster similar faces, and enable face-based search and organization

**Architecture:**

```mermaid
graph TB
    subgraph Input
        Photo[Uploaded Photo]
    end
    
    subgraph Detection Stage
        FaceDetector[Face Detector<br/>MTCNN/RetinaFace]
        QualityFilter[Quality Filter<br/>Blur/Angle/Size]
    end
    
    subgraph Recognition Stage
        Alignment[Face Alignment<br/>Landmark Detection]
        Embedding[Embedding Generator<br/>FaceNet/ArcFace]
        Normalize[L2 Normalization]
    end
    
    subgraph Storage Stage
        VectorDB[(Vector DB<br/>Milvus)]
        FacesDB[(Faces DB<br/>PostgreSQL)]
    end
    
    subgraph Clustering Stage
        Similarity[Similarity Search<br/>HNSW/IVF]
        Clustering[DBSCAN Clustering<br/>Nightly Batch]
        Merge[Cluster Merging]
    end
    
    subgraph Output
        PersonGroup[Person Groups]
    end
    
    Photo --> FaceDetector
    FaceDetector -->|Bounding Boxes| QualityFilter
    QualityFilter -->|High Quality Faces| Alignment
    QualityFilter -->|Low Quality| FacesDB
    Alignment --> Embedding
    Embedding --> Normalize
    Normalize --> VectorDB
    Normalize --> FacesDB
    
    VectorDB --> Similarity
    Similarity --> Clustering
    Clustering --> Merge
    Merge --> PersonGroup
    PersonGroup --> FacesDB
```text

**Pipeline Stages:**

**1. Face Detection:**

- **Model:** MTCNN (Multi-task Cascaded CNN) or RetinaFace
- **Process:**
  - Detect face bounding boxes in image
  - Extract facial landmarks (eyes, nose, mouth)
  - Calculate confidence score (0.0-1.0)
- **Filter:** Keep faces with confidence > 0.7
- **Performance:** ~2 seconds per photo on GPU (P100/V100)

```python
def detect_faces(image_path):
    """
    Detect faces in an image
    
    Args:
        image_path: Path to image file
    
    Returns:
        List of face bounding boxes with confidence scores
    """
    detector = MTCNN(device='cuda')
    image = cv2.imread(image_path)
    
    # Detect faces
    boxes, probs, landmarks = detector.detect(image, landmarks=True)
    
    faces = []
    for box, prob, landmark in zip(boxes, probs, landmarks):
        if prob > 0.7:  # Confidence threshold
            faces.append({
                'bounding_box': {
                    'x': int(box[0]),
                    'y': int(box[1]),
                    'width': int(box[2] - box[0]),
                    'height': int(box[3] - box[1])
                },
                'confidence': float(prob),
                'landmarks': landmark.tolist()
            })
    
    return faces
```text

**2. Quality Filtering:**

- **Filters:**
  - Minimum face size: 80x80 pixels
  - Blur detection (Laplacian variance > threshold)
  - Face angle (reject if pitch/yaw > 45 degrees)
  - Occlusion detection (sunglasses, masks)
- **Quality Score:** 0.0-1.0 based on multiple factors
- **Threshold:** Only process faces with quality > 0.5

**3. Face Alignment:**

- Align face to canonical pose using detected landmarks
- Normalize rotation, scale, and translation
- Crop to 160x160 pixels (FaceNet input size)
- Histogram equalization for lighting normalization

**4. Embedding Generation:**

- **Model:** FaceNet (Inception-ResNet-v1) or ArcFace
- **Output:** 512-dimensional embedding vector
- **Process:**

  ```python
  def generate_embedding(aligned_face):
      """
      Generate face embedding using FaceNet
      
      Args:
          aligned_face: Aligned and cropped face image (160x160)
      
      Returns:
          512-dimensional embedding vector
      """
      model = InceptionResnetV1(pretrained='vggface2').eval().cuda()
      
      # Preprocess
      face_tensor = transforms.ToTensor()(aligned_face)
      face_tensor = face_tensor.unsqueeze(0).cuda()
      
      # Generate embedding
      with torch.no_grad():
          embedding = model(face_tensor)
      
      # L2 normalization
      embedding = F.normalize(embedding, p=2, dim=1)
      
      return embedding.cpu().numpy()[0]
```text

- **Properties:**
  - Same person: embedding distance < 0.6
  - Different people: embedding distance > 1.0
  - Use cosine similarity or Euclidean distance

**5. Vector Storage:**

- Store embeddings in Milvus (vector database)
- Partition by user_id for data isolation
- Index type: HNSW (Hierarchical Navigable Small World)
- Metric: Cosine similarity
- Search performance: <100ms for 1M vectors

```python
def store_embedding(face_id, user_id, photo_id, embedding):
    """
    Store face embedding in vector database
    
    Args:
        face_id: Unique face identifier
        user_id: User who owns the photo
        photo_id: Photo containing the face
        embedding: 512-dimensional vector
    """
    collection = get_milvus_collection('face_embeddings')
    
    # Prepare data
    data = {
        'face_id': face_id,
        'user_id': user_id,
        'photo_id': photo_id,
        'embedding': embedding.tolist(),
        'quality_score': calculate_quality(embedding),
        'detected_at': int(time.time())
    }
    
    # Insert with partition
    collection.insert(
        data=[data],
        partition_name=f"user_{user_id}"
    )
    
    # Also store metadata in PostgreSQL
    store_face_metadata(face_id, user_id, photo_id, ...)
```text

**6. Face Clustering (Batch Job):**

Run nightly clustering for new faces:

```python
def cluster_new_faces(user_id, since_timestamp):
    """
    Cluster newly detected faces for a user
    
    Process:
    1. Get all unclustered faces
    2. Perform similarity search for each face
    3. Use DBSCAN to cluster similar faces
    4. Create person groups from clusters
    
    Args:
        user_id: User to cluster faces for
        since_timestamp: Only cluster faces after this time
    """
    # Get unclustered faces
    faces = get_unclustered_faces(user_id, since_timestamp)
    
    # Build similarity graph
    similarity_matrix = []
    for face in faces:
        # Search for similar faces in vector DB
        similar_faces = milvus_search(
            embedding=face.embedding,
            user_id=user_id,
            threshold=0.6,
            limit=100
        )
        similarity_matrix.append(similar_faces)
    
    # DBSCAN clustering
    # eps: maximum distance between two samples
    # min_samples: minimum faces to form a cluster
    clustering = DBSCAN(eps=0.6, min_samples=3, metric='cosine')
    labels = clustering.fit_predict(embeddings)
    
    # Create person groups from clusters
    for cluster_id in set(labels):
        if cluster_id == -1:  # Noise/outliers
            continue
        
        cluster_faces = faces[labels == cluster_id]
        
        # Create person group
        person_id = create_person_group(
            user_id=user_id,
            face_ids=[f.face_id for f in cluster_faces],
            representative_face=select_best_face(cluster_faces)
        )
        
        # Update faces with person_id
        update_faces_person_id(cluster_faces, person_id)
```text

**7. Incremental Clustering:**

When new faces are detected:

```python
def assign_face_to_person(new_face_embedding, user_id):
    """
    Assign newly detected face to existing person or create new cluster
    
    Args:
        new_face_embedding: Embedding of newly detected face
        user_id: User who owns the photo
    
    Returns:
        person_id or None if no match
    """
    # Search for similar faces
    results = milvus_search(
        embedding=new_face_embedding,
        user_id=user_id,
        threshold=0.6,
        limit=10
    )
    
    if not results:
        return None  # Will be clustered in nightly job
    
    # Get person assignments for similar faces
    person_votes = {}
    for result in results:
        if result.person_id:
            score = result.similarity_score
            person_votes[result.person_id] = person_votes.get(result.person_id, 0) + score
    
    if not person_votes:
        return None
    
    # Assign to person with highest vote
    best_person = max(person_votes, key=person_votes.get)
    
    # Confidence check: require multiple matches or very high similarity
    if person_votes[best_person] > 2.0 or results[0].similarity_score > 0.8:
        return best_person
    
    return None  # Not confident, defer to nightly clustering
```text

**Performance Optimizations:**

1. **GPU Acceleration:**
   - Batch processing: 100 photos per GPU
   - Mixed precision (FP16) for 2x speedup
   - Model optimization with TensorRT

2. **Parallel Processing:**
   - Separate workers for detection and embedding
   - Pipeline: while generating embedding for photo N, detect faces in photo N+1

3. **Smart Scheduling:**
   - Priority queue: recent uploads first
   - Throttle during peak hours
   - Background processing for old photos

**Technology Choices:**

**Face Detection Model:**

- **Choice:** RetinaFace
- **Pros:** High accuracy, good for various angles, real-time capable
- **Cons:** Slightly slower than simpler models
- **Justification:** Accuracy is critical for user trust; speed acceptable with GPU

**Face Recognition Model:**

- **Choice:** FaceNet (Inception-ResNet-v1) pre-trained on VGGFace2
- **Pros:** Industry-standard, 512-dim embeddings, excellent accuracy (99.6% on LFW)
- **Cons:** Model size (100MB), requires GPU
- **Justification:** Proven performance, manageable size, widely supported

**Vector Database:**

- **Choice:** Milvus
- **Pros:** Open-source, optimized for similarity search, scales horizontally, HNSW indexing
- **Cons:** Operational complexity, memory-intensive
- **Alternatives Considered:**
  - Pinecone (managed, easier, but vendor lock-in and cost)
  - Faiss (library only, need to build service layer)
- **Justification:** Best balance of performance, cost, and control for large-scale deployment

**Clustering Algorithm:**

- **Choice:** DBSCAN (Density-Based Spatial Clustering)
- **Pros:** No need to specify number of clusters, handles outliers well, works with cosine distance
- **Cons:** Requires tuning eps and min_samples parameters
- **Justification:** Suitable for face clustering where cluster count is unknown and varies by user

---

### 6.2 Upload Service Architecture

**Purpose:** Handle photo/video uploads with high throughput and reliability

**Internal Architecture:**

```mermaid
graph LR
    subgraph Upload Service
        Receiver[Request Receiver]
        Validator[File Validator]
        Chunker[Chunk Handler]
        S3Writer[S3 Writer]
        Queue[Kafka Producer]
    end
    
    Client -->|Multipart Upload| Receiver
    Receiver --> Validator
    Validator -->|Valid| Chunker
    Validator -->|Invalid| Client
    Chunker -->|Parallel Chunks| S3Writer
    S3Writer -->|Store| S3
    S3Writer -->|Success| Queue
    Queue -->|Event| ProcessingPipeline
```text

**Key Features:**

1. **Chunked Upload:**
   - Split files into 5MB chunks
   - Parallel upload to S3 using multipart upload API
   - Resume capability for failed uploads
   - Client-side MD5 checksum validation

2. **Validation:**
   - File type verification (magic number check)
   - File size limits (photos: 100MB, videos: 10GB)
   - Virus scanning integration
   - Duplicate detection using perceptual hashing

3. **S3 Storage Strategy:**
   - Initial upload to hot storage (S3 Standard)
   - Storage key pattern: `{user_id}/{year}/{month}/{photo_id}.{ext}`
   - Server-side encryption (SSE-S3)
   - Cross-region replication for disaster recovery

4. **Event Publishing:**
   - Publish to Kafka topic: `photo.uploaded`
   - Event payload: user_id, photo_id, S3 location, upload metadata
   - Exactly-once delivery semantics

**Scaling Strategy:**

- Horizontal scaling based on upload QPS
- Auto-scaling group: 100-2000 instances
- Connection pooling to S3
- Load balancer with sticky sessions for resumable uploads

**Technology Choice:**

- **Language:** Node.js for I/O-bound operations
- **Framework:** Express.js with Multer for multipart handling
- **Why:** Excellent async I/O, large ecosystem for file handling

---

### 6.2 Image Processing Pipeline

**Purpose:** Generate thumbnails, extract metadata, and prepare photos for fast delivery

**Architecture:**

```mermaid
graph TB
    Kafka[Kafka Topic<br/>photo.uploaded]
    
    subgraph Worker Pool
        Worker1[Image Processor 1]
        Worker2[Image Processor 2]
        WorkerN[Image Processor N]
    end
    
    subgraph Processing Steps
        Download[Download from S3]
        Extract[Metadata Extraction]
        Thumbnail[Thumbnail Generation]
        Optimize[Image Optimization]
    end
    
    subgraph Storage
        S3Thumb[S3 Thumbnails]
        Cassandra[Cassandra]
        ES[Elasticsearch]
    end
    
    Kafka -->|Consume| Worker1
    Kafka -->|Consume| Worker2
    Worker1 --> Download
    Download --> Extract
    Extract --> Thumbnail
    Thumbnail --> Optimize
    Optimize -->|Store| S3Thumb
    Extract -->|Metadata| Cassandra
    Extract -->|Index| ES
```text

**Processing Steps:**

1. **Metadata Extraction:**
   - Parse EXIF data (camera info, capture date, settings)
   - Extract GPS coordinates
   - Reverse geocoding for location names (Google Maps API)
   - Store in Cassandra

2. **Thumbnail Generation:**
   - Create 3 sizes: 150px, 400px, 1080px (longest edge)
   - Use ImageMagick for high-quality resizing
   - Progressive JPEG format for web
   - WebP format for modern browsers
   - Store with naming: `{photo_id}_{size}.jpg`

3. **Image Optimization:**
   - Compress original images (lossy/lossless based on quality)
   - Strip unnecessary metadata
   - Convert to efficient formats
   - Store optimized version

4. **Search Indexing:**
   - Index metadata in Elasticsearch
   - Full-text search on location, camera model, tags
   - Geo-spatial indexing for location-based search

**Worker Configuration:**

- Consumer group: `image-processing-workers`
- Parallelism: 2000 workers
- Processing timeout: 60s per photo
- Retry policy: 3 retries with exponential backoff
- Dead letter queue for failed processing

**Technology Choices:**

- **Language:** Python for image processing libraries
- **Library:** Pillow, ImageMagick bindings
- **Why:** Rich ecosystem for image manipulation, easy integration with ML libraries

**Performance Optimization:**

- Pre-allocate memory pools for image buffers
- GPU acceleration for batch thumbnail generation
- Lazy loading of processing libraries
- Connection pooling to S3 and Cassandra

---

### 6.3 Search Service

**Purpose:** Provide fast, relevant photo search across multiple dimensions

**Elasticsearch Index Design:**

**Index Settings:**

```json
{
  "settings": {
    "number_of_shards": 20,
    "number_of_replicas": 2,
    "refresh_interval": "5s"
  },
  "mappings": {
    "properties": {
      "photo_id": {"type": "keyword"},
      "user_id": {"type": "keyword"},
      "file_name": {"type": "text", "analyzer": "standard"},
      "upload_date": {"type": "date"},
      "capture_date": {"type": "date"},
      "tags": {"type": "keyword"},
      "location_name": {
        "type": "text",
        "fields": {
          "keyword": {"type": "keyword"}
        }
      },
      "geo_location": {"type": "geo_point"},
      "camera_make": {"type": "keyword"},
      "camera_model": {"type": "keyword"},
      "device_info": {"type": "text"},
      "width": {"type": "integer"},
      "height": {"type": "integer"},
      "is_favorite": {"type": "boolean"},
      "album_ids": {"type": "keyword"}
    }
  }
}
```text

**Query Examples:**

1. **Text Search:**

```json
{
  "query": {
    "bool": {
      "must": [
        {"match": {"location_name": "beach"}},
        {"term": {"user_id": "user-uuid"}}
      ]
    }
  }
}
```text

1. **Geo-spatial Search:**

```json
{
  "query": {
    "bool": {
      "must": {"term": {"user_id": "user-uuid"}},
      "filter": {
        "geo_distance": {
          "distance": "10km",
          "geo_location": {
            "lat": 37.7749,
            "lon": -122.4194
          }
        }
      }
    }
  }
}
```text

1. **Date Range Search:**

```json
{
  "query": {
    "bool": {
      "must": {"term": {"user_id": "user-uuid"}},
      "filter": {
        "range": {
          "capture_date": {
            "gte": "2025-01-01",
            "lte": "2025-12-31"
          }
        }
      }
    }
  }
}
```text

**Search Optimizations:**

- User-specific routing: route queries to shards by user_id
- Query caching for common searches
- Result caching with Redis (TTL: 5 minutes)
- Aggregations for faceted search (by date, location, camera)
- Highlighting for matched terms

**Scaling Strategy:**

- 20 shards for horizontal scaling
- 2 replicas for read scalability and fault tolerance
- Dedicated master nodes (3 nodes)
- Data nodes: auto-scaling based on index size
- Coordinating nodes for search requests

---

### 6.4 Storage Tiering Strategy

**Purpose:** Optimize storage costs while maintaining access performance

**Lifecycle Tiers:**

1. **Hot Storage (S3 Standard):**
   - Recent photos (< 30 days)
   - Frequently accessed photos
   - All thumbnails
   - Cost: $0.023/GB/month
   - Access latency: <10ms

2. **Warm Storage (S3 Intelligent-Tiering):**
   - Photos 30 days - 1 year old
   - Occasionally accessed photos
   - Automatic tiering based on access patterns
   - Cost: $0.0125/GB/month (after tiering)
   - Access latency: <10ms

3. **Cold Storage (S3 Glacier Flexible Retrieval):**
   - Photos > 1 year old
   - Rarely accessed photos
   - Cost: $0.0036/GB/month
   - Retrieval: 1-5 minutes

**Lifecycle Policies:**

```json
{
  "Rules": [
    {
      "Id": "Transition-to-IA",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 30,
          "StorageClass": "INTELLIGENT_TIERING"
        }
      ]
    },
    {
      "Id": "Transition-to-Glacier",
      "Status": "Enabled",
      "Transitions": [
        {
          "Days": 365,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```text

**Access Pattern Optimization:**

- Metadata always in Cassandra (fast access)
- Thumbnails always in hot storage
- Original photos transition based on age
- On-demand retrieval for cold storage with progress indicator
- Pre-warming: predictively move photos to hot storage based on access patterns

**Cost Savings:**

- Hot: 30 days × 1.5 PB/day × $0.023 = $1,035/day
- Warm: 335 days × 1.5 PB × $0.0125 = $6,281.25/day
- Cold: 4 years × 547.5 PB/year × $0.0036 = $7,884/day
- **Total: ~$15,200/day vs $46,200/day (all hot storage) = 67% savings**

---

### 6.5 Caching Strategy

**Purpose:** Reduce database load and improve response times

**Cache Layers:**

1. **CDN Layer (CloudFront):**
   - Cache: Photo URLs, thumbnails
   - TTL: 7 days for photos, 30 days for thumbnails
   - Invalidation: On photo deletion/update
   - Hit ratio target: >95%

2. **Application Cache (Redis):**
   - Cache: Metadata, user sessions, share permissions
   - TTL: Variable by data type
   - Eviction: LRU policy
   - Cluster: 50 nodes, 500GB total

3. **Database Cache:**
   - Cassandra row cache: Recent photo metadata
   - PostgreSQL query cache: Album queries

**Caching Patterns:**

**Cache-Aside (Lazy Loading):**

```python
def get_photo_metadata(photo_id):
    # Check cache first
    cached = redis.get(f"photo:{photo_id}")
    if cached:
        return cached
    
    # Cache miss - query database
    metadata = cassandra.query("SELECT * FROM photos WHERE photo_id = ?", photo_id)
    
    # Store in cache
    redis.setex(f"photo:{photo_id}", 86400, metadata)  # TTL: 24h
    
    return metadata
```text

**Write-Through (for critical data):**

```python
def update_photo_metadata(photo_id, metadata):
    # Update database
    cassandra.execute("UPDATE photos SET ... WHERE photo_id = ?", photo_id, metadata)
    
    # Update cache immediately
    redis.setex(f"photo:{photo_id}", 86400, metadata)
```text

**Cache Invalidation:**

- Photo deletion: Invalidate photo metadata, CDN URLs
- Album update: Invalidate album cache, user albums list
- Permission change: Invalidate share permissions cache
- Proactive invalidation on write operations

**What NOT to Cache:**

- Real-time statistics (storage usage)
- Upload in-progress data
- Authentication tokens (stored in secure session store)
- Financial/billing data

---

### 6.6 Key Trade-Offs Analysis

**Trade-Off 1: SQL vs NoSQL for Metadata**

**Decision:** Cassandra (NoSQL) for photo metadata, PostgreSQL (SQL) for user/album data

**Cassandra Pros:**

- Excellent write performance for high-volume uploads
- Partition by user_id for data locality
- Linear scalability
- Multi-datacenter replication built-in

**Cassandra Cons:**

- No joins (acceptable for photo metadata)
- Eventual consistency by default
- Complex aggregations difficult

**PostgreSQL Pros:**

- ACID transactions for user/album management
- Relational integrity for sharing permissions
- Rich query capabilities

**PostgreSQL Cons:**

- Vertical scaling limitations
- Complex sharding required at scale

**Justification:** Photo metadata access is primarily user-scoped timeline queries (perfect for Cassandra partitioning). User/album data requires relationships and ACID guarantees (PostgreSQL strength). Hybrid approach optimizes for both access patterns.

---

**Trade-Off 2: Synchronous vs Asynchronous Processing**

**Decision:** Asynchronous processing via Kafka

**Pros:**

- Non-blocking uploads improve perceived performance
- Better resource utilization (batch processing)
- Fault tolerance (retry failed processing)
- Decoupled services for independent scaling

**Cons:**

- Eventual availability of thumbnails
- Complex error handling
- Increased system complexity

**Justification:** Users prioritize fast upload confirmation over immediate thumbnail availability. Processing 18,000 photos/sec synchronously would require 90,000 workers (5s each) vs 2,000 workers with queuing.

---

**Trade-Off 3: Storage Location - S3 vs Custom Storage**

**Decision:** AWS S3 with CDN

**Pros:**

- Infinite scalability
- Built-in redundancy (11 9s durability)
- Lifecycle management
- CDN integration
- Cost-effective tiering

**Cons:**

- Vendor lock-in
- Egress costs
- Limited customization

**Justification:** Building custom distributed storage would require massive engineering investment. S3's durability and integration ecosystem far outweigh costs for photo storage use case.

---

**Trade-Off 4: Search - Elasticsearch vs Database Search**

**Decision:** Elasticsearch for search

**Pros:**

- Full-text search with relevance scoring
- Geo-spatial queries
- Fast aggregations for faceted search
- Horizontal scalability

**Cons:**

- Additional infrastructure
- Eventual consistency with source of truth
- Operational complexity

**Justification:** Database full-text search (PostgreSQL, Cassandra) can't match Elasticsearch performance for complex queries at scale. Photo search requires multi-field, geo-spatial, and date range queries that Elasticsearch excels at.

---

**Trade-Off 5: Video Processing**

**Decision:** AWS Elastic Transcoder for video processing (post-MVP: AWS MediaConvert)

**Pros:**

- Managed service (no infrastructure)
- Multiple format outputs
- Thumbnail extraction
- Scalable

**Cons:**

- Processing cost ($0.015/minute of video)
- Limited customization
- Vendor lock-in

**Justification:** Video transcoding is complex and resource-intensive. Managed service is cost-effective for MVP. Custom solution requires deep expertise and significant infrastructure investment.

---

## 7. BOTTLENECKS & IMPROVEMENTS

### 7.1 Potential Bottlenecks

**Bottleneck 1: Database Write Contention**

**Problem:**

- Peak upload QPS (18,000/s) generates massive write load to Cassandra
- Hot partitions for popular users with many uploads
- Write latency increases under load

**Solution:**

- Use time-bucketed partitioning: `(user_id, date_bucket)` as composite partition key
- Write with LOCAL_QUORUM consistency (balance consistency/performance)
- Separate clusters for read/write workloads
- Batch writes where possible (reduce overhead)

**Monitoring:**

- Track write latency P99 (alert if > 100ms)
- Monitor partition hotspots
- Track write rejection rate

---

**Bottleneck 2: S3 Request Rate**

**Problem:**

- S3 has request rate limits (3,500 PUT/s per prefix)
- 18,000 uploads/s exceeds single prefix limit
- 693,000 GET/s for views requires careful prefix design

**Solution:**

- Use randomized prefixes: `{hash_prefix}/{user_id}/{date}/{photo_id}`
- Hash prefix provides automatic distribution (2 chars = 256 prefixes)
- CDN for GET requests reduces S3 load by 95%
- Multi-region buckets for geographic distribution

**Monitoring:**

- Track 503 SlowDown errors from S3
- Monitor request distribution across prefixes
- CDN cache hit ratio

---

**Bottleneck 3: Thumbnail Generation Lag**

**Problem:**

- Peak: 18,000 photos/s × 5s processing = 90,000 concurrent operations
- Worker pool of 2,000 creates 45-second lag at peak
- User frustration if thumbnails unavailable immediately

**Solution:**

- Priority queue: separate queues for new users vs existing users
- Lazy thumbnail generation: generate on-demand for immediate view
- Pre-generate only one thumbnail (400px), others on-demand
- GPU-accelerated batch processing (10x faster)

**Monitoring:**

- Track queue depth (alert if > 50,000)
- Monitor processing lag (time from upload to thumbnail ready)
- Track on-demand generation requests

---

**Bottleneck 4: Search Index Lag**

**Problem:**

- Photos not immediately searchable after upload
- Elasticsearch indexing lag during peak traffic
- User searches missing recent uploads

**Solution:**

- Increase Elasticsearch refresh interval during peak (trade-off: latency vs throughput)
- Bulk indexing with batching
- Read-from-write optimization: cache recent uploads in Redis, merge with search results
- Separate index for recent photos (< 1 hour), merge results

**Monitoring:**

- Track indexing lag (time from upload to searchable)
- Monitor refresh rate and bulk queue size
- Alert if lag > 5 minutes

---

**Bottleneck 5: Single Points of Failure**

**Problem:**

- Load balancer failure blocks all traffic
- Kafka cluster failure stops all processing
- Database master failure impacts writes

**Solution:**

- Multi-AZ deployment for all components
- Load balancers: Active-active across AZs
- Kafka: 3-broker cluster with replication factor 3
- Cassandra: Multi-datacenter replication
- PostgreSQL: Primary-replica with automatic failover (RDS Multi-AZ)

**Monitoring:**

- Health checks on all critical components (30s interval)
- Automated failover testing (chaos engineering)
- Alert on single-component dependency

---

### 7.2 Scalability Improvements

**1. Geographic Distribution**

**Strategy:**

- Deploy full stack in 3 regions: US-East, EU-West, Asia-Pacific
- User-to-region routing based on latency (Route53 latency-based routing)
- Cross-region S3 replication for disaster recovery
- Cassandra multi-DC replication with LOCAL_QUORUM reads

**Benefits:**

- Reduce latency for global users (< 100ms)
- Disaster recovery (RPO: 1 minute, RTO: 5 minutes)
- Regulatory compliance (data residency)

**Implementation:**

```text
US-East (Primary):
- Full stack deployment
- Hot storage replication to EU/Asia

EU-West (Secondary):
- Full stack deployment
- Cassandra replica
- Local uploads stored locally, replicated to US

Asia-Pacific (Secondary):
- Full stack deployment
- Cassandra replica
- Local uploads stored locally, replicated to US
```text

---

**2. Advanced Caching**

**Strategy:**

- Edge caching with Lambda@Edge for personalized content
- Predictive cache warming based on user behavior
- Browser cache with service workers for offline access

**Implementation:**

```javascript
// Lambda@Edge for personalized photo feeds
exports.handler = async (event, context) => {
  const user_id = extractUserFromToken(event.headers.authorization);
  const cached_feed = await redis.get(`feed:${user_id}`);
  
  if (cached_feed) {
    return {
      statusCode: 200,
      body: cached_feed,
      headers: {'Cache-Control': 'public, max-age=300'}
    };
  }
  
  // Forward to origin
  return event;
};
```text

---

**3. Real-Time Features**

**Strategy:**

- WebSocket connections for live upload progress
- Server-Sent Events (SSE) for real-time album updates
- Push notifications for sharing events

**Implementation:**

```mermaid
graph LR
    Client[Mobile Client]
    Gateway[API Gateway WebSocket]
    Connection[Connection Manager<br/>Redis]
    Upload[Upload Service]
    
    Client -->|WebSocket Connect| Gateway
    Gateway -->|Register| Connection
    Upload -->|Progress Event| Connection
    Connection -->|Push| Gateway
    Gateway -->|Update| Client
```text

**Benefits:**

- Real-time upload progress (eliminate polling)
- Instant sharing notifications
- Live album collaboration

---

**4. Query Optimization**

**Photo Timeline Query Optimization:**

```sql
-- Before: Scan entire partition
SELECT * FROM photos 
WHERE user_id = ? 
ORDER BY upload_date DESC 
LIMIT 50;

-- After: Time-bucketed partitions
SELECT * FROM photos 
WHERE user_id = ? AND date_bucket = '2025-10'
ORDER BY upload_date DESC 
LIMIT 50;
```text

**Cassandra Materialized Views:**

```sql
-- Favorite photos view
CREATE MATERIALIZED VIEW user_favorites AS
  SELECT * FROM photos
  WHERE user_id IS NOT NULL 
    AND is_favorite = true
    AND photo_id IS NOT NULL
  PRIMARY KEY (user_id, is_favorite, upload_date, photo_id);
```text

---

### 7.3 Monitoring and Observability

**System Metrics:**

```yaml
API Layer:
  - Request rate (QPS)
  - Latency: P50, P95, P99
  - Error rate (4xx, 5xx)
  - Rate limit hits

Upload Service:
  - Upload throughput (MB/s)
  - Upload success rate
  - Chunk failure rate
  - S3 PUT latency

Processing Pipeline:
  - Queue depth (Kafka lag)
  - Processing time per photo
  - Worker utilization
  - Failed jobs count

Database:
  - Cassandra: Read/write latency P99
  - PostgreSQL: Connection pool usage
  - Elasticsearch: Query latency, indexing rate

Storage:
  - S3: Request rate per prefix
  - CDN: Cache hit ratio
  - Storage utilization by tier

Business Metrics:
  - Daily active users
  - Photos uploaded per day
  - Search queries per day
  - Share link creation rate
  - Storage usage growth rate
```text

**Alerting Strategy:**

```yaml
Critical (PagerDuty):
  - API error rate > 5% for 5 minutes
  - Database unavailable
  - Upload service down
  - Processing lag > 10 minutes

Warning (Slack):
  - API P99 latency > 2s for 15 minutes
  - Queue depth > 100,000
  - CDN cache hit ratio < 90%
  - Storage tier transition failures

Info (Dashboard):
  - Daily storage growth
  - Popular search terms
  - User engagement metrics
```text

**Logging:**

- Structured JSON logs (ELK stack)
- Distributed tracing (Jaeger) for request flows
- Correlation IDs across services
- Log retention: 30 days hot, 1 year cold

**Dashboards:**

- Real-time operations dashboard (Grafana)
- Business metrics dashboard
- Cost analysis dashboard
- Per-service health dashboards

---

### 7.4 Security Considerations

**1. Authentication & Authorization**

**Implementation:**

- OAuth 2.0 with JWT tokens (HS256 algorithm)
- Refresh token rotation (30-day expiry)
- Token blacklisting for logout
- Rate limiting per user and IP
- API keys for third-party integrations

**Photo Access Control:**

```python
def authorize_photo_access(user_id, photo_id):
    # Check ownership
    photo = get_photo_metadata(photo_id)
    if photo.user_id == user_id:
        return True
    
    # Check sharing permissions
    share = get_share_permissions(photo_id, user_id)
    if share and share.permission_level in ['view', 'edit']:
        return True
    
    return False
```text

---

**2. Data Protection**

**Encryption:**

- At-rest: S3 server-side encryption (AES-256)
- In-transit: TLS 1.3 for all communications
- Database encryption: Cassandra transparent data encryption

**Privacy:**

- EXIF stripping for shared photos (remove GPS, device info)
- Anonymous sharing links (no user identification)
- Right to deletion (GDPR compliance)
- Data export functionality

**Secure Upload:**

- Pre-signed S3 URLs (client-side upload)
- URL expiration (15 minutes)
- File type validation (magic number check)
- Virus scanning (ClamAV integration)

---

**3. DDoS Protection**

**Strategy:**

- CloudFlare DDoS protection (L3/L4)
- AWS Shield Standard (network/transport layer)
- WAF rules: Rate limiting, IP blocking, geo-blocking
- Challenge-response for suspicious traffic

**Rate Limiting:**

```yaml
Global Limits:
  - 10,000 requests/minute per IP
  - 100 uploads/hour per user
  - 1,000 search queries/hour per user

Endpoint Limits:
  - Login: 5 attempts/minute
  - Registration: 3 accounts/hour per IP
  - Share creation: 20/hour per user
```text

---

**4. Face Recognition Privacy**

**Privacy-First Design:**

- User opt-in required for face recognition
- Easy opt-out with complete data deletion
- Face data isolated per user (no cross-user learning)
- Embeddings stored separately from photos (can be deleted independently)
- No sharing of face data with third parties
- GDPR/CCPA compliant data handling

**Security Measures:**

- Face embeddings encrypted at rest
- Access controls on face database (stricter than photos)
- Audit logging for all face-related operations
- Rate limiting on face search to prevent abuse
- No API access to raw face embeddings (only search/match)

**User Controls:**

```json
// User privacy settings
{
  "face_recognition_enabled": true,
  "auto_face_clustering": true,
  "face_suggestions_enabled": true,
  "allow_face_tagging_by_others": false,
  "hide_my_face_from_suggestions": false
}
```text

**Data Deletion:**

```python
def delete_face_data(user_id):
    """
    Complete deletion of face recognition data
    
    When user opts out:
    1. Delete all face embeddings from vector DB
    2. Delete face records from PostgreSQL
    3. Delete person groups
    4. Keep photos intact (only face data deleted)
    """
    # Delete from Milvus
    milvus.delete(expr=f"user_id == '{user_id}'")
    
    # Delete from PostgreSQL
    db.execute("DELETE FROM faces WHERE user_id = ?", user_id)
    db.execute("DELETE FROM people WHERE user_id = ?", user_id)
    db.execute("DELETE FROM face_clusters WHERE user_id = ?", user_id)
    
    # Update user preferences
    db.execute("UPDATE users SET face_recognition_enabled = FALSE WHERE user_id = ?", user_id)
```text

**5. Secure Sandbox**

**For Future ML Features (object detection, scene recognition):**

- Isolated processing environment (Kubernetes pods)
- No network access during processing
- Input sanitization and validation
- Resource limits (CPU, memory, time)
- Audit logging for all ML operations

---

### 7.5 Future Enhancements

**Phase 2 Features:**

1. **Advanced AI/ML:**
   - Object/scene detection (beyond faces)
   - Smart semantic search ("photos with dogs", "sunset photos", "food photos")
   - Auto-tagging and categorization
   - Duplicate detection (perceptual hashing)
   - Photo quality assessment and auto-enhance suggestions

2. **Photo Editing:**
   - Basic edits (crop, rotate, filters)
   - Non-destructive editing
   - Edit history and undo
   - Batch editing

3. **Advanced Sharing:**
   - Collaborative albums (multiple contributors)
   - Comments on photos
   - Real-time collaboration
   - Album slide shows with music

4. **Mobile Optimizations:**
   - Intelligent sync (WiFi only option)
   - Low-data mode (quality reduction)
   - Offline viewing (downloaded albums)
   - Background upload queue

5. **Smart Organization:**
   - Automatic album creation (events, trips)
   - Memory suggestions ("One year ago today")
   - Best photo suggestions
   - Cleanup suggestions (blurry, duplicates)

6. **Integration:**
   - Third-party app integrations (photo printing)
   - Social media sharing
   - Import from other services
   - API for developers

7. **Analytics:**
   - Storage usage breakdown
   - Photo activity feed
   - Popular albums statistics
   - Sharing analytics

8. **Premium Features:**
   - Increased storage (100GB, 1TB, 2TB tiers)
   - Advanced editing tools
   - Priority support
   - Custom domains for sharing
   - Ad-free experience

---

## CONCLUSION

This Google Photos system design demonstrates a scalable, reliable architecture capable of serving 500M users with billions of photos and face recognition for millions of people. The hybrid database approach (Cassandra for metadata, PostgreSQL for relational data, Milvus for face embeddings) optimizes for different access patterns. Asynchronous processing via Kafka enables high upload throughput while maintaining system responsiveness.

Key design decisions prioritize:

- **Scalability:** Horizontal scaling across all layers, including ML pipelines
- **Performance:** Multi-tier caching, CDN, optimized queries, GPU-accelerated face processing
- **Reliability:** Multi-region deployment, redundancy, fault tolerance
- **Cost Efficiency:** Storage tiering saves 67% on storage costs
- **User Experience:** Fast uploads, sub-second photo loading, powerful search, intelligent face grouping
- **Privacy:** User control over face data, GDPR/CCPA compliance, opt-in/opt-out capabilities

**Face Recognition Capabilities:**

- Automatic face detection and clustering
- 99.6% accuracy using FaceNet embeddings
- Sub-second face search across millions of faces
- User-friendly person management and tagging
- Privacy-first design with complete data deletion option

The architecture is production-ready for MVP with face recognition as a core differentiator, while providing clear paths for advanced features (object detection, semantic search, real-time collaboration, advanced editing) in future phases.

**Resource Summary:**

- 1,400 API servers for 700K QPS
- 2,000 image processing workers
- 324 GPU workers for face recognition
- 12 EB storage over 5 years
- ~35 TB/s peak bandwidth
- 100-shard Cassandra cluster
- Milvus vector DB with billions of face embeddings

---

**Document Version:** 1.0  
**Last Updated:** October 1, 2025  
**Author:** System Design Interview Preparation
