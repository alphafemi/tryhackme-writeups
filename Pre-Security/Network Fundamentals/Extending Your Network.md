## Port Forwarding, Firewalls, VPNs, Routers & Switches

---

## 1. Port Forwarding

### What is Port Forwarding?
Port forwarding allows applications and services to be **accessible over the internet**, beyond just the local network.

### Without Port Forwarding (Intranet Only)
```
[Computer A] ─┐
               ├──► [Router] ──► Internet (❌ cannot access webserver)
[Computer B] ─┘
[Webserver 192.168.1.10:80] (only local devices can access it)
```

### With Port Forwarding (Internet Accessible)
```
[Computer A] ─┐
               ├──► [Router 82.62.51.70] ──► Internet (✅ publicly accessible)
[Computer B] ─┘         |
                   [Webserver :80]
```

### Key Points
- Configured at the **router**
- Opens **specific ports** for external access
- Different from a firewall — port forwarding **opens** ports; firewalls **control traffic** through them

---
---

## 2. Firewalls

### What is a Firewall?
A firewall is **border security for a network** — it controls what traffic is allowed **in or out** based on rules.

### What Firewalls Inspect
- **Source** — where is the traffic coming from?
- **Destination** — where is it going?
- **Port** — what port is it using?
- **Protocol** — is it TCP, UDP, or both?

---

### Two Main Types of Firewalls

| Type | How it Works | Pros | Cons |
|---|---|---|---|
| **Stateful** | Analyses the **entire connection** | Smarter, dynamic decisions | Uses more resources |
| **Stateless** | Uses **fixed rules** per individual packet | Lightweight, great for DDoS | Dumb — only as good as its rules |

### Stateful vs Stateless Illustrated

```
STATEFUL:
[Device] ──► [Firewall analyses whole connection] ──► Allow/Block entire device

STATELESS:
[Device] ──► [Firewall checks each packet against rules] ──► Allow/Block per packet
```

### Firewall Forms
- **Dedicated hardware** — large business networks
- **Residential routers** — home networks
- **Software firewalls** — e.g. Snort

---
---

## 3. VPN — Virtual Private Network

### What is a VPN?
A VPN creates a **secure encrypted tunnel** between devices on different networks over the internet, forming their own **private network**.

```
[Office #1] ════════════════════ [Office #2]
  Network #1    VPN Tunnel         Network #2
      └──────── Network #3 ───────┘
              (Private VPN Network)
```

---

### Benefits of a VPN

| Benefit | Description |
|---|---|
| 🌍 **Geographic Connection** | Links offices/networks in different locations |
| 🔒 **Privacy** | Encrypts data so it can't be intercepted or sniffed |
| 🕵️ **Anonymity** | Hides traffic from ISPs and intermediaries |

> ⚠️ A VPN that **logs your data** provides no more anonymity than not using one at all.

---

### VPN Technologies

| Technology | Description |
|---|---|
| **PPP** | Handles authentication & encryption using private key + public certificate. Not routable on its own |
| **PPTP** | Carries PPP data across networks. Easy to set up but **weakly encrypted** |
| **IPSec** | Encrypts data using existing IP framework. **Harder to set up** but **strongly encrypted** |

---
---

## 4. Routers

### What is a Router?
A router **connects networks** and passes data between them using **routing** — finding the best path for data to travel.

- Operates at **Layer 3** of the OSI model
- Has an admin interface for configuring port forwarding, firewalls, etc.

### How Routing Decisions are Made

```
        [Router A]
       /           \
[PC A]               [PC B]
       \           /
        [Router B]
```

Factors that determine the best path:

| Factor | Question |
|---|---|
| **Shortest path** | Fewest devices to pass through? |
| **Most reliable** | Least packet loss history? |
| **Fastest medium** | Fibre vs copper cable? |

---
---

## 5. Switches

### What is a Switch?
A switch **connects multiple devices** within a network (3 to 63 devices) using ethernet cables.

### Layer 2 Switch
- Operates at **Data Link layer**
- Forwards **frames** using **MAC addresses**
- Solely responsible for getting frames to the right device

```
[PC A] ─┐
[PC B] ─┼──► [Layer 2 Switch] ──► Sends frames via MAC address
[PC C] ─┘
```

### Layer 3 Switch
- More sophisticated — can do what Layer 2 does **plus** some router functions
- Forwards **frames** via MAC address AND routes **packets** via IP address

```
[PC A: 192.168.1.x] ─┐
                      ├──► [Layer 3 Switch] ──► Routes between IP ranges
[PC B: 192.168.2.x] ─┘
```

### Switch Comparison

| | Layer 2 Switch | Layer 3 Switch |
|---|---|---|
| **OSI Layer** | Layer 2 | Layer 3 |
| **Uses** | MAC addresses | MAC + IP addresses |
| **Can Route?** | No | Yes |
| **Complexity** | Simple | More sophisticated |

---

### VLAN — Virtual Local Area Network
VLANs **virtually split** devices within a network into separate groups — even if they share the same physical switch.

```
[Switch]
├── VLAN 1: Accounting devices
├── VLAN 2: HR devices
└── VLAN 3: Guest devices
```

- All devices can still share internet access
- But they are **treated separately** with their own rules
- Provides **security through segregation**

---
---



