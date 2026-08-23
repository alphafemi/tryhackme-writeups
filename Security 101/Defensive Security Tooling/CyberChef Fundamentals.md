# CyberChef Fundamentals


---

**CyberChef** is a web-based "Swiss Army knife" for data — a toolbox of operations ranging from simple encodings (XOR, Base64) to complex cryptography (AES, RSA). Operations are chained together into a **recipe**, executed in sequence.

---

### Accessing CyberChef

| Method | Details |
|---|---|
| **Online** | `https://gchq.github.io/CyberChef` — just a browser and internet connection |
| **Offline/Local** | Download the latest release from `https://github.com/gchq/CyberChef/releases` — works on Windows and Linux; use the most stable version |

---

### The Four Areas of CyberChef

| Area | Purpose |
|---|---|
| **Operations** | A categorized, searchable library of all available operations |
| **Recipe** | Where operations are dragged, ordered, and configured — the "heart" of the tool |
| **Input** | Where data is pasted, typed, or dragged in |
| **Output** | Displays the result of the recipe applied to the input |

#### Operations Area — Sample Operations

| Operation | Description | Example |
|---|---|---|
| **From Morse Code** | Translates Morse Code to alphanumeric | `- .... .-. . .- - ...` → `THREATS` |
| **URL Encode** | Percent-encodes problematic characters | `https://tryhackme.com/...` → `https%3A%2F%2F...` |
| **To Base64** | Encodes raw data as Base64 | `This is fun!` → `VGhpcyBpcyBmdW4h` |
| **To Hex** | Converts input to hex bytes | `This Hex conversion...` → `54 68 69 73 20...` |
| **To Decimal** | Converts input to an ordinal integer array | `This Decimal conversion...` → `84 104 105...` |
| **ROT13** | Caesar cipher rotating letters (default: 13) | `Digital Forensics and Incident Response` → `Qvtvgny Sberafvpf naq Vapvqrag Erfcbafr` |

> Hovering over any operation shows a sample, description, and a link to further reading.

#### Recipe Area — Features

| Feature | Function |
|---|---|
| **Save recipe** | Saves the current selection of operations |
| **Load recipe** | Loads a previously saved recipe |
| **Clear recipe** | Clears the current recipe |
| **BAKE!** | Processes the input through the recipe |
| **Auto Bake** | Automatically re-processes on every change, without needing to click BAKE! |

#### Input Area — Features

| Feature | Function |
|---|---|
| **Add new input tab** | Work with multiple separate input values |
| **Open folder as input** | Upload an entire folder as input |
| **Open file as input** | Upload a single file as input |
| **Clear input and output** | Clears both panes |
| **Reset pane layout** | Restores the default window layout |

#### Output Area — Features

| Feature | Function |
|---|---|
| **Save output to file** | Saves the result as a `.dat` file |
| **Copy raw output** | Copies the result to the clipboard |
| **Replace input with output** | Overwrites the input with the current output — useful for chaining manual steps |
| **Maximise output pane** | Expands the output view |

---

### The CyberChef Thought Process

A four-step loop for approaching any task:

```
1. Set a clear objective  →  2. Load your data  →  3. Choose operations  →  4. Check the output
        ↑                                                                          │
        └──────────────────────── repeat if not the desired result ────────────────┘
```

1. **Define the objective** — e.g., *"I found a gibberish string during an investigation and want to know if it hides a message."*
2. **Input the data** — paste or upload the string into the Input area.
3. **Select operations** — this can be tricky without knowing the data's origin. If research suggests encryption/encoding, try relevant operations (ROT13, Base64, Base85, ROT47, etc.).
4. **Evaluate the output** — did it match the objective? If not, repeat from step 1.

---

### Common Operation Categories

#### Extractors

| Operation | Description |
|---|---|
| **Extract IP addresses** | Pulls all valid IPv4/IPv6 addresses from the input |
| **Extract URLs** | Pulls URLs from the input (requires a protocol like `http://` to avoid false positives) |
| **Extract email addresses** | Pulls strings matching `anything@domain.com` |

#### Date and Time

| Operation | Description |
|---|---|
| **From UNIX Timestamp** | Converts a UNIX timestamp to a readable datetime string |
| **To UNIX Timestamp** | Converts a UTC datetime string to its UNIX timestamp |

> A UNIX timestamp counts seconds since **January 1, 1970 UTC** (the "UNIX epoch"). E.g., `Fri Sep 6 20:30:22 +04 2024` → `1725654622`.

#### Data Format (Base Encodings)

| Operation | Description | Example |
|---|---|---|
| **From Base64** | Decodes Base64 back to raw data | `V2VsY29tZSB0byB0cnloYWNrbWUh` → `Welcome to tryhackme!` |
| **URL Decode** | Converts percent-encoded characters back to raw values | `https%3A%2F%2F...` → `https://...` |
| **From Base85** | Decodes Base85 (generally more efficient than Base64) | `BOu!rD]j7BEbo7` → `hello world` |
| **From Base58** | Decodes Base58 (omits easily-confused characters like `l`, `I`, `0`, `O`) | `AXLU7qR` → `Thm58` |
| **To Base62** | Encodes using a restricted human-friendly symbol set; shorter than decimal/hex | `Thm62` → `6NiRkOY` |

> Base64/85/58/62 are all **base encodings** — they transform binary data into text using a defined set of ASCII characters.

---

### Manual Base64 Encoding Walkthrough

Encoding the string **"THM"** by hand:

**Step 1 — Convert each character to 8-bit binary and concatenate:**

| Char | Binary |
|---|---|
| T | `01010100` |
| H | `01001000` |
| M | `01001101` |

Combined (24 bits): `010101000100100001001101`

**Step 2 — Split into 6-bit groups and convert each to decimal:**

| 6-bit Group | Decimal |
|---|---|
| `010101` | 21 |
| `000100` | 4 |
| `100001` | 33 |
| `001101` | 13 |

**Step 3 — Map each decimal value to its Base64 index character:**

| Index | Character |
|---|---|
| 21 | V |
| 4 | E |
| 33 | h |
| 13 | N |

**Result:** `THM` → **`VEhN`**

---

### URL Decode — Common Percent-Encoded Characters (UTF-8)

| Character | Encoded |
|---|---|
| `:` | `%3A` |
| `/` | `%2F` |
| `.` | `%2E` |
| `=` | `%3D` |
| `#` | `%23` |

---

### CyberChef Quick Reference

| Concept | Summary |
|---|---|
| CyberChef = | A browser-based data toolkit built around chained "recipes" |
| 4 areas | Operations, Recipe, Input, Output |
| Access | Online (`gchq.github.io/CyberChef`) or offline (GitHub release) |
| Bake | Processes data through the recipe (or use Auto Bake) |
| Thought process | Objective → Input → Operations → Check output (loop) |
| Extractors | IP addresses, URLs, email addresses |
| Date/Time ops | To/From UNIX Timestamp |
| Base encodings | Base64, Base85, Base58, Base62 |
| Base64 mechanics | 8-bit chars → concatenate → split into 6-bit groups → map to index table |

---

## Summary

- **CyberChef** is a browser-based data toolkit organized around four areas — **Operations** (searchable library), **Recipe** (chained, ordered operations), **Input**, and **Output** — accessible either online or as an offline release.
- Approaching unfamiliar data follows a repeatable loop: **set an objective → load the data → choose operations → check the output**, repeating until the result matches the objective.
- Common operation categories include **extractors** (IPs, URLs, email addresses), **date/time conversions** (UNIX timestamp ↔ readable datetime), and **base encodings** (Base64, Base85, Base58, Base62), each transforming binary data into a defined ASCII character set.
- Base64 encoding works by converting each character to 8-bit binary, concatenating the bits, splitting the result into 6-bit groups, and mapping each group's decimal value to a character in the Base64 index table.
