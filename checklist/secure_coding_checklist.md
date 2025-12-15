## Secure Coding Review Checklist
**Practical Application Security Controls**
**Mapped with OWASP Top 10 & NIST CSF**

## Purpose
This checklist is used to **review production-bound code** and identify security weaknesses that commonly lead to real incidents such as account takeover, data leakage, privilege escalation, and service abuse.

It assumes the application is:
* Internet-facing or internally exposed
* API-driven or web-based
* Handling authentication, user data, or business logic

## 1. Authentication Failures
**OWASP:** A07 – Identification & Authentication Failures
**NIST CSF:** PR.AC

### Review Checks
* Authentication is enforced **server-side** on all protected routes
* No endpoints rely solely on frontend checks
* Passwords are **never stored or logged in plaintext**
* Password hashing uses **bcrypt / Argon2 / PBKDF2**
* Account lockout or throttling exists after repeated failures
* MFA is enforced for privileged or admin users

### Possible Failures Seen
* Login endpoint protected, but `/api/user/me` not authenticated
* Passwords hashed but **without salt**
* Admin routes protected by frontend role checks only
* Unlimited login attempts → credential stuffing success

## 2. Authorization & Access Control
**OWASP:** A01 – Broken Access Control
**NIST CSF:** PR.AC

### Review Checks
* Object-level authorization enforced (user can only access own data)
* Role checks implemented **per request**, not globally
* Admin-only actions explicitly protected
* No reliance on user-supplied IDs for access decisions
* Authorization logic centralized (middleware/service layer)

### Possible Failures Seen
* `/api/orders/{id}` returns other users’ orders (IDOR)
* Role checked at login but not on subsequent requests
* `isAdmin=true` accepted from client payload
* Business logic bypass via undocumented endpoints

## 3. Session & Token Handling
**OWASP:** A02 – Cryptographic Failures, A07
**NIST CSF:** PR.AC

### Review Checks
* JWTs have **short expiration times**
* Refresh tokens are rotated and invalidated
* Tokens are not stored in localStorage for sensitive apps
* Logout properly invalidates sessions/tokens
* Token contents do not expose sensitive data

### Possible Failures Seen

* JWT valid for an unlimited period of time → stolen token abuse
* Tokens reused after password change
* Sensitive user roles embedded in unsigned tokens
* No revocation mechanism after compromise

## 4. Input Validation & Injection
**OWASP:** A03 – Injection
**NIST CSF:** PR.DS

### Review Checks
* All inputs validated on client & server-side
* Strict schemas enforced for APIs (types, length, format)
* Parameterized queries used everywhere
* No string concatenation in SQL or OS commands
* User-controlled data never passed directly to interpreters

### Possible Failures Seen
* SQL injection via search or filter parameters
* Command injection through file processing features
* JSON injection in poorly validated APIs
* Trusting client-side validation only

## 5. API-Specific Security Controls
**OWASP:** A01, A03, A10
**NIST CSF:** PR.AC, DE.CM

### Review Checks
* Rate limiting enforced per user/IP/token
* BOLA and BFLA protections tested
* API versioning implemented and old versions disabled
* CORS configured with strict allow-lists
* Webhooks validated using signatures

### Possible Failures Seen
* No rate limiting → API abuse & DoS
* Internal admin APIs exposed publicly
* CORS set to `*` with credentials enabled
* Webhooks accepted without verification

## 6. Cryptography & Secrets Management
**OWASP:** A02
**NIST CSF:** PR.DS

### Review Checks
* TLS enforced everywhere (no HTTP fallback)
* Secrets stored in environment variables or vaults
* No secrets committed to repo history
* Secure key rotation process exists
* Deprecated crypto algorithms avoided

### Possible Failures Seen

* API keys hardcoded in source code
* `.env` files committed accidentally
* Weak encryption used for sensitive data
* Shared secrets reused across environments

## 7. Error Handling & Information Disclosure
**OWASP:** A09 – Logging & Monitoring Failures
**NIST CSF:** DE.CM

### Review Checks
* No stack traces exposed to users
* Error messages do not reveal internal logic
* Sensitive values masked in logs
* Different error responses for users vs logs
* Debug mode disabled in production

### Possible Failures Seen
* Stack traces revealing database schema
* Error messages leaking API keys or tokens
* Verbose logging exposing PII
* Debug endpoints left enabled

## 8. Logging, Monitoring & Detection
**OWASP:** A09
**NIST CSF:** DE.CM, RS.AN

### Review Checks
* Authentication and authorization events logged
* Failed access attempts tracked
* Logs protected from tampering
* Centralized logging enabled
* Alerts configured for suspicious behavior

### Real-World Failures Seen
* Breaches undetected due to missing logs
* No alerting on brute-force attempts
* Logs overwritten or deleted by attackers
* Security events mixed with noisy app logs

## 9. Dependency & Supply Chain Risks
**OWASP:** A06 – Vulnerable and Outdated Components
**NIST CSF:** ID.SC

### Review Checks
* Dependencies scanned for known CVEs
* Outdated libraries upgraded
* Lock files present and enforced
* Unused dependencies removed
* Third-party SDKs reviewed

### Possible Failures Seen
* Exploitation via outdated framework versions
* Typosquatting packages installed
* Transitive dependencies introducing RCE
* Blind trust in third-party SDKs

## 10. Secure Configuration & Deployment
**OWASP:** A05 – Security Misconfiguration
**NIST CSF:** PR.IP

### Review Checks
* Secure defaults enforced
* Least privilege applied to services
* Sensitive endpoints not publicly exposed
* Security headers configured
* Infrastructure configs reviewed

### Possible Failures Seen
* Admin consoles exposed to internet
* Over-privileged service accounts
* Default credentials left unchanged
* Missing security headers enabling XSS/clickjacking

## 11. Incident Readiness & Recovery
**OWASP:** A09
**NIST CSF:** RS, RC

### Review Checks
* Incident response steps documented
* Logs sufficient for forensics
* Ability to revoke credentials quickly
* Post-incident fixes tracked
* Lessons learned fed back into code

### Possible Failures Seen
* No way to trace attacker actions
* Delayed response due to poor visibility
* Same vulnerability exploited repeatedly
* No remediation ownership

## Usage/Purpose
This checklist should be applied:
* During code reviews
* Before production deployment
* After security incidents
* During audit and compliance preparation

## Reviewer Notes
* Findings should be rated by **impact and likelihood**
* Fixes should be validated, not assumed
* Security controls must be **tested, not just documented**

## Add yours if There's any