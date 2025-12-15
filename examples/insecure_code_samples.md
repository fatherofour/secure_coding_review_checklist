# Insecure Code Samples – Real‑World Scenarios
**Aligned with OWASP Top 10 & NIST Cybersecurity Framework (CSF)**

## 1. Broken Object Level Authorization (BOLA / IDOR)
**OWASP:** A01 – Broken Access Control
**NIST CSF:** PR.AC
**Real-world impact:** PII leakage, account takeover, regulatory violations

### Insecure Code (API – Python / FastAPI)
```python
@app.get("/api/users/{user_id}")
def get_user(user_id: int):
    return db.get_user(user_id)
```

### Why This Is Dangerous
* Any authenticated user can access **any user record**
* Common in APIs with predictable IDs
* Actively exploited in real breaches

### Secure Version
```python
@app.get("/api/users/{user_id}")
def get_user(user_id: int, current_user: User = Depends(auth_user)):
    if user_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Access denied")
    return db.get_user(user_id)
```

### Key Takeaway
Authorization must be enforced **server-side per object**, not just per endpoint.

## 2. SQL Injection via String Concatenation
**OWASP:** A03 – Injection
**NIST CSF:** PR.DS
**Real-world impact:** Data exfiltration, privilege escalation

### Insecure Code
```python
query = f"SELECT * FROM users WHERE email = '{email}'"
cursor.execute(query)
```

### Attack Example
```sql
' OR 1=1 --
```

### Secure Code
```python
cursor.execute(
    "SELECT * FROM users WHERE email = %s",
    (email,)
)
```

### Key Takeaway
All user input must be treated as **untrusted data**, never executable code.

## 3. Hardcoded Secrets in Source Code
**OWASP:** A02 – Cryptographic Failures
**NIST CSF:** PR.DS
**Real-world impact:** Credential compromise, cloud account takeover

### Insecure Code
```javascript
const AWS_SECRET_KEY = "AKIAxxxxxxxxxxxx";
```
### Why This Happens
* Developers testing locally
* Secrets accidentally committed
* GitHub scanning finds these fast

### Secure Code
```javascript
const AWS_SECRET_KEY = process.env.AWS_SECRET_KEY;
```
### Best Practice
* Store secrets in:
  * Environment variables
  * Vaults (AWS Secrets Manager, HashiCorp Vault)
* Rotate exposed secrets immediately

## 4. Weak Authentication Token Configuration
**OWASP:** A07 – Identification & Authentication Failures
**NIST CSF:** PR.AC
**Real-world impact:** Session hijacking

### Insecure Code
```python
jwt.encode(payload, SECRET_KEY, algorithm="HS256")
```
**Problems**
* Shared secret
* No key rotation
* Weak auditability

### Secure Code
```python
jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")
```

### Key Takeaway
Use **asymmetric signing (RS256)** for production APIs.

## 5. Sensitive Data Logged in Plain Text
**OWASP:** A09 – Security Logging & Monitoring Failures
**NIST CSF:** DE.CM
**Real-world impact:** Credential exposure during incident response

### Insecure Code
```javascript
logger.error("Login failed", { email, password });
```

### Secure Code

```javascript
logger.warn("Login failed", { email });
```

### Key Takeaway
Logs are a **high-value target** during breaches.

## 6. Missing Rate Limiting (Brute Force Risk)
**OWASP:** A01 – Broken Access Control
**NIST CSF:** PR.AC
**Real-world impact:** Credential stuffing attacks

### Insecure Code

```python
@app.post("/auth/login")
def login(credentials: LoginRequest):
    authenticate(credentials)
```

### Secure Code

```python
limiter.limit("5/minute")(login)
```

### Key Takeaway

Authentication endpoints **must** be rate-limited.

## 7. Insecure File Upload Handling
**OWASP:** A08 – Software & Data Integrity Failures
**NIST CSF:** PR.IP
**Real-world impact:** Remote Code Execution (RCE)

### Insecure Code
```python
file.save(f"/uploads/{file.filename}")
```

### Attack
Uploading:

```php
shell.php
```

### Secure Code

```python
if file.content_type not in ["image/png", "image/jpeg"]:
    raise HTTPException(status_code=400)

secure_name = uuid.uuid4().hex
file.save(f"/uploads/{secure_name}")
```

## 8. Overly Verbose Error Messages
**OWASP:** A09 – Logging & Monitoring Failures
**NIST CSF:** DE.CM
**Real-world impact:** Reconnaissance aid to attackers

### Insecure Code
```python
return {"error": str(exception)}
```

### Secure Code
```python
return {"error": "Internal server error"}
```

## 9. Insecure Docker Configuration
**OWASP:** A05 – Security Misconfiguration
**NIST CSF:** PR.IP
**Real-world impact:** Container escape, privilege abuse

### Insecure Dockerfile
```dockerfile
FROM python:latest
USER root
```

### Secure Dockerfile
```dockerfile
FROM python:3.11-slim
RUN useradd -m appuser
USER appuser
```

## 10. Missing HTTPS Enforcement
**OWASP:** A02 – Cryptographic Failures
**NIST CSF:** PR.DS
**Real-world impact:** Man-in-the-middle attacks

### Insecure Configuration
```nginx
server {
    listen 80;
}
```

### Secure Configuration
```nginx
server {
    listen 443 ssl;
    add_header Strict-Transport-Security "max-age=31536000";
}
```

## Risk Severity Summary

| Issue             | OWASP | Severity |
| ----------------- | ----- | -------- |
| BOLA / IDOR       | A01   | High     |
| SQL Injection     | A03   | Critical |
| Hardcoded Secrets | A02   | Critical |
| Weak JWT Config   | A07   | High     |
| Sensitive Logging | A09   | Medium   |
| No Rate Limiting  | A01   | High     |

## References
* OWASP Top 10 (2021)
* OWASP API Security Top 10
* NIST Cybersecurity Framework (CSF)
* NIST SP 800‑53 Rev.5
* MITRE ATT&CK (Credential Access, Initial Access)

## Feel Free to Add Yours




This document demonstrates:
* **Hands-on AppSec thinking**
* **Real production risks**
* **Audit-ready documentation**
* **Alignment with global standards**

This is exactly the level expected for:
* **EU security roles**
* **AppSec / SOC / Cloud Security**
* **ISO / audit-driven environments**
