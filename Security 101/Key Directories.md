Linux File Management, Permissions & Key Directories
 
### Command Arguments: Flags & Switches
 
Most Linux commands accept **arguments** — identified by a hyphen plus a keyword — that modify their default behavior. These are commonly called **flags** or **switches**.
 
```bash
ls          # default behavior — hidden files excluded
ls -a       # extended behavior — hidden files (--all) included
```
 
**Example:**
```bash
tryhackme@linux2:~$ ls
folder1
 
tryhackme@linux2:~$ ls -a
.hiddenfolder  folder1
```
 
> Without specifying flags, a command performs its **default** behavior only. Flags extend or alter that behavior — in this case, revealing files prefixed with `.` that are hidden by default.
 
---
 
### Discovering Available Options: `--help` and `man`
 
#### `--help`
 
Most commands that accept flags also support `--help`, which prints a condensed summary of available options directly to the terminal.
 
```bash
ls --help
```
 
```
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
 
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all            do not list implied . and ..
  -l                          use a long listing format
  --color[=WHEN]               colorize the output
  ...
```
 
#### `man` — The Manual Pages
 
The **manual pages** (`man`) provide the full, authoritative documentation for a command or application — available both locally on the system and online.
 
```bash
man ls
```
 
```
LS(1)                     User Commands                    LS(1)
 
NAME
       ls - list directory contents
 
SYNOPSIS
       ls [OPTION]... [FILE]...
 
DESCRIPTION
       List information about the FILEs (the current directory by default).
       ...
```
 
> `--help` gives a quick summary; `man` provides the complete reference, including detailed descriptions, examples, and related commands. Press `q` to exit a man page; press `h` for in-page help.
 
---
 
### File & Folder Management Commands
 
| Command | Full Name | Purpose |
|---|---|---|
| `touch` | touch | Create an empty file |
| `mkdir` | make directory | Create a folder |
| `cp` | copy | Copy a file or folder |
| `mv` | move | Move or rename a file or folder |
| `rm` | remove | Delete a file or folder |
| `file` | file | Identify a file's actual type/contents |
 
> **Tip:** All of these commands accept full paths (e.g., `directory1/directory2/note`), not just filenames in the current directory.
 
---
 
### Creating Files and Folders
 
#### `touch` — Create a File
 
Creates an empty file. Content must be added separately (e.g., via `echo` or a text editor like `nano`).
 
```bash
touch note
```
 
```bash
tryhackme@linux2:~$ touch note
tryhackme@linux2:~$ ls
folder1  note
```
 
#### `mkdir` — Create a Directory
 
```bash
mkdir mydirectory
```
 
```bash
tryhackme@linux2:~$ mkdir mydirectory
tryhackme@linux2:~$ ls
folder1  mydirectory  note
```
 
---
 
### Removing Files and Folders: `rm`
 
```bash
rm <filename>          # remove a file
rm -R <directory>       # remove a directory and its contents recursively
```
 
**Example:**
```bash
tryhackme@linux2:~$ rm note
tryhackme@linux2:~$ ls
folder1  mydirectory
 
tryhackme@linux2:~$ rm -R mydirectory
tryhackme@linux2:~$ ls
folder1
```
 
> `-R` (recursive) is **required** when removing a directory — `rm` alone will not delete folders. There is no built-in recovery mechanism for `rm` — exercise caution, especially with recursive or wildcard usage.
 
---
 
### Copying and Moving: `cp` & `mv`
 
#### `cp` — Copy
 
Takes two arguments: the existing file, and the name for the new copy. The original remains unchanged.
 
```bash
cp note note2
```
 
```bash
tryhackme@linux2:~$ cp note note2
tryhackme@linux2:~$ ls
folder1  note  note2
```
 
#### `mv` — Move / Rename
 
Also takes two arguments, but rather than duplicating the file, it **relocates or renames** it — the original location no longer holds the file.
 
```bash
mv note2 note3
```
 
```bash
tryhackme@linux2:~$ mv note2 note3
tryhackme@linux2:~$ ls
folder1  note  note3
```
 
> `mv` serves a dual purpose: moving a file into a different directory, **or** renaming it in place (moving it to a new name within the same directory).
 
#### `cp` vs. `mv`
 
| Command | Original File | Result |
|---|---|---|
| `cp source dest` | Remains | A new, independent copy is created |
| `mv source dest` | Removed | The file is relocated/renamed — no duplicate exists |
 
---
 
### Determining File Type: `file`
 
File extensions (e.g., `.txt`) are a **convention**, not a guarantee — a file's actual content may not match its name or extension. The `file` command inspects file content directly to determine its true type.
 
```bash
file note
```
 
```bash
tryhackme@linux2:~$ file note
note: ASCII text
```
 
---
 
### File Permissions
 
#### Viewing Permissions: `ls -l`
 
```bash
ls -lh
```
 
```
-rw-r--r-- 1 cmnatic cmnatic 0 Feb 19 10:37 file1
-rw-r--r-- 8 cmnatic cmnatic 0 Feb 19 10:37 file2
```
 
The first column (e.g., `-rw-r--r--`) encodes the file's permissions, split into **three groups of three characters**:
 
| Section | Applies To |
|---|---|
| First 3 characters | **Owner** |
| Next 3 characters | **Group** |
| Last 3 characters | **Others** |
 
Each group can contain:
 
| Symbol | Permission |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute |
| `-` | Permission not granted |
 
#### Users vs. Groups
 
Linux permissions are granular: a file has exactly one **owner**, but can also be associated with a **group** of users, who may be granted the same or different permissions — without altering the owner's own access.
 
> **Real-world example:** A web server process needs read/write access to serve and update files. A hosting provider wants customers to upload their own site files — but without granting them the same system privileges as the web server itself. Group-based permissions make this separation possible.
 
---
 
### Numeric (Octal) Permission Notation
 
Symbolic permissions (`rwxr-xr-x`) can be expressed more compactly as a 3-digit number, where each permission carries a fixed numeric value:
 
| Permission | Value |
|---|---|
| Read (`r`) | 4 |
| Write (`w`) | 2 |
| Execute (`x`) | 1 |
 
To calculate a group's numeric value, **sum** the values of its granted permissions.
 
#### Worked Example: `rwxrwxrwx`
 
| Group | Permissions | Calculation | Value |
|---|---|---|---|
| Owner | `rwx` | 4+2+1 | 7 |
| Group | `rwx` | 4+2+1 | 7 |
| Others | `rwx` | 4+2+1 | 7 |
 
```
rwxrwxrwx  =  777
```
 
#### Additional Common Examples
 
| Symbolic | Numeric | Meaning |
|---|---|---|
| `rwxr-xr-x` | `755` | Owner: full access; Group/Others: read + execute |
| `rw-r--r--` | `644` | Owner: read/write; Group/Others: read only |
| `rwx------` | `700` | Owner only — no access for anyone else |
 
#### Applying Permissions: `chmod`
 
```bash
chmod 750 system_overview.txt
```
 
| Digit | Group | Meaning |
|---|---|---|
| `7` | Owner | Full access (read, write, execute) |
| `5` | Group | Read + execute |
| `0` | Others | No access |
 
> Numeric notation is the standard format for tools and scripts (e.g., `chmod 755 file`) — understanding it is essential both for correctly configuring access and for **spotting overly permissive configurations** that pose a security risk.
 
---
 
### Switching Between Users: `su`
 
The `su` (substitute user) command allows switching to another user account, provided you know:
 
1. The **username** to switch to
2. That user's **password**
```bash
su <username>
```
 
```bash
tryhackme@linux2:~$ su user2
Password:
user2@linux2:/home/tryhackme$
```
 
> Note: without `-l`, the new session remains in the **previous** user's working directory.
 
#### `su -l` (or `--login`)
 
Starts a full login shell for the target user — inheriting their environment variables and dropping into **their own** home directory, much like a genuine fresh login.
 
```bash
su -l <username>
```
 
```bash
tryhackme@linux2:~$ su -l user2
Password:
user2@:~$ pwd
/home/user2
```
 
| Variant | Resulting Directory | Environment |
|---|---|---|
| `su user2` | Stays in the **current** user's directory | Inherits current shell's environment |
| `su -l user2` | Switches to **user2's own** home directory | Fresh environment, as if user2 logged in directly |
 
---
 
### Key System Directories
 
#### `/etc`
 
Stores **system-wide configuration files** used by the operating system. ("etc" historically stands for "etcetera.")
 
| File | Purpose |
|---|---|
| `sudoers` | Lists users/groups permitted to run `sudo` (commands as root) |
| `passwd` | Stores user account information |
| `shadow` | Stores encrypted user passwords (typically SHA-512 hashed) |
 
```bash
tryhackme@linux2:/etc$ ls
shadow  passwd  sudoers  sudoers.d
```
 
#### `/var`
 
Stores **variable data** — content that changes frequently as services and applications run.
 
| Subdirectory | Contents |
|---|---|
| `/var/log` | Log files from system services and applications |
| `/var/tmp`, `/var/backups`, `/var/opt` | Application-specific data, backups, optional package data |
 
```bash
tryhackme@linux2:/var$ ls
backups  log  opt  tmp
```
 
#### `/root`
 
The **home directory for the `root` user** specifically — distinct from `/home/<username>` directories used by regular users.
 
```bash
root@linux2:~# ls
myfile  myfolder  passwords.xlsx
```
 
#### `/tmp`
 
A **volatile, temporary storage** location — contents are cleared on system restart, similar in concept to RAM.
 
```bash
root@linux2:/tmp# ls
todelete  trash.txt  rubbish.bin
```
 
> **Security relevance:** `/tmp` is writable by any user by default, making it a common location to stage scripts or tools temporarily during a penetration test once initial access to a system has been obtained.
 
---
 
### Key Directory Summary
 
| Directory | Purpose | Persistence |
|---|---|---|
| `/etc` | System-wide configuration files | Persistent |
| `/var` | Variable/frequently-changing data (logs, etc.) | Persistent |
| `/root` | Home directory of the `root` user | Persistent |
| `/tmp` | Temporary, world-writable scratch space | Cleared on reboot |
