## Packets, Frames, TCP/UDP & Ports

---

## 1. Packets & Frames

### What are They?
Both are **small pieces of data** that together form a larger message — but they operate at different OSI layers:

| | Packet | Frame |
|---|---|---|
| **OSI Layer** | Layer 3 (Network) | Layer 2 (Data Link) |
| **Contains** | IP header + payload | Encapsulates the packet + MAC addresses |
| **Analogy** | The **letter** inside an envelope | The **envelope** itself |

### Why Small Pieces?
- Less chance of **bottlenecking** the network
- Example: A website image is broken into packets, sent separately, then **reconstructed** on your device

---

### IP Packet Headers

| Header | Purpose |
|---|---|
| **Time to Live (TTL)** | Expiry timer — stops packets clogging the network forever |
| **Checksum** | Checks data integrity — if data changes, checksum won't match |
| **Source Address** | IP of the sending device |
| **Destination Address** | IP of the receiving device |

---
---

## 2. TCP — Transmission Control Protocol

### The TCP/IP Model
A simplified 4-layer version of the OSI model:

```
OSI Model          TCP/IP Model
─────────          ────────────
Layer 7 ┐
Layer 6 ├──────►  Application
Layer 5 ┘
Layer 4 ──────►   Transport
Layer 3 ──────►   Internet
Layer 2 ┐
Layer 1 ├──────►  Network Interface
        ┘
```

---

### TCP Headers

| Header | Description |
|---|---|
| **Source Port** | Randomly chosen port (0–65535) used by sender |
| **Destination Port** | Port of the receiving application (e.g. port 80 for web) |
| **Source IP** | IP address of sending device |
| **Destination IP** | IP address of receiving device |
| **Sequence Number** | Random number assigned to first data piece |
| **Acknowledgement Number** | Sequence number + 1 for next piece |
| **Checksum** | Mathematical check for data integrity |
| **Data** | The actual bytes being transmitted |
| **Flag** | Determines how packet is handled (SYN, ACK, FIN etc.) |

---

### The Three-Way Handshake
TCP must **establish a connection** before sending data:

```
[Client]                          [Server]
   |                                  |
   |———— 1. SYN ————————————————————►|
   |        "Let's connect! My ISN=0" |
   |                                  |
   |◄——— 2. SYN/ACK ——————————————————|
   |   "Got it! My ISN=5000, ACK=0"  |
   |                                  |
   |———— 3. ACK ————————————————————►|
   |        "Acknowledged ISN=5000"   |
   |                                  |
   |———— 4. DATA ———————————————————►|
   |◄——— DATA ————————————————————————|
   |                                  |
   |———— 5. FIN ————————————————————►|
   |◄——— FIN/ACK —————————————————————|
   |———— ACK ————————————————————————►|
```

### TCP Messages Explained

| Step | Message | Description |
|---|---|---|
| 1 | **SYN** | Client initiates connection |
| 2 | **SYN/ACK** | Server acknowledges and syncs |
| 3 | **ACK** | Client confirms receipt |
| 4 | **DATA** | Actual data is transmitted |
| 5 | **FIN** | Cleanly closes the connection |
| — | **RST** | Abruptly ends connection (error/fault) |

### Sequence Number Example

| Device | Initial Sequence | Final Sequence |
|---|---|---|
| Client | 0 | 0+1 = 1 |
| Client | 1 | 1+1 = 2 |
| Client | 2 | 2+1 = 3 |

### TCP Advantages & Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Guarantees data integrity | Requires reliable connection |
| Syncs devices, prevents flooding | Slow connections cause bottlenecks |
| Error checking built in | Slower than UDP |

---
---

## 3. UDP — User Datagram Protocol

### How it Differs from TCP
- **Stateless** — no connection established before sending
- **No handshake**, no synchronisation, no acknowledgement
- Data is sent and **hoped to arrive** — no guarantee

### UDP Headers

| Header | Description |
|---|---|
| **Time to Live (TTL)** | Expiry timer for the packet |
| **Source Address** | IP of sending device |
| **Destination Address** | IP of receiving device |
| **Source Port** | Randomly chosen port (0–65535) |
| **Destination Port** | Port of receiving application |
| **Data** | Bytes being transmitted |

### UDP Advantages & Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Much faster than TCP | No guarantee data is received |
| No continuous connection needed | Unstable connections = poor experience |
| Flexible for developers | No data integrity checks |

---

### TCP vs UDP — Full Comparison

| Feature | TCP | UDP |
|---|---|---|
| **Speed** | Slower | Faster |
| **Reliability** | Guaranteed | Not guaranteed |
| **Handshake** | Yes (3-way) | No |
| **Error Checking** | Yes | No |
| **Connection Type** | Continuous | Connectionless |
| **Best For** | Files, email, browsing | Streaming, gaming, VoIP |

---
---

## 4. Ports

### What is a Port?
A **port** is a numerical endpoint (0–65535) through which data is exchanged. Think of it like a **harbour** — ships (data) can only dock at compatible ports.

```
Device
├── Port 80  → Web Traffic (HTTP)
├── Port 443 → Secure Web (HTTPS)
├── Port 22  → SSH
├── Port 21  → FTP
└── Port 3389 → Remote Desktop
```

### Common Port Numbers (0–1024)

| Protocol | Port | Description |
|---|---|---|
| **FTP** | 21 | File transfer (client-server) |
| **SSH** | 22 | Secure remote login (text-based) |
| **HTTP** | 80 | World Wide Web browsing |
| **HTTPS** | 443 | Secure web browsing (encrypted) |
| **SMB** | 445 | File & device sharing (e.g. printers) |
| **RDP** | 3389 | Remote desktop (visual interface) |

> **Note:** Applications can run on non-standard ports (e.g. web server on port `8080`), but you must specify it with a colon: `http://website.com:8080`

---
