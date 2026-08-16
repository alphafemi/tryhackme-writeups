# Digital Forensics Fundamentals


---

**Forensics** is the application of methods and procedures to investigate and solve crimes. **Digital forensics** is the branch of forensics dedicated to investigating **cyber crime** — any criminal activity conducted on or using a digital device. Investigators use a range of tools and techniques to thoroughly examine digital devices after a crime, finding and analyzing evidence to support legal action.

---

### The NIST Four-Phase Process

Different cases call for different tools, but the **National Institute of Standards and Technology (NIST)** defines a general four-phase process that applies across digital forensics investigations.

```
Collection → Examination → Analysis → Reporting
```

| Phase | Description |
|---|---|
| **Collection** | Identify and collect data from devices found at the scene (PCs, laptops, cameras, USBs, etc.), without tampering with the original data. Every collected item must be properly documented. |
| **Examination** | Filter the (often overwhelming) collected data down to what's relevant — e.g., media from a specific date/time, or files belonging to one user account among many. |
| **Analysis** | Correlate the filtered data across multiple pieces of evidence to draw conclusions and reconstruct the relevant activity in chronological order. |
| **Reporting** | Produce a detailed report covering methodology, findings, and recommendations — including an executive summary — for law enforcement and executive management. |

---

### Types of Digital Forensics

| Type | Focus |
|---|---|
| **Computer forensics** | The most common type — investigating computers, the devices most frequently involved in crime. |
| **Mobile forensics** | Investigating mobile devices; extracting call records, text messages, GPS locations, etc. |
| **Network forensics** | Investigation beyond a single device, across the whole network — primarily via network traffic logs. |
| **Database forensics** | Investigating intrusions into databases that result in data modification or exfiltration. |
| **Cloud forensics** | Investigating data stored on cloud infrastructure — often harder due to limited available evidence. |
| **Email forensics** | Investigating emails to determine involvement in phishing or fraud campaigns. |

---

### Evidence Acquisition Best Practices

| Practice | Why It Matters |
|---|---|
| **Proper Authorization** | Evidence must be collected with authorization from relevant authorities. Evidence collected without prior approval may be **inadmissible in court**, since it often contains private/sensitive data. |
| **Chain of Custody** | A formal document tracking evidence details end-to-end, preserving integrity and admissibility. |
| **Write Blockers** | Hardware/software that prevents any write operations to the original evidence during collection, keeping the source device in its original state. |

#### Chain of Custody — Key Details to Record

- Description of the evidence (name, type)
- Name of individual(s) who collected the evidence
- Date and time of collection
- Storage location of each piece of evidence
- Access log: who accessed the evidence, and when

> Without a chain of custody, there's no way to hold anyone accountable if evidence goes missing or is altered — the document creates a verifiable trail proving the evidence's integrity and reliability in court.

> **Write blocker example:** Attaching a suspect's hard drive directly to a forensic workstation risks background processes altering file timestamps, corrupting analysis. A write blocker prevents any such alteration, keeping the drive in its original state.

---

### Windows Forensic Imaging

Forensic images are **bit-by-bit copies** of a system, taken during the collection phase. Windows investigations typically involve two categories:

| Image Type | Contains | Volatility | Priority |
|---|---|---|---|
| **Disk image** | All data on the storage device (HDD/SSD) — files, media, documents, browsing history | **Non-volatile** — survives a restart | Standard |
| **Memory image** | Data in RAM — open files, running processes, active network connections | **Volatile** — lost on power-off/restart | **Capture first** |

> **Order matters:** Because memory is volatile, the memory image should be acquired *before* the disk image (or before any shutdown/restart) — otherwise that data is lost permanently.

---

### Common Forensics Tools

| Tool | Purpose | Notes |
|---|---|---|
| **FTK Imager** | Disk image acquisition & analysis | GUI-based; supports multiple image formats; used for both acquisition and analysis. |
| **Autopsy** | Disk image analysis | Open-source platform; keyword search, deleted file recovery, file metadata, extension mismatch detection. |
| **DumpIt** | Memory image acquisition | Command-line tool for capturing RAM from Windows; supports multiple output formats. |
| **Volatility** | Memory image analysis | Open-source, plugin-based; each artifact type has its own plugin; supports Windows, Linux, macOS, and Android. |

---

### Digital Forensics Quick Reference

| Concept | Summary |
|---|---|
| 4 Phases (NIST) | Collection → Examination → Analysis → Reporting |
| Common types | Computer, Mobile, Network, Database, Cloud, Email |
| Key acquisition practices | Authorization, Chain of Custody, Write Blockers |
| Disk image | Non-volatile — collect anytime |
| Memory image | Volatile — collect **first** |
| Acquisition tools | FTK Imager (disk), DumpIt (memory) |
| Analysis tools | Autopsy (disk), Volatility (memory) |

---

## Summary

- **Digital forensics** applies investigative methods and procedures to cyber crime, using the **NIST four-phase process**: Collection → Examination → Analysis → Reporting.
- Common types span **computer, mobile, network, database, cloud, and email forensics**, each with its own investigative focus and challenges.
- Evidence acquisition rests on three practices: **proper authorization** (or evidence risks being inadmissible), a documented **chain of custody**, and **write blockers** to prevent altering the original evidence.
- Windows imaging distinguishes **disk images** (non-volatile, standard priority) from **memory images** (volatile — must be captured *first*, before any shutdown or restart, or the data is lost).
- Key tools split by acquisition vs. analysis and disk vs. memory: **FTK Imager** and **Autopsy** for disk work, **DumpIt** and **Volatility** for memory work.

## Related Documents

- [54. Security Operations Center (SOC) Fundamentals](./54-security-operations-center-soc-fundamentals.md)
- [46. Hashing — Integrity, Password Storage & Cracking](./46-hashing-integrity-password-storage-cracking.md)
