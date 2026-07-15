tcpdump — Command-Line Packet Analysis

**tcpdump** is the command-line packet analyser — the scriptable, lightweight counterpart to Wireshark. It captures or reads packets from files and outputs them to the terminal, making it essential for remote server analysis and scripted workflows.

> Capturing live traffic requires **root/sudo** privileges. Reading from an existing `.pcap` file does not.

---

### Core Options

| Option | Explanation |
|---|---|
| `-i INTERFACE` | Capture on a specific interface (e.g., `eth0`, `wlo1`, `any`) |
| `-w FILE` | Write captured packets to a `.pcap` file (no live display) |
| `-r FILE` | Read packets from a `.pcap` file |
| `-c COUNT` | Stop after capturing `COUNT` packets |
| `-n` | Do not resolve IP addresses to hostnames |
| `-nn` | Do not resolve IP addresses **or** port numbers (e.g., keeps `443` instead of `https`) |
| `-v` | Verbose output (more packet detail) |
| `-vv` | More verbose |
| `-vvv` | Maximum verbosity |

#### List Available Interfaces

```bash
ip a s      # show all network interfaces and their addresses
```

#### Example Commands

```bash
# Capture 50 packets on eth0, verbose
sudo tcpdump -i eth0 -c 50 -v

# Capture all WiFi traffic to a file (runs until Ctrl+C)
sudo tcpdump -i wlo1 -w data.pcap

# Read a file, show 5 packets, no name resolution
tcpdump -r traffic.pcap -c 5 -n

# Capture on all interfaces, no resolution
sudo tcpdump -i any -nn
```

---

### Filtering Expressions

Without filters, tcpdump captures everything — filters are essential for focusing on traffic of interest.

#### Filter by Host

```bash
sudo tcpdump host example.com -w http.pcap   # both directions
sudo tcpdump src host 192.168.1.5             # from this IP only
sudo tcpdump dst host 192.168.1.5             # to this IP only
```

#### Filter by Port

```bash
sudo tcpdump port 53          # DNS (both directions)
sudo tcpdump src port 443     # HTTPS responses
sudo tcpdump dst port 22      # SSH connections
```

#### Filter by Protocol

```bash
sudo tcpdump icmp             # ICMP only (ping, traceroute)
sudo tcpdump udp              # UDP only
sudo tcpdump tcp              # TCP only
sudo tcpdump ip6              # IPv6 only
```

#### Logical Operators

| Operator | Meaning | Example |
|---|---|---|
| `and` | Both conditions must be true | `host 1.1.1.1 and tcp` |
| `or` | Either condition can be true | `udp or icmp` |
| `not` | Condition must be false | `not tcp` |

```bash
# SSH traffic on any interface
sudo tcpdump -i any tcp port 22

# NTP on WiFi
sudo tcpdump -i wlo1 udp port 123

# HTTPS to/from example.com, saved to file
sudo tcpdump -i eth0 host example.com and tcp port 443 -w https.pcap
```

#### Size Filters

```bash
tcpdump greater 512    # packets >= 512 bytes
tcpdump less 64        # packets <= 64 bytes
```

#### Count Output Lines (Pipe to `wc`)

```bash
tcpdump -r traffic.pcap src host 192.168.124.1 -n | wc
```

```
910  17415  140616    ← 910 matching packets
```

---

### Advanced Filtering — TCP Flags

#### Binary Operations (Background)

Tcpdump's header byte filters use **binary bitwise operations**:

| Operator | Name | Behaviour |
|---|---|---|
| `&` | AND | Returns 1 only if **both** input bits are 1 |
| `\|` | OR | Returns 1 if **either** input bit is 1 |
| `!` | NOT | Inverts the bit (0→1, 1→0) |

#### Header Byte Syntax

```
proto[expr:size]
```

| Part | Meaning |
|---|---|
| `proto` | Protocol name: `arp`, `ether`, `icmp`, `ip`, `ip6`, `tcp`, `udp` |
| `expr` | Byte offset within the header (0 = first byte) |
| `size` | Number of bytes to read (default: 1; can be 1, 2, or 4) |

**Examples:**
```bash
# Packets sent to a multicast Ethernet address
ether[0] & 1 != 0

# IP packets with options set
ip[0] & 0xf != 5
```

#### TCP Flag Filtering

`tcp[tcpflags]` references the TCP flags byte directly.

| Flag Token | TCP Flag |
|---|---|
| `tcp-syn` | SYN (Synchronize) |
| `tcp-ack` | ACK (Acknowledge) |
| `tcp-fin` | FIN (Finish) |
| `tcp-rst` | RST (Reset) |
| `tcp-push` | PSH (Push) |

```bash
# Only pure SYN packets (connection initiations)
tcpdump "tcp[tcpflags] == tcp-syn"

# Packets with SYN flag set (including SYN-ACK)
tcpdump "tcp[tcpflags] & tcp-syn != 0"

# Packets with either SYN or ACK set
tcpdump "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"
```

> Quoting the filter expression with `"..."` is recommended when using operators or parentheses to avoid shell interpretation issues.

---

### Output Format Options

| Option | Output Style |
|---|---|
| `-q` | **Quick** — brief one-line summary per packet |
| `-e` | Include **MAC addresses** (link-layer header) |
| `-A` | Print packet payload as **ASCII** |
| `-xx` | Print packet in **hexadecimal** (including link-layer header) |
| `-X` | Print packet in **hex + ASCII** side by side |

#### `-q` — Quick/Brief

```
18:59:59.979771 IP 104.18.12.149.https > g5000.45248: tcp 25
18:59:59.980574 IP g5000.45248 > 104.18.12.149.https: tcp 29
```

#### `-e` — Include MAC Addresses

```
18:59:59.979771 44:df:65:d8:fe:6c > 02:83:1e:40:5d:17, ethertype IPv4 (0x0800), length 91: ...
```

> Useful for ARP/DHCP analysis and identifying unexpected sources on the local network.

#### `-A` — ASCII View

```
18:59:59.979771 IP 104.18.12.149.https > g5000.45248: ...
E..M..@.5..)h.....BY.......|.;5}...
..1...k......j.3.2.....&9a.....-L
```

> Works well for **plaintext protocols** (unencrypted HTTP, SMTP, etc.). Encrypted traffic appears as garbled characters.

#### `-xx` — Hexadecimal

```
0x0000: 0283 1e40 5d17 44df 65d8 fe6c 0800 4500
0x0010: 004d fbd8 4000 3506 d229 6812 0c95 c0a8
...
```

> Universal — works for any content (encrypted, compressed, binary) since every byte maps to exactly two hex digits.

#### `-X` — Hex + ASCII Combined

```
0x0000: 4500 004d fbd8 4000 3506 d229 6812 0c95 E..M..@.5..)h...
0x0010: c0a8 4259 01bb b0c0 a0b1 037c aa3b 357d ..BY.......|.;5}
...
```

> The most comprehensive view — hex for binary precision, ASCII for human-readable portions.

---

### Wireshark vs. tcpdump

| Feature | Wireshark | tcpdump |
|---|---|---|
| Interface | GUI | Command-line |
| Platform | Desktop | Server/remote-friendly |
| Live analysis | Yes | Yes |
| File read/write | Yes (`.pcap`, `.pcapng`) | Yes (`.pcap`) |
| Protocol dissection | Deep, visual | Summary + raw bytes |
| Filtering | Display filters + capture filters | BPF (Berkeley Packet Filter) expressions |
| Scripting/automation | Limited | Excellent (pipes, `wc`, etc.) |
| Ideal for | Interactive visual inspection | Remote capture, automation, quick CLI checks |

---

### Complete Quick Reference

| Task | Command |
|---|---|
| List interfaces | `ip a s` |
| Capture on interface | `sudo tcpdump -i eth0` |
| Capture to file | `sudo tcpdump -i eth0 -w out.pcap` |
| Read from file | `tcpdump -r capture.pcap` |
| Limit to N packets | `tcpdump -c 20` |
| No name resolution | `tcpdump -nn` |
| Verbose output | `tcpdump -v` / `-vv` / `-vvv` |
| Filter by host | `tcpdump host 1.2.3.4` |
| Filter by port | `tcpdump port 80` |
| Filter by protocol | `tcpdump icmp` |
| Filter by source | `tcpdump src host 1.2.3.4` |
| Filter by destination | `tcpdump dst port 443` |
| Combine filters | `tcpdump host 1.1.1.1 and tcp port 443` |
| Exclude protocol | `tcpdump not arp` |
| SYN packets only | `tcpdump "tcp[tcpflags] == tcp-syn"` |
| Brief output | `tcpdump -q` |
| Show MACs | `tcpdump -e` |
| ASCII payload | `tcpdump -A` |
| Hex payload | `tcpdump -xx` |
| Hex + ASCII | `tcpdump -X` |
| Count matching packets | `tcpdump -r file.pcap host 1.2.3.4 -n \| wc` |

---
