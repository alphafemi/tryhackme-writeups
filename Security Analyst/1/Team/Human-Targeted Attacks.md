# Social Engineering & Human-Targeted Attacks


---

A SOC exists to defend against attacks — but many of the most effective attacks don't target firewalls or servers at all. They target **people**, the weakest link in cyber security. Rather than breaching a fortress's walls, it's often far easier to simply convince the gatekeeper to open the door.

---

### Why Humans Are Targeted

Humans are targeted for the **access** they can provide — to accounts, mailboxes, databases, or networks. Some attackers seek specific access; others breach as many accounts as possible and decide how to exploit them afterward.

| Attack | Attacker's Next Step |
|---|---|
| Breach an HR manager's Google account | Steal and sell the employee database |
| Trick a wealthy individual into running malware | Hijack their web banking session |
| Breach an IT administrator's VPN account | Gain access to the core corporate network |
| Trick a government worker into sharing secrets | Use the info to simplify future attacks |

---

### Social Engineering — The Common Thread

Attacks targeting humans manipulate the victim into (knowingly or unknowingly) helping the attacker — this is **social engineering**: exploiting human psychology rather than technical flaws. Effective social engineering is designed to be:

| Trait | Why It Matters |
|---|---|
| **Trustworthy** | The attacker must appear legitimate for the victim to comply |
| **Emotional** | The attack triggers urgency, fear, or curiosity to short-circuit careful thinking |

---

### Common Attack Methods

| Method | Description |
|---|---|
| **Phishing** | Deceptive emails (e.g., "Your account has been compromised — click here") lead to fake login pages that harvest credentials. The most common form of social engineering — an estimated **3.4 billion** malicious emails are sent daily. |
| **Malware Downloads** | Fake software downloads laced with malware, often driven by fake CAPTCHAs, malicious QR codes, and **SEO poisoning** to rank malicious sites highly in search results. |
| **Deepfakes** | AI-generated audio/video impersonating executives, colleagues, or partners. In one real case, a deepfake video call impersonating a company's boss tricked a finance worker into wiring **$25 million**. |
| **Impersonation** | Even without AI, simple impersonation is highly effective — e.g., attackers calling and posing as corporate IT, convincing victims to hand over account access for a fake "system repair." |
| **Other vectors** | USB drop campaigns, physical intrusion attacks, insider threats, and fake job offers all pose ongoing risk. |

---

### Defending Humans: Mitigation vs. Detection

Defense against social engineering rests on two complementary pillars:

| Pillar | Role |
|---|---|
| **Mitigation** | Prevents or reduces the chance/impact of attacks (e.g., employee training, anti-phishing tools) |
| **Detection** | Catches and investigates the attacks that slip past mitigation — the core job of the SOC analyst |

> No mitigation is perfect — eventually something gets through, which is exactly where SOC detection and investigation becomes critical.

---

### Key Mitigation Measures

| Mitigation | Description |
|---|---|
| **Anti-phishing solution** | Blocks phishing emails automatically, easing SOC workload before users even see them |
| **Antivirus / EDR** | Prevents malware from running successfully even if a user is tricked into downloading it |
| **"Trust but verify" principle** | Trains employees to detect deepfakes and independently verify suspicious "CEO" or "IT" requests |
| **Security awareness training** | Teaches phishing recognition, reinforced through simulated phishing exercises |

> As a SOC analyst, understanding these mitigations matters beyond just detection — proposing and championing them (with IT/management buy-in) reduces the volume of attacks reaching the SOC in the first place, making the whole organization more secure.

---

### Social Engineering Quick Reference

| Concept | Summary |
|---|---|
| Core tactic | Social engineering — exploiting human psychology, not technical flaws |
| Two key traits | Trustworthy, Emotional |
| Most common method | Phishing (~3.4 billion malicious emails/day) |
| Emerging threat | Deepfake video/audio impersonation |
| Low-tech but effective | Simple phone-based impersonation |
| Two defense pillars | Mitigation (prevent) + Detection (catch what slips through) |
| Key mitigations | Anti-phishing tools, AV/EDR, "trust but verify", security awareness training |
| SOC's role | Detect and investigate what mitigation misses |

---

## Summary

- Attackers target **people** rather than infrastructure because humans provide **access** — to accounts, mailboxes, databases, or networks — that can be exploited immediately or sold later.
- **Social engineering** exploits human psychology rather than technical flaws, and effective attacks are designed to appear **trustworthy** while triggering an **emotional** response (urgency, fear, curiosity) that short-circuits careful thinking.
- Common methods include **phishing** (the most prevalent, ~3.4 billion malicious emails/day), fake **malware downloads** (often boosted via SEO poisoning), **deepfake** impersonation (including a real $25 million wire-fraud case), simple phone-based **impersonation**, and other vectors like USB drops and insider threats.
- Defense rests on two pillars: **mitigation** (anti-phishing tools, AV/EDR, "trust but verify," security awareness training) to reduce attack volume and impact, and **detection** — the SOC's core role — to catch and investigate whatever slips through.

