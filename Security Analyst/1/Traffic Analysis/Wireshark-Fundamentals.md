# Wireshark Fundamentals

## Overview

This document covers the fundamentals of Wireshark, the most widely used open-source packet analyzer: its use cases, the main GUI sections and three-pane packet view, packet colouring, live capture controls, file management, packet dissection by layer, navigation and search, annotation and extraction features, time display formats, Expert Info, and the difference between capture and display filters with the right-click filtering options and basic display filter queries.

---

**Wireshark** is an open-source, cross-platform network packet analyzer capable of sniffing live traffic and inspecting packet captures (PCAP files) — one of the most widely used packet analysis tools available.

---

### Use Cases

- Detecting/troubleshooting network problems (load failures, congestion)
- Detecting security anomalies (rogue hosts, abnormal port usage, suspicious traffic)
- Investigating protocol details (response codes, payload data)

> **Important:** Wireshark is **not an IDS** — it only reads and displays packets, never modifies them. Detecting anomalies depends entirely on the analyst's own knowledge and investigation skill.

---

### The Main GUI — Five Key Sections

| Section | Function |
|---|---|
| **Toolbar** | Menus/shortcuts for sniffing, filtering, sorting, summarizing, exporting, merging |
| **Display Filter Bar** | Main query/filtering input |
| **Recent Files** | Quickly reopen previously investigated captures |
| **Capture Filter & Interfaces** | Choose capture filters and available sniffing interfaces (e.g., `lo`, `eth0`, `ens33`) |
| **Status Bar** | Tool status, profile, and packet count info |

---

### Loading and Viewing Packets

Once a PCAP is loaded, packet data appears across **three panes**:

| Pane | Shows |
|---|---|
| **Packet List Pane** | Summary per packet — source/destination, protocol, packet info |
| **Packet Details Pane** | Full protocol breakdown of the selected packet |
| **Packet Bytes Pane** | Hex + ASCII representation, highlighted in sync with the details pane |

---

### Packet Colouring

Wireshark colors packets by protocol/condition to speed up visual triage.

| Type | Persistence |
|---|---|
| **Temporary rules** | Session-only |
| **Permanent rules** | Saved to the profile, persist across sessions (via `View → Coloring Rules`) |

"Colourise Packet List" toggles coloring on/off.

---

### Traffic Sniffing

| Button | Action |
|---|---|
| **Blue "shark" button** | Start capturing |
| **Red button** | Stop capturing |
| **Green button** | Restart capturing |

The status bar shows the active interface and packet count in real time.

---

### File Management

| Feature | How |
|---|---|
| **Merge PCAP files** | `File → Merge` — combines two captures; save the result before further work |
| **View file details** | `Statistics → Capture File Properties` (or click the PCAP icon, bottom-left) — shows hash, capture time, comments, interface, and statistics |

---

### Packet Dissection — The 7 Layers

Clicking a packet reveals its full breakdown, roughly mapping to OSI layers:

| Layer | Shows |
|---|---|
| **Frame** | Physical-layer details of the captured frame |
| **Source [MAC]** | Data Link layer — source/destination MAC addresses |
| **Source [IP]** | Network layer — source/destination IPv4 addresses |
| **Protocol** | Transport layer — TCP/UDP details, source/destination ports |
| **Protocol Errors** | TCP segments requiring reassembly |
| **Application Protocol** | Application-layer protocol details (HTTP, FTP, SMB, etc.) |
| **Application Data** | The actual application-specific payload |

Clicking any detail highlights the corresponding bytes in the Packet Bytes pane.

---

### Navigating Packets

| Feature | Purpose |
|---|---|
| **Packet Numbers** | Unique sequential ID per packet — makes returning to a specific event easy |
| **Go to Packet** | Jump directly to a specific packet number, or navigate within a conversation |
| **Find Packets** (`Edit → Find Packet`) | Search by **Display filter**, **Hex**, **String**, or **Regex** across a chosen pane (list/details/bytes) — case-insensitive by default |

> **Key tip:** Search results depend on matching the right pane — searching the packet list pane won't find something only visible in the details pane.

---

### Annotating and Extracting

| Feature | Description |
|---|---|
| **Mark Packets** | Highlights a packet in black for quick reference; **lost when the file is closed** |
| **Packet Comments** | Persistent notes attached to a packet, saved with the capture file until manually removed |
| **Export Packets** | Save a filtered subset of packets (e.g., only suspicious ones) to a new file |
| **Export Objects** | Extract transferred files directly from supported protocol streams (**DICOM, HTTP, IMF, SMB, TFTP**) |

---

### Time Display Format

By default, times show as "Seconds Since Beginning of Capture." Switching to **UTC Time Display Format** (`View → Time Display Format`) is often more useful for correlating with other logs.

---

### Expert Info

Wireshark flags protocol states that may indicate anomalies — always treat these as **suggestions**, since false positives/negatives are possible.

| Severity | Color | Meaning |
|---|---|---|
| **Chat** | Blue | Normal workflow information |
| **Note** | Cyan | Notable events (e.g., app error codes) |
| **Warn** | Yellow | Warnings (unusual error codes/problems) |
| **Error** | Red | Serious problems (e.g., malformed packets) |

Common info groups: **Checksum** (errors), **Comment** (packet comment detection), **Deprecated** (deprecated protocol usage), **Malformed** (malformed packet detection).

Access via the status bar's bottom-left section or `Analyse → Expert Information`.

---

### Filtering — Capture vs. Display

| Filter Type | Purpose |
|---|---|
| **Capture Filters** | Determine what gets *captured* in the first place |
| **Display Filters** | Determine what's *shown* from an already-captured file |

> **Golden rule:** "If you can click on it, you can filter and copy it" — Wireshark's right-click menu covers most common filtering needs without writing raw queries.

#### Right-Click Filtering Options

| Option | Behavior |
|---|---|
| **Apply as Filter** | Filters on a single clicked field/value immediately |
| **Conversation Filter** | Shows all packets linked by IP/port to the selected packet (a full "conversation") |
| **Colourise Conversation** | Highlights linked packets without reducing the visible packet count (reversible via `View → Colourise Conversation → Reset Colourisation`) |
| **Prepare as Filter** | Builds the filter query but doesn't apply it yet — waits for Enter or further combination (AND/OR) |
| **Apply as Column** | Adds the clicked field as a new column in the packet list pane, for easy comparison across packets |
| **Follow Stream** (TCP/UDP/HTTP) | Reconstructs the full application-level conversation — can reveal plaintext usernames, passwords, or other transferred data. Server-originated data shown in blue, client-originated in red. Automatically applies a filter, removable via the "X" button on the display filter bar |

---

### Basic Display Filter Queries

| Filter Type | Syntax | Example |
|---|---|---|
| **By protocol name** | `<protocol>` | `http`, `arp`, `dhcp`, `ftp`, `smtp`, `pop`, `imap` |
| **By protocol port** | `tcp.port == <port>` / `udp.port == <port>` | `tcp.port == 80` |
| **By IP address** | `ip.addr == <IP>` | `ip.addr == 192.168.1.2` |

---

### Wireshark Quick Reference

| Concept | Summary |
|---|---|
| Wireshark = | Open-source packet analyzer (not an IDS — read-only) |
| Main panes | Packet List, Packet Details, Packet Bytes |
| Packet layer breakdown | Frame, MAC, IP, Transport Protocol, Protocol Errors, App Protocol, App Data |
| Search inputs | Display filter, Hex, String, Regex |
| Annotation | Marking (temporary), Comments (persistent) |
| Export types | Filtered packet subsets, or file objects (DICOM/HTTP/IMF/SMB/TFTP) |
| Expert Info severities | Chat, Note, Warn, Error |
| Filter types | Capture filters (what's recorded) vs. Display filters (what's shown) |
| Key right-click actions | Apply as Filter, Conversation Filter, Colourise Conversation, Prepare as Filter, Apply as Column, Follow Stream |
| Common filter syntax | `http`, `tcp.port == 80`, `ip.addr == <IP>` |

---

## Summary

- **Wireshark** is an open-source, cross-platform packet analyzer for live sniffing and PCAP inspection — used for troubleshooting network problems, spotting security anomalies, and investigating protocol details — but it is **not an IDS**: it only reads and displays packets, so finding anomalies depends on the analyst's own skill.
- The GUI centers on a **toolbar**, **display filter bar**, **recent files**, **capture filter and interface selection**, and a **status bar**, with packets viewed across three panes: **Packet List**, **Packet Details** (a layer-by-layer breakdown from Frame through Application Data), and **Packet Bytes** (synced hex and ASCII).
- Analysts navigate with packet numbers, **Go to Packet**, and **Find Packets** (by display filter, hex, string, or regex — matched against the correct pane), and annotate with temporary **marks** or persistent **packet comments**; **Export Packets** saves subsets and **Export Objects** extracts files from DICOM, HTTP, IMF, SMB, and TFTP streams.
- Switching the time display to **UTC** helps correlate with other logs, and **Expert Info** (Chat, Note, Warn, Error) flags possible anomalies that should be treated as suggestions rather than verdicts.
- **Capture filters** limit what is recorded while **display filters** limit what is shown; the right-click menu (**Apply as Filter, Conversation Filter, Colourise Conversation, Prepare as Filter, Apply as Column, Follow Stream**) covers most filtering needs, alongside basic queries like `http`, `tcp.port == 80`, and `ip.addr == <IP>`.
