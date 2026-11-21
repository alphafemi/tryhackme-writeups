# Network Discovery & Port Scanning Detection

## Overview

This document covers network discovery from both the attacker's and the defender's perspective and the challenge of telling good scanning from bad, the differences between external and internal scanning (including MITRE ATT&CK phase, severity, and response), reading scanning activity in exported logs, horizontal vs. vertical port scans, scan mechanics (ping sweep, TCP SYN scan, UDP scan), and the SOC practice of excluding authorized scans from detections.

---

Before launching an attack, adversaries first discover what's reachable — the **attack surface**. Defenders perform similar discovery for different reasons, making it a key SOC challenge to tell the two apart.

---

### Attackers and Network Discovery

During discovery, an attacker tries to answer:

- What assets can I access?
- What are the IP addresses, ports, OS, and services running on them?
- What service versions are running — do any have exploitable vulnerabilities?

In short: the attacker is searching for an opening to exploit the network.

### Defenders and Network Discovery

Defenders run similar scanning activity, but toward different goals:

- **Inventory** the organization's assets, ensuring everything is documented
- Ensure no unnecessary IP, port, or service is exposed — only what's needed stays open
- Ensure vulnerabilities are patched (at minimum, the exploitable ones)

In short: defenders aim to **reduce the attack surface** as much as possible.

---

### The Challenge: Telling Good Scanning from Bad

Attackers, defenders, research organizations, web crawlers, and search engines all perform similar discovery actions — making differentiation genuinely hard. SOC teams typically use a combination of:

| Technique | Description |
|---|---|
| **Allowlisting** | Known internal and benign external scanners are excluded from triggering alerts |
| **Threat Intel Integration** | Flag scanning activity only from known malicious/suspicious sources |
| **Severity Weighting** | Since allowlisting-by-source can miss real threats, some teams instead use threat intel to *raise severity* on matched alerts, while keeping generic scanning-behavior detections active as a backstop |

---

### External vs. Internal Scanning

#### External Scanning Activity

Scanning originating **outside** the organization's network, targeting internal/perimeter assets.

| Trait | Detail |
|---|---|
| **Source IP** | External |
| **Destination IP** | Belongs to the organization |
| **MITRE ATT&CK phase** | **Reconnaissance** — the attacker has no foothold yet; this is early-stage probing |
| **Severity** | Low — still pre-compromise |
| **Response** | Block the offending IP(s) on the perimeter firewall (note: the attacker may return under a different masked IP) |

#### Internal Scanning Activity

Scanning where **both** source and destination are internal (private) IP addresses — the scan originates *inside* the network.

| Trait | Detail |
|---|---|
| **Source IP** | Internal/private |
| **Destination IP** | Internal/private |
| **MITRE ATT&CK phase** | **Discovery** (sometimes called "internal reconnaissance") — the attacker already has a foothold and is preparing for lateral movement |
| **Severity** | High |
| **Response** | After ruling out authorized activity, escalate and initiate Incident Response — blocking the source IP alone is insufficient; deeper investigation and root cause analysis are required |

---

### Reading Scanning Activity in Logs

Real-world SIEM-exported logs (e.g., Zeek-based CSV exports) are often dense and hard to read directly.

**Example log preview (`head -n2 log-session-1.csv`):**

```
"@timestamp","source.ip","source.port","destination.ip","destination.port","rule.name","rule.category","rule.action","network.protocol",message,"event.dataset"
"Sep 7, 2025 @ 17:16:42.944","203.0.113.25",39120,"192.168.230.145",5922,"-","-","-","-","{...zeek.conn JSON...}","zeek.conn"
```

Once the file structure is understood, the `cut` utility (with `,` as delimiter) can filter specific columns for analysis — though care is needed since embedded commas (e.g., inside the timestamp or JSON message field) can throw off naive column counting.

---

### Types of Port Scans

#### Horizontal Scanning

Scanning the **same port across multiple destination IPs** — used to find which hosts expose a particular service.

```
Pattern: Same source IP → Same destination port → Multiple destination IPs
```

> **Example:** WannaCry ransomware spread by scanning for machines with port **445** (SMB) open, exploiting an SMBv1 vulnerability across every host it found listening on that port.

#### Vertical Scanning

Scanning **multiple ports on a single destination IP** — used to footprint one specific host and identify all its exposed services.

```
Pattern: Same source IP → Same destination IP → Multiple destination ports
```

> **Example:** If an organization exposes only one internet-facing server, an attacker targeting that organization would vertically scan that single server to map its open ports and running services.

> **Note:** Attackers sometimes combine both approaches (mixed horizontal/vertical scanning) to gain the benefits of each.

---

### Scan Mechanics

#### Ping Sweep

The most basic technique — identifies which hosts are online by sending an **ICMP** packet; an online host replies with its own ICMP packet. Increasingly blocked by modern security controls, making it easy to defeat.

#### TCP SYN Scan

Exploits the TCP three-way handshake (SYN → SYN-ACK → ACK):

```
Scanner sends SYN → Host replies SYN-ACK → Host is online AND that port is open
```

Considered a **stealthy** scan — it often blends in with legitimate traffic and is harder to detect than a full connection.

#### UDP Scan

Sends a (usually empty) UDP packet and interprets the response:

| Response | Interpretation |
|---|---|
| **ICMP Port Unreachable** | Port is closed, but host is online |
| **No response until timeout** | Port is marked open — but this is **not conclusive** evidence |
| **A UDP packet reply** | Rare, but confirms the port is genuinely open |

> UDP scanning is inherently **unreliable and slow**, since it often depends on waiting for a timeout rather than a definitive response.

---

### SOC Practice: Excluding Authorized Scans

Most organizations run their own internal scans for vulnerability checks, rogue-asset monitoring, and attack-surface reduction. A SOC analyst should know:

- The source IP(s) these authorized scans run from
- The scan type being performed
- The scanning schedule

Ideally, these known-good scans are **excluded from detection use cases** to reduce alert noise — freeing analyst attention for genuinely suspicious scanning activity.

---

### Network Discovery & Scanning Quick Reference

| Concept | Summary |
|---|---|
| Attacker discovery goal | Find an exploitable opening |
| Defender discovery goal | Reduce the attack surface |
| Differentiation techniques | Allowlisting, Threat Intel matching, Severity weighting |
| External scanning | Outside → inside; MITRE phase: Reconnaissance; Low severity; block the IP |
| Internal scanning | Inside → inside; MITRE phase: Discovery; High severity; escalate to IR |
| Horizontal scan | Same port, many destination IPs (e.g., WannaCry hunting for SMB/445) |
| Vertical scan | Same destination IP, many ports (footprinting one target) |
| Ping Sweep | ICMP-based host discovery; easily blocked |
| TCP SYN Scan | Uses handshake behavior; stealthy, blends with normal traffic |
| UDP Scan | Unreliable/slow; relies on ICMP Unreachable or timeout inference |
| SOC best practice | Document and exclude known authorized scans from detections |

---

## Summary

- Attackers run discovery to find an **exploitable opening** (reachable assets, ports, OS, services, and vulnerable versions), while defenders run similar scans to **inventory assets and reduce the attack surface** — so telling the two apart is a core SOC challenge, handled through **allowlisting**, **threat intel integration**, and **severity weighting** (raising severity on matched threat intel while keeping generic scan-behavior detections as a backstop).
- **External scanning** (outside → inside) maps to the MITRE **Reconnaissance** phase, is **low severity** since it is pre-compromise, and is handled by blocking the IP at the perimeter firewall; **internal scanning** (inside → inside) maps to **Discovery**, is **high severity** because the attacker already has a foothold, and requires escalation to incident response rather than just blocking the source IP.
- **Horizontal scans** hit the same port across many IPs (WannaCry hunting for SMB on port 445), while **vertical scans** hit many ports on a single IP to footprint one host; attackers sometimes mix both.
- Scan mechanics include the **ping sweep** (ICMP, easily blocked), the **TCP SYN scan** (stealthy, exploits handshake behavior), and the **UDP scan** (slow and unreliable, inferring state from ICMP Port Unreachable replies or timeouts).
- Exported logs such as Zeek CSVs can be read with tools like `cut`, mindful of embedded commas, and a good SOC practice is to **document and exclude known authorized scans** (source IPs, scan type, schedule) from detection use cases to cut alert noise.

## Related Documents

- [90. Network Perimeter Monitoring & Log Investigation](./90-network-perimeter-monitoring-log-investigation.md)
- [92. Data Exfiltration Detection (DNS, FTP, HTTP, ICMP)](./92-data-exfiltration-detection-dns-ftp-http-icmp.md)
- [42. Nmap — Network Scanning & Enumeration](./42-nmap-network-scanning-enumeration.md)
- [43. Active Reconnaissance Techniques](./43-active-reconnaissance-techniques.md)
- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
