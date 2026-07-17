Active Reconnaissance Techniques

### Passive vs. Active Reconnaissance

| | Passive Reconnaissance | Active Reconnaissance |
|---|---|---|
| **Definition** | Collecting data from public sources without sending traffic to the target | Directly interacting with the target — connecting, probing, transmitting |
| **Sources** | DNS records, WHOIS, certificate logs, Shodan | Web browsing, ping, traceroute, telnet, netcat |
| **Detectability** | Leaves no trace on the target | Leaves log entries, IDS alerts, WAF blocks |
| **Risk** | Minimal | Detectable; may trigger defenses |

> ⚠️ **Legal Warning:** Never perform active reconnaissance without **explicit, signed legal authorisation** (penetration testing contract or documented bug bounty scope). Unauthorized probing is illegal in most jurisdictions.

---

### Detection Context

Modern environments are significantly harder to probe silently than in previous years:

- **CDNs** (Cloudflare, Akamai) and **WAFs** log unusual probes and may block them outright
- **Zero-trust architectures** and **EDR/SIEM** solutions detect reconnaissance patterns quickly
- **IPv6** adoption means hosts may respond to `ping6` but filter ICMPv4
- **HTTPS** dominates — plaintext protocols are largely obsolete for production web traffic

> From a red team perspective: blend in by mimicking legitimate user behavior. From a blue team perspective: active probes surface in access logs, firewall logs, WAF events, and IDS alerts.

---

### 1. Web Browser as a Reconnaissance Tool

A browser is the **least suspicious** active reconnaissance tool — its traffic blends with normal user activity.

#### Transport Basics

| Protocol | Port | Notes |
|---|---|---|
| HTTP | TCP 80 | Nearly all sites redirect to HTTPS |
| HTTPS | TCP 443 | Standard for virtually all modern sites |
| HTTP/3 (QUIC) | UDP 443 | Faster; visible as `h3` in the Network tab |

Access non-standard ports directly in the URL:
```
https://target.com:8443/
http://192.168.1.100:8080/
```

#### Developer Tools (`Ctrl+Shift+I` / `Option+Cmd+I`)

| Tab | Reconnaissance Value |
|---|---|
| **Network** | Request/response headers (`Server`, `X-Powered-By`, `Content-Security-Policy`), status codes, cookies, timing |
| **Console** | Execute JavaScript, view errors, interact with the DOM |
| **Sources** | Browse JS/CSS/HTML files — frequently contain hardcoded API endpoints, internal paths, developer comments |
| **Application** | Inspect cookies, Local Storage, Session Storage — may expose session tokens or API keys |
| **Security** | Certificate details: issuer, validity, **Subject Alternative Names (SANs)** revealing additional subdomains |

> **Sources tab is particularly valuable:** JS source files often expose backend routes, internal service references, and developer comments never intended for public viewing.

#### Useful Browser Extensions

| Extension | Purpose |
|---|---|
| **FoxyProxy** | Switch between proxies (Burp Suite, ZAP, SOCKS5) |
| **User-Agent Switcher** | Emulate different browsers/OS/devices to discover mobile endpoints |
| **Wappalyzer** | Passively fingerprints CMS, web server, JS frameworks, CDNs, databases |

---

### 2. ping — Host Reachability Testing

`ping` uses ICMP to test whether a target is reachable and measures round-trip time.

- Sends: **ICMP Echo Request (Type 8)**
- Receives: **ICMP Echo Reply (Type 0)**

#### Basic Usage

```bash
# Linux/macOS
ping -c 5 MACHINE_IP       # 5 packets
ping -4 -c 5 MACHINE_IP    # force IPv4
ping -6 -c 5 MACHINE_IP    # force IPv6

# Windows
ping -n 5 MACHINE_IP
```

#### Interpreting Output

**Successful ping:**
```
64 bytes from MACHINE_IP: icmp_seq=1 ttl=64 time=0.512 ms
...
5 packets transmitted, 5 received, 0% packet loss
rtt min/avg/max/mdev = 0.478/0.494/0.512/0.012 ms
```

**No reply:**
```
From ATTACKBOX_IP icmp_seq=1 Destination Host Unreachable
5 packets transmitted, 0 received, +5 errors, 100% packet loss
```

#### TTL-Based OS Fingerprinting

The **TTL (Time To Live)** field decrements by 1 at each router hop — but the **starting value** is OS-specific:

| Starting TTL | Likely OS |
|---|---|
| 64 | Linux / macOS |
| 128 | Windows |
| 255 | Cisco network devices |

> A TTL of 58 suggests a Linux host that is **6 hops away** (64 − 6 = 58), not a different OS. Account for intermediate hops when interpreting.

#### Ping Result Interpretation

| Result | Likely Meaning | Next Step |
|---|---|---|
| Fast replies, low/no packet loss | Target is online and ICMP-responsive | Proceed to port scanning |
| "Destination Host Unreachable" | No route or host is down | Check if machine is powered on |
| 100% loss, no error message | ICMP is filtered/blocked | Try TCP/UDP discovery with Nmap |
| High latency or intermittent loss | Network congestion or long distance | Investigate path with traceroute |

**Common reasons for no ping response:** host is off; firewall blocks ICMP; NAT drops ICMP; Windows Firewall (default); AWS/Azure/GCP/CDN infrastructure often blocks ICMP.

---

### 3. traceroute / tracert — Network Path Mapping

Maps the route packets take from your system to a destination by exploiting the **TTL field**.

**How it works:** Sends packets with incrementally increasing TTL (1, 2, 3 …). Each router that drops a packet due to TTL=0 sends back an **ICMP Time-to-Live Exceeded (Type 11)** message — revealing its IP address.

| OS | Command |
|---|---|
| Linux/macOS | `traceroute MACHINE_IP` |
| Windows | `tracert MACHINE_IP` |
| IPv6 | `traceroute -6 MACHINE_IPV6` or `traceroute6` |
| TCP mode (bypass UDP filters) | `traceroute -T MACHINE_IP` |
| ICMP mode | `traceroute -I MACHINE_IP` |

#### Reading the Output

```
1  gateway (192.168.66.1)        4.4 ms    ← local router
2  100.66.8.86                  43.2 ms    ← ISP hop
3  * 100.66.16.176 *             8.0 ms    ← 2 of 3 probes got no reply
...
14 172.67.69.208                17.9 ms    ← destination reached
```

| Output | Meaning |
|---|---|
| IP address + RTT | Router responded — hop identified |
| `* * *` | Router suppressed ICMP Time-Exceeded — firewall or rate limiting |
| Multiple IPs per hop | Load balancing — each probe took a slightly different path |

> Routes are **not fixed** — dynamic routing (BGP/OSPF), load balancing, and CDN anycast mean the path may differ between consecutive runs. The number of hops can also vary significantly between runs.

#### `mtr` — Real-Time Combined Tool

```bash
mtr MACHINE_IP
```

Combines traceroute with continuous ping-like statistics — shows **per-hop packet loss and latency** in real time.

---

### 4. Telnet — Legacy Banner Grabbing

Telnet connects to any TCP port and displays what the server sends — useful for **banner grabbing** (reading the initial server response to identify software and version).

```bash
telnet MACHINE_IP PORT
```

**⚠️ Security note:** Telnet transmits everything in **plain text** including credentials — use SSH for actual remote administration. For modern TLS-encrypted services (HTTPS, SMTPS), use `curl` or `openssl s_client` instead.

#### HTTP Banner Grab via Telnet

```bash
telnet MACHINE_IP 80
GET / HTTP/1.1
host: telnet
                    ← blank line required
```

```
HTTP/1.1 200 OK
Server: nginx/1.6.2         ← software and version revealed
Date: ...
```

#### Other Services

| Service | Default Port | Banner Behavior |
|---|---|---|
| FTP | 21 | Banner sent immediately on connection — no commands needed |
| SMTP | 25 | Banner sent on connection |
| HTTP | 80 | Requires `GET / HTTP/1.1` + `host:` + blank line |

**For TLS-encrypted ports:** `curl --head https://MACHINE_IP` or `openssl s_client -connect MACHINE_IP:443`

---

### 5. Netcat (nc) — Versatile Port Tool

`nc` provides the same banner-grabbing capability as telnet plus the ability to **listen** on ports and act as a simple server. Generally preferred over telnet for active reconnaissance.

#### Banner Grabbing (Client Mode)

```bash
nc MACHINE_IP PORT
```

```bash
nc MACHINE_IP 80
GET / HTTP/1.1
host: netcat
                    ← blank line
```

```
HTTP/1.1 200 OK
Server: nginx/1.6.2
...
```

#### Listening (Server Mode)

```bash
# Start listening on port 1234
nc -lvnp 1234

# Connect from another machine
nc MACHINE_IP 1234
```

Once connected, text typed on one side is transmitted to the other.

#### Netcat Options

| Option | Meaning |
|---|---|
| `-l` | Listen mode |
| `-p PORT` | Specify port (must immediately precede the port number) |
| `-n` | Numeric only — no DNS resolution |
| `-v` | Verbose |
| `-vv` | Very verbose |
| `-k` | Keep listening after client disconnects |
| `-6` | IPv6 mode |

> Port numbers below 1024 require **root/sudo** to listen on. For encrypted transfer, use `ncat --ssl` (from the Nmap project) or pair with `stunnel`.

---

### Tool Comparison

| Tool | Purpose | Best For |
|---|---|---|
| **Browser + DevTools** | Web tech fingerprinting, header inspection, source code review | Web recon — certificates, JS sources, hidden endpoints |
| **ping** | Host reachability, RTT, OS fingerprinting via TTL | Quick liveness check |
| **traceroute / mtr** | Network path mapping, hop identification | Understanding routing, finding firewalls/filters |
| **telnet** | Legacy banner grabbing on TCP ports | Understanding protocol basics; legacy environments |
| **nc** | Banner grabbing, port probing, simple listeners | Versatile TCP/UDP probing and connectivity testing |
| **Nmap** | Automated discovery, port scanning, version/OS detection | Comprehensive enumeration — extends all of the above |

---

### Complete Quick Reference

| Command | Example |
|---|---|
| ping (Linux/macOS) | `ping -c 10 MACHINE_IP` |
| ping (Windows) | `ping -n 10 MACHINE_IP` |
| ping (IPv6) | `ping -6 MACHINE_IPV6` |
| traceroute (Linux) | `traceroute MACHINE_IP` |
| tracert (Windows) | `tracert MACHINE_IP` |
| traceroute (TCP mode) | `traceroute -T MACHINE_IP` |
| traceroute (ICMP mode) | `traceroute -I MACHINE_IP` |
| traceroute (IPv6) | `traceroute6 MACHINE_IPV6` |
| mtr (real-time) | `mtr MACHINE_IP` |
| telnet banner grab | `telnet MACHINE_IP PORT` |
| nc banner grab | `nc MACHINE_IP PORT` |
| nc listener | `nc -lvnp PORT` |
| nc (IPv6) | `nc -6 MACHINE_IPV6 PORT` |
| curl HTTP banner | `curl -I http://MACHINE_IP` |
| curl HTTPS banner | `curl -I https://MACHINE_IP` |
| openssl TLS probe | `openssl s_client -connect MACHINE_IP:443` |
| Developer Tools | `Ctrl+Shift+I` (Linux/Win) / `Option+Cmd+I` (macOS) |

---
