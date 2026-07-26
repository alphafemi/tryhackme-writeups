 John the Ripper — Hash Cracking in Practice
 
**John the Ripper** ("John") is a fast, versatile password-cracking tool. This section focuses on **Jumbo John** — the extended community edition with additional conversion tools.
 
> **Why cracking is possible:** Hash functions are one-way (computationally infeasible to reverse), but cracking doesn't reverse them — it hashes many candidate passwords and compares to the target. The **P vs NP** principle: computing a hash is fast (P); reversing one is intractable (NP).
 
---
 
### Setup
 
| Platform | Status |
|---|---|
| TryHackMe AttackBox | Pre-installed |
| Kali Linux | Pre-installed |
| Ubuntu/Debian | `sudo apt install john` (core only — missing some tools) |
| Fedora | `sudo dnf install john` |
| Full Jumbo John | Build from source — see official install guide |
 
```bash
john    # verify installation; should show version info
```
 
**Wordlist:** `rockyou.txt` — 14M+ real passwords from the 2009 RockYou breach:
 
```bash
/usr/share/wordlists/rockyou.txt
```
 
---
 
### Basic Syntax
 
```bash
john [options] [file containing hash]
```
 
#### Automatic Mode (auto-detect hash type)
 
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
 
> Convenient but unreliable — John may misidentify the hash type. Use format-specific cracking when you know the algorithm.
 
#### Format-Specific Cracking
 
```bash
john --format=[format] --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
 
**Common format names:**
 
| Hash Type | John Format Flag |
|---|---|
| MD5 | `--format=raw-md5` |
| SHA-1 | `--format=raw-sha1` |
| SHA-256 | `--format=raw-sha256` |
| SHA-512 | `--format=raw-sha512` |
| NTLM (Windows) | `--format=nt` |
| sha512crypt (`$6$`) | `--format=sha512crypt` |
| Bcrypt (`$2*$`) | `--format=bcrypt` |
 
> **Note:** Standard hash types typically need the `raw-` prefix (e.g., `raw-md5`). Verify with:
> ```bash
> john --list=formats | grep -iF "md5"
> ```
 
#### Identifying Unknown Hashes
 
```bash
# Download and run hash-identifier
wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py
python3 hash-id.py
# Enter hash → get list of probable formats
```
 
Or use online tools and check hash length/prefix as context clues.
 
---
 
### Cracking NTHash / NTLM (Windows Passwords)
 
Windows stores password hashes in the **SAM** (Security Account Manager) database using NTLM format (variant of MD4).
 
```bash
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt ntlm.txt
```
 
> **Pass-the-hash:** In Windows environments, you can sometimes use the NTLM hash directly for authentication without cracking it first (`pass-the-hash` attack).
 
---
 
### Cracking Linux `/etc/shadow` Passwords
 
Linux shadow hashes must be combined with `/etc/passwd` before John can process them.
 
#### Step 1: Unshadow
 
```bash
unshadow [passwd file] [shadow file] > unshadowed.txt
```
 
```bash
unshadow local_passwd local_shadow > unshadowed.txt
```
 
> You can use entire files or just the relevant lines for each user.
 
#### Step 2: Crack
 
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```
 
> The format depends on the prefix in the shadow file — `$6$` = `sha512crypt`, `$y$` = `yescrypt`, etc.
 
---
 
### Single Crack Mode
 
John generates a custom wordlist based on the **username** and GECOS information (full name, home dir, etc. from `/etc/passwd`), applying **word mangling** rules.
 
**Example — username "Markus":** John would generate candidates like `Markus1`, `Markus!`, `MArkus`, `markus2023`, etc.
 
```bash
john --single --format=[format] [hash file]
```
 
> **File format for single mode:** Prepend the username to the hash:
> ```
> mike:1efee03cdcb96d90ad48ccc7b8666033
> ```
 
---
 
### Custom Rules
 
Define custom password mutation rules in `john.conf` (typically `/etc/john/john.conf` or `/opt/john/john.conf`).
 
#### Rule Syntax
 
```
[List.Rules:RuleName]
<modifiers>"<character sets>"
```
 
#### Common Modifiers
 
| Modifier | Effect |
|---|---|
| `c` | Capitalise the first letter |
| `Az` | Append characters defined in the following set |
| `A0` | Prepend characters defined in the following set |
 
#### Character Set Examples
 
| Set | Matches |
|---|---|
| `[0-9]` | Any digit 0–9 |
| `[A-z]` | Any uppercase or lowercase letter |
| `[A-Z]` | Uppercase letters only |
| `[a-z]` | Lowercase letters only |
| `[!£$%@]` | Specific symbols |
 
#### Example: Password Pattern `Polopassword1!`
 
```ini
[List.Rules:PoloPassword]
cAz"[0-9][!£$%@]"
```
 
Reads as: **Capitalise** first letter, **Append** one digit (0–9) then one symbol from `!£$%@`.
 
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --rule=PoloPassword hash.txt
```
 
---
 
### Cracking Archive Passwords
 
Jumbo John includes conversion tools that extract crackable hashes from encrypted archives.
 
#### ZIP Files
 
```bash
# Step 1: Extract hash
zip2john zipfile.zip > zip_hash.txt
 
# Step 2: Crack
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```
 
#### RAR Archives
 
```bash
# Step 1: Extract hash
rar2john rarfile.rar > rar_hash.txt
 
# Step 2: Crack
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
```
 
---
 
### Cracking SSH Private Key Passphrases
 
Encrypted SSH private key files (`id_rsa`) can also be cracked if they're passphrase-protected.
 
```bash
# Step 1: Convert to crackable format
ssh2john id_rsa > id_rsa_hash.txt
# If ssh2john isn't in PATH:
python3 /opt/john/ssh2john.py id_rsa > id_rsa_hash.txt
 
# Step 2: Crack
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```
 
---
 
### Conversion Tools Summary
 
| Scenario | Conversion Tool | Output to |
|---|---|---|
| Linux `/etc/shadow` | `unshadow passwd shadow` | `unshadowed.txt` |
| ZIP password | `zip2john file.zip` | `zip_hash.txt` |
| RAR password | `rar2john file.rar` | `rar_hash.txt` |
| SSH private key passphrase | `ssh2john id_rsa` | `id_rsa_hash.txt` |
 
> All conversion tools redirect output with `>` and the result feeds directly into `john` for cracking.
 
---
 
### John the Ripper Quick Reference
 
| Task | Command |
|---|---|
| Auto-detect and crack | `john --wordlist=rockyou.txt hash.txt` |
| Format-specific crack | `john --format=raw-md5 --wordlist=rockyou.txt hash.txt` |
| List available formats | `john --list=formats` |
| Filter formats | `john --list=formats \| grep -iF "sha"` |
| Crack NTLM (Windows) | `john --format=nt --wordlist=rockyou.txt hash.txt` |
| Unshadow Linux passwd | `unshadow passwd shadow > unshadowed.txt` |
| Crack shadow file | `john --format=sha512crypt --wordlist=rockyou.txt unshadowed.txt` |
| Single crack mode | `john --single --format=raw-sha256 hashes.txt` |
| Custom rule crack | `john --wordlist=rockyou.txt --rule=RuleName hash.txt` |
| Crack ZIP password | `zip2john file.zip > hash.txt && john --wordlist=rockyou.txt hash.txt` |
| Crack RAR password | `rar2john file.rar > hash.txt && john --wordlist=rockyou.txt hash.txt` |
| Crack SSH key passphrase | `ssh2john id_rsa > hash.txt && john --wordlist=rockyou.txt hash.txt` |
 
