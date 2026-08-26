# Elastic Stack (ELK) & Kibana Basics


---

**Elastic Stack (ELK)** wasn't originally built as a SIEM — it was designed to store, search, and visualize large datasets (e.g., for application performance monitoring). Its powerful search/visualization capabilities made it popular for security operations too, and many SOC teams now use it much like a traditional SIEM.

---

### The Four Core Components

```
Beats → Logstash → Elasticsearch → Kibana
```

| Component | Role |
|---|---|
| **Elasticsearch** | Full-text search and analytics engine for JSON documents; stores, analyzes, and correlates data via a RESTful API |
| **Logstash** | Data processing engine — ingests data from various sources, filters/normalizes it, and routes it to a destination (Kibana, a port, Elasticsearch, or a file) |
| **Beats** | Lightweight, host-based "data shippers" — single-purpose agents that ship specific data types to Elasticsearch (e.g., Winlogbeat for Windows event logs, Packetbeat for network traffic) |
| **Kibana** | Web-based visualization tool — analyzes, investigates, and visualizes Elasticsearch data in real time via dashboards |

#### Logstash Configuration Structure

A Logstash config file has three sections:

| Section | Purpose |
|---|---|
| **Input** | Defines the data source being ingested |
| **Filter** | Specifies normalization/filtering rules applied to the ingested logs |
| **Output** | Specifies the destination — a listening port, Kibana, Elasticsearch, or a file |

#### How the Components Work Together

1. **Beats** collect data from endpoints (e.g., Winlogbeat → Windows event logs, Packetbeat → network flows)
2. **Logstash** ingests data from beats, ports, or files, parses/normalizes it into field-value pairs, and stores it in Elasticsearch
3. **Elasticsearch** acts as the searchable database
4. **Kibana** visualizes the stored data — dashboards, time charts, infographics, etc.

> As a SOC analyst, deep expertise in each backend component isn't required — but a basic understanding helps contextualize what's happening behind Kibana's interface, which is where most day-to-day work happens.

---

### The Discover Tab

The **Discover tab** is where SOC analysts spend most of their time — showing ingested logs, a search bar, normalized fields, and filtering tools.

| Element | Function |
|---|---|
| **Logs** | Each row is a single log entry with its fields/values |
| **Fields Pane** | Left panel listing parsed fields; click any field to add/remove it from the filter |
| **Index Pattern** | Determines which Elasticsearch data (log type) is being explored — a single pattern can span multiple indices |
| **Search Bar** | Where search queries and filters are entered |
| **Time Filter** | Narrows results to a specific time range |
| **Time Interval / Timeline** | Chart showing event counts over time — useful for spotting spikes |
| **Top Bar** | Save, open, or share searches |
| **Add Filter** | Apply a filter to a specific field without typing a full query |

#### Index Patterns

Kibana requires an **index pattern** to access Elasticsearch data — it defines which data (and its field structure) is being explored. Since each log source has a different structure, a dedicated index pattern normalizes each source's logs into consistent fields (e.g., `vpn_connections` for VPN logs).

#### Fields Pane

Clicking a field shows its top 5 values and occurrence percentage. Use **+** to filter *for* a value, **-** to filter it *out*.

#### Timeline

Shows event volume over time — clicking a bar filters logs to that period; the count at top-left shows total events in the selected range. Useful for spotting anomalous spikes (e.g., an unusual log spike on a specific date).

#### Creating Tables

Rather than viewing raw logs, select specific fields to build a focused table — reducing noise and improving readability. Table layouts can be saved for reuse.

> **Tip:** Set the Kibana time picker to include the relevant period (or a wide range like "Last 15 years") before running searches, or results may appear empty.

---

### KQL — Kibana Query Language

**KQL** is the search language used in the Discover tab's search bar, supporting two search styles:

#### Free Text Search

Searches for a term across **all fields**.

| Query | Behavior |
|---|---|
| `security` | Matches any document containing this exact term, in any field |
| `United States` | Returns all logs containing this exact phrase (e.g., 2,304 hits) |
| `United` (alone) | Returns **zero** results — KQL matches whole terms, not substrings |
| `United*` | Wildcard — matches "United" plus anything following it (e.g., also matches "United Nations") |

#### Logical Operators

| Operator | Example Query | Behavior |
|---|---|---|
| **AND** | `"United States" AND "Virginia"` | Logs containing both terms |
| **OR** | `"United States" OR "England"` | Logs containing either term |
| **NOT** | `"United States" AND NOT ("Florida")` | Logs from the United States, excluding Florida |

#### Field-Based Search

Uses `Field: Value` syntax (colon-separated):

```
Source_ip : 238.163.231.224 AND UserName : Suleman
```

This returns only logs where `Source_ip` matches the given value **and** `UserName` is `Suleman`. Clicking into the search bar shows all available fields to build queries from.

---

### Visualizations in Kibana

The **Visualize** tab builds tables, pie charts, bar charts, and more from indexed data.

#### Creating a Visualization

1. From the Discover tab, click a field and select **Visualize** (or navigate to the Visualize tab directly)
2. Choose a visualization type (table, pie chart, etc.)

#### Correlating Fields

Dragging an additional field into the visualization builder creates a correlation — e.g., pairing `Source_Country` with `Source_IP` to show a **Top 5 Source Countries** pie chart, or a table cross-referencing IPs against country counts.

#### Saving a Visualization

1. Build the visualization
2. Click **Save** (top-right)
3. Add a title and description
4. Click **Save and add to library**

#### Worked Example: Failed Connection Attempts Table

**Goal:** Show users and IPs involved in failed VPN connection attempts.

1. Use the `vpn_connections` data view
2. Set the time picker to include **January 2022**
3. Filter for `action: failed` (don't exclude failed events — the table should show *only* failed attempts)
4. Use **UserName** and **Source_ip** as the table fields

---

### ELK Quick Reference

| Concept | Summary |
|---|---|
| ELK = | Elasticsearch, Logstash, Kibana (+ Beats) |
| Beats | Lightweight per-purpose data shippers (Winlogbeat, Packetbeat, etc.) |
| Logstash sections | Input, Filter, Output |
| Elasticsearch | Searchable JSON document store |
| Kibana | Front-end visualization/analysis layer |
| Discover tab | Main workspace for exploring raw logs |
| Index pattern | Maps Kibana to a specific Elasticsearch data type/structure |
| KQL search types | Free text search, Field-based search |
| Wildcard | `*` matches partial terms |
| Logical operators | AND, OR, NOT |
| Field search syntax | `field: value` |
| Visualization types | Tables, pie charts, bar charts, correlations |

---

## Summary

- The **Elastic Stack (ELK)**, originally built for general data analytics rather than security, is now widely used as a SIEM alternative, built on four components: **Beats** (lightweight per-purpose data shippers), **Logstash** (ingest/filter/output pipeline), **Elasticsearch** (searchable JSON document store), and **Kibana** (visualization front end).
- Analysts spend most of their time in Kibana's **Discover tab**, which combines raw logs, a fields pane, an **index pattern** (mapping to a specific Elasticsearch data structure), a search bar, a time filter, and a timeline for spotting event spikes.
- Searches use **KQL**, supporting **free text search** (whole-term matching, with `*` wildcards) and **field-based search** (`field: value` syntax), combinable with **AND, OR,** and **NOT** operators.
- The **Visualize** tab turns indexed fields into tables, pie charts, and bar charts, with fields correlated together (e.g., country vs. IP) and layouts saved to a library — demonstrated in a worked example building a table of failed VPN login attempts filtered to `action: failed` within a specific time window.


- [73. EDR — Endpoint Detection and Response](./73-edr-endpoint-detection-response.md)
