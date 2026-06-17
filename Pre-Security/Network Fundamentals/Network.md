## What Is a Network?


A **network** is two or more devices connected together so they can communicate and share resources. Networks can be as small as two computers in a home or as large as the global internet.

 In computing, a network can be formed by anywhere from 2 devices to billions. These devices include everything from your laptop and phone to security cameras, traffic lights and even farming!

Networks allow devices to:
- Share files and data
- Communicate via email, messaging, and voice
- Share resources like printers and storage
- Access remote systems and the internet
---
 
## Network Types
 
| Type | Full Name | Scale | Example |
|---|---|---|---|
| **PAN** | Personal Area Network | ~1–10 metres | Bluetooth headset connected to a phone |
| **LAN** | Local Area Network | Building / campus | Office computers on the same switch |
| **MAN** | Metropolitan Area Network | City / metro area | Connected office buildings across a city |
| **WAN** | Wide Area Network | Country / global | The internet; corporate sites linked across countries |
| **WLAN** | Wireless LAN | Building / campus | Wi-Fi network in a home or office |
 
---
##  Internet
The first iteration of the Internet was within the ARPANET project in the late 1960s. This project was funded by the United States Defence Department and was the first documented network in action. However, it wasn't until 1989 when the Internet as we know it was invented by Tim Berners-Lee by the creation of the World Wide Web (WWW). It wasn't until this point that the Internet started to be used as a repository for storing and sharing information, just like it is today.

The internet is a global network of interconnected computers and devices that communicate with each other using standardized protocols (rules for exchanging data).

Here's a simple breakdown:
What it is physically: Millions of computers, servers, smartphones, and other devices linked together through cables (fiber optic, copper), wireless signals (Wi-Fi, cellular), and satellites.

How it works: - Data is broken into small packets, sent across the network, and reassembled at the destination. The main language it uses is called TCP/IP (Transmission Control Protocol / Internet Protocol).

Key components:
- Servers — computers that store and serve websites, videos, emails, etc.
- Routers — devices that direct data to the right destination
- ISPs (Internet Service Providers) — companies that connect you to the internet (like MTN or Airtel in Nigeria)
- DNS (Domain Name System) — translates website names like google.com into numerical IP addresses
---
What it enables:

The World Wide Web (websites and web pages)
- Email and messaging
- Streaming (video, music)
- Cloud storage and computing
- Social media, online banking, e-commerce, and much more
---



## Device Identification on a Network

Just like humans are identified by their **name** and **fingerprints**, devices on a network have two identifiers:

---

### 1. IP Address (Internet Protocol Address)
- A set of **four numbers (octets)** — e.g., `192.168.1.77`
- Can **change** from device to device (like a name)
- Two types:
  - **Private IP** — identifies a device *within* a local network
  - **Public IP** — identifies a device *on the internet* (assigned by your ISP)

**IPv4 vs IPv6:**
| | IPv4 | IPv6 |
|---|---|---|
| Addresses | ~4.29 billion (2³²) | 340+ trillion (2¹²⁸) |
| Status | Running out | Newer solution |

---

### 2. MAC Address (Media Access Control)
- A **12-character hexadecimal** number — e.g., `a4:c3:f0:85:ac:2d`
- Permanently assigned at the factory (like a fingerprint)
- The **first 6 characters** = manufacturer; **last 6** = unique device number
- However, MAC addresses can be **spoofed** (faked), which can bypass security systems that trust devices based on their MAC address

---
## Ping — A Fundamental Network Tool

### What is Ping?
**Ping** is a network utility used to test the **connection between two devices**. It checks:
- Whether a connection **exists**
- How **reliable** and **fast** that connection is

---

### How It Works
Ping uses **ICMP (Internet Control Message Protocol)** packets in a simple two-step process:

```
Device A  ──── ICMP Echo Request ────►  Device B
Device A  ◄─── ICMP Echo Reply ─────  Device B
```

1. Your device sends an **ICMP Echo Request** to the target
2. The target responds with an **ICMP Echo Reply**
3. The **time taken** for this round trip is measured (in milliseconds)

---

### Basic Syntax
```bash
ping <IP address or website URL>
```

**Examples:**
```bash
ping 192.168.1.1        # Ping a device on your local network
ping google.com         # Ping a website
```

---

### What Ping Tells You
| Result | Meaning |
|---|---|
| Reply received | Connection exists and is working |
| Low ms (e.g. 5ms) | Fast, reliable connection |
| High ms (e.g. 300ms) | Slow or unstable connection |
| Request timed out | Device unreachable or blocking ping |

---


