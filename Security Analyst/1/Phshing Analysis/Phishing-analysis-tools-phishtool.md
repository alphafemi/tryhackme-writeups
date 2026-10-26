# Phishing Analysis Tools & PhishTool

## Overview

This document covers the tooling used to investigate phishing emails: the header and body artifacts worth collecting, mail header analysis tools, IP and URL reputation services, manual and tool-assisted body analysis, safe attachment handling and hashing, malware sandboxes (ANY.RUN, Hybrid Analysis, JOESandbox), and PhishTool — a centralized platform that consolidates artifact collection, authentication and transmission analysis, URL and attachment review, and case resolution.

---

Beyond manual inspection, dedicated tools streamline phishing investigation — from header parsing to sandboxed malware detonation to centralized case documentation.

---

### Key Artifacts to Collect

#### Header Artifacts

| Artifact | Question It Answers |
|---|---|
| **Sender email address** | Where did the email originate? |
| **Sender IP address** | What's the source IP, and what does a reverse lookup reveal? |
| **Subject line** | Does it convey urgency or a call to action? |
| **Recipient address(es)** | Who's the intended recipient (To/CC/BCC)? |
| **Reply-To address** | Where are responses actually routed? |
| **Date and time** | When was it sent? |

#### Body Artifacts

| Artifact | Purpose |
|---|---|
| **URLs/Hyperlinks** | Identify all links; expand shortened URLs to reveal true destinations |
| **Attachment name(s)** | Check for suspicious names or extensions |
| **Attachment hash** | Generate for threat intelligence lookups |

---

### Mail Header Analysis Tools

| Tool | Function |
|---|---|
| **Messageheader** (Google Admin Toolbox) | Paste a full header to extract sender IP, routing path, and misconfigurations |
| **Message Header Analyzer** | Performs similar header parsing/analysis |

---

### IP and URL Reputation Analysis Tools

| Tool | Function |
|---|---|
| **IPinfo** | Look up geographic location and organization tied to an IP |
| **URLScan.io** | Safely simulates a real browsing session against a submitted URL, capturing a screenshot and full page behavior without visiting it directly |
| **Talos IP & Domain Reputation Center** (Cisco) | Assesses reputation/classification of IPs, domains, and networks |

---

### Mail Body Analysis

**Manually:** Right-click a link and select **Copy link address** to inspect the destination URL without clicking through.

**Tool-assisted:** URL extraction tools (or **CyberChef**) let you paste raw email content and automatically parse every embedded link — catching hidden/obfuscated URLs that manual review might miss.

---

### Attachment Analysis

Attachments should **only** be opened in a controlled environment (lab machine, sandbox) — never directly in a production mail client, to avoid accidental execution.

#### Generating a Hash

```bash
sha256sum shady_attachment.pdf
# 025ba9ce4a2118a9ca7b115c8869ff73bc16bad3732ba359cef1e60ad8f961f9  shady_attachment.pdf
```

The resulting hash can be checked against reputation services:

| Tool | Function |
|---|---|
| **Talos IP & Domain Reputation Center** | Also supports file hash lookups — can classify a file as e.g. Phishing/Malicious/Spam |
| **VirusTotal** | Checks files, URLs, IPs, and domains against dozens of security vendor engines simultaneously |

---

### Malware Sandboxes

Sandboxes let analysts safely detonate a suspicious file/URL and observe its behavior — without needing deep manual malware-analysis expertise, and without risking their own systems.

| Sandbox | Notes |
|---|---|
| **ANY.RUN** | Interactive sandbox — analysts can interact with the environment in real time, monitor processes, view network activity, and observe system changes as they happen |
| **Hybrid Analysis** | Free sandbox providing detailed behavioral reports (system changes, network activity, IOCs) |
| **JOESandbox** | Performs both static and dynamic analysis, generating comprehensive behavior/IOC/threat-classification reports |

**What sandboxes reveal:** URLs the sample tries to contact, additional payloads it downloads, and other IOCs — all observed safely in an isolated environment.

---

### PhishTool — Centralized Phishing Investigation Platform

**PhishTool** consolidates the entire investigation workflow — threat intelligence, OSINT, email metadata, and automated analysis — into one interface, useful across SOC triage, threat intel research, and phishing-kit investigation.

#### Identifying Artifacts

Uploading an email surfaces the key artifacts discussed above, alongside three views:

- **Rendered HTML** — how it would appear in an inbox
- **Raw HTML** — the underlying markup
- **Message source** — the full raw email

#### Further Analysis Tabs

| Tab | Covers |
|---|---|
| **Authentication** | Email authentication results (e.g., SPF/DKIM/DMARC outcomes) |
| **Transmission** | The routing path the email took |
| **URL Analysis** | Embedded URLs, with built-in **VirusTotal integration** for reputation/detection results without leaving the tool |
| **Attachments** | Direct review/analysis of any attached files |

#### Resolving a Case

Once analysis is complete, PhishTool lets analysts:

1. Mark the email's classification (e.g., **Malicious**)
2. Flag key artifacts — sender addresses, originating IP, embedded URLs
3. Add investigation notes
4. Click **Resolve** to formally close the case

This mirrors real SOC documentation and case-closure practices.

---

### Phishing Analysis Tools Quick Reference

| Category | Tools |
|---|---|
| Header analysis | Messageheader (Google), Message Header Analyzer |
| IP/URL reputation | IPinfo, URLScan.io, Talos Reputation Center |
| URL extraction | Dedicated URL extractors, CyberChef |
| Hashing | `sha256sum` (Linux) |
| File/indicator reputation | VirusTotal, Talos |
| Malware sandboxes | ANY.RUN, Hybrid Analysis, JOESandbox |
| Centralized platform | PhishTool (artifacts + authentication + transmission + URL + attachment analysis + case resolution) |
| Golden rule | Never open attachments directly — always in a sandbox/lab environment |

---

## Summary

- A phishing investigation starts by collecting **header artifacts** (sender address and IP, subject, recipients, Reply-To, date/time) and **body artifacts** (URLs, attachment names, attachment hashes).
- **Header tools** (Google's Messageheader, Message Header Analyzer) extract the sender IP and routing path; **reputation tools** (IPinfo, URLScan.io, Cisco Talos) assess IPs, domains, and URLs — URLScan.io does so by simulating a browsing session without the analyst visiting the page.
- Links can be inspected manually via **Copy link address** or extracted in bulk with URL extraction tools or **CyberChef**, which catches obfuscated URLs a manual review might miss.
- Attachments must only ever be opened in a **sandbox or lab environment**; generating a `sha256sum` hash lets analysts check them against **VirusTotal** or **Talos**, while sandboxes like **ANY.RUN, Hybrid Analysis,** and **JOESandbox** detonate files safely and surface contacted URLs, downloaded payloads, and other IOCs.
- **PhishTool** consolidates the whole workflow in one interface — artifact extraction with rendered/raw HTML and message-source views, **Authentication, Transmission, URL Analysis** (with VirusTotal integration), and **Attachments** tabs, and a case-resolution flow (classify, flag artifacts, add notes, resolve) that mirrors real SOC documentation practice.

