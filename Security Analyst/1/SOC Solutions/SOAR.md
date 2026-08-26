# SOAR — Security Orchestration, Automation, and Response


---

As threats grow more complex, traditional SOCs — even with SIEM, EDR, firewalls, and threat intel — face mounting operational challenges. **SOAR** (Security Orchestration, Automation, and Response) is the tool built to address them.

---

### How Traditional SOCs Work

A SOC's core capabilities:

| Capability | Description | Example |
|---|---|---|
| **Monitoring and Detection** | Continuously scanning for suspicious activity, mainly via SIEM | Detecting numerous failed logins or a login from an unknown location |
| **Recovery and Remediation** | Acting as first responders — isolating hosts, removing malware, stopping malicious processes, via EDR/firewalls/IAM | Isolating an endpoint via EDR, blocking an IP on the firewall, disabling a user in IAM |
| **Threat Intelligence** | Continuous ingestion of the latest IOCs (IPs, hashes, domains) | Blocking a domain flagged by a threat intel feed |
| **Communication** | Coordinating with IT and management to address incidents | Opening a ticket for IT to verify a recently deployed patch |

### Challenges Faced by SOCs

| Challenge | Description |
|---|---|
| **Alert Fatigue** | Numerous tools generate a flood of alerts, many false positives or low-value, overwhelming analysts |
| **Too Many Disconnected Tools** | Security tools deployed without integration force analysts to manually cross-reference disparate logs/systems |
| **Manual Processes** | Investigation procedures often live only as undocumented "tribal knowledge," slowing response |
| **Talent Shortage** | Difficulty recruiting/scaling teams, compounding alert overload and extending response times |

---

### What Is SOAR?

**SOAR** unifies the tools a SOC relies on — SIEM, EDR, firewall, and others — into a single interface, eliminating constant tool-switching. It also provides **ticketing and case management** for structured incident tracking and resolution.

SOAR's strength rests on three core capabilities:

#### 1. Orchestration

Coordinates multiple security tools together through predefined workflows called **Playbooks** — step-by-step logic for investigating a given alert type.

> **Example — VPN Brute Force Playbook:**
> 1. Receive alert from SIEM
> 2. Query SIEM to check if the user normally uses this IP
> 3. Check threat intel platforms for the IP's reputation
> 4. Query SIEM for any successful login attempts
> 5. Escalate to containment actions if warranted

Playbooks are **dynamic** — the outcome of each step determines the next action (e.g., a playbook might stop early if the IP is normal and failed attempts are minimal).

#### 2. Automation

SOAR executes playbooks **without manual analyst clicks** at each step:

> **The same VPN Brute Force scenario, automated:**
> 1. SOAR receives the alert
> 2. Automatically queries historical login data
> 3. Automatically checks IP reputation via threat intel
> 4. If malicious, automatically disables the user in IAM
> 5. Automatically opens a ticket with full investigation details

This saves substantial analyst time, letting teams handle far more alerts without burning out.

#### 3. Response

Actions across multiple tools are taken from one unified interface — automatically, per the playbook (e.g., blocking an IP on the firewall, disabling a user in IAM, and opening a ticket, all in one automated flow).

---

### Do We Still Need SOC Analysts?

**Yes.** SOAR automates repetitive tasks but doesn't replace human judgment for complex investigations or business-context decisions. Analysts also **design the playbooks** SOAR runs. SOAR eases the burden — it doesn't eliminate the need for skilled analysts.

---

### Example Playbooks

#### Phishing Playbook

Phishing remains the most common attack vector, and investigating it manually (analyzing attachments/URLs, checking threat intel) is time-consuming. A typical playbook logic:

```
Receive alert: "Suspicious email received"
  → Create a ticket
  → Does the email contain a URL or attachment?
      ├─ No  → Notify the user, close
      └─ Yes → Branch based on URL vs. attachment analysis
               (further "if this, do this; else, do this" branching)
```

This branching, conditional structure is the essence of a playbook — automating the repetitive investigative steps while flagging points that may still need analyst review.

#### CVE Patching Playbook

Vulnerability management requires SOC teams to track newly disclosed **CVEs**, determine if they affect the environment, and patch accordingly — a process that becomes overwhelming given how frequently CVEs are released, risking a growing backlog of unpatched systems.

A CVE patching playbook typically:

1. Analyzes CVE details
2. Assesses the risk threshold
3. Creates a patching ticket
4. Tests the patch before pushing to production

> Both example playbooks automate most steps but still include analyst checkpoints at key decision points — reinforcing that SOAR reduces manual burden without eliminating the need for human judgment on critical calls.

---

### SOAR Quick Reference

| Concept | Summary |
|---|---|
| SOAR = | Security Orchestration, Automation, and Response |
| 4 SOC challenges it addresses | Alert fatigue, disconnected tools, manual processes, talent shortage |
| 3 core capabilities | Orchestration, Automation, Response |
| Playbook = | Predefined, conditional workflow for investigating a specific alert type |
| Orchestration | Coordinates multiple tools under one interface via playbooks |
| Automation | Executes playbook steps without manual analyst action |
| Response | Takes action across tools from a single unified interface |
| Analysts still needed for | Complex judgment calls, business context, playbook design |
| Example playbooks | Phishing triage, CVE patching |

---

## Summary

- Traditional SOCs run on four core capabilities — **monitoring/detection, recovery/remediation, threat intelligence,** and **communication** — but face four compounding challenges as they scale: **alert fatigue, disconnected tools, undocumented manual processes,** and a **talent shortage**.
- **SOAR** unifies a SOC's tools (SIEM, EDR, firewall, etc.) into one interface with built-in ticketing/case management, resting on three capabilities: **Orchestration** (coordinating tools via dynamic, conditional **Playbooks**), **Automation** (running those playbooks without manual clicks at each step), and **Response** (taking cross-tool action from a single interface).
- A worked VPN brute-force example shows the same playbook logic first as a manual/orchestrated flow, then fully automated — saving significant analyst time on repetitive investigation steps.
- SOAR does **not** replace SOC analysts — it removes repetitive burden while analysts still handle complex judgment calls and **design the playbooks themselves**, illustrated by two further examples: a **Phishing** triage playbook with URL/attachment branching, and a **CVE Patching** playbook that still includes analyst checkpoints before production deployment.

)
