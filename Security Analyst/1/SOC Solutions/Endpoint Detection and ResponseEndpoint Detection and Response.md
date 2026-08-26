# EDR — Endpoint Detection and Response


**Endpoint Detection and Response (EDR)** is a security solution that monitors, detects, and responds to advanced threats at the endpoint level. It's widely adopted because remote work has pushed endpoints outside traditional network-perimeter protection, requiring a solution that protects devices no matter where they are.

**Common EDR solutions:** CrowdStrike Falcon, SentinelOne ActiveEDR, Microsoft Defender for Endpoint, OpenEDR, Symantec EDR.

---

### The Three Pillars of EDR

| Pillar | Description |
|---|---|
| **Visibility** | Collects detailed endpoint data — process, registry, file/folder modifications, user actions, and more — presented as structured process trees with full activity timelines and historical access for threat hunting |
| **Detection** | Combines signature-based, behavior-based, and ML-driven detection; catches fileless malware; supports custom IOC ingestion |
| **Response** | Lets analysts isolate hosts, terminate processes, quarantine files, and remotely execute actions — all from a central console |

> **Limitation:** EDR is a **host-only** solution — it does not detect network-level threats on its own.

---

### EDR vs. Antivirus (AV) — The Airport Analogy

Think of an endpoint as an airport. **AV** is like the immigration checkpoint — it checks passports (files) against a database of known criminals (signatures) and blocks matches. But a first-time offender with no record slips through undetected.

**EDR** is like security officers stationed *inside* the airport — constantly watching CCTV and motion sensors (endpoint behavior), regardless of whether someone passed the initial check. They notice:
- Loitering near restricted areas
- Suspicious behavior patterns
- Unattended bags left behind

Even if a threat evades AV entirely, EDR keeps watching its subsequent behavior and can act or alert on it.

#### Step-by-Step: AV vs. EDR Against an Advanced Attack

**Scenario:** Phishing email → malicious Word macro → spawns PowerShell → downloads obfuscated second-stage payload → injects into `svchost.exe` → attacker gains remote access.

| Step | AV's Response | EDR's Response |
|---|---|---|
| 1. Malicious doc downloaded | Nothing, if no known signature | Logs and monitors the download |
| 2. Document opened | Nothing — `winword.exe` is legitimate | Records `winword.exe` execution, keeps monitoring |
| 3. Macro spawns PowerShell | Nothing if the macro has no signature | Flags the unusual `winword.exe → PowerShell.exe` parent-child relationship |
| 4. Obfuscated PowerShell runs | Typically undetected | Flags the obfuscated script execution |
| 5. Payload injected into `svchost.exe` | Not flagged — no memory injection monitoring | Detects process injection in `svchost.exe` |
| 6. Outbound C2 connection | No network-level visibility | Flags the unexpected outbound connection from `svchost.exe` |
| **Final Action** | **May be marked clean** | **Generates a full attack-chain alert with response options** |

> Some modern AVs offer enhanced visibility/detection, but EDR is purpose-built to catch what signature-based AV structurally cannot.

---

### How EDR Works Under the Hood

```
Endpoint(s) → EDR Agent (Sensor) → EDR Console (correlation, ML, threat intel) → Alert
```

#### EDR Agents (Sensors)

Deployed on each endpoint, agents are the "eyes and ears" of the EDR — they monitor activity locally, perform basic signature/behavior-based detection, and stream data to the central console in real time.

#### EDR Console

The "brain" — correlates and analyzes agent data using complex logic, ML models, and threat intelligence matching to form detections (alerts). Presents a holistic dashboard of endpoint status across the organization.

#### After Detection

1. EDR assigns each alert a **severity** (Critical, High, Medium, Low, Informational)
2. Analyst prioritizes highest severity first
3. Analyst reviews all detection context — files, processes, network activity, registry changes
4. Analyst determines **True Positive vs. False Positive**
5. If true, the analyst takes response action directly from the console

#### EDR in the Broader Ecosystem

EDR is one piece of a larger stack — firewalls, DLPs, email security gateways, IAM, and EDR are typically all integrated into a **SIEM**, which becomes the central investigation point for analysts.

---

### Telemetry — The Endpoint's "Black Box"

**Telemetry** is the detailed activity data EDR agents collect and send to the console — everything needed for detection and investigation.

| Telemetry Type | What It Reveals |
|---|---|
| **Process Executions/Terminations** | Suspicious parent-child relationships, unusual executables spawning processes, malware payloads |
| **Network Connections** | C2 connections, unusual ports, data exfiltration, lateral movement |
| **Command Line Activity** | Malicious commands, obfuscated PowerShell — often missed by traditional AV |
| **Files/Folders Modifications** | Data staging, ransomware activity, malicious file drops |
| **Registry Modifications** | Configuration changes tied to malicious activity |

> Individually, these actions can look harmless — especially when malware abuses legitimate utilities. Only when observed together through detailed telemetry does the full malicious pattern emerge, helping both automated detection and human investigation reconstruct the attack timeline.

---

### Advanced Detection Techniques

| Technique | How It Works | Example |
|---|---|---|
| **Behavioral Detection** | Observes full file/process behavior rather than just matching signatures | `winword.exe` spawning `PowerShell.exe` — an unusual parent-child relationship |
| **Anomaly Detection** | Learns a baseline of normal endpoint behavior, flags deviations | An endpoint modifying an auto-start registry key it's never touched before |
| **IOC Matching** | Matches activity against threat intelligence feeds (hashes, IPs, domains) | A downloaded executable's hash matches a known malicious sample |
| **MITRE ATT&CK Mapping** | Tags detections with the relevant Tactic/Technique | Scheduled task creation → Tactic: *Persistence*, Technique: *Scheduled Task/Job* |
| **Machine Learning** | Trained on large normal/malicious datasets to catch complex, multi-step patterns that evade other methods | Detecting fileless or multi-stage intrusions where no single action looks malicious alone |

---

### Response Capabilities

| Action | When to Use |
|---|---|
| **Isolate Host** | Contains malicious activity by cutting the endpoint off from the network — critical for stopping lateral movement |
| **Terminate Process** | A lighter-touch option when full isolation would disrupt critical business operations |
| **Quarantine** | Moves a malicious file to an isolated location where it can't execute, pending review |
| **Remote Access** | Direct shell access to run custom scripts or commands beyond built-in response actions |
| **Artefact Collection** | Extracts memory dumps, event logs, specific folder contents, or registry hives for forensic investigation or legal purposes — without physically accessing the device |

---

### EDR Quick Reference

| Concept | Summary |
|---|---|
| EDR = | Endpoint-level monitoring, detection, and response solution |
| 3 Pillars | Visibility, Detection, Response |
| Key limitation | Host-only — no direct network-level detection |
| vs. AV | AV = signature matching only; EDR = behavior, ML, telemetry-based detection over time |
| Architecture | Agent (endpoint) → Console (correlation/analysis) → Alert |
| Telemetry types | Process, Network, Command Line, File/Folder, Registry |
| Detection techniques | Behavioral, Anomaly, IOC Matching, ATT&CK Mapping, Machine Learning |
| Response actions | Isolate Host, Terminate Process, Quarantine, Remote Access, Artefact Collection |
| Ecosystem role | Integrates with SIEM alongside firewalls, DLP, email security, IAM |

---

## Summary

- **EDR** monitors, detects, and responds to threats at the endpoint level, built on three pillars — **Visibility** (detailed process/registry/file telemetry), **Detection** (signature, behavioral, and ML-driven), and **Response** (isolate, terminate, quarantine) — with the key limitation that it is **host-only** and has no direct network-level visibility.
- Unlike **AV**, which only blocks files matching known signatures (an immigration checkpoint), EDR continuously watches endpoint *behavior* over time (security officers inside the building) — walked through a full attack chain (phishing → macro → PowerShell → process injection → C2), EDR flags every stage while signature-based AV may miss all of it.
- Architecturally, lightweight **agents** on each endpoint stream telemetry to a central **console**, which correlates the data using ML and threat intelligence to generate severity-ranked alerts that analysts review and act on.
- Key **telemetry types** — process executions, network connections, command-line activity, file/folder modifications, and registry modifications — individually look harmless but reveal a full attack pattern when correlated together.
- Detection relies on **behavioral detection, anomaly detection, IOC matching, MITRE ATT&CK mapping,** and **machine learning**, and confirmed threats can be handled via **host isolation, process termination, file quarantine, remote shell access,** or **forensic artefact collection** — all integrated into the broader SIEM ecosystem alongside firewalls, DLP, email security, and IAM.


- [74. Splunk Basics](./74-splunk-basics.md)
