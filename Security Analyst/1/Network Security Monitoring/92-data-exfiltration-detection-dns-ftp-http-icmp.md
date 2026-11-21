# Data Exfiltration Detection (DNS, FTP, HTTP, ICMP)

## Overview

This document covers detecting data exfiltration: why adversaries exfiltrate data and which real-world threat actors use which techniques, the four common exfiltration phases, an overview of network, host, cloud, covert, and insider techniques with where to look for each, and detailed detection walkthroughs using Wireshark and Splunk for DNS, FTP, HTTP, and ICMP exfiltration.

---

**Data exfiltration** is the unauthorized transfer of sensitive data from a computer or network to an external, adversary-controlled destination — a primary objective for attackers who have already breached a network. It can be deliberate (an insider) or carried out via malware/compromised credentials.

---

### Why Adversaries Exfiltrate Data

| Motive | Description |
|---|---|
| **Financial Gain** | Stolen data (credit card info, personal records) sold on the dark web or used for fraud |
| **Espionage** | Nation-state actors targeting IP, trade secrets, or classified data for strategic advantage |
| **Ransomware & Extortion** | Data stolen and leak threatened unless a ransom is paid |
| **Disruption & Sabotage** | Damaging reputation/operations by leaking internal data |
| **Persistence & Reconnaissance** | Exfiltrated data helps plan future attacks |

### Real-World Threat Actors & Techniques

| Threat Actor / Campaign | Exfiltration Technique | Description |
|---|---|---|
| **APT29 (Cozy Bear)** | HTTPS over legitimate domains | Encrypted HTTPS channels used against government networks |
| **FIN7** | HTTP POST to C2 servers | Stolen data embedded in POST requests to evade detection |
| **Lunar Spider (Zloader)** | Encrypted C2 channels | Two-month intrusion using encrypted, staged exfiltration |
| **DarkSide Ransomware** | Dual extortion (encryption + exfil) | Stole data before encrypting, then threatened public leaks |
| **APT10 (Cloud Hopper)** | Cloud-to-cloud transfer | Exfiltrated via managed service provider cloud APIs |

### Common Exfiltration Phases

```
Discovery/Collection → Staging/Compression → Exfiltration Transport → C2 Coordination
```

1. **Discovery/Collection** — attacker locates sensitive files
2. **Staging/Compression** — aggregates, compresses, encrypts, or encodes (ZIP, RAR, 7z, tar, base64, steganography)
3. **Exfiltration Transport** — transfer over network, removable media, cloud, or covert channels
4. **C2 Coordination** — orchestrates the transfer and confirms receipt

---

### Techniques and Indicators Overview

| Technique Category | Examples | Where to Look |
|---|---|---|
| **Network-based** | HTTP/HTTPS uploads to S3/Azure Blob/webmail, FTP/SFTP/SCP, DNS tunneling, ICMP/covert protocols, custom TCP/UDP | Proxy/gateway logs (large POSTs), firewall/NGFW flows (high bytes to one IP/ASN), NetFlow spikes, DNS logs (long hostnames, TXT queries) |
| **Host-based** | PowerShell/Invoke-WebRequest, rclone, awscli, curl/wget, archive creation, USB use, ADS/hidden streams | Sysmon/EDR (Process Create, Network Connect, File Create), Windows Security (4663/4656), auditd/shell history, removable-media events |
| **Cloud exfiltration** | S3 PutObject/multipart upload, Azure Blob uploads, GCS object inserts, Drive/SharePoint external sharing | CloudTrail / Azure Activity / GCP Audit, cloud storage access logs, unusual service-account/IP activity |
| **Covert & encoding** | DNS tunneling, base64/chunked encoding, steganography, low-and-slow small requests | DNS logs, proxy logs with many small POSTs, correlating intermittent uploads with suspicious process activity |
| **Insider & collaboration tools** | Slack/Teams/Dropbox/Drive/Box uploads or external sharing, compromised accounts | Audit logs (share events, downloads), mail logs |

> **General IoA pattern:** Large outbound volume to external IPs/domains, unknown destinations, suspicious process command lines, many file reads followed by an outbound connection, multipart/streamed uploads — correlated across Proxy/Firewall/NetFlow, DNS, Sysmon/EDR (Event IDs 1/3/11), and mail logs.

---

### DNS Exfiltration

DNS is normally allowed through firewalls and often unfiltered/forwarded to public resolvers — making it attractive for covert channels. Attackers encode data into subdomain labels or TXT responses.

#### Why Attackers Use DNS

- **Always-on** — DNS lookups are routine and almost always permitted outbound
- **High cover** — queries look normal unless inspected closely
- **Flexible payload** — data can be encoded into subdomains or TXT responses

#### Indicators of Attack

- Many DNS queries to a single external domain, far above baseline
- Long subdomain labels / full query names (>60–100 characters)
- High entropy or Base32/Base64-like patterns in the query name
- Rare record types (TXT, NULL) or many large TXT responses
- Unusual response behavior (frequent NXDOMAIN, TCP/large UDP fragments)
- Queries at regular intervals (beaconing)

#### Detecting via Wireshark (`dns_exfil.pcap`)

| Step | Filter |
|---|---|
| All DNS traffic | `dns` |
| Queries with no response | `dns.flags.response == 0` |
| Long queries (suspicious subdomains) | `dns && frame.len > 70` |
| Narrow to the suspicious domain | `dns && dns.qry.name contains <domain>` |

**Finding:** Multiple internal hosts compromised, all sending chunked data via DNS tunneling to a single external domain.

#### Detecting via Splunk

```spl
index=data_exfil sourcetype=DNS_logs
```

```spl
index="data_exfil" sourcetype="DNS_logs" | stats count by src_ip
```

```spl
index="data_exfil" sourcetype="dns_logs" | stats count by query | sort -count
```

```spl
index="data_exfil" sourcetype="DNS_logs" | where len(query) > 30
```

**Key indicators confirmed:** Large number of DNS requests with no response + unusually long query length.

---

### FTP Exfiltration

FTP is one of the oldest file-transfer protocols; attackers use it (legitimate/misconfigured servers, compromised credentials, non-standard ports) to move large volumes of data off a network.

#### Indicators of Attack

- `USER`/`PASS` commands (cleartext credentials)
- `STOR` (upload)/`RETR` (download) — repeated or large transfers
- Large data connections to unusual external IPs, especially off-hours
- PASV-mode ephemeral-port data channels paired with large payloads

#### Detecting via Wireshark (`ftp-lab.pcap`)

| Step | Filter |
|---|---|
| Isolate FTP control/data | `ftp \|\| ftp-data` |
| Login attempts | `ftp.request.command == "USER" \|\| ftp.request.command == "PASS"` |
| Upload activity | `ftp contains "STOR"` |
| Suspicious file types | `ftp contains "csv"` |
| Large payloads | `ftp && frame.len > 90` |

**Workflow:** Right-click a packet → **Follow → TCP Stream** to view file content directly.

**Finding:** A suspicious IP connected via a **Guest** account transferred sensitive CSV files to an external IP — confirmed via TCP stream reconstruction of a document containing sensitive information.

---

### HTTP Exfiltration

HTTP blends with normal web traffic, traverses firewalls/proxies easily, and can be obfuscated — making it a common exfiltration vector.

#### How Adversaries Use HTTP

| Method | Description |
|---|---|
| **POST uploads** | Bulk data sent to attacker-controlled hosts/cloud storage in request bodies |
| **GET with encoded data** | Small chunks squeezed into query strings/paths (low-and-slow) |
| **Common services/CDN abuse** | Exfil disguised as uploads to popular services or subdomains under reputable domains |
| **Custom headers** | Data hidden in headers (e.g., `X-Data: <base64>`) to bypass string-based DLP |
| **Chunked/multipart transfer** | Large payloads split across multiple requests to avoid size thresholds |
| **HTTPS/TLS tunneling** | Encrypted channel hides payload — needs TLS inspection, SNI analysis, or metadata detection |
| **Staging via cloud services** | Upload to Dropbox/GitHub/Gist, then fetch externally |

#### Indicators of Attack

- Unusually large POST requests to external/unexpected hosts
- Requests to low-reputation or rarely-seen domains
- Frequent small requests (beaconing) followed by a large upload
- Chunked/multipart transfers composing a larger file

#### Detecting via Splunk

```spl
index="data_exfil" sourcetype="http_logs"
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST | stats count avg(bytes_sent) max(bytes_sent) min(bytes_sent) by domain | sort - count
```

```spl
index="data_exfil" sourcetype="http_logs" method=POST bytes_sent > 600 | table _time src_ip uri domain dst_ip bytes_sent | sort - bytes_sent
```

#### Detecting via Wireshark (`http_lab.pcap`)

| Step | Filter |
|---|---|
| All HTTP traffic | `http` |
| POST requests only | `http.request.method == "POST"` |
| Progressively narrow by size | `http.request.method == "POST" and frame.len > 500` → `> 750` |

**Finding:** One matching entry correlating with the Splunk result — confirmed via HTTP stream as a secret document being exfiltrated to an external IP.

---

### ICMP Exfiltration

ICMP is a diagnostic/control protocol (ping, TTL exceeded) that's commonly allowed through firewalls and typically inspected less strictly than TCP/UDP — making it attractive for tunneling encoded data.

#### How Adversaries Use ICMP

| Technique | Description |
|---|---|
| **Echo tunneling** | Base64/hex-encoded file chunks placed inside ICMP echo request/reply payloads |
| **Custom types/codes** | Uncommon ICMP types/codes to dodge signature-based detection |
| **Fragmentation/reassembly** | Large payloads split across multiple packets |
| **Encryption/obfuscation** | Payloads encoded (commonly base64) to resemble random data |

#### Indicators of Attack

- Persistent ICMP sessions to an external host with no legitimate monitoring purpose
- Unusually large ICMP payloads (normal ping ≈ 64 bytes)
- High-entropy or base64/hex-patterned ICMP payload data
- Bursts of ICMP with no other legitimate app traffic from the same host
- Regular timing/periodicity of similar-sized payloads
- Reassembled fragments from the same src/dst pair

#### Detecting via Wireshark (`icmp_lab.pcap`)

| Step | Filter |
|---|---|
| All ICMP traffic | `icmp` |
| Echo requests only | `icmp.type == 8` |
| Large ICMP packets | `icmp.type == 8 and frame.len > 100` |

> **Rule of thumb:** Normal pings total ~74 bytes. Anything significantly over 100 bytes is suspicious and warrants payload inspection.

---

### Data Exfiltration Detection Quick Reference

| Concept | Summary |
|---|---|
| Data exfiltration = | Unauthorized transfer of sensitive data to an adversary-controlled destination |
| 5 Motives | Financial gain, Espionage, Ransomware/Extortion, Disruption/Sabotage, Persistence/Recon |
| 4 Exfil phases | Discovery/Collection → Staging/Compression → Transport → C2 Coordination |
| DNS exfil tell | High query volume/length to one domain, high entropy, no-response pattern |
| FTP exfil tell | STOR commands, cleartext creds, large transfers to unusual IPs |
| HTTP exfil tell | Large POSTs to unknown domains, chunked/multipart uploads, beaconing |
| ICMP exfil tell | Payload size far above ~64-byte baseline, regular timing, high entropy |
| Core detection principle | Correlate host, network, and cloud telemetry — not single-point alerts |
| Key tools used | Wireshark (pcap analysis), Splunk (log correlation) |
| Universal workflow | Filter → narrow by size/pattern → Follow TCP Stream / correlate logs → confirm exfiltrated content |

---

## Summary

- **Data exfiltration** is the unauthorized transfer of sensitive data to an adversary-controlled destination, driven by **financial gain, espionage, ransomware/extortion, disruption/sabotage,** or **persistence/reconnaissance**, and used by real actors such as APT29 (HTTPS over legitimate domains), FIN7 (HTTP POST to C2), Lunar Spider (encrypted staged exfiltration), DarkSide (dual extortion), and APT10 (cloud-to-cloud transfer).
- Exfiltration follows four phases — **Discovery/Collection → Staging/Compression → Exfiltration Transport → C2 Coordination** — across network, host, cloud, covert/encoded, and insider/collaboration-tool techniques, and the general indicator pattern is large outbound volume to unknown destinations, suspicious command lines, and many file reads followed by an outbound connection, best detected by **correlating** proxy/firewall/NetFlow, DNS, Sysmon/EDR, and mail logs.
- **DNS exfiltration** shows as high query volume to one domain, long high-entropy subdomains, rare record types like TXT or NULL, and queries with no response — found in Wireshark with filters like `dns && frame.len > 70` and in Splunk with `len(query) > 30`.
- **FTP exfiltration** shows as cleartext `USER`/`PASS`, `STOR` uploads, and large transfers to unusual external IPs, with **Follow → TCP Stream** revealing the file contents (in the example, sensitive CSVs sent via a Guest account); **HTTP exfiltration** shows as large POSTs to unknown domains, chunked/multipart uploads, and beaconing, confirmed by correlating Splunk results with Wireshark filters on POST size.
- **ICMP exfiltration** hides encoded data in echo payloads — a normal ping is roughly 64 bytes (about 74 on the wire), so packets well over 100 bytes (`icmp.type == 8 and frame.len > 100`), especially with regular timing and high-entropy payloads, warrant payload inspection.

## Related Documents

- [91. Network Discovery & Port Scanning Detection](./91-network-discovery-port-scanning-detection.md)
- [93. Man-in-the-Middle (MITM) Attack Detection — ARP Spoofing, DNS Spoofing & SSL Stripping](./93-man-in-the-middle-mitm-attack-detection-arp-spoofing-dns-spoofing-ssl-stripping.md)
- [88. Wireshark Traffic Analysis — Scans, ARP Poisoning, Tunneling & Cleartext Protocols](./88-wireshark-traffic-analysis-scans-arp-poisoning-tunneling-cleartext-protocols.md)
- [74. Splunk Basics](./74-splunk-basics.md)
