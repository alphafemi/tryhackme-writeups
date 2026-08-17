# Incident Response Fundamentals


---

Devices constantly run interactive processes (things you actively do, like playing a game) and non-interactive processes (background tasks). Both generate a huge volume of **events** — logged records of what a process did. Security solutions ingest these events as **logs** and analyze them to surface potentially harmful activity.

---

### Alerts: False Positives vs. True Positives

When a security solution flags an event (or group of events) as potentially harmful, it raises an **alert**. The security team then analyzes each alert:

| Type | Definition | Example |
|---|---|---|
| **False Positive** | Looks dangerous but isn't | A large data transfer to an external IP triggers an alert — investigation shows it was a routine cloud backup. |
| **True Positive** | Genuinely harmful | A phishing attempt alert is confirmed to be a real phishing email sent to compromise the user. |

A **true positive** alert is often escalated to the status of an **incident**.

### Incident Severity

Once classified as an incident, it's assigned a severity level to prioritize response when multiple incidents occur simultaneously:

```
Critical > High > Medium > Low
```

Critical-severity incidents always get the highest priority.

---

### Types of Security Incidents

Not every harmful digital event is "hacking" — incidents fall into distinct categories, and can occur independently or together against the same victim.

| Incident Type | Description |
|---|---|
| **Malware Infections** | Malicious programs (text, document, executable, etc.) that damage a system, network, or application. The most common incident type. |
| **Security Breaches** | Unauthorized access to confidential data — critical since businesses depend on data staying accessible only to authorized personnel. |
| **Data Leaks** | Exposure of confidential information to unauthorized parties — used for reputational damage or extortion. Unlike breaches, leaks can also happen unintentionally (human error, misconfiguration). |
| **Insider Attacks** | Attacks originating from within the organization (e.g., a disgruntled employee infecting the network via USB). Especially dangerous because insiders already have elevated access. |
| **Denial of Service (DoS)** | Flooding a system/network/application with false requests to exhaust resources and deny access to legitimate users — an attack on the *availability* pillar of security. |

> **Impact is relative:** The same incident type can be catastrophic for one organization and trivial for another. A data leak might not matter to a company whose data has no external value, while a DoS attack on its primary revenue-generating website could be devastating.

---

### Incident Response Frameworks

Structured frameworks provide a generic, repeatable approach for handling any incident. The two most widely used are from **SANS** and **NIST**.

#### SANS Framework — "PICERL" (6 Phases)

| Phase | Description | Example |
|---|---|---|
| **Preparation** | Build the resources needed to handle incidents: IR teams, an IR plan, and deployed security solutions. | Running phishing-awareness training for employees. |
| **Identification** | Detect abnormal behavior that may indicate an incident, using security solutions and monitoring techniques. | Noticing unusual outbound data volume from a host, traced to a phishing attachment. |
| **Containment** | Minimize the impact of the attack — e.g., isolating the affected machine, disabling compromised accounts. | Isolating the compromised host from the network to stop lateral movement. |
| **Eradication** | Remove the threat entirely from the environment. | Running a deep malware scan to remove malicious software. |
| **Recovery** | Restore affected systems from backup or rebuild them, then test before returning to production. | Reconfiguring the host and restoring exfiltrated data from backup. |
| **Lessons Learned** | Identify and document gaps in detection/analysis to improve future response. | Holding a post-incident review to analyze root cause and strengthen defenses. |

#### NIST Framework (4 Phases)

NIST condenses the same lifecycle into four phases (commonly: Preparation → Detection & Analysis → Containment, Eradication & Recovery → Post-Incident Activity), mapping closely to the SANS model above.

| SANS (6 phases) | NIST (4 phases) |
|---|---|
| Preparation | Preparation |
| Identification | Detection & Analysis |
| Containment, Eradication, Recovery | Containment, Eradication & Recovery |
| Lessons Learned | Post-Incident Activity |

---

### The Incident Response Plan

Organizations formalize their chosen framework into an **Incident Response Plan** — a document approved by senior management that governs procedures before, during, and after an incident. Key components include:

- Roles and Responsibilities
- Incident Response methodology
- Communication plan with stakeholders (including law enforcement)
- Escalation path

---

### Detection & Analysis Tooling

Manually spotting abnormal behavior at scale isn't feasible. Security solutions automate detection — and some also handle response:

| Solution | Role |
|---|---|
| **SIEM** | Centralizes and correlates logs from many sources to identify incidents. |
| **AV** (Antivirus) | Detects known malicious programs via regular system scans. |
| **EDR** | Deployed per-endpoint; detects advanced threats and can also **contain and eradicate** them. |

---

### Playbooks vs. Runbooks

| Term | Definition |
|---|---|
| **Playbook** | A guideline/checklist of steps for responding to a specific incident type — the "what to do." |
| **Runbook** | The detailed, step-by-step execution instructions for carrying out those steps — the "how to do it," which can vary based on available resources. |

#### Example Playbook: Phishing Email

1. Notify all stakeholders of the phishing email incident
2. Determine if the email is malicious via header and body analysis
3. Analyze any attachments included with the email
4. Determine whether anyone opened the attachment(s)
5. Isolate infected systems from the network
6. Block the sender

---

### Incident Response Quick Reference

| Concept | Summary |
|---|---|
| Alert outcomes | False Positive vs. True Positive |
| True positive → | Incident |
| Severity levels | Low → Medium → High → Critical |
| 5 Incident types | Malware, Security Breach, Data Leak, Insider Attack, DoS |
| SANS phases (PICERL) | Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned |
| NIST phases | Preparation, Detection & Analysis, Containment/Eradication/Recovery, Post-Incident Activity |
| Governing document | Incident Response Plan |
| Detection/response tools | SIEM, AV, EDR |
| Guidance docs | Playbook (what to do) vs. Runbook (how to do it) |

---

## Summary

- An alert is classified as either a **false positive** (looks harmful, isn't) or a **true positive** (genuinely harmful) — true positives are typically escalated to **incidents** and assigned a severity of Low, Medium, High, or Critical.
- Security incidents generally fall into five categories: **malware infections, security breaches, data leaks, insider attacks,** and **denial of service (DoS)** — the same incident type can carry very different real-world impact depending on the organization.
- The **SANS "PICERL" framework** (Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned) and the **NIST 4-phase framework** provide structured, repeatable approaches to incident handling and map closely onto each other.
- An organization's **Incident Response Plan** formalizes roles, methodology, communication, and escalation procedures, approved by senior management.
- **SIEM, AV,** and **EDR** are the core tooling for detecting (and, in EDR's case, also containing/eradicating) incidents, while **playbooks** (what to do) and **runbooks** (how to do it) guide the team's actual response.

## Related Documents

- [54. Security Operations Center (SOC) Fundamentals](./54-security-operations-center-soc-fundamentals.md)
- [55. Digital Forensics Fundamentals](./55-digital-forensics-fundamentals.md)
- [57. Log Analysis Fundamentals](./57-log-analysis-fundamentals.md)
