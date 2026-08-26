# Alert Reporting, Escalation & Communication


---

When an L1 analyst is uncertain how to classify an alert, or encounters a real breach needing immediate action, three concepts come into play: **reporting**, **escalation**, and **communication**.

---

### The Alert Funnel

```
L1 reviews ~100 alerts → escalates ~10 True Positives to L2 → ~1 incident requires DFIR
```

Most alerts are closed by L1 as False Positives or resolved at the L1 level. Complex or threatening ones move up the chain — which is where reporting, escalation, and communication matter.

---

### Alert Reporting

Before closing or escalating an alert — especially a **True Positive** — analysts may need to document the investigation in detail rather than leave a short comment, ensuring all relevant evidence is captured.

#### Why L1 Analysts Write Reports

| Purpose | Explanation |
|---|---|
| **Provide context for escalation** | A well-written report saves L2 significant time and helps them quickly understand what happened |
| **Save findings for the record** | Raw SIEM logs are typically retained only 3–12 months, but alerts (and their reports) are often kept indefinitely |
| **Improve investigation skills** | Writing a clear summary forces genuine understanding — "if you can't explain it simply, you don't understand it well enough" |

#### Report Format — The Five Ws

| W | Captures |
|---|---|
| **Who** | The user who logged in, ran the command, or downloaded the file |
| **What** | The exact action or event sequence performed |
| **When** | Exactly when the suspicious activity started and ended |
| **Where** | The device, IP, or website involved |
| **Why** | *(Most important)* The reasoning behind the final verdict |

---

### Alert Escalation

After forming a verdict and writing the report, decide whether to escalate to L2. General recommendations for **when to escalate**:

- The alert indicates a **major cyberattack** requiring deeper investigation or DFIR
- **Remediation actions** are needed (malware removal, host isolation, password reset)
- **Communication** with customers, partners, management, or law enforcement is required
- You simply **don't fully understand** the alert and need senior input

#### Escalation Steps

Typically as simple as reassigning the alert to the on-shift L2 and notifying them (chat or in person) — though some teams require a formal written escalation request with structured fields.

**SOC Dashboard Escalation Flow:**

1. Move the alert to **In Progress** and perform the analysis
2. Write the alert report and set your verdict (e.g., **True Positive**)
3. If escalation is warranted, assign the alert to the on-shift L2
4. L2 receives the notification and starts from your report

Once L2 has the ticket, they'll typically:

- Read your report and follow up with questions if needed
- Research the alert further and validate the True Positive verdict
- Communicate with other departments as needed
- For major incidents, initiate a formal **Incident Response** process

---

### Requesting L2 Support

It's entirely appropriate for L1 to ask for senior support when something is unclear — especially early in a role, it's always better to clarify than to blindly close an alert you don't fully understand. General flow: L1 asks L2 for help with the investigation; L2 accepts and provides a knowledge-sharing session.

---

### Communication

Beyond escalation, analysts often need to communicate with **other departments** during or after analysis — e.g., confirming with IT that admin privileges were intentionally granted, or checking with HR about a newly hired employee's status.

---

### Handling Difficult Communication Scenarios

Ideally, a SOC has formal **Crisis Communication** procedures. Absent that, general guidance for common scenarios:

| Scenario | Recommended Response |
|---|---|
| **Urgent/critical alert, L2 unresponsive for 30+ minutes** | Escalate up the chain — try L2, then L3, then your manager. Know your emergency contacts in advance. |
| **Suspected account compromise (e.g., Slack/Teams) requiring user validation** | Never validate through the potentially-compromised channel — use an alternative contact method (e.g., phone call). |
| **Overwhelming volume of alerts, some critical, arriving at once** | Prioritize per the standard workflow, but proactively inform your on-shift L2 of the situation. |
| **Realizing days later that an alert was misclassified and something malicious was missed** | Immediately notify L2 — threat actors can remain dormant for weeks before causing visible impact. |
| **Can't complete triage due to unparsed/unsearchable SIEM logs** | Don't skip the alert — investigate what's possible and report the tooling issue to L2 or a SOC engineer. |

#### Communication by L2

When L1 escalates something serious (e.g., a data leak alert), L2 may go on to initiate a full DFIR process and loop in Legal and PR departments — illustrating how escalation cascades into broader organizational response.

---

### Alert Reporting & Escalation Quick Reference

| Concept | Summary |
|---|---|
| The funnel | ~100 alerts (L1) → ~10 True Positives (L2) → ~1 DFIR incident |
| 3 core concepts | Reporting, Escalation, Communication |
| Report framework | The Five Ws (Who, What, When, Where, Why) |
| Why report | Context for L2, permanent record, sharpens analyst skill |
| Escalate when | Major attack, remediation needed, cross-team communication needed, or you're unsure |
| Escalation flow | In Progress → Report + Verdict → Assign to L2 |
| It's OK to ask for help | Especially early on — clarify rather than guess |
| Crisis scenarios | Chain-of-contact fallback, alternate comms channels, proactive status updates, immediate re-escalation on misclassification, report tooling issues rather than skipping |

---

## Summary

- Most alerts never leave L1 — the funnel typically narrows from roughly **100 alerts reviewed → 10 True Positives escalated to L2 → 1 incident requiring DFIR**.
- A good alert report follows the **Five Ws** (Who, What, When, Where, and — most importantly — Why) and matters for three reasons: it gives L2 fast context, it becomes the permanent record once raw SIEM logs age out (typically 3–12 months), and writing it sharpens the analyst's own understanding.
- Escalate to L2 when the alert signals a **major attack**, requires **remediation**, needs **cross-department communication**, or when the analyst simply doesn't fully understand it — the flow is: mark In Progress → write the report and verdict → assign to the on-shift L2.
- It's always appropriate for L1 to **ask L2 for help** rather than guess, and analysts regularly need to **communicate with other departments** (IT, HR) to validate context during an investigation.
- Difficult real-world scenarios have specific guidance: escalate up the chain if L2 is unresponsive, never validate a suspected compromise through the compromised channel itself, proactively flag alert overload to L2, immediately notify L2 of any missed/misclassified alert, and report tooling failures rather than skipping an alert.

