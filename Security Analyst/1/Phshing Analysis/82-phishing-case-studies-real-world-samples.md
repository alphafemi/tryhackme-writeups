# Phishing Case Studies — Real-World Samples

## Overview

This document walks through six worked phishing case studies based on real samples: a fake PayPal gift card receipt (URL shortening), a fake shipping notification (tracking pixel), a multi-stage OneDrive/Adobe credential-harvesting chain, a fake Netflix billing notice with a malicious PDF, a fake Apple purchase notice using BCC and a `.dot` attachment, and a fake DHL notice with a malicious Excel payload.

---

Building on phishing fundamentals, these worked examples illustrate how attackers mirror legitimate communications using spoofing, redirection, brand impersonation, and malicious attachments.

> **Caution:** Samples discussed here are based on real phishing emails. Never interact directly with IPs, domains, or attachments from suspicious emails — use dedicated analysis tools instead.

---

### Case 1: Fake PayPal Gift Card Receipt

**Techniques used:** Spoofed email address, URL shortening, branded HTML

| Observation | Red Flag |
|---|---|
| **Subject** | Fake transaction creates urgency to prompt a hasty reaction |
| **From address** | Displayed as `service@paypal.com` but actually `gibberish@sultanbogor.com` |
| **To address** | Unusual recipient address, not a standard domain |

**Body:** Appears as a PayPal gift-card purchase receipt with a **"Cancel the order"** button — the only interactive element, no attachments.

**Button investigation:** Inspecting the raw HTML source reveals the button links to a **shortened URL**, obscuring the real destination. Never click a link/button without first verifying where it actually leads — tools like **WhereGoes** can reveal a shortened URL's true destination without visiting it.

---

### Case 2: Fake Shipping Notification with Tracking Pixel

**Techniques used:** Spoofed email address, pixel tracking, link manipulation

| Observation | Red Flag |
|---|---|
| **Subject** | Fake tracking number creates urgency to check "package status" |
| **From address** | Display name `Distribution Center` doesn't match actual sender `contact@beginpro.club` |
| **Hyperlink** | Matches subject line wording, but true destination unknown until inspected |

**Why images were blocked:** The raw source revealed an embedded image (`Tracking.png`) functioning as a **tracking pixel** — a near-invisible image that notifies the attacker's server when the email is opened. This is exactly why email providers like Yahoo automatically block images by default in unfamiliar emails — and also why hovering to preview a link didn't work (Yahoo had disabled links entirely in this message).

---

### Case 3: Multi-Stage Credential Harvesting (OneDrive → Adobe → Fake Login)

**Techniques used:** Artificial urgency, brand impersonation (layered), link redirection chain, credential harvesting

| Observation | Red Flag |
|---|---|
| **Send date vs. expiration** | Document "expires" the same day it was sent — manufactured urgency |
| **Call to action** | A "Download Document Here" button drives the victim forward |

**The redirection chain:**

```
Email → "Download Document Here" button
      → Fake OneDrive share landing page
      → Fake Adobe-branded page (suspicious URL, nonsensical instructions)
      → Fake credential-harvesting login portal (mimics Outlook)
```

**The trap:** The final "login" page isn't authenticating anything — it simply forwards whatever credentials are typed straight to the attacker's server, then shows a generic **"Invalid Credentials"** error regardless of what was entered.

> **Note on quality:** This particular sample had noticeable grammar/formatting issues, but AI-generated phishing content is increasingly polished — don't rely on typos alone as a detection signal.

---

### Case 4: Fake Netflix Billing Notice with Malicious PDF

**Techniques used:** Spoofed sender ("Netllx billing"), urgency (account suspension), brand impersonation, poor grammar, malicious attachment

| Observation | Red Flag |
|---|---|
| **Subject** | Claims the recipient's account was suspended, demanding urgent action |
| **From address** | Display name/domain mismatch |
| **Body** | Rendered HTML imitates Netflix branding |

**The attachment:** A PDF containing an embedded **"Update Payment Account"** link pointing to a non-Netflix domain. Additional red flags: an atypical phone number format, and use of a legitimate-looking Netflix help center domain to build false trust — this technique combines a real domain reference with a malicious redirect elsewhere.

---

### Case 5: Fake Apple Purchase Notice with BCC & .dot Attachment

**Techniques used:** Spoofed sender ("Apple Support"), BCC-based recipient hiding, urgency, poor grammar, unusual attachment format (`.dot` — a Microsoft Word template)

| Observation | Red Flag |
|---|---|
| **Subject** | "Action Required" framing around a fake unauthorized purchase |
| **From address** | Display name mismatch, with typos in both From and To addresses |
| **Recipient field** | Victim was **BCC'd** rather than directly addressed — hides the true target list |

**The attachment:** A completely blank email body plus a suspicious `.dot` file — itself unusual for a "receipt." Interacting with an embedded image inside the document redirects to a phishing site whose URL, despite containing familiar terms (`apps`, `ios`), is excessively long and complex — a strong indicator of malicious intent.

---

### Case 6: Fake DHL Shipping Notice with Malicious Excel Payload

**Techniques used:** Spoofed sender ("DHL Express"), brand impersonation, malicious Excel attachment with executable payload

| Observation | Red Flag |
|---|---|
| **Subject** | Implies an incoming DHL shipment |
| **From address** | Display name/domain mismatch |
| **Body** | HTML styled to resemble DHL branding |

**The attachment:** An `.xlsx` file with glaring geographic inconsistencies — sent from a **German** domain, addressed to a city in **India**, containing **Mandarin** text. These conflicting signals are a classic red flag. The document contains one clickable link progressing the attack.

**The payload:** Clicking the link attempts to download and execute `regasms.exe` — a malicious executable. In this case a system error blocked execution, but had it succeeded, potential outcomes include:

- **Persistence** — backdoor or scheduled task surviving reboot
- **Data Exfiltration** — stealing files, credentials, or browser-stored passwords
- **Ransomware Deployment** — encrypting the system for extortion

---

### Phishing Case Studies Quick Reference

| Case | Primary Technique | Key Tell |
|---|---|---|
| PayPal gift card | URL shortening | Shortened link on the only clickable button |
| Shipping notification | Tracking pixel | Blocked images hiding a tracking beacon |
| OneDrive/Adobe chain | Multi-stage redirection | Fake login portal that always "fails" |
| Netflix billing | Malicious PDF attachment | Embedded link to a non-Netflix domain |
| Apple purchase notice | BCC + `.dot` attachment | Blank body, unusual file format, hidden recipient list |
| DHL shipping | Malicious Excel payload | Geographic inconsistencies + embedded executable link |
| Common thread | Urgency + brand impersonation | Nearly every sample manufactures urgency and mimics a trusted brand |
| Investigation tools | URL expanders (e.g., WhereGoes), raw source inspection | Reveal true destinations/tracking without direct interaction |

---

## Summary

- Six real-world phishing samples show how attackers combine **spoofed senders, brand impersonation, manufactured urgency,** and a range of delivery tricks — and nearly every sample both mimics a trusted brand and creates artificial time pressure.
- **Link-based tricks:** a fake PayPal receipt hides its destination behind a **shortened URL**, and a fake shipping notice embeds a **tracking pixel** (a near-invisible image that signals the attacker's server when the email is opened — the reason many providers block images by default).
- **Credential harvesting:** a OneDrive → Adobe → fake Outlook login **redirection chain** ends at a portal that forwards typed credentials to the attacker and always returns "Invalid Credentials" — and since AI-written phishing is increasingly polished, typos alone shouldn't be relied on as a detection signal.
- **Malicious attachments:** a fake Netflix notice carries a **PDF** with a link to a non-Netflix domain; a fake Apple notice uses **BCC** to hide its target list plus a blank body and an unusual `.dot` file; and a fake DHL notice carries an **Excel** file with geographic inconsistencies (German sender, Indian recipient city, Mandarin text) whose link attempts to run `regasms.exe`, which could lead to persistence, data exfiltration, or ransomware.
- Safe investigation relies on **raw source inspection** and **URL expanders** (like WhereGoes) to reveal true destinations without ever interacting directly with suspicious links, IPs, or attachments.

## Related Documents

- [81. Email Structure, Protocols & Phishing Analysis](./81-email-structure-protocols-phishing-analysis.md)
- [67. Social Engineering & Human-Targeted Attacks](./67-social-engineering-human-targeted-attacks.md)
- [83. Phishing Analysis Tools & PhishTool](./83-phishing-analysis-tools-phishtool.md)
- [77. Pyramid of Pain & Indicators of Compromise](./77-pyramid-of-pain-indicators-of-compromise.md)
