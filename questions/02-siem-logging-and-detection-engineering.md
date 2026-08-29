# 02 · SIEM, Logging and Detection Engineering

**12 questions · Q16–Q27 · 4 scenario-based**

[⬅ Previous: Triage & IR](01-l2-soc-triage-and-incident-response.md) · [Back to README](../README.md) · [Next: Windows & AD ➡](03-windows-active-directory-and-identity.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q16 | What a SIEM actually does | Core | Standard |
| Q17 | SIEM vs SOAR vs EDR/XDR | Core | Standard |
| Q18 | Parsing, normalisation and enrichment | Core | Standard |
| Q19 | A log source has gone silent | Advanced | Scenario-based |
| Q20 | From detection idea to production rule | Advanced | Standard |
| Q21 | Write KQL for brute force followed by success | Advanced | Standard |
| Q22 | Splunk fundamentals — index, sourcetype, SPL | Core | Standard |
| Q23 | Alert volume tripled overnight | Advanced | Scenario-based |
| Q24 | Priority log sources you insist on | Core | Standard |
| Q25 | Correlation rules vs behavioural/UEBA detection | Advanced | Standard |
| Q26 | Proving a detection gap exists | Advanced | Scenario-based |
| Q27 | The alert and the raw log disagree | Advanced | Scenario-based |

---

### Q16. What does a SIEM actually do, and how is it different from a log management platform?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand the tool you will live in all day.

**Model answer (say this aloud):**
> A SIEM collects logs from across the environment, parses and normalises them into a common schema, enriches them with context like asset criticality and threat intelligence, correlates events across different sources, and raises alerts against detection rules. It also stores the data so we can search historically for investigation and hunting, and it produces the metrics and reports management needs. A log management platform does the collection, storage and search part but not the detection and correlation part. The value of a SIEM is that it can join an identity event, an endpoint event and a network event into one picture — that is what a single log source can never do on its own.

**Deeper explanation:**
The SIEM pipeline is: **collect → parse → normalise → enrich → correlate → alert → store → report**. Enrichment is where most of the practical value sits: adding user department and privilege level, asset owner and criticality, geolocation and ASN, and threat intelligence context, so the analyst does not have to look those up manually for every alert. Correlation is what makes it a SIEM rather than a search engine — the ability to express "this authentication event, followed by that process event on the same host, within ten minutes." Modern platforms such as Microsoft Sentinel add scheduled analytics rules, entity behaviour analytics, watchlists, and automation playbooks. Practical constraints you should mention: log volume drives cost, retention drives investigative reach, and coverage gaps are invisible unless you actively monitor for them.

**Key terms to mention:** normalisation, enrichment, correlation, analytics rules, retention, entity behaviour, ingestion cost, log source coverage.

**Weak answer to avoid:** "A SIEM collects logs and shows alerts." True but shallow — mention correlation, enrichment and retention.

**Likely follow-up:** "If ingestion cost forced you to drop one log source, which would you drop and why?"

---

### Q17. Explain the difference between SIEM, SOAR and EDR/XDR, and how they work together in a real workflow.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Tooling literacy and understanding of an end-to-end workflow.

**Model answer (say this aloud):**
> EDR sits on the endpoint and gives deep visibility into processes, files, registry and network activity on that machine, and it can act on the machine — kill a process, quarantine a file, isolate the host. XDR extends that same detection and response idea across endpoint, identity, email and cloud so the correlation happens across domains rather than just on one device. SIEM is the central place where logs from everything, including the EDR, come together for correlation, search, long-term retention and reporting. SOAR is the automation and orchestration layer — it takes an alert and runs the repeatable steps automatically, like enriching indicators, opening the ticket, and executing pre-approved containment. In a real workflow, EDR or XDR detects, SIEM correlates it with the rest of the environment, SOAR handles the mechanical enrichment and response, and the analyst makes the judgement calls.

**Deeper explanation:**

| | Primary role | Where it sees | Response capability |
|---|---|---|---|
| **EDR** | Endpoint detection and response | One host, deeply — process tree, file, registry, network | Kill, quarantine, isolate host, collect forensics |
| **XDR** | Cross-domain detection and response | Endpoint + identity + email + cloud, correlated | Endpoint actions plus identity and mail actions |
| **SIEM** | Central correlation, search, retention, reporting | Everything that sends logs | Usually none directly; triggers automation |
| **SOAR** | Orchestration and automation of the response | Whatever it is integrated with | Executes pre-approved playbook actions |

The important nuance for an L2: automation should handle the **deterministic** work — reputation lookups, ticket creation, gathering the user's recent sign-ins, notifying the user. Judgement calls — declaring an incident, isolating a production server, disabling a VIP account — should stay human, or at minimum be automated only with an explicit approval step in the playbook. Over-automating containment is how a SOC causes its own outage.

**Key terms to mention:** endpoint telemetry, cross-domain correlation, orchestration, playbook, pre-approved actions, human-in-the-loop, mean time to respond.

**Weak answer to avoid:** Describing SOAR as "the tool that replaces analysts." SOAR removes repetitive steps; it does not make decisions.

**Likely follow-up:** "Which containment action would you never fully automate, and why?"

---

### Q18. What are parsing, normalisation and enrichment, and why do they matter to you as an analyst?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand why your searches sometimes return nothing.

**Model answer (say this aloud):**
> Parsing is breaking a raw log line into fields — source IP, user, action, result. Normalisation is mapping those fields into a common schema so that a firewall's source address field and a proxy's client IP field both end up in the same field name. Enrichment adds context that was not in the log — geolocation, asset owner, user privilege level, threat intelligence. They matter to me because if parsing is broken, my query returns nothing and I may wrongly conclude nothing happened. So when a search comes back empty, my first question is whether the data is really absent or whether the field I searched is not populated for that source.

**Deeper explanation:**
An empty result set has three very different causes: the activity did not happen; the log source is not being collected; or the data is present but the field is unparsed or named differently. A professional analyst distinguishes them by first querying the source with no filters to confirm data is arriving and to inspect the actual field names. Common normalisation schemas include the Advanced Security Information Model in Microsoft Sentinel and the Common Information Model in Splunk. Enrichment quality directly determines triage speed: an alert that already carries "this account is a Domain Admin and this host is a tier-0 asset" is triaged in seconds, while the same alert without context takes ten minutes of lookups.

**Key terms to mention:** field extraction, common schema, ASIM, CIM, enrichment, absence of evidence versus evidence of absence, unparsed fields.

**Weak answer to avoid:** "If the search returns nothing, nothing happened." That is the most dangerous assumption in SOC work.

**Likely follow-up:** "How would you confirm a log source is healthy without opening a ticket with the platform team?"

---

### Q19. `Scenario-based` You notice that a critical detection rule has not fired at all in nine days, when it normally fires several times a week. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you treat silence as suspicious — the discipline that separates monitoring from watching.

**Model answer (say this aloud):**
> Silence is a finding, not a relief. My first assumption is that we have lost visibility, not that attacks stopped. So I check whether the log source feeding that rule is still arriving at all, and how its volume compares with the same period last month. Then I check whether the rule itself was changed, disabled, or broken by a schema change. I also check whether the sending systems changed — an agent upgrade, a policy change, a firewall rule blocking the forwarder. Until I can explain the silence, I treat it as a live detection gap: I tell the shift lead, I record it as a visibility issue, and where possible I compensate with a manual hunt over whatever data is still available.

**Scenario walkthrough**

**Initial alert or situation**
A detection rule that historically fires regularly has produced zero alerts for nine days.

**Investigation steps, in order**
1. Query the underlying table or index directly with no filters for the last 24 hours to confirm whether any data at all is arriving from that source.
2. Compare daily ingestion volume for that source across the last 60 days to locate the exact day the drop began.
3. Check the rule's own change history — was it edited, disabled, its schedule changed, or its threshold raised.
4. Check whether a field the rule depends on has been renamed or stopped being populated after a product or parser update.
5. Check the collection path end to end: agent health on the sending systems, forwarder status, connectivity, and licence or quota limits.
6. Correlate the start date of the drop with any change record — upgrades, network changes, policy changes.
7. Run the rule's logic manually as a hunting query over the available period to see whether events exist that simply were not alerted.

**Evidence and log sources to review**
SIEM ingestion and health metrics per source, the analytics rule audit and version history, agent or connector health dashboards, change management records, and the raw table itself.

**Severity and business impact reasoning**
A silent detection over a critical rule is a blind spot with unknown duration — treat it as a High operational risk. The business impact is that any attack in that category during those nine days would not have been detected, so it also creates a retrospective hunting obligation.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Run manual hunting queries to cover the gap | Re-enabling a rule someone deliberately disabled |
| Raise a visibility incident and notify the shift lead | Making configuration changes on source systems |
| Document the gap window precisely | Accepting the risk of the gap without remediation |
| Restore a rule I own after confirming the cause | Changing collection architecture or licence tiers |

**Escalation and communication**
Report it as a monitoring gap incident with the exact start time and the affected detection coverage. State plainly: we could not have detected X between these dates. Under-reporting a visibility gap is as serious as missing an alert.

**Recovery, lessons learned, detection improvement**
Recovery: restore collection, validate with a controlled test that the rule fires again, then retro-hunt across the gap window. Lessons learned: no one noticed for nine days. Detection improvement: implement **log source health monitoring** — alert when a source's event volume drops below a percentage of its own baseline, and alert on any analytics rule being disabled or modified.

**Say this aloud to the interviewer:**
> "Silence is a finding. I would assume we lost visibility rather than assume attacks stopped — check ingestion volume against baseline, check whether the rule was changed or broken by a schema update, hunt manually to cover the gap, and report it as a monitoring gap with an exact window."

**Key terms to mention:** detection gap, log source health, ingestion baseline, retro-hunt, rule version history, schema change, compensating control.

**Weak answer to avoid:** "It just means there were no attacks that week." That is the answer that lets a blind spot last for months.

**Likely follow-up:** "How would you build monitoring that tells you a log source died within an hour?"

---

### Q20. Take me from a detection idea to a production rule. What are your steps?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Detection engineering discipline, not just rule copying.

**Model answer (say this aloud):**
> I start from the behaviour, not the tool. I define exactly what attacker behaviour I want to catch and map it to a MITRE ATT&CK technique so the coverage is measurable. Then I check whether we actually have the telemetry to see it, because a rule with no data is a false sense of security. I write the logic and run it historically over thirty to ninety days to see how much it would have fired and on what — that tells me the false positive load before it ever reaches the queue. I refine with thresholds, exclusions and enrichment, then test it with a safe, authorised simulation of the behaviour to confirm it actually fires. Before it goes live I write the triage steps into the alert itself, so whoever gets it at 3am knows what to do. Then I deploy, monitor the first weeks, and tune.

**Deeper explanation:**
The full pipeline: **hypothesis → ATT&CK mapping → data source validation → logic → historical backtest → tuning → authorised validation test → documentation → deployment → measurement**. Two steps candidates usually forget: the **historical backtest**, which predicts the alert volume and reveals benign patterns before the queue drowns; and the **response documentation attached to the rule**, which is what makes the detection usable by the rest of the team. Every rule should record: the behaviour it targets, the ATT&CK technique, its data dependencies, known benign triggers, the triage steps, and an owner. A rule without an owner rots. Validation should use authorised, controlled simulation in a test scope — never live malware on production, and never an unauthorised test.

**Key terms to mention:** ATT&CK mapping, telemetry validation, backtesting, threshold tuning, authorised detection validation, rule documentation, rule owner, coverage measurement.

**Weak answer to avoid:** "I would find a rule online and enable it." Untested imported rules are the single largest source of alert fatigue.

**Likely follow-up:** "How do you measure whether your detection coverage is improving over time?"

---

### Q21. Write me a query that finds brute force attempts followed by a successful login, and explain the logic.

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Practical query ability and whether you can explain logic out loud.

**Model answer (say this aloud):**
> The logic is to build two sets and join them on the entity. First, source IP and account pairs with a high count of failed authentications in a time window. Second, successful authentications for the same account and IP. Then I join them and keep only cases where the success happened during or shortly after the failure burst. The reason I join on both account and source IP is to avoid matching a user's own successful login from their own laptop against an attacker's failures from elsewhere. And the reason I keep a short window after the last failure is that a success at that moment is the definition of a successful brute force.

**Deeper explanation:**

Microsoft Sentinel — Entra ID sign-ins:

```kusto
let window = 1h;
let failureThreshold = 15;
let failures =
    SigninLogs
    | where TimeGenerated > ago(24h)
    | where ResultType == "50126"          // invalid username or password
    | summarize FailCount = count(),
                FirstFail = min(TimeGenerated),
                LastFail  = max(TimeGenerated)
              by UserPrincipalName, IPAddress
    | where FailCount >= failureThreshold;
let successes =
    SigninLogs
    | where TimeGenerated > ago(24h)
    | where ResultType == "0"              // successful sign-in
    | project UserPrincipalName, IPAddress, SuccessTime = TimeGenerated,
              AppDisplayName, Location, UserAgent;
failures
| join kind=inner successes on UserPrincipalName, IPAddress
| where SuccessTime between (FirstFail .. LastFail + window)
| project UserPrincipalName, IPAddress, FailCount, FirstFail, LastFail,
          SuccessTime, AppDisplayName, Location
| sort by FailCount desc
```

Windows domain equivalent, using security events:

```kusto
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID in (4625, 4624)
| summarize Failed  = countif(EventID == 4625),
            Success = countif(EventID == 4624),
            FirstSeen = min(TimeGenerated),
            LastSeen  = max(TimeGenerated)
          by Account, IpAddress, bin(TimeGenerated, 30m)
| where Failed >= 15 and Success >= 1
| sort by Failed desc
```

Splunk equivalent in SPL:

```
index=wineventlog (EventCode=4625 OR EventCode=4624)
| bucket _time span=30m
| stats count(eval(EventCode=4625)) as failed
        count(eval(EventCode=4624)) as success
        by _time, Account_Name, Source_Network_Address
| where failed >= 15 AND success >= 1
```

Refinements worth mentioning aloud: exclude known service accounts and vulnerability scanners; treat a **low-and-slow spray** differently — many accounts from one IP with only a few attempts each is password spraying, which this threshold-per-account rule will miss entirely; and always check whether MFA was satisfied on the success, because a successful password with a failed MFA is a very different outcome from a fully successful sign-in.

**Key terms to mention:** join on entity, time window, threshold, failure result code, password spraying versus brute force, MFA outcome on the success event.

**Weak answer to avoid:** Writing only "search for event ID 4625." Counting failures alone finds noise; the value is in the join to the success.

**Likely follow-up:** "Modify that logic to detect password spraying instead."

---

### Q22. Explain the Splunk concepts you would use daily — index, sourcetype, and how you structure a search.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you can work in a Splunk-based SOC, and whether you search efficiently.

**Model answer (say this aloud):**
> An index is the data store where events live, so specifying the index first is the biggest performance decision in the search. Sourcetype describes the format of the data and drives how it is parsed into fields. A good search goes narrow first and wide later: index, sourcetype and time range at the front, then filtering terms, then transforming commands like stats or table at the end. So I always start with the smallest possible time window and the specific index rather than searching everything, because an unbounded search is slow and in a live incident slow is expensive.

**Deeper explanation:**
Search structure follows the pipeline model: `index=... sourcetype=... earliest=... | filtering | transforming | formatting`. Key commands for an L2: `stats` for aggregation, `table` and `fields` for output shaping, `eval` for derived fields, `rex` for on-the-fly extraction, `transaction` or `stats` grouping for sessionising, `tstats` for fast searches over accelerated data models, and `lookup` for enrichment against asset or intelligence lists. Performance principle: filter in the base search rather than after the first pipe, because everything before the first pipe uses the index, and everything after it processes events one by one. Splunk's Common Information Model provides normalised field names across sources, which is what makes cross-source correlation possible.

**Key terms to mention:** index, sourcetype, search-time field extraction, stats, tstats, eval, rex, lookup, CIM, base search performance.

**Weak answer to avoid:** Writing searches with no index and no time bound. In an interview it signals you have not used the tool at scale.

**Likely follow-up:** "Your search takes eight minutes to return. How do you speed it up?"

---

### Q23. `Scenario-based` You arrive for your shift and alert volume has tripled overnight with no obvious incident. How do you handle it?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you can stay safe while drowning, and whether you look for the cause instead of just working faster.

**Model answer (say this aloud):**
> I do two things at once: I keep triage safe, and I find the cause. To keep triage safe, I do not work the queue chronologically — I sort by severity and by asset criticality so the alerts that matter are seen first, and I get help from the shift lead early rather than trying to clear it alone. To find the cause, I look at whether the spike comes from one rule, one source, or one asset, because that answers the question in a minute. A spike is usually a change, a broken parser, a scanner, or a genuine campaign — and I must not assume it is noise until I have checked, because a real attack is the one case where a flood is deliberate cover.

**Scenario walkthrough**

**Initial alert or situation**
Alert queue volume tripled overnight with no declared incident.

**Investigation steps, in order**
1. Group the alerts by detection rule and count them — a single rule producing most of the volume points straight at a tuning or parser problem.
2. Group by source host, source IP and account — one noisy asset or one scanning source explains many spikes.
3. Check the change record for the previous 24 hours: rule changes, new log sources, agent upgrades, network changes, new deployments.
4. Sample five alerts from the largest cluster and validate them properly — are they genuinely false, or is something real happening at scale.
5. Separately scan the queue for high-severity alerts on critical assets that could be buried in the noise.
6. Check whether the spike coincides with a genuine external event, such as an active exploitation campaign against something you run.

**Evidence and log sources to review**
Alert metadata grouped by rule, entity and time; SIEM rule change history; change management records; ingestion volume per source; and the raw events behind a sample of the alerts.

**Severity and business impact reasoning**
The spike itself is an operational risk because it degrades detection — real incidents get lost. If the cause is a genuine campaign, severity is driven by the campaign. Either way, treat "we cannot triage the queue" as a reportable condition, not a private struggle.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Reprioritise the queue by severity and asset criticality | Bulk-closing alerts without validating a sample |
| Validate a representative sample from each cluster | Disabling a detection rule to stop the noise |
| Request additional analyst support from the shift lead | Suppressing alerts on a critical asset class |
| Apply a narrow, documented, time-limited suppression once the cause is proven benign | Permanent exclusions without review |

**Escalation and communication**
Tell the shift lead within the first thirty minutes: volume, dominant cause, what you have prioritised, and what support you need. If the cause is a change made by another team, raise it through the proper channel rather than reversing their change yourself.

**Recovery, lessons learned, detection improvement**
Recovery: fix the root cause — tune the rule, fix the parser, or handle the campaign. Lessons learned: a change was made without assessing detection impact. Improvement: require detection-impact review for changes affecting monitored systems, and build an alert-volume anomaly monitor so the SOC learns about a spike from a dashboard, not from a drowning analyst.

**Say this aloud to the interviewer:**
> "I would keep triage safe and find the cause in parallel — reprioritise by severity and asset criticality rather than working chronologically, group the alerts by rule and entity to find the cause in minutes, validate a sample before assuming it is noise, and tell the shift lead early that I need support."

**Key terms to mention:** alert clustering by rule and entity, sampling before bulk action, time-limited suppression, alert fatigue, detection degradation, change impact review.

**Weak answer to avoid:** "I would bulk close them to clear the queue." Bulk-closing unvalidated alerts is how real intrusions get missed, and it is unrecoverable once done.

**Likely follow-up:** "How would you tell the difference between noise and an attacker deliberately generating noise as cover?"

---

### Q24. If you were setting up monitoring from scratch, which log sources would you insist on first?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Priorities — whether you know what actually detects attacks.

**Model answer (say this aloud):**
> I would prioritise by attack path, not by what is easy to collect. First, identity: authentication logs from the directory and the cloud identity provider, because almost every modern intrusion involves credentials. Second, endpoint: EDR process telemetry and Windows security and PowerShell logs, because that is where execution and persistence appear. Third, network egress: firewall, proxy and DNS, because that is where command and control and exfiltration show up. Fourth, email, since it is still the most common initial access route. Then cloud control plane and audit logs for administrative changes. After that, servers and applications by criticality. The principle is to cover the stages of an intrusion rather than to collect the most convenient sources.

**Deeper explanation:**

| Priority | Source | What it detects |
|---|---|---|
| 1 | Domain controller security logs, cloud sign-in and audit logs | Brute force, spraying, Kerberos abuse, privilege changes, account creation |
| 2 | EDR process telemetry, Windows Security and PowerShell logs, Sysmon | Execution, persistence, credential dumping, lateral movement |
| 3 | Firewall, proxy, DNS | C2 beaconing, exfiltration, malicious destinations |
| 4 | Email gateway and mailbox audit | Phishing, BEC, forwarding rules, mailbox delegation |
| 5 | Cloud control plane audit | Role assignments, key and secret changes, resource abuse |
| 6 | VPN and remote access | External access, impossible travel, unusual sessions |

Mention retention explicitly: authentication and endpoint telemetry need long retention because compromises are frequently discovered months late, and short retention silently caps how far back an investigation can reach.

**Key terms to mention:** identity-first, egress visibility, process telemetry, control plane audit, retention period, coverage by attack stage, dwell time.

**Weak answer to avoid:** "Everything." Unlimited collection is neither affordable nor useful; prioritisation is the skill being tested.

**Likely follow-up:** "You have one month of retention and an intrusion started four months ago. What now?"

---

### Q25. What is the difference between a correlation rule and a behavioural or anomaly-based detection, and when do you use each?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Detection strategy and awareness of each approach's failure mode.

**Model answer (say this aloud):**
> A correlation rule is deterministic — it fires when a defined set of conditions occurs together, so it is precise, explainable, and easy to tune, but it only catches what you thought to describe. A behavioural or anomaly detection compares activity against a learned baseline for that user, host or peer group and flags deviation, so it can catch things nobody wrote a rule for, but it is harder to explain and it fails badly if the baseline period contained the attack or if the environment changes. I use correlation for known attack techniques where I can state the logic clearly, and behavioural detection for things like unusual data volumes, first-time access to sensitive systems, or an account suddenly behaving unlike its peers. In practice the strongest detections combine them — a behavioural signal that only alerts when it also involves a privileged account or a sensitive asset.

**Deeper explanation:**
Correlation rules map cleanly to ATT&CK techniques and give reproducible results, which matters for audit and for tuning. Behavioural analytics — user and entity behaviour analytics — build profiles over time and score deviations; they excel at insider threat, compromised-account detection, and data exfiltration where no single event is malicious. Failure modes to name: correlation misses novel techniques and can be evaded by small variations; behavioural detection produces unexplainable alerts, drifts with organisational change, and can normalise an attacker who was present during baselining. Sound practice is layering: signature and correlation for known-bad, behavioural for unknown-bad, and threat hunting for what neither catches.

**Key terms to mention:** deterministic logic, baseline, peer group analysis, UEBA, risk scoring, explainability, baseline poisoning, layered detection.

**Weak answer to avoid:** Claiming machine learning replaces rules. Interviewers hear this as marketing rather than experience.

**Likely follow-up:** "An anomaly alert says a user is behaving unusually and gives no detail. How do you triage it?"

---

### Q26. `Scenario-based` Management asks you to prove that the SOC would detect a specific attack technique. How do you answer that?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you can evidence detection coverage instead of asserting it.

**Model answer (say this aloud):**
> I would not answer from memory or from the rule names, because a rule existing does not prove it fires. I would answer in three layers: first, do we collect the telemetry that this technique produces — if not, the answer is no regardless of rules. Second, is there a detection rule mapped to that technique, and is it enabled and healthy. Third, has it actually fired in a controlled, authorised test. Then I give management a clear, honest answer with evidence: detected, partially detected — meaning we would see it only after a later stage — or not detected, and what it would take to close the gap. An honest gap stated clearly is far more valuable to them than a confident yes.

**Scenario walkthrough**

**Initial alert or situation**
A management request to evidence detection coverage for a named technique, for example credential dumping from LSASS.

**Investigation steps, in order**
1. Map the technique to the specific observable artefacts it produces — process access to a protected process, particular command lines, driver loads, file writes, network patterns.
2. Confirm the telemetry exists: query the relevant tables for those artefact types over the last 30 days and verify the fields are populated across the estate, not just on a few pilot hosts.
3. Inventory the detection rules mapped to that technique and confirm each is enabled, scheduled, and has fired at least once historically.
4. Check coverage breadth: what percentage of endpoints and servers actually report the required telemetry.
5. Arrange an authorised, scheduled validation test in an approved scope with the appropriate written approval, and observe whether the alert fires and how long it takes.
6. Record the result with evidence: query outputs, rule identifiers, alert screenshots, and timings.

**Evidence and log sources to review**
Telemetry tables for the relevant artefacts, EDR sensor coverage and health reports, the analytics rule inventory with ATT&CK mappings, historical alert records, and the validation test results.

**Severity and business impact reasoning**
This is a risk-assurance activity rather than an incident, but discovered gaps are risk findings and should be recorded with a severity and an owner. A gap on a technique used against your sector is a High finding.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Query telemetry and inventory rules | Executing any attack simulation, however safe |
| Produce the coverage assessment and gap list | Deploying new detection rules to production |
| Recommend specific detection improvements | Changing endpoint policy or sensor configuration |
| Run passive validation using historical data | Testing on production systems without written approval |

**Escalation and communication**
Answer management in their language: "We would detect this at the execution stage within minutes on covered endpoints; 12% of servers do not report the required telemetry, so on those we would only detect a later stage." Give a defined coverage figure and a remediation recommendation with effort and owner. Never say "we are covered" without evidence.

**Recovery, lessons learned, detection improvement**
Outcome should be a documented coverage assessment, a prioritised gap list, and a repeatable validation cycle — coverage checked periodically and after major changes, rather than only when someone asks.

**Say this aloud to the interviewer:**
> "I would answer in three layers — do we have the telemetry, is there an enabled rule mapped to that technique, and has it fired in an authorised controlled test. Then I would give a clear answer with a coverage percentage and a named gap, because an honest gap is more useful to management than a confident yes."

**Key terms to mention:** ATT&CK coverage mapping, telemetry validation, sensor coverage percentage, authorised validation testing, detection assurance, evidenced answer.

**Weak answer to avoid:** "Yes, we have a rule for that." Rules can be disabled, broken, or starved of data.

**Likely follow-up:** "What approvals would you need before running any simulation?"

---

### Q27. `Scenario-based` The SIEM raises a high-severity alert, but when you open the raw log the details do not match the alert at all. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you trust the tool over the evidence, and whether you can diagnose data-quality problems.

**Model answer (say this aloud):**
> The raw log wins. The alert is an interpretation and the log is the evidence, so if they disagree, something in the pipeline is wrong and I investigate the pipeline as well as the alert. Usually it is a parsing or field-mapping problem — a value landing in the wrong field, a timezone mismatch, or a truncated field. I confirm by looking at several events from the same source, not just one. Crucially, I do not simply close the alert as a false positive, because a mis-parsed field can equally mean other alerts are being missed. So I raise it as a data quality issue as well as resolving the individual alert.

**Scenario walkthrough**

**Initial alert or situation**
A high-severity alert whose fields contradict the underlying raw event.

**Investigation steps, in order**
1. Retrieve the exact raw event the alert was generated from, using the alert's event identifier rather than a re-run of the query.
2. Compare each alert field with the raw field to identify which specific fields are wrong or shifted.
3. Pull ten other events from the same source and confirm whether the mismatch is systematic or specific to this event.
4. Check timestamps and timezone handling — a source sending local time interpreted as UTC produces alerts that appear to describe the wrong events entirely.
5. Check whether the source recently changed format, for example after a vendor upgrade that added or reordered fields.
6. Determine the real security question independently: using the raw data only, did anything malicious actually happen.

**Evidence and log sources to review**
The raw event with its original identifier, the parser or connector configuration, ingestion timestamps versus event timestamps, the detection rule logic, and change records for the source system.

**Severity and business impact reasoning**
Two separate impacts: the alert itself may be a false positive, which is minor; the parsing defect is potentially a broad detection failure across every rule using that source, which is a High operational risk.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Investigate the underlying activity using raw data | Editing production parsers or connector configuration |
| Document the parsing defect with examples | Disabling the affected rule while it is broken |
| Notify the platform or engineering team | Accepting the defect without remediation |
| Write a temporary hunting query against raw data | Deploying a corrected parser to production |

**Escalation and communication**
Report separately: the alert disposition, and the data quality defect with the list of affected detections. Engineers need the concrete examples — raw event, expected parse, actual parse — not just "the parser is broken."

**Recovery, lessons learned, detection improvement**
Recovery: fix the parser, revalidate the affected rules, and retro-hunt over the affected period for anything missed. Lessons learned: a source format change reached production without parser regression testing. Improvement: add parser validation to the change process and add a data-quality monitor that samples key fields for null or malformed values.

**Say this aloud to the interviewer:**
> "The raw log wins — the alert is an interpretation and the log is the evidence. I would prove whether the mismatch is systematic across the source, answer the security question from raw data, and raise the parsing defect separately, because a broken field means other alerts may be silently missing."

**Key terms to mention:** raw event as ground truth, field mapping, timezone handling, systematic versus single-event defect, data quality monitoring, retro-hunt, parser regression testing.

**Weak answer to avoid:** "The SIEM is wrong so I closed the alert." Closing it without investigating the pipeline leaves a silent detection failure in place.

**Likely follow-up:** "How would you find out which other detections use that same broken field?"

---

[⬅ Previous: Triage & IR](01-l2-soc-triage-and-incident-response.md) · [Back to README](../README.md) · [Next: Windows & AD ➡](03-windows-active-directory-and-identity.md)
