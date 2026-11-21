# Client-Side vs. Server-Side Web Attacks — Detection in Logs, Network Traffic & WAFs

## Overview

This document contrasts client-side and server-side web attacks and what a SOC can see of each: the common client-side attacks (XSS, CSRF, clickjacking) and why they are hard to detect from server-side tooling, the common server-side attacks (brute-force, SQL injection, command injection) with real-world breaches, detecting an attack chain in access logs versus network traffic, the limitations of each, and how WAF rules, challenge-response, and threat intelligence mitigate web attacks.

---

Web attacks are among the most common ways attackers gain entry into target systems. Public-facing websites often sit in front of databases and other valuable infrastructure, making them appealing targets.

---

### Client-Side Attacks

Client-side attacks abuse weaknesses in **user behavior** or on the **user's device** — often exploiting browser vulnerabilities or tricking users into unsafe actions to steal accounts/data. The growing use of third-party plugins in dynamic web apps keeps broadening this attack surface.

> **Example:** Clicking a product image on an e-commerce site triggers a hidden, invisible window loading a malicious site in the background — which silently steals the victim's session cookies, letting the attacker impersonate them without anything looking out of the ordinary.

#### SOC Limitations

Server-side logs and network captures give little to no visibility into what happens **inside a user's browser**. Client-side attacks can execute code, steal info, or manipulate the environment without generating suspicious HTTP requests or network traffic — making them difficult or impossible to detect from a pure SOC perspective without additional browser-side controls or endpoint monitoring.

#### Common Client-Side Attacks

| Attack | Description |
|---|---|
| **Cross-Site Scripting (XSS)** | The most common client-side attack — malicious scripts run inside a trusted site and execute in the victim's browser (e.g., an unfiltered comment box injecting `<script>` tags; in real attacks, used to steal cookies/session data rather than show a harmless popup) |
| **Cross-Site Request Forgery (CSRF)** | Tricks the browser into sending unauthorized requests on behalf of a trusted, authenticated user |
| **Clickjacking** | Overlays invisible elements on top of legitimate content, tricking users into interacting with something they can't see |

---

### Server-Side Attacks

Server-side attacks exploit vulnerabilities in the **web server, application code, or backend** — targeting the systems themselves rather than manipulating user interaction. Flaws in server logic, misconfigurations, or poor input handling let attackers gain access, steal data, or disrupt services.

> **Example:** A mishandled input field on a login or search form can let an attacker access sensitive customer or financial data stored in the backend database.

#### Catching Server-Side Attacks

Unlike client-side attacks, server-side attacks **leave a trail** — every web request is processed and recorded in logs, and travels across the network where traffic analysis can reveal suspicious behavior.

#### Common Server-Side Attacks

| Attack | Description | Real-World Example |
|---|---|---|
| **Brute-force** | Repeatedly attempting usernames/passwords, usually with automated tools | T-Mobile (2021) — brute-force breach exposed PII of 50+ million customers |
| **SQL Injection (SQLi)** | Attacking the backend database when queries are built via string concatenation instead of parameterized queries | MOVEit (2023) — SQLi exploited, affecting 2,700+ organizations including US government agencies, the BBC, and British Airways |
| **Command Injection** | User input passed to the system without validation, letting attackers run commands with the application's permissions | — |

---

### Detecting Attacks in Access Logs

#### Access Log Fields and Indicators

| Field | Potential Indicator |
|---|---|
| **1. Client IP Address** | Known-malicious or outside the expected geographic range |
| **2. Timestamp / Requested Page** | Requests at unusual hours or repeated in a short period |
| **3. Status Code** | Repeated 404s indicating page-not-found probing |
| **4. Response Size** | Significantly smaller/larger than normal |
| **5. Referrer** | Referring pages inconsistent with normal navigation |
| **6. User-Agent** | Outdated browsers or known attack tools (sqlmap, wpscan) |

#### Worked Attack Sequence in Logs

A condensed attack chain visible across log entries:

```
1. Directory Fuzzing   — Attacker probes directories/forms; 200 responses mark valid finds
2. Brute-Force         — Repeated POST requests to /login.php; final request returns 302 Found
                          (successful login), redirecting to /account
3. SQL Injection       — On /search, attacker tries ' OR '1'='1 and 1' OR 'a'='a
                          (if queries are built dynamically, the database can be dumped)
```

> In a real environment, benign traffic vastly outnumbers malicious entries — spotting these patterns amid normal activity is the core analyst skill.

#### Log Limitations

Access logs do **not** typically capture full request bodies — particularly POST data like submitted credentials.

**Example log line:**
```
10.10.10.100 [12/Aug/2025:14:32:10] "POST /login HTTP/1.1" 200 532 "/home.html" "Mozilla/5.0"
```

This shows the method, requested page, and status code — but **not** the actual credentials submitted. GET requests may log full paths/query strings, but even this depends entirely on the server software and logging configuration.

---

### Detecting Attacks in Network Traffic

Network traffic analysis examines the **raw data exchanged** between client and server — far more verbose than logs, revealing full HTTP headers, POST bodies, cookies, and uploaded/downloaded files.

> **Encryption caveat:** HTTPS/SSH limit payload visibility without decryption keys — this analysis focuses on plain HTTP traffic.

#### Worked Attack Sequence in Wireshark

Replaying the same attack chain from the logs, now visible at the packet level (filtered by destination IP and `http.user_agent`):

1. **Directory fuzz** — same sequence as in the logs
2. **Brute-force** — packet 13 is the successful login; inspecting it directly reveals the **actual password used**: `password123` (a weak credential, notably on an admin account)
3. **SQL injection** — inspecting the first SQLi packet shows the exact payload (`' OR '1'='1`) and its result: the Users table dumped, with first names and surnames visible **in cleartext**

> Network traffic analysis succeeds exactly where log analysis fails — recovering the actual credentials and injected payloads that logs only hint at. MySQL protocol traffic itself can also be inspected in Wireshark, showing both the query and the returned result set.

---

### Web Application Firewalls (WAFs) as Mitigation

WAFs act as gatekeepers, inspecting full request packets (including decrypted TLS traffic) and filtering malicious requests before they reach the server — essentially Wireshark-level visibility combined with active blocking.

#### WAF Rule Categories

| Rule Type | Description | Example Use Case |
|---|---|---|
| **Block common attack patterns** | Blocks known malicious payloads/indicators | Block the User-Agent `sqlmap` |
| **Deny known malicious sources** | Uses IP reputation, threat intel, or geo-blocking | Block IPs tied to recent botnet campaigns |
| **Custom-built rules** | Tailored to the specific application | Allow only GET/POST to `/login` |
| **Rate-limiting & abuse prevention** | Caps request frequency | Limit login attempts to 5/minute per IP |

**Example custom rule** (blocking a detected sqlmap-driven SQLi campaign):

```
If User-Agent contains "sqlmap"
then BLOCK
```

> Modern WAFs detect and block known suspicious User-Agents automatically — but custom rules like this let defenders target specific threats without impacting legitimate traffic.

#### Challenge-Response Mechanisms

Rather than outright blocking, WAFs can **challenge** suspicious requests (e.g., CAPTCHA) to confirm a human is behind them — valuable given that malicious bot traffic makes up roughly **37% of global web traffic**. This approach suits rules with a higher chance of catching legitimate traffic in the net.

#### Integrating Threat Intelligence

Many WAFs ship with built-in rulesets targeting the **OWASP Top 10**, and leverage threat intel feeds to automatically block known-malicious IPs and User-Agents — regularly updated to cover new threats, APT activity, and recently disclosed CVEs. Cloudflare, for example, maintains curated IP lists sourced from botnets, VPNs, anonymizers, and malware campaigns based on global threat intelligence.

---

### Client-Side vs. Server-Side Attack Detection Quick Reference

| Concept | Summary |
|---|---|
| Client-side attacks | Exploit user behavior/browser; nearly invisible to server-side SOC tooling |
| Common client-side attacks | XSS, CSRF, Clickjacking |
| Server-side attacks | Exploit server/app/backend flaws; leave a traceable trail |
| Common server-side attacks | Brute-force, SQL Injection, Command Injection |
| Real-world breaches | T-Mobile (brute-force), MOVEit (SQLi) |
| Access log strength | Shows request metadata — IP, timestamp, status, referrer, User-Agent |
| Access log weakness | Does NOT capture POST body data (e.g., submitted credentials) |
| Network traffic strength | Reveals full payloads — actual passwords, SQLi strings, dumped data |
| Encryption limitation | HTTPS/SSH hide payloads without decryption keys |
| WAF role | Active mitigation — inspects, decrypts TLS, and blocks/challenges requests |
| WAF rule types | Pattern blocking, source denial, custom rules, rate-limiting |
| WAF soft-block option | Challenge-response (CAPTCHA) instead of outright blocking |
| WAF intelligence integration | OWASP Top 10 rulesets + threat intel feeds (IPs, User-Agents, CVEs) |

---

## Summary

- **Client-side attacks** abuse user behavior or the user's browser — **XSS** (the most common), **CSRF**, and **clickjacking** — and are nearly invisible to a SOC, because they can run code and steal data inside the browser without generating suspicious HTTP requests or network traffic; **server-side attacks** exploit the web server, application code, or backend (**brute-force**, **SQL injection**, **command injection**) and leave a trail in logs and network traffic, as in the T-Mobile (2021, brute-force) and MOVEit (2023, SQLi) breaches.
- **Access logs** reveal attack chains through the client IP, timestamp, status code (repeated 404s suggest probing), response size, referrer, and User-Agent (tools like sqlmap or wpscan) — a worked sequence shows directory fuzzing, then a brute-force ending in a `302 Found`, then SQL injection attempts on `/search` — but logs **don't capture POST bodies**, so submitted credentials stay hidden.
- **Network traffic analysis** fills that gap for plain HTTP: in the same attack chain, Wireshark exposes the actual password used (`password123`), the exact SQLi payload (`' OR '1'='1`), and the dumped Users table in cleartext, though HTTPS/SSH limit this visibility without decryption keys.
- **WAFs** actively mitigate web attacks by inspecting full requests (including decrypted TLS) and applying **attack-pattern blocking** (e.g., blocking the `sqlmap` User-Agent), **malicious-source denial**, **custom rules** (e.g., allow only GET/POST to `/login`), and **rate-limiting** (e.g., 5 login attempts per minute per IP).
- WAFs can also **challenge** suspicious requests with a CAPTCHA instead of blocking outright — useful since malicious bots make up roughly 37% of global web traffic — and many ship with **OWASP Top 10** rulesets and threat intelligence feeds of known-malicious IPs and User-Agents.

## Related Documents

- [95. Web Application Security Fundamentals](./95-web-application-security-fundamentals.md)
- [97. Web Shell Detection — Logs, Auditd, File System & Network Traffic](./97-web-shell-detection-logs-auditd-file-system-network-traffic.md)
- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
- [51. SQL & Database Fundamentals](./51-sql-database-fundamentals.md)
