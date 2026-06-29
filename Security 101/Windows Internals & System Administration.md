Windows Internals & System Administration
 
### The NTFS File System
 
Modern Windows uses **NTFS** (New Technology File System), succeeding earlier systems:
 
| File System | Era / Use Today |
|---|---|
| **FAT16 / FAT32** | Predecessor to NTFS; still common on USB drives, MicroSD cards |
| **HPFS** | Predecessor file system, largely historical |
| **NTFS** | Standard on modern Windows PCs and servers |
 
> To check your system's file system: right-click the system drive (typically `C:\`) → **Properties**.
 
#### NTFS Capabilities Over FAT
 
| Feature | Description |
|---|---|
| **Journaling** | Logs changes before committing them, allowing automatic repair after a failure — not possible with FAT |
| **Large file support** | Supports individual files larger than 4GB |
| **Granular permissions** | Per-file and per-folder access control |
| **Compression** | Native folder/file compression |
| **Encryption** | Built-in via **EFS** (Encrypting File System) |
 
---
 
### NTFS Permissions
 
NTFS volumes support setting explicit permissions to grant or deny access:

| Permission | Description |
|---|---|
| **Full control** | Complete access — read, write, modify, delete, change permissions |
| **Modify** | Read, write, and delete |
| **Read & Execute** | View contents and run executables |
| **List folder contents** | View what's inside a folder (folders only) |
| **Read** | View contents only |
| **Write** | Add/modify contents |
 
#### Viewing Permissions
 
1. Right-click the file/folder → **Properties**
2. Select the **Security** tab
3. Select a user/group in the list to view their specific permissions
---
 
### Alternate Data Streams (ADS)
 
A Windows NTFS-specific feature allowing a file to hold **more than one stream of data**. Every file has at least one default stream (`$DATA`); ADS permits additional hidden streams attached to the same file.
 
| Use Case | Legitimate or Malicious? |
|---|---|
| Marking files downloaded from the internet | Legitimate (built-in Windows behavior) |
| Hiding malicious payloads inside an otherwise normal-looking file | Malicious |
 
> File Explorer does **not** natively display ADS content. Detection typically requires **PowerShell** or third-party tools.
 
---
 
### The Windows Directory & Environment Variables
 
The `C:\Windows` folder traditionally holds the core OS files — though its actual location is configurable and referenced via the **`%windir%`** environment variable, rather than being hardcoded.
 
> **Environment variables** store operating-system-level configuration data — paths, processor counts, temp folder locations, and more — referenced dynamically by the OS and applications rather than hardcoded.
 
#### `System32`
 
A critical subfolder within `Windows` containing files essential to core OS operation.
 
---
 
### User Account Types
 
| Account Type | Capabilities |
|---|---|
| **Administrator** | Full system control — add/remove users, modify groups and settings, install software |
| **Standard User** | Limited to personal files/folders — cannot perform system-level changes |
 
#### Viewing User Accounts
 
| Method | Path |
|---|---|
| Settings | Start Menu → search "Other User" → **Settings → Other users** |
| Local Users and Groups | `Win + R` → `lusrmgr.msc` |
 
> Only an **Administrator** account sees the "Add someone else to this PC" option — Standard Users do not.
 
#### User Profiles
 
Each user account has a profile folder under `C:\Users\<username>` (e.g., `C:\Users\Max`), created automatically on **first login** — visible briefly as the "User Profile Service" message during the login process.
 
Standard folders within every profile include:
 
```
C:\Users\<username>\
├── Desktop
├── Documents
├── Downloads
├── Music
└── Pictures
```
 
#### Local Users and Group Management (`lusrmgr.msc`)
 
Accessible via `Win + R` → `lusrmgr.msc`. Contains two sections:
 
| Section | Contents |
|---|---|
| **Users** | All local user accounts |
| **Groups** | Local groups, each with a description and an assigned set of permissions |
 
> Assigning a user to a group grants that user the group's permissions. A single user can belong to **multiple** groups simultaneously.
 
---
 
### User Account Control (UAC)
 
Most home users are logged in as local administrators by default — meaning malware running in that user's context inherits the same elevated rights, increasing compromise risk.
 
**UAC**, introduced with Windows Vista, mitigates this: even an administrator's session runs with **standard** privileges by default. When an action requires elevated rights, UAC prompts for explicit confirmation.
 
> **Note:** UAC does **not** apply to the built-in local administrator account by default.
 
#### Visual Indicator
 
Programs requiring elevation display a **shield icon** overlay — signaling that launching them will trigger a UAC prompt.
 
#### UAC Behavior for Standard Users
 
When a Standard User attempts an elevated action, the UAC prompt requests an **administrator's** credentials (username pre-filled, password required). If no password is entered within a timeout period, the prompt disappears and the action is canceled.
 
#### UAC Notification Levels
 
| Level | Behavior |
|---|---|
| **Always notify** | Highest security — notifies for any change (apps or user), dims the desktop (Secure Desktop) |
| **Notify for apps** (default) | Notifies only when apps attempt changes — not for manual Windows settings changes |
| **Notify without dimming** | Same as above, but screen does not dim |
| **Never notify** | No notifications at all — **not recommended** |
 
---
 
### System Configuration (`msconfig`)
 
A utility for advanced troubleshooting, primarily focused on diagnosing **startup issues**. Requires local administrator rights.
 
#### Tabs Overview
 
| Tab | Purpose |
|---|---|
| **General** | Boot mode: Normal, Diagnostic, or Selective |
| **Boot** | OS boot options/parameters |
| **Services** | Lists all configured services, running or stopped |
| **Startup** | Directs to Task Manager (modern Windows) for managing startup items |
| **Tools** | Quick-launch shortcuts to other diagnostic utilities |
 
> **Note:** `msconfig` is **not** a startup management tool on modern Windows — Microsoft directs users to **Task Manager** instead. On **Windows Server** systems specifically, neither Task Manager nor `msconfig` display startup items; instead, use `Win + R` → `shell:startup` to open the Startup folder directly and inspect shortcuts/executables configured to launch at login.
 
---
 
### Advanced System Settings
 
Accessed via search: **"View advanced system settings."**
 
#### Page File (Virtual Memory)
 
Windows uses a **page file** as overflow virtual memory when physical RAM is exhausted, preventing slowdowns or crashes.
 
**Path:** Advanced tab → Performance **Settings** → Advanced tab → Virtual Memory
 
| Configurable Value | Description |
|---|---|
| Drive | Which drive stores the page file |
| Initial size | Starting allocation (MB) |
| Maximum size | Upper allocation limit (MB) |
| Auto-management | Whether Windows manages sizing automatically |
 
#### Startup & Recovery (Crash Dumps)
 
Configures what data Windows captures during a critical failure (e.g., Blue Screen of Death), under **Advanced → Startup and Recovery → Settings**.
 
| Dump Type | Captures |
|---|---|
| **Automatic memory dump** | System-managed, balanced detail |
| **Kernel memory dump** | Kernel-level memory only |
| **Small memory dump (256 KB)** | Minimal — basic crash info only |
| **Complete memory dump** | Full memory contents — largest, most detailed |
| **None** | No dump generated |
 
---
 
### Computer Management (`compmgmt.msc`)
 
A consolidated console with three primary sections: **System Tools**, **Storage**, and **Services and Applications**.
 
#### System Tools
 
##### Task Scheduler
 
Creates and manages tasks that run automatically — at login/logoff, on a recurring schedule, or at a specific one-time point.
 
```
SystemInfoDailyLog → Trigger: Daily at 10:00 AM
One-off example → Trigger: "At 2:50 PM on 6/15/2025"
```
 
> Create new scheduled tasks via **Create Basic Task** (Actions pane).
 
##### Event Viewer
 
Displays a historical record (audit trail) of system, security, and application events — used for diagnosing issues and investigating activity.
 
**Layout:**
 
| Pane | Contents |
|---|---|
| Left | Hierarchical tree of event log providers |
| Middle | Summary/overview for the selected provider |
| Right | Actions pane |
 
##### Shared Folders
 
Lists all shared folders accessible to other systems, including default administrative shares (`C$`, `ADMIN$`).
 
| Subsection | Shows |
|---|---|
| **Shares** | All shared folders, including built-in administrative shares |
| **Sessions** | Users currently connected to shares |
| **Open Files** | Files/folders currently being accessed by connected users |
 
##### Local Users and Groups
 
Same functionality as `lusrmgr.msc` (covered above), embedded within Computer Management.
 
##### Performance Monitor (`perfmon`)
 
Displays real-time or logged performance data — used to troubleshoot performance issues locally or remotely.
 
##### Device Manager
 
View and configure attached hardware, including the ability to disable specific devices.
 
#### Storage
 
| Tool | Use |
|---|---|
| **Disk Management** | Set up new drives, extend/shrink partitions, assign drive letters |
| **Windows Server Backup** | Server-specific backup configuration (not covered in depth here) |
 
#### Services and Applications
 
##### Services
 
Lists all services and their current status. Right-click → **Properties** for details including the internal service name, executable path, and **Startup type**:
 
| Startup Type | Behavior |
|---|---|
| **Automatic** | Starts on every system boot |
| **Manual** | Starts only when triggered by another process or user |
| **Disabled** | Will not run |
 
##### WMI Control
 
Configures **Windows Management Instrumentation (WMI)** — enables scripting languages (PowerShell, VBScript) to manage Windows systems locally or remotely.
 
> The legacy command-line tool **WMIC** is deprecated as of Windows 10 21H1; **PowerShell** is now the recommended interface for WMI tasks.
 
---
 
### Diagnostic & Information Tools
 
#### System Information (`msinfo32`)
 
Provides a comprehensive snapshot of hardware, components, and software — useful for diagnosing system issues.
 
| Section | Contents |
|---|---|
| **Hardware Resources** | Low-level hardware/resource allocation details (advanced) |
| **Components** | Installed hardware devices and their status |
| **Software Environment** | Installed software, environment variables, network connections |
 
> Includes a built-in **search bar** for quickly locating specific values (e.g., searching "IP address" within Components).
 
#### Resource Monitor (`resmon`)
 
Provides detailed per-process and aggregate resource usage across four categories:
 
| Tab | Tracks |
|---|---|
| **CPU** | Per-process and total CPU usage |
| **Memory** | Per-process and total memory usage |
| **Disk** | Disk read/write activity per process |
| **Network** | Network activity per process |
 
> Also includes a process-analysis feature for identifying **deadlocked processes** and **file-locking conflicts** — useful for resolving issues without forcibly closing an unresponsive application (avoiding potential data loss).
 
---
 
### Command Prompt Basics
 
| Command | Purpose |
|---|---|
| `hostname` | Displays the computer's network name |
| `whoami` | Displays the currently logged-in username |
| `ipconfig` | Displays network address configuration |
| `netstat` | Displays protocol statistics and active TCP/IP connections |
| `net` | Manages network resources (uses sub-commands) |
| `cls` | Clears the terminal screen |
 
#### Getting Help
 
```cmd
ipconfig /?
```
> Most commands support `/?` for an in-terminal syntax/parameter reference.
 
> **Exception:** The `net` command does **not** use `/?` — instead, use:
> ```cmd
> net help
> net help user
> ```
 
---
 
### The Windows Registry
 
A central hierarchical database storing configuration data referenced continually by Windows during operation, including:
 
- User profile configuration
- Installed application/file-type associations
- Folder/icon property sheet settings
- Installed hardware inventory
- Active port usage
**Accessed via:** `regedit`
 
> ⚠️ **Warning:** The registry is intended for advanced users — incorrect modifications can destabilize or break normal system operation.
 
---
 
### Windows Security
 
Central dashboard for Windows' built-in protections, organized into:
 
| Section | Focus |
|---|---|
| **Virus & threat protection** | Malware detection/removal |
| **Firewall & network protection** | Network traffic control |
| **App & browser control** | SmartScreen, exploit protection |
| **Device security** | Hardware-based protections (TPM, etc.) |
 
#### Status Color Coding
 
| Color | Meaning |
|---|---|
| 🟢 Green | Sufficiently protected — no action needed |
| 🟡 Yellow | Safety recommendation to review |
| 🔴 Red | Immediate attention required |
 
---
 
### Virus & Threat Protection
 
#### Scan Options
 
| Scan Type | Coverage |
|---|---|
| **Quick scan** | Commonly-targeted folders only |
| **Full scan** | All files and running programs (can take over an hour) |
| **Custom scan** | User-selected files/locations |
 
#### Threat History
 
| Category | Meaning |
|---|---|
| **Last scan** | Most recent automatic scan result |
| **Quarantined threats** | Isolated, prevented from running (periodically purged) |
| **Allowed threats** | Detected items the user explicitly permitted to run |
 
#### Protection Settings
 
| Setting | Function |
|---|---|
| **Real-time protection** | Actively blocks malware installation/execution |
| **Cloud-delivered protection** | Faster protection via live cloud threat data |
| **Automatic sample submission** | Sends suspicious files to Microsoft for analysis |
| **Controlled folder access** | Blocks unauthorized apps from modifying protected folders — required for ransomware protection |
| **Exclusions** | Excludes specified files/folders from scanning (reduces false positives, but creates blind spots) |
 
> ⚠️ Excluded items are **not scanned at all** — only configure exclusions when fully confident in their safety.
 
---
 
### Firewall & Network Protection
 
A firewall controls traffic flow through network **ports** — functioning as a checkpoint for everything attempting to enter or exit the system.
 
#### Firewall Profiles
 
| Profile | Applies To |
|---|---|
| **Domain** | Networks where the system authenticates to a domain controller |
| **Private** | User-designated trusted networks (home, lab) |
| **Public** | Default/untrusted networks (coffee shops, airports, public Wi-Fi) |
 
Each profile supports:
- Turning the firewall **on/off**
- **Block all incoming connections**
- Per-application allow rules (viewable/configurable individually)
> **Quick access command:** `WF.msc` opens Windows Defender Firewall with Advanced Security directly.
 
---
 
### App & Browser Control
 
#### Microsoft Defender SmartScreen
 
Protects against phishing/malware sites and malicious file downloads.
 
| Setting | Effect |
|---|---|
| **Warn** | Alerts the user but allows proceeding |
| **Block** | Prevents the action outright |
| **Off** | No protection (not recommended) |
 
#### Exploit Protection
 
Built-in mitigations against common exploitation techniques — default settings are recommended unless you have specific, informed reasons to adjust them.
 
---
 
### Device Security
 
#### Core Isolation — Memory Integrity
 
Prevents malicious code injection into high-security system processes.
 
#### Trusted Platform Module (TPM)
 
A **hardware-based** secure crypto-processor performing cryptographic operations with built-in tamper-resistance — malicious software cannot interfere with its core security functions.
 
#### BitLocker Drive Encryption
 
Protects against data exposure from lost, stolen, or improperly decommissioned devices by encrypting the entire drive.
 
> Provides the **strongest** protection when paired with a **TPM (v1.2+)** — the TPM helps verify the system hasn't been tampered with while offline.
 
---
 
### Volume Shadow Copy Service (VSS)
 
Creates **shadow copies** (snapshots/point-in-time backups) of data, stored in the hidden `System Volume Information` folder on each protected drive.
 
When **System Protection** is enabled, VSS supports:
 
- Creating a **restore point**
- Performing a **system restore**
- Configuring restore settings
- Deleting existing restore points
> ⚠️ **Security note:** Ransomware frequently targets and deletes Shadow Copy files specifically to prevent recovery — **offline or off-site backups** remain the only reliable safeguard against this tactic.
 
---
 
### Quick Reference
 
| Tool | Launch Command | Purpose |
|---|---|---|
| Local Users and Groups | `lusrmgr.msc` | Manage local users/groups |
| System Configuration | `msconfig` | Boot/startup/service diagnostics |
| Computer Management | `compmgmt.msc` | Consolidated admin console |
| Windows Firewall (Advanced) | `WF.msc` | Firewall rule configuration |
| Registry Editor | `regedit` | Edit the Windows Registry |
| System Information | `msinfo32` | Hardware/software diagnostic summary |
| Resource Monitor | `resmon` | Detailed per-process resource usage |
| Run Dialog (general launcher) | `Win + R` | Quick-launch any of the above |
| Startup folder (servers) | `shell:startup` | View startup shortcuts directly |
 
| Command (CMD) | Purpose |
|---|---|
| `hostname` | Show computer name |
| `whoami` | Show current user |
| `ipconfig` | Show network configuration |
| `netstat` | Show network connections/statistics |
| `net help <subcommand>` | Help for `net` sub-commands |
| `cls` | Clear the screen |
