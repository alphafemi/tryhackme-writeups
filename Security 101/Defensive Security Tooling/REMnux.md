# REMnux — Malware Analysis Toolkit (oledump, INetSim, Volatility)



**REMnux** is a specialized Linux distro pre-loaded with malware analysis tools — Volatility, YARA, Wireshark, oledump, INetSim, and more — providing a ready-to-go sandbox for dissecting potentially malicious software without risking a primary system.

---

### Static Analysis: oledump.py

**oledump.py** analyzes **OLE2 files** (Object Linking and Embedding — Structured Storage / Compound File Binary Format), the container format Microsoft Office documents use to bundle multiple data types into one file. It's useful for extracting and examining embedded content, including malicious VBA macros.

#### Listing Streams

```bash
oledump.py agenttesla.xlsm
```

```
A: xl/vbaProject.bin
A1: 468 'PROJECT'
A2: 62 'PROJECTwm'
A3: m 169 'VBA/Sheet1'
A4: M 688 'VBA/ThisWorkbook'
A5: 7 'VBA/_VBA_PROJECT'
A6: 209 'VBA/dir'
```

- Each `A#` entry is a **data stream**.
- A capital **`M`** flag marks a stream containing a **Macro** — a strong signal to investigate further.

#### Extracting a Specific Stream

```bash
oledump.py agenttesla.xlsm -s 4
```

`-s` (`--select`) picks a specific stream by index (here, stream 4 — `VBA/ThisWorkbook`). Raw output appears as a **hex dump**, which is hard to read directly.

#### Decompressing VBA Macros

```bash
oledump.py agenttesla.xlsm -s 4 --vbadecompress
```

`--vbadecompress` automatically decompresses the VBA macro into readable source code, revealing the actual script logic — e.g., a `Workbook_Open()` macro that builds and executes an obfuscated PowerShell command.

#### Deobfuscating with CyberChef

Malicious VBA often obfuscates strings with junk characters (e.g., `*` and `^`) to evade static detection. Pasting the obfuscated string into **CyberChef** and applying **Find/Replace** twice (once for `*` → empty, once for `^` → empty) reveals the underlying command — e.g., a PowerShell one-liner that:

- Runs hidden (`-WindowStyle hidden`) and bypasses execution policy (`-executionpolicy bypass`)
- Downloads a payload via `Invoke-WebRequest -Uri <URL> -OutFile $TempFile`
- Executes the downloaded file via `Start-Process $TempFile`

> This is a common technique: a seemingly benign Office document triggers a macro that stages and launches a second-stage payload from a remote server, helping the attacker evade early detection.

---

### Dynamic Analysis: INetSim

**INetSim** (Internet Services Simulation Suite) simulates common internet services (HTTP/S, DNS, FTP, SMTP, POP3, etc.) so malware can be run and observed making "real" network connections — without actually reaching the internet.

#### Setup

1. **Find your REMnux VM's IP** (via `ifconfig` or the shell prompt).
2. **Edit the INetSim config:**
   ```bash
   sudo nano /etc/inetsim/inetsim.conf
   ```
   Uncomment and set `dns_default_ip` to the machine's own IP:
   ```
   dns_default_ip <MACHINE_IP>
   ```
   Save with `Ctrl+O`, Enter, then exit with `Ctrl+X`.
3. **Verify the change:**
   ```bash
   cat /etc/inetsim/inetsim.conf | grep dns_default_ip
   ```
4. **Start INetSim:**
   ```bash
   sudo inetsim
   ```
   Confirm `Simulation running.` appears (an `http_80_tcp - failed!` line can be ignored).

#### Simulating Malware Behavior

From a second machine (e.g., an AttackBox), visiting `https://<MACHINE_IP>` (accepting the self-signed cert warning) loads INetSim's fake homepage. Malware behavior — fetching a secondary payload — can be mimicked via:

```bash
sudo wget https://<MACHINE_IP>/second_payload.zip --no-check-certificate
sudo wget https://<MACHINE_IP>/second_payload.ps1 --no-check-certificate
```

These download **fake files** served by INetSim, simulating a malware sample retrieving a second-stage payload from a C2 server.

#### Connection Reports

Stopping INetSim generates a report of all captured connections:

```
Report written to '/var/log/inetsim/report/report.<PID>.txt'
```

```bash
sudo cat /var/log/inetsim/report/report.<PID>.txt
```

The report logs each connection's timestamp, protocol, method, requested URL, and the fake file served — giving analysts visibility into exactly what a sample tried to reach and retrieve.

---

### Memory Forensics: Volatility (Preprocessing)

For memory image analysis, **Volatility 3** extracts specific artifacts via plugins. Rather than reviewing each plugin's output live, evidence is commonly **preprocessed** — run once and saved to files for later, faster review.

#### Key Windows Plugins

| Plugin | Purpose |
|---|---|
| `windows.pstree.PsTree` | Lists processes in a tree by parent process ID |
| `windows.pslist.PsList` | Lists all currently active processes |
| `windows.cmdline.CmdLine` | Lists process command-line arguments |
| `windows.filescan.FileScan` | Scans for file objects in the memory image |
| `windows.dlllist.DllList` | Lists loaded modules (DLLs) |
| `windows.malfind.Malfind` | Lists process memory ranges that potentially contain injected code |
| `windows.psscan.PsScan` | Scans for processes present in the memory image |

**Running a single plugin:**

```bash
vol3 -f wcry.mem windows.pstree.PsTree
```

#### Bulk Preprocessing with a Loop

Rather than running each plugin manually, loop through them and save each output to its own text file:

```bash
for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree \
windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do
  vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt
done
```

- `-q` — quiet mode (suppresses progress output)
- `-f wcry.mem` — the memory image to analyze
- `> wcry.$plugin.txt` — saves each plugin's output to its own file, named after the plugin

This produces one `.txt` file per plugin in the working directory, ready for later review.

---

### Memory Forensics: Preprocessing with `strings`

The Linux `strings` utility extracts printable text from the memory image in multiple encodings, each potentially revealing different artifacts:

```bash
# ASCII strings
strings wcry.mem > wcry.strings.ascii.txt

# 16-bit little-endian Unicode strings
strings -e l wcry.mem > wcry.strings.unicode_little_endian.txt

# 16-bit big-endian Unicode strings
strings -e b wcry.mem > wcry.strings.unicode_big_endian.txt
```

| Flag | Extracts |
|---|---|
| *(none)* | Printable ASCII text |
| `-e l` | 16-bit little-endian strings |
| `-e b` | 16-bit big-endian strings |

Preprocessing evidence this way — running tools once and saving structured output — lets any analyst who picks up the case search and analyze faster without re-running lengthy tools.

---

### REMnux Quick Reference

| Concept | Summary |
|---|---|
| REMnux = | Pre-built Linux malware analysis sandbox |
| oledump.py | Analyzes OLE2 (Office) files for embedded streams/macros |
| Stream selection | `-s <index>` (`--select`) |
| Macro decompression | `--vbadecompress` |
| Macro deobfuscation | CyberChef Find/Replace on junk chars (e.g., `*`, `^`) |
| INetSim | Simulates internet services for safe dynamic analysis |
| INetSim config | `/etc/inetsim/inetsim.conf` → `dns_default_ip` |
| INetSim start | `sudo inetsim` |
| INetSim reports | `/var/log/inetsim/report/` |
| Volatility 3 | `vol3 -f <mem_image> <plugin>` |
| Key plugins | PsTree, PsList, CmdLine, FileScan, DllList, Malfind, PsScan |
| Bulk preprocessing | Shell `for` loop over plugins, redirecting output to files |
| `strings` encodings | ASCII (default), `-e l` (LE Unicode), `-e b` (BE Unicode) |

---

## Summary

- **REMnux** is a Linux distribution pre-loaded with malware analysis tools (Volatility, YARA, Wireshark, oledump, INetSim), providing a ready-made sandbox for safely dissecting malicious samples.
- **oledump.py** analyzes OLE2-format Office documents, listing embedded data streams and flagging macro-containing streams (`M`) — `-s` selects a specific stream, and `--vbadecompress` recovers readable VBA source code, which can then be deobfuscated in CyberChef to reveal staged payload downloads.
- **INetSim** simulates common internet services (HTTP/S, DNS, FTP, SMTP, etc.) so malware can be safely run and observed "phoning home" without ever reaching the real internet, with a full connection report generated on stop.
- **Volatility 3** extracts specific artifacts from memory images via plugins (`PsTree`, `PsList`, `CmdLine`, `FileScan`, `DllList`, `Malfind`, `PsScan`) — commonly run in bulk via a shell loop to preprocess evidence into per-plugin text files for later review, complemented by the `strings` utility for extracting ASCII and Unicode text directly from the memory image.


- [55. Digital Forensics Fundamentals](./55-digital-forensics-fundamentals.md)
