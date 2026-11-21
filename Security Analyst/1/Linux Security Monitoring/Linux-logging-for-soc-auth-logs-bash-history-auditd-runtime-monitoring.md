# Linux Logging for SOC — Auth Logs, Bash History & Auditd Runtime Monitoring

## Overview

This document covers the most common Linux logs a SOC analyst sees in a SIEM and how to read them directly on a host: the plain-text log format and how to filter and discover logs, authentication logs (`auth.log` login/logout, SSH, user management, and sudo events), generic system and app-specific logs, Bash history and its limitations, system calls and why runtime monitoring matters, auditd rules and key fields with worked `ausearch` examples, and auditd alternatives.

---

Linux has long led servers and embedded systems, and cloud adoption has only widened its footprint. SOC analysts are increasingly likely to investigate Linux alerts and incidents, from traditional on-premises servers to cloud-native containerized workloads.

---

### Log Format

Linux is not immune to malware, and Linux-targeted intrusions are a growing problem. Scoping notes:

- "Linux" here refers to distributions like Debian, Ubuntu, CentOS, or RHEL
- This section focuses on **Linux servers without a GUI**, not desktop logging

### Working With Logs

Unlike Windows, Linux logs most events as **plain text**, readable in any text editor — no specialized tool like Event Viewer required. The tradeoff: Linux logs are less structured, with no event codes or strict formatting rules. Most logs live under **`/var/log`**, starting with the aggregated system event stream `/var/log/syslog`:

```
root@thm-vm:~$ cat /var/log/syslog | head
2025-08-13T13:57:49.388941+00:00 thm-vm systemd-timesyncd[268]: Initial clock synchronization...
2025-08-13T13:59:39.970029+00:00 thm-vm systemd[888]: Starting dbus.socket...
2025-08-13T14:02:22.606216+00:00 thm-vm dbus-daemon[564]: [system] Successfully activated service...
2025-08-13T14:05:01.999677+00:00 thm-vm CRON[1027]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
```

**Filtering logs** — with thousands of lines per file, `grep` is essential:

```bash
# Only CRON-related entries
cat /var/log/syslog | grep CRON

# Exclude CRON entries
cat /var/log/syslog | grep -v CRON
```

**Discovering logs** — when you don't know which file holds what you need, list `/var/log` and grep broadly across it:

```bash
# List what's logged by your system
ls -l /var/log

# Search for potential logins across all logs
grep -R -E "auth|login|session" /var/log
```

> **Caveat:** Unlike Windows, Linux lets you freely change log format, verbosity, and storage location. With hundreds of distributions each customizing logging slightly, expect variation — some logs described here may look different, or not exist at all, on a given system.

---

### Authentication Logs — `/var/log/auth.log`

The most useful log to monitor is **`/var/log/auth.log`** (or **`/var/log/secure`** on RHEL-based systems). Despite the name, it covers far more than plain authentication — user management events, sudo command execution, and more. Format: event time, hostname, generating service, and an unstructured text message.

#### Login and Logout Events

Users can authenticate locally, via SSH, via `sudo`/`su`, or automatically (cron). Every session start/end is logged with `session opened` / `session closed`:

```
# Local, on-keyboard login/logout (login:session)
2025-08-02T16:04:43 thm-vm login[1138]: pam_unix(login:session): session opened for user bob(uid=1001) by bob(uid=0)
2025-08-02T19:23:08 thm-vm login[1138]: pam_unix(login:session): session closed for user bob

# Remote logins (SSH, then SMB) for Alice
2025-08-04T09:09:06 thm-vm sshd[839]: pam_unix(sshd:session): session opened for user alice(uid=1002) by alice(uid=0)
2025-08-04T12:46:13 thm-vm smbd[1795]: pam_unix(samba:session): session opened for user alice(uid=1002) by alice(uid=0)

# Cron job running as root (cron:session)
2025-08-06T19:35:01 thm-vm CRON[41925]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)

# "sudo su" to access root (sudo:session)
2025-08-07T09:12:32 thm-vm sudo: pam_unix(sudo:session): session opened for user root(uid=0) by carol(uid=1003)
```

The SSH daemon also writes its own success/failure entries to the same `auth.log`, in a slightly different format (`<result> <auth-method> for <user> from <ip>`):

```
2025-08-07T11:21:25 thm-vm sshd[3139]: Failed password for root from 222.124.17.227 port 50293 ssh2
2025-08-07T14:17:40 thm-vm sshd[3139]: Failed password for admin from 138.204.127.54 port 52670 ssh2
2025-08-09T20:30:51 thm-vm sshd[1690]: Accepted publickey for bob from 10.19.92.18 port 55050 ssh2: <key>
```

#### User Management Events

Since `auth.log` captures everything PAM-related, user-management commands show up too — filter for the underlying command names:

```
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['

2023-02-01T11:09:55 thm-vm passwd[644]: password for 'ubuntu' changed by 'root'
2025-08-07T22:11:11 thm-vm userdel[1887]: delete user 'oldbackdoor'
2025-08-07T22:11:29 thm-vm useradd[1878]: new user: name=backdoor, UID=1002, GID=1002, shell=/bin/sh
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to group 'sudo'
2025-08-07T22:11:54 thm-vm usermod[1906]: add 'backdoor' to shadow group 'sudo'
```

#### Commands Run With Sudo

Depending on configuration, `sudo` command executions are also logged — useful for tracking malicious actions:

```
cat /var/log/auth.log | grep -E 'COMMAND='

2025-08-07T11:21:49 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/systemctl stop edr
2025-08-07T11:23:18 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/ufw status numbered
2025-08-07T11:23:33 thm-vm sudo: ubuntu : TTY=pts/0 ; [...] COMMAND=/usr/bin/su
```

> **Example read:** the `ubuntu` user used `sudo` to stop the EDR, check firewall status, then escalate to root via `sudo su` — a classic defense-evasion-then-escalation pattern.

---

### Generic System Logs

Other `/var/log` files worth knowing, though noisier and more DFIR-oriented than daily-SOC:

| Log File | Content |
|---|---|
| `/var/log/kern.log` | Kernel messages and errors |
| `/var/log/syslog` (or `/var/log/messages`) | Consolidated stream of general Linux events |
| `/var/log/dpkg.log` (or `/var/log/apt`) | Package manager logs (Debian-based) |
| `/var/log/dnf.log` (or `/var/log/yum.log`) | Package manager logs (RHEL-based) |

### App-Specific Logs

Individual applications maintain their own logs — database query logs, mail logs (phishing investigation), container logs, and web server logs. Example — Nginx access logs:

```
root@thm-vm:~$ cat /var/log/nginx/access.log
10.0.1.12 - - [11/08/2025:14:32:10 +0000] "GET / HTTP/1.1" 200 3022
10.0.1.12 - - [11/08/2025:14:32:14 +0000] "GET /login HTTP/1.1" 200 1056
10.0.1.12 - - [11/08/2025:14:33:09 +0000] "POST /login HTTP/1.1" 302 112
10.0.5.21 - - [11/08/2025:17:56:23 +0000] "GET /admin HTTP/1.1" 403 104
```

---

### Bash History

**`~/.bash_history`** records each command entered at an interactive shell — buffered in memory during the session, then flushed to the per-user file on logout. The live `history` command shows both current and past-session commands:

```
cat /home/ubuntu/.bash_history
echo "hello" > world.txt
nano /etc/ssh/sshd_config
sudo su

history
1  echo "hello" > world.txt
2  nano /etc/ssh/sshd_config
3  sudo su
4  ls -la /home/ubuntu
```

**Limitations** — Bash history is rarely relied on in daily SOC work, because:

- It doesn't capture non-interactive commands (OS-initiated, cron jobs, web server actions)
- A **leading space** before a command skips logging entirely: `  echo "You will never see me in logs!"`
- Commands pasted into a script and run (`nano legit.sh && ./legit.sh`) bypass history
- Switching to a non-Bash shell (`sh`) stops history tracking altogether

---

### Runtime Monitoring

None of the log sources above reliably answer questions like *"Which programs did Bob launch today?"* or *"Who deleted my home folder, and when?"* — because by default, Linux doesn't log process creation, file changes, or network activity (**runtime events**). This mirrors the Windows limitation that required Sysmon; Linux needs an equivalent add-on.

#### System Calls

Every OS-level action — opening a file, creating a process, accessing hardware — goes through a **system call**. Linux has 300+ of them (e.g., `execve` to execute a program): a program issues the call, the kernel services it using hardware resources, and returns the result. Since there's essentially no way for attackers to bypass system calls, EDRs and logging tools build their visibility by monitoring a chosen subset of them.

---

### Audit Daemon (auditd)

**Auditd** is Linux's built-in runtime-auditing solution. Rules live in **`/etc/audit/rules.d/`** and define which system calls to watch and which filters to apply (e.g., rules to flag `wget` usage, specific file operations, or network connections made by `python3`).

> Monitoring every process, file, and network event can generate gigabytes of logs daily — more logs don't equal better detection, since an attack buried in terabytes of noise is still effectively invisible. SOC teams build focused, balanced rulesets around the highest-risk events.

#### Using Auditd

Raw logs live at `/var/log/audit/audit.log`, but the **`ausearch`** command formats output far more readably and supports filtering.

**Example — searching for `wget` execution** (key: `proc_wget`):

```
ausearch -i -k proc_wget
----
type=PROCTITLE msg=audit(08/12/25 12:48:19.093:2219) : proctitle=wget https://files.tryhackme.thm/report.zip
type=CWD msg=audit(08/12/25 12:48:19.093:2219) : cwd=/root
type=EXECVE msg=audit(08/12/25 12:48:19.093:2219) : argc=2 a0=wget a1=https://files.tryhackme.thm/report.zip
type=SYSCALL msg=audit(08/12/25 12:48:19.093:2219) : arch=x86_64 syscall=execve [...] ppid=3752 pid=3888 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/wget key=proc_wget
```

**Key fields:**

| Field | Meaning |
|---|---|
| `pid` / `ppid` | Process ID / Parent Process ID — links events into a process tree |
| `auid` | Audit User — the account originally used to log in (local or SSH), regardless of later privilege switches |
| `uid` | The user who actually ran the command — can differ from `auid` after `sudo`/`su` |
| `tty` | Session identifier — distinguishes events on multi-user servers |
| `exe` | Absolute path to the executed binary — often used to build detection rules |
| `key` | Optional tag set in the auditd rule, used for filtering (`-k`) |

**Example — file-change monitoring** (key: `file_sshconf`), tracking an SSH config edit via `nano`:

```
ausearch -i -k file_sshconf
----
type=PROCTITLE msg=audit(08/12/25 13:06:47.656:2240) : proctitle=nano /etc/ssh/sshd_config
type=CWD msg=audit(08/12/25 13:06:47.656:2240) : cwd=/
type=PATH msg=audit(08/12/25 13:06:47.656:2240) : item=0 name=/etc/ssh/sshd_config [...]
type=SYSCALL msg=audit(08/12/25 13:06:47.656:2240) : arch=x86_64 syscall=openat [...] ppid=3752 pid=3899 auid=ubuntu uid=root tty=pts1 exe=/usr/bin/nano key=file_sshconf
```

SOC teams commonly set rules around critical files/directories: SSH configs, cronjob definitions, system settings.

#### Auditd Alternatives

Auditd's verbose output is hard to read and awkward to ingest into a SIEM, so many SOC teams use alternatives — all based on the same system-call-monitoring principle:

| Tool | Best For |
|---|---|
| **Sysmon for Linux** | Teams already using Windows Sysmon and comfortable with XML |
| **Falco** | Modern, open-source, ideal for containerized environments |
| **Osquery** | General-purpose security querying across many use cases |
| **EDR solutions** | Broad Linux runtime event tracking as part of a commercial platform |

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| Log format | Plain text, unstructured — no event codes like Windows |
| Default log location | `/var/log/` |
| Aggregated system log | `/var/log/syslog` |
| Authentication log | `/var/log/auth.log` (`/var/log/secure` on RHEL) |
| Session login/logout | `session opened` / `session closed` keywords |
| SSH-specific logs | `Accepted` / `Failed` password/publickey lines |
| User management | `useradd`, `usermod`, `userdel`, `passwd` keywords |
| Sudo command tracking | `COMMAND=` entries in `auth.log` |
| Kernel log | `/var/log/kern.log` |
| Package manager logs | `/var/log/dpkg.log` (Debian) / `/var/log/dnf.log` (RHEL) |
| Interactive command history | `~/.bash_history` — easily evaded (leading space, scripts, alt shells) |
| Runtime monitoring gap | No default process/file/network logging on Linux |
| Core mechanism | System calls (e.g., `execve`) — nearly impossible for attackers to bypass |
| Built-in runtime auditor | **auditd** — rules in `/etc/audit/rules.d/`, logs via `ausearch -k <key>` |
| Key auditd fields | `pid`/`ppid`, `auid`, `uid`, `tty`, `exe`, `key` |
| Auditd alternatives | Sysmon for Linux, Falco, Osquery, EDR |

---

## Summary

- Linux logs most events as **plain text** under `/var/log` — readable in any editor but unstructured, with no event codes — and analysts rely on `grep` (including `grep -v` to exclude noise and `grep -R` to search across the whole directory); because format, verbosity, and storage location can all be customized, logs vary between distributions and some may not exist at all.
- **`/var/log/auth.log`** (`/var/log/secure` on RHEL) is the most useful log: it records **session opened/closed** events for local, SSH, sudo, and cron logins, SSH **Accepted/Failed** lines, user management via `useradd`, `usermod`, `userdel`, and `passwd`, and **`COMMAND=`** entries for sudo executions — such as an `ubuntu` user stopping the EDR, checking the firewall, and escalating via `sudo su`.
- Other logs include **`kern.log`**, **`syslog`/`messages`**, package manager logs (`dpkg.log`, `dnf.log`), and **app-specific logs** such as Nginx access logs; **`~/.bash_history`** records interactive commands but is easily evaded by a leading space, scripts, non-interactive commands, or switching to a different shell.
- By default Linux doesn't log **runtime events** (process creation, file changes, network activity); since every action goes through a **system call** that attackers can't realistically bypass, tools monitor a chosen subset of them, and the built-in **auditd** uses rules in `/etc/audit/rules.d/` and is read with **`ausearch -i -k <key>`**, with key fields `pid`/`ppid`, `auid` (original login account), `uid` (account that ran it), `tty`, `exe`, and `key` — balanced against the risk of gigabytes of noise.
- Because auditd output is verbose and awkward to ingest into a SIEM, teams often use alternatives built on the same principle: **Sysmon for Linux**, **Falco** (containers), **Osquery**, or a commercial **EDR**.

