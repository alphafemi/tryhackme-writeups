Cryptography Fundamentals

### Why Cryptography Matters

Cryptography ensures **secure communication in the presence of adversaries** — protecting both the **confidentiality** and **integrity** of data. It operates in the background of virtually every digital interaction:

| Scenario | Cryptographic Role |
|---|---|
| Logging into a website | Credentials encrypted in transit — cannot be intercepted |
| SSH session | Encrypted tunnel — eavesdropping is prevented |
| Online banking | Certificate verification — confirms the server is authentic, not an impostor |
| File download verification | Hash functions — confirms the file is identical to the original |

#### Compliance Requirements

Cryptography is not just best practice — it is often legally mandated:

| Standard/Law | Domain | Scope |
|---|---|---|
| **PCI DSS** | Payment card data | Data encrypted at rest and in transit |
| **HIPAA / HITECH** | Medical records (USA) | Healthcare data protection |
| **GDPR** | Personal data (EU) | Data privacy and protection |
| **DPA** | Personal data (UK) | Data protection obligations |

---

### Core Cryptography Terminology

```
Plaintext ──[Encryption: cipher + key]──▶ Ciphertext
Ciphertext ──[Decryption: cipher + key]──▶ Plaintext
```

| Term | Definition |
|---|---|
| **Plaintext** | The original, readable data — a document, image, message, or any binary data |
| **Ciphertext** | The scrambled, unreadable output of encryption — ideally reveals nothing about the plaintext except approximate size |
| **Cipher** | The algorithm/method used to convert plaintext to ciphertext and back |
| **Key** | A string of bits the cipher uses to encrypt or decrypt — must remain secret (except the public key in asymmetric encryption) |
| **Encryption** | The process of converting plaintext → ciphertext using a cipher and key |
| **Decryption** | The reverse process — converting ciphertext → plaintext using a cipher and key |

> **The cipher is public knowledge; the key must remain secret.** Security depends entirely on key secrecy, not algorithm secrecy.

---

### Historical Ciphers

#### Caesar Cipher (1st Century BCE)

Shifts each letter of the alphabet by a fixed number.

```
Plaintext:   T R Y H A C K M E
Key:         3 (right shift)
Ciphertext:  W U B K D F N P H
```

**Encryption:** shift right by 3 → `T → W`, `R → U`, `Y → B` ...

**Decryption:** shift left by 3 → recover original plaintext

**Why Caesar Cipher is insecure:** Only 25 possible keys (shifting by 26 returns to the original letter). A brute-force attack requires at most 25 attempts — trivial to break.

#### Other Historical Ciphers

| Cipher | Era | Notable Feature |
|---|---|---|
| **Vigenère cipher** | 16th century | Uses a keyword to apply variable shifts |
| **Enigma machine** | WWII | Electro-mechanical rotor cipher |
| **One-time pad** | Cold War | Theoretically unbreakable if used correctly |

---

### Symmetric vs. Asymmetric Encryption

#### Symmetric Encryption

Uses the **same key** for both encryption and decryption. Also called **private key cryptography**.

```
[Sender]                         [Recipient]
Plaintext ──[Key]──▶ Ciphertext ──[Key]──▶ Plaintext
                ↑                      ↑
           Same secret key        Same secret key
```

**Key challenge:** Distributing the shared key securely — you cannot send the key over an insecure channel alongside the encrypted data.

##### Symmetric Encryption Standards

| Algorithm | Key Size | Status |
|---|---|---|
| **DES** | 56 bits | Broken (< 24 hours, 1999) — **deprecated** |
| **3DES** | 168 bits (112 effective) | Deprecated in 2019 — avoid; may appear in legacy systems |
| **AES** | 128, 192, or 256 bits | Current standard — widely used |

#### Asymmetric Encryption

Uses a **pair of mathematically linked keys**: a **public key** (shared freely) and a **private key** (kept secret).

```
[Sender]                              [Recipient]
Plaintext ──[Recipient's PUBLIC key]──▶ Ciphertext
                                              │
                                   [Recipient's PRIVATE key]
                                              │
                                         Plaintext
```

- **Encrypt with:** recipient's **public key** (anyone can encrypt)
- **Decrypt with:** recipient's **private key** (only the recipient can decrypt)

##### Asymmetric Algorithms

| Algorithm | Typical Key Size | Notes |
|---|---|---|
| **RSA** | 2048, 3072, or 4096 bits | 2048-bit minimum recommended |
| **Diffie-Hellman** | 2048–4096 bits | Used for key exchange, not direct encryption |
| **ECC** (Elliptic Curve) | 256 bits ≈ 3072-bit RSA | Equivalent security at much shorter key lengths |

**Why asymmetric encryption is slower:** Based on mathematical problems that are easy to compute in one direction but practically infeasible to reverse (e.g., factoring enormous prime numbers).

#### Symmetric vs. Asymmetric Comparison

| Feature | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared secret key | Public/private key pair |
| Speed | Fast | Slower |
| Key size | Shorter (128–256 bits) | Longer (2048–4096 bits) |
| Key distribution | Difficult — must share securely | Easy — public key can be shared openly |
| Use case | Bulk data encryption | Key exchange, digital signatures, authentication |
| Example | AES | RSA, ECC, Diffie-Hellman |

> **In practice, both are used together:** Asymmetric encryption establishes a secure channel to exchange a symmetric key; symmetric encryption then handles bulk data encryption (e.g., TLS/HTTPS does exactly this).

---

### Mathematical Foundations

#### XOR (Exclusive OR) Operation

XOR compares two bits and returns `1` if they differ, `0` if they are the same.

| A | B | A ⊕ B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

**Example:** `1010 ⊕ 1100`
```
1 ⊕ 1 = 0
0 ⊕ 1 = 1
1 ⊕ 0 = 1
0 ⊕ 0 = 0
─────────
  0110
```

#### Key XOR Properties

| Property | Expression | Meaning |
|---|---|---|
| **Self-inverse** | `A ⊕ A = 0` | XOR with itself always gives 0 |
| **Identity** | `A ⊕ 0 = A` | XOR with 0 leaves unchanged |
| **Commutative** | `A ⊕ B = B ⊕ A` | Order doesn't matter |
| **Associative** | `(A ⊕ B) ⊕ C = A ⊕ (B ⊕ C)` | Grouping doesn't matter |

#### XOR as a Simple Symmetric Cipher

```
Encrypt: C = P ⊕ K
Decrypt: P = C ⊕ K

Proof:
C ⊕ K = (P ⊕ K) ⊕ K     (substituting C)
       = P ⊕ (K ⊕ K)      (associative)
       = P ⊕ 0              (self-inverse: K ⊕ K = 0)
       = P                  (identity: P ⊕ 0 = P)
```

> XOR forms the foundation of many modern ciphers (e.g., AES internals). In practice, the key must be at least as long as the plaintext for XOR encryption to be secure (the one-time pad principle).

---

### Modulo Operation

The **modulo operator** (`%` or `mod`) returns the **remainder** after division.

| Expression | Calculation | Result |
|---|---|---|
| `25 % 5` | 25 = 5 × 5 + **0** | 0 |
| `23 % 6` | 23 = 3 × 6 + **5** | 5 |
| `23 % 7` | 23 = 3 × 7 + **2** | 2 |

**Key properties:**
- Result is always in the range `0` to `n−1` (where `n` is the divisor)
- **Not reversible:** if `x % 5 = 4`, infinitely many values of `x` satisfy this (4, 9, 14, 19 …)

Modulo is foundational to asymmetric cryptography — RSA, Diffie-Hellman, and ECC all rely on modular arithmetic because certain operations are easy to perform but computationally infeasible to reverse (the "trapdoor" property).

> For large number calculations used in cryptography problems: Python's built-in `int` type handles arbitrarily large integers natively, or use WolframAlpha for one-off calculations.

---

### Summary

| Concept | Key Points |
|---|---|
| **Cryptography goal** | Confidentiality, integrity, authenticity |
| **Cipher** | Public algorithm; key must stay secret |
| **Symmetric** | Same key both ways; fast; key distribution is the challenge |
| **Asymmetric** | Public key encrypts; private key decrypts; solves key distribution |
| **XOR** | Self-inverse operation — basis of many ciphers |
| **Modulo** | Non-reversible remainder operation — foundation of asymmetric crypto math |

---
