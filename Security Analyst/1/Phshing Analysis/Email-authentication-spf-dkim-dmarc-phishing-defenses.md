# Email Authentication (SPF, DKIM, DMARC) & Phishing Defenses

## Overview

This document covers the layered standards and controls defenders use to reduce phishing success: SPF (authenticating the sending server), DKIM (authenticating the message itself via digital signature), DMARC (aligning SPF and DKIM results and enforcing a policy), S/MIME (digitally signed and encrypted messages), technical defenses (filtering, SEGs, link rewriting, sandboxing), and user-facing tools and training.

---

Since phishing remains one of the most effective initial-access techniques, defenders rely on a layered set of email authentication standards and technical/user-facing controls to reduce its success rate.

---

### SPF — Sender Policy Framework

**SPF** authenticates the *sending server* — a DNS TXT record listing which IP addresses are authorized to send mail for a domain.

```
Email sent → Receiving server looks up sender domain's SPF record in DNS
           → Compares sending server's IP against the authorized list
           → Result determines accept/flag/reject
```

| Verification Result | Intended Action |
|---|---|
| **Pass, Neutral, None** | Accept — allow and process |
| **SoftFail, PermError** | Flag — mark suspicious but still deliver |
| **Fail, TempError** | Reject — discard immediately |

#### Sample SPF Record

```
v=spf1 ip4:127.0.0.1 include:_spf.google.com -all
```

| Component | Meaning |
|---|---|
| `v=spf1` | Marks the start of the SPF record |
| `ip4:127.0.0.1` | Authorizes this specific IPv4 address to send mail |
| `include:_spf.google.com` | Authorizes IPs covered by this included domain's SPF record |
| `-all` | Hard fail — reject anything not explicitly authorized |

**Tools:** **SPF Surveyor** (dmarcian) visually inspects a domain's SPF record and validates syntax — e.g., TryHackMe's SPF record lists no direct IPs but authorizes three domains (`_spf.google.com`, `email.chargebee.com`, `7168674.spf05.hubspotemail.net`), meaning any IP those domains authorize is treated as legitimate. **Google Admin Toolbox Messageheader** shows SPF results directly from a header (e.g., a `softfail` result with an unverifiable sending IP).

---

### DKIM — DomainKeys Identified Mail

**DKIM** authenticates the *message itself* via a digital signature, and — unlike SPF — **survives forwarding**, making it a stronger foundation for email security.

```
Sending server signs the message with a private key
           → Receiving server fetches the public key from DNS (DKIM record)
           → Verifies signature match
           → Authentic (match) or flagged/rejected (mismatch)
```

#### Sample DKIM Record

```
v=DKIM1; k=rsa; p=<public_key>
```

| Component | Meaning |
|---|---|
| `v=DKIM1` | DKIM version (optional) |
| `k=rsa` | Key type — RSA is standard |
| `p=` | The public key used to verify the signature |

> Record formats vary by mail provider/implementation.

**Common failure result:** `dkim=permerror` — a permanent verification failure, which can stem from an invalid signature, a missing/incorrect DNS record, a forwarding server modifying the message, or a DKIM misconfiguration.

**Tools:** dmarcian's **DKIM Record Checker** and **Validator**.

---

### DMARC — Domain-Based Message Authentication, Reporting, and Conformance

**DMARC** ties SPF and DKIM results together via **alignment** — confirming the sender's domain matches what SPF/DKIM actually verified — and defines a policy for handling failures.

#### Sample DMARC Record

```
v=DMARC1; p=quarantine; rua=mailto:postmaster@website.com
```

| Component | Meaning |
|---|---|
| `v=DMARC1` | DMARC version (required) |
| `p=quarantine` | Policy — move failing emails to spam (other values include `none` and `reject`) |
| `rua=mailto:...` | Optional — where aggregate reports are sent |

**Tools:** dmarcian's **Domain Checker** inspects SPF, DKIM, and DMARC together — e.g., `microsoft.com` passes all three checks, with a DMARC policy of `p=reject` (failing emails are outright rejected).

---

### S/MIME — Secure/Multipurpose Internet Mail Extensions

A protocol for **digitally signed and encrypted** messages, built on public key cryptography (private key never shared; public key freely distributed).

| Component | Provides |
|---|---|
| **Digital Signature** | Authentication (confirms sender identity), Non-repudiation (sender can't deny sending), Data Integrity (detects tampering) |
| **Encryption** | Confidentiality (only the intended recipient can read it) |

#### Worked Example: Bob → Mary

1. Bob creates a digital certificate to generate his signature
2. Bob signs the message with his **private key**
3. Bob shares his **public key** with Mary (for signature verification)
4. Bob obtains Mary's **public key** to encrypt the message
5. Mary verifies the signature using Bob's public key
6. Mary decrypts the message using **her own private key**
7. For a reply, Mary repeats the same process from her side
8. Both parties now hold each other's certificates for future exchanges

---

### Technical Defenses

| Control | Function |
|---|---|
| **Email Filtering** | Blocks/quarantines based on IP and domain reputation |
| **Secure Email Gateways (SEGs)** | Deep scanning for impersonation, spoofing, and phishing techniques beyond basic filters |
| **Link Rewriting** | Replaces suspicious/unknown URLs with redirected, scannable versions |
| **Sandboxing** | Detonates suspicious links/attachments in an isolated environment to observe behavior |

---

### User-Facing Tools & Training

Since some phishing inevitably reaches inboxes, human-facing defenses matter too:

| Measure | Purpose |
|---|---|
| **Trust & Warning Indicators** | Visual banners (e.g., "External Sender," "Suspicious Link") to build user awareness in the moment |
| **Phishing Reporting** | Easy in-email reporting buttons for suspicious messages |
| **User Awareness Training** | Teaching employees to recognize phishing and social engineering tactics |
| **Phishing Simulation Exercises** | Controlled test campaigns to reinforce and measure training effectiveness |

---

### Email Authentication & Defenses Quick Reference

| Concept | Summary |
|---|---|
| SPF | Authenticates the sending *server* via authorized IP list |
| DKIM | Authenticates the *message* via cryptographic signature; survives forwarding |
| DMARC | Aligns SPF + DKIM results with the sender domain; sets enforcement policy |
| DMARC policies | `none`, `quarantine`, `reject` |
| S/MIME | Digital signing + encryption for individual messages |
| S/MIME signature provides | Authentication, Non-repudiation, Data Integrity |
| S/MIME encryption provides | Confidentiality |
| Technical defenses | Email Filtering, SEGs, Link Rewriting, Sandboxing |
| User-facing defenses | Warning banners, in-email reporting, awareness training, phishing simulations |
| Key tools | SPF Surveyor, Messageheader, DKIM Checker/Validator, Domain Checker (all dmarcian) |

---

## Summary

- **SPF** authenticates the *sending server* through a DNS TXT record listing authorized IPs — results map to accept (Pass/Neutral/None), flag (SoftFail/PermError), or reject (Fail/TempError), with `-all` in a record meaning a hard fail for anything unlisted.
- **DKIM** authenticates the *message itself* via a digital signature verified with a public key published in DNS, and unlike SPF it **survives forwarding**; a `dkim=permerror` result points to an invalid signature, a missing/incorrect DNS record, a message altered in forwarding, or misconfiguration.
- **DMARC** ties SPF and DKIM together through **alignment** with the sender's domain and sets a failure policy of `none`, `quarantine`, or `reject`, with optional aggregate reporting via `rua`.
- **S/MIME** adds digitally signed and encrypted messages built on public key cryptography — the signature provides authentication, non-repudiation, and integrity, while encryption provides confidentiality.
- Beyond authentication, defense layers **technical controls** (email filtering, secure email gateways, link rewriting, sandboxing) with **user-facing measures** (warning banners, in-email reporting, awareness training, and phishing simulations) since some phishing will always reach inboxes.


