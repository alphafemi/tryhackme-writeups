# NetworkMiner — Network Forensic Analysis Tool

## Overview

This document covers NetworkMiner, an open-source Network Forensic Analysis Tool (NFAT): its role in forensics and supported data types, core capabilities, operating modes, pros and cons, a feature comparison with Wireshark and the recommended combined workflow, the interface and its key analysis tabs, and the differences between versions 1.6 and 2.7+.

---

**NetworkMiner** is an open-source **Network Forensic Analysis Tool (NFAT)** for Windows, Linux, macOS, and FreeBSD. It works two ways: as a passive sniffer that fingerprints hosts/sessions/ports without generating traffic, or as a **PCAP parser** that reassembles files and certificates from captured traffic for offline review.

---

### NetworkMiner in Forensics

Network forensics aims to detect malicious activity, breaches, and anomalies from traffic data. NetworkMiner provides quick starting points:

- Context on captured hosts (IP/MAC, hostnames, OS)
- Potential attack indicators (traffic spikes, port scans)
- Tooling signatures used in an attack (e.g., Nmap)

### Supported Data Types

Of the three network forensics data types (**Live Traffic, Traffic Captures, Log Files**), NetworkMiner handles the first two: **live traffic** and **PCAP files**.

---

### Core Capabilities

| Capability | Description |
|---|---|
| **Traffic sniffing** | Intercepts and logs live packets |
| **PCAP parsing** | Displays detailed packet content from a capture file |
| **Protocol analysis** | Identifies protocols used within the capture |
| **OS fingerprinting** | Identifies host OS (relies on **Satori** and **p0f**) |
| **File extraction** | Pulls images, HTML files, and emails from the capture |
| **Credential grabbing** | Extracts credentials found in the capture |
| **Cleartext keyword parsing** | Extracts cleartext strings/keywords |

*(This covers NetworkMiner Free — the Professional edition adds more capabilities.)*

---

### Operating Modes

| Mode | Notes |
|---|---|
| **Sniffer Mode** | Windows-only; not considered reliable enough to be a primary sniffer — not a substitute for dedicated tools like Wireshark/tcpdump |
| **Packet Parsing/Processing** | The recommended primary use — quickly surfaces the "low-hanging fruit" before deeper investigation elsewhere |

---

### Pros and Cons

| Pros | Cons |
|---|---|
| OS fingerprinting | Not useful for active sniffing |
| Easy file extraction | Poor performance on large PCAPs |
| Credential grabbing | Limited filtering |
| Cleartext keyword parsing | Not built for manual, deep traffic investigation |
| Fast overall overview | |

---

### NetworkMiner vs. Wireshark

| Feature | NetworkMiner | Wireshark |
|---|---|---|
| **Purpose** | Quick overview, mapping, extraction | In-depth analysis |
| GUI | ✅ | ✅ |
| Sniffing | ✅ | ✅ |
| Handling PCAPs | ✅ | ✅ |
| **OS Fingerprinting** | ✅ | ❌ |
| **Parameter/Keyword Discovery** | ✅ | Manual |
| Credential Discovery | ✅ | ✅ |
| File Extraction | ✅ | ✅ |
| **Filtering Options** | Limited | ✅ |
| **Packet Decoding** | Limited | ✅ |
| **Protocol Analysis** | ❌ | ✅ |
| **Payload Analysis** | ❌ | ✅ |
| **Statistical Analysis** | ❌ | ✅ |
| **Host Categorisation** | ✅ | ❌ |

> **Best practice:** Record traffic for offline analysis, get a **quick overview with NetworkMiner**, then **go deep with Wireshark** for detailed investigation.

---

### Interface Overview

| Menu | Function |
|---|---|
| **File** | Load a PCAP (or drag-and-drop), or receive PCAP over IP |
| **Tools** | Clear the dashboard/remove captured data |
| **Help** | Version info and update details |
| **Case Panel** | Lists loaded PCAP files; reload, view metadata, or remove files (right-click → "Show Metadata") |

---

### Key Analysis Tabs

| Tab | Provides |
|---|---|
| **Hosts** | IP/MAC, OS type, open ports, sent/received packets, session counts, host details (OSINT lookup is premium-only) |
| **Sessions** | Frame number, client/server address, ports, protocol, start time — filterable by ExactPhrase, AllWords, AnyWord, or RegExp |
| **DNS** | Frame number, timestamp, client/server, ports, IP TTL, DNS time, transaction ID/type, query/answer (Alexa Top 1M lookup is premium-only) |
| **Credentials** | Extracted credentials/hashes — Kerberos, NTLM, RDP cookies, HTTP cookies/requests, IMAP, FTP, SMTP, MS SQL; export to **Hashcat** or **John the Ripper** for cracking |
| **Files** | Extracted files — filename, extension, size, source/destination, protocol, timestamp, reconstructed path (OSINT hash lookup/sample submission are premium-only) |
| **Images** | Extracted images with zoom and hover-to-view source/destination/path details |
| **Parameters** | Extracted parameter name/value pairs, with frame/host/port/timestamp context |
| **Keywords** | Custom keyword search across all processed data — **requires reloading case files** after adding new keywords |
| **Messages** | Extracted emails/chats — sender, receiver, protocol, size, timestamp, with attachment/attribute viewing |
| **Anomalies** | Flags select known patterns — e.g., EternalBlue exploit indicators and spoofing attempts (NetworkMiner is **not** a full IDS) |

---

### Version Differences (v1.6 vs. v2.7)

| Feature | v1.6 | v2.7+ |
|---|---|---|
| **MAC Address correlation** (conflict detection) | ❌ | ✅ |
| **Detailed packet handling** | ✅ | ❌ |
| **Frame processing** (frame counts/details) | ✅ | ❌ |
| **Extensive parameter capture** | Limited | ✅ (captures more parameters) |
| **Unified cleartext data view** | ✅ (single tab, but no packet matching) | ❌ |

> Later versions trade some legacy detail views (frame processing, unified cleartext tab) for stronger correlation and parameter capture capabilities.

---

### NetworkMiner Quick Reference

| Concept | Summary |
|---|---|
| NetworkMiner = | Open-source Network Forensic Analysis Tool (NFAT) |
| Primary use | PCAP parsing for a quick overview, not deep manual investigation |
| Best-practice workflow | NetworkMiner for overview → Wireshark for depth |
| OS fingerprinting basis | Satori, p0f |
| Credential types extracted | Kerberos, NTLM, RDP cookies, HTTP, IMAP, FTP, SMTP, MS SQL |
| Cracking tools to pair with | Hashcat, John the Ripper |
| Keyword search caveat | Must reload case files after adding keywords |
| Anomaly detection scope | Limited — e.g., EternalBlue, spoofing (not a full IDS) |
| v2+ advantage | MAC correlation, richer parameter capture |
| v1.6 advantage | Detailed packet/frame views, unified cleartext tab |

---

## Summary

- **NetworkMiner** is an open-source **Network Forensic Analysis Tool (NFAT)** that works as either a passive sniffer or, more usefully, a **PCAP parser** — handling live traffic and capture files (but not log files) to give analysts a quick view of hosts, OS fingerprints (via Satori and p0f), extracted files, credentials, and cleartext keywords.
- Its **sniffer mode** (Windows-only) isn't reliable enough to replace Wireshark or tcpdump, so **packet parsing** is the recommended use; the trade-offs are strong overview, extraction, and fingerprinting capabilities against poor performance on large PCAPs, limited filtering, and no built-in deep-investigation features.
- Compared with **Wireshark**, NetworkMiner wins on OS fingerprinting, keyword discovery, and host categorisation, while Wireshark wins on filtering, packet decoding, protocol, payload, and statistical analysis — hence the best practice of getting a **quick overview in NetworkMiner**, then going **deep in Wireshark**.
- The interface centers on a **Case Panel** of loaded PCAPs and tabs for **Hosts, Sessions, DNS, Credentials** (exportable to Hashcat or John the Ripper), **Files, Images, Parameters, Keywords** (case files must be reloaded after adding new keywords), **Messages**, and **Anomalies** (limited pattern flags like EternalBlue and spoofing — it is not a full IDS).
- **Version 2.7+** adds MAC address correlation and richer parameter capture, while **v1.6** retains detailed packet and frame views and a unified cleartext tab.

## Related Documents

- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
- [90. Network Perimeter Monitoring & Log Investigation](./90-network-perimeter-monitoring-log-investigation.md)
- [85. Network Traffic Analysis (NTA) Fundamentals](./85-network-traffic-analysis-nta-fundamentals.md)
- [47. John the Ripper — Hash Cracking in Practice](./47-john-the-ripper-hash-cracking-in-practice.md)
