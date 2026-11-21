# IP and Domain Enrichment — DNS, ASN, Geolocation, Exposed Services & VPN Detection

## Overview

This document covers IP and domain enrichment as a core L1 SOC skill: resolving DNS records (A/AAAA and TXT), WHOIS/RDAP and domain age, DNS-based attack techniques (CDN abuse, typosquatting, IDN attacks), basic IP reputation enrichment, Autonomous Systems and ASN risk tiers, geolocation, exposed services via Shodan and Censys, TLS certificate analysis, VPN/proxy/Tor detection, and a baseline analyst workflow for investigating a domain and an IP.

---

SecOps runbooks revolve around **verify → enrich → decide**, and IP/domain enrichment is a core L1 skill. This section covers pivoting on geolocation, ASNs, open-service footprints, and DNS records to judge whether an indicator is malicious — and to pull extra context about the attack.

---

### Resolving DNS Domains

DNS converts human-friendly names (`tryhackme.com`) into machine-usable IPs. Tools like `nslookup.io` or `dnschecker.org` surface what's enrichable from a domain.

| Record | What It Reveals |
|---|---|
| **A / AAAA** | Maps a domain to IPv4/IPv6 addresses. A resolution to an Amazon IP, for example, suggests AWS hosting or a proxy/CDN built on AWS (e.g., Vercel) — the resolved IP itself can then be checked against known APT infrastructure or attack campaigns |
| **TXT** | Mail security settings (SPF/DKIM) and tooling in use. A legitimate domain typically shows coherent SPF records tied to real services (e.g., Google Workspace, Zapier); attack infrastructure often has empty TXT records, or suspicious/faked SPF-DKIM setups commonly abused in phishing |

### WHOIS / RDAP

WHOIS (gradually replaced by RDAP) reveals registrar, registration/expiry dates, and crucially, **domain age**. Modern attack infrastructure rarely survives more than a few months — it's either taken down or rotated to dodge detection. A domain registered for years is a reassuring sign; malicious domains often live only days.

### Attack Techniques Using DNS

| Technique | Description |
|---|---|
| **CDN abuse** | Routing malicious traffic through legitimate CDNs (Cloudflare, Akamai, Fastly) hides the real origin. If an A record points into a CDN range, the IP alone tells you little, since it may front thousands of legitimate and malicious services simultaneously |
| **Typosquatting** | Lookalike domains (`tryhakme[.]com`, `micros0ft[.]net`) exploit visual similarity. Treat lookalikes of known brands as high risk — but note companies sometimes register their own typo variants defensively, so a typosquat resolving to the legitimate brand isn't automatically malicious |
| **IDN attacks** | Domains can include non-ASCII characters; attackers substitute visually identical Cyrillic/Greek letters for Latin ones (e.g., `tryhаckme[.]com` with a Cyrillic "а"). When in doubt, convert to **Punycode** (the `xn--` form) via a tool like punycoder.com — a result starting with `xn--` confirms non-ASCII content requiring closer inspection |

---

### IP Enrichment Within the SOC

Most SIEM/EDR alerts contain at least one IP. Without enrichment, analysts risk **overreacting** (blocking legitimate infrastructure) or **underreacting** (ignoring a real C2 server). The simplest L1 enrichment:

| Tool | What It Checks |
|---|---|
| **AbuseIPDB** | Involvement in port scans or brute-force attacks |
| **VirusTotal** | Overall IP reputation plus community comments |

> If the IP isn't within a known CDN range, even a single detection is a meaningful risk signal.

### Autonomous Systems (AS/ASN)

An **Autonomous System** is a collection of IP prefixes under one organization's control, identified by a unique **ASN**. **BGP.Tools** provides the most comprehensive AS lookup. The AS type helps assess an IP's likely role:

| AS Type | Risk Signal | Example |
|---|---|---|
| **Residential** | May indicate VPN usage or a compromised consumer device | AS124888 (Vodafone) |
| **Server Hosting** | Highest-risk role — frequently used to distribute malware | AS215439 (PLAY2GO) |
| **Cloud/CDN** | Used by both legitimate services and adversaries — needs deeper analysis | AS16509 (Amazon AWS) |

### Geolocation (GeoIP)

Geolocation complements ASN lookup, especially for anomalous logon investigation. Tools like `ipinfo.io` and `iplocation.net` give reasonably reliable **country-level** data — city-level accuracy is generally unreliable. Use cases:

- **User logon analysis** — a US-based employee logging in from the Netherlands may indicate intrusion
- **Network analysis** — unexpected traffic to a region unrelated to the monitored organization's footprint is worth investigating

---

### Exposed Services

Knowing what's exposed on an IP matters from two angles: on a **victim's** IP, it can reveal the likely entry point (e.g., exposed SSH); on an **attacker's** IP, outdated services or exposed RDP suggest the host itself was compromised and is being used as a jump point.

#### Shodan Reconnaissance

**Shodan** indexes Internet-connected devices/services, surfacing open ports, running services, and configurations for a given IP. Key data points:

- **Open ports** — the first fingerprint of exposure (e.g., exposed Nginx or Rsync)
- **Service banners** — hint at software versions; an old version usually means a vulnerable one

#### Censys Search

**Censys.io** is a strong Shodan alternative for blue teams — it surfaces exposed services even on non-standard ports and offers advanced search (some features require an account). It can sometimes reveal services Shodan misses (e.g., an SSH service on a non-standard port like `56003`).

### TLS Certificates

IPs exposing HTTPS add another enrichment angle: TLS certificates. Tools like **crt.sh** (occasionally unavailable under high request volume), **Censys**, or **SSL Shopper** can surface:

| Certificate Field | What to Look For |
|---|---|
| **Issuer** | A self-signed certificate is a strong investigation trigger |
| **Validity** | Both newly created *and* unusually long-lived certificates can indicate malware infrastructure |
| **Subject** | For self-signed certs, this can reveal the actual program behind the HTTPS service (e.g., pfSense) |

---

### VPN Detection

Scenario: an alert fires for a Microsoft 365 login outside working hours, but the location looks completely normal — same country, city, user-agent. An inexperienced analyst closes it as a false positive — exactly what attackers count on, since VPN services (ProtonVPN, NordVPN, and dozens more) make faking any location trivial. The real question: **is this a real user, or someone behind a VPN?**

**IP2Proxy and Spur** are two key resources for labeling VPN, proxy, and Tor exit nodes. Both offer a free personal tier, but a paid subscription integrated directly into the SIEM is worth pushing for. A match (e.g., confirming an IP is a known VPN node) means the apparent geolocation of that user **cannot be trusted**.

---

### SOC Analyst Workflow — Domain & IP Enrichment

A baseline workflow (steps may be skipped/reordered depending on the alert):

**Investigate the domain:**

1. Does it "look" legitimate? Any typosquatting signs?
2. What's its reputation — known malicious domain?
3. When was it registered? Recent registration is suspicious.
4. Resolve the domain to an IP, then investigate that IP.

**Investigate the IP:**

1. Is it within a CDN range? If so, further IP-level investigation may add little.
2. What's its reputation (VirusTotal, AbuseIPDB, etc.)?
3. Where is it located? Is it a VPN node? Does the location match the alert context? (e.g., a US-based user isn't expected to log in from Asia)
4. What AS type does it belong to, and does that match the alert context? (e.g., employees aren't expected to log in from cloud hosting ranges)

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| DNS record types to check | A/AAAA (resolution), TXT (SPF/DKIM, tooling) |
| Domain age signal | WHOIS/RDAP — long-registered = reassuring; days-old = high suspicion |
| CDN abuse | A record resolving into a CDN range — IP alone tells you little |
| Typosquatting | Lookalike brand domains — but some are defensively registered by the real company |
| IDN/Punycode attack | Non-ASCII lookalike characters — check for `xn--` prefix via punycoder.com |
| Basic IP enrichment | AbuseIPDB (scans/brute force), VirusTotal (reputation/comments) |
| ASN risk tiers | Residential (VPN/compromised device), Server Hosting (highest risk), Cloud/CDN (needs deeper look) |
| ASN lookup tool | BGP.Tools |
| Geolocation tools | ipinfo.io, iplocation.net — country-level reliable, city-level is not |
| Exposed service recon | Shodan (open ports, banners), Censys (non-standard ports, advanced search) |
| TLS cert enrichment | crt.sh, Censys, SSL Shopper — issuer, validity, subject |
| VPN/proxy/Tor detection | IP2Proxy, Spur |
| Domain investigation order | Legitimacy look → reputation → registration age → resolve to IP |
| IP investigation order | CDN check → reputation → geolocation/VPN check → ASN type check |

---

## Summary

- **Resolving a domain** surfaces **A/AAAA** records (hosting clues, such as an Amazon IP suggesting AWS or a CDN built on it) and **TXT** records (a legitimate domain shows coherent SPF/DKIM tied to real services, while attack infrastructure often has empty or faked ones), and **WHOIS/RDAP** adds **domain age** — years of registration is reassuring, while malicious domains often live only days.
- DNS-based attack techniques include **CDN abuse** (an A record in a CDN range tells you little about the real origin), **typosquatting** (`tryhakme[.]com`, `micros0ft[.]net` — though some are registered defensively by the brand itself), and **IDN attacks** that swap in Cyrillic or Greek lookalike letters, confirmed by converting to **Punycode** and checking for the `xn--` prefix.
- Basic **IP enrichment** uses **AbuseIPDB** (scans and brute force) and **VirusTotal** (reputation and community comments) — and outside a CDN range, even a single detection is meaningful; **ASN** lookups through BGP.Tools show the AS type, with **server hosting** the highest-risk role, **residential** suggesting VPN use or a compromised device, and **cloud/CDN** needing deeper analysis, while **geolocation** (ipinfo.io, iplocation.net) is reliable at country level only.
- **Exposed services** (via **Shodan** for open ports and banners, and **Censys** for non-standard ports) reveal a victim's likely entry point or show an attacker's host was itself compromised, and **TLS certificates** (crt.sh, Censys, SSL Shopper) are suspicious when self-signed, brand new, or unusually long-lived.
- **VPN detection** matters because a normal-looking location proves nothing when VPNs make faking it trivial — **IP2Proxy** and **Spur** label VPN, proxy, and Tor exit nodes, and a match means the apparent geolocation cannot be trusted; the baseline workflow investigates a **domain** (legitimacy, reputation, registration age, resolve to an IP) and then the **IP** (CDN check, reputation, geolocation and VPN check, ASN type).

## Related Documents

- [111. File and Hash Intelligence — Filepath Heuristics, VirusTotal, MalwareBazaar & Sandboxing](./111-file-and-hash-intelligence-filepath-heuristics-virustotal-malwarebazaar-sandboxing.md)
- [113. SIEM Log Sources in Practice — Sysmon, WinEventLogs, Linux Auth Logs & Web Logs (WINHOST05 Case Study)](./113-siem-log-sources-in-practice-sysmon-wineventlogs-linux-auth-logs-web-logs-winhost05-case-study.md)
- [110. Cyber Threat Intelligence — Lifecycle, TLP, STIX/TAXII & the CTI Analyst Workflow](./110-cyber-threat-intelligence-lifecycle-tlp-stix-taxii-cti-analyst-workflow.md)
- [10. Domain Name System (DNS)](./10-domain-name-system-dns.md)
- [42. Nmap — Network Scanning & Enumeration](./42-nmap-network-scanning-enumeration.md)
