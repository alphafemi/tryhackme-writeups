Linux Terminal Fundamentals
 
### Navigation Commands
 
| Command | Purpose |
|---|---|
| `pwd` | **P**rint **W**orking **D**irectory — shows your current location in the filesystem |
| `ls` | Lists files and directories in the current location |
| `ls -l` | Long listing — shows permissions, owner, size, and modification date |
| `ls -al` | Long listing including **hidden files** (those prefixed with `.`) |
| `cd <directory>` | Changes into the specified directory |
| `cd ..` | Moves up one directory level |
 
#### Example Session
 
```bash
ubuntu@tryhackme:~$ pwd
/home/ubuntu
 
ubuntu@tryhackme:~$ ls
Desktop  Downloads  Pictures  Templates  logs
Documents  Music  Public  Videos  projects
 
ubuntu@tryhackme:~$ cd Documents/
ubuntu@tryhackme:~/Documents$ pwd
/home/ubuntu/Documents
 
ubuntu@tryhackme:~/Documents$ cd ..
ubuntu@tryhackme:~$ pwd
/home/ubuntu
```
 
> **Hidden files** are not a security feature — they are simply filenames beginning with a `.` (e.g., `.bashrc`, `.Xauthority`) that Linux excludes from default `ls` output for readability. Use `ls -al` to reveal them.
 
### Locating Files: `find`
 
Searches recursively through a directory tree for files matching a given name.
 
```bash
find <starting_point> -name <filename>
```
 
**Example:**
```bash
ubuntu@tryhackme:~$ find ~ -name mission_brief.txt
/home/ubuntu/projects/mission_brief.txt
```
 
| Component | Meaning |
|---|---|
| `~` | Shorthand for the current user's home directory |
| `-name` | Matches files by exact filename |
 
> `find` traverses every subdirectory beneath the starting point, so searches from broad locations (e.g., `/`) can take noticeably longer than scoped searches (e.g., `~`).
 
### Reading File Contents: `cat`
 
Outputs the full contents of a file directly to the terminal.
 
```bash
cat <filename>
```
 
```bash
ubuntu@tryhackme:~/projects$ cat mission_brief.txt
Great job finding your way around the terminal.
...
```
 
### System Information Commands
 
| Command | Returns |
|---|---|
| `whoami` | The currently logged-in username |
| `uname` | The kernel/OS name only (e.g., `Linux`) |
| `uname -a` | Full system information: kernel, hostname, version, architecture, OS type |
| `df -h` | Disk usage in human-readable format |
| `cat /etc/os-release` | Distribution name, version, and codename |
 
#### `whoami`
 
```bash
ubuntu@tryhackme:~$ whoami
ubuntu
```
 
#### `uname -a`
 
```bash
ubuntu@tryhackme:~$ uname -a
Linux tryhackme 6.8.0-aws #17-Ubuntu SMP Mon Sep 2 13:48:07 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```
 
| Field | Meaning |
|---|---|
| `Linux` | Kernel name |
| `tryhackme` | Hostname |
| `6.8.0-aws` | Kernel version |
| `x86_64` | Hardware architecture (64-bit) |
| `GNU/Linux` | OS type — Linux kernel paired with GNU userland tools |
 
> Use plain `uname` for just the kernel name; use `uname -a` for the complete picture.
 
#### `df -h` — Disk Usage
 
```bash
ubuntu@tryhackme:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        30G   12G   17G  17% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           774M  1.2M  773M   1% /run
```
 
| Flag/Field | Meaning |
|---|---|
| `-h` | Human-readable sizes (e.g., `12G` instead of raw bytes) |
| `/dev/root` | The primary disk/filesystem |
| `tmpfs` | Temporary filesystems held in RAM, not persistent storage |
| `/dev/shm` | Shared memory region used for inter-process communication |
 
#### `/etc/os-release` — Distribution Details
 
The `/etc` directory holds system-wide configuration files. `os-release` reliably identifies the Linux distribution in use:
 
```bash
ubuntu@tryhackme:/etc$ cat os-release
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
```
 
> This file is generally more precise than `uname -a` for identifying the **distribution** (as opposed to the kernel) — useful when determining package managers, supported software repositories, or compatibility requirements.
 
### Command Summary
 
| Task | Command |
|---|---|
| Show current directory | `pwd` |
| List directory contents | `ls` / `ls -l` / `ls -al` |
| Change directory | `cd <path>` / `cd ..` |
| Search for a file | `find <path> -name <filename>` |
| Read a file | `cat <filename>` |
| Show current user | `whoami` |
| Show kernel/system info | `uname` / `uname -a` |
| Show disk usage | `df -h` |
| Show distribution info | `cat /etc/os-release` |
 
---
