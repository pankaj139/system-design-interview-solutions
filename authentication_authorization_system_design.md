# Authentication & Authorization System Design

**Difficulty Level:** ⭐⭐⭐⭐ Very Hard  
**Tags:** `Security`, `Identity Management`, `OAuth 2.0`, `JWT`, `SSO`, `Distributed Systems`, `Encryption`, `Access Control`, `Multi-Factor Authentication`, `SAML`

**File Purpose:** Interactive, multi-level learning resource for designing an enterprise-grade Authentication and Authorization system. This instructional guide takes you from beginner concepts to advanced production considerations, teaching you how to build a system that handles 10M+ users, processes 100K authentication requests per second, supports multiple protocols (OAuth 2.0, SAML, OIDC), achieves 99.99% availability, and maintains <50ms token validation latency.

**Author:** System Design Documentation  
**Created:** January 22, 2026  
**Last Updated:** January 22, 2026  
**Recent Updates:** Initial creation with comprehensive authentication and authorization patterns

---

## 🎓 Welcome to Authentication & Authorization System Design!

### What You're Going to Build

Imagine creating the security backbone for a major enterprise like Okta, Auth0, or AWS Cognito - a service that securely verifies who users are (authentication) and what they're allowed to do (authorization). 

By the end of this learning journey, you'll understand how to design a production-grade authentication and authorization system that:
- Handles millions of users across multiple applications
- Supports modern protocols like OAuth 2.0, SAML, and OpenID Connect
- Validates tokens in under 50 milliseconds
- Implements multi-factor authentication (MFA) for enhanced security
- Provides Single Sign-On (SSO) across multiple applications
- Scales globally across multiple data centers
- Stays available 99.99% of the time (that's only 52 minutes of downtime per year!)
- Protects against common security threats (brute force, token theft, session hijacking)

### 📚 Your Learning Path

This course is designed for three different learning levels. You can progress through all levels or focus on the one that matches your current needs:

```text
🟢 BEGINNER LEVEL (6-8 hours)
├─ Learn fundamental security concepts
├─ Understand authentication vs authorization
├─ Build intuition with everyday analogies
└─ Perfect for: New to security and system design

🟡 INTERMEDIATE LEVEL (8-10 hours)  
├─ Master OAuth 2.0, JWT, and SAML protocols
├─ Learn authorization models (RBAC, ABAC)
├─ Practice common interview questions
└─ Perfect for: Preparing for FAANG interviews

🔴 ADVANCED LEVEL (10-15 hours)
├─ Production security considerations
├─ Token management at scale
├─ Handle edge cases and security threats
└─ Perfect for: Senior engineers and security architects
```

### 🎯 Prerequisites

**For Beginners:**
- Basic understanding of HTTP and web requests
- Familiarity with usernames and passwords
- Understanding of what an API is
- No prior security experience needed!

**For Intermediate:**
- Comfortable with REST APIs and HTTP headers
- Understanding of cryptography basics (hashing, encryption)
- Familiarity with tokens and sessions
- Basic knowledge of distributed systems

**For Advanced:**
- Experience building secure systems
- Knowledge of cryptographic protocols
- Understanding of distributed systems and consistency models
- Familiarity with compliance standards (GDPR, SOC 2, ISO 27001)

### 📊 What Makes This Learning Experience Unique

Each section follows a proven learning pattern:
1. **What You'll Learn** - Clear learning objectives
2. **Why This Matters** - Real-world security context
3. **Multi-Level Content** - Tailored explanations for your level
4. **Real-World Examples** - How Auth0, Okta, and AWS Cognito actually do it
5. **Security Considerations** - Common vulnerabilities and how to prevent them
6. **Think About It** - Questions to deepen understanding
7. **Key Takeaways** - Summary of main points
8. **Practice Exercise** - Hands-on security challenges

💡 **Pro Tip:** Security is critical! Don't skip the "Security Considerations" sections - they cover vulnerabilities that can lead to major breaches!

---

## TABLE OF CONTENTS

- [Section 1: Understanding What We're Building](#section-1-understanding-what-were-building)
- [Section 2: Planning for Scale](#section-2-planning-for-scale)
- [Section 3: Designing the System Architecture](#section-3-designing-the-system-architecture)
- [Section 4: Authentication Protocols Deep Dive](#section-4-authentication-protocols-deep-dive)
- [Section 5: Authorization Models](#section-5-authorization-models)
- [Section 6: How Users Interact (API Design)](#section-6-how-users-interact-api-design)
- [Section 7: Storing Our Data](#section-7-storing-our-data)
- [Section 8: Token Management & Session Handling](#section-8-token-management--session-handling)
- [Section 9: Multi-Factor Authentication (MFA)](#section-9-multi-factor-authentication-mfa)
- [Section 10: Single Sign-On & Identity Federation](#section-10-single-sign-on--identity-federation)
- [Section 11: Growing the System (Scalability)](#section-11-growing-the-system-scalability)
- [Section 12: Protecting the System (Security)](#section-12-protecting-the-system-security)
- [Section 13: Keeping It Healthy (Monitoring)](#section-13-keeping-it-healthy-monitoring)
- [Section 14: Making Design Decisions](#section-14-making-design-decisions)
- [Putting It All Together](#putting-it-all-together)
- [Next Steps](#next-steps)

---

## Section 1: Understanding What We're Building

### What You'll Learn

By the end of this section, you'll be able to:
- Explain the difference between authentication (who you are) and authorization (what you can do)
- Define functional requirements for an authentication system
- Identify non-functional requirements (security, performance, availability)
- Understand common authentication flows (password-based, OAuth 2.0, SAML)
- Ask the right clarifying questions in a security system design interview
- Recognize threat models and security vulnerabilities

### Why This Matters

Authentication and authorization are the foundation of every secure application. Get it wrong, and you risk data breaches, account takeovers, and regulatory fines. Real-world example: Okta processes over 15,000 authentication requests per second for companies like FedEx, Slack, and T-Mobile. Understanding these fundamentals shapes how you design secure, scalable identity systems.

---

### 🟢 For Beginners: The Fundamentals

#### What is Authentication vs Authorization?

Think of going to a members-only club:

**Authentication (Who are you?):**
- You show your ID at the door
- The bouncer verifies "This is really John Smith"
- This proves your identity

**Authorization (What can you do?):**
- Your membership card shows you're a "VIP member"
- The bouncer checks what areas you can access
- VIP members get rooftop access, regular members don't
- This determines your permissions

**Real-World Example:**
```text
Logging into Facebook:
├─ Authentication: You enter email + password → Facebook verifies it's you
└─ Authorization: Facebook checks what you can do:
    ├─ Can post to your timeline? ✅ Yes
    ├─ Can delete someone else's post? ❌ No
    └─ Can access admin panel? ❌ No (unless you're a Facebook employee)
```

**Key Difference:**
- **Authentication** = Proving who you are (like showing your driver's license)
- **Authorization** = Checking what you're allowed to do (like having a backstage pass)

#### Why Do We Need a Dedicated Authentication System?

Let's explore the problems they solve:

1. **Security at Scale**:
   - Imagine if every app (Gmail, YouTube, Google Drive) had its own login
   - You'd have 50+ passwords to remember
   - Each app would need to implement security from scratch
   - One vulnerability would only affect that app

   **Better approach:** Central authentication system (like Google Account)
   - One secure login for all apps
   - Security experts focus on one system
   - Fix a vulnerability once, all apps benefit

2. **User Experience**:
   - ❌ Bad: Login to Gmail, login to Google Drive, login to YouTube
   - ✅ Good: Login once, access everything (Single Sign-On)

3. **Compliance & Auditing**:
   - Regulations require tracking who accessed what and when
   - Centralized logging makes audits easier
   - Example: HIPAA requires detailed access logs for healthcare data

4. **Advanced Security Features**:
   - Multi-Factor Authentication (MFA)
   - Biometric authentication
   - Adaptive authentication (detecting suspicious logins)
   - Rate limiting and brute force protection

#### Core Features: What Should It Do?

Let's think about what users and applications need:

**Core Features (MVP - Minimum Viable Product):**

1. **User Registration**
   - User provides: email, password, name
   - System: Creates account, sends verification email
   - Security: Hash password (never store plaintext!)
   - Time: <500ms

2. **User Login (Authentication)**
   - User provides: email + password
   - System verifies credentials
   - System returns: Access token (like a temporary pass)
   - Time: <200ms
   - Security: Lock account after 5 failed attempts

3. **Token Validation**
   - Application asks: "Is this token valid?"
   - System checks: Token exists, not expired, not revoked
   - Time: <50ms (happens on every API call!)
   - This is the most frequent operation

4. **Password Management**
   - Reset forgotten password via email
   - Change password when logged in
   - Password strength requirements
   - Password history (can't reuse last 5 passwords)

5. **Logout**
   - Invalidate user's token
   - Clear session data
   - Immediate effect across all devices

**Nice-to-Have Features (Future):**

- Multi-Factor Authentication (SMS, authenticator apps)
- Social login (Sign in with Google, Facebook)
- Single Sign-On (SSO) across multiple apps
- Biometric authentication (fingerprint, face ID)
- Session management (see all logged-in devices)
- Risk-based authentication (detect suspicious logins)

💡 **Pro Tip:** In security interviews, always mention encryption, hashing, and secure storage. Never say "store the password" - always say "hash the password with bcrypt/Argon2!"

#### Common Authentication Methods

Let's understand different ways users can prove their identity:

**1. Password-Based (Most Common)**
```text
Flow:
User → Enters email + password
     → Server hashes password
     → Compares with stored hash
     → Returns token if match

Pros: Simple, everyone understands it
Cons: Passwords are weak (people reuse them)
Security: MUST use strong hashing (bcrypt, Argon2)
```

**2. Multi-Factor Authentication (MFA)**
```text
Something you know (password)
+ Something you have (phone, security key)
= Much more secure!

Example:
1. Enter password ✅
2. System sends code to your phone
3. Enter code from phone ✅
4. Login successful

Why it works: Even if attacker steals your password,
               they don't have your phone!
```

**3. Biometric**
```text
Examples: Fingerprint, Face ID, retina scan

Pros: 
├─ Can't forget your fingerprint
├─ Hard to steal
└─ Great user experience

Cons:
├─ Expensive hardware required
├─ Privacy concerns
└─ Can't change your fingerprint if compromised
```

**4. Social Login (OAuth 2.0)**
```text
"Sign in with Google" button

Flow:
1. User clicks "Sign in with Google"
2. Redirect to Google's login page
3. User logs in to Google (not your app!)
4. Google sends you token proving user identity
5. Your app trusts Google's authentication

Benefits:
├─ No password management for you
├─ User doesn't create another account
└─ Leverages Google's security team
```

---

### 🟡 For Intermediate: Interview Patterns

#### Functional vs Non-Functional Requirements

When you're in a system design interview, you need to translate "we need authentication" into concrete technical requirements. Here's the framework:

**Functional Requirements** (What the system DOES):

| Requirement | Description | Interview Tip |
|------------|-------------|---------------|
| User Registration | Create new account with email/password | Ask: Email verification required? |
| User Login | Authenticate with credentials | Ask: Session-based or token-based? |
| Token Generation | Create JWT or session token | Clarify: Token expiration time? |
| Token Validation | Verify token on each request | Ask: Stateful or stateless validation? |
| Token Refresh | Get new token without re-login | Clarify: Refresh token rotation? |
| Password Reset | Reset via email/SMS | Ask: Token expiration for reset links? |
| Multi-Factor Auth | SMS, TOTP, hardware keys | Clarify: Required or optional? |
| Single Sign-On | Access multiple apps with one login | Ask: SAML or OAuth 2.0? |
| Role Management | Assign permissions to users | Clarify: RBAC or ABAC model? |
| Session Management | Track active sessions | Ask: Concurrent sessions allowed? |
| Audit Logging | Track all authentication events | Clarify: Retention period? |

**Non-Functional Requirements** (How WELL it does it):

```text
Security (Highest Priority):
├─ Password Hashing: bcrypt (cost factor 12) or Argon2
├─ Token Signing: RSA-256 or HMAC-SHA256
├─ Encryption in Transit: TLS 1.3
├─ Encryption at Rest: AES-256
├─ Protection: Rate limiting, brute force detection
└─ Compliance: GDPR, SOC 2, ISO 27001, HIPAA

Performance:
├─ Login: <200ms (P99)
│  └─ Why? Users expect instant feedback
│  └─ Interview insight: Cache user data in Redis
│
├─ Token Validation: <50ms (P99)
│  └─ Why? Happens on EVERY API call
│  └─ Interview insight: Use stateless JWT or cache
│
└─ Token Generation: <100ms
   └─ Why? Infrequent operation, can be slower

Availability: 99.99% uptime
└─ Why? If auth is down, ALL apps are down
└─ Interview insight: Multi-region, active-active

Scalability:
├─ 10M users
├─ 100K authentication requests per second
├─ 1M token validations per second
└─ Interview insight: Read-heavy (validations >> logins)

Durability:
└─ Zero data loss - audit logs are critical
└─ Interview insight: Replicate to multiple regions

Compliance:
├─ GDPR: Right to be forgotten, data portability
├─ SOC 2: Audit trails, access controls
├─ ISO 27001: Information security management
└─ PCI DSS (if handling payments): Strict requirements
```

#### The Clarifying Questions Framework

Great security engineers ask clarifying questions. Here's your interview script:

**Phase 1: Understand the Use Case**
- "Are we building authentication for a single app or multiple apps (SSO)?"
- "Is this for internal employees or external customers?"
- "What's the expected number of users and authentication requests?"

**Phase 2: Understand Authentication Methods**
- "Should we support password-based authentication, or also social login?"
- "Is Multi-Factor Authentication required or optional?"
- "Do we need to support legacy systems (SAML, LDAP)?"

**Phase 3: Understand Security Requirements**
- "What compliance standards do we need to meet (GDPR, SOC 2, HIPAA)?"
- "What's the token lifetime? (Short-lived access tokens + long-lived refresh tokens?)"
- "How do we handle compromised tokens?"

**Phase 4: Understand Authorization**
- "Do we need fine-grained permissions (ABAC) or simple roles (RBAC)?"
- "How many roles/permissions do we expect?"
- "Do permissions change frequently?"

**Phase 5: Understand Performance**
- "What's our latency target for token validation?"
- "Can we use stateless tokens (JWT) or need stateful sessions?"
- "What's our availability target (99.9% vs 99.99%)?"

⚠️ **Common Mistake:** Don't jump into OAuth 2.0 immediately. First understand if you need SSO, third-party login, or just basic authentication. OAuth 2.0 is complex - only use it when needed!

#### Making Assumptions Explicit

After asking questions, state your assumptions clearly:

```text
"Based on our discussion, I'm going to assume:

✅ Authentication Method: Password-based + MFA (optional)
   → This covers 90% of use cases
   → MFA is opt-in for users who want extra security

✅ Token Strategy: JWT (JSON Web Tokens)
   → Access token: 15-minute expiration
   → Refresh token: 30-day expiration
   → Stateless validation (no database lookup needed)

✅ Scale: 10M users, 100K auth requests/sec
   → Read-heavy workload (validations >> logins)
   → Optimize for token validation speed

✅ Authorization Model: RBAC (Role-Based Access Control)
   → Roles: Admin, Manager, User
   → Permissions embedded in JWT claims
   → Simpler than ABAC, covers most use cases

✅ Security Standards: SOC 2 compliant
   → Audit logging required
   → Encryption at rest and in transit
   → Password hashing with bcrypt (cost 12)

✅ High Availability: 99.99% uptime
   → Multi-region deployment
   → Active-active for reads, active-passive for writes

✅ User Experience: Single Sign-On within our ecosystem
   → One login, access multiple apps
   → Session sharing via shared Redis

Are these assumptions reasonable?"
```

This shows structured thinking and invites course correction early!

#### Authentication Flow Examples

Let's visualize the most common flows:

**Flow 1: Password-Based Login**

```mermaid
sequenceDiagram
    participant User
    participant Client
    participant AuthService
    participant Database
    participant Redis

    User->>Client: Enter email + password
    Client->>AuthService: POST /auth/login
    AuthService->>Database: Query user by email
    Database-->>AuthService: User record (hashed password)
    AuthService->>AuthService: Verify password (bcrypt.compare)
    
    alt Password Valid
        AuthService->>AuthService: Generate JWT (access + refresh)
        AuthService->>Redis: Store refresh token
        AuthService-->>Client: Return tokens
        Client-->>User: Login successful
    else Password Invalid
        AuthService-->>Client: 401 Unauthorized
        Client-->>User: Login failed
    end
```

**Flow 2: OAuth 2.0 (Social Login)**

```mermaid
sequenceDiagram
    participant User
    participant YourApp
    participant Google
    participant YourAuthService

    User->>YourApp: Click "Sign in with Google"
    YourApp->>Google: Redirect to Google login
    User->>Google: Enter Google credentials
    Google->>User: Show consent screen
    User->>Google: Grant permission
    Google->>YourApp: Redirect with authorization code
    YourApp->>Google: Exchange code for access token
    Google-->>YourApp: Access token + user info
    YourApp->>YourAuthService: Create/update user account
    YourAuthService->>YourApp: Your app's JWT
    YourApp-->>User: Login successful
```

**Flow 3: Token Validation (Every API Request)**

```mermaid
sequenceDiagram
    participant Client
    participant APIGateway
    participant AuthService
    participant Redis

    Client->>APIGateway: GET /api/data (with JWT)
    APIGateway->>APIGateway: Extract JWT from header
    
    alt Token in Cache
        APIGateway->>Redis: Check token cache
        Redis-->>APIGateway: Valid token ✅
    else Token Not in Cache
        APIGateway->>AuthService: Validate token
        AuthService->>AuthService: Verify signature
        AuthService->>AuthService: Check expiration
        AuthService-->>APIGateway: Token valid ✅
        APIGateway->>Redis: Cache validation result
    end
    
    APIGateway->>APIGateway: Extract user ID & roles
    APIGateway-->>Client: Proceed with request
```

---

### 🔴 For Advanced: Production Considerations

#### Security Architecture Principles

When designing authentication at scale, follow these principles:

**Principle 1: Defense in Depth (Multiple Security Layers)**

```text
Layer 1: Network Security
├─ DDoS protection (Cloudflare, AWS Shield)
├─ WAF (Web Application Firewall)
└─ Rate limiting at edge

Layer 2: Application Security
├─ Input validation
├─ SQL injection prevention
├─ XSS protection
└─ CSRF tokens

Layer 3: Authentication Security
├─ Strong password hashing (Argon2)
├─ Account lockout after failed attempts
├─ MFA for sensitive operations
└─ Anomaly detection

Layer 4: Data Security
├─ Encryption at rest (AES-256)
├─ Encryption in transit (TLS 1.3)
├─ Key rotation (every 90 days)
└─ Secrets management (HashiCorp Vault)

Layer 5: Monitoring & Response
├─ Real-time alerting
├─ Audit logging
├─ Incident response plan
└─ Security information and event management (SIEM)
```

**Principle 2: Least Privilege**

```text
Concept: Users/services should have minimum permissions needed

Implementation:
├─ Default: Deny all access
├─ Explicit: Grant specific permissions
├─ Time-bound: Temporary elevated access
└─ Regular audits: Remove unused permissions

Example:
❌ Bad: All developers have production database access
✅ Good: Developers request temporary access (expires in 4 hours)
         Access request logged and reviewed
```

**Principle 3: Zero Trust Architecture**

```text
Traditional: Trust internal network, protect perimeter
Zero Trust: Never trust, always verify

Implementation:
├─ Authenticate every request (even internal)
├─ Verify device health (is laptop patched?)
├─ Check user context (normal behavior?)
├─ Minimal access (just-in-time permissions)
└─ Continuous monitoring

Example: Google's BeyondCorp
├─ No VPN needed
├─ Every request authenticated
├─ Device must be corporate-managed
└─ Works from anywhere securely
```

#### Compliance & Regulatory Requirements

Real-world authentication systems must comply with regulations:

**GDPR (EU Data Protection):**

```text
Requirements:
├─ Right to Access: Users can download all their data
├─ Right to be Forgotten: Delete user data on request
├─ Data Minimization: Only collect necessary data
├─ Consent Management: Clear opt-in for data processing
├─ Breach Notification: Report breaches within 72 hours
└─ Data Portability: Export data in machine-readable format

Implementation Impact:
├─ User data export API endpoint
├─ Soft delete with grace period
├─ Audit log of all data access
├─ Geo-fencing (EU data stays in EU)
└─ Cookie consent management

Penalties: Up to €20M or 4% of global revenue
```

**SOC 2 (Security Controls):**

```text
Trust Service Criteria:
├─ Security: Access controls, encryption
├─ Availability: 99.9% uptime SLA
├─ Processing Integrity: Complete, accurate processing
├─ Confidentiality: Protect confidential data
└─ Privacy: Notice, choice, collection, access

Authentication-Specific Controls:
├─ MFA for all employees
├─ Password complexity requirements
├─ Session timeout (15 minutes idle)
├─ Audit logs retained for 1 year
├─ Quarterly access reviews
├─ Penetration testing annually
└─ Incident response plan tested

Audit Process: External auditor reviews controls annually
```

**ISO 27001 (Information Security Management):**

```text
Key Requirements:
├─ Risk Assessment: Identify threats, vulnerabilities
├─ Security Policies: Documented procedures
├─ Access Control: Authentication and authorization
├─ Cryptography: Encryption standards
├─ Incident Management: Detection and response
└─ Compliance: Regular audits

Authentication Controls:
├─ Password policy (minimum length, complexity)
├─ Account lockout policy
├─ Privileged access management
├─ Segregation of duties
└─ Security awareness training
```

**HIPAA (Healthcare Data):**

```text
If handling healthcare data:
├─ Access Control: Unique user IDs, emergency access
├─ Audit Controls: Log all access to PHI
├─ Integrity: Protect data from unauthorized changes
├─ Transmission Security: Encrypt data in transit
└─ Person Authentication: Verify user identity

Implementation:
├─ Role-based access to patient data
├─ Audit log: Who accessed which patient record, when
├─ Automatic logoff after 10 minutes
├─ Encrypt all databases containing PHI
└─ BAA (Business Associate Agreement) with vendors

Penalties: $100 - $50,000 per violation
```

#### Threat Modeling & Security Considerations

**Common Attack Vectors:**

1. **Brute Force Attacks**
```text
Threat: Attacker tries many passwords
Defense:
├─ Rate limiting (5 attempts per minute)
├─ Account lockout (after 5 failures, lock for 30 min)
├─ CAPTCHA (after 3 failures)
├─ Progressive delays (1s, 2s, 4s, 8s...)
└─ IP-based blocking (100 failures → block IP)
```

2. **Credential Stuffing**
```text
Threat: Attacker uses leaked passwords from other sites
Defense:
├─ Check against known breach databases (HaveIBeenPwned)
├─ Require password change if credential found in breach
├─ Anomaly detection (login from new location)
├─ Device fingerprinting
└─ MFA requirement for suspicious logins
```

3. **Session Hijacking**
```text
Threat: Attacker steals user's session token
Defense:
├─ Short-lived tokens (15 min access, 30 day refresh)
├─ Secure cookie flags (HttpOnly, Secure, SameSite)
├─ Token binding to IP/device
├─ Refresh token rotation (one-time use)
└─ Token revocation on logout
```

4. **Phishing & Social Engineering**
```text
Threat: Trick user into revealing credentials
Defense:
├─ Email verification for new devices
├─ Anomaly detection (new location, device)
├─ Security keys (FIDO2 - phishing-resistant)
├─ User education (security awareness training)
└─ Risk-based authentication (challenge on suspicious login)
```

5. **Token Theft**
```text
Threat: Attacker intercepts or steals JWT
Defense:
├─ Always use HTTPS (TLS 1.3)
├─ Short token lifetime (15 minutes)
├─ Token refresh flow
├─ Store tokens securely (HttpOnly cookies, not localStorage)
├─ Token introspection endpoint
└─ Real-time token revocation
```

#### Advanced Security Patterns

**Pattern 1: Adaptive Authentication**

```text
Concept: Adjust security based on risk level

Risk Factors:
├─ New device? +10 risk points
├─ New location? +15 risk points
├─ Failed login attempts recently? +20 risk points
├─ VPN/Tor usage? +25 risk points
└─ Impossible travel (US → China in 1 hour)? +50 risk points

Risk Score → Actions:
├─ 0-20: Normal login
├─ 21-40: Require email verification code
├─ 41-60: Require MFA
├─ 61-80: Require security questions + MFA
└─ 81-100: Block login, require manual review

Real-world: AWS uses this for console logins
```

**Pattern 2: Passwordless Authentication**

```text
Methods:
├─ Magic Links: Email a one-time login link
├─ WebAuthn: FIDO2 security keys
├─ Push Notifications: Approve login on your phone
└─ Biometrics: Fingerprint, Face ID

Benefits:
├─ No passwords to forget or steal
├─ Phishing-resistant (WebAuthn)
├─ Better user experience
└─ No password database to breach

Challenges:
├─ Fallback for lost device
├─ User adoption
└─ Device compatibility

Example: Slack uses magic links for login
```

**Pattern 3: Continuous Authentication**

```text
Traditional: Authenticate once, trust for session
Continuous: Constantly verify user behavior

Signals:
├─ Typing patterns (biometric behavior)
├─ Mouse movement patterns
├─ Navigation patterns (how user browses)
├─ Time-of-day patterns
└─ Device sensor data

Action on Anomaly:
├─ Mild: Increase monitoring
├─ Moderate: Require re-authentication
├─ Severe: Terminate session, alert security team

Use Cases:
├─ Financial trading platforms
├─ Healthcare systems
└─ Government systems
```

---

### Real-World Example: How Auth0 Evolved

Let's look at how Auth0 made architectural decisions:

**2013 - Launch (Developer Focus):**
```text
Core Need: Developers need easy authentication
├─ Feature: OAuth 2.0 + Social login
├─ Scale: Thousands of developers
├─ Decision: Developer experience first
└─ Result: Rapid adoption, 1000+ customers in 6 months
```

**2015 - Enterprise Push:**
```text
Customer Feedback: "We need SAML for enterprise SSO"
├─ Added: SAML 2.0 support
├─ Added: Active Directory integration
├─ Added: Audit logs for compliance
├─ Decision: Build enterprise features
└─ Result: Fortune 500 customers (Mazda, AMD)
```

**2018 - Security Hardening:**
```text
Market Trend: Increased breaches and regulations
├─ Added: Anomaly detection (ML-based)
├─ Added: Breached password detection
├─ Added: MFA enforcement policies
├─ Added: GDPR compliance features
├─ Decision: Security as competitive advantage
└─ Result: SOC 2 Type II certified
```

**2021 - Passwordless Future:**
```text
User Expectation: Easier, more secure authentication
├─ Added: WebAuthn/FIDO2 support
├─ Added: Biometric authentication
├─ Added: Passwordless SMS/Email
├─ Decision: Leading edge of authentication
└─ Result: 1.5B logins per month processed
```

📊 **By The Numbers:**
- 2013: 100 authentication requests/sec
- 2018: 10,000 requests/sec
- 2023: 100,000 requests/sec
- 2024: 15,000+ enterprise customers

Key Lesson: Started simple (OAuth), added complexity based on customer needs (SAML, MFA, passwordless), not guessing upfront!

---

### 🤔 Think About It

1. **For Beginners:** Why do you think we use "hashing" for passwords instead of "encryption"? What's the difference, and why does it matter? (Hint: Think about whether you need to "decrypt" a password)

2. **For Intermediate:** If you had to choose between session-based authentication (server stores session) vs token-based authentication (JWT), which would you choose for:
   - A banking app?
   - A mobile app?
   - A microservices architecture?
   Why?

3. **For Advanced:** How would your authentication design change if you were building for:
   - A government agency (Top Secret clearance)?
   - A healthcare provider (HIPAA compliance)?
   - A cryptocurrency exchange (high-value targets)?
   What specific security measures would you add?

---

### ✅ Key Takeaways

- **Authentication vs Authorization**: Authentication proves who you are (ID check), authorization determines what you can do (access level)
- **Security is paramount**: Hash passwords with bcrypt/Argon2, use HTTPS, implement rate limiting, enable MFA
- **Token strategy matters**: Short-lived access tokens (15 min) + long-lived refresh tokens (30 days) = security + UX
- **Compliance is non-negotiable**: GDPR, SOC 2, ISO 27001, HIPAA have specific requirements you must meet
- **Defense in depth**: Multiple security layers (network, application, authentication, data, monitoring)
- **Performance targets**: <50ms token validation (happens on every request), <200ms login
- **Scale considerations**: 10M users, 100K auth requests/sec, 1M validations/sec
- **Threat modeling**: Understand attack vectors (brute force, credential stuffing, session hijacking) and mitigate them

---

### 🎯 Practice Exercise

**Scenario:** You're designing an authentication system for a healthcare startup. They need to comply with HIPAA and handle patient data access for doctors, nurses, and patients.

**Your Task:**
1. List 5 functional requirements specific to healthcare authentication
2. List 5 non-functional requirements with HIPAA justifications
3. What clarifying questions would you ask the healthcare team?
4. Design a role hierarchy (doctors, nurses, patients) with permissions
5. What additional security measures would you implement beyond basic authentication?
6. How would you handle emergency access (doctor needs patient data urgently)?

**Think about:**
- Audit logging requirements
- Session timeout policies
- Access control granularity (which nurse can access which patient?)
- Compliance reporting
- Break-glass procedures (emergency access)

---

## Section 2: Planning for Scale

### What You'll Learn

By the end of this section, you'll be able to:
- Calculate authentication traffic estimates (login QPS and token validation QPS)
- Estimate storage requirements for users, sessions, and audit logs
- Determine bandwidth and resource needs for auth systems
- Understand the massive difference between authentication and authorization loads
- Perform back-of-the-envelope calculations for security-critical systems

### Why This Matters

"How many authentication requests per second?" "How large will our audit logs grow?" "How much cache do we need for token validation?" These questions are crucial for security systems because downtime means no one can access your application! Real example: Auth0 experienced a 2-hour outage in 2020 that locked out users from 15,000+ applications - poor capacity planning for a traffic spike caused cascading failures.

---

### 🟢 For Beginners: Understanding Auth System Scale

#### What Makes Auth Systems Different?

Authentication and authorization systems have unique scaling characteristics:

```text
Normal Application vs Auth System:

E-commerce Site:
├─ Users visit occasionally (few times/day)
├─ Some browse, some buy
├─ Traffic spread throughout day
└─ Users directly interact with site

Auth System (Supporting E-commerce):
├─ EVERY user action needs authorization
├─ Login: Once per session (low frequency)
├─ Token validation: EVERY API call (extremely high frequency)
├─ Must be fast (adds to every request!)
└─ Downtime blocks ALL access
```

The key insight: **Authorization checks happen far more frequently than authentication!**

#### Breaking Down the Numbers

Let's start with a realistic scenario and work through the math:

**Step 1: How many users do we have?**

```text
Our System Size:
├─ Total registered users: 10,000,000 (10M users)
├─ Daily Active Users (DAU): 2,000,000 (20% of total - typical for apps)
└─ Think of this as a medium-sized social media app or SaaS platform
```

**Step 2: Authentication Load (Login Requests)**

```text
How often do users log in?

Assumptions:
├─ 2M daily active users
├─ Users log in once per day on average
├─ Some log in multiple times (different devices)
└─ Multiply by 1.2× for multi-device logins

Daily Logins:
└─ 2,000,000 DAU × 1.2 = 2,400,000 logins per day

Logins Per Second (QPS):
├─ 2,400,000 logins ÷ 86,400 seconds/day
├─ = 27.7 logins/second
└─ ≈ 28 logins/second (average)

Peak Login Traffic:
├─ Morning rush (8-10 AM): 40% of daily logins
├─ Peak is 5× average traffic
└─ Peak: 28 × 5 = 140 logins/second

💡 Key Insight: Authentication is relatively low volume - 
                most users log in once and stay logged in!
```

**Step 3: Authorization Load (Token Validation)**

Here's where it gets interesting:

```text
How often do we validate tokens?

Each user action requires authorization check:
├─ Viewing a page: 1-5 API calls
├─ Posting content: 3-10 API calls
├─ Scrolling feed: 10-20 API calls
└─ Average: 10 API calls per user action

User Activity:
├─ Active user makes: 50 actions per day
├─ Each action: 10 API calls average
└─ Total API calls: 500 per user per day

Daily Authorization Checks:
├─ 2,000,000 DAU × 500 API calls
└─ = 1,000,000,000 (1 billion!) API calls per day

Authorization QPS:
├─ 1,000,000,000 ÷ 86,400 seconds
├─ = 11,574 authorizations/second
└─ ≈ 12,000 auth checks/second (average)

Peak Authorization Load:
├─ Peak is 3× average (evening usage spike)
└─ Peak: 12,000 × 3 = 36,000 auth checks/second

🚨 CRITICAL INSIGHT: Authorization checks are 400× more frequent 
                     than authentication! (12,000 vs 28 per second)
```

**Step 4: The Authorization Challenge**

```text
Why is this such a big deal?

Every API Request Flow:
1. User makes request (GET /posts/123)
2. ⏱️ Validate JWT token (who is this?)
3. ⏱️ Check permissions (can they access this?)
4. Process actual request
5. Return response

Authorization adds latency to EVERY request!

Target Performance:
├─ Token validation: <10ms
├─ Permission check: <5ms
├─ Total auth overhead: <15ms
└─ If 36,000 QPS, we need this consistently!

Without proper design:
├─ Database lookup per request: 50-100ms
├─ 36,000 × 100ms = 3,600,000ms = 3,600 seconds of DB time per second!
└─ Impossible! We need caching and optimization!
```

#### Storage Requirements

**User Data Storage:**

```text
Per User Storage:

User Record:
├─ User ID: 8 bytes (UUID or long integer)
├─ Email: 100 bytes average
├─ Password hash (bcrypt): 60 bytes
├─ Name: 50 bytes
├─ Phone: 20 bytes
├─ Created timestamp: 8 bytes
├─ Last login: 8 bytes
├─ Account status: 1 byte
├─ Metadata: 50 bytes (preferences, settings)
└─ Total: ~300 bytes per user

Total User Storage:
├─ 10,000,000 users × 300 bytes
├─ = 3,000,000,000 bytes
├─ = 3 GB raw data
└─ With indexes (30%): ~4 GB

💡 User data is tiny! Not our storage concern.
```

**Session Storage:**

```text
Active Session Data:

Per Session:
├─ Session ID: 32 bytes
├─ User ID: 8 bytes
├─ Device info: 100 bytes
├─ IP address: 16 bytes
├─ Refresh token: 128 bytes
├─ Expiry timestamp: 8 bytes
├─ Last activity: 8 bytes
└─ Total: ~300 bytes per session

Active Sessions:
├─ 2M DAU × 1.5 devices average = 3M active sessions
├─ 3,000,000 × 300 bytes = 900 MB
└─ Need in fast storage (Redis/Memcached)

Session History (30-day retention):
├─ 2.4M logins/day × 30 days = 72M sessions
├─ 72,000,000 × 300 bytes = 21.6 GB
└─ Can store in database (cheaper)
```

**Audit Log Storage (CRITICAL!):**

```text
Security Audit Logs:

Per Authentication Event:
├─ Event ID: 8 bytes
├─ User ID: 8 bytes
├─ Event type: 20 bytes (LOGIN, LOGOUT, MFA, etc.)
├─ Timestamp: 8 bytes
├─ IP address: 16 bytes
├─ Device fingerprint: 50 bytes
├─ Location (city/country): 30 bytes
├─ Success/failure: 1 byte
├─ Failure reason: 50 bytes
└─ Total: ~200 bytes per auth event

Daily Authentication Logs:
├─ 2.4M logins × 200 bytes = 480 MB/day
├─ Monthly: 480 MB × 30 = 14.4 GB/month
├─ Yearly: 14.4 GB × 12 = 173 GB/year
└─ 5-year retention: 865 GB

Per Authorization Event:
├─ Much smaller: just user ID + resource + result
├─ Total: ~80 bytes per check

Daily Authorization Logs:
├─ 1 billion checks × 80 bytes = 80 GB/day 😱
├─ Monthly: 80 GB × 30 = 2.4 TB/month
├─ Yearly: 2.4 TB × 12 = 28.8 TB/year
└─ 2-year retention: 57.6 TB

🚨 AUDIT LOGS ARE HUGE! This is the real storage challenge!

Common Strategies:
├─ Sample non-suspicious activity (log 1% of normal checks)
├─ Always log failures and suspicious patterns
├─ Compress old logs (10:1 compression ratio)
├─ Move to cold storage after 90 days
└─ Effective storage: ~10 TB (manageable)
```

**Token/Permission Cache:**

```text
Caching User Permissions:

Per User Cache Entry:
├─ User ID: 8 bytes
├─ Roles: 50 bytes (array of role IDs)
├─ Permissions: 200 bytes (array of permission IDs)
├─ Group memberships: 100 bytes
├─ Metadata: 50 bytes
└─ Total: ~400 bytes per user

Cache for Active Users:
├─ 2M DAU × 400 bytes = 800 MB
├─ With overhead: ~1 GB
└─ Must be in Redis for <5ms access

JWT Token Blacklist (for revoked tokens):
├─ Store revoked token IDs until expiry
├─ Average: 10,000 revoked tokens at any time
├─ 10,000 × 32 bytes = 320 KB
└─ Negligible storage
```

#### Resource Summary for Beginners

```text
📊 STORAGE SUMMARY (10M users, 2M DAU):

Hot Storage (Redis/Memcached):
├─ Active sessions: 900 MB
├─ User permissions cache: 1 GB
├─ Token blacklist: 1 MB
└─ Total: ~2 GB (needs fast memory)

Database Storage:
├─ User data: 4 GB
├─ Session history (30 days): 21 GB
├─ Auth audit logs (1 year): 173 GB
├─ Auth audit logs (sampled, 1 year): 3 TB
└─ Total: ~3.2 TB with optimizations

Real-World Cost:
├─ Redis cache (2 GB): $50/month
├─ Database (5 TB): $200/month
├─ Backup storage: $50/month
└─ Total storage: ~$300/month

The takeaway? Storage is cheap! The challenge is SPEED.
```

---

### 🟡 For Intermediate: Interview Calculation Techniques

#### The Auth System Calculation Framework

In interviews, follow this systematic approach for authentication systems:

**Step 1: Establish Scale and Assumptions**

```text
"Let me establish our system scale:

GIVEN:
├─ Total users: 10M registered
├─ Daily Active Users (DAU): 2M (20% - typical for SaaS)
├─ Average session duration: 4 hours
├─ API calls per user session: 500 calls
└─ Multi-device factor: 1.2× (some users on phone + laptop)

KEY ASSUMPTIONS:
├─ Users log in once per session (not per API call!)
├─ Every API call requires authorization check
├─ Token validation must be <10ms (doesn't block user)
└─ Audit logs required for compliance (SOC 2, ISO 27001)
```

**Step 2: Calculate Authentication Load (The Interview Script)**

```text
"Let me break down authentication traffic:

AUTHENTICATION (LOGIN) CALCULATIONS:

Daily Logins:
├─ 2M DAU × 1.2 multi-device factor = 2.4M logins/day
├─ Some users log out and back in (20% = 2 sessions/day)
├─ Adjusted: 2.4M × 1.2 = 2.88M ≈ 3M logins/day
└─ Simplify to: 3M logins/day

Average QPS:
├─ 3M logins ÷ 100K seconds ≈ 30 logins/second
└─ This is manageable for most systems

Peak QPS:
├─ Morning rush (8-10 AM): 30% of daily logins in 2 hours
├─ 3M × 0.3 = 900K logins in 7,200 seconds
├─ Peak: 900K ÷ 7,200 = 125 logins/second
└─ Design for 200 logins/sec (add 60% buffer)

AUTHENTICATION ENDPOINTS:
├─ POST /auth/login: 200 QPS peak
├─ POST /auth/refresh-token: 150 QPS peak (sessions expire)
├─ POST /auth/logout: 50 QPS peak
├─ POST /auth/mfa-verify: 100 QPS peak (50% of logins use MFA)
└─ Total auth endpoints: ~500 QPS peak
```

**Step 3: Calculate Authorization Load (Critical!)**

```text
"Now for authorization - this is where scale gets challenging:

AUTHORIZATION (PERMISSION CHECK) CALCULATIONS:

API Call Volume:
├─ 2M DAU × 500 API calls per day = 1B API calls/day
├─ Every API call needs authorization check
└─ This is 100× more than authentication!

Average Authorization QPS:
├─ 1,000,000,000 ÷ 100,000 seconds ≈ 10,000 checks/second
└─ This requires serious infrastructure

Peak Authorization QPS:
├─ Evening usage spike (7-10 PM): 35% of daily traffic
├─ Peak multiplier: 3×
├─ Peak: 10,000 × 3 = 30,000 checks/second
└─ Design for 50,000 checks/sec (include headroom)

LATENCY REQUIREMENTS:
├─ Each API call already has processing time (50-200ms)
├─ Authorization must add minimal overhead
├─ Target: <10ms for token validation
├─ Target: <5ms for permission check
└─ Total auth overhead: <15ms per request

AT SCALE:
├─ 50,000 requests/sec × 15ms = 750 seconds of work per second!
├─ Need ~800 cores just for authorization (if single-threaded)
├─ OR use aggressive caching to reduce to ~20 cores
└─ Caching is absolutely critical!
```

**Step 4: Storage Calculations with Audit Requirements**

```text
"Storage planning for auth systems has unique requirements:

USER DATA STORAGE:
├─ Per user: 300 bytes (credentials, profile)
├─ 10M users × 300 bytes = 3 GB
├─ With indexes (30%) = 4 GB
└─ Trivial storage requirement

SESSION STORAGE:
├─ Active sessions: 2M DAU × 1.5 devices = 3M sessions
├─ Per session: 300 bytes
├─ Total: 3M × 300 = 900 MB in Redis
├─ Historical (30 days): 3M × 30 = 90M sessions
├─ 90M × 300 bytes = 27 GB in database
└─ Session storage is manageable

AUDIT LOG STORAGE (The Real Challenge):
├─ Per auth event: 200 bytes
├─ Per authz event: 80 bytes

Authentication Logs:
├─ 3M logins/day × 200 bytes = 600 MB/day
├─ 1 year: 600 MB × 365 = 219 GB
└─ 5 years: 1.1 TB (required for compliance)

Authorization Logs:
├─ 1B checks/day × 80 bytes = 80 GB/day 😱
├─ 1 year: 80 GB × 365 = 29.2 TB
└─ THIS IS THE BOTTLENECK!

OPTIMIZATION STRATEGIES:
├─ Sample normal activity: Log 1% = 292 GB/year
├─ Always log: Failures, admin actions, sensitive resources
├─ Compression: 10:1 ratio = 29 GB/year actual storage
├─ Cold storage after 90 days: Move to S3 Glacier
└─ Final: ~500 GB hot + 5 TB cold storage

KEY INTERVIEW POINT:
└─ 'We need intelligent audit logging - full logging isn't 
    feasible at scale. Sample successes, log all failures.'
```

**Step 5: Cache Sizing for Performance**

```text
"Caching is critical for authorization performance:

PERMISSION CACHE (Redis):
├─ Cache user → permissions mapping
├─ 2M DAU × 400 bytes = 800 MB
├─ Add 50% overhead: 1.2 GB
├─ Distribute across 3 Redis nodes: 400 MB each
└─ TTL: 5 minutes (balance freshness vs load)

TOKEN VALIDATION CACHE:
├─ Cache valid tokens (avoid database lookup)
├─ Average tokens: 3M active sessions × 128 bytes = 384 MB
├─ Store: Token signature → user_id + expiry
├─ Hit ratio target: 99% (avoids DB for 99 out of 100 checks)
└─ This reduces database load by 100×!

REVOKED TOKEN BLACKLIST:
├─ Store revoked tokens until natural expiry
├─ Average: 10K revoked tokens × 32 bytes = 320 KB
├─ Check on every request: O(1) lookup in Redis
└─ Minimal storage, critical for security

CACHE EFFECTIVENESS:
├─ Without cache: 30,000 QPS × 20ms DB lookup = 600 seconds/sec
├─ With cache (99% hit): 30,000 × 1% × 20ms + 99% × 1ms = 6.3 seconds/sec
└─ Cache reduces load by 95×! Absolutely essential!
```

**Step 6: Geographic Distribution**

```text
"For a global auth system, we need multi-region deployment:

USER DISTRIBUTION:
├─ North America: 40% = 800K DAU
├─ Europe: 30% = 600K DAU
├─ Asia: 25% = 500K DAU
├─ Other: 5% = 100K DAU
└─ Deploy in at least 3 regions (US, EU, APAC)

PER-REGION REQUIREMENTS:
North America:
├─ Auth QPS: 80 logins/sec peak
├─ Authz QPS: 12,000 checks/sec peak
├─ Cache: 400 MB Redis
└─ Database: Primary in US-East, replicas in US-West

Europe:
├─ Auth QPS: 60 logins/sec peak
├─ Authz QPS: 9,000 checks/sec peak
├─ Cache: 300 MB Redis
└─ Database: Replica from primary (eventual consistency OK)

Asia:
├─ Auth QPS: 50 logins/sec peak
├─ Authz QPS: 7,500 checks/sec peak
├─ Cache: 250 MB Redis
└─ Database: Replica from primary

LATENCY TARGETS:
├─ Same region: <10ms auth overhead
├─ Cross-region: <50ms (acceptable for login, not for API calls)
└─ Strategy: Validate tokens locally, sync user changes globally
```

**Step 7: Infrastructure Sizing**

```text
"Let me size the infrastructure:

APPLICATION SERVERS (Auth Service):
├─ Assume: 2,000 requests/sec per server (mixed auth/authz)
├─ Peak: 50,000 authz + 500 auth = 50,500 QPS total
├─ Servers needed: 50,500 ÷ 2,000 = 26 servers
├─ With redundancy (2N): 52 servers globally
└─ Distribution: 22 US, 18 EU, 12 APAC

DATABASE SERVERS:
├─ Primary (writes): 1 server (200 login writes/sec is light)
├─ Read replicas: 5 servers (distribute read load)
├─ Specs: 16 vCPU, 64 GB RAM, 1 TB SSD each
└─ Cost: ~$500/month per server = $3,000/month

CACHE SERVERS (Redis):
├─ Total cache: 2 GB data + 50% overhead = 3 GB
├─ Distribute: 10 Redis nodes × 300 MB each
├─ Replication: 2× for HA = 20 Redis nodes
├─ Specs: 4 vCPU, 8 GB RAM each
└─ Cost: ~$100/month per node = $2,000/month

MESSAGE QUEUE (Audit Logs):
├─ Kafka for async audit log processing
├─ 3 brokers for HA
├─ Handle: 80 GB/day = 1 MB/sec sustained
└─ Cost: ~$300/month

LOAD BALANCERS:
├─ 3 load balancers (1 per region)
├─ Handle: 50,000 QPS total
└─ Cost: ~$300/month

TOTAL INFRASTRUCTURE:
├─ Application servers: $5,200/month (52 × $100)
├─ Database: $3,000/month
├─ Cache: $2,000/month
├─ Message queue: $300/month
├─ Load balancers: $300/month
├─ Monitoring & logging: $500/month
└─ Total: ~$11,300/month

COST PER USER:
├─ $11,300 ÷ 10M users = $0.00113 per user/month
└─ $0.0136 per user/year (very affordable!)
```

**Step 8: The Interview Presentation**

```text
"Let me summarize our auth system capacity planning:

📊 TRAFFIC SUMMARY:
├─ Authentication: 200 QPS peak (logins)
├─ Authorization: 50,000 QPS peak (permission checks)
├─ Ratio: 250:1 authorization is the real challenge!
└─ Key: Must optimize authorization path aggressively

💾 STORAGE SUMMARY:
├─ User data: 4 GB (trivial)
├─ Active sessions: 1 GB Redis (critical for speed)
├─ Session history: 27 GB (30-day window)
├─ Audit logs: 500 GB hot + 5 TB cold (compliance)
└─ Total: ~6 TB (manageable with tiering)

🖥️ INFRASTRUCTURE SUMMARY:
├─ Auth service: 52 servers globally
├─ Database: 6 servers (1 primary + 5 replicas)
├─ Cache: 20 Redis nodes (with replication)
├─ Message queue: 3 Kafka brokers
└─ Total: ~80 servers for 10M users

💰 COST SUMMARY:
├─ Infrastructure: $11,300/month
├─ Bandwidth: $200/month
├─ Storage: $300/month
└─ Total: $11,800/month = $0.00118/user/month

🎯 CRITICAL DESIGN DECISIONS:
├─ Aggressive caching (99% hit rate reduces DB load 100×)
├─ Sampled audit logging (1% sample for normal activity)
├─ Multi-region deployment (low latency globally)
├─ JWT tokens (stateless validation = no DB lookup)
└─ Async audit log processing (doesn't block requests)

📈 SCALABILITY HEADROOM:
├─ Current: 50K authz QPS
├─ Capacity: 100K authz QPS (2× headroom)
└─ Can handle 2× growth before major changes

Questions on any of these calculations?"
```

---

### 🔴 For Advanced: Production Capacity Planning

#### Real-World Capacity Modeling

Production auth systems face unique scaling challenges:

**Multi-Tenancy Considerations:**

```text
ENTERPRISE SAAS WITH 1,000 CUSTOMERS:

Tenant Distribution (Power Law):
├─ Top 10 tenants: 60% of traffic (6K QPS each at peak)
├─ Next 90 tenants: 30% of traffic (300 QPS each)
├─ Remaining 900 tenants: 10% of traffic (10 QPS each)
└─ Challenge: Prevent large tenants from starving small ones

Noisy Neighbor Problem:
├─ Tenant A: 10K QPS burst (misconfigured retry logic)
├─ Shared infrastructure: Slows down all other tenants
├─ Solution: Per-tenant rate limiting + quotas

Isolation Strategies:
├─ Shared database: Logical isolation (tenant_id in every query)
├─ Shared cache: Namespace by tenant (tenant_123:user_456:perms)
├─ Separate pools: Dedicated infrastructure for top 10 tenants
└─ Cost vs isolation trade-off:
    - Full isolation: $500K/month (10 dedicated stacks)
    - Shared with limits: $50K/month (rate limiting per tenant)
    - Hybrid: $150K/month (top 10 dedicated, others shared)

Recommendation: Hybrid approach
├─ Top 10 tenants: Dedicated auth services (predictable performance)
├─ Next 90: Premium shared pool (higher limits, priority queue)
├─ Remaining: Standard shared pool (lower limits, best effort)
└─ Avoids 90% of noisy neighbor issues at 30% of full isolation cost
```

**Token Validation at Scale:**

```text
JWT VALIDATION OPTIMIZATION:

Standard JWT Validation:
├─ Parse JWT: 0.1ms
├─ Verify signature: 2ms (RSA) or 0.5ms (HMAC)
├─ Check expiry: 0.01ms
├─ Fetch user permissions from DB: 20ms
└─ Total: ~22ms per request

At 50,000 QPS:
├─ 50,000 × 22ms = 1,100,000ms = 1,100 seconds of CPU per second
└─ Impossible without massive parallelization!

OPTIMIZATION LAYERS:

Layer 1 - Signature Caching:
├─ Cache: token_signature → validated_claims
├─ TTL: Token expiry time
├─ Hit rate: 95% (same token reused in short window)
├─ Cached validation: 0.2ms
├─ Result: 50,000 × (0.95 × 0.2ms + 0.05 × 2ms) = 14.5 seconds/sec
└─ 76× improvement!

Layer 2 - Permission Caching:
├─ Cache: user_id → permissions
├─ TTL: 5 minutes (balance freshness vs load)
├─ Hit rate: 99% (permissions don't change often)
├─ Cached lookup: 0.5ms vs 20ms DB
├─ Result: 14.5 × 0.025 = 0.36 seconds/sec
└─ 40× additional improvement!

Layer 3 - Read-Through Cache:
├─ On cache miss: Fetch from DB + update cache
├─ Single request triggers cache refresh
├─ Other concurrent requests: Wait for cache (not DB)
└─ Prevents cache stampede (100 concurrent requests → 1 DB query)

Layer 4 - Token Structure Optimization:
├─ Include basic permissions in JWT claims
├─ "roles": ["admin", "editor"]
├─ No DB lookup for common checks
├─ Only fetch full permissions for complex authorization
└─ Covers 80% of checks without DB

Final Performance:
├─ 80% checks: JWT only (2ms)
├─ 19% checks: JWT + cached permissions (2.5ms)
├─ 1% checks: JWT + DB lookup (22ms)
├─ Average: 0.8 × 2 + 0.19 × 2.5 + 0.01 × 22 = 2.3ms
├─ At 50,000 QPS: 50,000 × 2.3ms = 115 seconds/sec
└─ Requires only ~120 CPU cores (very achievable!)
```

**Audit Log Processing at Scale:**

```text
ASYNC AUDIT LOG ARCHITECTURE:

Challenge: 80 GB/day of audit logs
├─ Can't write synchronously (adds latency to every request)
├─ Can't lose logs (compliance requirement)
└─ Must be searchable (security investigation)

Async Pipeline Design:

Request → [Auth Service] → Response to user (fast!)
              ↓ (non-blocking)
          [Local Buffer]
              ↓ (batch)
          [Kafka Topic]
              ↓ (consume)
          [Processor]
              ↓ (parallel)
        [Elasticsearch] + [S3 Archive]

Component Details:

1. Local Buffer (in memory):
├─ Ring buffer: 10,000 events
├─ Flush when: Buffer full OR every 1 second
├─ If full: Drop low-priority events (success logs)
└─ Never drop: Failures, admin actions, sensitive resource access

2. Kafka (message queue):
├─ Topic: auth-audit-logs
├─ Partitions: 50 (parallel processing)
├─ Replication: 3× (durability)
├─ Retention: 7 days (reprocess if needed)
├─ Throughput: 1M messages/sec capability
└─ Cost: ~$500/month

3. Processing Service:
├─ Consumers: 50 parallel workers
├─ Each processes: 20K events/sec
├─ Total: 1M events/sec capacity
├─ Actions:
│   ├─ Enrich: Add geolocation, user metadata
│   ├─ Classify: Normal, suspicious, critical
│   ├─ Index: Send to Elasticsearch (searchable)
│   └─ Archive: Send to S3 (long-term storage)
└─ Latency: Logs searchable within 10 seconds

4. Elasticsearch (searchable):
├─ Hot data: Last 30 days (immediate search)
├─ Storage: 2.4 TB (with replication)
├─ Indices: Daily rotation (easy to manage retention)
├─ Query performance: <1 second for most queries
└─ Cost: ~$2,000/month

5. S3 Archive (long-term):
├─ All logs: Compressed Parquet format (10:1 ratio)
├─ Storage: 3 TB/year (5-year retention = 15 TB)
├─ Access: Rare (compliance audits, investigations)
├─ Query: AWS Athena (serverless SQL)
└─ Cost: $225/month ($0.023/GB × 10,000 GB)

Total Audit System Cost: ~$2,700/month
├─ Kafka: $500
├─ Processors: $200
├─ Elasticsearch: $2,000
└─ S3: $225

Cost per audit event: $2,700 ÷ 30B events = $0.00000009
└─ Less than 1 millionth of a cent per event!
```

**Disaster Recovery Planning:**

```text
RTO (Recovery Time Objective) AND RPO (Recovery Point Objective):

Auth System DR Requirements:
├─ RTO: 5 minutes (auth is critical - everything depends on it)
├─ RPO: 0 minutes (can't lose user registrations or permission changes)
└─ Availability target: 99.99% (52 minutes downtime/year)

Multi-Region Active-Active:

Primary Region (US-East):
├─ Auth services: 25 servers
├─ Database: Primary (all writes)
├─ Cache: Redis cluster
└─ Audit logs: Kafka + processors

Secondary Region (EU-West):
├─ Auth services: 18 servers (lower traffic)
├─ Database: Async replica (2-second lag)
├─ Cache: Redis cluster (separate)
└─ Audit logs: Kafka + processors

Tertiary Region (APAC):
├─ Auth services: 9 servers
├─ Database: Async replica
├─ Cache: Redis cluster
└─ Audit logs: Kafka + processors

Failover Scenarios:

Scenario 1: Primary region down
├─ Detection: Health checks fail (10 seconds)
├─ Action: DNS failover to EU-West (30 seconds)
├─ Impact: 40-second outage for writes
├─ Reads: Continue serving from replicas
└─ Data loss: Max 2 seconds of writes (RPO)

Scenario 2: Database failure
├─ Detection: 5 seconds
├─ Action: Promote replica to primary (15 seconds)
├─ Impact: 20-second outage for writes
├─ Reads: Unaffected
└─ Data loss: Possible 2 seconds of writes

Scenario 3: Cache failure
├─ Detection: Immediate (cache miss)
├─ Action: Route to database (automatic)
├─ Impact: Latency increases (10ms → 25ms)
├─ Mitigation: Database can handle traffic
└─ Data loss: None (cache is ephemeral)

Scenario 4: Total region failure (earthquake)
├─ Detection: 30 seconds (all health checks fail)
├─ Action: Failover to secondary + tertiary
├─ Impact: 60-second outage
├─ Capacity: Remaining regions handle 2× traffic
└─ Data loss: Max 2 seconds

Testing DR:
├─ Monthly: Failover drill (planned)
├─ Quarterly: Chaos engineering (random failures)
├─ Annually: Total region failure simulation
└─ Track: Actual RTO/RPO vs targets, improve procedures
```

**Capacity Growth Modeling:**

```text
EXPONENTIAL GROWTH PROJECTION:

Current State (Year 0):
├─ Users: 10M
├─ DAU: 2M
├─ Auth QPS: 200 peak
├─ Authz QPS: 50K peak
└─ Cost: $11,800/month

Growth Rate: 50% annually (fast-growing SaaS)

Year 1:
├─ Users: 15M (+50%)
├─ DAU: 3M (+50%)
├─ Auth QPS: 300 peak
├─ Authz QPS: 75K peak
├─ Infrastructure: Add 50% capacity = 80 → 120 servers
├─ Cost: $17,700/month
└─ Action: Optimize caching (defer major changes)

Year 2:
├─ Users: 22.5M (+50%)
├─ DAU: 4.5M (+50%)
├─ Auth QPS: 450 peak
├─ Authz QPS: 112K peak
├─ Infrastructure: 180 servers (approaching limits)
├─ Cost: $26,500/month
└─ Action: Implement database sharding (1 primary → 4 shards)

Year 3:
├─ Users: 33.8M (+50%)
├─ DAU: 6.8M (+50%)
├─ Auth QPS: 675 peak
├─ Authz QPS: 170K peak
├─ Infrastructure: 280 servers + 4 DB shards
├─ Cost: $42,000/month
└─ Action: Edge caching for authz checks (CDN-like)

Year 4:
├─ Users: 50.6M (+50%)
├─ DAU: 10M (+50%)
├─ Auth QPS: 1,000 peak
├─ Authz QPS: 250K peak
├─ Infrastructure: 400 servers + 8 DB shards
├─ Cost: $65,000/month
└─ Action: Consider service mesh for cross-region optimization

Year 5:
├─ Users: 76M (+50%)
├─ DAU: 15M (+50%)
├─ Auth QPS: 1,500 peak
├─ Authz QPS: 375K peak
├─ Infrastructure: 600 servers + 16 DB shards
├─ Cost: $95,000/month
└─ Action: Major architecture review (microservices for auth?)

Cost Optimization Over Time:
├─ Year 1: Basic optimization (caching) - save 20%
├─ Year 2: Reserved instances - save 40%
├─ Year 3: Spot instances for non-critical - save 15%
├─ Year 4: Custom hardware (TPUs for ML fraud detection) - save 25%
└─ Year 5: Actual cost: $50,000/month (vs $95K projected)

Key Planning Insights:
├─ Plan infrastructure changes 6 months ahead
├─ Database sharding is most complex (plan 9 months ahead)
├─ Don't over-optimize early (YAGNI principle)
├─ Cost per user decreases with scale (economies of scale)
└─ Year 0: $0.00118/user/month → Year 5: $0.00066/user/month
```

#### Performance Optimization Strategies

**Token Management Optimization:**

```text
REFRESH TOKEN STRATEGY:

Problem: JWT access tokens expire (15 min typical)
├─ Users make 500 API calls per session (4 hours)
├─ Without refresh: Force re-login every 15 min (bad UX)
├─ With refresh: Seamless token renewal
└─ But: Refresh adds load

Naive Approach (Expensive):
├─ Check token expiry on every API call
├─ If expires in <2 minutes: Auto-refresh
├─ Cost: 50,000 QPS × 2ms = 100 seconds/sec overhead
└─ Wasteful: Most tokens are fresh

Optimized Approach:
├─ Client-side: Track token expiry locally
├─ Client refreshes: 1 minute before expiry
├─ Refresh endpoint: POST /auth/refresh-token
├─ Frequency: Once per 15 min per user
├─ Load: 2M DAU ÷ (15 min × 60 sec) = 2,222 QPS
└─ 22× lower load!

Implementation:
1. Access token (short-lived, JWT):
   ├─ Expiry: 15 minutes
   ├─ Contains: user_id, roles, basic permissions
   └─ Stateless: No DB lookup needed

2. Refresh token (long-lived, opaque):
   ├─ Expiry: 30 days
   ├─ Stored: Database + Redis
   ├─ Rotates: Every use (security best practice)
   └─ Enables: Revocation (e.g., logout all devices)

Refresh Flow:
1. Client: Detects access token expires in 1 minute
2. Client: POST /auth/refresh-token with refresh_token
3. Server: Validates refresh token (Redis lookup: 1ms)
4. Server: Issues new access token + new refresh token
5. Server: Invalidates old refresh token
6. Client: Uses new access token for subsequent calls

Security Benefits:
├─ Access token leaked: Expires in 15 min (limited damage)
├─ Refresh token leaked: Can revoke (logout all)
├─ Token rotation: Old refresh tokens don't work
└─ Suspicious activity: Block refresh (force re-auth)
```

**Distributed Caching Strategy:**

```text
MULTI-TIER CACHE ARCHITECTURE:

L1 Cache - In-Memory (Application Server):
├─ Location: Each auth service instance
├─ Technology: Local HashMap/LRU cache
├─ Size: 100 MB per server (52 servers = 5.2 GB total)
├─ Contents: Hot permissions (last 1,000 users accessed)
├─ TTL: 1 minute (short - prevent stale permissions)
├─ Hit rate: 70% (Covers most common users)
├─ Latency: 0.01ms (nanoseconds!)
└─ Consistency: Eventually consistent (1-min lag acceptable)

L2 Cache - Distributed (Redis Cluster):
├─ Location: Centralized per region
├─ Technology: Redis cluster (10 shards)
├─ Size: 2 GB total (all active user permissions)
├─ Contents: All DAU permissions
├─ TTL: 5 minutes
├─ Hit rate: 29% (L1 miss → L2 hit)
├─ Latency: 1ms (network + lookup)
└─ Consistency: 5-min lag acceptable

L3 Cache - Database Read Replicas:
├─ Location: 5 replicas per region
├─ Technology: PostgreSQL with read-only replicas
├─ Size: Full dataset (10M users)
├─ Contents: All user permissions
├─ TTL: Real-time replication (2-sec lag)
├─ Hit rate: 1% (L1+L2 miss → DB)
├─ Latency: 20ms (SQL query)
└─ Consistency: 2-sec lag (async replication)

Combined Performance:
├─ 70% requests: L1 hit (0.01ms) = 35K QPS
├─ 29% requests: L2 hit (1ms) = 14.5K QPS
├─ 1% requests: L3 hit (20ms) = 500 QPS
├─ Weighted average: 0.70 × 0.01 + 0.29 × 1 + 0.01 × 20 = 0.5ms
└─ At 50K QPS: 50,000 × 0.5ms = 25 seconds/sec (manageable!)

Cache Invalidation Strategy:
├─ Permission change: Invalidate specific user in L2
├─ Propagation: L2 invalidation triggers L1 eviction (pubsub)
├─ Max staleness: 1 min (L1) + 5 min (L2) = 6 min total
└─ Critical changes: Force re-auth (logout user)

Cost Analysis:
├─ L1: Free (part of app servers)
├─ L2: $2,000/month (Redis cluster)
├─ L3: $3,000/month (DB replicas - needed anyway)
└─ Total incremental cost: $2,000/month

Value: Reduces DB load by 99×
├─ Without cache: 50K QPS × 20ms = 1,000 seconds/sec
├─ With cache: 500 QPS × 20ms = 10 seconds/sec
└─ Enables handling 100× more traffic with same DB
```

---

### Real-World Example: Auth0's Scale Evolution

**2015 - Early Growth:**
```text
Scale:
├─ 50K tenants
├─ 500M authentications/month
├─ Infrastructure: 50 servers, single region (US)
└─ Cost: ~$20K/month

Bottleneck:
└─ MongoDB database becoming overloaded

Solution:
└─ Sharded MongoDB, added Redis caching
```

**2018 - Enterprise Adoption:**
```text
Scale:
├─ 500K tenants
├─ 4.5B authentications/month (9× growth!)
├─ Infrastructure: 500 servers, 3 regions
└─ Cost: ~$150K/month

Bottleneck:
└─ Authorization checks (RBAC) slowing down API calls

Solution:
├─ Edge caching for permissions (Fastly CDN)
├─ JWT with embedded permissions
└─ Reduced authorization latency from 25ms → 5ms
```

**2023 - Massive Scale:**
```text
Scale:
├─ 8M+ tenants
├─ 42B+ authentications/month
├─ Infrastructure: 2,000+ servers, 10+ regions
└─ Cost: ~$500K/month (optimized down from $1.2M)

Architecture:
├─ Multi-region active-active
├─ Edge computing for token validation
├─ ML-based fraud detection (real-time)
└─ 99.99% uptime SLA

Optimizations:
├─ Custom ASICs for JWT signing (10× faster)
├─ Intelligent audit sampling (10:1 reduction)
├─ Tiered storage (hot/warm/cold)
└─ Reserved instances + spot instances (40% cost savings)

By The Numbers:
├─ Cost per auth: $0.012 (2023) vs $0.40 (2015)
├─ Latency: 5ms (2023) vs 100ms (2015)
└─ Key lesson: Continuous optimization essential!
```

---

### 🤔 Think About It

1. **For Beginners:** Why is authorization traffic 400× higher than authentication traffic in our calculations? Can you think of a scenario where this ratio would be different?

2. **For Intermediate:** If you had to choose between reducing authentication latency from 200ms to 100ms OR reducing authorization latency from 20ms to 10ms, which would have a bigger impact on user experience? Why?

3. **For Advanced:** Your audit logs are growing 80 GB/day. Your CFO says "We can't afford this storage cost." What's your response? What trade-offs would you present between compliance, cost, and security?

---

### ✅ Key Takeaways

- **Authentication vs Authorization**: Auth happens once, authz happens on every API call (100-400× more frequent)
- **Caching is critical**: Without 99% cache hit rate, authorization becomes a bottleneck
- **Audit logs dominate storage**: Authorization logs can be 100× larger than user data
- **Sample intelligently**: Log all failures and sensitive actions, sample routine successes
- **Multi-tier caching**: L1 (local) + L2 (Redis) + L3 (DB) = 99%+ hit rate
- **Global deployment**: Multi-region for latency, not just redundancy
- **Plan for 2× growth**: Before hitting capacity limits, not after
- **JWT enables scale**: Stateless validation = no DB lookup on every request
- **Cost per user decreases**: Economies of scale (Year 0: $0.00118/user → Year 5: $0.00066/user)
- **DR is non-negotiable**: Auth system down = entire application down (RTO: 5 min)

---

### 🎯 Practice Exercise

**Scenario:** You're planning an authentication system for a video streaming service (like Netflix).

**Given Information:**
- 100M registered users globally
- 40M daily active users (40% DAU)
- Each user streams 3 videos/day average
- Each video: 20 API calls (quality changes, progress tracking, recommendations)
- Geography: 35% Americas, 30% Europe, 25% Asia, 10% Other
- Each user logs in on 2.5 devices on average (phone, TV, laptop)
- Peak hours: 8 PM - 11 PM in each timezone (4× average traffic)

**Your Task:**
1. Calculate authentication load (login QPS average and peak)
2. Calculate authorization load (API call QPS average and peak)
3. Calculate storage requirements:
   - User data
   - Active sessions
   - Audit logs (1-year retention with intelligent sampling)
4. Size the infrastructure:
   - How many auth service servers?
   - How much Redis cache?
   - How many database servers?
5. Multi-region strategy:
   - How many regions?
   - How to distribute traffic?
   - Latency targets per region?
6. Estimate monthly cost (research AWS/GCP pricing)
7. What happens if a major region goes down? (Calculate RTO/RPO)

**Bonus Challenge:**
- Device authentication: Each user has 2.5 devices. Should they share sessions or have separate tokens? Why?
- Family accounts: 4 users share one account. How does this change your calculations?
- Offline viewing: Users download videos to watch offline. How do you handle authorization checks without internet?

---


## Section 3: Designing the System Architecture

### What You'll Learn

By the end of this section, you'll be able to:
- Design a high-level architecture for an Authentication & Authorization system
- Explain the purpose and interaction of each security component
- Understand data flow through authentication and authorization pipelines
- Choose appropriate technologies for each security layer
- Design for stateless and stateful authentication patterns

### Why This Matters

Architecture is the foundation of security. A well-designed auth system prevents breaches, scales to millions of users, and provides seamless user experiences. A poorly designed one leads to security vulnerabilities, performance bottlenecks, and compliance failures. Real example: In 2020, a major social media platform suffered a breach because their authentication architecture allowed token reuse across different security contexts, exposing 50M user accounts!

---

### 🟢 For Beginners: Building Blocks of Our Auth System

#### Thinking Like a Security Architect

Imagine you're designing security for a large office building. You don't just think about "checking IDs" - you think about:
- **Reception desk** (where visitors check in and get badges)
- **Security guards** (who verify badges at checkpoints)
- **Badge system** (what access each badge allows)
- **Audit logs** (who entered which room and when)

Our authentication & authorization system has similar components! Let's understand each building block:

#### The Simple Version (1,000 users)

When you're just starting out, keep it simple:

```mermaid
sequenceDiagram
    participant User as 👤 User Browser
    participant Server as 🖥️ Auth Server
    participant DB as 💾 Database

    Note over User,DB: User Login (Authentication)
    User->>Server: POST /login<br/>username + password
    Server->>DB: Verify credentials
    DB-->>Server: User found ✓
    Server->>Server: Generate session token
    Server-->>User: Return session token
    
    Note over User,DB: Later: API Call (Authorization)
    User->>Server: GET /api/profile<br/>Authorization: Bearer token
    Server->>DB: Validate token
    DB-->>Server: Token valid ✓
    Server-->>User: Return profile data
```

> **✅ Key Advantage:** Simple! One server, one database - perfect for getting started quickly and handling up to ~10,000 users with basic security needs.

#### The Problem: What Happens When You Grow?

Imagine your auth system becomes critical for a company with 1 million users:

```text
Problems:
❌ Single server gets overwhelmed (security bottleneck)
❌ Database becomes slow (everyone waiting to login)
❌ If auth server crashes, NOBODY can login (total outage)
❌ No multi-factor authentication (security risk)
❌ Session data stored in memory (lost on restart)
❌ Can't handle distributed systems (microservices need auth)

We need to design for scale AND security!
```

#### The Production Version: Breaking It Down

Let's add components one by one, understanding WHY we need each:

**Component 1: Load Balancer with SSL Termination (The Secure Gateway)**

```text
Think of this like a security checkpoint with metal detectors:

[Many Users] → [Load Balancer with SSL] → [Multiple Auth Servers]
                      ↓
         "Decrypt HTTPS traffic here,
          route to healthy auth server,
          prevent DDoS attacks!"

Why we need it:
✅ Distributes authentication load across servers
✅ Terminates SSL/TLS (encrypts sensitive credentials)
✅ DDoS protection (rate limiting)
✅ Health checks (routes away from failing servers)
✅ Geographic routing (send users to nearest data center)
```

**Component 2: Authentication Service (The Identity Verifier)**

```text
Instead of one monolithic server, separate concerns:

[Load Balancer]
    ↓
[Authentication Service] ← Handles login, signup, password reset
    ├─ Verify credentials
    ├─ Hash passwords (bcrypt/Argon2)
    ├─ Generate tokens (JWT/session)
    ├─ Enforce MFA
    └─ Track failed login attempts

Why separate:
✅ Focused responsibility (does one thing well)
✅ Can scale independently from authorization
✅ Easier to secure (smaller attack surface)
✅ Can update without affecting other services
```

**Component 3: Authorization Service (The Permission Checker)**

```text
Separate from authentication:

[Authorization Service] ← Handles permission checks
    ├─ Validate tokens (JWT signature verification)
    ├─ Check permissions (RBAC/ABAC)
    ├─ Enforce policies (can user X access resource Y?)
    ├─ Cache permission decisions
    └─ Log access attempts

Why separate:
✅ Authorization called 100x more than authentication
✅ Different scaling characteristics
✅ Can use different storage (permissions vs credentials)
✅ Easier to implement fine-grained access control
```

**Component 4: Token Service (The Badge Issuer)**

```text
Manages token lifecycle:

[Token Service]
    ├─ Issue access tokens (short-lived, 15-60 min)
    ├─ Issue refresh tokens (long-lived, 30-90 days)
    ├─ Revoke tokens (logout, security breach)
    ├─ Token blacklist (prevent reuse)
    └─ Token introspection (validate tokens)

Why separate:
✅ Centralized token management
✅ Can implement different token types (JWT, opaque)
✅ Easier to enforce token expiration policies
✅ Supports token refresh without re-authentication
```

**Component 5: User Service (The Identity Store)**

```text
[User Service]
    ├─ Manage user profiles
    ├─ Store encrypted credentials
    ├─ Track user attributes (email, phone, roles)
    ├─ Handle user lifecycle (create, update, delete)
    └─ Support federated identity (Google, Facebook login)

Why separate:
✅ User data changes frequently (profile updates)
✅ Different security requirements (PII protection)
✅ Can integrate with external identity providers
✅ Easier to implement GDPR compliance (data portability)
```

**Component 6: MFA Service (The Extra Security Layer)**

```text
[MFA Service]
    ├─ Send verification codes (SMS, Email, Push)
    ├─ Validate TOTP codes (Google Authenticator)
    ├─ Manage backup codes
    ├─ Support biometric verification
    └─ Track MFA enrollment

Why separate:
✅ Not all users need MFA (progressive security)
✅ Integrates with external services (Twilio, SendGrid)
✅ Can fail gracefully (fallback to email)
✅ Different SLA requirements (real-time SMS delivery)
```

**Component 7: Audit Service (The Security Logger)**

```text
[Audit Service]
    ├─ Log all authentication attempts
    ├─ Log all authorization decisions
    ├─ Track security events (suspicious activity)
    ├─ Store immutable audit trail
    └─ Generate compliance reports

Why separate:
✅ Doesn't slow down auth (async logging)
✅ Can handle millions of audit events
✅ Won't crash main system if audit fails
✅ Different storage requirements (long-term retention)
```

**Component 8: Redis Cache (The Speed Booster)**

```text
Think of cache like keeping frequently used badges at reception:

[Auth Service] → "Is token xyz valid?" → [Redis Cache]
                                            ├─ ✅ Found it! (1ms)
                                            └─ ❌ Not here, check DB

Why we need it:
✅ Super fast token validation (1-2ms vs 10-50ms)
✅ Reduces load on database
✅ Cache token blacklist (revoked tokens)
✅ Store rate limiting counters (prevent brute force)
✅ Session storage (distributed sessions)
```

**Component 9: Database with Replicas (The Truth Source)**

```text
[Primary Database] ← Write new users, update credentials
    ├─→ [Replica 1] ← Read user profiles
    ├─→ [Replica 2] ← Read permissions
    └─→ [Replica 3] ← Read audit logs

Why this setup:
✅ Writes go to one place (prevents conflicts)
✅ Reads can be distributed (handle more traffic)
✅ If primary fails, a replica can take over
✅ Separate credentials (encrypted) from permissions
```

#### The Complete Picture

Here's how it all works together:

```mermaid
graph TD
    Users[👥 Users Worldwide<br/>Web, Mobile, API]
    LB[⚖️ Load Balancer<br/>SSL Termination<br/>DDoS Protection]
    
    subgraph "🔐 Authentication Layer"
        AuthSvc[Authentication Service<br/>Login, Signup, Password Reset]
        MFASvc[MFA Service<br/>2FA, TOTP, SMS]
        UserSvc[User Service<br/>Profile Management]
    end
    
    subgraph "🛡️ Authorization Layer"
        AuthZSvc[Authorization Service<br/>Permission Checks]
        TokenSvc[Token Service<br/>JWT Issue/Validate]
    end
    
    Cache[(⚡ Redis Cache<br/>Token Validation<br/>Rate Limiting<br/>~1ms response)]
    
    subgraph "💾 Database Layer"
        DBPrimary[(🔵 Primary<br/>User Credentials<br/>Encrypted)]
        DBReplica1[(🔵 Replica 1<br/>Read Queries)]
        DBReplica2[(🔵 Replica 2<br/>Read Queries)]
    end
    
    AuditSvc[📊 Audit Service<br/>Security Logs<br/>Compliance]
    AuditDB[(📜 Audit Database<br/>Immutable Logs)]
    
    Users -->|1. HTTPS Request| LB
    LB --> AuthSvc
    LB --> AuthZSvc
    
    AuthSvc -->|2. Verify User| UserSvc
    AuthSvc -->|3. Check MFA| MFASvc
    AuthSvc -->|4. Issue Token| TokenSvc
    
    AuthZSvc -->|5. Validate Token| Cache
    AuthZSvc -->|6. Check Permissions| Cache
    AuthZSvc -->|7. Cache MISS| DBReplica1
    
    UserSvc -->|Write| DBPrimary
    UserSvc -->|Read| DBReplica1
    TokenSvc -->|Store Token Metadata| Cache
    
    DBPrimary -.->|Replicate| DBReplica1
    DBPrimary -.->|Replicate| DBReplica2
    
    AuthSvc -.->|Log Events| AuditSvc
    AuthZSvc -.->|Log Decisions| AuditSvc
    AuditSvc -.->|Store| AuditDB
    
    style LB fill:#FFE4B5
    style Cache fill:#90EE90
    style DBPrimary fill:#87CEEB
    style AuditSvc fill:#DDA0DD
```

#### What Each Component Does (Simple Explanation)

| Component | Job | Analogy |
|-----------|-----|---------|
| **Load Balancer** | Routes requests, SSL termination | Security checkpoint at building entrance |
| **Auth Service** | Verifies who you are | Reception desk checking your ID |
| **AuthZ Service** | Checks what you can access | Security guard checking your badge level |
| **Token Service** | Issues and validates tokens | Badge printing and scanning system |
| **User Service** | Manages user information | HR database of employees |
| **MFA Service** | Extra security verification | Security code sent to your phone |
| **Cache** | Fast token lookups | Quick reference list at security desk |
| **Database** | Permanent credential storage | Secure vault with all credentials |
| **Audit Service** | Logs all security events | Security camera and access logs |

💡 **Pro Tip:** In interviews, always explain WHY you separate services. "We separate authentication from authorization because auth is called 100x more frequently and has different scaling needs" shows deep understanding!

---

### 🟡 For Intermediate: Architecture Patterns and Decisions

#### The Architecture Decision Framework

When designing auth architecture, every component choice involves security and performance trade-offs. Here's how to think through them systematically:

**Layer 1: Entry Point (Load Balancer + API Gateway)**

```text
DECISION: Should we use an API Gateway vs Load Balancer?

Load Balancer (Layer 4/7):
├─ Simple traffic routing
├─ SSL termination
├─ Basic rate limiting
├─ Very fast, very scalable
└─ Tools: AWS ALB, NGINX

API Gateway:
├─ Advanced rate limiting (per user, per endpoint)
├─ Request validation (schema checking)
├─ API versioning
├─ Transformation (request/response modification)
├─ Built-in OAuth support
└─ Tools: Kong, AWS API Gateway, Apigee

VERDICT: ✅ Use API Gateway for production auth system
Reasoning:
├─ Need sophisticated rate limiting (prevent brute force)
├─ Want centralized authentication logic
├─ API versioning critical for backward compatibility
├─ Request validation prevents malformed attacks
└─ Interview tip: Explain security benefits!
```

**Layer 2: Authentication Flow Design**

```text
PATTERN 1: Stateless Authentication (JWT)

Flow:
1. User logs in → Server verifies credentials
2. Server generates JWT (signed token)
3. Client stores JWT (localStorage/cookie)
4. Client sends JWT with every request
5. Server validates JWT signature (NO database lookup!)

Pros:
✅ No database lookup per request (very fast)
✅ Horizontally scalable (no session state)
✅ Works well for microservices
✅ Can include claims (user info in token)

Cons:
❌ Can't revoke tokens (until expiration)
❌ Token size larger (sent with every request)
❌ Secret rotation complex (need to validate old + new)

WHEN TO USE: High-scale APIs, microservices, stateless preferred

---

PATTERN 2: Stateful Authentication (Sessions)

Flow:
1. User logs in → Server verifies credentials
2. Server creates session in Redis
3. Server sends session ID to client
4. Client sends session ID with every request
5. Server validates session ID against Redis

Pros:
✅ Can revoke sessions immediately (logout, security)
✅ Smaller token size (just session ID)
✅ Centralized session management
✅ Easy secret rotation

Cons:
❌ Requires database/cache lookup per request
❌ Horizontal scaling harder (need shared session store)
❌ Session store is single point of failure

WHEN TO USE: Web applications, need immediate revocation

---

PATTERN 3: Hybrid (JWT + Refresh Token)

Flow:
1. User logs in → Server issues short-lived JWT (15 min) + long-lived refresh token (30 days)
2. Client uses JWT for API calls
3. JWT expires → Client uses refresh token to get new JWT
4. Refresh token stored in database (can be revoked)

Pros:
✅ Best of both worlds!
✅ Fast validation (JWT) + revocation capability (refresh token)
✅ Limits damage if JWT stolen (short-lived)
✅ Can revoke refresh tokens (logout)

Cons:
❌ More complex implementation
❌ Two token types to manage
❌ Client needs refresh logic

VERDICT: ✅ Use Hybrid for production
└─ Industry standard (OAuth 2.0 pattern)
```

#### Service-to-Service Authentication

```text
PATTERN: Service Mesh + mTLS

Challenge:
├─ Microservice A needs to call Microservice B
├─ How does B trust A?
├─ Can't use user credentials
└─ Need fast, secure, automated

Solution: Mutual TLS (mTLS)

[Service A] ←→ [Service B]
     ↓              ↓
   Cert A        Cert B
     ↓              ↓
[Service Mesh: Istio/Linkerd]
     ↓
[Certificate Authority]

How it works:
1. Each service gets a unique certificate
2. Service mesh automatically handles TLS
3. Services verify each other's certificates
4. Certificates rotate automatically (daily)

Benefits:
✅ Automatic encryption (no code changes)
✅ Zero-trust security (verify every call)
✅ Fast (TLS handshake cached)
✅ Observability (mesh logs all traffic)

Implementation (Istio):
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT  # Require mTLS for all services

INTERVIEW TIP: Know when to use mTLS vs API keys vs OAuth!
```

#### Token Validation Strategy

```text
DECISION: Where to validate tokens?

Option A: Centralized Token Validation

[API Gateway] → Validates ALL tokens here
     ↓
[Microservices] ← Trust gateway's decision

Pros:
✅ Single point of validation (consistency)
✅ Easier to update validation logic
✅ Microservices stay simple

Cons:
❌ Gateway is bottleneck
❌ Single point of failure
❌ Gateway must scale heavily

---

Option B: Distributed Token Validation

[Microservices] → Each validates tokens independently
     ↓
[Shared Token Service] ← Fetch public keys

Pros:
✅ No bottleneck (each service scales independently)
✅ No single point of failure
✅ Lower latency (local validation)

Cons:
❌ Code duplication (each service has validation)
❌ Harder to update (deploy to all services)
❌ Inconsistent validation possible

---

VERDICT: ✅ Use Distributed Validation with JWT

Implementation:
1. Token Service publishes public key (JWK Set)
2. Each microservice caches public key
3. Each microservice validates JWT signature locally
4. No network call needed (fast!)

Code (Node.js):
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

// Fetch public key from token service
const client = jwksClient({
  jwksUri: 'https://auth.example.com/.well-known/jwks.json',
  cache: true,
  cacheMaxAge: 86400000  // 24 hours
});

function verifyToken(token) {
  const decoded = jwt.decode(token, {complete: true});
  const key = await client.getSigningKey(decoded.header.kid);
  const publicKey = key.getPublicKey();
  
  return jwt.verify(token, publicKey);  // Validate signature
}

Benefits:
✅ No database lookup per request
✅ Fast (local validation, ~1ms)
✅ Scalable (each service independent)
```

#### Multi-Region Architecture

```text
CHALLENGE: Global users, low latency, high security

Architecture:

US-East (Primary)     EU-West (Active)      Asia-Pacific (Active)
     ↓                     ↓                        ↓
[Auth Services]       [Auth Services]          [Auth Services]
[Token Service]       [Token Service]          [Token Service]
     ↓                     ↓                        ↓
[User DB Primary] → [User DB Replica] → [User DB Replica]

Token Validation Strategy:
├─ JWT validation is LOCAL (no cross-region call!)
├─ Public keys replicated to all regions
├─ Token issuance can happen in any region
└─ User data eventually consistent (read replicas)

Critical Decision: Token Signing Keys

Option A: Single Signing Key (Shared)
├─ All regions use same private key
├─ Tokens issued in US can be validated in EU
├─ Risk: If key leaked, all regions compromised
└─ Key rotation requires global coordination

Option B: Per-Region Signing Keys
├─ Each region has unique private key
├─ All regions publish public keys to JWK Set
├─ Tokens issued in US can be validated in EU (using US public key)
├─ If one key leaked, only one region compromised
└─ Key rotation simpler (per-region)

VERDICT: ✅ Use Per-Region Keys
└─ Security > Simplicity for auth systems

Configuration:
{
  "keys": [
    {
      "kid": "us-east-2026-01",
      "kty": "RSA",
      "use": "sig",
      "n": "...",  // Public key
      "e": "AQAB"
    },
    {
      "kid": "eu-west-2026-01",
      "kty": "RSA",
      "use": "sig",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

#### Handling Authentication Failures

```text
ANTI-PATTERN: Blocking Authentication

Problem:
User enters wrong password
  → Server immediately returns "Invalid credentials"
  → Attacker can brute force passwords quickly

This leaks information and enables attacks!

CORRECT PATTERN: Rate Limiting + Progressive Delays

# In authentication service
from redis import Redis
from time import sleep

redis = Redis()

def authenticate(username, password):
    # Check rate limit
    attempts_key = f"auth_attempts:{username}"
    attempts = redis.incr(attempts_key)
    redis.expire(attempts_key, 3600)  # 1 hour window
    
    if attempts > 5:
        # Progressive delay (1s, 2s, 4s, 8s, ...)
        delay = min(2 ** (attempts - 5), 300)  # Max 5 min
        sleep(delay)
        return {"error": "Too many attempts. Try again later."}
    
    # Verify credentials
    user = db.get_user(username)
    if not user or not verify_password(password, user.password_hash):
        # ALWAYS take same time (prevent timing attacks)
        sleep(random.uniform(0.1, 0.3))
        return {"error": "Invalid credentials"}
    
    # Success - clear attempts
    redis.delete(attempts_key)
    return generate_tokens(user)

Security Benefits:
✅ Prevents brute force (rate limiting)
✅ No information leakage (same error for user not found vs wrong password)
✅ Timing attack resistant (constant time)
✅ Progressive penalties (exponential backoff)
```

#### Technology Choices and Justifications

**Database Selection for User Credentials:**

```text
REQUIREMENTS:
├─ ACID compliance (users must be created exactly once)
├─ Encryption at rest (PII and credentials)
├─ Strong consistency for writes (no duplicate users)
├─ Read-heavy workload (authentication >> registration)
└─ Need to store 100M users (approx 100 GB)

OPTIONS ANALYSIS:

Option A: PostgreSQL
├─ Pros: ACID, excellent security features, mature
├─ Cons: Single master for writes, vertical scaling limits
├─ Verdict: ✅ Best choice for credential storage
└─ Why: Strong ACID guarantees critical for security

Option B: MongoDB
├─ Pros: Flexible schema, sharding built-in
├─ Cons: Weaker ACID than PostgreSQL
├─ Verdict: ⚠️ Possible but not optimal for credentials
└─ When to use: Need flexible user attributes

Option C: DynamoDB
├─ Pros: Fully managed, infinite scale
├─ Cons: Eventually consistent (default), more expensive
├─ Verdict: ⚠️ Good for sessions, not credentials
└─ When to use: Serverless architecture, need auto-scale

CHOICE: PostgreSQL for credentials, Redis for sessions
Reasoning:
├─ Credentials need strong consistency (PostgreSQL)
├─ Sessions need speed (Redis)
├─ Can handle 100M users easily
└─ Battle-tested security features
```

**Cache Technology:**

```text
Redis vs Memcached for Token Validation:

Redis:
├─ Data structures: Strings, Sets, Sorted Sets
├─ Persistence options (survive restarts)
├─ Atomic operations (INCR for rate limiting)
├─ Pub/sub for real-time invalidation
├─ Lua scripting for complex operations
└─ TTL per key (token expiration)

Memcached:
├─ Simple key-value only
├─ Slightly faster for pure caching
├─ No persistence (lose all data on restart)
└─ Less memory overhead

CHOICE: Redis
Reasoning:
├─ Need sorted sets for token blacklist (revoked tokens)
├─ Need atomic INCR for rate limiting counters
├─ Need pub/sub for cross-region token revocation
├─ Persistence critical (don't lose sessions on restart)
└─ Worth minimal performance overhead for features
```

#### Deployment Topology

**Single Region (MVP - First 6 months):**

```text
AWS us-east-1 (Northern Virginia)

┌────────────────────────────────────┐
│  Availability Zone 1               │
│  ├─ API Gateway (Primary)          │
│  ├─ Auth Service (3 instances)     │
│  ├─ AuthZ Service (5 instances)    │
│  ├─ Token Service (2 instances)    │
│  ├─ User Service (2 instances)     │
│  ├─ MFA Service (2 instances)      │
│  ├─ Database Primary               │
│  └─ Redis Cluster (3 nodes)        │
└────────────────────────────────────┘

┌────────────────────────────────────┐
│  Availability Zone 2               │
│  ├─ API Gateway (Failover)         │
│  ├─ Auth Service (3 instances)     │
│  ├─ AuthZ Service (5 instances)    │
│  ├─ Database Replica               │
│  └─ Redis Cluster (3 nodes)        │
└────────────────────────────────────┘

Benefits:
✅ Simple to manage
✅ High availability (multi-AZ)
✅ Low complexity
✅ Sufficient for 10M users

Limitations:
⚠️ High latency for EU/Asia users
⚠️ All eggs in one region
⚠️ Compliance issues (data residency)
```

**Multi-Region (Scale - Global deployment):**

```text
US-East (Primary)     EU-West (Active)      Asia-Pacific (Active)
     ↓                     ↓                        ↓
[Auth Services]       [Auth Services]          [Auth Services]
[Write & Read]        [Write & Read]           [Write & Read]
     ↓                     ↓                        ↓
[User DB Primary] ←→ [User DB Active] ←→ [User DB Active]

Replication Strategy: Active-Active with Conflict Resolution

User Registration:
├─ Each region can write
├─ Global user ID generator (prevent collisions)
├─ Cross-region replication (async, eventual consistency)
└─ CRDTs for conflict-free merges

Token Validation:
├─ All regions publish public keys
├─ JWT validated locally (no cross-region call)
├─ Token revocation: Pub/sub to all regions
└─ Eventual consistency acceptable (worst case: 5 sec delay)

Benefits:
✅ Low latency globally (<50ms)
✅ High availability (multi-region failover)
✅ Regulatory compliance (data stays in region)
✅ Disaster recovery

Challenges:
⚠️ Conflict resolution (concurrent user updates)
⚠️ Operational complexity
⚠️ Cost (3x infrastructure)
⚠️ Cross-region token revocation delay
```

---

### 🔴 For Advanced: Production Architecture Patterns

#### Handling Distributed Authentication State

**Problem: Token Revocation in Multi-Region**

```text
SCENARIO:
User logs in US region
  → Gets JWT (expires in 1 hour)
  → Travels to EU region
  → User's account compromised!
  → Admin revokes token in US
  → How does EU know token is revoked?

Challenge: JWT is stateless (no database lookup!)

SOLUTIONS:

Solution 1: Token Blacklist with Pub/Sub

Architecture:
[US Region]                  [EU Region]
     ↓                           ↓
[Token Revoked] → [Pub/Sub] → [Subscribe to revocations]
     ↓                           ↓
[Add to Redis]              [Add to Redis]

Implementation:
# When token revoked (any region)
redis.sadd("token_blacklist", token_id)
redis.expire(f"token_blacklist:{token_id}", token_ttl)
pubsub.publish("token_revoked", {
  "token_id": token_id,
  "timestamp": now()
})

# Validation (all regions)
def validate_token(token):
    # 1. Verify JWT signature (fast, local)
    decoded = jwt.verify(token, public_key)
    
    # 2. Check blacklist (Redis, ~1ms)
    if redis.sismember("token_blacklist", decoded.jti):
        raise TokenRevoked()
    
    return decoded

Pros:
✅ Fast revocation propagation (<5 seconds)
✅ Minimal storage (only revoked tokens)
✅ Works with JWT (stateless validation)

Cons:
❌ Requires Redis pub/sub infrastructure
❌ Network partition can delay revocation
❌ Need to store until token expires

---

Solution 2: Short-lived JWT + Refresh Token Pattern

Architecture:
[Access Token: 15 min, stateless] ← Fast validation
[Refresh Token: 30 days, stateful] ← Can be revoked

Flow:
1. User gets both tokens
2. Access token used for API calls (no DB lookup)
3. Access token expires after 15 min
4. Client uses refresh token to get new access token
5. Server checks refresh token in DB (can be revoked)

Implementation:
# Issue tokens
access_token = jwt.encode({
  'user_id': user.id,
  'exp': now() + timedelta(minutes=15),
  'jti': generate_uuid()
}, private_key)

refresh_token = generate_secure_random(32)
redis.setex(
  f"refresh:{refresh_token}",
  2592000,  # 30 days
  json.dumps({'user_id': user.id, 'issued_at': now()})
)

# Refresh flow
def refresh_access_token(refresh_token):
    # Check if refresh token valid (can be revoked!)
    data = redis.get(f"refresh:{refresh_token}")
    if not data:
        raise InvalidRefreshToken()
    
    user = db.get_user(data['user_id'])
    return generate_access_token(user)

Pros:
✅ Limits damage if access token stolen (15 min window)
✅ Can revoke refresh token (immediate effect)
✅ No blacklist needed (access token expires quickly)

Cons:
❌ Client needs refresh logic
❌ More complex implementation
❌ 15 min window if access token stolen

VERDICT: ✅ Use Solution 2 (Industry Standard)
└─ OAuth 2.0 pattern, battle-tested
```

#### Zero-Downtime Key Rotation

```text
CHALLENGE: Rotate JWT signing keys without breaking existing tokens

ANTI-PATTERN: Immediate key rotation
1. Generate new key
2. Replace old key
3. All existing tokens now INVALID! ← Users logged out!

CORRECT PATTERN: Gradual key rotation

Phase 1: Publish new key (Day 0)
{
  "keys": [
    {"kid": "key-2026-01", "..."},  ← Old key (still valid)
    {"kid": "key-2026-02", "..."}   ← New key (published)
  ]
}

Phase 2: Start using new key for signing (Day 1)
├─ New tokens signed with key-2026-02
├─ Old tokens (key-2026-01) still validate
└─ Both keys in JWK Set

Phase 3: Remove old key (Day 7)
├─ All tokens signed with old key expired (7 days > max token TTL)
├─ Safe to remove key-2026-01 from JWK Set
└─ Only key-2026-02 remains

Implementation:
class KeyRotationManager:
    def __init__(self):
        self.keys = self.load_keys()
        self.current_key_id = self.get_current_key_id()
    
    def sign_token(self, payload):
        # Always use current key for signing
        current_key = self.keys[self.current_key_id]
        payload['kid'] = self.current_key_id
        return jwt.encode(payload, current_key.private, algorithm='RS256')
    
    def verify_token(self, token):
        decoded = jwt.decode(token, options={"verify_signature": False})
        key_id = decoded['kid']
        
        # Validate with any key in JWK Set
        if key_id not in self.keys:
            raise InvalidKeyId()
        
        public_key = self.keys[key_id].public
        return jwt.verify(token, public_key)
    
    def rotate_keys(self):
        # 1. Generate new key pair
        new_key = generate_rsa_key_pair()
        new_key_id = f"key-{datetime.now().strftime('%Y-%m-%d')}"
        
        # 2. Add to key store
        self.keys[new_key_id] = new_key
        
        # 3. Publish JWK Set (includes old + new)
        self.publish_jwks()
        
        # 4. Update current key ID
        self.current_key_id = new_key_id
        
        # 5. Schedule old key removal (in 7 days)
        schedule_task(delete_old_keys, delay=timedelta(days=7))

Benefits:
✅ Zero downtime (no users logged out)
✅ Gradual transition (safe)
✅ Can rollback if issues detected
```

#### Performance Optimization: Token Validation at Scale

```text
PROBLEM: Validating 100K tokens/second

INEFFICIENT APPROACH:
For each request:
  1. Fetch public key from database (10-50ms) ← Bottleneck!
  2. Verify JWT signature (1-2ms)
  3. Check token blacklist (1ms)

Database can't handle 100K fetches/second!

OPTIMIZED APPROACH: Multi-Level Caching

Level 1: Application Memory Cache (L1)
├─ Store public keys in memory
├─ Update every 5 minutes
├─ Latency: 0.001ms (instant)
└─ Hit rate: 99.99%

Level 2: Redis Cache (L2)
├─ Store JWK Set in Redis
├─ TTL: 5 minutes
├─ Latency: 1ms
└─ Hit rate: 99.9% (if L1 miss)

Level 3: Database (L3)
├─ Fetch from database only if L1 and L2 miss
├─ Latency: 10-50ms
└─ Hit rate: 0.1%

Implementation (Node.js):
const NodeCache = require('node-cache');
const l1Cache = new NodeCache({ stdTTL: 300 });  // 5 min

async function getPublicKey(keyId) {
  // L1: Memory cache
  let key = l1Cache.get(keyId);
  if (key) return key;
  
  // L2: Redis cache
  key = await redis.get(`jwk:${keyId}`);
  if (key) {
    l1Cache.set(keyId, key);
    return key;
  }
  
  // L3: Database (rare)
  key = await db.query('SELECT public_key FROM keys WHERE id = ?', keyId);
  await redis.setex(`jwk:${keyId}`, 300, key);
  l1Cache.set(keyId, key);
  return key;
}

Performance Results:
├─ Before: 100K DB queries/sec (database overloaded)
├─ After: 10 DB queries/sec (0.01% miss rate)
├─ Latency: 0.001ms average (memory cache)
└─ Can handle 500K validations/second per server!
```

#### Disaster Recovery Strategy

```text
RTO and RPO Definitions for Auth Systems:

RPO (Recovery Point Objective):
└─ "How much auth data can we afford to lose?"
└─ For auth system: < 5 minutes of data
└─ Why: Can re-authenticate users, but lose recent registrations

RTO (Recovery Time Objective):
└─ "How quickly must we recover?"
└─ For auth system: < 5 minutes
└─ Why: Nobody can login during outage (business critical!)

Backup Strategy:

TIER 1: Real-time Replication (RTO: 0, RPO: 0)
├─ Database replicas in same region
├─ Automatic failover (< 30 seconds)
├─ Cost: 2x database costs
└─ Handles: Server failures

TIER 2: Cross-Region Replication (RTO: 2 min, RPO: 1 min)
├─ Async replication to other regions
├─ Automatic failover with Route53/Traffic Manager
├─ Cost: 3x database costs
└─ Handles: Regional outages

TIER 3: Backup to Object Storage (RTO: 2 hours, RPO: 1 hour)
├─ Hourly snapshots to S3/GCS
├─ Automated backup process
├─ Cost: $0.004/GB/month
└─ Handles: Complete disaster (ransomware, deletion)

TIER 4: Cold Backup to Different Cloud (RTO: 1 day, RPO: 24 hours)
├─ Daily backups to different provider
├─ Manual restore process
├─ Cost: Minimal
└─ Handles: Cloud provider total failure

Disaster Recovery Drill (Run Quarterly):
1. Simulate primary region failure
2. Failover to secondary region
3. Verify all auth flows work
4. Measure actual RTO/RPO
5. Document issues and improve
```

#### Advanced Security Pattern: Anomaly Detection

```text
PATTERN: ML-based Authentication Anomaly Detection

Challenge:
├─ User logs in from unusual location
├─ User logs in at unusual time
├─ Unusual device or browser
├─ Too many failed attempts from IP
└─ Should we block? Challenge? Allow?

Architecture:

[Authentication Request]
     ↓
[Auth Service] ─────→ [Anomaly Detection Service]
     ↓                          ↓
[Feature Extraction]    [ML Model Inference]
     ↓                          ↓
[Risk Score: 0-100]    [Decision Engine]
     ↓                          ↓
[0-30: Allow]          [Action: MFA Required]
[31-70: Challenge]
[71-100: Block]

Features Extracted:
1. Location-based:
   ├─ IP geolocation
   ├─ Distance from previous login
   └─ VPN/Proxy detection

2. Temporal:
   ├─ Time of day (unusual hours?)
   ├─ Time since last login
   └─ Login frequency

3. Device-based:
   ├─ User agent (new device?)
   ├─ Device fingerprint
   └─ Browser features

4. Behavioral:
   ├─ Failed login attempts
   ├─ Password reset requests
   └─ Account changes

Implementation:
class AnomalyDetector:
    def __init__(self):
        self.model = load_ml_model('auth_anomaly_model.pkl')
    
    def evaluate_risk(self, auth_request, user_history):
        features = {
            'location_change_km': calculate_distance(
                auth_request.ip_location,
                user_history.last_login_location
            ),
            'time_since_last_login_hours': (
                now() - user_history.last_login_time
            ).total_seconds() / 3600,
            'is_new_device': auth_request.device_id not in user_history.devices,
            'failed_attempts_last_hour': redis.get(
                f"failed_attempts:{user.id}"
            ) or 0,
            'is_vpn': check_vpn(auth_request.ip),
            'login_hour': now().hour,
            # ... more features
        }
        
        risk_score = self.model.predict_proba(features)[0][1] * 100
        
        return {
            'risk_score': risk_score,
            'action': self.get_action(risk_score),
            'reasons': self.explain_risk(features, risk_score)
        }
    
    def get_action(self, risk_score):
        if risk_score < 30:
            return 'allow'
        elif risk_score < 70:
            return 'challenge_mfa'  # Require MFA
        else:
            return 'block_and_notify'  # Block + email user

Production Results:
├─ 95% of legitimate logins: No additional friction
├─ 4% of legitimate logins: MFA challenge
├─ 1% of logins: Blocked (99% were actual attacks)
├─ Reduced account takeovers by 87%
└─ False positive rate: <0.1%
```

---

### Real-World Example: Auth0's Architecture Evolution

**2013 - Launch (Simple Auth Service):**
```text
Architecture:
├─ Single Node.js server
├─ Single MongoDB database
├─ Basic username/password auth
└─ Supports OAuth 2.0

Scale:
├─ 100 customers
├─ 10K authentications/day
└─ Cost: $200/month

Simple but worked!
```

**2015 - Growing Pains:**
```text
Problems:
❌ Database becoming bottleneck
❌ No multi-region support (high latency for EU)
❌ Limited protocols (no SAML)

Changes Made:
├─ Migrated to PostgreSQL (better reliability)
├─ Added Redis for token caching
├─ Implemented read replicas (5)
├─ Added SAML and LDAP support
├─ Deployed to 3 regions (US, EU, APAC)
└─ Introduced rate limiting

Scale:
├─ 1,000 customers
├─ 100M authentications/day
└─ Cost: $10,000/month

Key Learning: Multi-tenancy is hard!
```

**2018 - Enterprise Focus:**
```text
New Requirements:
├─ Enterprise customers need 99.99% SLA
├─ Compliance (SOC 2, ISO 27001, GDPR)
├─ Custom domains (auth.customer.com)
├─ Advanced MFA (biometric, hardware keys)

Architecture Changes:
├─ Kubernetes for orchestration
├─ Service mesh (Istio) for mTLS
├─ Multi-region active-active
├─ Separated tenant data (data isolation)
├─ Added anomaly detection (ML)
├─ Implemented log streaming (real-time)
└─ Built custom CDN for token validation

Scale:
├─ 10,000 customers
├─ 5B authentications/day
└─ Cost: $200,000/month

Key Metrics:
├─ P99 login latency: 180ms
├─ P99 token validation: 12ms
├─ Cache hit ratio: 98%
└─ Uptime: 99.98%
```

**2024 - AI-Powered Security:**
```text
Current Architecture:
├─ 20+ regions globally
├─ 50,000+ tenants
├─ Kubernetes (10,000+ pods)
├─ Database: PostgreSQL (sharded, 100+ nodes)
├─ Cache: Redis (500+ nodes)
├─ Message Queue: Kafka (handling 1M messages/sec)
├─ ML models for fraud detection
├─ Passwordless authentication (WebAuthn)
└─ 99.99% uptime SLA met

Scale:
├─ 15B authentications/day
├─ 500M token validations/second
└─ Cost: $2M/month

Key Features:
├─ Attack protection (blocked 10B malicious attempts/year)
├─ Breached password detection (prevented 50M compromises)
├─ Adaptive MFA (risk-based)
├─ Biometric authentication
└─ Passwordless authentication

By The Numbers:
├─ 2013: 1 server → 2024: 10,000+ servers
├─ 2013: 10K auths/day → 2024: 15B auths/day
├─ 2013: 500ms latency → 2024: 12ms latency
└─ Architecture evolved based on customer needs!
```

---

### 🤔 Think About It

1. **For Beginners:** If you could only add ONE security component to improve a simple username/password authentication system, what would it be and why? (Hint: Think about the most common attack vector)

2. **For Intermediate:** You're in an interview and the interviewer says "Your architecture uses JWT tokens. How would you handle a scenario where a user's JWT is stolen? Walk me through your response strategy." What would you propose?

3. **For Advanced:** You wake up at 3 AM to a security alert: "Suspicious authentication spike from Asia-Pacific region - 50K failed login attempts in 5 minutes targeting admin accounts." Walk through your incident response. What's happening? How do you mitigate? How do you prevent future attacks?

---

### ✅ Key Takeaways

- **Security first, always**: Every architectural decision must consider security implications
- **Stateless vs Stateful trade-offs**: JWT for performance, sessions for immediate revocation
- **Defense in depth**: Multiple security layers (rate limiting, MFA, anomaly detection)
- **Token lifecycle management**: Short-lived access tokens + long-lived refresh tokens
- **Separation of concerns**: Auth service ≠ AuthZ service (different scale, different concerns)
- **Multi-region complexity**: Token revocation across regions requires pub/sub or short TTL
- **Caching is critical**: 98%+ token validation should hit cache (sub-millisecond latency)
- **Monitoring and auditing**: Every auth event must be logged (compliance and security)
- **Gradual key rotation**: Never rotate keys immediately (zero-downtime pattern)
- **Real-world matters**: Start simple, add complexity based on actual security needs

---

### 🎯 Practice Exercise

**Scenario:** You're the security architect for "SecureApp," a B2B SaaS platform. Your CTO gives you these requirements:

**Given:**
- Expected: 10,000 business customers
- Expected: 500,000 end users (employees of customer companies)
- Expected: 2M authentication requests per day
- Expected: 50M authorization checks per day (API calls)
- Requirement: Support SSO (SAML, OAuth 2.0, OIDC)
- Requirement: 99.99% uptime SLA
- Requirement: SOC 2 and GDPR compliance
- Requirement: Multi-factor authentication (TOTP, SMS, Email)
- Budget: $15,000/month for infrastructure
- Team: 3 backend developers, 1 security engineer
- Timeline: Launch MVP in 3 months

**Your Task:**

1. **Architecture Design:**
   - Draw a high-level architecture diagram
   - Which components are essential vs nice-to-have?
   - How do you separate authentication from authorization?
   - Where do you store credentials vs tokens vs sessions?

2. **Security Design:**
   - How do you protect against brute force attacks?
   - How do you handle token revocation?
   - What's your MFA strategy?
   - How do you implement audit logging?

3. **Technology Choices:**
   - What database for user credentials? Why?
   - What cache for tokens? Why?
   - JWT vs session tokens? Or hybrid?
   - Which OAuth library/framework?

4. **Cost Breakdown:**
   - Estimate monthly costs for:
     - Compute (auth services)
     - Database (credentials + sessions)
     - Cache (Redis)
     - Message queue (audit logs)
     - External services (Twilio for SMS MFA)
   - Can you stay under $15,000/month?

5. **Compliance Strategy:**
   - How do you implement audit logging (SOC 2)?
   - How do you handle data residency (GDPR)?
   - What's your encryption strategy (at rest, in transit)?

6. **Scaling Plan:**
   - At what traffic level would your architecture need upgrades?
   - What would be the first bottleneck?
   - How would you monitor for security issues?

**Bonus Challenge:** 
One of your customers (a Fortune 500 company) requires:
- On-premise identity provider integration (LDAP)
- Custom domain (auth.customer.com)
- 99.999% uptime SLA (5 nines!)
- Data must stay in EU region (regulatory requirement)

How does this change your architecture? What additional components do you need?

**Discussion Points:**
- How does your architecture differ from Auth0's current architecture?
- What security trade-offs did you make given the constraints?
- How would you justify your decisions to the CTO?
- What would you defer to v2?

---

## Section 4: Authentication Protocols Deep Dive

### What You'll Learn

By the end of this section, you'll be able to:
- Understand and compare OAuth 2.0, OpenID Connect, SAML, and JWT
- Explain different OAuth 2.0 flows and when to use each
- Implement secure token-based authentication
- Choose the right protocol for different use cases
- Design authentication flows that meet security requirements

### Why This Matters

Modern authentication isn't just username and password anymore. Applications need to integrate with third parties, support Single Sign-On, and enable secure API access. Real-world example: When you click "Sign in with Google," you're using OAuth 2.0 and OpenID Connect. Understanding these protocols is essential for any authentication system!

---

### 🟢 For Beginners: Understanding Auth Protocols

#### What is a Protocol?

Think of a protocol like a recipe or set of rules that everyone agrees to follow. When baking a cake, you follow steps: mix ingredients, bake at 350°F for 30 minutes, let cool. Authentication protocols are similar - they're agreed-upon steps for proving identity safely.

**Why Do We Need Standard Protocols?**

Imagine if every website had its own way of handling "Sign in with Google." Chaos! Protocols ensure everyone does it the same secure way.

#### Session-Based vs Token-Based Authentication

**Session-Based (Traditional):**

```text
Analogy: Hotel wristband at an all-inclusive resort

1. Check-in (Login): Show ID, get wristband
2. Use Services: Show wristband to access pool, buffet, gym
3. Server Stores: Hotel computer remembers your wristband = Room 305
4. Check-out (Logout): Return wristband, hotel forgets you
```

**How it works:**
- User logs in with credentials
- Server creates session ID and stores user info in memory/database
- Server sends session ID as cookie to browser
- Browser sends cookie with every request
- Server looks up session to identify user

**Pros:**
- Easy to revoke (just delete session from server)
- Can store lots of user data server-side
- Familiar pattern

**Cons:**
- Server must store all sessions (memory/database)
- Hard to scale across multiple servers
- Doesn't work well for mobile apps or APIs

**Token-Based (Modern):**

```text
Analogy: Driver's license

1. Get License: Pass test, DMV issues license with your photo & info
2. Use Anywhere: Show license at bank, airport, bar - they trust it
3. Self-Contained: All info is on the license itself
4. No Central Database: Each place validates the license independently
```

**How it works:**
- User logs in with credentials
- Server creates JWT (JSON Web Token) containing user info
- Server signs token with secret key
- Client stores token (localStorage, memory)
- Client sends token in Authorization header
- Server validates signature and extracts user info from token

**Pros:**
- Stateless (server doesn't store anything)
- Scales easily across many servers
- Works for web, mobile, APIs
- Can be validated without database call

**Cons:**
- Harder to revoke before expiration
- Token size can be large
- Need careful security handling

#### JWT (JSON Web Token) - The Popular Choice

A JWT is like a sealed envelope with a transparent window. You can see what's inside, but you can't change it without breaking the seal.

**JWT Structure:**

```text
header.payload.signature

Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Three Parts:**

1. **Header** (Algorithm info):
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

2. **Payload** (User data):
```json
{
  "sub": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin",
  "exp": 1735689600
}
```

3. **Signature** (Proof it's real):
```text
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret_key
)
```

**Why JWT is Secure:**
- If anyone changes the payload, the signature won't match
- Only the server with the secret key can create valid signatures
- Like a wax seal on a letter - you can see if it's been tampered with

#### OAuth 2.0 - Letting Apps Act On Your Behalf

**Real-World Analogy:**

You want a house cleaning service to clean your home, but you don't want to give them a copy of your house key forever. Instead:
- You give them a temporary access code that works only this week
- The code only unlocks the front door, not your safe
- You can cancel the code anytime
- The code expires automatically after a week

OAuth 2.0 does the same for apps!

**Common Example: "Sign in with Google"**

```text
Step 1: You click "Sign in with Google" on CoolApp
Step 2: CoolApp redirects you to Google's login page
Step 3: You log in to Google (proving your identity to Google)
Step 4: Google asks: "CoolApp wants to access your profile and email. Allow?"
Step 5: You click "Allow"
Step 6: Google gives CoolApp a temporary access token
Step 7: CoolApp uses token to get your name and email from Google
Step 8: CoolApp creates your account and logs you in
```

**Key Benefit:** CoolApp never sees your Google password!

#### OpenID Connect (OIDC) - OAuth's Identity Layer

OAuth 2.0 is for authorization ("Can I access your photos?"), but OpenID Connect adds authentication ("Who are you?").

**Extension to OAuth 2.0:**
- OAuth 2.0: Access Token (for accessing APIs)
- OIDC: Access Token + ID Token (for identifying the user)

**ID Token contains:**
- User's unique identifier
- User's name and email
- When token was issued
- When it expires

#### SAML 2.0 - Enterprise SSO

SAML (Security Assertion Markup Language) is the "old reliable" protocol that enterprises love. It's more complex than OAuth but very powerful for corporate environments.

**Real-World Analogy:**

Your company badge lets you access the building, parking garage, gym, and cafeteria. You authenticate once (badge scan at entrance), and it works everywhere.

**Common Use Case: Company SSO**

```text
Employee tries to access Company App
↓
Company App redirects to Company Identity Provider (Okta/Azure AD)
↓
Employee logs in once to Identity Provider
↓
Identity Provider sends SAML Assertion (like a signed certificate)
↓
Company App trusts the assertion and logs employee in
↓
Employee can now access all company apps without re-logging in
```

**Why Enterprises Love SAML:**
- Centralized user management
- Single login for all apps
- Strong security with digital signatures
- Audit trail of all access

---

### 🟡 For Intermediate: Protocol Details

#### OAuth 2.0 Flows Comparison

OAuth 2.0 has different "flows" (recipes) for different situations:

**1. Authorization Code Flow (Most Common)**

```mermaid
sequenceDiagram
    participant User
    participant App
    participant AuthServer
    participant ResourceServer
    
    User->>App: 1. Click "Login with Google"
    App->>AuthServer: 2. Redirect to authorization page
    AuthServer->>User: 3. Show login & consent screen
    User->>AuthServer: 4. Login & approve
    AuthServer->>App: 5. Redirect with auth code
    App->>AuthServer: 6. Exchange code for tokens (with client secret)
    AuthServer->>App: 7. Return access token + refresh token
    App->>ResourceServer: 8. Call API with access token
    ResourceServer->>App: 9. Return protected data
```

**When to use:** 
- Web applications with a backend server
- Mobile apps with PKCE extension
- Highest security (client secret stored on server)

**Security:** ⭐⭐⭐⭐⭐ (Best)

**2. Client Credentials Flow**

```text
Service-to-Service Authentication

Step 1: Service A has client_id and client_secret
Step 2: Service A sends credentials to Auth Server
Step 3: Auth Server validates and returns access token
Step 4: Service A uses token to call Service B's API
```

**When to use:**
- Backend services talking to each other
- No user involved (machine-to-machine)
- Batch jobs, scheduled tasks

**Security:** ⭐⭐⭐⭐ (Very Good for M2M)

**3. PKCE (Proof Key for Code Exchange)**

Extension to Authorization Code Flow for public clients (mobile/SPA):

```text
Challenge: Mobile apps can't keep secrets (decompiled easily)

Solution: Use a one-time secret for each login

Step 1: App generates random code_verifier
Step 2: App creates code_challenge = SHA256(code_verifier)
Step 3: App sends code_challenge when requesting auth code
Step 4: Auth Server stores code_challenge
Step 5: App receives auth code
Step 6: App exchanges code + code_verifier for token
Step 7: Auth Server verifies: SHA256(code_verifier) == stored code_challenge
Step 8: If match, return access token
```

**When to use:**
- Mobile apps (iOS, Android)
- Single Page Applications (React, Vue, Angular)
- Any public client that can't store secrets

**Security:** ⭐⭐⭐⭐ (Good for public clients)

**Deprecated/Insecure Flows (DON'T USE):**

❌ **Implicit Flow:** Returns token directly in URL (vulnerable to XSS)
❌ **Resource Owner Password Flow:** App handles user's password (defeats purpose of OAuth)

#### JWT Deep Dive

**JWT Claims (Standard Fields):**

```json
{
  "iss": "https://auth.example.com",     // Issuer - who created token
  "sub": "user123",                       // Subject - who token is about
  "aud": "https://api.example.com",       // Audience - who should accept it
  "exp": 1735689600,                      // Expiration - when it expires (Unix timestamp)
  "nbf": 1735686000,                      // Not Before - when it becomes valid
  "iat": 1735686000,                      // Issued At - when it was created
  "jti": "abc-def-ghi",                   // JWT ID - unique identifier
  
  // Custom claims
  "email": "user@example.com",
  "roles": ["user", "admin"],
  "permissions": ["read:posts", "write:posts"]
}
```

**Token Validation Checklist:**

```text
✓ 1. Check signature is valid (token not tampered with)
✓ 2. Verify exp claim (token not expired)
✓ 3. Verify iss claim (token from trusted issuer)
✓ 4. Verify aud claim (token intended for this API)
✓ 5. Verify nbf claim if present (token is valid now)
✓ 6. Check token not in revocation list (if using one)
```

**JWT Security Best Practices:**

| Practice | Why | How |
|----------|-----|-----|
| Short expiration | Limit damage if stolen | 15 min for access tokens |
| Use refresh tokens | User stays logged in | Refresh token valid 7-30 days |
| Signature algorithm | Prevent tampering | RS256 (asymmetric) or HS256 (symmetric) |
| HTTPS only | Prevent interception | TLS 1.3 |
| Don't store sensitive data | JWT can be decoded | No passwords, SSN, credit cards |
| Validate everything | Defense in depth | Check all claims |

#### OpenID Connect (OIDC) Flow

OIDC extends OAuth 2.0 by adding an ID Token:

```text
Standard OAuth 2.0:
User -> Authorization Code -> Access Token -> Call API

OpenID Connect:
User -> Authorization Code -> Access Token + ID Token -> Call API + Know User Identity
```

**ID Token (JWT) Example:**

```json
{
  "iss": "https://accounts.google.com",
  "sub": "10769150350006150715113082367",
  "aud": "your-app-client-id",
  "exp": 1735689600,
  "iat": 1735686000,
  "auth_time": 1735685900,
  
  // Standard OpenID Claims
  "name": "John Doe",
  "given_name": "John",
  "family_name": "Doe",
  "email": "john.doe@example.com",
  "email_verified": true,
  "picture": "https://example.com/photo.jpg",
  "locale": "en-US"
}
```

**OIDC Endpoints:**

1. **Authorization Endpoint:** Start the login flow
```text
GET /authorize?
  response_type=code&
  client_id=your_app_id&
  redirect_uri=https://yourapp.com/callback&
  scope=openid profile email&
  state=random_string
```

2. **Token Endpoint:** Exchange code for tokens
```text
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=auth_code_here&
redirect_uri=https://yourapp.com/callback&
client_id=your_app_id&
client_secret=your_app_secret
```

3. **UserInfo Endpoint:** Get additional user claims
```text
GET /userinfo
Authorization: Bearer access_token_here
```

#### SAML 2.0 Flow Details

**SAML Components:**

- **Identity Provider (IdP):** Authenticates users (Okta, Azure AD, Google Workspace)
- **Service Provider (SP):** Your application that trusts the IdP
- **SAML Assertion:** Signed XML document proving user identity

**SP-Initiated SSO Flow:**

```text
1. User accesses Service Provider (your app)
2. SP generates SAML AuthnRequest
3. SP redirects user to IdP with AuthnRequest
4. IdP authenticates user (if not already logged in)
5. IdP generates SAML Response with signed assertion
6. IdP redirects user back to SP with SAML Response
7. SP validates signature and assertion
8. SP creates session for user
```

**SAML Assertion Example (Simplified):**

```xml
<saml:Assertion>
  <saml:Issuer>https://idp.example.com</saml:Issuer>
  
  <saml:Subject>
    <saml:NameID>john.doe@company.com</saml:NameID>
  </saml:Subject>
  
  <saml:Conditions>
    <saml:NotBefore>2026-01-22T08:00:00Z</saml:NotBefore>
    <saml:NotOnOrAfter>2026-01-22T09:00:00Z</saml:NotOnOrAfter>
    <saml:AudienceRestriction>
      <saml:Audience>https://yourapp.com</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
  
  <saml:AttributeStatement>
    <saml:Attribute Name="email">
      <saml:AttributeValue>john.doe@company.com</saml:AttributeValue>
    </saml:Attribute>
    <saml:Attribute Name="department">
      <saml:AttributeValue>Engineering</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
  
  <ds:Signature>... digital signature ...</ds:Signature>
</saml:Assertion>
```

**OAuth vs SAML - When to Use:**

| Criterion | OAuth 2.0/OIDC | SAML 2.0 |
|-----------|----------------|----------|
| **Age** | Modern (2012+) | Older (2005) |
| **Format** | JSON (JWT) | XML |
| **Use Case** | API access, social login, mobile | Enterprise SSO, legacy apps |
| **Complexity** | Simple | Complex |
| **Mobile Support** | Excellent | Poor |
| **Enterprise Adoption** | Growing | Dominant |
| **Best For** | New apps, APIs, consumer apps | Enterprise apps, B2B |

**Protocol Selection Decision Tree:**

```text
Need to integrate with third-party APIs? 
  → OAuth 2.0 + OIDC

Building enterprise SSO for internal apps?
  → SAML 2.0 (unless starting fresh, then OIDC)

Building consumer-facing app with social login?
  → OAuth 2.0 + OIDC

Service-to-service authentication?
  → OAuth 2.0 Client Credentials + JWT

Mobile app authentication?
  → OAuth 2.0 Authorization Code + PKCE

Traditional web app with sessions?
  → Session-based or OAuth 2.0 Authorization Code
```

---

### 🔴 For Advanced: Production Implementation

#### Token Security Patterns

**1. Token Binding**

Cryptographically bind token to TLS connection:

```text
Problem: Stolen token can be used from any device
Solution: Bind token to specific TLS connection

Implementation:
1. During TLS handshake, derive a unique key
2. Include hash of key in JWT
3. Validate token AND TLS binding on each request

Result: Token only works from original device/connection
```

**2. Refresh Token Rotation**

```text
Security Risk: Long-lived refresh tokens are high-value targets

Pattern: Rotate refresh tokens on each use

Flow:
1. Client uses refresh token to get new access token
2. Server issues new access token AND new refresh token
3. Server invalidates old refresh token
4. If old refresh token used again → Breach detected!
   → Revoke entire token family
   → Force re-authentication

Benefit: Limits window for stolen token abuse
```

**3. JWT with Opaque Refresh Tokens**

Best of both worlds hybrid approach:

```text
Access Token: JWT (self-contained, fast validation)
├─ Short-lived (15 minutes)
├─ Can be validated without database
├─ Contains user claims and permissions
└─ Stateless

Refresh Token: Opaque random string (stored in database)
├─ Long-lived (30 days)
├─ Requires database lookup to validate
├─ Can be revoked immediately
└─ Stateful

Benefits:
✓ Fast authorization checks (JWT)
✓ Immediate revocation capability (opaque refresh token)
✓ Limited damage if access token stolen (15 min window)
✓ Can detect token theft (refresh token rotation)
```

#### Key Management for JWT

**Asymmetric Keys (RS256 - Recommended):**

```text
Private Key (kept secret on auth server):
├─ Used to SIGN tokens
├─ Stored in HSM (Hardware Security Module) or secure vault
├─ Never exposed to other services
└─ Rotated every 90 days

Public Key (distributed to all services):
├─ Used to VERIFY token signatures
├─ Can be publicly accessible (JWKS endpoint)
├─ Cached by services for performance
└─ Multiple keys supported (key rotation)

Benefit: Services can validate tokens without talking to auth server
```

**JWKS (JSON Web Key Set) Endpoint:**

```json
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "key-2026-01",
      "alg": "RS256",
      "n": "base64-encoded-modulus...",
      "e": "AQAB"
    },
    {
      "kty": "RSA",
      "use": "sig",
      "kid": "key-2025-12",
      "alg": "RS256",
      "n": "base64-encoded-modulus...",
      "e": "AQAB"
    }
  ]
}
```

**Zero-Downtime Key Rotation:**

```text
Step 1: Generate new key pair (key-2026-02)
Step 2: Add public key to JWKS endpoint
Step 3: Services refresh JWKS cache (within 5 minutes)
Step 4: Start signing new tokens with key-2026-02
Step 5: Keep old key (key-2026-01) for validation (7 days)
Step 6: After 7 days, remove old key from JWKS
Step 7: Old tokens naturally expire

Timeline:
Day 0: Add new key to JWKS
Day 0: Services cache new key
Day 1: Start signing with new key
Day 7: Remove old key from JWKS
Day 8: All old tokens expired

Zero downtime achieved!
```

#### Advanced OAuth 2.0 Patterns

**1. Demonstrating Proof of Possession (DPoP)**

```text
Problem: Bearer tokens can be used by anyone who possesses them

Solution: Bind token to a specific client

How it works:
1. Client generates public/private key pair
2. Client includes public key proof in token request
3. Auth server binds token to that public key
4. When using token, client must sign request with private key
5. Resource server validates both token and signature

Result: Stolen token is useless without private key
```

**2. Pushed Authorization Requests (PAR)**

```text
Problem: Authorization parameters visible in browser URL

Solution: Push parameters to server first, then reference

Flow:
1. Client POSTs authorization request directly to server (backend)
2. Server returns request_uri (opaque identifier)
3. Client redirects user to authorization endpoint with request_uri
4. Server looks up parameters using request_uri
5. Proceeds with authorization

Benefits:
✓ Parameters not visible in browser history
✓ Prevents parameter tampering
✓ Supports large requests (> URL length limit)
```

**3. Rich Authorization Requests (RAR)**

```text
Problem: scope parameter is too simple for fine-grained authorization

Traditional: scope=read:email
RAR: Structured authorization requests

Example:
{
  "authorization_details": [
    {
      "type": "account_information",
      "actions": ["read", "write"],
      "locations": ["https://api.bank.com/accounts/123"],
      "amount": {
        "max": 1000,
        "currency": "USD"
      }
    },
    {
      "type": "payment_initiation",
      "creditor_account": "DE1234567890",
      "amount": 50.00
    }
  ]
}

Use Case: Open Banking, Fine-grained API access
```

#### SAML Security Hardening

**Critical Security Checks:**

```text
1. Signature Validation
   ✓ Verify assertion is signed by trusted IdP
   ✓ Check certificate is not expired
   ✓ Validate certificate chain
   ✓ Use strong algorithms (RSA-SHA256, not RSA-SHA1)

2. Audience Restriction
   ✓ Assertion intended for your SP (prevent token replay)
   ✓ Check AudienceRestriction matches your entity ID

3. Time Validation
   ✓ Current time within NotBefore and NotOnOrAfter
   ✓ Account for clock skew (allow ±5 minutes)
   ✓ Assertion not older than reasonable (< 5 minutes)

4. Recipient Validation
   ✓ Response sent to correct ACS (Assertion Consumer Service) URL
   ✓ Prevents token interception

5. InResponseTo Validation
   ✓ Links response to original request
   ✓ Prevents unsolicited responses
   ✓ Protects against CSRF

6. NameID Format
   ✓ Persistent vs transient identifier
   ✓ Validate format matches expectations

7. Attribute Validation
   ✓ Required attributes present
   ✓ Attribute values in expected format
   ✓ Sanitize values (XSS prevention)
```

**SAML Vulnerabilities to Prevent:**

| Vulnerability | Attack | Prevention |
|---------------|--------|------------|
| **XML Signature Wrapping** | Attacker wraps valid signature around malicious content | Validate signature covers entire assertion |
| **XML External Entity (XXE)** | Inject malicious XML entities | Disable external entities in XML parser |
| **SAML Response Replay** | Reuse valid SAML response | Check assertion ID not seen before, enforce NotOnOrAfter |
| **Token Substitution** | Replace SubjectConfirmationData | Validate InResponseTo matches request |
| **Comment Injection** | Hide malicious code in XML comments | Strip comments before parsing |

#### Protocol Performance Optimization

**JWT Validation Performance:**

```text
Without Optimization: ~50ms
├─ Fetch public key from JWKS endpoint: 30ms
├─ Validate signature: 15ms
└─ Validate claims: 5ms

With Optimization: ~2ms
├─ Use cached public key: 0ms (cached)
├─ Validate signature: 1.5ms
└─ Validate claims: 0.5ms

Optimization Techniques:
1. Cache public keys (5-15 min TTL)
2. Use multiple keys (kid in JWT header selects key)
3. Pre-compute key material
4. Use hardware acceleration for crypto
5. Short-circuit validation on cache hit

Result: 25x performance improvement
```

**SAML Response Caching:**

```text
Problem: SAML XML parsing is slow (50-100ms)

Strategy: Cache parsed and validated assertions

Cache Key: SAML Response ID
Cache Value: {user_id, attributes, expiration}
Cache TTL: Until NotOnOrAfter timestamp

Performance:
First request: 100ms (parse + validate)
Subsequent requests: 1ms (cache hit)
```

---

### 🤔 Think About It

**For Beginners:**
1. Why is OAuth 2.0 more secure than sending username/password to every app?
2. What's the difference between an access token and an ID token?
3. Why does JWT contain an expiration time?

**For Intermediate:**
4. When would you choose SAML over OAuth 2.0?
5. Why is the Authorization Code flow more secure than the Implicit flow?
6. What's the purpose of the state parameter in OAuth?

**For Advanced:**
7. How would you implement token binding for mobile apps?
8. What's the trade-off between JWT expiration time and security?
9. How do you handle key rotation without service disruption?

---

### 📝 Key Takeaways

**Protocol Selection:**
- **OAuth 2.0 + OIDC**: Modern standard for APIs and social login
- **SAML 2.0**: Enterprise SSO for legacy systems
- **JWT**: Stateless token format for scalability
- **Sessions**: Still valid for simple web apps

**Security Best Practices:**
- Use Authorization Code flow with PKCE for public clients
- Implement refresh token rotation
- Short-lived access tokens (15 min)
- Long-lived refresh tokens (7-30 days)
- Always validate all token claims
- Use asymmetric keys (RS256) for JWT

**Performance Optimization:**
- Cache public keys for JWT validation
- Use multiple keys for rotation
- Cache validated tokens/assertions
- Hardware acceleration for crypto operations

**Common Pitfalls:**
- Don't use deprecated flows (Implicit, Password)
- Never put secrets in mobile/SPA code
- Don't skip signature validation
- Don't trust client-provided data in tokens
- Always use HTTPS

---

### 💪 Practice Exercise

**Scenario:** You're designing authentication for a new mobile banking app.

**Requirements:**
- Secure login with username and password
- Support biometric authentication (Face ID/fingerprint)
- Enable "Log in from another device" with QR code
- Integrate with bank's existing SSO (SAML)
- Allow third-party apps to access account balance (with user consent)

**Your Task:**
1. Which protocols would you use for each feature?
2. Design the OAuth 2.0 flow for third-party app access
3. How do you handle the QR code login flow?
4. How long should access tokens and refresh tokens last?
5. What additional security measures would you add?

**Bonus Challenge:**
- User reports: "I lost my phone, please revoke all sessions"
- How do you implement this with token-based auth?
- What's the trade-off between immediate revocation and performance?

---


## Section 5: Authorization Models

### What You'll Learn

By the end of this section, you'll be able to:
- Understand different authorization models (RBAC, ABAC, ReBAC, ACL)
- Design permission systems that scale
- Implement fine-grained access control
- Choose the right model for your use case
- Handle complex authorization scenarios

### Why This Matters

Authorization determines what users can do after they're authenticated. Get it wrong, and users either can't do their jobs (too restrictive) or can access data they shouldn't (security breach). Real-world example: Facebook's Privacy Settings are a complex ReBAC system managing billions of authorization decisions per day!

---

### 🟢 For Beginners: Understanding Authorization

#### What is Authorization?

**Remember:** Authentication = "Who are you?" | Authorization = "What can you do?"

Think of authorization like permissions in your phone:
- Camera app can access camera (allowed)
- Random game can't access contacts (denied)
- You control these permissions in settings

In business applications:
- Employee can view their own salary (allowed)
- Employee can't view CEO's salary (denied)
- HR Manager can view all salaries (allowed)

#### Access Control List (ACL) - The Simplest Model

**Real-World Analogy:** Your phone's contact list privacy settings

```text
Photo Album: "Summer Vacation"
├─ Alice: Can view and edit
├─ Bob: Can view only
├─ Charlie: No access
└─ Everyone else: No access
```

**How it works:**
- Each resource (file, photo, document) has a list
- List says who can access it and what they can do
- Direct mapping: Resource → Users → Permissions

**Example: File Permissions**

```text
Document: "Q4_Financial_Report.pdf"
ACL:
├─ john@company.com: read, write, share
├─ finance-team@company.com: read
└─ ceo@company.com: read, write, delete
```

**Pros:**
- Simple to understand
- Easy to implement
- Fine-grained control per resource

**Cons:**
- Doesn't scale (imagine 1M files × 100 users each)
- Hard to manage (update permissions on thousands of resources?)
- No role abstraction

#### Role-Based Access Control (RBAC) - The Most Popular

**Real-World Analogy:** Job titles in a company

```text
Instead of: "Alice can edit invoices"
Use: "Accountants can edit invoices" + "Alice is an Accountant"
```

**Three Concepts:**

1. **Users:** People using the system (Alice, Bob, Charlie)
2. **Roles:** Job functions (Admin, Editor, Viewer)
3. **Permissions:** Specific actions (create:post, delete:user, read:reports)

**Relationship:**
```text
Users → assigned to → Roles → granted → Permissions
```

**Example: Blog Platform**

```text
Roles:
├─ Admin
│   ├─ create:post
│   ├─ edit:any_post
│   ├─ delete:any_post
│   └─ manage:users
│
├─ Editor
│   ├─ create:post
│   ├─ edit:own_post
│   └─ delete:own_post
│
└─ Viewer
    └─ read:post

Users:
├─ Alice → Admin role
├─ Bob → Editor role
└─ Charlie → Viewer role
```

**How it works:**
1. User logs in (Bob)
2. System loads Bob's roles (Editor)
3. System loads Editor's permissions (create:post, edit:own_post, delete:own_post)
4. Bob tries to delete a post
5. System checks: Does Bob have delete:own_post? Is this his own post? → Allow or Deny

**Pros:**
- Scales well (manage roles, not individual user permissions)
- Easy to understand (roles match real-world jobs)
- Reduces errors (change role once, affects all users)
- Audit-friendly (who has what role?)

**Cons:**
- Role explosion (need specific role for every permission combination)
- Can't handle contextual rules ("only edit your own posts")
- Inflexible for dynamic scenarios

#### Attribute-Based Access Control (ABAC) - The Flexible One

**Real-World Analogy:** Dynamic rules instead of fixed roles

```text
RBAC: "Managers can approve expenses"
ABAC: "Users can approve expenses if:
       - User's department = Expense's department AND
       - Expense amount < User's approval limit AND
       - Current time is business hours"
```

**Four Attributes:**

1. **Subject Attributes:** About the user (department, seniority, clearance level)
2. **Object Attributes:** About the resource (classification, owner, creation date)
3. **Action Attributes:** What they're trying to do (read, write, delete)
4. **Environment Attributes:** Context (time, location, IP address, device security)

**Example: Healthcare System**

```text
Policy: "Allow access to patient record if:
  - User is a Doctor AND
  - User's specialty matches patient's condition AND
  - User is assigned to patient's case AND
  - Access is from hospital network AND
  - Access is during user's shift hours"

Attributes:
User (Dr. Smith):
├─ role: doctor
├─ specialty: cardiology
├─ assigned_patients: [patient_123, patient_456]
└─ shift: 7am-3pm

Patient Record (patient_123):
├─ owner: patient_123
├─ condition: heart_disease
└─ classification: highly_sensitive

Environment:
├─ ip_address: 10.0.hospital.net
└─ current_time: 9:30am

Result: ALLOW (all conditions met)
```

**Pros:**
- Extremely flexible
- Handles complex, dynamic scenarios
- Fine-grained control
- Fewer policies than RBAC roles

**Cons:**
- Complex to design and implement
- Harder to debug ("why was I denied?")
- Performance overhead (evaluate policies on each request)
- Difficult to audit ("who has access?")

#### Relationship-Based Access Control (ReBAC) - For Social Networks

**Real-World Analogy:** Facebook privacy settings

```text
"Friends can see my photos"
"Friends of friends can see my public posts"
"Only me can see my birthdate"
```

**Based on relationships (graph-based):**

```text
Alice --- friend ---> Bob
Alice --- blocks ---> Charlie
Alice --- owns ---> Photo_1

Rules:
- Alice's friends can view Photo_1
- Blocked users can't view Photo_1
```

**Example: Google Docs Sharing**

```text
Relationships:
├─ Document_123 owned_by Alice
├─ Document_123 shared_with Bob (permission: edit)
├─ Document_123 shared_with charlie@company.com (permission: comment)
└─ Document_123 shared_with anyone_at_company.com (permission: view)

Query: "Can Dave (dave@company.com) access Document_123?"
Answer: Yes, via anyone_at_company.com rule (view permission)
```

**Famous Implementation: Google Zanzibar**

Used by Google Drive, Calendar, Cloud, YouTube, etc.

```text
Tuples (relationship facts):
├─ doc:doc_123#owner@alice
├─ doc:doc_123#editor@bob
├─ doc:doc_123#viewer@group:company#member
└─ group:company#member@dave

Query: "Is Dave a viewer of doc_123?"
Path: dave → member of company → viewer of doc_123 → Yes!
```

**Pros:**
- Natural for social and collaborative apps
- Handles inheritance (groups, organizations)
- Flexible sharing models
- Scales horizontally

**Cons:**
- Complex graph traversal
- Consistency challenges (eventual consistency)
- Harder to reason about than RBAC
- Requires specialized database (graph DB or specialized system)

---

### 🟡 For Intermediate: Implementation Patterns

#### RBAC Implementation

**Database Schema:**

```sql
-- Users table
CREATE TABLE users (
  id BIGINT PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Roles table
CREATE TABLE roles (
  id BIGINT PRIMARY KEY,
  name VARCHAR(50) UNIQUE NOT NULL,
  description TEXT
);

-- Permissions table
CREATE TABLE permissions (
  id BIGINT PRIMARY KEY,
  resource VARCHAR(50) NOT NULL,  -- e.g., 'post', 'user'
  action VARCHAR(50) NOT NULL,    -- e.g., 'create', 'read', 'update', 'delete'
  UNIQUE(resource, action)
);

-- User-Role assignment (many-to-many)
CREATE TABLE user_roles (
  user_id BIGINT REFERENCES users(id),
  role_id BIGINT REFERENCES roles(id),
  PRIMARY KEY (user_id, role_id)
);

-- Role-Permission assignment (many-to-many)
CREATE TABLE role_permissions (
  role_id BIGINT REFERENCES roles(id),
  permission_id BIGINT REFERENCES permissions(id),
  PRIMARY KEY (role_id, permission_id)
);

-- Indexes for fast lookups
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_role_permissions_role_id ON role_permissions(role_id);
```

**Permission Check Algorithm:**

```text
Input: user_id, required_permission (e.g., "update:post")

Step 1: Get user's roles
SELECT role_id FROM user_roles WHERE user_id = ?

Step 2: Get permissions for those roles
SELECT p.resource, p.action 
FROM permissions p
JOIN role_permissions rp ON p.id = rp.permission_id
WHERE rp.role_id IN (user_roles)

Step 3: Check if required_permission is in the list
IF "update:post" IN permissions THEN
  ALLOW
ELSE
  DENY
END IF

Optimization: Cache this for 5-15 minutes
```

**Role Hierarchy:**

```text
Admin (inherits from Editor)
  ├─ All Editor permissions
  ├─ Plus: manage:users, delete:any_post
  
Editor (inherits from Viewer)
  ├─ All Viewer permissions
  ├─ Plus: create:post, update:own_post
  
Viewer (base role)
  └─ read:post
```

**Database Schema for Hierarchy:**

```sql
CREATE TABLE role_hierarchy (
  parent_role_id BIGINT REFERENCES roles(id),
  child_role_id BIGINT REFERENCES roles(id),
  PRIMARY KEY (parent_role_id, child_role_id)
);

-- Admin inherits from Editor
INSERT INTO role_hierarchy VALUES (admin_role_id, editor_role_id);
-- Editor inherits from Viewer
INSERT INTO role_hierarchy VALUES (editor_role_id, viewer_role_id);
```

#### ABAC Policy Language

**Example: AWS IAM Policy Format**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::company-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": ["10.0.0.0/16"]
        },
        "DateGreaterThan": {
          "aws:CurrentTime": "2026-01-01T00:00:00Z"
        },
        "DateLessThan": {
          "aws:CurrentTime": "2026-12-31T23:59:59Z"
        },
        "StringEquals": {
          "aws:PrincipalOrgID": "o-123456789"
        }
      }
    }
  ]
}
```

**Policy Decision Point (PDP):**

```text
Request: {
  subject: {user_id: "user123", department: "engineering", level: "senior"},
  action: "read",
  resource: {document_id: "doc456", classification: "confidential", owner: "user789"},
  environment: {time: "2026-01-22T14:30:00Z", ip: "10.0.1.50"}
}

Policy:
IF subject.department == resource.owner.department AND
   subject.level IN ["senior", "lead", "manager"] AND
   resource.classification != "top_secret" AND
   environment.ip STARTS_WITH "10.0."
THEN ALLOW
ELSE DENY

Evaluation:
✓ Same department: True (both engineering)
✓ Appropriate level: True (senior)
✓ Not too sensitive: True (confidential < top_secret)
✓ Internal network: True (10.0.1.50)
Result: ALLOW
```

#### ReBAC with Zanzibar Pattern

**Tuple Format:**

```text
<namespace:object_id#relation@subject_id>

Examples:
doc:doc_123#owner@user:alice
doc:doc_123#viewer@user:bob
doc:doc_123#viewer@group:engineering#member
group:engineering#member@user:charlie
```

**Relation Definitions:**

```text
namespace doc {
  relation owner: user
  relation editor: user
  relation viewer: user | group#member
  
  permission can_view = viewer | editor | owner
  permission can_edit = editor | owner
  permission can_delete = owner
  permission can_share = owner
}

namespace group {
  relation member: user
  relation admin: user
  
  permission can_invite = admin
}
```

**Check Algorithm (Simplified):**

```text
Check: Can user:charlie view doc:doc_123?

Step 1: Expand doc:doc_123#viewer
  ├─ Direct: user:bob
  └─ Via group: group:engineering#member

Step 2: Check if charlie is in group:engineering#member
  ├─ Query: group:engineering#member@user:charlie?
  └─ Result: Yes (tuple exists)

Step 3: Permission granted through group membership

Answer: Yes, charlie can view doc_123
```

**Caching Strategy:**

```text
Cache positive results for 5 minutes
Cache negative results for 1 minute
Invalidate on relation change

Example:
1. Check: Can Alice edit Doc1? → Query DB → Yes → Cache 5min
2. Check: Can Alice edit Doc1? → Cache hit → Yes (instant)
3. Admin removes Alice's editor role → Invalidate cache
4. Check: Can Alice edit Doc1? → Query DB → No
```

---

### 🔴 For Advanced: Enterprise Patterns

#### Combining Models (Hybrid Approach)

Most real systems use a combination:

```text
Layer 1: RBAC for coarse-grained permissions
├─ Admin role: Access to admin panel
├─ User role: Access to user features

Layer 2: ABAC for fine-grained rules
├─ Can edit document if owner OR in shared_editors
├─ Can approve expense if amount < approval_limit

Layer 3: ReBAC for sharing and collaboration
├─ Can view file if in shared_with relationship
├─ Transitive permissions through group membership

Decision: Evaluate all three, must pass all applicable checks
```

**Example: Enterprise Document System**

```text
Alice wants to edit Document_123

Check 1 (RBAC): Does Alice have edit:documents permission?
├─ Alice has "Employee" role
├─ Employee role has "edit:documents"
└─ ✓ Pass

Check 2 (ABAC): Does Alice meet contextual requirements?
├─ Document classification: Internal (OK for Employee)
├─ Access from: Corporate network (OK)
├─ Time: Business hours (OK)
└─ ✓ Pass

Check 3 (ReBAC): Does Alice have relationship to document?
├─ Document owned_by Bob
├─ Document shared_with Alice (editor permission)
└─ ✓ Pass

Result: ALLOW (all three checks passed)
```

#### Permission at Scale: Patterns

**1. Lazy Evaluation**

```text
Problem: Evaluating all permissions is slow

Solution: Check permissions only when needed

Anti-pattern:
Load all user permissions at login → Store in JWT → 10KB token

Better:
Store only user_id and roles in JWT → Check specific permission on-demand

Example:
JWT contains: {user_id, roles: ["editor"]}
User tries to delete post → Check "delete:post" permission → Query cache/DB
```

**2. Permission Aggregation**

```text
Problem: User is in 50 groups, each group has permissions

Solution: Pre-compute aggregated permissions

Background job (every 5 minutes):
1. For each user, compute all permissions (from roles + groups)
2. Store aggregated list in cache
3. Cache key: user:{user_id}:permissions

Permission check:
1. Get user's aggregated permissions from cache
2. Check if required permission is in list
3. Fast lookup: O(1) hash lookup

Trade-off: Eventual consistency (up to 5 min delay)
```

**3. Permission Scoping**

```text
Problem: Need different permissions per resource instance

Pattern: Hierarchical permissions with scopes

Global scope: admin:users (can manage all users)
Org scope: admin:users:org_123 (can manage users in org 123)
Team scope: admin:users:org_123:team_456 (can manage users in team 456)

Check algorithm (most specific wins):
1. Check team-level permission
2. If not found, check org-level permission
3. If not found, check global permission
4. If not found, deny

Example:
User: {permissions: ["admin:users:org_123"]}
Check: Can admin user_999 in org_123? → Yes
Check: Can admin user_888 in org_456? → No
```

**4. Policy as Code**

```text
Define policies in code for version control and testing

Example: Rego (Open Policy Agent)

package authz

default allow = false

# Allow if user is admin
allow {
  input.user.role == "admin"
}

# Allow if user is owner of resource
allow {
  input.user.id == input.resource.owner_id
}

# Allow if user's department matches
allow {
  input.user.department == input.resource.department
  input.user.level >= input.resource.required_level
}

# Deny if resource is archived
deny {
  input.resource.status == "archived"
}

# Final decision
decision = "allow" {
  allow
  not deny
} else = "deny"
```

#### Audit and Compliance

**Authorization Audit Log:**

```text
Every authorization decision should be logged:

{
  "timestamp": "2026-01-22T14:30:00Z",
  "user_id": "user123",
  "user_roles": ["editor"],
  "action": "delete",
  "resource_type": "post",
  "resource_id": "post456",
  "decision": "denied",
  "reason": "missing_permission:delete:any_post",
  "context": {
    "ip_address": "10.0.1.50",
    "user_agent": "Mozilla/5.0...",
    "session_id": "session_abc"
  }
}

Use cases:
- Compliance audits (who accessed what?)
- Security investigations (unauthorized access attempts)
- User support (why was I denied?)
- Policy debugging (which rule denied access?)
```

**Permission Review Workflows:**

```text
Regulatory requirements (SOC 2, ISO 27001):
- Quarterly access reviews
- Recertification of elevated permissions
- Automated alerts for unused permissions

Implementation:
1. Generate report: Users with high-privilege roles
2. Send to managers: "Confirm these users need these permissions"
3. Auto-revoke unconfirmed permissions
4. Log all decisions for audit trail

Example automation:
- User hasn't used admin permission in 90 days → Flag for review
- User changed departments → Auto-remove old department permissions
- User promoted → Require approval for new elevated permissions
```

#### Performance at Scale

**Caching Strategy:**

```text
L1 Cache (in-memory, per server): 1-5 sec TTL
├─ Recently checked permissions
├─ 10,000 entries per server
└─ <1ms lookup

L2 Cache (Redis cluster): 5-15 min TTL
├─ User roles and permissions
├─ Policy evaluation results
└─ 2-5ms lookup

L3 Cache (Database query cache): 1-5 min TTL
├─ Role definitions
├─ Permission mappings
└─ 10-20ms lookup

Database (source of truth): No cache
├─ Authoritative data
├─ Invalidation triggers cache clear
└─ 20-50ms query
```

**Bulk Authorization Checks:**

```text
Problem: Loading list of 100 documents, need to check permissions for each

Anti-pattern:
FOR EACH document:
  IF user_can_view(user, document) THEN
    include in results
  END IF
END FOR
// Result: 100 database queries

Better:
// Single query with JOIN
SELECT d.* FROM documents d
WHERE EXISTS (
  SELECT 1 FROM document_permissions dp
  WHERE dp.document_id = d.id
  AND dp.user_id = :user_id
  AND dp.permission = 'view'
)

// Or: Batch check
permissions = check_permissions_batch(user_id, document_ids, 'view')
filtered_docs = [doc for doc in documents if permissions[doc.id]]

Result: 1-2 database queries instead of 100
```

---

### 🤔 Think About It

**For Beginners:**
1. When would you use ACL instead of RBAC?
2. What's the difference between a role and a permission?
3. Why is RBAC more scalable than ACL?

**For Intermediate:**
4. How would you implement "edit own post" permission in RBAC?
5. When should you choose ABAC over RBAC?
6. What's the trade-off between fine-grained and coarse-grained permissions?

**For Advanced:**
7. How do you handle permission inheritance in a deep hierarchy?
8. What's the consistency model for distributed permission checks?
9. How do you optimize authorization for a billion permission checks per day?

---

### 📝 Key Takeaways

**Model Selection:**
- **ACL**: Small scale, simple needs (< 1000 resources)
- **RBAC**: Most common, good for 80% of use cases
- **ABAC**: Complex rules, dynamic policies
- **ReBAC**: Social networks, collaborative apps
- **Hybrid**: Combine models for best results

**Implementation Best Practices:**
- Cache aggressively (roles, permissions, decisions)
- Use lazy evaluation (check only when needed)
- Implement hierarchical permissions (global > org > team)
- Audit all decisions (compliance and debugging)
- Test policies thoroughly (unit tests for permission logic)

**Performance:**
- Multi-tier caching (in-memory, Redis, database)
- Bulk permission checks (avoid N+1 queries)
- Pre-computed permission aggregations
- Deny-by-default (fail secure)

**Common Mistakes:**
- Role explosion (too many specific roles)
- Permission in JWT (token too large, can't revoke)
- No audit logging (can't debug or prove compliance)
- Synchronous permission checks (blocking operations)
- Not caching (every check hits database)

---

### 💪 Practice Exercise

**Scenario:** Design authorization for a project management tool like Jira.

**Requirements:**
- Organization → Projects → Issues hierarchy
- Roles: Org Admin, Project Admin, Developer, Reporter, Viewer
- Permissions: create/edit/delete issues, manage project, manage users
- Sharing: Issues can be shared with specific users
- Rules: Developers can only edit issues assigned to them

**Your Task:**
1. Choose authorization model(s) to use
2. Design the database schema
3. Define roles and their permissions
4. Explain how to check: "Can user123 delete issue456?"
5. How do you handle: "Share this issue with external consultant"?

**Bonus Challenge:**
- How do you implement: "Viewers can see issues but not sensitive fields like salary"?
- What's the caching strategy for 10,000 permission checks per second?
- How do you audit who accessed confidential issues?

---

## Section 6: How Users Interact (API Design)

### What You'll Learn

By the end of this section, you'll be able to:
- Design RESTful APIs for authentication and authorization
- Structure secure request/response formats for auth flows
- Implement OAuth 2.0, SAML, and OpenID Connect endpoints
- Handle token lifecycle management APIs
- Design multi-factor authentication endpoints
- Create admin and audit APIs for identity management

### Why This Matters

Your authentication API is the gateway to every secure system. A well-designed API is intuitive for developers, secure against attacks, and flexible enough to support multiple authentication methods. A bad API leads to security vulnerabilities, poor developer experience, and integration nightmares. Real example: Twitter's API key breach in 2018 exposed millions of passwords because of inadequate API security design!

---

### 🟢 For Beginners: What is an Authentication API?

#### The Security Guard Analogy

Think of an authentication API like a security checkpoint at a building:

```text
You (visitor) → Show ID → Guard checks → Get visitor badge → Enter building

In technical terms:
Your app → Login credentials → Auth API → Access token → Call protected APIs

Security Guard = Authentication API (checks who you are)
ID Card = Username/Password (your credentials)
Visitor Badge = Access Token (proof you're allowed in)
Building Access = Protected Resources (your data)
```

#### The Three Main Things Our Auth API Does

##### 1. Register User (Getting an ID Card)

```text
You say: "I'm new, need to register"
Guard: "Fill out this form" (Provide username, email, password)
System: Creates your account, sends verification email
You get: User account ready to use
```

##### 2. Login (Showing ID, Getting Badge)

```text
You say: "I want to login" (Send username and password)
Guard checks: "Are these credentials valid?" (Verify against database)
System: Generates access token
You get: Token to access protected resources
```

##### 3. Access Protected Resources (Using Your Badge)

```text
You say: "I want to see my profile" (Send token with request)
Guard checks: "Is this token valid?" (Verify token)
System: Returns your data if authorized
You get: Your profile information
```

#### What Does an Auth API Request Look Like?

Let's see a real example of registering a user:

##### Your Request (What you send)

```http
POST https://api.auth.example.com/v1/register
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecureP@ssw0rd!"
}
```

Think of this as saying: "I want to create an account!"

##### Server's Response (What you get back)

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "user_id": "usr_abc123",
  "username": "john_doe",
  "email": "john@example.com",
  "email_verified": false,
  "created_at": "2025-01-15T10:30:00Z",
  "message": "Verification email sent to john@example.com"
}
```

Think of this as: "Account created! Check your email to verify."

#### Understanding HTTP Methods for Auth

HTTP methods are like different security operations:

```text
POST = "Create new" / "Submit credentials"
├─ Example: POST /v1/login
├─ Like: "Check my credentials, let me in"
└─ Used for: Login, Register, Token refresh

GET = "Show me" / "Retrieve"
├─ Example: GET /v1/users/me
├─ Like: "Show me my profile"
└─ Used for: Get user info, permissions

PUT = "Update completely"
├─ Example: PUT /v1/users/me
├─ Like: "Update all my profile info"
└─ Used for: Update user profile

PATCH = "Update partially"
├─ Example: PATCH /v1/users/me/password
├─ Like: "Just change my password"
└─ Used for: Password reset, email update

DELETE = "Remove"
├─ Example: DELETE /v1/sessions
├─ Like: "Log me out, remove my session"
└─ Used for: Logout, delete account
```

#### Our Auth API's Menu (Main Endpoints)

Here's what you can do with our authentication API:

```text
1. Register User
   POST /v1/register
   "I want to create an account"

2. Login
   POST /v1/login
   "Let me in with my credentials"

3. Logout
   POST /v1/logout
   "Log me out, invalidate my token"

4. Get My Profile
   GET /v1/users/me
   "Show me my profile information"

5. Refresh Token
   POST /v1/token/refresh
   "My token is expiring, give me a new one"

6. Reset Password
   POST /v1/password/reset
   "I forgot my password, send reset link"

7. Verify MFA Code
   POST /v1/mfa/verify
   "Here's my 6-digit code from authenticator app"
```

#### What Can Go Wrong? (Common Auth Errors)

Just like with building security, things can go wrong:

```text
401 Unauthorized = "I don't know who you are"
├─ You: Try to access profile without token
├─ Server: "Please login first!"
└─ Example: Missing or expired token

403 Forbidden = "I know who you are, but you can't do that"
├─ You: Try to delete admin account as regular user
├─ Server: "You don't have permission!"
└─ Example: Insufficient permissions

400 Bad Request = "Invalid credentials or data"
├─ You: "My password is '123'"
├─ Server: "Password too weak!"
└─ Example: Password doesn't meet requirements

429 Too Many Requests = "Too many login attempts"
├─ You: Try wrong password 10 times
├─ Server: "Locked out for 15 minutes!"
└─ Example: Brute force protection triggered

409 Conflict = "Username already taken"
├─ You: Register with existing username
├─ Server: "That username is taken!"
└─ Example: Duplicate user registration
```

💡 **Pro Tip:** Security error messages should be helpful but not give away too much! Don't say "Password wrong" vs "Username wrong" - say "Invalid credentials" for both!

---

### 🟡 For Intermediate: RESTful Auth API Design Patterns

#### REST Principles Applied to Authentication

**Key REST Principles for Auth APIs:**

```text
1. Stateless (No Server Memory Between Requests)
   ✅ Good: Each request includes token for auth
   ❌ Bad: Server remembers "you logged in 5 mins ago"

2. Standard HTTP Methods
   ✅ Good: POST /login (create session)
   ❌ Bad: GET /doLogin (GET should be idempotent)

3. Proper Status Codes
   ✅ Good: 401 for auth failure, 403 for permission
   ❌ Bad: Everything returns 200 OK with error in body

4. Secure by Default
   ✅ Good: HTTPS only, secure headers
   ❌ Bad: HTTP allowed, no CORS protection

5. Token in Authorization Header
   ✅ Good: Authorization: Bearer <token>
   ❌ Bad: Token in query string (logs everywhere!)
```

#### Complete API Design

##### 1. User Management APIs

**1.1 Register User**

```http
POST /v1/register
Content-Type: application/json

Request Body:
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecureP@ssw0rd!",
  "first_name": "John",
  "last_name": "Doe",
  "phone": "+1-555-0123",           // Optional
  "consent": {
    "terms": true,
    "privacy": true,
    "marketing": false
  }
}

Success Response (201 Created):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "username": "john_doe",
    "email": "john@example.com",
    "email_verified": false,
    "created_at": "2025-01-15T10:30:00Z",
    "verification_sent": true,
    "next_steps": [
      "Check email for verification link",
      "Verification link expires in 24 hours"
    ]
  }
}

Error Response (400 Bad Request):
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Registration validation failed",
    "details": [
      {
        "field": "password",
        "reason": "Password must be at least 12 characters"
      },
      {
        "field": "email",
        "reason": "Invalid email format"
      }
    ]
  },
  "request_id": "req_abc123"
}

Error Response (409 Conflict):
{
  "status": "error",
  "error": {
    "code": "USER_EXISTS",
    "message": "User with this email already exists",
    "details": {
      "field": "email",
      "suggestion": "Try logging in or use password reset"
    }
  }
}
```

**1.2 Login**

```http
POST /v1/login
Content-Type: application/json

Request Body:
{
  "username": "john_doe",           // or email
  "password": "SecureP@ssw0rd!",
  "remember_me": true,              // Optional: longer session
  "device_info": {                  // Optional: for device tracking
    "device_id": "dev_xyz789",
    "device_name": "iPhone 13",
    "user_agent": "MyApp/1.0"
  }
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,                    // seconds
    "refresh_token": "rt_abc123xyz...",
    "user": {
      "user_id": "usr_abc123",
      "username": "john_doe",
      "email": "john@example.com",
      "roles": ["user"],
      "permissions": ["read:profile", "write:profile"]
    }
  }
}

Success Response with MFA Required (200 OK):
{
  "status": "success",
  "data": {
    "mfa_required": true,
    "mfa_token": "mfa_temp_token_xyz",     // Temporary token
    "mfa_methods": ["totp", "sms"],
    "message": "Please provide MFA code to complete login"
  }
}

Error Response (401 Unauthorized):
{
  "status": "error",
  "error": {
    "code": "INVALID_CREDENTIALS",
    "message": "Invalid username or password",
    "attempts_remaining": 3,
    "lockout_after": 5
  }
}

Error Response (423 Locked):
{
  "status": "error",
  "error": {
    "code": "ACCOUNT_LOCKED",
    "message": "Account temporarily locked due to too many failed attempts",
    "locked_until": "2025-01-15T11:00:00Z",
    "retry_after_seconds": 900
  }
}
```

**1.3 Logout**

```http
POST /v1/logout
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body (Optional):
{
  "logout_all_devices": false      // true = invalidate all sessions
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "Successfully logged out",
    "logged_out_at": "2025-01-15T10:30:00Z"
  }
}
```

**1.4 Get User Profile**

```http
GET /v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "username": "john_doe",
    "email": "john@example.com",
    "email_verified": true,
    "first_name": "John",
    "last_name": "Doe",
    "phone": "+1-555-0123",
    "phone_verified": false,
    "avatar_url": "https://cdn.example.com/avatars/abc123.jpg",
    "roles": ["user", "premium"],
    "mfa_enabled": true,
    "created_at": "2025-01-01T10:00:00Z",
    "last_login": "2025-01-15T10:30:00Z",
    "account_status": "active"
  }
}
```

**1.5 Update User Profile**

```http
PATCH /v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "first_name": "Jonathan",
  "phone": "+1-555-9999"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "updated_fields": ["first_name", "phone"],
    "updated_at": "2025-01-15T10:35:00Z"
  }
}
```

**1.6 Change Password**

```http
PATCH /v1/users/me/password
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "current_password": "OldP@ssw0rd!",
  "new_password": "NewSecureP@ssw0rd!",
  "logout_other_sessions": true     // Invalidate other tokens
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "Password changed successfully",
    "changed_at": "2025-01-15T10:40:00Z",
    "sessions_invalidated": 3
  }
}

Error Response (400 Bad Request):
{
  "status": "error",
  "error": {
    "code": "WEAK_PASSWORD",
    "message": "New password does not meet security requirements",
    "requirements": {
      "min_length": 12,
      "require_uppercase": true,
      "require_lowercase": true,
      "require_numbers": true,
      "require_special": true,
      "not_in_breach_database": true
    }
  }
}
```

**1.7 Request Password Reset**

```http
POST /v1/password/reset-request
Content-Type: application/json

Request Body:
{
  "email": "john@example.com"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "If this email exists, a reset link has been sent",
    "expires_in": 3600
  }
}

Note: Always return success even if email doesn't exist (security best practice)
```

**1.8 Reset Password with Token**

```http
POST /v1/password/reset
Content-Type: application/json

Request Body:
{
  "reset_token": "reset_abc123xyz...",
  "new_password": "NewSecureP@ssw0rd!"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "Password reset successful",
    "reset_at": "2025-01-15T10:50:00Z"
  }
}

Error Response (400 Bad Request):
{
  "status": "error",
  "error": {
    "code": "INVALID_TOKEN",
    "message": "Reset token is invalid or expired"
  }
}
```

##### 2. Token Management APIs

**2.1 Refresh Access Token**

```http
POST /v1/token/refresh
Content-Type: application/json

Request Body:
{
  "refresh_token": "rt_abc123xyz..."
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "rt_new456abc..."  // New refresh token (rotation)
  }
}

Error Response (401 Unauthorized):
{
  "status": "error",
  "error": {
    "code": "INVALID_REFRESH_TOKEN",
    "message": "Refresh token is invalid or expired",
    "action": "Please login again"
  }
}
```

**2.2 Validate Token (Introspection)**

```http
POST /v1/token/introspect
Content-Type: application/json
Authorization: Bearer <service_token>  // Service-to-service auth

Request Body:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "active": true,
    "token_type": "access_token",
    "user_id": "usr_abc123",
    "username": "john_doe",
    "scope": "read:profile write:profile",
    "client_id": "client_xyz",
    "exp": 1705329000,
    "iat": 1705325400,
    "iss": "https://auth.example.com"
  }
}

Inactive Token Response (200 OK):
{
  "status": "success",
  "data": {
    "active": false,
    "reason": "expired"
  }
}
```

**2.3 Revoke Token**

```http
POST /v1/token/revoke
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "token": "rt_abc123xyz...",
  "token_type_hint": "refresh_token"  // Optional: access_token or refresh_token
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "Token revoked successfully",
    "revoked_at": "2025-01-15T11:00:00Z"
  }
}
```

##### 3. Multi-Factor Authentication (MFA) APIs

**3.1 Setup MFA (TOTP)**

```http
POST /v1/mfa/totp/setup
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qr_code_url": "https://api.auth.example.com/v1/mfa/qr/abc123",
    "qr_code_data": "otpauth://totp/MyApp:john_doe?secret=JBSWY3DPEHPK3PXP&issuer=MyApp",
    "backup_codes": [
      "12345678",
      "23456789",
      "34567890",
      "45678901",
      "56789012"
    ],
    "instructions": [
      "Scan QR code with authenticator app (Google Authenticator, Authy)",
      "Enter 6-digit code to verify setup",
      "Save backup codes in secure location"
    ]
  }
}
```

**3.2 Verify MFA Setup**

```http
POST /v1/mfa/totp/verify-setup
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "code": "123456"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "MFA enabled successfully",
    "enabled_at": "2025-01-15T11:10:00Z",
    "backup_codes_count": 5
  }
}

Error Response (400 Bad Request):
{
  "status": "error",
  "error": {
    "code": "INVALID_MFA_CODE",
    "message": "The provided code is invalid",
    "attempts_remaining": 2
  }
}
```

**3.3 Verify MFA During Login**

```http
POST /v1/mfa/verify
Content-Type: application/json

Request Body:
{
  "mfa_token": "mfa_temp_token_xyz",  // From login response
  "code": "123456",
  "method": "totp"                    // or "sms", "backup_code"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "rt_abc123xyz..."
  }
}
```

**3.4 Generate New Backup Codes**

```http
POST /v1/mfa/backup-codes/regenerate
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "password": "SecureP@ssw0rd!"  // Require password for security
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "backup_codes": [
      "87654321",
      "76543210",
      "65432109",
      "54321098",
      "43210987"
    ],
    "message": "Previous backup codes have been invalidated",
    "generated_at": "2025-01-15T11:20:00Z"
  }
}
```

**3.5 Disable MFA**

```http
DELETE /v1/mfa
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "password": "SecureP@ssw0rd!",
  "code": "123456"  // Current MFA code
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "message": "MFA disabled successfully",
    "disabled_at": "2025-01-15T11:30:00Z"
  }
}
```

##### 4. OAuth 2.0 APIs

**4.1 Authorization Endpoint (Authorization Code Flow)**

```http
GET /v1/oauth/authorize
  ?response_type=code
  &client_id=client_abc123
  &redirect_uri=https://app.example.com/callback
  &scope=read:profile write:profile
  &state=random_state_string
  &code_challenge=BASE64URL(SHA256(code_verifier))  // PKCE
  &code_challenge_method=S256

User is redirected to login page, then back to redirect_uri:

Success Redirect:
https://app.example.com/callback
  ?code=auth_code_xyz789
  &state=random_state_string

Error Redirect:
https://app.example.com/callback
  ?error=access_denied
  &error_description=User+denied+access
  &state=random_state_string
```

**4.2 Token Exchange (Authorization Code → Access Token)**

```http
POST /v1/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)

Request Body:
grant_type=authorization_code
&code=auth_code_xyz789
&redirect_uri=https://app.example.com/callback
&code_verifier=original_code_verifier  // PKCE

Success Response (200 OK):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rt_abc123xyz...",
  "scope": "read:profile write:profile",
  "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."  // OpenID Connect
}
```

**4.3 Client Credentials Grant (Service-to-Service)**

```http
POST /v1/oauth/token
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:client_secret)

Request Body:
grant_type=client_credentials
&scope=service:access

Success Response (200 OK):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 7200,
  "scope": "service:access"
}
```

##### 5. Single Sign-On (SSO) / SAML APIs

**5.1 Initiate SSO Login (SAML)**

```http
GET /v1/sso/saml/login
  ?SAMLRequest=<base64_encoded_saml_request>
  &RelayState=<app_state>

Response: 302 Redirect to Identity Provider (IdP)
Location: https://idp.example.com/saml/login
  ?SAMLRequest=...
  &RelayState=...
```

**5.2 SAML Assertion Consumer Service (ACS)**

```http
POST /v1/sso/saml/acs
Content-Type: application/x-www-form-urlencoded

Request Body:
SAMLResponse=<base64_encoded_saml_response>
&RelayState=<app_state>

Success: 302 Redirect to application with session
Location: https://app.example.com/dashboard
Set-Cookie: session_id=xyz789; HttpOnly; Secure; SameSite=Strict
```

**5.3 SAML Metadata**

```http
GET /v1/sso/saml/metadata

Success Response (200 OK):
Content-Type: application/xml

<?xml version="1.0"?>
<EntityDescriptor xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
                  entityID="https://auth.example.com">
  <SPSSODescriptor>
    <AssertionConsumerService
      Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST"
      Location="https://auth.example.com/v1/sso/saml/acs"
      index="0"/>
  </SPSSODescriptor>
</EntityDescriptor>
```

##### 6. Permission Check APIs

**6.1 Check Permission**

```http
POST /v1/permissions/check
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "user_id": "usr_abc123",
  "resource": "document:doc_xyz789",
  "action": "read"
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "allowed": true,
    "reason": "User has 'editor' role on resource"
  }
}

Denied Response (200 OK):
{
  "status": "success",
  "data": {
    "allowed": false,
    "reason": "User lacks required permission"
  }
}
```

**6.2 Batch Permission Check**

```http
POST /v1/permissions/check-batch
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json

Request Body:
{
  "user_id": "usr_abc123",
  "checks": [
    {"resource": "document:doc_123", "action": "read"},
    {"resource": "document:doc_123", "action": "write"},
    {"resource": "document:doc_456", "action": "delete"}
  ]
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "results": [
      {"resource": "document:doc_123", "action": "read", "allowed": true},
      {"resource": "document:doc_123", "action": "write", "allowed": true},
      {"resource": "document:doc_456", "action": "delete", "allowed": false}
    ]
  }
}
```

**6.3 List User Permissions**

```http
GET /v1/users/me/permissions
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "permissions": [
      "read:profile",
      "write:profile",
      "read:documents",
      "write:documents",
      "delete:own_documents"
    ],
    "roles": [
      {
        "role_id": "role_editor",
        "role_name": "Editor",
        "scope": "organization:org_123"
      }
    ]
  }
}
```

##### 7. Admin APIs

**7.1 List Users (Admin)**

```http
GET /v1/admin/users
Authorization: Bearer <admin_token>

Query Parameters:
- page: 1 (default: 1)
- limit: 20 (default: 20, max: 100)
- status: active (options: active, suspended, deleted)
- search: john (search by username/email)
- sort: created_at (options: created_at, last_login, username)
- order: desc (options: asc, desc)

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "users": [
      {
        "user_id": "usr_abc123",
        "username": "john_doe",
        "email": "john@example.com",
        "status": "active",
        "mfa_enabled": true,
        "last_login": "2025-01-15T10:30:00Z",
        "created_at": "2025-01-01T10:00:00Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 50,
      "total_items": 1000,
      "items_per_page": 20
    }
  }
}
```

**7.2 Suspend User (Admin)**

```http
POST /v1/admin/users/{user_id}/suspend
Authorization: Bearer <admin_token>
Content-Type: application/json

Request Body:
{
  "reason": "Suspicious activity detected",
  "duration": 86400,  // seconds, null = indefinite
  "notify_user": true
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "status": "suspended",
    "suspended_at": "2025-01-15T12:00:00Z",
    "suspended_until": "2025-01-16T12:00:00Z",
    "reason": "Suspicious activity detected"
  }
}
```

**7.3 Assign Role (Admin)**

```http
POST /v1/admin/users/{user_id}/roles
Authorization: Bearer <admin_token>
Content-Type: application/json

Request Body:
{
  "role_id": "role_moderator",
  "scope": "organization:org_123"  // Optional: scope the role
}

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "user_id": "usr_abc123",
    "role_id": "role_moderator",
    "assigned_at": "2025-01-15T12:10:00Z"
  }
}
```

##### 8. Audit Log APIs

**8.1 Query Audit Logs**

```http
GET /v1/audit/logs
Authorization: Bearer <admin_token>

Query Parameters:
- start_date: 2025-01-01T00:00:00Z
- end_date: 2025-01-31T23:59:59Z
- user_id: usr_abc123 (optional)
- event_type: login (optional: login, logout, permission_change, etc.)
- resource: document:doc_123 (optional)
- page: 1
- limit: 50

Success Response (200 OK):
{
  "status": "success",
  "data": {
    "logs": [
      {
        "log_id": "log_xyz789",
        "timestamp": "2025-01-15T10:30:00Z",
        "event_type": "login",
        "user_id": "usr_abc123",
        "username": "john_doe",
        "ip_address": "203.0.113.42",
        "user_agent": "Mozilla/5.0...",
        "status": "success",
        "metadata": {
          "mfa_used": true,
          "device": "iPhone 13"
        }
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 10,
      "total_items": 500
    }
  }
}
```

**8.2 Export Audit Logs**

```http
POST /v1/audit/logs/export
Authorization: Bearer <admin_token>
Content-Type: application/json

Request Body:
{
  "start_date": "2025-01-01T00:00:00Z",
  "end_date": "2025-01-31T23:59:59Z",
  "format": "csv",  // or "json"
  "filters": {
    "event_types": ["login", "permission_change"],
    "user_ids": ["usr_abc123"]
  }
}

Success Response (202 Accepted):
{
  "status": "success",
  "data": {
    "export_id": "export_abc123",
    "status": "processing",
    "estimated_completion": "2025-01-15T12:30:00Z",
    "callback_url": "https://api.auth.example.com/v1/audit/exports/export_abc123"
  }
}
```

#### API Design Decisions

##### Decision 1: Token Storage Location

```text
OPTION A: Authorization Header (Recommended)
Authorization: Bearer <token>

Pros:
✅ Standard HTTP header
✅ Not logged in server access logs
✅ Not cached by browsers
✅ Secure, designed for credentials

Cons:
❌ Requires JavaScript for SPAs

OPTION B: Cookie
Set-Cookie: token=<token>; HttpOnly; Secure; SameSite=Strict

Pros:
✅ Automatic browser handling
✅ HttpOnly prevents XSS
✅ Works without JavaScript

Cons:
❌ CSRF vulnerability (need protection)
❌ Subdomain security concerns

OPTION C: Query String (Never do this!)
GET /api/data?token=<token>

Cons:
❌ Logged everywhere (server logs, proxy logs)
❌ Visible in browser history
❌ Leaked via Referer header

CHOICE: Authorization Header for APIs, Cookie for web apps
Reasoning: Best security practices, industry standard
```

##### Decision 2: API Versioning Strategy

```text
URL Path Versioning: /v1/login, /v2/login

Pros:
✅ Very explicit and clear
✅ Easy to route to different services
✅ Can deprecate old versions cleanly
✅ Works with all HTTP clients

Cons:
❌ URLs change with version
❌ Multiple endpoints to maintain

CHOICE: URL Path Versioning
Reasoning: 
├─ Auth APIs change infrequently
├─ Clear deprecation path
├─ Easy for developers to understand
└─ Interview tip: Discuss long-term support strategy
```

##### Decision 3: Password Requirements

```text
Industry Best Practices (NIST 2024):
├─ Minimum 12 characters (not 8!)
├─ Check against breach databases (HaveIBeenPwned)
├─ Allow spaces and special characters
├─ No maximum length limit (hash anyway)
├─ No forced periodic changes
└─ No complex rules that lead to "Password1!"

Implementation:
└─ Use zxcvbn library for strength estimation
```

#### Rate Limiting Implementation

**Rate Limit Headers:**

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 856
X-RateLimit-Reset: 1705329000
Retry-After: 60
```

**Rate Limit Tiers by Endpoint:**

```text
Authentication Endpoints:
├─ POST /v1/login: 5 per minute (per IP)
├─ POST /v1/register: 3 per hour (per IP)
├─ POST /v1/password/reset-request: 3 per hour (per email)
└─ POST /v1/mfa/verify: 5 per minute (per session)

Token Management:
├─ POST /v1/token/refresh: 10 per minute (per user)
├─ POST /v1/token/introspect: 1000 per minute (per service)
└─ POST /v1/token/revoke: 10 per minute (per user)

User APIs:
├─ GET /v1/users/me: 100 per minute (per user)
├─ PATCH /v1/users/me: 10 per minute (per user)
└─ PATCH /v1/users/me/password: 3 per hour (per user)

Admin APIs:
├─ GET /v1/admin/users: 100 per minute (per admin)
└─ POST /v1/admin/users/{id}/suspend: 50 per minute (per admin)

Reasoning:
├─ Lower limits for authentication (brute force protection)
├─ Higher limits for read operations
├─ Medium limits for write operations
└─ Very high limits for service-to-service
```

**Rate Limit Response:**

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1705329060

{
  "status": "error",
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many login attempts. Please try again in 60 seconds.",
    "details": {
      "limit": 5,
      "window": "1 minute",
      "retry_after_seconds": 60,
      "reset_at": "2025-01-15T12:01:00Z"
    }
  }
}
```

#### Error Handling Best Practices

**Consistent Error Format:**

```json
{
  "status": "error",
  "error": {
    "code": "ERROR_CODE",                    // Machine-readable
    "message": "Human readable description", // For developers
    "details": {},                           // Additional context
    "documentation_url": "https://docs.auth.example.com/errors/ERROR_CODE",
    "support_contact": "support@example.com"
  },
  "request_id": "req_abc123"                 // For support tickets
}
```

**Error Code Hierarchy:**

```text
4xx Client Errors:
├─ 400 VALIDATION_ERROR: Input validation failed
├─ 400 WEAK_PASSWORD: Password doesn't meet requirements
├─ 400 INVALID_MFA_CODE: MFA code incorrect
├─ 401 INVALID_CREDENTIALS: Wrong username/password
├─ 401 TOKEN_EXPIRED: Access token expired
├─ 401 INVALID_TOKEN: Malformed or invalid token
├─ 403 INSUFFICIENT_PERMISSIONS: User lacks permission
├─ 403 ACCOUNT_SUSPENDED: Account is suspended
├─ 404 USER_NOT_FOUND: User doesn't exist
├─ 409 USER_EXISTS: User already registered
├─ 409 ALIAS_TAKEN: Username/email taken
├─ 423 ACCOUNT_LOCKED: Too many failed attempts
└─ 429 RATE_LIMIT_EXCEEDED: Too many requests

5xx Server Errors:
├─ 500 INTERNAL_ERROR: Generic server error
├─ 502 IDP_ERROR: Identity provider failed
├─ 503 SERVICE_UNAVAILABLE: Maintenance mode
└─ 504 TIMEOUT: Request took too long
```

**Security-Conscious Error Messages:**

```text
DON'T reveal too much:
❌ "Password is incorrect" (reveals username exists)
❌ "Email not found" (username enumeration)
❌ "MFA code wrong, 3 attempts remaining" (timing attacks)

DO be helpful but vague:
✅ "Invalid credentials" (both username and password)
✅ "If this email exists, a reset link was sent"
✅ "Invalid or expired code" (no attempt count)
```

---

### 🔴 For Advanced: Production API Patterns

#### Idempotency for Safe Retries

**The Problem:**

```text
User clicks "Register" → Network timeout → User retries
└─ Without idempotency: 2 accounts created!
└─ With idempotency: Same account returned
```

**Implementation:**

```http
POST /v1/register
Idempotency-Key: unique-client-key-abc123
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "SecureP@ssw0rd!"
}
```

**Server-Side Architecture:**

```text
1. Check idempotency cache
   ├─ If key exists and within 24h: Return cached response
   └─ If key doesn't exist: Process request

2. Process request
   ├─ Start database transaction
   ├─ Create user
   ├─ Store response in cache with key
   └─ Commit transaction

3. Return response
   └─ Same response for same idempotency key

Cache Structure:
Key: "idempotency:abc123"
Value: {
  "status": 201,
  "body": { user data },
  "created_at": "2025-01-15T10:30:00Z"
}
TTL: 24 hours
```

**Benefits:**
- Safe retries after network failures
- Prevents duplicate user creation
- 24-hour cache window for retries

#### API Gateway Pattern for Auth

**Why API Gateway for Auth?**

```text
Without Gateway:
[Client] → [Auth Service]
        → [User Service]
        → [Permission Service]

Problems:
❌ Client needs multiple endpoints
❌ Duplicate rate limiting logic
❌ Complex CORS configuration
❌ No centralized logging

With Gateway:
[Client] → [API Gateway] → [Auth Service]
                         → [User Service]
                         → [Permission Service]

Benefits:
✅ Single entry point with TLS termination
✅ Centralized rate limiting and DDoS protection
✅ Request/response transformation
✅ Unified logging and monitoring
✅ API versioning and routing
```

**Gateway Configuration (Kong/AWS API Gateway):**

```yaml
# Authentication Service Routes
routes:
  - name: user_registration
    methods: [POST]
    paths: [/v1/register]
    service: auth_service
    plugins:
      - name: rate-limiting
        config:
          minute: 3
          policy: redis
          redis_host: redis.cache.local
      - name: bot-detection
        config:
          deny_bots: true
      - name: ip-restriction
        config:
          whitelist: null
          blacklist: [known_bad_ips]

  - name: user_login
    methods: [POST]
    paths: [/v1/login]
    service: auth_service
    plugins:
      - name: rate-limiting
        config:
          minute: 5
          policy: redis
          redis_host: redis.cache.local
      - name: request-transformer
        config:
          add:
            headers:
              - X-Request-ID:$(uuid)
              - X-Real-IP:$(remote_addr)
      - name: response-caching
        config:
          strategy: memory
          cache_ttl: 0  # Don't cache auth responses

  - name: token_refresh
    methods: [POST]
    paths: [/v1/token/refresh]
    service: token_service
    plugins:
      - name: rate-limiting
        config:
          minute: 10
          policy: redis

  - name: protected_resources
    methods: [GET, PUT, PATCH, DELETE]
    paths: [/v1/users/me*]
    service: user_service
    plugins:
      - name: jwt
        config:
          secret_is_base64: false
          claims_to_verify: [exp, nbf]
          key_claim_name: kid
      - name: rate-limiting
        config:
          minute: 100
          policy: redis

  - name: admin_operations
    methods: [GET, POST, PUT, DELETE]
    paths: [/v1/admin/*]
    service: admin_service
    plugins:
      - name: jwt
        config:
          claims_to_verify: [exp, nbf]
      - name: acl
        config:
          whitelist: [admin, super_admin]
      - name: rate-limiting
        config:
          minute: 100
          policy: redis
```

#### Token Rotation and Family Tracking

**Refresh Token Rotation:**

```text
Why Rotate Refresh Tokens?

Problem: Long-lived refresh tokens
├─ If stolen, attacker has long-term access
├─ Hard to detect compromise
└─ Can't revoke without affecting user

Solution: Rotate on every use
├─ Issue new refresh token with each refresh
├─ Invalidate old refresh token
├─ Track token families
└─ Detect replay attacks

Token Family Architecture:
[HLD Note: Core concept for interviews]

Data Structure:
{
  "family_id": "fam_abc123",
  "tokens": [
    {
      "token_id": "rt_v1",
      "issued_at": "2025-01-15T10:00:00Z",
      "expires_at": "2025-02-15T10:00:00Z",
      "used_at": "2025-01-15T10:05:00Z",
      "replaced_by": "rt_v2",
      "status": "used"
    },
    {
      "token_id": "rt_v2",
      "issued_at": "2025-01-15T10:05:00Z",
      "expires_at": "2025-02-15T10:05:00Z",
      "status": "active"
    }
  ]
}

Replay Detection Logic:
1. User sends refresh token rt_v1
2. System checks: rt_v1 status = "used" (already rotated to rt_v2)
3. ALERT: Token replay detected!
4. Action: Invalidate entire token family (all tokens)
5. User must login again

Benefits:
✅ Limits impact of token theft
✅ Detects replay attacks
✅ Automatic revocation on compromise
✅ User not affected by normal rotation
```

#### Webhook Delivery for Auth Events

**Auth Event Webhooks:**

```text
Webhook Events:
├─ user.registered: New user signed up
├─ user.login: User logged in
├─ user.logout: User logged out
├─ user.password_changed: Password updated
├─ user.mfa_enabled: MFA turned on
├─ user.mfa_disabled: MFA turned off
├─ user.suspended: Account suspended
├─ user.deleted: Account deleted
├─ permission.granted: New permission assigned
└─ permission.revoked: Permission removed
```

**Webhook Payload:**

```json
{
  "event_id": "evt_abc123",
  "event_type": "user.login",
  "timestamp": "2025-01-15T10:30:00Z",
  "api_version": "2025-01-15",
  "data": {
    "user_id": "usr_abc123",
    "username": "john_doe",
    "email": "john@example.com",
    "ip_address": "203.0.113.42",
    "user_agent": "Mozilla/5.0...",
    "mfa_used": true,
    "location": {
      "country": "US",
      "city": "San Francisco"
    }
  },
  "webhook_id": "wh_xyz789",
  "signature": "sha256=a1b2c3..."  // HMAC-SHA256 signature
}
```

**Reliable Delivery Architecture:**

```text
[HLD Note: Focus on architecture, not implementation]

Components:
├─ Event Producer: Auth service emits events
├─ Event Queue: Kafka/RabbitMQ for buffering
├─ Webhook Worker: Processes and delivers
└─ Retry Handler: Handles failures

Delivery Guarantees:
1. At-least-once delivery
   ├─ Store webhook deliveries in database
   ├─ Mark as "pending" initially
   └─ Update to "delivered" on success

2. Retry Strategy (exponential backoff)
   ├─ Attempt 1: Immediate
   ├─ Attempt 2: 1 minute later
   ├─ Attempt 3: 5 minutes later
   ├─ Attempt 4: 15 minutes later
   ├─ Attempt 5: 1 hour later
   └─ Give up after 24 hours

3. Verification
   ├─ Include HMAC signature in webhook
   ├─ Customer verifies signature
   └─ Prevents spoofing

4. Idempotency
   ├─ Include event_id in payload
   ├─ Customer can dedupe events
   └─ Safe to receive same event twice
```

#### Advanced Security: Certificate-Based Auth

**Mutual TLS (mTLS) for Service-to-Service:**

```text
Standard TLS:
[Client] → verifies → [Server Certificate]

Mutual TLS (mTLS):
[Client Certificate] ← verifies ← [Server]
[Client] → verifies → [Server Certificate]

Both sides authenticate!

Use Cases:
✅ Service-to-service communication
✅ High-security environments (banking)
✅ Zero-trust architecture
✅ IoT device authentication

Implementation at API Gateway:
└─ Require client certificate for /v1/service/* endpoints
```

#### Performance Optimization Patterns

**1. Token Validation Caching:**

```text
Problem: Every API call validates token
├─ Query database for revocation list
├─ Check token signature
├─ Verify expiration
└─ 10ms per validation × 100K requests = 1000 seconds!

Solution: Cache validation results
├─ Cache key: SHA256(token)
├─ Cache value: { valid: true, user_id, roles }
├─ TTL: 5 minutes
└─ Check revocation list every 5 min

Performance Impact:
├─ Before: 10ms per request
├─ After: 0.1ms per request (cached)
└─ 100x speedup!

Trade-off:
├─ Pro: Massive performance gain
├─ Con: Up to 5-minute delay for revocations
└─ Mitigation: Force cache invalidation for critical revocations
```

**2. Permission Check Optimization:**

```text
Problem: N+1 queries for permission checks
├─ Check permission for resource 1 → DB query
├─ Check permission for resource 2 → DB query
├─ Check permission for resource 3 → DB query
└─ 100 resources = 100 queries!

Solution 1: Batch permission checks
POST /v1/permissions/check-batch
└─ Single query for all resources

Solution 2: Pre-compute permission cache
├─ User logs in → Load all permissions into cache
├─ Cache key: "perms:usr_abc123"
├─ Cache value: Set of all permissions
└─ Permission check = O(1) set lookup

Solution 3: Permission bloom filter
├─ Probabilistic data structure
├─ Fast negative lookups
├─ If bloom filter says "no" → definitely no
├─ If bloom filter says "yes" → check database
└─ Reduces 90% of database queries
```

**3. Connection Pooling:**

```text
Database Connection Pooling:
├─ Reuse connections instead of creating new ones
├─ Pool size: 50-100 connections per service
├─ Max wait time: 5 seconds
└─ 10x faster than creating new connections

Redis Connection Pooling:
├─ Pool size: 100 connections
├─ Pipeline commands for batch operations
└─ Use connection multiplexing
```

#### API Deprecation Strategy

**How to Deprecate Auth APIs:**

```text
Step 1: Announce (6 months before)
├─ Add deprecation notice to docs
├─ Email all API consumers
└─ Add Sunset header to responses:
    Sunset: Sat, 31 Dec 2025 23:59:59 GMT

Step 2: Mark as deprecated (3 months before)
├─ Add Warning header to responses:
    Warning: 299 - "This API version is deprecated"
├─ Track usage metrics
└─ Contact heavy users directly

Step 3: Restrict (1 month before)
├─ Reduce rate limits
├─ Add intentional delays
└─ Force users to migrate

Step 4: Shutdown (D-day)
├─ Return 410 Gone
├─ Redirect to documentation
└─ Provide migration guide

Never deprecate critical security endpoints without extensive notice!
```

---

### 💡 Key Takeaways

**Essential API Design Principles for Auth:**

```text
1. Security First
   ├─ Always use HTTPS
   ├─ Never log tokens or passwords
   ├─ Token in Authorization header, not query string
   └─ Implement rate limiting aggressively

2. Developer Experience
   ├─ Consistent error format
   ├─ Detailed documentation
   ├─ Request IDs for debugging
   └─ Helpful error messages (but not too helpful!)

3. Scalability
   ├─ Stateless authentication (JWT)
   ├─ Cache token validations
   ├─ Batch permission checks
   └─ Use API gateway for traffic management

4. Reliability
   ├─ Idempotency for write operations
   ├─ Graceful degradation
   ├─ Circuit breakers for dependencies
   └─ Comprehensive monitoring

5. Compliance
   ├─ Audit logging for all auth events
   ├─ GDPR-compliant data handling
   ├─ SOC 2 / ISO 27001 requirements
   └─ Data retention policies
```

**Interview Discussion Points:**

```text
When asked about Auth API design:

1. Start with requirements
   └─ "What authentication methods do we need to support?"

2. Discuss security first
   └─ "Let's ensure we're following OAuth 2.0 best practices..."

3. Scale considerations
   └─ "At 100K requests/second, we'll need to cache token validations..."

4. Trade-offs
   └─ "JWT offers stateless validation, but we can't revoke immediately..."

5. Evolution path
   └─ "We'll start with basic auth, then add OAuth, then SAML for enterprise..."
```

---

### 🤔 Think About It

1. **Token Storage:** Why is it dangerous to put tokens in query strings? Where have you seen tokens in real applications?

2. **Error Messages:** Balance security vs helpfulness - how would you design error messages for failed login attempts?

3. **Rate Limiting:** Why do we rate limit login attempts more aggressively than profile updates?

4. **Idempotency:** When is idempotency critical for auth operations? Which endpoints need it most?

5. **Versioning:** How would you deprecate an auth API version without breaking existing users?

---

### 💪 Practice Exercise

**Scenario:** Design the complete API for a banking application's authentication system.

**Requirements:**
- Multi-factor authentication (TOTP, SMS)
- Biometric authentication (fingerprint, face ID)
- Device registration and management
- Step-up authentication for sensitive operations (transfer >$10K)
- Session management across web and mobile
- Compliance with PSD2 (EU) and PCI DSS

**Your Task:**
1. Design all API endpoints with full request/response formats
2. Define the authentication flow for a money transfer
3. How do you implement step-up authentication?
4. Design the device registration and trust flow
5. What rate limits would you implement?
6. How do you handle token refresh on mobile apps?
7. Design the audit logging API

**Bonus Challenge:**
- How do you detect and prevent account takeover attacks?
- Design an API for passwordless authentication (WebAuthn)
- How would you implement risk-based authentication (anomaly detection)?
- What's your strategy for handling tokens during app updates?

---



## Section 7: Storing Our Data

### What You'll Learn

By the end of this section, you'll be able to:
- Design a comprehensive database schema for authentication and authorization
- Choose between SQL and NoSQL for different data types (users, sessions, tokens, permissions)
- Understand indexing strategies for fast lookups and security queries
- Design for data durability, consistency, and compliance (GDPR, SOC 2)
- Handle sensitive data storage with encryption and security best practices

### Why This Matters

Your database is the most critical component of your authentication system - it stores every user's identity, credentials, permissions, and security tokens. A bad schema can lead to slow login times, security vulnerabilities, or compliance violations. Real example: LinkedIn suffered a massive breach in 2012 when 6.5 million password hashes were stolen because they used weak hashing (unsalted SHA-1) and inadequate database security. This section teaches you how to avoid such catastrophic failures!

---

### 🟢 For Beginners: Understanding Authentication Data Storage

#### What Information Do We Need to Store?

Think of your authentication database like a highly secure filing cabinet in a bank's vault. Each user is a folder, and we need several pieces of information:

```text
Imagine this as a secure Excel sheet:

| User ID | Email          | Password Hash      | Status | Created Date | Last Login |
|---------|----------------|--------------------|--------|--------------|------------|
| 1       | alice@ex.com   | $2b$12$kX9...     | Active | 2025-01-15   | 2025-01-20 |
| 2       | bob@ex.com     | $2b$12$pL2...     | Active | 2025-01-16   | 2025-01-21 |
| 3       | charlie@ex.com | $2b$12$mN8...     | Locked | 2025-01-17   | 2025-01-18 |

This is essentially what our database stores!
```

#### Breaking Down Each Piece of Information

Let's understand WHY we store each piece:

**1. User ID (The Unique Identifier)**
```text
Example: 12345
├─ Why we store it: Unique identifier for each user
├─ Never changes: Even if user changes email
├─ Used everywhere: Links to sessions, tokens, permissions
└─ Like a: Social Security Number (unique and permanent)

Technical term: "Primary Key"
```

**2. Email (The Username)**
```text
Example: "alice@example.com"
├─ Why we store it: How users log in
├─ Must be unique: No two users with same email
├─ Validated: Must be a valid email format
└─ Like a: Your login ID at the bank

Security note: Store in lowercase for case-insensitive lookups
```

**3. Password Hash (Never Store Plain Passwords!)**
```text
Example: "$2b$12$kX9LmN3pQr5..." (Bcrypt hash)
├─ Why we hash: If database is stolen, passwords are useless
├─ One-way: Can't reverse hash to get password
├─ Slow by design: Takes 100ms to hash (prevents brute force)
└─ Like a: Shredding a document - can't unshred it

⚠️ NEVER EVER store "password123" - always store hashes!
```

**4. Account Status**
```text
Example: "Active" / "Suspended" / "Locked" / "Pending"
├─ Active: User can log in normally
├─ Suspended: Admin temporarily disabled account
├─ Locked: Too many failed login attempts
├─ Pending: Email not verified yet
└─ Like a: Status on your credit card (active/frozen/closed)
```

**5. Created Timestamp**
```text
Example: "2025-01-15 14:30:22"
├─ Why we store it: Audit trail and analytics
├─ Never changes: Account age matters for security
└─ Like a: Date you opened your bank account
```

**6. Last Login Timestamp**
```text
Example: "2025-01-20 09:15:43"
├─ Why we store it: Detect inactive accounts
├─ Updated on each login: Shows user activity
├─ Security: Alert if login from unusual location
└─ Like a: Last time you visited your bank branch
```

#### What Are Sessions?

Think of sessions like numbered ticket stubs at a coat check:

```text
At a Restaurant Coat Check:
├─ You arrive: Give coat to attendant
├─ Attendant gives you: Ticket #42 (session ID)
├─ You keep ticket: Show it when you want coat back
├─ Attendant checks: "Ticket #42? Here's your coat!"
└─ Ticket expires: After restaurant closes

In Our System:
├─ User logs in: Provides username/password
├─ System gives: Session ID "abc123xyz"
├─ User keeps in cookie: Sends with each request
├─ System checks: "Session abc123xyz belongs to Alice"
└─ Session expires: After 30 minutes of inactivity

Session Storage:
| Session ID  | User ID | Created    | Expires    | Device      |
|-------------|---------|------------|------------|-------------|
| abc123xyz   | 1       | 10:00 AM   | 10:30 AM   | Chrome/Mac  |
| def456uvw   | 2       | 10:05 AM   | 10:35 AM   | Safari/iOS  |
```

#### What Are Refresh Tokens?

Refresh tokens are like a VIP pass that lets you get new tickets without going through security again:

```text
Airport Security Analogy:
├─ Access Token: Boarding pass (expires in 1 hour)
│  ├─ Short-lived: Must be recent
│  ├─ Shows: "Alice, Flight 123, Gate 5"
│  └─ Gets you: Through gate onto plane
│
└─ Refresh Token: TSA PreCheck card (expires in 1 year)
   ├─ Long-lived: Valid for months
   ├─ Shows: "Alice is trusted traveler"
   └─ Gets you: New boarding passes without full security

In Our System:
├─ Access Token (JWT): Valid 15 minutes
│  └─ User sends with each API request
│
└─ Refresh Token: Valid 30 days
   └─ User sends only to get new access token
```

#### What Are Roles and Permissions?

Think of roles like job titles and permissions like keys to different rooms:

```text
In an Office Building:
├─ Employee: Can enter office, use printer
├─ Manager: Can enter office, use printer, access HR files
├─ Admin: Can enter anywhere, change door codes
└─ Visitor: Can only enter lobby with escort

In Our System:
Roles Table:
| Role ID | Name    | Description           |
|---------|---------|----------------------|
| 1       | User    | Basic account access |
| 2       | Manager | Team management      |
| 3       | Admin   | Full system access   |

Permissions Table:
| Permission ID | Resource | Action | Description        |
|---------------|----------|--------|--------------------|
| 1             | profile  | read   | View own profile   |
| 2             | profile  | write  | Edit own profile   |
| 3             | users    | read   | View all users     |
| 4             | users    | write  | Edit any user      |

Role-Permission Mapping:
| Role ID | Permission ID |
|---------|---------------|
| 1       | 1             | User can: View own profile
| 1       | 2             | User can: Edit own profile
| 2       | 1             | Manager can: View own profile
| 2       | 2             | Manager can: Edit own profile
| 2       | 3             | Manager can: View all users
| 3       | 1-4           | Admin can: Everything
```

#### Why We Need Multiple Tables

```text
Bad Design (Everything in One Table):
Users Table:
| ID | Email | Password | Role | Permissions    | Sessions     |
|----|-------|----------|------|----------------|--------------|
| 1  | alice | hash123  | User | read,write     | session1,...|
└─ Problems: Hard to query, wastes space, inflexible

Good Design (Separate Tables):
Users Table: Just user info
├─ Sessions Table: Just session data
├─ Roles Table: Just role definitions
├─ Permissions Table: Just permission definitions
└─ Junction Tables: Connect users to roles and roles to permissions

Benefits:
✅ Easy to query: "Show all admins" = simple query
✅ No duplication: Permission defined once, used many times
✅ Flexible: Add new roles without changing user records
✅ Fast: Indexes work better on smaller tables
```

#### What is Encryption?

Think of encryption like a safe deposit box:

```text
Hashing (One-Way):
Password "pass123" → Hash "$2b$12$kX9..."
├─ Can't reverse: Hash → Password (impossible!)
├─ Same input → Same hash (always)
└─ Use for: Passwords, API keys

Encryption (Two-Way):
Plain text "Alice's SSN: 123-45-6789" 
└─→ Encrypt → "AES256:kL9mN3pQr8..."
   └─→ Decrypt → "Alice's SSN: 123-45-6789"
├─ Can reverse: With the right key
├─ Different each time: Adds random "salt"
└─ Use for: Sensitive data that you need to read later

In Our Database:
├─ Passwords: HASHED (bcrypt)
├─ MFA secrets: ENCRYPTED (AES-256)
├─ API keys: HASHED (SHA-256)
└─ PII (SSN, etc.): ENCRYPTED (AES-256)
```

💡 **Pro Tip:** Always hash passwords with bcrypt or Argon2 - never use MD5 or SHA-1, they're broken!

---

### 🟡 For Intermediate: Complete Schema Design

#### Complete Database Schema

Let's design a production-ready authentication schema with all security features:

**Table 1: users (Core Identity)**

```sql
CREATE TABLE users (
    -- Primary identifier
    user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    
    -- Login credentials
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    password_hash VARCHAR(255) NOT NULL,  -- bcrypt: $2b$12$...
    
    -- Account status
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
        -- Values: 'pending', 'active', 'suspended', 'locked', 'deleted'
    failed_login_attempts INT NOT NULL DEFAULT 0,
    locked_until TIMESTAMP NULL,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP NULL,
    password_changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Security metadata
    require_password_change BOOLEAN NOT NULL DEFAULT FALSE,
    mfa_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    
    -- Constraints
    CONSTRAINT chk_email_format CHECK (email REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'),
    CONSTRAINT chk_status CHECK (status IN ('pending', 'active', 'suspended', 'locked', 'deleted'))
);

-- Indexes for common queries
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_status ON users(status);
CREATE INDEX idx_last_login ON users(last_login_at DESC);
CREATE INDEX idx_locked_accounts ON users(locked_until) WHERE locked_until IS NOT NULL;
```

**Why Each Field?**

```text
email_verified:
Purpose: Ensure user owns the email
Impact: Can't use features until verified
Example: Sign up → Receive email → Click link → Verified

failed_login_attempts:
Purpose: Track brute force attempts
Impact: Lock account after 5 failures
Example: Wrong password 3x → Counter = 3 → 2 more tries left

locked_until:
Purpose: Temporary account lockout
Impact: User can't login until this time
Example: Too many failed attempts → Locked for 30 minutes

password_changed_at:
Purpose: Force periodic password changes
Impact: "Password is 90 days old, please change"
Example: Enterprise policy: Change every 60 days
```

**Table 2: sessions (Active User Sessions)**

```sql
CREATE TABLE sessions (
    -- Session identifier (random, unpredictable)
    session_id VARCHAR(128) PRIMARY KEY,
    
    -- Who does this session belong to
    user_id BIGINT NOT NULL,
    
    -- Session lifecycle
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    last_accessed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    
    -- Device & security context
    ip_address VARCHAR(45) NOT NULL,  -- IPv6 support
    user_agent TEXT,
    device_fingerprint VARCHAR(64),
    
    -- Session metadata
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    logout_reason VARCHAR(50),  -- 'user_logout', 'timeout', 'admin_revoke'
    
    -- Foreign key
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- Indexes
    INDEX idx_user_sessions (user_id, is_active),
    INDEX idx_expires (expires_at),
    INDEX idx_ip_address (ip_address)
);

-- Partition by month for performance
ALTER TABLE sessions PARTITION BY RANGE (YEAR(created_at) * 100 + MONTH(created_at)) (
    PARTITION p202501 VALUES LESS THAN (202502),
    PARTITION p202502 VALUES LESS THAN (202503),
    PARTITION p202503 VALUES LESS THAN (202504)
);
```

**Session Management Queries:**

```sql
-- Create new session
INSERT INTO sessions (session_id, user_id, expires_at, ip_address, user_agent)
VALUES ('abc123xyz', 42, NOW() + INTERVAL 30 MINUTE, '192.168.1.1', 'Chrome/120');

-- Validate session
SELECT user_id, expires_at 
FROM sessions 
WHERE session_id = 'abc123xyz' 
  AND is_active = TRUE 
  AND expires_at > NOW();

-- Update last access (extend session)
UPDATE sessions 
SET last_accessed_at = NOW(), 
    expires_at = NOW() + INTERVAL 30 MINUTE
WHERE session_id = 'abc123xyz';

-- Logout (invalidate session)
UPDATE sessions 
SET is_active = FALSE, 
    logout_reason = 'user_logout'
WHERE session_id = 'abc123xyz';

-- Cleanup expired sessions (cron job)
DELETE FROM sessions 
WHERE expires_at < NOW() - INTERVAL 7 DAY;
```

**Table 3: refresh_tokens (Long-Lived Tokens)**

```sql
CREATE TABLE refresh_tokens (
    -- Token identifier
    token_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    token_hash VARCHAR(64) UNIQUE NOT NULL,  -- SHA-256 hash
    
    -- Ownership
    user_id BIGINT NOT NULL,
    
    -- Token lifecycle
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    last_used_at TIMESTAMP,
    
    -- Token rotation (security feature)
    family_id VARCHAR(36) NOT NULL,  -- UUID for token family
    is_revoked BOOLEAN NOT NULL DEFAULT FALSE,
    revoked_at TIMESTAMP,
    revoked_reason VARCHAR(100),
    
    -- Device binding
    device_id VARCHAR(64),
    ip_address VARCHAR(45),
    
    -- Foreign key
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    -- Indexes
    INDEX idx_token_hash (token_hash),
    INDEX idx_user_tokens (user_id, is_revoked, expires_at),
    INDEX idx_family (family_id),
    INDEX idx_expires (expires_at)
);
```

**Token Rotation Strategy:**

```text
Problem: Stolen refresh token can be used indefinitely
Solution: Automatic token rotation

Flow:
1. User logs in → Refresh token RT1 (family: F1)
2. Use RT1 → Get new access token + RT2 (family: F1)
   └─ RT1 is revoked
3. Use RT2 → Get new access token + RT3 (family: F1)
   └─ RT2 is revoked
4. Someone tries RT1 (already revoked) → Security alert!
   └─ Revoke entire family F1 (all tokens)
   └─ User must re-authenticate

Detection of token theft:
└─ If revoked token is used → Assume compromise
└─ Revoke all tokens in that family
└─ Alert user via email
```

**Table 4: roles (Role Definitions)**

```sql
CREATE TABLE roles (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    is_system_role BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT chk_role_name CHECK (name REGEXP '^[A-Z_]+$')  -- ADMIN, USER_MANAGER
);

-- Seed system roles
INSERT INTO roles (name, description, is_system_role) VALUES
('SUPER_ADMIN', 'Full system access', TRUE),
('ADMIN', 'Administrative access', TRUE),
('USER_MANAGER', 'Can manage users', TRUE),
('USER', 'Standard user access', TRUE),
('GUEST', 'Limited read-only access', TRUE);
```

**Table 5: permissions (Permission Definitions)**

```sql
CREATE TABLE permissions (
    permission_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) UNIQUE NOT NULL,
    resource VARCHAR(50) NOT NULL,
    action VARCHAR(20) NOT NULL,
    description TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT chk_action CHECK (action IN ('create', 'read', 'update', 'delete', 'list', 'admin')),
    CONSTRAINT uniq_resource_action UNIQUE (resource, action)
);

-- Seed permissions
INSERT INTO permissions (name, resource, action, description) VALUES
('users:read', 'users', 'read', 'View user details'),
('users:write', 'users', 'update', 'Edit user details'),
('users:delete', 'users', 'delete', 'Delete users'),
('users:list', 'users', 'list', 'List all users'),
('roles:admin', 'roles', 'admin', 'Manage roles'),
('permissions:admin', 'permissions', 'admin', 'Manage permissions');
```

**Table 6: role_permissions (Junction Table)**

```sql
CREATE TABLE role_permissions (
    role_id INT NOT NULL,
    permission_id INT NOT NULL,
    granted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    granted_by BIGINT,  -- Which admin granted this
    
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES roles(role_id) ON DELETE CASCADE,
    FOREIGN KEY (permission_id) REFERENCES permissions(permission_id) ON DELETE CASCADE,
    FOREIGN KEY (granted_by) REFERENCES users(user_id) ON DELETE SET NULL,
    
    INDEX idx_role (role_id),
    INDEX idx_permission (permission_id)
);

-- Grant permissions to roles
INSERT INTO role_permissions (role_id, permission_id) VALUES
(1, 1), (1, 2), (1, 3), (1, 4), (1, 5), (1, 6),  -- SUPER_ADMIN: all
(4, 1), (4, 2);  -- USER: read and write own profile
```

**Table 7: user_roles (Junction Table)**

```sql
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id INT NOT NULL,
    assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    assigned_by BIGINT,  -- Which admin assigned this
    expires_at TIMESTAMP,  -- Temporary role grant
    
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(role_id) ON DELETE CASCADE,
    FOREIGN KEY (assigned_by) REFERENCES users(user_id) ON DELETE SET NULL,
    
    INDEX idx_user (user_id),
    INDEX idx_role (role_id),
    INDEX idx_expires (expires_at)
);
```

**Permission Check Query (Optimized):**

```sql
-- Check if user has specific permission
SELECT COUNT(*) > 0 as has_permission
FROM user_roles ur
JOIN role_permissions rp ON ur.role_id = rp.role_id
JOIN permissions p ON rp.permission_id = p.permission_id
WHERE ur.user_id = 42
  AND p.resource = 'users'
  AND p.action = 'delete'
  AND (ur.expires_at IS NULL OR ur.expires_at > NOW());

-- Get all user permissions (for caching)
SELECT p.name, p.resource, p.action
FROM user_roles ur
JOIN role_permissions rp ON ur.role_id = rp.role_id
JOIN permissions p ON rp.permission_id = p.permission_id
WHERE ur.user_id = 42
  AND (ur.expires_at IS NULL OR ur.expires_at > NOW());
```

**Table 8: oauth_clients (Third-Party Applications)**

```sql
CREATE TABLE oauth_clients (
    client_id VARCHAR(64) PRIMARY KEY,
    client_secret_hash VARCHAR(64) NOT NULL,  -- Hashed
    
    -- Client metadata
    name VARCHAR(255) NOT NULL,
    description TEXT,
    logo_url TEXT,
    website_url TEXT,
    
    -- OAuth configuration
    redirect_uris TEXT NOT NULL,  -- JSON array
    allowed_scopes TEXT NOT NULL,  -- JSON array: ["profile", "email"]
    grant_types TEXT NOT NULL,  -- JSON array: ["authorization_code", "refresh_token"]
    
    -- Security
    is_confidential BOOLEAN NOT NULL DEFAULT TRUE,
    is_approved BOOLEAN NOT NULL DEFAULT FALSE,
    owner_user_id BIGINT NOT NULL,
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (owner_user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    INDEX idx_owner (owner_user_id)
);
```

**Table 9: oauth_authorization_codes (Short-Lived Codes)**

```sql
CREATE TABLE oauth_authorization_codes (
    code VARCHAR(128) PRIMARY KEY,
    client_id VARCHAR(64) NOT NULL,
    user_id BIGINT NOT NULL,
    
    -- Authorization details
    redirect_uri TEXT NOT NULL,
    scopes TEXT,  -- JSON array
    
    -- PKCE (Proof Key for Code Exchange)
    code_challenge VARCHAR(128),
    code_challenge_method VARCHAR(10),  -- 'plain' or 'S256'
    
    -- Lifecycle
    expires_at TIMESTAMP NOT NULL,
    used_at TIMESTAMP,
    
    FOREIGN KEY (client_id) REFERENCES oauth_clients(client_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    INDEX idx_expires (expires_at)
);

-- Authorization codes expire in 10 minutes
-- Cleanup job: DELETE FROM oauth_authorization_codes WHERE expires_at < NOW() - INTERVAL 1 HOUR;
```

**Table 10: oauth_access_tokens (Access Tokens for OAuth)**

```sql
CREATE TABLE oauth_access_tokens (
    token_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    
    -- Token ownership
    client_id VARCHAR(64) NOT NULL,
    user_id BIGINT NOT NULL,
    
    -- Token metadata
    scopes TEXT,  -- JSON array
    token_type VARCHAR(20) NOT NULL DEFAULT 'Bearer',
    
    -- Lifecycle
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    revoked_at TIMESTAMP,
    
    FOREIGN KEY (client_id) REFERENCES oauth_clients(client_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    INDEX idx_token_hash (token_hash),
    INDEX idx_user (user_id),
    INDEX idx_client (client_id),
    INDEX idx_expires (expires_at)
);
```

**Table 11: mfa_settings (Multi-Factor Authentication)**

```sql
CREATE TABLE mfa_settings (
    user_id BIGINT PRIMARY KEY,
    
    -- TOTP (Authenticator app)
    totp_secret_encrypted VARCHAR(255),  -- AES-256 encrypted
    totp_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    totp_verified_at TIMESTAMP,
    
    -- SMS
    phone_number_encrypted VARCHAR(255),  -- AES-256 encrypted
    phone_verified BOOLEAN NOT NULL DEFAULT FALSE,
    sms_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    
    -- Backup codes (one-time use)
    backup_codes_encrypted TEXT,  -- JSON array of hashed codes
    
    -- Recovery
    recovery_email_encrypted VARCHAR(255),
    
    -- Timestamps
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

**MFA Backup Codes Structure:**

```json
{
  "codes": [
    {
      "code_hash": "$2b$12$...",
      "used": false,
      "used_at": null
    },
    {
      "code_hash": "$2b$12$...",
      "used": true,
      "used_at": "2025-01-15T10:30:00Z"
    }
  ],
  "generated_at": "2025-01-01T00:00:00Z"
}
```

**Table 12: audit_logs (Security Audit Trail)**

```sql
CREATE TABLE audit_logs (
    log_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    
    -- Who and what
    user_id BIGINT,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id VARCHAR(100),
    
    -- Context
    ip_address VARCHAR(45),
    user_agent TEXT,
    
    -- Result
    status VARCHAR(20) NOT NULL,  -- 'success', 'failure', 'denied'
    error_message TEXT,
    
    -- Additional data
    metadata JSON,
    
    -- Timestamp
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE SET NULL,
    
    -- Indexes
    INDEX idx_user_actions (user_id, created_at DESC),
    INDEX idx_action (action, created_at DESC),
    INDEX idx_ip (ip_address),
    INDEX idx_created (created_at DESC)
) PARTITION BY RANGE (YEAR(created_at) * 100 + MONTH(created_at));
```

**Audit Log Examples:**

```sql
-- Log successful login
INSERT INTO audit_logs (user_id, action, status, ip_address, metadata)
VALUES (42, 'login', 'success', '192.168.1.1', '{"method": "password"}');

-- Log failed login
INSERT INTO audit_logs (user_id, action, status, ip_address, error_message)
VALUES (42, 'login', 'failure', '192.168.1.1', 'Invalid password');

-- Log permission denied
INSERT INTO audit_logs (user_id, action, resource_type, resource_id, status, error_message)
VALUES (42, 'delete', 'user', '123', 'denied', 'Insufficient permissions');

-- Query: Find suspicious activity
SELECT user_id, ip_address, COUNT(*) as failed_attempts
FROM audit_logs
WHERE action = 'login'
  AND status = 'failure'
  AND created_at > NOW() - INTERVAL 1 HOUR
GROUP BY user_id, ip_address
HAVING failed_attempts >= 5;
```

**Table 13: failed_login_attempts (Rate Limiting)**

```sql
CREATE TABLE failed_login_attempts (
    attempt_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    
    -- Identifier (could be user_id, email, or IP)
    user_id BIGINT,
    email VARCHAR(255),
    ip_address VARCHAR(45) NOT NULL,
    
    -- Attempt details
    attempted_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    user_agent TEXT,
    
    INDEX idx_user_recent (user_id, attempted_at DESC),
    INDEX idx_email_recent (email, attempted_at DESC),
    INDEX idx_ip_recent (ip_address, attempted_at DESC),
    INDEX idx_cleanup (attempted_at)
);

-- Rate limiting query
SELECT COUNT(*) as recent_failures
FROM failed_login_attempts
WHERE email = 'alice@example.com'
  AND attempted_at > NOW() - INTERVAL 15 MINUTE;

-- Cleanup old attempts (older than 24 hours)
DELETE FROM failed_login_attempts 
WHERE attempted_at < NOW() - INTERVAL 24 HOUR;
```

**Table 14: password_history (Prevent Reuse)**

```sql
CREATE TABLE password_history (
    history_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    changed_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    
    INDEX idx_user_history (user_id, changed_at DESC)
);

-- Check if new password was used before
SELECT COUNT(*) > 0 as password_reused
FROM password_history
WHERE user_id = 42
ORDER BY changed_at DESC
LIMIT 5;  -- Check last 5 passwords
```

#### Database Technology Choices

**PostgreSQL (Primary Database):**

```text
✅ Use PostgreSQL for:
├─ Users table (ACID critical)
├─ Roles and permissions (complex queries)
├─ OAuth clients
└─ MFA settings

Why PostgreSQL:
├─ ACID compliance (strong consistency)
├─ JSON support (flexible metadata)
├─ Full-text search (user search)
├─ Row-level security (multi-tenant)
├─ Mature replication
└─ Excellent performance for <100M users

Scale limits:
├─ Single instance: 10M users comfortably
├─ With read replicas: 50M users
├─ With sharding: 500M+ users
```

**Redis (Session Store):**

```text
✅ Use Redis for:
├─ Active sessions (fast lookups)
├─ Rate limiting counters
├─ Permission cache
└─ Token blacklist

Why Redis:
├─ In-memory (sub-millisecond latency)
├─ TTL support (auto-expire sessions)
├─ Atomic operations (rate limiting)
└─ Pub/sub (logout all devices)

Example session storage:
SET session:abc123xyz "{user_id:42,expires:...}" EX 1800
GET session:abc123xyz  # <1ms latency

PostgreSQL vs Redis for sessions:
├─ PostgreSQL: 10-50ms per lookup
├─ Redis: <1ms per lookup
└─ Winner: Redis (50x faster)
```

**Cassandra (Audit Logs):**

```text
✅ Use Cassandra for:
├─ Audit logs (high write volume)
├─ Login history
└─ Security events

Why Cassandra:
├─ Scales horizontally
├─ High write throughput (100K writes/sec)
├─ Time-series optimized
└─ Never delete (append-only)

Schema design:
PRIMARY KEY ((user_id), created_at)
└─ Partition by user_id
└─ Sort by created_at (DESC)
└─ Query: "Show user's last 100 actions" = fast
```

#### ER Diagram (Entity Relationships)

```text
users
  │
  ├──< sessions (one user, many sessions)
  ├──< refresh_tokens (one user, many tokens)
  ├──< audit_logs (one user, many log entries)
  ├──< mfa_settings (one-to-one)
  ├──< password_history (one user, many old passwords)
  └──< user_roles (many-to-many with roles)
       │
       └──> roles
             └──< role_permissions (many-to-many with permissions)
                  │
                  └──> permissions

oauth_clients
  ├──< oauth_authorization_codes
  └──< oauth_access_tokens

Relationships:
├─ User ─[1:N]─> Sessions (user can have multiple active sessions)
├─ User ─[M:N]─> Roles (user can have multiple roles)
├─ Role ─[M:N]─> Permissions (role can have multiple permissions)
└─ OAuth Client ─[1:N]─> Access Tokens
```

---

### 🔴 For Advanced: Sharding, Encryption, and Compliance

#### When to Shard Authentication Database

**Vertical Scaling Limits:**

```text
Single PostgreSQL Server Limits:
├─ Users: 10-50M users (with optimization)
├─ Sessions: 1M active sessions
├─ Write QPS: 5,000-10,000
└─ Read QPS: 50,000-100,000 (with read replicas)

For Authentication System:
├─ We estimated 10M users, 100K auth requests/sec
├─ With caching: 95% cache hit → 5K DB queries/sec
└─ Verdict: Single server + replicas sufficient

When you DO need sharding:
├─ Users > 50M
├─ Auth requests > 200K/sec (even with cache)
├─ Geographic distribution required (GDPR data residency)
└─ Multi-tenant isolation requirements
```

#### Sharding Strategy: Hash-Based by User ID

```python
"""
Sharding Strategy for Authentication System
Purpose: Distributes users across N database shards
Constraint: User and all related data (sessions, tokens) on same shard
"""

def get_shard_for_user(user_id, num_shards=64):
    """
    Consistent hashing for user sharding
    
    Args:
        user_id: Integer user ID
        num_shards: Total database shards (must be power of 2)
    
    Returns:
        shard_id: 0 to num_shards-1
    
    Properties:
        - All user data on one shard (no distributed transactions)
        - Even distribution
        - User always routes to same shard
    """
    return user_id % num_shards

# Example usage
user_id = 123456789
shard = get_shard_for_user(user_id, num_shards=64)  # Returns 21

# All user's data on same shard
db = connect_to_shard(shard)
db.query("SELECT * FROM users WHERE user_id = %s", user_id)
db.query("SELECT * FROM sessions WHERE user_id = %s", user_id)
db.query("SELECT * FROM refresh_tokens WHERE user_id = %s", user_id)
```

**Shard Mapping:**

```text
64 Shards Configuration:

Shard 0:  user_id % 64 == 0  (users: 0, 64, 128, ...)
Shard 1:  user_id % 64 == 1  (users: 1, 65, 129, ...)
...
Shard 63: user_id % 64 == 63 (users: 63, 127, 191, ...)

Each shard:
├─ Independent PostgreSQL instance
├─ Contains: users, sessions, tokens, roles assignments
├─ Handles: ~1/64th of traffic
└─ Size: ~1TB (for 50M total users)

Global tables (replicated to all shards):
├─ roles (small, read-heavy)
├─ permissions (small, read-heavy)
├─ oauth_clients (small, read-heavy)
└─ Reasoning: Avoid cross-shard JOINs
```

**Challenges with Sharding:**

```text
Challenge 1: User Lookup by Email
Problem: Email not in shard key, must check all shards
├─ "Login with alice@example.com" → Which shard?
├─ Naive: Query all 64 shards (slow!)
└─ Solution: Separate email→user_id lookup service

Solution: Email Lookup Table
├─ Separate database: email_to_user_id
├─ Schema: (email → user_id, shard_id)
├─ Flow: 
   1. Lookup: "alice@example.com" → user_id=42, shard=21
   2. Query shard 21: SELECT * FROM users WHERE user_id=42

email_lookup table:
CREATE TABLE email_lookup (
    email VARCHAR(255) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    shard_id INT NOT NULL,
    INDEX idx_user (user_id)
);

Challenge 2: Counting Total Users
Problem: "How many total users?" requires querying all shards
├─ Must aggregate across 64 databases
├─ Slow and expensive
└─ Solution: Periodic aggregation to metrics database

Solution: Pre-computed Metrics
├─ Background job runs every 5 minutes
├─ Query each shard: SELECT COUNT(*) FROM users
├─ Store in metrics DB: total_users = sum(all_shards)
└─ Dashboard queries metrics DB (fast!)

Challenge 3: Global User ID Generation
Problem: Auto-increment doesn't work across shards
├─ Shard 0: user_id=1, 2, 3...
├─ Shard 1: user_id=1, 2, 3... (collision!)
└─ Need globally unique IDs

Solution: Snowflake ID (Twitter's approach)
├─ 64-bit ID structure:
│  ├─ 41 bits: Timestamp (milliseconds since epoch)
│  ├─ 10 bits: Shard ID (up to 1024 shards)
│  └─ 12 bits: Sequence (4096 IDs per millisecond per shard)
│
└─ Guarantees:
   ├─ Globally unique
   ├─ Time-sortable
   └─ No coordination needed

Python implementation:
def generate_user_id(shard_id):
    timestamp = int(time.time() * 1000) - EPOCH
    sequence = get_next_sequence()  # 0-4095
    user_id = (timestamp << 22) | (shard_id << 12) | sequence
    return user_id
```

#### Data Encryption Strategy

**Encryption at Rest:**

```sql
-- Database-level encryption (Transparent Data Encryption)
-- PostgreSQL 14+ with pgcrypto extension

-- Enable encryption
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Encrypt sensitive fields
CREATE TABLE users_encrypted (
    user_id BIGINT PRIMARY KEY,
    email VARCHAR(255),
    
    -- Encrypted fields (AES-256-GCM)
    ssn_encrypted BYTEA,  -- Social Security Number
    phone_encrypted BYTEA,  -- Phone number
    address_encrypted BYTEA,  -- Home address
    
    -- Encryption metadata
    encryption_key_id INT NOT NULL,  -- Which key encrypted this
    encrypted_at TIMESTAMP NOT NULL
);

-- Encrypt data
INSERT INTO users_encrypted (user_id, ssn_encrypted, encryption_key_id)
VALUES (
    42,
    pgp_sym_encrypt('123-45-6789', 'encryption_key_from_kms'),
    1
);

-- Decrypt data
SELECT 
    user_id,
    pgp_sym_decrypt(ssn_encrypted, 'encryption_key_from_kms') as ssn
FROM users_encrypted
WHERE user_id = 42;
```

**Key Management (KMS):**

```text
Encryption Key Hierarchy:

Level 1: Master Key (AWS KMS, Google Cloud KMS)
├─ Stored in: Hardware Security Module (HSM)
├─ Rotated: Annually
├─ Used to: Encrypt Data Encryption Keys (DEKs)
└─ Never leaves KMS

Level 2: Data Encryption Keys (DEK)
├─ Generated: Per table or per tenant
├─ Rotated: Quarterly
├─ Used to: Encrypt actual data
├─ Stored encrypted: DEK encrypted by Master Key
└─ Cached: In application memory (30 min TTL)

Encryption Flow:
1. App requests DEK from KMS (cached)
2. KMS decrypts DEK with Master Key
3. App uses DEK to encrypt/decrypt data
4. DEK never stored in plain text

Key Rotation:
1. Generate new DEK (v2)
2. Re-encrypt data with new DEK
3. Keep old DEK (v1) for backward compatibility
4. Background job: Migrate all data to v2
5. Retire v1 after 30 days
```

**Application-Level Encryption:**

```python
"""
Application-layer encryption for maximum security
Even DBA can't read encrypted data without key
"""

from cryptography.fernet import Fernet
import base64

class EncryptionService:
    def __init__(self, kms_client):
        self.kms = kms_client
        self.key_cache = {}  # Cache keys for 30 minutes
    
    def get_encryption_key(self, key_id):
        """Get DEK from KMS (with caching)"""
        if key_id in self.key_cache:
            return self.key_cache[key_id]
        
        # Fetch from KMS
        encrypted_dek = self.kms.get_data_key(key_id)
        dek = self.kms.decrypt(encrypted_dek)
        
        # Cache for 30 minutes
        self.key_cache[key_id] = dek
        return dek
    
    def encrypt_field(self, plaintext, key_id=1):
        """Encrypt sensitive field"""
        key = self.get_encryption_key(key_id)
        cipher = Fernet(key)
        ciphertext = cipher.encrypt(plaintext.encode())
        return base64.b64encode(ciphertext).decode()
    
    def decrypt_field(self, ciphertext, key_id=1):
        """Decrypt sensitive field"""
        key = self.get_encryption_key(key_id)
        cipher = Fernet(key)
        decoded = base64.b64decode(ciphertext)
        plaintext = cipher.decrypt(decoded)
        return plaintext.decode()

# Usage
enc = EncryptionService(kms_client)

# Encrypt before storing in DB
ssn_encrypted = enc.encrypt_field("123-45-6789")
db.execute(
    "INSERT INTO users (user_id, ssn_encrypted) VALUES (%s, %s)",
    (42, ssn_encrypted)
)

# Decrypt after reading from DB
row = db.query("SELECT ssn_encrypted FROM users WHERE user_id = 42")
ssn_plaintext = enc.decrypt_field(row['ssn_encrypted'])
```

#### GDPR Compliance Requirements

**Right to Access (Subject Access Request):**

```sql
-- Export all user data
CREATE OR REPLACE FUNCTION export_user_data(p_user_id BIGINT)
RETURNS JSON AS $$
DECLARE
    result JSON;
BEGIN
    SELECT json_build_object(
        'user', (SELECT row_to_json(u) FROM users u WHERE user_id = p_user_id),
        'sessions', (SELECT json_agg(s) FROM sessions s WHERE user_id = p_user_id),
        'roles', (SELECT json_agg(r) FROM user_roles ur 
                  JOIN roles r ON ur.role_id = r.role_id 
                  WHERE ur.user_id = p_user_id),
        'audit_logs', (SELECT json_agg(a) FROM audit_logs a WHERE user_id = p_user_id),
        'mfa_settings', (SELECT row_to_json(m) FROM mfa_settings m WHERE user_id = p_user_id)
    ) INTO result;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- Usage: SELECT export_user_data(42);
```

**Right to be Forgotten (Data Deletion):**

```sql
-- Pseudonymize user (soft delete with data retention)
CREATE OR REPLACE FUNCTION anonymize_user(p_user_id BIGINT)
RETURNS VOID AS $$
BEGIN
    -- Update user record
    UPDATE users SET
        email = CONCAT('deleted_', user_id, '@deleted.local'),
        password_hash = 'DELETED',
        status = 'deleted',
        updated_at = NOW()
    WHERE user_id = p_user_id;
    
    -- Delete sensitive data
    DELETE FROM sessions WHERE user_id = p_user_id;
    DELETE FROM refresh_tokens WHERE user_id = p_user_id;
    DELETE FROM mfa_settings WHERE user_id = p_user_id;
    
    -- Keep audit logs (anonymized)
    UPDATE audit_logs SET
        ip_address = '0.0.0.0',
        user_agent = 'DELETED',
        metadata = '{}'
    WHERE user_id = p_user_id;
    
    -- Log deletion
    INSERT INTO audit_logs (user_id, action, status)
    VALUES (p_user_id, 'account_deleted', 'success');
END;
$$ LANGUAGE plpgsql;
```

**Data Residency (EU users in EU, US users in US):**

```text
Geographic Sharding Strategy:

Primary shard key: (region, user_id)
├─ EU users: Shards 0-31 (Frankfurt datacenter)
├─ US users: Shards 32-47 (Virginia datacenter)
└─ APAC users: Shards 48-63 (Singapore datacenter)

Region detection:
1. User signs up from IP: 185.x.x.x (Germany)
2. Assign region: 'EU'
3. Generate user_id: Snowflake with region prefix
4. Route to: EU shards only

Constraints:
├─ User data never leaves region
├─ Backups stay in region
├─ Logs stay in region
└─ Cross-region queries prohibited

Compliance:
✅ GDPR (EU): Data stays in EU
✅ CCPA (California): Data stays in US
✅ PDPA (Singapore): Data stays in APAC
```

#### High Availability and Replication

**Multi-Region Active-Active Setup:**

```text
Topology:

US-EAST-1 (Primary):
[Primary DB] ──────────> [Replica 1]
     │                        │
     │                        │
     └────> [Replica 2]       └────> Redis Cache
                │
                └────────────────────────┐
                                         │
                                         ▼
EU-WEST-1 (Secondary):                   │
[Primary DB] <──────────────────────────┘
     │            (Async replication, 200ms lag)
     ├────> [Replica 1]
     └────> [Replica 2]

Routing Logic:
├─ EU users → EU region (low latency)
├─ US users → US region (low latency)
└─ Cross-region reads tolerate 200ms lag

Write Strategy:
├─ User writes to closest region (Primary)
├─ Async replication to other regions
├─ Conflict resolution: Last-write-wins
└─ Acceptable: 99.9% no conflicts (user accesses from one location)
```

**Backup and Recovery Strategy:**

```sql
-- Continuous WAL archiving (PostgreSQL)
archive_mode = on
archive_command = 'aws s3 cp %p s3://auth-db-backups/wal/%f'

-- Point-in-time recovery enabled
-- Can restore to any second in past 30 days

-- Backup schedule
├─ Continuous: WAL logs to S3 (real-time)
├─ Hourly: Incremental snapshots (1-hour RPO)
├─ Daily: Full backup to S3 (24-hour RPO)
├─ Weekly: Full backup to Glacier (compliance)
└─ Monthly: Long-term archive (7-year retention)

-- Disaster recovery test (quarterly)
1. Restore from backup to staging
2. Validate data integrity
3. Test application connectivity
4. Measure recovery time (RTO < 4 hours)
```

---

### Real-World Example: Auth0's Database Evolution

**2013 - MongoDB Single Cluster:**
```text
Setup:
├─ MongoDB 3-node replica set
├─ Users, sessions, tokens in one database
├─ Simple, fast to develop
└─ Handled 100K users

Problems at scale:
❌ MongoDB eventual consistency issues
❌ Hard to do complex queries (no joins)
❌ Difficult to maintain data integrity
❌ Token validation slow (200ms)
```

**2016 - Hybrid PostgreSQL + Redis:**
```text
Setup:
├─ PostgreSQL: Users, roles, OAuth clients
├─ Redis: Sessions, rate limiting
├─ Still MongoDB: Audit logs
└─ 32 PostgreSQL shards

Benefits:
✅ PostgreSQL ACID for critical data
✅ Redis 50x faster for sessions (<5ms)
✅ MongoDB perfect for logs (write-heavy)
✅ Handled 10M users

Challenges:
├─ Operational complexity (3 databases!)
├─ Data sync issues
└─ Higher costs
```

**2020 - PostgreSQL + Redis + Elasticsearch:**
```text
Setup:
├─ PostgreSQL (128 shards): Core auth data
├─ Redis (256 nodes): Sessions, cache
├─ Elasticsearch (40 nodes): Search, analytics
├─ Cassandra (60 nodes): Audit logs
└─ S3: Backup and archival

Architecture principles:
├─ PostgreSQL: Source of truth
├─ Redis: Performance layer
├─ Elasticsearch: Search and analytics
└─ Cassandra: Append-only logs

Results:
├─ 100M+ users supported
├─ 200K auth requests/sec
├─ <50ms token validation (p99)
├─ 99.99% uptime
└─ GDPR and SOC 2 compliant

Key lesson: Right database for each workload!
```

---

### 🤔 Think About It

1. **For Beginners:** Why do we store password *hashes* instead of passwords? What happens if someone steals our database?

2. **For Intermediate:** Your PM asks: "Can we store user's social security numbers in the database for verification?" What security measures must you implement? What compliance requirements apply?

3. **For Advanced:** You wake up to an alert: "User 42 had 50 failed login attempts in 1 minute from 50 different IPs". Your `failed_login_attempts` table is growing fast. How do you efficiently query and block this attack without impacting legitimate users?

---

### 🎯 Interview Questions: Authentication Database Design

#### Question 1: How do you store passwords securely?

**What the interviewer wants to know:**
- Do you understand hashing vs encryption?
- Can you explain why certain algorithms are better?

**Answer Framework:**

```text
❌ NEVER do this:
├─ Store plaintext: "password123"
├─ Use MD5: Broken, fast to crack
├─ Use SHA-1: Also broken
└─ Encrypt passwords: Reversible defeats the purpose

✅ DO this:
├─ Use bcrypt (adaptive, salted)
├─ Use Argon2 (modern, memory-hard)
└─ Use scrypt (alternative to Argon2)

Why bcrypt:
├─ Adaptive: Can increase "cost" as CPUs get faster
├─ Salted: Same password → Different hash
├─ Slow: Takes ~100ms to hash (prevents brute force)
└─ Industry standard: Used by Auth0, Okta, AWS

Storage format:
$2b$12$kX9LmN3pQr5sT6vU8wY0ZuABcDeFgHiJkLmNoPqRsTuVwXyZa1234
 │  │  │                                            │
 │  │  └─ Salt (random)                             └─ Hash
 │  └─ Cost factor (12 = 2^12 iterations ≈ 100ms)
 └─ Algorithm identifier (bcrypt)

Code example:
import bcrypt

# Registration
password = "user_password"
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
# Store hashed in database

# Login verification
stored_hash = db.get_password_hash(user_id)
if bcrypt.checkpw(password.encode(), stored_hash):
    # Correct password
```

#### Question 2: How do you design the session management system?

**Answer Framework:**

```text
Session Storage Options:

Option 1: Database Sessions (PostgreSQL)
├─ Pros: Persistent, survives server restart, multi-server
├─ Cons: Slower (10-50ms), DB load
└─ Use when: Strong consistency needed

Option 2: In-Memory Sessions (Redis)
├─ Pros: Fast (<1ms), auto-expiration (TTL)
├─ Cons: Lost if Redis crashes (unless persisted)
└─ Use when: Speed critical, can tolerate rare loss

Option 3: Stateless Tokens (JWT)
├─ Pros: No storage needed, scales infinitely
├─ Cons: Can't revoke, larger size
└─ Use when: Microservices, API-first

Recommended: Redis + PostgreSQL
├─ Redis: Primary session store (fast)
├─ PostgreSQL: Backup session store (durable)
├─ Write to both: Async write to PostgreSQL
├─ Read from Redis: Fall back to PostgreSQL if miss
└─ Best of both worlds: Speed + durability

Schema design:
Redis:
  Key: "session:{session_id}"
  Value: JSON { user_id, created_at, expires_at, ... }
  TTL: 1800 seconds (30 minutes)

PostgreSQL:
  Table: sessions
  Columns: session_id, user_id, created_at, expires_at, ...
  Purpose: Long-term storage and analytics
```

#### Question 3: How do you shard an authentication database?

**Answer Framework:**

```text
Sharding Key Decision:

Option 1: Shard by user_id
✅ Pros: All user data on one shard (no joins)
✅ Pros: Even distribution
❌ Cons: Email login requires lookup table
└─ Verdict: Best choice

Option 2: Shard by email
✅ Pros: Login by email is fast
❌ Cons: Uneven distribution (gmail.com users)
❌ Cons: User can change email
└─ Verdict: Not recommended

Option 3: Shard by geography
✅ Pros: Data residency (GDPR)
✅ Pros: Low latency
❌ Cons: User travels (which shard?)
└─ Verdict: Good for global apps

Implementation:
├─ Primary shard key: user_id % 64
├─ Global lookup service: email → user_id, shard_id
├─ Co-locate related data: User + sessions + tokens on same shard
└─ Global tables: Roles and permissions replicated to all shards

Challenges:
├─ Cross-shard queries: Avoid with good design
├─ Global transactions: Use saga pattern
└─ Shard rebalancing: Virtual shards (1024 virtual → 64 physical)
```

---

### ✅ Key Takeaways

- **Never store plaintext passwords**: Always use bcrypt or Argon2 with high cost factor
- **Separate concerns**: PostgreSQL for consistency, Redis for speed, Cassandra for logs
- **Index strategically**: Every auth query should use an index (<10ms)
- **Encryption in layers**: Database encryption + application encryption + KMS
- **GDPR requires design**: Right to access, right to be forgotten, data residency
- **Audit everything**: Every login, permission check, and security event
- **Sharding is geography**: User data stays in their region for compliance
- **Backup for disaster**: PITR, cross-region replication, quarterly DR tests

---

### 🎯 Practice Exercise

**Scenario:** You're designing the database for "SecureAuth," an authentication service specifically for healthcare providers with strict HIPAA compliance:

**Requirements:**
1. **PHI Storage**: Store patient health information (encrypted)
2. **Audit Trail**: Must track every access to patient data (who, when, what)
3. **Role-Based Access**: Doctors, nurses, admins have different permissions
4. **Consent Management**: Track patient consent for data access
5. **Emergency Access**: Break-glass access for emergencies (with audit)
6. **Data Retention**: Keep audit logs for 7 years (HIPAA requirement)
7. **Geographic Compliance**: US patient data stays in US

**Your Task:**

1. **Design the Complete Schema:**
   ```sql
   -- Design tables for:
   -- 1. users (with encrypted PHI)
   -- 2. patient_consent (granular permissions)
   -- 3. emergency_access_log
   -- 4. audit_trail (7-year retention)
   -- 5. roles_permissions (healthcare-specific)
   ```

2. **Data Encryption Strategy:**
   - Which fields need encryption?
   - How do you manage encryption keys?
   - How do you handle key rotation with patient data?

3. **Compliance Queries:**
   - "Show all accesses to patient 123's data in last 30 days"
   - "Which users have emergency access privileges?"
   - "Export all patient data for patient 456 (HIPAA access request)"

4. **Scaling Strategy:**
   - 100K healthcare providers
   - 10M patients
   - 1B audit log entries per year
   - How do you shard? What challenges arise?

5. **Disaster Recovery:**
   - RPO (Recovery Point Objective): <1 hour
   - RTO (Recovery Time Objective): <4 hours
   - Design backup and replication strategy

**Bonus Challenge:**
- How do you implement "break-glass" emergency access that bypasses normal permissions but leaves a clear audit trail?
- Design a system to detect abnormal access patterns (e.g., doctor accessing 100 patient records in 5 minutes)
- How do you handle a security breach where encryption keys are compromised?

---

## Section 8: Token Management & Session Handling

### What You'll Learn

By the end of this section, you'll be able to:
- Explain JWT structure (header, payload, signature) and validation process
- Compare access tokens vs refresh tokens and their security lifecycles
- Design token storage strategies across different client types
- Implement token rotation and secure refresh flows
- Choose between stateless (JWT) and stateful (session) authentication
- Protect against token theft, XSS, and CSRF attacks
- Design token revocation strategies at scale
- Implement key rotation for JWT signing keys

### Why This Matters

Token management is the heart of modern authentication systems. A poorly designed token system can lead to:
- **Security breaches**: Auth0 reported that 43% of security incidents involve token theft
- **Scalability issues**: Stateful sessions can become a bottleneck at scale
- **Poor user experience**: Frequent re-logins frustrate users
- **Compliance violations**: GDPR requires secure token handling and user data protection

Real-world impact:
- **Okta** handles 15 billion authentications per month using sophisticated token management
- **GitHub** uses short-lived access tokens (15 min) with refresh tokens for security
- **Zoom** experienced a 2020 breach partly due to improper token handling, leading to complete redesign

---

### 🟢 For Beginners: Token Fundamentals

#### What is a Token?

Think of a token like a **wristband at an amusement park**:

```text
Your Experience at "FunPark":
├─ Step 1: Buy ticket at entrance (authenticate with username/password)
├─ Step 2: Receive wristband (get token)
├─ Step 3: Show wristband at rides (use token to access resources)
└─ Step 4: Wristband expires at midnight (token expiration)

Benefits:
✅ Don't need ticket office at every ride (no need to check password every time)
✅ Wristband shows what you're allowed to do (VIP, regular, child)
✅ Can't easily copy wristband (cryptographic signature)
✅ Park can revoke wristband if needed (token revocation)
```

#### Understanding JWT (JSON Web Tokens)

JWT is like a **sealed letter with three parts**:

```text
JWT Structure:
┌─────────────────────────────────────────────────────────┐
│ HEADER.PAYLOAD.SIGNATURE                                │
├─────────────────────────────────────────────────────────┤
│ eyJhbGc.eyJ1c2VyX2lkIjox.SflKxwRJSMeKKF2Q               │
└─────────────────────────────────────────────────────────┘
     │          │              │
     │          │              └── Signature (seal)
     │          └── Payload (your information)
     └── Header (type of seal)
```

**Real JWT Example:**

```json
// HEADER (Algorithm and token type)
{
  "alg": "RS256",        // RSA encryption
  "typ": "JWT"           // Token type
}

// PAYLOAD (Your data - called "claims")
{
  "user_id": "12345",
  "email": "alice@example.com",
  "roles": ["user", "premium"],
  "iss": "auth.myapp.com",     // Issuer (who created token)
  "iat": 1705929600,           // Issued at (timestamp)
  "exp": 1705933200            // Expires (1 hour later)
}

// SIGNATURE (Cryptographic seal)
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret_key
)

// Complete JWT:
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzNDUiLCJlbWFpbCI6ImFsaWNlQGV4YW1wbGUuY29tIiwicm9sZXMiOlsidXNlciIsInByZW1pdW0iXSwiaXNzIjoiYXV0aC5teWFwcC5jb20iLCJpYXQiOjE3MDU5Mjk2MDAsImV4cCI6MTcwNTkzMzIwMH0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### How Token Validation Works

**The Restaurant Analogy:**

```text
You go to fancy restaurant:

1. Hostess gives you a table number card (token)
2. Card has:
   ├─ Table number (user_id)
   ├─ VIP status (roles)
   ├─ Valid until 10 PM (expiration)
   └─ Restaurant stamp (signature)

3. When waiter serves you, they check:
   ├─ Is stamp genuine? (verify signature)
   ├─ Is card still valid? (check expiration)
   └─ Does table match? (verify claims)

4. If all checks pass → You get service
   If any fails → Back to hostess!
```

**Technical Flow:**

```text
User Accesses Protected Resource:

[User] → "GET /api/profile" + Token → [API Server]
                                            ↓
                                    Validation Steps:
                                            ↓
                        ┌───────────────────────────────┐
                        │ 1. Decode token without verify│
                        │    (Read header, payload)     │
                        └───────────┬───────────────────┘
                                    ↓
                        ┌───────────────────────────────┐
                        │ 2. Verify signature           │
                        │    Use public key to check    │
                        │    token hasn't been tampered │
                        └───────────┬───────────────────┘
                                    ↓
                        ┌───────────────────────────────┐
                        │ 3. Check expiration           │
                        │    exp > current_time?        │
                        └───────────┬───────────────────┘
                                    ↓
                        ┌───────────────────────────────┐
                        │ 4. Verify issuer              │
                        │    iss == "auth.myapp.com"?   │
                        └───────────┬───────────────────┘
                                    ↓
                        ┌───────────────────────────────┐
                        │ 5. Check audience (optional)  │
                        │    aud == "api.myapp.com"?    │
                        └───────────┬───────────────────┘
                                    ↓
                            ✅ Valid Token
                                    ↓
                        Extract user_id, roles from payload
                                    ↓
                            Process Request
```

#### Access Tokens vs Refresh Tokens

**The Hotel Key Card Analogy:**

```text
Two Types of Keys:

ACCESS TOKEN (Room Key Card):
├─ Opens your room (access resources)
├─ Valid for 1 hour
├─ If lost, expires quickly (limited damage)
└─ Use this 100 times per day

REFRESH TOKEN (Master Key at Front Desk):
├─ Can't open rooms directly
├─ Used to get new room key cards
├─ Valid for 30 days
├─ Stored securely at front desk (not in pocket)
└─ Use this 1-2 times per day

Why Both?
├─ Short access token lifetime = Less theft risk
├─ Long refresh token lifetime = Better UX (no constant login)
└─ Compromise: Security + Convenience
```

**Typical Lifetimes:**

```text
Access Token:
├─ Lifetime: 15 minutes - 1 hour
├─ Use: Every API request
├─ Storage: Memory (best) or localStorage
└─ If stolen: Expires quickly

Refresh Token:
├─ Lifetime: 7 days - 90 days
├─ Use: Only when access token expires
├─ Storage: HTTP-only cookie (secure)
└─ If stolen: Can be detected and revoked
```

#### Token Storage: Where to Keep Tokens?

**The Wallet Analogy:**

```text
Where do you keep your money?

Option 1: IN YOUR HAND (JavaScript memory)
├─ Pros: Lost if you close your hand (tab closes)
├─ Pros: Nobody can pickpocket while you sleep (XSS can't steal)
├─ Cons: Have to get money again if you put hand down (refresh on reload)
└─ Best for: Access tokens

Option 2: IN YOUR WALLET (localStorage)
├─ Pros: Keep money even if you close hand
├─ Cons: Pickpocket can steal (XSS attacks)
├─ Cons: Anyone looking over shoulder can see (accessible to all scripts)
└─ Best for: Nothing sensitive (use with caution!)

Option 3: IN HOTEL SAFE (HTTP-only cookies)
├─ Pros: Can't be stolen by pickpocket (XSS can't access)
├─ Pros: Hotel gives you money when needed (browser sends automatically)
├─ Cons: Need CSRF protection (fake hotel key)
└─ Best for: Refresh tokens
```

**Storage Comparison Table:**

```text
╔══════════════════╦═══════════════╦═══════════════╦════════════════╗
║ Storage Type     ║ XSS Safe?     ║ CSRF Safe?    ║ Best For       ║
╠══════════════════╬═══════════════╬═══════════════╬════════════════╣
║ Memory           ║ ✅ Yes        ║ ✅ Yes        ║ Access tokens  ║
║ localStorage     ║ ❌ No         ║ ✅ Yes        ║ ⚠️ Avoid       ║
║ sessionStorage   ║ ❌ No         ║ ✅ Yes        ║ ⚠️ Avoid       ║
║ HTTP-only Cookie ║ ✅ Yes        ║ ❌ No*        ║ Refresh tokens ║
╚══════════════════╩═══════════════╩═══════════════╩════════════════╝
*Use SameSite=Strict to protect against CSRF
```

#### Simple Token Flow Example

**Login and Access Flow:**

```text
Step-by-Step Token Journey:

1. USER LOGS IN:
   [User] → POST /login {username, password} → [Auth Server]
                                                      ↓
                                                  Verify
                                                      ↓
   [User] ← access_token (15 min)              ← [Auth Server]
            refresh_token (30 days)

2. USER ACCESSES RESOURCE:
   [User] → GET /api/profile
            Header: Authorization: Bearer <access_token>
                                                      ↓
   [API Server] → Validate token (fast, no database)
                                                      ↓
   [User] ← {user: "alice", email: "alice@example.com"}

3. ACCESS TOKEN EXPIRES (after 15 min):
   [User] → GET /api/profile
            Header: Authorization: Bearer <expired_token>
                                                      ↓
   [API Server] → Token expired!
                                                      ↓
   [User] ← 401 Unauthorized {error: "Token expired"}

4. USER REFRESHES TOKEN:
   [User] → POST /refresh
            Body: {refresh_token}
                                                      ↓
   [Auth Server] → Validate refresh token (check database)
                 → Check if revoked
                 → Generate new access token
                                                      ↓
   [User] ← new_access_token (fresh 15 min)

5. REPEAT STEP 2 with new token
```

---

### 🟡 For Intermediate: Production Token Management

#### JWT Deep Dive: Claims and Best Practices

**Standard Claims (Registered Claims):**

```json
{
  // Who issued this token
  "iss": "auth.myapp.com",          // Issuer
  
  // Who is this token for
  "sub": "user:12345",              // Subject (user identifier)
  "aud": "api.myapp.com",           // Audience (which service)
  
  // Time-based claims
  "iat": 1705929600,                // Issued at (Unix timestamp)
  "exp": 1705933200,                // Expires at (1 hour later)
  "nbf": 1705929600,                // Not before (can't use until this time)
  
  // Security
  "jti": "a3f2b1c9-...",            // JWT ID (unique identifier, prevent replay)
  
  // Custom claims (your application data)
  "user_id": "12345",
  "email": "alice@example.com",
  "roles": ["user", "premium"],
  "tenant_id": "acme-corp",
  
  // Token metadata
  "token_type": "access",
  "scope": "read:profile write:posts"
}
```

**Best Practices for Claims:**

```text
DO:
✅ Keep payload small (<1KB) - sent with every request
✅ Use short expiration for access tokens (15-60 min)
✅ Include minimum necessary information
✅ Use 'jti' for token revocation tracking
✅ Add 'aud' to prevent token misuse across services

DON'T:
❌ Store sensitive data (SSN, passwords, credit cards)
❌ Make tokens too long (>2KB causes issues)
❌ Use long expiration (>1 hour for access tokens)
❌ Include data that changes frequently
❌ Forget to validate all claims during verification
```

#### Token Rotation: The Interview-Winning Strategy

**Why Token Rotation?**

```text
Problem: Refresh token stolen
├─ Without rotation: Attacker uses token for 30 days
├─ With rotation: Attacker's next use gets detected
└─ Benefit: Limits damage from theft

Rotation Strategy:
└─ Every time refresh token is used, issue new one
  └─ Old refresh token becomes invalid
    └─ If old token used again → Theft detected!
```

**Implementation Flow:**

```mermaid
sequenceDiagram
    participant User
    participant App
    participant AuthServer
    participant Redis
    
    Note over User,Redis: Initial Login
    User->>AuthServer: POST /login (credentials)
    AuthServer->>Redis: Store refresh_token_v1 (RT1)
    AuthServer->>User: access_token + RT1
    
    Note over User,Redis: Access Token Expires
    User->>AuthServer: POST /refresh (RT1)
    AuthServer->>Redis: Check RT1 valid?
    Redis->>AuthServer: Valid
    AuthServer->>Redis: Invalidate RT1, Store RT2
    AuthServer->>User: new_access_token + RT2
    
    Note over User,Redis: Token Theft Detected
    User->>AuthServer: POST /refresh (RT2)
    AuthServer->>Redis: Invalidate RT2, Store RT3
    AuthServer->>User: new_access_token + RT3
    
    Note over User,Redis: Attacker Uses Old Token
    User->>AuthServer: POST /refresh (RT1) [stolen token]
    AuthServer->>Redis: RT1 already used!
    AuthServer->>Redis: Revoke entire token family (RT1,RT2,RT3)
    AuthServer->>User: 401 + Security Alert
    AuthServer->>User: Email: "Suspicious activity detected"
```

**Token Rotation Implementation:**

```json
// Refresh token structure in Redis
{
  "token_family_id": "fam_abc123",     // Links all rotations
  "token_version": 3,                   // Current version
  "user_id": "12345",
  "issued_at": 1705929600,
  "expires_at": 1708521600,             // 30 days
  "last_used_at": 1705932000,
  "previous_tokens": [
    "rt_v1_hash",                       // Used at 2025-01-15 10:00
    "rt_v2_hash"                        // Used at 2025-01-15 14:00
  ],
  "device_id": "device_xyz",
  "ip_address": "192.168.1.100"
}

// When refresh token is used:
1. Verify token hasn't been used before
2. If already used → SECURITY BREACH
   └─ Revoke entire token_family_id
   └─ Force user re-login
   └─ Send security alert
3. If valid → Issue new token
   └─ Increment token_version
   └─ Invalidate current token
   └─ Add current token to previous_tokens
```

#### Stateless vs Stateful: The Critical Decision

**Decision Matrix:**

```text
STATELESS TOKENS (JWT):
┌────────────────────────────────────────────────────────┐
│ Token contains all information (self-contained)        │
│ No database lookup on every request                    │
└────────────────────────────────────────────────────────┘

Pros:
✅ Fast validation (<1ms, no DB query)
✅ Scales horizontally (no shared state)
✅ Works across microservices
✅ Reduces database load (10K→0 queries/sec)

Cons:
❌ Can't revoke immediately (must wait for expiration)
❌ Payload visible (base64 decoded easily)
❌ Size larger (1-2KB) than session ID (16 bytes)
❌ Logout doesn't work (token valid until expiration)

Best For:
├─ Microservices architecture
├─ High-scale read-heavy systems
├─ Short-lived tokens (15-60 min)
└─ API-to-API communication

STATEFUL SESSIONS (Session ID + Redis):
┌────────────────────────────────────────────────────────┐
│ Token is random ID, data stored server-side            │
│ Database lookup on every request                       │
└────────────────────────────────────────────────────────┘

Pros:
✅ Instant revocation (delete from Redis)
✅ Smaller token size (16 bytes)
✅ No data exposure (everything server-side)
✅ True logout (delete session immediately)

Cons:
❌ Requires database lookup (adds ~2-5ms latency)
❌ Harder to scale (sticky sessions or shared Redis)
❌ Single point of failure (Redis down = auth down)
❌ Not suitable for microservices (shared state)

Best For:
├─ Monolithic applications
├─ Security-critical systems
├─ Need instant logout/revocation
└─ Long-lived sessions
```

**Hybrid Approach (Best of Both Worlds):**

```text
RECOMMENDED: Use BOTH

Access Token (JWT):
├─ Lifetime: 15 minutes
├─ Stateless validation
├─ No database lookup
└─ Fast, scales well

Refresh Token (Stateful):
├─ Lifetime: 30 days
├─ Stored in Redis with metadata
├─ Can be revoked instantly
└─ Used rarely (once per 15 min)

Result:
├─ 99.9% of requests use JWT (fast, stateless)
├─ 0.1% of requests use Redis (refresh)
├─ Can revoke user by invalidating refresh token
└─ Best security + performance balance
```

#### Token Revocation Strategies

**Challenge: How to Revoke Stateless JWT?**

```text
Problem:
├─ User logs out at 10:00 AM
├─ JWT valid until 10:15 AM (15 min expiration)
├─ JWT is stateless (no database check)
└─ How to prevent use between 10:00-10:15?
```

**Strategy 1: Token Blacklist (Simple but Costly)**

```text
Implementation:
├─ Store revoked tokens in Redis
├─ On every request: Check if token in blacklist
└─ If in blacklist → Reject

Data Structure:
Key: "blacklist:<jti>"
Value: {"revoked_at": 1705929600, "user_id": "12345"}
TTL: Match token expiration (auto-cleanup)

Redis Storage:
blacklist:a3f2b1c9 → expires in 900 seconds

Pros:
✅ Works with existing JWT system
✅ Precise control (individual token revocation)

Cons:
❌ Requires Redis lookup on EVERY request (defeats stateless purpose)
❌ Blacklist grows large (10M active tokens = 10M Redis keys)
❌ Adds latency (2-5ms per request)

When to Use:
└─ Small scale (<10K concurrent users)
└─ Already doing Redis lookup for other reasons
```

**Strategy 2: Short-Lived Tokens with Refresh (Recommended)**

```text
Implementation:
├─ Access token: 15 minutes (stateless)
├─ Refresh token: 30 days (stateful in Redis)
├─ Revoke by deleting refresh token
└─ Access token expires naturally (max 15 min exposure)

Flow:
User logs out at 10:00:
├─ Delete refresh token from Redis (instant)
├─ Access token valid until 10:15 (acceptable risk)
└─ User can't get new access token (refresh revoked)

Pros:
✅ No database lookup for access tokens (scales!)
✅ Limited exposure window (15 min max)
✅ Simple implementation

Cons:
❌ 15-minute window where token still works
❌ Not suitable for high-security scenarios

When to Use:
└─ Most production systems (OAuth 2.0 standard)
└─ Acceptable security trade-off
```

**Strategy 3: Token Version Number (Granular Control)**

```text
Implementation:
├─ Store token_version in database (one per user)
├─ Include token_version in JWT payload
├─ On critical operations: Verify token_version matches DB
└─ To revoke: Increment user's token_version in DB

JWT Payload:
{
  "user_id": "12345",
  "token_version": 5,  // Must match DB
  "exp": 1705933200
}

Database:
users table:
user_id  | token_version | email
---------|---------------|-------------------
12345    | 5             | alice@example.com

Revocation:
UPDATE users SET token_version = 6 WHERE user_id = 12345;
-- All tokens with version 5 now invalid

Pros:
✅ Revoke all user tokens with one DB update
✅ No blacklist maintenance
✅ Only check DB on critical operations (not every request)

Cons:
❌ Revokes ALL user tokens (all devices)
❌ Still requires DB check for full validation

When to Use:
└─ Password change (revoke all sessions)
└─ Security incident (terminate user access)
└─ Permission changes (force re-authentication)
```

**Strategy 4: Bloom Filter (Advanced Optimization)**

```text
Implementation:
├─ Use Bloom filter for blacklist (probabilistic data structure)
├─ 10M tokens → 12 MB memory (99.9% accuracy)
├─ Check Bloom filter first (fast)
├─ If "maybe in blacklist" → Check Redis
└─ If "definitely not in blacklist" → Skip Redis

Memory Comparison:
Standard Redis blacklist: 10M keys × 200 bytes = 2 GB
Bloom filter: 10M elements × 10 bits = 12 MB
└─ 166x memory reduction!

Pros:
✅ Massive memory savings
✅ Very fast checks (~1μs)
✅ Scales to billions of tokens

Cons:
❌ False positives possible (1 in 1000)
❌ Can't remove from Bloom filter (use time-based partitions)
❌ Complex implementation

When to Use:
└─ Massive scale (100M+ tokens)
└─ Memory constrained
```

**Comparison Table:**

```text
╔═══════════════════════╦══════════╦═════════╦════════════╦═════════════╗
║ Strategy              ║ Latency  ║ Memory  ║ Accuracy   ║ Complexity  ║
╠═══════════════════════╬══════════╬═════════╬════════════╬═════════════╣
║ Blacklist (Redis)     ║ 2-5ms    ║ High    ║ 100%       ║ Low         ║
║ Short-lived + Refresh ║ <1ms     ║ Low     ║ ~98%*      ║ Low         ║
║ Token Version         ║ Variable ║ Minimal ║ 100%       ║ Medium      ║
║ Bloom Filter          ║ <1ms     ║ Very Low║ 99.9%      ║ High        ║
╚═══════════════════════╩══════════╩═════════╩════════════╩═════════════╝
*15-minute exposure window after revocation
```

#### Token Storage in Different Client Types

**Web Browser (SPA):**

```javascript
// OPTION 1: Memory Storage (Most Secure for Access Token)
class TokenManager {
  constructor() {
    this.accessToken = null;  // In-memory only
  }
  
  setAccessToken(token) {
    this.accessToken = token;
  }
  
  getAccessToken() {
    return this.accessToken;
  }
  
  clearTokens() {
    this.accessToken = null;
  }
}

// OPTION 2: HTTP-only Cookie (Best for Refresh Token)
// Set by server response:
Set-Cookie: refresh_token=abc123; HttpOnly; Secure; SameSite=Strict; Path=/auth/refresh; Max-Age=2592000

// Flags explained:
HttpOnly    → JavaScript can't access (XSS protection)
Secure      → Only sent over HTTPS
SameSite    → Prevents CSRF attacks
Path        → Only sent to /auth/refresh endpoint
Max-Age     → Expires in 30 days

// Client-side (automatic):
// Browser automatically sends cookie with requests to /auth/refresh
fetch('/auth/refresh', {
  method: 'POST',
  credentials: 'include'  // Include cookies
});
```

**Mobile App (iOS/Android):**

```text
Secure Storage Options:

iOS: Keychain
├─ Hardware-backed encryption
├─ Survives app reinstall
└─ Protected by device passcode

Android: KeyStore
├─ Hardware-backed encryption (on supported devices)
├─ Biometric protected
└─ Per-app isolated

Implementation:
├─ Store access token in KeyStore/Keychain
├─ Store refresh token in KeyStore/Keychain
├─ Never store in SharedPreferences (Android) or UserDefaults (iOS)
└─ Use encrypted database if KeyStore unavailable

Token Refresh Strategy:
├─ Check token expiration before each request
├─ Refresh proactively (5 min before expiration)
├─ Queue requests during refresh
└─ Handle offline scenarios (cache valid token)
```

**Server-to-Server:**

```text
Service Accounts:

Option 1: Long-lived API Keys
├─ Store in environment variables
├─ Rotate quarterly
├─ Use KMS for encryption at rest
└─ Audit all usage

Option 2: Short-lived JWTs
├─ Generate token on startup
├─ Refresh every hour
├─ Use service account credentials
└─ Scope to minimum permissions

Option 3: mTLS (Mutual TLS)
├─ Certificate-based authentication
├─ No tokens needed
├─ Hardware-backed security
└─ Best for high-security environments
```

---

### 🔴 For Advanced: Enterprise Token Security

#### Multi-Factor Authentication with Tokens

**Step-Up Authentication Flow:**

```text
Scenario: User wants to change password (high-risk operation)

Standard Flow:
1. User authenticated with regular token
2. Requests password change
3. System requires additional verification (MFA)

Implementation:
┌─────────────────────────────────────────────────────────┐
│ Regular Access Token (Low Assurance)                    │
│ {                                                        │
│   "user_id": "12345",                                   │
│   "auth_level": "single-factor",   // Only password    │
│   "exp": 1705933200                                     │
│ }                                                        │
│ → Can access: Profile, read posts                       │
│ → Cannot access: Change password, billing               │
└─────────────────────────────────────────────────────────┘

After MFA Challenge:
┌─────────────────────────────────────────────────────────┐
│ Elevated Access Token (High Assurance)                  │
│ {                                                        │
│   "user_id": "12345",                                   │
│   "auth_level": "multi-factor",    // Password + OTP   │
│   "mfa_verified_at": 1705930000,                        │
│   "exp": 1705930900                // 15 min only!     │
│ }                                                        │
│ → Can access: Everything, including sensitive ops       │
└─────────────────────────────────────────────────────────┘

Backend Validation:
def require_mfa(request):
    token = verify_token(request.headers['Authorization'])
    
    if token['auth_level'] != 'multi-factor':
        return {
            "error": "MFA required",
            "mfa_challenge": {
                "methods": ["totp", "sms", "push"],
                "challenge_id": "chall_abc123"
            }
        }, 403
    
    # Check MFA verification time (must be recent)
    mfa_age = current_time() - token['mfa_verified_at']
    if mfa_age > 900:  # 15 minutes
        return {"error": "MFA verification expired"}, 403
    
    # Proceed with sensitive operation
    return perform_sensitive_operation()
```

#### Token Binding: Preventing Token Theft

**Challenge:** Stolen token can be used by attacker from different device/location.

**Solution: Token Binding**

```text
Concept: Bind token to specific device/browser characteristics

Option 1: Device Fingerprint Binding
┌─────────────────────────────────────────────────────────┐
│ Generate device fingerprint:                             │
│ - User agent                                             │
│ - Screen resolution                                      │
│ - Timezone                                               │
│ - Canvas fingerprint                                     │
│ - WebGL renderer                                         │
│ Hash → device_id: "fp_abc123"                           │
└─────────────────────────────────────────────────────────┘

JWT Payload:
{
  "user_id": "12345",
  "device_id": "fp_abc123",  // Bound to device
  "exp": 1705933200
}

Validation:
1. User makes request with token
2. Calculate device fingerprint from request
3. Compare with device_id in token
4. If mismatch → Reject token

Limitations:
❌ Browser updates change fingerprint
❌ VPN/proxy changes fingerprint
❌ Privacy concerns (tracking)
⚠️ Use with caution
```

**Option 2: TLS Certificate Binding (Most Secure)**

```text
Concept: Bind token to TLS client certificate

Flow:
1. Client authenticates with mTLS certificate
2. Extract certificate thumbprint
3. Include thumbprint in JWT

JWT Payload:
{
  "user_id": "12345",
  "cnf": {  // Confirmation claim (RFC 8705)
    "x5t#S256": "bwcK0esc3ACC3DB"  // Cert thumbprint
  },
  "exp": 1705933200
}

Validation:
1. Extract client certificate from TLS handshake
2. Calculate thumbprint
3. Compare with cnf.x5t#S256 in token
4. If mismatch → Token stolen/replayed

Benefits:
✅ Cryptographically secure binding
✅ Prevents token replay attacks
✅ Industry standard (OAuth 2.0 mTLS)

Use Cases:
└─ Banking applications
└─ Healthcare (HIPAA)
└─ Government systems
```

**Option 3: IP Address Binding (Simple but Limited)**

```text
JWT Payload:
{
  "user_id": "12345",
  "ip_address": "192.168.1.100",
  "ip_hash": "hash of IP + secret",
  "exp": 1705933200
}

Validation:
if request.ip != token.ip_address:
    if security_level == "high":
        reject_token()
    else:
        send_security_alert()
        allow_with_warning()

Limitations:
❌ Mobile users change IPs frequently
❌ Corporate NAT (many users, same IP)
❌ VPN changes IP legitimately
└─ Only use as additional signal, not primary security
```

#### Key Rotation for JWT Signing

**Why Rotate Keys?**

```text
Security Risks of Static Keys:
├─ Key compromise goes undetected
├─ All historical tokens compromised
├─ Can't revoke without breaking all tokens
└─ Compliance requirements (PCI DSS, SOC 2)

Industry Standards:
├─ Auth0: Rotates every 90 days
├─ Okta: Rotates every 90 days
├─ Google: Rotates monthly
└─ Recommendation: 90 days or less
```

**Implementation: Graceful Key Rotation**

```text
Multi-Key Architecture:

Key States:
├─ ACTIVE: Signs new tokens
├─ EXPIRED: Validates old tokens, doesn't sign new
└─ REVOKED: Rejects all tokens

Key Lifecycle:
Day 0:   KEY_1 (ACTIVE)
Day 90:  KEY_2 (ACTIVE), KEY_1 (EXPIRED, validates for 15 min)
Day 90+: KEY_2 (ACTIVE), KEY_1 (REVOKED)
Day 180: KEY_3 (ACTIVE), KEY_2 (EXPIRED), KEY_1 (deleted)
```

**JWKS (JSON Web Key Set) Endpoint:**

```json
// Public endpoint: https://auth.myapp.com/.well-known/jwks.json
{
  "keys": [
    {
      "kid": "key-2025-01",      // Key ID
      "kty": "RSA",               // Key type
      "use": "sig",               // Usage: signature
      "alg": "RS256",
      "n": "0vx7agoebGcQ...",    // Public key (modulus)
      "e": "AQAB",                // Public key (exponent)
      "x5c": ["MIIDQjCC..."],    // X.509 certificate chain
      "x5t": "dGhpcyBpc...",     // X.509 thumbprint
      "status": "active"
    },
    {
      "kid": "key-2024-10",
      "kty": "RSA",
      "use": "sig",
      "alg": "RS256",
      "n": "xjlSKLSDFsd...",
      "e": "AQAB",
      "status": "expired",        // Still validates tokens
      "expires_at": 1705929600
    }
  ]
}
```

**Rotation Implementation:**

```text
[HLD Note: Detailed implementation removed for interview focus]

High-Level Architecture:

Signing Service:
├─ Component: Key Management Service (KMS)
├─ Purpose: Generate, store, rotate signing keys
├─ Technology: AWS KMS, HashiCorp Vault, Azure Key Vault
└─ Key concept: Focus on rotation strategy, not implementation

Key Selection Logic:
1. Get active key from KMS
2. Sign token with active key
3. Include "kid" (key ID) in JWT header

Validation Logic:
1. Extract "kid" from JWT header
2. Fetch public key from JWKS endpoint (cached)
3. Verify signature with public key
4. Check key status (active or expired)
5. Reject if key is revoked

Rotation Procedure:
1. Generate new key pair (KMS)
2. Mark new key as ACTIVE
3. Mark old key as EXPIRED
4. Wait for max token lifetime (15 min)
5. Mark old key as REVOKED
6. Remove old key after 90 days (audit compliance)
```

**Key Storage Security:**

```text
DO:
✅ Store private keys in HSM (Hardware Security Module)
✅ Use KMS with audit logging
✅ Separate key access by environment (prod keys ≠ dev keys)
✅ Require multi-party approval for key access
✅ Encrypt keys at rest (even in KMS)

DON'T:
❌ Store keys in code repository
❌ Store keys in environment variables (use secret manager)
❌ Reuse keys across environments
❌ Give developers access to production keys
❌ Log private keys (even encrypted)
```

#### Defense in Depth: Layered Token Security

**Layer 1: Token Generation**

```text
Secure Token Properties:
├─ Random, unpredictable (cryptographically secure RNG)
├─ Minimum 128 bits entropy (refresh tokens)
├─ Include rate limiting identifier
└─ Bind to device/IP (conditional)

Example:
refresh_token = base64url(
    random_bytes(32) +           # 256 bits entropy
    hmac(user_id, secret) +      # Integrity check
    timestamp                     # Replay prevention
)
```

**Layer 2: Token Transmission**

```text
Security Measures:
✅ HTTPS only (TLS 1.3)
✅ Certificate pinning (mobile apps)
✅ Token in Authorization header (not URL)
✅ Short TTL for tokens in transit
❌ Never send tokens in URL query parameters (logged!)
❌ Never send tokens in HTTP referrer header
```

**Layer 3: Token Storage**

```text
Client-Side:
├─ Access token: Memory only (cleared on tab close)
├─ Refresh token: HTTP-only cookie (XSS protection)
└─ Never localStorage (XSS vulnerable)

Server-Side:
├─ Hash tokens before storing (bcrypt with cost 12)
├─ Encrypt at rest (AES-256-GCM)
├─ Store in Redis with TTL (auto-cleanup)
└─ Separate database from main application DB
```

**Layer 4: Token Validation**

```text
Multi-Step Validation:
1. Signature verification (cryptographic)
2. Expiration check (timestamp)
3. Issuer verification (iss claim)
4. Audience verification (aud claim)
5. Scope verification (permissions)
6. Rate limit check (per token)
7. Anomaly detection (unusual usage pattern)

Example Validation Logic:
def validate_access_token(token, required_scope):
    # Step 1: Cryptographic verification
    try:
        payload = jwt.verify(token, public_key, algorithms=['RS256'])
    except jwt.InvalidSignatureError:
        log_security_event("invalid_signature", token_id)
        raise Unauthorized("Invalid token signature")
    
    # Step 2: Time-based checks
    if payload['exp'] < current_time():
        raise Unauthorized("Token expired")
    if payload.get('nbf', 0) > current_time():
        raise Unauthorized("Token not yet valid")
    
    # Step 3: Claims verification
    if payload['iss'] != 'auth.myapp.com':
        log_security_event("invalid_issuer", payload)
        raise Unauthorized("Invalid token issuer")
    if payload['aud'] != 'api.myapp.com':
        raise Unauthorized("Invalid token audience")
    
    # Step 4: Scope check
    if required_scope not in payload.get('scope', []):
        raise Forbidden("Insufficient scope")
    
    # Step 5: Rate limiting
    if check_rate_limit(payload['jti']) > 100:
        log_security_event("rate_limit_exceeded", payload['user_id'])
        raise TooManyRequests()
    
    # Step 6: Anomaly detection (advanced)
    if detect_anomaly(payload, request):
        log_security_event("anomaly_detected", payload['user_id'])
        require_step_up_auth()
    
    return payload
```

**Layer 5: Monitoring & Alerting**

```text
Security Metrics to Track:

Token Usage Patterns:
├─ Failed validation attempts (>10/min → alert)
├─ Expired token usage (indicates token theft)
├─ Same token from multiple IPs (token sharing)
├─ Rapid token refresh (>5/min → suspicious)
└─ Token use after logout (serious breach)

Anomaly Detection:
├─ Login from new country (step-up auth)
├─ Unusual API call patterns (ML model)
├─ Access time anomaly (user usually 9-5, now 3 AM)
└─ Velocity check (10 different IPs in 1 hour)

Alerting Strategy:
High Severity (Immediate Response):
├─ Multiple failed signature validations
├─ Token reuse after rotation (theft detected)
├─ Access from sanctioned country
└─ Privilege escalation attempt

Medium Severity (Review within 1 hour):
├─ Unusual access patterns
├─ Token sharing detected
├─ High refresh rate
└─ Failed MFA challenges

Low Severity (Daily Review):
├─ Expired token usage
├─ Minor anomalies
└─ Rate limit violations
```

#### Session Fixation Prevention

**Attack Scenario:**

```text
Without Protection:
1. Attacker gets session ID from legitimate site
2. Tricks user into using that session ID (phishing link)
3. User authenticates with attacker's session ID
4. Attacker now has authenticated session!

Example:
Attacker: https://myapp.com/login?session=ATTACKER_SESSION
User clicks, logs in
└─ User now authenticated in ATTACKER_SESSION
└─ Attacker uses ATTACKER_SESSION to access user account
```

**Prevention Strategy:**

```text
Defense 1: Regenerate Session ID After Login

Before Login:
session_id: "anonymous_abc123"

After Login:
session_id: "authenticated_xyz789"  // New ID!
└─ Old session_id invalidated
└─ Attacker's pre-set session ID useless

Implementation:
def login(username, password):
    user = verify_credentials(username, password)
    if user:
        old_session_id = request.session.id
        new_session_id = generate_secure_session_id()
        
        # Transfer session data to new ID
        migrate_session(old_session_id, new_session_id)
        
        # Invalidate old session
        delete_session(old_session_id)
        
        # Set new session cookie
        response.set_cookie('session_id', new_session_id)
        
        return response

Defense 2: Session Binding

Bind session to:
├─ User-Agent (browser fingerprint)
├─ IP address (with tolerance for mobile)
├─ TLS session ID (if available)
└─ Reject if mismatch detected

Defense 3: Short Session Lifetime

Public WiFi Scenario:
├─ Session expires after 15 minutes of inactivity
├─ Absolute timeout after 2 hours
└─ Forces re-authentication frequently
```

---

### Real-World Example: Auth0's Token Management

**Auth0's Token Architecture (as of 2024):**

```text
Token Types:
├─ Access Token (JWT)
│   ├─ Lifetime: Configurable (default 24 hours, recommend 15 min)
│   ├─ Size: ~1.5 KB average
│   ├─ Validation: Stateless (no DB lookup)
│   └─ Revocation: Via token version or blacklist
│
├─ Refresh Token (Opaque)
│   ├─ Lifetime: Configurable (7 days to 100 days)
│   ├─ Rotation: Automatic on each use
│   ├─ Validation: Database lookup required
│   └─ Revocation: Instant (delete from DB)
│
├─ ID Token (JWT)
│   ├─ Lifetime: Same as access token
│   ├─ Purpose: User profile information (OIDC)
│   ├─ Contents: name, email, picture, custom claims
│   └─ Use case: Display user info, not API access
│
└─ Client Credentials Token (M2M)
    ├─ Lifetime: 24 hours
    ├─ Purpose: Service-to-service authentication
    ├─ Validation: Stateless
    └─ Revocation: Via API

Evolution Timeline:

2016 - V1:
├─ Access tokens: 24-hour lifetime (too long!)
├─ No automatic rotation
├─ Logout didn't work (JWT still valid)
└─ Problem: Many security incidents

2018 - V2:
├─ Introduced refresh token rotation
├─ Recommended 15-min access tokens
├─ Added token introspection endpoint
└─ Improvement: 70% reduction in token theft impact

2020 - V3:
├─ Refresh token reuse detection
├─ Anomaly detection (ML-based)
├─ Automatic token family revocation
└─ Result: 95% reduction in successful attacks

2023 - Current:
├─ Device Code Flow for IoT
├─ Pushed Authorization Requests (PAR)
├─ Token Binding support
├─ JWT secured authorization request (JAR)
└─ Stats: 15B authentications/month, 99.99% uptime

Scale Numbers (2024):
├─ 15 billion authentications/month
├─ 50 million monthly active users
├─ <20ms token validation (P99)
├─ 99.99% availability
└─ Token theft detection: 99.7% accuracy
```

**Key Lessons from Auth0:**

```text
1. Start with Short-Lived Tokens
   └─ Auth0 learned the hard way: Long-lived JWTs = Security nightmare
   └─ Recommendation: 15 minutes max for access tokens

2. Refresh Token Rotation is Essential
   └─ Automatic rotation on every use
   └─ Reuse detection prevents token theft

3. Multiple Layers of Defense
   ├─ Token binding
   ├─ Anomaly detection
   ├─ Rate limiting
   └─ No single point of failure

4. Monitor Everything
   └─ Auth0 tracks 200+ security metrics
   └─ ML models detect unusual patterns
   └─ Alert within 10 seconds of breach attempt

5. Make It Easy for Developers
   └─ SDKs handle token management automatically
   └─ Secure defaults (developers can't misconfigure)
   └─ Clear documentation with security warnings
```

---

### 🤔 Think About It

1. **For Beginners:** You're building a mobile app. Your friend suggests storing the access token in localStorage so it persists after app restarts. Why is this a bad idea? What should you do instead?

2. **For Intermediate:** Your JWT access tokens are 2KB in size and sent with every request. With 10,000 requests per second, you're transferring 20 MB/s just in tokens. How would you reduce token size while maintaining security?

3. **For Advanced:** You've implemented refresh token rotation. An attacker steals both the access token AND the refresh token at the same time. Your rotation strategy detects the theft when the attacker uses the refresh token. But the attacker can still use the access token for 15 minutes. How would you design a system to detect this specific attack pattern and revoke access immediately?

---

### ✅ Key Takeaways

- **JWT = Self-contained**: Contains all information, no database lookup needed
- **Two-token strategy**: Short-lived access (15 min) + Long-lived refresh (30 days)
- **Token rotation prevents theft**: New token on each refresh, detect reuse
- **Storage matters**: Memory for access tokens, HTTP-only cookies for refresh
- **Stateless vs Stateful trade-off**: JWTs scale better, sessions revoke faster
- **Revocation is hard**: Use short lifetimes + stateful refresh tokens
- **Multiple defense layers**: Signature, expiration, binding, monitoring
- **Key rotation is essential**: Rotate signing keys every 90 days
- **Monitor token usage**: Anomaly detection catches 95% of breaches
- **HTTPS is mandatory**: Tokens in cleartext = Instant compromise

---

### 🎯 Practice Exercise

**Scenario:** You're designing the token management system for "HealthSecure," a HIPAA-compliant healthcare platform where doctors access patient medical records.

**Requirements:**
1. **High Security**: Medical records are extremely sensitive
2. **Compliance**: HIPAA requires audit trail of all data access
3. **User Experience**: Doctors shouldn't re-login every 15 minutes during rounds
4. **Mobile**: Doctors use mobile devices with spotty network
5. **Emergency Access**: In emergencies, doctors need immediate access (can't wait for MFA)
6. **Audit**: Every record access must be logged with doctor identity

**Your Task:**

1. **Design Token Strategy:**
   ```text
   Question: Would you use stateless JWT or stateful sessions? Why?
   Consider:
   - Audit requirements (must know who accessed what, when)
   - Revocation needs (terminate rogue doctor immediately)
   - Performance (doctors in same hospital, low latency critical)
   - Compliance (HIPAA logging requirements)
   ```

2. **Token Lifetimes:**
   ```text
   Design appropriate lifetimes for:
   - Access token: ??? minutes (justify)
   - Refresh token: ??? days (justify)
   - Emergency access token: ??? minutes (justify)
   
   Constraints:
   - Doctors do 8-hour shifts
   - Network interruptions common (need offline tolerance)
   - Patient safety > Security (in emergencies)
   ```

3. **Emergency Access Pattern:**
   ```text
   Design flow for emergency access:
   - Doctor needs immediate patient record access
   - No time for MFA
   - Must still be secure and auditable
   - Should require justification after the fact
   
   Consider:
   - How to detect abuse?
   - How to audit later?
   - What permissions during emergency?
   ```

4. **Token Revocation:**
   ```text
   Scenario: Doctor's phone is stolen at 2 PM
   Requirements:
   - Revoke access within 30 seconds
   - Can't wait for access token expiration
   - Must work even if phone is offline
   
   Design:
   - How to achieve instant revocation?
   - What about cached tokens?
   - How to handle offline scenarios?
   ```

5. **Audit Trail:**
   ```text
   Every API call must log:
   - Which doctor (user_id)
   - Which patient (patient_id)
   - What action (read, update, delete)
   - When (timestamp)
   - Device (device_id)
   - Location (IP, GPS if available)
   - Justification (why accessed)
   
   Design:
   - Where does this data come from? (Token? Request? Both?)
   - How do you prevent doctors from tampering with logs?
   - Storage: 10M records/day, 7-year retention (HIPAA)
   ```

**Bonus Challenge:**
- Design a system to detect anomalous access patterns (e.g., doctor accessing 100+ records in 10 minutes)
- How would you implement "break-glass" emergency access with automatic security review?
- Design token binding to prevent use of stolen tokens from different locations

---
