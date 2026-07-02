PowerShell Fundamentals
 
### What is PowerShell?
 
Per Microsoft: *"PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework."*
 
Unlike traditional text-based shells (`cmd.exe`), PowerShell is **object-oriented** — built on the .NET framework — meaning commands pass structured objects between one another rather than raw text strings. Originally Windows-only, PowerShell Core (2016) extended support to **macOS and Linux**.
 
#### Brief History
 
| Year | Milestone |
|---|---|
| Early 2000s | `cmd.exe` and batch files struggled with enterprise automation demands |
| 2006 | PowerShell 1.0 released — object-oriented approach using .NET, designed by Jeffrey Snover |
| 2016 | PowerShell Core released — open-source, cross-platform (Windows, macOS, Linux) |
 
> **Key insight (Snover's observation):** Windows uses structured data and APIs; Unix treats everything as text. Rather than porting Unix text tools to Windows, Snover built an object-oriented equivalent that aligned with how Windows actually works.
 
---
 
### Objects in PowerShell
 
An **object** encapsulates both **data (properties)** and **actions (methods)**:
 
| Concept | Definition | Example |
|---|---|---|
| **Property** | A characteristic or attribute of the object | File name, size, extension |
| **Method** | An action the object can perform | Copy a file, stop a process |
 
> In traditional `cmd.exe`, commands output **plain text** that must be parsed to extract meaning. In PowerShell, commands output **objects** — structured data that retains properties and methods, enabling direct manipulation without string parsing.
 
---
 
### Launching PowerShell
 
| Method | Steps |
|---|---|
| **Start Menu** | Search "powershell" → open Windows PowerShell |
| **Run Dialog** | `Win + R` → type `powershell` → Enter |
| **File Explorer** | Type `powershell` in address bar → Enter (opens in current folder) |
| **Task Manager** | File → Run new task → type `powershell` |
| **From CMD** | Type `powershell` and press Enter |
 
```cmd
C:\Users\captain> powershell
PS C:\Users\captain>
```
 
> The `PS` prefix in the prompt indicates an active PowerShell session.
 
---
 
### Cmdlet Syntax — `Verb-Noun`
 
PowerShell commands are called **cmdlets** (pronounced *command-lets*). They follow a strict `Verb-Noun` naming convention:
 
| Cmdlet | Verb | Noun | Function |
|---|---|---|---|
| `Get-Content` | Get | Content | Retrieves the content of a file |
| `Set-Location` | Set | Location | Changes the current directory |
| `Get-Process` | Get | Process | Lists running processes |
| `Remove-Item` | Remove | Item | Deletes a file or directory |
 
> This consistent naming means that once you know the verb and noun vocabulary, you can often guess a cmdlet's name before looking it up.
 
---
 
### Essential Discovery Cmdlets
 
#### `Get-Command` — List Available Commands
 
```powershell
Get-Command                          # all available cmdlets, functions, aliases, scripts
Get-Command -CommandType "Function"  # filter by type
```
 
> Returns `CommandInfo` objects — enabling filtering on any property.
 
#### `Get-Help` — Documentation
 
```powershell
Get-Help Get-Date           # basic help page
Get-Help Get-Date -Examples # practical usage examples
Get-Help Get-Date -Detailed # detailed parameter descriptions
Get-Help Get-Date -Full     # complete technical reference
Get-Help Get-Date -Online   # opens the Microsoft docs page
```
 
#### `Get-Alias` — Command Shortcuts
 
PowerShell includes aliases for familiar `cmd.exe` and Unix commands, easing the transition:
 
| Alias | Full Cmdlet |
|---|---|
| `dir` | `Get-ChildItem` |
| `cd` | `Set-Location` |
| `cat` | `Get-Content` |
| `ls` | `Get-ChildItem` |
| `cp` | `Copy-Item` |
| `mv` | `Move-Item` |
| `rm` | `Remove-Item` |
 
```powershell
Get-Alias   # list all defined aliases
```
 
---
 
### Module Management
 
Modules are collections of cmdlets distributed via repositories (e.g., **PowerShell Gallery**).
 
> Note: The following require internet access and are unavailable in offline environments.
 
```powershell
Find-Module -Name "PowerShell*"        # search for modules by name/pattern
Install-Module -Name "PowerShellGet"   # install a module from the gallery
```
 
---
 
### File System Operations
 
| Task | Cmdlet | CMD Equivalent |
|---|---|---|
| List contents | `Get-ChildItem` | `dir` |
| Change directory | `Set-Location -Path <path>` | `cd` |
| Create file/directory | `New-Item -Path <path> -ItemType <File\|Directory>` | `mkdir` / (no unified cmd) |
| Remove file/directory | `Remove-Item -Path <path>` | `del` / `rmdir` |
| Copy | `Copy-Item -Path <src> -Destination <dest>` | `copy` |
| Move/rename | `Move-Item -Path <src> -Destination <dest>` | `move` |
| Read file contents | `Get-Content -Path <file>` | `type` |
 
#### Examples
 
```powershell
# List current directory
Get-ChildItem
 
# Navigate into Documents
Set-Location -Path ".\Documents"
 
# Create a nested directory structure
New-Item -Path ".\logs\archive" -ItemType "Directory"
 
# Create a file
New-Item -Path ".\logs\output.txt" -ItemType "File"
 
# Copy a file
Copy-Item -Path .\report.txt -Destination .\report_backup.txt
 
# Read a file
Get-Content -Path ".\report.txt"
```
 
---
 
### Piping — Object-Based
 
PowerShell's pipe (`|`) passes **objects** — not text — between cmdlets. The receiving cmdlet can directly access any property of those objects without parsing.
 
```powershell
# Sort files by size
Get-ChildItem | Sort-Object Length
```
 
```
Mode    LastWriteTime  Length  Name
-a----  ...              0    captain-boots.txt
-a----  ...            264    captain-hat.txt
-a----  ...           2116    ship-flag.txt
```
 
> Compare this to CMD piping: in CMD, `dir | sort` would sort alphabetically on raw text output. In PowerShell, `Sort-Object Length` sorts on the `Length` **property of the actual file object** — unambiguous and precise.
 
---
 
### Filtering & Selection Cmdlets
 
#### `Where-Object` — Filter by Condition
 
```powershell
# Only .txt files
Get-ChildItem | Where-Object -Property "Extension" -eq ".txt"
 
# Files with names starting with "ship"
Get-ChildItem | Where-Object -Property "Name" -like "ship*"
 
# Files larger than 1000 bytes
Get-ChildItem | Where-Object -Property "Length" -gt 1000
```
 
#### Comparison Operators
 
| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-gt` | Greater than (strict) |
| `-ge` | Greater than or equal to |
| `-lt` | Less than (strict) |
| `-le` | Less than or equal to |
| `-like` | Matches a wildcard pattern (`*` = any characters) |
 
#### `Select-Object` — Choose Properties or Limit Results
 
```powershell
Get-ChildItem | Select-Object Name, Length
```
 
```
Name               Length
----               ------
captain-boots.txt       0
captain-hat.txt       264
ship-flag.txt        2116
```
 
#### `Sort-Object` — Sort by Property
 
```powershell
Get-ChildItem | Sort-Object Length          # ascending
Get-ChildItem | Sort-Object Length -Descending  # descending
```
 
#### `Select-String` — Search File Contents (like `grep`)
 
```powershell
Select-String -Path ".\logfile.txt" -Pattern "error"
```
 
> Fully supports **regular expressions (regex)** for complex pattern matching — more powerful than CMD's `findstr`.
 
---
 
### System Information Cmdlets
 
| Cmdlet | Purpose | CMD Equivalent |
|---|---|---|
| `Get-ComputerInfo` | Full system/OS/hardware snapshot | `systeminfo` (partial) |
| `Get-LocalUser` | All local user accounts | `net user` |
| `Get-NetIPConfiguration` | Network interfaces, IPs, DNS, gateway | `ipconfig` |
| `Get-NetIPAddress` | All configured IP addresses (active and inactive) | `ipconfig /all` (partial) |
 
#### Examples
 
```powershell
Get-ComputerInfo
# WindowsProductName : Windows Server 2022 Datacenter
# WindowsBuildLabEx  : 20348.859.amd64fre...
 
Get-LocalUser
# Name           Enabled  Description
# Administrator  True     Built-in administrator account
# captain        True     The beloved captain of this pirate ship.
 
Get-NetIPConfiguration
# InterfaceAlias : Ethernet
# IPv4Address    : 10.10.178.209
# DNSServer      : 10.0.0.2
 
Get-NetIPAddress
# Returns all IPs across all adapters including loopback and IPv6
```
 
---
 
### Process, Service & Network Monitoring
 
| Cmdlet | Purpose | CMD/Tool Equivalent |
|---|---|---|
| `Get-Process` | Running processes with CPU/memory stats | `tasklist` |
| `Get-Service` | Services and their status (Running/Stopped/Paused) | `sc query` / Services console |
| `Get-NetTCPConnection` | Active TCP connections and listening ports | `netstat` |
| `Get-FileHash` | Compute cryptographic hash (SHA256 by default) for a file | *(no CMD equivalent)* |
 
#### Examples
 
```powershell
Get-Process
# Handles  NPM(K)  PM(K)  WS(K)  CPU(s)  Id  ProcessName
# ...
# 309      13      18312  1256   0.52    1524  amazon-ssm-agent
 
Get-Service
# Status   Name           DisplayName
# Running  AmazonSSMAgent Amazon SSM Agent
# Stopped  AppIDSvc       Application Identity
 
Get-NetTCPConnection
# LocalAddress    LocalPort  RemoteAddress  RemotePort  State       OwningProcess
# 10.10.178.209   22         10.14.87.60    53523       Established 1444
 
Get-FileHash -Path .\report.txt
# Algorithm  Hash                        Path
# SHA256     54D2EC3C12BF3D...           C:\...\report.txt
```
 
---
 
### Viewing Alternate Data Streams (ADS)
 
```powershell
Get-Item -Path "C:\House\house_log.txt" -Stream *
```
 
```
Stream      Length
------      ------
:$DATA      13        # default data stream (normal file content)
housinginfo 21        # Alternate Data Stream — hidden from File Explorer
```
 
> ADS is an NTFS feature explored in [Section 32](#32-windows-internals--system-administration). PowerShell is one of the primary native tools for detecting and reading ADS content.
 
---
 
### Scripting & Remote Execution
 
#### Why PowerShell Scripting Matters for Cybersecurity
 
| Role | Application |
|---|---|
| **Blue Team** (incident response, threat hunting) | Automate log analysis, scan for IOCs, detect anomalies, reverse-engineer malware behavior |
| **Red Team** (pen testing) | System enumeration, remote command execution, obfuscated payload delivery |
| **System Administrators** | Enforce security policies, monitor health, respond to incidents at scale |
 
#### `Invoke-Command` — Remote Execution
 
Executes commands or scripts on **remote systems** — fundamental for both administration and penetration testing.
 
```powershell
# Run a local script on a remote computer
Invoke-Command -FilePath c:\scripts\test.ps1 -ComputerName Server01
 
# Run a command block on a remote computer with alternate credentials
Invoke-Command -ComputerName Server01 -Credential Domain01\User01 -ScriptBlock {
    Get-Culture
}
```
 
> The `-ScriptBlock { ... }` parameter accepts any sequence of commands — no pre-written script file required. The result is returned to the local session as if the command ran locally.
 
---
 
### PowerShell vs. CMD — Key Differences
 
| Aspect | CMD | PowerShell |
|---|---|---|
| Output format | Plain text | Structured objects |
| Piping | Text passed between commands | Objects passed — properties preserved |
| Filtering | Requires text parsing | Direct property-based filtering |
| Cross-platform | Windows only | Windows, macOS, Linux |
| Extensibility | Limited | Module system (PowerShell Gallery) |
| Remote execution | Limited | `Invoke-Command` — built-in |
| Scripting | Batch files (`.bat`) | PowerShell scripts (`.ps1`) |
 
---
 
### Quick Reference
 
| Task | Cmdlet |
|---|---|
| Discover available commands | `Get-Command` |
| Get help for a cmdlet | `Get-Help <Cmdlet> [-Examples\|-Full]` |
| List all aliases | `Get-Alias` |
| List directory contents | `Get-ChildItem` |
| Change directory | `Set-Location -Path <path>` |
| Create file or directory | `New-Item -Path <path> -ItemType <File\|Directory>` |
| Delete file or directory | `Remove-Item -Path <path>` |
| Copy item | `Copy-Item -Path <src> -Destination <dest>` |
| Move/rename item | `Move-Item -Path <src> -Destination <dest>` |
| Read file | `Get-Content -Path <file>` |
| Filter objects | `Where-Object -Property <prop> -<op> <value>` |
| Sort objects | `Sort-Object <property>` |
| Select properties | `Select-Object <prop1>, <prop2>` |
| Search file content | `Select-String -Path <file> -Pattern <regex>` |
| System information | `Get-ComputerInfo` |
| Local users | `Get-LocalUser` |
| Network configuration | `Get-NetIPConfiguration` |
| Running processes | `Get-Process` |
| Services | `Get-Service` |
| TCP connections | `Get-NetTCPConnection` |
| File hash | `Get-FileHash -Path <file>` |
| View ADS streams | `Get-Item -Path <file> -Stream *` |
| Run command remotely | `Invoke-Command -ComputerName <host> -ScriptBlock { <cmd> }` |
 
---
