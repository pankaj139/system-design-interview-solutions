# Photo Storage and Management System Design (Google Photos-like)

**File Purpose:** Interactive, multi-level learning resource for designing a cloud-based photo storage and management platform like Google Photos. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that supports 1 billion users with 4 trillion photos, handles 1.5 billion uploads daily, processes images with ML-powered face recognition and auto-tagging, delivers photos globally with <100ms latency, and achieves 99.99% availability (52 minutes downtime/year).

**Author:** System Design Documentation  
**Created:** October 1, 2025  
**Last Updated:** January 2026  
**Recent Updates:** Transformed into multi-level instructional format with learning objectives, real-world examples from Google Photos/iCloud/Amazon Photos, interview preparation, and practice exercises for educational platform

---

## 🎓 Welcome to Google Photos System Design!

### What You're Going to Build

Imagine creating a platform where over a billion people store their most precious memories - wedding photos, baby's first steps, family vacations - totaling 4 trillion photos. Every day, 1.5 billion new photos are uploaded from smartphones around the world. Your system must automatically recognize faces of loved ones across thousands of photos, organize images into smart albums, let users search for "beach sunset" and instantly find relevant photos from years ago, and deliver any photo to any device globally in under 100 milliseconds. 

By the end of this learning journey, you'll understand how to design a production-grade photo storage and management platform that:
- Stores and manages 4 trillion photos for 1 billion users (4 exabytes of data)
- Handles 1.5 billion photo uploads per day (17,361 uploads/second average, 52K QPS peak)
- Processes images with ML-powered face recognition across billions of faces
- Delivers photos globally with <100ms latency through intelligent CDN placement
- Achieves 99.99% availability (only 52 minutes of downtime per year)

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (6-8 hours)
├─ Learn fundamental concepts of photo storage
├─ Understand WHY we make design choices
├─ Build intuition with everyday analogies (photo albums, libraries)
└─ Perfect for: New to system design or photo/media systems

🟡 INTERMEDIATE LEVEL (8-10 hours)  
├─ Master interview techniques for media platforms
├─ Learn trade-off analysis for storage vs processing
├─ Practice common interview questions
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-14 hours)
├─ Production considerations for ML pipelines
├─ Performance optimization techniques at scale
├─ Handle edge cases and ML model failures
└─ Perfect for: Senior engineers and ML system architects
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of how photos are stored on your phone
- Familiarity with uploading files to websites
- No prior system design experience needed!

**For Intermediate:**
- Understanding of databases (SQL and NoSQL concepts)
- Basic knowledge of REST APIs
- Familiarity with object storage (like AWS S3)
- Basic understanding of machine learning concepts

**For Advanced:**
- Experience with distributed systems and CAP theorem
- Knowledge of image processing pipelines
- Understanding of ML model serving and inference
- Familiarity with vector databases and similarity search
- Understanding of consistency models and data replication

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:
1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context from Google Photos, iCloud, Amazon Photos
3. **Multi-Level Content** - Tailored explanations for your level (🟢🟡🔴)
4. **Real-World Examples** - How Google Photos, Apple iCloud, and Amazon Photos actually do it
5. **Think About It** - Questions to deepen understanding
6. **Key Takeaways** - Summary of main points
7. **Practice Exercise** - Hands-on challenge

💡 **Pro Tip:** Don't skip the "Think About It" sections - they're designed to help you internalize concepts so you can explain them in interviews or to your team!

---

## TABLE OF CONTENTS

- [Section 1: Understanding What We're Building](#section-1-understanding-what-were-building)
- [Section 2: Planning for Scale](#section-2-planning-for-scale)
- [Section 3: Designing the System Architecture](#section-3-designing-the-system-architecture)
- [Section 4: Storing Our Data](#section-4-storing-our-data)
- [Section 5: How Users Interact - API Design](#section-5-how-users-interact---api-design)
- [Section 6: Upload Pipeline - Getting Photos into the System](#section-6-upload-pipeline---getting-photos-into-the-system)
- [Section 7: Image Processing & ML Pipeline](#section-7-image-processing--ml-pipeline)
- [Section 8: Face Recognition at Scale](#section-8-face-recognition-at-scale)
- [Section 9: Smart Search & Discovery](#section-9-smart-search--discovery)
- [Section 10: Storage Architecture & CDN](#section-10-storage-architecture--cdn)
- [Section 11: Sharing & Collaboration](#section-11-sharing--collaboration)
- [Section 12: Growing the System - Scalability](#section-12-growing-the-system---scalability)
- [Section 13: Protecting the System - Security](#section-13-protecting-the-system---security)
- [Section 14: Keeping It Healthy - Monitoring](#section-14-keeping-it-healthy---monitoring)
- [Section 15: Making Design Decisions - Trade-offs](#section-15-making-design-decisions---trade-offs)
- [Section 16: Interview Preparation & Practice](#section-16-interview-preparation--practice)
- [Putting It All Together](#putting-it-all-together)
- [Next Steps](#next-steps)

---

## Section 1: Understanding What We're Building

### What You'll Learn

By the end of this section, you'll be able to:
- Explain what a photo storage platform like Google Photos does and why it exists
- Define functional requirements (features the system provides)
- Identify non-functional requirements (performance, scalability, reliability targets)
- Ask the right clarifying questions in a system design interview for media platforms

### Why This Matters

Before writing a single line of code or drawing diagrams, you need to understand WHAT you're building and WHY. This is where 60% of system design interviews are won or lost. Real-world example: Google Photos started as Google+ Photos in 2011 but failed because they didn't understand user requirements - people wanted unlimited free storage for memories, not a social network. When relaunched in 2015 with proper requirements (unlimited storage, auto-backup, smart search), it reached 1 billion users by 2018!

---

### 🟢 For Beginners: The Fundamentals

#### What is a Photo Storage Platform?

Think of Google Photos like a magical photo album that:
- Lives in the cloud (not on your phone)
- Never runs out of space
- Automatically organizes your photos
- Lets you find any photo instantly
- Works on all your devices

**Why not just keep photos on your phone?**

Let's compare:

```text
📱 Phone Storage (Traditional):
├─ Limited space (64GB phone = ~20,000 photos)
├─ Lose phone = lose all photos ❌
├─ Hard to share with family
└─ No automatic organization

☁️ Cloud Photo Platform (Google Photos):
├─ Unlimited space ✅
├─ Phone stolen? Photos still safe ✅
├─ Share albums instantly ✅
└─ Searches "dog" → finds all dog photos ✅
```

#### Why Do People Need Google Photos?

Real-world problems it solves:

1. **Storage Problem**: Your wedding photos (5,000 images) + vacation videos fill up your phone instantly

2. **Loss Problem**: Phone falls in pool → 10 years of memories gone forever

3. **Organization Problem**: You have 50,000 photos. Good luck finding that birthday photo from 2 years ago!

4. **Sharing Problem**: Want to share 200 wedding photos with family? Email has 25MB limit, texting takes forever

5. **Access Problem**: Photos on iPhone, but you're on Android tablet. Can't access them!

#### What Features Should It Have?

Let's think about what users need:

**Core Features (MVP - Must Have):**

1. **Upload Photos**
   - From phone camera automatically
   - From computer manually
   - Support all formats (JPEG, PNG, RAW, HEIC)

2. **Store Forever**
   - Never delete unless user wants to
   - Keep original quality available
   - Multiple backups (don't lose data!)

3. **View Anywhere**
   - Phone app (iOS, Android)
   - Web browser
   - Tablet app
   - Fast loading (< 1 second)

4. **Basic Organization**
   - Timeline view (sorted by date)
   - Create albums manually
   - See location on map

5. **Simple Sharing**
   - Share single photos via link
   - Share whole albums
   - Control who can see what

**Smart Features (Nice to Have):**

6. **Search**
   - Find photos by typing "beach" or "dog"
   - Search by date or location
   - Search by person's face

7. **Automatic Face Recognition**
   - Finds all photos of same person
   - Groups family members automatically
   - Suggests tagging names

8. **Smart Albums**
   - Auto-creates "Best of 2025"
   - Finds all photos from "Paris trip"
   - Makes collages and animations

💡 **Pro Tip:** In interviews, always separate "must-have" (MVP) from "nice-to-have" features. This shows you can prioritize! For Google Photos, search and face recognition are differentiators - they're what made it successful.

---

### 🟡 For Intermediate: Interview Patterns

#### Functional vs Non-Functional Requirements Framework

When you're in a system design interview for a photo platform, the interviewer is testing whether you understand the unique challenges of media systems. Here's the framework:

**Functional Requirements** (What the system DOES):

| Requirement | Description | Interview Tip |
|------------|-------------|---------------|
| Photo Upload | Users can upload photos/videos | Ask: Max file size? Formats supported? Chunked upload? |
| Storage | Store photos with redundancy | Ask: Retention policy? Original + compressed? |
| Viewing | Display photos in various sizes | Ask: Thumbnail sizes? Progressive loading? |
| Organization | Albums, timeline, folders | Ask: Max photos per album? Nesting depth? |
| Search | Find photos by content/metadata | Ask: Text search? Visual search? Latency SLA? |
| Sharing | Share photos with others | Ask: Permission levels? Expiration? Public links? |
| Face Recognition | Detect and group faces | Ask: Real-time or async? Privacy compliance? |

**Non-Functional Requirements** (How WELL it does it):

| Requirement | Target | Why It Matters | Google Photos Reality |
|------------|--------|----------------|----------------------|
| Availability | 99.99% | 52 min downtime/year | People access memories 24/7 |
| Upload Latency | < 5s for photos | User experience | Google: 2-3s average |
| View Latency | < 100ms | Feels instant | Google: 50-80ms globally |
| Storage Scale | Exabytes (1000s of PB) | 1B users × 4,000 photos each | Google: 4+ EB total |
| Throughput | 17K uploads/sec peak | Handle global traffic | Google: Handles 1.5B daily |
| Consistency | Eventually consistent OK | Rare conflicts acceptable | Except permissions: strong |

#### Critical Clarifying Questions

**Scale Questions:**
```text
Q: "How many daily active users?"
Why: Determines infrastructure size
Answer: 500M DAU (Google Photos actual: ~1B users)

Q: "How many photos per user?"
Why: Determines storage architecture
Answer: Average 4,000 photos/user (some have 50K+)

Q: "Upload vs view ratio?"
Why: Read-heavy systems design differently
Answer: 1:100 (typical photo platform - mostly viewing)

Q: "Peak traffic patterns?"
Why: Need to handle spikes (holidays, events)
Answer: 3x normal during Christmas, New Year
```

**Feature Scope Questions:**
```text
Q: "Do we need face recognition?"
Why: Massively increases complexity (ML pipeline)
Answer: Yes for Google Photos (differentiator)
Tip: Can be "Phase 2" for MVP

Q: "Video support?"
Why: Videos are 30x larger, need transcoding
Answer: Yes but different pipeline
Tip: "Let's focus on photos first"

Q: "Photo editing features?"
Why: Adds entire editing infrastructure
Answer: Usually out of scope for interview
Tip: "Users can edit before upload"

Q: "Live Photos (iPhone burst mode)?"
Why: Special handling for photo sequences
Answer: Usually "future enhancement"
```

**Technical Deep-Dive Questions:**
```text
Q: "What's the upload success rate SLA?"
Why: Determines retry logic complexity
Answer: 99.9% success (1 in 1000 can fail)

Q: "How do we handle duplicates?"
Why: Same photo uploaded multiple times
Answer: Perceptual hashing + deduplication

Q: "Consistency model for albums?"
Why: Multiple users editing same album
Answer: Eventually consistent (UI optimistic updates)

Q: "Data retention policy?"
Why: Legal compliance, storage costs
Answer: Forever (unless user deletes)
```

#### Interview Script

Here's how to structure this in a 45-minute interview:

**Minutes 0-10: Requirements Gathering**
```text
You: "Let me clarify the requirements. We're building a photo storage platform 
      like Google Photos. Let me confirm the key features:"

1. Upload photos from mobile/web
2. Store photos reliably with redundancy
3. View photos quickly across devices
4. Organize in albums and timeline
5. Share photos with others
6. Search by content and metadata

Interviewer: "Yes, and add face recognition."

You: "Great. For scale, should I assume:"
- 500M daily active users
- 100M photo uploads per day
- 20B photo views per day
- Global distribution (US, Europe, Asia)
- 99.99% availability target

Interviewer: "Yes, sounds reasonable."

You: "For scope, let's focus on photos first. Videos can follow similar
      architecture with added transcoding. Face recognition we can discuss
      as a separate ML pipeline. Should I proceed with high-level design?"

Interviewer: "Yes, please."
```

**Minutes 10-15: Capacity Estimates**
```text
You: "Let me do quick back-of-envelope calculations:

Storage:
- 100M uploads/day × 3MB avg = 300TB/day = 110PB/year
- With thumbnails + metadata: ~120PB/year
- 5-year total: 600PB

Bandwidth:
- Uploads: 100M × 3MB / 86,400s = ~3.5GB/s
- Views: 20B × 50KB (thumbnails) / 86,400s = ~12TB/s peak
- Need strong CDN presence globally

API Throughput:
- Upload API: ~1,200 QPS average, ~3,600 QPS peak
- View API: ~230K QPS average, ~700K QPS peak

This drives our architecture decisions. Should I proceed to design?"

Interviewer: "Yes."
```

**Minutes 15-45: Continue with architecture, databases, etc.**

💡 **Pro Tip:** Notice how the interview script shows you understand the problem before jumping to solutions. Many candidates fail by drawing boxes without clarifying requirements!

---

### 🔴 For Advanced: Production Considerations

#### Enterprise Requirements

When building Google Photos at production scale, additional requirements emerge:

**1. Compliance & Privacy**

```text
GDPR (Europe):
├─ Right to download all data (data export API)
├─ Right to delete (cascading deletion across all systems)
├─ Data residency (EU data stays in EU)
└─ Consent management (face recognition opt-in)

COPPA (Children's Privacy):
├─ No face recognition for users under 13
├─ Parental consent for accounts
└─ Restricted sharing features

CCPA (California):
├─ Do not sell data option
├─ Transparency in data usage
└─ Right to opt-out of ML training
```

**2. Business Requirements**

```text
Storage Tiers for Monetization:
├─ Free: 15GB (compressed quality)
├─ Paid ($1.99/mo): 100GB original quality
└─ Premium ($9.99/mo): 2TB + advanced features

Feature Gating:
├─ Free: Basic face grouping (1K faces)
├─ Paid: Advanced ML features (unlimited faces)
└─ Premium: Magic Eraser, HDR effects
```

**3. Cost Optimization (Critical at Scale)**

Google Photos specific costs:
- Storage: $0.023/GB/month (S3) × 4 EB = $94M/month just for storage!
- Bandwidth: 12 TB/s × $0.08/GB = $83M/month
- ML Inference: Face recognition on 1.5B photos/day = $30M/month
- Total: $200M+/month operating costs

Cost optimization strategies:
1. **Compression**: Store "High Quality" (compressed) by default
   - Reduces storage by 75%
   - Savings: $70M/month
   
2. **Tiered Storage**: Move old photos to cold storage (Glacier)
   - 80% of views are recent photos (last 30 days)
   - Cold storage: $0.004/GB vs $0.023/GB (82% cheaper)
   - Savings: $40M/month
   
3. **Smart Caching**: Cache popular photos at edge
   - 1% of photos = 80% of views (power law distribution)
   - CDN cache hit ratio: 95%
   - Bandwidth savings: $75M/month

**4. Multi-Tenancy & Isolation**

```text
Tenant Isolation for Enterprise Customers:
├─ Separate encryption keys per customer
├─ Dedicated storage buckets (compliance)
├─ Isolated metadata databases (performance)
└─ Separate ML model serving (IP protection)

Challenge: Google Workspace customers need:
- Corporate compliance (retain all edits for 7 years)
- eDiscovery support (legal holds)
- Admin controls (disable face recognition)
- Audit logs (who viewed what, when)
```

**5. Disaster Recovery & Business Continuity**

```text
RPO (Recovery Point Objective): 0 data loss
RTO (Recovery Time Objective): < 1 hour

Multi-Region Replication:
Primary:     US-EAST-1 (Virginia)
Replica 1:   US-WEST-2 (Oregon)
Replica 2:   EU-WEST-1 (Ireland)
Replica 3:   ASIA-SOUTHEAST-1 (Singapore)

Failure Scenarios:
1. Single server failure: Auto-healing (30s)
2. AZ failure: Traffic shifted (2 min)
3. Region failure: DR promotion (15 min)
4. Global disaster: Restore from tape (4 hours)
```

#### Advanced System Requirements

**1. Consistency Guarantees**

Different operations need different consistency:

| Operation | Consistency Model | Rationale |
|-----------|------------------|-----------|
| Upload photo | Eventual (10s) | Can show "Processing..." |
| Delete photo | Strong (immediate) | Must not show deleted photo |
| Album sharing | Eventual (1-2s) | Optimistic UI acceptable |
| Permissions | Linearizable | Security critical |
| Face clustering | Eventual (hours/days) | Batch processing acceptable |

**2. Performance SLAs by Percentile**

```text
Photo Load Time SLA:
├─ p50 (median): 50ms
├─ p95: 100ms
├─ p99: 200ms
└─ p99.9: 500ms (acceptable for slow networks)

Why percentiles matter:
- Average can hide problems (outliers)
- p99 affects real users (1% of 1B users = 10M people!)
- Google Photos targets: "Instantly feel responsive"
```

**3. ML Model Performance Requirements**

```text
Face Recognition Accuracy:
├─ Precision: > 99.5% (minimize false positives)
├─ Recall: > 95% (find most faces)
├─ Inference latency: < 100ms per photo
└─ Model size: < 50MB (mobile deployment)

Object Detection:
├─ 20,000 object classes (dog, car, sunset, beach...)
├─ Multi-label (one photo: "dog" + "beach" + "sunset")
├─ Inference: < 200ms per photo
└─ Accuracy: > 90% top-5
```

### Real-World Example: How Google Photos Does It

#### Google Photos Evolution

**2015 Launch:**
- Key decision: **Unlimited free storage** (compressed quality)
- Why: Competitive advantage over Apple iCloud ($0.99/month for 50GB)
- Result: Grew from 0 to 100M users in 5 months

**2016 Scaling Challenge:**
- Problem: 200M users, 24B photos, running out of money
- Solution: ML compression (further reduce storage), tiered storage
- Result: Cut costs by 60% while improving quality

**2021 Policy Change:**
- Ended unlimited free storage (now 15GB free)
- Why: Reached 4 trillion photos, unsustainable costs
- Business model: Drive users to Google One ($1.99/mo for 100GB)

#### Technical Decisions

```text
Storage: Google Colossus (successor to GFS)
├─ Reed-Solomon erasure coding (vs 3x replication)
├─ Saves 50% storage (14 data + 4 parity vs 3 copies)
└─ Rebuild from any 14 of 18 shards

CDN: Google Global Cache (in ISP networks)
├─ Cache nodes in 7,500+ ISP locations
├─ 95% of views served from ISP cache (< 10ms)
└─ Only 5% hit origin storage

Face Recognition: FaceNet (2015-2018) → FaceNet-V2 (2019+)
├─ 128-dimensional embeddings
├─ 99.63% accuracy on LFW benchmark
└─ Privacy: On-device clustering (iOS/Android), server for web
```

### 🎯 Interview Questions: Requirements Gathering

**Q1: Why does Google Photos offer unlimited free storage?**

<details>
<summary>Click to see answer</summary>

**Answer:**
Three strategic reasons:

1. **Competitive Advantage (2015)**: Apple charged $0.99/month, Google went free to rapidly gain users

2. **Data for ML Training**: More photos = better ML models for object detection, face recognition

3. **Ecosystem Lock-in**: Once you have 50,000 photos in Google Photos, you won't switch to Apple

**Trade-off**: Lost $200M+/year in 2015-2020, but gained 1B users. Later changed to 15GB free (2021) after achieving dominance.

**Interview Tip**: Shows you understand business strategy, not just technical design.
</details>

**Q2: Why does Google Photos compress photos by default instead of storing originals?**

<details>
<summary>Click to see answer</summary>

**Answer:**
Cost vs quality trade-off:

```text
Original Quality (JPEG from iPhone 14):
- 12 MP = 4000×3000 pixels
- File size: 3-5 MB
- Storage cost: 3 MB × 1 trillion photos = 3 EB
- Monthly cost: 3 EB × $0.023/GB = $70M/month

"High Quality" (Google's compression):
- Resized to 16 MP max (same as original for most phones)
- Aggressive JPEG compression (quality 85)
- File size: 0.8-1.2 MB (70% smaller)
- Monthly cost: $21M/month
- Savings: $49M/month = $588M/year!
```

**Visual quality**: 99% of users can't tell difference on phone screens

**Interview Tip**: At scale, storage costs dominate. Always consider compression.
</details>

**Q3: Why is face recognition the most complex feature to build?**

<details>
<summary>Click to see answer</summary>

**Answer:**
Multiple technical challenges:

**1. ML Pipeline Complexity:**
```text
Regular photo upload:
Photo → Storage → Done (simple)

Face recognition pipeline:
Photo → Face Detection → Embedding Extraction → Clustering → 
Verification → Update Index → Notify User
(6 steps, any can fail)
```

**2. Privacy Regulations:**
- GDPR: Must get consent before scanning faces
- CCPA: Users can opt-out
- BIPA (Illinois): Biometric data special protection
- Results: Different pipeline per region!

**3. Accuracy Requirements:**
- False positive (wrong person): Very bad UX ("That's not grandma!")
- False negative (missed face): Acceptable
- Need precision > 99.5%, recall > 95%

**4. Performance:**
- 1.5B photos uploaded daily
- ~3 faces per photo average
- = 4.5B face inferences/day
- = 52K inferences/second
- Needs massive GPU cluster

**5. Scale:**
- 1 billion users × 10 people per user = 10B face clusters
- Vector database with 10B entries
- Each query: Compare against 10B faces
- Need approximate nearest neighbor (ANN), not exact search

**Interview Tip**: Face recognition alone could be entire interview. Be ready to design the ML pipeline if asked!
</details>

**Q4: What's the difference between functional and non-functional requirements? Give examples for Google Photos.**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Functional Requirements** = WHAT the system does (features)
```text
Examples for Google Photos:
- Upload photos from mobile app
- Create albums and organize photos
- Share photos via link
- Search photos by content
- Recognize faces and group people
```

**Non-Functional Requirements** = HOW WELL it does it (quality attributes)
```text
Examples for Google Photos:
- Upload < 5 seconds (Performance)
- 99.99% uptime (Reliability)
- Handle 1B users (Scalability)
- Encrypt data at rest (Security)
- < 100ms photo load (Latency)
```

**Why Both Matter:**
- Functional only: System works but is slow/unreliable
- Non-functional only: Fast system that does nothing useful
- Need both: Useful system that performs well

**Interview Tip:** Always clarify both types early in interview. Shows structured thinking.
</details>

---

### 🤔 Think About It

Before moving to the next section, consider these questions:

1. **Storage Trade-off**: Would you offer unlimited free storage if you were designing a new photo platform in 2026? Why or why not? (Hint: Consider costs, competition, and business model)

2. **Privacy vs Features**: Face recognition is Google Photos' killer feature but raises privacy concerns. How would you balance this? Would you make it opt-in or opt-out?

3. **Mobile vs Web**: Should you optimize for mobile-first (where most photos are taken) or treat all platforms equally? What would change in your design?

4. **Freemium Model**: If you had to monetize, what features would you put behind a paywall? What must stay free to remain competitive?

---

### ✅ Key Takeaways

```text
✓ Photo platforms are read-heavy (1:100 write-read ratio)
✓ Storage costs dominate at scale ($70M/month for 3 EB)
✓ Compression saves 70% storage with minimal quality loss
✓ Face recognition is complex: ML pipeline + privacy + scale
✓ Functional requirements = features, Non-functional = quality
✓ Always clarify scale before designing (100K vs 1B users = different architecture)
✓ Google Photos' success: Unlimited free storage + smart search + face recognition
✓ Business drives technical: Free storage was strategy, not generosity
```

**Critical Numbers to Remember:**
- 1B users, 4 trillion photos, 1.5B uploads/day
- 99.99% availability = 52 minutes downtime/year
- < 100ms photo load time globally
- 1:100 write-to-read ratio (typical for media platforms)

---

### 🎯 Practice Exercise

**Exercise: Requirements Analysis**

You're in an interview. The interviewer says: *"Design Instagram."*

Using what you learned, write down:
1. 5 functional requirements (MVP features)
2. 5 non-functional requirements (with specific numbers)
3. 3 clarifying questions you'd ask
4. 1 feature you'd explicitly exclude from MVP

<details>
<summary>Click to see sample answer</summary>

**Functional Requirements (MVP):**
1. Users can upload photos (JPEG/PNG, max 10MB)
2. Users can follow other users
3. Users can view a feed of photos from people they follow
4. Users can like and comment on photos
5. Users can search for users by username

**Non-Functional Requirements:**
1. Availability: 99.9% uptime (8.76 hours downtime/year)
2. Upload latency: < 3 seconds for photos
3. Feed load time: < 500ms
4. Scale: 500M daily active users
5. Throughput: 100K photo uploads/second at peak

**Clarifying Questions:**
1. "Do we need video support, or just photos for MVP?" (Videos add transcoding complexity)
2. "What's the maximum number of followers per user?" (Affects feed generation strategy)
3. "Do we need real-time notifications for likes/comments?" (Adds pub-sub infrastructure)

**Explicitly Out of Scope for MVP:**
- Stories/Reels (temporary content)
- Direct messaging
- Shopping/commerce features
- Advanced filters/editing

**Why this answer works:**
- Shows prioritization (MVP vs future)
- Asks scope questions (avoid gold-plating)
- Specifies numbers (not just "fast" or "scalable")
- Understands business (social network vs photo storage)
</details>

---

**Ready to move on?** In the next section, we'll do back-of-the-envelope calculations to determine the scale of infrastructure we need. You'll learn how to estimate storage, bandwidth, and compute requirements - essential skills for any system design interview!

---

## Section 2: Planning for Scale

### What You'll Learn

By the end of this section, you'll be able to:
- Calculate storage requirements for billions of photos
- Estimate bandwidth needs for global photo delivery
- Determine server and database capacity requirements
- Translate business metrics into infrastructure sizing

### Why This Matters

"Back-of-the-envelope" calculations are THE most important skill in system design interviews. They help you avoid over-engineering (wasting money) or under-engineering (system crashes). Real-world example: In 2015, Google Photos launched with "unlimited free storage" without proper capacity planning. By 2020, they had 4 trillion photos costing $200M+/month in storage alone - forcing them to end the free unlimited policy in 2021. Proper planning prevents expensive surprises!

---

### 🟢 For Beginners: The Fundamentals

#### Why Do We Need to Calculate Scale?

Think of it like planning a wedding:

```text
🎊 Without Planning:
You: "Everyone's invited!"
Result: 500 people show up, you ordered food for 50
Disaster: Not enough food, chairs, space!

📊 With Planning:
You: "Let's estimate: 200 invited, 70% attend = 140 people"
Result: Order for 150 (buffer), rent right space, enough food
Success: Happy wedding!
```

Same for systems:
- Estimate users → size your databases
- Estimate traffic → size your servers
- Estimate storage → buy enough disks

#### What Numbers Do We Start With?

For Google Photos, we need to know:

1. **How many users?**
   - Total users: 1 billion
   - Daily Active Users (DAU): 500 million
   - Why DAU matters: They're using your system TODAY

2. **How many photos per person?**
   - Average user: 4,000 photos stored
   - Power users: 50,000+ photos
   - New photos per day: 5 photos per active user

3. **How big are photos?**
   - Phone photo: 3 MB average
   - Professional camera (RAW): 25 MB
   - Screenshot: 500 KB
   - We'll use: 3 MB average (most photos from phones)

4. **Upload vs view?**
   - Read-heavy system: For every 1 photo uploaded, 100 are viewed
   - Why: People upload daily, but browse all the time

💡 **Pro Tip:** In interviews, if you don't know a number, make a reasonable assumption and state it clearly: *"I'll assume average photo size is 3MB based on typical smartphone cameras. Does that sound reasonable?"*

#### Let's Calculate Storage (Simple Version)

**Step 1: Daily uploads**
```text
500M active users × 20% upload daily = 100M uploading users
100M users × 5 photos each = 500M photos/day
```

**Step 2: Daily storage needed**
```text
500M photos × 3 MB per photo = 1,500,000,000 MB
= 1,500,000 GB
= 1,500 TB
= 1.5 PB (petabytes) per day!
```

**What's a petabyte?**
```text
1 PB = 1,000 TB = 1,000,000 GB

To visualize:
- Your laptop: 500 GB
- 1 PB = 2,000 laptops worth of storage
- 1.5 PB/day = 3,000 laptops EVERY DAY!
```

**Step 3: Yearly storage**
```text
1.5 PB/day × 365 days = 547.5 PB/year

Over 5 years: 547.5 × 5 = 2,737 PB ≈ 2.7 EB (exabytes)!
```

**Step 4: But wait - thumbnails!**

We need multiple sizes for fast loading:
- Tiny (150×150 pixels): 10 KB (for grid view)
- Medium (400×400 pixels): 50 KB (for preview)
- Large (1080p): 200 KB (for phone display)

```text
Thumbnails per photo: 10 KB + 50 KB + 200 KB = 260 KB
Daily thumbnails: 500M photos × 260 KB = 130 TB/day
Yearly thumbnails: 130 TB × 365 = 47 PB/year
```

**Total Storage (5 years):**
```text
Original photos: 2,737 PB
Thumbnails: 235 PB
Total: ~3,000 PB = 3 EB!
```

#### Let's Calculate Bandwidth

**What's bandwidth?** How fast data moves through the internet pipe.

Think of it like water:
- Storage = water tank (how much you can hold)
- Bandwidth = pipe size (how fast water flows)

**Upload bandwidth:**
```text
500M photos/day ÷ 86,400 seconds/day = 5,787 photos/second
5,787 photos/s × 3 MB = 17,361 MB/s ≈ 17 GB/s average

Peak times (holidays, weekends): 3× average = 51 GB/s
```

**Download bandwidth (people viewing photos):**
```text
Read-heavy: 100 views for every 1 upload
20 billion views/day ÷ 86,400 seconds = 231,481 views/second

Mostly thumbnails (50 KB): 231,481 × 50 KB = 11.6 GB/s
Some full photos (10%): 23,148 × 3 MB = 69 GB/s
Total download: ~11.6 GB/s (thumbnails dominate)

Peak times: 3× = 35 GB/s download bandwidth
```

#### How Many Servers Do We Need?

**Web servers (handle API requests):**
```text
Peak traffic: ~700,000 requests/second (uploads + views)
One server handles: ~1,000 requests/second
Servers needed: 700,000 ÷ 1,000 = 700 servers
With redundancy (backup): 700 × 2 = 1,400 servers
```

**Storage servers:**
```text
Hard drive size: 10 TB each (modern HDD)
Total storage needed: 3,000 PB = 3,000,000 TB
Drives needed: 3,000,000 ÷ 10 = 300,000 drives
With redundancy (3 copies): 300,000 × 3 = 900,000 drives!
```

💡 **Pro Tip:** These big numbers show why Google uses custom data centers. At this scale, you can't just "use AWS S3" - too expensive!

---

### 🟡 For Intermediate: Interview Patterns

#### Capacity Planning Framework

When doing calculations in an interview, follow this structure:

**1. State Your Assumptions**
```text
Interviewer: "How much storage for Google Photos?"

You: "Let me state my assumptions:
- 500M daily active users
- 20% upload daily = 100M uploading users  
- 5 photos per uploading user = 500M photos/day
- Average photo size: 3 MB
- Planning horizon: 5 years
Does this sound reasonable?"

Interviewer: "Yes, proceed."
```

**Why this works:** Shows structured thinking, gives interviewer chance to correct you early.

**2. Calculate Step-by-Step**

```text
Storage Calculation:

Daily Photos:
500M DAU × 20% upload rate × 5 photos/user
= 100M users × 5 photos = 500M photos/day

Daily Storage:
500M photos × 3 MB/photo = 1,500 TB/day = 1.5 PB/day

Annual Storage:
1.5 PB/day × 365 days = 547.5 PB/year

5-Year Total:
547.5 PB/year × 5 years = 2,737.5 PB ≈ 2.7 EB
```

**3. Add Thumbnails and Metadata**

```text
Thumbnails (3 sizes):
- 150px: 10 KB
- 400px: 50 KB  
- 1080px: 200 KB
Total per photo: 260 KB

Daily thumbnails: 500M × 260 KB = 130 TB/day
Annual thumbnails: 47.45 PB/year
5-year thumbnails: 237 PB

Metadata (EXIF, location, tags):
- 2 KB per photo
- 500M photos/day × 2 KB = 1 TB/day = 365 TB/year
- 5-year metadata: 1.825 PB (negligible)

Total 5-Year Storage:
Photos:      2,737 PB
Thumbnails:    237 PB
Metadata:        2 PB
TOTAL:       2,976 PB ≈ 3 EB
```

**4. Calculate Bandwidth Requirements**

```text
Upload Bandwidth:

Average QPS:
500M photos/day ÷ 86,400 sec/day = 5,787 photos/sec

Average bandwidth:
5,787 photos/sec × 3 MB/photo = 17.4 GB/sec

Peak (3x average):
17.4 GB/s × 3 = 52 GB/s upload bandwidth


Download Bandwidth:

Views per day: 100× uploads = 50B views/day
View QPS: 50B ÷ 86,400 = 578,703 views/sec

Thumbnail bandwidth (90% of views):
520,833 views/sec × 50 KB = 26 GB/s

Full photo bandwidth (10% of views):
57,870 views/sec × 3 MB = 174 GB/s

Total download: 200 GB/s average, 600 GB/s peak
```

**5. Cost Estimation (Advanced)**

```text
Storage Costs (AWS S3 pricing):
- S3 Standard: $0.023/GB/month
- 3 EB = 3,000,000,000 GB
- Monthly cost: 3B GB × $0.023 = $69M/month
- Annual cost: $828M/year (just storage!)

Bandwidth Costs:
- Data transfer: $0.08/GB out
- 600 GB/s peak × 86,400 sec/day = 51,840 TB/day
- Daily cost: 51,840,000 GB × $0.08 = $4.1M/day
- Annual cost: $1.5B/year (just bandwidth!)

Total Infrastructure: $2.3B/year

Why Google uses its own data centers:
- Build your own: ~$100M initial investment
- Operating costs: ~$500M/year
- Saves: $1.8B/year!
```

#### QPS (Queries Per Second) Breakdown

| Operation | Daily Count | Average QPS | Peak QPS (3x) | Latency SLA |
|-----------|------------|-------------|---------------|-------------|
| Photo Upload | 500M | 5,787 | 17,361 | < 5s |
| Thumbnail View | 45B | 520,833 | 1,562,500 | < 100ms |
| Full Photo View | 5B | 57,870 | 173,611 | < 500ms |
| Search Query | 1B | 11,574 | 34,722 | < 300ms |
| Album Create | 10M | 116 | 347 | < 1s |
| Share Create | 50M | 579 | 1,736 | < 2s |
| Face Recognition | 500M | 5,787 | N/A (async) | < 1 hour |

**Total Peak Load: ~1.8M QPS** (mostly thumbnail views)

#### Database Sizing

**Photo Metadata Database (Cassandra):**
```text
Record Structure:
- photo_id: 16 bytes (UUID)
- user_id: 16 bytes
- upload_timestamp: 8 bytes
- location: 16 bytes (lat/long)
- EXIF data: 500 bytes (camera, settings)
- tags: 200 bytes
Total per photo: ~756 bytes ≈ 1 KB

Total photos (5 years): 500M/day × 365 × 5 = 912.5B photos
Metadata storage: 912.5B × 1 KB = 912.5 TB

With 3x replication: 2.7 PB metadata
```

**User Database (PostgreSQL):**
```text
Users: 1B
Per user data: 1 KB (name, email, settings, subscription)
Total: 1 TB (tiny compared to photos!)
```

**Face Embeddings (Vector Database):**
```text
Faces per photo: 3 average
Total faces: 912.5B photos × 3 = 2.7 trillion faces
Embedding size: 512 bytes (128-dim float32)
Total: 2.7T × 512 bytes = 1.38 PB

With HNSW index overhead (3x): 4.14 PB
```

#### Infrastructure Sizing

**Compute Instances:**
```text
API Servers (handle uploads/downloads):
- Peak QPS: 1.8M
- Capacity per server: 1K QPS
- Servers needed: 1,800
- With 2x redundancy: 3,600 servers

Image Processing Workers:
- Photos to process: 500M/day
- Processing time: 5 seconds/photo (thumbnails + metadata)
- Required capacity: 500M × 5s = 2.5B seconds/day
- Worker hours/day: 2.5B ÷ 3600 = 694,444 worker-hours
- Workers (24hr operation): 694,444 ÷ 24 = 28,935 workers
- Actual deployment: ~30,000 processing workers

ML Inference Servers (Face Recognition):
- Faces to process: 1.5B/day
- Inference time: 100ms/face
- GPU throughput: 100 faces/second (batch inference)
- GPUs needed: 1.5B / (86,400 × 100) = 174 GPUs
- With headroom: 250 GPUs (NVIDIA T4 or similar)
```

### 🔴 For Advanced: Production Considerations

#### Multi-Region Capacity Planning

Google Photos operates in 4 major regions with traffic distribution:

```text
Region Traffic Distribution:
├─ US-EAST (Virginia): 35% of traffic
│  ├─ Storage: 1.05 EB
│  ├─ API servers: 1,260
│  └─ Processing workers: 10,500
│
├─ US-WEST (Oregon): 15% of traffic
│  ├─ Storage: 450 PB
│  ├─ API servers: 540
│  └─ Processing workers: 4,500
│
├─ EU-WEST (Ireland): 30% of traffic
│  ├─ Storage: 900 PB
│  ├─ API servers: 1,080
│  └─ Processing workers: 9,000
│
└─ ASIA-SOUTHEAST (Singapore): 20% of traffic
   ├─ Storage: 600 PB
   ├─ API servers: 720
   └─ Processing workers: 6,000

Total Global Infrastructure:
- Storage: 3 EB
- API servers: 3,600
- Processing workers: 30,000
- ML GPUs: 250
```

#### Growth Projections & Capacity Planning

```text
Year 1 (Launch):
├─ Users: 100M
├─ Photos: 10B
├─ Storage: 30 PB
└─ Servers: 200

Year 2 (10x growth):
├─ Users: 1B
├─ Photos: 100B
├─ Storage: 300 PB
└─ Servers: 2,000

Year 5 (Current):
├─ Users: 1B (plateaued)
├─ Photos: 4T (accumulated)
├─ Storage: 3 EB
└─ Servers: 3,600

Year 7 (Projection):
├─ Users: 1.2B (slow growth)
├─ Photos: 6T
├─ Storage: 4.5 EB
└─ Servers: 5,000

Planning Strategy:
- Add capacity 6 months before projected need
- Storage: Grow 50%/year
- Compute: Grow 30%/year (better efficiency)
- GPU: Grow 100%/year (more ML features)
```

#### Cost Optimization at Scale

**Storage Tiering Strategy:**
```text
Hot Storage (Last 30 days): 20% of photos, 80% of views
├─ Technology: NVMe SSD
├─ Cost: $0.10/GB/month
├─ Size: 600 PB
└─ Monthly cost: $60M

Warm Storage (31-365 days): 30% of photos, 15% of views
├─ Technology: HDD (SATA)
├─ Cost: $0.023/GB/month
├─ Size: 900 PB
└─ Monthly cost: $20.7M

Cold Storage (1+ years): 50% of photos, 5% of views
├─ Technology: Tape/Glacier
├─ Cost: $0.004/GB/month
├─ Size: 1.5 EB
└─ Monthly cost: $6M

Total Storage Cost: $86.7M/month (vs $69M without tiering)
Wait, this is MORE expensive? 

With Compression:
Hot (SSD, compressed): $30M
Warm (HDD, compressed): $10M
Cold (Tape, compressed): $3M
Total: $43M/month
SAVINGS: $26M/month = $312M/year!
```

**Bandwidth Optimization:**
```text
Without CDN: 
- Origin bandwidth: 600 GB/s × $0.08/GB = $1.5B/year

With Global CDN (95% cache hit):
- Origin bandwidth: 30 GB/s (5% of traffic)
- CDN cost: Fixed $50M/year for global PoPs
- Origin cost: $75M/year (5% of $1.5B)
- Total: $125M/year
SAVINGS: $1.375B/year!

This is why CDN is CRITICAL for Google Photos!
```

#### Real Numbers from Google Photos

Based on publicly available information and industry reports:

```text
Google Photos (2023 data):
├─ Total users: 1+ billion
├─ Photos stored: 4+ trillion
├─ Daily uploads: 1.5 billion
├─ Storage: 4+ exabytes
├─ Annual costs: $200-300M (est)
├─ Revenue: $2-3B (Google One subscriptions)
└─ Profit margin: ~85% (highly profitable!)

Infrastructure:
├─ Data centers: 23 globally
├─ CDN PoPs: 7,500+ locations
├─ Servers: 50,000+ (est)
├─ TPUs for ML: 10,000+ (v4 pods)
└─ Network capacity: 1+ Tbps per DC
```

### Real-World Example: Capacity Planning Failure

**Instagram's Photo Growth Crisis (2012):**

```text
Problem:
- Launched Oct 2010: 25K users, 100 photos/hour
- Dec 2010: 1M users, 10K photos/hour (100x growth in 2 months!)
- Apr 2012: 30M users (before Facebook acquisition)
- Grossly underestimated storage needs

Result:
- Ran out of storage 3 times in first year
- Emergency migrations to AWS S3
- Had to re-architect entire storage system
- Almost crashed during Facebook acquisition due diligence

Lesson Learned:
- Plan for 10x growth in first year
- Plan for 100x growth in 3 years
- Always have 6-month storage buffer
- Automate capacity monitoring and alerting
```

### 🎯 Interview Questions: Capacity Planning

**Q1: How do you estimate the number of photos uploaded per day?**

<details>
<summary>Click to see answer</summary>

**Answer:**
Break it down step-by-step:

**Step 1: Identify active users**
- Total users: 1B
- Daily Active Users (DAU): Typically 30-50% for mobile apps
- Assume: 500M DAU

**Step 2: Identify uploaders**
- Not everyone uploads daily
- Typical: 15-25% of DAU upload
- Assume: 20% = 100M uploading users

**Step 3: Photos per uploader**
- Light users: 1-2 photos
- Active users: 5-10 photos
- Power users: 20+ photos
- Average: 5 photos/user (weighted)

**Calculation:**
```text
100M uploading users × 5 photos/user = 500M photos/day
```

**Validation:**
- Weekly: 3.5B photos
- Monthly: 15B photos  
- Yearly: 182.5B photos
- Does this make sense? Yes - Google Photos had 4T photos total over 8 years

**Interview Tip:** Always validate your final number makes sense in larger context!
</details>

**Q2: How would you estimate thumbnail storage requirements?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Why Multiple Thumbnails?**
Different use cases need different sizes:
- Grid view: 150×150 (tiny, load fast)
- Preview hover: 400×400 (medium quality)
- Mobile display: 1080p (full screen quality)

**Size Calculation:**
```text
Thumbnail Compression (JPEG quality 80):

150×150 pixels:
- 150 × 150 × 3 bytes (RGB) = 67.5 KB uncompressed
- JPEG compression (10:1): ~7-10 KB
- Use: 10 KB

400×400 pixels:
- 400 × 400 × 3 = 480 KB uncompressed
- JPEG compression: ~40-50 KB
- Use: 50 KB

1080p (1920×1080):
- 1920 × 1080 × 3 = 6.2 MB uncompressed
- JPEG compression: ~150-250 KB
- Use: 200 KB

Total per photo: 10 + 50 + 200 = 260 KB
```

**Daily Storage:**
```text
500M photos/day × 260 KB = 130 TB/day
```

**Yearly:**
```text
130 TB/day × 365 = 47.45 PB/year
```

**Ratio Check:**
```text
Thumbnails vs originals: 47 PB vs 547 PB = ~8.6%
This seems reasonable!
```

**Interview Tip:** Thumbnails are ~10% of original storage - use this as quick check.
</details>

**Q3: How many API servers do you need to handle peak traffic?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Step 1: Calculate Peak QPS**
```text
Daily views: 50B
Average QPS: 50B ÷ 86,400 = 578,703 QPS
Peak (3x): 1,736,111 QPS
```

**Step 2: Server Capacity**
Modern server (8-core, 32GB RAM):
- Can handle: 500-1,500 QPS (depending on operation)
- Conservative estimate: 1,000 QPS
- For light operations (thumbnails): 1,500 QPS
- For heavy operations (uploads): 500 QPS

**Step 3: Separate by Operation Type**
```text
Thumbnail Serves (90% of traffic):
- QPS: 1.56M
- Capacity: 1,500 QPS/server
- Servers: 1.56M ÷ 1,500 = 1,040 servers

Upload Processing (10% of traffic):
- QPS: 173K
- Capacity: 500 QPS/server
- Servers: 173K ÷ 500 = 346 servers

Total: 1,386 servers
```

**Step 4: Add Redundancy**
```text
For high availability (99.99%):
- Need N+1 redundancy per region
- 4 regions × 2x redundancy = 8x multiplier
- Actually: 2x is enough (one for failover)

Final: 1,386 × 2 = 2,772 servers
Round up: 3,000 API servers
```

**Interview Tip:** Always include redundancy! Never design single point of failure.
</details>

---

### 🤔 Think About It

1. **Storage Growth**: If Google Photos grows from 4 trillion to 6 trillion photos in 2 years, how much new storage capacity should you add each quarter? Consider lead time for hardware procurement.

2. **Cost vs Features**: Thumbnails cost 10% extra storage but make the app 10x faster. Would you generate thumbnails for ALL photos, or only for photos viewed at least once? What's the trade-off?

3. **Peak Traffic**: Photo platforms see 3-5x normal traffic on holidays (Christmas, New Year). How would you handle this WITHOUT keeping 5x servers idle 360 days/year?

---

### ✅ Key Takeaways

```text
✓ 500M DAU × 20% upload rate × 5 photos = 500M photos/day
✓ 500M photos × 3 MB = 1.5 PB/day storage needed
✓ Thumbnails add ~10% storage but crucial for performance
✓ Read-heavy: 100 views for every 1 upload (1:100 ratio)
✓ Peak traffic is 3x average - plan for peaks, not average
✓ Always add 2x redundancy for high availability
✓ Storage costs dominate: $69M/month for 3 EB on S3
✓ CDN saves $1.3B/year by reducing origin bandwidth
✓ Plan capacity 6 months in advance (procurement lead time)
✓ Validate numbers: Does yearly total match known data points?
```

**Critical Formulas to Remember:**
```text
Daily Storage = DAU × Upload% × Photos/User × Size/Photo
QPS = Daily_Operations ÷ 86,400
Peak_QPS = Average_QPS × 3
Servers = Peak_QPS ÷ Capacity_Per_Server × Redundancy_Factor
```

---

### 🎯 Practice Exercise

**Exercise: Capacity Planning for Instagram**

Instagram has:
- 500M DAU (same as our Google Photos estimate)
- But 50% upload daily (vs 20% for photos)
- Average upload: 2 photos/day (vs 5 for Google Photos)
- Average size: 2 MB (more compressed than Google Photos)

Calculate:
1. Daily photo uploads
2. Daily storage needed
3. Peak upload QPS
4. Number of API servers needed (assume 1K QPS per server with 2x redundancy)

<details>
<summary>Click to see solution</summary>

**Solution:**

**1. Daily Uploads:**
```text
500M DAU × 50% upload rate × 2 photos/user
= 250M users × 2 photos
= 500M photos/day
(Same as Google Photos!)
```

**2. Daily Storage:**
```text
500M photos × 2 MB/photo
= 1,000 TB/day
= 1 PB/day
(Less than Google Photos' 1.5 PB/day due to smaller size)
```

**3. Peak Upload QPS:**
```text
Average: 500M ÷ 86,400 = 5,787 photos/sec
Peak (3x): 17,361 photos/sec
```

**4. API Servers:**
```text
Assuming similar view ratio (1:100):
- Views: 50B/day
- Peak View QPS: 1.7M QPS
- Servers: 1.7M ÷ 1K = 1,700
- With 2x redundancy: 3,400 servers

Similar to Google Photos despite different usage patterns!
```

**Key Insight:** Instagram and Google Photos need similar infrastructure scale, but different feature focus (social vs storage).
</details>

---

**Ready for Section 3?** Next, we'll design the high-level system architecture, showing how all these components (API servers, storage, databases) fit together. You'll learn how to draw clean architecture diagrams that impress interviewers!

---

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
```

---

## Section 3: Designing the System Architecture

### What You'll Learn

By the end of this section, you'll be able to:
- Draw a clean high-level architecture diagram for a photo storage platform
- Explain each component and its responsibilities
- Describe the data flow for upload, view, and search operations
- Identify which components handle which scalability concerns

### Why This Matters

The high-level architecture is the foundation of your system design. It's what interviewers sketch on whiteboards and what guides all detailed design decisions. Real-world example: Google Photos' architecture evolved from a monolithic Google+ Photos (slow, coupled) to microservices in 2015 (fast, scalable). The new architecture enabled them to scale from 0 to 1 billion users in just 3 years by making each component independently scalable!

---

### 🟢 For Beginners: The Fundamentals

#### What is System Architecture?

Think of system architecture like a city plan:

```text
🏙️ City Plan:
├─ Roads (how traffic flows)
├─ Buildings (where things happen)
├─ Utilities (water, power)
└─ Connections (how it all works together)

💻 System Architecture:
├─ API Gateway (how requests arrive)
├─ Services (where work happens)
├─ Databases (where data lives)
└─ Connections (how components talk)
```

#### The Layers of Google Photos

Our system has 6 main layers (like floors in a building):

**Layer 1: Client Layer (What Users See)**
```text
Web Client → Browser (Chrome, Safari, Firefox)
Mobile Apps → iOS/Android apps on your phone
```

**Layer 2: CDN & Load Balancing (Traffic Directors)**
```text
CDN → Delivers photos fast (cached nearby)
Load Balancer → Spreads requests across many servers
```

**Layer 3: API Gateway (Front Door)**
```text
API Gateway → Routes requests to right service
Auth Service → Checks "Are you allowed in?"
```

**Layer 4: Service Layer (Workers)**
```text
Upload Service → Handles photo uploads
View Service → Delivers photos for viewing
Search Service → Finds photos by content
Share Service → Manages sharing permissions
Album Service → Organizes photos into albums
```

**Layer 5: Processing Layer (Background Workers)**
```text
Message Queue (Kafka) → Holds work to be done
Image Processor → Creates thumbnails
Metadata Extractor → Reads EXIF data (date, camera, location)
Face Detection → Finds faces in photos
Face Recognition → Identifies who each face is
```

**Layer 6: Data Layer (Where Everything Lives)**
```text
Databases:
├─ Metadata DB (Cassandra) → Photo information
├─ User DB (PostgreSQL) → User accounts, albums
├─ Search Index (Elasticsearch) → Fast photo search
├─ Face DB (Milvus) → Face recognition data
└─ Cache (Redis) → Temporary fast storage

Storage:
├─ Hot Storage (S3) → Recent photos (fast)
├─ Warm Storage (S3 IA) → Older photos (medium)
└─ Cold Storage (Glacier) → Ancient photos (slow, cheap)
```

#### How a Photo Upload Works (Simple Explanation)

Let's trace what happens when you upload a photo from your phone:

```text
Step 1: You click "Upload" in the app
   ↓
Step 2: Photo goes to nearest CDN (like a post office)
   ↓
Step 3: CDN sends to Load Balancer (traffic cop)
   ↓
Step 4: Load Balancer picks a server (not too busy)
   ↓
Step 5: API Gateway checks: "Is this user logged in?"
   ↓
Step 6: Upload Service receives photo
   ↓
Step 7: Photo stored in S3, you see "Upload complete!"
   ↓
Step 8: (Background) Queue schedules processing
   ↓
Step 9: (Background) Image Processor:
   - Makes thumbnails (small, medium, large)
   - Extracts metadata (date, location, camera)
   - Detects faces
   - Stores everything in databases
   ↓
Step 10: You can now search for and share your photo!
```

**Key Insight:** Steps 1-7 happen fast (< 5 seconds). Steps 8-10 happen in background (don't slow down upload!).

💡 **Pro Tip:** In interviews, always separate "critical path" (user waits) from "background jobs" (user doesn't wait). This shows you understand performance!

#### How Viewing a Photo Works

When you open Google Photos and scroll through your timeline:

```text
1. Your app requests: "Show me 50 photos"
   ↓
2. View Service checks Redis cache
   - Cache hit? Return instantly! ⚡
   - Cache miss? Query Cassandra database
   ↓
3. View Service returns URLs like:
   "https://cdn.googlephotos.com/photo123-thumb.jpg"
   ↓
4. Your app loads photos from CDN (super fast!)
```

**Why this is fast:**
- Cache hit: 1-5ms (from memory)
- CDN delivery: 10-50ms (from nearby server)
- Total: < 100ms to show 50 photos!

---

### 🟡 For Intermediate: Interview Patterns

#### How to Draw Architecture Diagrams in Interviews

**Step 1: Start with layers (top to bottom)**
```text
Whiteboard Layout:

[Clients]              ← Top: Where users are
    ↓
[Load Balancing]       ← Entry point
    ↓
[API Services]         ← Business logic
    ↓
[Databases/Storage]    ← Bottom: Where data lives
```

**Step 2: Add key components**
```text
You: "I'll start with the client layer. We support web and mobile..."
[Draw boxes for Web, iOS, Android]

You: "Next, CDN and load balancing for global distribution..."
[Draw CDN and LB boxes]

You: "API layer with authentication..."
[Draw API Gateway and Auth boxes]

You: "Service layer - I'll separate concerns..."
[Draw Upload, View, Search, Share, Album services]

You: "Background processing for async work..."
[Draw Kafka queue and processors]

You: "Finally, data layer with appropriate databases..."
[Draw Cassandra, PostgreSQL, Elasticsearch, Vector DB]
```

**Step 3: Show data flows**
```text
You: "Let me trace the upload flow..."
[Draw arrows with numbers]

You: "And the view flow is simpler..."
[Draw different colored arrows]
```

#### Component Responsibilities (Interview Answer Framework)

When asked "What does each component do?", structure your answer:

| Component | Responsibility | Why This Choice | Alternative |
|-----------|---------------|----------------|-------------|
| **API Gateway** | Route requests, rate limiting, auth | Single entry point, centralized control | Direct service calls (messy) |
| **Cassandra** | Photo metadata storage | Write-heavy, wide-column, scalable | MongoDB (less scalable at PB scale) |
| **PostgreSQL** | User/album data | ACID transactions, complex queries | Cassandra (overkill for small data) |
| **Elasticsearch** | Photo search | Full-text search, aggregations | PostgreSQL (slow for text search) |
| **Milvus** | Face embeddings | Vector similarity search | PostgreSQL pgvector (not scalable) |
| **Redis** | Caching layer | Sub-millisecond reads, reduces DB load | Memcached (less features) |
| **Kafka** | Async job queue | High throughput, replay capability | RabbitMQ (lower throughput) |
| **S3** | Photo storage | Unlimited scale, 11 9's durability | Self-hosted (expensive at scale) |

#### Data Flow Patterns

**Pattern 1: Synchronous (User Waits)**
```text
Upload Photo:
Client → API → Upload Service → S3 → Return Success
Time: ~3 seconds (critical path)

View Photo:
Client → API → Cache/DB → Return URL → CDN → Client
Time: ~100ms (critical path)
```

**Pattern 2: Asynchronous (Background)**
```text
Process Photo:
Upload Service → Kafka → (hours later) → Image Processor
Time: User doesn't wait!

Why async:
- Thumbnails take 3-5 seconds
- Face detection takes 10-15 seconds
- Don't make user wait 20 seconds!
```

**Pattern 3: Hybrid (Optimistic UI)**
```text
Share Photo:
1. Client shows "Shared!" immediately (optimistic)
2. Share Service updates permissions in background
3. If fails, show error and revert

Better UX: Feels instant even if backend is slow
```

#### Architecture Decision Interview Questions

**Q: "Why separate Upload and View services?"**

A: "Different scalability characteristics:
- Upload: Write-heavy, 6K QPS, CPU-intensive (validation)
- View: Read-heavy, 700K QPS, I/O-intensive (DB queries)
- Separate services let us scale independently
- Upload needs fewer, bigger machines
- View needs many, smaller machines"

**Q: "Why use both PostgreSQL AND Cassandra?"**

A: "Right tool for the job:
- PostgreSQL for relational data (users, albums)
  - Small data (1 TB)
  - Complex queries (JOIN user + albums + shares)
  - ACID transactions needed
  
- Cassandra for photo metadata
  - Huge data (3 PB)
  - Simple queries (get photos by user_id + date)
  - Eventual consistency OK
  - Must handle 6K writes/sec

Using one DB for both would be suboptimal."

**Q: "Why Kafka instead of direct processing?"**

A: "Decoupling and reliability:
1. **Decoupling:** Upload Service doesn't know about Face Detection Service
   - Can deploy independently
   - Can replace face detection without touching upload
   
2. **Buffering:** Kafka absorbs traffic spikes
   - Upload spike to 20K QPS? Kafka buffers
   - Processors drain at steady rate
   
3. **Replay:** If face detection bugs out, replay from Kafka
   - Don't lose processing work
   
4. **Multiple consumers:** Metadata extraction AND thumbnail generation both read from same queue"

---

### 🔴 For Advanced: Production Considerations

#### Multi-Region Architecture

Google Photos doesn't run in one data center. It's globally distributed:

```text
Global Deployment:

Primary Regions:
├─ us-east-1 (Virginia)
│  ├─ Serves: North America (40% traffic)
│  ├─ Storage: 1.2 EB
│  └─ Users: 400M
│
├─ eu-west-1 (Ireland)
│  ├─ Serves: Europe (30% traffic)
│  ├─ Storage: 900 PB
│  └─ Users: 300M
│
├─ asia-southeast-1 (Singapore)
│  ├─ Serves: Asia-Pacific (25% traffic)
│  ├─ Storage: 750 PB
│  └─ Users: 250M
│
└─ southamerica-east-1 (São Paulo)
   ├─ Serves: South America (5% traffic)
   ├─ Storage: 150 PB
   └─ Users: 50M

Replication Strategy:
- Photos: Stored in user's home region + 1 backup region
- Metadata: Multi-region with eventual consistency
- User data: Multi-region with quorum reads/writes
```

**Cross-Region Considerations:**

1. **Latency:**
   - Same-region: 10-30ms
   - Cross-region (US-EU): 80-120ms
   - Cross-region (US-Asia): 150-200ms
   - Solution: Keep user's photos in their primary region

2. **Data Sovereignty:**
   - GDPR: EU users' data must stay in EU
   - Solution: Region affinity in user profile
   - Cannot move data without user consent

3. **Disaster Recovery:**
   - Region failure: Traffic shifts to backup in 15 minutes
   - Data loss: RPO = 0 (continuous replication)
   - RTO: 1 hour for full failover

#### Service Mesh & Microservices

Production Google Photos uses 50+ microservices:

```text
Microservices Architecture:

Upload Domain:
├─ Upload Validation Service
├─ Duplicate Detection Service
├─ Upload Orchestration Service
└─ Multipart Upload Service

Processing Domain:
├─ Thumbnail Service (3 types)
├─ Metadata Extraction Service
├─ EXIF Parser Service
├─ GPS Enrichment Service
├─ Image Optimization Service
└─ Format Conversion Service

ML Domain:
├─ Face Detection Service
├─ Face Recognition Service
├─ Object Detection Service
├─ Scene Classification Service
├─ NSFW Content Filter Service
└─ Quality Assessment Service

Search Domain:
├─ Text Search Service
├─ Visual Search Service
├─ Location Search Service
└─ Face Search Service

Sharing Domain:
├─ Permission Service
├─ Link Generation Service
├─ Notification Service
└─ Collaboration Service

Service Mesh (Istio):
├─ Service Discovery
├─ Load Balancing
├─ Circuit Breaking
├─ Retry Logic
├─ Distributed Tracing
└─ mTLS Encryption
```

**Why so many services?**

1. **Team Ownership:** Each team owns 2-3 services
2. **Independent Deployment:** Can deploy Face Detection without touching Upload
3. **Technology Choice:** ML services use Python/TensorFlow, API services use Go
4. **Scaling:** Scale object detection 10x without scaling thumbnails
5. **Fault Isolation:** Bug in NSFW filter doesn't crash uploads

#### Advanced Data Flow: Upload with Deduplication

Real Google Photos detects duplicate photos:

```text
Upload Flow with Deduplication:

1. Client uploads photo
   ↓
2. Upload Service calculates perceptual hash
   (Even if slightly edited, same hash)
   ↓
3. Query Dedup DB: "Do we have this hash?"
   ├─ Yes → Return existing photo ID
   │         Save storage!
   │         User sees "Photo uploaded" (instant)
   └─ No → Continue to S3 upload
       ↓
4. Store in S3 with content-based key
   ↓
5. Save hash → photo_id mapping in Dedup DB
   ↓
6. Continue with processing pipeline

Benefits:
- Users upload same photo from 3 devices? Stored once!
- Save 20-30% storage (massive at EB scale)
- Instant "upload" if duplicate (already have it)
```

**Perceptual Hash Algorithm:**
```text
pHash (Perceptual Hash):
1. Resize to 32×32 pixels (normalize size)
2. Convert to grayscale
3. Compute Discrete Cosine Transform (DCT)
4. Extract low frequencies (8×8)
5. Compute average
6. Generate 64-bit hash based on frequencies above/below average

Properties:
- Identical photos: Same hash
- Slight edits (crop, brightness): Similar hash
- Different photos: Very different hash
- Fast: 10ms per photo
```

### Real-World Example: Google Photos Architecture Evolution

#### 2011-2015: Google+ Photos (Failed Architecture)

```text
Monolithic Architecture:
┌─────────────────────────────┐
│   Google+ (Monolith)        │
│  ┌───────────────────────┐  │
│  │  Social Feed          │  │
│  │  Photo Upload         │  │
│  │  Photo Viewing        │  │
│  │  Comments/Likes       │  │
│  │  Circles (Privacy)    │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
        ↓
   Single Database
   (Bottleneck!)

Problems:
- Couldn't scale photo storage independently
- Social features interfered with photo features
- Deploy social → risk breaking photos
- Database overloaded (social + photos)
Result: Only 300M users after 4 years
```

#### 2015-Present: Google Photos (Successful Architecture)

```text
Microservices Architecture:
┌───────────┐  ┌───────────┐  ┌──────────┐
│  Upload   │  │   View    │  │  Search  │
│  Service  │  │  Service  │  │  Service │
└───────────┘  └───────────┘  └──────────┘
      ↓              ↓              ↓
┌─────────────────────────────────────────┐
│         Independent Databases           │
│  Cassandra  PostgreSQL  Elasticsearch   │
└─────────────────────────────────────────┘

Benefits:
- Scale each service independently
- Deploy photos without Google+ coordination
- Photos-only focus (no social features)
- Faster development (smaller teams)
Result: 1B users in just 3 years!
```

**Key Architectural Changes:**

1. **Separated from Google+**
   - Before: Photo feature inside social network
   - After: Standalone photo platform
   - Impact: Focused product, clearer value prop

2. **Database per Service**
   - Before: Shared MySQL database
   - After: Cassandra (photos), PostgreSQL (users), Elasticsearch (search)
   - Impact: Right tool for each job

3. **Async Processing**
   - Before: Synchronous (upload waits for thumbnails)
   - After: Kafka queue (upload returns immediately)
   - Impact: 5x faster upload experience

4. **Global CDN**
   - Before: Served from US data centers only
   - After: CloudFlare + Google Global Cache in 7,500+ locations
   - Impact: 50ms load time globally vs 300ms before

### 🎯 Interview Questions: System Architecture

**Q1: Why use a message queue (Kafka) instead of direct service calls for image processing?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Without Message Queue (Direct Calls):**
```text
Upload Service ─────> Image Processor
                      (blocked until complete!)
Problem: Upload Service waits 20 seconds for processing
Result: Poor user experience, low throughput
```

**With Message Queue:**
```text
Upload Service ──→ Kafka ──→ Image Processor
     ↓ (returns immediately)
"Upload successful!"
```

**Benefits:**

1. **Decoupling:**
   - Upload Service doesn't care who processes images
   - Can change Image Processor without touching Upload Service
   - Can add new consumers (e.g., duplicate detection) easily

2. **Async Processing:**
   - User sees "Upload successful" in 3 seconds
   - Processing happens in background over 20 seconds
   - Better UX: Feels 7x faster!

3. **Buffering Traffic Spikes:**
   - Christmas morning: 50K uploads/sec spike
   - Kafka buffers messages
   - Processors drain at steady 6K/sec
   - No processors crash from overload

4. **Retry & Reliability:**
   - Image Processor crashes? Message stays in queue
   - Retry 3 times before giving up
   - Can replay failed messages from last hour

5. **Multiple Consumers:**
   - Thumbnail Generator reads queue
   - Metadata Extractor reads same queue
   - Face Detector reads same queue
   - All process in parallel!

**Trade-off:**
- Complexity: Need to manage Kafka cluster
- Latency: Processing happens later (OK for non-critical work)
- Cost: Kafka infrastructure ($50K/month for 100-node cluster)

**Interview Tip:** This pattern is called "Event-Driven Architecture" - very common in scalable systems!
</details>

**Q2: Why use multiple databases (Cassandra, PostgreSQL, Elasticsearch) instead of just one?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Polyglot Persistence** = Right database for each job

**Cassandra (Photo Metadata):**
```text
Use Case: Store 4 trillion photo records
Access Pattern: Get photos by user_id + date range
Volume: 3 PB of metadata
Why Cassandra:
✓ Write-heavy (1.5B writes/day)
✓ Time-series data (partition by date)
✓ Horizontal scaling (add nodes easily)
✓ Eventually consistent (OK for metadata)
✗ Bad at: JOINs, transactions
```

**PostgreSQL (Users & Albums):**
```text
Use Case: User accounts, album definitions, sharing
Access Pattern: Complex queries with JOINs
Volume: 1 TB (small!)
Why PostgreSQL:
✓ ACID transactions (critical for permissions)
✓ Complex queries (JOIN user + album + shares)
✓ Strong consistency (security requires it)
✓ Mature, well-understood
✗ Bad at: Billions of records, horizontal scaling
```

**Elasticsearch (Search Index):**
```text
Use Case: Search photos by text, tags, location
Access Pattern: Full-text search, fuzzy matching
Volume: 500 GB (just searchable fields)
Why Elasticsearch:
✓ Full-text search (best in class)
✓ Fuzzy matching ("beech" finds "beach")
✓ Aggregations (count by date, location)
✓ Fast (< 50ms for complex queries)
✗ Bad at: Primary storage, transactions
```

**Milvus (Face Embeddings):**
```text
Use Case: Find similar faces (vector similarity)
Access Pattern: K-nearest neighbors search
Volume: 4 PB (2.7T faces × 512 bytes × 3x index)
Why Milvus:
✓ Vector similarity search (purpose-built)
✓ HNSW index (sub-second search in billions)
✓ GPU acceleration
✗ Bad at: Everything except vectors
```

**Could you use just PostgreSQL?**

Technically yes, but:
- 4T rows in PostgreSQL? Super slow queries
- Full-text search in PostgreSQL? 100x slower than Elasticsearch
- Vector search in PostgreSQL? 1000x slower than Milvus
- Write throughput? PostgreSQL maxes at ~10K writes/sec vs Cassandra's 1M writes/sec

**Cost of complexity:**
- Need to learn 4 databases
- Data consistency across databases
- More operational overhead

**But benefits outweigh costs at Google Photos scale!**

**Interview Tip:** This question tests if you understand trade-offs. Don't just say "use PostgreSQL for everything" - show you know when to use specialized tools!
</details>

---

### 🤔 Think About It

1. **Service Boundaries:** If you were designing this system, would you separate Upload and View into different services, or keep them together? What are the trade-offs?

2. **Async vs Sync:** Which operations MUST be synchronous (user waits) and which can be async (background)? Why?

3. **Database Choices:** Could you use MongoDB instead of Cassandra for photo metadata? What would change?

---

### ✅ Key Takeaways

```text
✓ Architecture has 6 layers: Client → CDN → API → Services → Processing → Data
✓ Separate services by domain (Upload, View, Search, Share, Album)
✓ Use message queue (Kafka) to decouple services and handle async work
✓ Polyglot persistence: Multiple databases for different needs
✓ Critical path (user waits) vs background jobs (user doesn't wait)
✓ Multi-region for global latency and disaster recovery
✓ CDN is critical: 95% cache hit rate saves $1.3B/year
✓ Microservices enable independent scaling and deployment
✓ Real Google Photos uses 50+ services (we simplified to ~10 for interview)
```

**Critical Architecture Decisions:**
```text
1. Async processing → 7x faster perceived upload time
2. Polyglot persistence → Right tool for each job
3. CDN → 100ms global latency vs 300ms without
4. Multi-region → 99.99% availability despite region failures
5. Service separation → Independent scaling (view 100x more than upload)
```

---

### 🎯 Practice Exercise

**Exercise: Redesign for Video**

Google Photos also supports video. Videos have different requirements:
- Much larger (100 MB vs 3 MB for photos)
- Need transcoding (convert to streamable formats)
- Multiple quality levels (360p, 720p, 1080p, 4K)
- Longer processing time (5 minutes vs 5 seconds)

**Question:** How would you modify the architecture to support video? 

Consider:
1. Which services need changes?
2. What new services do you need?
3. How does storage change?
4. How does the upload flow change?

<details>
<summary>Click to see sample answer</summary>

**Answer:**

**New/Modified Services:**

1. **Upload Service (Modified):**
   - Add chunked upload for large files
   - Validate file size (reject > 10GB)
   - Stream directly to S3 (don't hold in memory)

2. **Video Transcoding Service (NEW):**
   ```text
   Input: Original video (100 MB, 4K)
   Output: Multiple formats
   ├─ 360p (10 MB)
   ├─ 720p (30 MB)
   ├─ 1080p (60 MB)
   └─ 4K (100 MB - passthrough)
   
   Tech: FFmpeg on GPU instances (NVIDIA T4)
   Time: ~5 minutes for 2-minute video
   ```

3. **Adaptive Streaming Service (NEW):**
   - Generate HLS/DASH manifests
   - Segment videos into chunks
   - Enable adaptive bitrate streaming

**Architecture Changes:**

```text
Upload Flow:
User ──→ Upload Service ──→ S3 (original)
                         ──→ Kafka (transcode job)

Transcode Flow:
Kafka ──→ GPU Worker ──→ Transcode ──→ S3 (all formats)
                                     ──→ Update metadata

View Flow:
User ──→ View Service ──→ CDN ──→ Adaptive streaming
                                  (auto-switch quality)
```

**Storage Impact:**
```text
Before (photos): 3 MB original + 260 KB thumbnails
After (videos): 100 MB original + 100 MB formats = 200 MB
                67x more storage per video!

Daily uploads: 50M videos × 200 MB = 10 PB/day (vs 1.5 PB for photos)
```

**Cost Impact:**
```text
Transcoding cost:
- 50M videos/day
- 5 minutes GPU time each
- = 250M GPU minutes/day
- = 4.17M GPU hours/day
- At $0.50/GPU-hour = $2M/day = $730M/year

(This is why YouTube compression is so aggressive!)
```

**Interview Insight:** Shows you understand video is fundamentally different from photos - not just "bigger photos"!
</details>

---

**Ready for Section 4?** Next, we'll design the database schemas for users, photos, albums, and face recognition. You'll learn how to structure data for billions of records with optimal query performance!

---

## Section 4: Storing Our Data

### What You'll Learn

By the end of this section, you'll be able to:
- Design database schemas for users, photos, albums, and face recognition
- Choose the right database type for different data patterns (PostgreSQL vs Cassandra vs Elasticsearch vs Vector DB)
- Implement efficient indexing strategies for billions of records
- Structure data for optimal query performance at scale

### Why This Matters

Database design makes or breaks scalability. Poor schema design can't be fixed with more servers. Real-world example: In 2016, Flickr (Yahoo's photo platform) struggled with slow queries because they stored everything in a single MySQL database. Their "All Photos" page took 30+ seconds to load! Meanwhile, Google Photos with proper database architecture loads 50 photos in < 100ms. The difference? Using the right database for each job (polyglot persistence).

---

### 🟢 For Beginners: The Fundamentals

#### Why Multiple Databases?

Think of databases like different types of storage in your home:

```text
🏠 Home Storage Analogy:

Closet (PostgreSQL):
├─ Your clothes (user accounts)
├─ Organized by type
├─ Small, frequently accessed
└─ Easy to find specific items

Warehouse (Cassandra):
├─ Old furniture, boxes (photos metadata)
├─ Tons of stuff!
├─ Organized by date
└─ Good for "show me all from 2020"

Library Card Catalog (Elasticsearch):
├─ Index of all books (search)
├─ Find by title, author, topic
├─ Don't store full books, just references
└─ Super fast to search

Photo Album (Vector DB):
├─ Face recognition data
├─ "Find similar faces"
├─ Special organization for similarity
└─ Different from normal storage
```

#### The 4 Main Databases We Use

**1. PostgreSQL (User & Album Data)**

What it stores:
- User accounts (1 billion users)
- Albums (5 billion albums)
- Sharing permissions

Why PostgreSQL:
- ✅ Handles relationships well (user HAS albums, album HAS photos)
- ✅ ACID transactions (changing permissions must be atomic)
- ✅ Data is small (< 1 TB total)
- ✅ Complex queries work (JOIN users with albums with shares)

Example query:
```sql
-- Get all albums shared with me
SELECT a.* FROM albums a
JOIN sharing s ON a.album_id = s.resource_id
WHERE s.shared_with_user_id = 'my_user_id'
```

**2. Cassandra (Photo Metadata)**

What it stores:
- Photo information (4 trillion photos!)
- Upload date, location, camera info
- Which albums contain each photo

Why Cassandra:
- ✅ Writes are super fast (1.5B uploads/day = 17K/sec)
- ✅ Handles massive data (3 PB of metadata)
- ✅ Time-series friendly (organized by upload date)
- ✅ Scales horizontally (just add more nodes)

Example structure:
```text
User's photos organized by date:
user_id: "john123"
  └─ 2025-01-15
      ├─ photo_001.jpg
      ├─ photo_002.jpg
      └─ photo_003.jpg
  └─ 2025-01-14
      ├─ photo_004.jpg
      └─ photo_005.jpg
```

**3. Elasticsearch (Search)**

What it stores:
- Searchable fields only (tags, location names, camera info)
- NOT the actual photos!

Why Elasticsearch:
- ✅ Search "beach" → finds all beach photos (< 50ms)
- ✅ Fuzzy matching ("beech" finds "beach")
- ✅ Filter by date + location + camera (fast!)
- ✅ Aggregations ("show count by year")

Example search:
```text
Search: "sunset beach california 2024"
→ Finds photos with ANY of these words
→ Ranks by relevance
→ Returns in 30ms!
```

**4. Milvus (Face Embeddings)**

What it stores:
- Face "fingerprints" (512 numbers per face)
- Used to find similar faces

Why Milvus:
- ✅ Finds similar faces (< 100ms among billions!)
- ✅ Purpose-built for AI vectors
- ✅ Much faster than normal databases for this

How it works:
```text
Your face → AI model → [0.234, 0.891, ..., 0.432]
                       512 numbers = your "face fingerprint"

Find similar:
"Show me faces like this one"
→ Compares 512 numbers
→ Returns top 100 similar faces
```

**5. Redis (Cache)**

What it stores:
- Temporary data (TTL: expires after hours/days)
- Recently viewed photos
- Popular albums

Why Redis:
- ✅ In-memory = super fast (1-5ms)
- ✅ Reduces load on main databases (95% cache hit!)
- ✅ Saves money (fewer database queries)

💡 **Pro Tip:** Use cache for anything accessed multiple times. First request: slow (from database). Next 1000 requests: fast (from cache)!

#### The Main Tables (Simple Version)

**Users Table (PostgreSQL):**
```text
user_id  | email           | storage_used | subscription
---------|-----------------|--------------|-------------
john123  | john@email.com  | 45.2 GB      | premium
mary456  | mary@email.com  | 8.7 GB       | free
```

**Photos Table (Cassandra):**
```text
user_id | upload_date | photo_id | file_name     | location      | storage_url
--------|-------------|----------|---------------|---------------|------------------
john123 | 2025-01-15  | photo001 | beach.jpg     | Malibu, CA    | s3://bucket/...
john123 | 2025-01-15  | photo002 | sunset.jpg    | Malibu, CA    | s3://bucket/...
```

**Faces Table (PostgreSQL + Milvus):**
```text
PostgreSQL (relationships):
face_id | photo_id | person_id | person_name
--------|----------|-----------|-------------
face001 | photo001 | person1   | John
face002 | photo001 | person2   | Mary

Milvus (AI embeddings):
face_id | embedding_vector (512 numbers)
--------|--------------------------------
face001 | [0.234, 0.891, ..., 0.432]
face002 | [0.123, 0.567, ..., 0.789]
```

---

### 🟡 For Intermediate: Interview Patterns

#### Database Selection Framework

When asked "Which database would you use?", follow this framework:

| Criteria | PostgreSQL | Cassandra | Elasticsearch | Milvus | Redis |
|----------|-----------|-----------|---------------|--------|-------|
| **Data Size** | < 10 TB | 100+ TB | < 1 TB (index only) | 1+ TB | < 100 GB |
| **Access Pattern** | Complex JOINs | Time-series, append-only | Full-text search | Vector similarity | Key-value lookups |
| **Write Throughput** | < 10K/sec | 100K+ /sec | < 5K/sec | 10K+ /sec | 100K+ /sec |
| **Consistency** | Strong (ACID) | Eventual | Eventual | Eventual | Eventual |
| **Query Type** | Transactional | By partition key | Text search | k-NN search | Get/Set |
| **Best For** | Users, permissions | Metrics, logs, photos | Search, aggregations | AI embeddings | Caching |

#### Cassandra Data Modeling (Critical for Interviews)

**Key Concept: Design for your queries**

In Cassandra, you must know your queries FIRST, then design the schema:

```text
❌ Wrong Approach (SQL thinking):
"Let me design a normalized photos table..."
→ Later: Can't query efficiently!

✅ Right Approach (Cassandra thinking):
"I need to query: Get photos by user + date range"
→ Design schema for THIS query!
```

**Partition Key Selection:**

```text
Query: "Get all photos for user john123"
Partition Key: user_id ← All john's photos on same node!

Why this works:
- Cassandra stores all rows with same partition key together
- Single-node query = fast (< 10ms)
- Cross-node query = slow (> 100ms)

Bad partition key: photo_id
- Each photo on different node
- Query for user's photos hits 1000 nodes!
- 100x slower
```

**Clustering Key for Sorting:**

```text
Schema:
Partition Key: user_id
Clustering Key: upload_date DESC

Result:
user_id: john123
  2025-01-15 → [photo1, photo2, photo3]
  2025-01-14 → [photo4, photo5]
  2025-01-13 → [photo6]
  
Query: "Get latest 50 photos"
→ Just read first 50 rows (already sorted!)
→ No sorting needed → fast!
```

#### Indexing Strategy

**PostgreSQL Indexes:**

```sql
-- Users table
CREATE INDEX idx_email ON users(email);  -- Login query
CREATE INDEX idx_username ON users(username);  -- Profile lookup

-- Albums table  
CREATE INDEX idx_user_created ON albums(user_id, created_at DESC);  
-- Get user's albums sorted by date

-- Sharing table
CREATE INDEX idx_share_link ON sharing(share_link);  -- Share link lookup
CREATE INDEX idx_resource_owner ON sharing(resource_id, owner_id);  
-- Check permissions
```

**Why these specific indexes?**

```text
Index on (user_id, created_at DESC):
Query: SELECT * FROM albums 
       WHERE user_id = 'john123' 
       ORDER BY created_at DESC 
       LIMIT 20;

Without index:
1. Scan ALL albums (billions!)
2. Filter by user_id
3. Sort by created_at
4. Return top 20
Time: 30+ seconds ❌

With index:
1. Jump to john's albums (index)
2. Already sorted by date!
3. Read first 20
Time: 5ms ✅
```

#### Data Partitioning Strategy

**Cassandra Partitioning (Photos):**

```text
Partition Key: user_id
Challenge: Some users have 100K+ photos!

Solution: Partition by user_id + year-month

Partition Key: (user_id, year_month)
Clustering Key: upload_date DESC, photo_id

Result:
john123_2025-01 → 500 photos (good size!)
john123_2024-12 → 450 photos
john123_2024-11 → 600 photos

Query pattern:
"Get photos from last 30 days"
→ Query 1-2 partitions only
→ Fast even for power users!
```

**Milvus Partitioning (Faces):**

```text
Partition by user_id:
- Each user's faces in separate partition
- Search only user's own partition
- Privacy: Can't accidentally search other users' faces
- Performance: Smaller index = faster search

Without partitioning:
- Search across 2.7 trillion faces
- Time: 10+ seconds

With partitioning (by user):
- Search user's 10,000 faces
- Time: 50ms!
```

#### Schema Evolution & Versioning

**Adding Fields to Cassandra:**

```text
Initial schema (2015):
- photo_id
- user_id  
- file_name
- upload_date

New requirement (2020): Add AI tags
ALTER TABLE photos ADD tags SET<TEXT>;

Cassandra handles gracefully:
- Old rows: tags = null
- New rows: tags = ['beach', 'sunset']
- No migration needed!
```

**PostgreSQL Migrations:**

```sql
-- Version 1: Basic users table
CREATE TABLE users (
  user_id UUID PRIMARY KEY,
  email VARCHAR(255),
  created_at TIMESTAMP
);

-- Version 2: Add storage tracking
ALTER TABLE users 
  ADD COLUMN storage_used_gb DECIMAL(10,2) DEFAULT 0,
  ADD COLUMN storage_quota_gb INT DEFAULT 15;

-- Version 3: Add subscription tiers
ALTER TABLE users 
  ADD COLUMN subscription_tier VARCHAR(20) DEFAULT 'free';

-- Each migration versioned and tracked
```

### 🔴 For Advanced: Production Considerations

#### Multi-Region Database Strategy

**Cassandra Multi-Region Setup:**

```text
Global Deployment (3 regions):

Region: US-EAST-1
├─ Replication Factor: 3
├─ Consistency: LOCAL_QUORUM
├─ Serves: North America users
└─ Data: 1.2 EB

Region: EU-WEST-1
├─ Replication Factor: 3
├─ Consistency: LOCAL_QUORUM  
├─ Serves: Europe users
└─ Data: 900 PB

Region: ASIA-SOUTHEAST-1
├─ Replication Factor: 3
├─ Consistency: LOCAL_QUORUM
├─ Serves: Asia users
└─ Data: 750 PB

Cross-Region Replication:
- Async replication between regions
- Eventual consistency (5-10 seconds)
- Each region self-sufficient (can operate alone)
```

**Consistency Trade-offs:**

```text
LOCAL_QUORUM reads/writes:
- 2 out of 3 local nodes must agree
- Latency: 10-30ms (same region)
- Availability: Tolerates 1 node failure
- Consistency: Strong within region

QUORUM across regions:
- 5 out of 9 nodes globally must agree
- Latency: 100-200ms (cross-region)
- Availability: Lower (network partitions)
- Consistency: Strong globally

Google Photos choice: LOCAL_QUORUM
- Fast (low latency)
- High availability
- Eventual consistency OK for photos
- Except: Permissions use global QUORUM!
```

#### Database Sizing & Sharding

**Cassandra Cluster Sizing:**

```text
Metadata Storage Calculation:
- 4 trillion photos
- 1 KB per photo metadata
- = 4 TB raw data
- × 3 replication = 12 TB
- × 1.5 overhead = 18 TB total

Node Capacity:
- Modern SSD: 4 TB per node
- Recommended: 1-2 TB per node (performance)
- Use: 2 TB per node

Cluster Size:
- 18 TB ÷ 2 TB per node = 9 nodes minimum
- Production: 12 nodes (33% headroom)
- Per region: 12 nodes × 3 regions = 36 nodes globally

Cost per node:
- i3.2xlarge: 8 vCPU, 61GB RAM, 1.9TB NVMe
- $0.624/hour = $4,489/month
- 36 nodes × $4,489 = $161,604/month
- Just for Cassandra cluster!
```

**PostgreSQL Sharding Strategy:**

```text
Users Table Sharding (when > 10TB):

Shard by user_id hash:
- user_id → hash → shard number
- Shard 0: users with hash 0-999
- Shard 1: users with hash 1000-1999
- ... 
- Shard 9: users with hash 9000-9999

10 shards = 100M users per shard

Query routing:
hash('john123') = 5432
→ Query Shard 5
→ Single shard query (fast!)

Challenges:
- Cross-shard JOINs impossible
- Rebalancing difficult (adding shards)
- Google Photos hasn't needed this yet (< 1TB)
```

#### Advanced Indexing Techniques

**Cassandra Secondary Indexes (When to Avoid):**

```text
❌ DON'T DO THIS:
CREATE INDEX ON photos(camera_make);

Query: SELECT * FROM photos WHERE camera_make = 'Canon';

Problem:
- Queries ALL nodes in cluster!
- 12 nodes × 100ms = 1.2 seconds
- Doesn't scale

✅ DO THIS INSTEAD:
Store in Elasticsearch:
{
  "photo_id": "...",
  "camera_make": "Canon",
  ...
}

Query Elasticsearch, get photo_ids, fetch from Cassandra
- Elasticsearch query: 30ms
- Cassandra fetch (by partition key): 10ms
- Total: 40ms (30x faster!)
```

**Milvus HNSW Index Configuration:**

```text
Index Parameters:
- Index type: HNSW (Hierarchical Navigable Small World)
- M: 16 (connections per layer)
- ef_construction: 200 (index build quality)
- ef_search: 100 (query accuracy vs speed)

Trade-offs:
M = 8:  Fast build, less accurate search
M = 16: Balanced (Google Photos uses this)
M = 32: Slow build, very accurate search

Performance:
- 2.7 trillion faces
- Partitioned per user (avg 10K faces/user)
- HNSW search: 50ms for top-100 similar faces
- Accuracy: 99%+ recall@100
```

#### Data Consistency Patterns

**Photo Upload Consistency:**

```text
Upload Flow (Multi-Database Write):

1. Upload photo → S3
   Status: Durable (11 9's)
   
2. Write metadata → Cassandra
   Status: Async replication (eventual)
   
3. Index → Elasticsearch
   Status: Near real-time (1-2 sec delay)
   
4. Face embedding → Milvus
   Status: Background job (minutes later)

Consistency Model: BASE (not ACID)
- Basically Available
- Soft state
- Eventually consistent

Why it's OK:
- User sees "Upload successful" after step 1
- Steps 2-4 happen in background
- If step 2 fails, retry from queue
- Idempotent writes (safe to retry)
```

**Share Permission Consistency (Strong):**

```text
Share Album Flow:

1. Begin transaction (PostgreSQL)
2. INSERT INTO sharing (album_id, shared_user_id, permission)
3. Update album SET is_shared = TRUE
4. Commit transaction

All-or-nothing:
- Either both writes succeed, or both fail
- No partial state possible
- Slower (50ms vs 10ms for Cassandra)
- Worth it for security!

Cache invalidation:
- On commit, delete cache key: share_permissions:{share_link}
- Next read gets fresh data
- Prevents showing wrong permissions
```

### Real-World Example: Google Photos Database Evolution

#### 2011-2015: Monolithic MySQL (Google+ Photos)

```text
Single MySQL Database:
├─ Users table (100M rows)
├─ Photos table (10B rows) ← Bottleneck!
├─ Albums table (500M rows)
├─ Comments table (5B rows)
└─ Shares table (1B rows)

Problems:
- Photos table too large (write throughput: 1K/sec max)
- Complex queries slow (JOINs across billions of rows)
- Hard to scale (vertical scaling only)
- Single point of failure

Result: Couldn't scale past 300M users
```

#### 2015-2020: Polyglot Persistence (Modern Google Photos)

```text
Separated by Access Pattern:

PostgreSQL (Relational):
├─ Users (100M rows)
├─ Albums (500M rows)
└─ Shares (1B rows)
Size: 500 GB
Write rate: 1K/sec
Read rate: 50K/sec

Cassandra (Time-Series):
├─ Photos metadata (4T rows)
└─ Videos metadata (200B rows)
Size: 3 PB
Write rate: 17K/sec
Read rate: 230K/sec

Elasticsearch (Search):
├─ Photos index (4T docs)
└─ Searchable fields only
Size: 500 GB (compressed)
Query rate: 10K/sec
Latency: < 50ms

Milvus (Vectors):
├─ Face embeddings (2.7T vectors)
Size: 4 PB (with HNSW index)
Search rate: 5K/sec
Latency: < 100ms

Result: Scaled to 1B users!
```

### 🎯 Interview Questions: Database Design

**Q1: Why partition Cassandra by (user_id, year_month) instead of just user_id?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Problem with user_id only:**
```text
Power user with 100K photos:
- All 100K in single partition
- Partition size: 100K × 1KB = 100 MB
- Cassandra limit: 100MB recommended, 2GB max
- Near the limit!
- Slow queries (scan 100K rows)
```

**Solution: (user_id, year_month):**
```text
Same power user, split by month:
- 2025-01: 500 photos (500 KB)
- 2024-12: 450 photos (450 KB)
- 2024-11: 600 photos (600 KB)
...
- Each partition small and fast!
```

**Query Pattern:**
```text
"Get photos from last 30 days"
WHERE user_id = 'john123' 
  AND year_month IN ('2025-01', '2024-12')
  AND upload_date >= '2024-12-27'

→ Query 2 partitions only
→ Return 500 + 450 = 950 photos
→ Fast even for power users!
```

**Trade-offs:**
- ✅ Prevents hot partitions
- ✅ Faster queries (smaller partitions)
- ✅ Better distribution across nodes
- ❌ More complex query logic (calculate year_month)
- ❌ Can't query across years efficiently (but rare!)

**Interview Tip:** Shows you understand Cassandra partition size limits and query patterns!
</details>

**Q2: How do you handle eventual consistency in the upload flow?**

<details>
<parameter name="answer">

**Answer:**

**The Inconsistency Window:**
```text
T=0: User uploads photo
T=0.1s: Written to S3 (durable)
T=0.2s: Metadata written to Cassandra
T=0.5s: Replicated to 2 more Cassandra nodes
T=1.0s: Indexed in Elasticsearch
T=30s: Face detection complete
T=60s: Face embedding in Milvus

During this time:
- User sees "Upload successful" (T=0.1s)
- Photo not in search yet (T=0 to T=1s)
- Face search doesn't find it (T=0 to T=60s)
```

**Handling Strategies:**

**1. Optimistic UI:**
```text
Client-side:
- Show photo immediately in user's library
- Mark as "Processing..."
- Don't wait for background jobs

User sees instant feedback!
```

**2. Read-Your-Own-Writes:**
```text
After upload, return:
{
  "photo_id": "abc123",
  "status": "processing",
  "uploaded_at": "2025-01-15T10:30:00Z"
}

Client stores this locally
Shows in UI even before Cassandra replication completes
```

**3. Retry Logic:**
```text
If Elasticsearch indexing fails:
1. Kafka retry queue
2. Retry 3 times with exponential backoff
3. If still fails, alert ops team
4. User photo still viewable (in Cassandra)
5. Just not searchable yet
```

**4. Status Tracking:**
```text
Processing Status Table (PostgreSQL):
photo_id | thumbnail_done | search_indexed | faces_detected
---------|----------------|----------------|----------------
abc123   | true           | true           | false

Show user: "Still processing faces... 90% complete"
```

**5. Cache Invalidation:**
```text
After upload:
- Invalidate: user_photos:{user_id}:page1
- Next view fetches fresh data from Cassandra
- Ensures user sees their new photo
```

**Interview Tip:** This shows you understand distributed systems reality - perfect consistency is expensive, eventual consistency with good UX is the pragmatic choice!
</details>

---

### 🤔 Think About It

1. **Sharding Strategy:** If user_id distribution is uneven (some users have 100K photos, most have 100), how would you prevent hot spots in your sharding scheme?

2. **Cross-Database Transactions:** What happens if photo is written to S3 and Cassandra, but Elasticsearch indexing fails? How do you ensure data doesn't become inconsistent?

3. **Database Choice:** Could you use just PostgreSQL for everything? What would break first at Google Photos scale?

---

### ✅ Key Takeaways

```text
✓ Use 5 different databases: PostgreSQL, Cassandra, Elasticsearch, Milvus, Redis
✓ PostgreSQL for relational data (users, albums) - ACID transactions needed
✓ Cassandra for time-series data (photos) - massive writes, partition by user_id + date
✓ Elasticsearch for search - text search 100x faster than SQL LIKE queries
✓ Milvus for vectors (faces) - billion-scale similarity search in < 100ms
✓ Redis for caching - 95% cache hit rate saves millions in DB costs
✓ Partition key determines query performance in Cassandra
✓ Design Cassandra schema for your queries, not for normalization
✓ Index only frequently queried fields - each index costs storage & write performance
✓ Eventual consistency OK for photos, strong consistency critical for permissions
✓ Multi-region with LOCAL_QUORUM for low latency & high availability
```

**Critical Database Decisions:**
```text
1. Polyglot persistence → Right tool for each job (30x faster than single DB)
2. Cassandra partitioning → (user_id, year_month) prevents hot spots
3. Milvus partitioning by user → Privacy + 200x faster face search
4. Redis caching → 95% hit rate = $50M/year savings
5. Eventual consistency → Better UX (instant upload) vs strong consistency (slower)
```

---

### 🎯 Practice Exercise

**Exercise: Design Schema for Instagram**

Instagram has similar requirements but different patterns:
- Users follow other users (social graph)
- Feed shows posts from people you follow (not all your posts)
- Comments and likes on posts
- Stories (temporary content, 24 hour expiration)

Design:
1. What tables would you create?
2. Which database for each table?
3. How would you partition the Posts table in Cassandra?
4. What indexes would you create?

<details>
<summary>Click to see sample answer</summary>

**Sample Answer:**

**PostgreSQL Tables:**
```sql
-- Users & Social Graph
users (user_id, username, email, bio, avatar_url)
follows (follower_id, following_id, created_at)
  INDEX: (follower_id, created_at) -- "Who I follow"
  INDEX: (following_id, created_at) -- "My followers"
```

**Cassandra Tables:**
```text
-- Posts (optimized for feed generation)
posts_by_user:
  Partition: user_id
  Clustering: created_at DESC, post_id
  → Fast: "Get user's posts"

posts_by_follower (feed table):
  Partition: follower_id
  Clustering: created_at DESC, post_id
  → Fast: "Get my feed" (fan-out on write)
  
posts_metadata:
  Partition: post_id
  → Photo URL, caption, location, likes count
```

**Redis (Stories with TTL):**
```text
story:{user_id} → JSON, TTL: 24 hours
active_stories → Sorted set (by expiry time)
```

**Key Differences from Google Photos:**
- Social graph needed (follows table)
- Feed table (fan-out pattern)
- Stories with auto-expiration (Redis TTL)
- More emphasis on real-time (WebSockets for likes)

**Partitioning:**
```text
posts_by_follower partition size issue:
- Kylie Jenner: 400M followers
- Each post fans out to 400M feed entries!
- Solution: Hybrid (celebrities use fan-out on read)
```
</details>

---

**Ready for Section 5?** Next, we'll design the complete API for Google Photos - endpoints for upload, view, search, sharing, and face recognition. You'll learn RESTful design patterns for media platforms!

---

## Section 5: How Users Interact - API Design

### What You'll Learn

By the end of this section, you'll be able to:
- Design RESTful APIs for media upload, retrieval, and management
- Structure endpoints for scalability and clear responsibility boundaries
- Implement authentication and authorization for photo access control
- Handle file uploads with chunking, resumability, and validation
- Design search and face recognition APIs with privacy considerations

### Why This Matters

Well-designed APIs are the contract between your frontend and backend - they determine developer experience and system evolvability. Real-world example: Instagram's API v1 was so poorly designed (no pagination limits, no rate limiting) that third-party apps crashed their servers. They had to deprecate it entirely and release v2 with proper pagination, rate limiting, and versioning. A good API design from day one saves years of technical debt!

---

### 🟢 For Beginners: The Fundamentals

#### What is an API?

Think of an API (Application Programming Interface) like a restaurant menu:

```text
🍽️ Restaurant Analogy:

Menu (API):
├─ Items you can order (endpoints)
├─ How to order (HTTP methods: GET, POST, PUT, DELETE)
├─ What you need to provide (parameters)
└─ What you'll get back (responses)

You don't need to know how the kitchen works!
You just order from the menu, and food arrives.

Same with APIs:
- Mobile app doesn't know how server stores photos
- Just calls /photos/upload with a file
- Server handles the complexity
- Returns success/failure
```

#### HTTP Methods (Verbs)

Different actions use different HTTP methods:

```text
GET - Read data (like asking "show me my photos")
  Example: GET /photos → Get list of user's photos

POST - Create new data (like "upload this photo")
  Example: POST /photos/upload → Upload new photo

PUT - Replace data (like "replace this photo")
  Example: PUT /photos/{id} → Replace entire photo

PATCH - Update part of data (like "change photo name only")
  Example: PATCH /photos/{id} → Update photo name

DELETE - Remove data (like "delete this photo")
  Example: DELETE /photos/{id} → Delete photo
```

#### Understanding Endpoints

Endpoints are like addresses for different features:

```text
Base URL: https://api.googlephotos.example.com/v1

Authentication:
  POST /auth/register → Create account
  POST /auth/login → Get access token
  POST /auth/logout → Invalidate token

Photos:
  POST /photos/upload → Upload new photo
  GET /photos → List my photos
  GET /photos/{photo_id} → Get specific photo details
  PATCH /photos/{photo_id} → Update photo (name, tags)
  DELETE /photos/{photo_id} → Delete photo

Albums:
  POST /albums → Create new album
  GET /albums → List my albums
  POST /albums/{album_id}/photos → Add photos to album
  DELETE /albums/{album_id} → Delete album

Search:
  GET /search/photos?query=beach → Search photos by text
  GET /search/location?lat=37.7&lon=-122.4 → Photos near location

People (Face Recognition):
  GET /people → List all people (face groups)
  GET /people/{person_id}/photos → Photos of this person
  PATCH /people/{person_id}/name → Name this person
```

#### What's in a Request?

When your app calls an API, it sends:

```text
1. URL: Where to send the request
   https://api.googlephotos.example.com/v1/photos

2. Method: What action to take
   GET (read), POST (create), etc.

3. Headers: Extra information
   Authorization: Bearer abc123... (your login token)
   Content-Type: application/json (format of data)

4. Body (for POST/PATCH): The actual data
   {
     "file_name": "beach.jpg",
     "tags": ["vacation", "beach"]
   }
```

#### What Comes Back (Response)?

The server responds with:

```text
1. Status Code: Did it work?
   200 OK - Success!
   201 Created - New thing created!
   400 Bad Request - You sent wrong data
   401 Unauthorized - You're not logged in
   404 Not Found - That photo doesn't exist
   500 Server Error - Oops, our bad!

2. Headers: Metadata
   Content-Type: application/json
   X-RateLimit-Remaining: 847 (requests left)

3. Body: The actual data
   {
     "photo_id": "abc123",
     "file_name": "beach.jpg",
     "thumbnail_url": "https://cdn.../photo.jpg"
   }
```

#### Example: Uploading a Photo (Simple Version)

```text
Step 1: App prepares request
  URL: POST https://api.googlephotos.example.com/v1/photos/upload
  Headers: 
    Authorization: Bearer <your_token>
    Content-Type: multipart/form-data
  Body:
    photo: [binary file data]
    file_name: "vacation.jpg"

Step 2: Server receives and processes
  - Checks: Are you logged in? (token valid?)
  - Checks: Is file size OK? (< 100MB?)
  - Uploads to S3
  - Creates thumbnail
  - Saves metadata to database

Step 3: Server responds
  Status: 201 Created
  Body:
    {
      "photo_id": "abc123",
      "upload_date": "2025-10-27T10:00:00Z",
      "status": "processing",
      "thumbnail_url": "https://cdn.../abc123_thumb.jpg"
    }

Step 4: App shows success
  "Photo uploaded! Processing..."
```

💡 **Pro Tip:** Always check the status code FIRST before looking at response body. Status code tells you if request succeeded or failed!

---

### 🟡 For Intermediate: Interview Patterns

#### RESTful API Design Principles

**1. Resource-Based URLs (Not Action-Based)**

```text
❌ Bad (action-based):
POST /uploadPhoto
POST /deletePhoto
GET /getUserPhotos

✅ Good (resource-based):
POST /photos (upload)
DELETE /photos/{id} (delete)
GET /photos (get user's photos)

Why better:
- Clear hierarchy
- HTTP methods convey action
- Easier to understand
```

**2. Consistent Naming Conventions**

```text
✅ Best Practices:
- Plural nouns: /photos, /albums, /users (not /photo)
- Lowercase: /photos/123 (not /Photos/123)
- Hyphens for multiple words: /shared-albums (not /sharedAlbums)
- No trailing slashes: /photos (not /photos/)
```

**3. Versioning Strategy**

```text
Why version APIs?
- Can't break existing apps
- Need to evolve features
- Deprecate old endpoints gradually

Methods:
1. URL path: /v1/photos, /v2/photos ✅ (Google Photos uses this)
2. Header: Accept: application/vnd.api.v2+json
3. Query param: /photos?version=2

Google Photos choice: URL path
- Clear and explicit
- Easy for developers
- Can run multiple versions simultaneously
```

#### Authentication Flow Design

**OAuth 2.0 with JWT Tokens:**

```text
Registration/Login Flow:

1. Register:
   POST /auth/register
   Body: {email, password, username}
   Response: {access_token, refresh_token}

2. Login:
   POST /auth/login
   Body: {email, password}
   Response: {access_token, refresh_token}

3. Use API:
   GET /photos
   Header: Authorization: Bearer <access_token>

4. Token Expires (after 1 hour):
   POST /auth/refresh
   Body: {refresh_token}
   Response: {new_access_token}

5. Logout:
   POST /auth/logout
   Header: Authorization: Bearer <access_token>
   Server blacklists token
```

**JWT Token Structure:**

```text
Token: eyJhbGciOiJIUzI1NiIs.eyJ1c2VyX2lkIjoi.SflKxwRJSMeKKF2Q

Decoded:
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "user@example.com",
    "tier": "premium",
    "exp": 1730124000,  // Expiry timestamp
    "iat": 1730120400   // Issued at
  },
  "signature": "..." // Signed with secret key
}

Why JWT?
- Stateless (no server-side session storage)
- Self-contained (includes user info)
- Can verify without database lookup
- Scales well (no session store bottleneck)
```

#### Pagination Strategy

**Problem:** User has 50,000 photos. Can't return all at once!

**Solution: Offset-Based Pagination**

```text
Request:
GET /photos?page=1&page_size=50

Response:
{
  "photos": [... 50 photos ...],
  "pagination": {
    "current_page": 1,
    "page_size": 50,
    "total_items": 3456,
    "total_pages": 70,
    "has_next": true,
    "has_previous": false
  }
}

Pros:
- Simple to implement
- Can jump to any page
- Users expect it (page numbers)

Cons:
- Performance degrades with deep pages (page 1000)
- Inconsistent if data changes (insertions/deletions)
```

**Alternative: Cursor-Based Pagination**

```text
Request:
GET /photos?limit=50&cursor=abc123

Response:
{
  "photos": [... 50 photos ...],
  "next_cursor": "def456",
  "has_more": true
}

Pros:
- Consistent even if data changes
- Better performance for deep pagination
- Used by Facebook, Twitter

Cons:
- Can't jump to specific page
- Harder to implement
```

**Google Photos Choice:** Offset-based for photos (stable dataset), cursor for real-time feeds (future feature)

#### Rate Limiting Design

**Why rate limit?**
- Prevent abuse
- Fair resource allocation
- DDoS protection
- Cost control

**Strategy:**

```text
Tiers:
- Free tier: 1,000 requests/hour
- Premium tier: 10,000 requests/hour
- Special limit for uploads: 100 uploads/hour (regardless of tier)

Headers returned with every response:
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1730127600 (Unix timestamp)

When limit exceeded:
Status: 429 Too Many Requests
Body:
{
  "error": "rate_limit_exceeded",
  "message": "You have exceeded 1000 requests/hour",
  "retry_after": 3600  // seconds
}
```

**Implementation:**

```text
Algorithm: Token Bucket
- Start with 1000 tokens
- Each request consumes 1 token
- Tokens refill at 1000 per hour
- If no tokens left, request denied

Storage: Redis
Key: rate_limit:{user_id}
Value: {tokens_remaining, last_refill_time}
TTL: 1 hour
```

#### File Upload API Design

**Problem:** Photos are large (3MB average, up to 100MB)

**Solution: Multipart Upload**

```text
POST /photos/upload
Content-Type: multipart/form-data

Request Body:
--boundary123
Content-Disposition: form-data; name="photo"; filename="beach.jpg"
Content-Type: image/jpeg

[binary file data]
--boundary123
Content-Disposition: form-data; name="file_name"

beach.jpg
--boundary123
Content-Disposition: form-data; name="tags"

vacation, beach, summer
--boundary123--
```

**Advanced: Chunked Upload (for large files)**

```text
Step 1: Initiate upload
POST /photos/upload/initiate
Response: {upload_id: "abc123", chunk_size: 5MB}

Step 2: Upload chunks
POST /photos/upload/abc123/chunk/1
Body: [5MB of data]

POST /photos/upload/abc123/chunk/2
Body: [5MB of data]

...

Step 3: Complete upload
POST /photos/upload/abc123/complete
Response: {photo_id: "xyz789", status: "processing"}

Benefits:
- Resume if network fails (just re-upload failed chunk)
- Parallel chunk uploads (faster)
- Progress tracking (show %)
```

#### Search API Design

**Text Search:**

```text
GET /search/photos?query=beach&start_date=2025-01-01&end_date=2025-12-31

Query parameters:
- query (string): Text to search
- start_date (ISO 8601): Filter by date range
- end_date (ISO 8601)
- location (string): Location name
- camera_make (string): Filter by camera brand
- tags (array): Filter by tags
- page, page_size: Pagination

Response:
{
  "results": [
    {
      "photo_id": "...",
      "thumbnail_url": "...",
      "capture_date": "...",
      "relevance_score": 0.95  // How well it matches query
    }
  ],
  "total_results": 127,
  "search_time_ms": 45  // How long search took
}
```

**Location-Based Search:**

```text
GET /search/location?latitude=37.7749&longitude=-122.4194&radius_km=10

Returns photos within 10km of San Francisco coordinates

Response:
{
  "results": [
    {
      "photo_id": "...",
      "distance_km": 2.5,  // How far from search center
      "location_name": "Golden Gate Bridge"
    }
  ],
  "center": {
    "latitude": 37.7749,
    "longitude": -122.4194
  }
}
```

#### Error Handling Design

**Consistent Error Format:**

```json
{
  "error": "error_code",
  "message": "Human-readable message for developers",
  "details": {
    "field": "email",
    "reason": "Email already registered"
  },
  "request_id": "req-550e8400",  // For support/debugging
  "timestamp": "2025-10-27T10:00:00Z"
}
```

**HTTP Status Codes:**

```text
2xx Success:
200 OK - Request succeeded
201 Created - Resource created
202 Accepted - Async processing started
204 No Content - Succeeded, no response body

4xx Client Errors:
400 Bad Request - Invalid data format
401 Unauthorized - Missing/invalid token
403 Forbidden - Valid token, but no permission
404 Not Found - Resource doesn't exist
409 Conflict - Duplicate resource
413 Payload Too Large - File too big
429 Too Many Requests - Rate limit exceeded

5xx Server Errors:
500 Internal Server Error - Something broke on our end
503 Service Unavailable - Temporary outage
```

---

### 🔴 For Advanced: Production Considerations

#### API Versioning Strategy

**Version Lifecycle Management:**

```text
Version Timeline:

v1 (2025-01-01): Initial release
├─ Active development
├─ New features added
└─ Bug fixes

v2 (2026-01-01): Major update
├─ Breaking changes allowed
├─ v1 enters maintenance mode
└─ v1 still fully supported

v1 Deprecation Notice (2026-06-01):
├─ Warning headers added to v1 responses
├─ Email notifications to developers
└─ Migration guide published

v1 End of Life (2027-01-01):
├─ v1 stopped
├─ Returns 410 Gone
└─ All clients must migrate to v2
```

**Breaking vs Non-Breaking Changes:**

```text
Non-Breaking (can add to same version):
✅ Add new endpoints
✅ Add optional fields to request
✅ Add new fields to response
✅ Make required field optional

Breaking (need new version):
❌ Remove endpoints
❌ Remove fields from response
❌ Change field types (string → integer)
❌ Make optional field required
❌ Change URL structure
```

#### Idempotency for Upload APIs

**Problem:** Network issues cause duplicate uploads

**Solution: Idempotency Keys**

```text
Request:
POST /photos/upload
Headers:
  Authorization: Bearer <token>
  Idempotency-Key: unique-key-12345

Server behavior:
1. First request with key "unique-key-12345":
   - Processes upload normally
   - Stores result with key
   - Returns 201 Created

2. Duplicate request (same key):
   - Detects duplicate
   - Returns cached result from first request
   - Returns 200 OK (not 201)

Storage:
Key: idempotency:{key}
Value: {status, photo_id, response_body}
TTL: 24 hours

Benefits:
- Safe retries (client can retry without creating duplicates)
- Network failures handled gracefully
- Exactly-once semantics
```

#### API Gateway Configuration

**Rate Limiting per Endpoint:**

```yaml
endpoints:
  /auth/login:
    rate_limit: 5 requests/minute  # Prevent brute force
    burst: 0  # No burst allowed
  
  /photos/upload:
    rate_limit: 100 uploads/hour
    burst: 10  # Allow 10 rapid uploads
  
  /photos:
    rate_limit: 1000 requests/hour
    burst: 50
  
  /search/photos:
    rate_limit: 500 requests/hour  # More expensive operation
    burst: 20
```

**Request/Response Transformation:**

```yaml
# Strip sensitive data from responses
response_filters:
  - remove_field: user.password_hash
  - remove_field: user.internal_id
  - add_header: X-Response-Time

# Add common headers
response_headers:
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Strict-Transport-Security: max-age=31536000
```

**API Analytics:**

```yaml
metrics:
  - endpoint_latency (P50, P95, P99)
  - error_rate (by status code)
  - requests_per_second
  - payload_size
  - cache_hit_ratio

logging:
  - request_id (for tracing)
  - user_id
  - endpoint
  - duration_ms
  - status_code
  - user_agent
```

#### GraphQL vs REST Trade-off

**Why Google Photos chose REST:**

```text
REST Advantages:
✅ Simpler implementation
✅ Better caching (HTTP caching works naturally)
✅ Wider client support
✅ Predictable performance (know exactly what each endpoint does)
✅ Easier to version
✅ Better for file uploads

REST Disadvantages:
❌ Multiple requests for related data
❌ Over-fetching (get full object when need one field)
❌ Under-fetching (need multiple requests)

GraphQL Advantages:
✅ Single request for complex queries
✅ Fetch exactly what you need
✅ Strongly typed schema

GraphQL Disadvantages:
❌ Complexity (need GraphQL server, client libs)
❌ Harder to cache
❌ Unpredictable performance (complex queries can be slow)
❌ File upload more complex

For Google Photos:
- Access patterns are predictable (photos list, album view, search)
- Caching is critical (95% CDN hit rate)
- File uploads are core feature (REST handles better)
- REST is simpler for mobile apps

→ REST is the right choice
```

#### API Security Best Practices

**1. Input Validation:**

```python
def validate_photo_upload(request):
    # File size
    if request.file.size > 100 * 1024 * 1024:  # 100MB
        raise ValidationError("File too large")
    
    # File type (magic number check, not extension)
    file_header = request.file.read(12)
    if not is_valid_image_header(file_header):
        raise ValidationError("Invalid file type")
    
    # Sanitize filename
    filename = secure_filename(request.filename)
    if len(filename) > 255:
        raise ValidationError("Filename too long")
    
    # SQL injection prevention (use parameterized queries)
    # XSS prevention (sanitize all text inputs)
    tags = [sanitize_html(tag) for tag in request.tags]
    
    return validated_data
```

**2. Rate Limiting by Multiple Dimensions:**

```text
Per User: 1000 req/hour
Per IP: 10,000 req/hour (multiple users behind NAT)
Per Endpoint: Different limits
Global: 10M req/hour (total system capacity)

DDoS Protection:
- CAPTCHA after failed logins
- Progressive backoff (1s, 2s, 4s, 8s delays)
- Temporary IP bans
```

**3. Sensitive Data Protection:**

```text
Never Return in API Responses:
❌ Password hashes
❌ Internal user IDs (use UUIDs)
❌ Raw face embeddings (privacy)
❌ Full credit card numbers
❌ API secrets/keys

Encrypt in Transit:
✅ TLS 1.3 for all connections
✅ Certificate pinning for mobile apps

Encrypt at Rest:
✅ S3 server-side encryption
✅ Database encryption (Cassandra TDE)
✅ Encrypted backups
```

### Real-World Example: Instagram API Evolution

#### Instagram API v1 (2010-2015) - Mistakes

```text
Problems:
1. No pagination limits
   GET /users/{id}/media → Could return 100,000 photos!
   → Crashed servers, slow responses

2. No rate limiting initially
   → Third-party apps DDoS'd Instagram accidentally
   → Had to add emergency rate limits

3. Inconsistent naming
   /users/{id}/media vs /media/{id}
   → Confusing for developers

4. No versioning
   → Couldn't make breaking changes
   → Stuck with bad design forever

Result: Had to deprecate entire API and start over with v2!
```

#### Instagram API v2 (2015+) - Lessons Learned

```text
Improvements:
✅ Pagination: max 50 items, cursor-based
✅ Rate limiting: 200 calls/hour per user
✅ Versioning: /v2/ in URL
✅ Consistent naming: All plural nouns
✅ Webhooks: Push updates instead of polling
✅ Granular permissions: Different scopes for different access

Deprecated v1 over 2 years:
- 6 months warning
- Migration tools
- Dedicated support
- Graceful shutdown
```

### 🎯 Interview Questions: API Design

**Q1: Design an API for uploading very large video files (up to 10GB). How do you handle resume capability?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Chunked Upload with Resume:**

```text
Phase 1: Initiate
POST /videos/upload/initiate
Request: {
  "file_name": "vacation.mp4",
  "file_size_bytes": 10737418240,  // 10GB
  "mime_type": "video/mp4"
}
Response: {
  "upload_id": "abc123",
  "chunk_size_bytes": 10485760,  // 10MB chunks
  "total_chunks": 1024,
  "upload_urls": [...]  // Pre-signed S3 URLs
}

Phase 2: Upload Chunks (parallel)
PUT /videos/upload/abc123/chunks/0
Body: [10MB binary data]
Response: 200 OK

PUT /videos/upload/abc123/chunks/1
Body: [10MB binary data]
...

Phase 3: Check Progress (if resumed)
GET /videos/upload/abc123/status
Response: {
  "uploaded_chunks": [0, 1, 2, 5, 6, 7],  // Chunks completed
  "pending_chunks": [3, 4, 8, 9, ...],
  "progress_percentage": 0.7
}

Phase 4: Resume (upload missing chunks)
PUT /videos/upload/abc123/chunks/3
PUT /videos/upload/abc123/chunks/4
...

Phase 5: Complete
POST /videos/upload/abc123/complete
Request: {
  "chunk_checksums": {
    "0": "md5hash1",
    "1": "md5hash2",
    ...
  }
}
Response: {
  "video_id": "xyz789",
  "status": "processing"
}
```

**Why this works:**
- Client tracks which chunks uploaded
- If network fails, resume from last successful chunk
- Server validates all chunks received before completing
- Checksums prevent data corruption

**Interview Tip:** Shows you understand large file handling and network reliability issues!
</details>

**Q2: How would you design an API to handle 1 million concurrent photo uploads?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Async Upload Architecture:**

```text
Step 1: Get Upload URL (Fast)
POST /photos/upload/presigned-url
Response: {
  "upload_url": "https://s3.../unique-key",
  "photo_id": "temp-abc123",
  "expires_in": 900  // 15 minutes
}
Time: 10ms (just generates pre-signed URL)

Step 2: Client Uploads Directly to S3 (Client-side)
PUT https://s3.../unique-key
Body: [binary photo data]
Headers: Content-MD5: ...

Client doesn't wait for our servers!
S3 handles the load (auto-scales)

Step 3: S3 Triggers Lambda (Event-driven)
S3 Event → Lambda → Kafka → Processing Pipeline

Step 4: Client Polls for Status (Optional)
GET /photos/temp-abc123/status
Response: {
  "status": "processing|completed|failed",
  "thumbnail_url": "..." (if ready)
}

Why this scales to 1M uploads:
- API servers only generate URLs (1M QPS possible)
- S3 handles actual file storage (infinite scale)
- Processing happens asynchronously (not blocking uploads)
- No file data goes through API servers (save bandwidth)

Cost:
- API servers: ~100 instances (10K QPS each)
- S3: Unlimited, pay per GB
- Lambda: Serverless, scales automatically
```

**Interview Tip:** Shows you understand async patterns and separation of concerns!
</details>

---

### 🤔 Think About It

1. **API Versioning**: If you need to remove a field from API response, how would you do it without breaking existing clients? How long would the migration take?

2. **Rate Limiting**: Should rate limits be per user or per API key? What if one user has multiple devices/apps?

3. **Error Messages**: How detailed should error messages be? Security vs developer experience trade-off?

---

### ✅ Key Takeaways

```text
✓ RESTful design: Resource-based URLs, HTTP methods convey actions
✓ Versioning: URL path (/v1/, /v2/) for clarity and parallel versions
✓ Pagination: Offset-based for stable data, cursor for real-time feeds
✓ Authentication: OAuth 2.0 + JWT for stateless, scalable auth
✓ Rate limiting: Multiple tiers, per-endpoint limits, graceful degradation
✓ File uploads: Multipart for small files, chunked for large, pre-signed URLs for scale
✓ Error handling: Consistent format, meaningful status codes, request IDs for debugging
✓ Idempotency: For uploads, use idempotency keys to handle retries
✓ Security: Input validation, HTTPS, rate limiting, no sensitive data in responses
✓ Monitoring: Track latency, error rates, rate limit hits per endpoint
```

**API Design Principles:**
```text
1. Consistency: Same patterns across all endpoints
2. Simplicity: Easy to understand and use
3. Flexibility: Versioning allows evolution
4. Security: Defense in depth (auth, rate limit, validation)
5. Performance: Pagination, caching, async processing
```

---

### 🎯 Practice Exercise

**Exercise: Design Album Sharing API**

Design an API for sharing photo albums with specific users or via public links.

Requirements:
- Share album with specific users (by email)
- Create public share links
- Different permission levels (view, comment, edit)
- Revoke access
- Track who viewed

Design:
1. List all API endpoints needed
2. Define request/response formats
3. Consider security (permissions checking)
4. Handle edge cases (share with yourself, circular shares)

<details>
<parameter name="summary">Click to see sample answer

Design:
1. List all API endpoints needed
2. Define request/response formats
3. Consider security (permissions checking)
4. Handle edge cases (share with yourself, circular shares)

<details>
<summary>Click to see sample answer</summary>

**Sample Answer:**

**1. API Endpoints:**

```text
POST /albums/{album_id}/share/invite
  - Share album with specific user by email
  
POST /albums/{album_id}/share/link
  - Create public share link

GET /albums/{album_id}/shares
  - List all shares for this album

PATCH /albums/{album_id}/share/{share_id}
  - Update share permissions

DELETE /albums/{album_id}/share/{share_id}
  - Revoke share access

GET /shared/{share_code}
  - Access shared album via public link

POST /shared/{share_code}/view
  - Track view event (analytics)
```

**2. Request/Response Formats:**

```json
// Share with specific user
POST /albums/{album_id}/share/invite
{
  "email": "friend@example.com",
  "permission_level": "view",  // view, comment, edit
  "message": "Check out my vacation photos!"
}

Response:
{
  "share_id": "share-uuid",
  "album_id": "album-uuid",
  "shared_with": {
    "email": "friend@example.com",
    "user_id": "user-uuid"  // if user exists
  },
  "permission_level": "view",
  "created_at": "2025-10-27T10:00:00Z",
  "status": "pending"  // pending if user not registered
}

// Create public link
POST /albums/{album_id}/share/link
{
  "permission_level": "view",
  "allow_download": false,
  "require_password": true,
  "password": "secret123",
  "expires_at": "2025-12-31T23:59:59Z"
}

Response:
{
  "share_id": "share-uuid",
  "share_link": "https://photos.example.com/shared/abc123def",
  "short_code": "abc123def",
  "qr_code_url": "https://api../qr/abc123def.png",
  "permission_level": "view",
  "view_count": 0,
  "created_at": "2025-10-27T10:00:00Z"
}
```

**3. Security Considerations:**

```text
Permission Checking:
1. Verify requester owns album (for share creation)
2. Check permission level before allowing actions
3. Validate email domains (prevent spam)
4. Rate limit share creation (max 20/hour)

Access Control Matrix:
┌──────────────┬──────┬─────────┬──────┐
│ Action       │ View │ Comment │ Edit │
├──────────────┼──────┼─────────┼──────┤
│ View photos  │  ✓   │    ✓    │  ✓   │
│ Download     │  ✓   │    ✓    │  ✓   │
│ Add comment  │  ✗   │    ✓    │  ✓   │
│ Add photos   │  ✗   │    ✗    │  ✓   │
│ Delete photo │  ✗   │    ✗    │  ✗   │ (owner only)
└──────────────┴──────┴─────────┴──────┘
```

**4. Edge Cases:**

```text
Edge Case 1: Share with yourself
Solution: Return 400 Bad Request
{
  "error": "cannot_share_with_self",
  "message": "You already have full access to this album"
}

Edge Case 2: User already has access
Solution: Update permission level
{
  "share_id": "existing-share-uuid",
  "message": "User already has access. Permission updated to 'edit'",
  "previous_permission": "view",
  "new_permission": "edit"
}

Edge Case 3: Share link already viewed 10,000 times
Solution: Warn and optionally disable
{
  "warning": "share_link_highly_used",
  "view_count": 10247,
  "suggestion": "Consider creating new link for security"
}

Edge Case 4: Album deleted while shared
Solution: Soft delete, maintain shares for 30 days
{
  "status": "album_deleted",
  "message": "Album deleted by owner",
  "access_until": "2025-11-27T00:00:00Z"
}
```

**Why this design works:**
- Clear permission hierarchy (view < comment < edit < owner)
- Supports both registered users and public links
- Tracks analytics (view counts) for owners
- Security-first with rate limiting and validation
- Handles edge cases gracefully

</details>

---

**Ready for Section 6?** Next, we'll dive deep into the upload pipeline - how photos get into the system with chunking, validation, deduplication, and async processing!

---

## Section 6: Upload Pipeline - Getting Photos into the System

### What You'll Learn

By the end of this section, you'll be able to:
- Design chunked upload mechanisms for large files with resume capability
- Implement validation pipelines (file type, size, virus scanning)
- Build deduplication systems using perceptual hashing
- Create async processing pipelines with Kafka
- Handle upload failures gracefully with retry logic

### Why This Matters

The upload pipeline is the first impression of your system - if uploads are slow or fail frequently, users abandon the platform. Real-world example: Flickr (2005-2010) had a terrible upload experience - single-threaded uploads that failed if you closed your browser. Users would wait hours to upload vacation photos, only to lose everything when their laptop went to sleep! Modern systems like Google Photos upload 50 photos in parallel with automatic resume - this is why they won 1 billion users.

---

### 🟢 For Beginners: The Fundamentals

#### What Happens When You Upload a Photo?

Think of uploading a photo like mailing a package:

```text
📦 Mailing a Package (Traditional Upload):

1. Pack your item (prepare file)
2. Drive to post office (send over internet)
3. Wait in line (upload to server)
4. Get receipt (upload confirmation)
5. Package sorted and delivered (processing happens later)

Problem: If you drop package on way to post office, start over!

📦 Modern Upload (Google Photos):

1. Pack your item (prepare file)
2. Break into smaller boxes (chunk into 5MB pieces)
3. Send boxes separately (parallel upload)
4. If one box lost, just resend that box! (resume)
5. Post office assembles boxes (server merges chunks)
6. Get receipt when all boxes arrive
7. Delivery happens in background

Much better!
```

#### Why Chunk Large Files?

Problem with single upload:

```text
10MB Photo Upload:
├─ Time: 10 seconds on WiFi
├─ If connection drops at 9 seconds → Start over!
└─ Total time with 1 retry: 20 seconds ❌

Same Photo with Chunking (2MB chunks):
├─ Upload chunk 1: 2s ✓
├─ Upload chunk 2: 2s ✓
├─ Upload chunk 3: 2s ✓
├─ Upload chunk 4: (connection drops) ✗
├─ Reconnect
├─ Upload chunk 4: 2s ✓ (just this chunk!)
├─ Upload chunk 5: 2s ✓
└─ Total time: 10s + 2s retry = 12 seconds ✓

Chunking wins!
```

#### The Upload Journey

**Step 1: Client Prepares**
```text
User clicks "Upload" on 10 photos

Client app:
- Resizes photos for upload (optional: save bandwidth)
- Compresses if needed
- Generates unique ID for each photo
- Calculates checksums (MD5) to verify integrity
```

**Step 2: Request Upload Permission**
```text
Client → Server: "I want to upload 10 photos"

Server checks:
- Is user logged in? ✓
- Does user have storage space? ✓
- Is upload rate OK? ✓

Server → Client: "OK, here are pre-signed upload URLs for S3"
```

**Step 3: Upload to S3 (Direct)**
```text
Client → S3 directly (not through our servers!)

Benefits:
- Our API servers don't handle file data (save bandwidth)
- S3 scales infinitely
- S3 handles retries automatically
```

**Step 4: Notify Server**
```text
Client → Server: "Upload complete for photo XYZ"

Server:
- Verifies file exists in S3
- Creates metadata record in database
- Triggers background processing (thumbnails, ML)
```

**Step 5: Background Processing**
```text
Server publishes event to Kafka:
{
  "event": "photo.uploaded",
  "photo_id": "abc123",
  "user_id": "user456",
  "s3_location": "s3://bucket/user456/abc123.jpg"
}

Workers pick up and process:
- Generate thumbnails (3 sizes)
- Extract EXIF metadata
- Detect faces
- Index for search
```

💡 **Pro Tip:** Direct-to-S3 upload is key for scale. If 18,000 photos/sec go through API servers, bandwidth cost would be $1.5B/year! Direct upload: $0.

---

### 🟡 For Intermediate: Interview Patterns

#### Chunked Upload Implementation

**Initiate Upload:**
```text
POST /photos/upload/initiate
Request:
{
  "file_name": "beach.jpg",
  "file_size_bytes": 10485760,  // 10MB
  "content_type": "image/jpeg",
  "md5_checksum": "abc123def456..."
}

Response:
{
  "upload_id": "upload-uuid",
  "chunk_size_bytes": 5242880,  // 5MB chunks
  "total_chunks": 2,
  "upload_urls": [
    "https://s3.../chunk-0?signature=...",  // Pre-signed URL
    "https://s3.../chunk-1?signature=..."
  ],
  "expires_at": "2025-10-27T11:00:00Z"  // 15 min to complete
}
```

**Upload Chunks (Parallel):**
```text
Client uploads chunks in parallel:

Thread 1: PUT https://s3.../chunk-0
  Body: [First 5MB of file]
  Header: Content-MD5: chunk0hash
  
Thread 2: PUT https://s3.../chunk-1
  Body: [Last 5MB of file]
  Header: Content-MD5: chunk1hash

S3 returns: ETag for each chunk (needed for completion)
```

**Complete Upload:**
```text
POST /photos/upload/{upload_id}/complete
Request:
{
  "chunks": [
    {"chunk_number": 0, "etag": "etag0"},
    {"chunk_number": 1, "etag": "etag1"}
  ]
}

Server:
1. Verifies all chunks received (check S3)
2. Calls S3 CompleteMultipartUpload (merges chunks)
3. Saves metadata to Cassandra
4. Publishes to Kafka for processing
5. Returns photo_id to client

Response:
{
  "photo_id": "photo-uuid",
  "status": "processing",
  "estimated_completion": "2025-10-27T10:05:00Z"
}
```

#### Validation Pipeline

**Multi-Layer Validation:**

```text
Layer 1: Client-Side (Instant feedback)
├─ File size < 100MB
├─ File type is image (extension check)
├─ Basic format validation
└─ Fast fail before upload starts

Layer 2: Server-Side (Before accepting upload)
├─ User has storage quota
├─ User hasn't exceeded rate limit
├─ File size matches declared size
└─ Content-Type is allowed

Layer 3: Post-Upload (After file in S3)
├─ Magic number check (real file type)
├─ Image parseable (not corrupted)
├─ Virus scanning
├─ Content moderation (NSFW filter)
└─ Perceptual hash (duplicate detection)

Layer 4: Processing (Async)
├─ Full image validation
├─ EXIF extraction
├─ Dimension limits (max 50,000 x 50,000 px)
└─ Format conversion if needed
```

**Magic Number Validation:**
```python
# Detect real file type (users can rename .exe to .jpg!)
def validate_file_type(file_path):
    """
    Read file header (magic numbers) to verify type
    """
    magic_numbers = {
        'JPEG': [b'\xFF\xD8\xFF\xE0', b'\xFF\xD8\xFF\xE1'],
        'PNG':  [b'\x89\x50\x4E\x47'],
        'GIF':  [b'\x47\x49\x46\x38'],
        'WEBP': [b'\x52\x49\x46\x46']  # RIFF container
    }
    
    with open(file_path, 'rb') as f:
        header = f.read(4)
    
    for file_type, signatures in magic_numbers.items():
        if any(header.startswith(sig) for sig in signatures):
            return file_type
    
    raise ValidationError("Unsupported file type")
```

#### Deduplication Strategy

**Why Deduplicate?**
- Users upload same photo from phone, tablet, computer
- Saves 20-30% storage at Google Photos scale
- Faster "upload" (instant if already have it)

**Perceptual Hashing:**
```python
def generate_perceptual_hash(image_path):
    """
    pHash: Generates similar hash for similar images
    - Resized version: Same hash
    - Cropped slightly: Similar hash
    - Different photo: Very different hash
    """
    img = Image.open(image_path)
    
    # Resize to 32x32 (normalize size)
    img = img.resize((32, 32), Image.LANCZOS)
    
    # Convert to grayscale
    img = img.convert('L')
    
    # Compute DCT (Discrete Cosine Transform)
    pixels = np.array(img).flatten()
    dct = scipy.fft.dct(pixels)
    
    # Use low frequencies (8x8 = 64 values)
    dct_low = dct[:64]
    
    # Hash: 1 if above median, 0 if below
    median = np.median(dct_low)
    hash_bits = (dct_low > median).astype(int)
    
    # Convert to hex string
    hash_value = ''.join(str(b) for b in hash_bits)
    return int(hash_value, 2)  # 64-bit hash
```

**Deduplication Flow:**
```text
Upload Request:
1. Client calculates perceptual hash locally
2. Sends hash with upload initiation

POST /photos/upload/initiate
{
  "file_name": "beach.jpg",
  "perceptual_hash": "a1b2c3d4e5f6",
  ...
}

Server checks:
SELECT photo_id FROM dedup_hashes 
WHERE user_id = ? AND perceptual_hash = ?

If match found:
- Return existing photo_id immediately
- No upload needed!
- Save user bandwidth
- Instant "upload"

If no match:
- Proceed with normal upload
- Store hash after upload completes
```

**Hamming Distance for Near-Duplicates:**
```text
Hash A: 1010101010101010
Hash B: 1010101110101010
              ^^
Hamming distance: 1 (1 bit different)

If distance < 5 bits: Probably same photo
If distance < 10 bits: Similar photo (cropped, filtered)
If distance > 15 bits: Different photo
```

#### Async Processing with Kafka

**Event Flow:**
```text
1. Photo Upload Complete
   ↓
2. API Server publishes to Kafka
   Topic: photo.uploaded
   Partition: hash(user_id) % num_partitions
   
3. Multiple Consumer Groups
   ├─ Thumbnail Generator (consumes event)
   ├─ Metadata Extractor (consumes event)
   ├─ Face Detector (consumes event)
   └─ Search Indexer (consumes event)

All process in parallel!
```

**Event Schema:**
```json
{
  "event_type": "photo.uploaded",
  "event_id": "event-uuid",
  "timestamp": "2025-10-27T10:00:00Z",
  "photo_id": "photo-uuid",
  "user_id": "user-uuid",
  "s3_bucket": "photos-production",
  "s3_key": "users/user-uuid/2025/10/photo-uuid.jpg",
  "file_size_bytes": 3145728,
  "content_type": "image/jpeg",
  "metadata": {
    "capture_date": "2025-09-15T14:30:00Z",
    "latitude": 37.7749,
    "longitude": -122.4194
  }
}
```

**Consumer Implementation:**
```python
def thumbnail_consumer():
    """
    Kafka consumer for generating thumbnails
    """
    consumer = KafkaConsumer(
        'photo.uploaded',
        group_id='thumbnail-generator',
        enable_auto_commit=False  # Manual commit after processing
    )
    
    for message in consumer:
        try:
            event = json.loads(message.value)
            photo_id = event['photo_id']
            s3_key = event['s3_key']
            
            # Download from S3
            photo_data = s3.get_object(
                Bucket=event['s3_bucket'],
                Key=s3_key
            )
            
            # Generate thumbnails
            thumbnails = generate_thumbnails(
                photo_data,
                sizes=[150, 400, 1080]
            )
            
            # Upload thumbnails to S3
            for size, thumb_data in thumbnails.items():
                s3.put_object(
                    Bucket='thumbnails-bucket',
                    Key=f'{photo_id}_{size}.jpg',
                    Body=thumb_data
                )
            
            # Update metadata
            update_thumbnail_urls(photo_id, thumbnails)
            
            # Commit offset (mark as processed)
            consumer.commit()
            
        except Exception as e:
            logger.error(f"Thumbnail generation failed: {e}")
            # Don't commit - will retry
            # Dead letter queue after 3 retries
```

#### Resume Capability

**Client-Side State Management:**
```text
Client stores upload state locally:
{
  "upload_id": "upload-uuid",
  "total_chunks": 10,
  "completed_chunks": [0, 1, 2, 5, 6],
  "pending_chunks": [3, 4, 7, 8, 9],
  "chunk_etags": {
    "0": "etag0",
    "1": "etag1",
    ...
  }
}

On resume:
1. Check which chunks completed
2. Only upload pending chunks
3. Complete upload when all done
```

**Server-Side Resume API:**
```text
GET /photos/upload/{upload_id}/status

Response:
{
  "upload_id": "upload-uuid",
  "status": "in_progress",
  "total_chunks": 10,
  "completed_chunks": [0, 1, 2, 5, 6],
  "expires_at": "2025-10-27T11:00:00Z",
  "upload_urls": {
    "3": "https://s3.../chunk-3?sig=...",
    "4": "https://s3.../chunk-4?sig=...",
    ...
  }
}

Client resumes from chunk 3
```

---

### 🔴 For Advanced: Production Considerations

#### Upload Optimization Techniques

**1. Parallel Chunk Uploads:**
```text
Strategy: Upload multiple chunks simultaneously

Single-threaded upload (10MB, 5 chunks):
Chunk 0: 2s
Chunk 1: 2s
Chunk 2: 2s
Chunk 3: 2s
Chunk 4: 2s
Total: 10s

Parallel upload (5 threads):
Chunks 0-4: 2s (all at once)
Total: 2s

5x faster!

Implementation:
- Mobile: 3-5 parallel uploads (don't saturate cellular)
- Desktop: 10-20 parallel uploads (faster connections)
- Auto-adjust based on connection speed
```

**2. Adaptive Chunk Size:**
```text
Connection Speed-Based Chunking:

Slow (< 1 Mbps): 1MB chunks
  - Smaller chunks resume faster
  - Less wasted bandwidth on failure

Medium (1-10 Mbps): 5MB chunks
  - Balance between resume and overhead

Fast (> 10 Mbps): 10MB chunks
  - Reduce number of requests
  - Less overhead

Implement:
- Measure upload speed during first chunk
- Adjust chunk size for remaining chunks
- Google Photos uses this!
```

**3. Upload Compression:**
```text
Client-Side Compression:

Original JPEG: 3MB
├─ Already compressed format
├─ Re-compressing saves little
└─ Don't compress (waste CPU)

PNG Screenshot: 5MB
├─ Lossless format (large)
├─ Convert to JPEG (quality 85)
├─ New size: 500KB
└─ Save 90% bandwidth!

Smart compression:
if file_type == "PNG" and size > 2MB:
    compress_to_jpeg(quality=85)
elif file_type == "HEIC":
    convert_to_jpeg()  # Better compatibility
else:
    upload_as_is()
```

#### Virus Scanning Integration

**ClamAV Integration:**
```python
def scan_uploaded_file(s3_bucket, s3_key):
    """
    Scan file for viruses before processing
    """
    # Lambda function triggered by S3 upload
    
    # Download file
    obj = s3.get_object(Bucket=s3_bucket, Key=s3_key)
    file_data = obj['Body'].read()
    
    # Scan with ClamAV
    scanner = clamd.ClamdUnixSocket()
    result = scanner.scan_stream(file_data)
    
    if result['stream'][0] == 'FOUND':
        # Virus detected!
        virus_name = result['stream'][1]
        
        # Delete file immediately
        s3.delete_object(Bucket=s3_bucket, Key=s3_key)
        
        # Notify user
        notify_user_malicious_file(user_id, virus_name)
        
        # Log for security team
        security_alert(user_id, s3_key, virus_name)
        
        return False
    
    # Clean - proceed with processing
    return True
```

**Scanning Architecture:**
```text
S3 Upload
   ↓
Lambda (Virus Scan)
   ├─ Clean → Continue processing
   └─ Infected → Delete + Alert

Async scanning (non-blocking):
- User sees "Upload successful" immediately
- Scanning happens in background
- If infected, delete and notify later
- 99.99% of uploads are clean

Batch scanning for cost:
- Scan 100 files at once
- ClamAV running on EC2 (cheaper than Lambda for volume)
- Trade-off: Slight delay vs cost
```

#### Error Handling & Retry Logic

**Retry Strategy:**
```text
Exponential Backoff with Jitter:

Attempt 1: Immediate
Attempt 2: Wait 1s + random(0-1s)
Attempt 3: Wait 2s + random(0-2s)
Attempt 4: Wait 4s + random(0-4s)
Attempt 5: Wait 8s + random(0-8s)
Give up after 5 attempts

Jitter prevents thundering herd:
- 1000 clients fail at same time
- Without jitter: All retry at exactly 1s (DDoS ourselves!)
- With jitter: Spread across 0-1s (smooth load)
```

**Idempotent Operations:**
```python
def upload_chunk(upload_id, chunk_number, data, etag_cache):
    """
    Idempotent chunk upload with retry
    """
    # Check if already uploaded
    if chunk_number in etag_cache:
        return etag_cache[chunk_number]
    
    max_retries = 5
    for attempt in range(max_retries):
        try:
            response = s3.upload_part(
                Bucket='uploads',
                Key=f'{upload_id}/chunk_{chunk_number}',
                PartNumber=chunk_number + 1,
                Body=data
            )
            
            etag = response['ETag']
            etag_cache[chunk_number] = etag
            return etag
            
        except ClientError as e:
            if attempt == max_retries - 1:
                raise
            
            # Exponential backoff with jitter
            wait_time = (2 ** attempt) + random.uniform(0, 2 ** attempt)
            time.sleep(wait_time)
    
    raise UploadError(f"Failed to upload chunk {chunk_number}")
```

**Dead Letter Queue:**
```text
Processing failures after 3 retries:

Kafka Consumer:
├─ Attempt 1: Process event
│  └─ Failed: Requeue (same partition)
├─ Attempt 2: Process event
│  └─ Failed: Requeue
├─ Attempt 3: Process event
│  └─ Failed: Move to Dead Letter Queue
└─ DLQ: Manual review required

DLQ Processing:
1. Alert on-call engineer
2. Investigate failure reason
3. Fix issue (code bug, corrupted file, etc.)
4. Replay from DLQ or mark as permanently failed
```

#### Monitoring & Observability

**Key Metrics:**
```yaml
Upload Success Rate:
  - Metric: successful_uploads / total_upload_attempts
  - Target: > 99%
  - Alert: < 95% for 10 minutes

Upload Latency:
  - P50: < 3s
  - P95: < 10s
  - P99: < 30s
  - By file size bucket (< 1MB, 1-10MB, > 10MB)

Chunk Failure Rate:
  - Metric: failed_chunks / total_chunks
  - Target: < 1%
  - Alert: > 5%

Processing Lag:
  - Time from upload to thumbnail ready
  - Target: < 5s (P95)
  - Alert: > 60s

Queue Depth:
  - Kafka lag: processed_offset - latest_offset
  - Target: < 10,000 messages
  - Alert: > 100,000
```

**Distributed Tracing:**
```text
Trace Upload Journey:

Trace ID: trace-abc123
├─ Span 1: /upload/initiate (10ms)
├─ Span 2: S3 chunk upload (2000ms)
├─ Span 3: /upload/complete (50ms)
├─ Span 4: Kafka publish (5ms)
├─ Span 5: Thumbnail generation (3000ms)
├─ Span 6: Metadata extraction (500ms)
├─ Span 7: Face detection (8000ms)
└─ Span 8: Search indexing (200ms)

Total: 13,765ms
Bottleneck: Face detection (8s)
```

### Real-World Example: Dropbox Upload Evolution

**Dropbox 2009 (Bad):**
```text
Problems:
- Single-threaded upload
- No chunking (entire file or nothing)
- No resume capability
- Uploads failed silently
- No progress indication

User Experience:
- Upload 1GB file over slow connection
- Takes 2 hours
- Connection drops at 99%
- Start over from beginning!
- Users furious 😠
```

**Dropbox 2011 (Better):**
```text
Improvements:
- Chunked upload (4MB chunks)
- Resume capability
- Progress bar
- Retry logic

User Experience:
- Upload 1GB file (250 chunks)
- Connection drops at chunk 200
- Resume from chunk 200
- Only lose 4MB of progress
- Users happier 😊
```

**Dropbox 2015+ (Best):**
```text
Advanced features:
- Variable chunk size (network adaptive)
- Parallel chunk uploads (10 at once)
- Content-addressable storage (deduplication)
- Delta sync (only upload changed parts)
- Client-side encryption before upload

User Experience:
- Upload appears instant (deduplicated)
- Parallel upload: 10x faster
- Encrypted: Privacy protected
- Users love it 😍

Result: 500M users by 2016
```

### 🎯 Interview Questions: Upload Pipeline

**Q1: How would you handle a user uploading 1000 photos at once?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Batch Upload Strategy:**

```text
Challenge:
- 1000 photos × 3MB = 3GB total
- Serial upload: 1000 × 5s = 5000s (83 minutes!)
- Parallel upload all: Overwhelm client device & network

Solution: Throttled Parallel Upload

Client Implementation:
1. Queue all 1000 photos
2. Upload 10 photos in parallel
3. As each completes, start next from queue
4. Continue until all complete

Pseudocode:
```

```python
def batch_upload(photos, max_parallel=10):
    queue = photos.copy()
    in_progress = []
    completed = []
    
    while queue or in_progress:
        # Start new uploads if slots available
        while len(in_progress) < max_parallel and queue:
            photo = queue.pop(0)
            upload_future = start_upload_async(photo)
            in_progress.append(upload_future)
        
        # Wait for any upload to complete
        done = wait_for_any(in_progress)
        in_progress.remove(done)
        completed.append(done)
        
        # Update progress
        progress = len(completed) / len(photos)
        update_ui(progress)
    
    return completed
```

```text
Server-Side Optimizations:

1. Batch Endpoints:
POST /photos/batch-upload/initiate
{
  "num_photos": 1000
}

Response:
{
  "batch_id": "batch-uuid",
  "upload_urls": [...]  // 1000 pre-signed URLs
}

2. Single Kafka Event for Batch:
{
  "event": "batch.uploaded",
  "batch_id": "batch-uuid",
  "photo_ids": [...],  // All 1000 IDs
  "user_id": "user-uuid"
}

3. Batch Processing:
- Process all 1000 in single worker
- Amortize overhead (S3 connections, etc.)
- Generate thumbnails for all at once
- Single database transaction for metadata

Results:
- Client: 10 parallel × 5s = ~500s total (8 minutes)
- Server: Process batch in 120s vs 5000s serial
- User sees steady progress, not stuck waiting
```

**Interview Tip:** Shows you understand concurrency control and batch optimization!

</details>

**Q2: What if a user uploads the same photo 100 times from different devices?**

<details>
<summary>Click to see answer</summary>

**Answer:**

**Multi-Level Deduplication:**

```text
Level 1: Exact Duplicate (MD5 Hash)
- Calculate MD5 of file content
- Check: SELECT * FROM files WHERE md5 = ?
- If match: Instant "upload" (already have it!)
- Storage saved: 99.99%

Level 2: Perceptual Duplicate (pHash)
- Photo edited slightly (cropped, filtered)
- MD5 different but pHash similar
- Hamming distance < 5 bits
- Ask user: "Similar photo exists. Upload anyway?"
- Storage saved: 30-40%

Level 3: User-Scoped Deduplication
- Same photo uploaded by different users
- Store once, reference multiple times
- Content-addressable storage (CAS)
- S3 key: sha256(file_content)
- Multiple users point to same S3 object
- Storage saved: 50-60%

Example:
100 identical uploads:
- Level 1: Store 1 copy
- Cost: 1 × 3MB = 3MB (vs 300MB)
- Savings: 99%!

Vacation photo uploaded by 1000 tourists:
- Level 3: Store 1 copy in S3
- 1000 database records point to same S3 key
- Cost: 1 × 3MB + 1000 × 1KB metadata = 4MB
- vs 3GB without deduplication
- Savings: 99.87%!
```

**Implementation:**

```python
def handle_upload_with_deduplication(file_data, user_id):
    # Calculate hashes
    md5 = hashlib.md5(file_data).hexdigest()
    perceptual = generate_perceptual_hash(file_data)
    
    # Check exact duplicate
    existing = db.query(
        "SELECT photo_id FROM user_photos "
        "WHERE user_id = ? AND md5 = ?",
        user_id, md5
    )
    
    if existing:
        return {
            "photo_id": existing.photo_id,
            "message": "Photo already uploaded",
            "upload_saved": True
        }
    
    # Check perceptual duplicate
    similar = db.query(
        "SELECT photo_id, perceptual_hash "
        "FROM user_photos "
        "WHERE user_id = ? AND perceptual_hash = ?",
        user_id, perceptual
    )
    
    if similar:
        # Calculate Hamming distance
        distance = hamming_distance(perceptual, similar.perceptual_hash)
        
        if distance < 5:  # Very similar
            return {
                "similar_photo_id": similar.photo_id,
                "message": "Similar photo exists",
                "action_required": "user_confirmation"
            }
    
    # No duplicate - proceed with upload
    s3_key = f"content/{sha256(file_data)}.jpg"
    
    # Check if content already in S3 (global dedup)
    if s3.object_exists(bucket, s3_key):
        # Just create metadata record
        photo_id = create_photo_metadata(user_id, s3_key, md5, perceptual)
        return {
            "photo_id": photo_id,
            "message": "Photo uploaded (deduplicated)",
            "storage_saved": True
        }
    
    # Upload to S3
    s3.put_object(Bucket=bucket, Key=s3_key, Body=file_data)
    photo_id = create_photo_metadata(user_id, s3_key, md5, perceptual)
    
    return {
        "photo_id": photo_id,
        "message": "Photo uploaded successfully"
    }
```

**Cost Impact:**

```text
Without Deduplication:
- 1B users × 4,000 photos × 3MB = 12 EB
- Cost: 12 EB × $0.023/GB/month = $276M/month

With Deduplication (30% savings):
- Actual storage: 8.4 EB
- Cost: $193M/month
- Savings: $83M/month = $1B/year!

This is why Google Photos invested heavily in deduplication!
```

</details>

---

### 🤔 Think About It

1. **Upload Bandwidth**: If 100M users upload photos simultaneously on Christmas morning, how would you handle the traffic spike without overprovisioning for 364 days?

2. **Virus Scanning**: Should you scan files synchronously (user waits) or asynchronously (scan after upload)? What's the security vs UX trade-off?

3. **Resume Windows**: How long should upload URLs remain valid? 15 minutes vs 24 hours - what are the pros/cons?

---

### ✅ Key Takeaways

```text
✓ Chunked uploads: 5MB chunks for resume capability and parallel upload
✓ Direct-to-S3: Client uploads to S3, not through API servers (saves bandwidth)
✓ Pre-signed URLs: Temporary credentials for S3 upload (security)
✓ Validation layers: Client, server, post-upload, async processing
✓ Deduplication: MD5 for exact, pHash for perceptual, CAS for global
✓ Async processing: Kafka events for thumbnails, metadata, ML
✓ Retry logic: Exponential backoff with jitter prevents thundering herd
✓ Resume capability: Store upload state, allow chunk-level resume
✓ Monitoring: Track success rate, latency, queue depth, processing lag
✓ Cost optimization: Deduplication saves 30% storage = $1B/year
```

**Upload Pipeline Stages:**
```text
1. Initiate → Get pre-signed URLs from server
2. Upload → Client sends chunks directly to S3
3. Complete → Server verifies and creates metadata
4. Process → Async workers generate thumbnails, extract metadata, detect faces
5. Index → Search engines update, user can find photo
```

---

### 🎯 Practice Exercise

**Exercise: Design Live Photo Upload**

Live Photos (iPhone feature) are actually a still image + 3-second video. Design an upload system for Live Photos.

Consider:
- How to associate video with photo?
- Upload order (photo first or video first)?
- What if video upload fails but photo succeeds?
- How to show progress (2 files, but 1 "Live Photo")?
- Storage optimization (video is 10x larger than photo)

<details>
<summary>Click to see sample answer</summary>

**Sample Answer:**

```text
**Live Photo Structure:**
- Still image: 3MB (HEIC/JPEG)
- Video: 30MB (HEVC, 3 seconds @ 1080p)
- Total: 33MB per Live Photo

**Upload Strategy:**

1. Atomic Upload (Both or Neither):
   Initiate:
   POST /live-photos/upload/initiate
   {
     "photo_file_size": 3145728,
     "video_file_size": 31457280,
     "live_photo_id": "client-generated-uuid"
   }
   
   Response:
   {
     "live_photo_id": "uuid",
     "photo_upload_url": "...",
     "video_upload_url": "..."
   }

2. Upload Order: Photo First
   Why: User sees photo immediately while video uploads in background
   
   Client:
   - Upload photo (fast: 3s)
   - Show photo in gallery immediately
   - Upload video (slower: 30s) in background
   - Add "Live" badge when video completes

3. Failure Handling:
   Photo succeeds, video fails:
   - Keep photo (it's useful standalone)
   - Mark as "incomplete Live Photo"
   - Retry video upload in background
   - After 3 retries, downgrade to regular photo
   
   Photo fails:
   - Don't upload video
   - Retry entire Live Photo

4. Progress Indication:
   Combined progress = (photo_progress × 0.1) + (video_progress × 0.9)
   Weight by file size:
   - Photo: 3MB / 33MB = 9%
   - Video: 30MB / 33MB = 91%
   
   Shows realistic progress to user

5. Storage Optimization:
   - Video compression: HEVC (50% smaller than H.264)
   - Tiered storage: Video to cold storage after 30 days
     (Most views are recent Live Photos)
   - Cost: 3MB (hot) + 30MB (cold after 30d)
     = $0.069 + $0.012/month = $0.081/month
     vs $0.759/month (both hot)
     Savings: 89%!

6. Playback:
   - Pre-load video when user views photo
   - Cache video for smooth playback
   - Fallback to still photo if video load fails
```

**Why this works:**
- Instant feedback (photo shows immediately)
- Graceful degradation (photo works even if video fails)
- Accurate progress (weighted by file size)
- Cost-optimized (cold storage for rarely-viewed videos)

</details>

---

**Ready for Section 7?** Next, we'll explore how to transform uploaded photos into optimized, searchable assets through intelligent processing pipelines! 🎨

---

## Section 7: Image Processing & ML Pipeline

### 📚 Learning Objectives

By the end of this section, you will understand:

- **🟢 Beginner**: What happens to photos after upload, thumbnail generation basics, image metadata
- **🟡 Intermediate**: Asynchronous processing pipelines, format optimization, EXIF extraction, event-driven architecture
- **🔴 Advanced**: GPU-accelerated batch processing, adaptive quality optimization, ML feature extraction, distributed image processing at scale

---

### 🎯 Why This Matters

**The Problem:**
When you upload a photo from your phone, it might be a 12MB RAW file taken on a 108MP camera. Loading this directly would:
- Take 30+ seconds on mobile networks
- Consume massive bandwidth
- Drain phone batteries
- Create terrible user experience

**The Solution:**
Image processing transforms raw uploads into optimized assets:
- **Multiple thumbnail sizes** for different contexts (grid view, full screen, sharing)
- **Format optimization** (WebP for modern browsers, JPEG fallback)
- **Metadata extraction** for search, organization, and smart features
- **Quality analysis** to identify blurry/duplicate photos

**Real-World Impact:**
- **Google Photos**: Processes 1.5 billion photos/day through their pipeline
- **Instagram**: Generates 5+ variants of each upload (thumbnails, optimized versions, stories format)
- **Dropbox**: Reduced bandwidth by 80% using smart image processing

**Interview Relevance:**
Image processing questions test your understanding of:
- Asynchronous processing at scale
- Trade-offs between quality and performance
- Event-driven architectures
- Resource optimization (CPU, GPU, memory)

---

### 🟢 Beginner Level: Image Processing Fundamentals

#### What is Image Processing?

**Analogy: The Photo Lab**
Think of image processing like an old-school photo lab:
1. **You drop off film** (upload raw photo)
2. **Lab develops negatives** (extract metadata, validate format)
3. **Create prints in different sizes** (generate thumbnails: wallet-size, 4x6, 8x10)
4. **Apply finishing touches** (optimize quality, adjust colors)
5. **File in catalog** (index for search)

**Why Multiple Sizes?**
Different screens need different image sizes:
- **Grid view (150px)**: Tiny thumbnails for photo library
- **Preview (400px)**: Medium size for quick viewing
- **Full screen (1080px)**: High quality for detail
- **Original**: Kept for editing and printing

#### Understanding Image Formats

**JPEG (Joint Photographic Experts Group):**
- **Use case**: Photographs with many colors
- **Compression**: Lossy (smaller files, some quality loss)
- **File size**: A 4000x3000 photo might be 3-5 MB
- **Best for**: Regular photos from cameras/phones

**PNG (Portable Network Graphics):**
- **Use case**: Graphics, screenshots, transparency needed
- **Compression**: Lossless (perfect quality, larger files)
- **File size**: Same photo might be 15-25 MB
- **Best for**: Logos, diagrams, images with text

**WebP (Google's format):**
- **Use case**: Modern web delivery
- **Compression**: Both lossy and lossless modes
- **File size**: 25-35% smaller than JPEG at same quality
- **Best for**: Fast web loading (supported by Chrome, Firefox, Edge)

**HEIC (High Efficiency Image Container):**
- **Use case**: iPhone photos since iOS 11
- **Compression**: Better than JPEG (50% smaller files)
- **File size**: 2-3 MB for same quality as 5 MB JPEG
- **Best for**: Mobile storage efficiency

#### What is Metadata?

**EXIF (Exchangeable Image File Format):**
Hidden data embedded in photos:
```text
Camera: iPhone 14 Pro
Date Taken: September 15, 2025 2:30 PM
Location: 37.7749° N, 122.4194° W (San Francisco)
Settings: f/1.8, 1/120s, ISO 100
Dimensions: 4032 x 3024 pixels
```

**Why Extract Metadata?**
- **Search**: "Show me photos from San Francisco"
- **Organization**: Auto-group by date/location
- **Smart features**: Create automatic albums from trips
- **Privacy**: Strip location data when sharing publicly

#### The Basic Processing Flow

```mermaid
graph LR
    Upload[Photo Upload Completes] --> Event[Kafka Event: photo.uploaded]
    Event --> Worker[Processing Worker]
    Worker --> Download[Download from S3]
    Download --> Process[Process Image]
    
    Process --> Thumb[Generate Thumbnails]
    Process --> Meta[Extract Metadata]
    Process --> Optimize[Optimize Format]
    
    Thumb --> S3T[Save to S3]
    Meta --> DB[Save to Database]
    Optimize --> S3O[Save to S3]
    
    S3T --> Done[Processing Complete]
    DB --> Done
    S3O --> Done
```

**Step-by-Step:**

1. **Upload Completes**: Photo stored in S3 (from Section 6)
2. **Event Published**: System publishes "photo.uploaded" event
3. **Worker Picks Up**: Processing worker receives event from queue
4. **Download Original**: Worker downloads photo from S3
5. **Generate Thumbnails**: Create 3 sizes (150px, 400px, 1080px)
6. **Extract Metadata**: Read EXIF data (date, location, camera)
7. **Optimize Format**: Convert to WebP for modern browsers
8. **Save Results**: Store thumbnails and optimized versions
9. **Update Database**: Save metadata for search
10. **Mark Complete**: Photo ready for viewing!

#### Simple Example: What Happens to Your Vacation Photo

```text
You upload: beach_sunset.jpg (5.2 MB, 4032x3024, JPEG)

Processing creates:
├─ Thumbnail 150px: beach_sunset_150.webp (15 KB)
├─ Thumbnail 400px: beach_sunset_400.webp (45 KB)
├─ Thumbnail 1080px: beach_sunset_1080.webp (180 KB)
├─ Original optimized: beach_sunset_original.webp (2.1 MB)
└─ Metadata extracted:
    ├─ Date: September 15, 2025 6:30 PM
    ├─ Location: Malibu Beach, California
    ├─ Camera: iPhone 14 Pro
    └─ Dimensions: 4032x3024 pixels

Total storage: 2.1 MB + 240 KB = 2.34 MB (saved 2.86 MB!)
```

#### Why Not Process During Upload?

**Bad Approach (Synchronous):**
```text
1. Upload 5 MB → 10 seconds
2. Wait for processing → 8 seconds
3. Return success → Total: 18 seconds ❌
```

**Good Approach (Asynchronous):**
```text
1. Upload 5 MB → 10 seconds
2. Return success → Total: 10 seconds ✓
3. Processing happens in background → 8 seconds (user doesn't wait)
```

**Benefits:**
- ✅ Faster upload confirmation
- ✅ Better use of server resources
- ✅ Can retry processing if it fails
- ✅ User can browse other photos while processing

---

### 🟡 Intermediate Level: Production Processing Pipeline

#### Event-Driven Architecture

**Kafka Topic Structure:**

```text
Topic: photo.uploaded
├─ Partition 0: Users A-H (evenly distributed)
├─ Partition 1: Users I-P
├─ Partition 2: Users Q-Z
└─ Partition 3: Priority uploads (new users, paying users)

Event Schema:
{
  "event_id": "evt_123abc",
  "event_type": "photo.uploaded",
  "timestamp": "2025-10-27T14:30:00Z",
  "user_id": "user_550e8400",
  "photo_id": "photo_123e4567",
  "s3_location": "s3://photos-prod/user_550e8400/2025/10/photo_123e4567.jpg",
  "file_size_bytes": 5242880,
  "original_format": "JPEG",
  "priority": "normal"
}
```

**Consumer Group Pattern:**

```python
# Image Processing Worker (simplified)
from kafka import KafkaConsumer
import json

consumer = KafkaConsumer(
    'photo.uploaded',
    group_id='image-processors',
    bootstrap_servers=['kafka1:9092', 'kafka2:9092', 'kafka3:9092'],
    auto_offset_reset='earliest',
    enable_auto_commit=False,  # Manual commit for reliability
    max_poll_records=10  # Batch processing
)

for message in consumer:
    event = json.loads(message.value)
    
    try:
        # Process the photo
        process_photo(event['photo_id'], event['s3_location'])
        
        # Commit offset only after success
        consumer.commit()
        
    except Exception as e:
        # Log error and continue (will retry on next poll)
        logger.error(f"Failed to process {event['photo_id']}: {e}")
        # Don't commit - will reprocess this message
```

**Why Kafka?**
- **Throughput**: Handles 1.5 billion events/day (18,000/second)
- **Durability**: Messages persisted to disk, survive crashes
- **Scalability**: Add more workers to consume faster
- **Ordering**: Photos from same user processed in order (partition by user_id)
- **Replay**: Can reprocess old photos if needed

#### Thumbnail Generation Deep-Dive

**Library Choice: Pillow (Python Imaging Library)**

```python
from PIL import Image
import os

def generate_thumbnails(input_path, photo_id):
    """
    Generate 3 thumbnail sizes from original photo
    
    Args:
        input_path: Path to original photo file
        photo_id: Unique photo identifier
    
    Returns:
        dict: Paths to generated thumbnails
    """
    # Define thumbnail sizes (max dimension)
    sizes = {
        'small': 150,    # Grid view
        'medium': 400,   # Preview
        'large': 1080    # Full screen mobile
    }
    
    # Open original image
    with Image.open(input_path) as img:
        # Get original dimensions
        width, height = img.size
        
        # Preserve EXIF orientation
        img = ImageOps.exif_transpose(img)
        
        thumbnails = {}
        
        for size_name, max_dim in sizes.items():
            # Calculate new dimensions (preserve aspect ratio)
            if width > height:
                new_width = max_dim
                new_height = int((max_dim / width) * height)
            else:
                new_height = max_dim
                new_width = int((max_dim / height) * width)
            
            # Resize with high-quality resampling
            thumb = img.resize(
                (new_width, new_height),
                Image.Resampling.LANCZOS  # High quality
            )
            
            # Save as WebP (modern format)
            webp_path = f"/tmp/{photo_id}_{size_name}.webp"
            thumb.save(
                webp_path,
                'WEBP',
                quality=85,  # Good balance of quality/size
                method=6     # Compression effort (0-6, higher=smaller)
            )
            
            # Also save JPEG fallback for older browsers
            jpeg_path = f"/tmp/{photo_id}_{size_name}.jpg"
            thumb.save(
                jpeg_path,
                'JPEG',
                quality=85,
                optimize=True,  # Optimize Huffman tables
                progressive=True  # Progressive JPEG (loads gradually)
            )
            
            thumbnails[size_name] = {
                'webp': webp_path,
                'jpeg': jpeg_path,
                'dimensions': (new_width, new_height)
            }
    
    return thumbnails
```

**Image Quality Settings:**

| Format | Quality | Use Case | File Size (400px) |
|--------|---------|----------|-------------------|
| WebP 85% | High | Modern browsers | 45 KB |
| JPEG 85% | High | Fallback | 60 KB |
| WebP 70% | Medium | Slow connections | 28 KB |
| JPEG 70% | Medium | Fallback | 40 KB |

**Progressive JPEG:**
```text
Traditional JPEG loading:
[        ] 0%
[        ] 0%
[████████] 100% ← Appears all at once

Progressive JPEG loading:
[░░░░░░░░] 0% ← Blurry preview
[▒▒▒▒▒▒▒▒] 25% ← Getting clearer
[▓▓▓▓▓▓▓▓] 50% ← Almost there
[████████] 100% ← Full quality
```

Better user experience on slow connections!

#### Metadata Extraction with ExifRead

**Reading EXIF Data:**

```python
import exifread
from datetime import datetime
from geopy.geocoders import Nominatim

def extract_metadata(image_path):
    """
    Extract EXIF metadata from photo
    
    Returns:
        dict: Structured metadata
    """
    with open(image_path, 'rb') as f:
        tags = exifread.process_file(f, details=False)
    
    metadata = {
        'camera': {},
        'capture': {},
        'technical': {},
        'location': {}
    }
    
    # Camera information
    if 'Image Make' in tags:
        metadata['camera']['make'] = str(tags['Image Make'])
    if 'Image Model' in tags:
        metadata['camera']['model'] = str(tags['Image Model'])
    
    # Capture date/time
    if 'EXIF DateTimeOriginal' in tags:
        date_str = str(tags['EXIF DateTimeOriginal'])
        metadata['capture']['datetime'] = datetime.strptime(
            date_str, '%Y:%m:%d %H:%M:%S'
        )
    
    # Camera settings
    if 'EXIF FNumber' in tags:
        metadata['technical']['aperture'] = str(tags['EXIF FNumber'])
    if 'EXIF ExposureTime' in tags:
        metadata['technical']['shutter_speed'] = str(tags['EXIF ExposureTime'])
    if 'EXIF ISOSpeedRatings' in tags:
        metadata['technical']['iso'] = str(tags['EXIF ISOSpeedRatings'])
    
    # GPS coordinates
    if 'GPS GPSLatitude' in tags and 'GPS GPSLongitude' in tags:
        lat = convert_to_degrees(tags['GPS GPSLatitude'])
        lon = convert_to_degrees(tags['GPS GPSLongitude'])
        
        # Handle N/S and E/W
        if tags['GPS GPSLatitudeRef'].values == 'S':
            lat = -lat
        if tags['GPS GPSLongitudeRef'].values == 'W':
            lon = -lon
        
        metadata['location']['latitude'] = lat
        metadata['location']['longitude'] = lon
        
        # Reverse geocode to get location name
        location_name = reverse_geocode(lat, lon)
        metadata['location']['name'] = location_name
    
    return metadata

def convert_to_degrees(value):
    """Convert GPS coordinates to degrees"""
    d = float(value.values[0].num) / float(value.values[0].den)
    m = float(value.values[1].num) / float(value.values[1].den)
    s = float(value.values[2].num) / float(value.values[2].den)
    return d + (m / 60.0) + (s / 3600.0)

def reverse_geocode(lat, lon):
    """Convert coordinates to location name"""
    geolocator = Nominatim(user_agent="google_photos")
    location = geolocator.reverse(f"{lat}, {lon}")
    return location.address if location else None
```

**Example Output:**

```json
{
  "camera": {
    "make": "Apple",
    "model": "iPhone 14 Pro"
  },
  "capture": {
    "datetime": "2025-09-15T18:30:00"
  },
  "technical": {
    "aperture": "f/1.78",
    "shutter_speed": "1/120",
    "iso": "100"
  },
  "location": {
    "latitude": 34.0259,
    "longitude": -118.7798,
    "name": "Malibu Beach, California, USA"
  }
}
```

#### Complete Processing Worker

```python
import boto3
from PIL import Image
import exifread
import json

class PhotoProcessor:
    def __init__(self):
        self.s3_client = boto3.client('s3')
        self.cassandra_session = get_cassandra_session()
        self.elasticsearch_client = get_es_client()
    
    def process_photo(self, event):
        """
        Main processing function
        
        Steps:
        1. Download from S3
        2. Extract metadata
        3. Generate thumbnails
        4. Optimize original
        5. Upload results to S3
        6. Save metadata to databases
        7. Index for search
        """
        photo_id = event['photo_id']
        user_id = event['user_id']
        s3_key = event['s3_location']
        
        start_time = time.time()
        
        try:
            # 1. Download original
            temp_file = f"/tmp/{photo_id}_original.jpg"
            self.s3_client.download_file(
                'photos-bucket',
                s3_key,
                temp_file
            )
            
            # 2. Extract metadata
            metadata = extract_metadata(temp_file)
            
            # 3. Generate thumbnails
            thumbnails = generate_thumbnails(temp_file, photo_id)
            
            # 4. Optimize original (convert to WebP)
            optimized_path = self.optimize_image(temp_file, photo_id)
            
            # 5. Upload to S3
            s3_urls = self.upload_results(
                photo_id, user_id, thumbnails, optimized_path
            )
            
            # 6. Save metadata to Cassandra
            self.save_metadata(photo_id, user_id, metadata, s3_urls)
            
            # 7. Index in Elasticsearch
            self.index_for_search(photo_id, user_id, metadata)
            
            processing_time = time.time() - start_time
            
            logger.info(
                f"Processed {photo_id} in {processing_time:.2f}s",
                extra={'user_id': user_id, 'photo_id': photo_id}
            )
            
            # Publish completion event
            self.publish_event('photo.processed', {
                'photo_id': photo_id,
                'user_id': user_id,
                'processing_time_ms': int(processing_time * 1000)
            })
            
        except Exception as e:
            logger.error(f"Failed to process {photo_id}: {e}")
            # Publish failure event for monitoring
            self.publish_event('photo.processing_failed', {
                'photo_id': photo_id,
                'error': str(e)
            })
            raise
    
    def optimize_image(self, input_path, photo_id):
        """Convert original to WebP for storage efficiency"""
        with Image.open(input_path) as img:
            output_path = f"/tmp/{photo_id}_optimized.webp"
            img.save(
                output_path,
                'WEBP',
                quality=90,  # High quality for original
                method=6
            )
            return output_path
    
    def upload_results(self, photo_id, user_id, thumbnails, optimized_path):
        """Upload thumbnails and optimized version to S3"""
        urls = {}
        
        # Upload thumbnails
        for size, paths in thumbnails.items():
            webp_key = f"thumbnails/{size}/{user_id}/{photo_id}.webp"
            self.s3_client.upload_file(
                paths['webp'],
                'photos-bucket',
                webp_key,
                ExtraArgs={'ContentType': 'image/webp'}
            )
            urls[f'thumbnail_{size}_webp'] = f"https://cdn.example.com/{webp_key}"
            
            jpeg_key = f"thumbnails/{size}/{user_id}/{photo_id}.jpg"
            self.s3_client.upload_file(
                paths['jpeg'],
                'photos-bucket',
                jpeg_key,
                ExtraArgs={'ContentType': 'image/jpeg'}
            )
            urls[f'thumbnail_{size}_jpeg'] = f"https://cdn.example.com/{jpeg_key}"
        
        # Upload optimized original
        optimized_key = f"photos/{user_id}/{photo_id}_optimized.webp"
        self.s3_client.upload_file(
            optimized_path,
            'photos-bucket',
            optimized_key,
            ExtraArgs={'ContentType': 'image/webp'}
        )
        urls['original_optimized'] = f"https://cdn.example.com/{optimized_key}"
        
        return urls
```

**Processing Performance:**

```text
Typical processing timeline for 5MB photo:

00.0s - Start processing
00.5s - Download from S3 (5 MB @ 10 MB/s)
01.0s - Extract metadata (EXIF parsing)
03.5s - Generate 3 thumbnails (2.5s)
04.5s - Optimize original to WebP (1s)
05.0s - Upload results to S3 (0.5s)
05.2s - Save to Cassandra (0.2s)
05.5s - Index in Elasticsearch (0.3s)
------
05.5s - Total processing time

Bottleneck: Thumbnail generation (45% of time)
Optimization opportunity: GPU acceleration or batch processing
```

---

### 🔴 Advanced Level: Production Optimizations

#### GPU-Accelerated Batch Processing

**Problem:** Processing 18,000 photos/second with CPU-only workers requires massive infrastructure.

**Solution:** GPU acceleration with batch processing.

**GPU Advantage:**

```text
CPU Processing (Pillow):
- Single-threaded per image
- 2.5 seconds per photo for 3 thumbnails
- Need 45,000 workers for 18,000 QPS

GPU Processing (NVIDIA DALI + cuPy):
- Batch 100 images simultaneously
- 25 seconds for 100 photos (0.25s each)
- Need only 4,500 workers (10x reduction!)
```

**NVIDIA DALI Pipeline:**

```python
import nvidia.dali as dali
import nvidia.dali.fn as fn
import nvidia.dali.types as types

@dali.pipeline_def
def image_processing_pipeline(file_list, batch_size):
    """
    GPU-accelerated image processing pipeline
    
    Processes batch of images in parallel on GPU:
    - Decode JPEG
    - Resize to multiple sizes
    - Encode to WebP/JPEG
    """
    # Read and decode images on GPU
    images = fn.readers.file(files=file_list)
    images = fn.decoders.image(images, device='mixed')  # CPU decode, GPU transfer
    
    # Generate multiple thumbnails in parallel
    thumb_150 = fn.resize(
        images,
        resize_longer=150,
        interp_type=types.INTERP_LANCZOS3,
        device='gpu'
    )
    
    thumb_400 = fn.resize(
        images,
        resize_longer=400,
        interp_type=types.INTERP_LANCZOS3,
        device='gpu'
    )
    
    thumb_1080 = fn.resize(
        images,
        resize_longer=1080,
        interp_type=types.INTERP_LANCZOS3,
        device='gpu'
    )
    
    return thumb_150, thumb_400, thumb_1080

# Usage
pipe = image_processing_pipeline(
    file_list=batch_of_100_photos,
    batch_size=100,
    num_threads=4,
    device_id=0
)
pipe.build()

# Process batch
outputs = pipe.run()
thumbs_150, thumbs_400, thumbs_1080 = outputs

# Each contains 100 processed images!
```

**GPU Worker Architecture:**

```mermaid
graph TB
    Kafka[Kafka Topic] --> Batcher[Batch Collector]
    Batcher -->|100 photos| GPU1[GPU Worker 1<br/>Tesla T4]
    Batcher -->|100 photos| GPU2[GPU Worker 2<br/>Tesla T4]
    Batcher -->|100 photos| GPU3[GPU Worker N<br/>Tesla T4]
    
    GPU1 --> S3Upload[Parallel S3 Upload]
    GPU2 --> S3Upload
    GPU3 --> S3Upload
    
    S3Upload --> Complete[Batch Complete Event]
```

**Cost Analysis:**

```text
CPU Workers:
- 45,000 workers × $0.05/hour = $2,250/hour
- Processing time: 2.5s per photo

GPU Workers (Tesla T4):
- 4,500 workers × $0.35/hour = $1,575/hour
- Processing time: 0.25s per photo
- Savings: $675/hour = $16,200/day = $5.9M/year!
```

#### Adaptive Quality Optimization

**Challenge:** Not all photos need same quality. Optimize based on content and usage.

**ML Quality Classifier:**

```python
import torch
import torchvision.models as models

class QualityClassifier:
    """
    Classify photos to determine optimal compression settings
    
    Categories:
    - High detail (landscapes, architecture): Use 90% quality
    - Medium detail (portraits): Use 85% quality
    - Low detail (screenshots, memes): Use 70% quality
    """
    
    def __init__(self):
        # Use pre-trained ResNet for feature extraction
        self.model = models.resnet50(pretrained=True)
        self.model.eval()
        
        # Custom classifier head
        self.classifier = torch.nn.Sequential(
            torch.nn.Linear(2048, 512),
            torch.nn.ReLU(),
            torch.nn.Dropout(0.3),
            torch.nn.Linear(512, 3)  # 3 quality categories
        )
    
    def predict_quality_tier(self, image_tensor):
        """
        Predict optimal quality tier for image
        
        Returns:
            'high', 'medium', or 'low'
        """
        with torch.no_grad():
            features = self.model(image_tensor)
            logits = self.classifier(features)
            category = torch.argmax(logits).item()
        
        return ['high', 'medium', 'low'][category]

def adaptive_compression(image_path, photo_id):
    """
    Apply adaptive compression based on content analysis
    """
    classifier = QualityClassifier()
    image = load_image(image_path)
    
    quality_tier = classifier.predict_quality_tier(image)
    
    quality_settings = {
        'high': {'webp': 90, 'jpeg': 92},
        'medium': {'webp': 85, 'jpeg': 87},
        'low': {'webp': 70, 'jpeg': 75}
    }
    
    settings = quality_settings[quality_tier]
    
    # Apply optimized compression
    save_with_quality(image, photo_id, settings)
    
    return {
        'quality_tier': quality_tier,
        'webp_quality': settings['webp'],
        'jpeg_quality': settings['jpeg'],
        'estimated_savings': calculate_savings(quality_tier)
    }
```

**Storage Savings:**

```text
Traditional approach (all photos at 85% quality):
- Average thumbnail size: 45 KB
- 1.5B daily uploads × 45 KB = 67.5 TB/day

Adaptive approach:
- High quality (20%): 60 KB × 300M = 18 TB
- Medium quality (60%): 45 KB × 900M = 40.5 TB  
- Low quality (20%): 25 KB × 300M = 7.5 TB
- Total: 66 TB/day
- Savings: 1.5 TB/day = 547 TB/year = $10K/year

Plus: Better quality where it matters!
```

#### Intelligent Format Selection

**Modern Browser Detection:**

```python
def select_optimal_format(user_agent, device_type):
    """
    Select best image format based on client capabilities
    
    Priority order:
    1. AVIF (newest, best compression) - if supported
    2. WebP (good compression, wide support)
    3. JPEG (universal fallback)
    """
    supports_avif = check_avif_support(user_agent)
    supports_webp = check_webp_support(user_agent)
    
    # Network condition heuristics
    is_mobile = device_type == 'mobile'
    is_slow_connection = check_network_speed() < 2.0  # Mbps
    
    if supports_avif and is_mobile:
        # AVIF 30% smaller than WebP, great for mobile
        return 'avif', 80  # Quality setting
    elif supports_webp:
        quality = 70 if is_slow_connection else 85
        return 'webp', quality
    else:
        quality = 75 if is_slow_connection else 85
        return 'jpeg', quality

# Usage in API response
@app.route('/photos/<photo_id>/thumbnail/<size>')
def serve_thumbnail(photo_id, size):
    user_agent = request.headers.get('User-Agent')
    device_type = detect_device(user_agent)
    
    format, quality = select_optimal_format(user_agent, device_type)
    
    # Redirect to appropriate CDN URL
    cdn_url = f"https://cdn.example.com/thumbnails/{size}/{photo_id}.{format}"
    return redirect(cdn_url)
```

**Format Comparison:**

| Format | Compression | Browser Support | Use Case | Size (400px) |
|--------|-------------|----------------|----------|--------------|
| AVIF | Best (40% smaller than JPEG) | Chrome 85+, Firefox 93+ | Modern mobile | 25 KB |
| WebP | Great (30% smaller than JPEG) | Chrome, Firefox, Edge, Safari 14+ | Modern web | 35 KB |
| JPEG | Good (baseline) | Universal | Fallback | 60 KB |
| JPEG 2000 | Better than JPEG | Safari only | Legacy Apple | 50 KB |

#### Distributed Processing with Celery

**Task Queue Architecture:**

```python
from celery import Celery, group, chain
from kombu import Queue

app = Celery('photo_processor')

# Define priority queues
app.conf.task_queues = (
    Queue('critical', routing_key='critical'),  # New users, paying users
    Queue('high', routing_key='high'),          # Recent uploads (<1 hour)
    Queue('normal', routing_key='normal'),      # Regular processing
    Queue('low', routing_key='low'),            # Batch reprocessing
)

@app.task(queue='normal', bind=True, max_retries=3)
def process_single_photo(self, photo_id, user_id, s3_location):
    """
    Process a single photo with retry logic
    """
    try:
        # Download
        local_path = download_from_s3(s3_location)
        
        # Process
        result = process_photo_pipeline(local_path, photo_id)
        
        return result
        
    except Exception as exc:
        # Exponential backoff: 1min, 2min, 4min
        retry_delay = 60 * (2 ** self.request.retries)
        raise self.retry(exc=exc, countdown=retry_delay)

@app.task
def process_batch_photos(batch_info):
    """
    Process multiple photos in parallel using GPU
    """
    photo_ids = batch_info['photo_ids']
    
    # Create parallel task group
    job = group(
        process_single_photo.s(pid, uid, s3loc)
        for pid, uid, s3loc in batch_info['photos']
    )
    
    result = job.apply_async()
    return result.get()  # Wait for all to complete

# Task chaining for complex workflows
@app.task
def extract_metadata_task(photo_id):
    metadata = extract_metadata(photo_id)
    return metadata

@app.task
def generate_thumbnails_task(metadata):
    photo_id = metadata['photo_id']
    thumbnails = generate_thumbnails(photo_id)
    return {**metadata, 'thumbnails': thumbnails}

@app.task
def index_in_search_task(data):
    index_in_elasticsearch(data)
    return data

# Chain tasks: metadata → thumbnails → search indexing
workflow = chain(
    extract_metadata_task.s('photo_123'),
    generate_thumbnails_task.s(),
    index_in_search_task.s()
)

workflow.apply_async()
```

**Worker Scaling Strategy:**

```yaml
Worker Configuration:
  Critical Queue:
    workers: 100
    concurrency: 4 (per worker)
    autoscale: 50-200 based on queue depth
    
  High Queue:
    workers: 500
    concurrency: 4
    autoscale: 250-1000
    
  Normal Queue:
    workers: 1000
    concurrency: 4
    autoscale: 500-2000
    
  Low Queue:
    workers: 100
    concurrency: 8 (can use more resources)
    autoscale: 50-200
    
Scaling Triggers:
  - Queue depth > 10,000: Scale up
  - Queue depth < 1,000: Scale down
  - Processing lag > 5 minutes: Scale up aggressively
  - Time of day: Pre-scale before peak hours (6-9 PM)
```

---

### 🌍 Real-World Example: Instagram's Image Processing Evolution

**Phase 1 (2010-2012): Simple Synchronous Processing**

```text
Early Instagram:
- Single square format (612x612)
- Simple filters applied on upload
- Processed during HTTP request
- Result: Slow uploads (15-30 seconds)
```

**Phase 2 (2012-2015): Asynchronous + Multiple Sizes**

```text
Growth phase:
- Async processing with Celery + RabbitMQ
- 3 thumbnail sizes generated
- Original photo preserved
- Result: 10x faster uploads, but processing lag during peaks
```

**Phase 3 (2015-2020): ML-Powered Optimization**

```text
Scale phase:
- GPU-accelerated batch processing (100 photos/batch)
- Adaptive quality based on content analysis
- Format optimization (WebP for modern clients)
- Smart cropping for different aspect ratios
- Result: 50% bandwidth reduction, better quality
```

**Phase 4 (2020-Present): Edge Processing + AVIF**

```text
Current architecture:
- Edge processing for immediate previews
- AVIF format for supported browsers (40% smaller)
- ML-powered quality prediction
- CDN-based format selection
- Video thumbnail extraction at multiple timestamps
- Result: Sub-second processing, optimal quality per device
```

**Key Lessons:**
1. **Start simple**: Synchronous processing fine for MVP
2. **Async early**: Decouple upload from processing before scaling
3. **GPU when it matters**: 10x cost savings at Instagram's scale
4. **ML for optimization**: Adaptive quality saves bandwidth without sacrificing UX
5. **Format diversity**: Support modern formats while maintaining fallbacks

---

### 💬 Interview Questions & Answers

#### Q1: "How would you handle processing 1.5 billion photos per day?"

**Structured Answer:**

**1. Calculate Requirements:**
```text
1.5B photos/day = 18,000 photos/second average
Peak (evenings): 3x average = 54,000 photos/second
Processing time: ~5 seconds per photo

Naive approach: 54,000 × 5s = 270,000 workers needed ❌
```

**2. Optimization Strategy:**

"I would use a multi-tier approach:

**A. Batch Processing with GPU:**
- Group photos into batches of 100
- Use GPU workers (NVIDIA T4 or similar)
- Parallel processing: 100 photos in 25 seconds
- Workers needed: 54,000 ÷ (100/25) = 13,500 workers
- Cost reduction: 20x vs CPU-only

**B. Priority Queuing:**
- Critical queue: New users, paying customers (immediate processing)
- High queue: Recent uploads (<1 hour old)
- Normal queue: Regular processing
- Low queue: Batch reprocessing, old photos

**C. Async Event-Driven:**
- Kafka for event streaming (proven at 18K+ events/sec)
- Consumer groups for parallel processing
- Dead letter queue for failed jobs
- Idempotent processing for retry safety

**D. Progressive Processing:**
- Generate smallest thumbnail (150px) first → immediate UI update
- Generate other sizes in background
- On-demand generation for rarely-accessed sizes"

**3. Monitoring & Scaling:**
- Track queue depth, processing lag
- Auto-scale based on metrics
- Pre-scale before peak hours (6-9 PM)

**Follow-up handling:**
"What about storage costs?" → "Use adaptive compression saving 30% storage..."

---

#### Q2: "How do you extract metadata without blocking the upload?"

**Answer:**

"Metadata extraction happens asynchronously after upload completes:

**Upload Flow:**
1. Client uploads photo to S3 (direct upload with pre-signed URL)
2. Upload service publishes 'photo.uploaded' event to Kafka
3. Returns success immediately to client ✅ (user doesn't wait)

**Processing Flow (Background):**
4. Processing worker consumes event from Kafka
5. Downloads photo from S3
6. Extracts EXIF metadata (0.5-1 second)
7. Saves metadata to Cassandra + indexes in Elasticsearch
8. Publishes 'metadata.extracted' event

**Why This Works:**
- User gets instant upload confirmation
- Processing happens in parallel across workers
- Failures can retry without impacting user
- Can process millions of photos without blocking uploads

**Edge Case Handling:**
- If metadata extraction fails → still show photo, mark metadata as 'unavailable'
- Client can poll processing status via WebSocket or API
- For critical metadata (date/location), extract on client side and send with upload as hint"

---

#### Q3: "How would you ensure generated thumbnails are always available?"

**Multi-Layer Approach:**

**1. Pre-Generation (Primary Strategy):**
```python
# Generate all thumbnail sizes during processing
for size in [150, 400, 1080]:
    generate_and_upload_thumbnail(photo_id, size)
```

**2. Lazy Generation (Fallback):**
```python
@app.route('/thumbnails/<size>/<photo_id>')
def serve_thumbnail(size, photo_id):
    # Check if thumbnail exists
    thumbnail_url = get_from_cache(f"thumb:{photo_id}:{size}")
    
    if thumbnail_url:
        return redirect(thumbnail_url)
    
    # Not found → generate on-demand
    original = download_from_s3(photo_id)
    thumbnail = resize_image(original, size)
    upload_to_s3(thumbnail, f"thumbnails/{size}/{photo_id}.webp")
    
    return serve_file(thumbnail)
```

**3. Monitoring & Healing:**
```python
# Nightly job to find missing thumbnails
missing_thumbnails = find_photos_without_thumbnails()

for photo_id in missing_thumbnails:
    retry_thumbnail_generation.delay(photo_id)
```

**4. CDN Caching:**
- All thumbnails cached at CDN (CloudFront)
- TTL: 30 days
- Reduces origin requests by 95%

**5. Graceful Degradation:**
```text
If thumbnail unavailable:
├─ Try next size up (400px → 1080px)
├─ Show placeholder with loading indicator
├─ Async generate in background
└─ Update UI when ready
```

**SLA Target:** 99.99% thumbnail availability (<1 minute to generate missing ones)

---

###  Practice Exercise: Design Video Thumbnail Extraction

**Scenario:** Extend your image processing pipeline to support video uploads.

**Requirements:**
1. Extract 3 thumbnails from video at different timestamps (beginning, middle, end)
2. Generate animated GIF preview (3 seconds)
3. Support videos up to 4K resolution
4. Process 100,000 videos/day

**Your Task:** Design the video processing pipeline.

<details>
<summary><strong>💡 Hint</strong></summary>

Consider:
- Video transcoding tools (FFmpeg, AWS MediaConvert)
- Keyframe extraction vs arbitrary timestamps
- GPU acceleration for encoding
- Storage costs (video vs image)
- Processing time (much longer than images)
- Format support (MP4, MOV, AVI, etc.)

</details>

<details>
<summary><strong>✅ Solution</strong></summary>

**Video Processing Pipeline Design:**

**1. Architecture:**

```mermaid
graph TB
    Upload[Video Upload] --> S3[S3 Storage]
    S3 --> Event[Kafka: video.uploaded]
    Event --> Worker[Video Worker Pool]
    
    Worker --> Extract[Extract Metadata<br/>FFprobe]
    Extract --> Keyframes[Extract Keyframes<br/>FFmpeg]
    Extract --> GIF[Generate GIF Preview<br/>FFmpeg]
    Extract --> Transcode[Transcode to Web Formats<br/>H.264, VP9]
    
    Keyframes --> S3T[S3 Thumbnails]
    GIF --> S3T
    Transcode --> S3V[S3 Video]
    
    S3T --> Index[Index Metadata]
    S3V --> Index
```

**2. Implementation:**

```python
class VideoProcessor:
    def process_video(self, video_id, s3_path):
        """
        Process uploaded video
        """
        # Download video
        local_path = download_from_s3(s3_path)
        
        # Extract metadata
        metadata = self.extract_video_metadata(local_path)
        duration = metadata['duration']  # seconds
        
        # Extract thumbnails at 3 timestamps
        timestamps = [
            duration * 0.1,   # 10% into video
            duration * 0.5,   # Middle
            duration * 0.9    # 90% into video
        ]
        
        thumbnails = []
        for i, ts in enumerate(timestamps):
            thumb_path = f"/tmp/{video_id}_thumb_{i}.jpg"
            self.extract_frame(local_path, ts, thumb_path)
            thumbnails.append(thumb_path)
        
        # Generate animated GIF preview (3 seconds from middle)
        gif_start = max(0, duration * 0.5 - 1.5)
        gif_path = self.generate_gif_preview(
            local_path, 
            start_time=gif_start,
            duration=3,
            fps=10,  # 10 frames/sec
            width=400
        )
        
        # Transcode to web-friendly formats
        web_formats = self.transcode_video(local_path, video_id)
        
        # Upload results
        self.upload_results(video_id, thumbnails, gif_path, web_formats)
        
        return metadata
    
    def extract_frame(self, video_path, timestamp, output_path):
        """Extract single frame at timestamp using FFmpeg"""
        cmd = [
            'ffmpeg',
            '-ss', str(timestamp),  # Seek to timestamp
            '-i', video_path,
            '-vframes', '1',        # Extract 1 frame
            '-q:v', '2',            # High quality
            output_path
        ]
        subprocess.run(cmd, check=True)
    
    def generate_gif_preview(self, video_path, start_time, duration, fps, width):
        """Generate animated GIF preview"""
        output_path = f"/tmp/{uuid.uuid4()}.gif"
        
        # FFmpeg command for GIF generation
        cmd = [
            'ffmpeg',
            '-ss', str(start_time),
            '-t', str(duration),
            '-i', video_path,
            '-vf', f'fps={fps},scale={width}:-1:flags=lanczos',
            '-c:v', 'gif',
            output_path
        ]
        subprocess.run(cmd, check=True)
        
        return output_path
    
    def transcode_video(self, input_path, video_id):
        """
        Transcode to multiple formats/qualities
        
        Outputs:
        - 1080p H.264 (most compatible)
        - 720p H.264 (mobile)
        - 480p H.264 (slow connections)
        """
        outputs = {}
        
        qualities = [
            ('1080p', 1920, 1080, 5000),  # width, height, bitrate (kbps)
            ('720p', 1280, 720, 2500),
            ('480p', 854, 480, 1000)
        ]
        
        for quality_name, width, height, bitrate in qualities:
            output_path = f"/tmp/{video_id}_{quality_name}.mp4"
            
            cmd = [
                'ffmpeg',
                '-i', input_path,
                '-c:v', 'libx264',           # H.264 codec
                '-preset', 'medium',         # Encoding speed
                '-crf', '23',                # Quality (18-28, lower=better)
                '-maxrate', f'{bitrate}k',
                '-bufsize', f'{bitrate*2}k',
                '-vf', f'scale={width}:{height}',
                '-c:a', 'aac',               # Audio codec
                '-b:a', '128k',
                '-movflags', '+faststart',   # Enable streaming
                output_path
            ]
            subprocess.run(cmd, check=True)
            
            outputs[quality_name] = output_path
        
        return outputs
```

**3. Scaling Considerations:**

```text
Processing Requirements:
- 100,000 videos/day = 1.16 videos/second average
- Average video: 1 minute, 1080p
- Processing time: ~3 minutes per video (transcoding bottleneck)

Workers Needed:
- 3 minutes processing × 1.16 videos/sec = ~210 workers
- With 2x peak factor = 420 workers

GPU Acceleration:
- Use NVIDIA NVENC for H.264 encoding
- 10x faster than CPU encoding
- Reduces workers needed to 42

Cost Optimization:
- Use AWS MediaConvert for transcoding ($0.015/minute)
- Self-hosted FFmpeg for thumbnail/GIF extraction
- Hybrid approach based on cost analysis
```

**4. Storage Strategy:**

```text
Per Video (1 minute, 1080p original):
├─ Original: 150 MB (stored in cold storage after processing)
├─ 1080p transcode: 37 MB
├─ 720p transcode: 18 MB  
├─ 480p transcode: 7 MB
├─ Thumbnails (3): 180 KB
└─ GIF preview: 2 MB

Total: 64.2 MB per video
100K videos/day = 6.42 TB/day storage
```

**5. Optimizations:**

- **Keyframe extraction**: Prefer I-frames for better quality
- **Smart timestamp selection**: Use scene detection for better thumbnails
- **Adaptive bitrate**: Analyze content to optimize bitrate
- **Progressive upload**: Stream processed chunks as they're ready
- **Caching**: Cache GIF previews aggressively (high reuse)

</details>

---

### 🎯 Key Takeaways

✅ **Asynchronous Processing is Critical**
- Decouple upload from processing for better UX
- Event-driven architecture scales to billions of operations

✅ **Multi-Format Strategy**
- Generate multiple thumbnail sizes for different contexts
- Support modern formats (WebP, AVIF) with fallbacks (JPEG)
- Adaptive quality based on content and network conditions

✅ **GPU Acceleration at Scale**
- 10-20x cost savings for image/video processing
- Batch processing maximizes GPU utilization
- Essential at Instagram/Google Photos scale

✅ **Metadata Extraction Enables Features**
- EXIF data powers search, organization, and smart features
- Privacy consideration: Strip metadata when sharing

✅ **Progressive Enhancement**
- Generate most important assets first (smallest thumbnail)
- Lazy generation for edge cases
- Graceful degradation when processing fails

✅ **Monitoring & Observability**
- Track processing lag, queue depth, success rates
- Auto-scale based on metrics
- Dead letter queue for failed jobs

✅ **Cost vs Quality Trade-offs**
- Adaptive compression saves 30% storage
- Modern formats (AVIF) reduce bandwidth 40%
- Balance quality with user's device/network capabilities

---

**Ready for Section 8?** Next, we'll explore Face Recognition at Scale - how to detect, embed, and cluster millions of faces while respecting user privacy! 👤

---

## Section 8: Face Recognition at Scale

### 📚 Learning Objectives

By the end of this section, you will understand:

- **🟢 Beginner**: What face recognition is, how it groups people in photos, basic privacy concerns
- **🟡 Intermediate**: Face detection models, embedding generation, clustering algorithms, vector databases
- **🔴 Advanced**: Production-scale face processing (millions of faces), ML model optimization, privacy-first design, GDPR compliance

---

### 🎯 Why This Matters

**The Problem:**
You've uploaded 10,000 photos over 5 years. Finding all photos of your mom requires:
- Manually scrolling through 10,000 photos
- Checking each photo individually
- Hours of tedious work

**The Solution:**
Face recognition automatically:
- Detects all faces in your photos (20,000+ faces found)
- Groups similar faces together (identifies 150 unique people)
- Lets you name people once, find all their photos instantly
- Creates "People" albums automatically

**Real-World Impact:**
- **Google Photos**: Processes billions of faces, identifies millions of unique people
- **Facebook**: Recognizes faces with 97.35% accuracy (better than humans at 97.53%)
- **Apple Photos**: On-device face recognition for privacy (no cloud processing)

**Interview Relevance:**
Face recognition tests understanding of:
- ML pipelines at scale
- Vector similarity search
- Privacy and GDPR compliance
- Clustering algorithms for unsupervised learning
- Trade-offs between accuracy and privacy

---

### 🟢 Beginner Level: Face Recognition Fundamentals

#### What is Face Recognition?

**Analogy: The School Yearbook**
Imagine organizing your school yearbook:

1. **Detection**: Find all faces in group photos (like circling faces)
2. **Recognition**: Identify who each person is
3. **Grouping**: Put all photos of the same person together
4. **Naming**: Label each group with a name

**In Google Photos:**

```text
Step 1: Upload family vacation photos (100 photos)
        ↓
Step 2: System finds 347 faces across all photos
        ↓
Step 3: Groups similar faces (finds 8 unique people)
        ↓
Step 4: You name them: "Mom", "Dad", "Sister", etc.
        ↓
Step 5: Search "Mom" → See all 87 photos with her!
```

#### The Three Core Steps

**1. Face Detection**
- **What**: Find where faces are in a photo
- **Output**: Bounding boxes around each face
- **Analogy**: Like drawing a square around each person's face in a group photo

```text
Photo with 3 people:
┌─────────────────────────┐
│   ┌────┐                │
│   │ 👤 │  ┌────┐        │  
│   └────┘  │ 👤 │  ┌────┐│
│           └────┘  │ 👤 ││
│                   └────┘│
└─────────────────────────┘
   Face 1   Face 2  Face 3
```

**2. Feature Extraction (Embedding)**
- **What**: Convert face image to a unique "fingerprint" (numbers)
- **Output**: 512 numbers that represent this specific face
- **Analogy**: Like a barcode that uniquely identifies a product

```text
Mom's face → ML Model → [0.23, -0.45, 0.67, ..., 0.12]
                        └─ 512 numbers (face embedding)

Different photo of Mom → [0.24, -0.44, 0.68, ..., 0.11]
                          └─ Very similar numbers!

Stranger's face → [0.89, 0.12, -0.34, ..., 0.78]
                   └─ Completely different numbers
```

**3. Clustering (Grouping)**
- **What**: Group similar embeddings together
- **Output**: "Person 1" has 87 faces, "Person 2" has 45 faces, etc.
- **Analogy**: Like sorting mixed coins into piles (pennies, nickels, dimes)

#### Why Use Numbers (Embeddings)?

**Human Approach (Doesn't Scale):**
```text
Compare faces pixel-by-pixel:
- Mom's photo 1: 4032×3024 = 12,192,768 pixels
- Mom's photo 2: Compare all 12 million pixels
- Time: Very slow, not reliable
```

**ML Approach (Scales):**
```text
Convert to embeddings:
- Mom's photo 1 → 512 numbers
- Mom's photo 2 → 512 numbers
- Compare: Just 512 numbers!
- Time: 0.001 seconds, highly accurate
```

#### Simple Face Recognition Flow

```mermaid
graph LR
    Photo[Upload Photo] --> Detect[Find Faces<br/>3 faces found]
    Detect --> Extract[Extract Features<br/>512 numbers per face]
    Extract --> Compare[Compare to Existing]
    Compare --> Match{Match Found?}
    
    Match -->|Yes| Existing[Add to Person Group<br/>"Mom"]
    Match -->|No| New[Create New Person<br/>"Person 42"]
    
    Existing --> Album[Show in Albums]
    New --> Album
```

#### Privacy: What You Control

**User Controls:**
1. **Opt-in Required**: Face recognition OFF by default
2. **Delete Anytime**: Remove all face data with one click
3. **Name Control**: Only you see names you've added
4. **No Sharing**: Face data never shared with others
5. **Local Processing** (option): Process faces on your device, not cloud

**What Google Photos Stores:**

```text
✅ Stores:
├─ Face bounding boxes (where face is in photo)
├─ Face embeddings (512 numbers per face)
└─ Person clusters (which faces belong together)

❌ Doesn't Store:
├─ Names (unless you add them)
├─ Biometric templates (different from embeddings)
└─ Cross-user face data (your faces ≠ other users' faces)
```

---

### 🟡 Intermediate Level: Face Recognition Architecture

#### Production Pipeline Architecture

```mermaid
graph TB
    Upload[Photo Uploaded] --> Event[Kafka: photo.processed]
    Event --> FaceWorker[Face Detection Worker]
    
    FaceWorker --> Detector[Face Detector<br/>MTCNN/RetinaFace]
    Detector --> Quality[Quality Filter<br/>Size, blur, angle]
    
    Quality -->|Good| Alignment[Face Alignment<br/>Normalize rotation/scale]
    Quality -->|Poor| Skip[Skip - too blurry]
    
    Alignment --> Embedding[Embedding Model<br/>FaceNet/ArcFace]
    Embedding --> Vector[512D Vector]
    
    Vector --> Milvus[(Milvus Vector DB)]
    Vector --> Postgres[(PostgreSQL<br/>Face Metadata)]
    
    Milvus --> Search[Similarity Search]
    Search --> Cluster[DBSCAN Clustering]
    Cluster --> PersonGroup[Person Groups]
    
    PersonGroup --> UI[User Interface]
```

#### Face Detection with MTCNN

**Multi-Task Cascaded Convolutional Networks (MTCNN):**

3-stage cascade for accurate detection:

```text
Stage 1: Proposal Network (P-Net)
├─ Quick scan at multiple scales
├─ Finds candidate face regions
└─ Outputs: 1000s of proposals

Stage 2: Refine Network (R-Net)
├─ Filters false positives
├─ Refines bounding boxes
└─ Outputs: ~100 candidates

Stage 3: Output Network (O-Net)
├─ Final classification
├─ Precise bounding boxes
├─ 5 facial landmarks (eyes, nose, mouth corners)
└─ Outputs: Final faces with confidence scores
```

**Code Example:**

```python
from mtcnn import MTCNN
import cv2

detector = MTCNN()

def detect_faces(image_path):
    """
    Detect faces in image using MTCNN
    
    Returns:
        List of face detections with bounding boxes and landmarks
    """
    image = cv2.imread(image_path)
    image_rgb = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)
    
    # Detect faces
    detections = detector.detect_faces(image_rgb)
    
    faces = []
    for detection in detections:
        # Only keep high-confidence detections
        if detection['confidence'] > 0.90:
            faces.append({
                'box': detection['box'],  # [x, y, width, height]
                'confidence': detection['confidence'],
                'keypoints': detection['keypoints'],  # Eyes, nose, mouth
                'quality_score': calculate_face_quality(detection)
            })
    
    return faces

def calculate_face_quality(detection):
    """
    Assess face quality for recognition
    
    Factors:
    - Size (larger = better)
    - Blur (sharper = better)
    - Angle (frontal = better)
    - Occlusion (visible = better)
    """
    box = detection['box']
    size_score = min(box[2] * box[3] / (80 * 80), 1.0)  # Normalize to 80x80 min
    
    # Placeholder for other quality metrics
    blur_score = 0.9  # Would use Laplacian variance
    angle_score = 0.95  # Would use facial landmarks
    
    quality = (size_score + blur_score + angle_score) / 3
    return quality
```

**Quality Filtering:**

```python
def filter_quality_faces(faces):
    """
    Filter faces by quality criteria
    
    Requirements:
    - Minimum size: 80x80 pixels
    - Confidence: >0.90
    - Quality score: >0.70
    - Not heavily occluded
    """
    filtered = []
    
    for face in faces:
        box = face['box']
        width, height = box[2], box[3]
        
        # Size check
        if width < 80 or height < 80:
            continue
        
        # Confidence check
        if face['confidence'] < 0.90:
            continue
        
        # Quality check
        if face['quality_score'] < 0.70:
            continue
        
        filtered.append(face)
    
    return filtered
```

#### Face Embedding with FaceNet

**FaceNet Architecture:**
- **Model**: Inception-ResNet-v1
- **Input**: 160×160 RGB face image (aligned)
- **Output**: 512-dimensional embedding vector
- **Training**: Triplet loss (anchor, positive, negative)

**Embedding Generation:**

```python
import torch
from facenet_pytorch import InceptionResnetV1, MTCNN
import numpy as np

class FaceEmbedder:
    def __init__(self):
        # Load pre-trained FaceNet model
        self.model = InceptionResnetV1(
            pretrained='vggface2'
        ).eval()
        
        if torch.cuda.is_available():
            self.model = self.model.cuda()
    
    def generate_embedding(self, face_image):
        """
        Generate 512D embedding from aligned face image
        
        Args:
            face_image: Aligned face image (160x160)
        
        Returns:
            512-dimensional numpy array
        """
        # Preprocess
        face_tensor = torch.from_numpy(face_image).float()
        face_tensor = face_tensor.permute(2, 0, 1)  # HWC -> CHW
        face_tensor = (face_tensor - 127.5) / 128.0  # Normalize to [-1, 1]
        face_tensor = face_tensor.unsqueeze(0)  # Add batch dimension
        
        if torch.cuda.is_available():
            face_tensor = face_tensor.cuda()
        
        # Generate embedding
        with torch.no_grad():
            embedding = self.model(face_tensor)
        
        # L2 normalize
        embedding = torch.nn.functional.normalize(embedding, p=2, dim=1)
        
        return embedding.cpu().numpy()[0]
    
    def compare_embeddings(self, emb1, emb2):
        """
        Compare two embeddings using cosine similarity
        
        Returns:
            Similarity score (0-1, higher = more similar)
            Distance threshold: <0.6 = same person
        """
        # Cosine similarity
        similarity = np.dot(emb1, emb2) / (
            np.linalg.norm(emb1) * np.linalg.norm(emb2)
        )
        
        # Convert to distance
        distance = 1 - similarity
        
        return {
            'similarity': similarity,
            'distance': distance,
            'is_same_person': distance < 0.6
        }
```

**Embedding Properties:**

```text
Same Person:
  Photo 1: [0.234, -0.456, 0.678, ..., 0.123]
  Photo 2: [0.236, -0.454, 0.680, ..., 0.125]
  Distance: 0.42 → Same person! ✓

Different People:
  Person A: [0.234, -0.456, 0.678, ..., 0.123]
  Person B: [0.789, 0.123, -0.345, ..., 0.890]
  Distance: 1.18 → Different people ✓

Threshold:
  Distance < 0.6 → Same person
  Distance > 1.0 → Different people
  0.6 - 1.0 → Uncertain (manual review)
```

#### Vector Database with Milvus

**Why Vector Database?**

Traditional databases can't efficiently search embeddings:

```text
PostgreSQL (traditional):
├─ 1 million faces × 512 numbers = 512 million values
├─ Full scan to find similar faces
└─ Search time: ~30 seconds ❌

Milvus (vector DB):
├─ Same 1 million faces × 512 dimensions
├─ HNSW index for fast similarity search
└─ Search time: ~50 milliseconds ✓ (600x faster!)
```

**Milvus Collection Setup:**

```python
from pymilvus import connections, Collection, CollectionSchema, FieldSchema, DataType

def create_face_collection():
    """
    Create Milvus collection for face embeddings
    """
    # Connect to Milvus
    connections.connect(
        alias="default",
        host='milvus-server',
        port='19530'
    )
    
    # Define schema
    fields = [
        FieldSchema(name="face_id", dtype=DataType.VARCHAR, max_length=64, is_primary=True),
        FieldSchema(name="user_id", dtype=DataType.VARCHAR, max_length=64),
        FieldSchema(name="photo_id", dtype=DataType.VARCHAR, max_length=64),
        FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=512),
        FieldSchema(name="quality_score", dtype=DataType.FLOAT),
        FieldSchema(name="detected_at", dtype=DataType.INT64)
    ]
    
    schema = CollectionSchema(
        fields=fields,
        description="Face embeddings for recognition"
    )
    
    # Create collection
    collection = Collection(
        name="face_embeddings",
        schema=schema,
        using='default',
        shards_num=4  # Partitions for parallelism
    )
    
    # Create index for fast similarity search
    index_params = {
        "metric_type": "COSINE",  # Cosine similarity
        "index_type": "HNSW",     # Hierarchical Navigable Small World
        "params": {
            "M": 16,              # Number of connections per layer
            "efConstruction": 200  # Build-time accuracy
        }
    }
    
    collection.create_index(
        field_name="embedding",
        index_params=index_params
    )
    
    return collection
```

Due to the length of this batch, let me save this and continue in the next message. Section 8 is progressing well!
    
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
```

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
```

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
  
  ```

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
```

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
```

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
```

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
```

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
```

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
```

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
```

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
```

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
```

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
```

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
```

**Write-Through (for critical data):**

```python
def update_photo_metadata(photo_id, metadata):
    # Update database
    cassandra.execute("UPDATE photos SET ... WHERE photo_id = ?", photo_id, metadata)
    
    # Update cache immediately
    redis.setex(f"photo:{photo_id}", 86400, metadata)
```

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

#### Trade-Off 1: SQL vs NoSQL for Metadata

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

#### Trade-Off 2: Synchronous vs Asynchronous Processing

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

#### Trade-Off 3: Storage Location - S3 vs Custom Storage

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

#### Trade-Off 4: Search - Elasticsearch vs Database Search

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

#### Trade-Off 5: Video Processing

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

#### Bottleneck 1: Database Write Contention

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

#### Bottleneck 2: S3 Request Rate

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

#### Bottleneck 3: Thumbnail Generation Lag

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

#### Bottleneck 4: Search Index Lag

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

#### Bottleneck 5: Single Points of Failure

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

#### 1. Geographic Distribution

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
```

---

#### 2. Advanced Caching

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
```

---

#### 3. Real-Time Features

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
```

**Benefits:**

- Real-time upload progress (eliminate polling)
- Instant sharing notifications
- Live album collaboration

---

#### 4. Query Optimization

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
```

**Cassandra Materialized Views:**

```sql
-- Favorite photos view
CREATE MATERIALIZED VIEW user_favorites AS
  SELECT * FROM photos
  WHERE user_id IS NOT NULL 
    AND is_favorite = true
    AND photo_id IS NOT NULL
  PRIMARY KEY (user_id, is_favorite, upload_date, photo_id);
```

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
```

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
```

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

#### 1. Authentication & Authorization

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
```

---

#### 2. Data Protection

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

#### 3. DDoS Protection

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
```

---

#### 4. Face Recognition Privacy

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
```

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
```

#### 5. Secure Sandbox

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
