# Identity/Asset Inventory & SOC Workbooks


---

Alert triage frequently requires gathering context about the people and systems involved. Consider an alert: *G.Baker logged into HQ-FINFS-02, downloaded a financial report, and shared it with R.Lund.* Properly triaging this means answering:

- Who is G.Baker — role, working hours?
- What is HQ-FINFS-02 — purpose, who can access it?
- Why would R.Lund need this data?

Two key lookup resources answer these questions: **Identity Inventory** and **Asset Inventory**.

---

### Identity Inventory

A catalogue of corporate **user accounts** (employees) and **machine accounts** (services), including privileges, contacts, and roles — used to determine whether observed activity matches a person's or service's expected behavior.

**Example entries:**

| Full Name | Username | Role | Location | Access |
|---|---|---|---|---|
| Gregory Baker | `G.Baker` | Chief Financial Officer | Europe, UK | VPN, HQ, FINANCE |
| Raymond Lund | `R.Lund` | US Financial Adviser | US, Texas | VPN, FINANCE |
| Kate Danner | `K.Danner` | Chief Technology Officer | Europe, UK | VPN, DA, HQ, AWS |
| `svc-veeam-06` | — | Backup Service Account | N/A | VEEAM, DMZ, HQ |
| `svc-nginx-pp` | — | Web App Service Account | N/A | DMZ |

**Sources of Identity Data:**

| Source | Examples | Notes |
|---|---|---|
| **Active Directory** | On-prem AD, Entra ID | AD itself functions as an identity database; widely used by SOC teams |
| **SSO Providers** | Okta, Google Workspace | Cloud-based alternative to AD, easy to search |
| **HR Systems** | BambooHR, SAP, HiBob | Limited to employees, but often has richer employee data |
| **Custom Solutions** | CSV/Excel sheets | Common homegrown option for IT/security teams |

---

### Asset Inventory

A catalogue of computing resources (servers, workstations) in the environment — this room's scope excludes software/other asset types. Used to understand a system's purpose, ownership, and location.

**Example entries:**

| Hostname | Location | IP Address | OS | Owner | Purpose |
|---|---|---|---|---|---|
| `HQ-FINFS-02` | UK Datacenter | `172.16.15.89` | Windows Server 2022 | Central IT | File server for financial records |
| `HQ-ADDC-01` | UK Datacenter | `172.16.15.10` | Windows Server 2019 | Central IT | Primary AD domain controller |
| `PC-891D` | London Office | `192.168.5.13` | Windows 11 Pro | Tech Support | Stationary PC for accountants |
| `L007694` | Remote | N/A | macOS 13 | A. Kelly, DevOps | Corporate laptop |
| `L005325` | Remote | N/A | macOS 13 | J. Eldridge, HR | Corporate laptop |

**Sources of Asset Data:**

| Source | Examples | Notes |
|---|---|---|
| **Active Directory** | On-prem AD, Entra ID | Doubles as a strong asset inventory database |
| **SIEM or EDR** | Elastic, CrowdStrike | Some agents collect host metadata automatically |
| **MDM Solutions** | MS Intune, Jamf | Purpose-built for listing/managing devices |
| **Custom Solutions** | CSV/Excel sheets | Same DIY pattern as identity inventory |

---

### SOC Workbooks

A **workbook** (also called a playbook, runbook, or workflow) is a structured document defining the exact steps to investigate and remediate a specific threat type — consistently and efficiently. Senior analysts create these to support L1 analysts, who are junior and not expected to handle every scenario perfectly on their own. Following workbooks precisely helps avoid mistakes and keeps triage quality consistent.

#### Workbook Structure — Three Logical Stages

| Stage | Purpose |
|---|---|
| **Enrichment** | Use Threat Intelligence and identity inventory to gather context about the affected user |
| **Investigation** | Combine gathered data with SIEM logs to determine whether the activity (e.g., a login) was expected |
| **Escalation** | Escalate to L2 or contact the user directly if the activity can't be confirmed as safe |

**Example — "Unusual Login Location" Workbook:**

A typical flow: receive the login alert → enrich the user's identity (e.g., via BambooHR) → check threat intelligence on the source IP → investigate in SIEM (e.g., Splunk) for surrounding user behavior → based on findings, either escalate to L2 or close the alert.

Most workbooks are diagrams supplemented by a detailed textual guide and links to the relevant tools/resources — ensuring every analyst follows the same evidence-based path to a verdict rather than guessing.

---

### Identity/Asset Inventory & Workbooks Quick Reference

| Concept | Summary |
|---|---|
| Identity inventory | Catalogue of user/service accounts, roles, privileges, contacts |
| Asset inventory | Catalogue of servers/workstations, their IPs, OS, owner, purpose |
| Identity sources | Active Directory, SSO providers, HR systems, custom sheets |
| Asset sources | Active Directory, SIEM/EDR, MDM solutions, custom sheets |
| SOC Workbook = | Playbook/runbook/workflow — structured investigation steps for a specific threat type |
| Why workbooks matter | Keeps L1 triage consistent and evidence-based, reduces missed steps |
| 3 Workbook stages | Enrichment → Investigation → Escalation |

---

## Summary

- Triaging an alert accurately often requires two lookup resources: an **Identity Inventory** (user and machine accounts, roles, privileges, contacts) and an **Asset Inventory** (servers/workstations, IPs, OS, owner, purpose).
- Identity data commonly comes from **Active Directory/Entra ID**, **SSO providers** (Okta, Google Workspace), **HR systems** (BambooHR, SAP, HiBob), or custom spreadsheets; asset data comes from the same AD source plus **SIEM/EDR** metadata and **MDM solutions** (Intune, Jamf).
- A **SOC Workbook** (playbook/runbook/workflow) is a structured, senior-analyst-authored document defining exact investigation steps for a given threat type, built around three stages: **Enrichment** (gather context), **Investigation** (correlate with SIEM logs), and **Escalation** (hand off to L2 or contact the user if the activity can't be confirmed safe).
- Workbooks exist to keep L1 triage consistent and evidence-based — a typical example (Unusual Login Location) chains identity enrichment, threat intel lookup, and SIEM investigation before reaching a close-or-escalate decision.

