Linux Search Tools & Shell Operators
 
### Where Linux Is Used
 
Linux is frequently perceived as more intimidating than consumer operating systems like Windows, but it is **lightweight, efficient, and ubiquitous** — often running invisibly behind everyday technology:
 
| Domain | Examples |
|---|---|
| **Web** | The majority of websites you visit |
| **Automotive** | Car entertainment/control panels |
| **Retail** | Point of Sale (PoS) systems, checkout tills |
| **Critical Infrastructure** | Traffic light controllers, industrial sensors |
 
---
 
### Foundational Commands
 
| Command | Description |
|---|---|
| `echo` | Outputs the provided text |
| `whoami` | Displays the currently logged-in username |
 
### Filesystem Navigation Commands
 
| Command | Full Name | Purpose |
|---|---|---|
| `ls` | **l**i**s**ting | Lists directory contents |
| `cd` | **c**hange **d**irectory | Moves between directories |
| `cat` | **cat**enate | Displays file contents |
| `pwd` | **p**rint **w**orking **d**irectory | Shows current location |
 
> These commands become muscle-memory with regular use, and form the foundation for more advanced, automated workflows — such as the search tools below.
 
---
 
### Using `find`
 
The `find` command searches the filesystem for files matching specified criteria, eliminating the need to manually navigate (`cd`/`ls`) through every directory by hand.
 
#### Searching by Exact Filename
 
```bash
find -name passwords.txt
```
 
**Example:**
```bash
tryhackme@linux1:~$ find -name passwords.txt
./folder1/passwords.txt
```
 
> Searches the current directory and all subdirectories, returning the relative path to any match.
 
#### Searching by Pattern (Wildcards)
 
When the exact filename is unknown, a **wildcard** (`*`) can match any file matching a pattern — such as a file extension.
 
```bash
find -name *.txt
```
 
**Example:**
```bash
tryhackme@linux1:~$ find -name *.txt
./folder1/passwords.txt
./Documents/todo.txt
```
 
> Returns every `.txt` file found beneath the current directory, regardless of filename.
 
---
 
### Using `grep`
 
While `find` locates **files**, `grep` searches **inside file contents** for matching text — essential when a file is too large to review manually.
 
#### Basic Usage
 
```bash
grep "search_term" filename
```
 
**Example scenario:** A web server access log (`access.log`) contains 244 entries:
 
```bash
tryhackme@linux1:~$ wc -l access.log
244 access.log
```
 
Manually reading 244 lines to find a specific IP address is impractical. Instead:
 
```bash
tryhackme@linux1:~$ grep "81.143.211.90" access.log
81.143.211.90 - - [25/Mar/2021:11:17 +0000] "GET / HTTP/1.1" 200 417 "-" "Mozilla/5.0 (Linux; Android 7.0; Moto G(4))"
```
 
> `grep` scans the entire file and returns only lines containing the specified value — in this case, every log entry from a given IP address.
 
#### Recursive Searching with `grep -R`
 
When the target information may be spread across **multiple files** within a directory tree, the `-R` (recursive) flag searches every file in the current directory and all subdirectories.
 
```bash
grep -R "PRETTY_NAME" /etc/
```
 
**Example:**
```bash
tryhackme@linux1:~$ grep -R "PRETTY_NAME" /etc/
grep: /etc/sudoers: Permission denied
/etc/os-release:PRETTY_NAME="Ubuntu"
```
 
> The matched file's path is shown before the matching line — making it straightforward to identify exactly where a result was found, even across an entire directory tree. (Permission-denied messages for restricted files are expected and can be ignored.)
 
#### `find` vs. `grep`
 
| Tool | Searches For | Use When |
|---|---|---|
| `find` | **Filenames** matching a pattern | You know (or can guess) part of a filename |
| `grep` | **Text content** inside files | You know a value/string that should appear *inside* a file |
 
---
 
### Shell Operators
 
| Operator | Description |
|---|---|
| `&` | Runs a command in the background |
| `&&` | Chains commands together; the next command only runs if the previous one succeeded |
| `>` | Redirects output to a file, **overwriting** existing contents |
| `>>` | Redirects output to a file, **appending** to existing contents |
 
#### `&` — Background Execution
 
Running a long process (e.g., copying a large file) normally blocks the terminal until it finishes. Appending `&` runs it in the background instead, freeing up the terminal for other work.
 
```bash
cp large_file.iso /backup/ &
```
 
#### `&&` — Conditional Command Chaining
 
Runs multiple commands sequentially on a single line — but **only proceeds to the next command if the previous one succeeded**.
 
```bash
command1 && command2
```
 
> If `command1` fails, `command2` will **not** execute. This is useful for ensuring dependent steps only run when prerequisites have genuinely completed.
 
#### `>` — Output Redirection (Overwrite)
 
Sends a command's output to a file instead of the terminal screen. If the target file already exists, its contents are **completely replaced**.
 
```bash
echo hey > welcome
cat welcome
# Output: hey
```
 
> ⚠️ **Caution:** Using `>` on an existing file destroys its previous contents with no warning or confirmation.
 
#### `>>` — Output Redirection (Append)
 
Functions identically to `>`, but **adds** the new output to the end of the file rather than overwriting it.
 
```bash
echo hey > welcome      # welcome now contains: hey
echo hello >> welcome   # welcome now contains: hey \n hello
cat welcome
# Output:
# hey
# hello
```
 
#### `>` vs. `>>` Comparison
 
| Operator | Existing File Contents | Result |
|---|---|---|
| `>` | Destroyed | File contains **only** the new output |
| `>>` | Preserved | File contains old output **followed by** new output |
 
---
