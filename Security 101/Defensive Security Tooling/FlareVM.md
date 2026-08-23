# FlareVM — Malware Analysis & Reverse Engineering Toolkit

## Overview

This document covers FlareVM, the FLARE Team's curated Windows-based malware analysis and reverse engineering toolkit. It walks through the eight major tool categories FlareVM bundles, deep dives on seven core investigation tools (Procmon, Process Explorer, HxD, Wireshark, CFF Explorer, PEStudio, FLOSS), and a worked example analyzing a suspicious `windows.exe` sample end-to-end using static analysis, string extraction, and cross-verified network behavior analysis.

---

**FlareVM** ("Forensics, Logic Analysis, and Reverse Engineering") is a curated Windows-based toolkit built by the FLARE Team at FireEye, designed for reverse engineers, malware analysts, incident responders, forensic investigators, and penetration testers.

---

### Tool Categories

| Category | Purpose | Example Tools |
|---|---|---|
| **Reverse Engineering & Debugging** | Taking a binary apart to understand how it works; finding and correcting errors | Ghidra (NSA), x64dbg, OllyDbg, Radare2, Binary Ninja, PEiD |
| **Disassemblers & Decompilers** | Breaking binaries into a more understandable format to study behavior, logic, control flow | CFF Explorer, Hopper Disassembler, RetDec |
| **Static & Dynamic Analysis** | Static = inspecting code without running it; dynamic = observing behavior while it runs | Process Hacker, PEview, Dependency Walker, DIE (Detect It Easy) |
| **Forensics & Incident Response** | Collecting/analyzing/preserving digital evidence; detecting, containing, eradicating, recovering from attacks | Volatility, Rekall, FTK Imager |
| **Network Analysis** | Studying network traffic and structure | Wireshark, Nmap, Netcat |
| **File Analysis** | Examining files for security threats and permission issues | FileInsight, Hex Fiend, HxD |
| **Scripting & Automation** | Automating repetitive analysis tasks | Python, PowerShell Empire |
| **Sysinternals Suite** | Managing, troubleshooting, and diagnosing Windows systems | Autoruns, Process Explorer, Process Monitor |

---

### Core Investigation Tools

| Tool | Investigative Value |
|---|---|
| **Procmon** (Process Monitor) | Tracks real-time system, file, registry, and thread/process activity |
| **Process Explorer** | Shows parent-child process relationships, loaded DLLs, and file paths |
| **HxD** | Hex editing/viewing for examining or altering malicious files |
| **Wireshark** | Observes and investigates network traffic for unusual activity |
| **CFF Explorer** | Generates file hashes for integrity verification and source authentication |
| **PEStudio** | Static analysis of executable properties without running the file |
| **FLOSS** | Extracts and de-obfuscates strings from malware using static analysis |

---

### Tool Deep Dives

#### Process Monitor (Procmon)
Records real-time file system, registry, and process/thread activity — valuable for malware research, troubleshooting, and forensic investigations.

> **Example:** Filtering for `lsass.exe` (Local Security Authority Subsystem Service) shows it reading `lsasrv.dll` — normal on its own, but LSASS is a common target for **credential dumping** (e.g., via Mimikatz). Watch for unusual access patterns or unexpected processes reading/writing `lsass.exe`.

#### Process Explorer (Procexp)
Provides deep insight into running processes — parent/child relationships, associated user accounts, and which process is accessing a given file or folder.

> Useful for tracing what a malicious document, LNK file, or ISO spawns as a child process — a common abuse pattern by threat actors.

#### HxD
A fast, flexible hex editor for files, memory, and drives — useful for forensic investigation, data recovery, debugging, and precise binary manipulation.

> A file beginning with hex bytes `4D 5A` ("MZ" in Little Endian) identifies it as a Windows **executable**. HxD's **Data Inspector** lets you interpret individual bytes as different data types (integer, float, etc.).

#### CFF Explorer
A PE (Portable Executable) editor/viewer that generates file hashes (MD5, SHA-1) for integrity verification and helps authenticate whether a system file has been altered — important since malicious code can hide inside modified legitimate files.

#### Wireshark
Captures and analyzes network traffic — protocol, source/destination IP, port — to hunt for suspicious connections, unusual protocols, or potential data exfiltration.

> `TLSv1.2` traffic indicates an encrypted connection, which can either be legitimate or used to mask malicious activity — encryption alone isn't proof of either.

#### PEStudio
Performs **static analysis** — studying an executable's properties without ever running it, reducing risk while surfacing suspicious indicators.

> **Entropy** is a key signal: high entropy (e.g., ~6.6) suggests possible packing or encryption, common in malware attempting to evade static detection. Tools with legitimate dual-use purposes (e.g., PsExec for remote administration) can also be abused for lateral movement — context and environment matter.

#### FLOSS (FLARE Obfuscated String Solver)
Automatically extracts and de-obfuscates strings from malware — including **stackstrings** and **tightstrings** that basic string extraction (like `strings.exe`) would miss — surfacing file paths, URLs (potential C2 servers), IPs, API calls, error messages, registry keys, and encryption keys.

```powershell
floss .\cobaltstrike.exe
```

If FLOSS extracts static strings but finds **no decoded strings**, that suggests the sample uses dynamically-generated or heavily obfuscated strings that weren't detected in that pass — a hallmark of more evasive malware.

---

### Worked Example: Analyzing `windows.exe`

**Scenario:** A user downloaded a suspicious `windows.exe`, flagged as a potential threat, on 09/24/2024. The file is provided for analysis.

#### Step 1 — Static Analysis with PEStudio

- **Hashes** (MD5/SHA-1) should be checked against threat intel sources like VirusTotal — no known detections raises the chance it's a new/unseen campaign.
- The file's metadata claims to be **REGEDIT** (Windows Registry Editor), but:
  - Legitimate REGEDIT normally runs from `C:\Windows\System32`, not a user's Downloads folder — a mismatch suggesting impersonation.
  - The version metadata contains **Russian-language text** (e.g., *"Редактор реестра"* — "Registry Editor"), suspicious if the organization has no Russian-speaking context.
- **No rich header** present — often indicates the file was packed/obfuscated to evade static analysis.
- Reviewing the **Import Address Table (IAT)** — the API calls the file imports, sortable by PEStudio's blacklist view — revealed:

| API / Function | Significance |
|---|---|
| `set_UseShellExecute` | Lets the process invoke the OS shell to launch other processes — common in malware spawning additional actions |
| `CryptoStream`, `RijndaelManaged`, `CipherMode`, `CreateDecryptor` | Indicates AES (Rijndael) cryptographic use — could support encrypted C2, file encryption, or ransomware behavior |

#### Step 2 — String Extraction with FLOSS

```powershell
FLOSS.exe .\windows.exe > windows.txt
```

The extracted strings corroborated the same suspicious API functions PEStudio had already surfaced, reinforcing the finding through a second, independent tool.

#### Step 3 — Network Behavior with Process Explorer & Process Monitor

Investigating a separate sample, `cobaltstrike.exe`, for potential **C2 connectivity**:

1. Run `cobaltstrike.exe`, open **Process Explorer**.
2. Confirm the parent/child relationship: launching manually makes `Explorer.exe` the parent and `cobaltstrike.exe` the child.
3. Right-click the process → **Properties → TCP/IP tab** to view active network connections and their state.
4. **Cross-verify with a second tool** — re-run the sample and observe it in **Process Monitor (Procmon)**:
   - Open the filter (`Ctrl+L`)
   - Set: **Process Name → contains → `cobalt`** → **Include** → **Add** → **Apply**
5. The filtered Procmon results confirmed the same finding: the binary connected out to an unknown IP address, **`47.120.46.210`**.

> **Key principle:** Never rely on a single tool's output — cross-verifying findings across independent tools (Process Explorer *and* Procmon here) increases confidence in the conclusion.

---

### FlareVM Quick Reference

| Concept | Summary |
|---|---|
| FlareVM = | FLARE Team's curated Windows malware analysis toolkit |
| 8 Tool categories | RE/Debugging, Disassemblers/Decompilers, Static/Dynamic Analysis, Forensics/IR, Network Analysis, File Analysis, Scripting/Automation, Sysinternals |
| Core investigation tools | Procmon, Process Explorer, HxD, Wireshark, CFF Explorer, PEStudio, FLOSS |
| MZ header | Identifies a file as a Windows executable |
| High entropy | Suggests packing/encryption |
| No rich header | Suggests obfuscation/evasion |
| IAT | Import Address Table — reveals imported API calls and likely behavior |
| FLOSS command | `floss <file>` or `FLOSS.exe <file> > output.txt` |
| Cross-verification | Confirm findings with a second independent tool before concluding |

---

## Summary

- **FlareVM** is the FLARE Team's curated Windows toolkit spanning eight categories — reverse engineering/debugging, disassemblers/decompilers, static/dynamic analysis, forensics/IR, network analysis, file analysis, scripting/automation, and the Sysinternals suite.
- Seven core tools anchor most investigations: **Procmon** (real-time system activity), **Process Explorer** (process relationships), **HxD** (hex editing), **Wireshark** (network traffic), **CFF Explorer** (PE hashing/integrity), **PEStudio** (static analysis), and **FLOSS** (obfuscated string extraction).
- Key static-analysis signals include the **MZ header** (identifies a Windows executable), **high entropy** (suggests packing/encryption), a **missing rich header** (suggests obfuscation), and suspicious entries in the **Import Address Table**.
- A worked example on a suspicious `windows.exe` combined PEStudio (metadata impersonation, missing rich header, suspicious IAT entries), FLOSS (corroborating string evidence), and cross-verification between Process Explorer and Procmon to confirm a sample's C2 network connection — illustrating the core principle of never relying on a single tool's output.

