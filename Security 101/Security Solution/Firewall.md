# Firewall Fundamentals


---

Just as a security guard checks who enters or leaves a building, a **firewall** inspects a network's or device's incoming and outgoing traffic, allowing or denying it based on configured rules. All traffic to or from a device/network passes through the firewall first.

---

### Types of Firewalls (by OSI Layer)

| Firewall Type | OSI Layer | Key Trait |
|---|---|---|
| **Stateless Firewall** | Layer 3–4 | Filters purely on predetermined rules, with **no memory** of prior connections — every packet is evaluated fresh. Fast, but can't apply context-aware policies (e.g., it won't automatically keep blocking a source just because earlier packets from it were denied). |
| **Stateful Firewall** | Layer 3–4 | Tracks connection history in a **state table**. Once a connection is allowed (or denied), subsequent packets from that connection are automatically handled without full re-inspection. |
| **Proxy Firewall** (Application-level Gateway) | Layer 7 | Acts as an intermediary between the private network and the internet, inspecting packet **content** and masking internal IPs for anonymity. Supports content filtering. |
| **Next-Generation Firewall (NGFW)** | Layer 3–7 | The most advanced type — deep packet inspection, built-in intrusion prevention, heuristic pattern analysis, and SSL/TLS decryption correlated with threat intelligence feeds. |

#### Characteristics Summary

| Firewall | Characteristics |
|---|---|
| **Stateless** | Basic filtering; no connection tracking; efficient for high-speed networks |
| **Stateful** | Recognizes traffic by pattern; supports complex rules; monitors connections |
| **Proxy** | Inspects packet contents; content filtering; application control; SSL/TLS decryption |
| **Next-Generation (NGFW)** | Advanced threat protection; intrusion prevention system; heuristic anomaly detection; SSL/TLS decryption |

---

### Firewall Rule Components

| Component | Description |
|---|---|
| **Source address** | The IP address originating the traffic |
| **Destination address** | The IP address receiving the traffic |
| **Port** | The port number involved |
| **Protocol** | The communication protocol (e.g., TCP) |
| **Action** | What happens to matching traffic |
| **Direction** | Whether the rule applies to inbound or outbound traffic |

### Rule Actions

| Action | Effect | Example Rule |
|---|---|---|
| **Allow** | Permits the defined traffic | `Allow 192.168.1.0/24 → Any, TCP/80, Outbound` (allow outgoing HTTP) |
| **Deny** | Blocks the defined traffic | `Deny Any → 192.168.1.0/24, TCP/22, Inbound` (block incoming SSH to a critical server) |
| **Forward** | Redirects traffic to another network segment (requires routing/gateway capability) | `Forward Any → 192.168.1.8, TCP/80, Inbound` (route incoming HTTP to a web server) |

### Rule Directionality

| Type | Applies To | Example |
|---|---|---|
| **Inbound Rules** | Incoming traffic | Allow incoming HTTP (port 80) to a web server |
| **Outbound Rules** | Outgoing traffic | Block outgoing SMTP (port 25) from everything except the mail server |
| **Forward Rules** | Traffic routed within the network | Forward incoming HTTP to an internal web server |

---

### Windows Defender Firewall

Windows' built-in firewall (open via **Windows Search → "Windows Defender Firewall"**) applies settings based on **Network Profiles**:

| Profile | Applies When | Typical Use |
|---|---|---|
| **Private** | Connected to a home/trusted network | Standard, more permissive settings |
| **Guest/Public** | Connected to an untrusted network (e.g., a coffee shop) | Stricter — often blocking all incoming connections while allowing only essential outbound traffic |

Windows determines the active profile automatically via **Network Location Awareness (NLA)**.

**Key dashboard options:**
- Allow/disallow specific apps per network profile
- Turn the firewall on/off (Microsoft recommends blocking connections instead of fully disabling it)
- Restore default settings

**Creating a custom rule** (example: block all outbound HTTP/HTTPS):

1. Open **Advanced Settings** from the main dashboard
2. Select **Outbound Rules → New Rule**
3. Choose **Custom** rule type
4. Apply to **All programs**
5. Protocol: **TCP**; Remote port: **Specific ports** → `80,443` (no spaces after commas)
6. Scope: leave local/remote IPs as-is
7. Action: **Block the connection**
8. Profile: leave all profiles checked
9. Name the rule and finish

Once applied, outbound web traffic (ports 80/443) is blocked — confirmed by testing a site and receiving a connection error.

---

### Linux Firewalls

Linux's firewall capability is built on the **Netfilter** framework (packet filtering, NAT, connection tracking), with several utilities built on top of it:

| Utility | Description |
|---|---|
| **iptables** | The most widely used Netfilter-based utility across distributions. |
| **nftables** | Successor to iptables, with enhanced filtering/NAT capabilities. |
| **firewalld** | Netfilter-based, with predefined rule sets and network zone configurations. |
| **ufw** (Uncomplicated Firewall) | A simplified, beginner-friendly interface that configures iptables under the hood. |

#### ufw — Common Commands

```bash
# Check firewall status
sudo ufw status

# Enable the firewall
sudo ufw enable

# Disable the firewall
sudo ufw disable

# Set default policy: allow all outgoing traffic
sudo ufw default allow outgoing

# Deny incoming SSH traffic (port 22/tcp)
sudo ufw deny 22/tcp

# List all active rules, numbered
sudo ufw status numbered

# Delete a rule by its number
sudo ufw delete 2
```

Example numbered rule list:

```
To                         Action      From
--                         ------      ----
[ 1] 22/tcp                DENY IN     Anywhere
[ 2] 22/tcp (v6)           DENY IN     Anywhere (v6)
```

---

### Firewall Quick Reference

| Concept | Summary |
|---|---|
| Stateless | L3–4, no connection memory, fast but basic |
| Stateful | L3–4, tracks connections via state table |
| Proxy | L7, inspects content, masks internal IPs |
| NGFW | L3–7, deep inspection + IPS + heuristics + SSL/TLS decryption |
| Rule components | Source, Destination, Port, Protocol, Action, Direction |
| Rule actions | Allow, Deny, Forward |
| Windows tool | Windows Defender Firewall (Private/Public profiles) |
| Linux framework | Netfilter |
| Linux utilities | iptables, nftables, firewalld, ufw |

---

## Summary

- A firewall inspects all traffic to/from a device or network and allows or denies it based on configured rules — the four types (**stateless, stateful, proxy, next-generation**) sit at increasing OSI layers and offer increasing inspection depth, at increasing resource cost.
- A firewall **rule** is built from source address, destination address, port, protocol, action, and direction — with **Allow, Deny,** and **Forward** as the three possible actions, applied to **inbound, outbound,** or **forward** traffic.
- **Windows Defender Firewall** applies different default behavior per **network profile** (Private vs. Public/Guest), determined automatically via Network Location Awareness, and supports fully custom inbound/outbound rules.
- Linux firewall capability is built on the **Netfilter** kernel framework, exposed through utilities like **iptables**, **nftables**, **firewalld**, and the beginner-friendly **ufw**, which wraps iptables with simplified commands.

## Related Documents

- [58. SIEM Fundamentals](./58-siem-fundamentals.md)
- [60. Intrusion Detection Systems (IDS) & Snort](./60-intrusion-detection-systems-ids-snort.md)
- [9. Port Forwarding, Firewalls, VPNs, Routers & Switches](./09-port-forwarding-firewalls-vpns-routers-switches.md)
