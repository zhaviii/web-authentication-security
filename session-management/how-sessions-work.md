# How Web Sessions Work

## The Problem HTTP Has

HTTP is **stateless** — every request is completely independent. The server remembers nothing between requests. Without sessions, you'd have to send your username and password on every single page load.

Sessions solve this.

---

## The Session Lifecycle

```
1. User submits login form (username + password)
         │
         ▼
2. Server verifies credentials against database
         │
         ▼
3. Server creates a session record in memory/database:
   { session_id: "a3f9x...", user_id: 42, created: now, expires: +2h }
         │
         ▼
4. Server sends session_id to browser as a cookie:
   Set-Cookie: session=a3f9x...; HttpOnly; Secure; SameSite=Strict
         │
         ▼
5. Browser stores the cookie, sends it automatically on every request:
   GET /dashboard
   Cookie: session=a3f9x...
         │
         ▼
6. Server looks up session_id → finds user_id: 42 → serves the page
         │
         ▼
7. On logout: server deletes the session record → cookie becomes useless
```

---

## What Makes a Good Session ID?

| Requirement | Why |
|-------------|-----|
| Long (≥128 bits) | Short IDs can be brute-forced |
| Cryptographically random | Predictable IDs can be guessed |
| Not derived from user data | Never encode username or user ID in the session ID |
| Single-use after login | Generate a new ID after authentication (prevents fixation) |

---

## Where Sessions Are Stored (Server Side)

| Storage | Pros | Cons |
|---------|------|------|
| In-memory (RAM) | Fast | Lost on server restart |
| Database (Redis/SQL) | Persistent, scalable | Extra latency |
| JWT (stateless) | No server storage needed | Can't invalidate before expiry |

---

## Session Expiry

Sessions must expire. Never create eternal sessions.

- **Idle timeout** — expire after X minutes of inactivity (e.g., 30 min)
- **Absolute timeout** — expire after X hours regardless of activity (e.g., 8h)
- **Logout** — immediately invalidate the session server-side (deleting the cookie alone is not enough)

---

## Common Session Mistakes

| Mistake | Risk |
|---------|------|
| Session ID in URL (`?session=abc`) | Logged in server logs, browser history, Referer headers |
| No expiry | Stolen session valid forever |
| Same session ID before and after login | Session fixation attack |
| Only deleting cookie on logout | Session still valid server-side if attacker has it |
| Weak random ID | Guessable by attacker |
