# Security Operations Center (SOC) Fundamentals


---

A **Security Operations Center (SOC)** is a dedicated facility staffed by a specialized security team that continuously monitors an organization's network and resources to identify suspicious activity before it causes damage. The SOC team operates **24/7**.

The SOC's core mission rests on two functions: **Detection** and **Response**. Security solutions integrate an organization's entire network and systems into one centralized location, enabling continuous monitoring so incidents can be caught and handled quickly.

---

### Detection

| Detection Goal | Description |
|---|---|
| **Detect vulnerabilities** | A vulnerability is a weakness an attacker can exploit to act beyond their permission level (e.g., unpatched MS Windows machines). Not always the SOC's direct responsibility, but unpatched vulnerabilities weaken the whole company's security posture. |
| **Detect unauthorized activity** | E.g., an attacker using a stolen username/password to log in. Clues like geographic location can help flag this quickly. |
| **Detect policy violations** | Breaches of a company's security policy (rules/procedures protecting against threats and ensuring compliance) — e.g., downloading pirated media, or sending confidential files insecurely. |
| **Detect intrusions** | Unauthorized access to systems/networks — e.g., a successfully exploited web app, or a user infected after visiting a malicious site. |

### Response

| Response Goal | Description |
|---|---|
| **Support incident response** | Once an incident is detected, the SOC helps minimize its impact and perform root cause analysis, working alongside the incident response team. |

---

### The 3 Pillars of a SOC

A mature SOC is built on three coexisting pillars:

```
        People
          │
   ┌──────┴──────┐
Process       Technology
```

| Pillar | Role |
|---|---|
| **People** | Trained analysts and engineers who interpret alerts and drive response |
| **Process** | Structured procedures (triage, reporting, incident response) the team follows |
| **Technology** | Security solutions (SIEM, EDR, Firewall, etc.) that enable detection and response at scale |

> **Analogy:** Imagine a fire brigade with centralized software integrating every fire alarm in a city. A flood of simultaneous alerts turns out to mostly be smoke from cooking — without skilled people to triage them, effort and resources are wasted chasing noise. Automation alone isn't enough; people are what separate real threats from irrelevant alerts.

---

### Pillar 1: People — The SOC Team Hierarchy

```
                SOC Manager
                     │
            ┌────────┼────────┐
     Detection    Security   Analyst
     Engineer     Engineer   (L3)
                                │
                          Analyst (L2)
                                │
                          Analyst (L1)
```

| Role | Responsibility |
|---|---|
| **SOC Analyst (Level 1)** | First responders. Perform basic alert triage to determine if a detection is genuinely harmful, and report it through proper channels. |
| **SOC Analyst (Level 2)** | Handle deeper investigations, correlating data across multiple sources when L1 escalates a detection. |
| **SOC Analyst (Level 3)** | Experienced professionals who proactively hunt for threat indicators and lead incident response (containment, eradication, recovery) for critical detections. |
| **Security Engineer** | Deploys and configures the security solutions the analysts rely on. |
| **Detection Engineer** | Builds the detection logic/rules behind security solutions (sometimes a dedicated role, sometimes handled by L2/L3 analysts). |
| **SOC Manager** | Owns SOC processes and liaises with the organization's **CISO**, reporting on the team's current security posture. |

> **Note:** Team size and role granularity scale with the organization's size and criticality.

---

### Pillar 2: Process

#### Alert Triage — The 5 Ws

Every alert triage starts with answering the **5 Ws** to determine severity and priority.

**Example alert:** `Malware detected on Host: GEORGE PC`

| W | Answer |
|---|---|
| **What?** | A malicious file was detected on a host inside the organization's network. |
| **When?** | Detected at `13:20` on `June 5, 2024`. |
| **Where?** | In a directory on host `GEORGE PC`. |
| **Who?** | User `George`. |
| **Why?** | Investigation revealed the file was downloaded from a pirated software site — the user wanted free software. |

#### Reporting

Harmful alerts are escalated as **tickets** to higher-level analysts for timely resolution. A good report includes:

- A full walkthrough of the 5 Ws
- Thorough analysis of the activity
- Screenshots as supporting evidence

#### Incident Response & Forensics

Critical detections may trigger a full **incident response** process. Some cases also require **digital forensics** — analyzing artifacts from a system or network to determine the incident's root cause.

---

### Pillar 3: Technology

Manually detecting and responding to threats across every device and application would be unmanageable. Security solutions centralize that visibility and automate detection/response.

| Solution | Layer | Capability |
|---|---|---|
| **SIEM** (Security Information and Event Management) | Network-wide | Collects logs from many log sources, applies detection rules, and correlates events across sources. Modern SIEMs add user behavior analytics and ML-driven threat intelligence. **Detection only** — no response capability. |
| **EDR** (Endpoint Detection and Response) | Endpoint | Real-time and historical visibility into endpoint activity, with both deep investigation and automated response capability. |
| **Firewall** | Network perimeter | Filters incoming/outgoing traffic between internal and external networks; blocks suspicious traffic via its own detection rules before it reaches the internal network. |

Other common technologies include **Antivirus, EPP, IDS/IPS, XDR,** and **SOAR**. The right mix depends on the organization's threat surface and available resources.

---

### SOC Quick Reference

| Concept | Summary |
|---|---|
| SOC focus | Detect + Respond, 24/7 |
| 3 Pillars | People, Process, Technology |
| Analyst tiers | L1 (triage) → L2 (investigate) → L3 (hunt/respond) |
| Triage framework | The 5 Ws — What, When, Where, Who, Why |
| Detection-only tool | SIEM |
| Detection + response tool | EDR |
| Perimeter control | Firewall |

---

## Summary

- A **SOC** is a dedicated, 24/7 team that continuously monitors an organization's network to detect and respond to threats before they cause damage.
- A mature SOC rests on three pillars: **People** (analysts and engineers), **Process** (triage, reporting, incident response procedures), and **Technology** (SIEM, EDR, Firewall, and other tools).
- The **analyst hierarchy** runs from L1 (first-line alert triage) through L2 (deeper investigation) to L3 (proactive threat hunting and incident response leadership), supported by Security Engineers, Detection Engineers, and a SOC Manager.
- Alert triage follows the **5 Ws** framework — What, When, Where, Who, and Why — to determine an alert's severity and priority before escalation.
- **SIEM** provides network-wide detection only, **EDR** adds endpoint-level detection *and* response, and **Firewall** enforces perimeter-level traffic filtering — together with tools like IDS/IPS, XDR, and SOAR, they form the technology layer of SOC operations.

## Related Documents

- [42. Nmap — Network Scanning & Enumeration](./42-nmap-network-scanning-enumeration.md)
- [52. Gobuster — Directory, DNS & VHost Enumeration](./52-gobuster-directory-dns-vhost-enumeration.md)
- [55. Digital Forensics Fundamentals](./55-digital-forensics-fundamentals.md)
