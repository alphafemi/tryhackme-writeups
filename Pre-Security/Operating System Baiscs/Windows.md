Windows Command Line Fundamentals
 
The Windows command line (**Command Prompt**) provides the same category of text-based control as a Linux terminal — direct, scriptable interaction with the operating system, favored by IT and security professionals for speed, precision, and access to tools unavailable through the GUI.
 
### Navigation Commands
 
| Command | Purpose |
|---|---|
| `cd` | Displays the current directory (no argument) |
| `cd <folder>` | Changes into the specified folder |
| `cd ..` | Moves up one directory level |
| `dir` | Lists visible files and folders in the current directory |
| `dir /a` | Lists **all** items, including hidden files and folders |
 
#### Example Session
 
```cmd
C:\Users\analyst> cd
C:\Users\analyst
 
C:\Users\analyst> dir
 Directory of C:\Users\analyst
 
 <DIR>  Desktop
 <DIR>  Documents
 <DIR>  Downloads
 <DIR>  Pictures
 ...
 16 Dir(s)
 
C:\Users\analyst> cd Documents
C:\Users\analyst\Documents> cd ..
C:\Users\analyst>
```
 
> As with Linux, **hidden** does not mean **secret** — Windows simply excludes certain files/folders from the default `dir` listing for clarity. `dir /a` reveals them.
 
### Locating Files: `dir /s`
 
Searches recursively through the current directory and all subfolders for a matching filename.
 
```cmd
dir /s <filename>
```
 
**Example:**
```cmd
C:\Users\analyst> dir /s task_brief.txt
 Directory of C:\Users\analyst\Documents\Projects
 
02/10/2026  10:48 AM    1,204 task_brief.txt
```
 
| Flag | Meaning |
|---|---|
| `/s` | Searches the current directory **and all subdirectories** |
 
Once located, navigate to the reported path and confirm with `dir`:
 
```cmd
C:\Users\analyst> cd Documents\Projects
C:\Users\analyst\Documents\Projects> dir
task_brief.txt
```
 
### Reading File Contents: `type`
 
Displays the contents of a text file directly in the terminal — the Windows equivalent of Linux's `cat`.
 
```cmd
type <filename>
```
 
```cmd
C:\Users\analyst\Documents\Projects> type task_brief.txt
I've left a file called task_brief.txt somewhere in your user folder...
```
 
### System Information Commands
 
| Command | Returns |
|---|---|
| `whoami` | The currently logged-in username |
| `hostname` | The name assigned to the computer on the network |
| `systeminfo` | Detailed OS, hardware, and configuration report |
| `ipconfig` | Network configuration (IP address, gateway, adapters) |
 
#### `whoami`
 
```cmd
C:\Users\analyst> whoami
desktop-kje57fd\analyst
```
 
#### `hostname`
 
```cmd
C:\Users\analyst> hostname
DESKTOP-KJE57FD
```
 
Useful for identifying a specific machine within a larger network inventory.
 
#### `systeminfo`
 
Returns a comprehensive report; the most relevant fields for quick identification are:
 
| Field | Indicates |
|---|---|
| **OS Name** | The Windows edition installed |
| **OS Version** | Specific build/version number |
| **System Type** | Architecture — 32-bit or 64-bit |
 
```cmd
C:\Users\analyst> systeminfo
 
OS Name:      Microsoft Windows 11 Enterprise
OS Version:   10.0.22631 N/A Build 22631
System Type:  x64-based PC
```
 
#### `ipconfig`
 
Displays the machine's network configuration — essential for understanding how a host connects to its network.
 
```cmd
C:\Users\analyst> ipconfig
 
Ethernet adapter Ethernet:
   IPv4 Address. . . . . . . . . . . : 192.168.1.74
   Default Gateway . . . . . . . . . : 192.168.1.254
```
 
| Field | Meaning |
|---|---|
| **IPv4 Address** | The device's private address on the local network |
| **Default Gateway** | The router address used to reach external networks |
 

 
### Command Summary
 
| Task | Command |
|---|---|
| Show current directory | `cd` |
| List directory contents | `dir` / `dir /a` |
| Change directory | `cd <path>` / `cd ..` |
| Search for a file | `dir /s <filename>` |
| Read a file | `type <filename>` |
| Show current user | `whoami` |
| Show computer name | `hostname` |
| Show OS/system details | `systeminfo` |
| Show network configuration | `ipconfig` |
 
