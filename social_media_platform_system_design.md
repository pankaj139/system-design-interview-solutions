# Social Media Platform System Design (Instagram/Twitter-like)

**Difficulty Level:** ⭐⭐⭐⭐⭐ Expert  
**Tags:** `Social Network`, `Newsfeed`, `Fan-out`, `Media Processing`, `Graph Database`, `Real-time Updates`, `WebSocket`, `Recommendation Engine`, `Content Moderation`, `Distributed Systems`, `High Throughput`, `Global Scale`, `ML/AI`

**File Purpose:** Interactive, multi-level learning resource for designing a large-scale social media platform. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that supports 500M daily active users, handles 200M posts per day, serves 10B feed impressions per day, processes media with 100K workers, and achieves 99.9% availability with <100ms feed load time globally.

**Author:** System Design Documentation  
**Created:** October 2, 2025  
**Last Updated:** November 12, 2025  
**Recent Updates:** Complete rewrite in educational template format with multi-level learning paths (🟢🟡🔴)

---

## 🎓 Welcome to Social Media Platform System Design!

### What You're Going to Build

Imagine building the social media platform that connects billions of people worldwide—like Instagram (1.4B users sharing 100M photos daily), Twitter (450M users posting 500M tweets daily), or TikTok (1B users watching 1B hours of video daily). You're designing a system that handles everything from photo uploads to real-time feed generation, from celebrity fan-out strategies to ML-powered content recommendations, processing 200 million posts every single day while serving personalized feeds to 500 million active users in under 100 milliseconds!

By the end of this learning journey, you'll understand how to design a production-grade social media platform that:

- **Handles massive scale**: 500M daily active users, 200M posts/day, 10B feed impressions/day, 12 exabytes of media storage
  - **What this means for beginners**: Imagine every photo, video, and story shared on Instagram in a single day—that's 200 million pieces of content! Your system needs to store them, process them, and make them instantly available to billions of users worldwide.
  - **How we achieve it**: We use **hybrid fan-out** (pre-computing feeds for regular users, computing on-demand for celebrities), **media processing pipelines** (100K workers processing images/videos in parallel), **CDN distribution** (caching content at 100+ edge locations globally), and **sharded databases** (splitting data across thousands of servers).

- **Generates personalized feeds**: ML-powered ranking algorithms, real-time updates, celebrity problem solutions
  - **What this means for beginners**: When you open Instagram, you see posts from people you follow, but they're not in chronological order—they're ranked by what Instagram thinks you'll like most! This "magic" is powered by machine learning algorithms that learn your preferences.
  - **How feed generation works**:
    - **Fan-out on write**: When a regular user posts, we immediately push it to all their followers' feeds (like delivering newspapers to subscribers)
    - **Fan-out on read**: When a celebrity posts, we compute their feed on-demand (like a library—you fetch books when needed, not pre-deliver to everyone)
    - **ML ranking**: Posts are scored by engagement probability, recency, relationship strength, and content quality
  - **The celebrity problem**: A celebrity with 100M followers would require 100M write operations if we fan-out on write—that's why we use fan-out on read for high-follower accounts!

- **Processes media at scale**: Image compression, video transcoding, thumbnail generation, 100K worker pipeline
  - **What this means for beginners**: When you upload a 10MB photo, Instagram doesn't store it as-is. It creates multiple sizes (thumbnail, medium, full), compresses it, and stores optimized versions. This saves 90% storage and makes photos load 10x faster!
  - **Media processing pipeline**:
    - **Upload**: User uploads original media (2MB photo, 50MB video)
    - **Processing**: Workers resize, compress, transcode, generate thumbnails
    - **Storage**: Multiple formats stored in object storage (S3)
    - **CDN**: Content cached at edge locations for fast delivery
  - **Why it's complex**: Processing 200M posts/day requires 100K workers running 24/7, handling failures, retries, and quality checks

- **Provides real-time updates**: WebSocket connections, instant notifications, live engagement counters
  - **What this means for beginners**: When someone likes your post, you see the like count update instantly—not after refreshing the page. This "real-time magic" uses WebSocket connections that stay open between your phone and Instagram's servers.
  - **Real-time architecture**:
    - **WebSocket servers**: Maintain persistent connections with 100M+ concurrent users
    - **Redis Pub/Sub**: Broadcasts engagement events (likes, comments) to all connected clients
    - **Push notifications**: Alerts users when they're offline (FCM for Android, APNs for iOS)
  - **Scale challenge**: Maintaining 100M WebSocket connections requires specialized servers and connection pooling strategies

- **Achieves high availability**: 99.9% uptime (8.76 hours downtime/year), multi-region deployment, automatic failover
  - **What this means for beginners**: Instagram is available 99.9% of the time—that means only 8.76 hours of downtime per year (less than a workday)! Even if one data center fails, the system continues running from other locations.
  - **High availability strategies**:
    - **Multi-region deployment**: Data centers in US, Europe, Asia—if one fails, others take over
    - **Replication**: Every post stored in 3+ locations (primary + replicas)
    - **Load balancing**: Traffic distributed across thousands of servers
    - **Circuit breakers**: Prevent cascading failures when one service is down
  - **Disaster recovery**: Automated backups, point-in-time recovery, and failover procedures ensure zero data loss

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (6-8 hours)
├─ Learn what social media platforms are and how they work
├─ Understand core concepts: feeds, fan-out, media processing
├─ Build intuition with everyday analogies (newspapers, libraries, restaurants)
├─ Master the fundamentals of feed generation and user engagement
└─ Perfect for: New to system design or social media architecture

🟡 INTERMEDIATE LEVEL (8-10 hours)  
├─ Master system design interview frameworks
├─ Learn to make technical trade-offs (write vs read optimization)
├─ Understand hybrid fan-out strategies and celebrity problem solutions
├─ Practice back-of-envelope calculations (traffic, storage, bandwidth)
└─ Perfect for: Preparing for FAANG system design interviews

🔴 ADVANCED LEVEL (10-14 hours)
├─ Deep-dive into ML-powered ranking algorithms and recommendation engines
├─ Understand media processing pipelines and CDN architectures
├─ Master production considerations (monitoring, security, content moderation)
├─ Learn from real-world case studies (Instagram, Twitter, TikTok architectures)
└─ Perfect for: Senior engineers and architects building social platforms
```

**Total Learning Time:** 24-32 hours for complete mastery across all levels

### 🎯 Prerequisites

**For Beginners:**

- Basic programming knowledge (any language)
- Understanding of web applications and databases
- No distributed systems experience needed!

**For Intermediate:**

- Familiarity with REST APIs and databases (SQL/NoSQL)
- Basic understanding of caching and load balancing
- Exposure to microservices concepts

**For Advanced:**

- Experience with distributed systems and scalability patterns
- Understanding of consistency models (eventual, strong)
- Knowledge of machine learning basics (for ranking algorithms)
- Familiarity with media processing and CDN architectures

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:

1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context
3. **Multi-Level Content** - Tailored explanations for your level (🟢🟡🔴)
4. **Real-World Examples** - How Instagram, Twitter, TikTok actually do it
5. **Think About It** - Questions to deepen understanding
6. **Key Takeaways** - Summary of main points
7. **Practice Exercise** - Hands-on challenge

💡 **Pro Tip:** Don't skip the "Think About It" sections - they're designed to help you internalize concepts so you can explain them in interviews or to your team!

---

## 📚 BEGINNER'S GLOSSARY: Technical Terms Explained

Before diving in, here are key technical terms you'll encounter (with everyday analogies):

### Core Social Media Terms

- **Feed/Timeline**: A personalized stream of posts from accounts you follow, ranked by relevance. *Like your personalized newspaper—you only see articles from sources you subscribe to, and they're ordered by what you're most likely to read first.*

- **Fan-out**: The process of distributing a new post to all followers' feeds. *Like a newspaper publisher delivering copies to all subscribers—when a journalist writes an article, it appears in all subscribers' newspapers.*

- **Fan-out on Write**: Pre-computing and storing posts in followers' feeds when a post is created. *Like pre-delivering newspapers to all subscribers' mailboxes immediately after printing.*

- **Fan-out on Read**: Computing a user's feed on-demand when they request it. *Like a library—you fetch books when needed, not pre-deliver to everyone.*

- **Hybrid Fan-out**: Using fan-out on write for regular users and fan-out on read for celebrities. *Like delivering newspapers to regular subscribers but letting celebrities' followers pick up copies from the library when they want.*

### Media Processing Terms

- **Transcoding**: Converting video from one format to another (e.g., MP4 to HLS for streaming). *Like converting a book from English to Spanish—same content, different format.*

- **Thumbnail**: A small preview image of a video or photo. *Like a book cover—gives you a quick preview before opening.*

- **CDN (Content Delivery Network)**: A network of servers distributed globally to cache and serve content closer to users. *Like having bookstores in every city instead of one central library—much faster to get books.*

### Engagement Terms

- **Like**: A simple engagement action indicating approval. *Like giving a thumbs-up.*

- **Comment**: Text-based engagement allowing users to respond to posts. *Like writing a note on someone's post.*

- **Share**: Reposting content to your own feed. *Like forwarding an email to your contacts.*

- **Story**: Temporary content that expires after 24 hours. *Like writing on a whiteboard that gets erased daily.*

### Database Terms

- **Sharding**: Splitting a database into smaller pieces distributed across multiple servers. *Like splitting a library's books across multiple buildings—each building has a different section.*

- **Replication**: Copying data to multiple servers for redundancy. *Like making photocopies of important documents and storing them in different locations.*

- **Graph Database**: A database optimized for relationships (follows, likes, comments). *Like a social network map showing who knows whom.*

### Real-time Terms

- **WebSocket**: A persistent connection between client and server for real-time communication. *Like a phone call that stays open—both sides can talk anytime without hanging up.*

- **Pub/Sub (Publish-Subscribe)**: A messaging pattern where publishers send messages to subscribers via a message broker. *Like a radio station—broadcasters send signals, listeners tune in.*

---

## TABLE OF CONTENTS

- [Section 1: Understanding Requirements & Clarification](#section-1-understanding-requirements--clarification)
- [Section 2: Planning for Scale (Capacity Estimation)](#section-2-planning-for-scale-capacity-estimation)
- [Section 3: Designing the System Architecture](#section-3-designing-the-system-architecture)
- [Section 4: Database Design & Data Modeling](#section-4-database-design--data-modeling)
- [Section 5: API Design](#section-5-api-design)
- [Section 6: Feed Generation & Ranking](#section-6-feed-generation--ranking)
- [Section 7: Media Processing & Storage](#section-7-media-processing--storage)
- [Section 8: Real-time Features (WebSocket & Notifications)](#section-8-real-time-features-websocket--notifications)
- [Section 11: Growing the System (Scalability)](#section-11-growing-the-system-scalability)
- [Section 12: Protecting the System (Security)](#section-12-protecting-the-system-security)
- [Section 13: Keeping It Healthy (Monitoring)](#section-13-keeping-it-healthy-monitoring)
- [Section 14: Making Design Decisions](#section-14-making-design-decisions)
- [Section 15: Interview Preparation & Practice](#section-15-interview-preparation--practice)
- [Putting It All Together](#putting-it-all-together)
- [Resources for Further Learning](#resources-for-further-learning)
- [Congratulations!](#congratulations)

---

## Section 1: Understanding Requirements & Clarification

### What You'll Learn

By the end of this section, you'll be able to:

- Identify functional and non-functional requirements for a social media platform
- Ask clarifying questions during system design interviews
- Understand the scale challenges of serving 500M daily active users
- Distinguish between MVP features and future enhancements

### Why This Matters

Requirements gathering is the foundation of any system design. Getting this wrong means building the wrong system! Real-world example: Instagram started as a simple photo-sharing app, but as it scaled to 1.4B users, requirements evolved—stories, reels, shopping, messaging. Understanding how to gather and prioritize requirements is crucial for both interviews and real-world projects.

---

### 🟢 For Beginners: The Fundamentals

#### What is a Social Media Platform?

Think of a social media platform like Instagram or Twitter as a digital town square where people:

1. **Share content**: Photos, videos, text posts, stories
2. **Connect with others**: Follow friends, celebrities, brands
3. **Engage**: Like, comment, share, save posts
4. **Discover**: Find new content through feeds, hashtags, recommendations

**Key Components Explained:**

- **Users**: People who create accounts and share content
- **Posts**: Content shared by users (photos, videos, text)
- **Feed**: A personalized stream of posts from accounts you follow
- **Followers**: People who subscribe to see your content
- **Engagement**: Actions like likes, comments, shares

**Real-World Analogy**: Imagine a newspaper where:
- **Journalists** = Users (create content)
- **Articles** = Posts (content shared)
- **Subscribers** = Followers (people who see your content)
- **Newspaper delivery** = Feed generation (distributing content to followers)
- **Letters to the editor** = Comments (engagement)

---

### 🟡 For Intermediate: Interview Patterns

#### Requirements Gathering Framework

In system design interviews, always start with requirements:

**1. Functional Requirements (WHAT the system does)**

Ask: "What features should the system support?"

**MVP Features:**
- User registration and authentication
- Create and share posts (photos/videos)
- Follow/unfollow users
- View personalized feed
- Like and comment on posts
- View user profiles

**Future Enhancements:**
- Direct messaging
- Stories (24-hour content)
- Live streaming
- Video calls
- Shopping/e-commerce
- Reels/short videos

**2. Non-Functional Requirements (HOW WELL the system performs)**

Ask: "What are the performance, scalability, and reliability requirements?"

**Key Metrics:**
- **Scale**: 500M DAU, 200M posts/day, 10B feed impressions/day
- **Performance**: <100ms feed load time, <2s upload acknowledgment
- **Availability**: 99.9% uptime (8.76 hours downtime/year)
- **Consistency**: Eventual consistency for feeds (acceptable), strong consistency for user actions

**3. Clarifying Questions**

Always ask about:
- **Geographic distribution**: Where are users located?
- **Read-to-write ratio**: How many reads vs writes?
- **Media file sizes**: Average photo/video sizes?
- **Peak traffic**: What's the peak-to-average ratio?
- **Celebrity accounts**: How many users have >1M followers?

---

### 🔴 For Advanced: Production Considerations

#### Requirements Prioritization & Trade-offs

**MVP vs Future Features:**

**MVP (Must Have):**
- Core posting and feed functionality
- Basic engagement (likes, comments)
- User profiles and follows

**Phase 2 (Should Have):**
- Stories feature
- Search and discovery
- Push notifications

**Phase 3 (Nice to Have):**
- Direct messaging
- Live streaming
- Shopping integration

**Trade-offs in Requirements:**

1. **Feed Consistency**: Eventual vs Strong
   - **Eventual**: Acceptable for feeds (users don't notice 1-2 second delay)
   - **Strong**: Required for financial operations (payments, purchases)

2. **Media Quality**: Storage vs Bandwidth
   - **High quality**: Better user experience, but 10x storage/bandwidth costs
   - **Optimized**: 90% storage savings, but requires processing pipeline

3. **Real-time Updates**: WebSocket vs Polling
   - **WebSocket**: Real-time, but requires maintaining 100M+ connections
   - **Polling**: Simpler, but higher latency and server load

---

### User Stories

- **As a user**, I want to create and share photos/videos so that I can express myself and connect with others
- **As a user**, I want to follow other users so that I can see their content in my feed
- **As a user**, I want to like, comment, and share posts so that I can engage with content
- **As a user**, I want to view a personalized feed so that I see relevant content quickly
- **As a content creator**, I want my posts to reach my followers instantly so that I can maintain engagement
- **As a celebrity user**, I want to efficiently broadcast to millions of followers without system degradation

### Functional Requirements

**Core Features (MVP):**

1. **User Management**
   - User registration and authentication
   - Profile creation and management
   - Follow/unfollow functionality

2. **Content Creation**
   - Upload photos and videos
   - Create stories (24-hour expiration)
   - Add captions, hashtags, and mentions
   - Support multiple media formats

3. **Feed Generation**
   - Home timeline (posts from followed users)
   - Explore feed (trending/recommended content)
   - User profile feed

4. **Engagement Features**
   - Like posts
   - Comment on posts
   - Share posts
   - Hashtag browsing and trending topics

5. **Media Processing**
   - Image resizing and optimization
   - Video transcoding
   - Thumbnail generation
   - CDN distribution

### Non-Functional Requirements

1. **Availability:** 99.9% uptime for core features (8.76 hours downtime/year)
2. **Performance:**
   - Feed loads in <500ms (p95)
   - Media upload acknowledgment in <2s
   - Real-time updates for likes/comments (<1s delay)
3. **Scalability:**
   - Support 500M daily active users
   - Handle 200M posts per day
   - Serve 10B feed impressions per day
   - Support celebrity accounts with 100M+ followers
4. **Consistency:**
   - Eventual consistency for feeds (acceptable)
   - Strong consistency for financial operations (if any)
   - Immediate consistency for user actions (like/unlike)
5. **Durability:**
   - Zero data loss for uploaded media
   - Reliable delivery of notifications

### Clarifying Questions & Assumptions

**Scale Questions:**

- **Q:** What's the geographic distribution?
  - **A:** Global platform with concentration in US, Europe, India, and Southeast Asia
- **Q:** What's the read-to-write ratio?
  - **A:** Very read-heavy (100:1 ratio - users browse more than post)
- **Q:** Average media file sizes?
  - **A:** Photos ~2MB, Videos ~50MB, Stories ~10MB

**Feature Scope:**

- **Q:** Do we need direct messaging?
  - **A:** Out of scope for MVP
- **Q:** Do we need video calls or live streaming?
  - **A:** Out of scope for MVP
- **Q:** Support for ads?
  - **A:** Architecture should accommodate future ad insertion

**Assumptions:**

1. Average user follows 200 accounts
2. Celebrity accounts: 0.1% of users have >1M followers
3. 40% of DAU post content (200M posts from 500M DAU)
4. Peak traffic is 3x average (US evening hours)
5. Users scroll through 50 posts per session on average
6. Stories have 24-hour TTL and are viewed more frequently than posts
7. Average post has 5 hashtags
8. Mobile-first platform (80% mobile, 20% web)

---

## Section 2: Planning for Scale (Capacity Estimation)

### What You'll Learn

By the end of this section, you'll be able to:

- Calculate traffic estimates (QPS, peak traffic) for a social media platform
- Estimate storage requirements (user data, posts, media) over time
- Calculate bandwidth needs (upload and download) for media-heavy systems
- Perform back-of-the-envelope calculations that impress interviewers
- Understand the relationship between scale metrics and infrastructure costs

### Why This Matters

Capacity planning is where you prove you can think at scale. Real-world example: Instagram started with simple calculations, but as they grew to 1.4B users, their storage calculations determined whether they needed 1 data center or 10—getting this wrong means millions in wasted infrastructure or catastrophic failures! In interviews, showing you can do these calculations demonstrates you understand real-world constraints.

---

### 🟢 For Beginners: The Fundamentals

#### What is Capacity Planning?

Think of capacity planning like planning a restaurant:

- **Traffic estimates**: How many customers will come? (Like calculating how many people will use your app)
- **Storage**: How much food do we need? (Like how much data we need to store)
- **Bandwidth**: How fast can we serve customers? (Like how fast we can deliver content)
- **Resources**: How many chefs and waiters do we need? (Like how many servers we need)

**Real-World Analogy**: Imagine Instagram needs to store photos. If 1 million users upload 1 photo per day, and each photo is 2MB:
- Daily storage: 1M × 2MB = 2TB per day
- Monthly storage: 2TB × 30 = 60TB per month
- Yearly storage: 60TB × 12 = 720TB per year

That's why Instagram uses compression and multiple storage tiers!

#### Understanding the Numbers

**Daily Active Users (DAU)**: Users who use the app each day
- Instagram: 500M DAU
- This means 500 million people open Instagram every single day!

**Posts per Day**: How much content is created
- Instagram: 200M posts/day
- That's 200 million photos/videos shared daily!

**Feed Impressions**: How many times feeds are viewed
- Instagram: 10B impressions/day
- Each user views their feed multiple times per day

**Why These Numbers Matter**: 
- More users = more servers needed
- More posts = more storage needed
- More impressions = more bandwidth needed

---

### 🟡 For Intermediate: Interview Patterns

#### The Capacity Planning Framework

In system design interviews, always follow this structure:

**Step 1: Traffic Estimates**
```text
1. Identify key operations (posts, feed views, likes)
2. Calculate average QPS (queries per second)
3. Calculate peak QPS (usually 3x average)
4. Show your math clearly
```

**Step 2: Storage Estimates**
```text
1. Calculate data per operation (post size, user data size)
2. Calculate daily storage growth
3. Calculate retention (how long to keep data)
4. Total storage = daily growth × retention period
```

**Step 3: Bandwidth Estimates**
```text
1. Upload bandwidth (users uploading content)
2. Download bandwidth (users viewing content)
3. Account for CDN cache hit ratio (90%+)
4. Calculate actual origin bandwidth needed
```

**Step 4: Resource Estimates**
```text
1. Calculate servers needed (based on QPS and server capacity)
2. Account for redundancy (3x replication)
3. Add 20% buffer for peak traffic
4. Calculate total infrastructure cost
```

#### Interview Script: Capacity Planning

**When the interviewer says "Design Instagram":**

**You say:** "Let me start by understanding the scale. Can I assume:
- 500M daily active users?
- 200M posts per day?
- 10B feed impressions per day?"

**Then calculate:**

```text
Posts per second (avg): 200M / 86,400 = ~2,315/sec
Posts per second (peak): 2,315 × 3 = ~7,000/sec

Feed requests per second (avg): 10B / 86,400 = ~115,740/sec
Feed requests per second (peak): 115,740 × 3 = ~347,000/sec
```

**⚠️ Common Mistake:** Forgetting to account for peak traffic (3x multiplier). Always mention: "I'm assuming peak traffic is 3x average, which is typical for social media platforms."

---

### 🔴 For Advanced: Production Considerations

#### Production-Grade Capacity Planning

**Beyond Basic Calculations:**

**1. Geographic Distribution**
```text
Traffic Distribution:
├─ US: 30% of traffic
├─ Europe: 25% of traffic
├─ Asia: 35% of traffic
└─ Other: 10% of traffic

Implication: Need data centers in each region
Cost Impact: 3x infrastructure (vs single region)
```

**2. Media Processing Costs**
```text
200M posts/day × 17.2MB avg = 3,440TB/day upload
Processing: 100K workers × $0.10/hour = $10K/hour = $240K/day
Storage: 3,440TB × $0.023/GB/month = $79K/day
Total: ~$320K/day = $9.6M/month for media processing
```

**3. CDN Cost Optimization**
```text
Without CDN: 16.5Pbps download = $50M/month bandwidth
With CDN (95% hit rate): 825Tbps origin = $2.5M/month
Savings: $47.5M/month = $570M/year!
```

**4. Storage Tiering Strategy**
```text
Hot Storage (S3 Standard): Recent 30 days = 103PB
├─ Cost: $2.3M/month
├─ Access: <100ms latency

Warm Storage (S3 IA): 30-90 days = 206PB
├─ Cost: $1.2M/month
├─ Access: <500ms latency

Cold Storage (Glacier): 90+ days = 515PB
├─ Cost: $0.3M/month
├─ Access: <12 hours retrieval

Total Savings: 67% vs all-hot storage
```

---

## Back-of-the-Envelope Calculations

### Traffic Estimates

**Daily Active Users (DAU):**

```text
DAU: 500,000,000 users
Monthly Active Users (MAU): ~1.5B users (assuming 3x DAU)
```

**Post Creation:**

```text
Posts per day: 200,000,000
Posts per second (avg): 200M / 86,400 = ~2,315 posts/sec
Posts per second (peak): 2,315 * 3 = ~7,000 posts/sec

Breakdown:
- Photos: 60% = 120M/day = 1,400/sec (peak: 4,200/sec)
- Videos: 30% = 60M/day = 700/sec (peak: 2,100/sec)
- Stories: 10% = 20M/day = 230/sec (peak: 700/sec)
```

**Feed Generation:**

```text
Feed impressions per day: 10,000,000,000 (10B)
Feed requests per second (avg): 10B / 86,400 = ~115,740 req/sec
Feed requests per second (peak): 115,740 * 3 = ~347,000 req/sec

Average posts viewed per user per day: 10B / 500M = 20 posts
Sessions per user per day: 20 / 50 posts per session = ~5 sessions
```

**Engagement Actions:**

```text
Assuming:
- 50% of viewed posts get liked
- 10% of viewed posts get commented
- 5% of viewed posts get shared

Likes per day: 10B * 0.5 = 5B likes = 57,870 likes/sec (peak: 173,600/sec)
Comments per day: 10B * 0.1 = 1B comments = 11,574/sec (peak: 34,700/sec)
Shares per day: 10B * 0.05 = 500M shares = 5,787/sec (peak: 17,360/sec)
```

**Follow/Unfollow Actions:**

```text
Assuming 5% of DAU follow/unfollow per day:
Follow actions per day: 500M * 0.05 = 25M
Follow actions per second: 25M / 86,400 = ~290/sec (peak: 870/sec)
```

### Storage Estimates

**User Data:**

```text
Total users: 1.5B (MAU)
User record size: ~1KB (profile info, metadata)
User data storage: 1.5B * 1KB = 1.5TB
```

**Post Metadata:**

```text
Posts per day: 200M
Days of data to keep: Indefinite (assume 5 years = 1,825 days)
Total posts in 5 years: 200M * 1,825 = 365B posts

Post metadata size: ~2KB (caption, hashtags, metadata, engagement counts)
Post metadata storage (5 years): 365B * 2KB = 730TB = ~730TB
```

**Media Storage:**

**Photos:**

```text
Photos per day: 120M
Photo size (original): 2MB
Photo size (thumbnails): 100KB + 300KB + 500KB = 900KB
Total per photo: 2MB + 900KB = 2.9MB

Daily storage: 120M * 2.9MB = 348TB/day
Yearly storage: 348TB * 365 = 127PB/year
5-year storage: 127PB * 5 = 635PB
```

**Videos:**

```text
Videos per day: 60M
Video size (original): 50MB
Video size (processed formats): 10MB + 20MB + 30MB = 60MB
Thumbnail: 500KB
Total per video: 50MB + 60MB + 0.5MB = 110.5MB

Daily storage: 60M * 110.5MB = 6,630TB = 6.6PB/day
Yearly storage: 6.6PB * 365 = 2,409PB/year
5-year storage: 2,409PB * 5 = 12,045PB = ~12EB
```

**Stories:**

```text
Stories per day: 20M
Story size (avg): 10MB with processing = 25MB total
Stories are deleted after 24 hours

Active storage: 20M * 25MB = 500TB (rolling 24-hour window)
```

**Total Storage (5 years):**

```text
User data: 1.5TB
Post metadata: 730TB
Photos: 635PB
Videos: 12EB
Stories: 500TB (active)

Total: ~12.6EB (primarily video content)
```

### Bandwidth Estimates

**Upload Bandwidth:**

```text
Media uploads per second (avg): 2,315 posts/sec
Average media size: (120M * 2MB + 60M * 50MB + 20M * 10MB) / 200M
                  = (240TB + 3,000TB + 200TB) / 200M
                  = 3,440TB / 200M = ~17.2MB per post

Upload bandwidth (avg): 2,315 * 17.2MB = ~40GB/sec = 320Gbps
Upload bandwidth (peak): 320Gbps * 3 = 960Gbps = ~1Tbps
```

**Download Bandwidth (Feed Serving):**

```text
Feed requests per second (avg): 115,740/sec
Average posts per feed load: 20 posts
Media per post (thumbnail): 300KB

Download per request: 20 * 300KB = 6MB
Download bandwidth (avg): 115,740 * 6MB = 694TB/sec = ~5.5Pbps
Download bandwidth (peak): 5.5Pbps * 3 = 16.5Pbps

Note: CDN will cache majority of this traffic (~95% cache hit rate)
Actual origin bandwidth: 16.5Pbps * 0.05 = ~825Tbps at peak
```

### Resource Estimates

**Database Sizing:**

```text
Primary databases:
- User DB: 1.5TB + indexes (~2x) = 3TB
- Post metadata DB: 730TB + indexes = 1.5PB (sharded)
- Social graph DB: 500M users * 200 follows * 16 bytes = 1.6TB
```

**Cache Sizing:**

```text
Feed cache (hot data):
- Active users at peak: 500M * 0.2 (concurrent) = 100M
- Cache per user: 50 posts * 2KB metadata = 100KB
- Total cache: 100M * 100KB = 10TB

Hot content cache:
- Top 10% of posts account for 90% of views
- Recent posts (last 24h): 200M posts
- Top posts to cache: 20M posts
- Cache size: 20M * 300KB (thumbnail) = 6TB
```

**Worker Pool Sizing:**

```text
Media processing workers:
- Peak uploads: 7,000/sec
- Processing time per media: ~10 seconds
- Concurrent workers needed: 7,000 * 10 = 70,000 workers
- With buffering and queuing: ~100,000 workers
```

### 🤔 Think About It

1. **For Beginners:** Why do we need to calculate peak traffic separately from average traffic? (Hint: Think about what happens when everyone uses the app at the same time, like during lunch break)

2. **For Intermediate:** If storage costs $0.023 per GB per month, and we're storing 12.6 exabytes, what's the monthly storage cost? How would you reduce this by 50%?

3. **For Advanced:** How would your capacity planning change if Instagram added a new feature that doubles video uploads but videos are 10x larger? Consider both immediate impact and 5-year projection.

### ✅ Key Takeaways

- **Capacity planning drives infrastructure decisions**: Getting calculations wrong means millions in wasted costs or catastrophic failures
- **Peak traffic is 3x average**: Always account for peak load (evening hours, viral content)
- **Media dominates storage**: Videos account for 95% of storage (12EB vs 635PB for photos)
- **CDN is critical**: 95% cache hit rate saves $570M/year in bandwidth costs
- **Storage tiering saves money**: Hot/warm/cold storage reduces costs by 67% vs all-hot
- **Geographic distribution multiplies costs**: Multi-region deployment requires 3x infrastructure

### 🎯 Practice Exercise

**Scenario:** You're designing a new social media platform for a specific niche (e.g., photographers, chefs, musicians) with different characteristics:
- 10M DAU (vs Instagram's 500M)
- 5M posts/day (vs Instagram's 200M)
- Average photo size: 10MB (vs Instagram's 2MB compressed)
- Videos are 4K quality: 200MB average (vs Instagram's 50MB)

**Your Task:**
1. Calculate traffic estimates (posts/sec, feed requests/sec)
2. Estimate 5-year storage requirements
3. Calculate bandwidth needs (upload and download)
4. Estimate infrastructure costs (servers, storage, bandwidth)

**Bonus Challenge:** How would your calculations change if 80% of users are in one geographic region vs evenly distributed globally?

---

## Section 3: Designing the System Architecture

### What You'll Learn

By the end of this section, you'll be able to:

- Design a high-level system architecture for a social media platform
- Identify and explain the purpose of each major component
- Understand data flow for write operations (post creation) and read operations (feed generation)
- Choose appropriate load balancing strategies for different use cases
- Justify architectural decisions in system design interviews

### Why This Matters

Architecture is the foundation of your system. Real-world example: Instagram's architecture evolved from a simple monolith to a complex microservices system handling 1.4B users—the initial architecture decisions determined whether they could scale or needed complete rewrites! In interviews, showing you can design a coherent architecture demonstrates you understand how components work together.

---

### 🟢 For Beginners: The Fundamentals

#### What is System Architecture?

Think of system architecture like a restaurant layout:

- **Front door** = Load Balancer (directs customers to available tables)
- **Dining area** = API Servers (where customers interact)
- **Kitchen** = Application Services (where work happens)
- **Pantry** = Databases (where ingredients/data are stored)
- **Delivery trucks** = CDN (brings food/content to customers)

**Key Components Explained:**

1. **Load Balancer**: Distributes incoming requests across multiple servers
   - Like a host at a restaurant directing customers to available tables
   - Prevents one server from being overwhelmed

2. **API Servers**: Handle user requests (post creation, feed requests)
   - Like waiters taking orders and serving food
   - Process requests and return responses

3. **Application Services**: Core business logic (feed generation, media processing)
   - Like chefs preparing food
   - Do the actual work (generate feeds, process images)

4. **Databases**: Store data (users, posts, relationships)
   - Like pantries storing ingredients
   - Persistent storage for all information

5. **CDN**: Caches and delivers media content globally
   - Like food delivery trucks bringing meals to customers
   - Makes content load faster by storing it closer to users

#### Basic Request Flow

**When a user creates a post:**

```text
1. User uploads photo → Load Balancer
2. Load Balancer → API Server (available one)
3. API Server → Media Processing Service
4. Media Processing → Object Storage (S3)
5. API Server → Database (save post metadata)
6. API Server → Feed Service (update followers' feeds)
7. Response → User ("Post created!")
```

**When a user views their feed:**

```text
1. User requests feed → Load Balancer
2. Load Balancer → API Server
3. API Server → Feed Service
4. Feed Service → Cache (check if feed exists)
5. If cache miss → Database (fetch posts)
6. Feed Service → Rank posts (ML algorithm)
7. Response → User (personalized feed)
```

---

### 🟡 For Intermediate: Interview Patterns

#### The Architecture Design Framework

**Step 1: Identify Core Components**

```text
For social media platform:
├─ Client Layer (mobile apps, web)
├─ Load Balancer / API Gateway
├─ API Servers (stateless)
├─ Application Services:
│  ├─ Feed Service
│  ├─ Media Processing Service
│  ├─ Notification Service
│  └─ Search Service
├─ Data Layer:
│  ├─ User Database (PostgreSQL)
│  ├─ Post Database (Cassandra)
│  ├─ Graph Database (Neo4j)
│  └─ Cache (Redis)
└─ CDN (for media delivery)
```

**Step 2: Explain Data Flow**

**Write Path (Post Creation):**
```text
Client → LB → API Server → Media Processing Queue
                                    ↓
                            Object Storage (S3)
                                    ↓
                            Post Metadata DB (Cassandra)
                                    ↓
                            Feed Service (fan-out)
                                    ↓
                            Feed Cache (Redis)
```

**Read Path (Feed Request):**
```text
Client → LB → API Server → Feed Service
                              ↓
                        Check Cache (Redis)
                              ↓
                    Cache Hit? → Return feed
                    Cache Miss? → Query DB → Rank → Cache → Return
```

**Step 3: Justify Component Choices**

**Why Microservices?**
- Independent scaling (feed service scales differently than media processing)
- Technology diversity (use best tool for each job)
- Fault isolation (one service failure doesn't bring down everything)

**Why Multiple Databases?**
- PostgreSQL: Strong consistency for user data (critical)
- Cassandra: High write throughput for posts (200M/day)
- Neo4j: Graph queries for social relationships (who follows whom)
- Redis: Low-latency caching for feeds (<100ms requirement)

---

### 🔴 For Advanced: Production Considerations

#### Production Architecture Patterns

**1. Multi-Region Deployment**

```text
Region 1 (US-East):
├─ Full stack deployment
├─ Primary database (writes)
└─ Replicas in other regions

Region 2 (EU-West):
├─ Full stack deployment
├─ Read replicas (from US-East)
└─ Local cache (warm data)

Region 3 (Asia-Pacific):
├─ Full stack deployment
├─ Read replicas (from US-East)
└─ Local cache (warm data)

Traffic Routing:
├─ GeoDNS: Route users to nearest region
├─ Write: Always go to primary (US-East)
├─ Read: Can go to local replica
└─ Consistency: Eventual (acceptable for feeds)
```

**2. Service Mesh Architecture**

```text
Benefits:
├─ Service Discovery: Automatic service location
├─ Load Balancing: Built-in LB per service
├─ Circuit Breaking: Prevent cascading failures
├─ Observability: Distributed tracing
└─ Security: mTLS between services

Implementation:
├─ Istio or Linkerd
├─ Sidecar proxy per service
└─ Centralized control plane
```

**3. Event-Driven Architecture**

```text
Post Creation Event:
├─ Publisher: API Server
├─ Event Bus: Kafka
├─ Subscribers:
│  ├─ Feed Service (update feeds)
│  ├─ Notification Service (notify followers)
│  ├─ Analytics Service (track metrics)
│  └─ Search Service (index post)
└─ Benefits: Decoupling, scalability, resilience
```

---

### System Architecture Diagram

```mermaid
graph TB
    subgraph Client Layer
        Mobile[Mobile Apps<br/>iOS/Android]
        Web[Web Client<br/>React]
    end
    
    subgraph CDN Layer
        CDN[CDN<br/>CloudFront/Akamai]
    end
    
    subgraph Load Balancing Layer
        LB[Load Balancer<br/>AWS ALB/Nginx]
    end
    
    subgraph API Gateway Layer
        API[API Gateway<br/>Kong/AWS API Gateway]
        Auth[Auth Service<br/>OAuth 2.0]
    end
    
    subgraph Application Services
        UserSvc[User Service<br/>Node.js/Go]
        PostSvc[Post Service<br/>Go]
        FeedSvc[Feed Service<br/>Go]
        EngageSvc[Engagement Service<br/>Go]
        MediaSvc[Media Upload Service<br/>Go]
    end
    
    subgraph Feed Generation
        FanoutSvc[Fanout Service<br/>Go]
        RankSvc[Ranking Service<br/>Python/ML]
    end
    
    subgraph Message Queue Layer
        MQ[Message Queue<br/>Kafka]
        PubSub[Pub/Sub<br/>Redis Streams]
    end
    
    subgraph Processing Layer
        MediaProc[Media Processor<br/>FFmpeg Workers]
        NotifProc[Notification Processor<br/>Go]
    end
    
    subgraph Data Layer - Databases
        UserDB[(User DB<br/>PostgreSQL)]
        PostDB[(Post DB<br/>Cassandra)]
        GraphDB[(Graph DB<br/>Neo4j/DGraph)]
        AnalyticsDB[(Analytics DB<br/>ClickHouse)]
    end
    
    subgraph Data Layer - Cache
        RedisCluster[Redis Cluster<br/>Feed Cache]
        RedisTimeline[Redis Sorted Sets<br/>Timeline Cache]
        Memcached[Memcached<br/>Object Cache]
    end
    
    subgraph Storage Layer
        S3[Object Storage<br/>S3/GCS]
        Search[Search Index<br/>Elasticsearch]
    end
    
    Mobile -->|1. Request| CDN
    Web -->|1. Request| CDN
    CDN -->|2. Cache Miss| LB
    CDN -.->|2a. Cache Hit| Mobile
    LB -->|3. Route| API
    API -->|4. Authenticate| Auth
    
    API -->|5a. User Request| UserSvc
    API -->|5b. Post Request| PostSvc
    API -->|5c. Feed Request| FeedSvc
    API -->|5d. Like/Comment| EngageSvc
    API -->|5e. Upload| MediaSvc
    
    UserSvc --> UserDB
    UserSvc --> Memcached
    
    PostSvc --> PostDB
    PostSvc -->|Publish Event| MQ
    PostSvc --> Memcached
    
    FeedSvc --> RedisTimeline
    FeedSvc --> RankSvc
    FeedSvc --> RedisCluster
    
    EngageSvc --> PostDB
    EngageSvc -->|Real-time Update| PubSub
    EngageSvc --> RedisCluster
    
    MediaSvc -->|Upload| S3
    MediaSvc -->|Queue Job| MQ
    
    MQ -->|New Post Event| FanoutSvc
    MQ -->|Process Media| MediaProc
    MQ -->|Send Notification| NotifProc
    
    FanoutSvc --> GraphDB
    FanoutSvc --> RedisTimeline
    FanoutSvc --> PostDB
    
    MediaProc --> S3
    MediaProc --> PostDB
    
    RankSvc --> AnalyticsDB
    RankSvc --> PostDB
    
    PostSvc --> Search
    UserSvc --> Search
```

### Data Flow Explanation

#### Post Creation Flow

1. **User uploads media** via mobile/web client
2. **CDN/Load Balancer** routes request to API Gateway
3. **API Gateway** authenticates user via Auth Service
4. **Media Upload Service** receives upload request
5. **Media Service** uploads original file to **Object Storage (S3)**
6. **Media Service** publishes job to **Message Queue (Kafka)**
7. **Media Processor workers** consume job and process media:
   - Generate thumbnails
   - Transcode videos to multiple formats
   - Extract metadata
   - Upload processed files back to S3
8. **Post Service** creates post metadata in **Post Database (Cassandra)**
9. **Post Service** publishes "new post" event to **Kafka**
10. **Fanout Service** consumes event and performs fan-out:
    - For regular users (<10k followers): Fan-out on write
    - For celebrities (>10k followers): Fan-out on read (lazy loading)
11. **Fanout Service** writes to followers' **Timeline Cache (Redis)**
12. **CDN** distributes media globally
13. User receives success response

#### Feed Generation Flow

1. **User requests feed** via mobile/web client
2. **CDN** serves if cached (for public/explore feeds)
3. **Load Balancer** routes to API Gateway
4. **Feed Service** receives request
5. **Feed Service** checks **Timeline Cache (Redis Sorted Sets)**:
   - If cache hit: Return cached timeline
   - If cache miss: Generate timeline on-demand
6. For timeline generation:
   - **Feed Service** queries **Graph DB** for followed users
   - For regular users: Read from pre-computed timeline
   - For celebrity followers: Merge celebrity content on-read
7. **Ranking Service** applies ML ranking algorithm:
   - Fetch user preferences from **Analytics DB**
   - Score posts based on engagement likelihood
   - Rerank timeline
8. **Feed Service** fetches post metadata from **Post DB**
9. **Feed Service** returns feed with CDN URLs for media
10. User's client downloads media from **CDN**

#### Engagement Flow (Like/Comment)

1. **User performs action** (like, comment, share)
2. **API Gateway** routes to **Engagement Service**
3. **Engagement Service** updates **Post Database** (increment counter)
4. **Engagement Service** updates **Cache** for real-time counts
5. **Engagement Service** publishes to **Pub/Sub (Redis Streams)**
6. **Real-time service** pushes update to connected clients via WebSocket
7. **Analytics DB** receives event for ranking algorithm updates

### Real-World Example: How Instagram Evolved Their Architecture

**2010-2012 - Monolithic Phase:**
```text
Architecture: Single Django application
├─ Scale: 30M users
├─ Database: PostgreSQL (single instance)
├─ Storage: Direct file storage
└─ Challenge: Database became bottleneck

Decision: Split into services
Result: Could scale individual components
```

**2012-2016 - Microservices Transition:**
```text
Architecture: Service-oriented
├─ Scale: 400M users
├─ Databases: PostgreSQL + Cassandra + Redis
├─ Storage: S3 with CDN
└─ Challenge: Service coordination complexity

Decision: Add message queues (Kafka)
Result: Decoupled services, better scalability
```

**2016-2020 - Global Scale:**
```text
Architecture: Multi-region microservices
├─ Scale: 1B+ users
├─ Regions: US, EU, Asia
├─ Databases: Sharded across regions
└─ Challenge: Consistency across regions

Decision: Eventual consistency for feeds
Result: Global low latency, acceptable consistency
```

**2020-Present - ML-Powered:**
```text
Architecture: ML-integrated microservices
├─ Scale: 1.4B users
├─ ML: Real-time ranking, content moderation
├─ Infrastructure: Kubernetes, service mesh
└─ Challenge: ML model serving at scale

Decision: Edge ML inference
Result: Personalized feeds, <100ms latency
```

📊 **By The Numbers:**
- 2010: 1M users, 1 data center
- 2015: 400M users, 3 data centers
- 2020: 1B users, 10+ data centers globally
- 2024: 1.4B users, edge computing + cloud

**Key Lesson:** Architecture evolution is incremental. Start simple, add complexity only when needed. Instagram's success came from evolving their architecture as they scaled, not from perfect initial design.

### 🎯 Interview Questions: System Architecture

#### Question 1: How would you design the architecture for a social media platform?

**What the interviewer wants to know:**
- Can you identify core components?
- Do you understand microservices vs monolith trade-offs?
- Can you explain data flow?

**Answer Framework:**

```text
1. Identify Core Components
   ├─ Client Layer: Mobile apps, web
   ├─ API Gateway: Authentication, routing
   ├─ Application Services: Feed, Media, Notification
   ├─ Data Layer: Multiple databases (right tool for job)
   └─ CDN: Media delivery

2. Explain Write Path
   ├─ User uploads → API Gateway
   ├─ Media Processing Queue
   ├─ Object Storage
   ├─ Database (metadata)
   └─ Fan-out Service (update feeds)

3. Explain Read Path
   ├─ User requests feed → API Gateway
   ├─ Check Cache
   ├─ If miss: Query DB → Rank → Cache
   └─ Return personalized feed

4. Justify Choices
   ├─ Microservices: Independent scaling
   ├─ Multiple DBs: Right tool for each job
   └─ CDN: Global low latency
```

**Follow-up: How do you handle a celebrity posting to 100M followers?**

```text
Solution: Hybrid fan-out
├─ Regular users (<1M followers): Fan-out on write
├─ Celebrities (>1M followers): Fan-out on read
└─ Result: Avoid 100M write operations
```

#### Question 2: How would you scale this architecture from 1M to 500M users?

**Answer Framework:**

```text
Phase 1 (1M users):
├─ Monolith: Single application
├─ Single database: PostgreSQL
└─ Simple caching: Redis

Phase 2 (10M users):
├─ Split services: Feed, Media, User
├─ Add read replicas
└─ Add CDN

Phase 3 (100M users):
├─ Microservices: Full separation
├─ Database sharding
├─ Multi-region deployment
└─ Advanced caching

Phase 4 (500M users):
├─ Service mesh: Istio/Linkerd
├─ Global distribution: 10+ regions
├─ ML-powered ranking
└─ Edge computing
```

### 🤔 Think About It

1. **For Beginners:** Why do we need a load balancer? What happens if we don't have one? (Hint: Think about what happens when everyone tries to use the same door at once)

2. **For Intermediate:** If you had to choose between a monolithic architecture and microservices for a startup with 10 engineers, which would you choose? Why?

3. **For Advanced:** How would your architecture change if you needed to support real-time collaborative editing (like Google Docs) for posts? Consider consistency requirements, conflict resolution, and WebSocket scaling.

### ✅ Key Takeaways

- **Architecture evolves with scale**: Start simple (monolith), add complexity (microservices) only when needed
- **Right tool for the job**: Use multiple databases (PostgreSQL for users, Cassandra for posts, Neo4j for graphs)
- **Stateless services enable scaling**: API servers should be stateless to allow horizontal scaling
- **Caching is critical**: 90%+ cache hit rate reduces database load by 10x
- **Event-driven architecture decouples services**: Message queues (Kafka) enable independent scaling
- **CDN reduces bandwidth costs**: 95% cache hit rate saves $570M/year

### 🎯 Practice Exercise

**Scenario:** You're designing a social media platform specifically for video creators (like YouTube Shorts or TikTok) with these characteristics:
- 50M DAU
- 10M videos uploaded per day (average 60 seconds, 1080p)
- 5B video views per day
- Real-time comments and reactions during playback

**Your Task:**
1. Design the high-level architecture (draw components and data flow)
2. Explain how video upload flow differs from photo upload
3. Design the real-time comment system architecture
4. Justify your technology choices (databases, message queues, CDN)

**Bonus Challenge:** How would you handle viral videos that suddenly get 10M views in 1 hour?

---

## Section 4: Database Design & Data Modeling

### What You'll Learn

By the end of this section, you'll be able to:

- Choose appropriate databases for different data types (users, posts, relationships)
- Design database schemas optimized for read/write patterns
- Understand when to use SQL vs NoSQL databases
- Design sharding strategies for horizontal scaling
- Handle data consistency patterns (strong vs eventual consistency)

### Why This Matters

Database design determines whether your system can scale. Real-world example: Instagram started with PostgreSQL, but as they grew to 200M posts/day, PostgreSQL couldn't handle the write load—they migrated posts to Cassandra, which handles 10x more writes! Choosing the wrong database means rewriting your entire data layer later.

---

### 🟢 For Beginners: The Fundamentals

#### What is a Database?

Think of databases like different types of storage:

- **SQL Database (PostgreSQL)**: Like a filing cabinet with organized folders
  - Structured data (users, profiles)
  - Strong consistency (data is always correct)
  - Good for: User accounts, financial data

- **NoSQL Database (Cassandra)**: Like a warehouse with many shelves
  - Unstructured data (posts, feeds)
  - High write throughput (millions of writes per second)
  - Good for: Posts, feeds, logs

- **Graph Database (Neo4j)**: Like a social network map
  - Relationships (who follows whom)
  - Fast relationship queries
  - Good for: Friend suggestions, social graphs

- **Cache (Redis)**: Like a fast-access drawer
  - Temporary data (feeds, counters)
  - Very fast (microseconds)
  - Good for: Frequently accessed data

#### Why Multiple Databases?

**Different data needs different storage:**

```text
User Data:
├─ Needs: Strong consistency (can't have duplicate accounts)
├─ Pattern: Read-heavy, occasional writes
└─ Database: PostgreSQL (SQL)

Posts:
├─ Needs: High write throughput (200M posts/day)
├─ Pattern: Write-heavy, many reads
└─ Database: Cassandra (NoSQL)

Social Graph:
├─ Needs: Relationship queries (who follows whom)
├─ Pattern: Complex graph queries
└─ Database: Neo4j (Graph)

Feeds:
├─ Needs: Very fast access (<100ms)
├─ Pattern: Frequent reads, temporary data
└─ Database: Redis (Cache)
```

---

### 🟡 For Intermediate: Interview Patterns

#### Database Selection Framework

**When choosing a database, consider:**

```text
1. Data Model
   ├─ Structured? → SQL (PostgreSQL)
   ├─ Unstructured? → NoSQL (Cassandra, MongoDB)
   └─ Relationships? → Graph (Neo4j)

2. Access Patterns
   ├─ Read-heavy? → SQL with read replicas
   ├─ Write-heavy? → NoSQL (Cassandra)
   └─ Both? → Hybrid (SQL + NoSQL)

3. Consistency Requirements
   ├─ Strong consistency needed? → SQL
   ├─ Eventual consistency OK? → NoSQL
   └─ Depends on use case

4. Scale Requirements
   ├─ <10M records? → Single SQL database
   ├─ 10M-1B records? → SQL with sharding
   └─ >1B records? → NoSQL (built for scale)
```

#### Interview Script: Database Design

**When asked "What database would you use for posts?":**

**You say:** "I'd use Cassandra for posts because:
1. **Write throughput**: 200M posts/day = 2,315 writes/sec (peak: 7,000/sec)
2. **Partitioning**: Posts can be partitioned by user_id or timestamp
3. **Eventual consistency**: Acceptable for posts (1-2 second delay is fine)
4. **Horizontal scaling**: Add nodes to increase capacity

For user data, I'd use PostgreSQL because:
1. **Strong consistency**: Critical for user accounts
2. **ACID transactions**: Need for financial operations
3. **Read-heavy**: More reads than writes
4. **Complex queries**: Need JOINs for user relationships"

---

### 🔴 For Advanced: Production Considerations

#### Multi-Database Architecture Patterns

**1. CQRS (Command Query Responsibility Segregation)**

```text
Write Model (Cassandra):
├─ Optimized for writes
├─ Denormalized data
└─ Eventual consistency

Read Model (Elasticsearch):
├─ Optimized for reads
├─ Pre-computed views
└─ Strong consistency (eventually)

Sync: Kafka CDC (Change Data Capture)
├─ Cassandra → Kafka → Elasticsearch
└─ Latency: <1 second
```

**2. Database Sharding Strategies**

```text
Sharding by User ID:
├─ Shard = hash(user_id) % num_shards
├─ Pros: Even distribution
└─ Cons: Hard to query across shards

Sharding by Geographic Region:
├─ US users → Shard 1
├─ EU users → Shard 2
└─ Asia users → Shard 3
├─ Pros: Low latency (data close to users)
└─ Cons: Uneven distribution

Sharding by Time:
├─ Recent posts → Hot shard (SSD)
├─ Old posts → Cold shard (HDD)
└─ Pros: Cost optimization
└─ Cons: Complex queries across time ranges
```

**3. Consistency Patterns**

```text
Strong Consistency (PostgreSQL):
├─ Use for: User accounts, financial data
├─ Trade-off: Higher latency, lower throughput
└─ Guarantee: All reads see latest write

Eventual Consistency (Cassandra):
├─ Use for: Posts, feeds, counters
├─ Trade-off: Lower latency, higher throughput
└─ Guarantee: Eventually all reads see write

Read-After-Write Consistency:
├─ Problem: User posts, then immediately views feed
├─ Solution: Read from primary (not replica) for 30 seconds
└─ Implementation: Route reads to primary based on timestamp
```

---

### Database Selection Strategy

**Multi-Layer Health Checks:**

```python
class HealthCheckService:
    def __init__(self):
        self.checks = {
            'shallow': self.shallow_health_check,
            'deep': self.deep_health_check,
            'dependency': self.dependency_health_check
        }
    
    def shallow_health_check(self):
        """Quick check - is the service responding?"""
        return {
            'status': 'healthy',
            'timestamp': time.time(),
            'version': '1.2.3'
        }
    
    def deep_health_check(self):
        """Comprehensive check - are all components working?"""
        checks = {}
        overall_healthy = True
        
        # Check database connectivity
        try:
            db.execute('SELECT 1')
            checks['database'] = 'healthy'
        except Exception as e:
            checks['database'] = f'unhealthy: {e}'
            overall_healthy = False
        
        # Check cache connectivity
        try:
            redis.ping()
            checks['cache'] = 'healthy'
        except Exception as e:
            checks['cache'] = f'unhealthy: {e}'
            overall_healthy = False
        
        # Check message queue
        try:
            kafka.list_topics(timeout=1)
            checks['message_queue'] = 'healthy'
        except Exception as e:
            checks['message_queue'] = f'unhealthy: {e}'
            overall_healthy = False
        
        # Check disk space
        disk_usage = psutil.disk_usage('/')
        if disk_usage.percent > 90:
            checks['disk_space'] = f'unhealthy: {disk_usage.percent}% used'
            overall_healthy = False
        else:
            checks['disk_space'] = 'healthy'
        
        # Check memory
        memory = psutil.virtual_memory()
        if memory.percent > 90:
            checks['memory'] = f'unhealthy: {memory.percent}% used'
            overall_healthy = False
        else:
            checks['memory'] = 'healthy'
        
        return {
            'status': 'healthy' if overall_healthy else 'unhealthy',
            'checks': checks,
            'timestamp': time.time()
        }
    
    def dependency_health_check(self):
        """Check external dependencies"""
        checks = {}
        
        # Check S3 connectivity
        try:
            s3.head_bucket(Bucket='socialmedia-media')
            checks['s3'] = 'healthy'
        except:
            checks['s3'] = 'unhealthy'
        
        # Check CDN
        try:
            response = requests.head('https://cdn.socialmedia.com/health', timeout=2)
            checks['cdn'] = 'healthy' if response.status_code == 200 else 'unhealthy'
        except:
            checks['cdn'] = 'unhealthy'
        
        return {
            'status': 'healthy' if all(v == 'healthy' for v in checks.values()) else 'degraded',
            'checks': checks,
            'timestamp': time.time()
        }
```

### Session Affinity (Sticky Sessions)

**When to Use:**

```text
Use sticky sessions for:
- WebSocket connections
- Stateful operations
- Temporary session data

DON'T use for:
- REST APIs (should be stateless)
- High-availability requirements
- Geographic distribution
```

**Implementation:**

```nginx
# Nginx sticky sessions using IP hash
upstream websocket_servers {
    ip_hash;  # Same client always goes to same server
    
    server ws-1.internal:8080;
    server ws-2.internal:8080;
    server ws-3.internal:8080;
}

# Alternative: Cookie-based sticky sessions
upstream api_servers {
    server api-1.internal:8080;
    server api-2.internal:8080;
    
    sticky cookie srv_id expires=1h domain=.socialmedia.com path=/;
}
```

---

## Database Design

### Database Selection Strategy

**Multi-Database Approach:**

- **PostgreSQL**: User data, authentication (ACID compliance needed)
- **Cassandra**: Post metadata, high write throughput, horizontal scalability
- **Neo4j/DGraph**: Social graph, follower/following relationships
- **Redis**: Caching, timeline storage, real-time counters
- **Elasticsearch**: Full-text search for users, hashtags, posts
- **ClickHouse**: Analytics data, aggregations, reporting

### Schema Definitions

#### Users Table (PostgreSQL)

```text
Table: users
- user_id (PK, UUID)
- username (VARCHAR(30), UNIQUE, INDEXED)
- email (VARCHAR(255), UNIQUE, INDEXED)
- password_hash (VARCHAR(255))
- full_name (VARCHAR(100))
- bio (TEXT, max 500 chars)
- profile_picture_url (VARCHAR(500))
- is_verified (BOOLEAN, DEFAULT false)
- is_private (BOOLEAN, DEFAULT false)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)
- last_login_at (TIMESTAMP)
- follower_count (INTEGER, DEFAULT 0, INDEXED)
- following_count (INTEGER, DEFAULT 0)
- post_count (INTEGER, DEFAULT 0)

Indexes:
- PRIMARY KEY (user_id)
- UNIQUE INDEX idx_username (username)
- UNIQUE INDEX idx_email (email)
- INDEX idx_follower_count (follower_count) -- for celebrity detection
```

#### Posts Table (Cassandra)

```text
Table: posts
Partition Key: user_id
Clustering Key: created_at DESC, post_id

Columns:
- post_id (UUID)
- user_id (UUID)
- post_type (VARCHAR) -- 'photo', 'video', 'carousel', 'story'
- caption (TEXT, max 2200 chars)
- media_urls (LIST<VARCHAR>) -- S3 URLs for original media
- thumbnail_urls (MAP<VARCHAR, VARCHAR>) -- size -> URL
- processed_urls (MAP<VARCHAR, VARCHAR>) -- format -> URL
- hashtags (SET<VARCHAR>)
- mentions (SET<UUID>)
- location (VARCHAR)
- like_count (COUNTER)
- comment_count (COUNTER)
- share_count (COUNTER)
- view_count (COUNTER)
- is_deleted (BOOLEAN)
- created_at (TIMESTAMP)
- expires_at (TIMESTAMP) -- for stories
- processing_status (VARCHAR) -- 'pending', 'processing', 'completed', 'failed'

Indexes:
- PRIMARY KEY ((user_id), created_at, post_id)
- INDEX idx_hashtags (hashtags) -- for hashtag queries
- INDEX idx_created_at (created_at) -- for trending queries
```

#### Post Metadata Table (Cassandra - for global post lookups)

```text
Table: posts_by_id
Partition Key: post_id

Columns:
- post_id (UUID, PK)
- user_id (UUID)
- [... same columns as posts table ...]

Indexes:
- PRIMARY KEY (post_id)
```

#### Likes Table (Cassandra)

```text
Table: likes
Partition Key: post_id
Clustering Key: created_at DESC, user_id

Columns:
- post_id (UUID)
- user_id (UUID)
- created_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((post_id), created_at, user_id)

Table: likes_by_user
Partition Key: user_id
Clustering Key: created_at DESC, post_id

Columns:
- user_id (UUID)
- post_id (UUID)
- created_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((user_id), created_at, post_id)
```

#### Comments Table (Cassandra)

```text
Table: comments
Partition Key: post_id
Clustering Key: created_at DESC, comment_id

Columns:
- comment_id (UUID)
- post_id (UUID)
- user_id (UUID)
- parent_comment_id (UUID, NULL for root comments)
- comment_text (TEXT, max 500 chars)
- like_count (COUNTER)
- reply_count (COUNTER)
- is_deleted (BOOLEAN)
- created_at (TIMESTAMP)
- updated_at (TIMESTAMP)

Indexes:
- PRIMARY KEY ((post_id), created_at, comment_id)
- INDEX idx_user_id (user_id)
```

#### Follows Table (Graph DB - Neo4j)

```text
Node: User
Properties:
- user_id (UUID, INDEXED)
- username (STRING)
- follower_count (INTEGER)
- is_celebrity (BOOLEAN) -- followers > 10k

Relationship: FOLLOWS
Properties:
- followed_at (TIMESTAMP)
- notification_enabled (BOOLEAN)

Cypher Queries:
// Get followers
MATCH (follower:User)-[:FOLLOWS]->(user:User {user_id: $userId})
RETURN follower

// Get following
MATCH (user:User {user_id: $userId})-[:FOLLOWS]->(following:User)
RETURN following

// Check if A follows B
MATCH (a:User {user_id: $userA})-[r:FOLLOWS]->(b:User {user_id: $userB})
RETURN r IS NOT NULL
```

#### Timeline Cache (Redis Sorted Sets)

```text
Key Pattern: timeline:{user_id}
Type: Sorted Set
Score: timestamp (for chronological ordering)
Member: post_id

Commands:
ZADD timeline:{user_id} {timestamp} {post_id}
ZREVRANGE timeline:{user_id} 0 49 -- Get top 50 posts
ZCARD timeline:{user_id} -- Get count
EXPIRE timeline:{user_id} 86400 -- 24-hour TTL
```

#### Feed Cache (Redis Hash)

```text
Key Pattern: feed:ranked:{user_id}
Type: Hash
Fields: post_id -> score

Commands:
HSET feed:ranked:{user_id} {post_id} {relevance_score}
HGETALL feed:ranked:{user_id}
EXPIRE feed:ranked:{user_id} 3600 -- 1-hour TTL
```

#### Engagement Counters (Redis Strings)

```text
Key Patterns:
- post:likes:{post_id} -> COUNT
- post:comments:{post_id} -> COUNT
- post:shares:{post_id} -> COUNT
- post:views:{post_id} -> COUNT

Commands:
INCR post:likes:{post_id}
DECR post:likes:{post_id} -- for unlike
GET post:likes:{post_id}
EXPIRE post:*:{post_id} 604800 -- 7-day TTL, sync to DB
```

#### Hashtag Index (Elasticsearch)

```json
{
  "index": "hashtags",
  "mappings": {
    "properties": {
      "hashtag": {
        "type": "keyword"
      },
      "post_count": {
        "type": "long"
      },
      "trending_score": {
        "type": "float"
      },
      "recent_posts": {
        "type": "nested",
        "properties": {
          "post_id": { "type": "keyword" },
          "timestamp": { "type": "date" }
        }
      },
      "created_at": {
        "type": "date"
      }
    }
  }
}
```

#### User Search Index (Elasticsearch)

```json
{
  "index": "users_search",
  "mappings": {
    "properties": {
      "user_id": {
        "type": "keyword"
      },
      "username": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": { "type": "keyword" }
        }
      },
      "full_name": {
        "type": "text",
        "analyzer": "standard"
      },
      "bio": {
        "type": "text"
      },
      "follower_count": {
        "type": "long"
      },
      "is_verified": {
        "type": "boolean"
      }
    }
  }
}
```

#### Analytics Events (ClickHouse)

```text
Table: user_events

Columns:
- event_id (UUID)
- user_id (UUID)
- event_type (String) -- 'post_view', 'like', 'comment', 'share', 'follow'
- post_id (Nullable(UUID))
- target_user_id (Nullable(UUID))
- session_id (UUID)
- timestamp (DateTime)
- device_type (String)
- platform (String)

Engine: MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, timestamp)
```

### Real-World Example: How Instagram Migrated from PostgreSQL to Cassandra

**2010-2012 - PostgreSQL Only:**
```text
Architecture: Single PostgreSQL database
├─ Scale: 30M users, 10M posts/day
├─ Challenge: Write bottleneck (2,000 writes/sec max)
├─ Problem: Posts table became slow, affecting feed generation
└─ Decision: Need to scale writes

Result: Started exploring NoSQL options
```

**2012-2014 - Hybrid Approach:**
```text
Architecture: PostgreSQL + Cassandra
├─ PostgreSQL: User data (strong consistency needed)
├─ Cassandra: Post metadata (high write throughput)
├─ Migration: Gradual migration of posts table
└─ Challenge: Dual-write complexity during migration

Decision: Use Kafka for dual-write
Result: Zero-downtime migration over 6 months
```

**2014-Present - Multi-Database:**
```text
Architecture: PostgreSQL + Cassandra + Redis + Neo4j
├─ PostgreSQL: Users, authentication
├─ Cassandra: Posts, feeds (200M posts/day)
├─ Redis: Timeline cache, counters
├─ Neo4j: Social graph (1.4B relationships)
└─ Challenge: Data consistency across databases

Decision: Eventual consistency + read-after-write
Result: Handles 1.4B users, 200M posts/day
```

📊 **By The Numbers:**
- 2010: 1 database (PostgreSQL), 2,000 writes/sec max
- 2014: 2 databases (PostgreSQL + Cassandra), 20,000 writes/sec
- 2020: 5 databases (multi-database), 200,000 writes/sec
- 2024: Optimized multi-database, handles 7,000 writes/sec peak

**Key Lesson:** No single database fits all use cases. Instagram's success came from using the right database for each data type—PostgreSQL for users (consistency), Cassandra for posts (throughput), Neo4j for relationships (graph queries).

### 🎯 Interview Questions: Database Design

#### Question 1: How would you design the database schema for a social media platform?

**What the interviewer wants to know:**
- Can you choose appropriate databases?
- Do you understand schema design?
- Can you justify your choices?

**Answer Framework:**

```text
1. Database Selection
   ├─ Users: PostgreSQL (strong consistency, ACID)
   ├─ Posts: Cassandra (high write throughput)
   ├─ Social Graph: Neo4j (relationship queries)
   └─ Cache: Redis (low latency)

2. Schema Design
   ├─ Users Table: user_id (PK), username, email, profile data
   ├─ Posts Table: post_id (PK), user_id, content, timestamps
   ├─ Follows: Graph relationships (Neo4j)
   └─ Timeline: Redis sorted sets (user_id -> [post_ids])

3. Indexing Strategy
   ├─ Users: Index on username, email (unique lookups)
   ├─ Posts: Partition by user_id, cluster by timestamp
   └─ Search: Elasticsearch for full-text search

4. Sharding Strategy
   ├─ Users: Shard by user_id (hash-based)
   ├─ Posts: Shard by user_id (co-locate user's posts)
   └─ Timeline: Shard by user_id (co-locate user's feed)
```

**Follow-up: How do you handle queries that span multiple shards?**

```text
Solution: Denormalization + Pre-computation
├─ Problem: "Get all posts from users I follow" spans shards
├─ Solution: Pre-compute timeline (fan-out on write)
├─ Store: Redis sorted set per user
└─ Trade-off: More storage, but fast reads
```

#### Question 2: How would you migrate from a single PostgreSQL database to a multi-database architecture?

**Answer Framework:**

```text
Phase 1: Dual-Write (6 months)
├─ Write to both PostgreSQL and Cassandra
├─ Read from PostgreSQL (source of truth)
├─ Verify data consistency
└─ Monitor performance

Phase 2: Gradual Read Migration
├─ Route 10% reads to Cassandra
├─ Monitor for errors
├─ Gradually increase to 100%
└─ Keep PostgreSQL as backup

Phase 3: Stop Writing to PostgreSQL
├─ Write only to Cassandra
├─ Keep PostgreSQL for historical data
└─ Archive old data

Tools:
├─ Kafka: Dual-write queue
├─ CDC: Change Data Capture for sync
└─ Monitoring: Track consistency metrics
```

### 🤔 Think About It

1. **For Beginners:** Why can't we use just one database (like PostgreSQL) for everything? (Hint: Think about what happens when 200 million people try to post at the same time)

2. **For Intermediate:** If you had to choose between strong consistency and high write throughput for posts, which would you choose? How would you explain this trade-off to a non-technical stakeholder?

3. **For Advanced:** How would you design a database schema that supports both chronological feeds and ML-ranked feeds? Consider data duplication, consistency, and query performance.

### ✅ Key Takeaways

- **Right database for the job**: PostgreSQL for users (consistency), Cassandra for posts (throughput), Neo4j for graphs (relationships)
- **Schema design follows access patterns**: Optimize for how data is queried, not just how it's stored
- **Sharding enables horizontal scaling**: Split data across multiple servers to handle growth
- **Caching reduces database load**: 90% cache hit rate means 10x less database queries
- **Eventual consistency is acceptable for feeds**: Users don't notice 1-2 second delays
- **Multi-database architecture is complex but necessary**: Trade complexity for scalability

### 🎯 Practice Exercise

**Scenario:** You're designing a database schema for a professional networking platform (like LinkedIn) with these requirements:
- 100M users
- 50M posts/articles per day
- Complex relationship queries (who knows whom, degrees of connection)
- Real-time feed with ML ranking
- Full-text search for users and content

**Your Task:**
1. Choose databases for each data type (users, posts, relationships, search)
2. Design schemas for each database
3. Design sharding strategy
4. Explain how you'd handle "2nd degree connections" queries (friends of friends)

**Bonus Challenge:** How would you handle a feature that shows "People you may know" based on mutual connections, location, and company?

---

### Database Sharding Strategy

#### Post Database Sharding (Cassandra)

```text
Sharding Strategy: User-based partitioning
Shard Key: user_id

Rationale:
- All posts from a user are co-located
- Efficient user profile page queries
- Natural distribution (users are well-distributed)

Number of shards: 512 (for future growth)

Example routing:
shard_id = hash(user_id) % 512
```

#### Timeline Cache Sharding (Redis)

```text
Sharding Strategy: Consistent hashing on user_id
Number of Redis clusters: 64

Rationale:
- Distribute load evenly
- Allow independent scaling
- Fault isolation
```

---

## Data Consistency Patterns (Extended)

### Read-After-Write Consistency

**Scenario:** User creates post and immediately refreshes feed

**Problem:**

```text
T0: User creates post
T1: Post written to primary DB (us-east)
T2: User refreshes feed (routed to us-west replica)
T3: Replication lag → Post not yet in replica
T4: User doesn't see their own post!
```

#### Solution: Route to Primary for Recent Writes

```python
class ConsistentReadRouter:
    def __init__(self):
        self.write_tracking = {}  # user_id -> timestamp
        self.replication_lag_threshold = 2  # seconds
    
    def record_write(self, user_id):
        """Track when user performed write"""
        self.write_tracking[user_id] = time.time()
    
    def route_read(self, user_id, operation):
        """Route read to ensure consistency"""
        last_write = self.write_tracking.get(user_id)
        
        if last_write:
            time_since_write = time.time() - last_write
            
            if time_since_write < self.replication_lag_threshold:
                # Route to primary for consistency
                return self.read_from_primary(operation)
            else:
                # Replication should have caught up
                del self.write_tracking[user_id]
                return self.read_from_replica(operation)
        else:
            # No recent writes, use replica
            return self.read_from_replica(operation)

# Usage
@router.post('/posts')
def create_post(post_data, user_id):
    post = db_primary.create_post(post_data)
    
    # Track write
    consistent_router.record_write(user_id)
    
    return post

@router.get('/feed')
def get_feed(user_id):
    # Route intelligently
    return consistent_router.route_read(user_id, lambda: fetch_feed(user_id))
```

### Conflict Resolution with CRDTs

**Scenario:** Offline-first mobile app allows likes while offline

**Problem:**

```text
Device A (offline):
- User likes post #123
- Stores locally: likes[123] = true

Device B (offline):
- Same user unlikes post #123
- Stores locally: likes[123] = false

Both sync later → Conflict!
```

#### Solution: Conflict-Free Replicated Data Type (CRDT)

```python
class LWW_Element_Set:
    """
    Last-Write-Wins Element Set CRDT
    Each element has a timestamp
    """
    def __init__(self):
        self.add_set = {}  # element -> timestamp
        self.remove_set = {}  # element -> timestamp
    
    def add(self, element, timestamp=None):
        """Add element with timestamp"""
        if timestamp is None:
            timestamp = time.time()
        
        self.add_set[element] = max(
            self.add_set.get(element, 0),
            timestamp
        )
    
    def remove(self, element, timestamp=None):
        """Remove element with timestamp"""
        if timestamp is None:
            timestamp = time.time()
        
        self.remove_set[element] = max(
            self.remove_set.get(element, 0),
            timestamp
        )
    
    def contains(self, element):
        """Check if element exists"""
        add_time = self.add_set.get(element, 0)
        remove_time = self.remove_set.get(element, 0)
        
        # Element exists if:
        # - It was added AND
        # - (Never removed OR add timestamp > remove timestamp)
        return add_time > 0 and add_time > remove_time
    
    def merge(self, other):
        """Merge with another CRDT (commutative, associative, idempotent)"""
        result = LWW_Element_Set()
        
        # Merge add sets (take max timestamp)
        all_elements = set(self.add_set.keys()) | set(other.add_set.keys())
        for element in all_elements:
            result.add_set[element] = max(
                self.add_set.get(element, 0),
                other.add_set.get(element, 0)
            )
        
        # Merge remove sets (take max timestamp)
        all_elements = set(self.remove_set.keys()) | set(other.remove_set.keys())
        for element in all_elements:
            result.remove_set[element] = max(
                self.remove_set.get(element, 0),
                other.remove_set.get(element, 0)
            )
        
        return result

# Usage for like system
class DistributedLikeSystem:
    def __init__(self):
        self.user_likes = {}  # user_id -> LWW_Element_Set of post_ids
    
    def like_post(self, user_id, post_id, timestamp=None):
        if user_id not in self.user_likes:
            self.user_likes[user_id] = LWW_Element_Set()
        
        self.user_likes[user_id].add(post_id, timestamp)
    
    def unlike_post(self, user_id, post_id, timestamp=None):
        if user_id not in self.user_likes:
            self.user_likes[user_id] = LWW_Element_Set()
        
        self.user_likes[user_id].remove(post_id, timestamp)
    
    def has_liked(self, user_id, post_id):
        if user_id not in self.user_likes:
            return False
        
        return self.user_likes[user_id].contains(post_id)
    
    def sync_with_server(self, user_id, server_likes):
        """Merge local changes with server state"""
        if user_id not in self.user_likes:
            self.user_likes[user_id] = server_likes
        else:
            self.user_likes[user_id] = self.user_likes[user_id].merge(server_likes)
```

---

## Section 5: API Design

### What You'll Learn

By the end of this section, you'll be able to:

- Design RESTful APIs for a social media platform
- Understand authentication and authorization patterns (OAuth 2.0, JWT)
- Design API endpoints for core features (posts, feeds, engagement)
- Handle API versioning and backward compatibility
- Implement rate limiting and API security best practices

### Why This Matters

API design is how your system communicates with the world. Real-world example: Twitter's API handles 500M tweets/day—poor API design means slow responses, breaking changes, and frustrated developers! In interviews, showing you can design clean, scalable APIs demonstrates you understand both technical and product considerations.

---

### 🟢 For Beginners: The Fundamentals

#### What is an API?

Think of an API (Application Programming Interface) like a restaurant menu:

- **Menu items** = API endpoints (what you can order)
- **Ordering process** = HTTP requests (how you place orders)
- **Food delivery** = API responses (what you get back)
- **Waiter** = API server (takes your order, brings food)

**Example API Request:**

```http
GET /api/v1/users/12345
Authorization: Bearer abc123token
```

**What this means:**
- **GET**: You want to read data (not create or update)
- **/api/v1/users/12345**: Get information about user with ID 12345
- **Authorization**: You're logged in (have permission)

**Example API Response:**

```json
{
  "user_id": "12345",
  "username": "johndoe",
  "full_name": "John Doe",
  "follower_count": 1500,
  "following_count": 500
}
```

#### Common API Operations

**CRUD Operations:**
- **Create**: POST (create a new post)
- **Read**: GET (get user profile, get feed)
- **Update**: PUT/PATCH (update profile)
- **Delete**: DELETE (delete a post)

**Real-World Analogy:**
- **POST** = Ordering food (creating something new)
- **GET** = Looking at menu (reading information)
- **PUT** = Changing your order (updating)
- **DELETE** = Canceling order (removing)

---

### 🟡 For Intermediate: Interview Patterns

#### API Design Framework

**Step 1: Identify Resources**

```text
Core Resources:
├─ Users: /api/v1/users/{user_id}
├─ Posts: /api/v1/posts/{post_id}
├─ Feeds: /api/v1/feeds/{user_id}
├─ Comments: /api/v1/posts/{post_id}/comments
└─ Likes: /api/v1/posts/{post_id}/likes
```

**Step 2: Design Endpoints**

```text
RESTful Pattern:
├─ GET /resource → List or get one
├─ POST /resource → Create new
├─ PUT /resource/{id} → Update entire resource
├─ PATCH /resource/{id} → Partial update
└─ DELETE /resource/{id} → Delete resource
```

**Step 3: Handle Authentication**

```text
Authentication Methods:
├─ OAuth 2.0: Industry standard (used by Google, Facebook)
├─ JWT Tokens: Stateless, scalable
└─ API Keys: For server-to-server

Our Choice: OAuth 2.0 + JWT
├─ OAuth 2.0: User login (get access token)
├─ JWT: Contains user info (stateless)
└─ Refresh Tokens: For long-lived sessions
```

#### Interview Script: API Design

**When asked "Design the API for creating a post":**

**You say:** "I'd design it as:

```http
POST /api/v1/posts
Authorization: Bearer {jwt_token}
Content-Type: multipart/form-data

Body:
- media_file: [binary]
- caption: "Beautiful sunset!"
- hashtags: ["sunset", "nature"]
- location: {"lat": 40.7128, "lng": -74.0060}
```

**Response:**
```json
{
  "post_id": "abc123",
  "status": "processing",
  "media_url": "https://cdn.example.com/posts/abc123.jpg",
  "created_at": "2025-11-12T10:30:00Z"
}
```

**Why this design:**
1. **POST**: Creating new resource
2. **multipart/form-data**: Supports file uploads
3. **Authorization header**: User must be logged in
4. **Status field**: Media processing is async, so return 'processing' status"

---

### 🔴 For Advanced: Production Considerations

#### API Versioning Strategy

**Why Version APIs?**

```text
Problem: Breaking changes break clients
├─ Change field name → Old clients break
├─ Remove endpoint → Mobile apps crash
└─ Change response format → Frontend errors

Solution: API Versioning
├─ URL versioning: /api/v1/, /api/v2/
├─ Header versioning: Accept: application/vnd.api+json;version=2
└─ Our choice: URL versioning (simpler, explicit)
```

**Versioning Best Practices:**

```text
Version Lifecycle:
├─ v1: Current stable (supported for 2 years)
├─ v2: New features (parallel to v1)
├─ v1 deprecated: 6-month notice before sunset
└─ v1 sunset: Stop serving, redirect to v2

Migration Strategy:
├─ Support both versions for 6 months
├─ Provide migration guide
├─ Monitor usage (when <5% use v1, sunset)
└─ Give 30-day final notice
```

#### Rate Limiting & Throttling

**Why Rate Limiting?**

```text
Protection Against:
├─ DDoS attacks (too many requests)
├─ Abuse (scraping, bots)
├─ Cost control (prevent runaway costs)
└─ Fair usage (prevent one user from hogging resources)

Rate Limit Strategy:
├─ Per-user: 1000 requests/hour
├─ Per-IP: 10,000 requests/hour
├─ Per-endpoint: Different limits (feed: 100/hour, post: 10/hour)
└─ Burst allowance: Allow 2x for 1 minute
```

**Implementation:**

```text
Token Bucket Algorithm:
├─ Bucket capacity: 1000 tokens
├─ Refill rate: 100 tokens/minute
├─ Request costs: 1 token
└─ Reject if bucket empty

Headers:
├─ X-RateLimit-Limit: 1000
├─ X-RateLimit-Remaining: 950
├─ X-RateLimit-Reset: 1636728000
└─ Retry-After: 60 (if exceeded)
```

---

### Real-World Example: How Twitter Designed Their API

**2006-2010 - Simple REST API:**
```text
Architecture: Basic REST endpoints
├─ Scale: 50M tweets/day
├─ Endpoints: ~20 endpoints
├─ Authentication: Basic Auth
└─ Challenge: No rate limiting, abuse

Decision: Add OAuth 1.0
Result: Better security, but complex
```

**2010-2015 - OAuth 2.0 Migration:**
```text
Architecture: OAuth 2.0 + REST
├─ Scale: 500M tweets/day
├─ Endpoints: ~100 endpoints
├─ Authentication: OAuth 2.0
├─ Rate Limiting: Per-user, per-endpoint
└─ Challenge: Breaking changes affect developers

Decision: API versioning (v1, v1.1)
Result: Backward compatibility, smoother transitions
```

**2015-Present - GraphQL + REST:**
```text
Architecture: REST + GraphQL hybrid
├─ Scale: 500M+ tweets/day
├─ REST: Public API (backward compatible)
├─ GraphQL: Internal API (flexible queries)
├─ Rate Limiting: Advanced (sliding window)
└─ Challenge: Two API paradigms

Decision: Keep REST public, GraphQL internal
Result: Best of both worlds
```

📊 **By The Numbers:**
- 2006: 20 endpoints, Basic Auth, no rate limiting
- 2010: 100 endpoints, OAuth 2.0, 1000 req/hour limit
- 2015: 200+ endpoints, API v1/v2, advanced rate limiting
- 2024: REST + GraphQL, 500M+ requests/day, ML-powered rate limiting

**Key Lesson:** API design evolves with scale. Start simple (REST), add complexity (versioning, rate limiting) only when needed. Twitter's success came from maintaining backward compatibility while adding new features.

### 🎯 Interview Questions: API Design

#### Question 1: How would you design the API for a social media platform?

**What the interviewer wants to know:**
- Can you design RESTful APIs?
- Do you understand authentication?
- Can you handle file uploads?

**Answer Framework:**

```text
1. Authentication
   ├─ POST /api/v1/auth/login → Get JWT token
   ├─ POST /api/v1/auth/refresh → Refresh token
   └─ All endpoints: Require Authorization header

2. User Endpoints
   ├─ GET /api/v1/users/{id} → Get user profile
   ├─ PUT /api/v1/users/{id} → Update profile
   └─ GET /api/v1/users/{id}/followers → Get followers

3. Post Endpoints
   ├─ POST /api/v1/posts → Create post (multipart)
   ├─ GET /api/v1/posts/{id} → Get post
   ├─ DELETE /api/v1/posts/{id} → Delete post
   └─ GET /api/v1/posts/{id}/comments → Get comments

4. Feed Endpoints
   ├─ GET /api/v1/feeds/home → Get home feed
   ├─ GET /api/v1/feeds/explore → Get explore feed
   └─ Pagination: ?cursor={cursor}&limit=20

5. Engagement Endpoints
   ├─ POST /api/v1/posts/{id}/likes → Like post
   ├─ DELETE /api/v1/posts/{id}/likes → Unlike
   └─ POST /api/v1/posts/{id}/comments → Comment
```

**Follow-up: How do you handle pagination for feeds?**

```text
Solution: Cursor-based pagination
├─ GET /api/v1/feeds/home?cursor=abc123&limit=20
├─ Response includes: next_cursor, has_more
├─ Advantages: No duplicates, works with real-time updates
└─ Implementation: Use post_id + timestamp as cursor
```

#### Question 2: How would you handle file uploads for photos/videos?

**Answer Framework:**

```text
1. Direct Upload to S3
   ├─ POST /api/v1/posts/upload-url → Get presigned URL
   ├─ Client uploads directly to S3
   └─ POST /api/v1/posts → Create post with S3 URL

2. Multipart Upload
   ├─ POST /api/v1/posts (multipart/form-data)
   ├─ Server receives file, uploads to S3
   └─ Returns post with processing status

3. Chunked Upload (for large videos)
   ├─ POST /api/v1/posts/upload/initiate → Get upload_id
   ├─ POST /api/v1/posts/upload/{id}/chunk → Upload chunk
   └─ POST /api/v1/posts/upload/{id}/complete → Finalize

Our Choice: Direct upload (presigned URLs)
├─ Reduces server load
├─ Faster (direct to S3)
└─ Better for large files
```

### 🤔 Think About It

1. **For Beginners:** Why do we need authentication for APIs? What happens if anyone can create posts? (Hint: Think about spam and abuse)

2. **For Intermediate:** If you had to choose between REST and GraphQL for a social media API, which would you choose? Consider developer experience, performance, and complexity.

3. **For Advanced:** How would you design an API that supports both real-time updates (WebSocket) and traditional request-response (REST)? Consider connection management, fallback strategies, and API consistency.

### ✅ Key Takeaways

- **RESTful design is standard**: Use HTTP methods correctly (GET for reads, POST for creates)
- **Authentication is critical**: OAuth 2.0 + JWT provides secure, scalable auth
- **API versioning prevents breaking changes**: Support multiple versions during transitions
- **Rate limiting protects your system**: Prevent abuse and control costs
- **Pagination is essential**: Cursor-based pagination works best for feeds
- **File uploads need special handling**: Use presigned URLs for direct S3 uploads

### 🎯 Practice Exercise

**Scenario:** Design APIs for a video-sharing platform (like TikTok) with:
- Video uploads (up to 60 seconds, 1080p)
- Real-time comments during playback
- Duet/remix features (users can create videos based on others' videos)
- Live streaming capabilities

**Your Task:**
1. Design RESTful endpoints for all features
2. Design authentication flow (OAuth 2.0)
3. Design file upload API (chunked upload for videos)
4. Design real-time comment API (WebSocket vs REST polling)

**Bonus Challenge:** How would you handle API deprecation when you need to change the video upload endpoint to support 4K videos?

---

## Section 6: Feed Generation & Ranking

### What You'll Learn

By the end of this section, you'll be able to:

- Design a hybrid feed generation system using fan-out on write and fan-out on read patterns
- Solve the "celebrity problem" where users with millions of followers would overload the system
- Implement ranking algorithms that personalize feeds based on user engagement and preferences
- Optimize feed generation for sub-500ms latency at billions of requests per day
- Understand how real-world platforms like Instagram and Twitter handle feed generation at scale

### Why This Matters

The feed is the most critical feature of any social media platform—it's what keeps users engaged and coming back. Real-world example: Instagram serves 10 billion feed impressions per day, and if feeds load slowly or show irrelevant content, users leave! In interviews, demonstrating you understand feed generation shows you can handle one of the most complex distributed systems challenges.

### 🟢 For Beginners: The Fundamentals

#### What is a Feed?

Think of a social media feed like a personalized newspaper delivered to your door every time you open the app:

- **Newspaper articles** = Posts from people you follow
- **Newspaper delivery** = Feed generation (collecting and organizing posts)
- **Personalization** = Showing you articles you're most likely to read
- **Delivery speed** = How fast your feed loads (critical for user experience!)

**The Challenge:**

When someone you follow posts something, how do you make sure it appears in YOUR feed? And how do you decide which posts to show first?

#### The Celebrity Problem

Imagine a celebrity with 100 million followers posts a photo. If we tried to update 100 million feeds immediately:

```text
100,000,000 followers × 1 millisecond per update = 100,000 seconds
= 27+ hours to update everyone's feed!
```

This is completely unacceptable! Users expect feeds to update instantly.

**Solution: Two Different Strategies**

1. **Regular Users** (< 10,000 followers): Update everyone's feed immediately when they post
2. **Celebrities** (> 10,000 followers): Don't update feeds immediately—merge their posts when users request their feed

💡 **Pro Tip:** This is called "fan-out on write" (regular users) vs "fan-out on read" (celebrities). The key insight: different user types need different strategies!

#### How Feed Generation Works

**Step 1: When Someone Posts**

```text
User creates post
    ↓
Post saved to database
    ↓
System checks: How many followers?
    ├─ < 10,000 followers → Update all followers' feeds NOW (fan-out on write)
    └─ > 10,000 followers → Just save the post, don't update feeds yet (fan-out on read)
```

**Step 2: When You Request Your Feed**

```text
You open the app
    ↓
System fetches:
    ├─ Posts from regular users (already in your feed cache)
    ├─ Recent posts from celebrities you follow (fetched now)
    └─ Merges and ranks everything
    ↓
Shows you the most relevant posts first
```

#### What is Ranking?

Ranking decides which posts appear first in your feed. Think of it like a restaurant menu—the chef puts the most popular dishes at the top!

**Factors that affect ranking:**

- ✅ **How recent** the post is (newer = higher)
- ✅ **How much you engage** with this author (more likes/comments = higher)
- ✅ **How popular** the post is (more likes = higher)
- ✅ **Your interests** (posts matching your hashtags = higher)

### 🟡 For Intermediate: Interview Patterns

#### The Feed Generation Framework

When designing a feed system in an interview, you need to address three core challenges:

**Challenge 1: The Celebrity Problem**

```text
Problem: Users with millions of followers would overload the system
Solution: Hybrid approach
├─ Regular users (<10k followers): Fan-out on write
├─ Influencers (10k-1M): Partial fanout (active followers only)
└─ Celebrities (>1M): Fan-out on read (merge at request time)
```

**Challenge 2: Feed Freshness**

```text
Problem: Users want to see new posts immediately
Solution: Real-time updates
├─ Regular posts: Already in cache (instant)
├─ Celebrity posts: Fetched on-demand (slight delay acceptable)
└─ WebSocket push: Notify users of new posts from people they follow
```

**Challenge 3: Personalization**

```text
Problem: Show users content they'll actually engage with
Solution: ML-based ranking
├─ Features: Post age, author affinity, engagement rate, content match
├─ Model: Trained on past engagement data
└─ Inference: Batch scoring during feed generation
```

#### Interview Script: Feed Generation

**When asked "How would you design a feed system?":**

**You say:** "I'd use a hybrid fan-out approach:

1. **For regular users** (<10k followers): Fan-out on write
   - When they post, immediately add to all followers' timeline caches
   - Use Redis sorted sets: `timeline:{user_id}` with post_id as member, timestamp as score
   - Batch writes using Redis pipelines for performance

2. **For celebrities** (>10k followers): Fan-out on read
   - Don't fan-out when they post (too expensive)
   - When a user requests feed, merge celebrity posts with regular timeline
   - Query: Get recent posts from celebrities user follows, merge with cached timeline

3. **Ranking algorithm:**
   - Score posts based on: time decay, author affinity, engagement rate, content match
   - Use ML model for personalization
   - Batch scoring for efficiency"

**Why this design:**
- Solves celebrity problem (no 100M writes)
- Fast for regular users (pre-computed feeds)
- Personalized (ML ranking)
- Scalable (caching + batching)

⚠️ **Common Mistake:** Trying to fan-out for ALL users, including celebrities. This doesn't scale! Always ask about follower distribution in interviews.

#### Making Decisions Explicit

```text
"Based on our discussion, I'm going to assume:

✅ Threshold: 10,000 followers separates regular users from celebrities
   → Regular users: Fan-out on write (<100ms for 10k writes)
   → Celebrities: Fan-out on read (merge at request time)

✅ Cache Strategy: Redis sorted sets for timelines
   → Fast reads (O(log N) for range queries)
   → Automatic sorting by timestamp
   → TTL: 24 hours (feeds refresh daily)

✅ Ranking: ML-based personalization
   → Features: Post age, engagement, author affinity
   → Batch inference: Score 1000 posts per call
   → Cache ranked results: 1-hour TTL

Are these assumptions reasonable?"
```

### 🔴 For Advanced: Production Considerations

#### Hybrid Fanout Strategy and Business Impact

When you're making feed generation decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Fan-out on Write vs Fan-out on Read**

```text
Scenario: User with 1M followers posts

Option A: Fan-out on Write
├─ Guarantee: All followers see post immediately
├─ Implementation: Write to 1M timeline caches
├─ Latency: 100+ seconds (unacceptable)
├─ Business Impact: System overload, user complaints
└─ Use Case: Never for large follower counts

Option B: Fan-out on Read
├─ Guarantee: Post appears when user requests feed
├─ Implementation: Merge celebrity posts at read time
├─ Latency: <200ms per feed request
├─ Business Impact: Acceptable delay, system stays healthy
└─ Use Case: Users with >10k followers

💡 Real-world: Instagram uses 10k threshold, Twitter uses similar approach.
```

**Trade-off 2: Cache Freshness vs Cost**

```text
Scenario: How often to refresh feed cache?

Option A: Real-time updates (every post)
├─ Freshness: Instant
├─ Cost: High (constant writes)
├─ Complexity: High (WebSocket infrastructure)
└─ Use Case: Premium tier, power users

Option B: Periodic refresh (every 5 minutes)
├─ Freshness: 5-minute delay
├─ Cost: Low (batch updates)
├─ Complexity: Low (scheduled jobs)
└─ Use Case: Free tier, casual users

💡 Real-world: Most platforms use hybrid—real-time for active users, periodic for inactive.
```

**Trade-off 3: Ranking Complexity vs Latency**

```text
Scenario: ML model complexity for feed ranking

Option A: Simple heuristic (time + engagement)
├─ Latency: <50ms
├─ Accuracy: 70% relevance
├─ Cost: Low (no ML infrastructure)
└─ Use Case: MVP, small scale

Option B: Deep learning model (100+ features)
├─ Latency: 200ms (batch inference)
├─ Accuracy: 90% relevance
├─ Cost: High ($50K/month ML infrastructure)
└─ Use Case: Production at scale

💡 Real-world: Instagram uses complex ML models, spending millions on ranking infrastructure.
```

#### Advanced Feed Generation Patterns

**Handling Edge Cases:**

```text
Edge Case 1: Viral Posts

Problem: Post goes viral, suddenly has 10M+ views
├─ Regular user becomes celebrity
├─ Need to switch from fan-out on write to read
└─ Implementation: Monitor follower count, auto-classify

Edge Case 2: Feed Staleness

Problem: User hasn't opened app in 7 days
├─ Cache expired (24h TTL)
├─ Need to rebuild feed from scratch
└─ Implementation: Fallback to database query, rebuild cache

Edge Case 3: Celebrity Post Spam

Problem: Celebrity posts 100 times/day
├─ Can't merge all posts (too many)
├─ Need to limit posts per celebrity
└─ Implementation: Limit to top 10 posts per celebrity per day
```

**Enterprise Requirements:**

When building for enterprise customers, requirements expand:

- **SLA guarantees:** 99.9% feed availability, <500ms p95 latency
- **Audit logs:** Track which posts shown to which users (compliance)
- **Custom ranking:** Allow enterprises to customize ranking algorithms
- **A/B testing:** Built-in framework for testing ranking changes
- **Multi-region:** Feeds generated in user's region for low latency

#### Architecture: Hybrid Approach: Fan-out on Write + Fan-out on Read

```text
┌─────────────────────────────────────────────────────────────┐
│                    Feed Generation System                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  New Post Event                                               │
│       │                                                       │
│       v                                                       │
│  ┌──────────────┐                                            │
│  │  Fanout      │                                            │
│  │  Dispatcher  │                                            │
│  └──────┬───────┘                                            │
│         │                                                     │
│         ├──────────────────┬──────────────────┐              │
│         v                  v                  v              │
│  ┌────────────┐     ┌────────────┐     ┌──────────────┐     │
│  │ Regular    │     │ Celebrity  │     │  Story       │     │
│  │ User Path  │     │ User Path  │     │  Handler     │     │
│  │ (Fan-out   │     │ (Lazy      │     │  (TTL-based) │     │
│  │  on Write) │     │  Loading)  │     │              │     │
│  └─────┬──────┘     └─────┬──────┘     └──────┬───────┘     │
│        │                  │                   │              │
│        v                  v                   v              │
│  ┌──────────────────────────────────────────────────┐        │
│  │       Redis Timeline Cache (Sorted Sets)         │        │
│  │  Key: timeline:{user_id}                         │        │
│  │  Score: timestamp, Member: post_id               │        │
│  └──────────────────────────────────────────────────┘        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

#### Regular User Path (Fan-out on Write)

For users with <10,000 followers:

1. **Post Creation:**
   - User creates post
   - Post metadata saved to Cassandra
   - Event published to Kafka topic `post.created`

2. **Fanout Service Processes Event:**
   - Query Graph DB for all followers (batch query)
   - For each follower, add post to their timeline cache
   - Use Redis pipeline for batch writes
   - Target: 10,000 writes in <100ms

3. **Timeline Cache Update:**

   ```text
   For each follower_id:
     ZADD timeline:{follower_id} {post_timestamp} {post_id}
     LTRIM timeline:{follower_id} 0 999  # Keep only top 1000 posts
     EXPIRE timeline:{follower_id} 86400  # 24-hour TTL
   ```

4. **Feed Request:**
   - User requests feed
   - Read from `timeline:{user_id}` (Redis sorted set)
   - Fetch post metadata from cache/DB
   - Return results

#### Celebrity User Path (Fan-out on Read)

For users with >10,000 followers (celebrity threshold):

1. **Post Creation:**
   - Post saved to Cassandra
   - Mark user as celebrity (if follower_count > 10k)
   - NO fanout to followers

2. **Feed Request (Merge on Read):**
   - User requests feed
   - Fetch regular timeline from cache
   - Query: "Get recent posts from celebrity users I follow"
   - Merge celebrity posts with regular timeline
   - Rank combined results
   - Return merged feed

3. **Celebrity Post Query:**

   ```text
   SELECT post_id, created_at
   FROM posts_by_user
   WHERE user_id IN (SELECT celebrity_ids FROM user_following WHERE user_id = {requesting_user})
   AND created_at > {last_24_hours}
   ORDER BY created_at DESC
   LIMIT 50
   ```

4. **Merge Logic:**
   - Fetch regular timeline (200 posts from cache)
   - Fetch celebrity posts (50 most recent)
   - Merge based on timestamp
   - Apply ranking algorithm
   - Return top 50 posts

#### Ranking Algorithm

**Input Features:**

```python
features = {
    # Content features
    'post_age': time_since_post_creation,
    'media_type': 'photo' | 'video' | 'carousel',
    'has_multiple_media': boolean,
    
    # Author features
    'author_follower_count': int,
    'author_is_verified': boolean,
    'user_follows_author': boolean,
    'user_author_interaction_score': float,  # Past engagement
    
    # Engagement features
    'like_count': int,
    'comment_count': int,
    'share_count': int,
    'view_count': int,
    'engagement_rate': (likes + comments + shares) / views,
    'engagement_velocity': engagement_count / time_since_post,
    
    # User preference features
    'user_past_engagement_with_type': float,
    'user_past_engagement_with_hashtags': float,
    'user_location_match': boolean,
    'user_interest_match': float,
    
    # Contextual features
    'time_of_day': hour,
    'day_of_week': int,
    'user_session_length': float
}
```

**Scoring Model:**

```python
def calculate_relevance_score(post, user, context):
    """
    Calculate relevance score for a post
    
    Returns:
        float: Score between 0 and 1
    """
    # Time decay (exponential)
    age_hours = (now - post.created_at).hours
    time_decay = exp(-age_hours / 24)  # 24-hour half-life
    
    # Author score
    author_score = (
        0.3 * user.following_affinity[post.author_id] +  # How much user engages with author
        0.2 * post.author.is_verified +
        0.1 * normalize(post.author.follower_count)
    )
    
    # Engagement score
    engagement_score = (
        0.4 * normalize(post.like_count / post.view_count) +  # Like rate
        0.3 * normalize(post.comment_count) +  # Absolute comments
        0.3 * normalize(post.engagement_velocity)  # Trending factor
    )
    
    # Content match score
    content_score = (
        0.5 * cosine_similarity(user.interests, post.hashtags) +
        0.3 * (1 if post.media_type in user.preferred_types else 0) +
        0.2 * (1 if post.location == user.location else 0)
    )
    
    # Final score (weighted combination)
    final_score = (
        0.25 * time_decay +
        0.30 * author_score +
        0.30 * engagement_score +
        0.15 * content_score
    )
    
    return final_score
```

**Implementation:**

- Model trained using TensorFlow/PyTorch
- Inference via TensorFlow Serving or ONNX Runtime
- Batch prediction for feed generation
- Model updated daily with recent engagement data
- A/B testing framework for model experiments

#### Performance Optimizations

1. **Caching Layers:**
   - L1: User timeline cache (Redis) - 100GB per cluster
   - L2: Post metadata cache (Memcached) - 50GB per cluster
   - L3: Ranked feed cache (Redis) - 1-hour TTL

2. **Batch Operations:**
   - Batch fetch post metadata (100 posts per query)
   - Batch scoring (1000 posts per ML inference call)
   - Redis pipelining for multiple ZADD operations

3. **Precomputation:**
   - Explore feed pre-generated every 10 minutes
   - Trending content updated every 5 minutes
   - User interest vectors updated daily

4. **Query Optimization:**
   - Fetch only necessary fields (lean queries)
   - Parallel queries to multiple shards
   - Query result pagination

### Real-World Example: How Instagram Evolved Their Feed System

Let's look at how Instagram evolved their feed generation system over time:

**2010-2012 - Chronological Feed:**
```text
Architecture: Simple chronological feed
├─ Feature: Show posts in order (newest first)
├─ Scale: 100M posts/day
├─ Decision: Fan-out on write for all users
└─ Result: Simple, but users missed important posts

Challenge: Users following 1000+ accounts missed 70% of posts
```

**2012-2016 - Algorithmic Feed:**
```text
Architecture: ML-based ranking
├─ Added: Ranking algorithm (engagement + time)
├─ Scale: 500M posts/day
├─ Decision: Hybrid fan-out (10k threshold)
└─ Result: 50% increase in engagement

Challenge: Users complained about "missing posts"
```

**2016-Present - Advanced Personalization:**
```text
Architecture: Deep learning ranking
├─ Added: 100+ features, neural networks
├─ Scale: 1B+ posts/day
├─ Decision: Real-time updates + pre-computed feeds
└─ Result: 90% relevance, <300ms latency

Key Innovation: Separate feeds for Stories, Reels, Posts
```

📊 **By The Numbers:**
- 2010: Chronological feed, 100M posts/day, 50% engagement rate
- 2016: Algorithmic feed, 500M posts/day, 75% engagement rate
- 2024: Advanced ML ranking, 1B+ posts/day, 90% engagement rate, <300ms p95 latency

**Key Lesson:** Feed generation evolves from simple (chronological) to complex (ML ranking). The key is balancing freshness, relevance, and performance. Instagram's success came from solving the celebrity problem early and continuously improving ranking algorithms.

### 🎯 Interview Questions: Feed Generation

#### Question 1: How would you design a feed system for a social media platform?

**What the interviewer wants to know:**
- Do you understand the celebrity problem?
- Can you design hybrid fan-out strategies?
- Do you think about ranking and personalization?

**Answer Framework:**

```text
1. Problem Analysis
   ├─ Identify: Celebrity problem (users with millions of followers)
   ├─ Identify: Need for personalization (ranking)
   └─ Identify: Latency requirements (<500ms)

2. Hybrid Fan-out Strategy
   ├─ Regular users (<10k): Fan-out on write
   │  ├─ Update all followers' caches immediately
   │  ├─ Use Redis sorted sets
   │  └─ Target: <100ms for 10k writes
   ├─ Celebrities (>10k): Fan-out on read
   │  ├─ Don't fan-out when posting
   │  ├─ Merge celebrity posts at feed request time
   │  └─ Target: <200ms per feed request
   └─ Threshold: 10,000 followers (configurable)

3. Ranking Algorithm
   ├─ Features: Post age, engagement, author affinity, content match
   ├─ ML Model: Trained on past engagement data
   ├─ Inference: Batch scoring (1000 posts per call)
   └─ Cache: Ranked results cached for 1 hour

4. Caching Strategy
   ├─ L1: Timeline cache (Redis sorted sets) - 24h TTL
   ├─ L2: Post metadata cache (Memcached) - 1h TTL
   └─ L3: Ranked feed cache (Redis) - 1h TTL

Scaling Example:
├─ 100M users: 10,000 Redis instances
├─ 1B posts/day: 11,500 posts/second
└─ Feed requests: 50,000 QPS (peak)
```

**Follow-up: How do you handle a user who suddenly gains 1 million followers?**

```text
Solution: Dynamic threshold adjustment
├─ Monitor follower count in real-time
├─ When user crosses 10k threshold:
│  ├─ Stop fan-out on write
│  ├─ Switch to fan-out on read
│  └─ Rebuild existing caches (gradual)
└─ Handle existing cached posts: Let them expire naturally
```

#### Question 2: How would you optimize feed generation latency?

**What the interviewer wants to know:**
- Can you identify bottlenecks?
- Do you understand caching strategies?
- Can you optimize ML inference?

**Answer Framework:**

```text
1. Identify Bottlenecks
   ├─ Database queries: Fetching post metadata
   ├─ ML inference: Scoring posts
   ├─ Network: Merging celebrity posts
   └─ Cache misses: Rebuilding timelines

2. Optimization Strategies
   ├─ Caching
   │  ├─ Pre-compute ranked feeds (1h TTL)
   │  ├─ Cache post metadata (1h TTL)
   │  └─ Cache celebrity posts (5min TTL)
   ├─ Batch Operations
   │  ├─ Batch fetch metadata (100 posts per query)
   │  ├─ Batch ML inference (1000 posts per call)
   │  └─ Redis pipelining for writes
   ├─ Parallel Processing
   │  ├─ Fetch regular + celebrity posts in parallel
   │  ├─ Parallel queries to multiple shards
   │  └─ Async ranking computation
   └─ Precomputation
      ├─ Pre-generate explore feeds (10min refresh)
      ├─ Pre-compute trending content (5min refresh)
      └─ Update user interest vectors daily

3. Latency Targets
   ├─ Regular feed (cache hit): <50ms
   ├─ Regular feed (cache miss): <200ms
   ├─ Feed with celebrities: <300ms
   └─ P95 latency: <500ms
```

#### Question 3: How do you ensure feed freshness while maintaining performance?

**What the interviewer wants to know:**
- Do you understand cache invalidation?
- Can you balance freshness vs performance?
- Do you think about real-time updates?

**Answer Framework:**

```text
1. Multi-Tier Freshness Strategy
   ├─ Real-time: WebSocket push for new posts from close connections
   ├─ Near real-time: 5-minute refresh for active users
   └─ Periodic: 1-hour refresh for inactive users

2. Cache Invalidation
   ├─ TTL-based: Automatic expiration (24h for timelines)
   ├─ Event-driven: Invalidate on new posts from followed users
   └─ Manual refresh: User pull-to-refresh

3. Hybrid Approach
   ├─ Active users (<24h since last login): Real-time updates
   ├─ Regular users (1-7 days): 5-minute refresh
   └─ Inactive users (>7 days): 1-hour refresh or on-demand

4. Implementation
   ├─ WebSocket: Push notifications for new posts
   ├─ Background jobs: Periodic feed refresh
   └─ On-demand: User-initiated refresh
```

### 🤔 Think About It

1. **For Beginners:** Why do we use different strategies for regular users vs celebrities? What would happen if we tried to update 100 million feeds every time a celebrity posts? (Hint: Think about how long 100 million operations would take)

2. **For Intermediate:** If you had to choose between showing users the most recent posts vs the most relevant posts, which would you prioritize? Consider user engagement, retention, and system complexity.

3. **For Advanced:** How would your feed generation system change if you were building for:
   - A professional network (LinkedIn-style) where users want to see industry-relevant content?
   - A photo-sharing app (Instagram-style) where visual content and engagement matter most?
   - A news platform (Twitter-style) where real-time updates and trending topics are critical?

### ✅ Key Takeaways

- **Hybrid fan-out solves the celebrity problem**: Use fan-out on write for regular users (<10k followers) and fan-out on read for celebrities (>10k followers)
- **Ranking is essential for engagement**: ML-based personalization increases user engagement by 50-90% compared to chronological feeds
- **Caching is critical for performance**: Multi-layer caching (timeline cache, metadata cache, ranked feed cache) enables sub-500ms latency
- **Batch operations improve efficiency**: Batch fetching, batch ML inference, and Redis pipelining reduce latency and cost
- **Thresholds are configurable**: The 10k follower threshold can be adjusted based on system capacity and user behavior
- **Real-time updates balance freshness and cost**: Use WebSocket for active users, periodic refresh for others

### 🎯 Practice Exercise

**Scenario:** Design a feed system for a professional networking platform (like LinkedIn) with these requirements:
- Users follow companies, individuals, and topics
- Feed should show industry-relevant content, not just chronological
- Companies can have millions of followers (celebrity problem)
- Users want to see posts from their industry, recent job postings, and updates from connections
- Feed should load in <300ms

**Your Task:**
1. Design the fan-out strategy (regular users vs companies with millions of followers)
2. Design the ranking algorithm (what factors matter for professional content?)
3. Design the caching strategy (what should be cached and for how long?)
4. Design real-time update mechanism (how to notify users of important updates?)

**Bonus Challenge:** How would you handle a viral post from a regular user that suddenly gets 10 million views? How does this affect your fan-out strategy?

---

## Section 7: Media Processing & Storage

### What You'll Learn

By the end of this section, you'll be able to:

- Design a scalable media processing pipeline that handles thousands of uploads per second
- Implement photo and video processing workflows (thumbnails, transcoding, optimization)
- Design storage architecture for media files with CDN distribution
- Handle media uploads with resumable chunked uploads and error recovery
- Optimize media processing for cost and performance at scale

### Why This Matters

Media processing is one of the most resource-intensive operations in a social media platform. Real-world example: Instagram processes over 100 million photos per day—if processing takes too long or fails, users can't share their content! In interviews, showing you can design efficient media pipelines demonstrates you understand distributed processing, queue management, and cost optimization.

### 🟢 For Beginners: The Fundamentals

#### What is Media Processing?

Think of media processing like a photo printing shop:

- **You bring photos** = Users upload photos/videos
- **Shop processes them** = System resizes, optimizes, creates thumbnails
- **Different sizes** = Small (profile), Medium (feed), Large (full view)
- **Delivery** = Processed media delivered via CDN (like having multiple shop locations)

**The Challenge:**

When someone uploads a photo, the system needs to:
1. Store the original
2. Create multiple sizes (thumbnails)
3. Optimize for web (compress, convert format)
4. Make it available worldwide (CDN)

All of this needs to happen quickly so users can see their posts!

#### Why Do We Process Media?

**Photo Processing:**

```text
Original photo: 5MB, 4000x3000 pixels
    ↓
Processing:
├─ Create thumbnail: 150x150 (0.05MB) - for profile grid
├─ Create medium: 640x640 (0.2MB) - for feed
├─ Create large: 1080x1080 (0.8MB) - for full view
└─ Optimize: Convert to WebP format (30% smaller)

Result: Users download smaller files = faster loading!
```

**Video Processing:**

```text
Original video: 500MB, 1080p
    ↓
Processing:
├─ Create 360p version (for slow connections)
├─ Create 720p version (for medium connections)
├─ Create 1080p version (for fast connections)
└─ Extract thumbnail frames

Result: Adaptive streaming = smooth playback on any connection!
```

💡 **Pro Tip:** Processing media takes time, so we do it asynchronously (in the background) while the user can continue using the app!

#### How Media Upload Works

**Step 1: Upload**

```text
User selects photo/video
    ↓
App uploads directly to cloud storage (S3)
    ↓
System receives notification: "New media uploaded"
    ↓
Adds to processing queue
```

**Step 2: Processing**

```text
Worker picks up job from queue
    ↓
Downloads media from storage
    ↓
Processes (resize, optimize, transcode)
    ↓
Uploads processed versions back to storage
    ↓
Updates database: "Media ready!"
```

**Step 3: Delivery**

```text
User views post
    ↓
App requests media URL
    ↓
CDN serves optimized version
    ↓
Fast loading, worldwide!
```

### 🟡 For Intermediate: Interview Patterns

#### The Media Processing Framework

When designing a media processing system in an interview, address these core challenges:

**Challenge 1: Scalability**

```text
Problem: Thousands of uploads per second
Solution: Distributed processing
├─ Queue-based architecture (Kafka/RabbitMQ)
├─ Worker pools (auto-scaling)
├─ Parallel processing (multiple workers)
└─ Direct upload to S3 (bypass API servers)
```

**Challenge 2: Reliability**

```text
Problem: Processing can fail (network, corruption, etc.)
Solution: Error handling
├─ Retry logic (exponential backoff)
├─ Dead letter queue (for failed jobs)
├─ Resumable uploads (chunked upload)
└─ Validation (file format, malware scanning)
```

**Challenge 3: Cost Optimization**

```text
Problem: Processing is expensive (CPU, storage)
Solution: Optimization strategies
├─ Spot instances for workers (70% cost savings)
├─ Lazy processing (process on-demand for some formats)
├─ CDN caching (reduce origin requests)
└─ Compression (smaller files = less storage/bandwidth)
```

#### Interview Script: Media Processing

**When asked "How would you handle media uploads and processing?":**

**You say:** "I'd design an asynchronous processing pipeline:

1. **Upload Strategy:**
   - Direct upload to S3 using pre-signed URLs (bypasses API servers)
   - Chunked upload for large files (resumable if connection drops)
   - Client-side validation (file size, format) before upload

2. **Processing Pipeline:**
   - Queue-based: Kafka topic for processing jobs
   - Workers: Auto-scaling Kubernetes pods (scale based on queue depth)
   - Processing: Generate thumbnails, optimize, transcode videos
   - Storage: Store processed versions in S3 with CDN distribution

3. **Error Handling:**
   - Retry failed jobs (max 3 retries with exponential backoff)
   - Dead letter queue for permanently failed jobs
   - Monitoring: Track processing time, success rates, queue depth"

**Why this design:**
- Scalable (handles traffic spikes with auto-scaling)
- Reliable (retries, resumable uploads)
- Cost-effective (spot instances, CDN caching)
- Fast (direct upload, parallel processing)

⚠️ **Common Mistake:** Processing media synchronously (blocking the API request). This doesn't scale! Always use async processing with queues.

#### Making Decisions Explicit

```text
"Based on our discussion, I'm going to assume:

✅ Upload: Direct to S3 (pre-signed URLs)
   → Reduces API server load
   → Faster uploads (direct to storage)
   → Supports resumable uploads

✅ Processing: Async queue (Kafka)
   → Non-blocking (users don't wait)
   → Scalable (add more workers)
   → Reliable (retry failed jobs)

✅ Storage: S3 + CDN (CloudFront/Akamai)
   → S3: Durable, scalable storage
   → CDN: Fast global delivery
   → Cost: Pay for what you use

✅ Workers: Kubernetes auto-scaling
   → Scale up: Queue depth > 1000
   → Scale down: Queue depth < 100
   → Use spot instances: 70% cost savings

Are these assumptions reasonable?"
```

### 🔴 For Advanced: Production Considerations

#### Media Processing Architecture and Business Impact

When you're making media processing decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Processing Time vs Cost**

```text
Scenario: Video transcoding for 1M videos/day

Option A: Fast processing (GPU instances)
├─ Processing time: 10 seconds per video
├─ Cost: $50K/month (GPU instances)
├─ Latency: Low (users see videos quickly)
└─ Use Case: Premium tier, high-value content

Option B: Slow processing (CPU instances)
├─ Processing time: 60 seconds per video
├─ Cost: $10K/month (CPU instances)
├─ Latency: Higher (users wait longer)
└─ Use Case: Free tier, cost-sensitive

💡 Real-world: Most platforms use hybrid—GPU for videos, CPU for photos.
```

**Trade-off 2: Storage Redundancy vs Cost**

```text
Scenario: Storing 1PB of media

Option A: Single region (S3 standard)
├─ Durability: 99.999999999% (11 9's)
├─ Cost: $23K/month
├─ Risk: Region outage = service unavailable
└─ Use Case: Cost-sensitive, acceptable downtime

Option B: Multi-region replication
├─ Durability: 99.999999999% per region
├─ Cost: $46K/month (2x storage)
├─ Risk: Region outage = automatic failover
└─ Use Case: High availability requirement

💡 Real-world: Instagram uses multi-region for critical media, single region for older content.
```

**Trade-off 3: Processing Quality vs Speed**

```text
Scenario: Photo optimization

Option A: High quality (2-pass encoding)
├─ Quality: 95% (near-lossless)
├─ Processing time: 5 seconds
├─ File size: Larger (more storage/bandwidth cost)
└─ Use Case: Professional photographers, premium content

Option B: Fast processing (1-pass encoding)
├─ Quality: 85% (good enough)
├─ Processing time: 2 seconds
├─ File size: Smaller (less cost)
└─ Use Case: General users, mobile uploads

💡 Real-world: Most platforms use adaptive quality—higher for popular content, lower for casual posts.
```

#### Advanced Media Processing Patterns

**Handling Edge Cases:**

```text
Edge Case 1: Viral Video

Problem: Video goes viral, needs processing ASAP
├─ Normal queue: 1-hour wait time
├─ Need: Immediate processing
└─ Implementation: Priority queue, dedicated GPU workers

Edge Case 2: Corrupted Upload

Problem: File corrupted during upload
├─ Detection: Validation after upload
├─ Recovery: Request re-upload from client
└─ Prevention: Checksum validation, chunked upload with verification

Edge Case 3: Storage Quota Exceeded

Problem: User exceeds storage limit
├─ Detection: Check quota before processing
├─ Action: Reject upload, notify user
└─ Implementation: Quota service, pre-upload validation
```

**Enterprise Requirements:**

When building for enterprise customers, requirements expand:

- **SLA guarantees:** 99.9% processing success rate, <30s processing time
- **Compliance:** GDPR (right to deletion), HIPAA (encrypted storage)
- **Audit logs:** Track all media processing operations
- **Custom processing:** Allow enterprises to define custom workflows
- **Multi-tenant:** Isolated processing queues per customer

#### Architecture: Media Processing Pipeline

```text
┌────────────────────────────────────────────────────────────┐
│                 Media Processing Pipeline                   │
├────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Upload          2. Queue         3. Process             │
│  ┌──────────┐      ┌──────────┐      ┌──────────────┐      │
│  │  API     │─────>│  Kafka   │─────>│   Workers    │      │
│  │ Gateway  │      │  Topic:  │      │  (FFmpeg)    │      │
│  │          │      │  media.  │      │              │      │
│  │ Direct   │      │  process │      │ 100k workers │      │
│  │ to S3    │      └──────────┘      └──────┬───────┘      │
│  └──────────┘                               │              │
│                                              v              │
│                                       ┌────────────┐        │
│                                       │  Generate  │        │
│  S3 Buckets                           │ Thumbnails │        │
│  ┌──────────────────────────────┐    │  Transcode │        │
│  │ /original/{post_id}/         │<───┤   Videos   │        │
│  │ /thumbnails/{post_id}/       │    │  Extract   │        │
│  │ /processed/{post_id}/        │    │  Metadata  │        │
│  └──────────────────────────────┘    └────────────┘        │
│                                                              │
│  4. CDN Distribution                                         │
│  ┌────────────────────────────────────────────────┐         │
│  │  CloudFront / Akamai                           │         │
│  │  - 200+ edge locations                         │         │
│  │  - Cache hit ratio: 95%+                       │         │
│  └────────────────────────────────────────────────┘         │
│                                                              │
└────────────────────────────────────────────────────────────┘
```

#### Processing Steps

**Photo Processing:**

1. **Upload:**
   - Multipart upload to S3 (client-side chunking)
   - Generate pre-signed URL for direct upload
   - Max file size: 10MB

2. **Thumbnail Generation:**
   - Small: 150x150 (profile grid)
   - Medium: 640x640 (feed)
   - Large: 1080x1080 (full view)

3. **Optimization:**
   - Format: Convert to WebP (30% smaller than JPEG)
   - Compression: 85% quality
   - Strip EXIF data (privacy)
   - Apply content-aware cropping

4. **Processing Time:** ~2 seconds per photo

**Video Processing:**

1. **Upload:**
   - Chunked upload (10MB chunks)
   - Resumable upload support
   - Max file size: 500MB
   - Max duration: 10 minutes

2. **Transcoding:**
   - Formats: 360p, 480p, 720p, 1080p
   - Codec: H.264 (compatibility), H.265 (efficiency)
   - Adaptive bitrate streaming (HLS/DASH)
   - Audio: AAC, 128kbps

3. **Thumbnail Generation:**
   - Extract frame at 10% duration
   - Generate 3 preview frames
   - Create animated GIF preview (3 seconds)

4. **Optimization:**
   - Two-pass encoding for better quality
   - Content-aware bitrate adjustment
   - Remove audio if silent

5. **Processing Time:** ~30 seconds per minute of video

#### Worker Architecture

**Horizontal Scaling:**

- Kubernetes-based worker pools
- Auto-scaling based on queue depth
- Target: Queue depth < 1000 messages
- Scale-up trigger: Depth > 1000 for 2 minutes
- Scale-down trigger: Depth < 100 for 10 minutes

**Worker Configuration:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: media-processor
spec:
  replicas: 1000
  template:
    spec:
      containers:
      - name: processor
        image: media-processor:v1
        resources:
          requests:
            cpu: "2"
            memory: "4Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        env:
        - name: KAFKA_TOPIC
          value: "media.process"
        - name: S3_BUCKET
          value: "socialmedia-media"
        - name: WORKER_THREADS
          value: "4"
```

**Processing Queue (Kafka):**

- Topic: `media.process`
- Partitions: 64 (for parallelism)
- Replication factor: 3
- Retention: 7 days
- Compression: LZ4

**Message Format:**

```json
{
  "job_id": "uuid",
  "post_id": "uuid",
  "user_id": "uuid",
  "media_type": "photo" | "video",
  "source_url": "s3://bucket/original/post_id/file.jpg",
  "output_bucket": "socialmedia-media",
  "priority": "high" | "normal" | "low",
  "processing_options": {
    "thumbnail_sizes": [150, 640, 1080],
    "video_formats": ["360p", "720p", "1080p"],
    "optimize_for_mobile": true
  },
  "created_at": "2025-10-02T20:00:00Z"
}
```

#### Error Handling

1. **Retry Logic:**
   - Max retries: 3
   - Backoff: Exponential (1s, 2s, 4s)
   - Dead letter queue for failed jobs

2. **Validation:**
   - File format validation
   - Malware scanning (ClamAV)
   - Content moderation (AWS Rekognition)
   - NSFW detection

3. **Monitoring:**
   - Processing time per job
   - Success/failure rates
   - Queue depth alerts
   - Worker utilization

### Real-World Example: How YouTube Processes Videos at Scale

Let's look at how YouTube evolved their video processing system:

**2005-2010 - Basic Processing:**
```text
Architecture: Single server processing
├─ Feature: Basic transcoding (one format)
├─ Scale: 100K videos/day
├─ Decision: Process videos sequentially
└─ Result: Long wait times, frequent failures

Challenge: Processing took hours, users waited days to see videos
```

**2010-2015 - Distributed Processing:**
```text
Architecture: Distributed workers
├─ Added: Multiple formats (360p, 480p, 720p, 1080p)
├─ Scale: 1M videos/day
├─ Decision: Queue-based processing, parallel workers
└─ Result: Processing time reduced to minutes

Challenge: Cost explosion (thousands of servers)
```

**2015-Present - Optimized Pipeline:**
```text
Architecture: GPU-accelerated, adaptive processing
├─ Added: GPU transcoding, adaptive bitrate (HLS/DASH)
├─ Scale: 10M+ videos/day
├─ Decision: Spot instances, lazy processing, CDN caching
└─ Result: 70% cost reduction, <30s processing time

Key Innovation: Process popular videos first, defer others
```

📊 **By The Numbers:**
- 2005: 100K videos/day, hours per video, single format
- 2010: 1M videos/day, minutes per video, 4 formats
- 2024: 10M+ videos/day, <30s per video, adaptive streaming, 70% cost reduction

**Key Lesson:** Media processing evolves from simple (single server) to complex (distributed GPU clusters). The key is balancing processing speed, cost, and quality. YouTube's success came from optimizing for cost while maintaining quality through spot instances and adaptive processing.

### 🎯 Interview Questions: Media Processing

#### Question 1: How would you design a media processing pipeline?

**What the interviewer wants to know:**
- Do you understand async processing?
- Can you design scalable worker systems?
- Do you think about error handling and reliability?

**Answer Framework:**

```text
1. Upload Strategy
   ├─ Direct upload to S3 (pre-signed URLs)
   ├─ Chunked upload for large files (resumable)
   └─ Client-side validation (file size, format)

2. Processing Pipeline
   ├─ Queue: Kafka topic for processing jobs
   ├─ Workers: Auto-scaling Kubernetes pods
   │  ├─ Scale up: Queue depth > 1000
   │  ├─ Scale down: Queue depth < 100
   │  └─ Use spot instances: 70% cost savings
   ├─ Processing: Generate thumbnails, optimize, transcode
   └─ Storage: Store processed versions in S3

3. Error Handling
   ├─ Retry logic: Max 3 retries, exponential backoff
   ├─ Dead letter queue: For permanently failed jobs
   └─ Monitoring: Track processing time, success rates

4. CDN Distribution
   ├─ CloudFront/Akamai: 200+ edge locations
   ├─ Cache hit ratio: 95%+
   └─ TTL: 1 year for processed media

Scaling Example:
├─ 7,000 uploads/sec: 100,000 workers (spot instances)
├─ Processing time: 2s photos, 30s per minute video
└─ Cost: $50K/month (vs $150K with on-demand instances)
```

**Follow-up: How do you handle a video that fails processing 3 times?**

```text
Solution: Dead letter queue + manual review
├─ Move to dead letter queue after 3 retries
├─ Alert operations team
├─ Manual investigation: Check file corruption, worker logs
└─ Recovery: Fix issue, requeue job or request re-upload
```

#### Question 2: How would you optimize media processing costs?

**What the interviewer wants to know:**
- Can you identify cost drivers?
- Do you understand spot instances and reserved capacity?
- Can you optimize storage costs?

**Answer Framework:**

```text
1. Compute Optimization
   ├─ Spot instances: 70% cost savings (vs on-demand)
   ├─ Reserved instances: 40% savings for baseline capacity
   ├─ Auto-scaling: Scale down during low traffic
   └─ GPU vs CPU: Use GPU for videos, CPU for photos

2. Storage Optimization
   ├─ Compression: WebP for photos (30% smaller)
   ├─ Lifecycle policies: Move old media to cheaper storage
   ├─ CDN caching: Reduce origin requests (95% cache hit)
   └─ Multi-region: Only replicate popular content

3. Processing Optimization
   ├─ Lazy processing: Process formats on-demand
   ├─ Priority queue: Process popular content first
   └─ Batch processing: Group similar jobs together

4. Cost Monitoring
   ├─ Track: Processing cost per media type
   ├─ Alert: Cost spikes (>20% increase)
   └─ Optimize: Regular cost reviews, spot instance usage

Cost Example:
├─ On-demand: $150K/month
├─ Spot instances: $45K/month (70% savings)
├─ Reserved baseline: $30K/month (40% savings)
└─ Total: $75K/month (50% cost reduction)
```

#### Question 3: How do you ensure media processing reliability?

**What the interviewer wants to know:**
- Do you understand failure modes?
- Can you design retry and recovery mechanisms?
- Do you think about monitoring and alerting?

**Answer Framework:**

```text
1. Failure Prevention
   ├─ Validation: File format, size, corruption checks
   ├─ Malware scanning: ClamAV before processing
   ├─ Quota checks: Verify user storage limits
   └─ Health checks: Monitor worker health

2. Failure Handling
   ├─ Retry logic: Exponential backoff (1s, 2s, 4s)
   ├─ Dead letter queue: For permanently failed jobs
   ├─ Resumable uploads: Chunked upload with verification
   └─ Fallback: Lower quality processing if high quality fails

3. Monitoring & Alerting
   ├─ Metrics: Processing time, success rate, queue depth
   ├─ Alerts: Queue depth > 5000, success rate < 95%
   ├─ Dashboards: Real-time processing status
   └─ Logging: Detailed logs for debugging failures

4. Recovery Procedures
   ├─ Automatic: Retry failed jobs
   ├─ Manual: Dead letter queue review
   └─ Escalation: Alert operations for critical failures
```

### 🤔 Think About It

1. **For Beginners:** Why do we process media asynchronously (in the background) instead of making users wait? What would happen if processing took 5 minutes and users had to wait? (Hint: Think about user experience and server resources)

2. **For Intermediate:** If you had to choose between processing all media formats immediately vs processing formats on-demand (lazy processing), which would you choose? Consider cost, user experience, and system complexity.

3. **For Advanced:** How would your media processing system change if you were building for:
   - A professional photography platform where image quality is critical?
   - A mobile-first app where users have limited data plans?
   - A live streaming platform where processing needs to happen in real-time?

### ✅ Key Takeaways

- **Async processing is essential**: Process media in background queues to avoid blocking users and API servers
- **Direct upload to storage**: Use pre-signed URLs for direct S3 uploads to reduce API server load
- **Auto-scaling workers**: Scale worker pools based on queue depth to handle traffic spikes cost-effectively
- **Spot instances reduce costs**: Use spot instances for workers to achieve 70% cost savings
- **CDN distribution is critical**: Serve processed media via CDN for fast global delivery and reduced origin load
- **Error handling is mandatory**: Implement retry logic, dead letter queues, and monitoring for reliable processing

### 🎯 Practice Exercise

**Scenario:** Design a media processing system for a video-sharing platform (like TikTok) with these requirements:
- Users upload videos up to 60 seconds, 1080p resolution
- Videos need multiple formats: 360p, 480p, 720p, 1080p
- Processing must complete within 30 seconds
- System handles 10,000 uploads per second at peak
- Cost must be minimized (budget: $100K/month)

**Your Task:**
1. Design the upload strategy (direct upload vs API upload)
2. Design the processing pipeline (queue, workers, scaling)
3. Design the storage architecture (S3, CDN, formats)
4. Design cost optimization strategies (spot instances, lazy processing, etc.)

**Bonus Challenge:** How would you handle a viral video that needs processing immediately while maintaining fairness for other users?

---

### 3. Fanout Service (Celebrity Problem Solution)

*Note: This content is already covered in Section 6: Feed Generation & Ranking. This section can be removed or kept as a reference.*

Solves the "celebrity problem" where a single post to 100M followers would cause system overload.

#### Problem Statement

**Naive Fanout (Doesn't Scale):**

```text
Celebrity with 100M followers posts:
- Need to write to 100M timelines
- At 1ms per write = 100,000 seconds = 27+ hours
- Completely unacceptable
```

#### Solution: Hybrid Fanout Strategy

**Decision Tree:**

```text
New Post Created
    │
    ├─> Follower Count < 10k
    │   └─> Fan-out on Write (push to all follower timelines)
    │
    ├─> Follower Count 10k - 1M
    │   └─> Partial Fanout (push to active followers only)
    │
    └─> Follower Count > 1M
        └─> Fan-out on Read (no pre-computation, merge at read time)
```

#### Implementation

**User Classification:**

```python
def classify_user(user):
    """
    Classify user type based on follower count
    """
    if user.follower_count < 10_000:
        return 'regular'
    elif user.follower_count < 1_000_000:
        return 'influencer'
    else:
        return 'celebrity'
```

**Regular User Fanout (< 10k followers):**

```python
def fanout_regular_user(post, author):
    """
    Push post to all follower timelines
    """
    # Get all followers (from Graph DB or cached)
    follower_ids = graph_db.get_followers(author.user_id)
    
    # Batch write to Redis (use pipeline)
    pipeline = redis_client.pipeline()
    for follower_id in follower_ids:
        pipeline.zadd(
            f'timeline:{follower_id}',
            {post.post_id: post.created_at.timestamp()}
        )
        pipeline.ltrim(f'timeline:{follower_id}', 0, 999)  # Keep top 1000
        pipeline.expire(f'timeline:{follower_id}', 86400)  # 24h TTL
    
    pipeline.execute()
```

**Celebrity User Handling (> 1M followers):**

```python
def handle_celebrity_post(post, author):
    """
    Store post in celebrity feed, no fanout
    """
    # Just save to celebrity's own timeline
    redis_client.zadd(
        f'celebrity:posts:{author.user_id}',
        {post.post_id: post.created_at.timestamp()}
    )
    
    # Mark in database that this user is celebrity
    db.update(
        'users',
        {'user_id': author.user_id},
        {'is_celebrity': True}
    )
```

**Feed Generation for Celebrity Followers:**

```python
def generate_feed_with_celebrities(user_id, limit=50):
    """
    Generate feed merging regular timeline and celebrity posts
    """
    # Step 1: Get user's regular timeline (from fanout)
    regular_timeline = redis_client.zrevrange(
        f'timeline:{user_id}',
        0, 200,  # Get top 200 posts
        withscores=True
    )
    
    # Step 2: Get celebrities this user follows
    celebrity_ids = graph_db.get_following_celebrities(user_id)
    
    # Step 3: Fetch recent posts from celebrities (parallel)
    celebrity_posts = []
    with ThreadPoolExecutor(max_workers=10) as executor:
        futures = [
            executor.submit(
                redis_client.zrevrange,
                f'celebrity:posts:{celeb_id}',
                0, 50,
                withscores=True
            )
            for celeb_id in celebrity_ids
        ]
        for future in as_completed(futures):
            celebrity_posts.extend(future.result())
    
    # Step 4: Merge and sort by timestamp
    all_posts = regular_timeline + celebrity_posts
    all_posts.sort(key=lambda x: x[1], reverse=True)  # Sort by score (timestamp)
    
    # Step 5: Fetch post metadata
    post_ids = [post[0] for post in all_posts[:limit]]
    posts_metadata = batch_fetch_posts(post_ids)
    
    # Step 6: Apply ranking algorithm
    ranked_posts = rank_posts(posts_metadata, user_id)
    
    return ranked_posts[:limit]
```

**Optimization: Partial Fanout for Influencers (10k - 1M followers):**

```python
def fanout_influencer(post, author):
    """
    Push to active followers only
    """
    # Get active followers (logged in last 7 days)
    active_followers = db.query("""
        SELECT follower_id
        FROM follows
        WHERE following_id = :author_id
        AND follower_id IN (
            SELECT user_id
            FROM users
            WHERE last_login_at > NOW() - INTERVAL '7 days'
        )
        LIMIT 100000
    """, author_id=author.user_id)
    
    # Fanout to active followers only
    pipeline = redis_client.pipeline()
    for follower_id in active_followers:
        pipeline.zadd(
            f'timeline:{follower_id}',
            {post.post_id: post.created_at.timestamp()}
        )
    pipeline.execute()
```

#### Performance Metrics

**Regular Users (<10k followers):**

- Fanout time: <100ms (10k writes via pipeline)
- Feed load time: <50ms (Redis read + metadata fetch)

**Influencers (10k-1M followers):**

- Fanout time: <500ms (100k active followers)
- Feed load time: <100ms (merge with celebrity content)

**Celebrities (>1M followers):**

- Fanout time: 0ms (no fanout)
- Feed load time: <200ms (merge from multiple celebrities)

---

## Section 8: Real-time Features (WebSocket & Notifications)

### What You'll Learn

By the end of this section, you'll be able to:

- Design WebSocket architectures for real-time communication at scale
- Implement connection management and message routing across distributed servers
- Design notification systems that deliver messages via WebSocket, push notifications, and email
- Handle presence indicators (online/offline status) and typing indicators
- Scale real-time systems to millions of concurrent connections

### Why This Matters

Real-time features are what make social media platforms feel alive—users see likes, comments, and messages instantly. Real-world example: WhatsApp handles 100 billion messages per day with real-time delivery—if messages are delayed, users switch platforms! In interviews, demonstrating you understand WebSocket scaling and notification delivery shows you can build engaging, responsive systems.

### 🟢 For Beginners: The Fundamentals

#### What are Real-time Features?

Think of real-time features like a walkie-talkie conversation:

- **Traditional web** = Sending letters (request → wait → response)
- **Real-time web** = Walkie-talkie (instant back-and-forth)
- **WebSocket** = The walkie-talkie connection (stays open)
- **Notifications** = Someone calling your walkie-talkie (push alerts)

**The Challenge:**

When someone likes your post, how do you see it instantly without refreshing the page? That's what real-time features solve!

#### What is WebSocket?

**Traditional HTTP (Request-Response):**

```text
You: "Show me new messages" (HTTP request)
    ↓
Server: "Here are messages" (HTTP response)
    ↓
Connection closes
    ↓
You: "Show me new messages again" (new HTTP request)
    ↓
Server: "Here are messages" (HTTP response)
```

**WebSocket (Persistent Connection):**

```text
You: "Connect me" (WebSocket handshake)
    ↓
Connection stays open
    ↓
Server: "New message!" (pushes instantly)
Server: "Someone liked your post!" (pushes instantly)
Server: "New comment!" (pushes instantly)
    ↓
Connection stays open (no need to ask again!)
```

💡 **Pro Tip:** WebSocket is like keeping a phone call open instead of hanging up and calling back every time you want to talk!

#### What are Notifications?

Notifications tell users about events even when they're not using the app:

**Types of Notifications:**

1. **In-App (WebSocket):** See likes/comments instantly while using the app
2. **Push Notifications:** Get alerts on your phone even when app is closed
3. **Email:** Get summaries of activity (daily digest)

**Example Flow:**

```text
Someone likes your post
    ↓
System checks: Is user online?
    ├─ Yes → Send via WebSocket (instant!)
    └─ No → Send push notification (when they open app)
```

### 🟡 For Intermediate: Interview Patterns

#### The Real-time Features Framework

When designing real-time features in an interview, address these core challenges:

**Challenge 1: Connection Management**

```text
Problem: Millions of concurrent WebSocket connections
Solution: Distributed WebSocket servers
├─ Load balancer with session affinity
├─ Redis Pub/Sub for cross-server messaging
├─ Connection tracking per server
└─ Health checks and reconnection logic
```

**Challenge 2: Message Routing**

```text
Problem: User connected to Server A, message arrives at Server B
Solution: Pub/Sub messaging
├─ Redis Pub/Sub channels per user
├─ Servers subscribe to user channels
├─ Messages published to user channel
└─ All servers with user connection receive message
```

**Challenge 3: Notification Delivery**

```text
Problem: Deliver notifications via multiple channels
Solution: Multi-channel notification service
├─ WebSocket: For online users (instant)
├─ Push notifications: For offline users (FCM/APNS)
├─ Email: For important events (daily digest)
└─ In-app: For all users (notification center)
```

#### Interview Script: Real-time Features

**When asked "How would you implement real-time notifications?":**

**You say:** "I'd design a multi-channel notification system:

1. **WebSocket for Online Users:**
   - Distributed WebSocket servers (10K connections per server)
   - Redis Pub/Sub for cross-server messaging
   - User channels: `user:{user_id}:updates`
   - When event occurs, publish to user's channel
   - All servers with user connection receive and forward

2. **Push Notifications for Offline Users:**
   - Check if user is online (WebSocket connection exists)
   - If offline, send via FCM (Android) or APNS (iOS)
   - Store notification in database for in-app notification center

3. **Connection Management:**
   - Session affinity at load balancer (sticky sessions)
   - Heartbeat/ping every 30 seconds to detect disconnections
   - Automatic reconnection with exponential backoff"

**Why this design:**
- Scalable (distributed servers, Pub/Sub)
- Reliable (multiple delivery channels)
- Efficient (only push to online users via WebSocket)

⚠️ **Common Mistake:** Trying to store all connections in a single server's memory. This doesn't scale! Always use distributed servers with Pub/Sub.

#### Making Decisions Explicit

```text
"Based on our discussion, I'm going to assume:

✅ WebSocket Architecture: Distributed servers
   → 10K connections per server
   → Redis Pub/Sub for cross-server messaging
   → Session affinity at load balancer

✅ Notification Channels: Multi-channel
   → WebSocket: Online users (instant)
   → Push: Offline users (FCM/APNS)
   → Email: Daily digest for important events

✅ Connection Management: Heartbeat + Reconnection
   → Ping every 30 seconds
   → Reconnect with exponential backoff
   → Clean up dead connections

✅ Scale: 10M concurrent connections
   → 1,000 WebSocket servers
   → 10K connections per server
   → Redis cluster for Pub/Sub

Are these assumptions reasonable?"
```

### 🔴 For Advanced: Production Considerations

#### Real-time Architecture and Business Impact

When you're making real-time feature decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: WebSocket vs Server-Sent Events (SSE)**

```text
Scenario: Real-time feed updates

Option A: WebSocket (bidirectional)
├─ Latency: <10ms
├─ Complexity: High (connection management, reconnection)
├─ Cost: Higher (persistent connections)
├─ Use Case: Chat, real-time collaboration, gaming
└─ Business Impact: Best UX, but higher infrastructure cost

Option B: Server-Sent Events (SSE, unidirectional)
├─ Latency: <50ms
├─ Complexity: Low (simpler than WebSocket)
├─ Cost: Lower (HTTP-based)
├─ Use Case: Feed updates, notifications, live scores
└─ Business Impact: Good UX, lower cost, simpler to maintain

💡 Real-world: Twitter uses SSE for feed updates, WebSocket for DMs.
```

**Trade-off 2: Push Notification Frequency**

```text
Scenario: Notification delivery strategy

Option A: Send all notifications (no batching)
├─ Freshness: Instant
├─ User experience: Notification fatigue (too many alerts)
├─ Cost: Higher (more API calls to FCM/APNS)
└─ Use Case: Critical notifications only

Option B: Batch notifications (group similar events)
├─ Freshness: Slight delay (5-10 seconds)
├─ User experience: Less spam, more readable
├─ Cost: Lower (fewer API calls)
└─ Use Case: Social media (likes, comments)

💡 Real-world: Instagram batches "X and Y liked your post" instead of sending 100 separate notifications.
```

**Trade-off 3: Connection Persistence vs Cost**

```text
Scenario: WebSocket connection management

Option A: Keep all connections open
├─ Latency: Instant (always connected)
├─ Cost: High (memory, CPU per connection)
├─ Scalability: Limited by server resources
└─ Use Case: High-value users, premium tier

Option B: Lazy connection (connect on-demand)
├─ Latency: 100-200ms (connection overhead)
├─ Cost: Low (connections only when needed)
├─ Scalability: Better (fewer idle connections)
└─ Use Case: Casual users, free tier

💡 Real-world: Most platforms use hybrid—keep connections for active users, lazy for inactive.
```

#### Advanced Real-time Patterns

**Handling Edge Cases:**

```text
Edge Case 1: Server Failure

Problem: WebSocket server crashes, users lose connection
├─ Detection: Health checks, connection monitoring
├─ Recovery: Automatic reconnection, failover to backup server
└─ Implementation: Load balancer health checks, client reconnection logic

Edge Case 2: Message Ordering

Problem: Messages arrive out of order across servers
├─ Detection: Sequence numbers, timestamps
├─ Recovery: Client-side ordering, server-side sequencing
└─ Implementation: Include sequence number in messages, sort on client

Edge Case 3: Notification Duplication

Problem: User receives same notification multiple times
├─ Detection: Deduplication IDs
├─ Recovery: Idempotent notification delivery
└─ Implementation: Include event_id in notifications, check before sending
```

**Enterprise Requirements:**

When building for enterprise customers, requirements expand:

- **SLA guarantees:** 99.9% notification delivery rate, <100ms WebSocket latency
- **Compliance:** GDPR (right to opt-out), HIPAA (encrypted notifications)
- **Audit logs:** Track all real-time events and deliveries
- **Rate limiting:** Prevent notification spam, configurable per user
- **Multi-tenant:** Isolated WebSocket servers per customer

#### Architecture: WebSocket Connection Management

```text
┌─────────────────────────────────────────────────────────┐
│           WebSocket Server Architecture                  │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  Client Connections                                       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│  │ User A  │  │ User B  │  │ User C  │                 │
│  │ Mobile  │  │  Web    │  │ Mobile  │                 │
│  └────┬────┘  └────┬────┘  └────┬────┘                 │
│       │            │            │                        │
│       ▼            ▼            ▼                        │
│  ┌───────────────────────────────────┐                  │
│  │    Load Balancer (L4/L7)          │                  │
│  │    - Session affinity             │                  │
│  │    - Health checks                │                  │
│  └───────────┬───────────────────────┘                  │
│              │                                           │
│       ┌──────┼──────┐                                   │
│       ▼      ▼      ▼                                   │
│  ┌────────┐ ┌────────┐ ┌────────┐                      │
│  │  WS    │ │  WS    │ │  WS    │                      │
│  │Server 1│ │Server 2│ │Server 3│                      │
│  │        │ │        │ │        │                      │
│  │10k conn│ │10k conn│ │10k conn│                      │
│  └────┬───┘ └────┬───┘ └────┬───┘                      │
│       │          │          │                           │
│       └──────────┼──────────┘                           │
│                  │                                       │
│                  ▼                                       │
│       ┌─────────────────────┐                           │
│       │  Redis Pub/Sub      │                           │
│       │  - User channels    │                           │
│       │  - Broadcast events │                           │
│       └─────────────────────┘                           │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

**Implementation:**

```python
import asyncio
import websockets
import redis.asyncio as redis
import json

class WebSocketServer:
    def __init__(self):
        self.connections = {}  # user_id -> set of websocket connections
        self.redis = redis.Redis(host='redis.internal', decode_responses=True)
        self.pubsub = None
    
    async def start(self):
        """Start WebSocket server"""
        # Start Redis Pub/Sub listener
        self.pubsub = self.redis.pubsub()
        asyncio.create_task(self.listen_to_redis())
        
        # Start WebSocket server
        async with websockets.serve(self.handle_connection, "0.0.0.0", 8080):
            await asyncio.Future()  # Run forever
    
    async def handle_connection(self, websocket, path):
        """Handle new WebSocket connection"""
        user_id = None
        
        try:
            # Authenticate
            auth_message = await websocket.recv()
            user_id = await self.authenticate(auth_message)
            
            if not user_id:
                await websocket.close(1008, "Authentication failed")
                return
            
            # Register connection
            if user_id not in self.connections:
                self.connections[user_id] = set()
            self.connections[user_id].add(websocket)
            
            # Subscribe to user's channel
            await self.pubsub.subscribe(f'user:{user_id}:updates')
            
            logger.info(f'User {user_id} connected, total connections: {len(self.connections[user_id])}')
            
            # Send initial state
            await self.send_initial_state(websocket, user_id)
            
            # Handle incoming messages
            async for message in websocket:
                await self.handle_message(user_id, message)
        
        except websockets.exceptions.ConnectionClosed:
            logger.info(f'User {user_id} disconnected')
        
        finally:
            # Cleanup
            if user_id and user_id in self.connections:
                self.connections[user_id].discard(websocket)
                if not self.connections[user_id]:
                    del self.connections[user_id]
                    await self.pubsub.unsubscribe(f'user:{user_id}:updates')
    
    async def listen_to_redis(self):
        """Listen to Redis Pub/Sub and broadcast to WebSocket clients"""
        async for message in self.pubsub.listen():
            if message['type'] == 'message':
                channel = message['channel']
                data = json.loads(message['data'])
                
                # Extract user_id from channel name
                user_id = channel.split(':')[1]
                
                # Send to all connections for this user
                await self.broadcast_to_user(user_id, data)
    
    async def broadcast_to_user(self, user_id, data):
        """Send message to all connections for a user"""
        if user_id not in self.connections:
            return
        
        # Send to all connections concurrently
        tasks = [
            ws.send(json.dumps(data))
            for ws in self.connections[user_id]
        ]
        
        await asyncio.gather(*tasks, return_exceptions=True)
    
    async def send_initial_state(self, websocket, user_id):
        """Send initial state when user connects"""
        # Fetch unread notifications count
        unread_count = await self.get_unread_notifications(user_id)
        
        await websocket.send(json.dumps({
            'type': 'initial_state',
            'unread_notifications': unread_count,
            'timestamp': time.time()
        }))
    
    async def handle_message(self, user_id, message):
        """Handle incoming WebSocket message"""
        try:
            data = json.loads(message)
            msg_type = data.get('type')
            
            if msg_type == 'ping':
                # Respond to keep-alive ping
                await self.send_to_user(user_id, {'type': 'pong'})
            
            elif msg_type == 'mark_notification_read':
                # Mark notification as read
                notification_id = data['notification_id']
                await self.mark_notification_read(user_id, notification_id)
        
        except json.JSONDecodeError:
            logger.error(f'Invalid JSON from user {user_id}')

# Integration with application
class NotificationService:
    def __init__(self):
        self.redis = redis.Redis()
    
    async def notify_user(self, user_id, event_type, data):
        """Publish notification to user's channel"""
        await self.redis.publish(
            f'user:{user_id}:updates',
            json.dumps({
                'type': event_type,
                'data': data,
                'timestamp': time.time()
            })
        )

# Usage
notification_service = NotificationService()

@event_handler('engagement.like')
async def on_post_liked(post_id, liker_id):
    post = get_post(post_id)
    author_id = post.author_id
    
    # Notify post author
    await notification_service.notify_user(
        author_id,
        'new_like',
        {
            'post_id': post_id,
            'liker_id': liker_id,
            'liker_username': get_user(liker_id).username
        }
    )
```

### Connection Scaling & Limits

**Per-Server Limits:**

```text
OS Limits:
- File descriptors: ulimit -n 1000000
- Network buffers: sysctl net.core.rmem_max=134217728

Application Limits:
- Connections per server: 10,000 (comfortable)
- Maximum: 50,000 (with optimization)
- Memory per connection: ~10 KB
- Total memory for 50k connections: ~500 MB

Scaling:
- 500M DAU, assume 20% concurrent: 100M concurrent users
- Servers needed: 100M / 10k = 10,000 WebSocket servers
- With redundancy (2x): 20,000 servers
```

**Connection Pooling:**

```python
class WebSocketConnectionPool:
    def __init__(self, max_connections=10000):
        self.max_connections = max_connections
        self.current_connections = 0
        self.waiting_queue = asyncio.Queue()
    
    async def acquire(self):
        """Acquire connection slot"""
        if self.current_connections < self.max_connections:
            self.current_connections += 1
            return True
        else:
            # Wait for slot to become available
            await self.waiting_queue.get()
            return True
    
    def release(self):
        """Release connection slot"""
        self.current_connections -= 1
        
        # Wake up waiting connection
        if not self.waiting_queue.empty():
            self.waiting_queue.put_nowait(True)
```

---

### Notification Service Architecture

#### Push Notification Architecture

**Components:**

```text
┌─────────────────────────────────────────────────────────┐
│         Notification Service Architecture                │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  Event Sources                                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │   Like   │  │ Comment  │  │  Follow  │              │
│  │  Event   │  │  Event   │  │  Event   │              │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘              │
│       │             │             │                      │
│       └─────────────┼─────────────┘                      │
│                     │                                     │
│                     ▼                                     │
│          ┌──────────────────────┐                        │
│          │    Kafka Topic       │                        │
│          │  'notifications'     │                        │
│          └──────────┬───────────┘                        │
│                     │                                     │
│                     ▼                                     │
│          ┌──────────────────────┐                        │
│          │  Notification        │                        │
│          │  Processor           │                        │
│          │  (Consumer Group)    │                        │
│          └──────────┬───────────┘                        │
│                     │                                     │
│          ┌──────────┴───────────┐                        │
│          │                      │                        │
│          ▼                      ▼                        │
│  ┌──────────────┐     ┌──────────────┐                  │
│  │  WebSocket   │     │ Push Service │                  │
│  │  (Real-time) │     │  (Offline)   │                  │
│  └──────────────┘     └──────┬───────┘                  │
│                               │                           │
│                    ┌──────────┼──────────┐               │
│                    ▼          ▼          ▼               │
│              ┌──────┐    ┌──────┐   ┌──────┐            │
│              │ FCM  │    │ APNS │   │Email │            │
│              │(Android)  │(iOS) │   │      │            │
│              └──────┘    └──────┘   └──────┘            │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

**Implementation:**

```python
class NotificationProcessor:
    def __init__(self):
        self.kafka_consumer = KafkaConsumer(
            'notifications',
            group_id='notification-processors',
            bootstrap_servers=['kafka:9092']
        )
        
        self.fcm_client = FCMClient()  # Firebase Cloud Messaging
        self.apns_client = APNSClient()  # Apple Push Notification Service
        self.email_client = EmailClient()
    
    async def process_notifications(self):
        """Main processing loop"""
        for message in self.kafka_consumer:
            event = json.loads(message.value)
            
            # Determine recipients
            recipients = await self.get_recipients(event)
            
            # Filter based on user preferences
            recipients = self.filter_by_preferences(recipients, event['type'])
            
            # Send notifications
            await self.send_notifications(recipients, event)
    
    async def get_recipients(self, event):
        """Determine who should receive this notification"""
        if event['type'] == 'new_like':
            # Notify post author
            post = get_post(event['post_id'])
            return [post.author_id]
        
        elif event['type'] == 'new_comment':
            # Notify post author + parent comment author
            post = get_post(event['post_id'])
            recipients = [post.author_id]
            
            if event.get('parent_comment_id'):
                parent_comment = get_comment(event['parent_comment_id'])
                recipients.append(parent_comment.author_id)
            
            return recipients
        
        elif event['type'] == 'new_follower':
            # Notify user who was followed
            return [event['followed_user_id']]
    
    def filter_by_preferences(self, recipients, event_type):
        """Filter based on user notification preferences"""
        filtered = []
        
        for user_id in recipients:
            prefs = get_user_notification_preferences(user_id)
            
            if prefs.get(event_type, {}).get('enabled', True):
                filtered.append(user_id)
        
        return filtered
    
    async def send_notifications(self, recipients, event):
        """Send notifications via appropriate channels"""
        for user_id in recipients:
            user = get_user(user_id)
            
            # Check if user is online (WebSocket)
            if is_user_online(user_id):
                await self.send_websocket_notification(user_id, event)
            else:
                # Send push notification
                await self.send_push_notification(user, event)
            
            # Store in database for notification center
            await self.store_notification(user_id, event)
    
    async def send_push_notification(self, user, event):
        """Send push notification to mobile device"""
        devices = get_user_devices(user.user_id)
        
        # Build notification payload
        notification = self.build_notification_payload(event)
        
        for device in devices:
            if device.platform == 'ios':
                await self.send_apns(device.token, notification)
            elif device.platform == 'android':
                await self.send_fcm(device.token, notification)
    
    async def send_fcm(self, device_token, notification):
        """Send via Firebase Cloud Messaging"""
        message = {
            'token': device_token,
            'notification': {
                'title': notification['title'],
                'body': notification['body']
            },
            'data': notification['data'],
            'android': {
                'priority': 'high',
                'notification': {
                    'sound': 'default',
                    'click_action': 'OPEN_APP'
                }
            }
        }
        
        response = await self.fcm_client.send(message)
        
        if response.get('error'):
            logger.error(f'FCM error: {response["error"]}')
            
            # Handle token expiration
            if response['error']['code'] == 'INVALID_TOKEN':
                await self.remove_device_token(device_token)
    
    def build_notification_payload(self, event):
        """Build notification message"""
        if event['type'] == 'new_like':
            liker = get_user(event['liker_id'])
            return {
                'title': 'New Like',
                'body': f'{liker.username} liked your post',
                'data': {
                    'type': 'new_like',
                    'post_id': event['post_id'],
                    'liker_id': event['liker_id']
                }
            }
        
        elif event['type'] == 'new_comment':
            commenter = get_user(event['commenter_id'])
            return {
                'title': 'New Comment',
                'body': f'{commenter.username} commented on your post: {event["comment_text"][:50]}...',
                'data': {
                    'type': 'new_comment',
                    'post_id': event['post_id'],
                    'comment_id': event['comment_id']
                }
            }
```

### Notification Batching & Throttling

**Problem:** User gets hundreds of likes in a minute → spam notifications

**Solution:**

```python
class NotificationBatcher:
    def __init__(self):
        self.batch_window = 60  # 1 minute
        self.pending_notifications = {}  # user_id -> list of events
        self.batch_timers = {}  # user_id -> timer
    
    def add_notification(self, user_id, event):
        """Add notification to batch"""
        if user_id not in self.pending_notifications:
            self.pending_notifications[user_id] = []
        
        self.pending_notifications[user_id].append(event)
        
        # Start timer if not already running
        if user_id not in self.batch_timers:
            self.batch_timers[user_id] = threading.Timer(
                self.batch_window,
                self.flush_batch,
                args=[user_id]
            )
            self.batch_timers[user_id].start()
    
    def flush_batch(self, user_id):
        """Send batched notifications"""
        events = self.pending_notifications.pop(user_id, [])
        del self.batch_timers[user_id]
        
        if not events:
            return
        
        # Group by type
        grouped = {}
        for event in events:
            event_type = event['type']
            if event_type not in grouped:
                grouped[event_type] = []
            grouped[event_type].append(event)
        
        # Send aggregated notification
        for event_type, events_list in grouped.items():
            self.send_aggregated_notification(user_id, event_type, events_list)
    
    def send_aggregated_notification(self, user_id, event_type, events):
        """Send single notification for multiple events"""
        if event_type == 'new_like':
            count = len(events)
            
            if count == 1:
                # Single like
                liker = get_user(events[0]['liker_id'])
                title = f'{liker.username} liked your post'
            else:
                # Multiple likes
                first_liker = get_user(events[0]['liker_id'])
                if count == 2:
                    second_liker = get_user(events[1]['liker_id'])
                    title = f'{first_liker.username} and {second_liker.username} liked your post'
                else:
                    title = f'{first_liker.username} and {count - 1} others liked your post'
            
            send_notification(user_id, title, event_type, events)
```

### Real-World Example: How WhatsApp Scales Real-time Messaging

Let's look at how WhatsApp evolved their real-time messaging system:

**2009-2012 - Basic Messaging:**
```text
Architecture: Single server, basic WebSocket
├─ Scale: 1M users, 1M messages/day
├─ Decision: Simple WebSocket server
└─ Result: Works for small scale, but doesn't scale

Challenge: Server overload with 10K+ concurrent connections
```

**2012-2016 - Distributed Architecture:**
```text
Architecture: Distributed WebSocket servers
├─ Added: Redis Pub/Sub for cross-server messaging
├─ Scale: 100M users, 1B messages/day
├─ Decision: Load balancing, session affinity
└─ Result: Handles millions of concurrent connections

Challenge: Message ordering across servers
```

**2016-Present - Optimized Pipeline:**
```text
Architecture: Erlang-based, optimized for concurrency
├─ Added: Message queuing, delivery receipts, read receipts
├─ Scale: 2B+ users, 100B messages/day
├─ Decision: Erlang/OTP for massive concurrency
└─ Result: <100ms message delivery, 99.9% reliability

Key Innovation: Erlang's lightweight processes handle millions of connections efficiently
```

📊 **By The Numbers:**
- 2009: 1M users, 1M messages/day, single server
- 2012: 100M users, 1B messages/day, distributed servers
- 2024: 2B+ users, 100B messages/day, <100ms delivery, 99.9% reliability

**Key Lesson:** Real-time systems evolve from simple (single server) to complex (distributed with Pub/Sub). The key is choosing the right technology stack (Erlang for WhatsApp, Node.js for many others) and implementing proper connection management and message routing.

### 🎯 Interview Questions: Real-time Features

#### Question 1: How would you design a real-time notification system?

**What the interviewer wants to know:**
- Do you understand WebSocket architecture?
- Can you design distributed systems?
- Do you think about multiple delivery channels?

**Answer Framework:**

```text
1. WebSocket Architecture
   ├─ Distributed servers (10K connections per server)
   ├─ Load balancer with session affinity
   ├─ Redis Pub/Sub for cross-server messaging
   └─ User channels: user:{user_id}:updates

2. Connection Management
   ├─ Heartbeat/ping every 30 seconds
   ├─ Automatic reconnection with exponential backoff
   ├─ Clean up dead connections
   └─ Health checks for servers

3. Notification Delivery
   ├─ Check if user online (WebSocket connection exists)
   ├─ Online: Send via WebSocket (instant)
   ├─ Offline: Send push notification (FCM/APNS)
   └─ Store in database for in-app notification center

4. Scaling
   ├─ 10M concurrent connections
   ├─ 1,000 WebSocket servers (10K connections each)
   ├─ Redis cluster for Pub/Sub
   └─ Auto-scaling based on connection count

Example Flow:
├─ Event occurs (like, comment, follow)
├─ Publish to user:{user_id}:updates channel
├─ All servers with user connection receive message
├─ Forward to user via WebSocket
└─ If offline, send push notification
```

**Follow-up: How do you handle a user connected to multiple devices?**

```text
Solution: Multi-device support
├─ Track all device connections per user
├─ Publish to user channel (all devices receive)
├─ Include device_id in message
├─ Client filters: Only show on relevant device
└─ Sync: Update notification center across devices
```

#### Question 2: How would you scale WebSocket connections to 10 million concurrent users?

**What the interviewer wants to know:**
- Can you calculate server requirements?
- Do you understand connection limits?
- Can you design for horizontal scaling?

**Answer Framework:**

```text
1. Server Capacity
   ├─ Each server: 10K connections (memory/CPU limits)
   ├─ Servers needed: 10M / 10K = 1,000 servers
   ├─ With redundancy (2x): 2,000 servers
   └─ Cost: ~$200K/month (on-demand) or $60K/month (spot)

2. Load Balancing
   ├─ Session affinity (sticky sessions)
   ├─ Health checks every 10 seconds
   ├─ Failover to backup servers
   └─ Connection draining during deployments

3. Message Routing
   ├─ Redis Pub/Sub channels per user
   ├─ Servers subscribe to user channels
   ├─ Messages published to user channel
   └─ All servers with user connection receive

4. Optimization
   ├─ Connection pooling: Reuse connections
   ├─ Message batching: Group multiple events
   ├─ Compression: Compress large messages
   └─ CDN: Cache static notification content

Scaling Example:
├─ 10M concurrent connections
├─ 1,000 WebSocket servers
├─ 10K connections per server
├─ Redis cluster: 10 nodes for Pub/Sub
└─ Load balancer: 10 instances for high availability
```

#### Question 3: How do you ensure notification delivery reliability?

**What the interviewer wants to know:**
- Do you understand failure modes?
- Can you design retry mechanisms?
- Do you think about delivery guarantees?

**Answer Framework:**

```text
1. Delivery Guarantees
   ├─ At-least-once: Retry until acknowledged
   ├─ Deduplication: Event IDs to prevent duplicates
   ├─ Ordering: Sequence numbers for message order
   └─ Acknowledgment: Client confirms receipt

2. Failure Handling
   ├─ WebSocket failure: Fallback to push notification
   ├─ Push failure: Retry with exponential backoff
   ├─ Max retries: 3 attempts
   └─ Dead letter queue: For permanently failed deliveries

3. Monitoring
   ├─ Metrics: Delivery rate, latency, failure rate
   ├─ Alerts: Delivery rate < 95%, latency > 1s
   ├─ Dashboards: Real-time delivery status
   └─ Logging: Track all delivery attempts

4. Reliability Strategies
   ├─ Redundancy: Multiple notification servers
   ├─ Queue persistence: Store notifications in queue
   ├─ Idempotency: Prevent duplicate notifications
   └─ Circuit breaker: Stop sending if failure rate high
```

### 🤔 Think About It

1. **For Beginners:** Why do we need WebSocket instead of just checking for new messages every few seconds? What's the problem with polling? (Hint: Think about server load and battery usage)

2. **For Intermediate:** If you had to choose between WebSocket and Server-Sent Events (SSE) for real-time feed updates, which would you choose? Consider complexity, cost, and use case.

3. **For Advanced:** How would your real-time system change if you were building for:
   - A financial trading platform where millisecond latency matters?
   - A collaborative document editor where multiple users edit simultaneously?
   - A gaming platform where real-time synchronization is critical?

### ✅ Key Takeaways

- **WebSocket enables real-time communication**: Persistent connections allow instant bidirectional messaging without polling
- **Distributed architecture is essential**: Use multiple WebSocket servers with Redis Pub/Sub for cross-server messaging
- **Multi-channel delivery improves reliability**: Combine WebSocket (online), push notifications (offline), and email (digest)
- **Connection management is critical**: Implement heartbeats, reconnection logic, and health checks for reliable connections
- **Session affinity enables scaling**: Use load balancer sticky sessions to route users to same server
- **Batching reduces costs**: Group similar notifications to reduce API calls and improve user experience

### 🎯 Practice Exercise

**Scenario:** Design a real-time chat system (like WhatsApp) with these requirements:
- Support 100 million concurrent users
- Messages must be delivered in <100ms
- Support group chats (up to 256 members)
- Handle offline users (deliver when they come online)
- Ensure message ordering and delivery guarantees

**Your Task:**
1. Design the WebSocket architecture (servers, load balancing, Pub/Sub)
2. Design the message routing system (how messages reach recipients)
3. Design offline message delivery (storing and delivering when users come online)
4. Design group chat message distribution (how to send to 256 members efficiently)

**Bonus Challenge:** How would you handle a user sending a message to a group chat where some members are online and others are offline? How do you ensure all members receive the message?

---

## Section 14: Making Design Decisions

### What You'll Learn

By the end of this section, you'll be able to:

- Analyze trade-offs between different technology choices (SQL vs NoSQL, caching strategies, etc.)
- Make informed design decisions based on requirements, scale, and constraints
- Justify your choices with clear reasoning and real-world examples
- Understand when to use which technology for specific use cases
- Evaluate alternatives and explain why you chose one approach over another

### Why This Matters

Design decisions are the foundation of any system—choosing the wrong technology can lead to scalability issues, high costs, or poor performance. Real-world example: Twitter initially used MySQL for everything, but had to migrate to distributed systems (Cassandra, Redis) as they scaled—understanding trade-offs upfront saves years of technical debt! In interviews, showing you can analyze trade-offs demonstrates critical thinking and production experience.

### 🟢 For Beginners: The Fundamentals

#### What are Design Decisions?

Think of design decisions like choosing a vehicle for a trip:

- **Sports car** = Fast but expensive (like using expensive databases)
- **SUV** = Versatile but slower (like using general-purpose databases)
- **Motorcycle** = Fast and cheap but limited (like using specialized databases)

**The Challenge:**

Every technology choice has trade-offs. There's no "perfect" solution—only the best solution for YOUR specific needs!

#### Understanding Trade-offs

**SQL vs NoSQL:**

```text
SQL (PostgreSQL, MySQL):
✅ Structured data
✅ Complex queries (JOINs)
✅ Strong consistency
❌ Harder to scale horizontally
❌ Slower writes at scale

NoSQL (Cassandra, MongoDB):
✅ Easy horizontal scaling
✅ Fast writes
✅ Flexible schema
❌ Limited query flexibility
❌ Eventual consistency
```

💡 **Pro Tip:** Use SQL when you need complex queries and strong consistency. Use NoSQL when you need to scale writes and can accept eventual consistency!

#### Key Decision Factors

When making design decisions, consider:

1. **Scale:** How much data? How many users?
2. **Performance:** What latency requirements?
3. **Consistency:** Do you need strong consistency or eventual consistency?
4. **Cost:** What's your budget?
5. **Complexity:** How hard is it to operate?

### 🟡 For Intermediate: Interview Patterns

#### The Design Decision Framework

When discussing trade-offs in an interview, use this framework:

**Step 1: Identify Requirements**

```text
Ask yourself:
├─ What are the performance requirements? (latency, throughput)
├─ What are the consistency requirements? (strong vs eventual)
├─ What is the scale? (data size, users, QPS)
└─ What are the constraints? (budget, team expertise)
```

**Step 2: Evaluate Options**

```text
For each option, consider:
├─ Pros: What are the advantages?
├─ Cons: What are the disadvantages?
├─ Use cases: When is this appropriate?
└─ Real-world examples: Who uses this and why?
```

**Step 3: Make Decision**

```text
Justify your choice:
├─ Explain why this option fits requirements
├─ Acknowledge trade-offs you're accepting
├─ Mention alternatives you considered
└─ Reference real-world examples
```

#### Interview Script: Design Decisions

**When asked "Why did you choose X over Y?":**

**You say:** "I chose X because:

1. **Requirements Analysis:**
   - We need [specific requirement]
   - Scale: [specific numbers]
   - Latency: [specific target]

2. **Option Comparison:**
   - Option A (X): [Pros] but [Cons]
   - Option B (Y): [Pros] but [Cons]
   - Option C (Z): [Pros] but [Cons]

3. **Decision:**
   - Chose X because [reasoning]
   - Trade-off: Accepting [trade-off] for [benefit]
   - Real-world: [Company] uses X for similar reasons"

⚠️ **Common Mistake:** Saying "X is better" without context. Always explain WHY based on requirements!

### 🔴 For Advanced: Production Considerations

#### Design Decision Analysis and Business Impact

When you're making design decisions, every choice has business implications. Let's think like a Principal Engineer:

**Decision Framework:**

```text
For each major decision, evaluate:
├─ Technical: Performance, scalability, reliability
├─ Business: Cost, time-to-market, maintenance
├─ Operational: Complexity, team expertise, tooling
└─ Risk: What happens if this fails?
```

### Decision 1: SQL vs NoSQL for Post Storage

**Choice:** Cassandra (NoSQL)

**Pros:**

- Horizontal scalability (easily add nodes)
- High write throughput (200M posts/day = 2,315 writes/sec)
- Natural partitioning by user_id
- Time-series data model fits use case
- Linear scalability for reads and writes

**Cons:**

- Limited query flexibility (no JOINs)
- Eventual consistency (acceptable for social media)
- More complex data modeling
- Requires careful partition key selection

**Justification:** The write-heavy workload (200M posts/day) and need for horizontal scalability make Cassandra ideal. Posts are naturally partitioned by user, and we rarely need complex queries across users.

---

### Decision 2: PostgreSQL for User Data

**Choice:** PostgreSQL (SQL)

**Pros:**

- ACID compliance for critical user data
- Rich querying capabilities
- Strong consistency for auth operations
- Mature tooling and ecosystem
- Good for <10TB datasets

**Cons:**

- Vertical scaling limitations
- Sharding complexity
- Slower writes compared to NoSQL

**Justification:** User data requires strong consistency (can't have duplicate usernames) and ACID guarantees. The dataset size (1.5TB) fits comfortably in PostgreSQL with read replicas.

---

### Decision 3: Graph Database for Social Graph

**Choice:** Neo4j/DGraph

**Pros:**

- Native graph traversal (efficient follower queries)
- Multi-hop queries (friends-of-friends)
- Relationship properties (followed_at, notification_enabled)
- Graph algorithms (influencer detection, community detection)

**Cons:**

- Specialized database (operational complexity)
- Scaling challenges for write-heavy workloads
- Higher cost compared to relational DB

**Justification:** Social relationships are inherently graph-structured. Neo4j excels at traversal queries like "get all followers" or "mutual friends", which are performance-critical for feed generation.

**Alternative Considered:** Store follows in PostgreSQL with proper indexes. Would work for simple queries but poor performance for multi-hop traversals.

---

### Decision 4: Redis for Timeline Cache

**Choice:** Redis Sorted Sets

**Pros:**

- In-memory speed (<1ms latency)
- Sorted sets perfect for timelines (score = timestamp)
- Built-in operations (ZADD, ZREVRANGE, ZCARD)
- Horizontal scaling (Redis Cluster)
- TTL support for automatic cleanup

**Cons:**

- Volatile data (cache miss requires regeneration)
- Memory cost (10TB for 100M active users)
- Complex cluster management

**Justification:** Feed load time requirement (<500ms) demands in-memory caching. Redis sorted sets are purpose-built for ranked timelines with O(log N) insert and O(1) retrieval.

---

### Decision 5: Kafka for Message Queue

**Choice:** Apache Kafka

**Pros:**

- High throughput (millions of messages/sec)
- Durability (replicated, persistent)
- Replay capability (reprocess failed jobs)
- Partitioning for parallelism
- Mature ecosystem

**Cons:**

- Operational complexity
- Over-engineering for simple queues
- Storage costs (7-day retention)

**Justification:** Handles 7,000 media processing jobs/sec at peak. Durability ensures no lost uploads. Replay capability allows reprocessing if bugs occur.

**Alternative Considered:** AWS SQS - simpler but less throughput and no replay capability.

---

### Decision 6: CDN for Media Delivery

**Choice:** CloudFront/Akamai

**Pros:**

- 95%+ cache hit ratio (massive bandwidth savings)
- <100ms latency globally (200+ edge locations)
- DDoS protection
- Automatic failover
- Pay-as-you-go pricing

**Cons:**

- Cache invalidation complexity
- Initial cache warming takes time
- Vendor lock-in

**Justification:** Serving 10B impressions/day directly from origin would require 16.5Pbps bandwidth. CDN reduces this by 95% and improves latency from 500ms to <100ms.

---

### Decision 7: Elasticsearch for Search

**Choice:** Elasticsearch

**Pros:**

- Full-text search with relevance scoring
- Fuzzy matching for typos
- Aggregations for trending hashtags
- Near real-time indexing
- Horizontal scaling

**Cons:**

- Resource-intensive (CPU, memory)
- Eventual consistency (search lags writes)
- Complex cluster management

**Justification:** Hashtag search and user search require full-text capabilities. Elasticsearch provides sub-100ms search with fuzzy matching and ranking.

**Alternative Considered:** PostgreSQL full-text search - sufficient for small scale but doesn't scale to billions of posts.

---

### Decision 8: Hybrid Fanout Strategy

**Choice:** Fan-out on write for regular users, fan-out on read for celebrities

**Pros:**

- Solves celebrity problem (no 100M writes per post)
- Fast feed load for most users (<50ms from cache)
- Scalable to any follower count

**Cons:**

- Complex implementation (two code paths)
- Slower feed load for celebrity followers (200ms vs 50ms)
- Threshold tuning required (10k followers)

**Justification:** Pure fan-out on write doesn't scale for celebrities (27+ hours to fanout to 100M followers). Pure fan-out on read is slow for everyone. Hybrid approach optimizes for the common case (regular users) while handling edge case (celebrities).

### Real-World Example: How Twitter Made Database Decisions

Let's look at how Twitter evolved their database architecture:

**2006-2010 - MySQL Everything:**
```text
Architecture: Single MySQL database
├─ Scale: 50M tweets/day
├─ Decision: MySQL for all data
└─ Result: Works initially, but hits scaling limits

Challenge: Database overload, frequent downtime
```

**2010-2015 - Distributed Databases:**
```text
Architecture: Multiple specialized databases
├─ Added: Cassandra for tweets, Redis for timelines
├─ Scale: 500M tweets/day
├─ Decision: Right database for right use case
└─ Result: Better performance, but operational complexity

Challenge: Managing multiple database systems
```

**2015-Present - Optimized Architecture:**
```text
Architecture: Multi-database with clear boundaries
├─ PostgreSQL: User data, auth (strong consistency)
├─ Cassandra: Tweets, posts (write-heavy, scalable)
├─ Redis: Timelines, cache (low latency)
├─ Neo4j: Social graph (graph queries)
└─ Elasticsearch: Search (full-text search)

Key Innovation: Each database optimized for specific access patterns
```

📊 **By The Numbers:**
- 2006: MySQL only, 50M tweets/day, frequent downtime
- 2010: Distributed databases, 500M tweets/day, better reliability
- 2024: Multi-database architecture, 500M+ tweets/day, 99.99% uptime

**Key Lesson:** Database decisions evolve from simple (one database) to complex (multiple specialized databases). The key is choosing the right tool for each job—use SQL for structured data requiring consistency, NoSQL for scalable writes, and specialized databases for specific use cases.

### 🎯 Interview Questions: Design Decisions

#### Question 1: Why did you choose Cassandra over PostgreSQL for storing posts?

**What the interviewer wants to know:**
- Do you understand the trade-offs?
- Can you justify your choices?
- Do you think about scale and requirements?

**Answer Framework:**

```text
1. Requirements Analysis
   ├─ Write-heavy: 200M posts/day (2,315 writes/sec)
   ├─ Read pattern: Mostly by user_id (partitioned access)
   ├─ Scale: Need horizontal scaling
   └─ Consistency: Eventual consistency acceptable

2. Option Comparison
   ├─ PostgreSQL:
   │  ├─ Pros: ACID, complex queries, strong consistency
   │  └─ Cons: Hard to scale horizontally, slower writes
   ├─ Cassandra:
   │  ├─ Pros: Horizontal scaling, fast writes, partitioned by user
   │  └─ Cons: Eventual consistency, limited query flexibility

3. Decision: Cassandra
   ├─ Reason: Write-heavy workload, need horizontal scaling
   ├─ Trade-off: Accepting eventual consistency for scalability
   ├─ Use case: Posts naturally partitioned by user_id
   └─ Real-world: Instagram, Twitter use Cassandra for posts

4. Alternative Considered
   └─ PostgreSQL with sharding: Too complex, still slower writes
```

**Follow-up: What if you needed to query posts across multiple users?**

```text
Solution: Use Elasticsearch for cross-user queries
├─ Cassandra: Store posts (partitioned by user)
├─ Elasticsearch: Index posts for search/explore
├─ Sync: Update Elasticsearch when posts created
└─ Query: Use Elasticsearch for complex queries, Cassandra for user-specific
```

#### Question 2: How do you decide between SQL and NoSQL?

**What the interviewer wants to know:**
- Can you evaluate trade-offs systematically?
- Do you understand when to use each?
- Can you provide clear decision criteria?

**Answer Framework:**

```text
1. Decision Criteria
   ├─ Data structure: Structured (SQL) vs Flexible (NoSQL)
   ├─ Query complexity: Complex JOINs (SQL) vs Simple queries (NoSQL)
   ├─ Consistency: Strong (SQL) vs Eventual (NoSQL)
   ├─ Scale: Vertical (SQL) vs Horizontal (NoSQL)
   └─ Write pattern: Moderate (SQL) vs High (NoSQL)

2. Use SQL When:
   ├─ Need ACID transactions
   ├─ Complex queries with JOINs
   ├─ Strong consistency required
   ├─ Dataset fits in single server (<10TB)
   └─ Examples: User accounts, financial data, auth

3. Use NoSQL When:
   ├─ Need horizontal scaling
   ├─ High write throughput
   ├─ Simple access patterns
   ├─ Eventual consistency acceptable
   └─ Examples: Posts, logs, time-series data

4. Hybrid Approach (Common)
   ├─ SQL: User data, auth (consistency critical)
   ├─ NoSQL: Posts, feeds (scalability critical)
   └─ Best of both worlds
```

#### Question 3: How do you evaluate caching strategies?

**What the interviewer wants to know:**
- Do you understand caching trade-offs?
- Can you design multi-layer caching?
- Do you think about cache invalidation?

**Answer Framework:**

```text
1. Caching Layers
   ├─ L1: In-memory cache (Redis) - fastest, smallest
   ├─ L2: Distributed cache (Memcached) - fast, larger
   ├─ L3: Application cache - medium speed, largest
   └─ L4: CDN cache - slowest, global distribution

2. Cache Strategy Decision
   ├─ Access pattern: Hot data (cache), cold data (don't cache)
   ├─ Update frequency: Rare updates (long TTL), frequent (short TTL)
   ├─ Size: Small data (memory cache), large data (CDN)
   └─ Cost: Memory cost vs compute cost

3. Cache Invalidation
   ├─ TTL-based: Automatic expiration
   ├─ Event-driven: Invalidate on updates
   ├─ Version-based: Include version in cache key
   └─ Manual: Admin-triggered invalidation

4. Trade-offs
   ├─ More layers: Better performance, higher complexity
   ├─ Longer TTL: Better hit rate, stale data risk
   └─ Larger cache: Better hit rate, higher cost
```

### 🤔 Think About It

1. **For Beginners:** Why might you use both SQL and NoSQL databases in the same system? What data would go in each? (Hint: Think about what needs to be consistent vs what needs to scale)

2. **For Intermediate:** If you had to choose between a simple architecture (one database) vs a complex architecture (multiple databases), which would you choose? Consider development speed, scalability, and operational complexity.

3. **For Advanced:** How would your database choices change if you were building for:
   - A financial platform where data consistency is critical?
   - A content platform where write throughput is the bottleneck?
   - A real-time analytics platform where query speed matters most?

### ✅ Key Takeaways

- **No single database fits all**: Use SQL for structured data requiring consistency, NoSQL for scalable writes, and specialized databases for specific use cases
- **Trade-offs are inevitable**: Every choice has pros and cons—understand what you're giving up
- **Requirements drive decisions**: Choose technology based on your specific needs (scale, consistency, query patterns), not trends
- **Hybrid approaches are common**: Most production systems use multiple databases, each optimized for specific access patterns
- **Real-world examples matter**: Reference how companies like Twitter, Instagram, and Facebook made similar decisions
- **Justify your choices**: Always explain WHY you chose an option based on requirements, not just WHAT you chose

### 🎯 Practice Exercise

**Scenario:** You're designing a social media platform and need to make database decisions. You have:
- User profiles (need strong consistency, complex queries)
- Posts (200M posts/day, mostly read by user_id)
- Social graph (follower relationships, need graph queries)
- Search (full-text search across posts and users)
- Real-time feeds (need <500ms latency)

**Your Task:**
1. Choose databases for each data type (justify each choice)
2. Explain the trade-offs you're accepting
3. Design how these databases interact
4. Consider what happens if one database fails

**Bonus Challenge:** How would your database choices change if you had a budget constraint that forced you to use fewer databases? What would you consolidate?

---

## Section 12: Protecting the System (Security)

### What You'll Learn

By the end of this section, you'll be able to:

- Design comprehensive security systems for social media platforms
- Implement authentication and authorization mechanisms at scale
- Protect against common attacks (DDoS, injection, data breaches)
- Design content moderation pipelines using ML and human review
- Ensure compliance with regulations (GDPR, CCPA, SOC 2)

### Why This Matters

Security breaches destroy user trust and can shut down companies. Real-world example: In 2018, Facebook's data breach exposed 50 million users' personal information, resulting in a $5 billion FTC fine and massive user trust loss. Instagram handles 100M photos daily—without proper security, malicious content, data breaches, and DDoS attacks could bring down the entire platform! In interviews, demonstrating you understand security shows you can build production-grade systems that protect users and comply with regulations.

### 🟢 For Beginners: The Fundamentals

#### What is Security?

Think of security like protecting a house:

```text
Physical Security (House):
├─ Locks on doors (authentication)
├─ Keys for different rooms (authorization)
├─ Security cameras (monitoring)
├─ Alarm system (intrusion detection)
└─ Insurance (backup/recovery)

Digital Security (Social Media Platform):
├─ Login systems (authentication)
├─ User permissions (authorization)
├─ Monitoring systems (security alerts)
├─ Firewalls (attack prevention)
└─ Backups (data recovery)
```

**The Challenge:**

Social media platforms handle billions of user interactions daily. Without proper security, attackers could:
- Steal user data (passwords, personal information)
- Post malicious content (spam, inappropriate material)
- Overwhelm servers (DDoS attacks)
- Access other users' accounts (unauthorized access)

#### The CIA Triad

Security has three core principles:

**C - Confidentiality**: Keep secrets secret

```text
✅ Only Alice can see Alice's posts
❌ Bob should NOT see Alice's private messages
```

**I - Integrity**: Data isn't tampered with

```text
✅ Post content can't be modified after publishing
❌ Hacker changes "Like count" from 100 to 1M
```

**A - Availability**: System remains accessible

```text
✅ Platform up 99.9% of time
❌ DDoS attack takes site offline
```

#### Authentication vs Authorization

**Authentication**: "Who are you?"

```text
Methods:
├─ Username and password
├─ Social login (Google, Facebook)
├─ Two-factor authentication (2FA)
└─ Biometric authentication (fingerprint, face)
```

**Authorization**: "What can you do?"

```text
Permissions:
├─ Read your own posts ✅
├─ Edit your own posts ✅
├─ Delete your own posts ✅
├─ Edit someone else's posts ❌
└─ Delete someone else's posts ❌
```

💡 **Pro Tip:** Authentication verifies identity, authorization controls access. Both are essential!

#### Common Security Threats

```text
1. Data Breaches:
   ├─ Hackers steal user passwords
   ├─ Personal information exposed
   └─ Solution: Encryption, secure storage

2. DDoS Attacks:
   ├─ Overwhelm servers with requests
   ├─ Make service unavailable
   └─ Solution: Rate limiting, CDN protection

3. Injection Attacks:
   ├─ SQL injection (malicious database queries)
   ├─ XSS (cross-site scripting)
   └─ Solution: Input validation, parameterized queries

4. Content Abuse:
   ├─ Spam posts
   ├─ Inappropriate content
   └─ Solution: Content moderation (ML + human review)
```

### 🟡 For Intermediate: Interview Patterns

#### The Security Interview Framework

When discussing security in interviews, follow this structure:

**Phase 1: Threat Assessment**

- "What are the main security threats to a social media platform?"
- "How do you protect user data?"
- "How do you prevent malicious content?"

**Phase 2: Security Architecture**

```text
Authentication & Authorization:
├─ JWT tokens (access + refresh)
├─ OAuth 2.0 for third-party login
├─ Role-based access control (RBAC)
└─ Permission checks on every request

Data Protection:
├─ Encryption at rest (AES-256)
├─ Encryption in transit (TLS 1.3)
├─ Password hashing (bcrypt)
└─ Key management (AWS KMS)

Attack Prevention:
├─ Rate limiting (prevent abuse)
├─ Input validation (prevent injection)
├─ DDoS protection (CloudFlare/AWS Shield)
└─ Content moderation (ML + human review)
```

**Phase 3: Compliance**

- GDPR (EU data protection)
- CCPA (California privacy)
- SOC 2 (security controls)

#### Interview Script: Security

**When asked "How do you secure a social media platform?":**

**You say:** "I'd implement a multi-layered security approach:

1. **Authentication & Authorization:**
   - JWT tokens with short expiration (15 min access, 30 day refresh)
   - OAuth 2.0 for social login
   - Role-based permissions (user, admin, moderator)

2. **Data Protection:**
   - Encryption at rest (AES-256 for sensitive data)
   - Encryption in transit (TLS 1.3 for all endpoints)
   - Password hashing (bcrypt with salt, cost factor 12)

3. **Attack Prevention:**
   - Rate limiting (prevent abuse, DDoS)
   - Input validation (prevent SQL injection, XSS)
   - Content moderation (ML detection + human review)

4. **Monitoring & Compliance:**
   - Security monitoring (detect anomalies)
   - Audit logs (track all access)
   - GDPR compliance (data export, deletion)"

⚠️ **Common Mistake:** Only focusing on authentication. Security requires multiple layers—authentication, authorization, encryption, monitoring, and compliance!

### 🔴 For Advanced: Production Considerations

#### Security Architecture and Business Impact

When you're making security decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Security vs User Experience**

```text
Scenario: Two-factor authentication (2FA)

Option A: Require 2FA for all users
├─ Security: High (prevents 99% of account takeovers)
├─ UX: Lower (extra step for users)
├─ Adoption: May reduce sign-ups by 10-15%
└─ Use Case: Financial apps, enterprise

Option B: Optional 2FA (opt-in)
├─ Security: Medium (only 20% users enable)
├─ UX: Better (no friction for most users)
├─ Adoption: Higher sign-up rate
└─ Use Case: Social media, consumer apps

💡 Real-world: Most social platforms use optional 2FA. 
   Instagram: 2FA optional, but required for business accounts.
```

**Trade-off 2: Content Moderation: Automated vs Human**

```text
Scenario: Detecting inappropriate content

Option A: Fully automated (ML only)
├─ Speed: Instant (<1 second)
├─ Cost: Low ($0.01 per image)
├─ Accuracy: 85% (15% false positives)
├─ Scale: Handles 100M posts/day
└─ Use Case: First-pass filtering

Option B: Human review only
├─ Speed: Slow (hours/days)
├─ Cost: High ($1 per review)
├─ Accuracy: 99% (1% errors)
├─ Scale: Limited (10K reviews/day)
└─ Use Case: Appeals, edge cases

Option C: Hybrid (ML + Human)
├─ Speed: Fast for clear cases, slow for edge cases
├─ Cost: Medium ($0.10 per post average)
├─ Accuracy: 95% (5% false positives)
├─ Scale: Handles 100M posts/day
└─ Use Case: Production (Instagram, Facebook)

💡 Real-world: All major platforms use hybrid approach.
   ML filters 90% automatically, humans review 10% flagged content.
```

**Trade-off 3: Encryption: Performance vs Security**

```text
Scenario: End-to-end encryption for messages

Option A: Full E2E encryption
├─ Security: Maximum (only sender/receiver can read)
├─ Performance: Lower (encryption overhead)
├─ Features: Limited (can't search encrypted messages)
├─ Compliance: Harder (can't scan for abuse)
└─ Use Case: WhatsApp, Signal

Option B: Server-side encryption
├─ Security: Good (encrypted in transit and at rest)
├─ Performance: Better (server can optimize)
├─ Features: Full (search, moderation, analytics)
├─ Compliance: Easier (can scan for abuse)
└─ Use Case: Instagram DMs, Twitter DMs

💡 Real-world: Most platforms use server-side encryption.
   E2E only for privacy-focused features (WhatsApp).
```

#### Advanced Security Patterns

**1. Zero Trust Architecture**

```text
Principle: Never trust, always verify

Implementation:
├─ Verify every request (even from internal services)
├─ Least privilege (users get minimum permissions)
├─ Micro-segmentation (isolate services)
└─ Continuous monitoring (detect anomalies)

Example:
├─ User requests feed → Verify JWT token
├─ Service calls database → Verify service certificate
├─ Admin accesses logs → Verify admin role + 2FA
└─ All actions logged → Audit trail
```

**2. Content Moderation Pipeline**

```text
Multi-Stage Pipeline:

Stage 1: Pre-upload Checks
├─ File type validation
├─ Size limits
└─ Virus scanning

Stage 2: ML Detection (Automated)
├─ NSFW detection (AWS Rekognition)
├─ Violence detection (custom ML model)
├─ Spam detection (text analysis)
└─ Profanity filter (keyword matching)

Stage 3: Human Review (Flagged Content)
├─ Queue for human moderators
├─ Priority scoring (high-risk first)
└─ Decision: Approve/Reject/Appeal

Stage 4: Post-Publication Monitoring
├─ User reports
├─ Automated re-scanning (improved models)
└─ Appeals process
```

**3. Rate Limiting Strategies**

```text
Token Bucket Algorithm:

Implementation:
├─ Each user gets a "bucket" of tokens
├─ Each action costs tokens
├─ Tokens refill over time
└─ Block if bucket empty

Example:
├─ Post creation: 100 tokens/hour
├─ Comment: 1000 tokens/hour
├─ Like: 5000 tokens/hour
└─ Follow: 200 tokens/hour

Advanced: Distributed Rate Limiting
├─ Use Redis for shared state
├─ Consistent hashing for sharding
└─ Sliding window for accuracy
```

**Enterprise Requirements:**

When selling to enterprises, security requirements expand:

```text
Compliance:
├─ SOC 2 Type II (security controls)
├─ ISO 27001 (information security)
├─ GDPR (EU data protection)
└─ HIPAA (if handling health data)

Features:
├─ SSO (Single Sign-On) integration
├─ Audit logs (all actions tracked)
├─ Data retention policies
└─ Custom security policies
```

### Real-World Example: How Instagram Secures 1.4B Users

Let's look at how Instagram (Meta) evolved their security:

**2010-2012 - Early Days:**

```text
Context: 10M users, simple photo-sharing
├─ Security: Basic password authentication
├─ Content Moderation: Manual review only
├─ Encryption: HTTPS only
└─ Result: Multiple security incidents

Lessons: Basic security isn't enough at scale
```

**2013-2016 - Growth Phase:**

```text
Context: 500M users, acquired by Facebook
├─ Added: Two-factor authentication (optional)
├─ Added: Automated content moderation (ML)
├─ Added: Rate limiting (prevent abuse)
├─ Added: Encryption at rest (user data)
└─ Result: Reduced security incidents by 80%

Lessons: Multi-layered security is essential
```

**2017-2020 - Scale Phase:**

```text
Context: 1B+ users, global platform
├─ Added: Advanced ML moderation (detect 95% automatically)
├─ Added: Human review teams (10K+ moderators)
├─ Added: End-to-end encryption (optional for DMs)
├─ Added: Privacy controls (GDPR compliance)
└─ Result: 99.9% security incident prevention

Lessons: Hybrid approach (ML + human) works best
```

**2021-2024 - Enterprise Phase:**

```text
Context: 1.4B users, business features
├─ Added: Zero Trust architecture
├─ Added: Advanced threat detection (AI-based)
├─ Added: Enterprise SSO integration
├─ Added: Compliance certifications (SOC 2, ISO 27001)
└─ Result: Enterprise-ready security

Lessons: Security is continuous, not one-time
```

📊 **By The Numbers:**

- 2018: 50M accounts secured with 2FA
- 2020: 95% of harmful content detected automatically
- 2022: 10K+ human moderators reviewing edge cases
- 2024: 99.9% security incident prevention rate

**Key Lesson:** Security evolves with scale. Start simple, add layers as you grow, and always balance security with user experience.

### 🎯 Interview Questions: Security

#### Question 1: How do you design authentication for 500M users?

**What the interviewer wants to know:**
- Do you understand JWT tokens and refresh tokens?
- Can you design scalable authentication?
- Do you think about security vs performance trade-offs?

**Answer Framework:**

```text
1. Token-Based Authentication
   ├─ Access token: JWT, 15-minute expiration
   ├─ Refresh token: Long-lived, stored in HttpOnly cookie
   ├─ Benefits: Stateless, scalable, secure
   └─ Storage: Redis for token blacklist (logout)

2. Token Refresh Flow
   ├─ Client sends refresh token
   ├─ Server validates refresh token
   ├─ Server issues new access token
   └─ Client uses new access token

3. Scalability Considerations
   ├─ Stateless: No server-side session storage
   ├─ Caching: Cache user permissions (Redis)
   ├─ Rate limiting: Prevent brute force attacks
   └─ Monitoring: Track failed login attempts

Example Numbers:
├─ 500M users × 10 logins/day = 5B logins/day
├─ Token validation: <5ms (JWT decode)
├─ Refresh token storage: 500M × 1KB = 500GB
└─ Cost: ~$50K/month (Redis cluster)
```

**Follow-up: What if a user's token is stolen?**

```text
Mitigation Strategies:
├─ Short token expiration (15 min limits damage)
├─ Token blacklist (Redis) for logged-out tokens
├─ Device fingerprinting (detect unusual devices)
├─ IP-based alerts (login from new location)
└─ Force re-authentication for sensitive actions
```

#### Question 2: How do you prevent DDoS attacks?

**What the interviewer wants to know:**
- Do you understand DDoS attack types?
- Can you design multi-layer protection?
- Do you think about cost vs protection?

**Answer Framework:**

```text
1. Layer 3/4 Protection (Network Layer)
   ├─ CloudFlare/AWS Shield (absorbs attacks)
   ├─ Geographic blocking (block high-risk countries)
   └─ Cost: $3K-5K/month for 10Gbps protection

2. Layer 7 Protection (Application Layer)
   ├─ Rate limiting (per IP, per user)
   ├─ Challenge-response (CAPTCHA for suspicious traffic)
   ├─ Anomaly detection (ML-based traffic analysis)
   └─ Cost: Included in CDN service

3. Application-Level Protection
   ├─ API Gateway rate limiting
   ├─ Circuit breakers (fail fast under load)
   ├─ Auto-scaling (handle legitimate traffic spikes)
   └─ Cost: Infrastructure scaling

Example:
├─ Attack: 100Gbps DDoS
├─ CDN absorbs: 95Gbps (filtered)
├─ Rate limiting blocks: 4.5Gbps (suspicious)
├─ Legitimate traffic: 0.5Gbps (served normally)
└─ Result: Service remains available
```

**Follow-up: What if the attack targets a specific endpoint?**

```text
Endpoint-Specific Protection:
├─ Identify target endpoint (/api/feed)
├─ Increase rate limit for that endpoint
├─ Add CAPTCHA for that endpoint
├─ Route to dedicated servers (isolate impact)
└─ Monitor and alert on unusual patterns
```

#### Question 3: How do you design content moderation?

**What the interviewer wants to know:**
- Do you understand ML-based moderation?
- Can you design a scalable pipeline?
- Do you think about false positives vs false negatives?

**Answer Framework:**

```text
1. Multi-Stage Pipeline
   ├─ Stage 1: Pre-upload validation (file type, size)
   ├─ Stage 2: ML detection (NSFW, violence, spam)
   ├─ Stage 3: Human review (flagged content)
   └─ Stage 4: Post-publication monitoring (user reports)

2. ML Models
   ├─ Image: AWS Rekognition (NSFW detection)
   ├─ Video: Custom ML model (violence detection)
   ├─ Text: NLP models (spam, profanity)
   └─ Accuracy: 85-95% (5-15% false positives)

3. Human Review Queue
   ├─ Priority scoring (high-risk first)
   ├─ 10K+ moderators worldwide
   ├─ SLA: Review within 24 hours
   └─ Appeals process for rejected content

Example Numbers:
├─ 200M posts/day
├─ ML auto-approves: 180M (90%)
├─ ML flags for review: 20M (10%)
├─ Human reviewers: 10K moderators
├─ Review capacity: 20M/day (2000 reviews/moderator/day)
└─ Cost: $50M/year (moderator salaries + ML infrastructure)
```

**Follow-up: How do you handle false positives?**

```text
False Positive Mitigation:
├─ Appeals process (users can contest)
├─ ML model retraining (learn from mistakes)
├─ Confidence thresholds (only high-confidence auto-reject)
├─ Human review for edge cases
└─ Continuous improvement (A/B test model updates)
```

### 🤔 Think About It

1. **For Beginners:** Why do you think we need both authentication (who you are) and authorization (what you can do)? (Hint: Think about what happens if someone steals your password)

2. **For Intermediate:** If you had to choose between requiring 2FA for all users (better security) or making it optional (better user experience), which would you prioritize for a social media platform? Why?

3. **For Advanced:** How would your security architecture change if you were building a social media platform specifically for:
   - Healthcare professionals (HIPAA compliance)?
   - Financial services (PCI DSS compliance)?
   - Government agencies (security clearances)?

### ✅ Key Takeaways

- **Security is multi-layered**: Authentication, authorization, encryption, monitoring, and compliance all work together
- **Balance security and UX**: Too much security hurts adoption, too little hurts trust
- **Content moderation requires hybrid approach**: ML for scale, humans for accuracy
- **Compliance is mandatory**: GDPR, CCPA, SOC 2 requirements must be built in from day one
- **Security evolves with scale**: Start simple, add layers as you grow
- **Zero Trust architecture**: Never trust, always verify—even internal services
- **Rate limiting prevents abuse**: Token bucket algorithm scales to millions of users
- **Encryption protects data**: Both at rest (storage) and in transit (network)
- **Monitoring detects threats**: Real-time anomaly detection catches attacks early
- **Human review is essential**: ML catches 90%, humans handle edge cases

### 🎯 Practice Exercise

**Scenario:** You're designing security for a new social media platform targeting 100M users. The platform will handle photos, videos, text posts, and direct messaging.

**Your Task:**

1. Design the authentication system (JWT tokens, refresh tokens, social login)
2. Design the authorization system (permissions, role-based access control)
3. Design the content moderation pipeline (ML detection + human review)
4. Design DDoS protection (multi-layer approach)
5. Design GDPR compliance features (data export, deletion, consent)

**Bonus Challenge:** How would you handle a security breach where 1M user passwords were compromised? Design the incident response plan including user notification, password reset, and prevention measures.

---

## Section 13: Keeping It Healthy (Monitoring)

### What You'll Learn

By the end of this section, you'll be able to:

- Design comprehensive monitoring and observability systems for social media platforms
- Define and track SLIs, SLOs, and SLAs for production systems
- Implement effective alerting strategies with proper prioritization
- Design distributed tracing and logging architectures
- Troubleshoot production issues using monitoring data

### Why This Matters

Without proper monitoring, you're flying blind. Real-world example: In 2021, Facebook experienced a 6-hour outage affecting 3.5B users—the root cause was a configuration error that took hours to diagnose because monitoring systems were also affected. Instagram serves 10B feed impressions daily—without monitoring, you can't detect issues, debug problems, or ensure reliability! In interviews, demonstrating you understand monitoring shows you can build production-grade systems that are observable and debuggable.

### 🟢 For Beginners: The Fundamentals

#### What is Monitoring?

Think of monitoring like a health checkup for your system:

```text
Health Checkup (Doctor):
├─ Check vital signs (heart rate, blood pressure)
├─ Run tests (blood work, X-rays)
├─ Review symptoms (pain, fatigue)
└─ Diagnose problems (illness, disease)

System Monitoring (Platform):
├─ Check metrics (CPU, memory, latency)
├─ Collect logs (errors, requests)
├─ Track performance (response times, throughput)
└─ Detect issues (slow queries, failures)
```

**The Challenge:**

Social media platforms handle billions of requests daily. Without monitoring, you can't:
- Know if the system is healthy
- Detect problems before users complain
- Debug issues when they occur
- Ensure the system meets performance goals

#### Key Metrics to Monitor

**System Metrics:**

```text
Latency:
├─ How long requests take (p50, p95, p99)
├─ Example: Feed load time < 500ms
└─ Why: Users notice slow responses

Throughput:
├─ How many requests per second
├─ Example: 347,000 requests/sec peak
└─ Why: Need to handle traffic spikes

Error Rate:
├─ Percentage of failed requests
├─ Example: < 0.1% errors
└─ Why: Errors indicate problems

Resource Usage:
├─ CPU, memory, disk, network
├─ Example: CPU < 70% average
└─ Why: Prevent resource exhaustion
```

**Business Metrics:**

```text
User Engagement:
├─ Daily Active Users (DAU)
├─ Posts per day
├─ Engagement rate (likes + comments / impressions)
└─ Why: Measure product success

Performance:
├─ Feed load time
├─ Media upload success rate
├─ User retention (D1, D7, D30)
└─ Why: User experience matters
```

💡 **Pro Tip:** Monitor both technical metrics (system health) and business metrics (user impact)!

#### What are SLIs, SLOs, and SLAs?

**SLI (Service Level Indicator)**: What you measure

```text
Example: Feed request latency
├─ Measurement: Percentage of requests < 500ms
├─ Current: 95% of requests complete in < 500ms
└─ Formula: (requests_under_500ms / total_requests) × 100
```

**SLO (Service Level Objective)**: What you promise

```text
Example: Feed latency SLO
├─ Target: 95% of requests < 500ms
├─ Error Budget: 5% can be slower
└─ Measurement Window: 7 days rolling
```

**SLA (Service Level Agreement)**: What you guarantee to customers

```text
Example: Availability SLA
├─ Promise: 99.9% uptime
├─ Penalty: Service credit if below 99.9%
└─ Calculation: Maximum 43.2 minutes downtime/month
```

### 🟡 For Intermediate: Interview Patterns

#### The Monitoring Interview Framework

When discussing monitoring in interviews, follow this structure:

**Phase 1: Metrics Definition**

- "What metrics should we track?"
- "How do we measure system health?"
- "What are our SLOs?"

**Phase 2: Monitoring Architecture**

```text
Metrics Collection:
├─ Application metrics (Prometheus)
├─ Infrastructure metrics (CloudWatch)
├─ Business metrics (custom dashboards)
└─ Real-time metrics (Grafana)

Logging:
├─ Structured logging (JSON format)
├─ Log aggregation (ELK Stack)
├─ Log retention (30 days hot, 1 year cold)
└─ Log indexing (by service, timestamp, trace_id)

Tracing:
├─ Distributed tracing (Jaeger, OpenTelemetry)
├─ Request tracing (end-to-end)
├─ Service dependencies (dependency graph)
└─ Performance analysis (bottleneck identification)
```

**Phase 3: Alerting**

- "What should we alert on?"
- "How do we prioritize alerts?"
- "How do we prevent alert fatigue?"

#### Interview Script: Monitoring

**When asked "How do you monitor a social media platform?":**

**You say:** "I'd implement comprehensive observability:

1. **Metrics:**
   - System metrics (latency, throughput, error rate)
   - Business metrics (DAU, engagement rate)
   - SLOs (95% of feed requests < 500ms)

2. **Logging:**
   - Structured logs (JSON format)
   - Centralized aggregation (ELK Stack)
   - Trace IDs for request correlation

3. **Tracing:**
   - Distributed tracing (Jaeger)
   - End-to-end request tracking
   - Service dependency mapping

4. **Alerting:**
   - P0 alerts (page immediately)
   - P1 alerts (notify during business hours)
   - P2 alerts (weekly review)"

⚠️ **Common Mistake:** Alerting on everything. Too many alerts cause alert fatigue—only alert on actionable issues!

### 🔴 For Advanced: Production Considerations

#### Monitoring Architecture and Business Impact

When you're making monitoring decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Monitoring Granularity vs Cost**

```text
Scenario: Log retention and metrics collection

Option A: High granularity (all metrics, long retention)
├─ Detail: Every metric, 1-year retention
├─ Cost: High ($500K/month for storage)
├─ Benefit: Complete visibility, historical analysis
└─ Use Case: Enterprise, compliance requirements

Option B: Medium granularity (key metrics, moderate retention)
├─ Detail: Important metrics, 30-day retention
├─ Cost: Medium ($50K/month)
├─ Benefit: Good visibility, reasonable cost
└─ Use Case: Most production systems

Option C: Low granularity (critical metrics only)
├─ Detail: Only critical metrics, 7-day retention
├─ Cost: Low ($5K/month)
├─ Benefit: Minimal cost, basic visibility
└─ Use Case: MVP, cost-sensitive startups

💡 Real-world: Instagram uses medium granularity—key metrics 
   with 30-day hot storage, 1-year cold storage for compliance.
```

**Trade-off 2: Real-time vs Batch Monitoring**

```text
Scenario: Metrics collection frequency

Option A: Real-time (1-second intervals)
├─ Latency: Immediate visibility
├─ Cost: High (more data points, storage)
├─ Use Case: Critical systems, low-latency requirements
└─ Example: Feed latency, error rates

Option B: Near real-time (1-minute intervals)
├─ Latency: 1-minute delay acceptable
├─ Cost: Medium (balanced)
├─ Use Case: Most production systems
└─ Example: Throughput, resource usage

Option C: Batch (5-minute intervals)
├─ Latency: 5-minute delay acceptable
├─ Cost: Low (fewer data points)
├─ Use Case: Non-critical metrics, cost optimization
└─ Example: Business metrics, weekly reports

💡 Real-world: Critical metrics (latency, errors) are real-time,
   business metrics (DAU, engagement) are batch (hourly/daily).
```

**Trade-off 3: Alert Sensitivity vs Alert Fatigue**

```text
Scenario: Alert threshold configuration

Option A: Sensitive alerts (alert on small changes)
├─ Detection: Catches issues early
├─ Noise: Many false positives
├─ Fatigue: High (engineers ignore alerts)
└─ Use Case: Critical systems (payment, auth)

Option B: Balanced alerts (alert on significant changes)
├─ Detection: Catches real issues
├─ Noise: Some false positives
├─ Fatigue: Medium (manageable)
└─ Use Case: Most production systems

Option C: Conservative alerts (alert only on major issues)
├─ Detection: Misses early warnings
├─ Noise: Few false positives
├─ Fatigue: Low (all alerts are important)
└─ Use Case: Non-critical systems

💡 Real-world: Use tiered alerting—P0 (sensitive) for critical 
   systems, P1/P2 (balanced) for others.
```

#### Advanced Monitoring Patterns

**1. Distributed Tracing**

```text
Purpose: Track requests across multiple services

Implementation:
├─ Trace ID: Unique ID per request
├─ Span: Individual operation within trace
├─ Context propagation: Pass trace ID between services
└─ Visualization: Show request flow through services

Example Trace:
User Request: Get Feed
├─ API Gateway (10ms)
├─ Auth Service (5ms)
├─ Feed Service (230ms)
│  ├─ Redis: Get Timeline (2ms)
│  ├─ Graph DB: Get Celebrities (15ms)
│  ├─ Post Service: Batch Fetch (50ms)
│  └─ Ranking Service: ML Inference (150ms)
└─ Response to Client (5ms)

Total: 245ms
Bottleneck: Ranking Service (150ms)
```

**2. SLO-Based Alerting**

```text
Purpose: Alert based on SLO violations, not raw metrics

Implementation:
├─ Calculate SLO compliance (rolling window)
├─ Track error budget consumption
├─ Alert when error budget at risk
└─ Prevent alert fatigue (only alert on SLO violations)

Example:
├─ SLO: 95% of feed requests < 500ms
├─ Error Budget: 5% (36,000 failures/day allowed)
├─ Current: 4.5% failures (32,400 failures/day)
├─ Remaining: 0.5% (3,600 failures/day)
└─ Alert: Only if error budget consumption accelerating
```

**3. Anomaly Detection**

```text
Purpose: Detect unusual patterns automatically

Implementation:
├─ Baseline: Learn normal patterns (ML-based)
├─ Detection: Flag deviations from baseline
├─ Alerting: Alert on significant anomalies
└─ Learning: Continuously update baseline

Example:
├─ Normal: 347K requests/sec peak
├─ Anomaly: 500K requests/sec (unusual spike)
├─ Detection: ML model flags as anomaly
└─ Alert: Investigate potential DDoS or viral content
```

**Enterprise Requirements:**

When operating at enterprise scale, monitoring requirements expand:

```text
Compliance:
├─ Audit logs (all actions tracked)
├─ Data retention (7-year retention for compliance)
├─ Access logs (who accessed what data)
└─ Compliance reporting (SOC 2, GDPR)

Features:
├─ Custom dashboards (per team, per service)
├─ Alert routing (on-call rotation)
├─ Incident management (PagerDuty integration)
└─ Post-mortem automation (incident analysis)
```

### Real-World Example: How Instagram Monitors 1.4B Users

Let's look at how Instagram (Meta) evolved their monitoring:

**2010-2012 - Early Days:**

```text
Context: 10M users, simple monitoring
├─ Monitoring: Basic server metrics (CPU, memory)
├─ Logging: File-based logs
├─ Alerting: Email alerts only
└─ Result: Slow incident detection

Lessons: Basic monitoring isn't enough at scale
```

**2013-2016 - Growth Phase:**

```text
Context: 500M users, acquired by Facebook
├─ Added: Centralized metrics (Prometheus)
├─ Added: Structured logging (ELK Stack)
├─ Added: PagerDuty integration (on-call)
├─ Added: SLO tracking (availability, latency)
└─ Result: Faster incident detection (minutes vs hours)

Lessons: Centralized monitoring is essential
```

**2017-2020 - Scale Phase:**

```text
Context: 1B+ users, global platform
├─ Added: Distributed tracing (Jaeger)
├─ Added: Real-time dashboards (Grafana)
├─ Added: ML-based anomaly detection
├─ Added: SLO-based alerting (error budgets)
└─ Result: Proactive issue detection (before users notice)

Lessons: Observability requires multiple tools
```

**2021-2024 - Enterprise Phase:**

```text
Context: 1.4B users, enterprise features
├─ Added: Advanced analytics (ML-based insights)
├─ Added: Automated incident response
├─ Added: Compliance monitoring (GDPR, SOC 2)
├─ Added: Cost monitoring (infrastructure optimization)
└─ Result: Self-healing systems, proactive optimization

Lessons: Monitoring enables automation and optimization
```

📊 **By The Numbers:**

- 2018: 1M+ metrics collected per second
- 2020: 10B+ log entries per day
- 2022: 99.9% SLO compliance (availability)
- 2024: <5 minute mean time to detection (MTTD)

**Key Lesson:** Monitoring evolves from reactive (detect issues) to proactive (prevent issues) to predictive (optimize before problems occur).

### 🎯 Interview Questions: Monitoring

#### Question 1: How do you design monitoring for a social media platform?

**What the interviewer wants to know:**
- Do you understand key metrics to track?
- Can you design a monitoring architecture?
- Do you think about SLIs, SLOs, and SLAs?

**Answer Framework:**

```text
1. Define Key Metrics
   ├─ System metrics: Latency (p50, p95, p99), throughput, error rate
   ├─ Business metrics: DAU, engagement rate, retention
   ├─ Resource metrics: CPU, memory, disk, network
   └─ Custom metrics: Feed load time, media upload success rate

2. Monitoring Architecture
   ├─ Metrics: Prometheus (collection) + Grafana (visualization)
   ├─ Logging: ELK Stack (Elasticsearch, Logstash, Kibana)
   ├─ Tracing: Jaeger or OpenTelemetry
   └─ Alerting: PagerDuty (on-call rotation)

3. SLO Definition
   ├─ Availability: 99.9% uptime
   ├─ Latency: 95% of feed requests < 500ms
   ├─ Error Rate: < 0.1% errors
   └─ Error Budget: Track and alert on consumption

Example Numbers:
├─ Metrics: 1M+ metrics/sec
├─ Logs: 10B+ log entries/day
├─ Traces: 100M+ traces/day
└─ Cost: ~$200K/month (monitoring infrastructure)
```

**Follow-up: How do you prevent alert fatigue?**

```text
Strategies:
├─ Tiered alerting (P0, P1, P2 priorities)
├─ SLO-based alerting (only alert on SLO violations)
├─ Alert grouping (group related alerts)
├─ On-call rotation (distribute load)
└─ Alert review (regular cleanup of noisy alerts)
```

#### Question 2: How do you debug a latency spike?

**What the interviewer wants to know:**
- Can you use monitoring data to debug issues?
- Do you understand distributed tracing?
- Can you identify bottlenecks?

**Answer Framework:**

```text
1. Identify the Issue
   ├─ Check dashboards: Which service has high latency?
   ├─ Check SLOs: Is error budget being consumed?
   ├─ Check alerts: Any related alerts?
   └─ Check time range: When did it start?

2. Use Distributed Tracing
   ├─ Find slow traces: Filter by latency > threshold
   ├─ Analyze trace: Which span is slow?
   ├─ Check dependencies: Is downstream service slow?
   └─ Identify bottleneck: Root cause service

3. Deep Dive
   ├─ Check logs: Errors, warnings in slow service
   ├─ Check metrics: Resource usage, queue depth
   ├─ Check dependencies: Database, cache, external APIs
   └─ Check recent changes: Deployments, config changes

Example:
├─ Issue: Feed latency p95 jumped from 300ms to 1.2s
├─ Trace analysis: Ranking Service taking 900ms (normal: 150ms)
├─ Logs: ML model inference timeout
├─ Root cause: ML model serving overloaded
└─ Fix: Scale up ML serving infrastructure
```

**Follow-up: How do you prevent this from happening again?**

```text
Prevention:
├─ Alert on latency SLO violations
├─ Alert on ML serving queue depth
├─ Auto-scaling based on queue depth
├─ Load testing before deployments
└─ Circuit breakers for downstream services
```

#### Question 3: How do you design SLOs for a feed service?

**What the interviewer wants to know:**
- Do you understand SLI, SLO, SLA concepts?
- Can you define appropriate SLOs?
- Do you think about error budgets?

**Answer Framework:**

```text
1. Define SLI (Service Level Indicator)
   ├─ What: Percentage of feed requests < 500ms
   ├─ How: Measure request latency, count successes
   └─ Formula: (requests_under_500ms / total_requests) × 100

2. Define SLO (Service Level Objective)
   ├─ Target: 95% of requests < 500ms
   ├─ Window: 7-day rolling window
   ├─ Error Budget: 5% can be slower
   └─ Calculation: Track compliance continuously

3. Define SLA (Service Level Agreement)
   ├─ Promise: 99.9% availability + 95% latency SLO
   ├─ Penalty: Service credit if below targets
   └─ Measurement: Monthly compliance report

Example:
├─ Total requests: 1B/day
├─ SLO: 95% < 500ms = 950M requests must be fast
├─ Error Budget: 50M requests can be slow
├─ Current: 960M fast (96% compliance) ✓
└─ Remaining Budget: 40M slow requests available
```

**Follow-up: What if error budget is being consumed too fast?**

```text
Response:
├─ Alert: Error budget consumption accelerating
├─ Investigation: Why are requests slow?
├─ Mitigation: Scale up, optimize queries, add caching
├─ Communication: Notify stakeholders
└─ Post-mortem: Learn and prevent recurrence
```

### 🤔 Think About It

1. **For Beginners:** Why do you think we need to monitor both system metrics (like CPU usage) and business metrics (like daily active users)? (Hint: Think about what each tells you about your system)

2. **For Intermediate:** If you had to choose between monitoring everything in real-time (expensive) or monitoring key metrics with delays (cheaper), which would you prioritize for a social media platform? Why?

3. **For Advanced:** How would your monitoring architecture change if you were building a social media platform specifically for:
   - Healthcare (HIPAA compliance, audit requirements)?
   - Financial services (PCI DSS, transaction monitoring)?
   - Government (security clearances, data retention)?

### ✅ Key Takeaways

- **Monitoring is essential**: You can't manage what you don't measure
- **Define SLIs, SLOs, SLAs**: Clear targets enable effective monitoring
- **Three pillars of observability**: Metrics, logs, and traces work together
- **Alert on SLOs, not raw metrics**: Prevents alert fatigue
- **Distributed tracing is crucial**: Debug issues across microservices
- **Structured logging enables debugging**: JSON format with trace IDs
- **Error budgets guide decisions**: Track and manage SLO compliance
- **Tiered alerting prevents fatigue**: P0 (critical), P1 (important), P2 (informational)
- **Monitor business metrics**: Technical metrics alone don't tell the full story
- **Automation reduces toil**: Auto-scaling, auto-remediation based on metrics

### 🎯 Practice Exercise

**Scenario:** You're designing monitoring for a social media platform serving 100M users. The platform has 10 microservices handling feeds, posts, media, notifications, and more.

**Your Task:**

1. Define key metrics to monitor (system, business, custom)
2. Design the monitoring architecture (metrics, logging, tracing)
3. Define SLIs and SLOs for the feed service
4. Design an alerting strategy (what to alert on, priorities)
5. Design a debugging workflow (how to use monitoring to debug issues)

**Bonus Challenge:** How would you design monitoring for a multi-region deployment where you need to track latency, availability, and data consistency across regions?

---

## Section 11: Growing the System (Scalability)

### What You'll Learn

By the end of this section, you'll be able to:

- Design multi-layer caching strategies to improve performance and reduce database load
- Identify and resolve system bottlenecks as you scale
- Implement horizontal scaling strategies for databases, caches, and application servers
- Optimize system performance through caching, database sharding, and load balancing
- Handle scaling challenges like hot partitions, cache invalidation, and distributed system coordination

### Why This Matters

Scalability determines whether your system can grow from thousands to millions of users. Real-world example: Facebook's TAO caching system handles 1 billion reads per second—without proper caching and scaling strategies, the system would collapse under load! In interviews, demonstrating you understand scalability shows you can build systems that grow with the business.

### 🟢 For Beginners: The Fundamentals

#### What is Scalability?

Think of scalability like a restaurant:

- **Small restaurant** = 10 tables, 1 waiter (works for small groups)
- **Growing restaurant** = 100 tables, 10 waiters (needs more staff)
- **Chain restaurant** = 1,000 locations, 10,000 waiters (needs systems to coordinate)

**The Challenge:**

As your platform grows from 1,000 users to 1 million users, you can't just buy a bigger server—you need to design systems that can grow horizontally (add more servers)!

#### What is Caching?

Think of caching like keeping your favorite snacks in your desk drawer:

- **Without cache:** Go to the store every time you want a snack (slow!)
- **With cache:** Grab snack from your drawer (fast!)

**In System Terms:**

```text
Without Cache:
User requests feed → Query database → Wait 500ms → Return feed

With Cache:
User requests feed → Check cache → Found! → Return feed in 50ms
```

💡 **Pro Tip:** Caching stores frequently accessed data in fast memory so you don't have to query slow databases every time!

#### Why Do We Need Multiple Cache Layers?

**Single Cache Problem:**

```text
One cache for everything:
├─ Problem: Popular data pushes out less popular data
├─ Problem: Can't optimize for different access patterns
└─ Problem: All-or-nothing (cache miss = slow)

Solution: Multiple cache layers!
├─ L1: Fastest, smallest (most popular data)
├─ L2: Fast, larger (popular data)
└─ L3: Slower, largest (all data)
```

### 🟡 For Intermediate: Interview Patterns

#### The Scalability Framework

When designing for scalability in an interview, address these core challenges:

**Challenge 1: Database Scaling**

```text
Problem: Database becomes bottleneck
Solution: Multiple strategies
├─ Read replicas: Scale reads horizontally
├─ Sharding: Partition data across servers
├─ Caching: Reduce database load
└─ Denormalization: Pre-compute expensive queries
```

**Challenge 2: Cache Strategy**

```text
Problem: Need fast access to frequently used data
Solution: Multi-layer caching
├─ L1: In-memory (Redis) - fastest, smallest
├─ L2: Distributed cache (Memcached) - fast, larger
├─ L3: Application cache - medium speed
└─ L4: CDN - slowest, global distribution
```

**Challenge 3: Hot Partitions**

```text
Problem: Some data accessed much more than others
Solution: Mitigation strategies
├─ Partitioning: Distribute hot data across partitions
├─ Caching: Cache hot data aggressively
├─ Replication: Replicate hot data to multiple servers
└─ Rate limiting: Prevent abuse of hot endpoints
```

#### Interview Script: Scalability

**When asked "How would you scale this system?":**

**You say:** "I'd implement a multi-pronged scaling strategy:

1. **Caching:**
   - Multi-layer cache (L1: Redis, L2: Memcached, L3: Application)
   - Cache hot data aggressively (90%+ hit ratio target)
   - TTL-based expiration + event-driven invalidation

2. **Database Scaling:**
   - Read replicas for read-heavy workloads
   - Sharding for write-heavy workloads
   - Denormalization for expensive queries

3. **Application Scaling:**
   - Horizontal scaling (add more servers)
   - Load balancing (distribute traffic)
   - Auto-scaling (scale based on load)

4. **Bottleneck Identification:**
   - Monitor key metrics (latency, throughput, error rate)
   - Identify hot partitions and optimize
   - Use profiling to find slow queries"

⚠️ **Common Mistake:** Trying to scale vertically (bigger servers) instead of horizontally (more servers). Horizontal scaling is the key to true scalability!

### 🔴 For Advanced: Production Considerations

#### Scalability Architecture and Business Impact

When you're making scalability decisions, every choice has business implications. Let's think like a Principal Engineer:

**Trade-off 1: Cache Consistency vs Performance**

```text
Scenario: Cache invalidation strategy

Option A: Strong consistency (invalidate immediately)
├─ Consistency: Always fresh data
├─ Performance: More cache misses, higher database load
├─ Cost: Higher (more database queries)
└─ Use Case: Financial data, critical operations

Option B: Eventual consistency (TTL-based)
├─ Consistency: Slight delay acceptable
├─ Performance: Better cache hit rate, lower database load
├─ Cost: Lower (fewer database queries)
└─ Use Case: Social media feeds, non-critical data

💡 Real-world: Most platforms use eventual consistency for feeds, strong consistency for critical operations.
```

**Trade-off 2: Cache Size vs Cost**

```text
Scenario: Cache capacity planning

Option A: Large cache (cache everything)
├─ Hit rate: 95%+ (excellent)
├─ Cost: High (memory expensive)
├─ Latency: Low (most requests hit cache)
└─ Use Case: High-traffic, cost-tolerant

Option B: Small cache (cache hot data only)
├─ Hit rate: 70% (good)
├─ Cost: Low (less memory)
├─ Latency: Higher (more cache misses)
└─ Use Case: Cost-sensitive, moderate traffic

💡 Real-world: Facebook's TAO cache is massive (petabytes) but reduces database load by 99%.
```

#### Advanced Scalability Patterns

**Handling Hot Partitions:**

```text
Hot Partition Problem: Celebrity post gets millions of likes
├─ Single partition overloaded
├─ Database contention
└─ Slow response times

Solutions:
├─ Partitioning: Distribute likes across multiple partitions
├─ Caching: Cache like counts aggressively
├─ Batching: Batch updates instead of individual writes
└─ Rate limiting: Prevent abuse
```

**Cache Invalidation Strategies:**

```text
Strategy 1: TTL-based (time-to-live)
├─ Simple: Set expiration time
├─ Pros: Automatic, no coordination needed
└─ Cons: Stale data until expiration

Strategy 2: Event-driven
├─ Complex: Invalidate on updates
├─ Pros: Always fresh data
└─ Cons: Requires coordination, can cause thundering herd

Strategy 3: Version-based
├─ Medium: Include version in cache key
├─ Pros: No invalidation needed, gradual rollout
└─ Cons: Multiple versions in cache simultaneously

💡 Real-world: Most platforms use hybrid—TTL for most data, event-driven for critical updates.
```

### Cache Layers

**Purpose:** Store pre-computed timelines for fast feed generation

**Data:**

```text
Key: timeline:{user_id}
Type: Sorted Set
Score: post_timestamp
Member: post_id
Size per user: ~50KB (1000 posts * 50 bytes/post_id)
Total size: 100M users * 50KB = 5TB
TTL: 24 hours
```

**Operations:**

```text
Write: ZADD timeline:{user_id} {timestamp} {post_id}
Read: ZREVRANGE timeline:{user_id} 0 49
Eviction: Automatic (TTL) + LRU
```

**Hit Ratio Target:** 90%+

**Miss Handling:** Regenerate from Graph DB + Post DB

---

#### L2: Post Metadata Cache (Memcached)

**Purpose:** Cache frequently accessed post details

**Data:**

```text
Key: post:{post_id}
Type: String (serialized JSON)
Size per post: ~2KB
Total size: 20M hot posts * 2KB = 40GB
TTL: 6 hours
```

**Cached Fields:**

```json
{
  "post_id": "uuid",
  "user_id": "uuid",
  "caption": "text",
  "media_urls": [],
  "like_count": 1542,
  "comment_count": 87,
  "created_at": "timestamp"
}
```

**Hit Ratio Target:** 95%+

**Miss Handling:** Fetch from Cassandra

---

#### L3: User Profile Cache (Redis)

**Purpose:** Cache user profile data for feed rendering

**Data:**

```text
Key: user:{user_id}
Type: Hash
Fields: username, full_name, profile_picture_url, is_verified
Size per user: ~500 bytes
Total size: 100M active users * 500B = 50GB
TTL: 1 hour
```

**Hit Ratio Target:** 98%+

---

#### L4: Ranked Feed Cache (Redis)

**Purpose:** Cache ML-ranked feeds to avoid recomputation

**Data:**

```text
Key: feed:ranked:{user_id}
Type: List
Members: [post_id_1, post_id_2, ..., post_id_50]
Size per user: ~2KB
Total size: 100M users * 2KB = 200GB
TTL: 15 minutes (fresh rankings)
```

**Hit Ratio Target:** 70% (many users don't return within 15 min)

---

### Cache Invalidation Strategy

#### 1. Time-Based Invalidation (TTL)

**Use Cases:**

- Timeline cache: 24 hours (old posts naturally expire)
- Post metadata: 6 hours (engagement counts eventually consistent)
- Ranked feed: 15 minutes (ensure fresh rankings)

**Rationale:** Social media can tolerate eventual consistency. Stale engagement counts for a few hours are acceptable.

---

#### 2. Event-Based Invalidation

**Trigger Events:**

```python
# User updates profile
on_profile_update(user_id):
    redis.delete(f'user:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')

# User follows someone
on_follow(user_id, target_id):
    redis.delete(f'timeline:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')

# User unfollows someone
on_unfollow(user_id, target_id):
    redis.delete(f'timeline:{user_id}')
    redis.delete(f'feed:ranked:{user_id}')
    # Remove target's posts from timeline
    target_post_ids = get_user_recent_posts(target_id)
    redis.zrem(f'timeline:{user_id}', *target_post_ids)

# Post deleted
on_post_delete(post_id, author_id):
    redis.delete(f'post:{post_id}')
    # Remove from all follower timelines (async)
    publish_event('post.deleted', {post_id, author_id})
```

---

#### 3. Approximate Counters

**Problem:** Incrementing like_count in cache and DB creates race conditions and consistency issues.

**Solution:** Use Redis for real-time counters, periodically sync to Cassandra.

```python
# Increment counter in Redis
def like_post(post_id, user_id):
    # Atomic increment
    redis.incr(f'post:likes:{post_id}')
    
    # Record individual like
    cassandra.insert('likes', {
        'post_id': post_id,
        'user_id': user_id,
        'created_at': now()
    })
    
    # Publish event for analytics
    kafka.publish('engagement.like', {post_id, user_id})

# Periodic sync (every 5 minutes)
def sync_counters_to_db():
    for post_id in get_active_posts():
        like_count = redis.get(f'post:likes:{post_id}')
        if like_count:
            cassandra.update('posts', {
                'post_id': post_id,
                'like_count': like_count
            })
```

**Tradeoff:** Counters may be slightly off (±1%) but eventual consistency is acceptable for social media.

---

### What NOT to Cache

1. **User Password Hashes:** Security risk
2. **Private/Sensitive Data:** GDPR compliance issues
3. **Real-Time Engagement:** Like/comment actions must be immediately visible to actor
4. **Financial Data:** Requires strong consistency
5. **Audit Logs:** Must be durable and accurate

---

## Bottlenecks & Improvements

### Potential Bottlenecks

#### Bottleneck 1: Database Write Contention

**Problem:**

- 173,600 like actions/sec at peak
- Hot posts can receive thousands of likes/sec
- Cassandra counter updates can create contention

**Solution:**

- Use Redis for real-time counters (INCR is atomic and fast)
- Batch sync to Cassandra every 5 minutes
- Use Cassandra COUNTER type for approximate counts
- Implement optimistic locking for critical updates

**Monitoring:**

```text
Metric: database_write_latency_p99
Threshold: > 100ms
Alert: Page on-call engineer
```

---

#### Bottleneck 2: Media Processing Queue Depth

**Problem:**

- Peak: 7,000 uploads/sec
- If workers fall behind, queue grows indefinitely
- User experience degrades (posts stuck in "processing")

**Solution:**

- Auto-scaling based on queue depth
- Scale-up trigger: Depth > 1000 for 2 minutes
- Scale-down trigger: Depth < 100 for 10 minutes
- Priority queue for verified users
- Reject uploads if queue > 10,000 (circuit breaker)

**Monitoring:**

```text
Metric: kafka_topic_lag (media.process)
Threshold: > 5000 messages
Alert: Auto-scale workers + notify team
```

---

#### Bottleneck 3: Graph Database Read Load

**Problem:**

- Every feed generation queries "get followers"
- 115,740 feed requests/sec = 115,740 graph queries/sec
- Neo4j read capacity: ~50,000 queries/sec per node

**Solution:**

- Cache follower lists in Redis
- Update cache on follow/unfollow events
- Use read replicas (3-5 replicas)
- Denormalize: Store follower_count in User DB

**Monitoring:**

```text
Metric: neo4j_query_latency_p95
Threshold: > 50ms
Alert: Add read replica
```

---

#### Bottleneck 4: CDN Origin Bandwidth

**Problem:**

- Cache miss rate: 5%
- Peak bandwidth: 16.5Pbps * 0.05 = 825Tbps
- Single origin can't handle this

**Solution:**

- Multi-region origin (10+ regions)
- Origin shield (secondary cache layer before origin)
- Increase cache TTL (1 hour → 6 hours for immutable media)
- Proactive cache warming for popular content

**Monitoring:**

```text
Metric: cdn_origin_requests_per_second
Threshold: > 100,000 req/sec per region
Alert: Add origin capacity
```

---

#### Bottleneck 5: Single Point of Failure (API Gateway)

**Problem:**

- All traffic goes through API Gateway
- If gateway fails, entire system is down

**Solution:**

- Multi-AZ deployment (3+ availability zones)
- Auto-scaling (target: 60% CPU utilization)
- Circuit breaker pattern (fail fast on downstream errors)
- Health checks with automatic failover
- Canary deployments (1% → 10% → 100%)

**Monitoring:**

```text
Metric: api_gateway_error_rate
Threshold: > 1%
Alert: Rollback deployment + page on-call
```

---

### Scalability Improvements

#### 1. Geographic Distribution

**Strategy:**

- Deploy in 5 regions: US-East, US-West, Europe, Asia, South America
- Route users to nearest region (latency-based routing)
- Replicate critical data across regions (user data, social graph)
- Local caching in each region

**Benefits:**

- Reduced latency (500ms → 100ms)
- Better availability (region failure doesn't affect other regions)
- Compliance (GDPR data residency)

**Implementation:**

```text
User Location: US-East
└─> Route to US-East cluster
    ├─> API Gateway (us-east-1)
    ├─> Feed Service (us-east-1)
    ├─> User DB (primary in us-east-1, replicas in us-west-2)
    └─> CDN (nearest edge location)
```

---

#### 2. Read Replicas

**Databases:**

- User DB (PostgreSQL): 1 primary + 5 read replicas
- Post DB (Cassandra): 3 replicas per datacenter
- Graph DB (Neo4j): 1 primary + 3 read replicas

**Read/Write Split:**

```python
# Writes go to primary
def create_user(user_data):
    return primary_db.insert('users', user_data)

# Reads go to replicas (round-robin)
def get_user(user_id):
    return replica_db.query('SELECT * FROM users WHERE user_id = :id', id=user_id)
```

**Benefits:**

- 5x read capacity
- Zero-downtime maintenance (rotate replicas)
- Disaster recovery (promote replica to primary)

---

#### 3. Sharding Strategy

**User DB Sharding:**

```text
Shard by user_id (consistent hashing)
Shard 0: user_id hash % 16 == 0
Shard 1: user_id hash % 16 == 1
...
Shard 15: user_id hash % 16 == 15

Start with 16 shards, grow to 256 shards over time
```

**Post DB Sharding:**

```text
Already sharded by user_id (Cassandra native)
512 virtual nodes per physical node
Automatic rebalancing on node addition
```

**Benefits:**

- Horizontal scalability (add shards as needed)
- Isolated failures (one shard down doesn't affect others)
- Parallel queries (query all shards simultaneously)

---

#### 4. Real-Time Updates (WebSocket)

**Current:** Polling (client requests feed every 30 seconds)

**Improvement:** WebSocket for live updates

```python
# Server-side
websocket_server = WebSocketServer()

@websocket_server.on_connect
def on_user_connect(user_id, websocket):
    # Subscribe to user's real-time updates
    pubsub.subscribe(f'user:{user_id}:updates', websocket)

@event_handler('engagement.like')
def on_like(post_id, user_id):
    # Notify post author
    post = get_post(post_id)
    pubsub.publish(f'user:{post.author_id}:updates', {
        'type': 'like',
        'post_id': post_id,
        'user_id': user_id
    })
```

**Benefits:**

- Instant updates (no 30-second polling delay)
- Reduced server load (no repeated polling requests)
- Better UX (real-time notifications)

---

### Monitoring and Observability

#### Key Metrics

**System Metrics:**

```text
- Latency (p50, p95, p99):
  - API Gateway: < 100ms p95
  - Feed Generation: < 500ms p95
  - Media Upload: < 2s p95

- Throughput:
  - Requests per second: 347,000/sec peak
  - Posts per second: 7,000/sec peak
  - Engagement actions per second: 220,000/sec peak

- Error Rates:
  - API errors: < 0.1%
  - Media processing failures: < 0.5%
  - Database errors: < 0.01%

- Resource Utilization:
  - CPU: < 70% average
  - Memory: < 80% average
  - Disk: < 75% used
  - Network: < 60% of capacity
```

**Business Metrics:**

```text
- Daily Active Users (DAU)
- Posts per day
- Engagement rate (likes + comments + shares / impressions)
- Feed load time
- Media upload success rate
- User retention (D1, D7, D30)
```

#### Alerting Strategy

**P0 Alerts (Page Immediately):**

```text
- API error rate > 1% for 5 minutes
- Database connection failures
- Payment system down
- Data loss detected
```

**P1 Alerts (Notify During Business Hours):**

```text
- Feed load time > 1s (p95)
- Media processing queue depth > 5000
- Cache hit rate < 80%
- Disk usage > 85%
```

**P2 Alerts (Weekly Review):**

```text
- Unusual traffic patterns
- Slow queries
- Cost anomalies
```

#### Logging

**Structured Logging Format:**

```json
{
  "timestamp": "2025-10-02T20:00:00Z",
  "level": "INFO",
  "service": "feed-service",
  "trace_id": "abc123",
  "user_id": "550e8400...",
  "action": "generate_feed",
  "duration_ms": 245,
  "result": "success",
  "metadata": {
    "posts_returned": 50,
    "cache_hit": true
  }
}
```

**Log Aggregation:**

- Tool: ELK Stack (Elasticsearch, Logstash, Kibana)
- Retention: 30 days (hot), 1 year (cold storage)
- Indexing: By service, timestamp, user_id, trace_id

#### Distributed Tracing

**Tool:** Jaeger or OpenTelemetry

**Trace Example:**

```text
User Request: Get Feed
├─ API Gateway (10ms)
├─ Auth Service (5ms)
├─ Feed Service (230ms)
│  ├─ Redis: Get Timeline (2ms)
│  ├─ Graph DB: Get Celebrities (15ms)
│  ├─ Post Service: Batch Fetch Metadata (50ms)
│  ├─ Ranking Service: ML Inference (150ms)
│  └─ Format Response (13ms)
└─ Response to Client (5ms)

Total: 245ms
```

---

## Extended Edge Cases & Failure Scenarios

### Edge Case 1: Concurrent Modifications

**Scenario:** Multiple users like/unlike the same post simultaneously

**Problem:**

```text
Time  User A          User B          Redis Counter
T0    Like post       -               counter = 100
T1    Read: 100       Like post       counter = 100
T2    Increment       Read: 100       counter = 100
T3    Write: 101      Increment       counter = 101
T4    -               Write: 101      counter = 101 (LOST UPDATE!)

Expected: 102, Actual: 101
```

#### Solution: Atomic Operations

```python
# Use Redis INCR for atomic increments
def like_post_atomic(post_id, user_id):
    # Check if user already liked (idempotency)
    if redis.sismember(f'post:likers:{post_id}', user_id):
        return {'already_liked': True}
    
    # Use Redis transaction for atomicity
    pipe = redis.pipeline()
    pipe.sadd(f'post:likers:{post_id}', user_id)  # Add to set
    pipe.incr(f'post:likes:{post_id}')  # Atomic increment
    pipe.execute()
    
    # Async: Update Cassandra
    kafka.publish('engagement.like', {
        'post_id': post_id,
        'user_id': user_id,
        'timestamp': now()
    })
    
    return {'success': True}
```

---

### Edge Case 2: Thundering Herd on Celebrity Post

**Scenario:** Celebrity posts, causing 1M+ concurrent feed requests for same content

**Problem:**

```text
Celebrity posts → 1M followers request feed simultaneously
→ Cache miss (post not yet cached)
→ 1M database queries for same data
→ Database overload
→ Cascading failures
```

#### Solution 1: Request Coalescing

```python
# Use distributed lock to ensure only one request fetches data
def get_post_with_coalescing(post_id):
    # Try to get from cache
    cached = redis.get(f'post:{post_id}')
    if cached:
        return cached
    
    # Try to acquire lock
    lock_key = f'lock:post:{post_id}'
    lock_acquired = redis.set(lock_key, '1', nx=True, ex=5)
    
    if lock_acquired:
        # This request fetches from DB
        post = cassandra.get_post(post_id)
        redis.setex(f'post:{post_id}', 3600, post)
        redis.delete(lock_key)
        return post
    else:
        # Wait for the other request to populate cache
        for i in range(10):  # Retry 10 times
            time.sleep(0.1)
            cached = redis.get(f'post:{post_id}')
            if cached:
                return cached
        
        # Fallback: fetch from DB
        return cassandra.get_post(post_id)
```

#### Solution 2: Pre-warming Cache

```python
@event_handler('post.created')
def prewarm_celebrity_post(post_id, author_id):
    user = get_user(author_id)
    
    if user.is_celebrity:
        # Pre-warm cache for celebrity posts
        post = get_post(post_id)
        redis.setex(f'post:{post_id}', 3600, post)
        
        # Pre-warm media CDN
        for media_url in post.media_urls:
            cdn.prewarm(media_url, regions=['all'])
```

---

### Edge Case 3: Split-Brain Scenario

**Scenario:** Network partition causes two data centers to operate independently

**Problem:**

```text
Region A                    Region B
User follows @celebrity     User unfollows @celebrity
(network partition occurs)
Both regions think they're primary
Writes to both regions
→ Data inconsistency when network heals
```

#### Solution: Multi-Region Consistency

```python
# Use consensus protocol (Raft/Paxos) for critical operations
class DistributedFollowService:
    def follow(self, follower_id, following_id):
        # Attempt to get consensus from majority of regions
        regions = ['us-east', 'us-west', 'eu-west']
        votes = []
        
        # Phase 1: Propose
        for region in regions:
            vote = region_client.propose_follow(
                follower_id, 
                following_id, 
                version=get_current_version()
            )
            votes.append(vote)
        
        # Phase 2: Commit if majority agrees
        if sum(votes) > len(regions) / 2:
            for region in regions:
                region_client.commit_follow(follower_id, following_id)
            return {'success': True}
        else:
            # Rollback
            for region in regions:
                region_client.abort_follow(follower_id, following_id)
            raise ConsensusFailedError()
```

#### Alternative: Last-Write-Wins with Vector Clocks

```python
class FollowRelationship:
    def __init__(self):
        self.follower_id = None
        self.following_id = None
        self.status = 'active'  # or 'inactive'
        self.vector_clock = {}  # {region: timestamp}
        self.tombstone = False
    
    def compare(self, other):
        """Detect conflicts using vector clocks"""
        # If one dominates, use it
        if self.dominates(other):
            return self
        elif other.dominates(self):
            return other
        else:
            # Conflict! Use application-level resolution
            # Rule: Unfollow wins over follow (conservative)
            if self.status == 'inactive' or other.status == 'inactive':
                return self if self.status == 'inactive' else other
            else:
                # Use timestamp as tiebreaker
                return self if self.get_latest_timestamp() > other.get_latest_timestamp() else other
```

---

### Edge Case 4: Cascading Failures

**Scenario:** Database overload causes feed service to slow down, leading to queue buildup

**Problem:**

```text
Database slow → Feed service timeout
→ Requests retry → More load on database
→ Queue depth increases → Memory exhaustion
→ Entire system failure
```

#### Solution: Circuit Breaker Pattern

```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN
        self.last_failure_time = None
    
    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            # Check if timeout has passed
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise CircuitBreakerOpenError('Circuit breaker is OPEN')
        
        try:
            result = func(*args, **kwargs)
            
            if self.state == 'HALF_OPEN':
                # Success in half-open state → close circuit
                self.state = 'CLOSED'
                self.failure_count = 0
            
            return result
        
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            
            if self.failure_count >= self.failure_threshold:
                self.state = 'OPEN'
                logger.error(f'Circuit breaker opened after {self.failure_count} failures')
            
            raise e

# Usage
feed_circuit_breaker = CircuitBreaker(failure_threshold=5, timeout=60)

def get_feed_with_circuit_breaker(user_id):
    try:
        return feed_circuit_breaker.call(feed_service.get_feed, user_id)
    except CircuitBreakerOpenError:
        # Return cached feed or error
        return get_fallback_feed(user_id)
```

#### Bulkhead Pattern for Isolation

```python
# Separate thread pools for different operations
feed_executor = ThreadPoolExecutor(max_workers=100, queue_size=1000)
post_executor = ThreadPoolExecutor(max_workers=50, queue_size=500)
engagement_executor = ThreadPoolExecutor(max_workers=200, queue_size=2000)

# Feed failures don't affect post creation
def handle_feed_request(user_id):
    future = feed_executor.submit(get_feed, user_id)
    try:
        return future.result(timeout=2.0)
    except TimeoutError:
        return get_fallback_feed(user_id)
```

---

### Edge Case 5: Duplicate Post Creation

**Scenario:** Network timeout causes client to retry, creating duplicate posts

**Problem:**

```text
T0: User creates post
T1: Request sent to server
T2: Server processes, but response times out
T3: Client retries (thinks request failed)
T4: Server processes again → Duplicate post!
```

#### Solution: Idempotency with Request IDs

```python
def create_post_idempotent(post_data, request_id):
    """
    Idempotent post creation using request ID
    """
    # Check if we've already processed this request
    existing = redis.get(f'idempotency:{request_id}')
    if existing:
        return json.loads(existing)
    
    # Acquire distributed lock
    lock = redis.set(
        f'idempotency:lock:{request_id}', 
        '1', 
        nx=True, 
        ex=60
    )
    
    if not lock:
        # Another request is processing this
        time.sleep(0.5)
        existing = redis.get(f'idempotency:{request_id}')
        if existing:
            return json.loads(existing)
        raise ConcurrentRequestError()
    
    try:
        # Create post
        post = create_post(post_data)
        
        # Store result for future duplicate requests
        redis.setex(
            f'idempotency:{request_id}',
            3600,  # 1-hour TTL
            json.dumps(post)
        )
        
        return post
    finally:
        redis.delete(f'idempotency:lock:{request_id}')

# Client-side request ID generation
request_id = f'{user_id}:{uuid4()}:{timestamp()}'
```

---

### Edge Case 6: Infinite Loop in Feed (Corrupted Data)

**Scenario:** Pagination cursor gets corrupted, causing infinite loop

**Problem:**

```text
User requests feed with cursor "abc123"
→ Server decodes cursor, fetches next 50 posts
→ Returns same cursor "abc123" due to bug
→ Client requests again with "abc123"
→ Infinite loop, same posts returned forever
```

#### Solution: Cursor Validation & Versioning

```python
def encode_cursor(last_post_id, timestamp, version=1):
    """
    Encode cursor with version and checksum
    """
    data = {
        'version': version,
        'last_post_id': last_post_id,
        'timestamp': timestamp,
        'nonce': random.randint(0, 1000000)
    }
    
    # Create checksum
    checksum = hashlib.sha256(
        json.dumps(data, sort_keys=True).encode()
    ).hexdigest()[:8]
    
    data['checksum'] = checksum
    
    # Base64 encode
    return base64.b64encode(
        json.dumps(data).encode()
    ).decode()

def decode_cursor(cursor_str):
    """
    Decode and validate cursor
    """
    try:
        data = json.loads(base64.b64decode(cursor_str))
        
        # Validate checksum
        checksum = data.pop('checksum')
        expected_checksum = hashlib.sha256(
            json.dumps(data, sort_keys=True).encode()
        ).hexdigest()[:8]
        
        if checksum != expected_checksum:
            raise InvalidCursorError('Checksum mismatch')
        
        # Validate version
        if data['version'] != 1:
            raise InvalidCursorError('Unsupported cursor version')
        
        # Validate timestamp (not too old)
        age = time.time() - data['timestamp']
        if age > 86400:  # 24 hours
            raise InvalidCursorError('Cursor expired')
        
        return data
    
    except Exception as e:
        logger.error(f'Invalid cursor: {cursor_str}, error: {e}')
        raise InvalidCursorError('Invalid cursor format')

def get_feed_with_safe_cursor(user_id, cursor=None):
    if cursor:
        try:
            cursor_data = decode_cursor(cursor)
            last_post_id = cursor_data['last_post_id']
        except InvalidCursorError:
            # Reset to beginning
            last_post_id = None
    else:
        last_post_id = None
    
    # Fetch posts
    posts = fetch_posts(user_id, after=last_post_id, limit=50)
    
    # Generate new cursor
    if posts:
        new_cursor = encode_cursor(
            posts[-1].post_id,
            time.time()
        )
    else:
        new_cursor = None
    
    return {
        'posts': posts,
        'next_cursor': new_cursor,
        'has_more': len(posts) == 50
    }
```

---

## Disaster Recovery & Business Continuity

### Recovery Time Objective (RTO) & Recovery Point Objective (RPO)

**Service Tiers:**

```text
Tier 1 (Critical - Feed, Login):
- RTO: 5 minutes
- RPO: 0 minutes (no data loss)
- Strategy: Active-active multi-region

Tier 2 (Important - Post Creation, Upload):
- RTO: 15 minutes
- RPO: 1 minute
- Strategy: Active-passive with fast failover

Tier 3 (Standard - Analytics, Recommendations):
- RTO: 1 hour
- RPO: 15 minutes
- Strategy: Backup and restore
```

### Multi-Region Failover Strategy

**Architecture:**

```text
Primary Region (US-East)          Secondary Region (US-West)
─────────────────────────          ──────────────────────────
┌─────────────────────┐            ┌─────────────────────┐
│  API Gateway        │◄─────────►│  API Gateway        │
│  (Active)           │   Sync    │  (Standby)          │
└─────────────────────┘            └─────────────────────┘
         │                                  │
         ▼                                  ▼
┌─────────────────────┐            ┌─────────────────────┐
│  Database           │            │  Database           │
│  (Master)           │───Async───►│  (Replica)          │
│                     │ Replication│                     │
└─────────────────────┘            └─────────────────────┘

Health Check System:
- Ping every 10 seconds
- 3 consecutive failures = trigger failover
- DNS update (TTL: 60 seconds)
- Promote replica to master
```

**Failover Implementation:**

```python
class RegionFailoverManager:
    def __init__(self):
        self.primary_region = 'us-east-1'
        self.secondary_region = 'us-west-2'
        self.health_check_interval = 10  # seconds
        self.failure_threshold = 3
        self.failure_count = 0
    
    def monitor_health(self):
        """Continuous health monitoring"""
        while True:
            if not self.check_primary_health():
                self.failure_count += 1
                logger.warning(f'Primary region health check failed: {self.failure_count}/{self.failure_threshold}')
                
                if self.failure_count >= self.failure_threshold:
                    self.initiate_failover()
            else:
                self.failure_count = 0
            
            time.sleep(self.health_check_interval)
    
    def check_primary_health(self):
        """Health check for primary region"""
        try:
            # Check API gateway
            api_response = requests.get(
                f'https://{self.primary_region}.api.socialmedia.com/health',
                timeout=5
            )
            
            # Check database
            db_response = self.check_database_health(self.primary_region)
            
            # Check cache
            cache_response = self.check_cache_health(self.primary_region)
            
            return all([
                api_response.status_code == 200,
                db_response,
                cache_response
            ])
        
        except Exception as e:
            logger.error(f'Health check failed: {e}')
            return False
    
    def initiate_failover(self):
        """Execute failover to secondary region"""
        logger.critical(f'Initiating failover from {self.primary_region} to {self.secondary_region}')
        
        # Step 1: Stop accepting writes in primary (if accessible)
        try:
            self.set_read_only_mode(self.primary_region)
        except:
            pass  # Primary might be completely down
        
        # Step 2: Promote secondary database to master
        self.promote_replica_to_master(self.secondary_region)
        
        # Step 3: Update DNS to point to secondary
        self.update_dns(self.secondary_region)
        
        # Step 4: Update service registry
        self.update_service_registry(self.secondary_region)
        
        # Step 5: Notify ops team
        self.send_alert('CRITICAL: Failover completed to ' + self.secondary_region)
        
        # Step 6: Update primary region status
        self.primary_region, self.secondary_region = self.secondary_region, self.primary_region
        self.failure_count = 0
    
    def promote_replica_to_master(self, region):
        """Promote read replica to master"""
        # PostgreSQL promotion
        postgres_client.execute(f"""
            SELECT pg_promote(
                server_name := '{region}-replica'
            );
        """)
        
        # Update application config
        config.set('database.master', f'{region}-db.internal')
        
        logger.info(f'Promoted {region} replica to master')
```

### Backup and Restore Procedures

**Backup Strategy:**

```python
class BackupManager:
    def __init__(self):
        self.s3_bucket = 'socialmedia-backups'
        self.retention_days = {
            'daily': 7,
            'weekly': 30,
            'monthly': 365
        }
    
    def backup_user_database(self):
        """Backup PostgreSQL user database"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_file = f'user_db_backup_{timestamp}.sql'
        
        # Create backup
        subprocess.run([
            'pg_dump',
            '-h', 'user-db.internal',
            '-U', 'postgres',
            '-F', 'c',  # Custom format for parallel restore
            '-f', f'/tmp/{backup_file}',
            'users_db'
        ])
        
        # Compress
        subprocess.run(['gzip', f'/tmp/{backup_file}'])
        
        # Upload to S3 with encryption
        s3.upload_file(
            f'/tmp/{backup_file}.gz',
            self.s3_bucket,
            f'postgresql/daily/{backup_file}.gz',
            ExtraArgs={'ServerSideEncryption': 'AES256'}
        )
        
        # Verify backup
        self.verify_backup(f'postgresql/daily/{backup_file}.gz')
        
        # Clean up old backups
        self.cleanup_old_backups('postgresql/daily', self.retention_days['daily'])
    
    def backup_cassandra_keyspace(self):
        """Backup Cassandra keyspace"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        
        # Take snapshot
        subprocess.run([
            'nodetool',
            'snapshot',
            '-t', timestamp,
            'socialmedia_keyspace'
        ])
        
        # Copy snapshot to S3
        snapshot_dir = f'/var/lib/cassandra/data/socialmedia_keyspace/*/snapshots/{timestamp}'
        
        for file in glob.glob(f'{snapshot_dir}/*'):
            s3.upload_file(
                file,
                self.s3_bucket,
                f'cassandra/daily/{timestamp}/{os.path.basename(file)}'
            )
    
    def restore_from_backup(self, backup_type, backup_date):
        """Restore database from backup"""
        logger.critical(f'Starting restore from {backup_type} backup: {backup_date}')
        
        if backup_type == 'postgresql':
            self.restore_postgresql(backup_date)
        elif backup_type == 'cassandra':
            self.restore_cassandra(backup_date)
        
        # Verify data integrity
        self.verify_data_integrity()
        
        logger.info('Restore completed successfully')
    
    def restore_postgresql(self, backup_date):
        """Restore PostgreSQL from backup"""
        backup_file = f'user_db_backup_{backup_date}.sql.gz'
        
        # Download from S3
        s3.download_file(
            self.s3_bucket,
            f'postgresql/daily/{backup_file}',
            f'/tmp/{backup_file}'
        )
        
        # Decompress
        subprocess.run(['gunzip', f'/tmp/{backup_file}'])
        
        # Restore (parallel restore for speed)
        subprocess.run([
            'pg_restore',
            '-h', 'user-db.internal',
            '-U', 'postgres',
            '-d', 'users_db',
            '-j', '8',  # 8 parallel jobs
            f'/tmp/{backup_file[:-3]}'
        ])
```

---

## Deployment Strategy

### Blue-Green Deployment

**Process:**

```text
Current State:
┌─────────────────┐
│   Blue (Live)   │ ◄── 100% traffic
│   Version 1.0   │
└─────────────────┘
┌─────────────────┐
│  Green (Idle)   │
│                 │
└─────────────────┘

Step 1: Deploy to Green:
┌─────────────────┐
│   Blue (Live)   │ ◄── 100% traffic
│   Version 1.0   │
└─────────────────┘
┌─────────────────┐
│  Green (Ready)  │ ◄── Test traffic
│   Version 1.1   │
└─────────────────┘

Step 2: Switch traffic:
┌─────────────────┐
│   Blue (Idle)   │
│   Version 1.0   │
└─────────────────┘
┌─────────────────┐
│  Green (Live)   │ ◄── 100% traffic
│   Version 1.1   │
└─────────────────┘

Step 3: Rollback if needed:
(Switch back to Blue immediately)
```

**Implementation:**

```yaml
# Kubernetes Blue-Green Deployment
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
    version: blue  # Change to 'green' to switch
  ports:
    - port: 80
      targetPort: 8080

---
# Blue deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-blue
spec:
  replicas: 10
  selector:
    matchLabels:
      app: api
      version: blue
  template:
    metadata:
      labels:
        app: api
        version: blue
    spec:
      containers:
      - name: api
        image: api:1.0
        
---
# Green deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-green
spec:
  replicas: 10
  selector:
    matchLabels:
      app: api
      version: green
  template:
    metadata:
      labels:
        app: api
        version: green
    spec:
      containers:
      - name: api
        image: api:1.1
```

```python
# Automated deployment script
class BlueGreenDeployment:
    def deploy(self, new_version):
        # Step 1: Identify current live environment
        current_env = self.get_live_environment()  # 'blue' or 'green'
        target_env = 'green' if current_env == 'blue' else 'blue'
        
        logger.info(f'Deploying {new_version} to {target_env}')
        
        # Step 2: Deploy to target environment
        self.deploy_to_environment(target_env, new_version)
        
        # Step 3: Run smoke tests
        if not self.run_smoke_tests(target_env):
            logger.error('Smoke tests failed, aborting deployment')
            return False
        
        # Step 4: Switch small percentage of traffic (canary)
        self.route_traffic(current_env, 95, target_env, 5)
        time.sleep(300)  # Monitor for 5 minutes
        
        # Step 5: Check metrics
        if not self.check_metrics(target_env):
            logger.error('Metrics degraded, rolling back')
            self.route_traffic(current_env, 100, target_env, 0)
            return False
        
        # Step 6: Switch all traffic
        self.route_traffic(current_env, 0, target_env, 100)
        
        logger.info(f'Deployment successful, {target_env} is now live')
        return True
```

### Canary Releases

**Process:**

```text
Phase 1: Deploy to 1% of users
Monitor: Error rate, latency, business metrics
Duration: 30 minutes

Phase 2: Increase to 10%
Monitor: Same metrics
Duration: 1 hour

Phase 3: Increase to 50%
Monitor: Same metrics
Duration: 2 hours

Phase 4: Rollout to 100%
Continue monitoring
```

**Implementation:**

```python
class CanaryDeployment:
    def __init__(self):
        self.phases = [
            {'percentage': 1, 'duration': 1800},   # 1% for 30 min
            {'percentage': 10, 'duration': 3600},  # 10% for 1 hour
            {'percentage': 50, 'duration': 7200},  # 50% for 2 hours
            {'percentage': 100, 'duration': 0}     # 100%
        ]
    
    def deploy_canary(self, new_version):
        for phase in self.phases:
            logger.info(f'Canary phase: {phase["percentage"]}% traffic')
            
            # Update traffic split
            self.set_traffic_split(
                stable_version='1.0',
                stable_percentage=100 - phase['percentage'],
                canary_version=new_version,
                canary_percentage=phase['percentage']
            )
            
            # Monitor for duration
            start_time = time.time()
            while time.time() - start_time < phase['duration']:
                # Check metrics every minute
                if not self.check_canary_health(new_version):
                    logger.error('Canary unhealthy, rolling back')
                    self.rollback()
                    return False
                
                time.sleep(60)
        
        logger.info('Canary deployment successful')
        return True
    
    def check_canary_health(self, canary_version):
        """Compare canary metrics vs stable"""
        stable_metrics = self.get_metrics('1.0')
        canary_metrics = self.get_metrics(canary_version)
        
        # Error rate should not increase by more than 50%
        if canary_metrics['error_rate'] > stable_metrics['error_rate'] * 1.5:
            return False
        
        # Latency should not increase by more than 20%
        if canary_metrics['p95_latency'] > stable_metrics['p95_latency'] * 1.2:
            return False
        
        # CPU usage should not increase by more than 30%
        if canary_metrics['cpu_usage'] > stable_metrics['cpu_usage'] * 1.3:
            return False
        
        return True
```

### Database Migration Strategy

**Zero-Downtime Schema Changes:**

```python
class DatabaseMigration:
    """
    Safe database migrations with zero downtime
    """
    
    def add_column_safe(self, table, column, default):
        """
        Add column without locking table
        
        Step 1: Add column as nullable
        Step 2: Backfill data in batches
        Step 3: Add NOT NULL constraint
        Step 4: Update application code
        """
        
        # Step 1: Add nullable column
        self.db.execute(f"""
            ALTER TABLE {table}
            ADD COLUMN {column} VARCHAR(255) DEFAULT NULL;
        """)
        
        # Step 2: Backfill in batches (avoid long locks)
        offset = 0
        batch_size = 10000
        
        while True:
            affected = self.db.execute(f"""
                UPDATE {table}
                SET {column} = {default}
                WHERE {column} IS NULL
                LIMIT {batch_size};
            """)
            
            if affected == 0:
                break
            
            # Sleep to avoid overwhelming database
            time.sleep(1)
        
        # Step 3: Add NOT NULL constraint
        self.db.execute(f"""
            ALTER TABLE {table}
            ALTER COLUMN {column} SET NOT NULL;
        """)
    
    def rename_column_safe(self, table, old_column, new_column):
        """
        Rename column with backward compatibility
        
        Step 1: Add new column
        Step 2: Dual-write to both columns
        Step 3: Backfill new column
        Step 4: Update application to read from new column
        Step 5: Remove old column
        """
        
        # Step 1: Add new column
        self.db.execute(f"""
            ALTER TABLE {table}
            ADD COLUMN {new_column} VARCHAR(255);
        """)
        
        # Step 2: Create trigger for dual-write
        self.db.execute(f"""
            CREATE TRIGGER sync_{old_column}_to_{new_column}
            BEFORE INSERT OR UPDATE ON {table}
            FOR EACH ROW
            EXECUTE FUNCTION copy_column('{old_column}', '{new_column}');
        """)
        
        # Step 3: Backfill
        self.backfill_column(table, old_column, new_column)
        
        # Step 4 & 5 done in separate deployment
```

---

## Testing Strategy

### Load Testing

**Test Scenarios:**

```python
# Locust load test
from locust import HttpUser, task, between

class SocialMediaUser(HttpUser):
    wait_time = between(1, 5)
    
    def on_start(self):
        """Login before starting tests"""
        response = self.client.post('/api/v1/auth/login', json={
            'email': f'user{random.randint(1, 1000000)}@test.com',
            'password': 'testpass123'
        })
        self.token = response.json()['access_token']
    
    @task(10)  # Weight: 10 (most common)
    def view_feed(self):
        self.client.get(
            '/api/v1/feed/home',
            headers={'Authorization': f'Bearer {self.token}'}
        )
    
    @task(5)
    def view_post(self):
        post_id = random.choice(self.popular_posts)
        self.client.get(
            f'/api/v1/posts/{post_id}',
            headers={'Authorization': f'Bearer {self.token}'}
        )
    
    @task(2)
    def like_post(self):
        post_id = random.choice(self.popular_posts)
        self.client.post(
            f'/api/v1/posts/{post_id}/like',
            headers={'Authorization': f'Bearer {self.token}'}
        )
    
    @task(1)
    def create_post(self):
        self.client.post(
            '/api/v1/posts',
            headers={'Authorization': f'Bearer {self.token}'},
            json={
                'caption': 'Load test post',
                'media_urls': ['https://test.com/image.jpg']
            }
        )

# Run test: locust -f load_test.py --users 100000 --spawn-rate 1000
```

**Performance Targets:**

```text
Load Test Results (Target vs Actual):

Feed Load Time:
- Target: p95 < 500ms
- Actual: p95 = 425ms ✓
- Actual: p99 = 850ms ✗ (needs optimization)

Post Creation:
- Target: p95 < 2s
- Actual: p95 = 1.2s ✓

Like Action:
- Target: p95 < 100ms
- Actual: p95 = 45ms ✓

Throughput:
- Target: 347,000 req/sec
- Actual: 312,000 req/sec ✗ (needs horizontal scaling)

Error Rate:
- Target: < 0.1%
- Actual: 0.08% ✓
```

### Chaos Engineering

**Chaos Experiments:**

```python
# Using Chaos Monkey
class ChaosExperiments:
    def terminate_random_instance(self):
        """Kill random API server instance"""
        instances = ec2.describe_instances(
            Filters=[{'Name': 'tag:Service', 'Values': ['api-server']}]
        )
        
        random_instance = random.choice(instances)
        logger.info(f'Terminating instance: {random_instance.id}')
        
        ec2.terminate_instances(InstanceIds=[random_instance.id])
        
        # Monitor: System should auto-heal within 2 minutes
        time.sleep(120)
        assert self.check_service_health(), 'System did not recover'
    
    def inject_network_latency(self):
        """Add 500ms latency to database connections"""
        for instance in get_api_instances():
            # Use tc (traffic control) to add latency
            ssh_execute(instance, """
                sudo tc qdisc add dev eth0 root netem delay 500ms 50ms
            """)
        
        # Monitor: p95 latency should increase but stay < 2s
        time.sleep(300)
        metrics = get_latency_metrics()
        assert metrics['p95'] < 2000, 'Latency exceeded threshold'
        
        # Cleanup
        for instance in get_api_instances():
            ssh_execute(instance, "sudo tc qdisc del dev eth0 root")
    
    def fill_disk_space(self):
        """Fill disk to 95% capacity"""
        instance = random.choice(get_api_instances())
        
        ssh_execute(instance, """
            fallocate -l 10G /tmp/fill_disk.img
        """)
        
        # Monitor: Should trigger disk space alert
        time.sleep(60)
        assert check_alert_fired('disk_space_critical'), 'Alert not triggered'
        
        # Cleanup should happen automatically
        time.sleep(300)
        disk_usage = get_disk_usage(instance)
        assert disk_usage < 80, 'Automatic cleanup did not occur'
```

---

## Advanced Optimization Techniques

### Connection Pooling Details

**Database Connection Pooling:**

```python
from sqlalchemy.pool import QueuePool

# Connection pool configuration
engine = create_engine(
    'postgresql://user:pass@db.internal/socialmedia',
    poolclass=QueuePool,
    pool_size=20,  # Min connections
    max_overflow=80,  # Max additional connections
    pool_timeout=30,  # Wait timeout for connection
    pool_recycle=3600,  # Recycle connections after 1 hour
    pool_pre_ping=True,  # Verify connection health before use
    echo_pool=True  # Log pool operations
)

class ConnectionPoolManager:
    def __init__(self):
        self.pools = {}  # db_name -> pool
        self.pool_stats = {}
    
    def get_connection(self, db_name='primary'):
        """Get connection from pool"""
        pool = self.pools.get(db_name)
        
        if not pool:
            raise ValueError(f'No pool for database: {db_name}')
        
        # Get connection
        conn = pool.get()
        
        # Track stats
        self.pool_stats[db_name]['active_connections'] += 1
        
        return conn
    
    def return_connection(self, db_name, conn):
        """Return connection to pool"""
        pool = self.pools[db_name]
        pool.return_connection(conn)
        
        self.pool_stats[db_name]['active_connections'] -= 1
    
    def get_pool_stats(self):
        """Get pool statistics"""
        stats = {}
        
        for db_name, pool in self.pools.items():
            stats[db_name] = {
                'size': pool.size(),
                'checked_out': pool.checked_out(),
                'overflow': pool.overflow(),
                'queue_size': pool.queue_size()
            }
        
        return stats
    
    def monitor_pools(self):
        """Monitor pool health"""
        for db_name, pool in self.pools.items():
            stats = {
                'checked_out': pool.checked_out(),
                'overflow': pool.overflow()
            }
            
            # Alert if pool is exhausted
            if stats['overflow'] >= pool.max_overflow * 0.9:
                logger.warning(f'Connection pool {db_name} near exhaustion')
                alert_ops_team(f'Database connection pool {db_name} at 90% capacity')
```

### Query Optimization Examples

**Problem Query:**

```sql
-- Slow query: Sequential scan, no indexes used
SELECT p.*, u.username, u.profile_picture_url
FROM posts p
JOIN users u ON p.user_id = u.user_id
WHERE u.user_id IN (
    SELECT following_id
    FROM follows
    WHERE follower_id = '123'
)
AND p.created_at > NOW() - INTERVAL '7 days'
ORDER BY p.created_at DESC
LIMIT 50;

-- Query plan shows:
-- Seq Scan on posts (cost=10000..50000)
-- Hash Join (cost=5000..10000)
-- Execution time: 2500ms
```

**Optimized Query:**

```sql
-- Add indexes
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);
CREATE INDEX idx_follows_follower ON follows(follower_id, following_id);

-- Optimized query with materialized CTE
WITH following_users AS MATERIALIZED (
    SELECT following_id
    FROM follows
    WHERE follower_id = '123'
)
SELECT p.post_id, p.caption, p.created_at,
       u.username, u.profile_picture_url
FROM posts p
INNER JOIN following_users f ON p.user_id = f.following_id
INNER JOIN users u ON p.user_id = u.user_id
WHERE p.created_at > NOW() - INTERVAL '7 days'
ORDER BY p.created_at DESC
LIMIT 50;

-- Query plan shows:
-- Index Scan using idx_posts_user_created
-- Execution time: 45ms (55x faster!)
```

**Query Result Caching:**

```python
from functools import lru_cache
import hashlib

class QueryCache:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.ttl = 300  # 5 minutes
    
    def cache_key(self, query, params):
        """Generate cache key from query and parameters"""
        key_data = f"{query}:{json.dumps(params, sort_keys=True)}"
        return f"query_cache:{hashlib.md5(key_data.encode()).hexdigest()}"
    
    def execute_with_cache(self, query, params):
        """Execute query with caching"""
        cache_key = self.cache_key(query, params)
        
        # Try cache
        cached_result = self.redis.get(cache_key)
        if cached_result:
            return json.loads(cached_result)
        
        # Execute query
        result = db.execute(query, params)
        
        # Cache result
        self.redis.setex(
            cache_key,
            self.ttl,
            json.dumps(result, default=str)
        )
        
        return result
```

### Network Optimization

**HTTP/2 Server Push:**

```python
# Push related resources to client
@app.route('/api/v1/posts/<post_id>')
def get_post(post_id):
    post = fetch_post(post_id)
    
    # Server push media files
    for media in post.media:
        # Push thumbnail immediately
        flask.g.push_queue.append(media.thumbnail_url)
    
    # Push user profile picture
    flask.g.push_queue.append(post.author.profile_picture_url)
    
    return jsonify(post)
```

**Request Multiplexing:**

```python
# Batch multiple API requests into one
@app.route('/api/v1/batch', methods=['POST'])
def batch_requests():
    """
    Request:
    {
        "requests": [
            {"method": "GET", "path": "/posts/123"},
            {"method": "GET", "path": "/users/456"},
            {"method": "POST", "path": "/posts/123/like"}
        ]
    }
    """
    batch = request.json['requests']
    responses = []
    
    for req in batch:
        try:
            # Execute sub-request
            if req['method'] == 'GET':
                result = execute_get(req['path'])
            elif req['method'] == 'POST':
                result = execute_post(req['path'], req.get('body'))
            
            responses.append({
                'status': 200,
                'body': result
            })
        except Exception as e:
            responses.append({
                'status': 500,
                'error': str(e)
            })
    
    return jsonify({'responses': responses})
```

---

## Cost Analysis

### Infrastructure Cost Breakdown (Monthly)

**Compute:**

```text
API Servers:
- Instance type: m5.2xlarge (8 vCPU, 32 GB RAM)
- Count: 100 instances (across regions)
- Cost: $0.384/hour * 100 * 730 hours = $28,032/month

Media Processing Workers:
- Instance type: c5.4xlarge (16 vCPU, 32 GB RAM)
- Count: 1000 instances (auto-scaling)
- Average utilization: 60%
- Cost: $0.68/hour * 1000 * 0.6 * 730 = $297,840/month

Total Compute: ~$326,000/month
```

**Storage:**

```text
S3 Storage (Media):
- Storage: 12.6 EB over 5 years = ~7 PB active
- Cost: $0.023/GB * 7,000,000 GB = $161,000/month

S3 Data Transfer Out:
- Bandwidth: 16.5 Pbps peak (with 95% CDN offload)
- Actual S3 egress: 16.5 Pbps * 0.05 = 825 Tbps
- Monthly: 825 TB * 730 * 3600 / 8 / 1024 / 1024 = ~300 TB
- Cost: $0.09/GB * 300,000 GB = $27,000/month

Total Storage: ~$188,000/month
```

**Database:**

```text
PostgreSQL (RDS):
- Instance: db.r5.8xlarge
- Count: 3 (1 primary + 2 replicas)
- Storage: 3 TB * 3 = 9 TB
- Cost: $2.88/hour * 3 * 730 = $6,307/month
- Storage: $0.115/GB * 9000 = $1,035/month

Cassandra (EC2):
- Instance: i3.4xlarge
- Count: 100 nodes (sharded)
- Cost: $1.248/hour * 100 * 730 = $91,104/month

Redis Cluster:
- Instance: r5.4xlarge
- Count: 64 nodes (sharded)
- Cost: $1.008/hour * 64 * 730 = $47,093/month

Total Database: ~$145,000/month
```

**CDN:**

```text
CloudFront:
- Data transfer: 10B impressions * 300 KB = 3 PB/day = 90 PB/month
- Cost: $0.085/GB * 90,000,000 GB = $7,650,000/month
- Cache hit ratio: 95%, so only 5% origin fetch
- Actual CDN cost (with volume discount): ~$300,000/month

Total CDN: ~$300,000/month
```

**Kafka (Message Queue):**

```text
- Instance: m5.2xlarge
- Count: 20 brokers
- Storage: 10 PB retention (7 days)
- Cost: $0.384/hour * 20 * 730 = $5,606/month
- Storage: $0.10/GB * 10,000,000 = $1,000,000/month

Total Kafka: ~$1,006,000/month (expensive! optimize retention)
```

**Total Monthly Cost: ~$1,965,000/month = $23.6M/year**

### Cost Optimization Strategies

```python
# Strategy 1: Right-size instances
def optimize_instance_sizes():
    """
    Monitor actual resource usage and downsize
    """
    for instance in get_all_instances():
        metrics = cloudwatch.get_metrics(instance.id, period='7d')
        
        avg_cpu = metrics['cpu']['average']
        avg_memory = metrics['memory']['average']
        
        # If consistently under-utilized, suggest smaller instance
        if avg_cpu < 30 and avg_memory < 40:
            smaller_instance = suggest_smaller_instance(instance.type)
            logger.info(f'Instance {instance.id} can be downsized to {smaller_instance}')
            # Estimated savings: $X/month

# Strategy 2: Use Spot Instances for workers
"""
Media processing workers can use Spot instances (70% cost savings)
- Stateless workloads
- Can handle interruptions
- Kafka queue provides durability

Savings: $297,840 * 0.7 = $208,488/month
"""

# Strategy 3: Optimize storage retention
"""
- Reduce Kafka retention from 7 days to 2 days
- Move old media to Glacier (90% cheaper)
- Compress media more aggressively

Estimated savings: $400,000/month
"""

# Strategy 4: CDN optimization
"""
- Increase cache TTL (reduce origin fetches)
- Use origin shield
- Negotiate volume discounts

Estimated savings: $50,000/month
"""

# Total potential savings: ~$650,000/month (33% reduction)
```

---

## SLA/SLO/SLI Definitions

### Service Level Indicators (SLIs)

**Availability SLI:**

```text
Definition: Percentage of successful requests

Measurement:
SLI = (successful_requests / total_requests) * 100

Example:
Total requests: 1,000,000
Successful (2xx, 3xx): 999,100
Failed (4xx, 5xx): 900
SLI = (999,100 / 1,000,000) * 100 = 99.91%
```

**Latency SLI:**

```text
Definition: Percentage of requests served within target latency

Measurement:
SLI = (requests_under_target_latency / total_requests) * 100

Example (target: 500ms):
Total requests: 1,000,000
Under 500ms: 950,000
SLI = (950,000 / 1,000,000) * 100 = 95%
```

### Service Level Objectives (SLOs)

**Availability SLO:**

```text
Objective: 99.9% of requests succeed (monthly)

Error Budget:
- Monthly requests: 1B
- Allowed failures: 1B * 0.001 = 1M failures
- Per day: 1M / 30 = 33,333 failures

Current Status (Example):
- Month-to-date requests: 500M
- Failed requests: 400K
- Remaining error budget: 600K failures
- Days remaining: 15
- Daily budget: 600K / 15 = 40K failures/day
```

**Latency SLO:**

```text
Objective: 95% of feed requests complete in <500ms (p95)

Measurement Window: 7 days rolling

Current Status:
- P50: 245ms ✓
- P95: 425ms ✓
- P99: 1,250ms ✗ (needs improvement)
```

### Service Level Agreements (SLAs)

**Customer-Facing SLA:**

```text
Availability SLA: 99.9% uptime

Calculation:
- Monthly uptime target: 99.9%
- Maximum downtime: 43.2 minutes/month

Penalties:
- 99.0-99.9% uptime: 10% service credit
- 95.0-99.0% uptime: 25% service credit
- <95.0% uptime: 50% service credit

Exclusions:
- Scheduled maintenance (with 7-day notice)
- Customer's own infrastructure issues
- Force majeure events
```

---

## Future Enhancements

### 1. Direct Messaging (DM)

**Architecture:**

- WebSocket-based real-time messaging
- Message storage in Cassandra (partition by conversation_id)
- End-to-end encryption (Signal Protocol)
- Read receipts and typing indicators
- Media sharing in DMs

**Implementation:**

```python
# Message schema
{
    'message_id': 'uuid',
    'conversation_id': 'uuid',
    'sender_id': 'uuid',
    'content': 'encrypted_blob',
    'media_url': 's3_url',
    'created_at': 'timestamp',
    'read_at': 'timestamp'
}

# WebSocket handler
@websocket.on_message
def handle_dm(message):
    recipient = message['recipient_id']
    
    # Store message
    db.insert('messages', message)
    
    # Send to recipient (if online)
    if is_user_online(recipient):
        websocket.send(recipient, message)
    else:
        # Queue for push notification
        push_notification(recipient, 'New message from {sender}')
```

**Estimated Resources:**

- 50M DM users (10% of DAU)
- 500M messages/day
- Storage: 500M * 1KB = 500GB/day = 180TB/year

---

### 2. Live Streaming

**Architecture:**

- WebRTC for peer-to-peer streaming
- Media servers for distribution (Wowza/Ant Media Server)
- Adaptive bitrate streaming
- Chat integration
- Live reactions

**Implementation:**

```python
# Live stream schema
{
    'stream_id': 'uuid',
    'user_id': 'uuid',
    'title': 'string',
    'status': 'live' | 'ended',
    'viewer_count': 'int',
    'started_at': 'timestamp',
    'ended_at': 'timestamp',
    'chat_enabled': 'boolean'
}

# Start stream
def start_live_stream(user_id, title):
    stream = create_stream(user_id, title)
    
    # Notify followers
    followers = get_followers(user_id)
    notify_followers(followers, f'{user.username} is live!')
    
    # Start media server
    media_server.start_stream(stream.stream_id)
    
    return stream
```

**Estimated Resources:**

- 100k concurrent streams at peak
- 10M viewers at peak
- Bandwidth: 10M * 2Mbps = 20Tbps

---

### 3. Stories Enhancements

**Features:**

- Interactive stickers (polls, questions, quizzes)
- Augmented reality (AR) filters
- Story replies (DM responses)
- Story highlights (save stories beyond 24 hours)
- Story analytics (view counts, engagement)

**Implementation:**

```python
# Interactive poll sticker
{
    'sticker_type': 'poll',
    'question': 'Favorite color?',
    'options': ['Red', 'Blue', 'Green'],
    'votes': {
        'Red': 42,
        'Blue': 35,
        'Green': 23
    }
}

# AR filter application
def apply_ar_filter(story_media, filter_id):
    # Use OpenCV/TensorFlow for face detection
    faces = detect_faces(story_media)
    
    # Apply filter (e.g., dog ears)
    filtered_media = apply_filter(story_media, faces, filter_id)
    
    return filtered_media
```

---

### 4. Recommendation System

**ML-Based Content Discovery:**

- Collaborative filtering (users like you also liked...)
- Content-based filtering (similar to posts you've liked)
- Hybrid approach
- Trending content boost

**Implementation:**

```python
# User embedding
user_embedding = embed_user_preferences(user_id)
# [interest_tech: 0.8, interest_sports: 0.3, ...]

# Post embedding
post_embedding = embed_post_content(post_id)
# [topic_tech: 0.9, topic_sports: 0.1, ...]

# Similarity score
similarity = cosine_similarity(user_embedding, post_embedding)

# Ranking
def recommend_posts(user_id, num_posts=50):
    # Candidate generation (narrow down from billions)
    candidates = get_candidate_posts(user_id)  # ~10k candidates
    
    # Scoring
    scores = []
    for post in candidates:
        score = calculate_relevance_score(post, user_id)
        scores.append((post, score))
    
    # Ranking
    ranked_posts = sorted(scores, key=lambda x: x[1], reverse=True)
    
    return ranked_posts[:num_posts]
```

**Model Training:**

- Training data: 100B engagement events
- Model: Two-tower neural network
- Features: 500+ features
- Training frequency: Daily
- Serving: TensorFlow Serving (5ms inference latency)

---

### 5. Ads Platform

**Sponsored Content:**

- Native ads (looks like regular posts)
- Targeting (demographics, interests, behavior)
- Auction-based pricing (CPM, CPC, CPA)
- Ad frequency capping
- Performance analytics

**Implementation:**

```python
def insert_ads_in_feed(feed_posts, user_id):
    """
    Insert ads every N organic posts
    """
    AD_FREQUENCY = 5  # Show ad every 5 posts
    
    # Fetch relevant ads
    ads = ad_targeting_service.get_ads(
        user_demographics=get_user_demographics(user_id),
        user_interests=get_user_interests(user_id),
        context='home_feed'
    )
    
    # Insert ads in feed
    feed_with_ads = []
    ad_index = 0
    for i, post in enumerate(feed_posts):
        feed_with_ads.append(post)
        
        if (i + 1) % AD_FREQUENCY == 0 and ad_index < len(ads):
            feed_with_ads.append({
                'type': 'ad',
                'content': ads[ad_index],
                'ad_id': ads[ad_index]['id']
            })
            ad_index += 1
    
    return feed_with_ads
```

**Revenue Model:**

- CPM (Cost Per Mille): $5-20 per 1000 impressions
- Estimated revenue: 10B impressions/day * $10 CPM = $100k/day = $36M/year

---

### 6. Analytics Dashboard

**User Analytics:**

- Profile views
- Follower growth over time
- Post reach and engagement
- Best time to post
- Audience demographics

**Content Creator Tools:**

- Detailed post analytics
- Story insights
- Hashtag performance
- Competitor analysis
- Engagement rate trends

**Implementation:**

```python
# Analytics schema (ClickHouse)
{
    'event_id': 'uuid',
    'user_id': 'uuid',
    'post_id': 'uuid',
    'event_type': 'view' | 'like' | 'comment' | 'share',
    'timestamp': 'datetime',
    'device_type': 'mobile' | 'web',
    'location': 'country_code'
}

# Analytics query
def get_post_analytics(post_id):
    return {
        'views': count_events(post_id, 'view'),
        'likes': count_events(post_id, 'like'),
        'comments': count_events(post_id, 'comment'),
        'shares': count_events(post_id, 'share'),
        'engagement_rate': (likes + comments + shares) / views,
        'demographics': get_viewer_demographics(post_id),
        'top_locations': get_top_locations(post_id)
    }
```

---

### 7. Machine Learning Enhancements

**Content Understanding:**

- Object detection in images
- Scene classification
- Auto-tagging
- Auto-captioning

**User Safety:**

- Cyberbullying detection
- Self-harm content detection
- Misinformation flagging
- Deepfake detection

**Personalization:**

- Feed ranking optimization
- Notification timing optimization
- Content format preferences
- Language translation

---

## Section 15: Interview Preparation & Practice

### What You'll Learn

By the end of this section, you'll be able to:

- Answer common system design interview questions for social media platforms
- Navigate different system variations (Instagram-like, Twitter-like, TikTok-like)
- Troubleshoot production issues during interviews with structured approaches
- Handle follow-up questions and deep dives confidently
- Demonstrate interview best practices and avoid common mistakes

### Why This Matters

System design interviews determine whether you get offers at FAANG companies, where senior engineers earn $300K-$500K+ annually. Real-world example: A candidate who could explain hybrid fan-out strategies and troubleshoot feed latency issues received offers from Meta ($350K) and Google ($380K)—the difference between passing and failing often comes down to interview execution, not just knowledge! This section teaches you how to showcase your system design skills effectively.

### 🟢 For Beginners: Common Interview Questions

#### Question 1: How would you design a social media platform like Instagram?

**What the interviewer is testing:**

- Can you gather requirements systematically?
- Do you understand scale challenges (500M users, 200M posts/day)?
- Can you design core components (feed generation, media processing)?
- Do you know when to ask clarifying questions?

**Step-by-Step Answer:**

**Step 1: Clarify Requirements (5 minutes)**

```text
Ask about:
├─ Scale: How many users? (Assume 500M DAU)
├─ Features: Posts, feeds, stories, messaging? (Start with MVP: posts + feeds)
├─ Media: Photos only or videos too? (Assume both)
├─ Geographic: Global or single region? (Assume global)
└─ Constraints: Latency? Availability? (Assume <500ms feed load, 99.9% uptime)
```

**Step 2: Capacity Planning (5 minutes)**

```text
Calculate:
├─ QPS: 200M posts/day = 2,315 posts/sec
├─ Feed requests: 500M users × 10 feeds/day = 57,870 req/sec
├─ Storage: 200M posts × 5MB = 1PB/day = 365PB/year
└─ Bandwidth: 10B impressions × 300KB = 3PB/day
```

**Step 3: High-Level Design (10 minutes)**

```text
Draw:
├─ Client (Mobile/Web)
├─ Load Balancer
├─ API Gateway
├─ Services:
│  ├─ User Service
│  ├─ Post Service
│  ├─ Feed Service
│  └─ Media Service
├─ Databases:
│  ├─ User DB (PostgreSQL)
│  ├─ Post DB (Cassandra)
│  └─ Feed Cache (Redis)
└─ Object Storage (S3) for media
```

**Step 4: Deep Dive (20 minutes)**

```text
Focus on:
├─ Feed Generation (hybrid fan-out)
│  ├─ Regular users: Fan-out on write
│  └─ Celebrities: Fan-out on read
├─ Media Processing Pipeline
│  ├─ Upload → Queue → Workers → Storage
│  └─ Multiple formats (thumbnail, medium, full)
└─ Database Sharding
   ├─ User DB: Shard by user_id
   └─ Post DB: Shard by user_id (Cassandra)
```

**Follow-up Questions You'll Get:**

1. **"How do you handle a celebrity with 100M followers posting?"**
   - Answer: Use fan-out on read—don't pre-compute, fetch on-demand when followers request feed

2. **"What if feed generation takes 2 seconds?"**
   - Answer: Add caching (Redis), pre-compute feeds for active users, use CDN for media

3. **"How do you scale media processing?"**
   - Answer: Horizontal scaling with workers, use spot instances (70% cost savings), process in parallel

4. **"What happens if a database shard fails?"**
   - Answer: Replication (3 replicas), automatic failover, promote replica to primary

5. **"How do you ensure feed freshness?"**
   - Answer: TTL-based cache (5 minutes), event-driven invalidation on new posts, real-time updates via WebSocket

### 🟡 For Intermediate: System Design Variations

#### Variation 1: Enterprise Social Network (LinkedIn-like)

**Unique Requirements:**

- Professional content focus (articles, job postings, company pages)
- Strong privacy controls (who viewed your profile)
- Search-heavy (find people, companies, jobs)
- Lower engagement frequency (users check weekly, not daily)
- Enterprise features (recruiting tools, sales navigator)

**Architecture Changes:**

```text
Key Modifications:
├─ Search Service (Elasticsearch)
│  ├─ Index: Users, Companies, Jobs, Posts
│  └─ Ranking: Relevance + Recency + Network strength
├─ Privacy Service
│  ├─ Track profile views (with privacy settings)
│  └─ Analytics dashboard (who viewed you)
├─ Feed Ranking
│  ├─ Professional relevance (industry, skills)
│  └─ Lower frequency (weekly digest option)
└─ Enterprise Features
   ├─ Recruiting tools (InMail, job postings)
   └─ Sales Navigator (lead generation)
```

**Interview Tip:** Emphasize search architecture (Elasticsearch), privacy engineering, and enterprise monetization features.

#### Variation 2: Photo-Focused Platform (Instagram-like)

**Unique Requirements:**

- Heavy media focus (photos, videos, stories)
- Visual discovery (explore page, hashtags)
- Stories (24-hour ephemeral content)
- Filters and editing tools
- Shopping integration

**Architecture Changes:**

```text
Key Modifications:
├─ Media Processing Pipeline (Critical)
│  ├─ Image optimization (multiple sizes)
│  ├─ Video transcoding (HLS for streaming)
│  └─ Thumbnail generation
├─ Explore Service
│  ├─ ML-powered recommendations
│  ├─ Trending hashtags
│  └─ Location-based discovery
├─ Stories Service
│  ├─ Ephemeral storage (24-hour TTL)
│  ├─ View tracking
│  └─ Expiration job (cleanup)
└─ Shopping Integration
   ├─ Product tagging
   └─ Checkout flow
```

**Interview Tip:** Deep dive into media processing pipeline, CDN strategy, and ML-powered discovery algorithms.

#### Variation 3: Short-Form Video Platform (TikTok-like)

**Unique Requirements:**

- Video-first (15-60 second clips)
- Infinite scroll feed (algorithm-driven)
- Real-time video processing
- Music integration (licensed tracks)
- Creator monetization

**Architecture Changes:**

```text
Key Modifications:
├─ Video Processing (Ultra-fast)
│  ├─ Real-time transcoding (<30 seconds)
│  ├─ Multiple resolutions (360p, 720p, 1080p)
│  └─ Thumbnail extraction
├─ Recommendation Engine (Core)
│  ├─ ML ranking (engagement prediction)
│  ├─ A/B testing framework
│  └─ Real-time personalization
├─ Music Service
│  ├─ Licensed music library
│  ├─ Audio extraction
│  └─ Copyright detection
└─ Creator Tools
   ├─ Analytics dashboard
   └─ Monetization (ads, tips)
```

**Interview Tip:** Focus on recommendation algorithm (collaborative filtering, content-based), video processing speed, and ML infrastructure.

#### Variation 4: Real-time Messaging Focus (Twitter-like)

**Unique Requirements:**

- Text-first (280-character limit)
- Real-time feed updates
- Trending topics
- Hashtag discovery
- Retweet and quote tweet mechanics

**Architecture Changes:**

```text
Key Modifications:
├─ Real-time Feed Updates
│  ├─ WebSocket for live updates
│  ├─ Redis Pub/Sub for broadcasting
│  └─ Push notifications (offline users)
├─ Trending Service
│  ├─ Real-time aggregation (sliding window)
│  ├─ Geographic trends
│  └─ Topic clustering
├─ Search Service
│  ├─ Full-text search (Elasticsearch)
│  ├─ Hashtag indexing
│  └─ User mentions
└─ Retweet Handling
   ├─ Reference to original tweet
   └─ Feed insertion (fan-out)
```

**Interview Tip:** Emphasize real-time architecture (WebSocket scaling), trending algorithm, and search infrastructure.

#### Variation 5: Professional Network (LinkedIn-like, Different Focus)

**Unique Requirements:**

- Long-form content (articles, posts)
- Company pages and job postings
- Messaging (InMail)
- Learning platform integration
- Analytics and insights

**Architecture Changes:**

```text
Key Modifications:
├─ Content Management
│  ├─ Rich text editor
│  ├─ Article publishing workflow
│  └─ Version control
├─ Company Pages
│  ├─ Hierarchical structure
│  ├─ Employee management
│  └─ Analytics dashboard
├─ Job Posting Service
│  ├─ Search and filtering
│  ├─ Application tracking
│  └─ Recommendation engine
└─ Learning Platform
   ├─ Course catalog
   ├─ Progress tracking
   └─ Certificate generation
```

**Interview Tip:** Discuss content management systems, search architecture for jobs, and enterprise features.

### 🔴 For Advanced: Production Troubleshooting Scenarios

#### Scenario 1: Feed Latency Spike (p95 > 1s)

**Interview Simulation:**

**Interviewer:** "Your feed service p95 latency jumped from 300ms to 1.2 seconds in the last hour. Users are complaining. How do you debug this?"

**Step 1: Gather Information**

```text
Ask:
├─ When did it start? (1 hour ago)
├─ Which endpoints affected? (Feed generation only)
├─ Error rate? (No errors, just slow)
├─ Traffic spike? (Normal traffic)
└─ Recent deployments? (No deployments)
```

**Step 2: Check Monitoring Dashboards**

```text
Investigate:
├─ Database latency: Normal (<50ms)
├─ Cache hit rate: Dropped from 85% to 45%
├─ Redis latency: Spiked to 200ms (normal: 5ms)
├─ Feed service CPU: 95% (normal: 60%)
└─ Queue depth: Media processing queue = 50K (normal: 5K)
```

**Step 3: Identify Root Cause**

```text
Analysis:
├─ Cache hit rate drop → More DB queries → Higher latency
├─ Redis latency spike → Cache itself is slow
├─ High CPU → Feed service overloaded
└─ Media queue backlog → Workers stuck processing

Root Cause: Media processing workers crashed, queue backing up,
            causing feed service to wait for media metadata
```

**Step 4: Immediate Mitigation**

```text
Actions:
├─ Scale up media workers (add 100 workers)
├─ Increase Redis capacity (add nodes)
├─ Add circuit breaker (fail fast if Redis slow)
└─ Route traffic away from affected region
```

**Step 5: Long-term Fix**

```text
Prevention:
├─ Health checks for workers (auto-restart)
├─ Queue depth alerts (alert at 10K)
├─ Redis monitoring (alert at 50ms latency)
└─ Load testing (test worker failures)
```

**Interview Tip:** Show systematic debugging (gather info → check metrics → identify cause → mitigate → prevent).

#### Scenario 2: Celebrity Post Causing System Overload

**Interview Simulation:**

**Interviewer:** "A celebrity with 50M followers just posted, and the system is struggling. Feed generation is failing for many users. What's happening and how do you fix it?"

**Step 1: Understand the Problem**

```text
Issue:
├─ Celebrity post → Fan-out on read triggered
├─ 50M followers requesting feed simultaneously
├─ Each request fetches celebrity's post
├─ Database overloaded (50M queries)
└─ Feed service timing out
```

**Step 2: Immediate Mitigation**

```text
Quick Fixes:
├─ Cache celebrity's post aggressively (Redis, 1-hour TTL)
├─ Rate limit feed requests (throttle to 10K/sec)
├─ Use CDN for celebrity content
└─ Batch requests (fetch multiple posts at once)
```

**Step 3: Long-term Solution**

```text
Architecture Fix:
├─ Pre-compute celebrity feeds (hybrid approach)
│  ├─ For top 1000 celebrities: Fan-out on write
│  └─ For others: Fan-out on read with caching
├─ Dedicated celebrity feed cache
│  ├─ Separate Redis cluster
│  └─ Pre-warm on post creation
└─ Load balancing
   ├─ Route celebrity requests to dedicated servers
   └─ Auto-scale based on traffic
```

**Interview Tip:** Demonstrate understanding of celebrity problem and hybrid fan-out solution.

#### Scenario 3: Media Processing Pipeline Backlog

**Interview Simulation:**

**Interviewer:** "Media processing queue has 100K items backlogged. Users are complaining their photos aren't appearing. How do you resolve this?"

**Step 1: Assess Impact**

```text
Current State:
├─ Queue depth: 100K items (normal: 5K)
├─ Processing rate: 100 items/sec (normal: 1000/sec)
├─ Worker count: 50 (normal: 500)
└─ Estimated time to clear: 16+ hours
```

**Step 2: Identify Root Cause**

```text
Investigation:
├─ Workers: 50 active (should be 500)
├─ Worker health: 450 workers crashed (OOM errors)
├─ Memory usage: Workers using 8GB each (limit: 4GB)
└─ Root cause: Large video files causing OOM
```

**Step 3: Immediate Actions**

```text
Fix:
├─ Restart crashed workers (auto-scaling)
├─ Scale up workers (500 → 1000 temporarily)
├─ Increase worker memory (4GB → 8GB)
├─ Prioritize queue (process photos first, videos later)
└─ Add circuit breaker (skip processing if memory >90%)
```

**Step 4: Long-term Prevention**

```text
Improvements:
├─ Video processing: Use spot instances (cheaper, can handle failures)
├─ Chunk processing: Split large videos into segments
├─ Queue monitoring: Alert at 10K depth
└─ Auto-scaling: Scale based on queue depth
```

**Interview Tip:** Show understanding of queue management, worker scaling, and failure handling.

#### Scenario 4: Database Shard Hot Partition

**Interview Simulation:**

**Interviewer:** "One database shard is handling 10x more traffic than others. Users in that shard are experiencing slow responses. How do you fix this?"

**Step 1: Identify Hot Shard**

```text
Analysis:
├─ Shard 7: 50K QPS (others: 5K QPS)
├─ Latency: 500ms (others: 50ms)
├─ CPU: 95% (others: 30%)
└─ Users affected: ~3M users (shard 7)
```

**Step 2: Understand Why**

```text
Investigation:
├─ Shard assignment: user_id % 16
├─ Shard 7 contains: Popular users (celebrities, influencers)
├─ Problem: Uneven distribution (hash collision)
└─ Solution: Re-shard or rebalance
```

**Step 3: Immediate Mitigation**

```text
Quick Fix:
├─ Add read replicas for shard 7 (5 replicas)
├─ Route reads to replicas (reduce primary load)
├─ Cache hot data aggressively (Redis)
└─ Add connection pooling (reduce connection overhead)
```

**Step 4: Long-term Solution**

```text
Re-sharding:
├─ Increase shard count (16 → 64 shards)
├─ Rebalance data (migrate users to new shards)
├─ Use consistent hashing (better distribution)
└─ Monitor shard distribution (alert on imbalance)
```

**Interview Tip:** Demonstrate understanding of sharding, hot partitions, and rebalancing strategies.

### Architecture Evolution: 1K → 1M → 100M Users

#### Stage 1: MVP (0 → 1,000 Users)

**Architecture:**

```text
Simple Monolith:
├─ Single server (t2.medium)
├─ PostgreSQL (single instance)
├─ Redis (single instance, caching)
├─ S3 (media storage)
└─ No CDN (direct S3 access)

Tech Stack:
├─ Backend: Node.js/Python (monolith)
├─ Database: PostgreSQL
├─ Cache: Redis
└─ Storage: S3

Cost: ~$200/month
Decisions: Keep it simple, optimize later
```

**Key Metrics:**

- Users: 1,000
- Posts/day: 1,000
- QPS: 10
- Storage: 5GB

#### Stage 2: Growth (1K → 100K Users)

**Architecture:**

```text
Separate Services:
├─ API Gateway (load balancer)
├─ Services:
│  ├─ User Service
│  ├─ Post Service
│  └─ Feed Service
├─ Database: PostgreSQL (read replicas)
├─ Cache: Redis Cluster (3 nodes)
├─ Media: S3 + CloudFront CDN
└─ Queue: RabbitMQ (media processing)

Tech Stack:
├─ Microservices (Docker containers)
├─ Database: PostgreSQL (1 primary + 2 replicas)
├─ Cache: Redis Cluster
├─ CDN: CloudFront
└─ Queue: RabbitMQ

Cost: ~$2,000/month
Decisions: Add caching, CDN, read replicas
```

**Key Metrics:**

- Users: 100,000
- Posts/day: 100,000
- QPS: 1,000
- Storage: 500GB

**Optimizations:**

- Read replicas (3x read capacity)
- CDN (90% cache hit rate)
- Caching (80% cache hit rate)
- Media processing queue (async)

#### Stage 3: Scale (100K → 1M Users)

**Architecture:**

```text
Distributed System:
├─ Multi-region deployment (US, EU)
├─ Services: Microservices (Kubernetes)
├─ Databases:
│  ├─ User DB: PostgreSQL (sharded, 16 shards)
│  ├─ Post DB: Cassandra (distributed)
│  └─ Graph DB: Neo4j (relationships)
├─ Cache: Redis Cluster (16 nodes)
├─ CDN: Multi-CDN (CloudFront + Fastly)
├─ Queue: Kafka (media processing)
└─ Monitoring: Prometheus + Grafana

Tech Stack:
├─ Orchestration: Kubernetes
├─ Databases: PostgreSQL (sharded), Cassandra, Neo4j
├─ Cache: Redis Cluster
├─ Message Queue: Kafka
└─ Monitoring: Prometheus

Cost: ~$20,000/month
Decisions: Sharding, distributed databases, Kafka
```

**Key Metrics:**

- Users: 1,000,000
- Posts/day: 1,000,000
- QPS: 10,000
- Storage: 5TB

**Optimizations:**

- Database sharding (horizontal scaling)
- Hybrid fan-out (celebrity problem)
- Multi-region (lower latency)
- Kafka (reliable message processing)

#### Stage 4: Enterprise (1M → 100M Users)

**Architecture:**

```text
Global Scale:
├─ Multi-region (US, EU, Asia, LatAm)
├─ Services: 100+ microservices
├─ Databases:
│  ├─ User DB: PostgreSQL (256 shards, multi-region)
│  ├─ Post DB: Cassandra (1000 nodes, global)
│  ├─ Graph DB: Neo4j (sharded, 64 shards)
│  └─ Search: Elasticsearch (100 nodes)
├─ Cache: Redis Cluster (500 nodes)
├─ CDN: Global CDN (100+ edge locations)
├─ Queue: Kafka (1000 brokers)
├─ ML: TensorFlow Serving (recommendations)
└─ Monitoring: Distributed tracing (Jaeger)

Tech Stack:
├─ Everything from Stage 3 +
├─ ML Infrastructure: TensorFlow Serving
├─ Search: Elasticsearch
├─ Tracing: Jaeger
└─ Auto-scaling: Kubernetes HPA

Cost: ~$2,000,000/month
Decisions: ML-powered ranking, global scale, SRE practices
```

**Key Metrics:**

- Users: 100,000,000
- Posts/day: 20,000,000
- QPS: 200,000
- Storage: 500TB

**Optimizations:**

- ML-powered feed ranking
- Global CDN (sub-100ms latency worldwide)
- Auto-scaling (handle traffic spikes)
- Advanced monitoring (SRE practices)

### Key Takeaways for Interviews

**Do's:**

- ✅ **Start with requirements**: Always clarify scale, features, constraints before designing
- ✅ **Show your math**: Calculate QPS, storage, bandwidth—interviewers love back-of-envelope calculations
- ✅ **Draw diagrams**: Visual architecture helps communicate your design
- ✅ **Discuss trade-offs**: Explain why you chose an approach and what you're giving up
- ✅ **Think out loud**: Explain your reasoning as you design
- ✅ **Ask questions**: Clarify ambiguous requirements—shows you think critically
- ✅ **Consider scale**: Design for growth (1M → 100M users)
- ✅ **Mention monitoring**: Always discuss how you'd monitor and debug the system
- ✅ **Discuss failures**: Explain how system handles failures (database down, cache miss, etc.)
- ✅ **Be collaborative**: Invite feedback and show you're easy to work with

**Don'ts:**

- ❌ **Jump to solutions**: Don't start coding or drawing before understanding requirements
- ❌ **Ignore scale**: Don't design for 1000 users when asked for millions
- ❌ **Forget caching**: Caching is crucial for performance—always mention it
- ❌ **Single point of failure**: Don't design systems with no redundancy
- ❌ **Over-engineer**: Don't add unnecessary complexity for MVP
- ❌ **Skip capacity planning**: Always calculate QPS, storage, bandwidth
- ❌ **Ignore consistency**: Discuss consistency models (strong vs eventual)
- ❌ **Forget security**: Mention authentication, authorization, encryption
- ❌ **No monitoring**: Always discuss how you'd monitor the system
- ❌ **Rush**: Take time to think—it's better to be thorough than fast

**Interview Checklist:**

**Before Interview:**

- [ ] Review system design fundamentals (caching, sharding, load balancing)
- [ ] Practice drawing architecture diagrams (5-minute sketches)
- [ ] Memorize key numbers (500M users, 200M posts/day, etc.)
- [ ] Review common patterns (fan-out, pub/sub, consistent hashing)
- [ ] Prepare clarifying questions framework

**During Interview:**

- [ ] Clarify requirements (5 minutes)
- [ ] Calculate capacity (5 minutes)
- [ ] Draw high-level design (10 minutes)
- [ ] Deep dive into 2-3 areas (20 minutes)
- [ ] Discuss trade-offs (10 minutes)
- [ ] Ask for feedback

**After Interview:**

- [ ] Reflect on what went well
- [ ] Note areas to improve
- [ ] Practice explaining concepts you struggled with

### Interview Strategy & Best Practices

**How to Approach Any System Design Interview:**

```text
Phase 1: Requirements Gathering (5 minutes)
├─ Functional requirements (what features?)
├─ Non-functional requirements (scale, latency, availability)
├─ Constraints (budget, team size, timeline)
└─ Assumptions (clarify ambiguous points)

Phase 2: Capacity Planning (5 minutes)
├─ Calculate QPS (queries per second)
├─ Estimate storage (data volume)
├─ Calculate bandwidth (network transfer)
└─ Show your math (back-of-envelope calculations)

Phase 3: High-Level Design (10 minutes)
├─ Draw architecture diagram
├─ Identify core components
├─ Show data flow (write path, read path)
└─ Explain component responsibilities

Phase 4: Deep Dive (20 minutes)
├─ Pick 2-3 areas to go deep
├─ Common choices:
│  ├─ Database design (sharding, replication)
│  ├─ Caching strategy (multi-layer)
│  ├─ Feed generation (fan-out strategies)
│  └─ Media processing (pipeline design)
├─ Discuss trade-offs
└─ Show technical depth

Phase 5: Trade-offs & Scalability (10 minutes)
├─ Discuss alternatives you considered
├─ Explain why you chose your approach
├─ Mention what you'd change at different scales
└─ Show context-aware thinking

Throughout: Communicate clearly!
├─ Think out loud
├─ Ask clarifying questions
├─ Invite feedback
└─ Show you're easy to work with
```

**Time Management Tips:**

1. **Don't rush requirements**: Spending 5 minutes clarifying saves 20 minutes of redesign
2. **Show calculations**: Even rough estimates show you think about scale
3. **Draw as you talk**: Visual diagrams help both you and the interviewer
4. **Save time for deep dive**: Leave 20 minutes for technical depth
5. **End with trade-offs**: Always discuss alternatives and scalability

### Interview Red Flags to Avoid

**Common Mistakes:**

| ❌ Wrong | ✅ Right |
|----------|---------|
| "I'll use microservices" (without reason) | "I'll start with monolith, then extract services when needed" |
| "Database can handle it" (no sharding) | "I'll shard by user_id when we hit 10M users" |
| "We'll cache everything" (no strategy) | "I'll cache hot data with TTL-based expiration" |
| "System will scale" (no details) | "I'll scale horizontally using load balancers and auto-scaling" |
| "No failures" (unrealistic) | "I'll handle failures with replication, circuit breakers, and graceful degradation" |

**Strong Interview Signals:**

- ✅ **Asks clarifying questions**: "Are we designing for 1M or 100M users?"
- ✅ **Shows calculations**: "200M posts/day = 2,315 posts/sec, so we need..."
- ✅ **Discusses trade-offs**: "Fan-out on write is faster but uses more storage..."
- ✅ **Considers failures**: "If the database fails, we'll promote a replica..."
- ✅ **Mentions monitoring**: "We'll monitor p95 latency and alert if it exceeds 500ms..."
- ✅ **Thinks about scale**: "At 100M users, we'll need to shard the database..."
- ✅ **Explains reasoning**: "I chose Cassandra because we need high write throughput..."
- ✅ **Invites feedback**: "Does this approach make sense? What would you change?"

---

## Putting It All Together

### System Architecture Summary

You've designed a production-grade social media platform that handles:

- **500M daily active users** across the globe
- **200M posts per day** (photos, videos, stories)
- **10B feed impressions per day** (personalized content delivery)
- **12 exabytes of media storage** (5-year retention)
- **99.9% availability** (8.76 hours downtime/year)

### Key Architectural Patterns

**1. Hybrid Fan-out Strategy**
- Fan-out on write for regular users (<1M followers)
- Fan-out on read for celebrities (>1M followers)
- Solves the celebrity problem while maintaining performance

**2. Multi-Database Architecture**
- PostgreSQL: User data, relationships (strong consistency)
- Cassandra: Posts, feeds (high write throughput)
- Neo4j: Graph relationships (follows, recommendations)
- Redis: Caching, real-time counters
- Elasticsearch: Search, hashtags, mentions

**3. Media Processing Pipeline**
- 100K workers processing images/videos in parallel
- Multiple format generation (thumbnails, different sizes)
- CDN distribution for global low latency
- 90% storage savings through optimization

**4. Real-time Updates**
- WebSocket servers maintaining 100M+ connections
- Redis Pub/Sub for event broadcasting
- Push notifications for offline users

### Interview Framework: 45-Minute Design Session

**Minutes 0-5: Requirements Gathering**
- Clarify functional requirements (MVP features)
- Understand scale (users, posts, impressions)
- Ask about geographic distribution, peak traffic

**Minutes 5-15: Capacity Planning**
- Calculate QPS (posts/sec, feed requests/sec)
- Estimate storage (user data, posts, media)
- Calculate bandwidth (upload, download)
- Estimate costs

**Minutes 15-25: High-Level Design**
- Draw system architecture diagram
- Identify core components (API, feed service, media pipeline)
- Explain data flow (post creation → processing → feed generation)

**Minutes 25-35: Deep Dive**
- Feed generation system (hybrid fan-out)
- Media processing pipeline
- Database design (sharding, replication)
- Caching strategy

**Minutes 35-45: Trade-offs & Scalability**
- Discuss trade-offs (write vs read optimization)
- Address bottlenecks (celebrity problem, media processing)
- Explain scaling strategies (horizontal scaling, sharding)

### Production Deployment Timeline

**Phase 1: MVP (Months 1-3)**
- Basic posting and feed functionality
- Single-region deployment
- PostgreSQL + Redis
- Simple fan-out on write

**Phase 2: Scale (Months 4-6)**
- Add Cassandra for posts
- Implement hybrid fan-out
- Media processing pipeline
- CDN integration

**Phase 3: Optimization (Months 7-9)**
- Multi-region deployment
- ML-powered ranking
- Advanced caching
- Real-time updates

**Phase 4: Advanced Features (Months 10-12)**
- Stories feature
- Search and discovery
- Content moderation
- Analytics pipeline

### Production Checklist

**Infrastructure:**
- [ ] Multi-region deployment configured
- [ ] Database replication and failover tested
- [ ] CDN configured with cache invalidation
- [ ] Load balancers configured with health checks
- [ ] Auto-scaling policies set up

**Application:**
- [ ] Hybrid fan-out implemented and tested
- [ ] Media processing pipeline handling failures gracefully
- [ ] WebSocket connections properly managed
- [ ] Rate limiting and DDoS protection enabled
- [ ] Monitoring and alerting configured

**Data:**
- [ ] Database sharding strategy implemented
- [ ] Backup and recovery procedures tested
- [ ] Data retention policies configured
- [ ] Privacy compliance (GDPR, CCPA) implemented

---

## Resources for Further Learning

### Engineering Blogs & Case Studies

**Instagram (Meta):**
- "Scaling Instagram Infrastructure" (500M+ users)
- "How Instagram Feeds Work" (hybrid fan-out strategy)
- Link: https://engineering.fb.com/tag/instagram/

**Twitter:**
- "The Architecture Twitter Uses to Deal with 150M Active Users"
- "How Twitter Handles 3,000 Images Per Second"
- Link: https://blog.twitter.com/engineering

**TikTok (ByteDance):**
- "How TikTok Recommends Videos" (ML-powered feed)
- "Scaling to 1 Billion Users"
- Link: https://www.tiktok.com/about/engineering

**Pinterest:**
- "Building Pinterest's Real-time Feed"
- "Scaling Pinterest from 0 to 200M Users"
- Link: https://medium.com/pinterest-engineering

**LinkedIn:**
- "The LinkedIn Feed: Building a Feed Ranking System"
- "Scaling LinkedIn's Feed Infrastructure"
- Link: https://engineering.linkedin.com/blog

### Books

1. **"Designing Data-Intensive Applications" by Martin Kleppmann**
   - Chapter on Feed Systems is excellent
   - Covers fan-out strategies and trade-offs
   - Essential for understanding distributed systems

2. **"System Design Interview" by Alex Xu (Volumes 1 & 2)**
   - Real interview questions and solutions
   - Covers social media platform design
   - Great for interview preparation

3. **"Building Microservices" by Sam Newman**
   - Microservices architecture patterns
   - Service communication strategies
   - Relevant for large-scale social platforms

### Online Courses

**System Design Courses:**
- "Grokking the System Design Interview" (Educative.io)
- "System Design Primer" (GitHub)
- "High Scalability" blog for real-world architectures

**ML/AI Courses (for Ranking Algorithms):**
- "Machine Learning" by Andrew Ng (Coursera)
- "Deep Learning Specialization" (Coursera)
- "Recommendation Systems" courses

### Conference Talks

**QCon, Strange Loop, SREcon:**
- Search for "feed systems", "social media architecture"
- Real-world case studies from major platforms
- Performance optimization techniques

**ML Conferences:**
- RecSys (Recommendation Systems Conference)
- KDD, ICML (for ranking algorithms)

### Hands-On Practice

**Build Your Own:**
1. Simple feed system with fan-out on write
2. Add celebrity handling (fan-out on read)
3. Implement media processing pipeline
4. Add real-time updates with WebSocket

**Open Source Projects:**
- Study Mastodon (federated social network)
- Examine Pleroma (ActivityPub implementation)
- Learn from Diaspora (distributed social network)

---

## Congratulations! 🎉

You've completed an in-depth journey through Social Media Platform System Design! Here's what you've accomplished:

### What You've Learned

**Foundations:**
- ✅ What social media platforms are and how they work at scale
- ✅ Core concepts: feeds, fan-out, media processing, real-time updates
- ✅ How to calculate capacity (traffic, storage, bandwidth, cost)

**System Design Skills:**
- ✅ How to gather requirements systematically
- ✅ How to solve the celebrity problem with hybrid fan-out
- ✅ How to make design trade-offs (write vs read optimization)
- ✅ How to present your design in interviews with frameworks

**Production Knowledge:**
- ✅ Media processing pipelines (100K workers, transcoding)
- ✅ Multi-database architectures (right tool for each job)
- ✅ Real-time systems (WebSocket, Pub/Sub)
- ✅ ML-powered ranking algorithms

### Your Readiness Level

**For Interviews:**
- 🟢 **Entry-level**: You can explain social media basics with analogies
- 🟡 **Mid-level**: You can design a system for 500M users and justify trade-offs
- 🔴 **Senior-level**: You can discuss production concerns (celebrity problem, media processing, ML ranking)

**Next Steps for Interview Prep:**
1. Practice drawing the architecture diagram in 5 minutes
2. Memorize key numbers (500M DAU, 200M posts/day, 10B impressions/day)
3. Practice explaining hybrid fan-out strategy
4. Review "Think About It" questions—interviewers love these!

**For Real-World Work:**
- 🏗️ You can design a social media platform for your organization
- 📊 You can perform capacity planning and cost estimation
- 🔧 You can troubleshoot common issues (feed latency, celebrity problem)
- 📈 You can plan for growth (scaling from 1M to 500M users)

### Continue Your Learning Journey

**Next System Designs to Study:**
1. **Newsfeed System** (Deep dive into feed ranking algorithms)
2. **Chat Application** (Real-time messaging at scale)
3. **Video Streaming** (Media processing and CDN)

**Deep Dive Topics:**
1. **ML Ranking Algorithms**: Learn how Instagram/TikTok rank content
2. **Graph Databases**: Deep dive into Neo4j for relationship queries
3. **Media Processing**: FFmpeg, video transcoding, image optimization

**Advanced Concepts:**
1. **Content Moderation**: AI/ML for detecting harmful content
2. **Recommendation Engines**: Collaborative filtering, content-based filtering
3. **Graph Neural Networks**: For friend suggestions and content discovery

### Final Thoughts

Social media platforms are among the most complex systems in the world, handling:
- Billions of users and trillions of interactions
- Exabytes of media content
- Real-time updates at massive scale
- ML-powered personalization

**The skills you've learned here apply broadly:**
- Feed generation systems
- Media processing pipelines
- Real-time update architectures
- ML-powered ranking systems

**You're now equipped to:**
- ✅ Design social media platforms in interviews (FAANG-ready!)
- ✅ Make informed architectural decisions at work
- ✅ Communicate complex technical concepts clearly
- ✅ Understand trade-offs and justify your choices

### Keep in Touch!

As you continue learning:
- Revisit this document when working on real social media projects
- Use it as a reference during interviews
- Share it with teammates learning about feed systems

**Remember:** System design is about trade-offs. There's no perfect solution—only solutions that fit your specific requirements. The framework you've learned here (requirements → capacity planning → architecture → trade-offs) applies to ANY system you design.

**Good luck with your interviews and projects! You've got this! 🚀**

---

**Document Status:** ✅ Complete Educational Template Format  
**Last Updated:** November 12, 2025  
**Total Learning Time:** 6-8 hours (Beginner), 8-10 hours (Intermediate), 10-14 hours (Advanced)  
**Word Count:** ~50,000+ words  
**Target Audience:** Software Engineers preparing for FAANG interviews and production system design

---

*End of Document*
