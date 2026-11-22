# Cyber Threat Intelligence — Lifecycle, TLP, STIX/TAXII & the CTI Analyst Workflow

## Overview

This document covers Cyber Threat Intelligence (CTI) as an everyday SOC skill: why CTI matters, the progression from data to information to intelligence, the core labels (IOC, IOA, TTP), indicator types and where to enrich each, feeds versus platforms, intelligence sources and classifications, the six-phase CTI lifecycle with the Traffic Light Protocol (TLP) and a worked scenario, how CTI ties into the standards and frameworks covered elsewhere, and sharing intelligence through STIX and TAXII.

---

**Cyber Threat Intelligence (CTI)** translates an abstract discipline into everyday SOC tasks: enriching indicators, triaging alerts, and turning raw data into the stories that matter. The goal is moving an analyst from *"I recognize an IP address"* to *"I recognize why this IP address spells trouble, and I can prove it."*

---

### Why CTI Matters

A junior analyst starting a shift can face hundreds of fresh alerts — mostly benign noise, but occasionally something genuinely dangerous. CTI provides the context to tell them apart, answering three essential questions:

1. **Who or what** is on the other end of this alert indicator?
2. **What was their behavior** in the past?
3. **How does my organization respond**, and what should I do right now?

Answered early (ideally within minutes), these questions buy precious downstream time for Incident Response. CTI isn't exclusive to intel specialists — it's a front-line tool that elevates SOC triage from guesswork to calculated action.

---

### From Raw Data to Usable Intelligence

Information security distinguishes **data**, **information**, and **intelligence** — a distinction worth making explicit:

| Layer | Definition | Alert-Queue Example | SOC L1 Action |
|---|---|---|---|
| **Data** | An unprocessed observable | `45.155.205.3 :443` | Capture the artifact |
| **Information** | Data plus factual annotation | IP registered to Hetzner, first seen 2023-07-14 | Record attributes |
| **Intelligence** | Analyzed information that answers "so what" | IP belongs to the current BumbleBee C2; block immediately | Escalate or suppress |

An L1 analyst is responsible for enriching artifacts until they qualify as intelligence — or demonstrating that they never will — via rapid, methodical lookups across public, commercial, and internal sources.

**Three core labels every analyst should know:**

| Label | Meaning | Example |
|---|---|---|
| **Indicator of Compromise (IOC)** | Evidence of a breach | A C2 address in the logs |
| **Indicator of Attack (IOA)** | A malicious action currently underway | PowerShell launching an unknown service |
| **Tactics, Techniques, and Procedures (TTP)** | An adversary's detailed methodology, expressed via MITRE ATT&CK IDs (see Section 80) | T1110.003 Password Guessing |

---

### Indicator Types Essential to First-Line Triage

Each artifact type demands a tailored enrichment path — knowing where to look matters more than memorizing every tool:

| Indicator | Example | First Resources | Associated IOA/TTP Example |
|---|---|---|---|
| **IPv4/IPv6** | `45.155.205.3` | WHOIS, VirusTotal Relations, Shodan | IOA: repeated SSH failures → TTP: T1110.003 Password Guessing |
| **Domain/FQDN** | `malicious-updates[.]net` | WHOIS age, passive-DNS (RiskIQ/SecurityTrails), urlscan.io | IOA: surge of DNS queries to a 24h-old domain |
| **URL** | `hxxp://malicious-updates[.]net/login` | URLhaus, urlscan.io behavior graph, Any.Run (network off) | IOA: browser POST to `/gateway.php` with payload |
| **File hash** | `e99a18c428cb38d5…` | VirusTotal, Hybrid Analysis, MalShare | TTP: T1055 Process Injection into `regsvr32.exe` |
| **Email address** | `billing@evil-corp.com` | MXToolbox header analysis, Have I Been Pwned | IOA: SPF failure plus recent domain registration |
| **Local artifact** | `HKCU\Software\Run\updater.exe` | Sigma rules, EDR prevalence query, vendor knowledge base | TTP: T1060.001 Registry Run Keys |

> **Tip:** Keep a bookmark folder or SIEM launcher panel that opens preferred lookups with the indicator pre-populated — 30 seconds saved per alert adds up to hours over a month.

---

### Feeds vs. Platforms

- **Feed** — a scheduled stream of indicators (CSV, JSON, STIX, or via TAXII). Over-ingesting feeds without curation drowns analysts in false positives and erodes trust in the program.
- **Platform** — a structured repository storing indicators, tracking enrichment, mapping relationships, and enforcing sharing permissions. **MISP** and **OpenCTI** are leading open-source examples.

Sound practice introduces feeds gradually, confirms alignment with the organization's threat model, and promotes them into the platform only after measuring actionability — the platform then becomes the single source of truth.

### Sources of Cyber Threat Intelligence

| Source Type | Description |
|---|---|
| **Internal telemetry** | SIEM logs, EDR detections, phishing-mailbox submissions — highest immediate relevance |
| **Commercial services** | Vendor premium feeds, paid sandboxes, closed-source analytics — high fidelity, but export/sharing may be license-limited |
| **Open-source (OSINT)** | AbuseIPDB, URLhaus, public blogs, academic research — cross-confirm before applying |
| **Communities & ISACs** | Sector-specific lists with rich context (e.g., **FS-ISAC**) |

### Threat Intelligence Classifications

| Classification | Focus | Example |
|---|---|---|
| **Strategic** | High-level org threat landscape, trends/patterns for business decisions | Annual ransomware report predicting a shift to data-wiping extortion in healthcare |
| **Tactical** | Adversary TTPs | Advisory on new T1059.005 (Visual Basic) abuse in malspam |
| **Operational** | Campaign-specific motive/intent, targeted critical assets | Understanding which people/processes/tech an active campaign targets |
| **Technical** | Atomic indicators and artifacts | IPs, hashes tied to a specific attack |

> L1 analysts typically escalate many **Technical** IOCs, observe and document **Tactical** IOAs, and feed patterns into **Operational** reporting.

---

### The CTI Lifecycle — Six Phases

CTI follows a six-phase lifecycle that turns raw data into contextualized, action-oriented insight:

```
Direction → Collection → Processing → Analysis → Dissemination → Feedback
```

It is walked through below via a single SOC L1 analyst, **Alex**, protecting TryHatMe Corp's production PostgreSQL database.

#### Prerequisite Concepts

**Traffic Light Protocol (TLP)** — a four-color labeling scheme (FIRST.org) governing how widely intel may be shared:

| TLP Label | Sharing Boundary | Typical SOC L1 Behavior |
|---|---|---|
| **TLP:CLEAR** | No restriction | Post to the internal wiki/platform |
| **TLP:GREEN** | Peer community, not public | Upload to MISP/Slack restricted to partner SOCs |
| **TLP:AMBER** | Org-wide; external only with need-to-know | Keep within the CTI platform; reference, don't copy, in tickets |
| **TLP:RED** | Named recipients only | Store in an encrypted note; no posting to the ticketing system without clearance |

> An IOC's TLP label must travel with it into any platform or triage note — violating the label can breach contractual obligations and erode partner trust.

**Intel Formats** — **STIX** (Structured Threat Information Expression) is a JSON schema describing threat indicators, relationships, and context in machine-readable form.

#### Scenario: TryHatMe Corp

TryHatMe holds sensitive customer data in a segmented-subnet PostgreSQL server, fronted by an NGFW and monitored by EDR. Management asked the SOC to "bring in CTI" so controls react faster to emerging threats — Alex is assigned to build and exercise the workflow.

**Step 1 — Direction (Defining the Mission):** Alex, the CTI lead, and the DB admin translate the mandate into a concrete intelligence requirement — primary asset (PostgreSQL DB), business risk (GDPR fines, trust loss), available controls (NGFW IP/domain blocking, EDR hash quarantine), and two measurable questions:

- **Q1:** Which external IPs/domains currently exploit PostgreSQL services or exfiltrate records?
- **Q2:** Which malware families targeting PostgreSQL drivers/credentials are active this week, and what are their hashes?

**Step 2 — Collection (Assembling Raw Material):** Four sources feed the effort:

| Source | Rationale | Example Artifacts |
|---|---|---|
| Commercial NGFW-vendor feed | Tailored to the same firewall model, high fidelity | 37 IPs flagged as DB-exfil C2 in 24h |
| AbuseIPDB (tagged `PostgreSQL-brute-force`) | Rapid community updates | 15 IPs, 4 domains |
| Internal MISP | Historical incident view | 2 SHA-256 hashes of PgSQL credential stealers |
| This week's vendor threat report | Strategic insight distilled into technical IOCs | 1 new hash, 3 C2 domains |

Each feed is exported as STIX/CSV and stored dated in the SOC's "raw-intel" bucket for reproducibility.

**Step 3 — Processing (Normalizing and Correlating):** Raw feeds rarely align perfectly. Alex runs scheduled scripts that:

- Normalize indicator syntax (compress IPv6, lowercase domains, strip subnet masks)
- Correlate/deduplicate against the platform's existing indicator table
- Tag each entry with source, date, and TLP
- Produce action files: `firewall_blocklist.csv` and `edr_hash_rules.yar`

> **TLP collision example:** one IP appears as `TLP:AMBER` in the NGFW feed but `TLP:CLEAR` in AbuseIPDB — the **stricter label always wins**, so the combined record inherits `TLP:AMBER`, preventing inadvertent disclosure.

**Step 4 — Analysis (Turning Information Into Judgment):** Blocking everything suspected would create false positives, so Alex validates relevance against the Step 1 questions:

- **Firewall indicators (Q1):** a Splunk query confirms one NGFW IP attempted (and failed) TCP 5432 against production — validating applicability
- **Hash indicators (Q2):** OpenCTI links the hash to the **PgSteal** malware family; Any.Run confirms credential-dump behavior; since the org uses the exact named ODBC driver, it's marked high priority

Each indicator is graded:

| Confidence | Source Agreement | Local Sightings | Action |
|---|---|---|---|
| **High** | Same IOC in ≥2 sources | ≥1 local attempt | Immediate block |
| **Medium** | Single trusted source | No local hits | Alert-only |
| **Low** | OSINT only | No context | Monitor for 14 days |

Result: 7 IPs and 1 hash meet the High bar; the rest are queued Medium for monitoring.

**Step 5 — Dissemination (Reaching the Right Consumers):** Unused intelligence is shelfware. Alex tailors output per stakeholder:

| Stakeholder | Format | Why |
|---|---|---|
| Firewall team | CSV upload + change ticket | They own block rules; ticket documents risk and TLP |
| Endpoint team | YARA rule set in EDR console | Loads rules into policy |
| CTI platform | Indicator objects, fully tagged | Keeps history, supports future correlation, honors TLP |
| Management | 200-word summary in the weekly cyber-risk memo | Demonstrates ROI of the process |

**Step 6 — Feedback (Measuring and Refining):** Two weeks later:

| KPI | Baseline | After First Cycle |
|---|---|---|
| Median dwell time of PgSQL brute-force IPs | 48h | 0h (pre-emptive block) |
| False-positive rate on new blocks | — | 0% (none revoked) |

Because results met the Step 1 objectives, management approves expansion (e.g., adding DNS tunneling IOCs for the same asset group next sprint) — Alex updates the direction document and schedules the next iteration.

---

### Standards and Frameworks (Cross-References)

This track previously covered several frameworks CTI analysts rely on daily — summarized here with pointers:

| Framework | Role in CTI |
|---|---|
| **MITRE ATT&CK** (Section 80) | The common technique vocabulary; an analyst matches alert behavior to a tactic/technique pair (e.g., `T1071.001` web-based C2) so Level 2/IR instantly know applicable mitigations and threat-actor profiles |
| **MITRE D3FEND** (Section 80) | The defensive counterpart to ATT&CK; given a `T1048.003` DNS tunnel alert, D3FEND surfaces the matching defensive technique (e.g., **D3-NTDN**, DNS request analysis) with practical controls |
| **Cyber Kill Chain** (Section 78) and **Unified Kill Chain** (Section 79) | Phase models for breaking down adversary actions (Reconnaissance → Weaponization → Delivery → Exploitation → Installation → C2 → Actions on Objectives) |
| **CVE / CVSS / NVD** (Section 61) | CVE catalogues vulnerabilities (e.g., `CVE-2023-4863`), CVSS scores severity 0–10, and NVD is the canonical repository linking CVEs to CVSS scores, exploits, and affected products |

---

### Sharing and Processing Intel — STIX & TAXII

- **STIX** — the structured JSON schema for describing threat information (indicators, relationships, context)
- **TAXII** (Trusted Automated eXchange of Indicator Information) — a set of secure APIs for near-real-time threat intel exchange, supporting two models: **Collection** (a producer hosts intel for pull) and **Channel** (a central server pushes intel to subscribers)

Sharing intelligence shortens the gap between another organization's incident and one's own preventive action, and community contribution builds reciprocal trust. However, **not every indicator should be shared** — privacy laws, customer NDAs, or internal competitive concerns may forbid disclosure, and early sharing of specific IOCs can tip off adversaries that their campaign has been detected.

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| Data → Information → Intelligence | Raw observable → annotated fact → "so what" judgment |
| Core labels | IOC (evidence of breach), IOA (action in progress), TTP (adversary methodology) |
| Feed vs. Platform | Feed = scheduled indicator stream; Platform = structured, enriched repository (MISP, OpenCTI) |
| Four intel sources | Internal telemetry, commercial, OSINT, communities/ISACs |
| Four intel classifications | Strategic, Tactical, Operational, Technical |
| CTI lifecycle phases | Direction → Collection → Processing → Analysis → Dissemination → Feedback |
| TLP labels | CLEAR, GREEN, AMBER, RED — stricter label always wins on conflict |
| Confidence grading | High (multi-source + local sighting) / Medium (single source) / Low (OSINT only) |
| STIX | JSON schema for machine-readable threat intel |
| TAXII | Secure API for near-real-time intel exchange (Collection or Channel model) |
| Related frameworks | MITRE ATT&CK & D3FEND (Section 80), Cyber Kill Chain & UKC (Sections 78–79), CVE/CVSS/NVD (Section 61) |
| Why not share everything | Privacy laws, NDAs, competitive concerns, risk of tipping off the adversary |

---

## Summary

- **CTI** moves an analyst from recognizing an indicator to understanding why it matters, by answering who or what is behind it, what their past behavior was, and how the organization should respond; the progression is **data** (a raw observable) → **information** (data plus factual annotation) → **intelligence** (analyzed information that answers "so what"), and the core labels are **IOC** (evidence of a breach), **IOA** (an attack in progress), and **TTP** (an adversary's methodology, expressed in MITRE ATT&CK IDs).
- Each **indicator type** (IP, domain, URL, file hash, email address, local artifact) has its own enrichment path, from WHOIS, VirusTotal, and Shodan to URLhaus, urlscan.io, and Any.Run; **feeds** are scheduled indicator streams (which drown analysts in false positives if over-ingested), while **platforms** like **MISP** and **OpenCTI** are the structured single source of truth, and intelligence comes from **internal telemetry, commercial services, OSINT,** and **communities/ISACs**, classified as **strategic, tactical, operational,** or **technical**.
- The **six-phase CTI lifecycle** — **Direction, Collection, Processing, Analysis, Dissemination, Feedback** — is walked through with analyst Alex protecting a PostgreSQL database: defining two measurable intelligence questions, collecting from four sources, normalizing and tagging with **TLP**, grading indicators by confidence (**High** blocks immediately, **Medium** alerts only, **Low** is monitored for 14 days), tailoring output to the firewall team, endpoint team, CTI platform, and management, and measuring results (median dwell time dropping from 48h to 0h with no false positives).
- The **Traffic Light Protocol** — **CLEAR, GREEN, AMBER, RED** — governs how widely intel may be shared and must travel with every indicator; when two sources conflict, the **stricter label always wins**.
- **STIX** (a JSON schema for machine-readable threat intel) and **TAXII** (secure APIs for near-real-time exchange via **Collection** or **Channel** models) enable sharing, but not every indicator should be shared — privacy laws, NDAs, competitive concerns, and the risk of tipping off the adversary can all argue against it.


