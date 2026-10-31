# Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols

## Overview

This document applies Wireshark's filtering capabilities to identify specific attack patterns: a TCP flags reference, identifying Nmap TCP connect, SYN, and UDP scans, detecting ARP poisoning with a worked MITM investigation, identifying hosts and users via DHCP, NetBIOS, and Kerberos, detecting ICMP and DNS tunneling, analyzing cleartext protocols (FTP and HTTP, including User-Agent analysis and a Log4j example), decrypting HTTPS traffic with key log files, and two bonus features — extracting cleartext credentials and generating firewall ACL rules.

---

Applying Wireshark's filtering capabilities to identify specific attack patterns: network scans, ARP spoofing, host identification, protocol tunneling, and cleartext credential exposure.

---

### TCP Flags Reference

| Flag Combo | Filter (value) | Filter (named) |
|---|---|---|
| SYN only | `tcp.flags == 2` | `tcp.flags.syn == 1` |
| ACK only | `tcp.flags == 16` | `tcp.flags.ack == 1` |
| SYN, ACK | `tcp.flags == 18` | `(tcp.flags.syn==1) and (tcp.flags.ack==1)` |
| RST only | `tcp.flags == 4` | `tcp.flags.reset == 1` |
| RST, ACK | `tcp.flags == 20` | `(tcp.flags.reset==1) and (tcp.flags.ack==1)` |
| FIN only | `tcp.flags == 1` | `tcp.flags.fin == 1` |

---

### Identifying Nmap Scans

#### TCP Connect Scan (`nmap -sT`)

- **Completes** the full three-way handshake
- Used by **non-privileged (non-root)** users — their only option
- Typically has a **window size > 1024 bytes** (expects data, per protocol behavior)

| Port State | Pattern |
|---|---|
| Open | `SYN →` / `← SYN,ACK` / `ACK →` |
| Closed | `SYN →` / `← RST,ACK` |

**Detection filter:**
```
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size > 1024
```

#### SYN Scan (`nmap -sS`)

- **Never completes** the handshake (half-open scan)
- Requires **privileged/root** access
- Typically has a **window size ≤ 1024 bytes** (no data expected)

| Port State | Pattern |
|---|---|
| Open | `SYN →` / `← SYN,ACK` / `RST →` |
| Closed | `SYN →` / `← RST,ACK` |

**Detection filter:**
```
tcp.flags.syn==1 and tcp.flags.ack==0 and tcp.window_size <= 1024
```

#### UDP Scan (`nmap -sU`)

- No handshake required
- **Open ports:** typically no response
- **Closed ports:** trigger an **ICMP Type 3, Code 3** (Destination Unreachable — Port Unreachable) message

The ICMP error encapsulates the original request as data — expanding the ICMP section in the details pane reveals the original packet that triggered it.

**Detection filter:**
```
icmp.type==3 and icmp.code==3
```

---

### ARP Poisoning / Spoofing (MITM)

ARP has no authentication, isn't routable, and isn't secure by design — making it a classic MITM vector. The attacker's goal: manipulate IP-to-MAC mappings to intercept traffic meant for another host.

| Purpose | Filter |
|---|---|
| Global search | `arp` |
| ARP requests (opcode 1) | `arp.opcode == 1` |
| ARP responses (opcode 2) | `arp.opcode == 2` |
| Possible ARP scanning | `arp.dst.hw_mac==00:00:00:00:00:00` |
| Possible ARP poisoning | `arp.duplicate-address-detected or arp.duplicate-address-frame` |
| Possible ARP flooding from a MAC | `((arp) && (arp.opcode == 1)) && (arp.src.hw_mac == target-mac-address)` |

**Detection principle:** A suspicious signal is **two different ARP responses claiming the same IP** — Wireshark's Expert Info flags this, though only the second (conflicting) occurrence is highlighted, leaving the analyst to determine which entry is legitimate.

#### Worked ARP Investigation Example

Progressive findings while investigating suspicious ARP traffic:

| Finding | Detail |
|---|---|
| 1. IP-to-MAC match | MAC `...b4` → IP `192.168.1.25` |
| 2. Possible spoofing | Two MACs claim IP `192.168.1.1` (the likely gateway): `50:78:b3:f3:cd:f4` and `...b4` |
| 3. Possible flooding | MAC `...b4` crafts requests against a *range* of IPs |
| 4. Confirmed MITM | Adding **MAC address columns** to HTTP traffic reveals all HTTP packets destined to `...b4` — the attacker is relaying/intercepting victim traffic |

**Final roles identified:**

| Role | MAC | IP |
|---|---|---|
| Attacker | `00:0c:29:e2:18:b4` | `192.168.1.25` |
| Gateway | `50:78:b3:f3:cd:f4` | `192.168.1.1` |
| Victim | `00:0c:29:98:c7:a8` | `192.168.1.12` |

> **Key technique:** Adding MAC address as a column in the packet list pane reveals the actual link-layer relay path — invisible at the IP layer alone.

---

### Identifying Hosts and Users

Beyond IP↔MAC matching, three protocols reveal actual hostnames and usernames:

#### DHCP

| Purpose | Filter |
|---|---|
| Global search | `dhcp` or `bootp` |
| DHCP Request | `dhcp.option.dhcp == 3` |
| DHCP ACK | `dhcp.option.dhcp == 5` |
| DHCP NAK | `dhcp.option.dhcp == 6` |
| Hostname (Option 12) | `dhcp.option.hostname contains "keyword"` |
| Domain name (Option 15, in ACK) | `dhcp.option.domain_name contains "keyword"` |

Other relevant Request options: **Option 50** (requested IP), **Option 51** (lease time), **Option 61** (client MAC). NAK packets carry rejection reasons in **Option 56** — best read manually since messages vary case-by-case.

#### NetBIOS (NBNS)

| Purpose | Filter |
|---|---|
| Global search | `nbns` |
| Query name search | `nbns.name contains "keyword"` |

#### Kerberos

| Purpose | Filter |
|---|---|
| Global search | `kerberos` |
| Username search | `kerberos.CNameString contains "keyword"` |
| Usernames only (excluding hostnames) | `kerberos.CNameString and !(kerberos.CNameString contains "$")` |
| Protocol version | `kerberos.pvno == 5` |
| Domain/realm | `kerberos.realm contains ".org"` |
| Service name | `kerberos.SNameString == "krbtg"` |

> **Note:** In Kerberos, values ending in `$` typically indicate a **hostname** rather than a username — filter these out when hunting for actual user accounts.

---

### Tunneling Detection: ICMP and DNS

Tunneling smuggles data through trusted, typically-unfiltered protocols to bypass security perimeters.

#### ICMP Tunneling

Legitimate ICMP is small (typically ~64 bytes); tunneling often (but not always) produces **larger or anomalous packet sizes**, since extra payload data is being smuggled inside.

**Detection filter:**
```
data.len > 64 and icmp
```

> Sophisticated attackers can craft ICMP packets matching normal size — so size alone isn't conclusive, but it's a strong starting signal.

#### DNS Tunneling

Malware/C2 channels disguise commands as encoded **subdomains** — e.g., `encoded-commands.maliciousdomain.com`. Look for:

- Abnormal query length
- Non-regular/anomalous domain names
- Long addresses with encoded-looking subdomains
- Known tool signatures (e.g., **dnscat**, **dns2tcp**)
- Statistically anomalous DNS request volume to a single target

**Useful filter:** `!mdns` (excludes local mDNS device discovery noise)

---

### Cleartext Protocol Analysis

#### FTP

FTP prioritizes simplicity over security — exposing it to MITM, credential theft, phishing, malware planting, and data exfiltration.

| Response Series | Meaning |
|---|---|
| `1xx` | Information/status responses (e.g., 211 System status, 213 File status) |
| `2xx` | Connection messages (e.g., 220 Service ready, 227 Passive mode) |
| `3xx` | Authentication messages (e.g., 230 Login success, 331 Valid username, 530 Invalid login) |

| Filter | Purpose |
|---|---|
| `ftp.response.code == 230` | Successful logins |
| `ftp.response.code == 530` | Failed login attempts (brute-force signal) |
| `(ftp.response.code == 530) and (ftp.response.arg contains "username")` | Failed attempts targeting a specific username |
| `ftp.request.command == "USER"` | Username submissions |
| `ftp.request.command == "PASS"` | Password submissions |
| `(ftp.request.command == "PASS") and (ftp.request.arg == "password")` | Password spray signal (same password across targets) |

#### HTTP

Unencrypted and rarely blocked, HTTP analysis can surface phishing pages, web attacks, data exfiltration, and C2 traffic.

**Common response codes:**

| Code | Meaning |
|---|---|
| 200 | OK |
| 301/302 | Moved permanently/temporarily |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 405 | Method Not Allowed |
| 408 | Request Timeout |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

**Key filters:**

| Filter | Purpose |
|---|---|
| `http.request.method == "GET"` / `"POST"` | Filter by request type |
| `http.response.code == 404` (etc.) | Filter by response code |
| `http.user_agent contains "nmap"` | Detect scanning tool signatures |
| `http.request.uri contains "admin"` | Suspicious path targeting |
| `http.server contains "apache"` | Identify server software |
| `http.host contains "keyword"` | Filter by requested host |
| `data-text-lines contains "keyword"` | Search cleartext response body content |

#### User-Agent Analysis

The User-Agent field can reveal reconnaissance/exploitation tools — but **never whitelist based on it alone**, since attackers can spoof convincing values.

**Red flags:**
- Different User-Agents from the same host in a short window
- Non-standard/custom values
- Subtle spelling errors (e.g., "Mozlilla" vs. "Mozilla")
- Known audit tool signatures: **Nmap, Nikto, Wfuzz, sqlmap**
- Payload-like data embedded in the field

**Filter:**
```
(http.user_agent contains "sqlmap") or (http.user_agent contains "Nmap") or (http.user_agent contains "Wfuzz") or (http.user_agent contains "Nikto")
```

#### Log4j Detection Example

Known indicators from research: the exploit typically starts with a **POST** request and contains recognizable strings like `jndi:ldap` and `Exploit.class`.

**Filters:**
```
http.request.method == "POST"
(ip contains "jndi") or (ip contains "Exploit")
(frame contains "jndi") or (frame contains "Exploit")
(http.user_agent contains "$") or (http.user_agent contains "==")
```

---

### Decrypting HTTPS Traffic

HTTPS (via TLS) encrypts traffic content, hiding URLs and response data from plain packet inspection — legitimately for security, but also exploitable by attackers hosting malicious content over HTTPS.

**Key filters:**

| Filter | Purpose |
|---|---|
| `tls` | Global TLS search |
| `tls.handshake.type == 1` | TLS Client Hello |
| `tls.handshake.type == 2` | TLS Server Hello |
| `ssdp` | Local service discovery (SSDP) traffic |
| `(http.request or tls.handshake.type == 1) and !(ssdp)` | Isolate Client Hello, excluding SSDP noise |
| `(http.request or tls.handshake.type == 2) and !(ssdp)` | Isolate Server Hello, excluding SSDP noise |

#### Decryption via Key Log Files

An **SSLKEYLOGFILE** environment variable, configured in a browser (Chrome/Firefox support this), causes the browser to log per-session TLS key pairs as it browses. To use it for decryption:

1. Set up the `SSLKEYLOGFILE` environment variable **before** capturing traffic (keys must be logged during the session — they can't be generated retroactively)
2. In Wireshark: right-click, or `Edit → Preferences → Protocols → TLS`, and point to the key log file

Once applied, previously opaque HTTPS traffic becomes readable — revealing decrypted content in formats like:

- Frame
- Decrypted TLS
- Decompressed Header
- Reassembled TCP
- Reassembled SSL

---

### Bonus: Hunting Cleartext Credentials

Wireshark can automatically extract cleartext credentials from certain protocol dissectors: **FTP, HTTP, IMAP, POP, SMTP** (Wireshark v3.1+).

Access via `Tools → Credentials` — shows packet number, protocol, username, and related info in a clickable list, jumping directly to the relevant packet when clicked.

> Since this only works for specific protocols, don't rely on it exclusively — manual verification remains important.

---

### Bonus: Actionable Results — Firewall ACL Rules

Wireshark can generate ready-to-implement firewall rules directly from observed traffic via `Tools → Firewall ACL Rules`, producing IP/port/MAC-based rules for:

- **Netfilter** (iptables)
- **Cisco IOS** (standard/extended)
- **IP Filter** (ipfilter)
- **IPFirewall** (ipfw)
- **Packet filter** (pf)
- **Windows Firewall** (netsh, new/old format)

> Rules are intended for implementation on an **external-facing** firewall interface.

---

### Wireshark Traffic Analysis Quick Reference

| Concept | Summary |
|---|---|
| TCP Connect Scan | Full handshake, window >1024, non-privileged users |
| SYN Scan | Half-open (no handshake completion), window ≤1024, privileged users |
| UDP Scan | Closed ports → ICMP Type 3 Code 3; open ports often silent |
| ARP poisoning tell | Two conflicting ARP responses for the same IP |
| Host/user ID protocols | DHCP, NetBIOS (NBNS), Kerberos |
| Kerberos hostname tell | Values ending in `$` |
| Tunneling protocols | ICMP, DNS (via encoded subdomains) |
| ICMP tunneling signal | Packet size >64 bytes (not conclusive alone) |
| FTP response series | 1xx info, 2xx connection, 3xx auth |
| HTTP investigation focus | Methods, response codes, User-Agent, URI |
| User-Agent red flags | Inconsistency, typos, known tool signatures (Nmap, sqlmap, etc.) |
| Log4j indicators | POST method + `jndi:ldap` / `Exploit.class` strings |
| HTTPS decryption | Requires SSLKEYLOGFILE captured *during* the session |
| Credential hunting | `Tools → Credentials` (FTP/HTTP/IMAP/POP/SMTP only) |
| Firewall rule generation | `Tools → Firewall ACL Rules` |

---

## Summary

- **Nmap scans** leave distinct fingerprints: a **TCP connect scan** completes the handshake and typically uses a window size above 1024 (non-privileged users), a **SYN scan** never completes the handshake and typically uses a window of 1024 or less (privileged users), and a **UDP scan** triggers **ICMP Type 3, Code 3** replies from closed ports while open ports usually stay silent.
- **ARP poisoning** is the classic man-in-the-middle vector because ARP has no authentication; the telltale sign is **two different ARP responses claiming the same IP**, and in the worked example, adding MAC address columns to HTTP traffic exposed the attacker (`192.168.1.25`) relaying traffic between the victim (`192.168.1.12`) and the gateway (`192.168.1.1`).
- Hosts and users can be identified through **DHCP** (hostname, requested IP, lease, and NAK reason options), **NetBIOS** (`nbns.name`), and **Kerberos** (`CNameString`, where values ending in `$` are hostnames rather than usernames).
- **Tunneling** hides data in trusted protocols: **ICMP tunneling** often shows packets larger than ~64 bytes (`data.len > 64 and icmp`, though not conclusive alone), and **DNS tunneling** shows long or encoded-looking subdomains, tool signatures like dnscat or dns2tcp, and anomalous query volume.
- **Cleartext analysis** covers FTP (response codes 230/530 and `USER`/`PASS` commands for brute-force and spray detection) and HTTP (methods, response codes, URI and host filters, User-Agent red flags like inconsistency, typos, or tool signatures from Nmap, Nikto, Wfuzz, and sqlmap, and Log4j indicators such as a POST with `jndi:ldap` and `Exploit.class`).
- **HTTPS traffic** can be decrypted only if an `SSLKEYLOGFILE` was set up **before** the capture, and two bonus features round out the toolkit: `Tools → Credentials` (extracts cleartext credentials from FTP, HTTP, IMAP, POP, and SMTP) and `Tools → Firewall ACL Rules` (generates ready-to-use rules for iptables, Cisco IOS, ipfilter, ipfw, pf, and Windows Firewall).

## Related Documents

- [87. Wireshark Statistics & Advanced Filtering](./87-wireshark-statistics-advanced-filtering.md)
- [89. NetworkMiner — Network Forensic Analysis Tool](./89-networkminer-network-forensic-analysis-tool.md)
- [42. Nmap — Network Scanning & Enumeration](./42-nmap-network-scanning-enumeration.md)
- [85. Network Traffic Analysis (NTA) Fundamentals](./85-network-traffic-analysis-nta-fundamentals.md)
