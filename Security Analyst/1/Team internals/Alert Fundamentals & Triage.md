
# Alert Fundamentals & Triage

--

An **alert** is a core SOC concept — how well it's handled determines whether a breach is caught in time or missed entirely. This is foundational knowledge for any SOC L1 analyst.

---

### From Events to Alerts

The pipeline from raw activity to something an analyst reviews:

```
Event occurs → System logs it → Logs shipped to SIEM/EDR → Security solution generates an Alert
```

1. An **event** occurs — a login, process launch, file download, etc.
2. The originating system (OS, firewall, cloud provider) **logs** it.
3. Logs are shipped to a security solution (**SIEM**, **EDR**).
4. The security solution flags suspicious event(s) as an **alert** — a notification generated when a specific event or sequence of events matches detection logic.

> **Why alerts matter:** A SOC can receive millions of raw logs daily. Alerts filter that down to the dozens that actually need human review — turning an impossible manual task into a manageable one.

### Alert Management Platforms

| Platform Type | Examples | Notes |
|---|---|---|
| **SIEM** | Splunk ES, Elastic SIEM | Solid alert management; a common default choice for most SOC teams |
| **EDR / NDR** | MS Defender, CrowdStrike | Have their own alert dashboards, but centralizing via SIEM/SOAR is generally preferred |
| **SOAR** | Splunk SOAR, Cortex SOAR | Used by larger teams to aggregate and centralize alerts across multiple solutions |
| **ITSM** | Jira, TheHive | Custom ticket-management setups some teams use instead |

---

### L1's Role in Alert Triage

Everyone in the SOC touches alert triage in some way:

| Role | Responsibility |
|---|---|
| **L1 Analysts** | Review alerts, separate real threats from noise, escalate confirmed threats to L2 |
| **L2 Analysts** | Receive escalations, perform deeper analysis and remediation |
| **SOC Engineers** | Ensure alerts carry enough context/data for efficient triage |
| **SOC Manager** | Tracks triage speed and quality to ensure real attacks aren't missed |

---

### Alert Properties

| # | Property | Description | Example |
|---|---|---|---|
| 1 | **Alert Time** | When the alert was created (usually a few minutes after the actual event) | Alert Time: `March 21, 15:35`; Event Time: `March 21, 15:32` |
| 2 | **Alert Name** | Summary based on the detection rule's name | *Unusual Login Location*, *Windows RDP Bruteforce* |
| 3 | **Alert Severity** | Urgency level, initially set by detection engineers (adjustable by analysts) | 🟢 Low, 🟡 Medium, 🟠 High, 🔴 Critical |
| 4 | **Alert Status** | Whether the alert is being worked or resolved | 🆕 New, 🔄 In Progress, ✅ Closed |
| 5 | **Alert Verdict** | Whether it's a real threat or noise (aka alert classification) | 🔴 True Positive, 🟢 False Positive |
| 6 | **Alert Assignee** | The analyst who owns/reviews the alert (aka alert owner) | — |
| 7 | **Alert Description** | Explains the rule's logic, why the activity may indicate an attack, and (optionally) how to triage it | — |
| 8 | **Alert Fields** | Specific data values that triggered the alert | Affected Hostname, Entered Commandline |

---

### Alert Prioritization

With many alerts in the queue, deciding **which to pick up first** is critical for timely detection. A common, generic prioritization approach:

1. **Filter** — only take **new**, unassigned, unresolved alerts; never interfere with what a teammate is already handling.
2. **Sort by severity** — Critical → High → Medium → Low, since rules are designed so higher-severity alerts are far more likely to represent real, high-impact threats.
3. **Sort by time** — oldest first. An older breach means the attacker has likely had more time to act (e.g., already exfiltrating data), while a newer one may still be in early discovery.

---

### Alert Triage Workflow

Alert review is also called *alert handling*, *alert processing*, *alert investigation*, or *alert analysis* — this doc uses **Alert Triage**. The flow generally has three stages:

#### 1. Initial Actions

- Assign the alert to yourself
- Move its status to **In Progress**
- Familiarize yourself with the alert's name, description, and key indicators

*(This prevents duplicate work and confirms you're ready to investigate.)*

#### 2. Investigation

The most complex step — applying technical knowledge to determine whether the activity is legitimate or malicious, using SIEM/EDR logs. Teams may provide **Workbooks** (aka playbooks/runbooks) with category-specific investigation steps. Without one, general guidance:

- Identify **who/what is affected** — user, hostname, cloud resource, network, website
- Note the **specific action** described — suspicious login, malware, phishing, etc.
- Review **surrounding events** — look for suspicious activity shortly before/after the alert
- Use **threat intelligence platforms** or other resources to verify hypotheses

#### 3. Final Actions

- Determine the verdict: **True Positive** (malicious) or **False Positive** (not malicious)
- Write a detailed comment explaining your analysis steps and reasoning
- Return to the dashboard and move the alert to **Closed**

*(Escalation and commenting mechanics are covered in more detail elsewhere — the core outcome here is a documented, defensible verdict.)*

---

### Alert Triage Quick Reference

| Concept | Summary |
|---|---|
| Pipeline | Event → Log → Ship to SIEM/EDR → Alert |
| Why alerts exist | Reduce millions of raw logs to a manageable review queue |
| Platform types | SIEM, EDR/NDR, SOAR, ITSM |
| L1's core job | Triage: distinguish real threats from noise, escalate to L2 |
| 8 Alert properties | Time, Name, Severity, Status, Verdict, Assignee, Description, Fields |
| Prioritization order | Filter (unassigned) → Severity (Critical first) → Time (oldest first) |
| Triage stages | Initial Actions → Investigation → Final Actions |
| Final verdicts | True Positive vs. False Positive |
| Investigation aids | Workbooks/playbooks/runbooks, threat intel platforms |

---

## Summary

- The alert pipeline runs **Event → Log → Ship to SIEM/EDR → Alert**, reducing millions of daily raw logs down to a manageable set that actually needs human review.
- Alerts are managed across four platform types — **SIEM** (Splunk ES, Elastic SIEM), **EDR/NDR** (MS Defender, CrowdStrike), **SOAR** (Splunk SOAR, Cortex SOAR) for centralizing across tools, and **ITSM** (Jira, TheHive) for custom ticketing.
- Every alert carries eight core properties: **Time, Name, Severity, Status, Verdict, Assignee, Description,** and **Fields** — with L1 owning day-to-day triage, L2 handling escalations, Engineers ensuring alert context, and the Manager tracking triage speed and quality.
- Alerts in a queue are prioritized by **filtering** to unassigned/unresolved items first, then **sorting by severity** (Critical → Low), then **sorting by time** (oldest first, since older breaches mean more time for an attacker to act).
- Triage itself follows three stages: **Initial Actions** (self-assign, mark In Progress, review the alert), **Investigation** (identify affected assets, the specific action, surrounding events, and verify via threat intel), and **Final Actions** (determine True Positive vs. False Positive, document reasoning, close the alert).

