## LAN Topologies & Network Devices

### What is a Topology?
A **topology** is the **design or layout** of a network — how devices are arranged and connected to each other.

---

## The Three Main Topologies

### ⭐ 1. Star Topology
All devices connect to a **central device** (switch or hub).
<img width="560" height="475" alt="image" src="https://github.com/user-attachments/assets/e0c5c7fe-2b30-43e4-88fe-201395f2ea5b" />


```
    Device A
       |
Device D — [Switch/Hub] — Device B
       |
    Device C
```

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Most reliable | Most expensive |
| Easy to scale | More maintenance needed |
| Easy to add devices | Central device failure = whole network down |

> Most **commonly used** topology today.

---

### 🚌 2. Bus Topology
All devices connect along a **single backbone cable**.
<img width="1140" height="801" alt="image" src="https://github.com/user-attachments/assets/103fb53a-ee74-4ca2-a22f-4129721226d6" />

```
Device A — Device B — Device C — Device D
|_____________backbone cable_____________|
```

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Cheap & easy to set up | Gets slow/bottlenecked quickly |
| Less cabling needed | Hard to troubleshoot |
| | Single point of failure (the cable) |

---

### 🔄 3. Ring Topology
Devices are connected in a **loop**, passing data around until it reaches its destination.
<img width="878" height="801" alt="image" src="https://github.com/user-attachments/assets/b5b78635-170d-452a-b044-d3d446585613" />

```
Device A → Device B
   ↑              ↓
Device D ← Device C
```

| ✅ Advantages | ❌ Disadvantages |
|---|---|
| Easy to troubleshoot | Not efficient — data travels the whole loop |
| Less prone to bottlenecks | One break = entire network fails |
| Less cabling needed | |

---

## Quick Comparison

| | Star | Bus | Ring |
|---|---|---|---|
| **Cost** | High | Low | Medium |
| **Reliability** | High | Low | Medium |
| **Scalability** | High | Low | Low |
| **Troubleshooting** | Medium | Hard | Easy |

---

## Network Devices

### 🔀 Switch
- Connects **multiple devices** in a network via ethernet ports
- Available in 4, 8, 16, 24, 32, or 64 ports
- **Smart** — tracks which device is on which port and sends data **only to the intended target** (unlike a hub which broadcasts to everyone)
- Found in businesses, schools, larger networks

### 🔁 Hub (Lesser version of Switch)
- Sends incoming data to **every port** — inefficient and creates unnecessary traffic

  <img width="1409" height="801" alt="image" src="https://github.com/user-attachments/assets/b8fe52f9-4943-4190-9917-b282f537d354" />


## Subnetting — Splitting Networks into Smaller Pieces

### What is Subnetting?
**Subnetting** is the process of dividing a large network into smaller, more manageable **sub-networks (subnets)**. Think of it like **slicing a cake** — there's only so much to go around, and subnetting decides who gets what slice.

---

### Why is it Needed?
In a business, different departments need to be organised on a network, just like in real life:

```
         [Main Network]
        /       |       \
  Accounting  Finance   HR
  Subnet      Subnet    Subnet
```

Each department gets its **own subnet**, keeping traffic organised and secure.

---

### How it Works — The Subnet Mask
Just like an IP address, a **subnet mask** is made of **4 octets (32 bits)**, ranging from 0–255.

Example: `255.255.255.0`

Subnets use IP addresses in **three key ways**:

| Type | Purpose | Example |
|---|---|---|
| **Network Address** | Identifies the network itself | `192.168.1.0` |
| **Host Address** | Identifies a specific device on the subnet | `192.168.1.100` |
| **Default Gateway** | Device that sends data *outside* the network | `192.168.1.254` |

---

### Real-World Example — A Café
A café would have **two separate subnets**:

```
         [Internet]
              |
         [Router]
        /           \
[Staff Subnet]   [Public Subnet]
 Cash registers    Customer Wi-Fi
 Staff PCs         Hotspot
```

- **Staff subnet** — secure, for registers and internal devices
- **Public subnet** — open hotspot for customers
- Both can access the internet but are **isolated from each other**

---


### 🔗 Switches + Routers Together
- Can be connected to each other to create **redundancy**
- If one path fails, data takes another route = **no downtime**

---

## ARP — Address Resolution Protocol

### What is ARP?
**ARP** is the technology that allows devices to **link their MAC address to their IP address** on a network. It essentially answers the question:

> *"I know the IP address I want to talk to — but what is its MAC address?"*

---

### Why is it Needed?
Recall that devices have **two identifiers**:
- **IP Address** — logical, can change
- **MAC Address** — physical, permanent

To actually communicate, devices need **both**. ARP bridges the gap between them.

---

### How ARP Works — Step by Step

#### The ARP Cache
Every device keeps a **cache (a ledger/table)** storing the IP-to-MAC mappings of other devices it has communicated with.

```
ARP Cache Example:
┌─────────────────┬───────────────────┐
│   IP Address    │    MAC Address    │
├─────────────────┼───────────────────┤
│ 192.168.1.100   │ a4:c3:f0:85:ac:2d │
│ 192.168.1.254   │ b2:e1:d3:44:fc:11 │
└─────────────────┴───────────────────┘
```

---

#### The Two ARP Messages

**1. ARP Request** — Broadcasted to the entire network:
```
Device A → [Everyone on network]:
"Who has IP address 192.168.1.100?
 Please tell me your MAC address!"
```

**2. ARP Reply** — Only the matching device responds:
```
Device B → Device A:
"That's me! My MAC address is a4:c3:f0:85:ac:2d"
```

---

### Full ARP Flow

```
[Device A]                    [Network]                [Device B]
    |                             |                         |
    |——— ARP Request (Broadcast) →|→→→→→→→→→→→→→→→→→→→→→→→|
    |                             |                         |
    |                             |      (Only Device B     |
    |                             |       recognises its IP)|
    |                             |                         |
    |←←←←←←←←←←←←←←←←←←←←←←←←←|←—— ARP Reply ———————————|
    |                             |                         |
    | (Stores MAC in ARP cache)   |                         |
```

---

### Key Points Summary

| Concept | Detail |
|---|---|
| **ARP stands for** | Address Resolution Protocol |
| **Purpose** | Maps IP addresses to MAC addresses |
| **ARP Request** | Broadcast to the whole network |
| **ARP Reply** | Sent only by the device that owns the IP |
| **ARP Cache** | Local table storing known IP-to-MAC mappings |
| **Benefit** | Cached mappings avoid repeated ARP requests |

---

### Simple Analogy
ARP is like shouting in a room:
> *"Hey! Who is John?"* **(ARP Request)**
> Only John replies: *"That's me!"* **(ARP Reply)**
> You now remember John's face for next time **(ARP Cache)**

---

Would you like to continue with **DHCP**, the **OSI model**, or another networking topic?
## DHCP — Dynamic Host Configuration Protocol

### What is DHCP?
**DHCP** is a protocol that **automatically assigns IP addresses** to devices when they join a network — so you don't have to manually configure every device.

---

### Two Ways to Assign an IP Address

| Method | How | Best For |
|---|---|---|
| **Manual (Static)** | You type the IP address in yourself | Servers, printers, routers |
| **Automatic (DHCP)** | A DHCP server assigns it automatically | PCs, phones, everyday devices |

---

### How DHCP Works — The 4-Step Process

#### 1. 📡 DHCP Discover
The new device broadcasts to the network:
> *"Is there a DHCP server out there? I need an IP address!"*

#### 2. 📬 DHCP Offer
The DHCP server responds:
> *"Yes! I'm here. How about using IP address 192.168.1.100?"*

#### 3. ✅ DHCP Request
The device replies, confirming it wants that address:
> *"Yes please! I'd like to use 192.168.1.100!"*

#### 4. 🎉 DHCP ACK (Acknowledgement)
The DHCP server confirms and finalises:
> *"Done! 192.168.1.100 is yours. Welcome to the network!"*

---
<img width="636" height="870" alt="image" src="https://github.com/user-attachments/assets/1032fabf-5772-4f64-8c0b-25dec54ea394" />

### Visual Flow

```
[New Device]                          [DHCP Server]
     |                                      |
     |———— DHCP Discover (Broadcast) ———————►|
     |                                      |
     |◄——— DHCP Offer (Here's an IP) ————————|
     |                                      |
     |———— DHCP Request (I'll take it!) ————►|
     |                                      |
     |◄——— DHCP ACK (It's yours!) ————————————|
     |                                      |
  [Device now has an IP & joins network]
```

---

### Easy Memory Aid — **D.O.R.A**

| Letter | Step | Meaning |
|---|---|---|
| **D** | Discover | Device looks for a DHCP server |
| **O** | Offer | Server offers an IP address |
| **R** | Request | Device requests that IP |
| **A** | ACK | Server confirms and assigns it |

---

### Key Takeaway
> DHCP makes network management effortless — instead of manually assigning IP addresses to every device, the DHCP server handles it all **automatically**, making it the most common method used in homes, schools, and businesses.

Would you like to move on to the **OSI Model**, **DNS**, or another networking topic?
