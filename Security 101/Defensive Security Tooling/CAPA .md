# CAPA — Malware Capability Analysis


---

**CAPA** (Common Analysis Platform for Artifacts), developed by the FireEye Mandiant team, identifies the *capabilities* of executable files — PE, ELF, .NET modules, shellcode, and even sandbox reports — by matching them against a large set of behavior-describing rules. It encapsulates years of reverse-engineering knowledge into an automated tool, letting analysts quickly understand what a binary can do (network communication, file manipulation, process injection, etc.) without manually reverse engineering it. It's especially valuable for **malware analysis and threat hunting**.

---

### Running CAPA

```powershell
# Navigate to the CAPA directory, then run it against a binary
PS C:\Users\Administrator\Desktop\capa> capa.exe .\cryptbot.bin
```

| Flag | Description |
|---|---|
| `-h` / `--help` | Show help and exit |
| `-v` / `--verbose` | Enable verbose output (slower) |
| `-vv` / `--vverbose` | Enable very verbose output (slower still) |

Processing large binaries can take several minutes. Pre-processed output can also be reviewed later:

```powershell
Get-Content .\cryptbot.txt
```

---

### Anatomy of CAPA's Output

CAPA's default report is organized into several blocks:

#### 1. File Information

| Field | Meaning |
|---|---|
| `md5` / `sha1` / `sha256` | Cryptographic hashes of the file |
| `analysis` | How CAPA performed the analysis (e.g., `static`) |
| `os` | The OS context the capabilities apply to |
| `arch` | Architecture (e.g., `i386`) |
| `path` | Location of the analyzed file |

#### 2. MITRE ATT&CK Mapping

The **MITRE ATT&CK** framework documents adversary tactics and techniques across the attack lifecycle. CAPA maps identified capabilities to it.

| Format | Example | Breakdown |
|---|---|---|
| `Tactic::Technique::ID` | `Defense Evasion::Obfuscated Files or Information::T1027` | Tactic = `DEFENSE EVASION`; Technique = `Obfuscated Files or Information`; ID = `T1027` |
| `Tactic::Technique::Sub-Technique::ID.SubID` | `Defense Evasion::Obfuscated Files or Information::Indicator Removal from Tools T1027.005` | Adds a Sub-Technique (`Indicator Removal from Tools`) and Sub-Technique ID (`005`) |

#### 3. MAEC (Malware Attribute Enumeration and Characterization)

A standardized language for encoding malware attributes, behaviors, and relationships. CAPA's most common MAEC values:

| MAEC Value | Description |
|---|---|
| **Launcher** | Behaves similarly to malware — e.g., dropping payloads, activating persistence, contacting C2 servers, executing specific functions |
| **Downloader** | Fetches and executes additional payloads/resources — e.g., pulling updates, retrieving config files, executing secondary stages |

#### 4. MBC (Malware Behavior Catalogue)

A catalogue of malware objectives and behaviors, complementary to (but not duplicating) ATT&CK.

**Two output formats:**

| Format | Example | Breakdown |
|---|---|---|
| `Objective::Behavior::Method[ID]` | `Anti-Static Analysis::Executable Code Obfuscation::Argument Obfuscation [B0032.020]` | Objective, Behavior, **Method** (sub-technique), and Identifier |
| `Objective::Behavior::[ID]` | `Communication::HTTP Communication::[C0002]` | Objective, Behavior, and Identifier (no method) |

**Key MBC concepts:**

| Concept | Description |
|---|---|
| **Objective** | Based on ATT&CK tactics adapted for malware characterization (e.g., Anti-Behavioral Analysis, Anti-Static Analysis, Collection, Command and Control, Credential Access, Defense Evasion, Discovery, Execution, Exfiltration, Impact, Lateral Movement, Persistence, Privilege Escalation) |
| **Micro-Objective** | Lower-level groupings like Process, Memory, Communication, Data — actions that aren't inherently malicious but are often abused |
| **MBC Behavior** | A specific behavior under an objective (e.g., `Lab Machine Detection [B0009]`) |
| **Micro-Behavior** | A specific low-level action under a micro-objective (e.g., `Allocate Memory [C0007]`) |
| **Method** | A sub-technique tied to a behavior (e.g., `Base64 [C0026.001]` under `Encode Data`) |

> **Example decode:** `DATA::Encode Data::Base64 [C0026.001]` → the file has the DATA objective's Encode Data behavior, using the Base64 method — i.e., **this file can Base64-encode data.**

#### 5. Capability and Namespace

The final block lists individual matched **capabilities** (rule names) and their **namespace**.

**Format:** `Capability (Rule Name)::TLN/Namespace`

Example: `reference anti-VM strings :: Anti-Analysis/anti-vm/vm-detection`

| Top-Level Namespace (TLN) | Covers |
|---|---|
| **anti-analysis** | Techniques to evade analysis (obfuscation, packing, anti-debugging) |
| **collection** | Data-gathering behaviors |
| **communication** | Network interaction (C2, data transmission, etc.) |
| **compiler** | Build-environment/compiler fingerprinting |
| **data-manipulation** | Data transformation (encoding, encryption) |
| **executable** | Attributes of the executable file itself (PE sections, debug info) |
| **host-interaction** | Interaction with the host system (files, directories, processes) |
| **impact** | Potential consequences of the malware's behavior |
| **internal** | Rules for CAPA's internal use, not for reporting |
| **lib** | Building blocks used by other rules |
| **linking** | Dynamic loading/linking of external code or libraries |
| **load-code** | Runtime code loading/execution |
| **malware-family** | Signatures tied to known malware families |
| **nursery** | Staging ground for unpolished/unfinished rules |
| **persistence** | Maintaining access on a compromised system |
| **runtime** | Identifying the language/platform the program runs on |
| **targeting** | Behaviors related to interacting with ATMs |

> **Rule-to-file naming:** A capability's name matches its YAML rule filename, with spaces replaced by dashes (e.g., capability *"reference anti-VM strings"* → `reference-anti-vm-strings.yml`).
>
> **Note on exceptions:** Some capabilities that logically belong under a specific TLN (e.g., `reference cryptocurrency strings` under Impact) are actually filed under **Nursery** if the rule isn't fully polished yet.

---

### Detailed Rule Inspection (`-vv` and JSON Export)

For deep inspection of *why* a rule matched, use very-verbose mode and export to JSON for easier analysis:

```powershell
# Very verbose output
capa -vv .\cryptbot.bin

# Very verbose output as JSON (recommended for large results)
capa.bin -j -vv .\cryptbot.bin > cryptbot_vv.json
```

Raw `-vv` output can run to thousands of lines — impractical to review in a terminal or text editor, which is where **CAPA Web Explorer** comes in.

---

### CAPA Web Explorer

A browser-based UI for exploring CAPA's JSON output — either the online version or an offline HTML page bundled with the tool.

**Workflow:**
1. Open CAPA Web Explorer (online or offline `.html`)
2. Click **Upload from local** and select the `_vv.json` file
3. Browse capabilities, drill into matched **features** (e.g., specific strings/regex matched), and use the **Global Search box** and filters to navigate large result sets far more easily than a text editor

**Example — matched rule for `reference anti-VM strings targeting VMWare`:**

```yaml
rule:
  meta:
    name: reference anti-VM strings targeting VMWare
    namespace: anti-analysis/anti-vm/vm-detection
    att&ck:
      - Defense Evasion::Virtualization/Sandbox Evasion::System Checks [T1497.001]
    mbc:
      - Anti-Behavioral Analysis::Lab Machine Detection [B0009]
  features:
    - or:
        - string: /VMWare/i
        - string: /VMTools/i
        - string: /SOFTWARE\\VMware, Inc\.\\VMware Tools/i
        # ... additional VM-related string patterns
```

This shows CAPA matched the file against a list of known VMware-related strings/registry paths — a classic anti-VM detection technique used by malware to avoid running inside analysis sandboxes.

**Example — matched rule for `schedule task via schtasks`:**

```yaml
rule:
  meta:
    name: schedule task via schtasks
    namespace: persistence/scheduled-tasks
    att&ck:
      - Persistence::Scheduled Task/Job::Scheduled Task [T1053.005]
  features:
    - and:
        - match: host-interaction/process/create
        - or:
            - and:
                - string: /schtasks/i
                - string: /\/create /i
            - string: /Register-ScheduledTask /i
```

This confirms the file creates a process and references `schtasks /create` or `Register-ScheduledTask` — indicating it registers a **scheduled task for persistence**.

---

### CAPA Quick Reference

| Concept | Summary |
|---|---|
| CAPA does | Identifies binary capabilities via automated rule matching |
| Input file types | PE, ELF, .NET, shellcode, sandbox reports |
| Key flags | `-v` (verbose), `-vv` (very verbose), `-j` (JSON output) |
| ATT&CK format | `Tactic::Technique[::Sub-Technique]::ID[.SubID]` |
| MAEC values | Launcher, Downloader |
| MBC layers | Objective → Behavior → Method, with Micro-Objective/Micro-Behavior for low-level actions |
| Namespace format | `Capability :: TLN/Namespace` |
| Rule ↔ filename | Capability name = YAML filename with dashes instead of spaces |
| Deep-dive tool | CAPA Web Explorer (upload `-j -vv` JSON output) |
| Unpolished rules live in | `nursery` TLN |

---

## Summary

- **CAPA** automatically identifies what a binary (PE, ELF, .NET, shellcode, or sandbox report) can *do* by matching it against a large rule set, sparing analysts from manually reverse engineering every capability.
- CAPA's report is organized into five blocks: **file information** (hashes, OS, architecture), **MITRE ATT&CK mapping** (`Tactic::Technique::ID`), **MAEC** classification (Launcher, Downloader), **MBC** (Malware Behavior Catalogue — Objective::Behavior::Method), and a final **capability/namespace** listing.
- Top-level namespaces (TLNs) like `anti-analysis`, `persistence`, `communication`, and `host-interaction` group related capabilities, with unpolished rules temporarily filed under `nursery`.
- Running CAPA with `-vv -j` and exporting to JSON, then loading that file into **CAPA Web Explorer**, is the recommended workflow for deeply inspecting *why* a specific rule matched — down to the individual strings or patterns detected.

