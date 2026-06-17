## DNS — Domain Name System

---

## 1. What is DNS?

### The Problem DNS Solves
Every device on the internet has a unique **IP address** (e.g. `104.26.10.229`) — but remembering these numbers is impractical.

**DNS translates human-friendly domain names into IP addresses.**

```
tryhackme.com  ──────►  104.26.10.229
   (easy to remember)    (hard to remember)
```

> Think of it like a **phonebook** — you look up a name to find a number.

---
---

## 2. Anatomy of a Domain Name

```
        jupiter   .   servers   .   tryhackme   .   com
            │              │              │            │
        Subdomain     Subdomain    Second-Level     TLD
                                      Domain
```

---

### TLD (Top-Level Domain)
The **rightmost** part of a domain name.

| Type | Purpose | Examples |
|---|---|---|
| **gTLD** (Generic) | Originally indicated purpose | `.com`, `.org`, `.edu`, `.gov` |
| **ccTLD** (Country Code) | Geographic location | `.ca` (Canada), `.co.uk` (UK) |

> Newer gTLDs now exist too: `.online`, `.club`, `.biz`, etc. (2000+ TLDs in total)

---

### Second-Level Domain
The **main name** of the website — e.g. `tryhackme` in `tryhackme.com`

**Rules:**
- Max **63 characters** (plus TLD)
- Only `a-z`, `0-9`, and hyphens
- Cannot start/end with a hyphen or have consecutive hyphens

---

### Subdomain
Sits to the **left** of the Second-Level Domain — e.g. `admin` in `admin.tryhackme.com`

**Rules:**
- Same character rules as Second-Level Domain
- **Unlimited** number of subdomains allowed
- Total domain name length ≤ **253 characters**

Example with multiple subdomains:
```
jupiter.servers.tryhackme.com
   ↑        ↑
Subdomain Subdomain
```

---
---

## 3. DNS Record Types

| Record | Resolves To | Example |
|---|---|---|
| **A** | IPv4 address | `104.26.10.229` |
| **AAAA** | IPv6 address | `2606:4700:20::681a:be5` |
| **CNAME** | Another domain name | `store.tryhackme.com` → `shops.shopify.com` |
| **MX** | Mail server address | `alt1.aspmx.l.google.com` |
| **TXT** | Free text data | SPF, DKIM, domain verification |

---

### CNAME Example Flow
```
store.tryhackme.com ──► CNAME ──► shops.shopify.com ──► (new DNS lookup) ──► IP Address
```

### MX Record — Priority System
MX records include a **priority flag** — tells the client which mail server to try first, and which to fall back to if the main one is down.

### TXT Record Use Cases
- **Domain ownership verification** (third-party services)
- **SPF** — lists servers allowed to send email for the domain (anti-spam)
- **DMARC** — email authentication policy

```
@ TXT "v=spf1 ip4:192.0.2.0/24 include:_spf.google.com ~all"
```

---
---

## 4. How a DNS Request Works

### Step-by-Step Flow

```
[Your Computer]
      │
      ▼
1. Check Local Cache ──► Found? ──► ✅ Use cached result (DONE)
      │
      ▼ Not Found
2. Recursive DNS Server (usually from your ISP)
      │
      ▼ Check its cache ──► Found? ──► ✅ Return result (DONE)
      │
      ▼ Not Found
3. Root DNS Server
      │ "This is a .com domain"
      ▼
4. TLD Server (.com)
      │ "Here's the authoritative server for tryhackme.com"
      ▼
5. Authoritative Server (Nameserver)
      │ "Here's the actual IP address!"
      ▼
6. Result travels back up ──► Cached at Recursive DNS Server
      │
      ▼
[Your Computer receives the IP address]
```

---

### Key Players Explained

| Component | Role |
|---|---|
| **Local Cache** | Your device's own memory of recent lookups |
| **Recursive DNS Server** | Usually provided by your ISP; does the "legwork" of finding answers |
| **Root DNS Server** | The internet's "backbone" — directs to the correct TLD server |
| **TLD Server** | Holds records pointing to the authoritative server |
| **Authoritative Server (Nameserver)** | Stores the actual DNS records for the domain |

> Multiple nameservers usually exist per domain as **backups** — e.g. TryHackMe uses both `kip.ns.cloudflare.com` and `uma.ns.cloudflare.com`

---

### TTL (Time to Live)
- A value (in **seconds**) telling devices **how long to cache** a DNS response
- Saves the need to repeat the **entire lookup process** every time
- Shorter TTL = more up-to-date, but more requests
- Longer TTL = faster repeated lookups, but slower to update if changed

---
---

