# Log Analysis Fundamentals


---

Attackers try to minimize the traces they leave behind, but security teams can often reconstruct exactly how — and sometimes who — carried out an attack. Just as physical evidence (footprints, damage, CCTV footage) helps investigators piece together a crime scene, **logs** are the digital equivalent: footprints left behind by every activity on a system, whether normal or malicious.

---

### Use Cases of Logs

| Use Case | Description |
|---|---|
| **Security Events Monitoring** | Real-time monitoring of logs helps detect anomalous behavior as it happens. |
| **Incident Investigation & Forensics** | Logs provide detailed traces of what happened, enabling root cause analysis. |
| **Troubleshooting** | Logs capture system/application errors, aiding diagnosis and fixes. |
| **Performance Monitoring** | Logs offer insight into application performance over time. |
| **Auditing & Compliance** | Logs establish a trail of activity needed for compliance requirements. |

---

### Types of Logs

Logs are segregated by category so investigators can go straight to the relevant file instead of wading through everything.

| Log Type | Usage | Example Events |
|---|---|---|
| **System Logs** | Troubleshooting OS-level issues | Startup/shutdown, driver loading, system errors, hardware events |
| **Security Logs** | Detecting and investigating security incidents | Authentication, authorization, policy changes, account changes, abnormal activity |
| **Application Logs** | Tracking events specific to an application | User interaction, application changes/updates, application errors |
| **Audit Logs** | Compliance and security monitoring of changes | Data access, system change, user activity, policy enforcement |
| **Network Logs** | Troubleshooting and investigating network activity | Incoming/outgoing traffic, connection logs, firewall logs |
| **Access Logs** | Tracking access to resources | Webserver, database, application, and API access logs |

> **Note:** Additional log types exist depending on the specific applications and services in use.

---

### Windows Event Logs

Windows stores activity in segregated log files, the most crucial being:

| Log File | Captures |
|---|---|
| **Application** | Application errors, warnings, compatibility issues |
| **System** | Driver/hardware issues, service info, startup/shutdown |
| **Security** | Authentication, account changes, security policy changes — the most security-critical log |

#### Event Viewer

Windows provides a built-in GUI tool, **Event Viewer**, to browse and search these logs (`Start → Event Viewer → Windows Logs`). Each event entry includes key fields:

| Field | Description |
|---|---|
| **Description** | Detailed information about the activity |
| **Log Name** | Which log file the event belongs to |
| **Logged** | Timestamp of the activity |
| **Event ID** | A unique identifier for the specific activity type |

#### Important Event IDs

| Event ID | Meaning |
|---|---|
| `4624` | Successful account logon |
| `4625` | Failed account logon |
| `4634` | Successful account logoff |
| `4720` | User account created |
| `4722` | User account enabled |
| `4724` | Password reset attempt |
| `4725` | User account disabled |
| `4726` | User account deleted |

Event Viewer's **Filter Current Log** feature lets you filter by a specific Event ID — e.g., filtering `4624` surfaces every successful login, without manually scanning the full log.

---

### Web Server Access Logs

Every request to a website (page views, logins, uploads) is logged by the web server. A typical **Apache access log** (e.g., `/var/log/apache2/access.log`) records:

| Field | Example | Description |
|---|---|---|
| **IP Address** | `172.16.0.1` | Client making the request |
| **Timestamp** | `[06/Jun/2024:13:58:44]` | When the request occurred |
| **HTTP Method** | `GET` | Action requested |
| **URL** | `/` | Requested resource |
| **Status Code** | `200` | Server's response result |
| **User-Agent** | `Mozilla/5.0 (Macintosh; ...) Chrome/58.0.3029.110 Safari/537.36` | Client OS/browser info |

---

### Manual Log Analysis — Linux Commands

| Command | Purpose | Example |
|---|---|---|
| `cat` | Display file contents | `cat access.log` |
| `cat a b > c` | Combine multiple log files into one | `cat access1.log access2.log > combined_access.log` |
| `grep "pattern" file` | Search for a string/pattern within a log | `grep "192.168.1.1" access.log` |
| `less file` | Page through large logs one screen at a time | `less access.log` |

**`less` navigation shortcuts:**

| Key | Action |
|---|---|
| `Spacebar` | Next page |
| `b` | Previous page |
| `/pattern` + Enter | Search forward for a pattern |
| `n` | Next match |
| `N` | Previous match |

```bash
# Example: combine and search logs
cat access1.log access2.log > combined_access.log
grep "192.168.1.1" combined_access.log
```

---

### Log Analysis Quick Reference

| Concept | Summary |
|---|---|
| Logs = | Digital footprints of activity (normal or malicious) |
| 6 Core log use cases | Monitoring, Investigation/Forensics, Troubleshooting, Performance, Auditing/Compliance |
| 6 Common log types | System, Security, Application, Audit, Network, Access |
| Windows tool | Event Viewer |
| Key Windows Event IDs | 4624 (login success), 4625 (login fail), 4720 (account created), 4726 (account deleted) |
| Web log source | Apache `access.log` (IP, timestamp, method, URL, status, user-agent) |
| Linux analysis tools | `cat`, `grep`, `less` |

---

## Summary

- Logs act as **digital footprints** left behind by every process — interactive and non-interactive — running on a system, whether the activity is benign or malicious.
- Logs serve five core purposes: **security monitoring, incident investigation/forensics, troubleshooting, performance monitoring,** and **auditing/compliance**.
- The six common log categories are **System, Security, Application, Audit, Network,** and **Access** logs — each scoped to a specific area of activity so investigators can go directly to the relevant source.
- On Windows, the **Event Viewer** exposes Application, System, and Security logs, each entry tagged with a **Event ID** (e.g., `4624` for successful logon, `4625` for failed logon, `4726` for account deletion) that can be filtered directly.
- Web servers like Apache record every request in an **access log**, capturing IP address, timestamp, HTTP method, URL, status code, and User-Agent.
- On Linux, `cat`, `grep`, and `less` form the core toolkit for manually combining, searching, and paging through log files.

## Related Documents

- [56. Incident Response Fundamentals](./56-incident-response-fundamentals.md)
- [54. Security Operations Center (SOC) Fundamentals](./54-security-operations-center-soc-fundamentals.md)
- [55. Digital Forensics Fundamentals](./55-digital-forensics-fundamentals.md)
