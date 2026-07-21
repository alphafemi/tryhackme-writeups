 Public Key Cryptography — RSA, Diffie-Hellman, SSH & Digital Signatures

### The Security Goals

| Goal | Definition |
|---|---|
| **Authentication** | Confirming the identity of who you are communicating with |
| **Authenticity** | Verifying that a message genuinely came from the claimed sender |
| **Integrity** | Ensuring data has not been altered or tampered with in transit |
| **Confidentiality** | Only authorized parties can access the data |

| Cryptographic Tool | Primary Goals Addressed |
|---|---|
| Symmetric encryption (private key) | Confidentiality |
| Asymmetric encryption (public key) | Authentication, authenticity, integrity |

---

### Why Asymmetric Encryption for Key Exchange?

Asymmetric encryption is slower than symmetric encryption, but it solves the **key distribution problem**: how do two parties agree on a shared secret over an insecure channel without transmitting it in the clear?

**The padlocked box analogy:**

| Analogy Component | Cryptographic Equivalent |
|---|---|
| Secret code (instructions) | Symmetric encryption cipher and key |
| Padlock | Recipient's public key |
| Key to the padlock | Recipient's private key |

**Flow:** Use asymmetric crypto once to securely exchange a symmetric key → then communicate confidentially using fast symmetric encryption.

---

### RSA

**RSA** (Rivest–Shamir–Adleman) is a public-key encryption algorithm for secure data transmission. Its security relies on the practical infeasibility of factoring very large numbers.

#### The Mathematical Basis

Multiplying two large prime numbers is easy; factoring their product back into the original primes is computationally infeasible at sufficient sizes.

```
Easy:  982451653031 × 169743212279 = 166764499494295486767649
Hard:  factor 166764499494295486767649 back into its prime components
```

> Computers can factor numbers with ~60 digits in reasonable time; numbers with **600+ digits** are practically unfactorable with current technology.

#### RSA Variables (CTF Reference)

| Variable | Meaning |
|---|---|
| `p`, `q` | Two large prime numbers (secret) |
| `n` | `p × q` — the modulus (public) |
| `e` | Public exponent (public, part of public key) |
| `d` | Private exponent (secret, part of private key) |
| `m` | Original plaintext message |
| `c` | Encrypted ciphertext |
| **Public key** | `(n, e)` |
| **Private key** | `(n, d)` |

#### Simplified Numerical Example

```
p = 157, q = 199
n = p × q = 31243
φ(n) = n - p - q + 1 = 30888
e = 163  (relatively prime to φ(n))
d = 379  (e × d mod φ(n) = 1)

Public key:  (n=31243, e=163)
Private key: (n=31243, d=379)

Encrypt (x=13): c = x^e mod n = 13^163 mod 31243 = 16341
Decrypt (c=16341): x = c^d mod n = 16341^379 mod 31243 = 13 ✓
```

> Real-world RSA uses prime numbers with **~300 digits each** and a modulus of **2048+ bits**.

---

### Diffie-Hellman Key Exchange

**Diffie-Hellman** solves the key exchange problem without requiring asymmetric encryption — two parties independently derive an identical shared secret over a public channel, without ever transmitting it directly.

#### How It Works

```
Public values agreed upon: p (large prime), g (generator)
Alice: private key a, public key A = g^a mod p
Bob:   private key b, public key B = g^b mod p

Alice sends A to Bob; Bob sends B to Alice.

Alice computes: B^a mod p = g^(ab) mod p
Bob computes:   A^b mod p = g^(ab) mod p

Both arrive at the same shared secret: g^(ab) mod p
```

#### Simplified Numerical Example

```
Public: p = 29, g = 3
Alice:  a = 13 (private)   A = 3^13 mod 29 = 19 (public)
Bob:    b = 15 (private)   B = 3^15 mod 29 = 26 (public)

Exchange: Alice sends 19; Bob sends 26.

Alice: 26^13 mod 29 = 10  (shared secret)
Bob:   19^15 mod 29 = 10  (shared secret) ✓
```

#### DH + RSA Together

| Protocol | Role |
|---|---|
| **Diffie-Hellman** | Key agreement — establishes a shared session key |
| **RSA** | Digital signatures, authentication, key transport — proves identity to prevent MITM |

Together, they prevent man-in-the-middle attacks where an attacker might intercept and substitute their own DH parameters.

---

### SSH Key Authentication

#### Why SSH Keys Over Passwords?

SSH key authentication replaces password-based login with a cryptographic public/private key pair — more secure because the private key never leaves your machine and is not transmitted.

#### Generating SSH Key Pairs: `ssh-keygen`

```bash
ssh-keygen -t ed25519    # generate Ed25519 key pair
ssh-keygen -t rsa        # generate RSA key pair (longer keys)
```

**Supported algorithms (`-t`):**

| Algorithm | Notes |
|---|---|
| **rsa** | Classical; long keys (2048–4096 bits) |
| **ecdsa** | Elliptic curve variant of DSA; smaller keys |
| **ed25519** | Modern, fast, secure; recommended default |
| **dsa** | Older standard for digital signatures |
| **ecdsa-sk / ed25519-sk** | Hardware security key variants |

#### Example Key Generation

```bash
ssh-keygen -t ed25519
# Saves: ~/.ssh/id_ed25519 (private) and ~/.ssh/id_ed25519.pub (public)
```

```
Public key (share freely):
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINqNM... strategos@host

Private key (never share):
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAA...
-----END OPENSSH PRIVATE KEY-----
```

#### Key Security Rules

| Rule | Reason |
|---|---|
| Never share the private key | Anyone with it can authenticate as you |
| Passphrase encrypts the private key | Adds a second layer — key useless without the passphrase |
| Set file permissions to 600 | SSH client ignores keys with looser permissions |
| Generate keys on your own machine | Private key should never exist on a remote host |

```bash
ssh -i privateKeyFile user@host    # connect using a specific key
ssh-copy-id user@host              # copy public key to remote authorized_keys
```

#### SSH Key Files

| File | Purpose |
|---|---|
| `~/.ssh/id_ed25519` | Private key — keep secret |
| `~/.ssh/id_ed25519.pub` | Public key — share freely |
| `~/.ssh/authorized_keys` | Public keys permitted to log into this account |
| `~/.ssh/known_hosts` | Previously seen server fingerprints — MITM detection |

> **First connection prompt:** SSH shows the server's fingerprint and asks you to verify it. Once accepted, the fingerprint is stored in `known_hosts` — future connections will warn if the server key changes (potential MITM attack).

#### Server Authentication Flow

```
Client → connects to server
Server → presents its public key fingerprint
Client → checks known_hosts:
    - Not seen before? Prompt user to verify
    - Seen before, matches? Connect silently
    - Seen before, DIFFERENT? WARN — possible MITM
```

---

### Digital Signatures

A **digital signature** proves both the **authenticity** (who sent it) and **integrity** (unchanged in transit) of a document.

#### How Signing Works

```
[Bob signs a document]

1. Bob computes: hash = H(document)
2. Bob encrypts:  signature = hash encrypted with Bob's PRIVATE key
3. Bob sends:     document + signature

[Alice verifies]

4. Alice decrypts: recovered_hash = signature decrypted with Bob's PUBLIC key
5. Alice computes: hash = H(received document)
6. If recovered_hash == hash → ✓ Authentic and unaltered
```

> Only Bob can create a valid signature (only he has his private key). Anyone with his public key can verify it. Any alteration to the document changes its hash — verification fails.

**Digital signature ≠ scanned image of a handwritten signature.** An image proves nothing about integrity; a cryptographic digital signature does.

---

### Certificates and the Chain of Trust

**Certificates** are digitally signed documents binding a public key to an identity — foundational to HTTPS.

#### Chain of Trust

```
Root CA (pre-trusted in OS/browser)
    └── Intermediate CA (signed by Root CA)
            └── Website Certificate (signed by Intermediate CA)
                    └── tryhackme.com public key
```

When your browser connects to `tryhackme.com`:
1. The server presents its certificate
2. The browser traces the signing chain back to a trusted **Root CA**
3. If the chain is valid → connection is trusted (🔒 padlock)
4. If the chain breaks or the cert is self-signed → browser warning

> **Let's Encrypt** provides free, automated, CA-signed TLS certificates, removing the cost barrier to HTTPS adoption.

---

### GPG / OpenPGP

**GPG** (GnuPG) is an open-source implementation of OpenPGP — used for file/email encryption, digital signing, and key management.

#### Generating a GPG Key Pair

```bash
gpg --full-gen-key
# Select algorithm (ECC/Ed25519 recommended)
# Set expiry date
# Provide name, email, comment
```

#### Common GPG Operations

```bash
gpg --import backup.key                    # import a key
gpg --decrypt confidential_message.gpg    # decrypt a file
gpg --list-keys                            # list all keys in keyring
```

> GPG private keys can be passphrase-protected. Tools like **John the Ripper** (`gpg2john`) can attack weak passphrases — underscoring the importance of strong passphrases and keeping private keys secured.

---

### Cryptanalysis Terms

| Term | Definition |
|---|---|
| **Cryptography** | The science of securing communication using codes and ciphers |
| **Cryptanalysis** | The study of breaking or bypassing cryptographic systems without knowing the key |
| **Brute-Force Attack** | Trying every possible key or password combination |
| **Dictionary Attack** | Trying known words or word combinations — faster than full brute-force when passwords follow patterns |

---

### Quick Reference

| Concept | Key Fact |
|---|---|
| RSA security basis | Difficulty of factoring large numbers |
| RSA public key | `(n, e)` |
| RSA private key | `(n, d)` |
| Diffie-Hellman purpose | Key exchange without transmitting the shared secret |
| SSH key generation | `ssh-keygen -t ed25519` |
| Connect with key | `ssh -i keyfile user@host` |
| Copy public key | `ssh-copy-id user@host` |
| Authorized keys file | `~/.ssh/authorized_keys` |
| Digital signature | Sign with private key; verify with public key |
| Certificate chain | Website cert → Intermediate CA → Root CA (pre-trusted) |
| GPG decrypt | `gpg --decrypt file.gpg` |
| GPG import | `gpg --import key.asc` |

---
