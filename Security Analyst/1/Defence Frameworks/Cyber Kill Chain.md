# The Cyber Kill Chain


---

The **Cyber Kill Chain®**, established by Lockheed Martin in 2011, adapts a military attack-structure concept (target identification → decision to attack → destruction) into a 7-phase model of how cyberattacks unfold. Understanding it helps SOC analysts, threat hunters, and incident responders recognize intrusion attempts and adversary objectives — and identify where defensive gaps exist.

```
Reconnaissance → Weaponization → Delivery → Exploitation → Installation → C2 → Actions on Objectives
```

*(Illustrated throughout via a hypothetical attacker, "Megatron.")*

---

### 1. Reconnaissance

The research/planning phase — gathering infrastructure details, employee data, business processes, and exposed technologies. Usually **passive and undetected**. Poor recon leads to sloppy, easily-caught attacks; thorough recon enables highly targeted, believable attacks.

#### OSINT (Open-Source Intelligence)

Publicly available information gathered from:

- Search engines
- Print/online media
- Social media accounts
- Online forums and blogs
- Public record databases
- WHOIS and technical data

#### Recon Types

| Type | Description |
|---|---|
| **Passive Recon** | No direct target interaction — WHOIS lookups, social media scraping, breach data review |
| **Active Recon** | Direct contact with the target — social engineering, port scanning, banner grabbing, service probing |

#### Email Harvesting

Collecting email addresses from public/paid/free sources, often to enable phishing.

| Tool | Purpose |
|---|---|
| **theHarvester** | Gathers emails, names, subdomains, IPs, and URLs from multiple public sources |
| **Hunter.io** | Finds contact info associated with a domain |
| **OSINT Framework** | A categorized directory of OSINT tools |

---

### 2. Weaponization

Turning raw recon into an actionable attack tool — crafting malware/exploits into a **payload**. Attackers may build custom malware, use automated generation tools, or purchase ready-made payloads on the Dark Web.

**Key terminology:**

| Term | Definition |
|---|---|
| **Malware** | Software designed to damage, disrupt, or gain unauthorized access |
| **Exploit** | Code that takes advantage of a specific vulnerability or flaw |
| **Payload** | The malicious code the attacker actually runs on the target system |

**Common weaponization tactics:**

- Infected Office documents with malicious macros/VBA
- Malicious payloads/worms implanted on USB drives for public distribution
- Setting up C2 infrastructure for remote command execution
- Backdoors to bypass security mechanisms
- Convincing phishing templates or malicious OAuth-consent apps

---

### 3. Delivery

Transmitting the payload to the target environment.

| Method | Description |
|---|---|
| **Phishing Email** | Targeted (spear phishing) or mass emails with malicious links/attachments |
| **USB Drops** | Physical delivery in public places, sometimes disguised as legitimate branded gifts |
| **Watering Hole Attacks** | Compromising a site the target group frequents, redirecting to malicious content or triggering a drive-by download |

---

### 4. Exploitation

The moment the attacker's code actually executes on the target, leveraging a vulnerability.

| Technique | Description |
|---|---|
| **Malicious Macro Execution** | E.g., a phishing attachment executing ransomware on open |
| **Zero-Day Exploits** | Unknown, unpatched flaws — undetectable initially |
| **Known CVEs** | Exploiting unpatched, publicly disclosed vulnerabilities |

Post-access, the attacker may escalate privileges or move laterally.

**Signs of exploitation:**
- Unexpected process spawns
- Registry changes or newly created services
- Suspicious command-line arguments in system logs

---

### 5. Installation

Establishing **persistence** — ensuring continued access even if the initial connection is lost, the intrusion is detected, or the system is patched.

| Persistence Method | Description |
|---|---|
| **Web Shell** | A malicious script (PHP, ASP, JSP, etc.) on a web server maintaining access — often hard to distinguish from benign scripts |
| **Backdoor Tools** | E.g., Meterpreter (a Metasploit payload) providing an interactive remote shell |
| **Windows Service Creation/Modification** | MITRE ATT&CK technique **T1543.003** — using tools like `sc.exe` or `Reg` to create/modify services, sometimes masquerading as legitimate OS components |
| **Registry Run Keys / Startup Folder** | Adding entries that auto-execute the payload on user login (per-user or system-wide startup locations) |

**Anti-forensics:** Attackers may use **Timestomping** — modifying file timestamps (modify/access/create/change) to evade forensic detection and blend in with legitimate files.

---

### 6. Command and Control (C2)

Once persistence is established, the attacker opens a **C2 channel** to remotely control the compromised host. The infected host regularly communicates ("beacons") with the C2 server.

**Common C2 channels:**

| Channel | Notes |
|---|---|
| **HTTP (80) / HTTPS (443)** | Blends malicious traffic with legitimate web traffic, evading firewalls |
| **DNS** | Constant DNS requests to an attacker-controlled server — known as **DNS Tunneling** |
| *(Legacy)* **IRC** | Historically common, but easily detected by modern security tools today |

> Note: The C2 infrastructure owner may be the original adversary or another already-compromised host acting as relay infrastructure.

---

### 7. Actions on Objectives

The final phase — the attacker achieves their actual goals with hands-on-keyboard access:

- Collecting user credentials
- Privilege escalation (e.g., workstation access → domain admin via misconfiguration)
- Internal reconnaissance (probing internal software for further vulnerabilities)
- Lateral movement across the environment
- Collecting and exfiltrating sensitive data
- Deleting backups/shadow copies (Microsoft's snapshot/backup technology) to hinder recovery
- Overwriting or corrupting data

---

### Limitations of the Traditional Cyber Kill Chain

The model hasn't been updated since its **2011** creation, creating gaps:

- **Designed for perimeter/malware defense** — struggles against modern multi-TTP, blended attack strategies
- **Evadable threat intel** — attackers routinely rotate file hashes and IPs, requiring AI/advanced detection to catch subtle variations
- **Blind to Insider Threats** — per CISA, an insider threat is *"the potential for an insider to use their authorized access or understanding of an organization to harm that organization"* — a scenario the traditional model, focused on external malware delivery, doesn't address

**Recommendation:** Don't rely on the Cyber Kill Chain alone — complement it with **MITRE ATT&CK** and the **Unified Kill Chain** for a more comprehensive defensive approach.

---

### Cyber Kill Chain Quick Reference

| Phase | Core Activity |
|---|---|
| 1. Reconnaissance | Gather target information (often via OSINT) |
| 2. Weaponization | Build/acquire the malicious payload |
| 3. Delivery | Transmit the payload to the target |
| 4. Exploitation | Execute code by leveraging a vulnerability |
| 5. Installation | Establish persistence on the compromised system |
| 6. Command & Control (C2) | Establish remote control channel/beaconing |
| 7. Actions on Objectives | Achieve the attacker's actual goals (data theft, disruption, etc.) |
| Key limitation | Doesn't address insider threats or modern multi-TTP attacks |
| Complementary frameworks | MITRE ATT&CK, Unified Kill Chain |

---


- The **Cyber Kill Chain**, created by Lockheed Martin in 2011, models an attack across seven phases: **Reconnaissance** (OSINT-driven target research), **Weaponization** (building the payload), **Delivery** (phishing, USB drops, watering holes), **Exploitation** (code execution via a vulnerability), **Installation** (establishing persistence via web shells, backdoors, service/registry manipulation, and anti-forensic timestomping), **Command and Control** (beaconing over HTTP/S, DNS tunneling, or legacy IRC), and **Actions on Objectives** (credential theft, privilege escalation, lateral movement, exfiltration, and backup destruction).
- The model's core limitation is its age — unchanged since 2011, it was designed for perimeter/malware defense and struggles against modern multi-TTP blended attacks, evadable threat intel (rotating hashes/IPs), and is entirely **blind to insider threats**.
- The recommended approach is to pair the Cyber Kill Chain with complementary frameworks like **MITRE ATT&CK** and the **Unified Kill Chain** rather than relying on it in isolation.

