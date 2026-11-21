# Web Application Security Fundamentals

## Overview

This document covers web application security fundamentals: the shift from desktop to web applications and its security tradeoffs, notable real-world breaches, the request-response cycle, the three components of a web service, common web servers, best practices for securing each component, access logging, and the roles of CDNs, WAFs (types and detection methods), and antivirus in a defense-in-depth strategy.

---

The internet underpins modern life — banking, shopping, social media, and beyond — making websites and web applications among attackers' most targeted assets.

---

### The Shift from Desktop to Web

| Decade | Trend |
|---|---|
| **1990s** | Desktop applications dominant (speed/connectivity limitations) |
| **2000s** | Dynamic web applications rise (email, social media, banking) |
| **2010s** | Massive growth of cloud computing and SaaS |
| **Today** | Nearly everything is done in a browser |

#### Tradeoffs from a Security Perspective

The shift brings accessibility, faster updates, better compatibility, and reduced resource use — but also expands the attack surface. Web apps are common entry points because they're **always available and exposed**, and they often connect to backend systems (databases, infrastructure) — making a vulnerable web app a frequent first stage in a larger attack sequence.

| Risk | As a Web App Owner | As a Web App User |
|---|---|---|
| **Availability** | App is online and must be secured 24/7 | — |
| **Exposure** | Accessible globally, at any time | Browser breach puts all accounts at risk |
| **Threat landscape** | Hard to keep up with emerging threats | Identity theft/financial loss from a breach |
| **Responsibility** | Must secure users' data | Privacy can be permanently compromised |

#### Real-World Examples

| Incident | Year | Cause | Impact |
|---|---|---|---|
| **Equifax** | 2017 | Apache vulnerability | ~150 million Americans' sensitive data compromised; attackers accessed internal databases |
| **Capital One** | 2019 | Misconfigured WAF | 100+ million customers' personal/financial data exposed via cloud infrastructure access |

---

### The Request-Response Cycle

When you visit a website, your browser sends a **request** to a web server. The server processes it, verifies access, and returns a **response** — a webpage, image, or data. Attackers abuse this cycle by overwhelming servers (DoS), bypassing access controls, or tricking the server into executing harmful commands.

### Components of a Web Service

| Component | Role |
|---|---|
| **Application** | The code, images, styles, and icons dictating how the site looks/functions |
| **Web Server** | Hosts the application; listens for requests and returns responses |
| **Host Machine** | The underlying OS (Linux/Windows) running the web server and app |

**Example request path:** Browser → Internet → Apache web server (accepts/routes the request) → PHP web app (processes it).

---

### Web Servers

Positioned in front of websites/applications, web servers are publicly exposed and handle all incoming requests — making them a common attacker target.

| Web Server | Common Use |
|---|---|
| **Apache** | Most popular for simple sites/blogs, especially WordPress |
| **Nginx** | Industry standard for high-performance apps (Netflix, Airbnb, GitHub) |
| **IIS (Internet Information Services)** | Microsoft's web server, common in enterprise environments |

---

### Best Practices — Securing the Three Components

#### Protecting the Application

| Measure | Description |
|---|---|
| **Secure Coding** | Avoid insecure functions, handle errors properly, remove sensitive info |
| **Input Validation & Sanitization** | Prevent injection attacks |
| **Access Control** | Restrict access based on user roles |

#### Protecting the Web Server

| Measure | Description |
|---|---|
| **Logging** | Detailed access logs of all web requests |
| **WAF (Web Application Firewall)** | Filters/blocks harmful traffic by rule |
| **CDN (Content Delivery Network)** | Reduces direct server exposure, often with an integrated WAF |

#### Protecting the Host Machine

| Measure | Description |
|---|---|
| **Least Privilege** | Run services under low-privilege users |
| **System Hardening** | Disable unnecessary services, close unused ports |
| **Antivirus** | Endpoint-level protection against known malware |

#### Security Tips for All Three

- **Strong Authentication** — restrict access to code, admin panels, and host machine
- **Patch Management** — keep app dependencies, web server, and host machine up to date

---

### Logging (Access Logs)

Web servers generate **access logs** for every request — recording client IP, timestamp, requested page/data, response status, and user agent. These fields are critical for investigation and tracing attacker behavior.

**Example benign log sequence:**

```
Client 10.10.10.100 → GET /index.html         (visits homepage)
Client 10.10.10.100 → GET /login.html         (navigates to login)
Client 10.10.10.100 → POST /login.html        (submits credentials)
Client 10.10.10.100 → GET /myaccount.html     (accesses account page)
```

> **GET** retrieves a resource (e.g., a page); **POST** submits data (e.g., login credentials). Even ordinary, expected sequences like this show how verbose access logs let analysts reconstruct a full browsing/attack sequence step by step.

---

### Content Delivery Network (CDN)

A CDN caches and serves content from **edge servers** positioned closer to users, reducing latency — while also acting as a security buffer between users and the origin server.

#### Security Benefits

| Benefit | Description |
|---|---|
| **IP Masking** | Hides the origin server's real IP, making direct targeting harder |
| **DDoS Protection** | Absorbs large traffic volumes, blunting denial-of-service attacks |
| **Enforced HTTPS** | Most CDNs enforce TLS encryption by default |
| **Integrated WAF** | Many CDNs (Cloudflare, Amazon CloudFront, Azure Front Door) bundle a WAF |

---

### Web Application Firewall (WAF)

A WAF inspects incoming HTTP traffic and blocks/logs potentially harmful requests based on security rules — like a bouncer checking every person (request) trying to enter a venue.

#### Types of WAFs

| Type | Description |
|---|---|
| **Cloud-based (Reverse Proxy)** | Sits in front of the web server; easy to deploy, highly scalable |
| **Host-based** | Software deployed directly on the web server; per-application control |
| **Network-based** | Physical/virtual appliance at the network perimeter; suited to enterprise environments |

#### WAF Detection Methods

| Method | Description | Example |
|---|---|---|
| **Signature-Based Detection** | Matches known attack patterns/payloads | Request with a User-Agent matching a known tool (e.g., `sqlmap/1.8.1`) |
| **Heuristic-Based Detection** | Analyzes request context/behavior | A long query string with special characters, e.g. `search?q=%3Cscript%20(1)` |
| **Anomaly & Behavioral Analysis** | Flags deviations from normal traffic | One IP making repeated login attempts in a short window |
| **Location & IP Reputation Filtering** | Blocks via geolocation/threat intel | A request from an IP outside the organization's normal business region |

> These detection methods aren't exhaustive — they continually evolve, and custom rules can be tailored to a specific application's needs. A Cloudflare dashboard, for example, shows total requests over 24 hours alongside everything the integrated WAF mitigated or blocked.

---

### Antivirus (AV)

AV is primarily built to protect **endpoints** (desktops, laptops, servers) from known malicious files — mostly via **signature-based detection** (comparing files against a known-malware database).

While web attacks typically target the application layer rather than the host machine, AV still plays a role in host protection — detecting malicious uploads like **web shells**, post-exploitation tools, and other malware. AV is one layer in a broader **defense-in-depth** strategy and should never stand alone.

---

### Web Application Security Quick Reference

| Concept | Summary |
|---|---|
| Web shift timeline | 1990s desktop → 2000s dynamic web → 2010s cloud/SaaS → today: browser-everything |
| Why web apps are targeted | Always online/exposed, often connected to backend systems |
| Real-world breaches | Equifax (Apache vuln, 2017), Capital One (misconfigured WAF, 2019) |
| 3 web service components | Application, Web Server, Host Machine |
| Common web servers | Apache, Nginx, IIS |
| Access log fields | Client IP, timestamp, requested resource, response status, user agent |
| GET vs. POST | GET retrieves a resource; POST submits data |
| CDN security benefits | IP masking, DDoS protection, enforced HTTPS, integrated WAF |
| WAF types | Cloud-based (reverse proxy), Host-based, Network-based |
| WAF detection methods | Signature-based, Heuristic-based, Anomaly/Behavioral, Location/IP reputation |
| AV's role | Endpoint protection against known malware (e.g., web shells); one layer of defense-in-depth |

---

## Summary

- The move from **desktop** (1990s) to **dynamic web apps** (2000s) to **cloud/SaaS** (2010s) to **browser-everything** (today) brings accessibility and faster updates but expands the attack surface, since web apps are always online, globally exposed, and often connected to backend systems — as the **Equifax** (2017, Apache vulnerability, ~150 million people) and **Capital One** (2019, misconfigured WAF, 100+ million customers) breaches show.
- A web service has three components — the **application**, the **web server** (Apache, Nginx, or IIS), and the **host machine** — and each needs its own protections: **secure coding, input validation, and access control** for the application; **logging, a WAF, and a CDN** for the web server; and **least privilege, system hardening, and antivirus** for the host, with **strong authentication** and **patch management** across all three.
- **Access logs** record the client IP, timestamp, requested resource, response status, and user agent for every request, letting analysts reconstruct a full browsing or attack sequence — including the difference between **GET** (retrieve) and **POST** (submit data) requests.
- A **CDN** masks the origin server's IP, absorbs DDoS traffic, enforces HTTPS, and often bundles a WAF (Cloudflare, Amazon CloudFront, Azure Front Door), while a **WAF** (cloud-based reverse proxy, host-based, or network-based) detects threats through **signature-based, heuristic-based, anomaly/behavioral,** and **location/IP reputation** methods.
- **Antivirus** protects endpoints from known malware via signatures and can catch uploaded web shells, but it is only one layer of a broader **defense-in-depth** strategy and should never stand alone.

## Related Documents

- [94. Snort In-Depth — IDS/IPS Modes, Parameters & Rule Writing](./94-snort-in-depth-ids-ips-modes-parameters-rule-writing.md)
- [96. Client-Side vs. Server-Side Web Attacks — Detection in Logs, Network Traffic & WAFs](./96-client-side-vs-server-side-web-attacks-detection-in-logs-network-traffic-wafs.md)
- [49. Web Application Fundamentals & HTTP Security Headers](./49-web-application-fundamentals-http-security-headers.md)
- [13. Web Infrastructure](./13-web-infrastructure.md)
- [59. Firewall Fundamentals](./59-firewall-fundamentals.md)
