# LEETCODE SYSTEM DESIGN
## Online Coding Platform with Judge System

**Document Purpose:** This file contains a comprehensive system design for a LeetCode-like online coding platform that allows users to solve coding problems, submit solutions, and get real-time feedback on their code execution.

**Last Updated:** October 1, 2025

---

## RULE 1: REQUIREMENTS & CLARIFICATION

### User Stories

**As a problem solver**, I want to browse and filter coding problems by difficulty and topic so that I can practice specific skills.

**As a problem solver**, I want to write and submit code solutions in multiple programming languages so that I can use my preferred language.

**As a problem solver**, I want to receive immediate feedback on my submission so that I know if my solution is correct.

**As a problem solver**, I want to see my submission history and statistics so that I can track my progress.

**As a content creator**, I want to create and publish coding problems with test cases so that others can practice.

**As a competitive coder**, I want to participate in timed coding contests so that I can challenge myself and compete with others.

---

### Functional Requirements (MVP)

1. **Problem Management**
   - Browse problems with filtering (difficulty, topic, status)
   - View problem description, constraints, and examples
   - Search problems by title or tags

2. **Code Submission & Execution**
   - Write code in an online editor with syntax highlighting
   - Submit code for evaluation
   - Run code against test cases
   - Support multiple programming languages (Python, Java, C++, JavaScript)

3. **Judge System**
   - Execute submitted code in isolated environment
   - Run against predefined test cases
   - Return execution results (pass/fail, runtime, memory usage)
   - Handle timeouts and runtime errors

4. **User Management**
   - User registration and authentication
   - User profile with submission history
   - Track solved problems and statistics

5. **Submission History**
   - View past submissions
   - See submission status and details
   - Access submitted code

---

### Non-Functional Requirements

**Availability:**
- 99.9% uptime (8.76 hours downtime per year)
- Judge system should handle failures gracefully

**Performance:**
- Code execution results within 10 seconds for 95% of submissions
- Problem listing page load < 2 seconds
- Support concurrent code executions

**Scalability:**
- Support 10 million registered users
- Handle 100,000 daily active users
- Process 500,000 submissions per day

**Security:**
- Sandboxed code execution environment
- Prevention of malicious code execution
- No access to system resources from user code
- Rate limiting on submissions

**Consistency:**
- Eventual consistency acceptable for leaderboards
- Strong consistency for submission results
- Strong consistency for user account data

---

### Clarifying Questions & Assumptions

**Scale Questions:**
- **Q:** How many daily active users?
  - **A:** Assuming 100,000 DAU
- **Q:** What's the expected growth rate?
  - **A:** Assuming 20% YoY growth
- **Q:** Geographic distribution?
  - **A:** Global, primarily US, Europe, Asia

**Usage Pattern Questions:**
- **Q:** What's the read/write ratio?
  - **A:** 80:20 (browsing vs submitting)
- **Q:** Average submissions per active user?
  - **A:** 5 submissions per day per active user
- **Q:** Peak usage times?
  - **A:** Evenings and weekends, 3x average load

**Feature Scope Questions:**
- **Q:** Do we need real-time collaboration features?
  - **A:** No, out of scope for MVP
- **Q:** Do we need video explanations?
  - **A:** No, out of scope for MVP
- **Q:** Do we need discussion forums?
  - **A:** No, out of scope for MVP
- **Q:** Do we need contests?
  - **A:** Phase 2, not in MVP

**Technical Questions:**
- **Q:** Which programming languages to support?
  - **A:** Python, Java, C++, JavaScript (can add more later)
- **Q:** Maximum code execution time?
  - **A:** 10 seconds timeout
- **Q:** Maximum memory per execution?
  - **A:** 256 MB per execution

---

## RULE 2: BACK-OF-THE-ENVELOPE CALCULATIONS

### Traffic Estimates

```
Daily Active Users (DAU): 100,000
Active users submitting code: 100,000 × 100% = 100,000 users

Submissions per user per day: 5
Total daily submissions: 100,000 × 5 = 500,000 submissions

Seconds in a day: 86,400
Average submissions per second: 500,000 / 86,400 ≈ 6 QPS
Peak submissions per second (3x): 6 × 3 = 18 QPS

Problem views per user per day: 10
Total daily problem views: 100,000 × 10 = 1,000,000 views
Average problem views per second: 1,000,000 / 86,400 ≈ 12 QPS
Peak problem views per second (3x): 36 QPS

Code execution requests (including test runs): 500,000 × 2 = 1,000,000 executions/day
Average executions per second: 1,000,000 / 86,400 ≈ 12 QPS
Peak executions per second (3x): 36 QPS
```

### Storage Estimates

```
PROBLEM DATA:
- Total problems in system: 3,000 problems
- Average problem size: 5 KB (description, examples, constraints)
- Total problem storage: 3,000 × 5 KB = 15 MB

TEST CASES:
- Test cases per problem: 50 (average)
- Size per test case: 1 KB (input + expected output)
- Total test case storage: 3,000 × 50 × 1 KB = 150 MB

USER DATA:
- Total registered users: 10,000,000
- User profile data: 2 KB per user
- Total user storage: 10,000,000 × 2 KB = 20 GB

SUBMISSION DATA:
- Daily submissions: 500,000
- Average code size: 2 KB
- Metadata per submission: 500 bytes (status, runtime, memory, timestamp)
- Size per submission: 2 KB + 500 bytes = 2.5 KB

Annual submissions: 500,000 × 365 = 182,500,000 submissions
Annual submission storage: 182,500,000 × 2.5 KB = 456 GB ≈ 0.5 TB

3-year submission storage: 0.5 TB × 3 = 1.5 TB
5-year submission storage: 0.5 TB × 5 = 2.5 TB

TOTAL STORAGE (3 years):
- Problems: 15 MB
- Test cases: 150 MB
- Users: 20 GB
- Submissions: 1.5 TB
- Total: ≈ 1.5 TB (submissions dominate)
```

### Resource Estimates

```
CONCURRENT EXECUTIONS AT PEAK:
- Peak execution QPS: 36
- Average execution time: 3 seconds
- Concurrent executions: 36 × 3 = 108 concurrent executions

JUDGE WORKERS:
- Concurrent executions needed: 108
- Buffer for spikes (2x): 216
- Recommended worker pool size: 250 workers

COMPUTE RESOURCES PER WORKER:
- CPU: 1 vCPU per worker
- Memory: 512 MB per worker (256 MB for code + overhead)
- Total cluster: 250 vCPUs, 125 GB RAM

DATABASE CONNECTIONS:
- API servers: 20 instances × 50 connections = 1,000 connections
- Background services: 200 connections
- Total: 1,200 connections (well within PostgreSQL limits)
```

### Bandwidth Estimates

```
SUBMISSION REQUEST:
- Code: 2 KB
- Metadata: 500 bytes
- Total request size: 2.5 KB

SUBMISSION RESPONSE:
- Results: 1 KB (status, runtime, memory, test case results)
- Total response size: 1 KB

PROBLEM VIEW REQUEST:
- Request: 500 bytes
- Response: 5 KB (problem data)

Peak bandwidth (submissions): 18 QPS × 3.5 KB = 63 KB/s ≈ 0.5 Mbps
Peak bandwidth (problem views): 36 QPS × 5.5 KB = 198 KB/s ≈ 1.6 Mbps
Total peak bandwidth: ≈ 2 Mbps (negligible)
```

---

## RULE 3: HIGH-LEVEL DESIGN

### System Architecture Overview

```mermaid
graph TB
    subgraph Client Layer
        User[User/Browser]
    end
    
    subgraph CDN Layer
        CDN[CDN<br/>CloudFront/Cloudflare]
    end
    
    subgraph Load Balancing
        LB[Load Balancer<br/>AWS ELB/Nginx]
    end
    
    subgraph API Layer
        API1[API Server 1<br/>Node.js/Express]
        API2[API Server 2<br/>Node.js/Express]
        API3[API Server N<br/>Node.js/Express]
    end
    
    subgraph Service Layer
        ProblemSvc[Problem Service<br/>Go/Java]
        UserSvc[User Service<br/>Go/Java]
        SubmissionSvc[Submission Service<br/>Go/Java]
        JudgeSvc[Judge Coordinator<br/>Go]
    end
    
    subgraph Execution Layer
        Queue[Task Queue<br/>RabbitMQ/Kafka]
        Worker1[Judge Worker 1<br/>Docker]
        Worker2[Judge Worker 2<br/>Docker]
        Worker3[Judge Worker N<br/>Docker]
    end
    
    subgraph Cache Layer
        Redis[Redis Cache<br/>Problem data, User sessions]
    end
    
    subgraph Data Layer
        ProblemDB[(Problem DB<br/>PostgreSQL)]
        UserDB[(User DB<br/>PostgreSQL)]
        SubmissionDB[(Submission DB<br/>PostgreSQL)]
        TestCaseStore[Test Case Storage<br/>S3/Object Storage]
    end
    
    subgraph Storage Layer
        CodeStore[Code Storage<br/>S3/Object Storage]
    end
    
    User -->|1. Browse/Submit| CDN
    CDN -->|2. Static Assets| User
    User -->|3. API Requests| LB
    LB -->|4. Route| API1
    LB -->|4. Route| API2
    LB -->|4. Route| API3
    
    API1 -->|5. Get Problems| ProblemSvc
    API2 -->|5. Authenticate| UserSvc
    API3 -->|5. Submit Code| SubmissionSvc
    
    ProblemSvc -->|6. Query| ProblemDB
    ProblemSvc -->|6. Cache| Redis
    UserSvc -->|6. Query| UserDB
    UserSvc -->|6. Session| Redis
    
    SubmissionSvc -->|7. Store Code| CodeStore
    SubmissionSvc -->|8. Create Record| SubmissionDB
    SubmissionSvc -->|9. Enqueue Job| Queue
    
    Queue -->|10. Pick Job| Worker1
    Queue -->|10. Pick Job| Worker2
    Queue -->|10. Pick Job| Worker3
    
    Worker1 -->|11. Get Test Cases| TestCaseStore
    Worker2 -->|11. Get Test Cases| TestCaseStore
    Worker3 -->|11. Get Test Cases| TestCaseStore
    
    Worker1 -->|12. Update Result| SubmissionDB
    Worker2 -->|12. Update Result| SubmissionDB
    Worker3 -->|12. Update Result| SubmissionDB
    
    JudgeSvc -->|Monitor| Queue
    JudgeSvc -->|Scale| Worker1
```

### Data Flow Explanation

**User Browsing Problems:**
1. User accesses the platform through browser
2. CDN serves static assets (HTML, CSS, JS, images)
3. Browser makes API request to load balancer
4. Load balancer routes request to available API server
5. API server calls Problem Service to fetch problems
6. Problem Service checks Redis cache first
7. If cache miss, queries Problem DB and updates cache
8. Response flows back through API server to user

**Code Submission Flow:**
1. User submits code through the web interface
2. Request hits load balancer
3. Load balancer routes to API server
4. API server authenticates user via User Service
5. Submission Service stores code in S3/Object Storage
6. Submission Service creates submission record in Submission DB
7. Submission Service enqueues execution job in task queue
8. Judge Worker picks up job from queue
9. Worker fetches test cases from Test Case Storage
10. Worker executes code in isolated Docker container
11. Worker runs code against test cases with timeout/memory limits
12. Worker updates submission status and results in Submission DB
13. User can poll or receive real-time updates on submission status

**Component Purpose:**

- **CDN:** Serves static assets globally with low latency
- **Load Balancer:** Distributes traffic across API servers
- **API Servers:** Handle HTTP requests, authentication, routing
- **Problem Service:** Manages problem CRUD operations
- **User Service:** Handles authentication, user profiles, statistics
- **Submission Service:** Manages submission lifecycle
- **Judge Coordinator:** Monitors queue health, scales workers
- **Task Queue:** Decouples submission from execution (async processing)
- **Judge Workers:** Execute code in sandboxed Docker containers
- **Redis Cache:** Reduces DB load for frequently accessed data
- **Databases:** Store persistent data with strong consistency
- **Object Storage:** Store large binary objects (code files, test cases)

---

## RULE 4: DEEP-DIVE INTO DETAILS

### Database Design

#### Users Table
```sql
users
- user_id (PK, UUID)
- username (VARCHAR(50), UNIQUE, NOT NULL)
- email (VARCHAR(255), UNIQUE, NOT NULL)
- password_hash (VARCHAR(255), NOT NULL)
- full_name (VARCHAR(100))
- avatar_url (VARCHAR(500))
- created_at (TIMESTAMP, DEFAULT NOW())
- updated_at (TIMESTAMP, DEFAULT NOW())
- last_login_at (TIMESTAMP)
- is_active (BOOLEAN, DEFAULT TRUE)
- role (ENUM: 'user', 'admin', 'content_creator')

Indexes:
- PRIMARY KEY (user_id)
- UNIQUE INDEX idx_username (username)
- UNIQUE INDEX idx_email (email)
- INDEX idx_created_at (created_at)
```

#### Problems Table
```sql
problems
- problem_id (PK, UUID)
- title (VARCHAR(200), NOT NULL)
- slug (VARCHAR(200), UNIQUE, NOT NULL)
- description (TEXT, NOT NULL)
- difficulty (ENUM: 'easy', 'medium', 'hard', NOT NULL)
- acceptance_rate (DECIMAL(5,2))
- total_accepted (INT, DEFAULT 0)
- total_submissions (INT, DEFAULT 0)
- created_by (FK -> users.user_id)
- created_at (TIMESTAMP, DEFAULT NOW())
- updated_at (TIMESTAMP, DEFAULT NOW())
- is_premium (BOOLEAN, DEFAULT FALSE)
- is_active (BOOLEAN, DEFAULT TRUE)

Indexes:
- PRIMARY KEY (problem_id)
- UNIQUE INDEX idx_slug (slug)
- INDEX idx_difficulty (difficulty)
- INDEX idx_created_at (created_at)
- INDEX idx_acceptance_rate (acceptance_rate)
```

#### Problem_Tags Table (Many-to-Many)
```sql
problem_tags
- problem_id (FK -> problems.problem_id)
- tag_id (FK -> tags.tag_id)
- created_at (TIMESTAMP, DEFAULT NOW())

Indexes:
- PRIMARY KEY (problem_id, tag_id)
- INDEX idx_tag_id (tag_id)
```

#### Tags Table
```sql
tags
- tag_id (PK, UUID)
- name (VARCHAR(50), UNIQUE, NOT NULL)
- slug (VARCHAR(50), UNIQUE, NOT NULL)
- description (TEXT)
- created_at (TIMESTAMP, DEFAULT NOW())

Indexes:
- PRIMARY KEY (tag_id)
- UNIQUE INDEX idx_name (name)
- UNIQUE INDEX idx_slug (slug)
```

#### Submissions Table
```sql
submissions
- submission_id (PK, UUID)
- user_id (FK -> users.user_id, NOT NULL)
- problem_id (FK -> problems.problem_id, NOT NULL)
- language (VARCHAR(20), NOT NULL)
- code_s3_key (VARCHAR(500), NOT NULL)
- status (ENUM: 'pending', 'running', 'accepted', 'wrong_answer', 
         'time_limit_exceeded', 'memory_limit_exceeded', 
         'runtime_error', 'compile_error', 'system_error')
- runtime_ms (INT)
- memory_kb (INT)
- test_cases_passed (INT)
- total_test_cases (INT)
- error_message (TEXT)
- submitted_at (TIMESTAMP, DEFAULT NOW())
- judged_at (TIMESTAMP)

Indexes:
- PRIMARY KEY (submission_id)
- INDEX idx_user_problem (user_id, problem_id, submitted_at DESC)
- INDEX idx_user_submitted (user_id, submitted_at DESC)
- INDEX idx_problem_submitted (problem_id, submitted_at DESC)
- INDEX idx_status (status)
```

#### Test_Cases Table
```sql
test_cases
- test_case_id (PK, UUID)
- problem_id (FK -> problems.problem_id, NOT NULL)
- input_s3_key (VARCHAR(500), NOT NULL)
- expected_output_s3_key (VARCHAR(500), NOT NULL)
- is_sample (BOOLEAN, DEFAULT FALSE)
- is_hidden (BOOLEAN, DEFAULT TRUE)
- order_index (INT, NOT NULL)
- time_limit_ms (INT, DEFAULT 3000)
- memory_limit_kb (INT, DEFAULT 262144)
- created_at (TIMESTAMP, DEFAULT NOW())

Indexes:
- PRIMARY KEY (test_case_id)
- INDEX idx_problem_order (problem_id, order_index)
- INDEX idx_problem_sample (problem_id, is_sample)
```

#### User_Problem_Status Table
```sql
user_problem_status
- user_id (FK -> users.user_id)
- problem_id (FK -> problems.problem_id)
- status (ENUM: 'attempted', 'solved')
- attempts (INT, DEFAULT 1)
- first_attempted_at (TIMESTAMP, DEFAULT NOW())
- solved_at (TIMESTAMP)
- best_runtime_ms (INT)
- best_memory_kb (INT)

Indexes:
- PRIMARY KEY (user_id, problem_id)
- INDEX idx_user_status (user_id, status)
- INDEX idx_solved_at (solved_at)
```

#### User_Statistics Table
```sql
user_statistics
- user_id (PK, FK -> users.user_id)
- problems_solved (INT, DEFAULT 0)
- easy_solved (INT, DEFAULT 0)
- medium_solved (INT, DEFAULT 0)
- hard_solved (INT, DEFAULT 0)
- total_submissions (INT, DEFAULT 0)
- acceptance_rate (DECIMAL(5,2))
- ranking (INT)
- last_updated (TIMESTAMP, DEFAULT NOW())

Indexes:
- PRIMARY KEY (user_id)
- INDEX idx_ranking (ranking)
- INDEX idx_problems_solved (problems_solved DESC)
```

#### Sessions Table (Could use Redis, but showing SQL structure)
```sql
sessions
- session_id (PK, UUID)
- user_id (FK -> users.user_id, NOT NULL)
- token_hash (VARCHAR(255), NOT NULL)
- ip_address (VARCHAR(45))
- user_agent (TEXT)
- created_at (TIMESTAMP, DEFAULT NOW())
- expires_at (TIMESTAMP, NOT NULL)
- last_accessed_at (TIMESTAMP, DEFAULT NOW())

Indexes:
- PRIMARY KEY (session_id)
- INDEX idx_user_id (user_id)
- INDEX idx_token_hash (token_hash)
- INDEX idx_expires_at (expires_at)
```

**Database Sharding Strategy:**
- Users DB: Shard by user_id (consistent hashing)
- Problems DB: Can remain unsharded initially (only 3K problems)
- Submissions DB: Shard by user_id (most queries are user-specific)
- Partition Submissions by date (time-series data)

---

### API Design

#### Base Configuration

**Base URL:** `https://api.leetcode.com/v1`

**Authentication:** 
- JWT (JSON Web Tokens) for stateless authentication
- Access token valid for 15 minutes
- Refresh token valid for 7 days
- Bearer token in Authorization header

**Versioning Strategy:** URL path versioning (`/v1/`, `/v2/`)

**Rate Limiting:**
- Anonymous: 20 requests/minute
- Authenticated users: 100 requests/minute
- Premium users: 500 requests/minute
- Code submissions: 10 submissions/minute per user

---

#### Authentication Endpoints

##### 1. Register User
```
POST /v1/auth/register
```

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "password": "SecureP@ss123",
  "full_name": "John Doe"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "user": {
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "email": "john@example.com",
      "full_name": "John Doe",
      "created_at": "2025-10-01T10:30:00Z"
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 900
    }
  }
}
```

**Response (400 Bad Request):**
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Username already exists",
    "field": "username"
  }
}
```

---

##### 2. Login
```
POST /v1/auth/login
```

**Request Body:**
```json
{
  "email": "john@example.com",
  "password": "SecureP@ss123"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "email": "john@example.com"
    },
    "tokens": {
      "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expires_in": 900
    }
  }
}
```

---

##### 3. Refresh Token
```
POST /v1/auth/refresh
```

**Request Body:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 900
  }
}
```

---

##### 4. Logout
```
POST /v1/auth/logout
```

**Request Headers:**
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

#### Problem Endpoints

##### 5. List Problems
```
GET /v1/problems
```

**Query Parameters:**
- `difficulty` (string, optional): `easy`, `medium`, `hard`
- `tags` (string, optional): Comma-separated tag slugs
- `status` (string, optional): `solved`, `attempted`, `todo` (requires auth)
- `search` (string, optional): Search by title
- `page` (integer, default: 1): Page number
- `limit` (integer, default: 20, max: 100): Items per page
- `sort` (string, default: `id`): `id`, `title`, `difficulty`, `acceptance_rate`
- `order` (string, default: `asc`): `asc`, `desc`

**Request Headers:**
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." 
}
```
*(Optional, for personalized status)*

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "problems": [
      {
        "problem_id": "550e8400-e29b-41d4-a716-446655440001",
        "title": "Two Sum",
        "slug": "two-sum",
        "difficulty": "easy",
        "acceptance_rate": 45.5,
        "total_accepted": 2500000,
        "total_submissions": 5500000,
        "tags": ["array", "hash-table"],
        "is_premium": false,
        "user_status": "solved"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 150,
      "total_items": 3000,
      "items_per_page": 20,
      "has_next": true,
      "has_previous": false
    }
  }
}
```

**Caching:** Cache-Control: `public, max-age=300` (5 minutes)

---

##### 6. Get Problem Details
```
GET /v1/problems/{slug}
```

**Path Parameters:**
- `slug` (string): Problem slug (e.g., "two-sum")

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "problem_id": "550e8400-e29b-41d4-a716-446655440001",
    "title": "Two Sum",
    "slug": "two-sum",
    "difficulty": "easy",
    "description": "Given an array of integers nums and an integer target...",
    "constraints": "- 2 <= nums.length <= 10^4\n- -10^9 <= nums[i] <= 10^9",
    "examples": [
      {
        "input": "nums = [2,7,11,15], target = 9",
        "output": "[0,1]",
        "explanation": "Because nums[0] + nums[1] == 9, we return [0, 1]."
      }
    ],
    "tags": ["array", "hash-table"],
    "acceptance_rate": 45.5,
    "total_accepted": 2500000,
    "total_submissions": 5500000,
    "is_premium": false,
    "sample_test_cases": [
      {
        "input": "[2,7,11,15]\n9",
        "expected_output": "[0,1]"
      }
    ],
    "code_template": {
      "python": "class Solution:\n    def twoSum(self, nums: List[int], target: int) -> List[int]:\n        ",
      "java": "class Solution {\n    public int[] twoSum(int[] nums, int target) {\n        \n    }\n}",
      "cpp": "class Solution {\npublic:\n    vector<int> twoSum(vector<int>& nums, int target) {\n        \n    }\n};",
      "javascript": "var twoSum = function(nums, target) {\n    \n};"
    }
  }
}
```

**Response (404 Not Found):**
```json
{
  "success": false,
  "error": {
    "code": "PROBLEM_NOT_FOUND",
    "message": "Problem not found"
  }
}
```

**Caching:** Cache-Control: `public, max-age=3600` (1 hour)

---

##### 7. Get Problem Statistics
```
GET /v1/problems/{slug}/statistics
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "total_submissions": 5500000,
    "total_accepted": 2500000,
    "acceptance_rate": 45.5,
    "submissions_by_language": {
      "python": 2200000,
      "java": 1800000,
      "cpp": 1100000,
      "javascript": 400000
    },
    "ac_rate_by_language": {
      "python": 48.2,
      "java": 44.5,
      "cpp": 42.1,
      "javascript": 46.8
    }
  }
}
```

---

#### Submission Endpoints

##### 8. Submit Code
```
POST /v1/submissions
```

**Request Headers:**
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "problem_id": "550e8400-e29b-41d4-a716-446655440001",
  "language": "python",
  "code": "class Solution:\n    def twoSum(self, nums: List[int], target: int) -> List[int]:\n        hashmap = {}\n        for i, num in enumerate(nums):\n            complement = target - num\n            if complement in hashmap:\n                return [hashmap[complement], i]\n            hashmap[num] = i"
}
```

**Response (202 Accepted):**
```json
{
  "success": true,
  "data": {
    "submission_id": "650e8400-e29b-41d4-a716-446655440000",
    "status": "pending",
    "submitted_at": "2025-10-01T10:35:00Z",
    "message": "Submission received and queued for judging"
  }
}
```

**Response (429 Too Many Requests):**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many submissions. Please wait before submitting again.",
    "retry_after": 30
  }
}
```

**Rate Limiting:** 10 submissions per minute per user

---

##### 9. Get Submission Status
```
GET /v1/submissions/{submission_id}
```

**Request Headers:**
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK) - Pending:**
```json
{
  "success": true,
  "data": {
    "submission_id": "650e8400-e29b-41d4-a716-446655440000",
    "problem_id": "550e8400-e29b-41d4-a716-446655440001",
    "problem_title": "Two Sum",
    "language": "python",
    "status": "running",
    "submitted_at": "2025-10-01T10:35:00Z"
  }
}
```

**Response (200 OK) - Completed:**
```json
{
  "success": true,
  "data": {
    "submission_id": "650e8400-e29b-41d4-a716-446655440000",
    "problem_id": "550e8400-e29b-41d4-a716-446655440001",
    "problem_title": "Two Sum",
    "language": "python",
    "status": "accepted",
    "runtime_ms": 45,
    "runtime_percentile": 85.2,
    "memory_kb": 14256,
    "memory_percentile": 72.3,
    "test_cases_passed": 57,
    "total_test_cases": 57,
    "submitted_at": "2025-10-01T10:35:00Z",
    "judged_at": "2025-10-01T10:35:03Z"
  }
}
```

**Response (200 OK) - Failed:**
```json
{
  "success": true,
  "data": {
    "submission_id": "650e8400-e29b-41d4-a716-446655440000",
    "status": "wrong_answer",
    "test_cases_passed": 52,
    "total_test_cases": 57,
    "failed_test_case": {
      "input": "[3,2,4]\n6",
      "expected_output": "[1,2]",
      "actual_output": "[0,2]"
    },
    "submitted_at": "2025-10-01T10:35:00Z",
    "judged_at": "2025-10-01T10:35:03Z"
  }
}
```

**Polling:** Client should poll every 1-2 seconds while status is `pending` or `running`

---

##### 10. Run Code (Test Against Sample Cases)
```
POST /v1/problems/{slug}/run
```

**Request Body:**
```json
{
  "language": "python",
  "code": "class Solution:\n    def twoSum(self, nums: List[int], target: int) -> List[int]:\n        return [0, 1]",
  "test_input": "[2,7,11,15]\n9"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "status": "success",
    "output": "[0, 1]",
    "expected_output": "[0, 1]",
    "runtime_ms": 42,
    "memory_kb": 14120,
    "stdout": "",
    "stderr": ""
  }
}
```

**Response (200 OK) - Runtime Error:**
```json
{
  "success": true,
  "data": {
    "status": "runtime_error",
    "error_message": "IndexError: list index out of range",
    "line_number": 3,
    "stdout": "",
    "stderr": "Traceback (most recent call last):\n  ..."
  }
}
```

**Rate Limiting:** 30 runs per minute per user

---

##### 11. Get User Submissions
```
GET /v1/users/{username}/submissions
```

**Query Parameters:**
- `problem_id` (string, optional): Filter by problem
- `status` (string, optional): Filter by status
- `page` (integer, default: 1)
- `limit` (integer, default: 20, max: 100)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "submissions": [
      {
        "submission_id": "650e8400-e29b-41d4-a716-446655440000",
        "problem_id": "550e8400-e29b-41d4-a716-446655440001",
        "problem_title": "Two Sum",
        "problem_slug": "two-sum",
        "language": "python",
        "status": "accepted",
        "runtime_ms": 45,
        "memory_kb": 14256,
        "submitted_at": "2025-10-01T10:35:00Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 25,
      "total_items": 500,
      "items_per_page": 20
    }
  }
}
```

---

##### 12. Get Submission Code
```
GET /v1/submissions/{submission_id}/code
```

**Request Headers:**
```json
{
  "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "submission_id": "650e8400-e29b-41d4-a716-446655440000",
    "language": "python",
    "code": "class Solution:\n    def twoSum(self, nums: List[int], target: int) -> List[int]:\n        hashmap = {}\n        for i, num in enumerate(nums):\n            complement = target - num\n            if complement in hashmap:\n                return [hashmap[complement], i]\n            hashmap[num] = i"
  }
}
```

---

#### User Profile Endpoints

##### 13. Get User Profile
```
GET /v1/users/{username}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user_id": "550e8400-e29b-41d4-a716-446655440000",
    "username": "johndoe",
    "full_name": "John Doe",
    "avatar_url": "https://cdn.leetcode.com/avatars/johndoe.jpg",
    "created_at": "2024-01-15T08:30:00Z",
    "ranking": 12567,
    "statistics": {
      "problems_solved": 523,
      "easy_solved": 245,
      "medium_solved": 213,
      "hard_solved": 65,
      "total_submissions": 1247,
      "acceptance_rate": 41.9
    },
    "recent_submissions": [
      {
        "problem_title": "Two Sum",
        "problem_slug": "two-sum",
        "status": "accepted",
        "submitted_at": "2025-10-01T10:35:00Z"
      }
    ]
  }
}
```

---

##### 14. Get User Statistics
```
GET /v1/users/{username}/statistics
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "problems_solved": 523,
    "easy_solved": 245,
    "medium_solved": 213,
    "hard_solved": 65,
    "total_submissions": 1247,
    "acceptance_rate": 41.9,
    "ranking": 12567,
    "submission_heatmap": {
      "2025-09-30": 5,
      "2025-10-01": 3
    },
    "languages_used": {
      "python": 450,
      "java": 320,
      "cpp": 200,
      "javascript": 80
    }
  }
}
```

---

#### Tag Endpoints

##### 15. List Tags
```
GET /v1/tags
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "tags": [
      {
        "tag_id": "750e8400-e29b-41d4-a716-446655440000",
        "name": "Array",
        "slug": "array",
        "problem_count": 450
      },
      {
        "tag_id": "750e8400-e29b-41d4-a716-446655440001",
        "name": "Hash Table",
        "slug": "hash-table",
        "problem_count": 320
      }
    ]
  }
}
```

**Caching:** Cache-Control: `public, max-age=86400` (24 hours)

---

#### Search Endpoint

##### 16. Search Problems
```
GET /v1/search
```

**Query Parameters:**
- `q` (string, required): Search query
- `type` (string, default: `all`): `problems`, `users`, `all`
- `page` (integer, default: 1)
- `limit` (integer, default: 20, max: 50)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "problems": [
      {
        "problem_id": "550e8400-e29b-41d4-a716-446655440001",
        "title": "Two Sum",
        "slug": "two-sum",
        "difficulty": "easy",
        "acceptance_rate": 45.5
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 5,
      "total_items": 95
    }
  }
}
```

---

### Cross-Cutting API Concerns

**Standard Error Response Format:**
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "field": "field_name",
    "details": {}
  }
}
```

**Common HTTP Status Codes:**
- `200 OK` - Successful GET, PUT, PATCH
- `201 Created` - Successful POST (resource created)
- `202 Accepted` - Async operation accepted
- `204 No Content` - Successful DELETE
- `400 Bad Request` - Validation error
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Service temporarily unavailable

**Pagination Strategy:**
- Offset-based pagination for problems (supports random access)
- Cursor-based pagination for submissions (time-series data)
- Default limit: 20, max limit: 100

**Idempotency:**
- Use idempotency keys for submission endpoints
- Header: `Idempotency-Key: {unique_key}`
- Prevents duplicate submissions

**Compression:**
- Support gzip and brotli compression
- Request header: `Accept-Encoding: gzip, br`
- Response header: `Content-Encoding: gzip`

**Security Headers:**
```
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

**CORS Policy:**
- Allow origins: `*.leetcode.com`
- Allow methods: `GET, POST, PUT, PATCH, DELETE, OPTIONS`
- Allow headers: `Authorization, Content-Type, Idempotency-Key`
- Max age: 3600 seconds

---

### API Trade-Offs

**Decision:** REST vs GraphQL
**Choice:** REST API
**Pros:**
- Simpler to implement and maintain
- Better caching support (HTTP caching)
- Standardized and well-understood
- Better tooling and monitoring
**Cons:**
- Over-fetching data (getting more than needed)
- Under-fetching (multiple requests needed)
- Less flexible for clients
**Justification:** For a LeetCode-style platform, REST is sufficient. The data models are well-defined, and endpoints have predictable access patterns. The slight inefficiency from over/under-fetching is acceptable given the simplicity benefits.

---

**Decision:** Synchronous vs Asynchronous Code Execution
**Choice:** Asynchronous (queue-based)
**Pros:**
- Decouples submission from execution
- Better resource utilization
- Can handle traffic spikes
- Easy to scale workers independently
- Retry failed executions
**Cons:**
- Added complexity (queue management)
- Requires polling or WebSocket for real-time updates
- Slightly higher latency (queuing delay)
**Justification:** Code execution can take several seconds. Synchronous execution would tie up API server threads and create poor user experience. Async execution allows the API to remain responsive while workers handle the heavy processing.

---

**Decision:** Offset vs Cursor Pagination
**Choice:** Offset for problems, Cursor for submissions
**Pros (Offset):**
- Allows random page access
- Simple to implement
- Good for small, stable datasets
**Pros (Cursor):**
- Consistent results for real-time data
- Better performance for large datasets
- No missing/duplicate items during pagination
**Cons:** Mixed approach adds complexity
**Justification:** Problems are relatively static (3K items), so offset pagination is fine. Submissions are time-series data with constant inserts, so cursor pagination prevents inconsistencies during pagination.

---

**Decision:** Endpoint Granularity
**Choice:** Separate endpoints for different resources
**Pros:**
- Clear separation of concerns
- Easier to version and evolve
- Better caching strategies per resource
- Follows REST principles
**Cons:**
- More network requests for related data
- Potential for over-fetching
**Justification:** Aligns with REST best practices and allows independent scaling of different services.

---

### Component Deep-Dive

#### 1. Judge Worker System (Most Critical Component)

**Purpose:**
Execute user-submitted code in a secure, isolated environment and evaluate it against test cases.

**Internal Architecture:**

```
Judge Worker Container
├── Job Listener (pulls from queue)
├── Language Runtime Manager
│   ├── Python Runtime
│   ├── Java Runtime
│   ├── C++ Runtime
│   └── JavaScript Runtime
├── Sandbox Manager (Docker-in-Docker or gVisor)
├── Resource Monitor (CPU, Memory, Time)
├── Test Case Runner
└── Result Reporter
```

**Execution Flow:**
1. **Job Reception:** Worker pulls submission job from RabbitMQ/Kafka queue
2. **Code Retrieval:** Fetches user code from S3
3. **Test Case Loading:** Retrieves all test cases for the problem from S3
4. **Container Preparation:** Creates isolated Docker container with:
   - Specific language runtime
   - CPU limit: 1 core
   - Memory limit: 256 MB
   - Network disabled
   - Read-only filesystem (except /tmp)
   - No privileged operations
5. **Code Compilation** (for compiled languages):
   - Compile with timeout (30 seconds)
   - Capture compilation errors
6. **Test Execution:**
   - Run code against each test case sequentially
   - Set timeout per test case (default 3 seconds)
   - Monitor memory usage
   - Capture stdout/stderr
   - Compare output with expected output
7. **Result Aggregation:**
   - Count passed/failed test cases
   - Record runtime and memory statistics
   - Identify first failed test case
8. **Cleanup:** Destroy container
9. **Result Update:** Update submission status in database

**Technology Choices:**
- **Containerization:** Docker for isolation
- **Security:** gVisor or Kata Containers for additional sandboxing
- **Resource Limiting:** cgroups for CPU/memory limits
- **Timeout Handling:** Process monitoring with SIGKILL

**Scaling Strategy:**
- Horizontal scaling: Add more worker instances
- Auto-scaling based on queue depth
- Separate worker pools per language (optional optimization)
- Kubernetes for orchestration

**Failure Handling:**
- Worker crash: Job returns to queue (using message acknowledgment)
- Timeout: Kill container, mark as TLE
- Out of memory: Mark as MLE
- Malicious code: Sandboxing prevents system compromise
- Network failures: Retry mechanism with exponential backoff

**Security Measures:**
1. **Sandbox Isolation:**
   - No network access
   - Limited system calls (seccomp filters)
   - No file system writes outside /tmp
   - Process isolation
2. **Resource Limits:**
   - CPU time limit
   - Memory limit
   - File size limit
   - Process count limit
3. **Code Analysis:**
   - Static analysis for dangerous patterns (optional)
   - Blacklist dangerous libraries/imports
4. **Execution Environment:**
   - Read-only container filesystem
   - Minimal base image
   - Non-root user execution

---

#### 2. Caching Strategy

**Redis Cache Architecture:**

```
Redis Cluster
├── Problem Cache (Hot problems)
├── User Session Cache
├── Rate Limit Counter Cache
└── Leaderboard Cache
```

**What to Cache:**

1. **Problem Data** (Cache-aside pattern)
   - **Key:** `problem:{slug}` or `problem:{id}`
   - **Value:** Full problem details JSON
   - **TTL:** 1 hour
   - **Reason:** Problems rarely change, high read frequency
   - **Invalidation:** On problem update/delete

2. **Problem List with Filters** (Cache-aside)
   - **Key:** `problems:list:{difficulty}:{tags}:{page}`
   - **Value:** Paginated problem list
   - **TTL:** 5 minutes
   - **Reason:** Reduce DB load for common filters
   - **Invalidation:** On problem count change

3. **User Session Data** (Write-through)
   - **Key:** `session:{token_hash}`
   - **Value:** User ID, permissions, expiry
   - **TTL:** 15 minutes (access token lifetime)
   - **Reason:** Avoid DB lookup on every authenticated request

4. **User Statistics** (Cache-aside with lazy update)
   - **Key:** `user:stats:{user_id}`
   - **Value:** Problems solved, ranking, acceptance rate
   - **TTL:** 10 minutes
   - **Reason:** Frequently viewed, calculation intensive
   - **Invalidation:** On successful submission

5. **Submission Status** (Write-through)
   - **Key:** `submission:{submission_id}`
   - **Value:** Current status, results
   - **TTL:** 1 hour (or until completion)
   - **Reason:** Reduce polling load on database
   - **Invalidation:** On status update

6. **Rate Limit Counters** (Write-through)
   - **Key:** `ratelimit:{user_id}:{endpoint}:{window}`
   - **Value:** Request count
   - **TTL:** 1 minute (window size)
   - **Reason:** Fast rate limiting decisions

7. **Test Cases** (Worker-level cache)
   - **Location:** Worker local memory
   - **Value:** Test case input/output for each problem
   - **TTL:** Until worker restart
   - **Reason:** Avoid repeated S3 fetches

**What NOT to Cache:**

1. **Submission Code:** Too large, infrequently accessed (use S3 directly)
2. **Authentication Credentials:** Security risk
3. **Financial/Payment Data:** Compliance requirements
4. **Rarely Accessed Problems:** Not worth memory

**Cache Invalidation Strategy:**

- **Problem Updates:** Explicit invalidation via API
- **User Stats:** Lazy invalidation (update on next calculation)
- **Time-based:** TTL for most data
- **Event-driven:** Publish invalidation events via message queue

**Cache Eviction Policy:**
- **Algorithm:** LRU (Least Recently Used)
- **Memory Limit:** 80% threshold triggers eviction

---

### Trade-Offs Analysis

**Decision:** SQL vs NoSQL for Submissions
**Choice:** PostgreSQL (SQL)
**Pros:**
- ACID transactions for data consistency
- Complex queries (user stats, rankings)
- Strong data integrity
- Mature ecosystem
**Cons:**
- Harder to scale horizontally
- Fixed schema
- Write throughput limitations
**Justification:** Submissions require transactional consistency (updating user stats atomically). Complex analytical queries for leaderboards and statistics favor SQL. Can scale via sharding by user_id.

---

**Decision:** Message Queue vs Direct Worker Communication
**Choice:** RabbitMQ/Kafka Message Queue
**Pros:**
- Decouples submission service from workers
- Load balancing across workers
- Retry and dead-letter handling
- Persistent queue (durability)
- Easy to scale workers independently
**Cons:**
- Added latency (queuing overhead)
- Additional infrastructure complexity
- Potential queue bottleneck
**Justification:** Essential for async execution pattern. Provides resilience and scalability. The minimal latency cost (<100ms) is acceptable for the reliability benefits.

---

**Decision:** S3 vs Database for Code Storage
**Choice:** S3 Object Storage
**Pros:**
- Cost-effective for large blobs
- Unlimited scalability
- Built-in redundancy
- Offloads DB storage pressure
**Cons:**
- Additional network call
- Slight latency increase
- Eventual consistency (S3)
**Justification:** Code files are large and rarely updated. Storing in DB would bloat tables and increase costs. S3 is purpose-built for this use case.

---

**Decision:** WebSocket vs Polling for Submission Status
**Choice:** Polling (with potential WebSocket upgrade path)
**Pros (Polling):**
- Simpler to implement
- No persistent connections
- Works with standard HTTP caching
- Lower server resource usage for low-traffic scenarios
**Cons (Polling):**
- Higher latency (1-2 second polling interval)
- Unnecessary requests if status unchanged
- More bandwidth usage
**Justification:** For MVP, polling is simpler and sufficient. Most submissions complete in 3-5 seconds, so 1-2 polling requests are acceptable. Can add WebSocket later for premium users or high-traffic scenarios.

---

**Decision:** Microservices vs Monolith
**Choice:** Microservices Architecture
**Pros:**
- Independent scaling (scale judge workers separately)
- Technology diversity (different languages per service)
- Fault isolation
- Team autonomy
**Cons:**
- Increased complexity
- Network overhead
- Distributed tracing needed
- More operational overhead
**Justification:** Judge system has very different scaling needs than API layer. Problem service is read-heavy, submission service is write-heavy. Microservices allow optimizing each independently.

---

**Decision:** Synchronous vs Asynchronous User Statistics Update
**Choice:** Asynchronous (Event-Driven)
**Pros:**
- Faster submission response
- Decouples statistics calculation
- Can handle calculation failures independently
- Easier to add new statistics
**Cons:**
- Eventual consistency (stats slightly delayed)
- More complex data flow
**Justification:** User stats are not critical path for submission. Acceptable to have 1-2 second delay in stat updates. Improves submission latency.

---

## RULE 5: BOTTLENECKS & IMPROVEMENTS

### Potential Bottlenecks

#### 1. Judge Worker Capacity

**Problem:**
During peak hours or contests, submission queue can grow faster than workers can process, leading to long wait times.

**Solution:**
- **Auto-scaling:** Scale worker pool based on queue depth
  - Target: Queue depth < 100 jobs
  - Scale up when queue > 200 for 2 minutes
  - Scale down when queue < 50 for 10 minutes
- **Priority Queues:** Separate queues for different user tiers
  - Premium users: High priority queue
  - Regular users: Normal priority queue
- **Worker Pool Optimization:** Pre-warm containers to reduce startup time
- **Resource Allocation:** Increase worker pool size during known peak times

**Monitoring:**
- Metric: Queue depth, average wait time, worker utilization
- Alert: Wait time > 10 seconds
- Dashboard: Real-time queue visualization

---

#### 2. Database Write Contention (Submissions Table)

**Problem:**
High submission rate (36 QPS peak) can cause write contention on submissions table, especially with index updates.

**Solution:**
- **Database Sharding:** Shard submissions by user_id
  - Each shard handles subset of users
  - Consistent hashing for shard selection
- **Write-Optimized Indexes:** Minimize indexes on submission table
  - Only essential indexes (user_id, problem_id)
- **Batch Updates:** Batch statistical updates (user stats, problem stats)
- **Read Replicas:** Offload submission history queries to read replicas

**Monitoring:**
- Metric: Write latency (p95, p99), lock wait time
- Alert: p95 write latency > 100ms
- Dashboard: Database throughput and latency graphs

---

#### 3. Cache Stampede on Popular Problems

**Problem:**
When cache expires for popular problems, multiple requests simultaneously hit the database, causing load spike.

**Solution:**
- **Cache Warming:** Proactively refresh cache before expiry
- **Probabilistic Early Expiration:** Randomly refresh cache slightly before TTL
- **Request Coalescing:** Deduplicate simultaneous requests for same resource
- **Longer TTL for Popular Items:** Adaptive TTL based on access frequency

**Monitoring:**
- Metric: Cache hit rate, database query rate
- Alert: Cache hit rate < 85%
- Dashboard: Cache performance metrics

---

#### 4. S3 Bandwidth Limitations

**Problem:**
Workers fetching code and test cases from S3 can hit bandwidth limits during high traffic.

**Solution:**
- **Local Caching:** Cache test cases on worker nodes
- **CloudFront CDN:** Cache test cases in CDN (they don't change)
- **Batching:** Fetch multiple test cases in single request
- **S3 Transfer Acceleration:** Enable for faster uploads/downloads

**Monitoring:**
- Metric: S3 request latency, bandwidth usage
- Alert: S3 latency p95 > 500ms

---

#### 5. Single Point of Failure - Message Queue

**Problem:**
If message queue goes down, no submissions can be processed.

**Solution:**
- **Queue Clustering:** RabbitMQ cluster with multiple nodes
- **Replication:** Kafka with replication factor 3
- **Fallback Mechanism:** Temporary database-backed queue
- **Health Checks:** Continuous monitoring with automatic failover

**Monitoring:**
- Metric: Queue health, message throughput, consumer lag
- Alert: Queue unavailable
- Dashboard: Queue cluster health

---

### Scalability Improvements

#### 1. Geographic Distribution

**Current State:** Single region deployment

**Improvement:**
- **Multi-Region Deployment:**
  - US East, US West, Europe, Asia data centers
  - Route users to nearest region (latency-based routing)
  - Replicate problem data globally (read replicas)
  - Shard user data by region
  
- **Benefits:**
  - Lower latency for global users
  - Better availability (region failure tolerance)
  - Compliance with data residency laws

- **Challenges:**
  - Cross-region data consistency
  - Increased operational complexity
  - Higher infrastructure costs

---

#### 2. CDN for Static Assets

**Current State:** API servers serve all content

**Improvement:**
- **CDN Integration:**
  - CloudFront/Cloudflare for static assets
  - Cache problem descriptions, images, code templates
  - Cache user avatars
  - Edge caching for API responses (GET endpoints)

- **Benefits:**
  - Reduced server load
  - Faster page loads
  - Lower bandwidth costs

---

#### 3. Database Query Optimization

**Improvements:**
- **Materialized Views:** Pre-compute leaderboards, popular problems
- **Denormalization:** Store frequently accessed data redundantly
  - Problem title in submissions table (avoid JOIN)
- **Connection Pooling:** PgBouncer for connection management
- **Query Result Caching:** Cache expensive analytical queries

---

#### 4. Real-Time Features with WebSocket

**Current State:** Polling for submission status

**Improvement:**
- **WebSocket Connection:**
  - Establish WebSocket after submission
  - Push status updates in real-time
  - Close connection after result received

- **Benefits:**
  - Instant updates (no polling delay)
  - Reduced server load (no repeated polling requests)
  - Better user experience

- **Implementation:**
  - Socket.IO or native WebSockets
  - Redis Pub/Sub for broadcasting updates
  - Sticky sessions or shared state for WebSocket servers

---

#### 5. Advanced Caching with Redis Cluster

**Current State:** Single Redis instance

**Improvement:**
- **Redis Cluster:**
  - Multiple Redis nodes for horizontal scaling
  - Data sharding across nodes
  - Replication for high availability

- **Benefits:**
  - Higher throughput
  - More memory capacity
  - Fault tolerance

---

### Monitoring and Observability

#### Metrics to Track

**System Metrics:**
- **Latency:**
  - API response time (p50, p95, p99)
  - Database query time
  - Code execution time
  - Queue wait time
- **Throughput:**
  - Requests per second
  - Submissions per second
  - Queue processing rate
- **Error Rates:**
  - HTTP 5xx errors
  - Timeout errors
  - Judge system errors

**Business Metrics:**
- **User Engagement:**
  - Daily active users
  - Submissions per user
  - Average session duration
- **Platform Health:**
  - Submission success rate
  - Problem completion rate
  - Average time to solve

**Infrastructure Metrics:**
- **Compute:**
  - CPU utilization
  - Memory usage
  - Container count
- **Network:**
  - Bandwidth usage
  - Network latency
- **Storage:**
  - Disk usage
  - S3 storage size
  - Database size

#### Alerting Strategy

**Critical Alerts (Page immediately):**
- API availability < 99.9%
- Database connection failures
- Message queue unavailable
- Judge workers all down

**Warning Alerts (Notify, don't page):**
- API latency p95 > 500ms
- Queue depth > 500
- Cache hit rate < 80%
- Error rate > 2%

**Info Alerts (Log only):**
- Auto-scaling events
- Cache expiration
- Routine maintenance

**Alert Conditions:**
```
Alert: High API Latency
Condition: p95 latency > 500ms for 5 minutes
Severity: Warning
Action: Check database, cache, and downstream services

Alert: Judge Queue Backlog
Condition: Queue depth > 1000 for 10 minutes
Severity: Critical
Action: Scale up workers immediately

Alert: Database Write Latency
Condition: Write p99 > 200ms for 5 minutes
Severity: Warning
Action: Check for slow queries, lock contention
```

---

### Security Considerations

#### 1. Code Execution Security

**Measures:**
- **Sandboxing:** Docker containers with gVisor/Kata
- **Resource Limits:** CPU, memory, disk, network
- **Network Isolation:** No outbound connections
- **System Call Filtering:** Seccomp profiles
- **User Namespace:** Non-root execution
- **Read-Only Filesystem:** Except /tmp

**Threats Mitigated:**
- Malicious code execution
- Resource exhaustion attacks
- Data exfiltration
- System compromise

---

#### 2. API Security

**Measures:**
- **Authentication:** JWT tokens with short expiry
- **Authorization:** Role-based access control (RBAC)
- **Rate Limiting:** Per-user, per-IP, per-endpoint
- **Input Validation:** Sanitize all user inputs
- **SQL Injection Prevention:** Parameterized queries
- **XSS Prevention:** Content Security Policy headers
- **CSRF Protection:** CSRF tokens for state-changing operations

---

#### 3. Data Security

**Measures:**
- **Encryption at Rest:**
  - Database encryption (AWS RDS encryption)
  - S3 server-side encryption
- **Encryption in Transit:**
  - TLS 1.3 for all connections
  - HTTPS only (HSTS headers)
- **Password Security:**
  - bcrypt hashing with salt
  - Minimum password complexity
  - Password breach detection (HaveIBeenPwned API)
- **Secret Management:**
  - AWS Secrets Manager / HashiCorp Vault
  - Rotate secrets regularly
  - Never log secrets

---

#### 4. DDoS Protection

**Measures:**
- **Rate Limiting:** Multiple layers
  - WAF (CloudFlare/AWS WAF)
  - API Gateway rate limiting
  - Application-level rate limiting
- **Traffic Analysis:** Detect abnormal patterns
- **Geo-Blocking:** Block suspicious regions
- **Challenge Responses:** CAPTCHA for suspicious activity

---

#### 5. Secure Sandbox Isolation

**Techniques:**
- **Container Technology:** Docker with seccomp/AppArmor
- **Hypervisor-Based:** gVisor for syscall filtering
- **Resource Quotas:** Strict CPU/memory/disk limits
- **Network Policies:** Deny all outbound traffic
- **Process Isolation:** PID namespace isolation
- **File System Controls:** Mount code as read-only

---

### Future Enhancements

#### Phase 2 Features

1. **Coding Contests:**
   - Timed competitions
   - Real-time leaderboards
   - Contest ratings (ELO system)
   - Virtual contests (replay past contests)

2. **Discussion Forums:**
   - Problem-specific discussions
   - Solution sharing
   - Upvoting/downvoting
   - Moderator tools

3. **Video Explanations:**
   - Integrated video player
   - Instructor-led solutions
   - User-generated content

4. **Interview Preparation Mode:**
   - Mock interviews
  - Timed problem sets
   - Company-specific problem lists

5. **Code Review System:**
   - Peer code review
   - Best solution showcase
   - Code quality metrics

#### Advanced Technical Features

1. **AI-Powered Hints:**
   - Progressive hints based on user struggle
   - ML model trained on solution patterns
   - Natural language hint generation

2. **Code Similarity Detection:**
   - Detect plagiarism
   - Find similar solutions
   - Learning pattern analysis

3. **Performance Optimization Suggestions:**
   - Identify inefficient code patterns
   - Suggest time/space complexity improvements
   - Automated code review

4. **Multi-Language Code Translation:**
   - Convert solution from one language to another
   - Help users learn new languages

5. **Collaborative Coding:**
   - Real-time pair programming
   - Shared code editor (CRDT-based)
   - Voice/video integration

#### Platform Improvements

1. **Mobile Apps:**
   - Native iOS/Android apps
   - Offline problem reading
   - Push notifications for submissions

2. **IDE Integration:**
   - VS Code extension
   - IntelliJ plugin
   - Submit from local IDE

3. **Advanced Analytics:**
   - Personalized learning paths
   - Skill gap analysis
   - Progress tracking dashboards

4. **Gamification:**
   - Achievement badges
   - Streak tracking
   - Daily challenges
   - Referral rewards

5. **Premium Features:**
   - Priority judge queue
   - Advanced analytics
   - Interview coaching
   - Company-specific prep

---

## CONCLUSION

This LeetCode-style system design demonstrates a scalable, secure, and performant platform for online coding practice. Key architectural decisions include:

- **Microservices architecture** for independent scaling
- **Asynchronous job processing** for code execution
- **Multi-layered caching** for performance
- **Secure sandboxing** for code execution safety
- **Horizontal scalability** at every layer

The system can handle **100,000 DAU**, **500,000 submissions/day**, and scale to millions of users with proper sharding and geographic distribution.

---

**Interview Tips:**
- Emphasize security in code execution sandbox
- Discuss trade-offs between sync/async execution
- Explain caching strategy for different data types
- Show understanding of database scaling challenges
- Demonstrate monitoring and observability thinking
- Be ready to dive deeper into any component

---

**Document Metadata:**
- **Created:** October 1, 2025
- **System:** LeetCode Clone
- **Scale:** 100K DAU, 500K submissions/day
- **Focus Areas:** Security, Scalability, Performance

---

