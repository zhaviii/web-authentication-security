# Understanding Session-Based Attacks — A Defender's Overview

This document explains session attack techniques from a defender's perspective. Understanding how these attacks work is essential for knowing what controls to put in place.

---

## Why Sessions Are Targeted

Passwords are protected by hashing. Sessions are not. A valid session cookie gives immediate access to an authenticated account — no password cracking required.

Once authentication happens, **the session cookie IS the authentication**. This makes it a high-value target.

---

## Attack 1: Session Hijacking

**Concept:** Steal a valid session cookie and use it to impersonate the victim.

**How the cookie is obtained:**
- XSS vulnerability in the application (if `HttpOnly` is missing)
- Network interception on unencrypted connections (if `Secure` is missing)
- Malware on the victim's device reading browser storage
- Adversary-in-the-middle proxy that relays traffic and saves cookies

**What the attacker does with it:**
- Opens browser developer tools → Application → Cookies
- Replaces their own session cookie value with the stolen one
- Refreshes the page — they are now logged in as the victim

**Defender's controls:** HttpOnly, Secure, SameSite, short session expiry, HTTPS everywhere

---

## Attack 2: Session Fixation

**Concept:** Instead of stealing a session, the attacker *creates* a session ID first and tricks the victim into authenticating with it.

**Attack flow:**
1. Attacker visits login page → gets a pre-auth session ID (`session=ATTACKER_ID`)
2. Attacker tricks victim into using that same URL (e.g., via link: `?session=ATTACKER_ID`)
3. Victim logs in — server sees existing session ID and marks it as authenticated
4. Attacker already has `ATTACKER_ID` → now they're authenticated too

**Defender's control:** Always generate a **new** session ID after successful login, regardless of any existing session ID.

---

## Attack 3: Cross-Site Request Forgery (CSRF)

**Concept:** Trick a victim's browser into making a request to a site where they're logged in — the browser automatically includes their session cookie.

**Classic example:**
```html
<!-- Attacker's page: -->
<img src="https://bank.com/transfer?to=attacker&amount=5000">
```
If the victim is logged in to bank.com and visits attacker's page, their browser sends the request with their cookie. The transfer executes.

**Defender's controls:** `SameSite=Strict` cookie flag, CSRF tokens in forms, checking `Origin`/`Referer` headers

---

## Adversary-in-the-Middle Authentication Proxy

**Concept:** A proxy sits between the victim and the real website, passing everything through transparently — including 2FA codes. After the victim completes login (including 2FA), the proxy captures the resulting session cookie and the attacker uses it directly.

**Why this defeats standard 2FA:** The proxy doesn't need to know the 2FA code ahead of time. It just relays it in real time. What it captures is the *post-authentication session* — the cookie the server issued after successful 2FA.

**Defender's controls:**
- FIDO2/passkeys — the credential is cryptographically bound to the real domain; the proxy's domain doesn't match
- Short session lifetimes — limits the window the stolen cookie is useful
- Step-up authentication for sensitive operations (require re-authentication before wire transfers, etc.)
- Phishing-resistant training — verify the URL before entering any credentials

---

## Key Takeaways

1. **Session security ≠ login security.** A strong login (2FA) doesn't protect a session cookie once it's been issued.
2. **Cookie flags are your first line of defense** against the most common theft vectors.
3. **FIDO2 is the only currently practical defense** against real-time phishing and AITM proxies.
4. **Defense in depth** — no single control covers everything. Layer them.
