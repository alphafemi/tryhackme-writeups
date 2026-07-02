Windows Command Line — Networking, Files & Processes

### Environment & System Basics
 
#### Checking the Executable Path
 
Windows only runs commands that exist within directories listed in the `PATH` environment variable. View all current environment variables (including `Path`) with:
 
```cmd
set
```
 
```
ALLUSERSPROFILE=C:\ProgramData
OS=Windows_NT
Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;...
```
 
> If a command is not found, the executable may exist on disk but its location is not in `Path`.
 
#### OS Version
 
```cmd
ver
```
 
```
Microsoft Windows [Version 10.0.17763.1821]
```
 
#### Detailed System Information
 
```cmd
systeminfo
```
 
```
Host Name:     WIN-SRV-2019
OS Name:       Microsoft Windows Server 2019 Datacenter
OS Version:    10.0.17763 N/A Build 17763
```
 
#### Paging Long Output
 
For commands producing lengthy output (e.g., `driverquery`, `systeminfo`), pipe through `more` to view the result one page at a time:
 
```cmd
driverquery | more
```
 
> Press `Space` to advance one page, `Enter` to advance one line, `Ctrl + C` to exit.
 
---
 
### Networking Commands
 
#### `ipconfig` — Network Configuration
 
| Variant | Returns |
|---|---|
| `ipconfig` | IP address, subnet mask, default gateway |
| `ipconfig /all` | Full details — MAC address, DHCP status, DNS servers, lease times |
 
```cmd
C:\>ipconfig /all
 
Ethernet adapter Ethernet 3:
   IPv4 Address. . . . . . . : 10.10.230.237
   Subnet Mask . . . . . . . : 255.255.0.0
   Default Gateway . . . . . : 10.10.0.1
   DHCP Enabled. . . . . . . : Yes
   DHCP Server . . . . . . . : 10.10.0.1
   DNS Servers . . . . . . . : 10.0.0.2
```
 
#### `ping` — Connectivity Testing
 
Sends ICMP echo request packets to a target and measures round-trip response time.
 
```cmd
ping example.com
```
 
```
Reply from 93.184.215.14: bytes=32 time=78ms TTL=52
...
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Average round trip times: 78ms
```
 
> No reply (`Request timed out`) indicates the host is unreachable — or is configured to silently drop ICMP packets.
 
#### `tracert` — Trace Route
 
Identifies the sequence of routers ("hops") a packet traverses to reach a destination, using TTL-based ICMP expiry notifications.
 
```cmd
tracert example.com
```
 
```
Tracing route to example.com [93.184.215.14]
 
  1    42ms   ec2-3-248-240-3.eu-west-1.compute.amazonaws.com
  2     *  *  *  Request timed out.
  ...
 16    78ms   93.184.215.14
 
Trace complete.
```
 
> `* * *` indicates a hop that does not respond to ICMP (firewall-filtered) — not necessarily a network failure.
 
#### `nslookup` — DNS Lookup
 
Resolves a hostname to its IP address(es), optionally using a specific name server.
 
```cmd
nslookup example.com             # uses the system's default DNS server
nslookup example.com 1.1.1.1    # queries Cloudflare's public DNS (1.1.1.1)
```
 
```
Server:  one.one.one.one
Address: 1.1.1.1
 
Name:    example.com
Addresses: 2606:2800:21f:cb07:6820:80da:af6b:8b2c
           93.184.215.14
```
 
> Useful for verifying DNS resolution and confirming which name server is answering queries.
 
#### `netstat` — Active Connections
 
Displays current network connections and listening ports.
 
| Flag | Effect |
|---|---|
| *(none)* | Established connections only |
| `-a` | All connections **and** listening ports |
| `-b` | Executable associated with each connection |
| `-o` | Process ID (PID) associated with each connection |
| `-n` | Use numeric addresses/ports instead of resolving names |
 
**Combined usage (`-abon`):**
 
```cmd
netstat -abon
```
 
```
Proto  Local Address       Foreign Address     State       PID
TCP    0.0.0.0:22          0.0.0.0:0           LISTENING   2116
[sshd.exe]
TCP    10.10.230.237:22    10.11.81.126:53486  ESTABLISHED 2116
[sshd.exe]
```
 
> Combining `-a`, `-b`, `-o`, and `-n` gives a comprehensive view: all ports, the responsible executable, the PID, and addresses in numeric form — particularly useful for identifying unexpected listeners or active connections.
 
#### Networking Command Summary
 
| Command | Purpose |
|---|---|
| `ipconfig` | Basic IP/gateway/subnet info |
| `ipconfig /all` | Full adapter details incl. MAC, DHCP, DNS |
| `ping <host>` | Test reachability and measure latency |
| `tracert <host>` | Trace the route to a destination |
| `nslookup <host>` | DNS name-to-IP resolution |
| `nslookup <host> <server>` | DNS lookup via a specific name server |
| `netstat` | View established connections |
| `netstat -abon` | Full view: all ports, executables, PIDs, numeric |
 
---
 
### Directory Operations
 
| Command | Purpose |
|---|---|
| `cd` | Show current directory |
| `cd <path>` | Change to directory |
| `cd ..` | Move up one level |
| `dir` | List directory contents |
| `dir /a` | Include hidden and system files |
| `dir /s` | Recurse into subdirectories |
| `tree` | Visual tree of subdirectory structure |
| `mkdir <name>` | Create a directory |
| `rmdir <name>` | Remove a directory |
 
#### `tree` Example
 
```cmd
C:\Users\strategos>tree
C:.
├───Desktop
├───Documents
├───Downloads
├───Music
└───Pictures
```
 
---
 
### File Operations
 
| Command | Purpose |
|---|---|
| `type <file>` | Print file contents to the terminal |
| `more <file>` | View file contents one page at a time |
| `copy <src> <dest>` | Copy a file |
| `move <src> <dest>` | Move (or rename) a file |
| `del <file>` / `erase <file>` | Delete a file |
 
#### Using Wildcards
 
The `*` wildcard matches multiple filenames:
 
```cmd
copy *.md C:\Markdown
```
 
> Copies **all** `.md` files in the current directory to `C:\Markdown`.
 
#### `type` vs. `more` for Files
 
| Command | Best For |
|---|---|
| `type <file>` | Short files that fit within the terminal window |
| `more <file>` | Long files — displays one page at a time (Space = next page, Enter = next line) |
 
---
 
### Process Management
 
#### Listing Processes: `tasklist`
 
```cmd
tasklist
```
 
```
Image Name                PID   Session Name   Mem Usage
svchost.exe               704   Services       23,432 K
lsass.exe                 592   Services       16,108 K
...
```
 
**Filtering by image name:**
 
```cmd
tasklist /FI "imagename eq sshd.exe"
```
 
```
Image Name   PID    Session Name   Mem Usage
sshd.exe     2116   Services       6,992 K
sshd.exe     2712   Services       7,668 K
```
 
> Use `tasklist /?` to see all available filter options.
 
#### Terminating a Process: `taskkill`
 
```cmd
taskkill /PID <pid>
```
 
> Equivalent of Task Manager's "End Task," but usable from the command line — useful for terminating unresponsive or targeted processes by PID.
 
---
 
### Additional Useful Commands
 
| Command | Purpose |
|---|---|
| `chkdsk` | Checks disk volume/filesystem for errors and bad sectors |
| `driverquery` | Lists all installed device drivers |
| `sfc /scannow` | Scans for and repairs corrupted system files |
| `cls` | Clears the terminal screen |
| `help <command>` | Help for a specific command |
| `<command> /?` | Inline help for most commands |
 
> **Exception:** `net` does not support `/?` — use `net help` or `net help <subcommand>` instead.
 
---
 
### `more` — Dual Usage Summary
 
`more` serves two distinct roles throughout the Windows CLI:
 
| Use | Syntax | Effect |
|---|---|---|
| **View a text file** | `more <file.txt>` | Displays file contents one page at a time |
| **Page long command output** | `<command> \| more` | Pipes any command's output through the pager |
 
---
