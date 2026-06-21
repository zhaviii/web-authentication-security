# How Two-Factor Authentication Works

## The Core Idea

Authentication factors:
- **Something you know** — password
- **Something you have** — phone, hardware key
- **Something you are** — fingerprint, face

2FA requires two of these. If one is compromised, the attacker still can't get in alone.

---

## Method 1: TOTP (Time-Based One-Time Password)

Used by: Google Authenticator, Authy, Microsoft Authenticator

**How it works:**
```
1. During setup: server generates a secret key (e.g., BASE32: JBSWY3DPEHPK3PXP)
2. Secret is shared with the authenticator app (via QR code scan)
3. At login: app and server both compute:
       TOTP = HMAC-SHA1(secret, floor(unix_time / 30))
   → Both get the same 6-digit code (changes every 30 seconds)
4. User types the code → server verifies it matches
```

**Why it works:** The secret never travels over the network after setup. Even if someone intercepts your login, they don't have the TOTP secret.

**Why it's not perfect:** The code is still typed by the user → a fake website can capture it in real time and use it immediately (real-time phishing).

---

## Method 2: SMS OTP

**How it works:** Server generates a random 6-digit code, sends it via SMS.

**Problems:**
- SIM swapping — attacker convinces carrier to transfer your number to their SIM
- SS7 protocol vulnerabilities — telecom-level interception
- SMS is plaintext over the carrier network
- Phishing sites can ask for SMS codes just like TOTP codes

**Verdict:** Better than no 2FA. Worse than TOTP. Much worse than hardware keys. Avoid for high-security accounts.

---

## Method 3: Push Notification (Duo, Microsoft Authenticator)

**How it works:** Server sends a push to your registered device. You tap "Approve."

**Weakness:** MFA fatigue / push bombing — attacker triggers many login attempts at 3 AM hoping you approve one accidentally.

**Mitigation:** Number matching (app shows "Did you see the number 47 on screen?" before approving).

---

## Method 4: FIDO2 / WebAuthn / Passkeys

**How it works:**
```
1. Setup: device generates a public/private key pair per website
   - Private key stays on device, never leaves
   - Public key registered with the website

2. Login: website sends a challenge (random bytes)
   Device signs the challenge with the private key
   Website verifies signature with stored public key

3. The domain name is part of what gets signed
   → A fake website gets a challenge signed for "evil.com" not "bank.com"
   → The real bank's server rejects it
```

**Why this is the best:** The private key never leaves your device. The credential is domain-bound — phishing sites can't capture and replay it. No codes to type.

**Examples:** YubiKey, Apple Passkeys, Google Passkeys, Windows Hello

---

## Comparison Table

| Method | Phishing resistant | SIM swap resistant | Easy to use |
|--------|-------------------|-------------------|-------------|
| SMS OTP | ❌ | ❌ | ✅ |
| TOTP (Authenticator app) | ❌ | ✅ | ✅ |
| Push notification | ❌ | ✅ | ✅ |
| FIDO2 / Hardware key | ✅ | ✅ | ✅ |

---

## Important: 2FA Protects Login, Not Sessions

2FA proves you are who you say you are **at the moment of login**. Once the server issues a session cookie, that cookie is what keeps you authenticated — not your 2FA.

**If the session cookie is stolen after login, 2FA does not protect you.**

This is why session security (short expiry, HttpOnly, Secure flags, proper logout) matters as much as strong authentication.
