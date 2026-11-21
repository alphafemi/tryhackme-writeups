# Windows Command & Control, Persistence & Impact

## Overview

This document covers how a compromised Windows host becomes part of a larger attack: Command and Control (with and without a dedicated C2 channel), persistence through backdoored users (creation, privilege escalation, and password resets) and malware persistence via services, scheduled tasks, the Startup folder, and Registry Run keys with the event IDs that detect each, the reasons attackers maintain access, and a closing look at Active Directory, ransomware, and the full threat detection chain.

---

What happens when attackers aren't satisfied with just breaching a host, but want to stay, establish control, and use it as a launchpad for future operations? This section covers how a compromised host becomes part of a larger attack, focusing on **Command and Control**, **Persistence**, and **Impact**.

---

### Command and Control (C2)

The MITRE **Command and Control** tactic covers how threat actors send commands and keep control of a victim host after the breach.

**Attacks without C2:** Sometimes no C2 is needed — e.g., threat actors type commands directly in an active RDP session after an RDP breach. But since this access disappears the moment RDP is closed or secured, most attackers still set up a dedicated C2 channel immediately.

**Simplest C2:** For non-interactive Initial Access methods (phishing, USB), attackers need a process that connects back to them and waits for commands 24/7.

- In the simplest case, the phishing attachment itself is that process (e.g., a Cobalt Strike beacon)
- In more advanced cases, the initial attachment downloads a separate C2 payload, hides it (e.g., in `C:\Temp`), and runs it as a new stealthy process — this way the C2 survives even if the victim deletes the original attachment (seen in recent ransomware cases and the APT29 phishing campaign)

---

### Persistence Overview

Data-stealer infections are often "smash and grab" — breach, collect, exfiltrate, exit, all within minutes. Most other attacks, however, need reliable access that survives for days or months, including reboots and password changes. This is the **Persistence** tactic.

---

### Persisting via RDP — Backdoored Users

Many Windows breaches start from an exposed service (weak-password RDP, vulnerable mail server, misconfigured web app). Attackers can keep re-using that same service until it's fixed, but they often add an extra Persistence layer too:

- Create an additional hidden vulnerability in the service itself (backdoor, web shell)
- **Create a new user (T1136), make it an administrator (T1098)**, and use it for further RDP logins

User management can be done via GUI (`lusrmgr.msc` / Computer Management) or command line:

```
# 1. Two methods to create the "mr.backd00r" user
CMD:        net user "mr.backd00r" "p@ssw0rd!" /add
PowerShell: New-LocalUser "mr.backd00r" -Password [...]

# 2. Two methods to add the user to Administrators
CMD:        net localgroup Administrators "mr.backd00r" /add
PowerShell: Add-LocalGroupMember "Administrators" -Member "mr.backd00r"
```

#### Detecting Backdoored Users

Every user-creation event logs as **Security Event ID 4720**. Since attackers can name accounts however they like, don't rely on spotting obvious names — instead investigate:

- Who created the account, and can they confirm doing so?
- What source IP and time is tied to the creator's login — expected or not?
- What other suspicious events appear in the creator's session?

**Making users privileged:** A plain new user has no RDP or admin rights by default, so attackers add the account to a privileged group (most commonly **Administrators** or **Remote Desktop Users**) — logged as **Security Event ID 4732**.

**Resetting passwords:** In more advanced cases, attackers skip account creation entirely and simply reset the password of an old/unused account — logged as **Security Event ID 4724**.

---

### Malware Persistence

Backdoored-user persistence requires RDP access; phishing and USB infections need a different approach — actively running malware that re-establishes its C2 connection even after a reboot.

#### Services and Tasks

There are well over a hundred Windows persistence techniques, but two are by far the most common:

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| **Create a Windows Service** (runs on OS startup) | `sc create "BadService" binpath= "C:\malware.exe" start= auto` | Launch of `sc.exe`: Sysmon Event ID 1 · Service creation: **Security 4697** |
| **Create a Scheduled Task** (runs on OS startup) | `schtasks /create /tn "BadTask" /tr "C:\malware.exe" /sc onstart /ru System` | Launch of `schtasks.exe`: Sysmon Event ID 1 · Task creation: **Security 4698** |

**Detecting Services:** Services are viewable via `services.msc`; creating or modifying one requires admin privileges and `sc.exe`. Detect malicious services three ways:

1. Launch of `sc.exe create` via Sysmon Event ID 1
2. Service creation via **Security 4697** or **System 7045**
3. Suspicious processes with a `services.exe` parent

**Detecting Tasks:** Scheduled tasks are managed via `taskschd.msc` or `schtasks.exe`, and are easier to configure/hide than services — making them the most common persistence method in the wild (seen in both APT28 and APT41 attacks). Detect them three ways:

1. Launch of `schtasks.exe /create` via Sysmon Event ID 1
2. Task creation via **Security 4698**
3. Suspicious processes with a `svchost.exe [...] -s Schedule` parent

#### Run Keys and Startup

Services and scheduled tasks need admin rights and run at boot. For per-user, login-triggered persistence (used by both legitimate tools and malware), Windows offers two lighter-weight mechanisms:

| Persistence Method | Attack Example | Event ID Logging |
|---|---|---|
| **Startup Folder** (runs on user login) | `copy C:\malware.exe "%AppData%\Microsoft\Windows\Start Menu\Programs\Startup\malware.exe"` | New startup item: **Sysmon Event ID 11** |
| **Registry "Run" Keys** (runs on user login) | `reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v BadKey /t REG_SZ /d "C:\malware.exe"` | New registry value: **Sysmon Event ID 13** |

**Startup folder locations:**

```
C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp   (all users)
```

The folder is normally near-empty, so any dropped file is suspicious (e.g., the Lumma Stealer used this technique). Detect via **Sysmon Event ID 11** file-creation events inside the Startup folder — but note that programs launched from Startup will show an `explorer.exe` parent, which can blend in with legitimate activity.

**Run key locations:**

```
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run   (all users)
```

View entries via `regedit.exe`. Run-key persistence shares the same MITRE technique as the Startup folder — the only real difference is the mechanism (registry value vs. dropped file). Detect via **Sysmon Event ID 13** registry-change events targeting the Run keys.

---

### Need for Persistence

Why maintain access instead of just stealing data and leaving? Common reasons:

- **Add the host to a botnet** — e.g., the Kraken Botnet combines crypto-mining, data-stealing, and C2 capability in one
- **Long-term espionage** — e.g., Volt Typhoon remained undetected in the US electric grid for nearly a year
- **Network entry point** — using the host as a beachhead for a much larger breach, sometimes taking months (one case documented 29 days spent breaching a full network)

---

### Active Directory and Ransomware

A Windows *network* usually means a large **Active Directory** deployment, with its own attack surface — the most dangerous outcome being **ransomware**, which can halt an entire organization. The McLaren hospital ransomware incident affected **743,000 patients** and included ransom notes printed automatically on office printers alongside encryption and data theft.

### Threat Detection Recap

Active Directory and ransomware are large topics on their own, but every complex attack starts from a single simple breach. The overall chain covered across this Windows threat detection track: **Initial Access → Discovery → Collection/Credential Access → Exfiltration → Command and Control → Persistence → Impact** — all mapped against the MITRE ATT&CK matrix. The earlier a SOC analyst can detect and interrupt this chain, ideally right after Initial Access, the better the outcome.

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| C2 without a beacon | Live RDP session; commands typed directly |
| Simplest C2 | Phishing attachment itself calls home, or downloads a separate hidden payload |
| Backdoor account creation | `net user ... /add` / `New-LocalUser` → **Security 4720** |
| Privilege escalation of backdoor account | `net localgroup Administrators ... /add` → **Security 4732** |
| Password reset abuse | **Security 4724** |
| Service persistence | `sc create ... start= auto` → Sysmon 1 (launch) + **Security 4697** (creation) |
| Scheduled task persistence | `schtasks /create ... /sc onstart` → Sysmon 1 (launch) + **Security 4698** (creation) |
| Startup folder persistence | File dropped in `...\Start Menu\Programs\Startup\` → **Sysmon 11** |
| Run key persistence | `reg add HKCU\...\CurrentVersion\Run` → **Sysmon 13** |
| Most common persistence method | Scheduled tasks (easy to configure/hide) |
| Persistence motives | Botnet, espionage, long-term network entry point |
| Worst-case Impact | Full-network ransomware (e.g., McLaren hospitals, 743,000 patients) |
| Overall kill chain covered | Initial Access → Discovery → Collection → Exfiltration → C2 → Persistence → Impact |

---

## Summary

- **Command and Control** covers how attackers send commands and keep control after the breach: sometimes none is needed (typing directly in a live RDP session), but since that access vanishes once RDP is closed or secured, most attackers set up a dedicated channel — either the phishing attachment itself as a Cobalt Strike-style beacon, or a separately downloaded payload hidden in a folder like `C:\Temp` so the C2 survives even if the original attachment is deleted.
- **Persistence** keeps access alive through reboots and password changes; with RDP access, attackers create a **backdoored user** (**T1136**) and make it an administrator (**T1098**), detected through **Security 4720** (account creation — investigate who created it and from where), **4732** (added to Administrators or Remote Desktop Users), and **4724** (password reset of an old or unused account).
- **Malware persistence** relies mainly on **services** (`sc create`, detected via Sysmon 1, **Security 4697**, **System 7045**, or a `services.exe` parent) and **scheduled tasks** (`schtasks /create`, detected via Sysmon 1, **Security 4698**, or a `svchost.exe ... -s Schedule` parent) — scheduled tasks being the most common method seen in the wild (APT28, APT41) — plus per-user login persistence through the **Startup folder** (Sysmon **11**) and **Registry Run keys** (Sysmon **13**).
- Attackers maintain access to add the host to a **botnet** (Kraken), run **long-term espionage** (Volt Typhoon stayed in the US electric grid for nearly a year), or use it as a **network entry point** for a larger breach (one case took 29 days).
- The worst outcome is **ransomware** across an **Active Directory** network, as in the McLaren hospital incident (743,000 patients, with ransom notes printed on office printers), and the full chain — **Initial Access → Discovery → Collection/Credential Access → Exfiltration → Command and Control → Persistence → Impact** — maps onto MITRE ATT&CK, with earlier interruption (ideally right after Initial Access) giving the better outcome.

