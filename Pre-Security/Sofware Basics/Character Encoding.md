Character Encoding
 
### The Core Problem
 
Computers only store `0`s and `1`s. To store text, every party — manufacturers, programmers, and applications — must agree in advance on which bit pattern represents which character. This agreement is a **character encoding standard**.
 
---
 
### ASCII (American Standard Code for Information Interchange)
 
One of the earliest standards (1963), using **7 bits** to represent 128 values: English letters, digits, punctuation, and control characters.
 
#### Sample ASCII Table
 
| Decimal | Hex | Binary | Symbol | Description |
|---|---|---|---|---|
| 48 | `30` | `00110000` | `0` | Zero |
| 57 | `39` | `00111001` | `9` | Nine |
| 65 | `41` | `01000001` | `A` | Uppercase A |
| 88 | `58` | `01011000` | `X` | Uppercase X |
| 90 | `5A` | `01011010` | `Z` | Uppercase Z |
| 91 | `5B` | `01011011` | `[` | Opening bracket |
| 97 | `61` | `01100001` | `a` | Lowercase a |
| 122 | `7A` | `01111010` | `z` | Lowercase z |
| 127 | `7F` | `01111111` | `DEL` | Delete |
 
> **Key pattern:** Characters are assigned sequentially — knowing the code for `a` lets you derive `b`, `c`, etc. by simple increment. This holds true for `A`–`Z` and `0`–`9` as well.
 
#### Worked Example: "TryHackMe" in ASCII
 
**Binary representation** (each byte = one character, plus a trailing newline):
```
01010100 01110010 01111001 01001000 01100001 01100011 01101011 01001101 01100101 00001010
   T        r        y        H        a        c        k        M        e       \n
```
 
**Hexadecimal representation** (more practical to read/write):
```
54 72 79 48 61 63 6b 4d 65 0a
```
 
**Decimal representation** (rarely used in practice):
```
84 114 121 72 97 99 107 77 101 10
```
 
> Hexadecimal is preferred over binary or decimal when inspecting raw file contents, since each byte maps cleanly to exactly two hex digits.
 
---
 
### Extending Beyond English
 
ASCII's 128 characters cannot represent accented or non-Latin characters needed by other European languages (e.g., `ñ`, `ü`, `ł`, `č`, `ț`). Using the 8th bit (unused by standard ASCII), several **extended encodings** were introduced:
 
| Standard | Coverage |
|---|---|
| **ISO-8859-1 (Latin-1)** | Western European: German, French, Spanish, Italian, Portuguese, Nordic languages |
| **ISO-8859-2 (Latin-2)** | Central/Eastern European: Polish, Czech, Hungarian, Croatian, Romanian, Slovak |
 
#### The Encoding Mismatch Problem
 
Each ISO-8859 variant assigns *different* characters to the same upper byte-range values. If a document is **saved** in one encoding but **opened** using another, characters render incorrectly:
 
```
Saved as:    ISO-8859-1  →  character "Ø"
Opened as:   ISO-8859-2  →  displayed as "Ř"  (incorrect)
```
 
> This is why specifying the correct encoding (e.g., in HTML `<meta charset>` tags or email headers) is essential — without it, the sender and recipient may silently disagree on what each byte means.
 
#### Scale of the Problem Beyond Europe
 
Non-European writing systems require vastly more characters than any single-byte encoding can support:
 
| Language | Approximate Character Count |
|---|---|
| English (upper + lowercase) | 52 |
| Arabic (with ligatures/diacritics) | 250+ |
| Japanese (daily-use Kanji) | 2,136 |
| Japanese (JIS X 0208 standard) | 6,879 |
| Chinese (commonly recognized) | ~8,000 |
| Chinese (GB 18030-2022 standard) | 87,887+ |
 
A single byte (256 max values) is structurally incapable of covering even one of these scripts, let alone all of them simultaneously — which is the problem Unicode was designed to solve.
 
---
 
### Unicode
 
**Unicode** is a universal character encoding standard that assigns a unique **code point** to every character across all writing systems — modern, historical, and symbolic (including emoji).
 
| Notation Example | Character |
|---|---|
| `U+0041` | Latin "A" |
| `U+03A9` | Greek "Ω" |
| `U+3042` | Japanese Hiragana "あ" |
| `U+9F8D` | Chinese "龍" (dragon) |
| `U+265E` | Chess black knight "♞" |
| `U+1F525` | Fire emoji "🔥" |
 
> Unicode 17.0 defines approximately **157,000 characters**, of which nearly **4,000** are emoji sequences.
 
The core advantage of Unicode: sender and recipient no longer need to negotiate or guess which regional encoding was used — both can rely on the same universal standard, including mixing multiple languages in a single document.
 
---
 
### UTF-8, UTF-16, and UTF-32
 
These are **implementations** of the Unicode standard — different strategies for translating Unicode code points into actual stored bytes.
 
| Encoding | Bytes Used | Strategy | ASCII Compatible? |
|---|---|---|---|
| **UTF-8** | 1–4 bytes (variable) | Allocates only as many bytes as needed per character | Yes — 1-byte range identical to ASCII |
| **UTF-16** | 2 or 4 bytes (variable) | Common characters use 2 bytes; rare ones use a 4-byte "surrogate pair" | No |
| **UTF-32** | 4 bytes (fixed) | Every character uses exactly 4 bytes, regardless of complexity | No |
 
#### UTF-8 (most common on the modern web)
 
| Character Range | Bytes Used |
|---|---|
| ASCII (`U+0000`–`U+007F`) | 1 byte |
| Extended Latin/Greek (e.g., `Ω`) | 2 bytes |
| Most Asian scripts | 3 bytes |
| Emoji and rare scripts (e.g., `🔥`) | 4 bytes |
 
> UTF-8's backward compatibility with ASCII (identical 1-byte encoding for the first 128 characters) is a major reason for its widespread adoption — legacy ASCII text is already valid UTF-8.
 
#### UTF-16
 
```
A   (U+0041)         → 1 unit  (2 bytes)
🔥  (U+1F525)         → 2 units (4 bytes) → encoded as surrogate pair U+D83D U+DD25
```
 
#### UTF-32
 
```
A   → U+00000041   (4 bytes, always)
🔥  → U+0001F525   (4 bytes, always)
```
 
> UTF-32 is the simplest conceptually but the most wasteful in storage, since even simple ASCII characters consume the full 4 bytes.
 
---
 
### Worked Examples Across Encodings
 
| Character | Meaning | Unicode (UTF-16) | Unicode (UTF-32) |
|---|---|---|---|
| 龍 | Chinese "dragon" | `U+9F8D` | `U+00009F8D` |
| 😊 | Smiling face | — | `U+0001F60A` |
| ツ | Japanese "tsu" | `U+30C4` | `U+000030C4` |
| ت | Arabic "taa" | `U+062A` | — |
| ♞ | Chess black knight | `U+265E` | — |
 
---
 
### Summary
 
| Concept | Key Takeaway |
|---|---|
| **ASCII** | 7-bit, 128 characters, English-only |
| **Extended ASCII (ISO-8859-x)** | 8-bit regional patches; mismatched encodings cause garbled text |
| **Unicode** | Universal standard assigning a unique code point to every character, in every language |
| **UTF-8 / UTF-16 / UTF-32** | Different byte-storage strategies for representing Unicode code points |
