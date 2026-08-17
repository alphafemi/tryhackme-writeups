# Intrusion Detection Systems (IDS) & Snort

## Overview

This document covers Intrusion Detection Systems (IDS): the difference between HIDS and NIDS deployment modes, signature-based vs. anomaly-based vs. hybrid detection, and Snort — one of the most widely used open-source IDS tools — including its modes of operation, directory layout, rule syntax, and how to build, test, and run custom detection rules.

---

A firewall guards the network's boundary, but if an attacker slips past it through a legitimate-looking connection, something needs to catch what happens *after* entry. That's the role of an **Intrusion Detection System (IDS)** — like surveillance cameras inside a building complementing the gatekeeper at the door. An IDS monitors network traffic for signature- and anomaly-based indicators of malicious activity and **alerts** administrators. Critically, an IDS only **detects and notifies** — it does not act on what it finds (that's the job of an IPS/response system).

---

### Deployment Modes

| Mode | Scope | Trade-off |
|---|---|---|
| **HIDS** (Host Intrusion Detection System) | Installed on an individual host; detects threats specific to that host with detailed visibility | Resource-intensive and harder to manage across large networks (one instance per host) |
| **NIDS** (Network Intrusion Detection System) | Monitors traffic across the whole network, regardless of host | Centralized view of all detections network-wide |

### Detection Modes

| Mode | How It Works | Strength | Weakness |
|---|---|---|---|
| **Signature-Based** | Matches traffic against a database of known attack patterns ("signatures") | Fast, reliable detection of known threats | Cannot detect **zero-day attacks** (no existing signature) |
| **Anomaly-Based** | Learns a baseline of normal behavior, then flags deviations | Can catch zero-day attacks | Prone to **false positives**, since legitimate-but-unusual activity can resemble malicious behavior; tunable to reduce this |
| **Hybrid** | Combines both — signature matching for known threats, anomaly detection for new ones | Leverages the strengths of both approaches | — |

> **Trade-off summary:** Signature-based IDS suits a small, well-understood threat surface with low overhead. Anomaly-based and hybrid IDS are better suited to catching the rising volume of modern zero-day attacks, at the cost of more false positives / processing overhead.

---

### Snort

**Snort** is one of the most widely used open-source IDS solutions, first developed in 1998. It uses both signature-based and anomaly-based detection, driven by rule files that can be extended with custom rules or have built-in rules disabled as needed.

#### Modes of Snort

| Mode | Description | Use Case |
|---|---|---|
| **Packet Sniffer Mode** | Reads and displays packets with no analysis | Diagnosing network performance issues by observing raw traffic flow |
| **Packet Logging Mode** | Logs traffic to a **PCAP** file for later analysis | Forensic investigators performing root cause analysis after an attack |
| **NIDS Mode** | Real-time monitoring against rule files, generating alerts on matches | Primary IDS functionality — proactive threat monitoring |

> **Promiscuous mode:** By default Snort only captures traffic addressed to its own host. To monitor and detect intrusions across the whole network, the host's network interface must be switched into **promiscuous mode**.

#### Snort Directory Layout

Snort's configuration, rules, and reference files typically live in `/etc/snort` (location can vary — e.g., `/usr/local/etc/snort` for source builds). The main config file is **`snort.lua`**, loaded via the `-c` flag; custom/enabled rule files live in the `rules` subfolder.

```bash
ls /etc/snort
# Intro_to_IDS.pcap  community-sid-msg.map  reference.config  snort.conf  snort.lua  unicode.map
# classification.config  gen-msg.map  rules  snort.debian.conf  threshold.conf
```

---

### Snort Rule Format

A Snort rule follows a fixed structure. Example — detect any ICMP (ping) traffic reaching the home network:

```
alert icmp any any -> $HOME_NET any (msg:"Ping Detected"; sid:10001; rev:1;)
```

| Component | Meaning | Example Value |
|---|---|---|
| **Action** | What to do when the rule matches | `alert` |
| **Protocol** | Protocol to match | `icmp` |
| **Source IP** | Origin address to match | `any` |
| **Source port** | Origin port to match | `any` |
| **Destination IP** | Target address to match | `$HOME_NET` (variable set in Snort's config to the monitored network range) |
| **Destination port** | Target port to match | `any` |
| **msg** | Alert message shown on trigger | `"Ping Detected"` |
| **sid** | Unique Signature ID for the rule | `10001` |
| **rev** | Rule revision number, incremented on edits | `1` |

---

### Creating & Testing a Custom Rule

1. **Edit the custom rules file:**
   ```bash
   sudo nano /etc/snort/rules/local.rules
   ```
2. **Append a new rule** (detect pings to loopback), keeping existing rules intact:
   ```
   alert icmp any any -> 127.0.0.1 any (msg:"Loopback Ping Detected"; sid:10003; rev:1;)
   ```
3. Save with `Ctrl+X`, then `Y` to confirm.
4. **Run Snort in NIDS mode** to load the rule set and start detecting:
   ```bash
   sudo snort -q -l /var/log/snort -i lo -A alert_fast -c /etc/snort/snort.lua
   ```
   *(Replace `lo` with the correct interface name if your loopback interface differs.)*
5. **Trigger the rule:**
   ```bash
   ping 127.0.0.1
   ```
6. **Resulting alert output:**
   ```
   07/24-10:46:52.401504 [**] [1:1000001:1] "Loopback Ping Detected" [**] [Priority: 0] {ICMP} 127.0.0.1 -> 127.0.0.1
   07/24-10:46:53.406552 [**] [1:1000001:1] "Loopback Ping Detected" [**] [Priority: 0] {ICMP} 127.0.0.1 -> 127.0.0.1
   ```

---

### Running Snort Against PCAP Files

For forensic analysis of historical traffic already captured in a PCAP file, Snort can process it directly rather than monitoring live traffic:

```bash
sudo snort -q -l /var/log/snort -r Task.pcap -A alert_fast -c /etc/snort/snort.lua
```

*(Replace `Task.pcap` with the path to the target PCAP file.)*

---

### IDS Quick Reference

| Concept | Summary |
|---|---|
| IDS role | Detect and alert — does **not** take action |
| Deployment modes | HIDS (per-host) vs. NIDS (network-wide) |
| Detection modes | Signature-based, Anomaly-based, Hybrid |
| Weakness of signature-based | Misses zero-days |
| Weakness of anomaly-based | Higher false positive rate |
| Tool | Snort (open-source, since 1998) |
| Snort modes | Packet Sniffer, Packet Logging, NIDS |
| Snort config file | `snort.lua`, loaded via `-c` |
| Custom rule file | `rules/local.rules` |
| Rule syntax | `action proto src_ip src_port -> dst_ip dst_port (msg; sid; rev;)` |
| Live capture flag | `-i <interface>` |
| PCAP analysis flag | `-r <file.pcap>` |

---

## Summary

- An **IDS** monitors network or host traffic for malicious indicators and raises alerts — it detects and notifies but does not take action, unlike an IPS.
- Deployment splits into **HIDS** (per-host, detailed but resource-intensive) and **NIDS** (network-wide, centralized).
- Detection relies on **signature-based** matching (fast, reliable, but blind to zero-days), **anomaly-based** baselining (catches novel threats, but prone to false positives), or a **hybrid** of both.
- **Snort**, an open-source IDS since 1998, runs in three modes — **packet sniffer**, **packet logging**, and **NIDS** — and requires the capture interface in **promiscuous mode** to see network-wide traffic.
- Snort rules follow a fixed structure — action, protocol, source/destination IP and port, then a metadata block with `msg`, `sid`, and `rev` — and can be created in `rules/local.rules`, then run live (`-i`) or against a saved capture (`-r`).

