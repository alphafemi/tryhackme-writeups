# Email Structure, Protocols & Phishing Analysis

## Overview

This document covers the fundamentals of email and phishing analysis: the anatomy of an email address, the three core email protocols (SMTP, POP3, IMAP) and an email's journey from sender to recipient, email header and body structure, reconstructing attachments from raw message source, the types of malicious emails, the common traits of a phishing email, and defanging as a safe-handling practice.

---

**Spam** and **phishing** remain the most common social engineering threats. While spam is generally low-risk, phishing tricks users into disclosing sensitive data or unknowingly deploying malware — a single click can give an attacker their initial network foothold.

---

### Anatomy of an Email Address

Email as we know it was popularized in the 1970s on **ARPANET** by Ray Tomlinson, who introduced the `@` symbol.

| Component | Role |
|---|---|
| **Username** | Identifies the specific recipient's mailbox |
| **@ symbol** | Separates username from domain, tells the system where to route the message |
| **Domain name** | Identifies the mail server responsible for receiving the message |

> **Analogy:** The domain is like a street/apartment building; the username is the specific person/mailbox within it. Together, the mail server (postal worker) knows exactly where to deliver the message.

---

### Email Protocols

| Protocol | Role |
|---|---|
| **SMTP** (Simple Mail Transfer Protocol) | Sends emails |
| **POP3** (Post Office Protocol) | Downloads emails to a single device |
| **IMAP** (Internet Message Access Protocol) | Syncs emails across multiple devices |

#### POP3 vs. IMAP

| | POP3 | IMAP |
|---|---|---|
| **Storage** | Downloaded to a single device | Stored on the server |
| **Sent messages** | Stored only on the sending device | Stored on the server |
| **Multi-device access** | No — single device only | Yes — synced across devices |
| **Server retention** | Typically removed after download | Remains unless explicitly deleted |

#### An Email's Journey

```
Sender's client → SMTP → Sending mail server
                              ↓
                      Query DNS for recipient's mail server
                              ↓
                      DNS responds with server address
                              ↓
                      Message delivered across the internet
                              ↓
                Recipient's client connects to their mail server
                              ↓
                Message retrieved via POP3 (download) or IMAP (sync)
```

---

### Email Structure

An email has two main parts:

| Part | Contents |
|---|---|
| **Header** | Metadata — sender, recipient, servers involved in delivery |
| **Body** | The actual message — plain text or HTML |

#### Email Header Fields

| Field | Meaning |
|---|---|
| **From** | Sender's email address |
| **To** | Receiver's email address |
| **Reply-To** | Where replies are routed (optional) |
| **Subject** | The email's subject line |
| **Date** | When the email was sent |

#### Viewing Message Source

Viewing the **raw message source** (e.g., in Thunderbird: **View → Message Source**, or `Ctrl+U`) reveals the full header data plus the raw body — including technical details like the **originating IP address** that aren't visible in the standard inbox view. This is essential for phishing investigation.

---

### Email Body Analysis

Emails are either **plain text** or **HTML** (supporting images, links, styling). Most clients render HTML automatically — inspecting the raw HTML source reveals the actual structure, including elements a client might hide by default (e.g., blocked images), and can expose disguised links or embedded malicious content.

#### Reconstructing Attachments

Attachments embedded in an email's raw source include key headers:

| Header | Purpose |
|---|---|
| **Content-Type** | The file type (e.g., `application/pdf`) |
| **Content-Disposition** | Confirms it's an attachment and provides its filename |
| **Content-Transfer-Encoding** | Encoding method used (typically `base64`) |

The base64-encoded data following these headers represents the actual file, and can be decoded (e.g., via **CyberChef** or a base64-to-file converter) to reconstruct the original attachment for analysis.

---

### Types of Malicious Emails

| Type | Description |
|---|---|
| **Spam** | Unsolicited bulk email; a more dangerous variant is called **malspam** |
| **Phishing** | Impersonates a trusted entity to extract sensitive information |
| **Spear Phishing** | Targeted phishing aimed at a specific individual/organization using personalized info |
| **Whaling** | Spear phishing specifically targeting executives (CEO, CFO) |
| **Smishing** | Phishing via SMS/text message |
| **Vishing** | Phishing via voice calls |

---

### Anatomy of a Phishing Email

Regardless of the attacker's end goal (credential harvesting, malware delivery, unauthorized access), phishing emails tend to share common traits:

| Trait | Example |
|---|---|
| **Spoofed From Address** | `noreply@microsof.com` (typosquatted domain) |
| **Urgent Subject/Message** | *"Your account will be locked in 24 hours"* |
| **Brand Impersonation** | Logos/colors mimicking a real company |
| **Grammar & Spelling Issues** | Awkward phrasing (less common now with AI-assisted writing) |
| **Generic Content** | *"Dear Customer"* instead of a real name |
| **Hidden/Shortened Links** | `bit.ly/secure-login` masking the true destination |
| **Malicious Attachments** | Disguised file types (e.g., `invoice.pdf.exe`) |

---

### Safe Analysis: Defanging

When handling suspicious links/IPs during investigation, **defang** them to prevent accidental clicks — replacing special characters (`.`, `@`, `://`) with safe placeholders:

| Original | Defanged |
|---|---|
| `http://www.suspiciousdomain.com` | `hxxp[://]www[.]suspiciousdomain[.]com` |

This keeps the reference readable for analysis while making it non-clickable/non-resolvable by accident.

---

### Email Analysis Quick Reference

| Concept | Summary |
|---|---|
| Email address parts | Username, @ symbol, Domain |
| Send protocol | SMTP |
| Receive protocols | POP3 (single device, downloaded) vs. IMAP (synced, server-stored) |
| Email structure | Header (metadata) + Body (content) |
| Key header fields | From, To, Reply-To, Subject, Date |
| View raw source | Thunderbird: View → Message Source (`Ctrl+U`) |
| Attachment headers | Content-Type, Content-Disposition, Content-Transfer-Encoding |
| Attachment decoding | Base64 → CyberChef or similar tool |
| 6 Malicious email types | Spam, Phishing, Spear Phishing, Whaling, Smishing, Vishing |
| 7 Phishing red flags | Spoofed sender, urgency, brand impersonation, grammar issues, generic greeting, hidden links, malicious attachments |
| Safety practice | Defang URLs/IPs/emails before sharing or analyzing |

---

## Summary

- **Spam** and **phishing** are the most common social engineering threats — spam is generally low-risk, but a single click on a phishing email can give an attacker their initial network foothold.
- An email address is a **username**, an **@ symbol**, and a **domain name**; email travels via **SMTP** (sending), then is retrieved via **POP3** (downloaded to a single device, typically removed from the server) or **IMAP** (stored on the server and synced across devices), with DNS used to locate the recipient's mail server.
- Every email has a **header** (From, To, Reply-To, Subject, Date, plus delivery metadata) and a **body** (plain text or HTML) — viewing the **raw message source** (`Ctrl+U` in Thunderbird) exposes details like the originating IP address, and base64-encoded attachments can be decoded (e.g., in CyberChef) using their `Content-Type`, `Content-Disposition`, and `Content-Transfer-Encoding` headers.
- Malicious email types include **spam/malspam, phishing, spear phishing, whaling, smishing,** and **vishing**, and phishing emails commonly show seven red flags: a spoofed sender, urgent messaging, brand impersonation, grammar issues, generic greetings, hidden/shortened links, and malicious attachments.
- Always **defang** suspicious URLs, IPs, and emails (e.g., `hxxp[://]www[.]suspiciousdomain[.]com`) before sharing or analyzing them, so they can't be clicked or resolved by accident.


