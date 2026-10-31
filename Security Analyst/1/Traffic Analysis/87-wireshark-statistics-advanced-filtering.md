# Wireshark Statistics & Advanced Filtering

## Overview

This document builds on Wireshark basics, covering the Statistics menu for high-level capture overviews (including name resolution and GeoIP mapping), a recap of capture vs. display filters and their syntax, comparison and logical operators, the filter toolbar's colour feedback, protocol-specific filters for IP, TCP/UDP, HTTP, and DNS, advanced operators and functions (`contains`, `matches`, `in`, `upper()`, `lower()`, `string()`), and tools for reusing filters (bookmarks, filter buttons, and profiles).

---

Building on Wireshark basics, the **Statistics** menu provides a high-level overview of a capture, while advanced **display filter** syntax enables precise, protocol-level investigation.

---

### The Statistics Menu

| Option | Purpose |
|---|---|
| **Resolved Addresses** | Lists IPs and their resolved hostnames (from DNS answers in the capture) — quickly reveals accessed resources |
| **Protocol Hierarchy** | Tree view of all protocols by packet count/percentage — right-click any entry to filter on it directly |
| **Conversations** | Traffic between two specific endpoints, viewable in 5 formats: Ethernet, IPv4, IPv6, TCP, UDP |
| **Endpoints** | Unique values for a single field type (Ethernet, IPv4, IPv6, TCP, UDP) rather than paired conversations |
| **IPv4/IPv6 Statistics** | Narrows statistics to a specific IP version |
| **DNS** | Breaks down DNS traffic — rcode, opcode, class, query type, service/query stats |
| **HTTP** | Breaks down HTTP traffic — request/response codes and original requests |

#### Name Resolution

- **MAC → Manufacturer:** Resolved via the first 3 bytes of the MAC (OUI), toggled via the "Name resolution" button in the Endpoints window
- **IP/Port name resolution:** Off by default — enable via `Edit → Preferences → Name Resolution`; once enabled, resolved names appear in the packet list, Conversations, and Endpoints views

#### GeoIP Mapping

Not enabled by default — requires a **MaxMind GeoIP database**, configured via `Edit → Preferences → Name Resolution → MaxMind database directories`. Once set up, matching IPs show geolocation details under IP protocol info, and an active internet connection is required to render the map view itself.

---

### Capture Filters vs. Display Filters (Recap)

| Filter Type | When Set | Purpose |
|---|---|---|
| **Capture Filters** | Before capturing; fixed during capture | Save only specific traffic to the file |
| **Display Filters** | Anytime | Narrow the *view* of already-captured packets; changeable freely |

> Capture and display filter syntax is **not interchangeable** — an expression valid in one won't necessarily work in the other.

#### Capture Filter Syntax

Uses byte offsets, hex values, and masks with boolean logic — less intuitive than display filters.

| Element | Options |
|---|---|
| **Scope** | `host`, `net`, `port`, `portrange` |
| **Direction** | `src`, `dst`, `src or dst`, `src and dst` |
| **Protocol** | `ether`, `wlan`, `ip`, `ip6`, `arp`, `rarp`, `tcp`, `udp` |

**Example:** `tcp port 80`

#### Display Filter Syntax

Wireshark's most powerful feature — supports ~3,000 protocols for granular, field-level filtering.

**Example:** `tcp.port == 80`

---

### Comparison Operators

| English | C-Like | Example |
|---|---|---|
| `eq` | `==` | `ip.src == 10.10.10.100` |
| `ne` | `!=` | `ip.src != 10.10.10.100` |
| `gt` | `>` | `ip.ttl > 250` |
| `lt` | `<` | `ip.ttl < 10` |
| `ge` | `>=` | `ip.ttl >= 0xFA` |
| `le` | `<=` | `ip.ttl <= 0xA` |

> Wireshark accepts both decimal and hexadecimal values in filters.

### Logical Expressions

| English | C-Like | Example |
|---|---|---|
| `and` | `&&` | `(ip.src == 10.10.10.100) AND (ip.src == 10.10.10.111)` |
| `or` | `\|\|` | `(ip.src == 10.10.10.100) OR (ip.src == 10.10.10.111)` |
| `not` | `!` | `!(ip.src == 10.10.10.222)` |

> Use `!(value)` rather than `!=value` — the latter is deprecated and can produce inconsistent results.

---

### The Filter Toolbar

| Feature | Detail |
|---|---|
| **Lowercase syntax** | All filters are written in lowercase |
| **Autocomplete** | Breaks down protocol fields, each level separated by a `.` |
| **Color feedback** | 🟢 Green = valid, 🔴 Red = invalid, 🟡 Yellow = works but unreliable — replace with a valid filter |

---

### Protocol-Specific Filters

#### IP Filters (Network Layer)

| Filter | Description |
|---|---|
| `ip` | All IP packets |
| `ip.addr == 10.10.10.111` | All packets containing this IP (either direction) |
| `ip.addr == 10.10.10.0/24` | All packets within this subnet |
| `ip.src == 10.10.10.111` | Packets originating from this IP |
| `ip.dst == 10.10.10.111` | Packets destined to this IP |

> `ip.addr` ignores direction; `ip.src`/`ip.dst` filter by direction specifically.

#### TCP/UDP Filters (Transport Layer)

| Filter | Description |
|---|---|
| `tcp.port == 80` | All TCP packets on port 80 |
| `udp.port == 53` | All UDP packets on port 53 |
| `tcp.srcport == 1234` | TCP packets originating from port 1234 |
| `udp.srcport == 1234` | UDP packets originating from port 1234 |
| `tcp.dstport == 80` | TCP packets destined to port 80 |
| `udp.dstport == 5353` | UDP packets destined to port 5353 |

#### HTTP/DNS Filters (Application Layer)

| Filter | Description |
|---|---|
| `http` | All HTTP packets |
| `dns` | All DNS packets |
| `http.response.code == 200` | HTTP responses with code 200 |
| `dns.flags.response == 0` | DNS requests |
| `http.request.method == "GET"` | HTTP GET requests |
| `dns.flags.response == 1` | DNS responses |
| `http.request.method == "POST"` | HTTP POST requests |
| `dns.qry.type == 1` | DNS "A" record queries |

---

### Display Filter Expressions (Built-in Reference)

`Analyse → Display Filter Expression` provides a browsable guide to every supported protocol field, its accepted value type, and any predefined values — essential since memorizing every field across ~3,000 supported protocols isn't realistic.

---

### Advanced Filtering — Operators and Functions

| Filter | Type | Description | Example |
|---|---|---|---|
| **`contains`** | Comparison | Case-sensitive substring search within a field | `http.server contains "Apache"` |
| **`matches`** | Comparison | Case-insensitive regex pattern match | `http.host matches "\.(php\|html)"` |
| **`in`** | Set Membership | Matches a field against a defined set/range | `tcp.port in {80 443 8080}` |
| **`upper()`** | Function | Converts a string field to uppercase before comparison | `upper(http.server) contains "APACHE"` |
| **`lower()`** | Function | Converts a string field to lowercase before comparison | `lower(http.server) contains "apache"` |
| **`string()`** | Function | Converts a non-string value to a string for pattern matching | `string(frame.number) matches "[13579]$"` |

---

### Bookmarks, Filter Buttons, and Profiles

| Feature | Purpose |
|---|---|
| **Filter Bookmarks** | Save frequently-used/complex filters in the filter toolbar's bookmark section for quick reuse |
| **Filter Buttons** | One-click buttons that apply a saved filter instantly |
| **Profiles** (`Edit → Configuration Profiles`) | Save entire sets of preferences (coloring rules, filter buttons, etc.) per investigation type, avoiding repetitive reconfiguration between cases |

---

### Wireshark Statistics & Filtering Quick Reference

| Concept | Summary |
|---|---|
| Statistics menu highlights | Resolved Addresses, Protocol Hierarchy, Conversations, Endpoints, IPv4/6 Stats, DNS, HTTP |
| Name resolution | MAC (via OUI), IP/Port (manual enable) |
| GeoIP | Requires MaxMind DB + config; map view needs internet access |
| Filter types | Capture (pre-capture, fixed) vs. Display (post-capture, flexible) |
| Comparison operators | `==`, `!=`, `>`, `<`, `>=`, `<=` |
| Logical operators | `and`/`&&`, `or`/`\|\|`, `not`/`!` |
| Filter color codes | Green (valid), Red (invalid), Yellow (works but unreliable) |
| Advanced operators | `contains`, `matches`, `in`, `upper()`, `lower()`, `string()` |
| Reusability tools | Bookmarks, Filter Buttons, Profiles |

---

## Summary

- The **Statistics menu** gives a high-level view of a capture — **Resolved Addresses** (IPs mapped to hostnames from DNS answers), **Protocol Hierarchy** (protocols by share of packets, filterable by right-click), **Conversations** (paired endpoints across Ethernet, IPv4, IPv6, TCP, and UDP), **Endpoints** (unique values per field), plus **IPv4/IPv6**, **DNS**, and **HTTP** breakdowns; MAC addresses resolve to manufacturers via the OUI, IP/port name resolution is off by default, and GeoIP mapping needs a MaxMind database plus an internet connection for the map view.
- **Capture filters** (set before capturing, fixed during the capture, using `host`/`net`/`port`/`portrange` syntax) and **display filters** (changeable anytime, covering roughly 3,000 protocols) use **different syntax** and are not interchangeable.
- Display filters combine **comparison operators** (`==`, `!=`, `>`, `<`, `>=`, `<=`, accepting decimal or hex) with **logical operators** (`and`/`&&`, `or`/`||`, `not`/`!`) — preferring `!(value)` over the deprecated `!=value` — and the filter toolbar's colour feedback signals validity (green valid, red invalid, yellow works but unreliable).
- Protocol-specific filters target IP (`ip.addr`, `ip.src`, `ip.dst`), TCP/UDP (`tcp.port`, `udp.srcport`, `tcp.dstport`, etc.), and HTTP/DNS (`http.request.method == "GET"`, `http.response.code == 200`, `dns.flags.response`, `dns.qry.type`), with `Analyse → Display Filter Expression` as a built-in field reference.
- Advanced operators and functions — **`contains`** (case-sensitive), **`matches`** (case-insensitive regex), **`in`** (set membership), and **`upper()`**, **`lower()`**, **`string()`** — enable more precise matching, while **filter bookmarks, filter buttons,** and **configuration profiles** save time reusing filters and settings across investigations.

## Related Documents

- [86. Wireshark Fundamentals](./86-wireshark-fundamentals.md)
- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
- [40. Wireshark — Packet Analysis](./40-wireshark-packet-analysis.md)
- [85. Network Traffic Analysis (NTA) Fundamentals](./85-network-traffic-analysis-nta-fundamentals.md)
