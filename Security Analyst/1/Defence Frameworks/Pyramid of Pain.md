# Pyramid of Pain & Indicators of Compromise



The **Pyramid of Pain** models different types of indicators of compromise (IOCs) by how much "pain" it causes an attacker when defenders detect and block each type — the higher up the pyramid, the harder it is for the attacker to adapt.

```
        TTPs                (hardest for attacker to change)
      Tools
    Network/Host Artifacts
  Domain Names
Hash Values                  (easiest for attacker to change)
```

---

### Hash Values (Bottom of the Pyramid)

A **hash value** is a fixed-length numeric string uniquely identifying data, produced by a hashing algorithm.

| Algorithm | Bit Length | Notes |
|---|---|---|
| **MD5** | 128-bit | Designed by Ron Rivest (1992), defined in RFC 1321. **Not cryptographically secure** — RFC 6151 (2011) documented collision attacks against it. |
| **SHA-1** | 160-bit (40-hex-digit) | Developed by the NSA (1995), defined in RFC 3174. Deprecated by NIST in 2011, banned for digital signatures by end of 2013 due to brute-force susceptibility. |
| **SHA-2 (e.g., SHA-256)** | 256-bit (64-hex-digit) | Developed by NIST/NSA (2001) to replace SHA-1; SHA-256 is the most common variant. |

> A hash is not cryptographically secure if two different files can produce the same hash (a **collision**).

**Use in security:** Hashes uniquely identify malware samples for lookup via tools like **VirusTotal** and **MetaDefender Cloud (OPSWAT)**. Security reports (e.g., from The DFIR Report) commonly list sample hashes as reference IOCs.

**Weakness — trivial to defeat:** Appending even a single byte to a file completely changes its hash:

```powershell
# Before modification
Get-FileHash .\OpenVPN_2.5.1_I601_amd64.msi -Algorithm MD5
# MD5: D1A008E3A606F24590A02B853E955CF7

# Append a string to the file
echo "AppendTheHash" >> .\OpenVPN_2.5.1_I601_amd64.msi

# After modification
Get-FileHash .\OpenVPN_2.5.1_I601_amd64.msi -Algorithm MD5
# MD5: 9D52B46F5DE41B73418F8E0DACEC5E9F
```

This trivial mutability is exactly why hash-based detection sits at the **bottom** of the pyramid — it's the easiest IOC type for an attacker to invalidate.

---

### Domain Names

More costly for attackers to change — requires purchasing, registering, and configuring DNS for a new domain (though loose DNS provider standards can ease this).

#### Punycode Attacks

**Punycode** converts non-ASCII characters into ASCII-compatible encoding for domain names — attackers abuse this to register lookalike domains.

> **Example:** `adıdas.de` (with a Turkish dotless-i-like character) encodes to Punycode as `xn--addas-o4a.de` — visually near-identical to the legitimate brand domain, tricking users into visiting a malicious site.

Modern browsers (IE, Chrome, Edge, Safari) generally now render the full Punycode string rather than the deceptive visual rendering, mitigating this.

**Detection:** Proxy logs and web server logs can surface suspicious domain activity.

#### URL Shorteners

Attackers frequently hide malicious domains behind shortening services (`bit.ly`, `goo.gl`, `ow.ly`, `tinyurl.com`, `x.co`, etc.). Appending `+` to many shortened URLs reveals the underlying redirect destination before clicking.

#### Sandbox Analysis (Any.run example)

Sandboxing tools like **Any.run** detonate a sample and record its network behavior across three tabs:

| Tab | Shows |
|---|---|
| **HTTP Requests** | Resources retrieved from a web server (e.g., a dropper or callback) |
| **Connections** | Communications with other hosts (e.g., C2 traffic, FTP transfers) |
| **DNS Requests** | Domain lookups — malware often checks for internet connectivity ("call home" checks) before proceeding |

> **Caution:** Never directly visit IPs or URLs observed in a malware sandbox report — they are, by definition, malicious infrastructure.

---

### Host & Network Artifacts (Yellow Zone)

At this level, detection meaningfully disrupts the attacker — they must revise tools and methodology, costing time and resources.

#### Host Artifacts

Traces left on a compromised system: registry values, suspicious process executions, dropped files, or other environment-specific IOCs (e.g., a Word process spawning something suspicious, or files dropped by a malicious application).

#### Network Artifacts

Observable network-level indicators: **User-Agent strings**, C2 infrastructure details, or URI patterns in HTTP POST requests.

> A malware family may consistently use an unusual or previously-unseen **User-Agent** string. Detecting and blocking this custom string forces the attacker to change infrastructure.

**Detection tools:** Wireshark/TShark for PCAP analysis, or IDS tools like Snort for live alerting.

```bash
tshark -Y http.request -T fields -e http.host -e http.user_agent -r analysis_file.pcap
```

This extracts hostnames and User-Agent strings from HTTP requests in a packet capture — useful for spotting known-malicious signatures (e.g., custom User-Agents associated with the **Emotet** trojan).

---

### Tools (Red Zone)

Detecting an attacker's **tools** (custom malware, backdoors, maldocs, password crackers) is a major setback — the attacker may need to invest money or time building or acquiring a replacement tool entirely.

**Key defensive resources:**

| Resource | Purpose |
|---|---|
| **Antivirus signatures, detection rules, YARA rules** | Direct detection weapons against known tool artifacts |
| **MalwareBazaar, Malshare** | Access to malware samples, feeds, and YARA scan results for threat hunting/IR |
| **SOC Prime Threat Detection Marketplace** | Shared detection rules covering emerging threats and actively-exploited CVEs |
| **Fuzzy Hashing (e.g., SSDeep)** | Similarity analysis — matches files with minor variations, unlike exact-match hashing |

> **Fuzzy hashing advantage:** Unlike MD5/SHA which change completely with any file modification, fuzzy hashes (e.g., SSDeep) can still flag files as *similar* despite minor tool variations — closing the loophole attackers exploit at the hash-value level.

---

### TTPs (Apex of the Pyramid)

**TTPs** — Tactics, Techniques, and Procedures — represent the complete adversary playbook, mapped comprehensively by the **MITRE ATT&CK Matrix**: every step from initial phishing through persistence and exfiltration.

Detecting and responding at the TTP level is the most disruptive to an attacker.

> **Example:** Detecting a **Pass-the-Hash** attack via Windows Event Log monitoring lets defenders identify the compromised host quickly and stop lateral movement before it spreads.

At this level, the attacker faces two costly choices:

1. Retreat, retrain, and rebuild their entire approach
2. Abandon the target and move on

Option 2 is often the more resource-efficient choice for the attacker — meaning **detecting TTPs causes maximum disruption** to their operation.

---

### Pyramid of Pain Quick Reference

| Level | IOC Type | Attacker Cost to Change | Example Tools |
|---|---|---|---|
| 1 (bottom) | **Hash Values** | Trivial — append 1 byte | VirusTotal, MetaDefender |
| 2 | **Domain Names** | Moderate — buy/register a new domain | Proxy logs, web server logs |
| 3 (yellow) | **Host/Network Artifacts** | Moderate-High — revise tooling/methodology | Wireshark, TShark, Snort |
| 4 (red) | **Tools** | High — build/acquire new tooling | YARA, MalwareBazaar, SSDeep |
| 5 (apex) | **TTPs** | Highest — retrain, rebuild entire approach | MITRE ATT&CK, Windows Event Log monitoring |

---

## Summary

- The **Pyramid of Pain** ranks IOC types by how much disruption detecting each one causes an attacker, from easiest-to-change (bottom) to hardest (apex): **Hash Values → Domain Names → Host/Network Artifacts → Tools → TTPs**.
- **Hash values** (MD5, SHA-1, SHA-256) are trivial to invalidate — appending a single byte to a file completely changes its hash — making them the weakest, though still useful for known-sample lookups via VirusTotal or MetaDefender.
- **Domain names** cost more to replace (registration, DNS setup) and are abused via **Punycode** lookalike domains and **URL shorteners**; sandboxing tools like Any.run reveal a sample's HTTP, connection, and DNS behavior for domain-level IOC extraction.
- **Host and network artifacts** (registry changes, dropped files, unusual User-Agent strings) meaningfully disrupt an attacker, detected via tools like Wireshark, TShark, and Snort.
- Detecting an attacker's **tools** (malware, backdoors, password crackers) forces costly rebuilding, aided by YARA rules, MalwareBazaar/Malshare feeds, and fuzzy hashing (SSDeep) that catches minor variations exact-match hashing would miss.
- Detecting **TTPs** — the complete adversary playbook mapped by MITRE ATT&CK — is the most disruptive, often forcing an attacker to abandon the target entirely rather than rebuild their whole approach.

## Related Documents

- [80. MITRE ATT&CK, CAR, D3FEND & Related Frameworks](./80-mitre-attck-car-d3fend-related-frameworks.md)
- [78. The Cyber Kill Chain](./78-the-cyber-kill-chain.md)
- [46. Hashing — Integrity, Password Storage & Cracking](./46-hashing-integrity-password-storage-cracking.md)
- [63. CAPA — Malware Capability Analysis](./63-capa-malware-capability-analysis.md)
