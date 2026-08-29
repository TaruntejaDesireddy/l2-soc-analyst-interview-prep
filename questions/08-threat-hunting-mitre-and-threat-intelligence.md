# 08 · Threat Hunting, MITRE ATT&CK and Threat Intelligence

**8 questions · Q72–Q79 · 3 scenario-based**

[⬅ Previous: Phishing & Email](07-phishing-and-email-security.md) · [Back to README](../README.md) · [Next: Vuln Mgmt & Forensics ➡](09-vulnerability-management-and-digital-forensics.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q72 | What threat hunting is and how it differs from monitoring | Core | Standard |
| Q73 | MITRE ATT&CK — structure and practical use | Core | Standard |
| Q74 | IOC-based vs behavioural detection | Advanced | Standard |
| Q75 | Building a hunt hypothesis from a threat intel report | Advanced | Scenario-based |
| Q76 | Pyramid of Pain | Advanced | Standard |
| Q77 | Hunting for a technique with no existing alert | Advanced | Scenario-based |
| Q78 | Evaluating a threat intelligence feed's quality | Core | Standard |
| Q79 | Hunt turns up a real but very quiet intrusion | Advanced | Scenario-based |

---

### Q72. What is threat hunting, and how is it different from normal monitoring?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand hunting as proactive and hypothesis-driven, not just "looking at logs more."

**Model answer (say this aloud):**
> Monitoring is reactive — I wait for a detection rule to fire and then investigate it. Threat hunting is proactive — I start from a hypothesis about how an attacker might already be present without triggering any existing alert, and I go looking for evidence of that specifically. The reason hunting matters is that it assumes detection gaps exist, which is realistic, and it is how those gaps actually get found and closed before an attacker exploits them for months. A hunt always starts with a specific, testable question — not "let me look around" — and it ends with either evidence of compromise, or a documented negative result, or a new detection rule if the hunt reveals a gap worth closing permanently.

**Deeper explanation:**
A hunt hypothesis is typically sourced from: a new adversary technique published in threat intelligence, a gap identified during detection coverage assessment, an unusual pattern noticed during routine work that was never chased down, or a specific concern about the organisation's exposure. The hunting cycle: hypothesis → data source identification → query and analysis → findings (compromise found, or confirmed absence, or detection gap identified) → if a gap is found, convert it into a standing detection so the next occurrence is caught automatically rather than requiring another manual hunt. That conversion step — hunt finding becomes permanent detection — is what separates a mature hunting program from one-off exercises that never compound.

**Key terms to mention:** hypothesis-driven, proactive versus reactive, assumed breach mindset, negative result as a valid outcome, converting hunt findings into standing detections.

**Weak answer to avoid:** "Threat hunting is looking through logs for bad stuff." No hypothesis, no structure — this misses the entire point of the discipline.

**Likely follow-up:** "Give me an example of a hunt hypothesis you would write, in one sentence."

---

### Q73. Explain the structure of MITRE ATT&CK and how you actually use it in daily SOC work.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Practical application, not just naming the framework.

**Model answer (say this aloud):**
> ATT&CK is organised into tactics, which are the attacker's objectives like initial access or privilege escalation, and under each tactic, techniques and sub-techniques, which are the specific ways to achieve that objective, like Kerberoasting under credential access. I use it in three practical ways. First, when I investigate an incident, I map each stage to a technique, which gives me a common language to describe what happened and helps me predict what the attacker might do next based on typical technique chains. Second, when I write or review detections, I map them to techniques so I can see our coverage and our gaps visually rather than by feel. Third, when I read a threat intelligence report about a specific actor, ATT&CK gives me their known technique list, which I can turn directly into hunt hypotheses or detection checks.

**Deeper explanation:**
Structure: **Tactics** (the why — 14 categories from Reconnaissance through Impact), **Techniques** and **Sub-techniques** (the how, and specific variants), and for each technique, ATT&CK documents known **procedures** (real-world examples by threat groups), **detection guidance**, and **mitigations**. Practical L2 uses: incident timelines annotated with technique IDs for consistent reporting; a coverage matrix (a heatmap of which techniques have detections, partial detections, or none) used in management reporting on detection maturity; and technique-driven hunting, where a specific actor's known technique list from a threat report becomes a checklist of hunts to run. It also standardises communication — saying "T1003 credential dumping" is unambiguous in a way "they stole passwords somehow" is not.

**Key terms to mention:** tactics versus techniques versus sub-techniques, procedures, coverage heatmap, technique-driven hunting, standardised incident language.

**Weak answer to avoid:** "It's a list of attacker techniques." True but says nothing about how you actually use it operationally, which is what the question is really asking.

**Likely follow-up:** "Name three tactics in order that a typical ransomware intrusion would move through."

---

### Q74. What is the difference between IOC-based detection and behavioural detection, and why does that distinction matter?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Understanding of detection durability — a key theme across SIEM, endpoint, and hunting topics.

**Model answer (say this aloud):**
> An indicator of compromise is a specific artefact — a file hash, an IP address, a domain — tied to one particular instance of an attack. Behavioural detection looks at the technique or pattern of activity instead, regardless of which specific tool or infrastructure implements it. IOCs are fast to deploy and very precise, but they expire quickly, because an attacker changes a hash or rotates infrastructure in minutes. Behavioural detections are much more durable, because they target something the attacker cannot easily change without changing their entire approach — like the fact that Kerberoasting must request a service ticket, regardless of what tool does it. In practice I use both: IOCs for fast, tactical blocking of a known active threat, and behavioural detection for lasting coverage against the technique itself.

**Deeper explanation:**
This distinction is best explained through the **Pyramid of Pain** concept — hash values and IP addresses are trivial for an attacker to change, giving you very low "pain" imposed on them when you block them; TTPs (tactics, techniques and procedures) sit at the top, and detecting those forces an attacker to fundamentally change their entire methodology, which is expensive and slow for them. Practical guidance: use IOCs immediately when a specific active threat is identified — they are cheap and fast — but always ask "what technique produced this IOC" and build the durable, behavioural detection alongside it, so the next variant using different infrastructure is still caught.

**Key terms to mention:** IOC versus TTP, Pyramid of Pain, durability of detection, tactical versus strategic response, technique-level coverage.

**Weak answer to avoid:** "IOCs are indicators and behavioural is behaviour." Restating the terms without explaining durability or the Pyramid of Pain misses the substance of the question.

**Likely follow-up:** "You just got 200 new malicious IPs from a threat feed. Is blocking all of them a good use of your time?"

---

### Q75. `Scenario-based` A threat intelligence report describes a group actively targeting organisations in your sector, using a specific PowerShell-based loader and a known set of C2 domains. How do you turn this into action?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Turning intelligence into concrete, prioritised SOC action rather than just filing the report away.

**Model answer (say this aloud):**
> I extract two different things from a report like this: the specific indicators, which I act on immediately, and the technique and behaviour pattern, which I turn into a durable hunt and detection. For the indicators — the C2 domains — I check historical logs for any past contact with them, and I add them for blocking going forward, since that closes the door fast. For the technique, I read exactly how the PowerShell loader behaves — what it downloads, how it persists, what it looks like on the wire — and I write a hunt query for that behaviour rather than relying only on the domains, because the domains will be retired by the group soon after this report becomes public. Given it names our sector specifically, I also brief management on our exposure and prioritise this over routine work.

**Scenario walkthrough**

**Initial alert or situation**
A credible threat intelligence report names a group actively targeting your sector, with specific indicators (C2 domains) and a described technique (a PowerShell-based loader).

**Investigation steps, in order**
1. Extract all concrete indicators from the report — domains, hashes, file names — and check historical DNS, proxy, and firewall logs for any past contact, going back as far as retention allows.
2. Add the indicators to blocking and alerting per standing procedure so any future contact is caught immediately.
3. Read the technical description of the loader's behaviour in detail: delivery method, persistence mechanism, command-and-control pattern, and any distinctive command-line or network characteristics.
4. Map the described behaviour to ATT&CK techniques and check current detection coverage against each one.
5. Write and run a hunt query for the behavioural pattern — not just the named domains — across the environment's full retention window.
6. Check whether the report includes any sector-specific delivery method, such as a particular lure or targeted vector, and check whether your organisation has already been targeted.
7. Document findings regardless of outcome — confirmed hit, or confirmed clean — since a documented negative result still has value.

**Evidence and log sources to review**
Historical DNS, proxy and firewall logs for indicator matches, EDR process and script-block logs for the loader's described behaviour, email gateway logs if a specific delivery lure is described, and the current detection rule inventory for coverage assessment.

**Severity and business impact reasoning**
Treat as a proactive, elevated-priority task rather than an incident unless a match is found — in which case it immediately becomes a live incident at a severity matching the group's known objectives, which the report itself should describe.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Add indicators to blocking per standing threat-intel procedure | Broader policy changes based on the report |
| Run historical and forward-looking hunts | Declaring an incident if no evidence is found — none needed |
| Document coverage gaps found during the hunt | Prioritising this over other active incidents without shift lead sign-off |

**Escalation and communication**
Brief the shift lead and, given the sector-specific targeting, recommend this be raised to management as a proactive posture update — not alarming, but factual: this group is actively targeting our sector, here is what we checked, here is our current exposure and coverage status.

**Recovery, lessons learned, detection improvement**
If a match is found, it becomes a full incident with its own lifecycle. If not, the outcome is still valuable: a documented negative result, new or improved detections for the loader's behaviour, and indicators added preventively. Lessons learned should track how quickly the SOC can turn a report into action — that turnaround time is itself a maturity metric.

**Say this aloud to the interviewer:**
> "I would split this into two tracks — fast indicator blocking and historical lookback for the domains, and a proper behavioural hunt for the loader itself, because domains get retired fast but the technique persists. Given it names our sector, I would also brief management on exposure, and either way I would document the outcome and turn any gap I find into a lasting detection."

**Key terms to mention:** indicator lookback, behavioural hunt beyond named IOCs, ATT&CK mapping, sector-specific targeting, documented negative result, coverage gap closure.

**Weak answer to avoid:** "I would block the C2 domains." That is necessary but far from sufficient — it misses the durable behavioural response and the historical lookback that answers "were we already hit."

**Likely follow-up:** "The report is six weeks old by the time you read it. Does that change how you prioritise the domain blocking versus the behavioural hunt?"

---

### Q76. What is the Pyramid of Pain, and how does it guide what you prioritise as an analyst?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** A specific, well-known threat intelligence model — frequently asked to test conceptual depth.

**Model answer (say this aloud):**
> The Pyramid of Pain ranks types of indicators by how much difficulty it causes an attacker when you detect and block them. At the bottom are hash values, which are trivial to change — recompiling or repacking a file produces a new hash instantly, so blocking a hash causes almost no pain. Above that, IP addresses and domains are a bit more costly for the attacker to rotate. Higher up, network and host artefacts — specific file paths, registry keys, mutex names — take more effort to change because they are tied to how the tool actually works. Near the top, tools themselves are expensive to replace entirely. At the very top are TTPs — tactics, techniques and procedures — and detecting those forces the attacker to change their fundamental methodology, which is the most expensive and disruptive thing you can make them do. As an analyst, this tells me to treat IOC blocking as useful but temporary, and to invest my real effort in detections that sit as high up the pyramid as possible.

**Deeper explanation:**
The full pyramid, bottom to top: hash values (trivial to change) → IP addresses (easy to change) → domain names (a bit harder — requires new registration) → network/host artefacts (moderate effort — specific to how the tool is built) → tools (significant effort — replacing an entire toolset) → TTPs (maximum effort — requires rethinking the whole approach). The practical takeaway for prioritisation: a SOC that only ever blocks hashes and IPs is playing an endless, low-value game the attacker always wins cheaply; a SOC that detects TTPs makes every future variant of that attack harder to execute, not just the one instance observed. This directly connects to the IOC-versus-behavioural detection discussion — TTP-level detection is the behavioural end of that spectrum.

**Key terms to mention:** hash values, IP/domain rotation cost, host and network artefacts, tools, TTPs, cost imposed on the attacker, durable versus disposable detection.

**Weak answer to avoid:** Naming the levels with no explanation of *why* the ordering matters — the pyramid's value is entirely in the "cost to the attacker" reasoning, not the list itself.

**Likely follow-up:** "Where would a specific mutex name used by a piece of malware sit on the pyramid, and why?"

---

### Q77. `Scenario-based` You want to hunt for a specific ATT&CK technique that has never had a detection rule written for it in your environment. Walk me through how you would run that hunt.

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you can actually execute a hunt end to end, not just describe the concept abstractly.

**Model answer (say this aloud):**
> I would use unmanaged PowerShell execution outside normal administrative tooling as the example. First I define exactly what I am looking for and why — the hypothesis is that an attacker may be executing PowerShell for reconnaissance or lateral movement without going through our approved management tools. Then I check what data I actually have to test that — process creation logs, script block logging, network telemetry. I write a query that surfaces PowerShell execution and then filter out everything explained by known legitimate administrative sources, which usually takes several rounds of refinement because the first pass returns a lot of noise. What survives that filtering, I investigate individually. At the end I document the entire hunt — hypothesis, data used, query, findings — regardless of outcome, and if anything looks like a genuine gap, I write it up as a new detection rule.

**Scenario walkthrough**

**Initial alert or situation**
No prior detection exists for a chosen ATT&CK technique; the analyst decides to proactively hunt for it.

**Investigation steps, in order**
1. Write the specific, testable hypothesis: what behaviour, on what asset types, over what time window.
2. Identify and confirm the required telemetry exists and is populated — there is no point hunting for something you cannot actually see.
3. Write a broad initial query for the raw behaviour, expecting a high volume of legitimate results at first.
4. Iteratively exclude known-legitimate sources — approved tools, service accounts, scheduled automation — narrowing the result set round by round.
5. Investigate whatever remains individually, applying the same evidence-based triage used for any alert.
6. Cross-reference survivors against other data sources for corroboration — if something looks odd in process logs, check whether network or file activity around the same time and host supports or contradicts the concern.
7. Document the full hunt regardless of outcome: hypothesis, queries used, exclusions applied, findings, and time spent.

**Evidence and log sources to review**
Process creation and script-block logs (or the equivalent for whichever technique is being hunted), the approved administrative tooling inventory to build exclusions, network telemetry for corroboration, and historical baseline data to judge what is normal.

**Severity and business impact reasoning**
Not applicable in the traditional sense unless the hunt finds something — the value here is proactive risk reduction and detection maturity, and any finding is triaged with the same severity logic as any other investigation.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Run read-only hunting queries across available data | Any containment action, if the hunt surfaces a genuine finding — follows normal incident procedure at that point |
| Document the hunt and its results | Deploying a new detection rule to production without review |
| Draft a proposed detection rule from the findings | |

**Escalation and communication**
No escalation needed unless the hunt finds evidence of compromise, at which point it converts immediately into standard incident handling. Share the completed hunt write-up with the team regardless of outcome, since a documented negative result still tells the team something useful about current exposure.

**Recovery, lessons learned, detection improvement**
The direct output of a successful hunting program is new or improved detections — that is the entire point. Track how many hunts convert into a standing rule over time as a simple maturity metric for the hunting program itself.

**Say this aloud to the interviewer:**
> "I would write a specific, testable hypothesis, confirm the data exists to test it, then run a broad query and narrow it round by round by excluding known-legitimate sources. Whatever survives gets investigated individually, and I document the whole hunt — even a clean result — and turn any real gap into a permanent detection rule."

**Key terms to mention:** testable hypothesis, telemetry validation, iterative exclusion, corroboration across sources, documented negative result, hunt-to-detection conversion.

**Weak answer to avoid:** "I would search the logs for that technique and see what comes up." No hypothesis structure, no exclusion process, no plan for what happens with the findings.

**Likely follow-up:** "Your first query returns 50,000 results. What is your very next move?"

---

### Q78. How do you judge whether a threat intelligence feed or source is actually worth using?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Critical evaluation skill — not every feed is equally useful, and treating them all the same causes alert fatigue.

**Model answer (say this aloud):**
> I judge a feed on relevance, accuracy, timeliness, and context, in that order. Relevance means it actually applies to threats targeting our sector, our region, or our technology stack — a feed full of indicators for point-of-sale malware is not useful to an organisation with no retail systems. Accuracy means I check its false positive rate over time, because a feed that floods me with stale or generic indicators wastes triage time and trains the team to ignore it. Timeliness matters because a domain-based IOC that is even a few days late is often already retired by the attacker. And context matters most of all — a feed that just gives me a bare IP with no description of what it relates to is far less useful than one that tells me the actor, the campaign, the technique, and the confidence level, because that context is what lets me actually act on it intelligently.

**Deeper explanation:**
Practical evaluation criteria: **relevance** to sector/geography/technology; **accuracy**, tracked by measuring the feed's own false positive rate once ingested; **timeliness**, since IOC value decays fast; **confidence and source reliability**, ideally using a structured rating like the Admiralty source-reliability and information-credibility scale; **context depth** — actor attribution, campaign, technique, and recommended action, not a bare indicator; and **actionability** — whether the feed integrates cleanly into existing tooling or requires manual work that will not scale. A mature SOC actively curates its feeds rather than ingesting everything available, and periodically re-evaluates each source's actual contribution — how many real detections it has driven versus how much noise.

**Key terms to mention:** relevance to sector/geography, false positive rate tracking, IOC decay/timeliness, source reliability rating, context and actionability, feed curation and re-evaluation.

**Weak answer to avoid:** "More feeds means more coverage." Uncurated feed sprawl is a leading cause of alert fatigue and wasted analyst time.

**Likely follow-up:** "A feed has a 40% false positive rate on your environment after three months. What do you do with it?"

---

### Q79. `Scenario-based` During a routine hunt you find a service account authenticating to a handful of servers every few days, always at 04:00, doing nothing unusual each time — but this account was never known to anyone on the team. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Recognising a quiet, low-and-slow intrusion pattern that deliberately avoids triggering normal alerts — the hardest and most valuable kind of hunting finding.

**Model answer (say this aloud):**
> This is exactly the pattern a patient attacker wants to create — low frequency, consistent timing, nothing individually alarming, and specifically designed to sit under any threshold-based detection. The fact that nobody on the team recognises the account is the single most important detail, so my first move is simply to find out whether it is genuinely unknown or just unfamiliar to me — I check the account's creation record, its owner, and whether it is documented anywhere. If it turns out to be genuinely undocumented, I treat this as a probable long-term compromise and escalate, because low-and-slow persistence like this is usually a sign of a patient, capable actor, not noise.

**Scenario walkthrough**

**Initial alert or situation**
A hunt surfaces a previously unknown service account authenticating on a regular but infrequent schedule to several servers, with no individually alarming activity.

**Investigation steps, in order**
1. Check the account's creation record — who created it, when, and whether any change or provisioning ticket documents it.
2. Check whether the account is documented anywhere in asset or service inventories, and ask relevant teams directly whether they recognise it, rather than assuming it is malicious immediately.
3. If genuinely unrecognised, build a full history of everything the account has done since creation — every authentication, every host touched, every action taken during each session.
4. Look closely at what "nothing unusual" actually means on each server — check for subtle actions like file access, small file transfers, or reconnaissance commands that a threshold-based detection would not have flagged individually.
5. Check how the account authenticates — password, certificate, or key — and whether its credential material could be traced to an initial compromise elsewhere.
6. Check what privileges the account holds and what it is capable of reaching, even if it has not exercised that full capability yet.
7. Check the regularity itself — a consistent 04:00 schedule over weeks or months is a strong indicator of automated, deliberate attacker tooling rather than a coincidence.

**Evidence and log sources to review**
Account creation and provisioning records, full authentication history for the account across all systems, file and process activity on each accessed server during every session, privilege and group membership for the account, and any available data on how or when the account's credentials were first used.

**Severity and business impact reasoning**
Critical once confirmed unauthorised. Long-duration, low-frequency access specifically designed to avoid detection thresholds is a hallmark of a sophisticated actor with a long-term objective, and the extended dwell time means the true scope may be considerably larger than what one hunt has surfaced so far.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Continue investigating quietly to build full scope before acting | Disabling the account — premature action can tip off a patient attacker and cause them to accelerate or hide deeper |
| Preserve all evidence found so far | Any containment action — this needs a coordinated, carefully timed response, not an immediate reflexive block |
| Escalate with full findings to the incident manager | Public or wide internal communication before containment is ready |

**Escalation and communication**
Escalate immediately once the account is confirmed genuinely unrecognised, even before the investigation is complete, because a suspected long-term intrusion needs incident manager involvement to plan a coordinated response rather than a single analyst quietly working it alone. Be explicit that premature action risks alerting a patient attacker.

**Recovery, lessons learned, detection improvement**
Recovery for a confirmed long-term intrusion is a major, carefully planned operation — full scope determination across every system the account or related indicators touched, coordinated simultaneous containment, credential rotation, and thorough eradication, typically involving more senior responders than a single L2 investigation. Lessons learned: how did this account exist undetected for so long, and what threshold-based detections did it successfully evade by design. Detection improvement: build rare/first-seen-account alerting independent of volume thresholds, and periodically hunt specifically for low-frequency, high-regularity authentication patterns, since this exact profile is what threshold-based detection structurally cannot catch.

**Say this aloud to the interviewer:**
> "This pattern is deliberately built to sit under normal thresholds, so the fact that nobody recognises the account is the key detail. I would confirm it's genuinely undocumented, then build its full history quietly before doing anything visible, because acting too fast on a patient, low-and-slow intrusion can tip the attacker off. I would escalate to the incident manager as a suspected long-term compromise rather than handling it alone."

**Key terms to mention:** low-and-slow persistence, dwell time, threshold evasion by design, quiet scoping before action, coordinated containment, rare/first-seen account detection.

**Weak answer to avoid:** "I would disable the account immediately since it's unauthorised." Acting before scope is understood can alert a sophisticated, patient attacker and cause them to destroy evidence or dig in deeper elsewhere.

**Likely follow-up:** "How would you scope this without alerting the attacker while you investigate?"

---

[⬅ Previous: Phishing & Email](07-phishing-and-email-security.md) · [Back to README](../README.md) · [Next: Vuln Mgmt & Forensics ➡](09-vulnerability-management-and-digital-forensics.md)
