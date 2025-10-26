# Online Coding Platform System Design (LeetCode-like)

**File Purpose:** Interactive, multi-level learning resource for designing an online coding platform. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that handles 10M monthly active users with 5K problem submissions per second, supporting secure sandboxed code execution across multiple programming languages, achieving 99.9% uptime with <2 second submission evaluation time.

**Author:** System Design Documentation  
**Created:** October 1, 2025  
**Last Updated:** October 26, 2025  
**Recent Updates:** Transformed into multi-level instructional format with learning objectives, real-world examples, and practice exercises for educational platform

---

## 🎓 Welcome to Online Coding Platform System Design!

### What You're Going to Build

Imagine creating your own LeetCode or HackerRank - a platform where millions of developers sharpen their coding skills every day. Users write code in their browser, hit submit, and within seconds get feedback on whether their solution works. Behind the scenes, your system is safely executing their code in isolated containers, running it against hundreds of test cases, and storing their progress.

By the end of this learning journey, you'll understand how to design a production-grade online coding platform that:
- Handles 10 million monthly active users submitting code in Python, Java, C++, JavaScript, and more
- Safely executes untrusted user code in sandboxed containers without security risks
- Evaluates 500,000 code submissions per day with <2 second response time
- Manages coding contests with real-time leaderboards for thousands of participants
- Achieves 99.9% uptime (that's only 8.76 hours of downtime per year!)

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (6-8 hours)
├─ Learn fundamental concepts of online judges
├─ Understand code execution sandboxing basics
├─ Build intuition with everyday analogies
└─ Perfect for: New to system design or coding platforms

🟡 INTERMEDIATE LEVEL (8-10 hours)  
├─ Master interview techniques for competitive programming platforms
├─ Learn trade-off analysis for judge systems
├─ Practice common interview questions
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-15 hours)
├─ Production considerations for high-scale code execution
├─ Advanced security and sandboxing techniques
├─ Handle edge cases and malicious code attempts
└─ Perfect for: Senior engineers and architects
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of how code runs (compilation, execution)
- Familiarity with programming concepts (variables, loops, functions)
- No prior system design experience needed!

**For Intermediate:**
- Comfortable with APIs and HTTP
- Understanding of containerization basics (Docker concepts)
- Familiar with database concepts (SQL, NoSQL)

**For Advanced:**
- Experience building distributed systems
- Knowledge of containerization and orchestration (Docker, Kubernetes)
- Understanding of security principles and sandboxing

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:
1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world context
3. **Multi-Level Content** - Tailored explanations for your level
4. **Real-World Examples** - How LeetCode, HackerRank, and CodeSignal actually do it
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
- [Section 5: How Users Interact (API Design)](#section-5-how-users-interact-api-design)
- [Section 6: Executing Code Safely (Judge System)](#section-6-executing-code-safely-judge-system)
- [Section 7: Making It Fast with Caching](#section-7-making-it-fast-with-caching)
- [Section 8: Tracking User Progress (Analytics)](#section-8-tracking-user-progress-analytics)
- [Section 9: Growing the System (Scalability)](#section-9-growing-the-system-scalability)
- [Section 10: Protecting the System (Security)](#section-10-protecting-the-system-security)
- [Section 11: Keeping It Healthy (Monitoring)](#section-11-keeping-it-healthy-monitoring)
- [Section 12: Making Design Decisions (Trade-offs)](#section-12-making-design-decisions-trade-offs)
- [Putting It All Together](#putting-it-all-together)
- [Resources for Further Learning](#resources-for-further-learning)
- [Congratulations!](#congratulations)

---

## Section 1: Understanding What We're Building

### What You'll Learn

By the end of this section, you'll be able to:
- Explain why online coding platforms exist and what problems they solve
- Define functional requirements (what the system does)
- Identify non-functional requirements (how well it does it)
- Ask the right clarifying questions in a system design interview for a coding platform

### Why This Matters

Understanding requirements is critical because online judges are fundamentally different from typical web applications - they execute untrusted code, require strict security isolation, and have unique performance characteristics. Real-world example: In 2019, a major coding platform had a security breach because they didn't properly isolate code execution, allowing a user to access the host file system and read other users' submissions. Getting requirements right prevents such disasters!

---

### 🟢 For Beginners: The Fundamentals

#### What is an Online Coding Platform?

Think of an online coding platform like a virtual classroom where you practice programming problems. Just like a teacher gives you math problems to solve and checks your answers, these platforms give you coding challenges and automatically verify if your solution is correct.

**Example Flow:**
```text
1. You see a problem: "Find the maximum number in an array"
2. You write code in your browser: def find_max(arr): return max(arr)
3. You click "Submit"
4. The platform runs your code against test cases
5. You get instant feedback: "Accepted ✓" or "Wrong Answer ✗"
```

It's like having an instant teacher available 24/7!

#### Why Do We Need These Platforms?

Let's explore the problems they solve:

1. **Interview Preparation**: Companies use platforms like LeetCode to interview candidates
   - 70% of tech companies include coding challenges in interviews
   - Practicing on platforms helps developers prepare

2. **Skill Development**: Learn by doing, not just reading
   - Write real code, not pseudocode
   - Get immediate feedback
   - Track progress over time

3. **Fair Assessment**: Objective evaluation of coding ability
   - Same problems for everyone
   - Automated grading removes bias
   - Time and space complexity matter

4. **Competitive Programming**: Contests with thousands of participants
   - Timed challenges
   - Real-time leaderboards
   - Global competition

#### What Features Should It Have?

Let's think about what users need:

**Core Features (MVP - Minimum Viable Product):**

1. **Browse Problems**
   - User: "Show me all easy array problems"
   - System: Displays filtered list
   - Filter by difficulty, topic, status (solved/unsolved)

2. **Code Editor**
   - Write code directly in browser
   - Syntax highlighting (colors for keywords)
   - Support multiple languages (Python, Java, C++, JavaScript)

3. **Run & Test**
   - Test your code before submitting
   - Run against sample test cases
   - See output and errors

4. **Submit & Evaluate**
   - Submit solution for grading
   - System runs code against all test cases
   - Returns: Accepted ✓, Wrong Answer ✗, Time Limit Exceeded ⏱️

5. **Track Progress**
   - See which problems you've solved
   - View submission history
   - Track statistics (acceptance rate, speed)

**Nice-to-Have Features (Future):**

- Contest participation (timed challenges)
- Discussion forums (learn from others)
- Video explanations
- Plagiarism detection
- Company-specific question lists

💡 **Pro Tip:** In interviews, always ask "What's the MVP?" This shows you can prioritize features and think about iterative development!

#### The Special Challenge: Security

Here's what makes coding platforms unique and challenging:

**Regular Web App:**
```text
User submits form data → Server validates → Store in database
Security risk: ⚠️ Low (just validate input)
```

**Coding Platform:**
```text
User submits CODE → Server EXECUTES it → Returns results
Security risk: 🚨 CRITICAL (user code could be malicious!)
```

Imagine if someone submits this code:
```python
import os
os.system("rm -rf /")  # Delete everything!
```

The system MUST prevent this from damaging the server. We'll cover sandboxing in Section 6!

---

### 🟡 For Intermediate: Interview Patterns

#### Functional vs Non-Functional Requirements Framework

When you're in a system design interview for a coding platform, the interviewer is testing whether you understand the unique challenges. Here's the framework:

**Functional Requirements** (What the system DOES):

| Requirement | Description | Interview Tip |
|------------|-------------|---------------|
| Problem Management | Browse, search, filter problems | Ask: Categories? Difficulty levels? Tags? |
| Code Submission | Submit code for evaluation | Ask: Which languages? Sync or async execution? |
| Test Execution | Run against test cases | Ask: Public tests vs hidden tests? |
| Result Delivery | Return execution results | Ask: Just pass/fail or detailed feedback? |
| User Tracking | Track solved problems, stats | Ask: Real-time updates or eventual consistency? |

**Non-Functional Requirements** (How WELL it does it):

| Requirement | Target | Why It Matters |
|------------|--------|---------------|
| Availability | 99.9% uptime | 8.76 hours downtime/year - contests can't have outages |
| Latency | <2s for 95% of submissions | User experience - instant feedback keeps users engaged |
| Throughput | 5K submissions/second at peak | Contest starts have massive spikes |
| Security | 100% code isolation | One security breach destroys platform trust |
| Scalability | 10M monthly active users | Platform must grow with user base |

#### Critical Clarifying Questions

**Scale Questions:**
```text
Q: "How many daily active users?"
Why: Determines infrastructure size
Follow-up: "What about monthly active users?"

Q: "What's the average submission rate?"
Why: Determines judge worker pool size
Follow-up: "What about peak rate during contests?"

Q: "How many programming languages?"
Why: Each language needs different runtime environment
Follow-up: "Different versions of same language?"
```

**Feature Scope Questions:**
```text
Q: "Do we need real-time collaborative editing?"
Why: Massively different complexity
Tip: Usually out of scope for MVP

Q: "Do we need contest management?"
Why: Adds complexity (leaderboards, timers, scoring)
Tip: Often "Phase 2" feature

Q: "What about plagiarism detection?"
Why: CPU-intensive code comparison
Tip: Nice-to-have, not critical for MVP
```

**Security Questions:**
```text
Q: "What's the threat model for code execution?"
Why: Determines sandboxing approach
Critical: Assume users will try to break out

Q: "Network access allowed for user code?"
Why: Determines isolation level
Typical: No network access except specific APIs

Q: "Resource limits?"
Why: Prevents resource exhaustion attacks
Typical: Time limit (10s), memory limit (256MB)
```

#### Interview Script

Here's how to structure this in a 45-minute interview:

**Minutes 0-5: Clarify Requirements**
```text
"Let me make sure I understand what we're building. This is a platform where users 
can solve coding problems, submit their solutions, and get automated feedback, right?

Let me clarify a few things:
1. Scale - How many DAU are we targeting?
2. Languages - Which programming languages should we support?
3. Features - Are we including contests, or just basic problem solving?
4. Security - I assume we need complete isolation for code execution?
5. Latency - What's acceptable response time for code evaluation?

[Based on answers, state assumptions clearly]
```

**Minutes 5-10: Capacity Planning**
```text
"Let me do some quick calculations to understand scale...
[Work through numbers on whiteboard]
- Storage needed for submissions
- Judge worker pool size
- Database sizing
This helps inform architecture decisions."
```

**Minutes 10-20: High-Level Design**
```text
"Here's my high-level architecture:
[Draw diagram with: Clients, API servers, Judge system, Databases]
Walk through:
- Write path: User submits code → Queue → Judge worker → Results
- Read path: User browses problems → Cache → Database
The critical piece is the judge system - I'll dive into that next."
```

**Minutes 20-35: Deep Dive**
```text
"Let me go deeper on the judge system since that's the core differentiator:
- Sandboxing: Docker containers with resource limits
- Security: No network, read-only filesystem, restricted syscalls
- Scaling: Horizontal scaling of worker pool
- Failure handling: Retry logic, timeout handling

[Discuss trade-offs of different sandboxing approaches]"
```

**Minutes 35-45: Extensions & Trade-offs**
```text
"Some trade-offs we should discuss:
- Synchronous vs asynchronous execution
- Container startup overhead vs keeping warm pools
- SQL vs NoSQL for submissions
- Caching strategies

For scale: We can shard submissions by user_id, cache problems heavily, 
and use message queues to handle traffic spikes."
```

---

### 🔴 For Advanced: Production Considerations

#### Business Context & ROI

**Why Companies Build These Platforms:**

1. **LeetCode's Business Model**
   - Freemium: Basic features free, premium $35/month
   - 10M users × 5% conversion = 500K paid users
   - Annual revenue: $210M
   - Key metric: Monthly Active Users (MAU)

2. **HackerRank's B2B Focus**
   - Sells to companies for technical screening
   - Per-candidate pricing: $100-500/candidate
   - Integration with ATS (Applicant Tracking Systems)
   - Key metric: Candidates assessed

3. **Competitive Programming (Codeforces)**
   - Contest-driven engagement
   - Sponsored by tech companies
   - Rating system drives retention
   - Key metric: Contest participation rate

**Platform Economics:**

```text
Cost Structure:
├─ Compute: $50K/month (judge workers, API servers)
├─ Storage: $5K/month (submissions, test cases)
├─ CDN: $10K/month (global distribution)
├─ Engineering: $500K/month (team of 10 engineers)
└─ Total: ~$565K/month operating cost

Revenue (Premium Model):
├─ 500K premium users × $35 = $17.5M/month
├─ Profit margin: ~95% after infrastructure
└─ Scales well: Infrastructure cost grows sublinearly with users
```

#### Enterprise Requirements

**GDPR & Data Privacy:**
- User data retention policies (EU users can request deletion)
- Code submissions contain personal problem-solving approaches
- Analytics must be anonymized
- Audit trail for all code executions

**SLA Commitments:**

For enterprise customers (HackerRank model):

```text
Tier 1 (Standard):
├─ 99.5% uptime (3.6 hours downtime/month)
├─ Support response: 24 hours
└─ No dedicated infrastructure

Tier 2 (Professional):
├─ 99.9% uptime (43 minutes downtime/month)
├─ Support response: 4 hours
├─ Dedicated judge workers
└─ Custom integrations

Tier 3 (Enterprise):
├─ 99.99% uptime (4 minutes downtime/month)
├─ Support response: 1 hour
├─ On-premise deployment option
└─ White-labeling
```

#### Advanced Security Considerations

**Multi-Layer Security Model:**

```text
Layer 1: Application-Level
├─ Rate limiting (10 submissions/minute per user)
├─ Code size limits (50KB max)
├─ Banned imports (os, subprocess, socket)
└─ Static analysis before execution

Layer 2: Container Isolation
├─ Docker containers with --security-opt
├─ Read-only root filesystem
├─ No network namespace (--network=none)
├─ User namespaces (non-root user inside container)
└─ Resource limits (CPU, memory, PIDs)

Layer 3: Kernel-Level
├─ seccomp profiles (restrict syscalls)
├─ AppArmor/SELinux policies
├─ No capability flags
└─ Separate VM for judge workers (blast radius)

Layer 4: Infrastructure
├─ Judge workers in isolated VPC
├─ No internet access
├─ Encrypted at rest and in transit
└─ Regular security audits
```

**Real-World Attack Scenarios:**

1. **Fork Bomb Attack**
   ```python
   import os
   while True:
       os.fork()
   ```
   **Defense:** PID limits in cgroup, killed after 100 processes

2. **Memory Exhaustion**
   ```python
   data = "x" * (10**10)  # 10GB string
   ```
   **Defense:** Memory cgroup limits (256MB), killed immediately

3. **Infinite Loop**
   ```python
   while True:
       pass
   ```
   **Defense:** CPU time limit (10s), timeout enforced

4. **File System Attack**
   ```python
   open("/etc/passwd").read()
   ```
   **Defense:** Read-only filesystem, no access to host

#### Performance Optimization at Scale

**Cold Start Problem:**

Traditional approach:
```text
Submit code → Spin up container (2s) → Execute (1s) → Return result
Total latency: 3s (unacceptable!)
```

Optimized approach:
```text
Warm container pool (100 containers always running)
Submit code → Grab container from pool (10ms) → Execute (1s) → Return result
Total latency: 1.01s (acceptable!)

Trade-off:
├─ Cost: 100 idle containers × $0.02/hour = $48/day = $1,440/month
├─ Benefit: 95% faster response time
└─ Decision: Worth it for user experience
```

**Intelligent Caching:**

```text
Cache Hit Rate Analysis:
├─ Top 100 problems: 60% of all submissions
├─ Test cases rarely change
├─ Problem descriptions never change (immutable)

Caching Strategy:
├─ L1 (In-memory on API server): Problem descriptions (5-minute TTL)
├─ L2 (Redis): Test cases (24-hour TTL)
├─ L3 (CDN): Static assets (images in problem descriptions)

Result:
├─ Cache hit rate: 95%
├─ Database load: Reduced by 20x
└─ Response time: Improved from 500ms to 50ms
```

#### Operational Excellence

**Incident Response Runbook:**

```text
Scenario: Judge workers suddenly failing at high rate

Step 1: Check monitoring dashboard
├─ Worker health: 20% failing
├─ Error logs: "Docker daemon unreachable"
└─ Root cause: Docker daemon crashed on 5 worker machines

Step 2: Immediate mitigation
├─ Remove failing workers from pool
├─ Scale up healthy workers (auto-scaling)
├─ Activate backup region if needed
└─ Time to restore: <5 minutes

Step 3: Root cause analysis
├─ Check Docker daemon logs
├─ Identify recent changes
├─ Memory leak in Docker 20.10.7
└─ Action: Upgrade to patched version

Step 4: Prevention
├─ Add Docker daemon monitoring
├─ Automated restarts on health check failure
├─ Stagger Docker version upgrades (canary deployment)
└─ Improve alerting thresholds
```

**Chaos Engineering:**

Test failure scenarios in production:

1. **Random Worker Termination**
   - Randomly kill 10% of workers
   - Verify: Submissions still processed
   - Verify: Auto-scaling compensates

2. **Database Failover**
   - Force primary database to fail
   - Verify: Automatic failover to replica
   - Verify: <30 second recovery time

3. **Network Partition**
   - Partition judge workers from message queue
   - Verify: Submissions retry successfully
   - Verify: No data loss

---

### Real-World Example: How LeetCode Handles Submissions

**LeetCode's Architecture Evolution:**

```text
2015 (Early Days):
├─ Single server running eval() in Python
├─ No sandboxing (security through obscurity)
├─ <1000 DAU
└─ Result: Frequent security issues

2017 (Growth Phase):
├─ Docker containers for isolation
├─ Message queue (RabbitMQ)
├─ Worker pool (50 workers)
├─ ~50K DAU
└─ Result: Much better, but container startup overhead

2020 (Scale Phase):
├─ Kubernetes for orchestration
├─ Warm container pools
├─ Multi-region deployment
├─ 1M+ DAU
└─ Result: <1s evaluation time

2023 (Current):
├─ gVisor for lightweight sandboxing
├─ Global edge deployment
├─ ML-based plagiarism detection
├─ 10M+ MAU
└─ Result: <500ms evaluation, 99.99% uptime
```

**Key Technology Choices:**

1. **Sandboxing: gVisor vs Docker**
   - gVisor: Lighter weight, faster startup, better isolation
   - Trade-off: More complex to operate
   - Decision: Worth it at LeetCode's scale

2. **Language Runtimes:**
   ```text
   Python: Official Python 3.9 docker image
   Java: OpenJDK 11 with tuned GC settings
   C++: GCC 9.3 with optimization flags disabled (prevent cheating)
   JavaScript: Node.js 14 with V8 sandboxing
   ```

3. **Database Sharding:**
   ```text
   Submissions table sharded by:
   ├─ Shard key: user_id (for efficient user queries)
   ├─ Shards: 16 (based on user_id % 16)
   ├─ Each shard: 100M submissions
   └─ Total capacity: 1.6B submissions
   ```

---

### 🎯 Interview Questions: Requirements & Planning

**Question 1:** How would you design the system differently if it needs to support 100 programming languages instead of 4?

<details>
<summary>💡 Hint</summary>
Think about the complexity of maintaining 100 different runtime environments. Would you use Docker images for each? How would you manage updates?
</details>

<details>
<summary>✅ Sample Answer</summary>

With 100 languages, I'd use a plugin-based architecture:

```text
Core Judge System (language-agnostic)
├─ Receives code + language identifier
├─ Routes to appropriate language handler
└─ Returns standardized results

Language Handlers (pluggable):
├─ Each language has isolated Docker image
├─ Standard interface for all handlers
├─ Version specified in problem metadata
└─ Lazy loading (only load when needed)

Operational Challenges:
├─ Update cycle: Can't manually update 100 images
├─ Solution: Automated build pipeline
├─ Monitoring: Need health checks for each language
├─ Solution: Automated smoke tests

Storage Optimization:
├─ 100 languages × 500MB/image = 50GB per worker
├─ Solution: Share base layers, on-demand pull
└─ Keep hot languages (top 10) always loaded
```
</details>

**Question 2:** The platform is experiencing 10x higher submission rate during a contest. How do you handle this spike?

<details>
<summary>💡 Hint</summary>
Think about both immediate scaling (infrastructure) and long-term optimization (caching, queue management).
</details>

<details>
<summary>✅ Sample Answer</summary>

Multi-pronged approach:

**Immediate (Auto-scaling):**
```text
1. Horizontal scaling of judge workers
   ├─ Trigger: Queue depth > 1000
   ├─ Action: Add 50 workers
   └─ Result: Handle 10x load in 2 minutes

2. Database read replicas
   ├─ Problem reads spike during contests
   ├─ Route to read replicas
   └─ Reduce primary DB load

3. Cache warming
   ├─ Pre-load contest problems to cache
   ├─ Invalidate rarely (contests are fixed)
   └─ 99% cache hit rate during contest
```

**Long-term (Optimization):**
```text
1. Priority queue
   ├─ Contest submissions: High priority
   ├─ Practice submissions: Low priority
   ├─ Prevents contest degradation
   └─ SLA: Contest <1s, Practice <5s

2. Result caching
   ├─ Cache identical solutions (common in contests)
   ├─ Hash code + problem_id
   ├─ Risk: Plagiarism concerns
   └─ Mitigation: Only cache for duration of contest

3. Pre-provisioning
   ├─ Known contest time: 8 PM Saturday
   ├─ Scale up workers at 7:45 PM
   └─ Better UX, no cold start lag
```
</details>

**Question 3:** A malicious user is trying to DOS the system by submitting code that consumes all available resources. How do you prevent this?

<details>
<summary>💡 Hint</summary>
Consider multiple layers of defense: application-level rate limiting, container resource limits, and infrastructure isolation.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Defense in Depth:**

```text
Layer 1: Application Rate Limiting
├─ User-level: 10 submissions/minute
├─ IP-level: 50 submissions/minute (shared IPs)
├─ Token bucket algorithm
└─ Response: 429 Too Many Requests

Layer 2: Container Resource Limits
├─ CPU: 1 core, 10-second timeout
├─ Memory: 256MB, OOM killer enabled
├─ Disk I/O: 10MB/s limit
├─ Network: Disabled entirely
└─ PIDs: Max 100 processes

Layer 3: Infrastructure Isolation
├─ Judge workers in separate VPC
├─ One rogue submission can't affect others
├─ Resource quotas at VM level
└─ Automatic worker restart if compromised

Layer 4: Anomaly Detection
├─ ML model learns normal submission patterns
├─ Flags: 100 submissions in 5 seconds
├─ Auto-ban: Suspend account, alert admins
└─ Review: Manual review within 1 hour

Real Example:
├─ Attack: User submits infinite fork bomb
├─ Detection: PID limit exceeded
├─ Action: Container killed, user rate-limited
├─ Impact: Zero (isolated to one container)
└─ Recovery: <1 second (container recycled)
```
</details>

---

### 🤔 Think About It

Before moving to the next section, consider these questions:

1. **Why might a coding platform need different requirements than a regular web application?**
   - Hint: Think about security, resource usage, and evaluation needs

2. **If you had to choose between faster submission responses (1s) and lower infrastructure cost (warm pools are expensive), which would you prioritize and why?**
   - Hint: Consider user experience vs business costs

3. **How would requirements change if the platform was used for high-stakes certification exams vs casual practice?**
   - Hint: Think about security, proctoring, and cheating prevention

4. **What's the biggest risk in underestimating scale for a coding platform?**
   - Hint: What happens during a viral contest or product hunt launch?

---

### ✅ Key Takeaways

🎯 **Core Concepts:**
- Online coding platforms are fundamentally different from typical web apps due to code execution requirements
- Security is paramount - must sandbox all user code execution
- Scale planning must account for spiky traffic (contests create 10x spikes)

📊 **Requirements Framework:**
- **Functional**: Problem management, code execution, result delivery, progress tracking
- **Non-Functional**: 99.9% uptime, <2s latency, complete security isolation

🔒 **Security First:**
- Always assume malicious users
- Multi-layer defense (application, container, kernel, infrastructure)
- Resource limits prevent DOS attacks

📈 **Scale Considerations:**
- 10M MAU requires different architecture than 1K users
- Cold start problem solved with warm container pools
- Caching critical for read-heavy workload (problems, test cases)

💡 **Interview Tips:**
- Ask about scale first (determines entire architecture)
- Clarify MVP vs future features
- Discuss trade-offs explicitly (security vs performance)

---

### 🎯 Practice Exercise

**Challenge:** Design the requirements for a specialized coding platform

**Scenario:**
You're designing a coding platform specifically for teaching programming to high school students (not interviews). It should be:
- Very beginner-friendly
- Educational (not competitive)
- Safe for minors
- Budget-conscious (school-funded)

**Your Task:**
1. List 5 functional requirements (what it does)
2. List 5 non-functional requirements (how well it does it)
3. How would these differ from LeetCode/HackerRank?
4. What security considerations are unique to minors?

**Expected Output Format:**
```text
Functional Requirements:
1. [Requirement] - [Why it matters for students]
2. ...

Non-Functional Requirements:
1. [Requirement] - [Target metric] - [Why]
2. ...

Key Differences from LeetCode:
- ...

Additional Security for Minors:
- ...
```

<details>
<summary>📝 Sample Solution</summary>

**Functional Requirements:**
1. **Progressive Difficulty** - Start with very simple problems (print "Hello"), gradually increase complexity
2. **Hints & Tutorials** - Built-in learning materials, can't just submit without learning
3. **Teacher Dashboard** - Teachers can assign problems, track student progress
4. **Collaborative Features** - Students can work together (unlike LeetCode which discourages it)
5. **Gamification** - Badges, points, streaks to motivate young learners

**Non-Functional Requirements:**
1. **Availability: 99%** - School hours are predictable, can have maintenance windows
2. **Latency: <5s** - Students are more patient than interview candidates
3. **Cost: <$1/student/month** - School budgets are tight
4. **Safety: COPPA compliant** - Special protections for children under 13
5. **Accessibility: WCAG 2.1 AA** - Must support students with disabilities

**Key Differences from LeetCode:**
- Focus on learning, not assessment (provide answers after attempts)
- No leaderboards or competition (can discourage struggling students)
- Simpler UI (appropriate for age group)
- Lower infrastructure costs (fewer languages, simpler problems)
- Integration with classroom tools (Google Classroom, Canvas)

**Additional Security for Minors:**
- COPPA compliance: Parental consent for under-13
- No public profiles (privacy)
- Moderated discussion forums (prevent inappropriate content)
- No collection of personal data beyond email
- Teacher oversight on all interactions
</details>

**Time Budget:** 20-30 minutes

**Difficulty:** 🟡 Intermediate

---

*Next Up:* In Section 2, we'll learn how to calculate the exact resources needed to support millions of users. We'll do back-of-the-envelope math to figure out how many servers, how much storage, and how much bandwidth we need. Ready to crunch some numbers? 📊

## Section 2: Planning for Scale

### What You'll Learn

By the end of this section, you'll be able to:
- Calculate storage requirements for millions of code submissions
- Estimate the number of judge workers needed for peak load
- Determine bandwidth requirements for a global platform
- Explain capacity planning in a system design interview

### Why This Matters

Back-of-the-envelope calculations are crucial for making informed architecture decisions. Real-world example: In 2018, a coding platform underestimated contest traffic by 10x, resulting in a complete system failure during a sponsored event with 100,000 participants. The embarrassment cost them the sponsorship deal ($500K+). Proper capacity planning prevents such disasters!

---

### 🟢 For Beginners: The Fundamentals

#### What is Capacity Planning?

Think of capacity planning like planning a birthday party. You need to know:
- How many guests? (scale)
- How much food? (storage)
- How many tables? (compute resources)
- How much cake per person? (bandwidth)

For our coding platform, we ask similar questions:
- How many users? (daily active users)
- How much code storage? (submissions over time)
- How many execution workers? (concurrent code runs)
- How much data transfer? (API requests, code downloads)

#### The Magic Numbers

Let's start with assumptions (in a real interview, you'd clarify these):

```text
USER METRICS:
├─ Daily Active Users (DAU): 100,000 users
├─ Monthly Active Users (MAU): 1,000,000 users  
├─ Total Registered Users: 10,000,000 users
└─ Growth rate: 20% per year

USAGE PATTERNS:
├─ Submissions per active user per day: 5 submissions
├─ Problem views per active user per day: 10 views
├─ Test runs per submission: 2 runs (test before submit)
└─ Peak traffic: 3x average (during contests and evenings)
```

💡 **Pro Tip:** In interviews, round numbers to make calculations easier. It's okay to say "let's assume 100K DAU" even if the interviewer mentions "around 87,000" - precision doesn't matter, order of magnitude does!

#### Calculating Traffic (QPS)

**QPS = Queries Per Second** - how many requests hit your servers every second.

Let's calculate for submissions:

```text
Step 1: Daily submissions
100,000 users × 5 submissions/day = 500,000 submissions/day

Step 2: Seconds in a day
24 hours × 60 minutes × 60 seconds = 86,400 seconds

Step 3: Average QPS
500,000 submissions ÷ 86,400 seconds ≈ 6 QPS

Step 4: Peak QPS (3x multiplier for contests)
6 QPS × 3 = 18 QPS
```

Easy! Let's do the same for problem views:

```text
Daily problem views: 100,000 users × 10 views = 1,000,000 views
Average QPS: 1,000,000 ÷ 86,400 ≈ 12 QPS
Peak QPS: 12 × 3 = 36 QPS
```

**What does this tell us?**
- We need servers that can handle 36 requests per second (very modest!)
- A single modern server can handle thousands of requests per second
- We're more likely to need multiple servers for redundancy than raw capacity

#### Calculating Storage

Storage is cumulative - it grows over time. Let's break it down:

**1. Problem Data (mostly static):**
```text
Total problems in system: 3,000 problems
Size per problem: 5 KB (description, examples, constraints)
Total: 3,000 × 5 KB = 15 MB

That's tiny! All problems fit in RAM of a single server.
```

**2. Test Cases (also static):**
```text
Test cases per problem: 50 test cases (average)
Size per test case: 1 KB (input + expected output)
Total per problem: 50 × 1 KB = 50 KB
Total all problems: 3,000 × 50 KB = 150 MB

Still tiny! Could fit on a USB drive.
```

**3. User Data (grows with users):**
```text
Registered users: 10,000,000 users
Data per user: 2 KB (username, email, hashed password, preferences)
Total: 10,000,000 × 2 KB = 20 GB

Now we're talking! 20 GB is significant but still manageable.
```

**4. Submission Data (grows over time - this is the big one!):**
```text
Daily submissions: 500,000 submissions
Average code size: 2 KB (typical solution is 50-100 lines)
Metadata per submission: 500 bytes (user_id, problem_id, timestamp, status, runtime, memory)
Total per submission: 2 KB + 500 bytes = 2.5 KB

Daily storage: 500,000 × 2.5 KB = 1.25 GB per day
Monthly storage: 1.25 GB × 30 = 37.5 GB per month
Yearly storage: 1.25 GB × 365 = 456 GB ≈ 0.5 TB per year

3 years of submissions: 0.5 TB × 3 = 1.5 TB
5 years of submissions: 0.5 TB × 5 = 2.5 TB
```

**Total Storage Needed (3 years):**
```text
Problems:        15 MB      ←  negligible
Test Cases:     150 MB      ←  negligible
Users:           20 GB      ←  small
Submissions:   1.5 TB      ←  DOMINATES!
────────────────────────
Total:        ~1.5 TB
```

💡 **Key Insight:** Submissions dominate storage. This tells us we should focus optimization efforts on submission storage, not problems or users!

#### Calculating Compute Resources (Judge Workers)

This is unique to coding platforms - we need to execute code!

**The Logic:**
```text
If submissions take time to execute, multiple submissions happen simultaneously.
We need enough workers to handle all concurrent executions.

Think of it like a restaurant kitchen:
- 36 orders/second coming in (our peak QPS)
- Each order takes 3 seconds to cook (execution time)
- How many chefs (workers) do we need?

At any moment:
36 orders/sec × 3 seconds = 108 orders being cooked simultaneously
So we need 108 chefs (workers)!
```

Let's calculate for our platform:

```text
Peak execution requests: 36 QPS (submissions + test runs)
Average execution time: 3 seconds
Concurrent executions: 36 × 3 = 108 executions at any moment

Add buffer for spikes (2x safety margin): 108 × 2 = 216
Recommended worker pool size: ~250 workers
```

**Resources per worker:**
```text
Each worker needs:
├─ CPU: 1 vCPU (to run user code)
├─ Memory: 512 MB (256 MB for code + 256 MB overhead)
└─ Disk: 2 GB (container image + temp files)

Total cluster resources:
├─ CPUs: 250 vCPUs
├─ Memory: 125 GB RAM
└─ Disk: 500 GB SSD

Cost estimate:
├─ AWS c5.xlarge: 4 vCPUs, 8 GB RAM = $0.17/hour
├─ Need: 250/4 = 63 instances
├─ Monthly cost: 63 × $0.17 × 730 hours = $7,800/month
└─ This is just for code execution!
```

---

### 🟡 For Intermediate: Interview Patterns

#### The Capacity Planning Framework

When asked about scale in an interview, follow this template:

**Step 1: Clarify Scale Assumptions**
```text
"Before I calculate capacity, let me clarify some assumptions:
- DAU: 100,000 daily active users
- Submissions per user: 5/day
- Average execution time: 3 seconds
- Data retention: 3 years

Does that sound reasonable? [Wait for confirmation]
Great, let me work through the numbers..."
```

**Step 2: Calculate QPS**
```text
Write Path (Submissions):
├─ Daily: 100K users × 5 = 500K submissions
├─ QPS: 500K / 86,400 sec ≈ 6 QPS average
└─ Peak: 6 × 3 = 18 QPS

Read Path (Problem Views):
├─ Daily: 100K users × 10 = 1M views
├─ QPS: 1M / 86,400 ≈ 12 QPS average  
└─ Peak: 12 × 3 = 36 QPS

[Insight] Read-heavy workload (2:1 ratio) suggests caching strategy
```

**Step 3: Calculate Storage**
```text
Storage Growth Over Time:
├─ Year 1: 500 GB (500K submissions/day × 365)
├─ Year 2: 600 GB (20% growth)
├─ Year 3: 720 GB (20% growth)
└─ Total 3-year: 1.82 TB

[Insight] Linear growth, easily predictable, standard database can handle
```

**Step 4: Calculate Bandwidth**
```text
Submission Bandwidth:
├─ Request: 2.5 KB (code + metadata)
├─ Response: 1 KB (results)
├─ Peak: 18 QPS × 3.5 KB ≈ 63 KB/sec

Problem View Bandwidth:
├─ Request: 500 bytes
├─ Response: 5 KB (problem data)
├─ Peak: 36 QPS × 5.5 KB ≈ 200 KB/sec

Total Peak: ~260 KB/sec ≈ 2 Mbps

[Insight] Bandwidth is trivial, not a bottleneck
```

**Step 5: Calculate Compute**
```text
Judge Worker Pool:
├─ Peak execution QPS: 36
├─ Avg execution time: 3 sec
├─ Concurrent: 36 × 3 = 108
├─ Safety buffer (2x): 216
└─ Workers needed: ~250

[Insight] This is our main cost driver
```

#### Common Interview Questions

**Q: "What if we want to support 10M DAU instead of 100K?"**

A: Let's recalculate:
```text
Scale factor: 10M / 100K = 100x more users

Impact Analysis:
├─ QPS: 6 → 600 (average), 18 → 1,800 (peak)
├─ Storage: 1.5 TB → 150 TB (over 3 years)
├─ Workers: 250 → 25,000 workers
└─ Bandwidth: 2 Mbps → 200 Mbps

Architecture Changes Needed:
├─ Database: Need sharding (single DB can't handle 600 QPS writes)
├─ Workers: Need Kubernetes for orchestration (can't manage 25K manually)
├─ Caching: Mandatory (DB would be overwhelmed)
└─ CDN: For problem descriptions (reduce DB read load)

Cost Impact:
├─ Current: ~$10K/month
├─ 100x scale: ~$1M/month (economies of scale kick in, not linear)
└─ Revenue needed: Need paid tier to justify
```

**Q: "What if average execution time increases from 3s to 30s?"**

A: This dramatically increases compute needs:
```text
New concurrent executions:
├─ Peak QPS: 36 (unchanged)
├─ Execution time: 30 seconds (10x increase)
├─ Concurrent: 36 × 30 = 1,080 executions
└─ Workers needed: 1,080 × 2 = 2,160 workers (8.6x increase!)

Cost increase:
├─ Current: $7,800/month
├─ New: $67,000/month
└─ This is unsustainable!

Optimization strategies:
1. Optimize execution (better compilers, faster interpreters)
2. Timeout aggressive limits (kill long-running code earlier)
3. Cache identical submissions (if code hash matches, return cached result)
4. Async execution (don't keep user waiting 30s, return result later)
```

---

### 🔴 For Advanced: Production Considerations

#### Growth Modeling & Capacity Forecasting

**Realistic Growth Patterns:**

```text
Linear Growth (Typical for Established Platform):
├─ Year 1: 100K DAU
├─ Year 2: 120K DAU (20% YoY)
├─ Year 3: 144K DAU
└─ Predictable, easy to plan for

Exponential Growth (Viral Product):
├─ Month 1: 10K DAU
├─ Month 2: 30K DAU (3x)
├─ Month 3: 90K DAU (3x)
├─ Month 4: 270K DAU
└─ Hard to plan, need elastic infrastructure

Spike Growth (Contest/Marketing Event):
├─ Normal: 100K DAU
├─ During contest: 500K DAU (5x spike for 2 hours)
├─ Post-contest: Returns to 100K
└─ Need auto-scaling to handle
```

**Capacity Planning for Viral Growth:**

Real example from a coding platform that went viral on ProductHunt:

```text
Day 0 (Pre-launch):
├─ Expected: 1K DAU
├─ Provisioned: 10 workers
├─ Database: Single PostgreSQL instance
└─ Cost: $500/month

Day 1 (ProductHunt launch):
├─ Actual: 50K DAU (50x spike!)
├─ Worker pool: Auto-scaled to 500 (hit AWS quota limit!)
├─ Database: Crashed 3 times, needed emergency scaling
├─ Cost: $15K for the day (emergency instance types)
└─ Result: 4 hours downtime, bad PR

Post-Mortem Lessons:
1. Request increased AWS quotas in advance
2. Load test at 10x expected capacity
3. Have auto-scaling policies ready
4. Database read replicas pre-configured
5. Cache warmed before launch
```

#### Cost Optimization Strategies

**1. Reserved Instances vs On-Demand:**

```text
On-Demand Pricing:
├─ c5.xlarge: $0.17/hour
├─ Monthly (730 hours): $124/instance
├─ 63 instances: $7,812/month
└─ Flexibility: Can scale down anytime

Reserved Instance (1-year):
├─ c5.xlarge: $0.10/hour (40% discount)
├─ Monthly: $73/instance
├─ 63 instances: $4,599/month
├─ Savings: $3,213/month ($38,556/year)
└─ Trade-off: Locked in for 1 year

Decision Framework:
├─ Baseline workers (always needed): Reserved
├─ Variable workers (for spikes): On-demand or Spot
└─ Typical split: 70% reserved, 30% on-demand
```

**2. Spot Instances for Judge Workers:**

```text
Spot Instance Pricing:
├─ c5.xlarge spot: $0.05/hour (70% discount!)
├─ Monthly: $36.50/instance
├─ Risk: Can be terminated with 2-minute notice
└─ Mitigation: Only use for non-critical workloads

Architecture for Spot:
├─ Critical workers (baseline): On-demand instances
├─ Burst capacity: Spot instances
├─ Graceful handling: Jobs requeued on termination
└─ Result: 50% cost reduction without impacting SLA
```

**3. Right-Sizing Instances:**

```text
Over-Provisioned (Common Mistake):
├─ Instance: c5.2xlarge (8 vCPU, 16 GB)
├─ Actual usage: 2 vCPU, 4 GB
├─ Cost: $0.34/hour
└─ Waste: Paying 4x more than needed!

Right-Sized:
├─ Instance: c5.large (2 vCPU, 4 GB)
├─ Usage: 1.8 vCPU, 3.5 GB
├─ Cost: $0.085/hour
└─ Savings: 75% reduction

Monitoring for Right-Sizing:
├─ Track CPU/memory utilization over 30 days
├─ Identify consistently <50% utilized instances
├─ Downsize in stages (test performance)
└─ Result: Typical savings of 30-40%
```

#### Advanced Storage Optimization

**Compression:**

```text
Uncompressed Submissions:
├─ Average code: 2 KB
├─ 500K/day: 1 GB/day
└─ Yearly: 365 GB

Gzip Compression:
├─ Compression ratio: 5:1 (text compresses well)
├─ Compressed size: 0.4 KB per submission
├─ Yearly: 73 GB
└─ Savings: 80% reduction

Trade-offs:
├─ CPU cost: ~1ms per compression/decompression
├─ Storage savings: $20/TB/month × 0.3 TB = $6/month saved
├─ CPU cost: Negligible (batched compression)
└─ Decision: Always compress!
```

**Tiered Storage:**

```text
Hot Tier (Recent submissions - frequently accessed):
├─ Last 7 days: ~3.5 GB
├─ Storage: SSD (fast access)
├─ Cost: $0.10/GB/month = $0.35/month
└─ Access pattern: 80% of reads

Warm Tier (1-6 months old - occasionally accessed):
├─ 6 months: ~175 GB
├─ Storage: HDD
├─ Cost: $0.02/GB/month = $3.50/month
└─ Access pattern: 15% of reads

Cold Tier (>6 months - rarely accessed):
├─ 2+ years: ~730 GB
├─ Storage: S3 Glacier
├─ Cost: $0.004/GB/month = $2.92/month
└─ Access pattern: 5% of reads

Total Cost:
├─ All SSD: $100/month
├─ Tiered: $6.77/month
└─ Savings: 93% reduction!
```

#### Disaster Recovery & Backup Sizing

**Backup Requirements:**

```text
RPO (Recovery Point Objective): 1 hour
├─ Max acceptable data loss: 1 hour of submissions
├─ That's: 500K/day ÷ 24 = ~21K submissions
└─ Backup frequency: Hourly database snapshots

RTO (Recovery Time Objective): 30 minutes
├─ Max acceptable downtime: 30 minutes
├─ Requirements:
   ├─ Hot standby database (replica)
   ├─ Pre-warmed worker pool in backup region
   └─ Automated failover scripts
└─ Cost: 2x infrastructure (primary + backup)

Backup Storage:
├─ Daily full backup: 1.5 TB
├─ Incremental backups: ~10 GB/day
├─ Retention: 30 days
├─ Total: 1.5 TB + (10 GB × 30) = 1.8 TB
└─ Cost: 1.8 TB × $0.023/GB (S3) = $41/month
```

---

### Real-World Example: LeetCode's Infrastructure Scaling

**LeetCode's Growth Journey:**

```text
2015:
├─ DAU: 1,000
├─ Infrastructure: 1 server (app + database + judge)
├─ Cost: $100/month
└─ Storage: 10 GB

2017:
├─ DAU: 50,000
├─ Infrastructure: 3 app servers, 1 database, 20 judge workers
├─ Cost: $3,000/month
└─ Storage: 500 GB

2020:
├─ DAU: 500,000
├─ Infrastructure: 30 app servers, sharded database, 200 workers
├─ Cost: $40,000/month
└─ Storage: 10 TB

2023:
├─ DAU: 2,000,000
├─ Infrastructure: Kubernetes (auto-scaling), 500+ workers
├─ Cost: $150,000/month
└─ Storage: 50 TB

Key Insight: Cost scales sub-linearly with users due to:
├─ Better caching (higher cache hit rate at scale)
├─ Reserved instance discounts
├─ Infrastructure efficiency improvements
└─ Revenue from premium users offsets costs
```

**Their Scaling Strategies:**

1. **Caching Everything:**
   - Problems cached indefinitely (rarely change)
   - Test cases cached for 24 hours
   - User sessions in Redis
   - Result: 95% cache hit rate, DB load reduced 20x

2. **Lazy Loading:**
   - Don't provision 100 language runtimes on every worker
   - Load top 10 languages (Python, Java, C++, JavaScript)
   - Pull others on-demand (2-second delay acceptable)
   - Result: Worker startup time reduced from 30s to 5s

3. **Smart Sharding:**
   - Submissions sharded by user_id (keeps user data together)
   - 16 shards initially, can expand to 256
   - Each shard: 5M submissions
   - Result: Linear scalability, no single database bottleneck

---

### 🎯 Interview Questions: Capacity Planning

**Question 1:** How would you estimate storage needs for a platform that allows users to upload video explanations for their solutions (1-5 minute videos)?

<details>
<summary>💡 Hint</summary>
Think about video file sizes (much larger than code!), compression, and whether all users would upload videos.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Assumptions:**
```text
- 10% of submissions include video (not everyone records)
- Average video length: 3 minutes
- Video quality: 720p at 2 Mbps bitrate
- Daily submissions: 500K
```

**Calculations:**
```text
Video size:
├─ 3 minutes × 2 Mbps = 6 Mb = 0.75 MB per second
├─ 3 min × 60 sec × 0.75 MB = 135 MB per video (uncompressed)
├─ H.264 compression: ~5:1 ratio
└─ Compressed: ~27 MB per video

Daily video submissions:
├─ 500K submissions × 10% = 50K videos
├─ 50K × 27 MB = 1.35 TB per day
└─ Yearly: 1.35 TB × 365 = 493 TB ≈ 500 TB/year!

Storage cost:
├─ S3 storage: $0.023/GB/month
├─ 500 TB = 500,000 GB
├─ Monthly cost: 500,000 × $0.023 = $11,500/month
└─ Just for storage! Plus egress costs for viewing.

Optimization strategies:
1. Tiered storage (old videos to Glacier)
2. Client-side compression before upload
3. Charge premium users for video storage
4. Delete videos after 1 year (save 80% cost)
5. Use CDN with video-specific compression (reduce egress cost)

Conclusion:
├─ Video storage dominates costs (1000x more than code)
├─ Need revenue model to justify (premium feature)
└─ Architecture must use object storage (S3), not database
```
</details>

**Question 2:** During a major contest, submission rate spikes to 5,000 QPS (100x normal). How do you ensure the system doesn't crash?

<details>
<summary>💡 Hint</summary>
Think about queue depth, worker scaling, database bottlenecks, and graceful degradation.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Immediate Problem:**
```text
Normal: 18 QPS, 250 workers
Contest: 5,000 QPS, need 250 × (5000/18) = 69,444 workers!
├─ Can't provision 70K workers instantly
└─ Queue will build up massively
```

**Solution: Multi-Layer Defense**

**1. Queue Management:**
```text
Use message queue (Kafka/RabbitMQ):
├─ Buffer submissions when workers busy
├─ Queue capacity: 1M messages
├─ At 5K QPS, fills in: 1M / 5K = 200 seconds
├─ Gives time to scale workers
└─ Set queue TTL: 5 minutes (submissions expire)
```

**2. Auto-Scaling:**
```text
Pre-provision before contest:
├─ Schedule: Contest at 8 PM Saturday
├─ Scale up at 7:30 PM: 1,000 workers (4x normal)
├─ Auto-scale during contest: 
   ├─ Trigger: Queue depth > 10K
   ├─ Action: Add 500 workers
   ├─ Max: 5,000 workers (cost cap)
└─ Scale down after: Gradual reduction over 2 hours
```

**3. Priority Queues:**
```text
Two queue tiers:
├─ High priority: Contest submissions (SLA <2s)
├─ Low priority: Practice submissions (SLA <30s)
├─ Worker allocation: 80% contest, 20% practice
└─ Prevents practice degrading contest experience
```

**4. Database Protection:**
```text
Write path optimization:
├─ Batch writes: Buffer 100 submissions, bulk insert
├─ Async writes: Return result before DB write completes
├─ Sharding: Distribute writes across 16 shards
└─ Result: 5K QPS → 313 QPS per shard (manageable)

Read path optimization:
├─ Cache all contest problems (won't change)
├─ Cache leaderboard for 10 seconds (eventual consistency OK)
├─ Read replicas for leaderboard queries
└─ Result: 90% cache hit rate, DB untouched
```

**5. Graceful Degradation:**
```text
If system still overwhelmed:
├─ Disable non-essential features (discussions, profiles)
├─ Rate limit aggressively (5 submissions/minute)
├─ Show queue position: "Your submission is #1,234 in queue"
└─ Better than crashing!
```

**Cost Analysis:**
```text
Normal: 250 workers × $0.17/hour = $42.50/hour
Contest (2 hours): 5,000 workers × $0.17 × 2 = $1,700
├─ Use spot instances: 70% discount = $510
├─ Cost per contest: $500
├─ Revenue from contest entries: $5,000 (1,000 × $5)
└─ ROI: 10x positive
```
</details>

**Question 3:** The system needs to support 50 programming languages instead of 4. How does this affect capacity planning?

<details>
<summary>💡 Hint</summary>
Think about Docker image sizes, worker memory, and the probability distribution of language usage.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Impact Analysis:**

**1. Storage (Docker Images):**
```text
Current (4 languages):
├─ Python: 500 MB
├─ Java: 700 MB
├─ C++: 400 MB
├─ JavaScript: 300 MB
└─ Total per worker: 1.9 GB

50 languages:
├─ Average image size: 500 MB
├─ Total: 50 × 500 MB = 25 GB per worker
├─ 250 workers: 250 × 25 GB = 6.25 TB
└─ Problem: Too large, slow provisioning
```

**2. Memory:**
```text
Current: 512 MB per worker (loads 4 runtimes)
50 languages: Can't load all in memory

Solution: Lazy loading
├─ Load only on first use
├─ Keep in memory after first load
├─ Evict least-used after 1 hour idle
└─ Memory: Still 512 MB (holds ~5 runtimes)
```

**3. Worker Specialization:**
```text
Pareto Principle applies:
├─ Top 10 languages: 90% of submissions
   ├─ Python, Java, C++, JavaScript, C, Ruby, Go, Rust, Swift, Kotlin
   └─ Workers: Generalist (have these pre-loaded)
├─ Other 40 languages: 10% of submissions
   ├─ Haskell, Scala, Perl, Erlang, etc.
   └─ Workers: Specialist (on-demand loading)

Worker Pool Strategy:
├─ 200 generalist workers (always ready)
├─ 50 specialist workers (lazy load)
├─ Total: 250 workers (same as before!)
└─ Result: No capacity increase needed
```

**4. Provisioning Strategy:**
```text
Don't pre-load all 50 languages:

Tier 1 (Hot): Pre-loaded on all workers
├─ Python, Java, C++, JavaScript
├─ Startup time: <1 second
└─ Handles 80% of traffic

Tier 2 (Warm): Loaded on first use, cached
├─ C, Ruby, Go, Rust, Swift, Kotlin
├─ First-time startup: 5 seconds (acceptable)
├─ Cached startup: <1 second
└─ Handles 15% of traffic

Tier 3 (Cold): Pull image on-demand
├─ Haskell, Scala, Perl, etc. (40 languages)
├─ First-time startup: 30 seconds (acceptable for rare languages)
├─ Cached startup: <1 second
└─ Handles 5% of traffic
```

**5. Cost Impact:**
```text
Storage:
├─ Current: 1.9 GB × 250 = 475 GB
├─ Optimized (tiered): 5 GB × 250 = 1.25 TB
├─ Cost increase: 1.25 TB × $0.10/GB = $125/month
└─ Negligible compared to compute

Compute:
├─ No change (same worker count)
├─ Lazy loading prevents overhead
└─ Cost: $0 increase
```

**Implementation Timeline:**
```text
Phase 1: Add top 6 languages (Python, Java, C++, JS, C, Ruby)
├─ Effort: 2 weeks
├─ Impact: Covers 85% of users
└─ Risk: Low

Phase 2: Add next 10 languages (Go, Rust, Swift, etc.)
├─ Effort: 1 month
├─ Impact: Covers 95% of users
└─ Risk: Medium (more complex build pipeline)

Phase 3: Long tail (40 languages)
├─ Effort: 3 months
├─ Impact: Covers 100% of users
├─ Risk: High (many edge cases)
└─ Defer until product-market fit proven
```
</details>

---

### 🤔 Think About It

Before moving to the next section, consider these questions:

1. **Why does the read/write ratio (80:20) matter for capacity planning?**
   - Hint: Think about caching strategy and database load

2. **If you had unlimited budget, would you over-provision infrastructure by 10x "just to be safe"? Why or why not?**
   - Hint: Consider cost, complexity, and what you learn from operating at capacity

3. **How would capacity planning differ for a B2C platform (LeetCode) vs B2B (HackerRank for enterprise hiring)?**
   - Hint: Think about predictability, SLAs, and cost sensitivity

4. **What's the risk of underestimating contest traffic by 2x? What about 10x?**
   - Hint: Queue behavior, user experience, brand damage

---

### ✅ Key Takeaways

🎯 **Core Calculations:**
- QPS = (Daily Operations / 86,400 seconds) × Peak Multiplier
- Storage = (Daily New Data × 365 days × Years) + Static Data
- Workers = (Peak QPS × Avg Execution Time) × Safety Buffer

📊 **Scale Insights:**
- 100K DAU is modest - single database can handle it
- Submissions dominate storage (1.5 TB over 3 years)
- Judge workers are main cost driver ($7,800/month)
- Bandwidth is negligible (<2 Mbps)

🔧 **Optimization Strategies:**
- Caching: 95% hit rate possible for problems
- Compression: 80% storage reduction for code
- Tiered storage: 93% cost reduction
- Spot instances: 70% compute cost reduction
- Reserved instances: 40% discount for baseline capacity

💡 **Interview Tips:**
- Always round numbers (100K not 97,344)
- Show your work (write calculations on whiteboard)
- Acknowledge uncertainty (20-30% margin of error is fine)
- Discuss trade-offs (cost vs performance)

---

### 🎯 Practice Exercise

**Challenge:** Capacity planning for a niche platform

**Scenario:**
You're building a specialized coding platform for competitive programming contests only (no practice problems). Key differences:
- Contests run 2-4 hours, twice a week
- 50,000 simultaneous users during contest
- Rest of week: nearly zero traffic
- Very spiky traffic pattern

**Your Task:**
1. Calculate QPS during contest vs off-hours
2. Estimate worker pool size needed
3. Propose auto-scaling strategy
4. Calculate monthly cost (can't keep 1000s of workers idle!)

**Expected Output Format:**
```text
Traffic Analysis:
├─ Contest QPS: [calculation]
├─ Off-hours QPS: [calculation]
└─ Peak multiplier: [calculation]

Worker Pool:
├─ Baseline (off-hours): [number] workers
├─ Contest (peak): [number] workers
└─ Scaling strategy: [description]

Cost Optimization:
├─ Reserved instances: [number] workers
├─ On-demand/Spot: [number] workers
├─ Monthly cost: [calculation]
└─ Cost per contest: [calculation]

Justification:
[Why this approach is optimal for spiky traffic]
```

<details>
<summary>📝 Sample Solution</summary>

**Traffic Analysis:**
```text
Contest (4 hours, twice a week):
├─ Participants: 50,000
├─ Submissions per user per contest: 10
├─ Contest duration: 4 hours = 14,400 seconds
├─ Total submissions: 50,000 × 10 = 500,000
├─ QPS: 500,000 / 14,400 ≈ 35 QPS
├─ With test runs (2x): 70 QPS
└─ Peak (first/last 30 minutes, 5x): 350 QPS

Off-hours (162 hours per week):
├─ Usage: Nearly zero
├─ QPS: <1 QPS (just monitoring, cron jobs)
└─ Workers needed: 5 (minimal)

Peak multiplier: 350 / 1 = 350x spike!
```

**Worker Pool:**
```text
Baseline (off-hours):
├─ Needed: 5 workers
├─ Type: Reserved instances (always running)
├─ Cost: 5 × $73/month = $365/month
└─ Purpose: Keep system warm, monitoring

Contest (peak):
├─ Peak QPS: 350
├─ Avg execution: 3 seconds
├─ Concurrent: 350 × 3 = 1,050
├─ Buffer (2x): 2,100 workers needed!
└─ Only for 8 hours per week

Scaling strategy:
├─ Pre-provision 1 hour before contest: 500 workers (warm pool)
├─ Auto-scale during contest: Up to 2,000 workers
├─ Scale trigger: Queue depth > 5,000
├─ Use Spot instances (70% discount, acceptable interruption risk)
└─ Scale down immediately after contest (within 30 minutes)
```

**Cost Optimization:**
```text
Reserved instances (baseline):
├─ 5 workers × $73/month = $365/month
└─ Running 730 hours/month

Spot instances (contests):
├─ 2,000 workers × $0.05/hour (spot price)
├─ 8 hours/week × 4 weeks = 32 hours/month
├─ Cost: 2,000 × $0.05 × 32 = $3,200/month
└─ Running only during contests

Total monthly cost:
├─ Reserved: $365
├─ Spot: $3,200
├─ Database/API servers: ~$1,000
├─ Storage: ~$100
└─ Total: $4,665/month

Cost per contest:
├─ 8 contests/month
├─ Variable cost: $3,200 / 8 = $400/contest
└─ Fixed cost: $1,465 / 8 ≈ $183/contest
└─ Total: ~$583/contest
```

**Justification:**
```text
Why this approach is optimal:

1. Cost efficiency:
   ├─ On-demand for 2,000 workers: $0.17/hour × 2,000 × 32 = $10,880/month
   ├─ Spot instances: $3,200/month
   └─ Savings: $7,680/month (70% reduction!)

2. Risk mitigation:
   ├─ Baseline workers (reserved): Always available, no interruption
   ├─ Contest workers (spot): Interruption acceptable (job requeues)
   ├─ 20% on-demand buffer: For critical contest traffic if spot unavailable
   └─ Total reliability: 99.9%

3. Predictable traffic:
   ├─ Contest schedule known in advance
   ├─ Pre-provisioning avoids cold start
   ├─ Auto-scaling handles variance
   └─ No surprises

4. Alternative considered (rejected):
   ├─ Keep 2,000 workers always: $248,000/month (53x more expensive!)
   ├─ Pure on-demand: $10,880/month (2.3x more expensive)
   ├─ Serverless (Lambda): Cold start latency unacceptable for contests
   └─ Our approach: Best balance of cost and performance
```
</details>

**Time Budget:** 30-40 minutes

**Difficulty:** 🔴 Advanced

---

*Next Up:* In Section 3, we'll design the high-level system architecture. We'll draw diagrams showing how all the pieces fit together - API servers, databases, judge workers, and more. Ready to become a systems architect? 📐


## Section 3: Designing the System Architecture

### What You'll Learn

By the end of this section, you'll be able to:
- Draw a high-level architecture diagram for a coding platform
- Explain the role of each component in the system
- Understand data flow for both problem browsing and code submission
- Identify the critical path that affects user experience

### Why This Matters

The architecture diagram is the foundation of your system design. It's the blueprint that guides all implementation decisions. Real-world example: A coding platform initially designed their judge system as a synchronous service (blocking API calls). This created cascading failures during contests when submission spikes occurred. Redesigning to an async architecture with message queues solved the problem but required 6 months of refactoring. Getting the architecture right from the start saves massive rework!

---

### 🟢 For Beginners: The Fundamentals

#### What Does "System Architecture" Mean?

Think of system architecture like a city map. Just as a city has different zones (residential, commercial, industrial), our coding platform has different components that handle specific jobs.

**City Analogy:**
```text
City Layout                      Coding Platform
─────────────────────────────   ─────────────────────────────
🏠 Residential (people live)  →  👤 Users (write code)
🏪 Stores (sell goods)        →  💻 API Servers (handle requests)
�� Factories (make products)  →  ⚙️ Judge Workers (run code)
🏦 Bank (store money)         →  💾 Database (store submissions)
📮 Post Office (delivery)     →  📨 Message Queue (task routing)
```

#### The Big Picture

Let's start with a simple view:

```text
USER'S JOURNEY:
1. Open website → 2. Browse problems → 3. Write code → 4. Submit → 5. Get results

SYSTEM COMPONENTS INVOLVED:
1. Web Browser (client)
2. Load Balancer (traffic director)
3. API Servers (handle requests)
4. Database (store data)
5. Judge System (run code)
6. Cache (speed things up)
```

#### Component-by-Component Explanation

**1. Users/Clients (The Front Door)**

This is you! Users access the platform through:
- Web browser (most common)
- Mobile app (iOS/Android)
- API clients (for integrations)

What they do:
- Browse problems
- Write code in online editor
- Submit solutions
- View results and statistics

**2. CDN (Content Delivery Network) - The Fast Delivery Truck**

Think of CDN like Amazon Prime warehouses in every city. Instead of shipping from one central warehouse, they keep products close to you for faster delivery.

For our platform:
```text
Without CDN:
User in Tokyo → Request to San Francisco → 200ms latency 😞

With CDN:
User in Tokyo → CDN server in Tokyo → 20ms latency 😊
```

What's cached in CDN:
- Static assets (CSS, JavaScript, images)
- Problem descriptions (rarely change)
- User interface components

**3. Load Balancer (The Traffic Cop)**

Imagine a mall with multiple entrances. The load balancer is like a security guard who directs you to the least crowded entrance.

```text
Incoming Traffic:
    ↓
Load Balancer (decides which server to use)
    ├─→ API Server 1 (60% busy)  ← chosen!
    ├─→ API Server 2 (90% busy)
    └─→ API Server 3 (50% busy)
```

Benefits:
- No single server gets overwhelmed
- If one server crashes, others keep working
- Can add more servers during busy times

**4. API Servers (The Receptionists)**

API servers are like hotel receptionists - they handle your requests:
- "Show me problem #42" → Fetch from database
- "Submit my code" → Send to judge system
- "Show my profile" → Fetch user data

They DON'T execute your code (that's dangerous!). They just coordinate.

**5. Judge System (The Code Executors)**

This is the heart of the platform! The judge system:
1. Receives your code
2. Runs it in a safe container (isolated sandbox)
3. Tests it against test cases
4. Returns results

Think of it like a science lab with safety equipment:
```text
Your Code → Put in sealed container → Run experiment → Report results
           (Can't affect outside)
```

**6. Message Queue (The Waiting Line)**

When many people submit code at once (like during a contest), we need a queue system.

Analogy: Coffee shop during morning rush
```text
Without Queue:
10 customers arrive → Barista overwhelmed → Chaos!

With Queue:
10 customers arrive → Get number → Wait in order → Served systematically
```

For submissions:
```text
Peak time: 100 submissions per second
Judge workers: Can handle 50/second
Queue: Buffers the extra 50 until workers are ready
Result: No one gets an error, just waits a bit longer
```

**7. Database (The Filing Cabinet)**

Stores all the data:
- User accounts and profiles
- Problems and test cases
- Submissions and results
- Statistics and rankings

We'll actually use multiple databases:
- **PostgreSQL**: For structured data (users, problems, submissions)
- **Redis**: For fast caching (active sessions, popular problems)
- **S3 (Object Storage)**: For large files (code files, test case files)

**8. Cache (The Speed Booster)**

Caching is like keeping frequently used items on your desk instead of in a filing cabinet across the room.

```text
Without Cache:
Every problem view → Database query (50ms)
1000 views → 50,000ms = 50 seconds of DB time!

With Cache:
First view → Database (50ms) → Store in cache
Next 999 views → Cache (1ms each) → 999ms
Total: 50ms + 999ms = 1,049ms = 1 second!
```

Popular problems (like "Two Sum") might be viewed 1000 times/hour. We cache them!

---

### The Complete Flow

**Flow 1: Browsing Problems (Read Path)**

```text
1. User clicks "Show problems"
   ↓
2. Request hits Load Balancer
   ↓
3. Routed to API Server
   ↓
4. API Server checks Cache (Redis)
   ├─ HIT: Return cached problem list (fast! 5ms)
   └─ MISS: Query Database → Cache result → Return (slower, 50ms)
   ↓
5. Response sent back to user
   ↓
6. User sees problem list

Total time: 5-50ms (barely noticeable!)
```

**Flow 2: Submitting Code (Write Path)**

```text
1. User writes code and clicks "Submit"
   ↓
2. Code sent to Load Balancer
   ↓
3. Routed to API Server
   ↓
4. API Server validates:
   - Is user logged in?
   - Is code size < 50KB?
   - Has user hit rate limit (10/min)?
   ↓
5. API Server stores submission in Database (status: "pending")
   ↓
6. API Server sends task to Message Queue (RabbitMQ/Kafka)
   ↓
7. API Server returns to user: "Submission received! Status: Pending"
   (User sees: "Judging... ⏳")
   ↓
8. Judge Worker picks up task from Queue
   ↓
9. Judge Worker:
   - Creates isolated Docker container
   - Copies code into container
   - Runs code against test cases
   - Measures time and memory usage
   - Destroys container
   ↓
10. Judge Worker updates Database (status: "Accepted" or "Wrong Answer")
    ↓
11. User's browser polls API every 1 second OR gets WebSocket update
    ↓
12. User sees result: "Accepted ✓ Runtime: 45ms, Memory: 12MB"

Total time: 1-3 seconds (acceptable!)
```

💡 **Pro Tip:** Notice how submission is async (doesn't block)? This prevents the API server from freezing while code executes. The user's browser polls for updates or uses WebSockets for real-time results!

---

### 🟡 For Intermediate: Interview Patterns

#### Drawing the Architecture Diagram

In a whiteboard interview, you'd draw this step-by-step:

**Step 1: Start with Clients**
```text
[Clients]
```

**Step 2: Add Load Balancing**
```text
[Clients]
    ↓
[Load Balancer]
```

**Step 3: Add API Layer**
```text
[Clients]
    ↓
[Load Balancer]
    ↓
[API Servers] (3 instances)
```

**Step 4: Add Core Services**
```text
[Clients]
    ↓
[Load Balancer]
    ↓
[API Servers] (3 instances)
    ↓
┌─────────┬──────────┬────────────┐
│         │          │            │
[Cache] [Database] [Queue] [Judge System]
```

**Step 5: Add Details**
```text
                [Clients]
                    ↓
                  [CDN]
                    ↓
            [Load Balancer]
                    ↓
        ┌───────────┼───────────┐
        ↓           ↓           ↓
   [API Server 1][API Server 2][API Server 3]
        │           │           │
        └───────────┼───────────┘
                    ↓
    ┌───────────────┼───────────────┐
    ↓               ↓               ↓
[Redis Cache]   [PostgreSQL]   [RabbitMQ Queue]
                                    ↓
                            [Judge Workers Pool]
                            (250 Docker containers)
                                    ↓
                                [S3 Storage]
                            (Test cases, code)
```

#### Critical Design Decisions to Explain

When presenting this architecture, call out these key decisions:

**Decision 1: Sync vs Async Execution**
```text
Interviewer: "Why use a message queue instead of API calling judge directly?"

Answer: "Synchronous execution would block the API server:
- Problem: If code takes 10 seconds, API server is blocked for 10s
- At 100 submissions/sec, we'd need 1000 API servers (expensive!)
- Solution: Async with queue:
  - API server quickly enqueues task (10ms)
  - Judge workers process independently
  - Can scale judge workers separately from API servers
  - If judges are busy, queue buffers (graceful degradation)
  
Trade-off: Slightly more complex (need queue), but essential for scale."
```

**Decision 2: PostgreSQL vs NoSQL**
```text
Interviewer: "Why not use MongoDB/Cassandra?"

Answer: "Let's analyze our access patterns:
- Queries: Mostly by user_id or problem_id (simple lookups)
- Joins: Need to join users, problems, submissions (relational)
- Transactions: When updating stats, need ACID guarantees
- Scale: 500K submissions/day = 6 writes/sec (PostgreSQL can handle)

PostgreSQL is sufficient because:
- Strong consistency needed (contest rankings must be accurate)
- Complex queries (user statistics, leaderboards)
- Proven reliability
- Scale doesn't require NoSQL yet (could shard later if needed)

If we were at LeetCode scale (10M DAU), I'd consider:
- Cassandra for submissions (time-series, write-heavy)
- PostgreSQL for users/problems (read-heavy, relational)
- Hybrid approach for best of both worlds"
```

**Decision 3: Stateless API Servers**
```text
Interviewer: "Where do you store user sessions?"

Answer: "API servers are stateless - they don't store session data locally.

Stateful (BAD):
[User] → [API Server 1] (stores session in memory)
[User] → [Load Balancer] → [API Server 2] ← Session not here! ❌

Stateless (GOOD):
[User] → [API Server 1] → Check Redis for session ✓
[User] → [API Server 2] → Check Redis for session ✓

Benefits:
- Can scale API servers horizontally (add/remove anytime)
- Server crash doesn't lose sessions
- Load balancer can route to any server
- Simpler deployment (no sticky sessions needed)

Implementation:
- JWT tokens (self-contained, no server state needed), OR
- Session IDs stored in Redis (centralized, fast access)"
```

#### Data Flow Patterns

**Read-Heavy Operations (Problem Browsing):**
```text
90% of traffic is reads (browsing problems, viewing stats)

Optimization: Aggressive caching

Cache Layers:
1. Browser Cache (5 minutes)
   ├─ Static content (images, CSS, JS)
   └─ Immutable problem descriptions

2. CDN Cache (1 hour)
   ├─ Problem pages
   └─ Leaderboards

3. Redis Cache (10 minutes)
   ├─ Problem list (filtered by difficulty/tag)
   ├─ User statistics
   └─ Recent submissions

4. Database (source of truth)
   └─ Only hit if all caches miss

Result:
├─ Cache hit rate: 95%
├─ Database load: 5% of original
└─ Response time: 10ms instead of 50ms
```

**Write-Heavy Operations (Code Submissions):**
```text
10% of traffic is writes (code submissions)

Optimization: Async processing

Write Path:
1. API receives submission (fast: 10ms)
   ├─ Validate request
   ├─ Rate limit check
   └─ Write to database (status: pending)

2. Enqueue to RabbitMQ (fast: 5ms)
   └─ Return "received" to user immediately

3. Judge worker processes (slow: 1-3s)
   ├─ Run code in sandbox
   ├─ Execute test cases
   └─ Update database with results

4. User polls/WebSocket for result
   └─ Eventually receives "Accepted" or "Wrong Answer"

Benefit:
├─ API server not blocked
├─ Can scale workers independently
├─ Queue acts as buffer during spikes
└─ Graceful degradation (longer wait, not failure)
```

---

### 🔴 For Advanced: Production Considerations

#### Multi-Region Architecture

**Global Deployment for Low Latency:**

```text
Single Region (Problem):
User in India → US-East servers → 250ms latency 😞

Multi-Region (Solution):
User in India → India region → 20ms latency 😊

Architecture:
┌─────────────────────────────────────────────────┐
│                   Global Layer                   │
│  ┌──────────────┐                               │
│  │ Route 53 DNS │ (Geographic routing)          │
│  └──────────────┘                               │
└─────────────────────────────────────────────────┘
          ↓                    ↓
┌─────────────────┐  ┌─────────────────┐
│   US-East-1     │  │   AP-South-1    │
│                 │  │   (India)       │
│ [API Servers]   │  │ [API Servers]   │
│ [Judge Workers] │  │ [Judge Workers] │
│ [Redis Cache]   │  │ [Redis Cache]   │
│                 │  │                 │
│ [Read Replica]  │  │ [Read Replica]  │
│       │         │  │       │         │
│       ↓         │  │       ↓         │
└───────┼─────────┘  └───────┼─────────┘
        │                    │
        └────────┬───────────┘
                 ↓
    ┌────────────────────────┐
    │   Primary Database     │
    │   (US-East-1)          │
    │   Multi-AZ Replication │
    └────────────────────────┘
```

**Data Consistency Challenges:**

```text
Problem: User submits code in India, switches to US laptop
Expectation: Should see their submission

Eventual Consistency:
├─ Submission in India → Writes to primary DB (US)
├─ Replication lag: 100-500ms
├─ User queries US read replica → Might not see submission yet!
└─ Solution: Read-your-writes consistency

Implementation:
1. After write, set cookie: last_write_timestamp=1634567890
2. On read, if timestamp is recent (<5s ago):
   ├─ Read from primary (guaranteed to have latest data)
   └─ Slightly slower but consistent
3. If timestamp is old (>5s ago):
   ├─ Read from regional replica (faster)
   └─ Replication will have caught up
```

#### Service Mesh for Microservices

**Evolution from Monolith to Microservices:**

```text
Phase 1: Monolith (0-10K users)
┌──────────────────────────┐
│    Single Application    │
│  ┌────────────────────┐  │
│  │ User Management    │  │
│  │ Problem Service    │  │
│  │ Submission Service │  │
│  │ Judge Coordinator  │  │
│  └────────────────────┘  │
└──────────────────────────┘
Pro: Simple, Con: Can't scale parts independently

Phase 2: Microservices (10K-1M users)
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ User   │ │Problem │ │Submiss │ │ Judge  │
│Service │ │Service │ │Service │ │Service │
└────────┘ └────────┘ └────────┘ └────────┘
    ↓          ↓          ↓          ↓
┌──────────────────────────────────────┐
│           Service Mesh (Istio)       │
│  - Service discovery                 │
│  - Load balancing                    │
│  - Circuit breaker                   │
│  - Tracing & monitoring             │
└──────────────────────────────────────┘
Pro: Independent scaling, Con: Complexity
```

**Service Mesh Benefits:**

1. **Circuit Breaker Pattern:**
```text
Without Circuit Breaker:
Judge Service down → Submission Service keeps trying → All requests time out → Cascade failure

With Circuit Breaker:
Judge Service down (3 failures detected)
    ↓
Circuit OPEN (stop sending requests)
    ↓
Return cached response or error immediately
    ↓
Periodically test if Judge Service recovered
    ↓
Circuit CLOSED (resume normal operation)

Result: Failure is isolated, doesn't bring down entire system
```

2. **Retry Logic with Exponential Backoff:**
```text
Request fails → Wait 100ms → Retry
Request fails → Wait 200ms → Retry  
Request fails → Wait 400ms → Retry
Request fails → Wait 800ms → Give up

Prevents: Thundering herd problem where all retries hit at once
```

3. **Distributed Tracing:**
```text
User request → Trace ID: abc123
├─ API Server (5ms)
├─ Problem Service (10ms)
│  └─ Database query (8ms)
├─ Cache lookup (2ms)
└─ Total: 17ms

When slow: Can pinpoint which service is bottleneck
```

#### Advanced Caching Strategies

**Cache Invalidation (The Hard Problem):**

Phil Karlton said: "There are only two hard things in Computer Science: cache invalidation and naming things."

**Problem:**
```text
Scenario:
1. Problem #42 cached with description "Find maximum"
2. Admin updates description to "Find minimum"
3. Cache still shows old description!
4. Users see stale data until cache expires

This is unacceptable for a production system.
```

**Solutions:**

**1. Cache Invalidation on Write:**
```text
When problem updated:
├─ Write to database
├─ DELETE from cache (invalidate)
└─ Next read will miss cache → Fetch fresh from DB → Cache it

Code:
```python
def update_problem(problem_id, new_data):
    # 1. Update database
    db.update(problem_id, new_data)
    
    # 2. Invalidate cache
    cache.delete(f"problem:{problem_id}")
    cache.delete(f"problem_list:*")  # Also invalidate lists
    
    # 3. Optionally: Warm cache immediately
    problem = db.get(problem_id)
    cache.set(f"problem:{problem_id}", problem, ttl=3600)
```
```

**2. Cache-Aside Pattern with Versioning:**
```text
Key: problem:42:v5 (version included)

When updated:
├─ Increment version to v6
├─ Write problem:42:v6 to cache
└─ Old cache (v5) expires naturally

Benefit: No need to invalidate, old caches just expire
Challenge: Need to track versions
```

**3. Write-Through Cache:**
```text
Write path:
User updates problem
    ↓
Write to cache (with new data)
    ↓
Asynchronously write to database
    ↓
Cache is always fresh!

Trade-off:
├─ Pro: Cache always consistent
├─ Con: If cache fails, lose data (unless write to DB first)
└─ Mitigation: Write to DB first, then update cache
```

---

### Real-World Example: LeetCode's Architecture Evolution

**2015: Monolithic Architecture**
```text
Single Rails application:
├─ 1 server (app + database)
├─ No caching
├─ Synchronous code execution
└─ Cost: $200/month, Supported: 1K users

Problem: Couldn't scale beyond 5K users (server maxed out)
```

**2017: Basic Microservices**
```text
Separated concerns:
├─ API servers (Node.js): 3 instances
├─ Judge workers (Python): 20 containers
├─ PostgreSQL database: Single instance
├─ Redis cache: Problem caching
└─ Cost: $3K/month, Supported: 50K users

Problem: Database became bottleneck at 100K users
```

**2020: Distributed Architecture**
```text
Fully distributed:
├─ API servers: Auto-scaling (10-50 instances)
├─ Judge workers: Kubernetes (100-500 pods)
├─ Database: Sharded PostgreSQL (16 shards)
├─ Cache: Redis cluster (5 nodes)
├─ Message queue: Kafka (high throughput)
└─ Cost: $40K/month, Supported: 1M users

Key improvements:
├─ Database sharding (16x capacity)
├─ Kubernetes auto-scaling (handle spikes)
├─ Kafka for reliability (no lost submissions)
└─ Multi-region deployment (global coverage)
```

**2023: Global Platform**
```text
Current state:
├─ Multi-region: 5 regions (US-East, US-West, EU, India, Asia-Pacific)
├─ Microservices: 20+ services (user, problem, submission, contest, etc.)
├─ Judge workers: 1000+ pods globally
├─ Database: 64 shards, multi-master replication
├─ CDN: CloudFront with custom caching rules
└─ Cost: $150K/month, Supported: 10M users

Technology stack:
├─ Kubernetes (orchestration)
├─ Istio (service mesh)
├─ Prometheus + Grafana (monitoring)
├─ ELK stack (logging)
└─ Terraform (infrastructure as code)
```

---

### 🎯 Interview Questions: Architecture Design

**Question 1:** The judge system is experiencing high latency (5+ seconds) during peak hours. How would you diagnose and fix this?

<details>
<summary>💡 Hint</summary>
Think about different layers where latency could be introduced: queue depth, worker pool size, container startup time, test case fetching.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Diagnosis Process:**

**Step 1: Identify where time is spent**
```text
Total latency: 5 seconds
Break down:
├─ Queue wait time: ? seconds
├─ Container startup: ? seconds
├─ Code execution: ? seconds
└─ Test case loading: ? seconds

Use distributed tracing (Jaeger/Zipkin) to measure each phase
```

**Step 2: Likely culprits**
```text
Scenario A: Queue wait time is 4s
├─ Problem: Not enough judge workers
├─ Evidence: Queue depth keeps growing
└─ Fix: Scale up worker pool

Scenario B: Container startup is 3s
├─ Problem: Cold container creation
├─ Evidence: Slow for all languages
└─ Fix: Warm container pool

Scenario C: Test case loading is 2s
├─ Problem: Fetching from S3 is slow
├─ Evidence: Worse for problems with many test cases
└─ Fix: Cache test cases in worker memory

Scenario D: Code execution is 4s
├─ Problem: User code is legitimately slow OR timeout too generous
├─ Evidence: Certain problems consistently slow
└─ Fix: Reduce timeout OR optimize test cases
```

**Solution (Assuming Scenario B: Cold Start):**

**Current state:**
```text
Submit code → Create new container (3s) → Run code (1s) → Destroy container
Total: 4s
```

**Optimized: Warm Pool**
```text
Maintain pool of 100 pre-created containers
Submit code → Grab from pool (50ms) → Run code (1s) → Reset & return to pool
Total: 1.05s (4x faster!)

Implementation:
```python
class WarmContainerPool:
    def __init__(self, size=100):
        self.pool = Queue()
        self.size = size
        self._initialize_pool()
    
    def _initialize_pool(self):
        for _ in range(self.size):
            container = self._create_container()
            self.pool.put(container)
    
    def get_container(self, language):
        # Wait up to 1 second for available container
        container = self.pool.get(timeout=1)
        return container
    
    def return_container(self, container):
        # Reset container state
        container.reset()
        self.pool.put(container)
    
    def execute_code(self, code, language, test_cases):
        container = self.get_container(language)
        try:
            result = container.run(code, test_cases)
            return result
        finally:
            self.return_container(container)
```
```

**Cost-Benefit Analysis:**
```text
Cost:
├─ 100 idle containers × $0.02/hour = $48/day
└─ Monthly: $1,440

Benefit:
├─ Latency: 5s → 1s (5x improvement)
├─ User experience: Dramatically better
├─ Contest viability: Can handle real-time competition
└─ Increased usage: Better UX = more users = more revenue

ROI: Easily justified for production platform
```

**Alternative Solutions (Trade-offs):**

1. **Lambda/Serverless**
   ```text
   Pros:
   ├─ No warm pool maintenance
   ├─ Pay only for execution time
   └─ Auto-scaling built-in
   
   Cons:
   ├─ Cold start: 1-3s (worse than warm pool!)
   ├─ Execution time limit: 15 minutes (OK for coding problems)
   ├─ Cost at scale: Can be more expensive than containers
   └─ Less control over environment
   
   Verdict: Not ideal for low-latency judge system
   ```

2. **Long-Running Workers (No Containers)**
   ```text
   Pros:
   ├─ No startup overhead (0ms)
   └─ Very fast execution
   
   Cons:
   ├─ Security risk: User code could compromise worker
   ├─ State leakage: One submission could affect another
   ├─ Resource limits: Harder to enforce strictly
   └─ Complex isolation logic needed
   
   Verdict: Too risky without perfect sandboxing
   ```

3. **gVisor (Lightweight Sandbox)**
   ```text
   Pros:
   ├─ Faster than Docker (50ms startup vs 3s)
   ├─ Strong isolation (kernel-level)
   └─ Lower resource overhead
   
   Cons:
   ├─ More complex to operate
   ├─ Compatibility issues with some languages
   └─ Requires expertise to troubleshoot
   
   Verdict: Best for large-scale platforms (LeetCode uses this!)
   ```

</details>

**Question 2:** Design an architecture that supports real-time collaborative coding (like Google Docs but for code). How does this change the system?

<details>
<summary>💡 Hint</summary>
Think about WebSockets for real-time communication, operational transformation for concurrent edits, and consistency challenges.
</details>

<details>
<summary>✅ Sample Answer</summary>

**New Requirements:**
```text
Traditional (single user):
├─ User writes code locally
├─ Submits when ready
└─ One-way communication

Collaborative (multiple users):
├─ Multiple users edit same code simultaneously
├─ See each other's cursors and changes in real-time
├─ Conflict resolution (if both edit same line)
└─ Two-way, low-latency communication required
```

**Architecture Changes:**

**1. Add WebSocket Layer:**
```text
Current:
[Client] → HTTP requests → [API Server]

New:
[Client] ←→ WebSocket connection ←→ [Collaboration Server]
    ↓                                        ↓
[Buffer local changes]              [Broadcast to other clients]
```

**2. Operational Transformation (OT) Server:**
```text
Problem: Two users edit same line

User A: "hello" → Insert "x" at position 2 → "hexllo"
User B: "hello" → Insert "y" at position 3 → "helyklo"

Without OT:
├─ Both changes sent to server
├─ Final state: Undefined! Could be "hexyllo" or "heyxllo"
└─ Data corruption!

With OT:
├─ Server receives both operations
├─ Transforms operations to be compatible
├─ User A's operation executed first
├─ User B's operation transformed (position adjusted)
└─ Final state: "hexylllo" (consistent for all users)

Libraries:
├─ ShareDB (Node.js)
├─ Yjs (JavaScript, excellent for code editors)
└─ Automerge (CRDT-based, simpler conflict resolution)
```

**3. New Microservice:**
```text
[Collaboration Service]
├─ Handles WebSocket connections
├─ Manages editing sessions (rooms)
├─ Applies operational transformation
├─ Broadcasts changes to participants
└─ Persists snapshots periodically

Technology:
├─ Node.js with Socket.io (WebSocket library)
├─ Redis for presence (who's online)
├─ Yjs for OT algorithm
└─ MongoDB for session snapshots
```

**4. Scalability Challenges:**

**Connection Management:**
```text
Problem:
├─ WebSocket connections are stateful
├─ 10K concurrent editing sessions
├─ Each server can handle 10K connections
└─ But user must stay connected to same server!

Solution:
├─ Use sticky sessions (load balancer pins user to server)
├─ OR use Redis Pub/Sub to sync across servers:

[User A] ←→ [Server 1] ←→ [Redis Pub/Sub] ←→ [Server 2] ←→ [User B]

When User A types:
1. Server 1 receives change
2. Publishes to Redis channel "session:123"
3. Server 2 subscribed to that channel
4. Server 2 pushes change to User B
```

**5. Complete Flow:**

```text
User A types "x":
    ↓
[1] WebSocket message to Collaboration Server
    {
      sessionId: "sess-123",
      userId: "user-a",
      operation: {
        type: "insert",
        position: 5,
        content: "x"
      }
    }
    ↓
[2] Collaboration Server applies OT
    - Transform against concurrent operations
    - Update document state
    - Generate sequence number
    ↓
[3] Broadcast to all participants
    Via Redis Pub/Sub to other servers
    Via WebSocket to connected clients
    ↓
[4] User B receives:
    {
      from: "user-a",
      operation: {
        type: "insert",
        position: 5,
        content: "x"
      },
      seqNo: 142
    }
    ↓
[5] User B's editor applies operation
    - Cursor position adjusted if needed
    - Shows User A's cursor at position 5
    - Renders "x" at position 5
```

**6. Data Model:**

```sql
-- Editing Sessions
CREATE TABLE editing_sessions (
    session_id UUID PRIMARY KEY,
    problem_id UUID NOT NULL,
    created_by UUID NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    last_snapshot JSONB,  -- Latest code state
    snapshot_seq_no INT DEFAULT 0
);

-- Session Participants
CREATE TABLE session_participants (
    session_id UUID,
    user_id UUID,
    joined_at TIMESTAMP DEFAULT NOW(),
    last_seen_at TIMESTAMP,
    cursor_position INT,
    PRIMARY KEY (session_id, user_id)
);

-- Operation Log (for replay/recovery)
CREATE TABLE operation_log (
    session_id UUID,
    seq_no INT,
    user_id UUID,
    operation JSONB,
    timestamp TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (session_id, seq_no)
);
```

**7. Cost Impact:**

```text
New infrastructure:
├─ Collaboration servers: 10 instances × $200/month = $2,000
├─ Redis Pub/Sub: $500/month
├─ MongoDB for snapshots: $300/month
├─ Increased bandwidth (real-time sync): $500/month
└─ Total: $3,300/month additional

Optimization:
├─ Only enable for premium users (revenue offset)
├─ Auto-close sessions after 1 hour idle
├─ Limit participants per session (max 10 users)
└─ Use WebRTC for direct peer-to-peer (reduce server load)
```

**8. Alternative: WebRTC (Peer-to-Peer):**

```text
Instead of server relay:
[User A] ←→ Direct connection ←→ [User B]

Pros:
├─ Lower latency (no server hop)
├─ Lower server cost (only signaling needed)
└─ Better for small groups (2-5 users)

Cons:
├─ Doesn't work through restrictive firewalls
├─ Hard to manage for large groups (10+ users)
├─ No central authority (harder to persist/moderate)
└─ NAT traversal complexity

Verdict: Hybrid approach
├─ Use WebRTC for 2-3 user sessions
└─ Use server relay for larger groups or strict firewalls
```

</details>

---

### 🤔 Think About It

Before moving to the next section, consider:

1. **Why is the message queue crucial for handling spikes in code submissions?**
   - Hint: What happens without it when 1000 submissions arrive in 1 second?

2. **Could we use a single monolithic server for 100K DAU? Why or why not?**
   - Hint: Think about failure scenarios and scalability limits

3. **What would happen if the cache layer failed completely?**
   - Hint: Would the system still work? How would performance be affected?

4. **Why separate API servers from judge workers instead of running code execution in the API process?**
   - Hint: Security, resource isolation, scaling

---

### ✅ Key Takeaways

🎯 **Architecture Principles:**
- Separation of concerns: API, execution, storage are separate
- Async processing: Don't block user-facing requests
- Stateless services: Can scale horizontally easily
- Caching layers: Reduce database load by 20x

🏗️ **Core Components:**
- **CDN**: Static content delivery
- **Load Balancer**: Distribute traffic
- **API Servers**: Handle business logic (stateless!)
- **Message Queue**: Buffer and route tasks
- **Judge Workers**: Execute code in isolation
- **Cache**: Speed up reads (Redis)
- **Database**: Source of truth (PostgreSQL)
- **Object Storage**: Large files (S3)

🔄 **Data Flows:**
- **Read Path**: CDN → Cache → Database (95% cache hit rate)
- **Write Path**: API → Queue → Worker → Database (async)

💡 **Interview Tips:**
- Draw diagram incrementally (clients → LB → API → services)
- Explain WHY for each component
- Discuss trade-offs (sync vs async, SQL vs NoSQL)
- Mention scaling strategy for each layer

---

### 🎯 Practice Exercise

**Challenge:** Design the architecture for a subset of features

**Scenario:**
You're designing a minimal viable product (MVP) for a coding platform with just these features:
- Users can browse 100 problems (no search, no filters)
- Users can submit Python code only
- Results shown synchronously (user waits for result)
- No contests, no statistics

**Constraints:**
- Budget: $500/month maximum
- Expected users: 1,000 DAU
- Expected submissions: 5,000/day

**Your Task:**
1. Draw simplified architecture (fewer components than full system)
2. Choose specific technologies (PostgreSQL vs MongoDB, etc.)
3. Calculate server counts and costs
4. Justify why this is sufficient for MVP

<details>
<summary>📝 Sample Solution</summary>

**Simplified Architecture:**

```text
[Users]
   ↓
[Nginx Load Balancer]
   ($5/month - single t2.micro)
   ↓
[API Server] (2 instances for redundancy)
   ($20/month each - t2.small)
   ├─ Handles auth, problem listing
   └─ Executes Python code synchronously
   ↓
[PostgreSQL RDS]
   ($25/month - db.t3.micro)
   ├─ users table
   ├─ problems table  
   └─ submissions table
   ↓
[S3 Bucket]
   ($5/month)
   └─ Stores problem descriptions, test cases

Total: $75/month (well under $500 budget!)
```

**Justification:**

**1. No separate judge workers:**
```text
Why: Synchronous execution is acceptable for MVP
├─ 5,000 submissions/day = 0.06 QPS
├─ Even at 3s execution time, single server can handle it
├─ Users wait 3s for result (acceptable for MVP)
└─ Saves infrastructure cost and complexity

Trade-off: Can't scale to 100K users, but that's not the goal
```

**2. No caching layer (Redis):**
```text
Why: Database can handle the load
├─ 1,000 DAU × 10 problem views = 10,000 views/day
├─ = 0.12 QPS
├─ PostgreSQL can handle 1000s of QPS
└─ Save $25/month on Redis

Trade-off: Slower responses (50ms vs 5ms), but acceptable for MVP
```

**3. Single region (no CDN):**
```text
Why: Users likely in same geography initially
├─ If all users in US, host in us-east-1
├─ Latency: 50-100ms (acceptable)
└─ Save $50/month on CDN

Trade-off: Slower for international users, but MVP likely local
```

**4. PostgreSQL over NoSQL:**
```text
Why: Simple, relational data
├─ 100 problems (tiny dataset)
├─ 5,000 submissions/day (trivial write load)
├─ Familiar SQL queries
└─ Can easily scale to 100K submissions/day later

Cost: $25/month for managed RDS (auto-backups, auto-updates)
```

**5. Python only:**
```text
Why: Simplifies infrastructure
├─ Single Docker image (smaller)
├─ One runtime environment to maintain
├─ Faster deployment
└─ Most popular language for beginners

Trade-off: Can add more languages later with minimal changes
```

**Growth Path:**
```text
When you reach 10K DAU:
1. Add Redis caching ($25/month) → 10x faster responses
2. Separate judge workers ($100/month) → Non-blocking submissions
3. Add more languages ($0 cost, just deployment)

When you reach 100K DAU:
1. Add CDN ($50/month) → Global performance
2. Shard database ($200/month) → Handle write load
3. Auto-scaling ($500/month) → Handle spikes
```

This MVP approach:
├─ Validates product-market fit quickly
├─ Minimal complexity (easier to build and debug)
├─ 85% under budget (room for experimentation)
└─ Clear growth path when validated
</details>

**Time Budget:** 30 minutes

**Difficulty:** 🟡 Intermediate

---

*Next Up:* In Section 4, we'll dive deep into database design. We'll design schemas for users, problems, submissions, and more. Ready to become a database architect? 🗄️


## Section 4: Storing Our Data

### What You'll Learn

By the end of this section, you'll be able to:
- Design database schemas for users, problems, and submissions
- Choose appropriate indexes for query optimization
- Explain when to use SQL vs NoSQL databases
- Design a sharding strategy for horizontal scaling

### Why This Matters

Database design is the foundation of your application. Poor schema design leads to slow queries, data integrity issues, and scaling problems. Real-world example: A coding platform stored submission code directly in database rows, leading to 500GB database size and 5-second query times. Migrating code storage to S3 reduced database size by 90% and improved query speed by 10x!

---

### 🟢 For Beginners: The Fundamentals

#### What is a Database Schema?

Think of a database schema like a filing system for a large library:
- **Tables** = Different file cabinets (Users cabinet, Books cabinet, Loans cabinet)
- **Columns** = Information categories (Name, Email, Date Joined)
- **Rows** = Individual records (One row per user)
- **Relationships** = How cabinets connect (A user borrows books)

For our coding platform, we need to store:
1. **Users** - Who's using the platform?
2. **Problems** - What coding challenges exist?
3. **Submissions** - What code did users submit?
4. **Test Cases** - How do we verify solutions?

#### The Core Tables

Let's design each table step by step:

**1. Users Table (Who's on the platform?)**

```sql
users
├─ user_id: Unique identifier (like a student ID number)
├─ username: Display name (like "codewiz99")
├─ email: For login and notifications
├─ password_hash: Encrypted password (never store plaintext!)
├─ created_at: When they joined
└─ is_active: Can they still log in?

Example row:
user_id: "550e8400-e29b-41d4-a716-446655440000"
username: "alice"
email: "alice@example.com"
password_hash: "$2b$10$N9qo8uLOickgx2ZMRZoMy..."
created_at: "2025-01-15 10:30:00"
is_active: true
```

💡 **Pro Tip:** Use UUIDs for IDs instead of auto-incrementing integers. Why? If you shard the database later, UUIDs won't collide across shards!

**2. Problems Table (What challenges exist?)**

```sql
problems
├─ problem_id: Unique identifier
├─ title: "Two Sum", "Reverse Linked List"
├─ description: Problem explanation (can be long!)
├─ difficulty: "easy", "medium", or "hard"
├─ acceptance_rate: What % of submissions pass?
└─ total_submissions: How many times attempted?

Example row:
problem_id: "prob-001"
title: "Two Sum"
description: "Given an array of integers, return indices..."
difficulty: "easy"
acceptance_rate: 45.2
total_submissions: 2,500,000
```

**3. Submissions Table (What code did users write?)**

```sql
submissions
├─ submission_id: Unique identifier
├─ user_id: Who submitted? (links to users table)
├─ problem_id: Which problem? (links to problems table)
├─ code_s3_key: Where's the code stored? (S3 path, not the actual code!)
├─ language: "python", "java", "c++", "javascript"
├─ status: "accepted", "wrong_answer", "time_limit_exceeded"
├─ runtime_ms: How long did it take?
├─ memory_kb: How much memory used?
└─ submitted_at: When?

Example row:
submission_id: "sub-12345"
user_id: "550e8400-e29b-41d4-a716-446655440000"
problem_id: "prob-001"
code_s3_key: "submissions/2025/01/15/sub-12345.py"
language: "python"
status: "accepted"
runtime_ms: 45
memory_kb: 12288
submitted_at: "2025-01-15 14:22:33"
```

💡 **Key Decision:** Why store code in S3 instead of the database?
- **Small database:** Code can be 1-50KB. With millions of submissions, that's TB of data!
- **Fast queries:** Database stays small, queries stay fast
- **Cost:** S3 is 5x cheaper than database storage
- **Scalability:** S3 scales infinitely, databases don't

**4. Test Cases Table (How do we verify solutions?)**

```sql
test_cases
├─ test_case_id: Unique identifier
├─ problem_id: Which problem is this for?
├─ input_s3_key: Where's the input data? (S3 path)
├─ expected_output_s3_key: What's the correct answer? (S3 path)
├─ is_sample: Is this shown to users?
├─ is_hidden: Secret test case?
└─ time_limit_ms: How long can code run?

Example rows:
Test Case 1 (shown to user):
├─ input: [2, 7, 11, 15], target = 9
├─ expected_output: [0, 1]
└─ is_sample: true (user can see this)

Test Case 2 (hidden):
├─ input: [3, 2, 4], target = 6
├─ expected_output: [1, 2]
└─ is_hidden: true (user can't see this until after submission)
```

#### Understanding Relationships

Tables connect to each other:

```text
users (one) ←→ submissions (many)
├─ One user can have many submissions
└─ Each submission belongs to one user

problems (one) ←→ submissions (many)
├─ One problem can have many submissions
└─ Each submission is for one problem

problems (one) ←→ test_cases (many)
├─ One problem has many test cases (usually 50+)
└─ Each test case belongs to one problem
```

Real example:
```text
Alice (user) has submitted:
├─ Submission 1: Two Sum problem in Python (Accepted)
├─ Submission 2: Two Sum problem in Java (Wrong Answer)
└─ Submission 3: Reverse List problem in Python (Accepted)

Two Sum problem has:
├─ 50 test cases
├─ 2,500,000 submissions from all users
└─ Alice's 2 submissions are included in that count
```

---

### 🟡 For Intermediate: Interview Patterns

#### Index Design Strategy

Indexes are like a book's table of contents - they help find data quickly without reading every page.

**Problem:** Query without index
```sql
-- Find all submissions by user "alice" (user_id = "550e...")
SELECT * FROM submissions WHERE user_id = '550e8400-e29b-41d4-a716-446655440000';

Without index:
├─ Database scans ALL rows (100 million submissions)
├─ Checks each row: "Is this user_id '550e...'?"
└─ Time: 30 seconds! ⏰ (unacceptable)
```

**Solution:** Add index
```sql
CREATE INDEX idx_user_submissions ON submissions(user_id, submitted_at DESC);

With index:
├─ Database looks up user_id in index (like finding page 247 in table of contents)
├─ Jumps directly to that user's submissions
└─ Time: 10 milliseconds! ⚡ (3,000x faster!)
```

**Critical Indexes for Our Platform:**

```sql
-- Users table
CREATE INDEX idx_username ON users(username);  -- Login lookup
CREATE INDEX idx_email ON users(email);        -- Email lookup

-- Problems table
CREATE INDEX idx_difficulty ON problems(difficulty);  -- Filter by difficulty
CREATE INDEX idx_acceptance_rate ON problems(acceptance_rate DESC);  -- Sort by acceptance

-- Submissions table (most important!)
CREATE INDEX idx_user_problem ON submissions(user_id, problem_id, submitted_at DESC);
-- Why this composite index?
-- ├─ Query: "Show me all my submissions for problem X, newest first"
-- └─ Covers: user_id filter, problem_id filter, submitted_at sort (all in one index!)

CREATE INDEX idx_user_recent ON submissions(user_id, submitted_at DESC);
-- Why? Query: "Show me my 10 most recent submissions" (homepage)

CREATE INDEX idx_status ON submissions(status);
-- Why? Query: "Show me all accepted submissions" (stats)
```

#### Query Patterns

**Pattern 1: User's Submission History**
```sql
-- Use case: User clicks "My Submissions" tab
SELECT 
    s.submission_id,
    s.problem_id,
    p.title,
    s.status,
    s.runtime_ms,
    s.submitted_at
FROM submissions s
JOIN problems p ON s.problem_id = p.problem_id
WHERE s.user_id = '550e8400-e29b-41d4-a716-446655440000'
ORDER BY s.submitted_at DESC
LIMIT 20;

Performance:
├─ Uses index: idx_user_recent
├─ Time: 5ms (even with 100M submissions in table)
└─ Returns: Latest 20 submissions
```

**Pattern 2: Problem Statistics**
```sql
-- Use case: Show problem difficulty and acceptance rate
SELECT 
    problem_id,
    title,
    difficulty,
    total_submissions,
    total_accepted,
    (total_accepted * 100.0 / NULLIF(total_submissions, 0)) as acceptance_rate
FROM problems
WHERE difficulty = 'easy'
ORDER BY acceptance_rate DESC
LIMIT 100;

Performance:
├─ Uses index: idx_difficulty
├─ Time: 10ms
└─ Returns: Top 100 easiest problems
```

**Pattern 3: Leaderboard (Complex)**
```sql
-- Use case: Contest leaderboard
SELECT 
    u.username,
    COUNT(CASE WHEN s.status = 'accepted' THEN 1 END) as problems_solved,
    SUM(s.runtime_ms) as total_runtime
FROM users u
LEFT JOIN submissions s ON u.user_id = s.user_id
WHERE s.submitted_at >= '2025-01-01'  -- Contest start time
GROUP BY u.user_id, u.username
ORDER BY problems_solved DESC, total_runtime ASC
LIMIT 100;

Performance:
├─ Time: 500ms (expensive query!)
├─ Optimization: Cache results, update every 10 seconds
└─ Alternative: Pre-compute rankings in separate table
```

#### Sharding Strategy

**When to Shard?**

Single database limits:
```text
PostgreSQL can handle:
├─ ~10,000 queries per second (with good hardware)
├─ ~10 TB of data (before performance degrades)
└─ ~500 concurrent connections

Our growth:
├─ 100K DAU: 6 writes/sec, 12 reads/sec ✅ Single DB fine
├─ 1M DAU: 60 writes/sec, 120 reads/sec ✅ Still OK
├─ 10M DAU: 600 writes/sec, 1,200 reads/sec ⚠️ Getting close
└─ 100M DAU: 6,000 writes/sec ❌ Must shard!
```

**How to Shard Submissions Table:**

```text
Shard by user_id (consistent hashing):

Shard Function:
shard_id = user_id % 16  (16 shards)

Example:
user_id "550e8400..." → hash → 7 → Shard 7
user_id "a71f3c92..." → hash → 3 → Shard 3

Benefits:
├─ All user's submissions in same shard (fast queries!)
├─ Even distribution (each shard has ~6.25% of users)
└─ Can query single shard for user's history

Trade-off:
├─ Cross-shard queries are hard (e.g., "all submissions for problem X")
└─ Solution: Denormalize or use separate index service
```

**Sharding Layout:**

```text
Application Layer
    ↓
Shard Router (determines which shard based on user_id)
    ↓
┌────────┬────────┬────────┬─────────┐
│Shard 0 │Shard 1 │Shard 2 │... │Shard 15│
│Users   │Users   │Users   │    │Users   │
│0,16,32 │1,17,33 │2,18,34 │    │15,31,47│
└────────┴────────┴────────┴─────────┘

Each shard:
├─ Independent PostgreSQL instance
├─ Contains ~6.25M users (if 100M total)
├─ Contains submissions for those users only
└─ Can be scaled independently
```

---

### 🔴 For Advanced: Production Considerations

#### Database Partitioning for Submissions

Submissions table grows forever - need time-based partitioning:

```sql
-- Partition by month
CREATE TABLE submissions_2025_01 PARTITION OF submissions
FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE submissions_2025_02 PARTITION OF submissions
FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Automatically route to correct partition
SELECT * FROM submissions WHERE submitted_at = '2025-01-15 14:22:33';
-- PostgreSQL automatically queries only submissions_2025_01 partition!

Benefits:
├─ Faster queries (query only relevant month)
├─ Easy archival (drop old partitions)
├─ Better index performance (indexes are smaller)
└─ Can archive old partitions to cold storage
```

**Archival Strategy:**

```text
Age-based tiers:
├─ Hot (0-7 days): SSD, all indexes, fast queries
├─ Warm (7-90 days): HDD, partial indexes, slower OK
├─ Cold (90+ days): S3 Glacier, no indexes, restore on-demand
└─ Archived (1+ year): Compressed, deleted after 3 years

Cost comparison:
├─ Hot: $0.10/GB/month × 10GB = $1.00/month
├─ Warm: $0.02/GB/month × 100GB = $2.00/month
├─ Cold: $0.004/GB/month × 1TB = $4.00/month
└─ Total: $7.00/month instead of $110 (all hot)
```

#### Read Replicas for Scale

**Problem:** Too many reads overwhelm primary database

```text
Primary database:
├─ Handles ALL writes (submissions)
├─ Handles ALL reads (queries)
└─ At 1M QPS, maxed out!
```

**Solution:** Read replicas

```text
       Writes            Reads
         ↓                ↓
    [Primary DB]    [Load Balancer]
         │              ↙  │  ↘
    (Replicates)   [Replica 1] [Replica 2] [Replica 3]
         │
         ↓
    All replicas have same data (with slight lag)
```

**Configuration:**

```sql
-- On application side
if query_type == "write":
    connection = primary_db  # All writes to primary
elif query_type == "read":
    connection = read_replica_pool.get_random()  # Random replica

Examples:
INSERT INTO submissions ... → Primary DB
SELECT * FROM problems ... → Replica 1, 2, or 3 (random)
UPDATE users SET ... → Primary DB
SELECT * FROM submissions WHERE user_id ... → Replica
```

**Replication Lag Challenge:**

```text
Problem:
1. User submits code (writes to Primary)
2. User immediately refreshes page (reads from Replica)
3. Replica hasn't replicated yet (lag: 100ms)
4. User doesn't see their submission! Bug? No, replication lag!

Solution: Read-your-writes consistency
```python
def submit_code(user_id, code):
    # Write to primary
    primary_db.insert(submission)
    
    # Set cookie with write timestamp
    set_cookie("last_write_ts", current_time())
    
def get_user_submissions(user_id):
    last_write = get_cookie("last_write_ts")
    
    if last_write and (current_time() - last_write) < 5_seconds:
        # Recent write, read from primary (guaranteed fresh)
        return primary_db.query(user_id)
    else:
        # Old data, read from replica (faster)
        return replica_db.query(user_id)
```
```

#### Multi-Master Replication for Global Scale

**Single Primary Problem:**

```text
Users globally → All writes to US-East primary → High latency for Asia/Europe
```

**Multi-Master Solution:**

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   US-East    │     │   EU-West    │     │  AP-South    │
│   (Primary)  │────▶│   (Primary)  │◀────│  (Primary)   │
│              │     │              │     │              │
│ US users     │     │ EU users     │     │ Asia users   │
│ write here   │     │ write here   │     │ write here   │
└──────────────┘     └──────────────┘     └──────────────┘
         ▲                   ▲                   ▲
         └───────────────────┴───────────────────┘
              Bidirectional replication

Benefits:
├─ Low latency writes (write to nearest region)
├─ High availability (if one region fails, others continue)
└─ Global scalability
```

**Conflict Resolution:**

```text
Conflict example:
├─ User edits profile in US: name = "Alice Johnson"
├─ Same user edits profile in EU: name = "Alicia Johnson"
├─ Both writes happen simultaneously
└─ Which one wins?

Resolution strategies:
1. Last-Write-Wins (LWW):
   ├─ Timestamp-based (whichever write happened later)
   └─ Simple but can lose data

2. Application-level merge:
   ├─ Keep both versions
   ├─ Show user: "You have conflicting changes"
   └─ Let user decide

3. CRDT (Conflict-Free Replicated Data Types):
   ├─ Mathematical guarantee of convergence
   ├─ Complex to implement
   └─ Used by distributed systems like Cassandra
```

---

### Real-World Example: LeetCode's Database Evolution

**2015: Single PostgreSQL**
```text
├─ One database: 10 GB
├─ Tables: users, problems, submissions
├─ No sharding, no replication
└─ Supported: 10K users
```

**2017: Read Replicas**
```text
├─ Primary: Handles writes
├─ 3 Read Replicas: Handle reads
├─ Size: 500 GB
└─ Supported: 500K users
```

**2020: Sharding + Partitioning**
```text
├─ 16 shards (by user_id)
├─ Each shard partitioned by month
├─ Size: 10 TB total (625 GB per shard)
└─ Supported: 5M users
```

**2023: Hybrid SQL + NoSQL**
```text
PostgreSQL:
├─ Users, Problems (relational data)
├─ 1 TB
└─ Strong consistency

Cassandra:
├─ Submissions (time-series data)
├─ 50 TB
└─ Eventually consistent

Redis:
├─ Sessions, Cache
├─ 100 GB in-memory
└─ Ephemeral data

S3:
├─ Code files, Test cases
├─ 100 TB
└─ Cheap storage
```

**Technology Choices Evolution:**

```text
Why PostgreSQL for Users/Problems?
├─ ACID transactions (profile updates must be atomic)
├─ Complex joins (user stats require joining multiple tables)
├─ Mature tooling (pg_dump, pg_restore, replication)
└─ Developer familiarity

Why Cassandra for Submissions?
├─ Write-heavy (500K submissions/day)
├─ Time-series data (mostly query by timestamp)
├─ Horizontal scalability (add nodes easily)
└─ High availability (multi-datacenter replication)

Why not just PostgreSQL for everything?
├─ Submissions table: 10 TB and growing
├─ PostgreSQL hard to scale beyond 10 TB
├─ Cassandra designed for 100+ TB
└─ Use right tool for right job!
```

---

### 🎯 Interview Questions: Database Design

**Question 1:** Should we store submission code in the database or object storage (S3)? Justify your choice.

<details>
<summary>💡 Hint</summary>
Think about data size, query patterns, cost, and scalability.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Analysis:**

**Option A: Store in Database (TEXT column)**
```text
Pros:
├─ Transactional (update code + metadata atomically)
├─ Easy to query (no separate service needed)
└─ Simpler architecture

Cons:
├─ Database bloat: 2KB per submission × 100M = 200 GB just code!
├─ Slower queries: Larger table = slower scans
├─ Expensive: Database storage costs $0.10/GB vs S3 $0.023/GB
└─ Backup size: 200 GB extra in every backup
```

**Option B: Store in S3 (Reference in Database)**
```text
Pros:
├─ Small database: Store only 100-char S3 key, not 2KB code
├─ Fast queries: Smaller table = faster
├─ Cheap: S3 is 5x cheaper than database storage
├─ Scalable: S3 scales to exabytes, database doesn't
└─ Separation of concerns: Code is just a file

Cons:
├─ Two-phase commit: Insert DB record, then upload S3 (can fail!)
├─ Complex cleanup: If DB insert fails, need to delete S3 object
├─ Extra latency: Need to fetch from S3 to display code
└─ More infrastructure: Need to manage S3 buckets
```

**Recommendation: Store in S3**

**Implementation:**

```python
def submit_code(user_id, problem_id, code, language):
    # 1. Generate unique S3 key
    timestamp = datetime.now()
    s3_key = f"submissions/{timestamp.year}/{timestamp.month:02d}/{timestamp.day:02d}/{uuid4()}.{language}"
    
    # 2. Upload to S3 first (fail fast if upload fails)
    try:
        s3_client.put_object(
            Bucket='leetcode-submissions',
            Key=s3_key,
            Body=code.encode('utf-8'),
            ServerSideEncryption='AES256'  # Encrypt at rest
        )
    except S3Error:
        return {"error": "Failed to store code"}
    
    # 3. Insert database record (references S3)
    try:
        submission_id = db.insert({
            'user_id': user_id,
            'problem_id': problem_id,
            'code_s3_key': s3_key,
            'language': language,
            'status': 'pending',
            'submitted_at': timestamp
        })
    except DBError:
        # Rollback: Delete S3 object
        s3_client.delete_object(Bucket='leetcode-submissions', Key=s3_key)
        return {"error": "Failed to save submission"}
    
    # 4. Enqueue for judging
    queue.send({
        'submission_id': submission_id,
        'code_s3_key': s3_key
    })
    
    return {"submission_id": submission_id}

def get_submission_code(submission_id):
    # 1. Query database for S3 key
    submission = db.query(f"SELECT code_s3_key FROM submissions WHERE submission_id = {submission_id}")
    
    # 2. Fetch from S3
    code = s3_client.get_object(
        Bucket='leetcode-submissions',
        Key=submission['code_s3_key']
    ).read().decode('utf-8')
    
    return code
```

**At scale (10M DAU):**
```text
Submissions per day: 5M
Code size avg: 2KB
Daily S3 storage: 10 GB

Annual storage cost:
├─ S3: 3.65 TB × $0.023/GB = $84/month
├─ Database (if stored there): 3.65 TB × $0.10/GB = $365/month
└─ Savings: $281/month = $3,372/year
```

**Conclusion:** S3 storage is clearly superior for large binary/text data that doesn't need transactional guarantees.

</details>

**Question 2:** Design a database schema to support coding contests with real-time leaderboards.

<details>
<summary>💡 Hint</summary>
Think about new entities (contests, contest_participants), real-time updates, and denormalization for performance.
</details>

<details>
<summary>✅ Sample Answer</summary>

**New Tables Needed:**

**1. Contests Table:**
```sql
contests
├─ contest_id (PK, UUID)
├─ title (VARCHAR(200))
├─ description (TEXT)
├─ start_time (TIMESTAMP)
├─ end_time (TIMESTAMP)
├─ duration_minutes (INT) -- 90 minutes, 120 minutes, etc.
├─ created_by (FK -> users.user_id)
├─ is_active (BOOLEAN)
└─ total_participants (INT) -- Denormalized for quick access

CREATE INDEX idx_contest_start ON contests(start_time DESC);
CREATE INDEX idx_contest_active ON contests(is_active, start_time);
```

**2. Contest_Problems Table (Many-to-Many):**
```sql
contest_problems
├─ contest_id (FK -> contests.contest_id)
├─ problem_id (FK -> problems.problem_id)
├─ points (INT) -- 100 points for easy, 200 for medium, 300 for hard
├─ order_index (INT) -- Problem order in contest
└─ PRIMARY KEY (contest_id, problem_id)

CREATE INDEX idx_contest_problems ON contest_problems(contest_id, order_index);
```

**3. Contest_Participants Table:**
```sql
contest_participants
├─ contest_id (FK -> contests.contest_id)
├─ user_id (FK -> users.user_id)
├─ registered_at (TIMESTAMP)
├─ started_at (TIMESTAMP) -- When user started (may differ from contest start)
├─ finished_at (TIMESTAMP)
├─ total_score (INT) -- Denormalized for leaderboard
├─ penalty_time (INT) -- For tie-breaking (submission time penalty)
├─ rank (INT) -- Denormalized rank
└─ PRIMARY KEY (contest_id, user_id)

CREATE INDEX idx_leaderboard ON contest_participants(contest_id, total_score DESC, penalty_time ASC);
```

**4. Contest_Submissions Table:**
```sql
contest_submissions
├─ submission_id (PK, references submissions.submission_id)
├─ contest_id (FK -> contests.contest_id)
├─ user_id (FK -> users.user_id)
├─ problem_id (FK -> problems.problem_id)
├─ submitted_at (TIMESTAMP)
├─ time_offset_seconds (INT) -- Seconds from contest start
├─ points_earned (INT) -- 0 if wrong, full points if accepted
└─ is_first_accepted (BOOLEAN) -- Bonus points for first to solve?

CREATE INDEX idx_user_contest_subs ON contest_submissions(user_id, contest_id, submitted_at);
CREATE INDEX idx_problem_contest_subs ON contest_submissions(problem_id, contest_id, submitted_at);
```

**Leaderboard Query (Real-Time):**

```sql
-- Get top 100 leaderboard
SELECT 
    cp.rank,
    u.username,
    cp.total_score,
    cp.penalty_time,
    COUNT(cs.submission_id) FILTER (WHERE cs.points_earned > 0) as problems_solved,
    COUNT(cs.submission_id) as total_attempts
FROM contest_participants cp
JOIN users u ON cp.user_id = u.user_id
LEFT JOIN contest_submissions cs ON cp.user_id = cs.user_id AND cp.contest_id = cs.contest_id
WHERE cp.contest_id = '{{contest_id}}'
GROUP BY cp.user_id, u.username, cp.rank, cp.total_score, cp.penalty_time
ORDER BY cp.total_score DESC, cp.penalty_time ASC
LIMIT 100;

-- Performance: Uses idx_leaderboard, < 50ms for 10K participants
```

**Denormalization Strategy:**

```text
Problem: Calculating score in real-time for 10K participants is slow
Solution: Denormalize total_score and rank

Update strategy:
1. When submission is judged:
   ├─ Update contest_participants.total_score
   ├─ Recalculate rank (only for affected users)
   └─ Trigger: After each submission

2. Batch rank updates:
   ├─ Instead of recalculating all ranks on each submission
   ├─ Update ranks every 10 seconds
   └─ Eventual consistency OK for leaderboard

3. Use materialized view (PostgreSQL):
   ```sql
   CREATE MATERIALIZED VIEW contest_leaderboard AS
   SELECT ...
   -- Refresh every 10 seconds
   REFRESH MATERIALIZED VIEW CONCURRENTLY contest_leaderboard;
   ```
```

**Real-Time Updates (WebSocket):**

```python
# When submission is judged
def on_submission_judged(submission_id, status, points):
    submission = db.get_submission(submission_id)
    
    # Update user's contest score
    db.execute("""
        UPDATE contest_participants
        SET total_score = total_score + %s,
            penalty_time = penalty_time + %s
        WHERE contest_id = %s AND user_id = %s
    """, [points, submission.time_offset_seconds, submission.contest_id, submission.user_id])
    
    # Get updated leaderboard (top 100)
    leaderboard = db.query("SELECT ... ORDER BY total_score DESC LIMIT 100")
    
    # Broadcast via WebSocket to all contest participants
    websocket_broadcast(f"contest:{submission.contest_id}", {
        "type": "leaderboard_update",
        "leaderboard": leaderboard,
        "user_update": {
            "user_id": submission.user_id,
            "new_score": updated_score,
            "new_rank": updated_rank
        }
    })
```

**Scaling for Large Contests:**

```text
Problem: 100K participants, 1M submissions in 2 hours
Solution: Separate database for contest data

Architecture:
├─ Main DB: Users, Problems (read-heavy)
├─ Contest DB: Contest-specific data (write-heavy during contest)
└─ After contest: Merge contest results back to main DB

Benefits:
├─ Isolation: Contest spikes don't affect main platform
├─ Optimized: Contest DB tuned for high write throughput
├─ Recovery: If contest DB crashes, main platform unaffected
└─ Cleanup: Drop contest DB after contest ends
```

**Alternative: Redis for Leaderboard:**

```python
# Use Redis Sorted Set for leaderboard
def update_leaderboard(contest_id, user_id, score):
    # Sorted set: score as score, user_id as member
    redis.zadd(f"leaderboard:{contest_id}", {user_id: score})

def get_leaderboard(contest_id, limit=100):
    # Get top 100 (sorted by score, descending)
    top_users = redis.zrevrange(f"leaderboard:{contest_id}", 0, limit-1, withscores=True)
    
    # Enrich with user details from database
    users = db.query(f"SELECT user_id, username FROM users WHERE user_id IN ({','.join(top_users)})")
    
    return [(user.username, score) for user, score in zip(users, top_users)]

# Performance: O(log N) insert, O(log N + M) range query (very fast!)
# Caveat: Redis is in-memory (expensive for large datasets)
```

**Cost-Benefit:**
```text
PostgreSQL approach:
├─ Pros: Persistent, ACID, familiar
├─ Cons: Slower updates (50ms per update)
└─ Cost: Included in DB cost

Redis approach:
├─ Pros: Very fast (1ms per update), real-time
├─ Cons: In-memory (lost on restart), extra infrastructure
└─ Cost: $100/month for Redis cluster

Hybrid approach:
├─ Redis for real-time leaderboard (during contest)
├─ PostgreSQL for persistence (write-behind)
├─ After contest: Flush Redis → PostgreSQL (permanent record)
└─ Best of both worlds!
```

</details>

---

### 🤔 Think About It

1. **Why use UUIDs instead of auto-incrementing integers for primary keys?**
   - Hint: Think about sharding and uniqueness guarantees

2. **When would you choose NoSQL (Cassandra/MongoDB) over PostgreSQL?**
   - Hint: Think about data model, query patterns, and scale

3. **What's the risk of too many indexes?**
   - Hint: Every index speeds up reads but slows down writes

4. **How would you handle a user deleting their account (GDPR right to deletion)?**
   - Hint: Submissions reference user_id - cascade delete or anonymize?

---

### ✅ Key Takeaways

🎯 **Schema Design Principles:**
- **Normalize** for data integrity (users, problems separate tables)
- **Denormalize** for performance (cache total_score in contest_participants)
- **Use foreign keys** to maintain referential integrity
- **Store large blobs externally** (S3 for code, not database TEXT)

📊 **Index Strategy:**
- **Index frequently queried columns** (user_id, problem_id, submitted_at)
- **Composite indexes** for multi-column queries (user_id, submitted_at DESC)
- **Don't over-index** - each index slows down writes

🔧 **Scaling Techniques:**
- **Read replicas** for read-heavy workloads (95% cache hit rate helps!)
- **Sharding** when single DB can't handle load (10M+ users)
- **Partitioning** for time-series data (submissions by month)
- **Hybrid SQL/NoSQL** for different data types

💡 **Interview Tips:**
- Start simple (single database), explain when to scale
- Justify technology choices (PostgreSQL vs Cassandra)
- Discuss trade-offs (normalization vs denormalization)
- Show awareness of costs (S3 vs database storage)

---

### 🎯 Practice Exercise

**Challenge:** Design a schema for a premium subscription system

**Requirements:**
- Users can subscribe monthly ($10/month) or yearly ($100/year)
- Track payment history (for accounting)
- Premium users get:
  - Access to premium problems
  - No ads
  - Video explanations
- Must support:
  - Free trial (14 days)
  - Discounts (50% off promo codes)
  - Subscription cancellation (but access until period end)

**Deliverables:**
1. Design 3-4 tables needed
2. Show relationships (foreign keys)
3. Index strategy
4. Query to check "Is user currently premium?"

<details>
<summary>📝 Sample Solution</summary>

**Tables:**

```sql
-- 1. Subscription Plans
subscription_plans
├─ plan_id (PK, UUID)
├─ name (VARCHAR) -- "Monthly", "Yearly", "Student"
├─ price_cents (INT) -- 1000 for $10.00
├─ duration_days (INT) -- 30 for monthly, 365 for yearly
├─ features (JSONB) -- {"premium_problems": true, "no_ads": true, "video_explanations": true}
└─ is_active (BOOLEAN)

Example rows:
plan_id: "plan-monthly"
name: "Monthly"
price_cents: 1000
duration_days: 30

plan_id: "plan-yearly"
name: "Yearly"
price_cents: 10000
duration_days: 365

-- 2. User Subscriptions
user_subscriptions
├─ subscription_id (PK, UUID)
├─ user_id (FK -> users.user_id)
├─ plan_id (FK -> subscription_plans.plan_id)
├─ status (ENUM: 'active', 'cancelled', 'expired', 'trial')
├─ started_at (TIMESTAMP)
├─ expires_at (TIMESTAMP) -- When subscription ends
├─ cancelled_at (TIMESTAMP, NULL) -- If user cancelled
├─ auto_renew (BOOLEAN, DEFAULT TRUE) -- Will it renew?
├─ discount_code (VARCHAR, NULL) -- "SUMMER50"
└─ discount_percent (INT, NULL) -- 50

CREATE INDEX idx_user_active_sub ON user_subscriptions(user_id, status, expires_at);
CREATE INDEX idx_expiring_subs ON user_subscriptions(expires_at, status) WHERE status = 'active';

-- 3. Payments
payments
├─ payment_id (PK, UUID)
├─ user_id (FK -> users.user_id)
├─ subscription_id (FK -> user_subscriptions.subscription_id)
├─ amount_cents (INT)
├─ currency (VARCHAR(3)) -- "USD", "EUR"
├─ status (ENUM: 'pending', 'completed', 'failed', 'refunded')
├─ payment_method (ENUM: 'card', 'paypal', 'stripe')
├─ stripe_payment_id (VARCHAR) -- External payment provider ID
├─ paid_at (TIMESTAMP)
├─ failed_reason (TEXT, NULL)
└─ refunded_at (TIMESTAMP, NULL)

CREATE INDEX idx_user_payments ON payments(user_id, paid_at DESC);
CREATE INDEX idx_subscription_payments ON payments(subscription_id);

-- 4. Promo Codes
promo_codes
├─ code (PK, VARCHAR(50)) -- "SUMMER50"
├─ discount_percent (INT) -- 50
├─ valid_from (TIMESTAMP)
├─ valid_until (TIMESTAMP)
├─ max_uses (INT) -- 1000 people can use it
├─ current_uses (INT, DEFAULT 0)
├─ is_active (BOOLEAN)
└─ created_by (FK -> users.user_id, NULL) -- Admin who created

CREATE INDEX idx_promo_active ON promo_codes(code, is_active, valid_until);
```

**Critical Query: Check if user is premium**

```sql
-- Method 1: Direct query
SELECT EXISTS(
    SELECT 1
    FROM user_subscriptions
    WHERE user_id = '{{user_id}}'
      AND status = 'active'
      AND expires_at > NOW()
) AS is_premium;

-- Method 2: Get subscription details
SELECT 
    us.subscription_id,
    us.plan_id,
    sp.name AS plan_name,
    us.expires_at,
    us.auto_renew,
    (us.expires_at > NOW() AND us.status = 'active') AS is_currently_premium
FROM user_subscriptions us
JOIN subscription_plans sp ON us.plan_id = sp.plan_id
WHERE us.user_id = '{{user_id}}'
  AND us.status IN ('active', 'trial')
ORDER BY us.expires_at DESC
LIMIT 1;

-- Method 3: Denormalized (for performance)
-- Add column to users table:
ALTER TABLE users ADD COLUMN is_premium BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN premium_expires_at TIMESTAMP;

-- Update via trigger when subscription changes
CREATE TRIGGER update_user_premium_status
AFTER INSERT OR UPDATE ON user_subscriptions
FOR EACH ROW
EXECUTE FUNCTION sync_user_premium_status();

-- Then query is just:
SELECT is_premium FROM users WHERE user_id = '{{user_id}}';
-- Blazing fast! But requires keeping denormalized data in sync.
```

**Business Logic:**

```python
class SubscriptionService:
    def is_user_premium(self, user_id):
        """Check if user has active premium subscription"""
        sub = db.query("""
            SELECT expires_at, status
            FROM user_subscriptions
            WHERE user_id = %s
              AND status IN ('active', 'trial')
            ORDER BY expires_at DESC
            LIMIT 1
        """, [user_id])
        
        if not sub:
            return False
        
        return sub.expires_at > datetime.now() and sub.status in ['active', 'trial']
    
    def subscribe_user(self, user_id, plan_id, promo_code=None):
        """Subscribe user to a plan"""
        plan = db.get_plan(plan_id)
        
        # Check promo code
        discount = 0
        if promo_code:
            promo = db.query("SELECT * FROM promo_codes WHERE code = %s AND is_active = TRUE", [promo_code])
            if promo and promo.valid_until > datetime.now() and promo.current_uses < promo.max_uses:
                discount = promo.discount_percent
                db.execute("UPDATE promo_codes SET current_uses = current_uses + 1 WHERE code = %s", [promo_code])
        
        # Calculate price
        price = plan.price_cents * (1 - discount / 100)
        
        # Create subscription
        expires_at = datetime.now() + timedelta(days=plan.duration_days)
        subscription_id = db.insert({
            'user_id': user_id,
            'plan_id': plan_id,
            'status': 'active',
            'started_at': datetime.now(),
            'expires_at': expires_at,
            'discount_code': promo_code,
            'discount_percent': discount,
            'auto_renew': True
        }, table='user_subscriptions')
        
        # Process payment
        payment = stripe.charge(user_id, price)
        db.insert({
            'user_id': user_id,
            'subscription_id': subscription_id,
            'amount_cents': price,
            'status': 'completed' if payment.success else 'failed',
            'stripe_payment_id': payment.id,
            'paid_at': datetime.now()
        }, table='payments')
        
        return subscription_id
    
    def cancel_subscription(self, user_id):
        """Cancel auto-renewal but keep access until period end"""
        db.execute("""
            UPDATE user_subscriptions
            SET auto_renew = FALSE,
                cancelled_at = NOW()
            WHERE user_id = %s
              AND status = 'active'
        """, [user_id])
        
        # User keeps premium access until expires_at
        # Cron job will set status = 'expired' when expires_at passes
```

**Cron Job: Expire Subscriptions**

```python
# Run every hour
def expire_subscriptions():
    """Find expired subscriptions and mark them"""
    expired = db.query("""
        SELECT subscription_id, user_id
        FROM user_subscriptions
        WHERE status = 'active'
          AND expires_at < NOW()
          AND auto_renew = FALSE
    """)
    
    for sub in expired:
        db.execute("UPDATE user_subscriptions SET status = 'expired' WHERE subscription_id = %s", [sub.subscription_id])
        
        # Update denormalized user.is_premium
        db.execute("UPDATE users SET is_premium = FALSE WHERE user_id = %s", [sub.user_id])

def renew_subscriptions():
    """Auto-renew subscriptions that are expiring"""
    expiring = db.query("""
        SELECT s.subscription_id, s.user_id, s.plan_id
        FROM user_subscriptions s
        JOIN subscription_plans p ON s.plan_id = p.plan_id
        WHERE s.status = 'active'
          AND s.expires_at < NOW() + INTERVAL '1 day'  -- Expiring in next 24 hours
          AND s.auto_renew = TRUE
    """)
    
    for sub in expiring:
        try:
            # Charge user
            plan = db.get_plan(sub.plan_id)
            payment = stripe.charge(sub.user_id, plan.price_cents)
            
            if payment.success:
                # Extend subscription
                new_expires_at = sub.expires_at + timedelta(days=plan.duration_days)
                db.execute("""
                    UPDATE user_subscriptions
                    SET expires_at = %s
                    WHERE subscription_id = %s
                """, [new_expires_at, sub.subscription_id])
                
                # Record payment
                db.insert({
                    'user_id': sub.user_id,
                    'subscription_id': sub.subscription_id,
                    'amount_cents': plan.price_cents,
                    'status': 'completed',
                    'paid_at': datetime.now()
                }, table='payments')
            else:
                # Payment failed, mark subscription as expiring
                db.execute("""
                    UPDATE user_subscriptions
                    SET status = 'payment_failed',
                        auto_renew = FALSE
                    WHERE subscription_id = %s
                """, [sub.subscription_id])
                
                # Send email to user
                send_email(sub.user_id, "Payment failed for your subscription")
        except Exception as e:
            log_error(f"Failed to renew subscription {sub.subscription_id}: {e}")
```

**Edge Cases Handled:**

1. **Free trial converts to paid:**
   ```sql
   -- User starts 14-day trial
   INSERT INTO user_subscriptions (user_id, plan_id, status, expires_at)
   VALUES ('user-123', 'plan-monthly', 'trial', NOW() + INTERVAL '14 days');
   
   -- After 14 days, if they don't cancel, charge them and convert to active
   UPDATE user_subscriptions
   SET status = 'active', expires_at = NOW() + INTERVAL '30 days'
   WHERE user_id = 'user-123' AND status = 'trial';
   ```

2. **User subscribes while already subscribed (extend):**
   ```python
   # Don't create new subscription, extend existing
   db.execute("""
       UPDATE user_subscriptions
       SET expires_at = expires_at + INTERVAL '30 days'
       WHERE user_id = %s AND status = 'active'
   """, [user_id])
   ```

3. **Refunds:**
   ```python
   def refund_subscription(subscription_id):
       # Mark payment as refunded
       db.execute("""
           UPDATE payments
           SET status = 'refunded', refunded_at = NOW()
           WHERE subscription_id = %s
       """, [subscription_id])
       
       # Immediately expire subscription
       db.execute("""
           UPDATE user_subscriptions
           SET status = 'expired', expires_at = NOW()
           WHERE subscription_id = %s
       """, [subscription_id])
   ```

</details>

**Time Budget:** 40 minutes  
**Difficulty:** 🔴 Advanced

---

*Next Up:* In Section 5, we'll design the REST API that powers the platform. We'll define endpoints for authentication, submissions, and more. Ready to become an API architect? 🔌


## Section 5: How Users Interact (API Design)

### What You'll Learn

By the end of this section, you'll be able to:
- Design RESTful API endpoints for a coding platform
- Implement JWT-based authentication and authorization
- Apply rate limiting strategies to prevent abuse
- Version APIs for backward compatibility

### Why This Matters

API design determines how developers interact with your platform. Poor API design leads to frustrated developers, security vulnerabilities, and scaling problems. Real-world example: Twitter changed their API from free to paid without proper versioning, breaking thousands of third-party applications overnight and causing massive backlash. Good API design with versioning prevents such disasters!

---

### 🟢 For Beginners: The Fundamentals

#### What is an API?

Think of an API (Application Programming Interface) like a restaurant menu:
- **Menu** = List of available actions (endpoints)
- **Order** = Your request (HTTP request)
- **Kitchen** = Backend system (processes request)
- **Food delivered** = Response (data returned)

You don't need to know how the kitchen works - you just order from the menu!

**Example API Call:**
```text
You: "Hey API, give me problem #42"
API: "Here's the problem description, test cases, and examples"

Behind the scenes:
1. Your browser sends: GET /v1/problems/two-sum
2. API server receives request
3. Checks: Are you logged in? Do you have permission?
4. Fetches problem from database
5. Returns JSON response
```

#### Core API Concepts

**1. HTTP Methods (Verbs)**

```text
GET    = Read (get me data, don't change anything)
POST   = Create (make something new)
PUT    = Update (replace entirely)
PATCH  = Update (change specific fields)
DELETE = Delete (remove it)

Examples:
├─ GET /v1/problems → "Show me all problems"
├─ POST /v1/submissions → "Submit my code"
├─ PATCH /v1/users/me → "Update my profile"
└─ DELETE /v1/sessions → "Log me out"
```

**2. Endpoints (URLs)**

```text
Base URL: https://api.leetcode.com/v1

Endpoints:
├─ /v1/auth/login → Authentication
├─ /v1/problems → Browse problems
├─ /v1/problems/{id} → Get specific problem
├─ /v1/submissions → Submit code
└─ /v1/users/me → My profile

Pattern: /version/resource/action
Why /v1/? So we can release /v2/ later without breaking old apps!
```

**3. Request & Response**

Every API interaction has two parts:

**Request (from you):**
```http
POST /v1/submissions
Authorization: Bearer abc123token
Content-Type: application/json

{
  "problem_id": "prob-42",
  "language": "python",
  "code": "def twoSum(nums, target): ..."
}
```

**Response (from server):**
```json
{
  "success": true,
  "data": {
    "submission_id": "sub-12345",
    "status": "pending",
    "message": "Code submitted successfully"
  }
}
```

#### Authentication: Who Are You?

**Why authentication matters:**
```text
Without auth:
Anyone: "Show me all users' submissions"
Server: "Here's everyone's code!" ❌ Security disaster!

With auth:
User: "Show me MY submissions" + [login token]
Server: "You're Alice. Here are YOUR submissions." ✅ Secure!
```

**JWT (JSON Web Token) - Like a Concert Wristband:**

```text
Concert analogy:
1. Buy ticket (login)
2. Get wristband (JWT token)
3. Show wristband to enter VIP area (authenticated requests)
4. Security checks wristband without asking box office
   (Server verifies token without database lookup - fast!)

JWT structure:
header.payload.signature
eyJhbGci.eyJ1c2VyX2lk.SflKxwRJ

Decoded:
{
  "user_id": "550e8400",
  "username": "alice",
  "exp": 1635724800  (expires in 15 minutes)
}
```

**Login Flow:**

```text
1. User submits username/password
   ↓
2. Server verifies credentials
   ↓
3. Server creates JWT token
   {user_id: "alice", expires: 15_minutes_from_now}
   ↓
4. Server signs token (so it can't be tampered with)
   ↓
5. Server returns token to user
   ↓
6. User stores token (browser localStorage or cookie)
   ↓
7. Every future request includes token in header:
   Authorization: Bearer eyJhbGci...
   ↓
8. Server verifies signature and checks expiry
   ↓
9. If valid, processes request. If expired, ask user to login again.
```

💡 **Pro Tip:** Access tokens expire quickly (15 min) for security. Refresh tokens last longer (7 days) to get new access tokens without re-entering password!

---

### 🟡 For Intermediate: Interview Patterns

#### REST API Design Principles

**1. Resource-Oriented URLs**

```text
Good (noun-based):
✅ GET /v1/problems
✅ GET /v1/problems/42
✅ POST /v1/submissions
✅ GET /v1/users/alice/submissions

Bad (verb-based):
❌ GET /v1/getProblems
❌ POST /v1/submitCode
❌ GET /v1/fetchUserSubmissions
```

**2. HTTP Status Codes**

```text
2xx Success:
├─ 200 OK: Request succeeded
├─ 201 Created: Resource created successfully
├─ 204 No Content: Success, no data to return

4xx Client Errors:
├─ 400 Bad Request: Invalid input
├─ 401 Unauthorized: Not logged in
├─ 403 Forbidden: Logged in but no permission
├─ 404 Not Found: Resource doesn't exist
├─ 429 Too Many Requests: Rate limit exceeded

5xx Server Errors:
├─ 500 Internal Server Error: Bug in our code
├─ 503 Service Unavailable: System overloaded
└─ 504 Gateway Timeout: Request took too long

Interview tip: Always explain the difference between 401 and 403!
├─ 401: "Who are you?" (authentication)
└─ 403: "I know who you are, but you can't do this" (authorization)
```

**3. Pagination Pattern**

```text
Problem: Returning all 3,000 problems in one response
├─ Response size: 15 MB
├─ Load time: 30 seconds
└─ User experience: Terrible!

Solution: Pagination
GET /v1/problems?page=1&limit=20

Response:
{
  "data": [...20 problems...],
  "pagination": {
    "current_page": 1,
    "total_pages": 150,
    "total_items": 3000,
    "has_next": true,
    "has_previous": false
  }
}

Interview questions:
Q: "Offset vs cursor pagination?"
A: "Offset (page=5) has issues with data changes. Cursor (after=id_100) is more reliable but complex."
```

#### Complete API Specification

**Authentication Endpoints:**

```http
POST /v1/auth/register
├─ Body: {username, email, password}
├─ Returns: {user, access_token, refresh_token}
└─ Status: 201 Created

POST /v1/auth/login
├─ Body: {email, password}
├─ Returns: {user, access_token, refresh_token}
└─ Status: 200 OK

POST /v1/auth/refresh
├─ Body: {refresh_token}
├─ Returns: {access_token}
└─ Status: 200 OK

POST /v1/auth/logout
├─ Headers: Authorization: Bearer {token}
├─ Returns: {success: true}
└─ Status: 200 OK
```

**Problem Endpoints:**

```http
GET /v1/problems
├─ Query: ?difficulty=easy&tags=array&page=1&limit=20
├─ Returns: {problems[], pagination}
├─ Caching: 5 minutes (problems rarely change)
└─ Status: 200 OK

GET /v1/problems/{slug}
├─ Path: slug (e.g., "two-sum")
├─ Returns: {problem details, examples, constraints}
├─ Caching: 1 hour (individual problems even more stable)
└─ Status: 200 OK or 404 Not Found

GET /v1/problems/{slug}/submissions
├─ Auth: Required
├─ Returns: User's submissions for this problem
└─ Status: 200 OK or 401 Unauthorized
```

**Submission Endpoints:**

```http
POST /v1/submissions
├─ Auth: Required
├─ Body: {problem_id, language, code}
├─ Returns: {submission_id, status: "pending"}
├─ Rate limit: 10/minute per user
└─ Status: 201 Created

GET /v1/submissions/{id}
├─ Auth: Required (only owner can view)
├─ Returns: {submission details, status, results}
└─ Status: 200 OK or 403 Forbidden

POST /v1/submissions/{id}/run
├─ Auth: Required
├─ Body: {test_cases: ["case1", "case2"]}
├─ Returns: {output, errors}
├─ Note: Runs code without judging (for testing)
└─ Status: 200 OK
```

**User Endpoints:**

```http
GET /v1/users/me
├─ Auth: Required
├─ Returns: {user profile, stats, solved_problems}
└─ Status: 200 OK

PATCH /v1/users/me
├─ Auth: Required
├─ Body: {full_name?, avatar_url?}
├─ Returns: {updated user}
└─ Status: 200 OK

GET /v1/users/{username}
├─ Auth: Optional (public profiles)
├─ Returns: {public profile, stats}
└─ Status: 200 OK or 404 Not Found
```

#### Rate Limiting Strategy

**Why rate limiting?**

```text
Without rate limiting:
Attacker: Sends 10,000 requests/second
Server: Crashes under load ❌

With rate limiting:
Attacker: Sends 10,000 requests/second
Server: Accepts first 100, blocks the rest ✅
Returns: 429 Too Many Requests
```

**Rate Limits by User Type:**

```text
Anonymous (no login):
├─ 20 requests/minute
├─ Prevents scraping without account
└─ Encourages registration

Authenticated Free User:
├─ 100 requests/minute
├─ Allows normal usage
└─ Prevents abuse

Premium User:
├─ 500 requests/minute
├─ Power users need higher limits
└─ Revenue incentive

Special: Code Submissions
├─ 10 submissions/minute (all users)
├─ Prevents brute-force spamming
└─ Protects judge system resources
```

**Implementation (Token Bucket Algorithm):**

```python
class RateLimiter:
    def __init__(self, max_requests=100, window_seconds=60):
        self.max_requests = max_requests
        self.window = window_seconds
        self.redis = Redis()
    
    def is_allowed(self, user_id):
        key = f"ratelimit:{user_id}"
        current_time = time.time()
        
        # Get request count in current window
        count = self.redis.incr(key)
        
        if count == 1:
            # First request in window, set expiration
            self.redis.expire(key, self.window)
        
        if count > self.max_requests:
            # Rate limit exceeded
            ttl = self.redis.ttl(key)
            raise RateLimitExceeded(f"Try again in {ttl} seconds")
        
        return True

# Usage in API endpoint
@app.post("/v1/submissions")
def submit_code(request):
    user_id = get_user_from_token(request.headers['Authorization'])
    
    # Check rate limit
    if not rate_limiter.is_allowed(user_id):
        return JSONResponse(
            status_code=429,
            content={"error": "Rate limit exceeded"},
            headers={"Retry-After": "60"}
        )
    
    # Process submission...
```

**Rate Limit Response Headers:**

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1635724800

(User has made 13 requests, 87 remaining in this window)
```

---

### 🔴 For Advanced: Production Considerations

#### API Versioning Strategies

**Why versioning matters:**

```text
Scenario: You need to change response format
Old API: {user_id: "123", name: "Alice"}
New API: {id: "123", firstName: "Alice", lastName: "Smith"}

Without versioning:
├─ Change breaks all existing mobile apps
├─ Users see blank screens
└─ 1-star reviews flood in ❌

With versioning:
├─ /v1/users → Old format (still works)
├─ /v2/users → New format (new apps use this)
└─ Gradual migration, no breaking changes ✅
```

**Versioning Approaches:**

**1. URL Path Versioning (Recommended):**
```text
https://api.leetcode.com/v1/problems
https://api.leetcode.com/v2/problems

Pros:
├─ Explicit and clear
├─ Easy to route different versions to different servers
├─ Can deprecate entire /v1/ and shut down servers
└─ Industry standard (Stripe, Twitter, GitHub use this)

Cons:
├─ URL changes (but that's the point!)
└─ Need to maintain code for multiple versions
```

**2. Header Versioning:**
```text
GET /problems
Accept: application/vnd.leetcode.v1+json

Pros:
├─ URL stays clean
└─ RESTful purists prefer this

Cons:
├─ Less visible (developers might miss it)
├─ Harder to debug (version in headers not URL)
└─ Can't cache different versions easily
```

**3. Query Parameter:**
```text
GET /problems?version=1

Pros:
├─ Easy to implement

Cons:
├─ Ugly URLs
├─ Optional parameter (users might forget it)
└─ Not recommended
```

**Deprecation Strategy:**

```text
v1 Released: Jan 2023
v2 Released: Jan 2024
├─ Announce v1 deprecation (6 months notice)
├─ Add deprecation header: Sunset: Sat, 01 Jul 2024 23:59:59 GMT
├─ Send emails to developers still using v1
└─ Provide migration guide

v1 Sunset: Jul 2024
├─ Return 410 Gone for v1 requests
├─ Point to migration docs
└─ Shut down v1 infrastructure (cost savings)

Timeline:
├─ Development: 3 months (build v2)
├─ Beta: 1 month (test with partners)
├─ Public release: Announce to all users
├─ Deprecation notice: 6 months before shutdown
└─ Shutdown: Delete v1 code
```

#### API Gateway Architecture

**Centralized API Gateway Benefits:**

```text
Without Gateway:
[Clients] → [Problem Service] → Auth logic duplicated
         → [Submission Service] → Auth logic duplicated
         → [User Service] → Auth logic duplicated

With Gateway:
[Clients] → [API Gateway] → [Problem Service]
              ├─ Auth          → [Submission Service]
              ├─ Rate Limiting → [User Service]
              ├─ Logging
              └─ Caching

Centralized cross-cutting concerns:
├─ Single point for authentication
├─ Consistent rate limiting
├─ Unified logging/metrics
└─ Easier to add features (all traffic goes through gateway)
```

**API Gateway Implementation (Kong/AWS API Gateway):**

```yaml
# Kong configuration example
services:
  - name: submission-service
    url: http://submission-service:3000
    routes:
      - paths: ["/v1/submissions"]
    plugins:
      - name: jwt
        config:
          secret: "jwt-secret-key"
      - name: rate-limiting
        config:
          minute: 100
          hour: 5000
      - name: request-transformer
        config:
          add:
            headers: ["X-Service:submission"]
      - name: response-cache
        config:
          ttl: 300

  - name: problem-service
    url: http://problem-service:3000
    routes:
      - paths: ["/v1/problems"]
    plugins:
      - name: response-cache
        config:
          ttl: 3600  # Cache problems for 1 hour
```

#### Advanced Authentication Patterns

**OAuth 2.0 for Third-Party Integrations:**

```text
Use case: "Login with LeetCode" button on other sites

Flow:
1. User clicks "Login with LeetCode" on thirdparty.com
   ↓
2. Redirected to leetcode.com/oauth/authorize
   ↓
3. User logs in and approves permission:
   "Allow thirdparty.com to view your profile and solved problems?"
   ↓
4. LeetCode redirects back with authorization code:
   thirdparty.com/callback?code=abc123
   ↓
5. thirdparty.com exchanges code for access token:
   POST /oauth/token {code: "abc123", client_secret: "xxx"}
   ↓
6. LeetCode returns access token
   ↓
7. thirdparty.com uses token to fetch user data:
   GET /v1/users/me {Authorization: Bearer token}
   ↓
8. thirdparty.com creates account for user

Security benefits:
├─ thirdparty.com never sees user's password
├─ User can revoke access anytime
├─ Limited scope (only approved permissions)
└─ Token expires (refresh token for long-term access)
```

**API Key for Programmatic Access:**

```text
Use case: Automated scripts, CI/CD integrations

Example:
# Generate API key (once)
POST /v1/api-keys {name: "CI/CD Pipeline", scopes: ["read:problems"]}
Returns: {api_key: "sk_live_abc123...", expires: never}

# Use API key in scripts
curl -H "Authorization: Bearer sk_live_abc123..." \
     https://api.leetcode.com/v1/problems

Advantages:
├─ No user login required (for automation)
├─ Can be rotated without changing code (regenerate key)
├─ Scoped permissions (read-only, write-only, etc.)
└─ Easy revocation (delete key)

Security:
├─ Store in environment variables, never in code
├─ Rotate regularly (every 90 days)
├─ Monitor usage (alert on unusual patterns)
└─ Rate limit per API key
```

#### GraphQL Alternative

**Why GraphQL for coding platforms?**

```text
REST Problems:
├─ Over-fetching: GET /problems returns ALL fields, even if you only need title
├─ Under-fetching: Need 3 requests to get problem + submissions + user stats
└─ No flexibility: Mobile app needs different fields than web

GraphQL Solution:
query {
  problem(slug: "two-sum") {
    title
    difficulty
    mySubmissions(limit: 5) {
      status
      runtime
    }
  }
  me {
    solvedCount
  }
}

Single request, exact data needed!
```

**GraphQL Schema Example:**

```graphql
type Query {
  problems(
    difficulty: Difficulty
    tags: [String]
    page: Int
    limit: Int
  ): ProblemConnection!
  
  problem(slug: String!): Problem
  
  me: User!
}

type Problem {
  id: ID!
  title: String!
  slug: String!
  difficulty: Difficulty!
  description: String!
  tags: [Tag!]!
  submissions(limit: Int): [Submission!]!
  acceptanceRate: Float!
}

type Submission {
  id: ID!
  code: String!
  language: Language!
  status: SubmissionStatus!
  runtime: Int
  memory: Int
  submittedAt: DateTime!
}

enum Difficulty {
  EASY
  MEDIUM
  HARD
}

enum SubmissionStatus {
  PENDING
  ACCEPTED
  WRONG_ANSWER
  TIME_LIMIT_EXCEEDED
}
```

**REST vs GraphQL Trade-offs:**

```text
REST Advantages:
├─ Simpler (easier to learn and implement)
├─ Better caching (HTTP cache works out of box)
├─ Smaller payload (no query parsing overhead)
└─ Mature ecosystem (tools, libraries, knowledge)

GraphQL Advantages:
├─ Flexible queries (clients get exactly what they need)
├─ Single endpoint (no versioning needed, just add fields)
├─ Strongly typed (schema validation)
└─ Great for complex, nested data

Recommendation for coding platform:
├─ REST for public API (simplicity, caching)
├─ GraphQL for internal tools (flexibility)
└─ Hybrid: Offer both (Stripe, GitHub do this)
```

---

### Real-World Example: LeetCode's API Evolution

**2015: Simple REST API**
```text
Endpoints:
├─ GET /problems
├─ POST /submit
└─ GET /user/stats

Problems:
├─ No versioning (breaking changes broke mobile apps)
├─ No rate limiting (scrapers overloaded servers)
└─ No pagination (returning 1000+ problems in one request)
```

**2017: Versioned REST API**
```text
Improvements:
├─ /v1/ prefix for versioning
├─ JWT authentication (was basic auth before)
├─ Rate limiting (100 req/min authenticated)
├─ Pagination (page/limit params)
└─ API documentation (Swagger/OpenAPI)

Still problems:
├─ Over-fetching (mobile app got too much data)
└─ Multiple requests needed for dashboard (slow on mobile)
```

**2020: GraphQL + REST Hybrid**
```text
Added:
├─ GraphQL endpoint (/graphql) for web/mobile
├─ REST API (/v1/) maintained for backward compatibility
├─ API Gateway (Kong) for unified management
├─ OAuth 2.0 for third-party integrations
└─ Webhooks for real-time notifications

Architecture:
[Clients]
    ↓
[CloudFlare CDN] (DDoS protection)
    ↓
[Kong API Gateway]
    ├─ Auth, rate limiting, logging
    ↓
[GraphQL Federation]
    ├─ Problem subgraph
    ├─ User subgraph
    ├─ Submission subgraph
    └─ Contest subgraph

Benefits:
├─ 50% fewer API calls (GraphQL efficiency)
├─ 90% cache hit rate (CDN + Redis)
├─ Zero downtime deployments (blue-green via gateway)
└─ API versioning without code duplication
```

**2023: Current State**
```text
REST API (/v1/):
├─ Used by: Public integrations, CI/CD tools, scrapers
├─ Cached heavily (CloudFlare + Redis)
├─ Rate limited strictly (prevent abuse)
└─ Simple CRUD operations

GraphQL (/graphql):
├─ Used by: Web app, mobile apps, internal tools
├─ Batching/caching with DataLoader
├─ Subscriptions for real-time updates (contest leaderboards)
└─ Complex, nested queries

Technology Stack:
├─ API Gateway: Kong Enterprise
├─ GraphQL: Apollo Federation (Node.js)
├─ Authentication: Auth0 (managed service)
├─ Rate Limiting: Redis + Lua scripts
└─ Monitoring: Datadog APM
```

---

### 🎯 Interview Questions: API Design

**Question 1:** A user submits code, which is judged asynchronously. How should the API handle this?

<details>
<summary>💡 Hint</summary>
Think about async processing, polling vs WebSockets, and when to return a response.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Problem:**
```text
Code execution takes 1-3 seconds. Can't block API request for that long.
```

**Solution: Async Processing with Multiple Options**

**Option 1: Polling (Simpler)**

```text
POST /v1/submissions
├─ Immediately returns: {submission_id: "sub-123", status: "pending"}
├─ Status: 202 Accepted (not 200 OK, signals async processing)
└─ Client polls for results

Client polls:
GET /v1/submissions/sub-123 (every 1 second)
├─ While pending: {status: "pending"}
├─ Once complete: {status: "accepted", runtime: 45ms, memory: 12MB}
└─ Client stops polling
```

**Implementation:**
```python
@app.post("/v1/submissions")
def submit_code(request):
    # 1. Validate and save submission
    submission = db.insert({
        'user_id': request.user_id,
        'problem_id': request.problem_id,
        'code_s3_key': upload_to_s3(request.code),
        'status': 'pending'
    })
    
    # 2. Enqueue for judging (non-blocking)
    queue.send({
        'submission_id': submission.id,
        'code_s3_key': submission.code_s3_key
    })
    
    # 3. Return immediately
    return JSONResponse(
        status_code=202,  # Accepted
        content={
            "submission_id": submission.id,
            "status": "pending",
            "poll_url": f"/v1/submissions/{submission.id}"
        },
        headers={
            "Location": f"/v1/submissions/{submission.id}"
        }
    )

@app.get("/v1/submissions/{id}")
def get_submission(id):
    submission = db.get(id)
    
    if submission.status == 'pending':
        return JSONResponse(
            status_code=200,
            content={"status": "pending"},
            headers={"Retry-After": "1"}  # Suggest polling interval
        )
    else:
        return JSONResponse(
            status_code=200,
            content={
                "status": submission.status,
                "runtime": submission.runtime_ms,
                "memory": submission.memory_kb,
                "test_cases_passed": submission.test_cases_passed
            }
        )
```

**Option 2: WebSockets (More Complex, Real-Time)**

```text
1. User connects to WebSocket:
   ws://api.leetcode.com/v1/submissions/listen

2. User submits code via REST:
   POST /v1/submissions
   Returns: {submission_id: "sub-123", status: "pending"}

3. Server pushes updates via WebSocket:
   {"submission_id": "sub-123", "event": "judging_started"}
   {"submission_id": "sub-123", "event": "test_case_1_passed"}
   {"submission_id": "sub-123", "event": "test_case_2_passed"}
   {"submission_id": "sub-123", "event": "completed", "status": "accepted"}

4. User receives real-time updates (no polling needed!)
```

**Implementation:**
```python
from fastapi import WebSocket

@app.websocket("/v1/submissions/listen")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    user_id = authenticate_websocket(websocket)
    
    # Subscribe to Redis channel for this user's submissions
    pubsub = redis.pubsub()
    pubsub.subscribe(f"user:{user_id}:submissions")
    
    try:
        for message in pubsub.listen():
            # Forward updates to client
            await websocket.send_json(message['data'])
    except WebSocketDisconnect:
        pubsub.unsubscribe()

# When judge completes, publish to Redis
def on_submission_complete(submission_id, user_id, results):
    redis.publish(f"user:{user_id}:submissions", {
        "submission_id": submission_id,
        "event": "completed",
        "results": results
    })
```

**Option 3: Webhooks (For Integrations)**

```text
User registers webhook URL:
POST /v1/webhooks
{
  "url": "https://myapp.com/leetcode-callback",
  "events": ["submission.completed"]
}

When submission completes:
Server sends:
POST https://myapp.com/leetcode-callback
{
  "event": "submission.completed",
  "submission_id": "sub-123",
  "status": "accepted",
  "timestamp": "2025-01-15T10:30:00Z"
}
```

**Comparison:**

```text
Polling:
├─ Pros: Simple, works everywhere (HTTP only)
├─ Cons: Inefficient (many unnecessary requests), delayed updates
└─ Use when: Simplicity matters, real-time not critical

WebSockets:
├─ Pros: True real-time, efficient (push-based)
├─ Cons: Complex (maintain connections), harder to scale, some firewalls block
└─ Use when: Real-time critical (contests, live coding interviews)

Webhooks:
├─ Pros: No polling, event-driven, scalable
├─ Cons: Requires receiver to have public endpoint, retry logic complex
└─ Use when: Server-to-server integrations, CI/CD pipelines

Recommendation for LeetCode:
├─ Public users: Polling (simple, works everywhere)
├─ Contests: WebSockets (real-time leaderboards)
└─ Integrations: Webhooks (automated workflows)
```

</details>

**Question 2:** How would you design an API that returns different data for free vs premium users without duplicating endpoints?

<details>
<summary>💡 Hint</summary>
Think about field-level authorization and dynamic response filtering.
</details>

<details>
<summary>✅ Sample Answer</summary>

**Approach: Single Endpoint with Dynamic Filtering**

```text
Bad (duplication):
❌ GET /v1/problems (free users)
❌ GET /v1/premium/problems (premium users)

Good (single endpoint):
✅ GET /v1/problems (dynamic based on user tier)
```

**Implementation Strategies:**

**1. Field-Level Filtering:**

```python
@app.get("/v1/problems/{slug}")
def get_problem(slug: str, current_user: User):
    problem = db.get_problem(slug)
    
    # Base response (available to all)
    response = {
        "title": problem.title,
        "difficulty": problem.difficulty,
        "description": problem.description,
        "examples": problem.examples
    }
    
    # Premium-only fields
    if current_user.is_premium:
        response.update({
            "hints": problem.hints,  # Premium only
            "video_explanation_url": problem.video_url,  # Premium only
            "similar_problems": problem.similar_problems,  # Premium only
            "optimal_solution": problem.optimal_solution  # Premium only
        })
    
    # Premium indicator (so free users know what they're missing)
    if not current_user.is_premium:
        response["premium_features_available"] = [
            "hints", "video_explanation", "similar_problems", "optimal_solution"
        ]
    
    return response
```

**Response for Free User:**
```json
{
  "title": "Two Sum",
  "difficulty": "easy",
  "description": "Given an array...",
  "examples": [...],
  "premium_features_available": ["hints", "video_explanation", ...]
}
```

**Response for Premium User:**
```json
{
  "title": "Two Sum",
  "difficulty": "easy",
  "description": "Given an array...",
  "examples": [...],
  "hints": ["Try using a hash map", "Think about O(n) time"],
  "video_explanation_url": "https://...",
  "similar_problems": ["Three Sum", "Four Sum"],
  "optimal_solution": "Use hash map for O(n) time..."
}
```

**2. GraphQL Approach (Even Better):**

```graphql
type Problem {
  title: String!
  difficulty: Difficulty!
  description: String!
  examples: [Example!]!
  
  # Premium fields with @auth directive
  hints: [String!] @requiresPremium
  videoExplanation: String @requiresPremium
  similarProblems: [Problem!] @requiresPremium
}

# Custom directive
directive @requiresPremium on FIELD_DEFINITION
```

**Resolver with field-level auth:**
```javascript
const resolvers = {
  Problem: {
    hints: (problem, args, context) => {
      if (!context.user.isPremium) {
        throw new Error("Premium subscription required");
      }
      return problem.hints;
    },
    
    videoExplanation: (problem, args, context) => {
      if (!context.user.isPremium) {
        return null;  // Or throw error, depending on UX
      }
      return problem.videoExplanationUrl;
    }
  }
};
```

**Query:**
```graphql
query {
  problem(slug: "two-sum") {
    title
    difficulty
    description
    hints  # Only returned if user is premium
    videoExplanation  # Only returned if user is premium
  }
}
```

**3. Response Transformation Middleware:**

```python
class PremiumFilterMiddleware:
    def __init__(self, app):
        self.app = app
    
    async def __call__(self, request, call_next):
        response = await call_next(request)
        
        # Parse response JSON
        body = await response.body()
        data = json.loads(body)
        
        # Filter based on user tier
        user = request.state.user
        if not user.is_premium:
            data = self.filter_premium_fields(data)
        
        # Return modified response
        return JSONResponse(content=data)
    
    def filter_premium_fields(self, data):
        premium_fields = ['hints', 'video_explanation_url', 'optimal_solution']
        
        # Recursively remove premium fields
        if isinstance(data, dict):
            return {
                k: self.filter_premium_fields(v)
                for k, v in data.items()
                if k not in premium_fields
            }
        elif isinstance(data, list):
            return [self.filter_premium_fields(item) for item in data]
        else:
            return data
```

**4. Rate Limiting by Tier:**

```python
def get_rate_limit(user):
    if user.is_premium:
        return 500  # requests per minute
    elif user.is_authenticated:
        return 100
    else:
        return 20  # anonymous

@app.get("/v1/problems")
async def list_problems(request):
    user = get_user_from_token(request)
    limit = get_rate_limit(user)
    
    if not check_rate_limit(user.id, limit):
        return JSONResponse(
            status_code=429,
            content={
                "error": "Rate limit exceeded",
                "limit": limit,
                "upgrade_url": "/pricing" if not user.is_premium else None
            }
        )
    
    # Process request...
```

**5. API Documentation (OpenAPI):**

```yaml
/problems/{slug}:
  get:
    summary: Get problem details
    parameters:
      - name: slug
        in: path
        required: true
        schema:
          type: string
    responses:
      200:
        description: Problem details
        content:
          application/json:
            schema:
              type: object
              properties:
                title:
                  type: string
                difficulty:
                  type: string
                  enum: [easy, medium, hard]
                description:
                  type: string
                hints:
                  type: array
                  items:
                    type: string
                  description: "⭐ Premium only"
                video_explanation_url:
                  type: string
                  description: "⭐ Premium only"
```

**Benefits:**

```text
Single Endpoint:
├─ No code duplication
├─ Easier to maintain (one place to update)
├─ Consistent API contract
└─ Clear upgrade path (users see what they're missing)

Field-Level Control:
├─ Fine-grained permissions
├─ Can mix free and premium features
├─ Easy to A/B test (make field premium or free)
└─ Future-proof (add new tiers easily)

Business Value:
├─ Free users see what they're missing (conversion funnel)
├─ Premium users get more data (value for money)
├─ Can track which premium features are most accessed
└─ Easy to offer limited-time promotions (temporarily unlock premium fields)
```

</details>

---

### 🤔 Think About It

1. **Why use JWT instead of storing sessions in database?**
   - Hint: Think about scalability and stateless servers

2. **What's the security risk of returning detailed error messages in API responses?**
   - Hint: "User with email alice@example.com not found" vs "Invalid credentials"

3. **How would you handle API requests from mobile apps that go offline?**
   - Hint: Request queuing, offline-first architecture

4. **Should you version every change to the API or only breaking changes?**
   - Hint: Backward compatible additions vs breaking removals

---

### ✅ Key Takeaways

🎯 **API Design Principles:**
- **Resource-oriented URLs** (nouns not verbs): /problems not /getProblems
- **HTTP status codes** matter: 401 (who are you?) vs 403 (you can't do this)
- **Pagination** is mandatory: Never return thousands of items at once
- **Versioning from day one**: /v1/ prefix prevents future headaches

🔐 **Authentication & Security:**
- **JWT for stateless auth**: Scales horizontally, no server-side session storage
- **Rate limiting by tier**: Anonymous (20/min), Free (100/min), Premium (500/min)
- **Separate submission limits**: 10 submissions/minute prevents abuse

📊 **API Patterns:**
- **Async processing**: 202 Accepted for long-running operations
- **Polling vs WebSockets**: Polling for simplicity, WebSockets for real-time
- **Field-level authorization**: Single endpoint, dynamic response based on user tier

💡 **Interview Tips:**
- Always mention rate limiting (prevents DOS attacks)
- Discuss versioning strategy (URL path is industry standard)
- Explain authentication flow (JWT creation, verification, expiry)
- Consider mobile use cases (offline support, smaller payloads)

---

### 🎯 Practice Exercise

**Challenge:** Design API endpoints for a coding contest feature

**Requirements:**
- Users can register for upcoming contests
- Contests have start/end times (e.g., 2 hours)
- During contest, users submit code for contest problems
- Real-time leaderboard (updated every 10 seconds)
- After contest, users can view all solutions

**Deliverables:**
1. List 5-7 API endpoints needed
2. Specify HTTP methods and request/response formats
3. Explain authentication requirements
4. Discuss real-time leaderboard strategy (polling vs WebSocket)

<details>
<summary>📝 Sample Solution</summary>

**API Endpoints:**

```http
1. POST /v1/contests/{id}/register
   ├─ Auth: Required
   ├─ Body: {}
   ├─ Response: {registered: true, contest_start: "2025-01-15T10:00:00Z"}
   └─ Idempotent: Re-registering returns same response

2. GET /v1/contests/{id}
   ├─ Auth: Optional (public info visible, private only for participants)
   ├─ Response:
      {
        "title": "Weekly Contest 123",
        "start_time": "2025-01-15T10:00:00Z",
        "end_time": "2025-01-15T12:00:00Z",
        "duration_minutes": 120,
        "problems": [...] // Only visible to registered users during contest
        "participant_count": 5000
      }

3. GET /v1/contests/{id}/problems
   ├─ Auth: Required (must be registered)
   ├─ Available: Only during contest time window
   ├─ Response: [{problem_id, title, difficulty, points}, ...]
   └─ Rate limit: 100/minute (prevent scraping)

4. POST /v1/contests/{id}/submissions
   ├─ Auth: Required
   ├─ Available: Only during contest time window
   ├─ Body: {problem_id, language, code}
   ├─ Response: {submission_id, status: "pending"}
   ├─ Rate limit: 10/minute
   └─ Validation: Ensure contest is active, user is registered

5. GET /v1/contests/{id}/leaderboard
   ├─ Auth: Optional (public after contest starts)
   ├─ Available: During and after contest
   ├─ Query params: ?page=1&limit=100
   ├─ Response:
      {
        "rankings": [
          {
            "rank": 1,
            "username": "alice",
            "score": 300,
            "penalty_time": 1200,  // Seconds
            "problems_solved": 3
          }
        ],
        "updated_at": "2025-01-15T10:15:30Z"
      }
   ├─ Caching: 10 seconds during contest, 1 hour after
   └─ Alternative: WebSocket at /v1/contests/{id}/leaderboard/stream

6. GET /v1/contests/{id}/my-submissions
   ├─ Auth: Required
   ├─ Response: User's submissions for this contest
   └─ Use case: "Show me my submission history"

7. GET /v1/contests/{id}/solutions
   ├─ Auth: Required
   ├─ Available: Only after contest ends
   ├─ Response: Top solutions for each problem (community voted)
   └─ Purpose: Learning from others after contest
```

**Real-Time Leaderboard Strategy:**

**Option A: Polling (Recommended for MVP)**
```javascript
// Client-side
let leaderboardInterval;

function startContest() {
  // Poll every 10 seconds during contest
  leaderboardInterval = setInterval(() => {
    fetch(`/v1/contests/${contestId}/leaderboard`)
      .then(res => res.json())
      .then(data => updateLeaderboard(data.rankings));
  }, 10000);  // 10 seconds
}

function endContest() {
  clearInterval(leaderboardInterval);
}
```

**Server-side caching:**
```python
@app.get("/v1/contests/{id}/leaderboard")
def get_leaderboard(id: str):
    # Check cache first
    cached = redis.get(f"contest:{id}:leaderboard")
    if cached:
        return json.loads(cached)
    
    # Compute leaderboard
    rankings = db.query("""
        SELECT 
            cp.user_id,
            u.username,
            cp.total_score,
            cp.penalty_time,
            cp.rank
        FROM contest_participants cp
        JOIN users u ON cp.user_id = u.user_id
        WHERE cp.contest_id = %s
        ORDER BY cp.total_score DESC, cp.penalty_time ASC
        LIMIT 100
    """, [id])
    
    # Cache for 10 seconds
    redis.setex(
        f"contest:{id}:leaderboard",
        10,  # TTL
        json.dumps(rankings)
    )
    
    return rankings
```

**Benefits:**
- Simple to implement
- Works with HTTP load balancers
- 10K users polling every 10s = 1000 QPS (manageable with caching)

**Option B: WebSocket (For Better UX)**
```javascript
// Client-side
const ws = new WebSocket(`wss://api.leetcode.com/v1/contests/${contestId}/leaderboard/stream`);

ws.onmessage = (event) => {
  const update = JSON.parse(event.data);
  
  if (update.type === 'full_leaderboard') {
    updateLeaderboard(update.rankings);
  } else if (update.type === 'rank_change') {
    updateSingleRank(update.user, update.new_rank);
  }
};
```

**Server-side:**
```python
from fastapi import WebSocket

@app.websocket("/v1/contests/{id}/leaderboard/stream")
async def leaderboard_stream(websocket: WebSocket, id: str):
    await websocket.accept()
    
    # Subscribe to Redis pub/sub
    pubsub = redis.pubsub()
    pubsub.subscribe(f"contest:{id}:leaderboard_updates")
    
    # Send initial state
    leaderboard = get_leaderboard(id)
    await websocket.send_json({
        "type": "full_leaderboard",
        "rankings": leaderboard
    })
    
    # Stream updates
    try:
        for message in pubsub.listen():
            await websocket.send_json(message['data'])
    except WebSocketDisconnect:
        pubsub.unsubscribe()

# When submission is judged, publish update
def on_submission_judged(contest_id, user_id, new_score):
    # Recalculate rank
    new_rank = calculate_rank(contest_id, user_id)
    
    # Publish to all connected clients
    redis.publish(f"contest:{contest_id}:leaderboard_updates", {
        "type": "rank_change",
        "user_id": user_id,
        "new_score": new_score,
        "new_rank": new_rank
    })
```

**Benefits:**
- True real-time (updates within 100ms)
- Efficient (push-based, not polling)
- Better UX during live contests

**Challenges:**
- More complex infrastructure
- Need sticky sessions (or Redis pub/sub for multi-server)
- 10K concurrent WebSockets = more memory/connections

**Hybrid Approach (Best):**
```text
During contest:
├─ Top 100 users: WebSocket (competitive, want real-time)
├─ Other users: Polling every 30 seconds (less competitive)
└─ Automatically upgrade to WebSocket if user enters top 100

After contest:
├─ Everyone: HTTP with 1-hour cache
└─ WebSocket connections closed
```

**Authentication for Contest Endpoints:**

```python
def verify_contest_access(contest_id: str, user_id: str, action: str):
    contest = db.get_contest(contest_id)
    participant = db.get_participant(contest_id, user_id)
    
    # Check if user is registered
    if action in ['submit', 'view_problems']:
        if not participant:
            raise HTTPException(403, "Not registered for contest")
    
    # Check if contest is active
    if action == 'submit':
        now = datetime.now()
        if now < contest.start_time:
            raise HTTPException(403, "Contest hasn't started yet")
        if now > contest.end_time:
            raise HTTPException(403, "Contest has ended")
    
    # Check if solutions are available
    if action == 'view_solutions':
        if datetime.now() < contest.end_time:
            raise HTTPException(403, "Solutions available after contest")
    
    return True
```

</details>

**Time Budget:** 30 minutes  
**Difficulty:** 🟡 Intermediate

---

*Next Up:* In Section 6, we'll dive deep into the judge system - the heart of our platform. We'll explore Docker containerization, sandboxing strategies, and security measures to safely execute untrusted code. Ready to build a secure code execution engine? 🔒

