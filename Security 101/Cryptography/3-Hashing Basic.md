Hashing — Integrity, Password Storage & Cracking

### What is a Hash Function?

A hash function takes **input data of any size** and produces a **fixed-size digest (hash value)**. Unlike encryption, there is **no key**, and the process is **one-way** — you cannot reverse the output to recover the input.

#### Key Properties of Good Hash Functions

| Property | Description |
|---|---|
| **Deterministic** | Same input always produces the same output |
| **Fixed output size** | Regardless of input size, output is always the same length |
| **Fast to compute** | Forward computation is efficient |
| **Infeasible to reverse** | Cannot derive input from output (preimage resistance) |
| **Avalanche effect** | Even a single bit change in input causes massive output change |
| **Collision resistant** | Hard to find two different inputs with the same output |

#### Avalanche Effect Demonstrated

```bash
# File 1: "T" (0x54 = 01010100)
# File 2: "U" (0x55 = 01010101) — differs by ONE bit

md5sum:    b9ece18c950afbfa6b0fdbfa4ff731d3  file1.txt
           4c614360da93c0a041b22e537de151eb  file2.txt

sha1sum:   c2c53d66948214258a26ca9ca845d7ac0c17f8e7  file1.txt
           b2c7c0caa10a0cca5ea7d69e54018ae0c0389dd6  file2.txt

sha256sum: e632b7095b0bf32c260fa4c539e9fd7b852d0de454e9be26f24d0d6f91d069d3  file1.txt
           a25513c7e0f6eaa80a3337ee18081b9e2ed09e00af8531c8f7bb2542764027e7  file2.txt
```

> One-bit difference in input → completely different hashes across all algorithms.

#### Hash Output Encoding

Hash functions output raw bytes, typically encoded as:
- **Hexadecimal** — used by `md5sum`, `sha1sum`, `sha256sum`, `sha512sum` (each byte = 2 hex characters)
- **Base64** — more compact; common in web applications and certificates

---

### Common Hash Algorithms

| Algorithm | Output Size | Status |
|---|---|---|
| **MD5** | 128 bits (32 hex chars) | **Broken** — collision attacks exist; avoid for security |
| **SHA-1** | 160 bits (40 hex chars) | **Broken** — collision demonstrated (SHAttered); avoid |
| **SHA-256** | 256 bits (64 hex chars) | Secure — widely used |
| **SHA-512** | 512 bits (128 hex chars) | Secure — stronger than SHA-256 |
| **Bcrypt** | 60 chars | Secure — designed for password hashing; slow by design |
| **Argon2** | Variable | Secure — current recommended standard for passwords |
| **Scrypt** | Variable | Secure — memory-hard; resistant to GPU cracking |
| **PBKDF2** | Variable | Secure — stretches key via iterations |

> **MD5 and SHA-1** are broken for collision resistance but still useful for non-security purposes like checksums where collision attacks are not a concern.

---

### Hash Collisions

A **collision** occurs when two different inputs produce the same hash output.

**The pigeonhole effect:** Hash functions produce a fixed number of possible outputs but accept unlimited input sizes. Since `inputs > outputs`, collisions are mathematically unavoidable — but good algorithms make them computationally infeasible to engineer intentionally.

| Algorithm | Collision Status |
|---|---|
| MD5 | Collision attacks demonstrated — **avoid** |
| SHA-1 | Collision demonstrated (SHAttered, 2017) — **avoid** |
| SHA-256 | No known practical collision attack |
| Bcrypt/Argon2 | Designed for password hashing; not directly vulnerable to collision attacks |

---

### Password Storage

#### Why Not Store Passwords in Plaintext?

A database breach exposes all user passwords immediately — and since many users reuse passwords across services, one breach can compromise many accounts.

#### Three Insecure Practices

| Practice | Risk |
|---|---|
| **Plaintext storage** | Direct exposure on breach (e.g., RockYou breach — 14M+ passwords) |
| **Deprecated encryption** | Key compromise decrypts all passwords (e.g., Adobe breach) |
| **Insecure hash (no salt)** | Rainbow table attacks; identical passwords produce identical hashes (e.g., LinkedIn SHA-1 breach) |

#### Rainbow Tables

A **rainbow table** is a precomputed lookup table mapping hash values to their corresponding plaintext passwords — trading disk space for crack time.

| Hash | Password |
|---|---|
| `e10adc3949ba59abbe56e057f20f883e` | `123456` |
| `e99a18c428cb38d5f260853678922e03` | `abc123` |
| `fcea920f7412b5da7be0cf42b8c93759` | `1234567` |

Online services like **CrackStation** and **Hashes.com** use massive rainbow tables to instantly crack unsalted hashes.

#### Password Salting

A **salt** is a unique random value added to each password before hashing — eliminating rainbow table effectiveness and ensuring identical passwords produce different hashes.

```
User A password: "password123" + salt "X7k#mP2!" = hash("password123X7k#mP2!")
User B password: "password123" + salt "9fQ*vL4@" = hash("password123 9fQ*vL4@")
→ Different hashes despite identical passwords
```

- The salt is stored alongside the hash (it does not need to be secret)
- Must be **unique per user**
- Algorithms like **Bcrypt**, **Argon2**, and **Scrypt** handle salting automatically

#### Secure Password Storage Workflow

```
1. Select a secure algorithm: Argon2, Scrypt, Bcrypt, or PBKDF2
2. Generate a unique salt per user: e.g., "Y4UV*^(=go_!"
3. Concatenate: "AL4RMc10k" + "Y4UV*^(=go_!" → "AL4RMc10kY4UV*^(=go_!"
4. Hash the combined value
5. Store: hash value + salt (never the plaintext password)
```

> Why not encrypt passwords? Encryption requires storing a key — if an attacker gets the key, all passwords are instantly decrypted. Hashing has no key to steal.

---

### Linux Password Storage

Hashed passwords are stored in `/etc/shadow` (root-readable only). Format:

```
$prefix$options$salt$hash
```

**Example line:**
```
strategos:$y$j9T$76UzfgEM5PnymhQ7TlJey1$/OOSg64dhfF.TigVPdzqiFang6uZA4QA1pzzegKdVm4
```

| Field | Value | Meaning |
|---|---|---|
| Algorithm prefix | `$y$` | yescrypt |
| Options | `j9T` | Algorithm parameters |
| Salt | `76UzfgEM5PnymhQ7TlJey1` | Unique salt |
| Hash | `/OOSg64dhfF...` | The actual hash value |

#### Common Linux Hash Prefixes (Strongest to Weakest)

| Prefix | Algorithm |
|---|---|
| `$y$` | **yescrypt** — current recommended default |
| `$gy$` | gost-yescrypt |
| `$7$` | scrypt |
| `$2b$`, `$2y$`, `$2a$` | **bcrypt** |
| `$6$` | sha512crypt — common on older Linux systems |
| `$md5` | SunMD5 (Solaris) |
| `$1$` | md5crypt — **avoid** |

### Windows Password Storage

Windows stores password hashes in the **SAM** (Security Accounts Manager) database using **NTLM** (a variant of MD4).

> NTLM hashes are visually identical to MD4/MD5 hashes — context is essential for correct identification. Tools like **mimikatz** can extract SAM hashes (requires elevated privileges).

---

### Hash Cracking

> You cannot "decrypt" a hash — you can only find an input that produces the same hash. This is done by hashing candidate passwords and comparing.

#### Tools

| Tool | Notes |
|---|---|
| **Hashcat** | GPU-accelerated; extremely fast; best on host (not VM) |
| **John the Ripper** | CPU-based; works well in VMs |
| **CrackStation / Hashes.com** | Online rainbow table lookup — instant for common unsalted hashes |

#### Hashcat Basic Syntax

```bash
hashcat -m <hash_type> -a <attack_mode> hashfile wordlist
```

| Flag | Meaning |
|---|---|
| `-m 0` | MD5 |
| `-m 100` | SHA-1 |
| `-m 1800` | sha512crypt (`$6$`) |
| `-m 3200` | Bcrypt (`$2*$`) |
| `-m 1000` | NTLM (Windows) |
| `-a 0` | Dictionary attack (straight) |

```bash
# Crack a bcrypt hash using rockyou.txt
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

> **GPU cracking:** Bcrypt/Argon2/Scrypt are designed to resist GPU acceleration — they provide no speed advantage over CPU for these algorithms, making offline cracking much slower.

#### The `rockyou.txt` Wordlist

The most famous password wordlist, derived from a 2009 RockYou data breach where 14M+ plaintext passwords were exposed:

```bash
wc -l /usr/share/wordlists/rockyou.txt
# 14344392 (14+ million passwords)
```

---

### Integrity Checking with Hashes

Hash functions provide a reliable way to verify that a file has not been modified.

```bash
# Verify a downloaded ISO against its published SHA-256 checksum
sha256sum fedora-workstation.iso
# Compare to the value published by Fedora
```

> If even a single bit changed during download or tampering occurred, the hash will be completely different.

**Other uses:**
- Detecting duplicate files (identical hashes = identical content)
- Forensic evidence integrity verification
- Software supply chain verification

---

### HMAC — Keyed Hash Message Authentication Code

**HMAC** combines a cryptographic hash function with a secret key to verify both **authenticity** and **integrity** of a message.

```
HMAC(K, M) = H((K ⊕ opad) || H((K ⊕ ipad) || M))
```

| Symbol | Meaning |
|---|---|
| `K` | Secret key |
| `M` | Message |
| `H` | Hash function |
| `ipad` / `opad` | Inner/outer padding constants |

**What HMAC proves:**
- The message came from someone who knows the secret key (**authenticity**)
- The message was not modified in transit (**integrity**)

Unlike a plain hash, HMAC cannot be forged without the secret key.

---

### Hashing vs. Encryption vs. Encoding

| | Hashing | Encryption | Encoding |
|---|---|---|---|
| **Reversible?** | No (one-way) | Yes (with key) | Yes (always) |
| **Key required?** | No | Yes | No |
| **Purpose** | Integrity, password storage | Confidentiality | Data format compatibility |
| **Example** | SHA-256, Bcrypt | AES, RSA | Base64, UTF-8 |
| **Security** | Depends on algorithm | Depends on key secrecy | None — not a security mechanism |

> **Encoding** (Base64, ASCII, UTF-8) converts data format — it provides **no security**. Anyone can decode it. Do not confuse encoding with encryption.

```bash
# Base64 encoding example
echo "TryHackMe" | base64     # → VHJ5SGFja01lCg==
echo "VHJ5SGFja01lCg==" | base64 -d   # → TryHackMe
```

---

### Quick Reference

| Task | Command |
|---|---|
| MD5 hash a file | `md5sum <file>` |
| SHA-1 hash a file | `sha1sum <file>` |
| SHA-256 hash a file | `sha256sum <file>` |
| SHA-512 hash a file | `sha512sum <file>` |
| View Linux shadow file | `sudo cat /etc/shadow` |
| Base64 encode | `echo "text" \| base64` |
| Base64 decode | `echo "encoded" \| base64 -d` |
| Hashcat (dictionary attack) | `hashcat -m <type> -a 0 hash.txt wordlist.txt` |
| Identify hash type | Check prefix (`$y$`, `$6$`, `$2b$`) or use hashID |
| Rockyou wordlist location | `/usr/share/wordlists/rockyou.txt` |

---
