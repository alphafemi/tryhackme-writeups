# Splunk Basics


---

**Splunk** is one of the leading SIEM solutions, collecting, analyzing, and correlating network and machine logs in real time to improve visibility and speed up detection.

---

### The Three Core Components

```
Log Sources → Forwarder → Indexer → Search Head
```

| Component | Role |
|---|---|
| **Forwarder** | Lightweight agent installed on monitored endpoints; collects data and sends it to the Splunk instance with minimal performance impact |
| **Indexer** | Parses and normalizes incoming data into field-value pairs, categorizes it, and stores it as searchable **events** |
| **Search Head** | Where users search indexed data (via **Search & Reporting**), using **SPL** (Search Processing Language), and build visualizations |

#### Forwarder — Example Data Sources

- Web servers generating web traffic logs
- Windows machines generating Event Logs, PowerShell, and Sysmon data
- Linux hosts generating host-centric logs
- Databases generating connection requests, responses, and errors

#### Search Head

Queries run via **SPL**, a purpose-built query language for indexed data. Results return as field-value pairs and can be transformed into tables and visualizations (pie, bar, column charts, etc.).

---

### The Splunk Home Screen

| Section | Purpose |
|---|---|
| **Splunk Bar** | Top panel — Messages (system notifications), Settings, Activity (search job progress), Help (docs/tutorials), Find (search across the app); also lets you switch between installed apps |
| **Apps Panel** | Lists installed Splunk apps; every installation includes **Search & Reporting** by default |
| **Explore Splunk** | Quick links to add data, install new apps, and access documentation |
| **Home Dashboard** | Displays selected dashboards (none by default); dashboards can be chosen from a dropdown, browsed via the listing page, or custom-built and viewed under the **Yours** tab |

---

### Data Ingestion

Splunk can ingest virtually any data source — event logs, website logs, firewall logs, and more — transforming each into individual searchable **events**. Data sources are grouped into categories in the **Add Data** interface, with **Upload** being one option for local files.

#### 5-Step Upload Process

1. **Select Source** — choose the log file and data source
2. **Select Source Type** — specify the log format (JSON, syslog, etc.)
3. **Input Settings** — choose the destination **index** and associated **hostname**
4. **Review** — confirm all configuration choices
5. **Done** — complete the upload; data becomes ready for analysis

---

### Worked Example: Ingesting VPN Logs

**Scenario:** Uploading a newline-delimited JSON file (`VPN_logs`) so Splunk treats each line as a distinct event.

**Steps:**

1. Open **Add Data → Upload**
2. Select the `VPN_logs` file
3. Keep the auto-detected **JSON** source type
4. On **Input Settings**, create or select the index `VPN_Logs`
5. After upload, open **Search & Reporting** and set the time picker to **All time**
6. If field names don't appear automatically, append `| spath` to the search — this tells Splunk to parse the JSON fields from each event

#### Useful Verification Searches

```spl
index=VPN_Logs
| stats count
```

```spl
index=VPN_Logs
| spath
| search UserName="Maleena"
| stats count
```

```spl
index=VPN_Logs
| spath
| search Source_ip="107.14.182.38"
| stats values(UserName) as UserName count
```

```spl
index=VPN_Logs
| spath
| search Source_Country!="France"
| stats count
```

```spl
index=VPN_Logs
| spath
| search Source_ip="107.3.206.58"
| stats count
```

---

### Splunk Quick Reference

| Concept | Summary |
|---|---|
| 3 core components | Forwarder → Indexer → Search Head |
| Forwarder role | Collects and ships log data from endpoints |
| Indexer role | Parses, normalizes, stores data as searchable events |
| Search Head role | Query interface using SPL; builds visualizations |
| Default app | Search & Reporting |
| Data upload flow | Select Source → Select Source Type → Input Settings → Review → Done |
| JSON parsing fix | Append `\| spath` if fields aren't auto-parsed |
| Basic search pattern | `index=<name> \| spath \| search <field>="<value>" \| stats count` |

---

## Summary

- **Splunk** is a leading SIEM built around three components — the **Forwarder** (collects and ships log data from endpoints), the **Indexer** (parses/normalizes data into searchable events), and the **Search Head** (where users query data via **SPL** and build visualizations).
- The home screen centers on the **Splunk Bar** (notifications, settings, activity, help, search), the **Apps Panel** (always includes Search & Reporting by default), and a customizable **Home Dashboard**.
- Ingesting new data follows a five-step flow: **Select Source → Select Source Type → Input Settings (index/hostname) → Review → Done**.
- A worked example uploading a JSON VPN log file shows that when fields don't auto-parse, appending **`| spath`** to a search extracts them — after which standard SPL patterns like `index=<name> | spath | search <field>="<value>" | stats count` filter and summarize the ingested events.

