Wireshark — Packet Analysis

### What is Wireshark?

Wireshark is the most widely used **network traffic analyser** — an open-source tool that captures and dissects packets in real time or from saved capture files (`.pcap`/`.pcapng`).

> **Important:** Wireshark is **not an IDS (Intrusion Detection System)**. It does not modify packets or automatically flag threats — it is a passive inspection and analysis tool. Effective use depends on the analyst's knowledge of protocols and network behavior.

#### Common Use Cases

| Category | Examples |
|---|---|
| **Network troubleshooting** | Diagnosing congestion, load failures, misconfigured services |
| **Security analysis** | Identifying rogue hosts, abnormal port usage, suspicious traffic patterns |
| **Protocol learning** | Inspecting response codes, headers, and payload data at every OSI layer |

---

### GUI Overview

| Section | Purpose |
|---|---|
| **Toolbar** | Menus and shortcuts for capture, filtering, sorting, exporting, and merging |
| **Display Filter Bar** | Primary query input — filter which packets are shown |
| **Recent Files** | Quick access to recently opened capture files (double-click to reload) |
| **Capture Filter & Interfaces** | Select network interface to capture from; set pre-capture filters |
| **Status Bar** | Active profile, tool status, and packet count summary |

#### Three-Pane Packet View

| Pane | Contents |
|---|---|
| **Packet List** | One row per packet — source/destination addresses, protocol, and summary info |
| **Packet Details** | Hierarchical breakdown of the selected packet by protocol layer |
| **Packet Bytes** | Hex + ASCII representation of raw packet bytes; highlights the field clicked in the details pane |

---

### Loading Capture Files

Wireshark accepts `.pcap` and `.pcapng` files via:
- **File → Open**
- Drag-and-drop onto the Wireshark window
- Double-click the file in a file manager
- `File → Merge` to combine two capture files into one

> After merging, the combined capture must be **saved** before analysis begins.

---

### Traffic Sniffing (Live Capture)

| Button | Action |
|---|---|
| 🔵 Blue shark button | Start capturing |
| 🔴 Red button | Stop capturing |
| 🟢 Green button | Restart capturing |

The status bar shows the active interface and running packet count during capture.

---

### Packet Colouring

Wireshark colour-codes packets by protocol and condition to aid quick visual identification.

| Type | Scope | How to Set |
|---|---|---|
| **Permanent rules** | Persist across sessions (saved to profile) | Right-click menu or View → Coloring Rules |
| **Temporary rules** | Active only for the current session | View → Conversation Filter |

> "Colourise Packet List" in the View menu toggles all colouring on/off.

---

### Packet Dissection — OSI Layers in Wireshark

When you select a packet, the **Packet Details** pane shows it broken down by OSI layer:

| Layer | Wireshark Label | OSI Layer | Contains |
|---|---|---|---|
| 1 | Frame | Physical (1) | Frame number, capture timestamp, frame size |
| 2 | Source [MAC] | Data Link (2) | Source and destination MAC addresses |
| 3 | Source [IP] | Network (3) | Source and destination IP addresses |
| 4 | Protocol | Transport (4) | Protocol (TCP/UDP), source/destination ports |
| 4+ | Protocol Errors | Transport (4) | TCP segment reassembly details |
| 5–7 | Application Protocol | Application (5–7) | HTTP, FTP, SMTP details, etc. |
| 5–7 | Application Data | Application (5–7) | Raw application-level payload |

> Clicking any field in the Packet Details pane **highlights the corresponding bytes** in the Packet Bytes pane, and vice versa.

---

### Navigation & Investigation Features

#### Go to Packet
Navigate directly to a specific packet by number: **Go → Go to Packet** or `Ctrl + G`.

#### Find Packets (`Edit → Find Packet`)

Searches across the capture for packets matching a criterion.

| Input Type | Use When |
|---|---|
| **Display Filter** | Searching by protocol field/value |
| **Hex** | Searching for specific byte sequences |
| **String** | Searching for readable text within packet data |
| **Regex** | Pattern matching — most flexible |

> Searches are **case-insensitive by default** (toggle available). The search field matters: packet list, packet details, and packet bytes each contain different information — search in the pane where the target data appears.

#### Mark Packets (`Edit → Mark/Unmark Packet`)

Highlights specific packets in **black** for easy reference during investigation.

> Marked packets are **session-only** — marks are lost when the capture file is closed.

#### Packet Comments

Add persistent notes to individual packets — comments are **stored in the capture file** and survive across sessions until explicitly removed.

#### Export Packets (`File → Export Specified Packets`)

Extract a subset of packets from a large capture for focused sharing or analysis.

#### Export Objects (`File → Export Objects`)

Extract files that were transferred within the captured traffic. Supported protocols:

| Protocol | Use Case |
|---|---|
| HTTP | Recover downloaded files, images, etc. |
| SMB | Recover shared file transfers |
| FTP-DATA | Recover FTP file transfers |
| TFTP | Recover TFTP transfers |
| IMF / DICOM | Email/medical imaging data |

#### Time Display Format (`View → Time Display Format`)

Default: **Seconds Since Beginning of Capture**
Recommended for log correlation: **UTC Date and Time**

#### Expert Info (`Analyse → Expert Information`)

Wireshark automatically flags protocol anomalies:

| Severity | Colour | Meaning |
|---|---|---|
| **Chat** | Blue | Normal workflow events |
| **Note** | Cyan | Notable events (e.g., application error codes) |
| **Warn** | Yellow | Unusual errors or problem statements |
| **Error** | Red | Malformed packets or serious problems |

| Group | Covers |
|---|---|
| Checksum | Checksum validation errors |
| Comment | Detected packet comments |
| Deprecated | Use of deprecated protocols |
| Malformed | Malformed/corrupt packet detection |

> Expert Info is a starting point — it may produce false positives/negatives and should be interpreted alongside analyst knowledge.

---

### Filtering

Wireshark has **two distinct filter types**:

| Filter Type | Applied At | Purpose |
|---|---|---|
| **Capture Filter** | During capture | Limits which packets are recorded — reduces file size |
| **Display Filter** | After capture | Controls which packets are shown — does not delete packets |

> **Golden rule:** *"If you can click on it, you can filter and copy it."*

#### Applying Filters

| Method | How |
|---|---|
| **Apply as Filter** | Right-click a packet field → "Apply as Filter" — immediately filters on that exact value |
| **Prepare as Filter** | Right-click → "Prepare as Filter" — adds the query to the filter bar without applying; allows `AND`/`OR` chaining before execution |
| **Conversation Filter** | Right-click → "Conversation Filter" — shows only packets in the same conversation (by IP + port pair), hiding all others |
| **Colourise Conversation** | Right-click → "Colourise Conversation" — highlights the conversation without hiding other packets |
| **Apply as Column** | Right-click a field → "Apply as Column" — adds that field as a visible column in the packet list pane |

#### Follow Stream (`Analyse → Follow TCP/UDP/HTTP Stream`)

Reassembles and displays the complete application-level data exchange for a selected stream:

```
Client → Server:  shown in RED
Server → Client:  shown in BLUE
```

> Following a stream automatically applies a display filter isolating that stream. Remove it with the **X button** on the display filter bar to return to the full capture view.

---

### Common Display Filter Queries

#### By Protocol Name

```
http
arp
dhcp
dns
ftp
smtp
pop
imap
tcp
udp
```

#### By Port Number

```
tcp.port == 80       # HTTP
tcp.port == 443      # HTTPS
tcp.port == 22       # SSH
udp.port == 53       # DNS
```

#### By IP Address

```
ip.addr == 192.168.1.2           # source OR destination
ip.src == 192.168.1.2            # source only
ip.dst == 192.168.1.2            # destination only
ip.addr == 192.168.1.0/24        # entire subnet
```

#### Combined Filters

```
http and ip.addr == 192.168.1.5          # HTTP involving a specific IP
tcp.port == 80 and ip.src == 10.0.0.1   # HTTP from a specific source
not arp                                   # exclude ARP traffic
```

---

### Quick Reference

| Task | Method |
|---|---|
| Open a capture file | File → Open / drag-and-drop |
| Start/stop live capture | Blue/Red shark buttons |
| Merge two captures | File → Merge |
| View file properties | Statistics → Capture File Properties |
| Find a packet by content | Edit → Find Packet (`Ctrl + F`) |
| Jump to packet by number | Go → Go to Packet (`Ctrl + G`) |
| Mark a packet | Right-click → Mark/Unmark |
| Add a comment | Right-click → Packet Comment |
| Export selected packets | File → Export Specified Packets |
| Extract transferred files | File → Export Objects |
| Follow a stream | Right-click → Follow TCP/UDP/HTTP Stream |
| View protocol anomalies | Analyse → Expert Information |
| Change time format | View → Time Display Format → UTC |
| Apply a quick filter | Right-click field → Apply as Filter |
| Chain filter conditions | Right-click → Prepare as Filter → `and/or` |
| Add a packet field as column | Right-click field → Apply as Column |

---
