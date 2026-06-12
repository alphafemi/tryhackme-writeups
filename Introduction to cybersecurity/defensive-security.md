# Defensive Security — Overview

> **Topic:** Introduction to Defensive Security  
> Covers: Key defensive security areas, SOC team roles and responsibilities

---

## Table of Contents

1. [What Is Defensive Security?](#what-is-defensive-security)
2. [Key Areas of Defensive Security](#key-areas-of-defensive-security)
   - [Monitoring and Detecting](#monitoring-and-detecting)
   - [Incident Response](#incident-response)
   - [Threat Intelligence](#threat-intelligence)
   - [Vulnerability Management](#vulnerability-management)
   - [Investigation and Analysis](#investigation-and-analysis)
3. [The Defensive Security Team](#the-defensive-security-team)
4. [Key Takeaways](#key-takeaways)

---

## What Is Defensive Security?

Defensive security focuses on **protecting** an organisation's systems, networks, and data from attack. While offensive security looks for vulnerabilities to exploit (like a simulated attacker), defensive security works to detect, prevent, respond to, and recover from threats.

A defensive security team operates continuously — threats don't follow business hours.

---

## Key Areas of Defensive Security

### Monitoring and Detecting

Continuously observing network and system activity to identify suspicious behaviour before it becomes a serious incident.

**What this looks like in practice:**
- Watching for login attempts from unexpected geographic locations (e.g., an employee based in London logging in from another country)
- Alerting on unusual volumes of data being transferred
- Detecting connections to known malicious IP addresses

The goal is to spot anomalies early — the sooner a threat is detected, the less damage it can cause.

---

### Incident Response

When suspicious activity is confirmed, the incident response process begins immediately. The team comes together to contain and resolve the threat.

**The general incident response process:**

1. **Preparation** — having plans and tools ready before an incident occurs
2. **Identification** — confirming that an incident has occurred
3. **Containment** — limiting the spread and impact of the threat
4. **Eradication** — removing the threat from the environment
5. **Recovery** — restoring systems and business operations to normal
6. **Lessons Learned** — reviewing what happened to improve defences

> **Key point:** Speed matters in incident response. The faster the team contains a threat, the less damage to data, systems, and the organisation's reputation.

---

### Threat Intelligence

Gathering and analysing information about attackers — their tactics, techniques, procedures (TTPs), targets, and trends — to strengthen defences proactively.

**Examples:**
- Learning that an attacker group is actively targeting a specific software application used by your organisation
- Understanding the common methods used in phishing campaigns against your industry
- Tracking newly discovered vulnerabilities being exploited in the wild

**Why it matters:** Knowing your adversary allows you to prepare for likely attack methods rather than waiting to react.

---

### Vulnerability Management

Identifying and fixing security weaknesses in software and systems before attackers can exploit them.

**Approaches:**
- **Manual review** — security engineers manually assess configurations and code
- **Automated scanning tools** — tools scan systems and report known vulnerabilities
- **Prioritisation** — focus on the vulnerabilities attackers are most likely to target first

**Common output:** A list of vulnerabilities ranked by severity, with recommended patches or mitigations.

> **Key point:** Vulnerability management is preventative — fixing a flaw before it is exploited is far cheaper and less damaging than responding to a breach.

---

### Investigation and Analysis

Defensive security team members continuously analyse what is happening inside the organisation to separate **normal activity** from **suspicious behaviour**.

**What this involves:**
- Examining logs, alerts, and network traffic
- Identifying patterns that indicate an attack (even if no alert fired)
- Building a timeline of events to understand what happened
- Providing context — "is this unusual, or does this user always log in at 2am?"

Think of it as solving a puzzle: assembling small pieces of information into a clear picture of what is occurring or what occurred.

---

## The Defensive Security Team

A defensive security team is made up of specialised roles that each contribute a distinct function.

### Bob — SOC Analyst

**Role:** Security Operations Centre (SOC) Analyst

- Monitors events on the organisation's network and systems
- Identifies suspicious or expected behaviour in real time
- At the **frontline** of protecting the organisation
- Triages alerts — determines which events need escalation

> The SOC Analyst is often the first person to spot that something is wrong.

---

### Aaliyah — Incident Responder

**Role:** Incident Responder

- Investigates and responds to **active** security incidents
- Monitors attacker activity in real time and works to stop them
- Coordinates containment and eradication efforts
- Shares **lessons learned** after each incident to prevent future attacks

> The Incident Responder is the person who steps in when the alarm has been raised and the threat is live.

---

### Zoe — Security Engineer

**Role:** Security Engineer

- Develops and maintains the **tools and systems** that the defensive team relies on
- Builds monitoring platforms, detection rules, and alerting systems
- Ensures the team has the technical capability to detect and investigate threats

> Without the Security Engineer's work, the rest of the team would have no visibility into what is happening on the network.

---

### Bill — Digital Forensics

**Role:** Digital Forensics Analyst

- Called in after an incident to understand **exactly what happened**
- Gathers, preserves, and analyses **evidence** from networks and systems
- Works to uncover attacker identity, methods, and the full scope of an incident
- Ensures evidence is handled properly for potential legal proceedings

> Digital forensics is like detective work — finding and interpreting the digital footprints left behind by an attacker.

---

## Key Takeaways

| Topic | Key Point |
|---|---|
| Monitoring & Detection | Continuous observation; catch threats early |
| Incident Response | Contain → eradicate → recover → learn |
| Threat Intelligence | Know your attacker's methods before they attack you |
| Vulnerability Management | Fix weaknesses before attackers exploit them |
| Investigation & Analysis | Separate normal from suspicious; build the full picture |
| SOC Analyst | Frontline monitoring; first to spot issues |
| Incident Responder | Active threat response; real-time containment |
| Security Engineer | Builds and maintains team tools and systems |
| Digital Forensics | Post-incident evidence gathering and analysis |

---

> 📚 This document is part of a defensive security study series covering SOC operations, incident response, threat intelligence, and forensics.
