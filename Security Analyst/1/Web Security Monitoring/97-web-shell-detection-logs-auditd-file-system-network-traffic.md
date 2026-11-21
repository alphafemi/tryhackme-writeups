# Web Shell Detection — Logs, Auditd, File System & Network Traffic

## Overview

This document covers detecting web shells: what they are and how they get deployed (with real-world examples from Hafnium and Conti), the anatomy of a basic PHP web shell, detection through web server logs (HTTP methods, request patterns, User-Agents, query strings), correlation with auditd on Linux, SIEM benefits, file system analysis (default paths, suspicious names, helpful `find` and `grep` commands), and network traffic indicators with useful Wireshark filters.

---

A **web shell** is a malicious program uploaded to a target web server that enables remote command execution — serving as both an initial access method (via file upload vulnerabilities) and a persistence mechanism. Once deployed, attackers use it to progress through the kill chain: reconnaissance, privilege escalation, lateral movement, and data exfiltration.

---

### Web Shell Deployment

Requires a **file upload vulnerability**, **misconfiguration**, or prior access — typically arising when an application fails to validate file type, extension, content, or destination.

> **Example:** A pet-photo upload site built insecurely could let an attacker upload `shell.php` or `mydog.aspx` and gain command execution on the server.

#### Examples in the Wild

| Threat Actor | Technique |
|---|---|
| **Hafnium (ProxyLogon)** | Uploaded `.aspx` web shells to Windows Exchange servers (`\inetpub\wwwroot\aspnet_client\`) and modified existing `.aspx` files (`\install_path\FrontEnd\HttpProxy\owa\auth\`); continued to dump credentials, create persistence accounts, move laterally, exfiltrate data, and cover tracks |
| **Conti Ransomware** | Abused the same Microsoft Exchange vulnerability, uploading `aspnetclient_log.aspx` to the same directory; deployed a backup web shell within minutes and mapped the network's computers, domain controllers, and domain admins |

---

### Anatomy of a Web Shell

Web shells abuse **legitimate functions** — in PHP, system execution functions like `shell_exec()`, `exec()`, `system()`, and `passthru()` can be leveraged to gain command execution.

**Basic PHP web shell logic:**

1. Checks if a `cmd` parameter is present in the URL (`?cmd=whoami`)
2. Stores the user-supplied command in a variable
3. Executes it via `shell_exec()`
4. Displays the output

**Accessing a deployed web shell:**

```
http://MACHINE_IP:8080/files/awebshell.php?cmd=<encoded_command>
```

> Commands must be URL-encoded when accessed via command line (e.g., `curl`) — `ls -la` becomes `ls%20-la`. **CyberChef** (Section 62) is useful for this encoding.

Web shells range from simple one-liners executing via a URL parameter to fully-featured GUIs with password protection and built-in file managers.

---

### Detecting Web Shells via Web Server Logs

Access logs generally follow a consistent structure across web servers. Two fields worth noting: the **remote log name** and **authenticated user** fields usually appear as a hyphen (`-`) unless the server required prior auth.

#### Web Indicators

**Unusual HTTP Methods & Request Patterns:**

| Method | Normal Usage | Possible Abuse |
|---|---|---|
| **GET** | Retrieve a resource | Recon or interacting with a web shell |
| **POST** | Submit data to the server | Upload or interact with a web shell |
| **PUT** | Upload/replace a file | Upload a web shell |
| **DELETE** | Remove a resource | Cleanup after the attack |
| **OPTIONS** | List supported methods | Reconnaissance |
| **HEAD** | Like GET but headers only | Detect file existence |

**Key patterns:**
- Repeated GET requests in quick succession — probing for an upload location
- POST requests to valid upload locations following repeated GETs
- Repeated GET/POST to the same file — possible ongoing web shell interaction

**Worked attack sequence:** Directory fuzzing (finds a valid directory) → discovers an abusable upload form → uploads the web shell → accesses it directly — all sharing the same client IP and User-Agent, worth correlating across the sequence.

**Suspicious User-Agents & IPs:**

| Indicator | Example |
|---|---|
| **Altered User-Agent** | `Mozilla/4.0+(+Windows+NT+5.1)` shortened to `Mozilla/4.0` |
| **Outdated User-Agent** | `Mozilla/4.0 (compatible; MSIE 6.0)` — released 2001 |
| **Blacklisted User-Agent** | `curl/1.XX.X`, `wget/1.XX.X` |
| **Suspicious IP** | External IP on a network that normally sees only internal traffic |

**Query strings:**
- Abnormally long or suspicious strings, especially containing `cmd=` or `exec=`
- Encoded query strings (e.g., `?query=whoami` → `?query=d2hvYW1p` in Base64) — decode with CyberChef

**Missing Referrer:** Can indicate web shell activity, but has valid benign causes (privacy-blocked referrers, direct URL access) — not a standalone indicator.

**Composite suspicious request example:** A known-malicious IP, an abnormal (off-hours) timestamp, a POST with a search-style query string to a malicious file, no referrer, and an unusual User-Agent — taken together, a strong composite signal.

---

### Auditd

A native Linux utility tracking/recording events into an audit trail. Rules can target specific conditions (program execution, file modifications in a directory).

```bash
ausearch -k web_shell
```

```
time->Wed Jul 23 06:20:36 2025
"name = /uploads/webshell.php"
"OGID = www-data"
```

#### Web & Auditd Correlation

Combining web access/error logs with auditd confirms whether a file was created, modified, or executed, and by which user/process. A suspicious POST in web logs linked to an auditd `creat` or `execve` syscall event confirms a script wrote a file or ran commands — building a clearer attack picture than either source alone.

---

### Leveraging SIEM Platforms

| Benefit | Description |
|---|---|
| **Centralized collection/correlation** | Especially valuable across many log types |
| **Targeted queries** | Surfaces malicious activity including web shells |
| **Efficient search/analysis** | Lets analysts work across large log volumes quickly |

---

### File System Analysis

Since a web shell must be stored somewhere, analyzing server files is critical. **Note:** Platforms like WordPress/Django store content in a database rather than the filesystem — malicious code may be injected into posts, themes, or settings and won't surface in a normal filesystem search.

#### Common Web Shell Locations

| Server | Default Path |
|---|---|
| **Apache** | `/var/www/html/` (default on most Linux distros) |
| **Nginx** | `/usr/share/nginx/html/` (default on many Linux setups) |

Even with a custom root path, attackers scan/guess common upload directories (`/uploads/`, `/images/`, `/admin/`). Insecurely permissioned temp directories (`/tmp`) can also be abused.

#### Suspicious File Names

- Executable extensions (`.php`, `.jsp`) deserve scrutiny
- **Double extensions** disguising malicious files (`image.jpg.php`)

#### Helpful Commands

```bash
# Find PHP files modified within a date range
find /var/www -type f -name "*.php" -newerct "2025-07-01" ! -newerct "2025-08-01"
# /var/www/html/uploads/awebshell.php

# Search for suspicious functions (e.g., eval()) within files
grep -r "eval(" wp-content
# /wp-content/uploads/awebshell2.php :eval(b64_dd($['cmd']));
```

---

### Network Traffic Analysis for Web Shells

Network analysis goes beyond logs by exposing the full data exchanged. Many log-analysis indicators carry over, plus several network-specific ones:

| Indicator | Description |
|---|---|
| Unusual HTTP methods & request patterns | Same as log-based indicators |
| Suspicious User-Agents & IPs | Same as log-based indicators |
| Encoded payloads | Base64 or other obfuscation in request bodies |
| Malicious code/commands in request bodies | Directly visible in the packet payload |
| Unexpected protocols/ports | Traffic deviating from expected web ports |
| Unexpected resource usage | Abnormal server load/traffic volume |
| Web server processes spawning command-line tools | A strong compromise indicator |

> **Example:** A PCAP capturing `POST /upload.php` may show the complete PHP web shell source code directly in the payload — confirming exploitation beyond doubt.

#### Useful Wireshark HTTP Filters

| Filter | Purpose |
|---|---|
| `http.request.method == "METHOD"` | Hunt for repeated/unusual request types |
| `http.request.uri contains ".php"` | Find suspicious or modified PHP files |
| `http.user_agent` | Locate unusual or outdated User-Agents |

---

### Web Shell Detection Quick Reference

| Concept | Summary |
|---|---|
| Web shell = | Malicious uploaded program enabling remote command execution |
| Dual role | Initial access vector + persistence mechanism |
| Deployment requirement | File upload vuln, misconfiguration, or prior access |
| Real-world examples | Hafnium/ProxyLogon, Conti Ransomware (both via Exchange) |
| Abused PHP functions | shell_exec(), exec(), system(), passthru() |
| Log indicators | Repeated GET/POST to the same file, suspicious User-Agent/IP, encoded query strings, missing referrer |
| Auditd | Correlates file creation/execution with the responsible user/process |
| SIEM value | Centralized correlation across log sources |
| Common web shell paths | `/var/www/html/` (Apache), `/usr/share/nginx/html/` (Nginx), `/uploads/`, `/tmp` |
| Filesystem hunting commands | `find -newerct`, `grep -r "eval("` |
| Network indicators | Encoded payloads, malicious code in request bodies, unexpected ports, CLI tools spawned by web server processes |
| Key Wireshark filters | `http.request.method`, `http.request.uri contains ".php"`, `http.user_agent` |

---

## Summary

- A **web shell** is a malicious program uploaded to a web server to enable remote command execution, serving as both an **initial access** method and a **persistence** mechanism; deployment requires a file upload vulnerability, misconfiguration, or prior access, as seen with **Hafnium** (ProxyLogon `.aspx` shells on Exchange) and **Conti** (a backup shell deployed within minutes through the same Exchange flaw).
- Web shells abuse legitimate functions — in PHP, `shell_exec()`, `exec()`, `system()`, and `passthru()` — typically taking a command through a URL parameter like `?cmd=whoami`, and range from simple one-liners to full-featured GUIs with file managers.
- **Web server logs** reveal them through repeated GETs probing for upload locations, POSTs to upload paths, repeated requests to one file, altered/outdated/blacklisted User-Agents (`curl`, `wget`, MSIE 6.0), suspicious external IPs, long or Base64-encoded query strings (`cmd=`, `exec=`), and missing referrers (not conclusive alone) — strongest when correlated across the sequence.
- On Linux, **auditd** (e.g., `ausearch -k web_shell`) confirms which user or process created or executed a file, so correlating a suspicious POST in the web logs with an auditd `creat` or `execve` event gives a clearer picture than either source alone, and a **SIEM** centralizes that correlation across log types.
- **File system analysis** targets default web roots (`/var/www/html/`, `/usr/share/nginx/html/`), upload directories, and `/tmp`, looking for `.php`/`.jsp` files and double extensions (`image.jpg.php`) using `find -newerct` and `grep -r "eval("` — noting that WordPress and Django content lives in a database, not the filesystem — while **network analysis** adds encoded payloads, code in request bodies, unexpected ports, and web server processes spawning command-line tools, with Wireshark filters on `http.request.method`, `http.request.uri contains ".php"`, and `http.user_agent`.

## Related Documents

- [96. Client-Side vs. Server-Side Web Attacks — Detection in Logs, Network Traffic & WAFs](./96-client-side-vs-server-side-web-attacks-detection-in-logs-network-traffic-wafs.md)
- [98. Denial-of-Service (DoS/DDoS) Attacks — Detection & Mitigation](./98-denial-of-service-dos-ddos-attacks-detection-mitigation.md)
- [53. Shells — Reverse, Bind & Web Shells](./53-shells-reverse-bind-web-shells.md)
- [78. The Cyber Kill Chain](./78-the-cyber-kill-chain.md)
- [62. CyberChef Fundamentals](./62-cyberchef-fundamentals.md)
