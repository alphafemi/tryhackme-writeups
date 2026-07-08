Networking Models & Protocols — Deep Dive
 

 
### The OSI Model — Expanded Reference
 
Developed by the International Organization for Standardization (ISO), the OSI model defines a **conceptual framework** for network communications — theoretical in nature, but essential for understanding layered networking and terminology such as "Layer 3 switch" or "Layer 7 firewall."
 
> **Mnemonic (bottom to top):** *"Please Do Not Throw Spinach Pizza Away"*
> (Physical, Data Link, Network, Transport, Session, Presentation, Application)
 
| Layer | Name | Main Function | Protocol/Standard Examples |
|---|---|---|---|
| 7 | **Application** | Services and interfaces for user applications | HTTP, FTP, DNS, SMTP, IMAP, POP3 |
| 6 | **Presentation** | Data encoding, encryption, compression | Unicode, MIME, JPEG, PNG, MPEG |
| 5 | **Session** | Establish, maintain, synchronize application sessions | NFS, RPC |
| 4 | **Transport** | End-to-end communication, segmentation, flow control | TCP, UDP |
| 3 | **Network** | Logical addressing and routing between networks | IP, ICMP, IPSec |
| 2 | **Data Link** | Reliable transfer between adjacent nodes on same segment | Ethernet (802.3), WiFi (802.11) |
| 1 | **Physical** | Physical transmission medium and signal encoding | Electrical, optical, wireless signals |
 
#### Layer Detail Notes
 
**Layer 1 — Physical**
Defines the transmission medium and binary signal encoding. Examples include Ethernet cable, optical fibre, and WiFi radio bands (2.4 GHz, 5 GHz, 6 GHz).
 
**Layer 2 — Data Link**
Governs communication between nodes on the **same network segment** (e.g., 10 computers connected to a single switch). Addresses are **MAC addresses** — 6-byte hexadecimal identifiers where the leftmost 3 bytes identify the manufacturer.
 
```
MAC Address:   cc:5e:f8:02:21:a7
               └── Vendor (3 bytes) ──┘ └─ Device (3 bytes) ─┘
```
 
Every Ethernet/WiFi frame contains:
- **Destination MAC** — where the frame is going
- **Source MAC** — where the frame came from
**Layer 3 — Network**
Handles **routing between different networks** — not just within a single segment. Devices operating at this layer (routers) inspect IP addresses and forward packets toward their destination via the best available path.
 
**Layer 4 — Transport**
Enables communication between **processes** on different hosts using port numbers. Supports flow control, segmentation, and (with TCP) error correction.
 
**Layer 5 — Session**
Manages the lifecycle of application-to-application connections — initiation, maintenance, synchronization, and recovery from transmission failures.
 
**Layer 6 — Presentation**
Translates data between network format and application-readable format. Example: MIME encodes binary email attachments as 7-bit ASCII for transmission.
 
**Layer 7 — Application**
Directly serves end-user applications — the protocols users interact with most (HTTP, DNS, FTP, etc.).
 
---
 
### The TCP/IP Model
 
Developed in the 1970s by the US Department of Defense (DoD), the TCP/IP model is the **implemented standard** underlying the modern internet. Its key design goal: allow the network to continue operating even as portions of it fail (critical for military resilience).
 
#### OSI → TCP/IP Layer Mapping
 
| OSI Layer | OSI Name | TCP/IP Model (4-layer RFC 1122) | TCP/IP Model (5-layer, modern texts) |
|---|---|---|---|
| 7 | Application | Application | Application |
| 6 | Presentation | Application | Application |
| 5 | Session | Application | Application |
| 4 | Transport | Transport | Transport |
| 3 | Network | Internet | Network |
| 2 | Data Link | Link | Link |
| 1 | Physical | *(not defined in RFC 1122)* | Physical |
 
> The 5-layer variant (which explicitly includes the Physical layer) is increasingly common in modern networking textbooks (e.g., Kurose & Ross, *Computer Networking: A Top-Down Approach*).
 
---
 
### IP Addressing — In Depth
 
Every networked host requires a **unique IP address** — functioning like a postal address — to be unambiguously reachable.
 
#### IPv4 Structure
 
An IPv4 address consists of **4 octets (32 bits total)**, each representing a decimal value from `0` to `255`:
 
```
192 . 168 . 66 . 89
 │      │    │    │
 8 bits each = 32 bits total
```
 
- Total unique IPv4 addresses: 2³² ≈ **4.29 billion**
- `0` = network address (reserved)
- `255` = broadcast address (reserved)
#### IPv4 vs. IPv6
 
| | IPv4 | IPv6 |
|---|---|---|
| Address length | 32 bits | 128 bits |
| Notation | `192.168.1.1` | `fe80::73e1:ca5e:3f93:b1b3` |
| Address space | ~4.29 billion | ~340 undecillion |
 
#### Subnet Notation
 
A subnet mask restricts which part of an address identifies the **network** vs. the **host**.
 
| Notation | Equivalent | Meaning |
|---|---|---|
| `255.255.255.0` | `/24` | Leftmost 24 bits fixed — 254 usable hosts |
| `192.168.66.89/24` | Network: `192.168.66.0` | Host range: `.1`–`.254`; Broadcast: `.255` |
 
#### Private IP Address Ranges (RFC 1918)
 
| Range | CIDR | Description |
|---|---|---|
| `10.0.0.0` – `10.255.255.255` | `10/8` | Large private range |
| `172.16.0.0` – `172.31.255.255` | `172.16/12` | Medium private range |
| `192.168.0.0` – `192.168.255.255` | `192.168/16` | Small private range (most home networks) |
 
> Private addresses **cannot be reached from the internet** directly — a router with a **public IP** and **NAT (Network Address Translation)** must bridge the gap.
 
#### Checking Your IP Configuration
 
| OS | Command | Output |
|---|---|---|
| **Windows** | `ipconfig` | IP, mask, gateway |
| **Linux/Unix** | `ifconfig` | IP, mask, broadcast, MAC |
| **Linux (modern)** | `ip a s` | IP with CIDR notation, broadcast, MAC |
 
```bash
$ ifconfig
wlo1: inet 192.168.66.89  netmask 255.255.255.0  broadcast 192.168.66.255
 
$ ip a s
wlo1: inet 192.168.66.89/24  brd 192.168.66.255
```
 
#### Routing
 
A **router** operates at Layer 3, examining destination IP addresses and forwarding packets hop-by-hop through the network toward their destination — similar to a post office routing parcels through intermediate sorting centers.
 
```
Host A → Router 1 → Router 2 → ... → Router N → Host B
         (each router inspects IP and forwards to next best hop)
```
 
---
 
### Transport Layer Protocols — UDP and TCP
 
#### UDP (User Datagram Protocol)
 
| Attribute | Detail |
|---|---|
| Connection type | **Connectionless** — no handshake required |
| Reliability | **No delivery guarantee** — packets may be lost, duplicated, or reordered |
| Speed | Fast — minimal overhead |
| Analogy | Standard mail with no tracking or delivery confirmation |
 
> **Port numbers** (2 bytes, range: 1–65535) identify the specific process on the host. UDP uses them to direct incoming datagrams to the correct application.
 
#### TCP (Transmission Control Protocol)
 
| Attribute | Detail |
|---|---|
| Connection type | **Connection-oriented** — requires establishment before data transfer |
| Reliability | **Guaranteed delivery** — sequencing, acknowledgment, and retransmission |
| Speed | Slower — higher overhead than UDP |
| Analogy | Tracked mail delivery with signature confirmation |
 
TCP assigns a **sequence number** to every data byte, allowing the receiver to detect lost or duplicated packets and request retransmission. The receiver sends **acknowledgement numbers** confirming successful receipt.
 
#### TCP Three-Way Handshake
 
```
Client                            Server
  │── SYN (client ISN) ──────────▶│
  │◀── SYN-ACK (server ISN) ───────│
  │── ACK ─────────────────────────▶│
  │   (connection established)      │
  │── DATA ────────────────────────▶│
```
 
| Packet | Flag(s) | Purpose |
|---|---|---|
| 1 | `SYN` | Client initiates; sends its initial sequence number (ISN) |
| 2 | `SYN-ACK` | Server acknowledges; sends its own ISN |
| 3 | `ACK` | Client acknowledges server's ISN; connection is established |
 
#### UDP vs. TCP
 
| Feature | UDP | TCP |
|---|---|---|
| Connection | Connectionless | Connection-oriented |
| Delivery guarantee | None | Yes |
| Ordering | Not maintained | Maintained via sequence numbers |
| Speed | Faster | Slower |
| Overhead | Minimal | Higher |
| Best for | Streaming, DNS, DHCP, gaming | Web, email, file transfer |
 
---
 
### Encapsulation
 
**Encapsulation** is the process of each OSI layer wrapping the data it receives from the layer above with its own header (and sometimes a trailer) before passing it down.
 
```
Application Data
      ↓  [Transport adds TCP/UDP header]
  TCP Segment / UDP Datagram
      ↓  [Network adds IP header]
  IP Packet
      ↓  [Data Link adds Ethernet/WiFi header + trailer]
  Frame
      ↓  [Physical converts to electrical/optical/wireless signal]
  Bits
```
 
On the **receiving side**, the process reverses — each layer strips its own header before passing data up to the layer above, until the original application data is recovered.
 
#### Packet Life — Worked Example
 
A search query sent to TryHackMe:
 
| Step | Layer | Action |
|---|---|---|
| 1 | Application | Browser formats an HTTPS/HTTP request |
| 2 | Transport | TCP creates a segment; three-way handshake establishes connection |
| 3 | Internet | IP adds source (your IP) and destination (TryHackMe server IP) addresses |
| 4 | Link | Ethernet/WiFi adds MAC header + trailer; packet sent to router |
| 5 | Routers | Each router strips the link frame, reads the IP destination, re-encapsulates, and forwards |
| 6 | Destination | Process reverses — headers stripped layer by layer until HTTP request reaches the web server |
 
---
 
### Telnet — Manual Protocol Interaction
 
**Telnet** is a text-based protocol originally designed for remote terminal access, but its simplicity makes it useful for **manually connecting to any listening TCP port** to test or probe services directly.
 
```bash
telnet <ip_address> <port>
```
 
> ⚠️ Telnet transmits data in **plain text** with no encryption — it should not be used for sensitive remote administration (use SSH instead). However, it remains a valid diagnostic tool for testing TCP connectivity.
 
#### Echo Server (Port 7)
 
```bash
$ telnet 10.114.181.127 7
Connected to 10.114.181.127.
Hi
Hi                     ← server echoes back
How are you?
How are you?           ← server echoes back
^]
telnet> quit
```
 
#### Daytime Server (Port 13)
 
```bash
$ telnet 10.114.181.127 13
Connected to 10.114.181.127.
Thu Jun 20 12:36:32 PM UTC 2024
Connection closed by foreign host.
```
 
#### Web Server — Manual HTTP Request (Port 80)
 
```bash
$ telnet 10.114.181.127 80
Connected to 10.114.181.127.
GET / HTTP/1.1
Host: telnet.thm
                        ← blank line required to terminate the HTTP request
HTTP/1.1 200 OK
Content-Type: text/html
...
```
 
> The blank line after `Host:` is **required** — it signals the end of the HTTP request headers per the HTTP specification. The server then responds with headers and the page content.
 
#### Why Telnet is Useful for Learning
 
By manually crafting protocol messages through Telnet, you can directly observe how application-layer protocols (HTTP, SMTP, FTP, etc.) behave at the byte level — without any browser or client abstracting the interaction.
 
---
 
### Quick Reference
 
| Concept | Key Fact |
|---|---|
| OSI layers | 7 layers, numbered 1 (Physical) to 7 (Application) |
| TCP/IP layers (RFC 1122) | 4 layers: Link, Internet, Transport, Application |
| IPv4 address length | 32 bits (4 octets) |
| IPv6 address length | 128 bits |
| Private address ranges | 10.x.x.x, 172.16–31.x.x, 192.168.x.x |
| Port number range | 1–65535 (port 0 reserved) |
| UDP | Connectionless, fast, no delivery guarantee |
| TCP | Connection-oriented, reliable, sequenced |
| TCP handshake | SYN → SYN-ACK → ACK |
| Encapsulation | Each layer adds a header (data travels down); each layer strips a header (data travels up) |
| Telnet | `telnet <ip> <port>` — manual TCP connection to any listening port |
| Check IP (Linux) | `ifconfig` or `ip a s` |
| Check IP (Windows) | `ipconfig` |
