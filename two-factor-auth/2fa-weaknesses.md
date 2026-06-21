# 2FA Weaknesses and How to Mitigate Them

Understanding these weaknesses helps defenders build better systems and helps users make smarter security decisions.

---

## Weakness 1: Real-Time Phishing (TOTP / SMS)

**What happens:**
1. Attacker creates a convincing fake login page (looks identical to real site)
2. Victim enters username, password, and 2FA code on fake page
3. Attacker's server immediately relays all credentials to the real site
4. Attacker gets a valid session — in under one second

**Why standard 2FA doesn't stop this:** TOTP codes are valid for 30 seconds. That's enough time to relay them automatically. The code is just another piece of data the proxy passes through.

**Mitigations:**
- Use FIDO2/passkeys — credentials are domain-bound and cannot be relayed
- Check the URL before entering any credentials
- Browser extensions that verify site identity
- Certificate Transparency monitoring

---

## Weakness 2: Session Cookie Theft After Login

**What happens:** After successful 2FA login, the server issues a session cookie. If that cookie is stolen (via XSS, network interception, malware), the attacker has a valid session — completely bypassing the 2FA requirement, since authentication already happened.

**Mitigations:**
- `HttpOnly` + `Secure` cookie flags (see cookie security docs)
- Short session timeout (30–60 minutes idle)
- IP/User-Agent binding on sensitive sessions
- Re-authenticate (including 2FA) before sensitive operations (wire transfers, password changes)

---

## Weakness 3: SIM Swapping (SMS 2FA)

**What happens:** Attacker calls the mobile carrier, impersonates the victim, convinces support to transfer the phone number to a new SIM. Attacker now receives all SMS messages including OTP codes.

**Mitigations:**
- Don't use SMS 2FA for important accounts
- Add a SIM lock / carrier PIN to your account
- Use TOTP or hardware key instead

---

## Weakness 4: MFA Fatigue / Push Bombing

**What happens:** Attacker has the password and triggers dozens of push notification requests hoping the victim taps Approve by mistake, out of annoyance, or half-asleep.

**Mitigations:**
- Enable number matching on push notifications
- Set a limit on push requests per hour
- Alert user when unusual number of push requests occur
- Use FIDO2 instead (no codes to approve)

---

## Weakness 5: Account Recovery Bypass

**What happens:** Strong 2FA on the login page means nothing if the account recovery flow lets you reset everything with just an email link or security questions.

**Mitigations:**
- Recovery codes should be stored securely (printed, in a password manager)
- Recovery flow should also require identity verification
- Audit your recovery paths as carefully as your login path

---

## Summary for Defenders

| Attack | Stopped by |
|--------|-----------|
| Real-time phishing | FIDO2 / passkeys |
| Session theft after login | HttpOnly · Secure · short expiry |
| SIM swap | Avoid SMS 2FA |
| Push bombing | Number matching · FIDO2 |
| Recovery bypass | Secure recovery design |
