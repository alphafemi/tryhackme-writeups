# Gobuster — Directory, DNS & VHost Enumeration
 

---
 
**Gobuster** is an open-source offensive tool written in Go that brute-forces web directories, DNS subdomains, and virtual hosts using wordlists. It sits between the reconnaissance and scanning phases of ethical hacking.
 
> **Enumeration:** Listing all available resources (accessible or not).
> **Brute Force:** Trying every entry in a wordlist until a match is found.
 
---
 
### Common Flags (All Modes)
 
| Short | Long | Description |
|---|---|---|
| `-t` | `--threads` | Number of concurrent threads (default: 10) |
| `-w` | `--wordlist` | Path to the wordlist |
| `-o` | `--output` | Write results to a file |
| | `--delay` | Wait time between requests (e.g., `1500ms`) |
| | `--debug` | Enable debug output |
| `-q` | `--quiet` | Suppress banner and noise |
 
---
 
### Mode 1: `dir` — Directory & File Enumeration
 
Discovers directories and files on a web server by appending wordlist entries to the base URL.
 
#### Key `dir` Flags
 
| Short | Long | Description |
|---|---|---|
| `-u` | `--url` | Target URL (required) |
| `-w` | `--wordlist` | Wordlist path (required) |
| `-x` | `--extensions` | File extensions to search (e.g., `.php,.js`) |
| `-r` | `--followredirect` | Follow HTTP redirects |
| `-s` | `--status-codes` | Only show these status codes |
| `-b` | `--status-codes-blacklist` | Hide these status codes |
| `-k` | `--no-tls-validation` | Skip TLS certificate check (useful for CTFs) |
| `-c` | `--cookies` | Pass cookies with each request |
| `-H` | `--headers` | Pass custom headers |
| `-U` / `-P` | `--username` / `--password` | Authenticated scanning |
 
#### Examples
 
```bash
# Basic directory scan
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirb/small.txt
 
# With 64 threads for speed
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 64
 
# Follow redirects
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r
 
# Also search for specific file types
gobuster dir -u "http://www.example.thm" -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.js
```
 
> **Note:** Gobuster does **not** enumerate recursively. To enumerate a subdirectory, run a separate scan with that path as the base URL.
 
---
 
### Mode 2: `dns` — Subdomain Enumeration
 
Brute-forces DNS subdomains by performing DNS lookups for each wordlist entry prepended to the target domain.
 
#### Key `dns` Flags
 
| Short | Long | Description |
|---|---|---|
| `-d` | `--domain` | Target domain (required) |
| `-w` | `--wordlist` | Wordlist path (required) |
| `-i` | `--show-ips` | Show resolved IP addresses |
| `-c` | `--show-cname` | Show CNAME records (cannot combine with `-i`) |
| `-r` | `--resolver` | Use a custom DNS server |
 
#### Example
 
```bash
gobuster dns -d example.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```
 
**Sample output:**
```
Found: www.example.thm
Found: shop.example.thm
Found: academy.example.thm
Found: primary.example.thm
```
 
---
 
### Mode 3: `vhost` — Virtual Host Enumeration
 
Discovers virtual hosts (different websites sharing one IP) by sending HTTP requests with different `Host:` header values.
 
#### `vhost` vs `dns` — Key Difference
 
| Mode | Method | What it finds |
|---|---|---|
| `dns` | DNS lookup | Subdomains (DNS-based) |
| `vhost` | HTTP request with modified `Host:` header | Virtual hosts (IP-based, may not be in DNS) |
 
#### Key `vhost` Flags
 
| Short | Long | Description |
|---|---|---|
| `-u` | `--url` | Target base URL (required) |
| `-w` | `--wordlist` | Wordlist path (required) |
| | `--domain` | Domain appended to wordlist entries for the `Host:` header |
| | `--append-domain` | Appends the domain to each wordlist entry (prevents false positives) |
| | `--exclude-length` | Filter out responses by body size (removes false positives) |
| `-r` | `--follow-redirect` | Follow HTTP redirects |
| `-m` | `--method` | HTTP method (default: GET) |
 
#### Example
 
```bash
gobuster vhost -u "http://MACHINE_IP" \
  --domain example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain \
  --exclude-length 250-320
```
 
**How the `Host:` header is built:**
```
GET / HTTP/1.1
Host: blog.example.thm       ← wordlist entry + --domain value
```
 
- `blog` = wordlist entry (the subdomain part)
- `.example.thm` = set by `--domain`
- `--append-domain` ensures the domain is appended — without it, `Host:` would just be `blog`, causing false positives
---
 
### Gobuster Mode Summary
 
| Mode | Command | Discovers | Required Flags |
|---|---|---|---|
| Directory | `gobuster dir` | Web directories & files | `-u`, `-w` |
| DNS subdomain | `gobuster dns` | DNS subdomains | `-d`, `-w` |
| Virtual host | `gobuster vhost` | Virtual hosts (IP-based) | `-u`, `-w` |
 
---
 
### Common Wordlists
 
| Wordlist | Location | Best For |
|---|---|---|
| `dirb/small.txt` | `/usr/share/wordlists/dirb/` | Quick directory scans |
| `directory-list-2.3-medium.txt` | `/usr/share/wordlists/dirbuster/` | Thorough directory scans |
| `subdomains-top1million-5000.txt` | `/usr/share/wordlists/SecLists/Discovery/DNS/` | DNS/vhost subdomain discovery |
 
---
 
## Summary
 
- **Gobuster** is a Go-based brute-forcing tool for discovering web directories/files, DNS subdomains, and virtual hosts, sitting between recon and scanning phases.
- The **`dir`** mode brute-forces paths against a base URL — it is not recursive, so subdirectories require separate scans.
- The **`dns`** mode performs DNS lookups to find subdomains; the **`vhost`** mode instead sends HTTP requests with modified `Host:` headers to find IP-based virtual hosts that may not appear in DNS.
- `--append-domain` and `--exclude-length` are key flags for reducing false positives during `vhost` scans.
- Wordlist choice (`dirb/small.txt` for speed vs. `directory-list-2.3-medium.txt` for thoroughness) trades scan speed against coverage.
