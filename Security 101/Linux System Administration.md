Linux System Administration
 
### Terminal Text Editors
 
Storing multi-line content using `echo` and redirection operators (`>`/`>>`) is impractical beyond trivial cases. **Terminal text editors** solve this problem directly within the shell.
 
#### Nano
 
A beginner-friendly editor, opened by providing a filename (creating it if it doesn't exist):
 
```bash
nano myfile
```
 
```
  GNU nano 4.8                    myfile
 
Hello TryHackMe
I can write things into "myfile"
 
 
^G Get Help  ^O Write Out  ^W Where Is   ^K Cut Text   ^J Justify
^X Exit      ^R Read File  ^\ Replace    ^U Paste Text ^T To Spell
```
 
| Action | Shortcut |
|---|---|
| Exit | `Ctrl + X` |
| Write out (save) | `Ctrl + O` |
| Search | `Ctrl + W` |
| Cut text | `Ctrl + K` |
| Paste text | `Ctrl + U` |
| Go to line number | `Ctrl + _` |
 
> On Linux, `^` represents the `Ctrl` key — `^X` means `Ctrl + X`.
 
#### VIM
 
A more advanced, steeper-learning-curve editor favored for serious development work.
 
| Benefit | Description |
|---|---|
| **Customizable** | Keybindings can be remapped to preference |
| **Syntax highlighting** | Valuable for writing/reading code |
| **Universal availability** | Present on virtually all Linux systems, even where `nano` is absent |
 
> VIM has a substantial learning curve but offers far greater long-term efficiency; cheatsheets and dedicated tutorials are widely available.
 
#### Nano vs. VIM
 
| | Nano | VIM |
|---|---|---|
| Learning curve | Low | High |
| Customization | Minimal | Extensive |
| Syntax highlighting | Limited | Strong |
| Availability | Not always pre-installed | Nearly universal |
 
---
 
### File Transfer
 
#### `wget` — Downloading Files Over HTTP
 
Downloads a file from a web address, as if retrieving it through a browser.
 
```bash
wget https://assets.tryhackme.com/additional/linux-fundamentals/part3/myfile.txt
```
 
#### `scp` — Secure Copy Over SSH
 
Transfers files between two machines using the **SSH protocol**, providing both authentication and encryption — effectively `cp`, but across a network.
 
```
scp SOURCE DESTINATION
```
 
**Uploading a local file to a remote system:**
 
| Variable | Value |
|---|---|
| Remote IP | `192.168.1.30` |
| Remote user | `ubuntu` |
| Local file | `important.txt` |
| Remote destination filename | `transferred.txt` |
 
```bash
scp important.txt ubuntu@192.168.1.30:/home/ubuntu/transferred.txt
```
 
**Downloading a remote file to the local system:**
 
| Variable | Value |
|---|---|
| Remote IP | `192.168.1.30` |
| Remote user | `ubuntu` |
| Remote file | `documents.txt` |
| Local destination filename | `notes.txt` |
 
```bash
scp ubuntu@192.168.1.30:/home/ubuntu/documents.txt notes.txt
```
 
> **Pattern:** Whichever side of the colon (`:`) the path appears on indicates whether it refers to the **remote** machine. A bare path is always local.
 
#### Serving Files: Python's `http.server`
 
Python 3 ships with a built-in lightweight web server module, useful for quickly sharing files between machines on a network.
 
```bash
python3 -m http.server
```
 
```
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
 
> Serves files from the **current working directory** by default. The terminal running this command is occupied for as long as the server runs — a **separate terminal** is needed to download from it.
 
**Downloading from the served directory (from another machine/terminal):**
 
```bash
wget http://MACHINE_IP:8000/file
```
 
```
Connecting to http://127.0.0.1:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 51095 (50K) [text]
Saving to: 'file'
file 100%[=========================>] 49.90K in 0.04s
```
 
> **Limitation:** `http.server` has no built-in file index/browsing — you must already know the exact filename and path. More advanced lightweight alternatives (e.g., **Updog**) address this limitation.
 
#### File Transfer Method Comparison
 
| Method | Encrypted? | Authentication? | Use Case |
|---|---|---|---|
| `wget` (HTTP) | No (unless HTTPS) | No | Downloading publicly available files |
| `scp` (SSH) | Yes | Yes (credentials) | Secure transfer between trusted hosts |
| `python3 -m http.server` | No | No | Quick, informal sharing on a trusted local network |
 
---
 
### Process Management
 
A **process** is a running instance of a program, managed by the kernel and identified by a unique, sequentially-assigned **PID** (Process ID).
 
#### Viewing Processes: `ps`
 
```bash
ps        # processes in the current user's session
ps aux    # all processes, across all users, including system processes
```
 
| Flag Combination | Shows |
|---|---|
| `ps` | Processes in the current session only |
| `ps aux` | Every process on the system, regardless of owner (including `root` and system processes) |
 
#### Real-Time Monitoring: `top`
 
```bash
top
```
 
> Unlike the one-time snapshot from `ps`, `top` provides a continuously refreshing (every ~10 seconds, or on key press) live view of running processes and resource usage.
 
#### Terminating Processes: `kill`
 
```bash
kill <PID>
```
 
Sends a **signal** to a process, instructing it how to terminate:
 
| Signal | Effect |
|---|---|
| `SIGTERM` | Requests termination, allowing the process to clean up first |
| `SIGKILL` | Forces immediate termination — no cleanup |
| `SIGSTOP` | Suspends (pauses) the process without terminating it |
 
---
 
### How Processes Start
 
#### Namespaces
 
The OS uses **namespaces** to partition system resources (CPU, RAM, priority) among processes — similar to slicing a cake, where each process group ("slice") only gets a portion of total system resources.
 
> Namespaces also serve a **security** purpose: processes in different namespaces are generally isolated from and unaware of one another.
 
#### PID 0 and `systemd`
 
The very first process started at boot is assigned **PID 1** (commonly `systemd` on modern Ubuntu systems) — the system's **init** process, sitting between the kernel and user-launched programs.
 
```
Boot → systemd (PID 1) → Child Process A
                       → Child Process B
                       → Child Process C
```
 
> Every subsequently launched program runs as a **child process** of `systemd` (or further down the resulting process tree), inheriting and sharing resources while remaining individually identifiable and manageable.
 
---
 
### Managing Services with `systemctl`
 
`systemctl` allows interaction with `systemd`-managed services (daemons) — e.g., starting a web server automatically at boot.
 
```
systemctl [option] [service]
```
 
| Option | Effect |
|---|---|
| `start` | Starts the service immediately |
| `stop` | Stops the running service |
| `enable` | Configures the service to start automatically at boot |
| `disable` | Prevents the service from starting automatically at boot |
| `status` | Displays the service's current running state |
 
**Example:**
```bash
systemctl start apache2     # start the Apache web server now
systemctl enable apache2    # ensure Apache starts on every future boot
```
 
---
 
### Backgrounding & Foregrounding Processes
 
Processes can run in the **foreground** (occupying the terminal, returning output directly) or the **background** (running independently, freeing the terminal for other commands).
 
#### Sending a Process to the Background
 
| Method | When to Use |
|---|---|
| `command &` | Start a new command directly in the background |
| `Ctrl + Z` | Suspend/background an **already-running** foreground process |
 
```bash
echo "Hi THM" &
```
> Rather than printing the message, this returns a **process ID** — the command is now running in the background.
 
#### Bringing a Process Back: `fg`
 
```bash
fg
```
 
> Restores the most recently backgrounded process to the foreground, resuming visible output/interaction.
 
#### Foreground vs. Background Summary
 
| State | Terminal Behavior | Output Visibility |
|---|---|---|
| **Foreground** | Occupied until command finishes | Direct, immediate |
| **Background** | Freed for other commands | Suppressed (unless explicitly redirected) |
 
---
 
### Task Scheduling: `cron` & Crontabs
 
**Cron** is a system process that executes scheduled tasks ("cron jobs") at specified times, managed via **crontab** files.
 
#### Crontab Field Structure
 
A crontab entry requires **6 fields**, in order:
 
| Field | Description |
|---|---|
| `MIN` | Minute to execute (0–59) |
| `HOUR` | Hour to execute (0–23) |
| `DOM` | Day of month (1–31) |
| `MON` | Month (1–12) |
| `DOW` | Day of week (0–6) |
| `CMD` | The actual command to run |
 
#### Worked Example: Backup Every 12 Hours
 
```cron
0 */12 * * * cp -R /home/cmnatic/Documents /var/backups/
```
 
| Field | Value | Meaning |
|---|---|---|
| `MIN` | `0` | At minute 0 |
| `HOUR` | `*/12` | Every 12 hours |
| `DOM` | `*` | Any day of the month |
| `MON` | `*` | Any month |
| `DOW` | `*` | Any day of the week |
 
> The **asterisk (`*`)** acts as a wildcard, meaning "any value" for that field — used whenever a specific schedule constraint doesn't apply.
 
#### Editing Crontabs
 
```bash
crontab -e
```
 
> Opens the current user's crontab in a configured text editor (e.g., `nano`) for direct editing.
 
> **Helper tools:** Online resources such as crontab generators and schedule-explanation sites can help translate between human-readable schedules and crontab syntax while learning.
 
---
 
### Package Management
 
#### Repositories
 
Software is distributed via **repositories** — registries that package maintainers submit approved software to, which `apt` then references when installing or updating packages.
 
| Repository Type | Description |
|---|---|
| **Official/vendor repositories** | Maintained directly by the OS vendor (e.g., Ubuntu) |
| **Community/third-party repositories** | Added manually for software not in the default repository set |
 
#### Adding a Repository — `add-apt-repository`
 
```bash
sudo add-apt-repository ppa:some/repository
```
 
Or manually, by creating a `.list` file under `/etc/apt/sources.list.d/`.
 
#### Manual Repository Addition Workflow
 
**1. Trust the publisher's GPG key:**
```bash
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```
 
> **GPG keys** verify software integrity and authenticity — confirming that a package genuinely originates from its claimed publisher and hasn't been tampered with. If the key doesn't match what the system trusts, the install will fail.
 
**2. Create a dedicated source list file:**
```bash
sudo nano /etc/apt/sources.list.d/sublime-text.list
```
 
> Best practice: use a **separate file per third-party repository**, keeping sources organized and easy to remove individually later.
 
**3. Refresh package lists:**
```bash
sudo apt update
```
 
**4. Install the software:**
```bash
sudo apt install sublime-text
```
 
#### Removing a Repository & Package
 
```bash
sudo add-apt-repository --remove ppa:PPA_Name/ppa
sudo apt remove sublime-text
```
 
> Repositories can also be removed by manually deleting the corresponding `.list` file from `/etc/apt/sources.list.d/`.
 
#### Repository Workflow Summary
 
```
Trust GPG Key → Add Repository Source → apt update → apt install <package>
```
 
---
 
### Log Files
 
Located primarily in **`/var/log`**, log files record activity from running services and the operating system itself. Most distributions automatically manage log growth through **log rotation** — archiving or discarding old entries to prevent unbounded disk usage.
 
#### Common Service Logs
 
| Service | Purpose |
|---|---|
| **Apache2** | Web server request/error logs |
| **fail2ban** | Monitors and blocks repeated authentication failures (brute-force attempts) |
| **UFW** | Firewall activity logs |
 
#### Key Log Types
 
| Log Type | Contents |
|---|---|
| **Access log** | Records of every request made to a service (e.g., web server hits) |
| **Error log** | Records of failures, exceptions, and abnormal conditions |
 
> Logs serve a dual purpose: **operational** (diagnosing performance issues) and **security** (investigating unauthorized access attempts or intrusions). Authentication logs in particular are critical for identifying brute-force attempts or unauthorized login activity.
 
---
 
### Quick Reference
 
| Task | Command Pattern |
|---|---|
| Edit a file with Nano | `nano <filename>` |
| Download a file via HTTP | `wget <url>` |
| Copy a file to a remote host | `scp <local_file> <user>@<ip>:<remote_path>` |
| Copy a file from a remote host | `scp <user>@<ip>:<remote_path> <local_file>` |
| Start a quick web server | `python3 -m http.server` |
| List current user's processes | `ps` |
| List all system processes | `ps aux` |
| Live process monitoring | `top` |
| Terminate a process | `kill <PID>` |
| Start/stop/enable/disable a service | `systemctl [start\|stop\|enable\|disable\|status] <service>` |
| Background a running process | `Ctrl + Z` |
| Bring a background process to foreground | `fg` |
| Edit the current user's crontab | `crontab -e` |
| Update package lists | `sudo apt update` |
| Install a package | `sudo apt install <package>` |
| Remove a package | `sudo apt remove <package>` |
 
---
