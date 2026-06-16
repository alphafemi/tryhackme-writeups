## The OSI Model — Open Systems Interconnection

### What is the OSI Model?
The **OSI Model** is a framework that dictates how all networked devices **send, receive, and interpret data**. It allows devices with different designs and functions to communicate using a **common standard**.

> The process of adding information to data as it passes through each layer is called **Encapsulation**.

---

### The 7 Layers Overview

```
Layer 7 ── Application
Layer 6 ── Presentation
Layer 5 ── Session
Layer 4 ── Transport
Layer 3 ── Network
Layer 2 ── Data Link
Layer 1 ── Physical
```
> Memory Aid: **"All People Seem To Need Data Processing"**

---

## Layer 7 — Application
- The layer **users interact with most**
- Provides a **GUI** for users to send/receive data
- Protocols include:
  - **DNS** — translates website names to IP addresses
  - **HTTP/HTTPS** — web browsing
  - **Email clients**, browsers, file browsers (e.g. FileZilla)

---

## Layer 6 — Presentation
- Acts as a **translator** between the application and the rest of the model
- Standardises data format so different software can understand each other
- Example: Two people using **different email clients** still see the same email content
- Handles **data encryption** (e.g. HTTPS for secure websites)

---

## Layer 5 — Session
- **Creates and maintains** connections (sessions) between devices
- A session is opened when a connection is established and closed when idle or lost
- Supports **checkpoints** — if data is lost, only the newest data needs to be resent (saves bandwidth)
- Sessions are **unique** — data cannot travel across different sessions

```
Device A ——[Session Created]——► Device B
         ——[Data Transfer]———►
         ——[Session Closed]———►
```

---

## Layer 4 — Transport
Transmits data using one of two protocols:

### TCP — Transmission Control Protocol

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Guarantees data accuracy | Requires reliable connection |
| Syncs devices to prevent flooding | Slow connections cause bottlenecks |
| Error checking built in | Slower than UDP |

> **Used for:** File sharing, web browsing, email — where **accuracy matters**

### UDP — User Datagram Protocol

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Much faster than TCP | No guarantee data is received |
| No continuous connection needed | Unstable connections = poor experience |
| Flexible for developers | No error checking |

> **Used for:** Video streaming, ARP, DHCP — where **speed matters more than perfection**

### TCP vs UDP Summary

| Feature | TCP | UDP |
|---|---|---|
| **Speed** | Slower | Faster |
| **Reliability** | Guaranteed | Not guaranteed |
| **Error Checking** | Yes | No |
| **Connection** | Continuous | Connectionless |
| **Best For** | Email, browsing, files | Streaming, gaming, VoIP |

---

## Layer 3 — Network
- Handles **routing** and **reassembly** of data
- Determines the **most optimal path** for data using:
  - **Shortest path** — fewest devices to travel through
  - **Most reliable path** — least packet loss history
  - **Fastest path** — fibre over copper
- Routing protocols include:
  - **OSPF** — Open Shortest Path First
  - **RIP** — Routing Information Protocol
- Everything dealt with via **IP addresses** (e.g. `192.168.1.100`)
- Routers are **Layer 3 devices**

---

## Layer 2 — Data Link
- Focuses on **physical addressing** of data transmission
- Receives packets from Layer 3 and adds the **MAC address** of the destination
- Every device has a **NIC (Network Interface Card)** with a unique MAC address burned in at manufacture
- Also presents data in a **format suitable for transmission**
- MAC addresses **cannot be changed** (but can be spoofed)

---

## Layer 1 — Physical
- The **lowest layer** — actual physical hardware
- Transfers data as **electrical signals** in binary (1s and 0s)
- Includes cables, switches, network interface cards, etc.

---

## Complete OSI Model Summary Table

| Layer | Name | Key Responsibility | Examples |
|---|---|---|---|
| **7** | Application | User interaction with data | Browsers, email, DNS |
| **6** | Presentation | Translation & encryption | HTTPS, data formatting |
| **5** | Session | Creating & managing sessions | Checkpoints, session control |
| **4** | Transport | Data delivery (TCP/UDP) | TCP, UDP |
| **3** | Network | Routing via IP addresses | Routers, OSPF, RIP |
| **2** | Data Link | Physical MAC addressing | NIC, MAC addresses |
| **1** | Physical | Electrical signal transmission | Cables, binary signals |

---

## How Encapsulation Works

```
Sender                                    Receiver
  │                                           │
Layer 7 [Data]                         Layer 7 [Data]
Layer 6 [Data + Formatting]            Layer 6 [Data]
Layer 5 [Data + Session info]          Layer 5 [Data]
Layer 4 [Data + TCP/UDP header]        Layer 4 [Data]
Layer 3 [Data + IP header]             Layer 3 [Data]
Layer 2 [Data + MAC header]            Layer 2 [Data]
Layer 1 [Binary electrical signals] ──► Layer 1 [Binary signals]
```
> Each layer **adds its own information** (encapsulation) when sending, and **strips it off** (de-encapsulation) when receiving.

---

Would you like all topics from **LAN Topologies to the OSI Model** compiled into one full document, or shall we continue with more topics like **DNS** or **Packets & Frames**?
