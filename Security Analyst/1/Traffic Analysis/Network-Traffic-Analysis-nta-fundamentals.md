# Network Traffic Analysis (NTA) Fundamentals

## Overview

This document covers Network Traffic Analysis (NTA): why it goes beyond logs, illustrated by a DNS tunneling scenario; what each TCP/IP layer reveals (and which attacks hide at each); the two categories of traffic sources and two categories of traffic flow; the limits of logs; full packet capture via network TAPs and port mirroring with storage considerations; and flow-metadata protocols (NetFlow and IPFIX).

---

**Network Traffic Analysis (NTA)** is the process of capturing, inspecting, and analyzing data as it flows through a network, aiming for full visibility into internal and external communications. NTA is **not** just "using Wireshark" — it combines log correlation, deep packet inspection, and network flow statistics toward specific defensive goals.

---

### Why NTA Matters: A DNS Tunneling Scenario

**Alert:** Host `WIN-016` (`192.168.1.16`) is generating unusual DNS query volume, with repeated queries to the same TLD using different subdomains:

```
2025-10-03 09:15:23 SRC=192.168.1.16 QUERY=aj39skdm.malicious-tld.com QTYPE=A
2025-10-03 09:15:31 SRC=192.168.1.16 QUERY=msd91azx.malicious-tld.com QTYPE=A
2025-10-03 09:15:45 SRC=192.168.1.16 QUERY=cmd01.malicious-tld.com QTYPE=TXT
```

**What DNS logs alone tell us:** query/query type, subdomain/TLD (checkable via AbuseIPDB/VirusTotal), source host IP, destination IP, and timestamp — enough to build a timeline, but **not** enough to confirm intent, since logs don't capture query *content*.

**Why deeper inspection matters:** Threat actors can hide C2 instructions inside DNS **TXT records**. Inspecting the actual packet reveals the payload:

```
Queries: cmd1.evilc2.com: type TXT, class IN
Answers: cmd1.evilc2.com: type TXT, TTL 60, TXT length: 20
TXT: "SSBsb3ZlIHlvdXIgY3VyaW91c2l0eQ=="
```

This base64-encoded TXT response is a textbook example of **DNS tunneling** used for covert C2 — invisible to log-only analysis, but visible via full packet inspection.

---

### Why We Analyze Network Traffic

**General uses:**
- Monitoring network performance
- Detecting abnormalities (performance spikes, slowdowns)
- Inspecting suspicious internal/external communication (e.g., DNS exfiltration, malicious file downloads, lateral movement)

**SOC-specific uses:**
- Detecting suspicious/malicious activity
- Reconstructing attacks during incident response
- Verifying and validating alerts

**Additional scenario examples:**
- A system deviating from baseline behavior at a specific time, traced to a suspicious HTTP request delivering a malicious ZIP file
- Elevated DNS request volume traced to data exfiltration via DNS tunneling

---

### The TCP/IP Stack — What Each Layer Reveals

Logs typically capture only fragments of header data — never the full packet — which is exactly why deeper traffic analysis is necessary.

#### Application Layer

Contains both header information and the actual payload (application data). Logs (e.g., from web proxies) usually only capture the headers, **not** the payload.

**Example — HTTP GET request/response:**

```
GET /downloads/suspicious_package.zip HTTP/1.1
Host: www.tryhackrne.thn
User-Agent: curl/7.85.0
```

```
HTTP/1.1 200 OK
Content-Type: application/zip
Content-Length: 10485760
Content-Disposition: attachment; filename="suspicious_package.zip"
[binary ZIP file bytes follow — 10,485,760 bytes]
```

The headers alone confirm a ZIP file transfer succeeded — but nothing about the file's actual (malicious) contents.

#### Transport Layer

Encapsulates data into segments with TCP/UDP headers. Firewall logs often capture ports and flags but omit sequence numbers — which matter for detecting **session hijacking**.

**Example — session hijack attempt:**

```
1. SYN            (normal handshake)
2. SYN, ACK       (normal handshake)
3. ACK            (normal handshake)
4. PSH, ACK Seq=1 (legitimate data transfer)
5. ACK            (legitimate data transfer)
6. PSH, ACK Seq=34567232  ← massive sequence jump from a different source — injection attempt
```

#### Internet Layer

Adds IP-layer headers; large segments get fragmented (each fragment gets its own header). Most logs capture only source/destination IP and TTL — insufficient for detecting **fragmentation attacks**, which require inspecting the fragment offset and total length fields.

**Example — overlapping fragment attack:**

```
1. Offset=0,    Len=1480
2. Offset=1480, Len=1480
3. Offset=1480, Len=64   ← overlaps with fragment 2 — can be used to evade IDS reassembly logic
```

#### Link Layer

Adds MAC addressing. Standard logs capture MACs but miss subtler attacks like **ARP poisoning** — e.g., a single IP replying to multiple ARP requests with a spoofed MAC, or the same MAC appearing across multiple interfaces.

**Example — ARP poisoning:**

```
192.168.1.200 claims to be 192.168.1.10 (spoofed reply)
192.168.1.200 claims to be 192.168.1.1  (spoofed reply)
→ subsequent traffic gets relayed through the attacker's machine
```

---

### Traffic Sources and Flows

#### Two Source Categories

| Category | Description | Examples |
|---|---|---|
| **Intermediary** | Devices traffic passes *through*; generate comparatively little traffic themselves | Firewalls, switches, web proxies, IDS/IPS, routers, access points, WLAN controllers, ISP infrastructure |
| **Endpoint** | Devices where traffic originates/terminates; account for the bulk of bandwidth | Servers, hosts, IoT devices, printers, cloud resources, mobile devices |

Intermediary devices generate traffic via routing protocols (EIGRP, OSPF, BGP), management protocols (SNMP, PING), logging (SYSLOG), and supporting protocols (ARP, STP, DHCP).

#### Two Flow Categories

| Flow Type | Description | Common Protocols |
|---|---|---|
| **North-South** | Traffic entering/exiting the LAN via the firewall | HTTPS, DNS, SSH, VPN, SMTP, RDP |
| **East-West** | Traffic staying within the LAN (including LAN extended to cloud) | Directory/Auth services, file/print shares, infrastructure services, application communication, backup/replication, monitoring/management |

> **Why East-West matters despite being monitored less:** Attackers who compromise a network often exploit internal services to move laterally — making EW traffic critical to watch even though it doesn't cross the perimeter.

#### Example Flow Walkthroughs

| Flow | Summary |
|---|---|
| **HTTPS (TLS-inspecting proxy)** | Client believes it has one session with the web server, but actually has two: client↔proxy and proxy↔web server, with the proxy inspecting decrypted content in between |
| **External DNS** | Host → internal DNS server (checks cache) → router → firewall → external DNS server → same path back |
| **SMB with Kerberos** | Host authenticates to the Domain Controller's KDC → receives a Ticket Granting Ticket → requests a service ticket → uses that ticket to establish the SMB session |

---

### Logs — First Line of Visibility

No universal logging standard exists — each vendor implements logging (and decides what to log) independently.

**Example — Syslog auth log:**
```
Oct 8 11:20:15 web01 sshd[2145]: Accepted password for gensane from 192.168.1.50 port 52234 ssh2
```

**Example — Apache access log (CLF format):**
```
192.168.1.50 - - [08/Oct/2025:11:20:18 +0200] "GET /index.html HTTP/1.1" 200 2326 "-" "Mozilla/5.0"
```

Some protocols do offer standardized log transport (e.g., **Syslog**, **SNMP**), but logging *content* still varies by vendor.

---

### Going Deeper: Full Packet Capture

When logs aren't enough, capture the full packet via:

| Method | Description |
|---|---|
| **Network TAP** | A physical inline device that copies all traffic without affecting performance. Operates at the **link layer only** — no MAC/IP needed, since it copies raw electrical/light signals to a monitoring port. Near-zero performance impact. |
| **Port Mirroring** | A software approach copying packets from one port to another (e.g., Cisco's **SPAN**). Can also be configured virtually (e.g., VMware vSwitch) or in the cloud (e.g., AWS VPC Traffic Mirroring). |

**Example — Cisco SPAN configuration:**

```
Switch(config)# monitor session 1 source interface fastEthernet0/1
Switch(config)# monitor session 1 destination interface fastEthernet0/2
```

#### Best Practices

| Consideration | Why It Matters |
|---|---|
| **Placement** | The TAP/mirror must sit where the traffic of interest actually flows |
| **Duration/Storage** | Full packet capture demands significant storage — capturing a full 1 Gbps line for a day requires roughly **10.8 TB** |
| **TAP vs. Mirror** | Physical TAPs cause near-zero performance impact; port mirroring can degrade performance under heavy traffic load |

#### Analysis Tools

- **Wireshark**
- **TCPdump**
- **IDS/IPS tools:** Snort, Suricata, Zeek

---

### Network Statistics — NetFlow & IPFIX

Rather than full packets, these protocols capture **metadata** about traffic flows (e.g., counting DNS requests per host) — useful for spotting C2 traffic, data exfiltration, and lateral movement patterns.

| Protocol | Notes |
|---|---|
| **NetFlow** | Developed by Cisco; originally proprietary, gained cross-vendor templating support from v9 onward |
| **IPFIX** | The vendor-neutral successor to NetFlow, developed via IETF collaboration; offers similar features with more configurable field capture |

Most modern NGFWs, IPS, and IDS support NetFlow/IPFIX natively — no dedicated new infrastructure is typically required, just enabling and configuring the protocol with a destination to send the metadata.

---

### NTA Quick Reference

| Concept | Summary |
|---|---|
| NTA = | Capturing, inspecting, and analyzing network traffic for full visibility |
| Key insight | Logs capture header fragments, never full packet content — full inspection fills the gap |
| TCP/IP layers examined | Application, Transport, Internet, Link |
| Layer-specific attack examples | Session hijacking (Transport), fragmentation attacks (Internet), ARP poisoning (Link) |
| 2 Traffic source types | Intermediary, Endpoint |
| 2 Traffic flow types | North-South (perimeter-crossing), East-West (internal) |
| Full packet capture methods | Network TAP (link-layer, physical), Port Mirroring/SPAN (software) |
| Capture tools | Wireshark, TCPdump, Snort/Suricata/Zeek |
| Flow-metadata protocols | NetFlow (Cisco), IPFIX (vendor-neutral successor) |
| Storage consideration | ~10.8 TB/day for full 1 Gbps capture |

---

## Summary

- **NTA** captures, inspects, and analyzes network traffic for full visibility, combining log correlation, deep packet inspection, and flow statistics — a DNS tunneling scenario shows why: logs reveal query names and timing, but only full packet inspection exposes the base64-encoded C2 instructions hidden in a TXT record.
- Logs capture only fragments of header data, so each TCP/IP layer hides attacks that logs miss: payload contents at the **application** layer, sequence-number jumps signalling **session hijacking** at the transport layer, overlapping fragments in **fragmentation attacks** at the internet layer, and spoofed replies in **ARP poisoning** at the link layer.
- Traffic comes from **intermediary** devices (firewalls, switches, proxies, routers) and **endpoint** devices (servers, hosts, IoT), and flows either **north-south** (crossing the perimeter) or **east-west** (staying inside the LAN) — east-west traffic is monitored less but is where attackers move laterally.
- When logs aren't enough, **full packet capture** is done with a physical **network TAP** (near-zero performance impact, link layer only) or software **port mirroring/SPAN**, with placement and storage as key considerations — a fully saturated 1 Gbps link needs roughly 10.8 TB per day — analyzed with Wireshark, TCPdump, or IDS/IPS tools like Snort, Suricata, and Zeek.
- **NetFlow** (Cisco) and **IPFIX** (its vendor-neutral successor) capture traffic **metadata** instead of full packets, making them a lighter-weight way to spot C2 traffic, exfiltration, and lateral movement, and most modern NGFWs, IPS, and IDS support them natively.


