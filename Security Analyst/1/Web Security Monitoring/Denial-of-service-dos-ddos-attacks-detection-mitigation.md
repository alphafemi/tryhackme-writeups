# Denial-of-Service (DoS/DDoS) Attacks — Detection & Mitigation

## Overview

This document covers application-layer (Layer 7) denial-of-service attacks: the difference between DoS and DDoS, the six attack types, seven attacker motives with real-world incidents (BBC 2015, Microsoft 2023), detecting attacks in web server logs through six indicators and commonly targeted endpoints, SIEM-assisted analysis, prevention and mitigation at the application and network levels, record-scale provider mitigations, and attacker bypass techniques.

---

**Denial-of-Service (DoS)** attacks overwhelm a website or app so legitimate users can't use it — customers can't log in, shop, or access services, costing the business money and trust. This section focuses on **application-layer (Layer 7)** DoS, where websites and web applications are the target.

---

### Denial-of-Service (DoS)

Any DoS attack — small or large — is considered **successful** if it prevents a service from functioning as intended.

> **Example:** An e-commerce site's search form queries a database on user input. If the app fails to validate/handle input properly, an attacker can submit malformed data causing the app to hang or crash — or simply flood the same form with many (or one massive) request to trigger an outage.

### Distributed Denial-of-Service (DDoS)

A basic DoS attack is capped by a single machine's CPU, memory, bandwidth, and connection. To scale up, attackers use **botnets** — armies of compromised devices (computers, IoT devices, even servers) infected with malware and remotely controlled — instructed to flood a target simultaneously, far exceeding what one machine could achieve.

> **Example:** The same e-commerce site, built for steady daily traffic but not millions of requests in a short window, gets swarmed by a botnet and exhausts its resources, going down.

---

### Types of DoS/DDoS Attacks

| Attack Type | Description |
|---|---|
| **Slowloris** | Many partial HTTP requests that tie up server resources |
| **HTTP Flood** | A large volume of HTTP requests overwhelming the server |
| **Cache Bypass** | Bypassing CDN edge servers to force the origin server to respond directly |
| **Oversized Query** | Forcing the server to process large, resource-intensive requests |
| **Login/Form Abuse** | Overloading authentication logic with login attempts or password resets |
| **Faulty Input Validation Abuse** | Exploiting poorly designed input handling |

> Any of these can be carried out by a single attacker (DoS) or distributed across a botnet (DDoS).

---

### Attack Motives

| Motive | Description | Example Scenario |
|---|---|---|
| **Financial Loss** | Disrupt services to reduce sales/revenue | Flooding an e-commerce site during peak holiday sales |
| **Extortion** | Demand payment to stop an ongoing attack | Threatening a bank with a ransom DDoS |
| **Hacktivism** | Disruption for social/political protest | Attacking government sites during election season |
| **Distraction** | Redirect defenders' attention while other attacks proceed | Launching a DDoS while attacking other infrastructure |
| **Competition** | Disrupt a rival's service to raise their costs or steal market share | A competitor launches a DDoS during a product launch |
| **Denial of Wallet** | Force the victim to rack up usage-based service costs | Repeatedly accessing AWS S3 data, generating per-request costs |
| **Reputational Damage** | Erode customer trust | Game servers crashing on launch day |

> Not exhaustive — attackers may combine multiple motives based on resources and target.

#### In the Wild

| Incident | Year | Details |
|---|---|---|
| **BBC** | 2015 (New Year's Eve) | DDoS took the site offline for hours; New World Hacking claimed responsibility as a "capability test" |
| **Microsoft** | 2023 | Large-scale Layer 7 DDoS using HTTP flooding and Slowloris caused Azure, OneDrive, and Outlook outages; Anonymous Sudan claimed responsibility |

---

### Detecting DoS/DDoS in Web Server Logs

| Indicator | Example | Description |
|---|---|---|
| **High Request Rate** | `10.10.10.100 → 1000 GET /login` | A resource-heavy page (auth checks, DB queries) is flooded |
| **Odd User-Agents** | `curl/7.6.88 → /index` repeatedly | Spoofed/outdated User-Agents to blend in or evade filters |
| **Geographic Anomalies** | IPs scattered worldwide | Legitimate traffic usually comes from a few regions; global botnets don't |
| **Burst Timestamps** | 50 requests in 1 second → `/search` | Unnaturally concentrated timing suggests automation |
| **Server Errors (5xx)** | Spike in 503 Service Unavailable | Resources maxed out, service struggling |
| **Logic Abuse** | `GET /products?limit=999999` | Crafted queries forcing excessive data loads |

> Analysts should look for **multiple, layered signals** forming a composite picture — e.g., a global botnet hitting several resource-heavy endpoints, using varied or consistent User-Agents across a wide geographic IP spread.

#### Commonly Targeted Endpoints

| Endpoint | Why It's Targeted |
|---|---|
| `/login` | Involves authentication processing |
| `/search` | Requires complex database queries |
| `/api` endpoints | Critical for dynamic content delivery |
| `/register` or `/signup` | Database writes and validation |
| `/contact` or `/feedback` | Database entries, may trigger email notifications |
| `/cart` or `/checkout` | Session management, inventory checks, payment processing |

#### Worked Log Example

```
Normal traffic   → Users request pages every few seconds, receive expected responses
10:01:10 onward  → 203.0.113.55 begins sending repeated GET requests to /login.php
Server degraded  → Legitimate users now receive 503 responses — service unavailable
```

> Real DoS/DDoS logs may contain hundreds or thousands of flooding requests simultaneously — far beyond this condensed example.

#### SIEM-Assisted Analysis

SIEM platforms (Splunk, etc.) let analysts filter/sort by IP, User-Agent, or response code rather than scrolling raw log files — visualized as:

- A **timechart** showing normal traffic vs. a sudden spike (e.g., 1,000 requests to `/login.php` within one minute)
- A **pie chart** breaking down the User-Agents/IPs that made up the attack traffic

---

### Prevention & Mitigation Strategies

#### Application-Level Defense

| Measure | Description |
|---|---|
| **Secure Development Practices** | Input validation on forms/search fields — like a librarian with clear search rules ("titles under 50 characters") who can respond quickly instead of getting bogged down by malformed requests |
| **CAPTCHA Challenges** | Small friction for humans, effective block/slowdown for bots |
| **JavaScript Challenges** | Silent background checks confirming real users vs. automated tools; legitimate users rarely notice, bots/botnets often fail |

#### Network & Infrastructure Defenses

**CDN (Content Delivery Network):**

- Caches/serves content from edge servers, absorbing most request volume before it reaches the origin
- Provides load-balancing across servers, rerouting if one becomes unavailable
- **Example:** A 16 TB DDoS on a Cloudflare-protected site showed total bandwidth spiking far above the site's normal monthly volume, with **nearly all of it cached/absorbed at the edge** before reaching the backend — and Cloudflare's dashboard let analysts break down the attack's origin regions geographically

**WAF (Web Application Firewall):**

- Usually integrated with CDNs; inspects traffic and allows/challenges/blocks based on rules and threat intel
- **Example custom rule:** Rate-limit `/login.php` to 5 requests/minute per IP — exceeding the limit triggers a block or a human-verification challenge

#### Large-Scale Provider Mitigation

| Provider | Scale Mitigated |
|---|---|
| **Google (2023)** | Peaked at 398 million requests per second |
| **Cloudflare** | Claims the largest DDoS ever recorded, peaking at 11.5 Tbps over 35 seconds |

These demonstrate how major providers use globally distributed infrastructure and traffic filtering to absorb even record-scale attacks.

---

### Attacker Bypass Techniques

Even with CDNs, caching, and WAFs in place, attackers look for ways around them:

| Technique | How It Works |
|---|---|
| **Random Query Parameters** | Appending `?a=abcd` to a normally-cached URL (e.g., `/products?a=abcd`) prevents the CDN from serving a cached copy, forcing the origin server to respond directly |
| **User-Agent/Referrer Spoofing** | Rotating these values helps evade WAF pattern-matching rules |
| **Geographic Diversity** | Launching requests from many regions to dodge geo-based filtering |

---

### DoS/DDoS Quick Reference

| Concept | Summary |
|---|---|
| DoS = | Overwhelming a service to deny legitimate use (single source) |
| DDoS = | Same goal, distributed via a botnet of compromised devices |
| Focus of this section | Application layer (Layer 7) |
| 6 Attack types | Slowloris, HTTP Flood, Cache Bypass, Oversized Query, Login/Form Abuse, Faulty Input Validation Abuse |
| 7 Motives | Financial Loss, Extortion, Hacktivism, Distraction, Competition, Denial of Wallet, Reputational Damage |
| Real-world cases | BBC (2015), Microsoft (2023, Slowloris + HTTP flood) |
| 6 Log indicators | High Request Rate, Odd User-Agents, Geographic Anomalies, Burst Timestamps, 5xx Errors, Logic Abuse |
| Commonly targeted endpoints | /login, /search, /api, /register, /contact, /cart-checkout |
| Mitigation layers | Application (input validation, CAPTCHA, JS challenges), Network (CDN, WAF) |
| Record-scale mitigations | Google (398M req/s, 2023), Cloudflare (11.5 Tbps) |
| Bypass techniques | Random query params (cache-busting), UA/referrer spoofing, geographic diversity |

---

## Summary

- A **DoS** attack overwhelms a website or application so legitimate users can't use it, and counts as successful if it stops the service working as intended; a **DDoS** scales this up through a **botnet** of compromised devices, since a single machine is capped by its own CPU, memory, bandwidth, and connection.
- Layer 7 attacks come in six types — **Slowloris, HTTP Flood, Cache Bypass, Oversized Query, Login/Form Abuse,** and **Faulty Input Validation Abuse** — driven by motives such as **financial loss, extortion, hacktivism, distraction, competition, denial of wallet,** and **reputational damage**, as in the BBC (2015) and Microsoft (2023, HTTP flooding plus Slowloris) incidents.
- In web server logs, DoS/DDoS shows as a **high request rate**, **odd User-Agents**, **geographic anomalies**, **burst timestamps**, **5xx errors** (503s), and **logic abuse** (e.g., `?limit=999999`), most convincing when several signals line up, and typically aimed at resource-heavy endpoints like `/login`, `/search`, `/api`, `/register`, `/contact`, and `/cart`/`/checkout`; a SIEM timechart and pie chart make the spike and its sources easy to see.
- Mitigation layers **application-level defenses** (input validation, CAPTCHA and JavaScript challenges) with **network defenses** — a **CDN** that absorbs most volume at the edge and a **WAF** with rules such as limiting `/login.php` to 5 requests per minute per IP — and major providers have absorbed record attacks, including Google at 398 million requests per second (2023) and Cloudflare at 11.5 Tbps.
- Attackers look for bypasses even so: **random query parameters** (cache-busting, such as `?a=abcd`) force the origin server to respond, **User-Agent and referrer spoofing** evade WAF pattern rules, and **geographic diversity** dodges geo-based filtering.

