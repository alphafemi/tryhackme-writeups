# Snort In-Depth — IDS/IPS Modes, Parameters & Rule Writing

## Overview

This document builds on Snort fundamentals with an in-depth look at IDS vs. IPS and their types and techniques, Snort's three primary modes, command-line parameters for sniffer, logger, IDS/IPS, and PCAP investigation modes, inline IPS operation, rule structure (actions, protocols, IP/port filtering, direction), rule option categories (general, payload, non-payload), Snort's main components, ruleset tiers, and the key sections of `snort.conf` including DAQ modules.

---

Snort is an open-source, rule-based **Network Intrusion Detection and Prevention System (NIDS/NIPS)**, developed and maintained by Martin Roesch, open-source contributors, and the Cisco Talos team. Building on Snort fundamentals (Section 60), this section covers its command-line modes, parameters, and rule-writing mechanics in depth.

---

### IDS vs. IPS

| System | Behavior |
|---|---|
| **IDS (Intrusion Detection System)** | Passive — detects malicious activity/patterns, abnormal incidents, policy violations and generates alerts |
| **IPS (Intrusion Prevention System)** | Active — detects AND stops/prevents/terminates the suspicious event as soon as it's identified |

#### IDS Types

| Type | Scope |
|---|---|
| **NIDS** | Monitors traffic across the whole subnet; generates an alert on signature match |
| **HIDS** | Monitors a single endpoint device; generates an alert on signature match |

#### IPS Types

| Type | Scope | Trigger |
|---|---|---|
| **NIPS** | Monitors the entire subnet | Terminates connection on signature match |
| **Behaviour-based (Network Behaviour Analysis / NBA)** | Monitors the entire subnet | Terminates connection on **anomaly** match; requires a training/baselining period to learn "normal" before flagging "abnormal" — more effective against new/unknown threats, but a breach during training corrupts the baseline |
| **WIPS (Wireless IPS)** | Wireless network traffic | Terminates connection on signature match |
| **HIPS (Host IPS)** | Single endpoint device | Terminates connection on signature match (same mechanism as HIDS, but acts rather than just alerting) |

#### Detection/Prevention Techniques

| Technique | Approach |
|---|---|
| **Signature-Based** | Rules matching known malicious patterns — detects known threats |
| **Behaviour-Based** | Compares known/normal vs. unknown/abnormal behavior — detects new/unknown threats |
| **Policy-Based** | Compares activity against system configuration and security policy — detects policy violations |

> **Summary:** IDS identifies threats but needs human assistance to stop them; IPS identifies **and** blocks threats with less manual intervention at detection time.

---

### Snort Capabilities & Modes

**Capabilities:** Live traffic analysis, attack/probe detection, packet logging, protocol analysis, real-time alerting, modules & plugins, preprocessors, cross-platform support (Linux & Windows).

**Three primary use modes:**

| Mode | Function |
|---|---|
| **Sniffer Mode** | Read and display IP packets in the console |
| **Packet Logger Mode** | Log all inbound/outbound IP packets |
| **NIDS/NIPS Mode** | Log/drop packets matching user-defined rules |

---

### First Interactions — Version & Configuration Checks

```bash
snort -V                                      # version check
sudo snort -c /etc/snort/snort.conf -T        # test configuration validity
```

| Parameter | Description |
|---|---|
| `-V` / `--version` | Instance version info |
| `-c` | Identify the configuration file |
| `-T` | Self-test parameter — validates setup |
| `-q` | Quiet mode — suppresses the default banner/startup info |

---

### Sniffer Mode Parameters

| Parameter | Description |
|---|---|
| `-v` | Verbose — display TCP/IP output in console |
| `-d` | Display packet data (payload) |
| `-e` | Display link-layer (Ethernet) headers |
| `-X` | Display full packet details in HEX |
| `-i` | Define a specific network interface to sniff |

**Usage examples:**

```bash
sudo snort -v -i eth0      # verbose mode on a named interface
sudo snort -v              # verbose mode, default interface
sudo snort -d              # payload dump mode
sudo snort -de             # payload + link-layer headers
sudo snort -X              # full hex packet dump
```

> Parameters can be combined: `snort -vd`, `snort -de`, `snort -v -d -e`, `snort -X`, etc.

---

### Packet Logger Mode Parameters

| Parameter | Description |
|---|---|
| `-l` | Logger mode; sets log/alert output directory (default: `/var/log/snort`) |
| `-K ASCII` | Log packets in human-readable ASCII format |
| `-r` | Reading option — review logged events |
| `-n` | Limit the number of packets processed/read |

#### Logfile Ownership

Since Snort requires root to sniff, log files are root-owned. Two approaches to investigate them as a regular user:

1. **Elevate privileges** (sudo) to read the files directly
2. **Change ownership**: `sudo chown username file` or `sudo chown username -R directory` (`-R` for recursive)

#### Binary vs. ASCII Logging

```bash
sudo snort -dev -l.              # binary (tcpdump format) logging — single all-in-one file
sudo snort -dev -K ASCII -l.     # ASCII logging — separate, human-readable files per IP/connection
```

ASCII mode creates folders named by IP address, each containing categorized connection files (e.g., `ICMP_ECHO`, `UDP:36648-53`) — readable directly in a text editor. Binary format requires Snort, tcpdump, or Wireshark to interpret.

#### Reading Logged Files

```bash
sudo snort -r snort.log.<timestamp>                       # read back a binary log
sudo tcpdump -r snort.log.<timestamp> -ntc 10              # alternative: read with tcpdump
```

Binary logs can also be filtered using **Berkeley Packet Filters (BPF)**:

```bash
sudo snort -r logname.log -X
sudo snort -r logname.log icmp
sudo snort -r logname.log tcp
sudo snort -r logname.log 'udp and port 53'
snort -dvr logname.log -n 10        # process only first 10 packets
```

> Note: logs created with `-K ASCII` **cannot** be re-read by Snort itself — only via a text editor.

---

### IDS/IPS Mode Parameters

| Parameter | Description |
|---|---|
| `-c` | Define the configuration file |
| `-T` | Test the configuration file |
| `-N` | Disable logging (console/other outputs still work if enabled) |
| `-D` | Background/daemon mode |
| `-A` | Alert mode — see table below |

#### Alert Modes (`-A`)

| Mode | Description |
|---|---|
| **full** (default) | All possible information about the alert |
| **fast** | Alert message, timestamp, source/destination IP and ports |
| **console** | Fast-style alerts displayed directly on the console |
| **cmg** | Basic header details plus payload in hex and text |
| **none** | Disables alerting entirely (still logs traffic in binary) |

**Example commands:**

```bash
sudo snort -c /etc/snort/snort.conf -N                    # disable logging
sudo snort -c /etc/snort/snort.conf -D                    # background/daemon mode
sudo snort -c /etc/snort/snort.conf -A console             # console alerts
sudo snort -c /etc/snort/snort.conf -A cmg                 # cmg alerts (full packet + hex/text)
sudo snort -c /etc/snort/snort.conf -A fast                 # fast-style alert file (no console output)
sudo snort -c /etc/snort/snort.conf -A full                 # full detail alert file (no console output)
sudo snort -c /etc/snort/snort.conf -A none                 # logs traffic, no alert file generated
```

**Managing a daemon instance:**

```bash
ps -ef | grep snort       # find the background Snort process
sudo kill -9 <PID>        # stop the daemon
```

> Daemon mode (`-D`) is primarily used to automate Snort via scripts — not recommended without a stable, well-tested configuration.

#### Running Without a Configuration File

Rules can be tested directly without a full `snort.conf`, at reduced performance:

```bash
sudo snort -c /etc/snort/rules/local.rules -A console
```

---

### IPS Mode — Actually Dropping Packets

IPS (inline) mode is activated via the DAQ module and requires **at least two network interfaces**:

```bash
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console
```

Alert output in this mode shows `[Drop]` prefixing matched packets — confirming Snort actively blocked them rather than just alerting.

---

### PCAP Investigation Mode

| Parameter | Description |
|---|---|
| `-r` / `--pcap-single=` | Read a single PCAP file |
| `--pcap-list=""` | Read multiple PCAPs (space-separated) |
| `--pcap-show` | Display the active PCAP's name on console during processing |

**Examples:**

```bash
snort -r icmp-test.pcap                                                   # basic read (statistics only)
sudo snort -c /etc/snort/snort.conf -q -r icmp-test.pcap -A console -n 10 # applying ruleset to a single PCAP

sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console
sudo snort -c /etc/snort/snort.conf -q --pcap-list="icmp-test.pcap http2.pcap" -A console --pcap-show
```

> `--pcap-show` is essential when processing multiple PCAPs together — without it, alerts from different files are indistinguishable in the combined output.

---

### Snort Rule Structure

Every rule needs: **action, protocol, source/destination IP, source/destination port**, plus optional **rule options** in parentheses.

#### Actions

| Action | Effect |
|---|---|
| `alert` | Generate an alert and log the packet |
| `log` | Log the packet only |
| `drop` | Block and log the packet |
| `reject` | Block the packet, log it, and terminate the session |

#### Protocol

Snort 2 supports only **IP, TCP, UDP, ICMP** as protocol filters — application-layer protocols (e.g., FTP) must be detected indirectly via port + content matching (e.g., TCP port 21 for FTP).

#### IP Filtering Examples

```
alert icmp 192.168.1.56 any <> any any (msg:"ICMP Packet From"; sid:100001; rev:1;)
alert icmp 192.168.1.0/24 any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)
alert icmp [192.168.1.0/24,10.1.1.0/24] any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)
alert icmp !192.168.1.0/24 any <> any any (msg:"ICMP Packet Found"; sid:100001; rev:1;)  # negation operator
```

#### Port Filtering Examples

```
alert tcp any any <> any 21 (msg:"FTP Port 21 Activity"; sid:100001; rev:1;)
alert tcp any any <> any !21 (msg:"Non-FTP-21 Traffic"; sid:100001; rev:1;)
alert tcp any any <> any 1:1024 (msg:"Ports 1-1024"; sid:100001; rev:1;)
alert tcp any any <> any :1024 (msg:"Ports 0-1024"; sid:100001; rev:1;)
alert tcp any any <> any 1025: (msg:"Non-System Ports"; sid:100001; rev:1;)
alert tcp any any <> any [21,23] (msg:"FTP and Telnet Activity"; sid:100001; rev:1;)
```

#### Direction Operators

| Operator | Meaning |
|---|---|
| `->` | Source → Destination (one-way) |
| `<>` | Bidirectional |
| *(no `<-`)* | Snort has no reverse-only operator |

---

### Rule Option Categories

| Category | Purpose |
|---|---|
| **General** | Fundamental metadata for the rule |
| **Payload Detection** | Investigate the actual payload content/patterns |
| **Non-Payload Detection** | Focus on non-payload fields (flags, size, IDs, etc.) |

#### General Rule Options

| Option | Description |
|---|---|
| `msg` | One-line message shown on trigger |
| `sid` | Unique rule ID. `<100` = reserved; `100–999,999` = built-in; `≥1,000,000` = user-created. SIDs must never overlap |
| `reference` | Points to a CVE or external data explaining the threat |
| `rev` | Revision number — tracks how many times the rule was edited (no auto-history, analysts must track manually) |

```
alert icmp any any <> any any (msg:"ICMP Packet Found"; sid:100001; reference:cve,CVE-XXXX; rev:1;)
```

#### Payload Detection Options

| Option | Description |
|---|---|
| `content` | Matches payload data (ASCII and/or HEX); case-sensitive; can be used multiple times (at a performance cost) |
| `nocase` | Disables case sensitivity for a content match |
| `fast_pattern` | Prioritizes which `content` match is used for the initial packet filter — required when multiple `content` options are present; always case-insensitive; usable once per rule |

```
alert tcp any any <> any 80 (msg:"GET Request Found"; content:"GET"; sid:100001; rev:1;)                    # ASCII
alert tcp any any <> any 80 (msg:"GET Request Found"; content:"|47 45 54|"; sid:100001; rev:1;)              # HEX
alert tcp any any <> any 80 (msg:"GET Request Found"; content:"GET"; nocase; sid:100001; rev:1;)
alert tcp any any <> any 80 (msg:"GET Request Found"; content:"GET"; fast_pattern; content:"www"; sid:100001; rev:1;)
```

#### Non-Payload Detection Options

| Option | Description | Example |
|---|---|---|
| `id` | Filters the IP ID field | `id:123456;` |
| `flags` | Filters TCP flags (F=FIN, S=SYN, R=RST, P=PSH, A=ACK, U=URG) | `flags:S;` |
| `dsize` | Filters packet payload size | `dsize:100<>300;`, `dsize:>100`, `dsize:<100` |
| `sameip` | Flags packets where source and destination IP are identical | `sameip;` |

```
alert tcp any any <> any any (msg:"FLAG TEST"; flags:S; sid:100001; rev:1;)
alert ip any any <> any any (msg:"SEQ TEST"; dsize:100<>300; sid:100001; rev:1;)
alert ip any any <> any any (msg:"SAME-IP TEST"; sameip; sid:100001; rev:1;)
```

> User-created rules belong in `/etc/snort/rules/local.rules`, edited directly (e.g., `sudo gedit /etc/snort/rules/local.rules`).

---

### Snort's Main Components

| Component | Function |
|---|---|
| **Packet Decoder** | Collects and prepares packets for pre-processing |
| **Pre-processors** | Arrange/modify packets ahead of the detection engine |
| **Detection Engine** | Processes, dissects, and analyzes packets by applying rules |
| **Logging and Alerting** | Generates logs and alerts |
| **Outputs and Plugins** | Integration modules (syslog, MySQL, etc.) and additional plugin support |

---

### Snort Ruleset Tiers

| Tier | Access |
|---|---|
| **Community Rules** | Free, GPLv2, publicly accessible, no registration needed |
| **Registered Rules** | Free with registration; subscriber rules delayed 30 days |
| **Subscriber Rules (Paid)** | Paid, main ruleset, updated twice weekly (Tuesdays/Thursdays) |

---

### snort.conf — Key Configuration Sections

| Step | Section | Tag/Setting | Purpose |
|---|---|---|---|
| **#1** | Network variables | `HOME_NET` | The network being protected (e.g., `192.168.1.1/24` or `any`) |
| | | `EXTERNAL_NET` | External network (`any` or `!$HOME_NET`) |
| | | `RULE_PATH` | Hardcoded rule path (`/etc/snort/rules`) |
| | | `SO_RULE_PATH` / `PREPROC_RULE_PATH` | Paths for registered/subscriber rule add-ons |
| **#2** | Decoder/DAQ config | `#config daq` | Selects IPS mode (e.g., `afpacket`) |
| | | `#config daq_mode` | Activates inline mode |
| | | `#config logdir` | Hardcoded default log path |
| **#6** | Output plugins | — | Manages logging/alerting output formats |
| **#7** | Ruleset customization | `include $RULE_PATH/local.rules` | Activates local/user-generated rules |
| | | `#include $RULE_PATH/<rulename>` | Activates a specific downloaded/default rule file |

> `#` is Snort's comment operator — a line must be uncommented to take effect.

#### DAQ (Data Acquisition) Modules

| Module | Purpose |
|---|---|
| **PCAP** | Default sniffer mode |
| **Afpacket** | Inline/IPS mode (most common for single-node IPS) |
| **Ipq** | Inline on Linux via Netfilter (replaces the old snort_inline patch) |
| **Nfq** | Inline on Linux |
| **Ipfw** | Inline on OpenBSD/FreeBSD via divert sockets (pf/ipfw) |
| **Dump** | Testing mode for inline/normalization |

> Never directly overwrite a working `snort.conf` — edit manually and carefully, or use dedicated rule-update tools/modules to avoid crashes or missing features.

---

### Snort In-Depth Quick Reference

| Concept | Summary |
|---|---|
| IDS | Passive detection + alert |
| IPS | Active detection + block/terminate |
| 3 primary modes | Sniffer, Packet Logger, NIDS/NIPS |
| Sniffer flags | `-v`, `-d`, `-e`, `-X`, `-i` |
| Logger flags | `-l`, `-K ASCII`, `-r`, `-n` |
| IDS/IPS flags | `-c`, `-T`, `-N`, `-D`, `-A` |
| Alert modes | full, fast, console, cmg, none |
| IPS activation | `-Q --daq afpacket` + 2 interfaces |
| PCAP flags | `-r`, `--pcap-list`, `--pcap-show` |
| Rule actions | alert, log, drop, reject |
| Supported protocols | IP, TCP, UDP, ICMP only |
| Direction operators | `->` (one-way), `<>` (bidirectional) |
| General options | msg, sid, reference, rev |
| Payload options | content, nocase, fast_pattern |
| Non-payload options | id, flags, dsize, sameip |
| SID ranges | <100 reserved, 100–999,999 built-in, ≥1,000,000 user |
| 5 main components | Packet Decoder, Pre-processors, Detection Engine, Logging/Alerting, Outputs/Plugins |
| 3 ruleset tiers | Community (free), Registered (free, delayed), Subscriber (paid) |
| Key config file | `snort.conf` — HOME_NET, EXTERNAL_NET, RULE_PATH, DAQ settings |

---

## Summary

- An **IDS** passively detects threats and raises alerts (needing human help to stop them), while an **IPS** actively blocks or terminates them; IDS types are **NIDS** and **HIDS**, IPS types are **NIPS**, **behaviour-based (NBA)**, **WIPS**, and **HIPS**, and detection techniques are **signature-based**, **behaviour-based** (better against unknown threats, but a breach during its baseline-training period corrupts the baseline), and **policy-based**.
- Snort runs in three modes — **Sniffer** (`-v`, `-d`, `-e`, `-X`, `-i`), **Packet Logger** (`-l`, `-K ASCII`, `-r`, `-n`; binary logs are read back with Snort, tcpdump, or Wireshark and can be filtered with BPF, while ASCII logs can't be re-read by Snort), and **NIDS/NIPS** (`-c`, `-T`, `-N`, `-D`, `-A` with alert modes full, fast, console, cmg, and none).
- **IPS (inline) mode** requires the DAQ module and at least two interfaces (`-Q --daq afpacket -i eth0:eth1`), with `[Drop]` in the output confirming packets are being blocked; **PCAP investigation** uses `-r`, `--pcap-list`, and `--pcap-show` (essential to tell alerts apart when processing multiple files).
- A rule needs an **action** (alert, log, drop, reject), **protocol** (Snort 2 supports only IP, TCP, UDP, ICMP), **IPs and ports** (with ranges, lists, and `!` negation), and a **direction** (`->` or `<>`), plus options: **general** (`msg`, `sid` — user-created at 1,000,000 or above — `reference`, `rev`), **payload** (`content`, `nocase`, `fast_pattern`), and **non-payload** (`id`, `flags`, `dsize`, `sameip`), with custom rules kept in `/etc/snort/rules/local.rules`.
- Snort's five components are the **Packet Decoder, Pre-processors, Detection Engine, Logging/Alerting,** and **Outputs/Plugins**; rules come in **Community**, **Registered**, and **Subscriber** tiers; and `snort.conf` controls network variables (`HOME_NET`, `EXTERNAL_NET`, `RULE_PATH`), DAQ settings (PCAP, Afpacket, Ipq, Nfq, Ipfw, Dump), output plugins, and rule inclusion — and a working config should never be overwritten directly.

