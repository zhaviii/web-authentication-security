# Web Server Authentication Hardening Checklist

Use this checklist when reviewing or configuring any web application that handles user authentication.

---

## 🍪 Cookie Security

- [ ] Session cookies have `HttpOnly` flag set
- [ ] Session cookies have `Secure` flag set (requires HTTPS)
- [ ] Session cookies have `SameSite=Strict` or `SameSite=Lax`
- [ ] Cookie `Max-Age` or `Expires` is set (no eternal sessions)
- [ ] Session IDs are long (≥128 bits) and cryptographically random
- [ ] Session ID rotated after login (prevents session fixation)
- [ ] Logout deletes session server-side (not just clears cookie)

---

## 🔒 HTTPS / TLS

- [ ] All traffic forced to HTTPS (HTTP redirects to HTTPS)
- [ ] `Strict-Transport-Security` (HSTS) header set
- [ ] TLS 1.2 minimum (TLS 1.3 preferred)
- [ ] No mixed content (HTTP assets on HTTPS pages)
- [ ] Certificate is valid and not expired
- [ ] Certificate from trusted CA

---

## 🔑 Authentication

- [ ] Passwords stored as hashed (bcrypt/Argon2 — NOT MD5/SHA1)
- [ ] Login page protected against brute force (rate limiting / lockout)
- [ ] Username enumeration prevented (same error for "wrong user" and "wrong password")
- [ ] 2FA available and encouraged (TOTP or FIDO2 preferred)
- [ ] Password reset flow sends link to email, not password itself
- [ ] Password reset links are single-use and expire quickly (15–60 min)

---

## 🛡 HTTP Security Headers

Add these headers to all responses:

```nginx
# nginx.conf
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Content-Security-Policy "default-src 'self'";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
```

| Header | Protects Against |
|--------|-----------------|
| `X-Frame-Options: DENY` | Clickjacking |
| `X-Content-Type-Options: nosniff` | MIME type sniffing attacks |
| `Content-Security-Policy` | XSS |
| `Strict-Transport-Security` | HTTP downgrade attacks |
| `Referrer-Policy` | Sensitive URL leakage |

---

## 📋 Session Management

- [ ] Sessions expire after idle period (15–60 minutes)
- [ ] Sessions have absolute maximum lifetime (8–24 hours)
- [ ] Concurrent session limit enforced for sensitive apps
- [ ] Session invalidated on password change
- [ ] Session invalidated on logout (server-side)
- [ ] Admin sessions have shorter timeout than regular sessions

---

## 🔍 Logging & Monitoring

- [ ] Failed login attempts are logged
- [ ] Successful logins are logged (IP, timestamp, user agent)
- [ ] Account lockouts are alerted
- [ ] Unusual session activity monitored (IP change mid-session)
- [ ] Logs stored securely, not accessible to application users
