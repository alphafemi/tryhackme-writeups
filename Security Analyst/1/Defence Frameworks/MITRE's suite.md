# MITRE ATT&CK, CAR, D3FEND & Related Frameworks



MITRE maintains a suite of interconnected cybersecurity frameworks — **ATT&CK**, **CAR**, **D3FEND**, **Engage**, and emerging domain-specific matrices — that help both red and blue teams understand adversary behavior and build stronger defenses.

---

### MITRE ATT&CK®

A globally accessible knowledge base of adversary **tactics and techniques**, based on real-world observations, used as a foundation for threat models across industry, government, and security vendors. Created in **2013** to document and categorize the standard TTPs used by APT groups.

#### TTP Breakdown

| Term | Definition |
|---|---|
| **Tactic** | The adversary's goal/objective — the "why" |
| **Technique** | How the adversary achieves that goal |
| **Procedure** | The specific implementation of the technique |

#### Evolution

Originally focused on Windows, ATT&CK expanded into the **Enterprise matrix** (Windows, macOS, Linux, cloud, etc.), plus dedicated matrices for **Mobile** and **Industrial Control Systems (ICS)**. It's community-driven and used by both defenders (building detections) and red teams (planning realistic attack simulations).

#### The ATT&CK Matrix

Tactics run across the top; each contains nested **techniques**, which can expand into **sub-techniques**. The **ATT&CK Navigator** tool helps annotate and explore these matrices.

**Example hierarchy:**

```
Tactic: Reconnaissance
  └─ Technique: Active Scanning
       ├─ Sub-technique: Scanning IP Blocks
       ├─ Sub-technique: Vulnerability Scanning
       └─ Sub-technique: Wordlist Scanning
```

Each technique's page includes a description, unique **Technique ID**, procedure examples (real groups/software/campaigns that used it), mitigations, detections, and references.

---

### Why ATT&CK Matters

| Value | Explanation |
|---|---|
| **Standard language** | Provides consistent terminology and unique IDs so the same adversary behavior isn't described a dozen different ways across the industry |
| **Bridges intel and defense** | Threat reports often describe *what* happened but not *how to detect it*; mapping activity to TTPs lets defenders build real detection logic, queries, and playbooks from raw intelligence |

#### Who Uses ATT&CK

| Role | Goal | How They Use ATT&CK |
|---|---|---|
| **CTI Teams** | Improve security posture via threat analysis | Map observed actor behavior to TTPs, building actionable profiles |
| **SOC Analysts** | Investigate/triage alerts | Link activity to tactics/techniques for alert context and prioritization |
| **Detection Engineers** | Design/improve detections | Map SIEM/EDR rules to ATT&CK to close coverage gaps |
| **Incident Responders** | Investigate incidents | Map incident timelines to tactics/techniques for clearer visualization |
| **Red & Purple Teams** | Emulate adversaries to test defenses | Build emulation plans aligned with known techniques/group operations |

#### Mapping in Action: Threat Group Example

**Mustang Panda (G0129)** — a threat group targeting government entities, non-profits, and NGOs — has been mapped to ATT&CK techniques based on years of observed activity:

- **Initial Access:** Phishing
- **Persistence:** Scheduled tasks
- **Defense Evasion:** File obfuscation
- **Command & Control:** Ingress tool transfer

This mapping (viewable via the group's ATT&CK Navigator matrix) lets defenders quickly understand a specific adversary's preferred TTPs for building targeted detections.

---

### CAR — Cyber Analytics Repository

**CAR** is "a knowledge base of analytics developed by MITRE based on the ATT&CK adversary model" — in plain terms, a collection of **ready-made detection analytics** built around specific ATT&CK techniques.

Each CAR analytic describes:

- The behavior pattern to detect
- **Pseudocode** (plain, human-readable logic)
- Example queries for real tools (e.g., **Splunk**, **EQL**, **LogPoint**)
- Sometimes **Unit Tests** to validate the analytic actually works

**Example — CAR-2020-09-001: Scheduled Task - File Access:**

Includes a description, associated ATT&CK tactics/techniques, and implementation examples (pseudocode + Splunk query + LogPoint search) — translating a known TTP directly into something a defender can run against their SIEM.

CAR also has its own **ATT&CK Navigator layer**, mapping its analytics onto the matrix for browsing.

---

### D3FEND — Detection, Denial, and Disruption Framework Empowering Network Defense

If ATT&CK explains *how attacks happen*, **D3FEND** explains *how to stop them* — a structured framework mapping **defensive techniques**, giving defenders a common language for describing security controls.

#### The D3FEND Matrix — 7 Tactics

| Tactic |
|---|
| Model |
| Harden |
| Detect |
| Isolate |
| Deceive |
| Evict |
| Restore |

**Example — Credential Rotation (D3-CRO):** Regularly rotating passwords to prevent reuse of stolen credentials. D3FEND documents how this defense works, implementation considerations, and its relationship to specific digital artifacts and corresponding ATT&CK techniques — connecting the attacker's move to the defender's countermeasure.

---

### Other MITRE Projects

| Project | Purpose |
|---|---|
| **Adversary Emulation Library** (via CTID) | Free, step-by-step emulation plans that mimic real-world threat group behavior |
| **Caldera** | Automated adversary emulation tool for simulating real attacker behavior using ATT&CK, useful for both red team exercises and blue team detection/incident-response practice |
| **AADAPT** (Adversarial Actions in Digital Asset Payment Technologies) | ATT&CK-style matrix for threats targeting blockchain, smart contracts, digital wallets, and other digital asset technologies |
| **ATLAS** (Adversarial Threat Landscape for Artificial-Intelligence Systems) | Knowledge base/matrix documenting real-world attack techniques, vulnerabilities, and mitigations specific to AI/ML systems |

---

### MITRE Frameworks Quick Reference

| Framework | Focus |
|---|---|
| **ATT&CK** | Adversary tactics, techniques, procedures (offense-side knowledge base) |
| **CAR** | Ready-made detection analytics tied to ATT&CK techniques |
| **D3FEND** | Structured defensive techniques/countermeasures |
| **Caldera** | Automated adversary emulation |
| **Adversary Emulation Library** | Step-by-step real-world group emulation plans |
| **AADAPT** | Digital asset/blockchain-specific threat matrix |
| **ATLAS** | AI/ML system-specific threat matrix |
| TTP hierarchy | Tactic (why) → Technique (how) → Procedure (implementation) |
| ATT&CK matrices | Enterprise, Mobile, ICS |
| Navigator tool | Annotates/explores ATT&CK (and CAR) matrices |

---

## Summary

- **MITRE ATT&CK**, created in 2013, is a community-driven knowledge base of real-world adversary **Tactics** (why), **Techniques** (how), and **Procedures** (specific implementation), organized into Enterprise, Mobile, and ICS matrices and explored via the **ATT&CK Navigator** — used by CTI teams, SOC analysts, detection engineers, incident responders, and red/purple teams alike, illustrated by a real mapping of the threat group Mustang Panda.
- **CAR (Cyber Analytics Repository)** turns specific ATT&CK techniques into **ready-made detection analytics** — pseudocode plus real Splunk/EQL/LogPoint queries, sometimes with unit tests — and has its own Navigator layer for browsing.
- **D3FEND** is the defensive counterpart to ATT&CK, organizing countermeasures into seven tactics (**Model, Harden, Detect, Isolate, Deceive, Evict, Restore**) and explicitly linking each defensive technique (e.g., Credential Rotation) back to the ATT&CK techniques it counters.
- Related MITRE projects extend this ecosystem further: the **Adversary Emulation Library** and **Caldera** for practical red/blue team exercises, **AADAPT** for blockchain/digital-asset threats, and **ATLAS** for AI/ML-specific threats.

