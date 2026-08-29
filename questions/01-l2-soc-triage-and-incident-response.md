# 01 · L2 SOC Triage and Incident Response

**15 questions · Q1–Q15 · 8 scenario-based**

[⬅ Back to README](../README.md) · [Next: SIEM & Detection ➡](02-siem-logging-and-detection-engineering.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q1 | First 10 minutes on a P1 alert | Core | Standard |
| Q2 | Validating true positive vs false positive | Core | Standard |
| Q3 | Two critical alerts at once — prioritise | Advanced | Scenario-based |
| Q4 | Event vs alert vs incident | Core | Standard |
| Q5 | The incident response lifecycle you follow | Core | Standard |
| Q6 | Business owner demands an isolated host back | Advanced | Scenario-based |
| Q7 | What L2 does that L1 does not | Core | Standard |
| Q8 | Thin escalation from L1 | Core | Scenario-based |
| Q9 | Reducing false positives without losing coverage | Advanced | Standard |
| Q10 | Three low alerts that are really one intrusion | Advanced | Scenario-based |
| Q11 | Beaconing host — block now or observe? | Advanced | Scenario-based |
| Q12 | The incident is bigger than the ticket | Advanced | Scenario-based |
| Q13 | Documentation that survives an audit | Core | Standard |
| Q14 | Handover in the middle of a live incident | Core | Scenario-based |
| Q15 | You made a containment mistake | Advanced | Scenario-based |

---

### Q1. Walk me through exactly what you do in the first ten minutes after a P1 alert lands in your queue.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you have a repeatable method under time pressure, or you improvise.

**Model answer (say this aloud):**
> First I acknowledge and take ownership of the alert so no one duplicates my work. Then I read the alert properly: detection rule, timestamp in UTC, source host, user account, and severity. I confirm the assets involved — is this a domain controller, a server, or a normal user laptop — because that decides the business impact. Next I validate: I pull the raw logs behind the alert instead of trusting the alert summary, and I check whether the same rule has fired before and was closed as a false positive. In parallel I build a short timeline of what happened just before and just after. By the ten minute mark I want three answers: is it real, how far has it spread, and does it need immediate containment or escalation.

**Deeper explanation:**
The first ten minutes are about **scoping**, not solving. A disciplined sequence is: acknowledge → read the detection logic → identify asset criticality → validate against raw telemetry → check historical disposition → build a mini timeline → decide contain/escalate/monitor. Pulling raw telemetry matters because alert summaries are normalised and often lose the field that proves or kills the case — the parent process, the full command line, the source IP's ASN. Checking historical disposition of the same rule tells you in seconds whether this is a known noisy detection. Asset criticality changes everything: identical malware on a kiosk and on a domain controller are not the same incident.

**Key terms to mention:** acknowledge and own, alert triage, raw telemetry vs alert summary, asset criticality, UTC timestamps, timeline, scope before response.

**Weak answer to avoid:** "I would open the alert and investigate it." That says nothing. Never describe investigation without naming the specific fields and log sources you open first.

**Likely follow-up:** "What if the raw logs for that host aren't in the SIEM?"

---

### Q2. How do you decide whether an alert is a true positive or a false positive?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Evidence-based reasoning rather than gut feeling.

**Model answer (say this aloud):**
> I test the alert against evidence rather than against my expectations. I ask what the detection rule actually looks for, then I check whether the observed activity genuinely matches that behaviour or only matches it superficially. I look for a legitimate explanation first: is there a change ticket, a scheduled job, a known admin tool, a patch window, a vulnerability scanner. Then I look for corroboration from a second source — if the SIEM says a suspicious process ran, the EDR should show the same process, its parent, and its command line. If the two sources agree and there is no legitimate business explanation, I call it a true positive. If I cannot prove either way, I do not silently close it. I document what is unresolved and escalate it as inconclusive.

**Deeper explanation:**
False positive determination is a **proof obligation**, not a default. Practical checks: (1) does an authorised change record, maintenance window, or scanner schedule explain it; (2) is the source account a known service or admin account performing its normal function; (3) does independent telemetry corroborate — EDR process tree, proxy logs, firewall flow records; (4) is the destination infrastructure benign by reputation *and* by behaviour. A key distinction is **false positive** (the activity did not happen as described, or is benign by design) versus **benign true positive** (the activity really happened but was authorised — a pentest, an admin, a scanner). The second must be documented as authorised activity, not closed as "FP", because that record is what protects you later. Closing an alert with no written reason is the single most damaging habit in a SOC.

**Key terms to mention:** corroborating telemetry, benign true positive, authorised activity, change record, detection logic, documented disposition.

**Weak answer to avoid:** "It looked like normal traffic so I closed it." Never close on appearance alone, and never close without a written reason.

**Likely follow-up:** "How would you record a benign true positive so the next analyst benefits?"

---

### Q3. `Scenario-based` It is 02:00. A ransomware note is detected on a file server, and at the same moment an executive reports a phishing email they clicked. You are the only L2 on shift. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Prioritisation under pressure, and whether you can run two workstreams without dropping one.

**Model answer (say this aloud):**
> Both are real, but they are not equal. The ransomware note on a file server is active destruction affecting shared data, so it takes priority and gets my hands-on attention immediately. The executive phishing click is high risk but is at an earlier stage, so I contain it fast with actions that take seconds — revoke the sessions, force a password reset, and have the mailbox and endpoint checked — and I hand the follow-up investigation to L1 with clear instructions while I stay on the ransomware. I notify the shift lead within the first few minutes that I am running two incidents simultaneously and that I need support, because escalating early is not an admission of failure, it is how the team keeps control. I keep two separate tickets so evidence never gets mixed.

**Scenario walkthrough**

**Initial alert or situation**
File server reports a ransom note file and mass file modification; separately, an executive self-reports clicking a link and entering credentials.

**Investigation steps, in order**
1. Confirm the ransomware is live: check for ongoing file-modification volume, the ransom note contents, and encrypted file extensions on the share.
2. Identify the account and process performing the encryption on the file server — process name, parent process, full command line, and the user context.
3. Determine whether the encryption is running locally on the server or over SMB from another host; check share access events to find the source host.
4. In parallel, for the executive: revoke active sessions, force password reset, and check sign-in logs for a successful authentication from an unfamiliar IP after the click.
5. Check the executive's mailbox for newly created inbox rules or forwarding — this is the fastest sign of a real compromise.
6. Return to the ransomware: identify patient zero and whether backups or shadow copies have been deleted.

**Evidence and log sources to review**
File server security logs (file share and object access events), EDR process telemetry and its file-modification records, backup system logs, SMB session records, identity sign-in logs for both accounts, mailbox audit logs for the executive, email gateway records for the original phishing message and other recipients.

**Severity and business impact reasoning**
Ransomware on a shared file server is Critical: it destroys availability and integrity of data used by many people, it spreads over SMB, and every minute of delay is more encrypted files. The phishing click is High: it risks confidentiality and account takeover, but at this instant it is a single identity and containment is a few clicks. Priority is set by active damage rate and blast radius, not by who reported it.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the infected server at the EDR level per standing procedure | Powering off the server or pulling it from the network physically |
| Revoke the executive's sessions and force password reset | Disabling the executive's account entirely |
| Block the malicious sender and URL at the mail gateway | Shutting down the file share for all users |
| Preserve volatile evidence and take EDR snapshots | Restoring from backup or rebuilding the server |
| Block known malicious IPs and domains from the pre-approved list | New firewall rules outside pre-approved scope |

**Escalation and communication**
Notify the shift lead and the incident manager immediately with a two-line factual summary: what is happening, which assets, what I have contained, what I need. Ransomware on shared infrastructure meets the criteria for waking senior management. I do not speculate about the ransomware family or the entry point until I have evidence.

**Recovery, lessons learned, detection improvement**
Recovery is owned by IT with SOC verification — restore from a known-clean backup after confirming the backup is not itself encrypted, and only reconnect once the entry point is closed. Lessons learned should ask why mass file modification was not detected before a ransom note appeared. Detection improvement: an alert on high-volume file modification per account per minute on file servers, and an alert on shadow copy or backup catalogue deletion, both of which fire far earlier than a ransom note.

**Say this aloud to the interviewer:**
> "I would work the ransomware hands-on because it is causing damage right now, contain the executive account with fast reversible actions, hand the phishing follow-up to L1 with specific instructions, and tell the shift lead within minutes that I am running two incidents and need support."

**Key terms to mention:** blast radius, active damage rate, patient zero, shadow copy deletion, session revocation, parallel workstreams, escalate early.

**Weak answer to avoid:** Saying you would "handle the executive first because they are senior." Seniority of the reporter does not set incident priority; impact does.

**Likely follow-up:** "How do you decide when to wake up your manager at 2am?"

---

### Q4. What is the difference between an event, an alert, and an incident?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Precise vocabulary — sloppy terminology causes real confusion in reports.

**Model answer (say this aloud):**
> An event is any recorded observation on a system — a login, a process starting, a connection. Most events are completely normal and there are millions of them. An alert is an event or a pattern of events that matched a detection rule and was raised for human review. An incident is an alert or set of alerts that has been validated as a genuine security issue with actual or potential adverse impact, which means it now needs a response process, a ticket, an owner, and communication. So the funnel is events to alerts to incidents, and the analyst's job is to make that funnel accurate — not to let real incidents drop out and not to promote noise upward.

**Deeper explanation:**
The distinction is operationally important because each level has different obligations. Events are retained for search and forensics. Alerts require triage within a defined time. Incidents trigger the response lifecycle, notification requirements, evidence handling, and management reporting. A related term is **true positive incident** versus **security event of interest** — activity that is real and unusual but does not yet meet the incident threshold, and which should be recorded so a pattern can be spotted later. In reporting to management, using these words precisely stops the classic misunderstanding where "we had 4,000 incidents this month" is really "we had 4,000 alerts and nine incidents."

**Key terms to mention:** event, alert, incident, detection rule, adverse impact, triage funnel, incident threshold.

**Weak answer to avoid:** Using "alert" and "incident" as synonyms. It makes your metrics and your reports meaningless.

**Likely follow-up:** "Who in your organisation should have the authority to declare an incident?"

---

### Q5. Which incident response lifecycle do you follow, and what actually happens in each phase?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you know a recognised framework and can apply it, not just recite it.

**Model answer (say this aloud):**
> I follow the NIST 800-61 lifecycle: preparation, detection and analysis, containment eradication and recovery, and post-incident activity. Preparation is everything done before the incident — playbooks, log coverage, access, contact lists. Detection and analysis is where I validate the alert, scope it, and classify severity. Containment is stopping the spread, first short-term to stop the bleeding, then longer-term while we prepare a clean rebuild. Eradication removes the attacker's access and artefacts — malware, persistence, and any accounts they created. Recovery restores service with monitoring in place to confirm the attacker is really gone. Post-incident activity is the lessons-learned review and the detection improvements, and in my view that is the phase that decides whether the same incident happens again next month.

**Deeper explanation:**
NIST SP 800-61 has four phases; the SANS model splits the same work into six (preparation, identification, containment, eradication, recovery, lessons learned). Either is acceptable if applied properly. Two practical points separate strong candidates: first, **containment has two stages** — short-term (isolate the host, block the C2) and long-term (temporary controls that let the business keep running while a clean rebuild is prepared). Second, **you cannot eradicate before you have scoped**, because removing malware from one host while the attacker still holds valid credentials just tells them you are watching. Recovery must include a defined monitoring period with heightened detection on the affected assets, and a documented criterion for declaring the incident closed.

**Key terms to mention:** NIST SP 800-61, SANS six-step, short-term and long-term containment, eradication, scope before eradicate, monitoring period, lessons learned.

**Weak answer to avoid:** Listing the phases as a memorised sequence with no description of what you personally do in each.

**Likely follow-up:** "Give me an example where eradicating too early made an incident worse."

---

### Q6. `Scenario-based` You isolated a server because of a confirmed malware detection. Twenty minutes later a senior business manager calls you directly, angry, and demands you put it back online because it is affecting operations. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Professional conduct, chain of command, and whether you fold under pressure from authority.

**Model answer (say this aloud):**
> I stay calm and respectful, and I do not argue. I explain in plain language what I found, what the risk is if we reconnect, and what I am doing to shorten the outage. But I do not reverse a containment decision because of a phone call, because that decision is not mine alone to reverse — it belongs to the incident manager and the agreed process. So I tell the manager that I understand the operational impact, that I am escalating their concern to the incident manager right now so a proper risk decision can be made at the right level, and I give them a realistic time for the next update. Then I actually escalate it immediately and I record the request and my response in the incident ticket.

**Scenario walkthrough**

**Initial alert or situation**
Confirmed malware detection on a production server; host isolated under standard procedure; business stakeholder demands immediate reconnection.

**Investigation steps, in order**
1. Re-confirm the detection is a true positive before any conversation — check the process tree, file hash, and any network connections the malware made.
2. Determine what the server actually does and how many users are affected, so the operational impact is a fact and not a guess.
3. Establish whether a partial restoration is safe — for example, restoring access to a service while keeping the host blocked from the internet.
4. Check whether the malware has already spread, since that changes the answer completely.
5. Prepare a short factual brief for the incident manager: what was found, what is contained, what the business impact is, and what the options are.

**Evidence and log sources to review**
EDR detection details and process tree, file hashes and their reputation, firewall and proxy logs for outbound connections, authentication logs for the server, asset inventory for business criticality and owner.

**Severity and business impact reasoning**
There are two impacts in tension: security impact of leaving a compromised host on the network, and operational impact of the outage. My job is to present both accurately, not to choose for the organisation. A confirmed compromise on a production server that has outbound connectivity is High or Critical; an isolated host with no evidence of spread is a contained High.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Keep the existing isolation in place | Removing isolation from a confirmed compromised host |
| Add monitoring for related activity elsewhere | Accepting the risk of reconnection |
| Propose a partial-access option to the incident manager | Approving that partial-access option |
| Document the business request in the ticket | Communicating a restoration timeline to the business |

**Escalation and communication**
Escalate to the incident manager or shift lead within minutes, with the business impact stated clearly so it is weighed properly. Give the manager a specific next-update time rather than a vague promise. Never say "security policy says no" as your only answer — explain the actual risk in business terms.

**Recovery, lessons learned, detection improvement**
If the decision is to restore, restore with conditions — restricted network access, heightened monitoring, and a documented risk acceptance recorded by the right authority. Lessons learned should cover whether the business owner knew in advance that isolation was a possible outcome. Detection and process improvement: pre-agree isolation criteria and expected outage windows for critical systems, so this conversation is had before an incident, not during one.

**Say this aloud to the interviewer:**
> "I would not reverse containment on my own authority under pressure. I would acknowledge the impact, escalate the request immediately to the incident manager so the risk decision is made at the right level, give a clear time for the next update, and record the whole exchange in the ticket."

**Key terms to mention:** chain of command, risk acceptance, incident manager, documented decision, partial restoration, business impact in business language.

**Weak answer to avoid:** Either "I would put it back online because he is senior" or "I would refuse and hang up." Both are failures — one of judgement, one of professionalism.

**Likely follow-up:** "What if the incident manager is unreachable and the pressure continues?"

---

### Q7. What do you expect to do as an L2 that an L1 analyst does not?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand the role you are applying for.

**Model answer (say this aloud):**
> L1 is about coverage and speed — monitoring the queue, triaging alerts against playbooks, gathering the basic facts, and escalating anything that does not fit. L2 is about depth and decisions. I take the escalations and prove what actually happened: I pivot across data sources, build the timeline, determine root cause and scope, and decide whether it is an incident. I lead containment within my authority, and I know exactly where my authority ends. I also give back to the system — tuning noisy rules, writing or improving detections, updating playbooks, and coaching L1 so the same escalation comes in better next time. And I am the person who communicates clearly with management during an incident.

**Deeper explanation:**
The practical L2 responsibilities are: deep-dive investigation across SIEM, EDR, identity and network telemetry; incident declaration and severity classification; leading containment and coordinating with IT; root cause analysis; detection engineering and tuning; playbook maintenance; mentoring L1; and incident reporting. The mental shift from L1 to L2 is from *"does this alert match the playbook?"* to *"what is the truth of what happened on this network, and what should we do about it?"* An L2 is also expected to recognise when something exceeds their capability and hand it to L3 or forensics **with the evidence preserved** rather than damaged by well-meaning poking.

**Key terms to mention:** deep-dive investigation, pivoting, root cause, scope determination, detection tuning, playbook ownership, mentoring L1, knowing the limits of your authority.

**Weak answer to avoid:** "L2 is the same as L1 but with harder alerts." It misses the ownership, decision-making, and improvement responsibilities entirely.

**Likely follow-up:** "Tell me about a time you improved a process rather than just closing tickets."

---

### Q8. `Scenario-based` An L1 escalates a ticket to you that just says "suspicious login, please check." There is a username and a timestamp and nothing else. What do you do?

- **Difficulty:** Core
- **Type:** Scenario-based
- **What the interviewer is testing:** Investigation independence plus how you treat junior colleagues.

**Model answer (say this aloud):**
> I do not send it back and I do not complain. I work it myself with the two facts I have, because the username and timestamp are enough to start. Then separately — after the investigation, not during — I go back to the L1 and show them what I looked at and what three or four fields would have made the escalation immediately actionable. That way the ticket gets worked and the next one arrives better. If the same gap keeps appearing across the team, that is a playbook or template problem, not an individual problem, and I raise it as a template fix.

**Scenario walkthrough**

**Initial alert or situation**
Vague escalation: one username, one timestamp, no source IP, no detection rule name, no reason for suspicion.

**Investigation steps, in order**
1. Pull all authentication events for that account in a window around the timestamp — successes and failures, interactive and non-interactive.
2. Establish the account's baseline: what does this user normally do, from which locations, devices, and hours, and is it a normal user, an admin, or a service account.
3. Identify the source IP and device for the login in question, and check whether the device is known and managed.
4. Check the authentication method — was MFA satisfied, and how; was it a legacy protocol that bypasses MFA.
5. Look at what happened *after* the login: mailbox rule changes, privilege changes, file access, new device registration, or process execution on an endpoint.
6. Check whether other accounts show the same source IP or the same pattern, which would turn a single-account question into a campaign.

**Evidence and log sources to review**
Identity provider sign-in logs (interactive and non-interactive), directory audit logs for changes made by the account, VPN and firewall logs for the source IP, endpoint logon events, mailbox audit logs, and any EDR activity on the device used.

**Severity and business impact reasoning**
Start at Low-to-Medium and let the evidence move it. It becomes High immediately if the account is privileged, if MFA was not properly satisfied, if the source is unexpected infrastructure, or if post-login activity shows persistence such as a new inbox rule or a new registered device.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke active sessions for the account | Disabling the account, if it is a VIP or service account |
| Force a password reset per standard procedure | Blocking a whole source IP range at the perimeter |
| Remove an attacker-created mailbox rule after preserving it | Removing legitimate access the user needs to work |
| Raise the account's risk monitoring | Notifying the user directly if the case is sensitive |

**Escalation and communication**
If it turns out to be a genuine account compromise, escalate to the incident manager with the timeline and the containment already taken. Keep the L1 in the loop and explain the outcome — that is what converts a weak escalation into a trained analyst.

**Recovery, lessons learned, detection improvement**
Recovery: confirm credentials rotated, sessions revoked, no persistence left behind. Lessons learned: the escalation template is missing mandatory fields. Detection improvement: make the alert itself carry the source IP, ASN, device, MFA result, and prior-baseline comparison so the analyst does not have to hunt for them.

**Say this aloud to the interviewer:**
> "I would investigate it properly with the two facts I had, then go back to the L1 afterwards and show them the fields that would have made it actionable. If the gap is repeating across the team, I would fix the escalation template rather than blame the person."

**Key terms to mention:** baseline behaviour, non-interactive sign-ins, legacy authentication, post-login activity, persistence indicators, escalation template, coaching not blaming.

**Weak answer to avoid:** "I would reject the ticket and ask them to fill it in properly." It wastes time during a possible compromise and damages the team.

**Likely follow-up:** "What five fields would you make mandatory in an escalation template?"

---

### Q9. How do you reduce false positives without reducing detection coverage?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Detection maturity — whether you tune with evidence or just switch rules off.

**Model answer (say this aloud):**
> I start with data, not opinion. I look at which rules generate the most alerts and what percentage of those close as false positives, because that tells me where the noise really is. Then for the worst offenders I find the specific cause — usually a scanner, a backup agent, a service account, or an admin tool doing its job. I tune narrowly: exclude that exact process on that exact host under that exact account, rather than disabling the rule or excluding a whole subnet. I document every exclusion with a reason and an owner, and I review exclusions periodically, because an undocumented exclusion is a permanent blind spot that an attacker can walk through. If a rule cannot be made accurate, I would rather convert it into a lower-priority hunting query than leave it flooding the queue.

**Deeper explanation:**
Tuning principles: (1) **narrow the exclusion** — scope it by process path, account, host and, where possible, command line, never by broad subnet or "all admin accounts"; (2) **prefer thresholds and correlation to exclusions** — a rule that fires on ten failures in a minute rather than one failure keeps coverage while killing noise; (3) **raise the fidelity of the logic** — add parent process, add reputation, add "first time seen in the environment"; (4) **document and expire exclusions**, with a named owner and a review date. Measure the effect with alert volume, false positive rate, and mean time to triage before and after. The danger of blunt tuning is real: attackers deliberately mimic scanners, backup accounts, and admin tools precisely because those are the things SOCs exclude.

**Key terms to mention:** false positive rate, narrow exclusions, thresholds, correlation, detection fidelity, documented exclusion with review date, alert fatigue, blind spot.

**Weak answer to avoid:** "I would disable the noisy rule." That is how coverage is lost silently.

**Likely follow-up:** "How would you prove to management that your tuning did not lose detection coverage?"

---

### Q10. `Scenario-based` Over six hours you see three separate low-severity alerts: a single failed VPN login from an unusual country, a PowerShell execution alert on a laptop, and a small outbound transfer to a file-sharing site. Individually they were closed. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Correlation thinking — the core L2 skill of seeing one attack in scattered noise.

**Model answer (say this aloud):**
> I treat these as one hypothesis rather than three tickets. The pattern reads like initial access, then execution, then exfiltration, which maps to a real attack chain. So I check whether they share a common entity — the same user, the same host, the same IP, or the same time window. If they do, I reopen them under a single incident and investigate as one case. This is exactly why low-severity alerts should never be closed without linking them to an entity: individually they are noise, together they are a kill chain.

**Scenario walkthrough**

**Initial alert or situation**
Three low-severity alerts across six hours, closed separately, that together resemble initial access, execution, and exfiltration.

**Investigation steps, in order**
1. Pivot on the user account: was the same identity present in all three alerts, and if not, are the accounts related by device or department.
2. Pivot on the host: did the PowerShell execution and the outbound transfer come from the same machine, and did that machine's user also attempt the VPN login.
3. Build a single timeline in UTC across the three events and look for the connective tissue between them — what happened in the gaps.
4. Examine the PowerShell in detail: full script block content, encoded commands, parent process, and whether it downloaded anything.
5. Examine the outbound transfer: destination domain, volume, whether it was a personal account, and what files were read on the host immediately before.
6. Check for a successful VPN or cloud login after the failed one, from that country or that ASN.
7. Sweep for the same indicators on other hosts to establish whether it is one machine or several.

**Evidence and log sources to review**
VPN authentication logs, identity sign-in logs, PowerShell script block logging, EDR process and network telemetry for the host, proxy and DNS logs for the file-sharing destination, file access auditing on the source host, and the original three alert records with their closure notes.

**Severity and business impact reasoning**
Reclassify to High or Critical. Three chained stages of an attack indicate an operator with hands on a system, not a random event. Business impact depends on what was in the transferred data — confidentiality impact is the driving concern, and in a government environment potential data disclosure sets the severity floor.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the endpoint at EDR level | Taking the VPN concentrator or a gateway offline |
| Revoke sessions and reset the user's credentials | Disabling a whole group of accounts |
| Block the file-sharing destination per standing policy | Blocking a widely used business service |
| Preserve memory and disk artefacts on the host | Reimaging before evidence capture is complete |

**Escalation and communication**
Escalate as a suspected active intrusion with possible data exfiltration. Report it as one incident with three linked alerts, and state clearly what is confirmed versus what is still hypothesis. Data leaving the organisation triggers notification duties, so management and the data owner must be told promptly and factually.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the endpoint, rotate credentials, verify no persistence, monitor the user and host for a defined period. Lessons learned: three analysts each closed a piece of one attack because the alerts were not entity-linked. Detection improvement: build a correlation rule that raises severity when the same user or host appears in alerts from different attack stages within a rolling window, and require entity linkage before a low-severity alert can be closed.

**Say this aloud to the interviewer:**
> "Individually they were noise. Together they read as initial access, execution and exfiltration on the same entity, so I would reopen them as one incident, build a single timeline, and confirm whether the same user or host connects all three."

**Key terms to mention:** correlation, entity pivoting, kill chain stages, rolling time window, alert linkage, exfiltration, reclassification.

**Weak answer to avoid:** Investigating the three alerts separately again and reaching the same three separate conclusions.

**Likely follow-up:** "Describe the logic of a correlation rule that would have caught this automatically."

---

### Q11. `Scenario-based` A workstation is making regular outbound connections to an unknown external IP every five minutes with a consistent small payload. Do you block it immediately or watch it?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Containment judgement — the trade-off between stopping the threat and preserving intelligence.

**Model answer (say this aloud):**
> The default is contain, because leaving a live command and control channel open on the network is not something an L2 decides to do alone. But before I block I spend a few minutes gathering what I would lose if I blocked — the destination, the process making the connections, the beacon interval and jitter, and whether other hosts talk to the same infrastructure. Then I contain at the host level rather than only blocking the IP, because blocking the IP alone leaves the implant in place and simply tells the attacker to switch to a backup channel. If there is a genuine intelligence reason to observe rather than block, that is a decision for the incident manager and it needs an explicit time limit and monitoring plan, never an open-ended "let's watch it."

**Scenario walkthrough**

**Initial alert or situation**
Regular, low-volume, periodic outbound connections from a workstation to an external IP with no business reputation — the classic beaconing pattern.

**Investigation steps, in order**
1. Confirm the pattern is genuinely periodic — measure the interval and the variation, since machine-regular timing with small jitter is a strong beaconing indicator.
2. Identify the process and its full path, parent process and command line on the endpoint making the connections.
3. Determine how the process got there and when it first appeared, which gives you the likely initial access.
4. Check the destination: resolution history, hosting provider, certificate details, first-seen date, and whether it is a known service.
5. Determine whether the traffic is encrypted, and what the request pattern looks like — consistent size and consistent URI structure are meaningful.
6. Search the whole environment for other hosts contacting the same infrastructure or running the same binary hash.
7. Check for persistence on the host — scheduled tasks, run keys, services — because that tells you whether blocking will actually stop it.

**Evidence and log sources to review**
Firewall and proxy connection records, DNS query logs, EDR network and process telemetry, endpoint autoruns and scheduled task inventory, TLS metadata such as certificate and server name indication, and threat intelligence on the destination.

**Severity and business impact reasoning**
Active command and control is High or Critical regardless of what has been stolen yet, because it means an external party has interactive or automated control over an internal asset. Impact escalates sharply if the host holds privileged credentials or has access to sensitive shares.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the endpoint via EDR under standing procedure | Deciding to leave a live C2 channel open for observation |
| Block the destination IP and domain per pre-approved threat blocking | Perimeter changes outside pre-approved scope |
| Collect volatile evidence and preserve the binary | Wiping or reimaging the host |
| Sweep other endpoints for the same indicators | Environment-wide blocking that could break business traffic |

**Escalation and communication**
Escalate as suspected active C2. State the beacon interval, the destination, the process, and how many hosts are affected. If leadership wants to observe rather than block, get that decision recorded, with an owner and a stop time.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the host from a clean image, rotate any credentials used on it, and confirm the persistence mechanism is gone. Lessons learned: how did the implant arrive and why did no earlier control stop it. Detection improvement: a beaconing detection based on connection regularity and low payload variance rather than only on IP reputation, since reputation lists always lag new infrastructure.

**Say this aloud to the interviewer:**
> "My default is to contain, because an open C2 channel is not mine to leave running. I would capture the process, destination and beacon pattern first so we do not lose the intelligence, then isolate the host — not just block the IP, because that leaves the implant in place."

**Key terms to mention:** beaconing, jitter, command and control, isolate at host level, backup C2 channel, persistence, indicator sweep, time-boxed observation.

**Weak answer to avoid:** "I would just block the IP on the firewall." That treats the symptom, and modern malware fails over to a second channel within minutes.

**Likely follow-up:** "The process is signed by a legitimate vendor. Does that change your answer?"

---

### Q12. `Scenario-based` You are investigating a single infected laptop. Two hours in, you find the same malicious binary on four servers. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you re-scope and re-escalate when the facts change, instead of finishing the ticket you started.

**Model answer (say this aloud):**
> The moment the scope changes, the incident changes. This is no longer a malware ticket on a laptop, it is a suspected multi-host compromise, so I stop, re-classify the severity upward, and escalate immediately rather than continuing quietly. I inform the shift lead and incident manager with what I have found and what is confirmed. Then I widen the sweep across the whole estate for the same indicators, because four servers found means there are probably more I have not looked for yet. I also shift my focus from the malware to the mechanism — how did it get onto servers, which almost always means credentials or a management tool were abused, and that is a bigger problem than the binary.

**Scenario walkthrough**

**Initial alert or situation**
A single-endpoint malware investigation expands to four servers hosting the same malicious binary.

**Investigation steps, in order**
1. Confirm the match is the same binary by hash, not just by filename, and confirm the file is actually executing rather than sitting dormant.
2. Establish first-seen time on each host and order them chronologically to find the earliest and identify patient zero.
3. Identify the deployment mechanism — which account wrote the file, over which protocol, and from where; look for remote service creation, scheduled tasks, remote WMI, or a software deployment tool.
4. Identify the account used for that deployment and check where else that account authenticated in the same window.
5. Sweep the entire estate for the hash, for the filename, for the persistence mechanism, and for the C2 destination — four separate sweeps, because the attacker may vary the file.
6. Check the servers' roles and what data or credentials they hold, particularly whether a domain controller or a management server is involved.

**Evidence and log sources to review**
EDR file and process telemetry across all hosts, file creation events, service and scheduled task creation events, remote authentication logs on each server, the deployment account's authentication history, software deployment tool logs, and network telemetry to shared C2 infrastructure.

**Severity and business impact reasoning**
Escalate to Critical. Malware on multiple servers deployed by a common mechanism means an actor with valid privileged credentials and the ability to move laterally. Assume credential compromise until proven otherwise. Business impact covers availability of server workloads and confidentiality of anything those credentials could reach.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the original laptop | Isolating production servers — needs incident manager and IT |
| Block the C2 infrastructure per standing procedure | Domain-wide privileged password reset |
| Preserve evidence on each affected host | Taking a domain controller offline |
| Continue and widen the indicator sweep | Declaring a major incident to executives |

**Escalation and communication**
Escalate immediately, before finishing the technical work. The message should be short and factual: same binary confirmed on five hosts including four servers, deployment mechanism under investigation, suspected credential compromise, recommending major incident declaration. Do not wait until you have a complete picture — leadership needs to start mobilising in parallel.

**Recovery, lessons learned, detection improvement**
Recovery: coordinated rebuild rather than piecemeal cleaning, plus rotation of any credential exposed on affected hosts, done in a planned order so the attacker cannot simply re-enter. Lessons learned: why lateral deployment to servers was possible and was not detected at the first hop. Detection improvement: alert on the same file hash appearing on multiple hosts within a short window, and on remote service or scheduled task creation on servers by non-approved accounts.

**Say this aloud to the interviewer:**
> "When the scope changed, the incident changed. I would re-classify it upward and escalate straight away rather than finishing my ticket quietly, then widen the sweep and shift focus from the binary to how it was deployed — because that usually means stolen credentials."

**Key terms to mention:** re-scoping, patient zero, deployment mechanism, lateral movement, credential compromise assumption, estate-wide sweep, coordinated remediation.

**Weak answer to avoid:** "I would clean the four servers and close the ticket." Cleaning without finding the mechanism guarantees reinfection.

**Likely follow-up:** "In what order would you reset credentials in a domain-wide compromise?"

---

### Q13. How do you document an incident so that it stands up to internal review or audit months later?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Discipline, accuracy, and awareness that documentation is evidence.

**Model answer (say this aloud):**
> I write for someone who was not there and who may read it a year later. Everything gets a UTC timestamp, including when I was notified, when I acted, and when I escalated. I separate facts from assessment explicitly — what the logs show, and separately what I believe that means and with what confidence. I record every action I took and who authorised it, because in an investigation the record of who approved containment matters as much as the containment. I reference evidence by identifiers — hashes, log queries, ticket numbers, file paths — so it can be retrieved and verified rather than taken on trust. And I never edit history: if I was wrong, I add a correction with a timestamp rather than quietly changing the earlier entry.

**Deeper explanation:**
An incident record should contain: an executive summary in plain language; a UTC timeline; affected assets, accounts and data; the detection source; the investigation steps and the queries used; the evidence collected and where it is stored, with integrity hashes; the containment, eradication and recovery actions with approvers; the root cause; the impact assessment; and the recommendations with owners. Two habits distinguish professionals: **separating observation from interpretation** with explicit confidence language ("the logs show X"; "I assess with medium confidence that this indicates Y"), and **append-only correction**, which preserves the integrity of the record. In a government or national-security environment, assume the record may be reviewed by people outside the SOC and write it to the applicable classification and handling standard.

**Key terms to mention:** UTC timeline, facts versus assessment, confidence levels, chain of custody, evidence identifiers and hashes, approver recorded, append-only corrections, handling and classification.

**Weak answer to avoid:** "I write a summary at the end of the incident." Documentation written only at the end loses the detail that matters and invites reconstruction errors.

**Likely follow-up:** "How would you write the same incident for a non-technical director in five lines?"

---

### Q14. `Scenario-based` Your shift ends in fifteen minutes and you are in the middle of an active incident that is not resolved. How do you hand it over?

- **Difficulty:** Core
- **Type:** Scenario-based
- **What the interviewer is testing:** Continuity discipline — one of the most common real failures in 24/7 SOCs.

**Model answer (say this aloud):**
> I hand over in writing and verbally, never one or the other. In writing I bring the ticket completely up to date so the record is self-contained: current status, confirmed facts, what is still unproven, actions taken and by whom, and what is pending. Verbally I do a direct handover to the named analyst taking it, walking through the timeline and the immediate next steps, and I confirm they have the access and the context they need. I do not leave until the receiving analyst has explicitly accepted ownership, and I record that handover in the ticket with the time and the names. If the incident is critical, I also tell the shift lead that the handover happened, and I make myself reachable for a period afterwards.

**Scenario walkthrough**

**Initial alert or situation**
Active, unresolved incident at shift change with investigation in progress and containment partially applied.

**Investigation steps, in order**
1. Update the ticket with everything done so far, in UTC, before the conversation — the written record must exist first.
2. State clearly what is confirmed fact versus current hypothesis, so the next analyst does not inherit an assumption as truth.
3. List containment already in place and any approvals obtained, so nothing is repeated or accidentally reversed.
4. List the specific next steps in priority order, with the exact queries or hosts to check.
5. Name any external parties already contacted and what they were told, so messaging stays consistent.
6. Conduct the verbal walkthrough and confirm the receiving analyst accepts ownership.

**Evidence and log sources to review**
The incident ticket itself, the saved queries and their results, the evidence store and its inventory, any exported artefacts and their hashes, and the communication log.

**Severity and business impact reasoning**
Handover risk is proportional to incident severity. For a Critical incident a silent handover is itself a risk event — a dropped thread at shift change is how a contained incident becomes an uncontained one.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Ensure existing containment stays in place through handover | Making a new containment decision in the last minutes of shift |
| Fully document status and next steps | Closing the incident to avoid handing it over |
| Confirm and record acceptance of ownership | Leaving without a named receiving analyst |

**Escalation and communication**
Notify the shift lead of the handover for any High or Critical incident. If no qualified analyst is available to receive it, that is an escalation in itself — say so rather than leaving the incident unowned.

**Recovery, lessons learned, detection improvement**
Lessons learned should include whether the handover preserved momentum. Process improvement: a standard handover template with mandatory fields — status, confirmed facts, open questions, containment applied, approvals, next actions, and named owner — plus a rule that no High or Critical incident may change shift without a verbal handover.

**Say this aloud to the interviewer:**
> "In writing and verbally, both. I update the ticket so it is self-contained, separate confirmed facts from hypotheses, walk the receiving analyst through it, and record their acceptance of ownership with the time. I do not close an incident just to avoid handing it over."

**Key terms to mention:** handover template, self-contained ticket, confirmed facts versus hypothesis, ownership acceptance, continuity of evidence, shift lead notification.

**Weak answer to avoid:** "I would stay until it is finished." It sounds committed but it does not scale, and it hides a broken handover process. Say you would stay if genuinely needed *and* that you would still hand over properly.

**Likely follow-up:** "What is the single most common thing lost in a handover?"

---

### Q15. `Scenario-based` During containment you block an IP address and it turns out to be a production service, causing an outage. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Integrity, ownership of mistakes, and calm recovery — a deliberate character test.

**Model answer (say this aloud):**
> I report it immediately, in my own words, before anyone else finds it. The first priority is restoring service: I revert the block, verify the service recovers, and confirm with the affected team. Then I explain clearly what I did and why, including what information I had at the time. I do not hide it, minimise it, or wait to see whether anyone notices, because in a security team integrity is the whole job — if I cannot be trusted to report my own error, nothing else I report can be trusted either. Afterwards I look at why it happened: usually the block was made without checking asset ownership, so the fix is a verification step in the procedure, not just being more careful next time.

**Scenario walkthrough**

**Initial alert or situation**
A containment block causes an unintended production outage.

**Investigation steps, in order**
1. Confirm the outage is caused by your block, not coincidence — check the timing against the rule change and the service failure.
2. Revert the block immediately if the security risk allows, or narrow it so the legitimate traffic passes while the threat stays blocked.
3. Verify service restoration with the affected team rather than assuming.
4. Reconstruct what the IP actually is — asset inventory, DNS records, service owner — and why it appeared malicious.
5. Re-examine the original threat: if the IP was genuinely involved, the threat still exists and needs a different containment approach.

**Evidence and log sources to review**
Firewall or proxy change logs with the exact rule and timestamp, service monitoring and availability data, asset inventory and DNS records for the address, the original threat intelligence or alert that motivated the block.

**Severity and business impact reasoning**
Report both impacts honestly — the duration and scope of the outage, and the residual security risk now that the block is reverted. Do not understate either one to make the mistake look smaller.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revert my own change to restore service | Deciding to leave the block in place despite the outage |
| Narrow the rule to a specific port or host | Any new blocking approach after the incident |
| Notify the affected team and the shift lead | External communication about the outage cause |

**Escalation and communication**
Tell the shift lead immediately and factually: what I blocked, when, what broke, what I have done, and the current status. Apologise once, concisely, then focus on facts and remediation. Volunteering the error early is far stronger than being asked about it later.

**Recovery, lessons learned, detection improvement**
Recovery: service restored and verified, and a safe alternative containment implemented for the original threat. Lessons learned: the procedure allowed a block without an ownership check. Process improvement: require an asset inventory and DNS lookup before blocking any internal-facing or ambiguous address, maintain a never-block list of critical infrastructure, and prefer narrowly scoped blocks over broad ones.

**Say this aloud to the interviewer:**
> "I would restore service first, then report it myself immediately and in full, before anyone found it. Then I would fix the procedure that allowed a block without an ownership check — because 'be more careful' is not a control."

**Key terms to mention:** immediate disclosure, revert and verify, never-block list, narrowly scoped containment, procedural control, integrity, blameless post-incident review.

**Weak answer to avoid:** "I would fix it quietly and see if anyone noticed." In a high-trust environment this answer ends the interview.

**Likely follow-up:** "How do you build a team culture where people report their own mistakes?"

---

[⬅ Back to README](../README.md) · [Next: SIEM & Detection ➡](02-siem-logging-and-detection-engineering.md)
