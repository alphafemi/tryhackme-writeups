# The Unified Kill Chain (UKC) & Threat Modelling



A **"Kill Chain"** (a military-origin term) describes the stages an attacker moves through to intrude on a target. Understanding an attacker's kill chain lets defenders pre-emptively protect systems or disrupt an attack in progress.

---

### Threat Modelling

**Threat modelling** is a structured process for improving system security by identifying risk. It generally involves:

1. **Identifying** which systems/applications need securing and their business function (e.g., is it critical to operations, or does it hold sensitive data like payment info?)
2. **Assessing** what vulnerabilities/weaknesses exist and how they could be exploited
3. **Creating a plan** to secure the systems against identified vulnerabilities
4. **Implementing policies** to prevent recurrence (e.g., an SDLC process, or phishing awareness training)

Threat modelling produces a high-level overview of an organization's IT assets and a plan to resolve their vulnerabilities. Frameworks like **STRIDE**, **DREAD**, and **CVSS** are commonly used for this purpose. The UKC supports threat modelling by helping identify potential attack surfaces and how they might be exploited.

---

### What Is the Unified Kill Chain?

Published by **Paul Pols in 2017**, the **Unified Kill Chain (UKC)** complements (rather than competes with) other kill chain frameworks like Lockheed Martin's Cyber Kill Chain and MITRE ATT&CK. It defines **18 phases** across an entire attack lifecycle — reconnaissance through exfiltration and attacker motive.

### UKC vs. Traditional Frameworks

| UKC Advantage | How Other Frameworks Compare |
|---|---|
| Modern (2017, updated 2022) | MITRE ATT&CK dates to 2013 — a very different threat landscape |
| Extremely detailed (18 phases) | Other frameworks typically cover only a handful of phases |
| Covers the entire attack lifecycle, including attacker motivation | Other frameworks cover a more limited scope |
| Reflects realistic attack behavior — phases recur (e.g., re-entering recon after pivoting to a new system) | Other frameworks assume a mostly linear, one-directional progression |

The UKC also explicitly accounts for **persistence** mechanisms and the reality that attackers combine multiple tactics throughout an attack, rather than following phases in strict sequence.

---

### UKC Phases — "In" (Initial Foothold)

#### Reconnaissance (MITRE TA0043)

Gathering target information via passive/active means, used throughout later UKC phases. Typical findings:

- Systems/services running on the target (informs weaponization/exploitation)
- Employee/contact lists for social engineering or phishing
- Potential credentials useful for pivoting or initial access
- Network topology and other systems available for pivoting

#### Weaponization (MITRE TA0001)

Setting up attack infrastructure — e.g., a C2 server, or a system to catch reverse shells and deliver payloads.

#### Social Engineering (MITRE TA0001)

Manipulating employees into aiding the attack:

- Convincing a user to open a malicious attachment
- Impersonating a login page to harvest credentials
- Impersonating a person (e.g., a utility engineer, or a caller requesting a password reset) to gain physical or logical access

#### Exploitation (MITRE TA0002)

Abusing vulnerabilities to achieve **code execution**:

- Uploading/executing a reverse shell via a web application
- Interfering with an automated script to trigger code execution
- Exploiting a web app vulnerability to execute code on its host system

#### Persistence (MITRE TA0003)

Maintaining access to a system already compromised:

- Creating a service allowing regain of access
- Adding the system to a C2 server for remote command execution
- Backdoors triggered by specific conditions (e.g., a reverse shell firing when an admin logs in)

#### Defence Evasion (MITRE TA0005)

Techniques to bypass defensive controls — WAFs, network firewalls, antivirus, IDS. Analyzing this phase is especially valuable for shaping incident response and improving future defenses.

#### Command & Control (MITRE TA0011)

Combines earlier weaponization work to establish ongoing communication with the target, enabling:

- Remote command execution
- Data/credential theft
- Using the controlled host to pivot further into the network

---

### UKC Phases — "Through" (Network Propagation)

Once an initial foothold is established, if defensive controls block further progress, the attacker seeks additional access/privileges — using the compromised system as a **pivot point** to explore the internal network.

#### Pivoting (MITRE TA0008)

The compromised system becomes a staging site and tunnel between the attacker's operations and the victim's internal network — later used to distribute malware and backdoors further inside.

#### Discovery (MITRE TA0007)

Building a knowledge base of the internal environment: active user accounts, granted permissions, installed applications/software, browser activity, files/directories/network shares, and system configurations.

#### Privilege Escalation (MITRE TA0004)

Using discovery findings (vulnerabilities, misconfigurations) to gain higher-level access:

- SYSTEM/ROOT
- Local Administrator
- Admin-like user account
- User account with specific elevated functions

#### Execution (MITRE TA0002)

Deploying malicious code from the pivot system — remote trojans, C2 scripts, malicious links, scheduled tasks — to maintain a recurring, persistent presence.

#### Credential Access (MITRE TA0006)

Stealing account names/passwords (keylogging, credential dumping) — working alongside privilege escalation, and helping the attacker blend in using **legitimate credentials**.

#### Lateral Movement (MITRE TA0008)

Using stolen credentials and elevated privileges to move to other targeted systems toward the primary objective — the stealthier, the better.

---

### UKC Phases — "Out" (Achieving the Objective)

The final stretch, where the attacker has access to critical assets and works to compromise the **CIA triad** (Confidentiality, Integrity, Availability).

#### Collection (MITRE TA0009)

Gathering valuable data of interest — drives, browsers, audio, video, email — compromising **confidentiality** and setting up the next phase.

#### Exfiltration (MITRE TA0010)

Stealing the collected data, typically encrypted/compressed to avoid detection, often leveraging the C2 channel/tunnel established earlier.

#### Impact (MITRE TA0040)

Compromising **integrity and availability** — manipulating, interrupting, or destroying assets to disrupt business operations. Examples: removing account access, disk wipes, ransomware encryption, defacement, or DoS attacks.

#### Objectives

The attacker's ultimate strategic goal — e.g., financially motivated ransomware demands, or reputational damage via public release of confidential data.

---

### Unified Kill Chain Quick Reference

| Stage Group | Phases |
|---|---|
| **In** (Initial Foothold) | Reconnaissance, Weaponization, Social Engineering, Exploitation, Persistence, Defence Evasion, Command & Control |
| **Through** (Network Propagation) | Pivoting, Discovery, Privilege Escalation, Execution, Credential Access, Lateral Movement |
| **Out** (Achieving Objectives) | Collection, Exfiltration, Impact, Objectives |
| UKC published | 2017 by Paul Pols (updated 2022) |
| Total phases | 18 |
| Complements | Lockheed Martin Cyber Kill Chain, MITRE ATT&CK |
| Key advantage | Reflects realistic, non-linear, recurring attack behavior |
| Threat modelling frameworks | STRIDE, DREAD, CVSS |

---

## Summary

- **Threat modelling** is a structured process — identify critical systems, assess their vulnerabilities, plan mitigations, and implement preventive policies — commonly supported by frameworks like **STRIDE, DREAD,** and **CVSS**.
- The **Unified Kill Chain (UKC)**, published by Paul Pols in 2017 (updated 2022), maps 18 phases across an entire attack lifecycle, offering more detail and realism (non-linear, recurring phases) than the Cyber Kill Chain or MITRE ATT&CK alone — the three frameworks are meant to complement, not replace, each other.
- The **"In"** stage (Reconnaissance, Weaponization, Social Engineering, Exploitation, Persistence, Defence Evasion, Command & Control) covers gaining and holding an initial foothold.
- The **"Through"** stage (Pivoting, Discovery, Privilege Escalation, Execution, Credential Access, Lateral Movement) covers expanding access across the internal network once further outward progress is blocked.
- The **"Out"** stage (Collection, Exfiltration, Impact, Objectives) covers achieving the attacker's actual goal — compromising the CIA triad and ultimately serving a strategic motive like financial gain or reputational damage.

