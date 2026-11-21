# Man-in-the-Middle (MITM) Attack Detection — ARP Spoofing, DNS Spoofing & SSL Stripping

## Overview

This document covers detecting Man-in-the-Middle (MITM) attacks: how they work and the common types, where MITM fits in the Cyber Kill Chain, and detailed Wireshark detection walkthroughs for ARP spoofing, DNS spoofing, and SSL stripping — culminating in a full chained attack timeline showing how all three techniques combine into one credential-stealing attack.

---

A **Man-in-the-Middle (MITM) attack** is a cyberattack where an attacker secretly intercepts and potentially alters communication between two parties without their knowledge — eavesdropping to steal credentials/credit card info, or injecting malicious content.

---

### How MITM Attacks Work

Two main steps:

1. **Interception** — the attacker inserts themselves into a communication stream, often via ARP, DNS, or IP spoofing
2. **Manipulation/Decryption** — the attacker accesses or modifies the communication, decrypting data or injecting harmful content (altered responses, fake login forms)

### Common Types of MITM Attacks

| Type | Description |
|---|---|
| **Packet sniffing** | Capturing unencrypted packets, often on open Wi-Fi |
| **Session hijacking** | Stealing/reusing session tokens to impersonate users |
| **SSL stripping** | Downgrading HTTPS to insecure HTTP to steal/alter data |
| **DNS spoofing** | Redirecting legitimate traffic to fraudulent domains via manipulated DNS responses |
| **IP spoofing** | Crafting malicious packets appearing to come from trusted systems |
| **Rogue Wi-Fi access point** | Fake networks to intercept user traffic |

> **Real-world example:** The 2017 Equifax breach involved an MITM attack exposing sensitive data for over 100 million users. High-profile ISP/state-actor cases have also used SSL spoofing to intercept search traffic.

---

### MITM and the Cyber Kill Chain

MITM is a versatile technique primarily leveraged during the **Exploitation** and **Installation** phases of the Cyber Kill Chain (see Section 78):

| Role | How MITM Fits |
|---|---|
| **As an Exploitation Technique** | Exploits the inherent trust/design limitations of protocols like ARP and DNS; manipulating them to intercept a communication channel violates network integrity and provides the adversary's initial foothold |
| **As an Installation Vector** | Once positioned, the attacker controls the data stream and can inject malicious payloads (browser exploits, droppers, RATs) into legitimate unencrypted downloads — establishing persistence or deploying further tools |

> Detecting an MITM attack is significant — it means the adversary is mid-intrusion, giving the SOC a critical window to disrupt the attack chain before final objectives are reached.

---

### ARP Spoofing

**ARP (Address Resolution Protocol)** maps IP addresses to MAC addresses on a local network via "who has this IP?" / "is-at" exchanges.

**ARP spoofing:** An attacker sends fake ARP replies to trick devices into associating the attacker's MAC with a legitimate IP (usually the default gateway) — intercepting, modifying, or redirecting traffic.

#### Why It Works

ARP has **no authentication** — any device can send unsolicited "is-at" messages. An attacker sends a fake reply (e.g., `192.168.10.100 is at <attacker-MAC>` claiming to be the gateway), poisoning the victim's ARP cache so all gateway-bound traffic flows through the attacker first.

#### Indicators of Attack

| Indicator | Description |
|---|---|
| **Duplicate MAC-to-IP Mappings** | Multiple MACs claiming the same IP — impersonation |
| **Unsolicited ARP Replies** | High volume of replies without matching requests ("gratuitous ARP") |
| **Abnormal ARP Traffic Volume** | Large packet counts in short intervals |
| **Unusual Traffic Routing** | Traffic rerouted through the attacker's MAC |
| **Gateway Redirection Patterns** | Multiple destination MACs claiming the same gateway IP |
| **ARP Probe/Reply Loops** | Repeated "who has X? tell Y" patterns |

#### Detecting via Wireshark

| Step | Filter |
|---|---|
| All ARP traffic | `arp` |
| ARP requests only | `arp.opcode == 1` |
| ARP responses only | `arp.opcode == 2` |
| Gratuitous (unsolicited) ARP | `arp.isgratuitous` |
| Traffic tied to known gateway MAC | `arp && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == 02:aa:bb:cc:00:01` |
| Replies claiming the gateway IP | `arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1` |
| Confirm via info column | `arp.opcode == 2 && _ws.col.info contains "192.168.10.1 is at"` |
| Isolate attacker's MAC | `arp.opcode == 2 && arp.src.proto_ipv4 == 192.168.10.1 && eth.src == <attacker-MAC>` |
| Duplicate address detection | `arp.duplicate-address-detected \|\| arp.duplicate-address-frame` |

**Finding:** Repeated ARP replies pointing the gateway's IP to a suspicious MAC address — confirming the attacker performed ARP spoofing and positioned themselves between the victim and the gateway.

---

### DNS Spoofing (DNS Cache Poisoning)

DNS translates human-friendly names into IPs — spoofing corrupts this, giving a victim's machine the wrong IP for a domain.

#### How It Enables MITM

```
Victim queries my-real-bank.com
    → Attacker (already on-network, e.g., via ARP spoofing) intercepts the query
    → Attacker sends a fake DNS response: "my-real-bank.com is at <attacker-IP>"
    → Victim's cache trusts and stores the spoofed entry
    → Victim connects to the attacker's server (a replica of the real site)
    → Attacker captures everything typed, including credentials
```

#### Indicators of Attack

| Indicator | Description |
|---|---|
| **Multiple DNS responses for the same query** | A legitimate resolver AND a forged responder both reply — the single most reliable indicator |
| **DNS response from an unexpected source** | Reply from an IP not matching any configured resolver |
| **Suspiciously short TTLs** | Attackers use very low TTLs (1–30s) to keep poisoned entries short-lived and easily reassertable |
| **Unsolicited DNS responses** | A reply with no corresponding request from the victim |

#### Detecting via Wireshark

| Step | Filter |
|---|---|
| All DNS traffic | `dns` |
| Legitimate responses (baseline, e.g., from 8.8.8.8) | `dns.flags.response == 1 && ip.src == 8.8.8.8` |
| All DNS responses | `dns.flags.response == 1` |
| Traffic for the domain of interest | `dns && dns.qry.name == "corp-login.acme-corp.local"` |
| Legit responses for that domain | `dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"` |
| **Responses NOT from the real DNS server** | `dns.flags.response == 1 && ip.src != 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"` |

**Finding:** A rogue internal host acting as a fake DNS server, sending spoofed responses for the target domain — confirming DNS spoofing.

#### Multi-Stage Attack Correlation

The analysis so far reveals a **chained attack**: the attacker first poisoned the ARP mapping for the gateway, then used that position to inject forged DNS replies redirecting the victim to the attacker's IP.

---

### SSL Stripping

An MITM technique where the attacker intercepts and modifies traffic to **remove or prevent TLS encryption** between client and server — the client ends up communicating over plain HTTP while the attacker maintains a secure HTTPS session with the real server, relaying (and reading) everything in between.

#### How It Works

```
Victim initiates HTTPS request
    → Attacker intercepts (via ARP spoofing / rogue AP)
    → Attacker connects to the real site over HTTPS
    → Attacker relays the response to the victim over plain HTTP
    → Victim unknowingly transmits sensitive data in plaintext
```

#### Indicators of SSL Stripping

| Indicator | Description |
|---|---|
| **Initial request vs. response mismatch** | Request starts as HTTPS (443) but subsequent packets shift to HTTP (80) for the same domain |
| **Redirects/Link Rewriting** | Persistent 301/302 redirects from HTTPS to an HTTP resource |
| **Certificate Errors** | Failed TLS handshake or self-signed cert shown if the attacker proxies more directly |

#### Detecting via Wireshark

| Step | Filter |
|---|---|
| All SSL/TLS traffic | `tls \|\| ssl` |
| Confirm the domain normally uses TLS | `tls.handshake.type == 1 && tls.handshake.extensions_server_name == "corp-login.acme-corp.local"` |
| DNS spoofing evidence preceding the strip | `dns.flags.response == 1 && ip.src == <attacker-IP> && dns.qry.name == "corp-login.acme-corp.local"` |
| Confirm TLS absence post-spoof (plain HTTP to attacker) | `http && ip.src == <victim-IP> && ip.dst == <attacker-IP>` |

**Finding:** Following the DNS spoof, the victim connects to the attacker over plain HTTP instead of HTTPS, and submits login credentials — visible **in cleartext** within the HTTP stream, confirming successful credential capture via SSL stripping.

---

### Full Attack Timeline — Chained MITM Investigation

```
1. ARP Spoofing (cache poisoning)
   → Attacker sends unsolicited ARP "is-at" claiming the gateway's IP

2. DNS Spoofing (forged responses)
   → Victim queries corp-login.acme-corp.local
   → Attacker's forged DNS response redirects the domain to the attacker's IP

3. SSL Stripping (TLS downgrade / credential capture)
   → Victim connects to the resolved (attacker) IP over HTTP instead of HTTPS
   → Credential POST observed in cleartext
```

This demonstrates how three distinct MITM techniques — ARP spoofing, DNS spoofing, and SSL stripping — chain together into one complete, credential-stealing attack, each stage building on the position gained by the previous one.

---

### MITM Detection Quick Reference

| Concept | Summary |
|---|---|
| MITM = | Secretly intercepting/altering communication between two parties |
| 2 Core steps | Interception, Manipulation/Decryption |
| 6 Common types | Packet sniffing, Session hijacking, SSL stripping, DNS spoofing, IP spoofing, Rogue AP |
| Kill Chain phases | Exploitation (interception) and Installation (payload injection) |
| ARP spoofing root cause | No authentication in ARP protocol |
| ARP tell | Duplicate MAC-to-IP mappings, gratuitous ARP replies |
| DNS spoofing tell | Multiple responses for one query; response from unexpected source; short TTLs |
| SSL stripping tell | HTTPS request followed by HTTP response for the same domain |
| Chained attack pattern | ARP Spoofing → DNS Spoofing → SSL Stripping → Credential Theft |
| Primary analysis tool | Wireshark (ARP/DNS/TLS/HTTP filters + Follow TCP Stream) |

---

## Summary

- A **Man-in-the-Middle (MITM) attack** secretly intercepts and potentially alters communication between two parties in two steps — **interception** (via ARP, DNS, or IP spoofing) and **manipulation/decryption** — and takes forms including packet sniffing, session hijacking, SSL stripping, DNS spoofing, IP spoofing, and rogue Wi-Fi access points.
- In the Cyber Kill Chain, MITM serves as an **Exploitation** technique (abusing the trust built into protocols like ARP and DNS to gain a foothold) and an **Installation** vector (injecting payloads into unencrypted traffic), and detecting it means the adversary is mid-intrusion, giving the SOC a window to disrupt the chain.
- **ARP spoofing** works because ARP has no authentication; its indicators are duplicate MAC-to-IP mappings, unsolicited/gratuitous replies, abnormal volume, and multiple MACs claiming the gateway IP — surfaced in Wireshark with filters like `arp.opcode == 2`, `arp.isgratuitous`, and `arp.duplicate-address-detected`.
- **DNS spoofing** is most reliably indicated by multiple responses to the same query, replies from an unexpected source, very short TTLs (1–30 seconds), and unsolicited responses — found by filtering for `dns.flags.response == 1 && ip.src != <real-resolver>`; **SSL stripping** shows as an HTTPS request followed by plain HTTP for the same domain, with credentials visible in cleartext in the HTTP stream.
- The three techniques **chain together** — ARP spoofing positions the attacker, DNS spoofing redirects the victim to the attacker's IP, and SSL stripping downgrades the connection to HTTP so credentials can be captured — each stage building on the position gained by the previous one.

## Related Documents

- [92. Data Exfiltration Detection (DNS, FTP, HTTP, ICMP)](./92-data-exfiltration-detection-dns-ftp-http-icmp.md)
- [94. Snort In-Depth — IDS/IPS Modes, Parameters & Rule Writing](./94-snort-in-depth-ids-ips-modes-parameters-rule-writing.md)
- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
- [78. The Cyber Kill Chain](./78-the-cyber-kill-chain.md)
- [4. Address Resolution Protocol (ARP)](./04-address-resolution-protocol-arp.md)
