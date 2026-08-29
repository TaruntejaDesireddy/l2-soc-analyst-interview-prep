# Mock Interview · 45-Minute L2 SOC Analyst Simulation

[⬅ Back to README](README.md)

A structured 45-minute mock interview mixing technical, scenario, behavioral, and management-communication questions — matched to what a face-to-face interview with management is actually likely to cover. Run this out loud, either alone (speaking every answer aloud, timed) or with someone else reading the questions and scoring you.

## How to run this

1. Set a timer for 45 minutes.
2. Have someone (or a recording of yourself) ask each question below, in order, without letting you see the linked model answer first.
3. Answer **out loud**, as if the interviewer is in front of you — do not just think the answer.
3. Score yourself immediately after each answer using the rubric below, before moving to the next question.
4. After all 15, total your scores and use the "strong candidate" checklist to judge overall readiness.

## The 15 questions

Roughly 3 minutes per question, allowing for a short follow-up on several of them — that's the real interview rhythm.

| # | Question | Type | Answer key (find by number in the linked file) |
|---|---|---|---|
| 1 | Tell me about yourself. | Opening | README — "Tell me about yourself" template |
| 2 | Walk me through exactly what you do in the first ten minutes after a P1 alert lands in your queue. | Technical | [Q1, questions/01](questions/01-l2-soc-triage-and-incident-response.md) |
| 3 | It is 02:00. A ransomware note is detected on a file server, and at the same moment an executive reports a phishing email they clicked. You are the only L2 on shift. What do you do? | Scenario | [Q3, questions/01](questions/01-l2-soc-triage-and-incident-response.md) |
| 4 | Which Windows Event IDs do you rely on most, and what does each one tell you? | Technical | [Q28, questions/03](questions/03-windows-active-directory-and-identity.md) |
| 5 | At 03:00 a new user account is created and added to Domain Admins by an account that belongs to a member of IT staff. What do you do? | Scenario | [Q33, questions/03](questions/03-windows-active-directory-and-identity.md) |
| 6 | A user reports receiving dozens of MFA push notifications overnight, and this morning tapped "Approve" by accident. What do you do? | Scenario | [Q50, questions/05](questions/05-azure-microsoft-365-and-cloud-security.md) |
| 7 | EDR raises an alert for PowerShell running with an encoded command on a user's laptop. What do you do? | Scenario | [Q58, questions/06](questions/06-edr-malware-and-endpoint-security.md) |
| 8 | A user calls you directly, panicked, saying they clicked a link, it took them to a Microsoft-looking login page, and they entered their credentials. What do you do? | Scenario | [Q68, questions/07](questions/07-phishing-and-email-security.md) |
| 9 | What is the Pyramid of Pain, and how does it guide what you prioritise as an analyst? | Technical | [Q76, questions/08](questions/08-threat-hunting-mitre-and-threat-intelligence.md) |
| 10 | What metrics do you think actually reflect whether a SOC is doing a good job, versus metrics that look good but don't mean much? | Technical / Governance | [Q86, questions/10](questions/10-governance-reporting-and-stakeholder-communication.md) |
| 11 | You need to brief non-technical senior management on a critical, still-ongoing incident. What do you say, and how do you structure it? | Management Communication | [Q85, questions/10](questions/10-governance-reporting-and-stakeholder-communication.md) |
| 12 | Tell me about a time you disagreed with a senior colleague's technical decision. How did you handle it? | Behavioral | [Q91, questions/11](questions/11-behavioral-management-and-pressure-questions.md) |
| 13 | A senior officer, under time pressure during an urgent case, instructs you to skip the formal evidence documentation step "just this once" to move faster. What do you do? | Behavioral / Integrity | [Q95, questions/11](questions/11-behavioral-management-and-pressure-questions.md) |
| 14 | Why do you want to work with Lekhwiya management in this kind of environment? | Closing | README — "Why Lekhwiya / Why this role?" template |
| 15 | Why should we trust you with this level of responsibility, and what does integrity mean to you in this role? | Closing | [Q100, questions/11](questions/11-behavioral-management-and-pressure-questions.md) |

*Each linked file's questions are numbered headings (`### Q1`, `### Q3`, ...) — use your editor/GitHub's in-page search (Ctrl+F) for the exact question number rather than relying on a deep link, which can break depending on how the viewer renders heading anchors.*

## Scoring rubric

Score each answer **1–5** on each of the five dimensions immediately after giving it. Be honest — this is diagnostic, not a performance.

| Dimension | 1 (weak) | 3 (adequate) | 5 (strong) |
|---|---|---|---|
| **Technical accuracy** | Wrong or missing key facts | Correct but generic | Correct, specific, uses precise terminology |
| **Investigation logic** | No clear order, or skips steps | Reasonable order, some gaps | Clear, evidence-first sequence with nothing skipped |
| **Communication** | Rambling, jargon-heavy, or too short | Understandable but not polished | Clear, structured, sounds natural spoken aloud |
| **Judgement** | Missed the actual risk or authority boundary | Got the right general idea | Correctly distinguishes what you can do vs what needs escalation |
| **Professionalism** | Defensive, vague, or evasive | Reasonable composure | Calm, honest, owns gaps in knowledge without floundering |

**Per-question total:** out of 25. **Full mock total:** out of 375 (15 × 25).

## Score interpretation

| Total score | Reading |
|---|---|
| 300+ | Strong — you are ready; focus remaining time on your two weakest-scoring questions only |
| 225–299 | Solid — re-read the model answers for anything scoring 3 or below, then re-run just those questions |
| Below 225 | Needs more prep time — work through the full [study plan](README.md#one-day-study-plan) methodically rather than jumping around |

## Final "strong candidate" feedback checklist

Read this after finishing all 15 — it's what a good interviewer is silently checking off as you talk.

- [ ] Never said "I would investigate further" without naming the exact fields, logs, or checks
- [ ] Consistently separated what they can decide alone from what needs escalation or approval
- [ ] Used precise terminology naturally (not name-dropped, actually used correctly in context)
- [ ] Gave concise, spoken-style answers — not reading a script, not rambling
- [ ] On every scenario, led with **evidence-based** reasoning, not assumption or guesswork
- [ ] On every pressure/integrity question, chose the honest, disclosed, escalate-don't-hide path
- [ ] Distinguished confirmed facts from hypothesis explicitly, out loud, at least twice
- [ ] Showed calm under the two or three hardest follow-up questions, without getting defensive
- [ ] Closed with a specific, genuine reason for wanting this exact role — not a generic answer
- [ ] Demonstrated — through examples, not claims — that they can be trusted with sensitive, high-stakes information

If most of these are checked, you are ready. Go through [`quick-reference/`](quick-reference/README.md) once more the morning of, and you're set.

[⬅ Back to README](README.md)
