# Offensive Security — Overview

> **Topic:** Introduction to Offensive Security  
> Covers: What offensive security is, key concepts, common techniques, roles, and how it relates to defensive security

---

## Table of Contents

1. [What Is Offensive Security?](#what-is-offensive-security)
2. [The Offensive Security Mindset](#the-offensive-security-mindset)
3. [Key Concepts](#key-concepts)
   - [Penetration Testing](#penetration-testing)
   - [Red Teaming](#red-teaming)
   - [Vulnerability Research](#vulnerability-research)
   - [Social Engineering](#social-engineering)
4. [Common Offensive Techniques](#common-offensive-techniques)
5. [Offensive Security Roles](#offensive-security-roles)
6. [Offensive vs. Defensive Security](#offensive-vs-defensive-security)
7. [Rules of Engagement & Ethics](#rules-of-engagement--ethics)
8. [Key Takeaways](#key-takeaways)

---

## What Is Offensive Security?

Offensive security is the practice of **thinking and acting like an attacker** — finding weaknesses in systems, networks, and applications before a real attacker does.

Rather than waiting for an attack to happen and then responding, offensive security teams actively **hunt for vulnerabilities** and attempt to exploit them in a controlled, authorised way. The goal is to discover and fix weaknesses before a malicious actor can take advantage of them.

> **Simple analogy:** Hiring someone to try to break into your house before a burglar does — so you can fix the locks.

---

## The Offensive Security Mindset

Offensive security practitioners think like attackers. This means asking:

- **What can I see?** — Reconnaissance: what information is publicly available about the target?
- **What can I access?** — Scanning: what systems and services are exposed?
- **How can I get in?** — Exploitation: are there known vulnerabilities I can use?
- **What can I do once inside?** — Post-exploitation: can I move deeper, steal data, or maintain access?
- **How do I cover my tracks?** — Evasion: how would a real attacker avoid detection?

Understanding this thought process helps both attackers (in authorised testing) and defenders (knowing what to protect against).

---

## Key Concepts

### Penetration Testing

A **penetration test** (pen test) is an authorised, simulated attack on a system, network, or application to identify exploitable vulnerabilities.

**Phases of a penetration test:**

| Phase | Description |
|---|---|
| **Reconnaissance** | Gather information about the target (passive and active) |
| **Scanning & Enumeration** | Identify open ports, services, and potential entry points |
| **Exploitation** | Attempt to exploit discovered vulnerabilities |
| **Post-Exploitation** | Determine what access was gained; assess the impact |
| **Reporting** | Document findings, evidence, and recommended fixes |

**Types of pen testing by knowledge level:**

| Type | What the Tester Knows | Simulates |
|---|---|---|
| **Black box** | Nothing about the target | External attacker with no insider knowledge |
| **Grey box** | Some information (e.g., user account) | Attacker with limited access (e.g., compromised employee) |
| **White box** | Full knowledge (source code, architecture) | Thorough internal review or auditor |

---

### Red Teaming

A **red team** engagement goes further than a standard pen test. Rather than simply finding vulnerabilities, a red team simulates a **full, realistic attack campaign** against an organisation over an extended period.

- Red teams operate stealthily — the goal is to go undetected as long as possible
- They test not just technology, but also **people** (social engineering) and **processes** (incident response)
- A corresponding **blue team** (defenders) tries to detect and stop the red team
- A **purple team** exercise involves red and blue working together openly to share knowledge

---

### Vulnerability Research

Offensive security researchers actively look for **new, previously unknown vulnerabilities** in software and systems.

- **Zero-day vulnerability** — a flaw that is unknown to the vendor and has no patch yet
- Researchers responsibly disclose findings to vendors so patches can be released
- Understanding how vulnerabilities are found helps defenders prioritise what to protect

---

### Social Engineering

Not all attacks target technology — many target **people**. Social engineering manipulates individuals into revealing information or taking actions that compromise security.

**Common social engineering techniques:**

| Technique | Description |
|---|---|
| **Phishing** | Deceptive emails that trick users into clicking links or entering credentials |
| **Spear phishing** | Targeted phishing aimed at a specific person or organisation |
| **Vishing** | Voice phishing — phone calls impersonating trusted entities |
| **Pretexting** | Creating a fabricated scenario to extract information |
| **Baiting** | Leaving infected USB drives for victims to find and plug in |

> **Key insight:** The human element is often the weakest link. Technical defences can be bypassed entirely if an attacker can trick a person into providing access.

---

## Common Offensive Techniques

| Technique | Description |
|---|---|
| **Port scanning** | Identifying open ports and running services (e.g., using Nmap) |
| **Exploitation** | Using known vulnerabilities to gain unauthorised access |
| **Password attacks** | Brute forcing, credential stuffing, or cracking password hashes |
| **Privilege escalation** | Moving from limited access to admin/root level after initial compromise |
| **Lateral movement** | Moving from one compromised system to others on the same network |
| **Persistence** | Establishing backdoors to maintain access after the initial compromise |
| **Data exfiltration** | Extracting sensitive data from the compromised environment |
| **C2 (Command & Control)** | Maintaining communication with compromised systems to issue commands |

---

## Offensive Security Roles

### Penetration Tester

- Conducts authorised simulated attacks against systems and applications
- Produces detailed reports of findings with remediation recommendations
- Works across web applications, networks, mobile, and cloud environments

### Red Team Operator

- Simulates advanced, persistent threat actors over extended engagements
- Tests the full security posture — technology, people, and processes
- Operates stealthily to avoid detection by the blue team

### Bug Bounty Hunter

- Independently researches vulnerabilities in systems that have public bug bounty programmes
- Reports findings to the organisation in exchange for a financial reward
- Operates within a defined scope set by the organisation

### Vulnerability Researcher

- Discovers and analyses new vulnerabilities in software and hardware
- May work for vendors, security companies, or independently
- Responsible disclosure to vendors before public release is standard practice

---

## Offensive vs. Defensive Security

| Aspect | Offensive Security | Defensive Security |
|---|---|---|
| Goal | Find weaknesses before attackers do | Detect, prevent, and respond to attacks |
| Mindset | "How can I break in?" | "How do I stop someone breaking in?" |
| Activities | Pen testing, red teaming, exploitation | Monitoring, incident response, forensics |
| Timing | Proactive — before an attack | Reactive + proactive — during and before attacks |
| Key roles | Penetration tester, red team, bug bounty | SOC analyst, incident responder, forensics |
| Output | Vulnerability reports with remediation guidance | Incident reports, threat intelligence, patched systems |

> **Important:** Both are essential. Offensive security finds the gaps; defensive security closes them and responds when something gets through.

---

## Rules of Engagement & Ethics

Offensive security is only legitimate when properly **authorised**. Without written permission, the same actions used in a pen test are illegal under computer misuse laws in most countries.

**Key principles:**

- **Written authorisation** — always obtain explicit permission before testing
- **Defined scope** — know exactly what systems are in scope and which are off-limits
- **No collateral damage** — avoid disrupting systems or data outside the agreed scope
- **Responsible disclosure** — report vulnerabilities to the affected vendor before going public
- **Confidentiality** — findings from a pen test are sensitive; protect them accordingly

> **Remember:** The difference between a penetration tester and a criminal is a signed contract.

---

## Key Takeaways

| Topic | Key Point |
|---|---|
| Offensive security | Authorised, simulated attacks to find vulnerabilities before real attackers |
| Penetration testing | Structured attack simulation: recon → scan → exploit → report |
| Black / grey / white box | Different levels of prior knowledge given to the tester |
| Red teaming | Full attack simulation over extended period; tests tech, people, and process |
| Zero-day | Unknown vulnerability with no available patch |
| Social engineering | Attacks targeting people, not just technology; phishing is most common |
| Privilege escalation | Going from limited to admin access after initial compromise |
| Lateral movement | Moving through a network after gaining an initial foothold |
| Authorisation | Offensive techniques are legal only with written permission |
| Offensive + defensive | Both are needed — offensive finds gaps, defensive closes and monitors them |

---

> 📚 This document is part of a defensive security study series. Understanding offensive techniques is essential for defenders — you cannot protect against what you do not understand.
