# L2 SOC Analyst Interview Prep — One-Day Sprint

A complete, high-yield study repository built for **one day of preparation** before a **face-to-face interview with Lekhwiya management** for an **L2 SOC Analyst** role in Qatar.

This repo does not claim any non-public knowledge of any specific organisation's internal infrastructure, tools, procedures, or incidents. It is framed for a high-trust, government/national-security-adjacent SOC environment generally: confidentiality, disciplined escalation, evidence handling, accurate documentation, teamwork, and chain of command — alongside the full range of L2 technical skill a genuine interview will test. Everything here focuses on **defensive security, authorised investigation, and incident response only.**

## Table of Contents

- [One-Day Study Plan](#one-day-study-plan)
- [How to Use This Repository](#how-to-use-this-repository)
- [L2 SOC Interview Checklist](#l2-soc-interview-checklist)
- [60-Second "Tell Me About Yourself" Template](#60-second-tell-me-about-yourself-template)
- ["Why Lekhwiya / Why This Role?" Template](#why-lekhwiya--why-this-role-answer-template)
- [Final 15-Minute Pre-Interview Checklist](#final-15-minute-pre-interview-checklist)
- [Repository Contents](#repository-contents)
- [Question Count Verification](#question-count-verification)

---

## One-Day Study Plan

A realistic 6–8 hour plan, ordered by what actually earns you the most interview points per hour spent. Adjust the clock times to your own interview time, but keep the **order and proportions** — this order is deliberate: highest-frequency, highest-impact material first, closing rehearsal last.

| Time block | Duration | Focus | What to do |
|---|---|---|---|
| **Block 1** | 60 min | [L2 SOC Triage & Incident Response](questions/01-l2-soc-triage-and-incident-response.md) | This is the category most likely to open the interview. Read all 15 questions once through; speak the model answers aloud for Q1, Q3, Q5, Q9 |
| **Block 2** | 45 min | [Quick Reference: Triage Framework](quick-reference/l2-incident-triage-framework.md) + [Windows Event IDs](quick-reference/windows-event-ids.md) | Memorise the universal sequence and the containment authority boundary table. Drill Event IDs until you can recite them without looking |
| **Block 3** | 50 min | [SIEM & Detection Engineering](questions/02-siem-logging-and-detection-engineering.md) + [Windows/AD & Identity](questions/03-windows-active-directory-and-identity.md) | Focus especially on Q21 (KQL), Q31/Q32 (Kerberoasting, pass-the-hash), Q36 (DCSync) — these come up constantly |
| **Block 4** | 45 min | [Azure/M365 & Cloud Security](questions/05-azure-microsoft-365-and-cloud-security.md) + [Quick Reference: Azure/M365 Cheat Sheet](quick-reference/azure-m365-investigation-cheat-sheet.md) | MFA fatigue (Q50) and OAuth consent phishing (Q51) are high-probability questions — know them cold |
| **— Short break — 10 min** | | | |
| **Block 5** | 40 min | [Network Security & Traffic Analysis](questions/04-network-security-and-traffic-analysis.md) + [Quick Reference: Ports & Protocols](quick-reference/network-ports-and-protocols.md) | Speed-drill the ports table; read the beaconing and DNS-abuse sections twice |
| **Block 6** | 35 min | [EDR/Malware/Endpoint](questions/06-edr-malware-and-endpoint-security.md) + [Phishing & Email Security](questions/07-phishing-and-email-security.md) | Q58 (decode-first), Q63 (ransomware precursors), Q68 (credential phishing response) are must-knows |
| **Block 7** | 30 min | [Threat Hunting, MITRE & Threat Intel](questions/08-threat-hunting-mitre-and-threat-intelligence.md) + [MITRE Cheat Sheet](quick-reference/mitre-attack-cheat-sheet.md) | Be able to explain the Pyramid of Pain and the tactics list from memory |
| **Block 8** | 25 min | [Vuln Mgmt & Digital Forensics](questions/09-vulnerability-management-and-digital-forensics.md) + [Governance & Reporting](questions/10-governance-reporting-and-stakeholder-communication.md) | Order of volatility (Q82) and the executive-briefing structure (Q85) are the two highest-value items here |
| **— Short break — 10 min** | | | |
| **Block 9** | 45 min | [Scenario Playbooks](scenarios/README.md) — all 12 | Skim each flowchart, read the "say this aloud" closing line for every one. This is the fastest way to internalise the shape of a complete answer |
| **Block 10** | 40 min | [Behavioral, Management & Pressure Questions](questions/11-behavioral-management-and-pressure-questions.md) | This is the category management will ask *directly*. Rehearse Q92 (confidentiality), Q95 (integrity under pressure), Q100 (closing) out loud |
| **Block 11** | 45 min | [Mock Interview](mock-interview.md) | Run the full 45-minute simulation once, out loud, timed. Score yourself honestly |
| **Block 12** | 15 min | This README | Read the checklists and templates below one final time. Memorise your "Tell me about yourself" and "Why this role" answers word-for-shape (not word-for-word) |

**Total: ~7 hours.** If you only have 4–5 hours, keep Blocks 1, 2, 4, 9, 10, and 11 in full and compress the rest to a skim of headings and tables only.

---

## How to Use This Repository

- **Every question follows the same format** — question, difficulty, what's being tested, a spoken-style model answer, deeper explanation, key terms, a weak-answer warning, and a likely follow-up. Read the model answer *out loud*, not silently — the goal is to sound natural in the room, not to recite a script.
- **`Scenario-based`** questions (47 of the 100) go further: initial situation, ordered investigation steps, evidence sources, severity reasoning, a containment table splitting what you can do yourself from what needs approval, escalation guidance, recovery/lessons-learned, and a ready "say this aloud" closing line.
- **[`scenarios/`](scenarios/README.md)** holds 12 complete incident playbooks with Mermaid flowcharts — read these when you want the *shape* of a full answer fast, without the surrounding explanation.
- **[`quick-reference/`](quick-reference/README.md)** is pure recall — dense tables with no prose, meant for the last hour before you walk in.
- **[`mock-interview.md`](mock-interview.md)** is a timed 45-minute rehearsal with a scoring rubric — run it once you've been through the material, to find your weak spots before the real thing.
- Every file cross-links to its neighbours (Previous / Back to README / Next) so you can move through the whole repository in one continuous pass without hunting for the next file.

---

## L2 SOC Interview Checklist

Run through this the morning of the interview:

- [ ] Can explain the difference between event, alert, and incident without hesitation
- [ ] Can name the containment actions I can take alone vs what needs approval — instantly, for any scenario
- [ ] Can recite at least 15 Windows Event IDs and what each means
- [ ] Can explain Kerberoasting, pass-the-hash, and DCSync in one or two sentences each
- [ ] Can explain MFA fatigue and OAuth consent phishing, and why a password reset alone fails on the second one
- [ ] Can decode and reason about an encoded PowerShell command scenario without saying "encoded = automatically bad"
- [ ] Can state the six-part structure for briefing non-technical management on a live incident
- [ ] Can give a calm, honest answer to "tell me about a time you made a mistake"
- [ ] Can explain, without hesitation, what I'd do if asked to skip a documented procedure under pressure
- [ ] Have my 60-second "tell me about yourself" and "why this role" answers ready, not memorised word-for-word but shape-perfect
- [ ] Have at least three genuine questions ready to ask the interviewer at the end
- [ ] Know my own resume cold — no scenario in it I can't speak to in detail

---

## 60-Second "Tell Me About Yourself" Template

Fill in your own specifics in the bracketed parts. Practise it until it takes **about 60 seconds spoken aloud**, not read silently — that's a very different pace.

> "I'm a security analyst with a background in [your background — e.g., SOC monitoring, IT support transitioning into security, a specific certification path], and my focus has been on [your specific strength — e.g., SIEM-based threat detection, incident triage, Windows and identity investigation].
>
> Day to day, I work on [what you actually do or have done — validating alerts, investigating suspicious activity, correlating evidence across log sources, and escalating and documenting incidents properly]. What I've found I'm genuinely good at is [a specific strength — e.g., staying calm and methodical under pressure, or turning a messy alert into a clear, evidence-backed conclusion].
>
> What draws me to this role specifically is [a genuine, specific reason — the seriousness of the mission, the level of discipline and trust the environment demands, the chance to work at a higher tier of investigation]. I take confidentiality and process discipline seriously, and I understand that in an environment like this, being trusted with sensitive information is earned through consistent, careful, honest work — not claimed.
>
> That's roughly where I am, and I'm genuinely looking forward to talking through how I'd approach the kind of work this role involves."

**Delivery notes:**
- Say it standing up, out loud, at least three times before the interview.
- Do not read it from memory word-for-word in the room — know the *shape*, not the script, so it sounds natural rather than recited.
- Keep it to roughly 4–6 sentences spoken at a normal pace — if you're going over 75 seconds, cut detail, not structure.

---

## "Why Lekhwiya / Why This Role?" Answer Template

Fill in your genuine reasoning — the template below gives you the shape and the tone, not the content, since your real motivation should come through here more than anywhere else in the interview.

> "A few things draw me to this specifically. First, the mission itself — working in a role that protects national and public interests carries a level of seriousness and purpose I want in my career, not just a technical challenge.
>
> Second, the level of discipline this kind of environment demands — around confidentiality, evidence handling, and chain of command — is exactly the kind of rigour I want to be held to. I'd rather work somewhere that expects that discipline than somewhere that doesn't ask for it.
>
> Third, [a genuine personal reason — e.g., wanting to work at a higher tier of investigation than I currently do, wanting to be part of a team defending critical infrastructure, a specific connection to Qatar or the region]. This role is a step toward the kind of security work I want to be doing for the long term, not just the next job.
>
> And honestly, I want to be somewhere I can be trusted with real responsibility, and where getting that trust right — through consistent, careful, honest work — actually matters."

**Delivery notes:**
- Lead with something true about you, not a generic line about "passion for cybersecurity" — interviewers hear that constantly and it doesn't land.
- If you genuinely don't yet have a personal connection to Qatar or this specific organisation, that's fine — lean into the mission-and-discipline framing instead, and be honest that the opportunity itself is what draws you.
- Keep it under 45 seconds — this should feel like a confident, considered answer, not a speech.

---

## Final 15-Minute Pre-Interview Checklist

Do this immediately before you walk in — not the night before.

- [ ] **Re-read [Red Flags and Escalation Triggers](quick-reference/red-flags-and-escalation-triggers.md)** once, top to bottom — it's the fastest single page to refresh your judgement calls
- [ ] **Say your "Tell me about yourself" answer out loud once**, standing, at normal pace
- [ ] **Say your "Why this role" answer out loud once**
- [ ] **Review the containment authority table** in the [Triage Framework](quick-reference/l2-incident-triage-framework.md) — "what can I do alone vs what needs approval" should be instant
- [ ] **Silently rehearse one technical answer and one scenario answer** you feel least confident on
- [ ] **Prepare your questions for the interviewer** — have at least three ready (e.g., about the team structure, escalation process, or what a strong first 90 days looks like in this role)
- [ ] **Check practical readiness**: documents, ID, directions/timing, professional appearance
- [ ] **Breathe.** You have covered the material — the goal now is calm, clear delivery, not last-minute cramming of new content
- [ ] **Remember the one-sentence anchor for any pressure question**: *"I don't skip the process under pressure — a rushed shortcut that damages evidence or trust almost always costs more time later than the delay would have."*

---

## Repository Contents

```
l2-soc-analyst-interview-prep/
├── README.md                                    ← you are here
├── mock-interview.md                             45-minute timed simulation + scoring rubric
├── questions/                                     100 questions across 11 categories
│   ├── 01-l2-soc-triage-and-incident-response.md          (15 questions, 8 scenario-based)
│   ├── 02-siem-logging-and-detection-engineering.md       (12 questions, 4 scenario-based)
│   ├── 03-windows-active-directory-and-identity.md        (10 questions, 4 scenario-based)
│   ├── 04-network-security-and-traffic-analysis.md        (10 questions, 4 scenario-based)
│   ├── 05-azure-microsoft-365-and-cloud-security.md       (10 questions, 6 scenario-based)
│   ├── 06-edr-malware-and-endpoint-security.md             (8 questions, 4 scenario-based)
│   ├── 07-phishing-and-email-security.md                   (6 questions, 3 scenario-based)
│   ├── 08-threat-hunting-mitre-and-threat-intelligence.md  (8 questions, 3 scenario-based)
│   ├── 09-vulnerability-management-and-digital-forensics.md (5 questions, 2 scenario-based)
│   ├── 10-governance-reporting-and-stakeholder-communication.md (6 questions, 3 scenario-based)
│   └── 11-behavioral-management-and-pressure-questions.md  (10 questions, 6 scenario-based)
├── scenarios/                                     12 end-to-end incident playbooks with Mermaid flowcharts
│   ├── 01-ransomware-critical-server.md
│   ├── 02-m365-account-compromise.md
│   ├── 03-mfa-fatigue-attack.md
│   ├── 04-phishing-bec-executive.md
│   ├── 05-suspicious-powershell-execution.md
│   ├── 06-bruteforce-then-success.md
│   ├── 07-impossible-travel-signin.md
│   ├── 08-malware-privileged-workstation.md
│   ├── 09-suspected-lateral-movement.md
│   ├── 10-data-exfiltration-alert.md
│   ├── 11-insider-threat-warning.md
│   └── 12-ddos-availability-incident.md
└── quick-reference/                               7 dense, scannable cheat sheets
    ├── l2-incident-triage-framework.md
    ├── windows-event-ids.md
    ├── network-ports-and-protocols.md
    ├── mitre-attack-cheat-sheet.md
    ├── azure-m365-investigation-cheat-sheet.md
    ├── management-communication-phrases.md
    └── red-flags-and-escalation-triggers.md
```

---

## Question Count Verification

| Category | Questions | Scenario-based |
|---|---:|---:|
| 01 · L2 SOC Triage and Incident Response | 15 | 8 |
| 02 · SIEM, Logging and Detection Engineering | 12 | 4 |
| 03 · Windows, Active Directory and Identity | 10 | 4 |
| 04 · Network Security and Traffic Analysis | 10 | 4 |
| 05 · Azure, Microsoft 365 and Cloud Security | 10 | 6 |
| 06 · EDR, Malware and Endpoint Security | 8 | 4 |
| 07 · Phishing and Email Security | 6 | 3 |
| 08 · Threat Hunting, MITRE ATT&CK and Threat Intelligence | 8 | 3 |
| 09 · Vulnerability Management and Digital Forensics | 5 | 2 |
| 10 · Governance, Reporting and Stakeholder Communication | 6 | 3 |
| 11 · Behavioral, Management and Pressure Questions | 10 | 6 |
| **Total** | **100** ✅ | **47** ✅ (required: ≥40) |

Good luck. Walk in knowing the material, and let the calm, evidence-based way you talk about it do the rest.
