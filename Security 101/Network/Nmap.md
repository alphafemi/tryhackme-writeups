 Nmap — Network Scanning & Enumeration

**Nmap** (Network Mapper) is the industry-standard open-source tool for network discovery and security auditing — used to identify live hosts, open ports, running services, OS versions, and more.

> Most Nmap scan types require **root/sudo** privileges. Running as a non-root user limits you to basic ICMP echo and TCP connect scans.

---

### Specifying Targets

| Format | Example | Meaning |
|---|---|---|
| Single IP | `192.168.0.1` | One specific host |
| IP range | `192.168.0.1-10` | All IPs from `.1` to `.10` |
| CIDR subnet | `192.168.0.0/24` | All 256 addresses in the subnet |
| Hostname | `example.thm` | Resolved via DNS |

---

### Host Discovery — Who is Online?

#### `-sn` — Ping Scan (Host Discovery Only)

Discovers live hosts **without** scanning ports — lower noise, faster.

```bash
nmap -sn 192.168.66.0/24
```

```
Nmap scan report for XiaoQiang (192.168.66.1)
Host is up (0.0069s latency).
MAC Address: 44:DF:65:D8:FE:6C (Unknown)

Nmap scan report for 192.168.66.88
Host is up (0.090s latency).
MAC Address: 7C:DF:A1:D3:8C:5C (Espressif)
...
7 hosts up in 2.64 seconds
```

**Local network:** Nmap sends **ARP requests** — MAC addresses and vendor info are visible.

**Remote network (through a router):** Nmap uses ICMP echo, ICMP timestamp, TCP SYN (port 443), and TCP ACK (port 80) to probe targets.

#### `-sL` — List Scan (No Actual Scanning)

Lists all targets Nmap would scan — useful for verifying scope before running:

```bash
nmap -sL 192.168.0.1/24    # lists 256 targets without scanning
```

---

### Port Scanning

#### TCP Connect Scan — `-sT`

Completes the full **TCP three-way handshake**. Works without root. Generates more logs — the full connection is established and then torn down.

```
Open port:   SYN → SYN-ACK → ACK → (data exchange) → RST-ACK (Nmap closes)
Closed port: SYN → RST-ACK (target rejects immediately)
```

#### SYN Scan (Stealth) — `-sS`

Sends only the **SYN** packet — never completes the handshake. Requires root. Generates fewer logs since no full connection is established — considered relatively stealthy.

```
Open port:   SYN → SYN-ACK → RST (Nmap resets instead of completing)
Closed port: SYN → RST-ACK (same as connect scan)
```

#### UDP Scan — `-sU`

Scans for **UDP services** — DNS, DHCP, NTP, SNMP, VoIP, etc. UDP ports don't respond to probes the way TCP does — closed UDP ports typically return ICMP "port unreachable."

```bash
sudo nmap -sU 192.168.1.1
```

#### Controlling Which Ports Are Scanned

| Option | Behavior |
|---|---|
| *(default)* | Scans the **1,000 most common** ports |
| `-F` | **Fast** — scans the **100 most common** ports |
| `-p 22,80,443` | Scan **specific ports** only |
| `-p 10-1024` | Scan a **port range** |
| `-p-` | Scan **all 65,535 ports** (equivalent to `-p1-65535`) |
| `-p1-1023` | Scan all **well-known ports** |

---

### Scan Type Summary

| Option | Scan Type | Notes |
|---|---|---|
| `-sT` | TCP connect | Full handshake; works without root; more logs |
| `-sS` | TCP SYN (stealth) | Half-open; requires root; fewer logs |
| `-sU` | UDP | Requires root; slower |
| `-sn` | Ping/host discovery | No port scan |
| `-sL` | List scan | No actual scanning |

---

### Detection Options

#### `-O` — OS Detection

Uses various indicators (TTL, TCP window size, etc.) to make an educated guess about the target's operating system:

```bash
sudo nmap -sS -O 192.168.124.211
```

```
Running: Linux 4.X|5.X
OS details: Linux 4.15 - 5.8
```

> OS detection is probabilistic — it may be inaccurate. Always interpret results with appropriate skepticism.

#### `-sV` — Service and Version Detection

Identifies what services are running on open ports and their versions:

```bash
sudo nmap -sS -sV 192.168.124.211
```

```
PORT    STATE  SERVICE  VERSION
22/tcp  open   ssh      OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux)
```

#### `-A` — Aggressive Scan

Combines OS detection (`-O`), version detection (`-sV`), script scanning, and traceroute in a single flag:

```bash
sudo nmap -A 192.168.1.1
```

#### `-Pn` — Skip Host Discovery

Treats all targets as online, even if they don't respond to ping probes — scans ports regardless:

```bash
sudo nmap -sS -Pn 192.168.1.1
```

> Useful when a host has ICMP blocked by a firewall but still has open ports.

---

### Detection Options Summary

| Option | Function |
|---|---|
| `-O` | OS detection |
| `-sV` | Service and version detection |
| `-A` | OS + version + scripts + traceroute |
| `-Pn` | Treat all hosts as up (skip host discovery) |

---

### Timing & Performance

Nmap offers **six timing templates** balancing speed against stealth and reliability:

| Template | Name | Speed | Notes |
|---|---|---|---|
| `-T0` | Paranoid | Extremely slow (hours) | Maximum stealth — 5 min between probes |
| `-T1` | Sneaky | Very slow (minutes) | ~15 sec between probes |
| `-T2` | Polite | Slow (~40 sec) | ~0.4 sec between probes |
| `-T3` | Normal | Fast (default) | No intentional delay |
| `-T4` | Aggressive | Very fast | Assumes fast/reliable network |
| `-T5` | Insane | Fastest | May sacrifice accuracy |

```bash
nmap -sS -T4 192.168.1.0/24    # aggressive timing
nmap -sS -T1 192.168.1.0/24    # sneaky timing
```

#### Additional Timing/Rate Options

| Option | Purpose |
|---|---|
| `--min-parallelism <n>` | Minimum number of parallel probes |
| `--max-parallelism <n>` | Maximum number of parallel probes |
| `--min-rate <n>` | Minimum packets per second |
| `--max-rate <n>` | Maximum packets per second |
| `--host-timeout <time>` | Maximum time to wait per host |

---

### Verbosity & Output

#### Verbosity

| Option | Output Level |
|---|---|
| *(default)* | Summary only after scan completes |
| `-v` | Real-time progress updates |
| `-vv` / `-vvv` | Increased verbosity |
| `-v2`, `-v4` | Specify verbosity level directly |
| `-d` | Debug-level output |
| `-d9` | Maximum debugging (very verbose) |

> You can press `v` during a running scan to increase verbosity without restarting.

#### Output Formats

| Option | Format | Use For |
|---|---|---|
| `-oN <file>` | Normal (human-readable) | General reporting |
| `-oX <file>` | XML | Tool integration, parsing |
| `-oG <file>` | Greppable | `grep`/`awk` post-processing |
| `-oA <basename>` | All three formats simultaneously | Comprehensive archiving |

```bash
# Save results in all three formats
sudo nmap -sS 192.168.139.1 -oA gateway
# Creates: gateway.nmap, gateway.xml, gateway.gnmap
```

---

### Practical Examples

```bash
# Discover live hosts on a /24 subnet
sudo nmap -sn 192.168.1.0/24

# Fast SYN scan of 100 common ports
sudo nmap -sS -F 192.168.1.0/24

# Full SYN scan with version and OS detection
sudo nmap -sS -sV -O 192.168.1.100

# Aggressive scan with all additions
sudo nmap -A 192.168.1.100

# Scan all 65535 ports, verbose, save all formats
sudo nmap -sS -p- -v 192.168.1.100 -oA full_scan

# Scan specific ports
sudo nmap -sS -p 22,80,443,8080 192.168.1.100

# Treat host as online even if ping fails
sudo nmap -sS -Pn 192.168.1.100

# Stealthy slow scan to avoid IDS detection
sudo nmap -sS -T1 192.168.1.0/24
```

---

### Quick Reference

| Task | Command |
|---|---|
| Host discovery only | `nmap -sn <target>` |
| List targets without scanning | `nmap -sL <target>` |
| TCP connect scan | `nmap -sT <target>` |
| SYN (stealth) scan | `sudo nmap -sS <target>` |
| UDP scan | `sudo nmap -sU <target>` |
| Fast scan (100 ports) | `nmap -F <target>` |
| Scan all ports | `nmap -p- <target>` |
| OS detection | `sudo nmap -O <target>` |
| Version detection | `nmap -sV <target>` |
| Aggressive (all-in-one) | `sudo nmap -A <target>` |
| Skip ping (treat as up) | `nmap -Pn <target>` |
| Timing template | `nmap -T<0-5> <target>` |
| Verbose output | `nmap -v <target>` |
| Save normal output | `nmap -oN file.nmap <target>` |
| Save all formats | `nmap -oA basename <target>` |
