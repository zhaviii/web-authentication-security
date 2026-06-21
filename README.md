# 🔐 Web Authentication Security

> A defensive security research repository covering session management, cookie hardening, 2FA implementation, and authentication best practices — from a blue team / system administrator perspective.

![Security](https://img.shields.io/badge/Defensive_Security-blue?style=flat)
![Blue Team](https://img.shields.io/badge/Blue_Team-0078D6?style=flat)
![Research](https://img.shields.io/badge/Research-Educational-green?style=flat)

---

## 📌 Purpose

As a System Administrator managing real infrastructure, understanding how web authentication works — and how it fails — is essential for hardening the systems I protect.

This repository documents:
- How session-based authentication works under the hood
- What cookie security flags do and why missing them is dangerous
- How 2FA works, and what its implementation weaknesses are
- Practical hardening checklists for servers and web applications
- Attack concepts explained from a **defender's perspective** — to understand what to protect against

> ⚠️ All content here is for **defensive and educational purposes only**. Understanding how attacks work is necessary to build effective defenses.

---

## 📁 Contents

```
web-authentication-security/
├── session-management/
│   ├── how-sessions-work.md        # HTTP sessions from scratch
│   ├── session-hijacking.md        # What it is and how to prevent it
│   └── session-fixation.md         # Attack concept + defense
├── cookie-security/
│   ├── cookie-flags-explained.md   # HttpOnly, Secure, SameSite deep dive
│   ├── cookie-theft-vectors.md     # How cookies get stolen (and defenses)
│   └── secure-cookie-config.md     # Correct configuration examples
├── two-factor-auth/
│   ├── how-2fa-works.md            # TOTP, SMS, hardware keys explained
│   ├── 2fa-weaknesses.md           # Real-world bypass techniques + mitigations
│   └── fido2-passkeys.md           # Modern phishing-resistant authentication
├── hardening-checklists/
│   ├── web-server-auth-checklist.md
│   └── nginx-secure-headers.md
└── attack-concepts/
    └── understanding-session-attacks.md  # Defender's overview
```

---

## 🗂 Quick Navigation

| Topic | What You'll Learn |
|-------|------------------|
| [How Sessions Work](session-management/how-sessions-work.md) | The full lifecycle of a login session |
| [Cookie Flags](cookie-security/cookie-flags-explained.md) | HttpOnly · Secure · SameSite — what each one does |
| [Cookie Theft Vectors](cookie-security/cookie-theft-vectors.md) | XSS, network interception, how defenders respond |
| [2FA Explained](two-factor-auth/how-2fa-works.md) | TOTP, SMS, hardware tokens from first principles |
| [2FA Weaknesses](two-factor-auth/2fa-weaknesses.md) | Real bypass techniques and how to mitigate them |
| [FIDO2 / Passkeys](two-factor-auth/fido2-passkeys.md) | The future of phishing-resistant login |
| [Hardening Checklist](hardening-checklists/web-server-auth-checklist.md) | Practical checklist for securing authentication |

---

## 🧠 Key Concepts at a Glance

### What is a Session?
When you log in to a website, the server creates a **session** — a temporary record that you are authenticated. It gives your browser a **session cookie** (a random ID). On every request, your browser sends that ID back, and the server says "yes, I know this person, they're logged in."

If someone steals that cookie → they are logged in as you. No password needed.

### The Three Cookie Security Flags

| Flag | What it does | Without it |
|------|-------------|------------|
| `HttpOnly` | JavaScript cannot read the cookie | XSS attack can steal it |
| `Secure` | Cookie only sent over HTTPS | Intercepted on HTTP connections |
| `SameSite=Strict` | Cookie not sent on cross-site requests | CSRF attacks possible |

### Why 2FA Can Still Be Bypassed
Standard 2FA (TOTP codes, SMS) proves you have a device at login time. But once the server issues a session cookie after login, **that cookie represents your authenticated session — not your 2FA state**. If the cookie is stolen after login, the attacker has full access without needing your 2FA code.

This is why **session security is as important as login security**.

---

## 📚 Learning Path

```
1. Start here → session-management/how-sessions-work.md
2. Then →       cookie-security/cookie-flags-explained.md
3. Then →       cookie-security/cookie-theft-vectors.md
4. Then →       two-factor-auth/how-2fa-works.md
5. Then →       two-factor-auth/2fa-weaknesses.md
6. Apply it →   hardening-checklists/web-server-auth-checklist.md
```
