# SOC Metrics & SLA


---

A SOC exists to protect the **confidentiality, integrity, and availability** of an organization's digital assets — achieved through developing, receiving, and triaging alerts. To measure how well this is working, SOC teams track a set of core metrics.

---

### The Four Core Alert Metrics

| Metric | Formula | Measures |
|---|---|---|
| **Alerts Count (AC)** | Total count of alerts received | Overall load on SOC analysts |
| **False Positive Rate (FPR)** | False Positives / Total Alerts | Level of noise in the alert stream |
| **Alert Escalation Rate (AER)** | Escalated Alerts / Total Alerts | L1 analyst experience/independence |
| **Threat Detection Rate (TDR)** | Detected Threats / Total Threats | Overall reliability of the SOC team |

#### Alerts Count

Too many unresolved alerts (e.g., 80 in a queue) risks burying real threats in noise. Too few (e.g., a week with none) can signal a **SIEM visibility gap** rather than genuine calm. A healthy general benchmark: **5–30 alerts per day per L1 analyst** (varies by company size).

#### False Positive Rate

If 94% of alerts (75 of 80) turn out to be noise, analysts become desensitized and more likely to miss real threats. **0% FPR is unachievable**, but **80%+ is a serious problem**, typically fixed through detection rule tuning — sometimes called **"False Positive Remediation."**

#### Alert Escalation Rate

Measures how often L1 escalates rather than resolves independently. Too high suggests inexperience or over-caution; the target is generally **below 50%**, ideally **below 20%**.

#### Threat Detection Rate

The **most critical** metric — every missed threat can lead to serious consequences (ransomware, data exfiltration).

> **Example:** Of 6 attacks in a year, 4 were caught, 1 was missed due to a broken detection rule, and 1 was misclassified as a False Positive by an L1 analyst → **TDR = 4/6 = 67%** — a very poor result. TDR should always be targeted at **100%**.

---

### Service Level Agreement (SLA) Timing Metrics

Catching an alert isn't enough — it must be triaged and responded to *quickly enough* to stop an attacker before they succeed. These requirements are formalized in an **SLA** — an agreement between the internal SOC and company management (or between an MSSP and its customers).

```
Attack Occurs → [MTTD] → Detected → [MTTA] → Acknowledged/Triage Starts → [MTTR] → Threat Contained
```

| Metric | Common SLA Target | Description |
|---|---|---|
| **SOC Team Availability** | 24/7 | The SOC's working schedule — often 8/5 (Mon–Fri business hours) or full 24/7 coverage |
| **Mean Time to Detect (MTTD)** | 5 minutes | Average time between the attack occurring and its detection by SOC tools |
| **Mean Time to Acknowledge (MTTA)** | 10 minutes | Average time for L1 analysts to begin triaging a new alert |
| **Mean Time to Respond (MTTR)** | 60 minutes | Average time for the SOC to actually stop the breach from spreading |

> **Note:** Exact definitions and formulas can vary between teams depending on what they choose to measure — the above reflects common conventions.

---

### Why Metrics Matter to L1 Analysts

Metrics serve two purposes: they make the SOC more efficient (reducing successful attacks), and they're often used to **evaluate individual performance** — feeding directly into career growth toward roles like L2 analyst.

---

### Improving the Metrics

| Issue | Recommendations |
|---|---|
| **False Positive Rate over 80%** | 1. Exclude trusted activities (e.g., routine system updates) from detection rules<br>2. Automate triage of common alert types using SOAR or custom scripts |
| **Mean Time to Detect over 30 min** | 1. Work with SOC engineers to speed up or increase the frequency of detection rules<br>2. Verify SIEM logs are ingested in real time, without ingestion delay |
| **Mean Time to Acknowledge over 30 min** | 1. Ensure analysts get real-time notifications for new alerts<br>2. Distribute the alert queue evenly across analysts on shift |
| **Mean Time to Respond over 4 hours** | 1. As L1, prioritize fast escalation of confirmed threats to L2<br>2. Document response procedures for different attack scenarios in advance |

---

### SOC Metrics Quick Reference

| Concept | Summary |
|---|---|
| SOC's core goal | Confidentiality, Integrity, Availability of digital assets |
| 4 core alert metrics | Alerts Count, False Positive Rate, Alert Escalation Rate, Threat Detection Rate |
| Healthy alert volume | ~5–30 alerts/day per L1 analyst |
| FPR danger threshold | 80%+ is a serious problem |
| AER target | Below 50%, ideally below 20% |
| TDR target | 100% (every miss can be devastating) |
| SLA timeline | MTTD → MTTA → MTTR |
| Common SLA targets | MTTD: 5 min, MTTA: 10 min, MTTR: 60 min |
| Why metrics matter to L1 | Drives SOC efficiency + individual performance review/career growth |

---

## Summary

- A SOC's performance is tracked through four core alert metrics: **Alerts Count** (overall load, healthy at ~5–30/day per L1 analyst), **False Positive Rate** (noise level, dangerous above 80%), **Alert Escalation Rate** (analyst independence, targeted below 50% and ideally below 20%), and **Threat Detection Rate** (overall reliability, always targeted at 100% since every miss can be devastating).
- Beyond catching alerts, an SLA formalizes how *quickly* the SOC must act, following the pipeline **Attack → MTTD (detect) → MTTA (acknowledge) → MTTR (respond/contain)** — with common targets of 5, 10, and 60 minutes respectively, alongside a defined SOC availability window (8/5 or 24/7).
- These metrics serve a dual purpose: they drive overall SOC efficiency and also directly feed into **individual performance evaluation**, making them personally relevant to an L1 analyst's career growth toward L2.
- Each metric has concrete improvement levers — tuning detection rules and automating triage for high FPR, speeding up rule execution and verifying real-time log ingestion for slow MTTD, ensuring real-time notifications and even queue distribution for slow MTTA, and prioritizing fast escalation with pre-documented response procedures for slow MTTR.

