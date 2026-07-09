
This section explores the core protocols that handle automatic network configuration, address resolution, diagnostics, routing, and address translation — explaining what actually happens when a device silently joins a new network.
 
---
 
### DHCP — Dynamic Host Configuration Protocol
 
When a device joins a network, it needs at minimum three things configured:
 
| Configuration | Purpose |
|---|---|
| IP address + subnet mask | Identifies the device on the network |
| Default gateway (router) | Routes packets to other networks |
| DNS server | Resolves domain names to IP addresses |
 
**Manual configuration** suits servers (static, known addresses). **DHCP automates** this for mobile and end-user devices, preventing address conflicts and eliminating manual intervention.
 
#### Technical Details
 
| Detail | Value |
|---|---|
| Protocol layer | Application (runs over UDP) |
| Server port | **UDP 67** |
| Client port | **UDP 68** |
 
#### The DORA Process
 
```
Client                                  DHCP Server
  │── DHCP Discover (broadcast) ────────▶│
  │                                       │ (server selects an available IP)
  │◀── DHCP Offer ─────────────────────────│
  │── DHCP Request (broadcast) ────────▶│
  │                                       │ (server confirms assignment)
  │◀── DHCP ACK ────────────────────────────│
  │ (client now has full network config)  │
```
 
| Step | Message | Description |
|---|---|---|
| **D** | DHCP Discover | Client broadcasts from `0.0.0.0` to `255.255.255.255` — no IP yet |
| **O** | DHCP Offer | Server offers an available IP to the client's MAC address |
| **R** | DHCP Request | Client broadcasts acceptance of the offered address |
| **A** | DHCP Acknowledge | Server confirms the lease — client now owns the IP |
 
#### Packet-Level Observation
 
```
1  0.000000  0.0.0.0 → 255.255.255.255   DHCP Discover  (src MAC only — no IP yet)
2  0.013904  192.168.66.1 → 192.168.66.133  DHCP Offer
3  4.115318  0.0.0.0 → 255.255.255.255   DHCP Request   (IP not yet in use)
4  4.228117  192.168.66.1 → 192.168.66.133  DHCP ACK
```
 
> In steps 1 and 3, the client sends from `0.0.0.0` (no IP assigned yet) to the broadcast address `255.255.255.255`, and the link-layer destination is the broadcast MAC `ff:ff:ff:ff:ff:ff`.
 
Upon completing DORA, the client receives:
- A **leased IP address** for local and internet access
- The **default gateway** for routing packets off-network
- A **DNS server address** for domain name resolution
---
 
### ARP — Address Resolution Protocol (Packet Level)
 
Devices on the same Ethernet/WiFi network communicate using **MAC addresses** at Layer 2, but applications and routing decisions use **IP addresses** at Layer 3. ARP bridges this gap.
 
#### When ARP is Needed
 
```
Host A wants to send a packet to IP 192.168.66.1
→ Knows the IP, but needs the MAC to build an Ethernet frame
→ Sends an ARP Request to the broadcast MAC: ff:ff:ff:ff:ff:ff
→ Host with that IP replies with its MAC address
→ Communication can now proceed at Layer 2
```
 
#### ARP Exchange
 
```bash
# tshark output
1  cc:5e:f8:02:21:a7 → ff:ff:ff:ff:ff:ff  ARP  Who has 192.168.66.1? Tell 192.168.66.89
2  44:df:65:d8:fe:6c → cc:5e:f8:02:21:a7  ARP  192.168.66.1 is at 44:df:65:d8:fe:6c
 
# tcpdump output (equivalent)
ARP Request  who-has 192.168.66.1  tell 192.168.66.89
ARP Reply    192.168.66.1 is-at 44:df:65:d8:fe:6c
```
 
#### ARP Encapsulation
 
> ARP packets are **not** encapsulated within IP or UDP — they are encapsulated **directly within Ethernet frames**, operating between Layers 2 and 3.
 
| Property | Detail |
|---|---|
| ARP Request | Sent to broadcast MAC (`ff:ff:ff:ff:ff:ff`) — all hosts on the segment receive it |
| ARP Reply | Sent directly (unicast) back to the requesting MAC |
| Layer classification | Layer 2 (MAC-based), but supporting Layer 3 (IP) operations |
| Result | Sender caches the IP-to-MAC mapping in its **ARP cache** for future use |
 
---
 
### ICMP — Internet Control Message Protocol
 
ICMP is used primarily for **network diagnostics and error reporting**. Two foundational network tools are built on it.
 
#### `ping` — Connectivity Testing
 
Sends **ICMP Echo Request (Type 8)** packets; the target responds with **ICMP Echo Reply (Type 0)**.
 
```bash
ping 192.168.11.1 -c 4
```
 
```
64 bytes from 192.168.11.1: icmp_seq=1 ttl=63 time=11.2 ms
64 bytes from 192.168.11.1: icmp_seq=2 ttl=63 time=3.81 ms
64 bytes from 192.168.11.1: icmp_seq=3 ttl=63 time=3.99 ms
64 bytes from 192.168.11.1: icmp_seq=4 ttl=63 time=23.4 ms
 
4 packets transmitted, 4 received, 0% packet loss
rtt min/avg/max/mdev = 3.805/10.596/23.366/7.956 ms
```
 
| Flag | Purpose |
|---|---|
| `-c <n>` | Stop after sending `n` packets (Linux) |
 
> Absence of a reply can mean the host is offline, or that a **firewall** is blocking ICMP along the path — not necessarily that the host doesn't exist.
 
#### `traceroute` / `tracert` — Route Discovery
 
Exploits the **TTL (Time to Live)** field in IP packets to map each hop along the route:
 
1. Sends packets with TTL=1 — first router decrements to 0, drops packet, returns **ICMP Time Exceeded (Type 11)**
2. Sends with TTL=2 — second router responds
3. Continues until the destination is reached
```bash
traceroute example.com
```
 
```
1  _gateway (192.168.66.1)        4.4 ms
2  192.168.11.1                   5.8 ms
3  100.104.0.1                   11.1 ms
4  10.149.1.45                    6.2 ms
5  * * *                              (no response — firewall filtering ICMP)
...
16 93.184.215.14                140.6 ms
```
 
| Output | Meaning |
|---|---|
| IP/hostname with RTT | Router responded — hop identified |
| `* * *` | Router dropped the packet without sending ICMP Time Exceeded — firewall or configured to not respond |
 
> The route may change on subsequent runs due to dynamic routing decisions.
 
| Tool | OS | Command |
|---|---|---|
| `traceroute` | Linux/Unix | `traceroute <target>` |
| `tracert` | Windows | `tracert <target>` |
 
---
 
### Routing Protocols
 
Routers need to **share topology information** with each other to determine optimal packet paths across the internet's millions of interconnected networks. Routing protocols automate this.
 
| Protocol | Full Name | Type | Description |
|---|---|---|---|
| **OSPF** | Open Shortest Path First | Link-state | Routers share complete network topology maps; each calculates the most efficient path independently |
| **EIGRP** | Enhanced Interior Gateway Routing Protocol | Hybrid | Cisco proprietary; shares reachable networks and associated costs (bandwidth, delay); selects most efficient paths |
| **BGP** | Border Gateway Protocol | Path-vector | The primary internet routing protocol; enables different ISPs and networks to exchange routing information and determine cross-network paths |
| **RIP** | Routing Information Protocol | Distance-vector | Simple, used in small networks; routes chosen by fewest hops; limited scalability |
 
> **BGP** is the protocol that actually makes the internet work at scale — it connects the routing tables of different autonomous systems (ISPs, cloud providers, enterprises) together.
 
---
 
### NAT — Network Address Translation
 
**Problem:** IPv4 supports ~4 billion addresses — insufficient for billions of connected devices.
 
**NAT solution:** Allow **many private IP addresses** to share a **single public IP address**, translating between the two at the router.
 
```
[Private Network]                     [Internet]
192.168.0.100  ─┐
192.168.0.101  ─┼──▶ [NAT Router] ──▶ 212.3.4.5 (single public IP)
192.168.0.102  ─┘     (maintains
                        translation
                           table)
```
 
#### How NAT Works
 
The router maintains a **translation table** mapping each internal connection to a unique external port:
 
| Internal IP | Internal Port | External IP | External Port |
|---|---|---|---|
| 192.168.0.129 | 15401 | 212.3.4.5 | 19273 |
| 192.168.0.130 | 22841 | 212.3.4.5 | 19274 |
 
- Outgoing: router rewrites source IP/port from private → public before forwarding
- Incoming: router reverses the translation — rewrites destination IP/port from public → private
- The remote server only ever sees the **public IP** — the private network is invisible from the internet
#### NAT vs. Routing
 
| | Routing | NAT |
|---|---|---|
| Address visibility | Both endpoints see each other's real IPs | Remote host sees only the NAT router's public IP |
| State tracking | Stateless (each packet routed independently) | Stateful (router tracks ongoing connections in a translation table) |
| IP conservation | No conservation — each device needs a unique routable IP | Significant conservation — many private devices share one public IP |
 
---
 
### Protocol Quick Reference
 
| Protocol | Layer | Transport | Port(s) | Purpose |
|---|---|---|---|---|
| **DHCP** | Application | UDP | Server: 67 / Client: 68 | Automatic network configuration |
| **ARP** | Layer 2/3 | — (direct Ethernet) | — | IP-to-MAC address resolution |
| **ICMP** | Network (3) | — (direct IP) | — | Diagnostics (`ping`, `traceroute`) and error reporting |
| **OSPF** | Network (3) | — | — | Link-state routing protocol |
| **BGP** | Application | TCP | 179 | Inter-network (internet-scale) routing |
| **RIP** | Application | UDP | 520 | Simple distance-vector routing |
 
---
