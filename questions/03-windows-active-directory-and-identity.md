# 03 · Windows, Active Directory and Identity

**10 questions · Q28–Q37 · 4 scenario-based**

[⬅ Previous: SIEM & Detection](02-siem-logging-and-detection-engineering.md) · [Back to README](../README.md) · [Next: Network Security ➡](04-network-security-and-traffic-analysis.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q28 | The Windows Event IDs you use daily | Core | Standard |
| Q29 | Logon types and why they matter | Core | Standard |
| Q30 | Failed logon storm then success on a DC | Advanced | Scenario-based |
| Q31 | Kerberoasting — mechanism and detection | Advanced | Standard |
| Q32 | Pass-the-hash, pass-the-ticket, Golden Ticket | Advanced | Standard |
| Q33 | New account added to Domain Admins at 03:00 | Advanced | Scenario-based |
| Q34 | Investigating a suspect service account | Advanced | Standard |
| Q35 | LSASS memory access alert | Advanced | Scenario-based |
| Q36 | DCSync — what it is and how to catch it | Advanced | Standard |
| Q37 | Repeated account lockouts | Core | Scenario-based |

---

### Q28. Which Windows Event IDs do you rely on most, and what does each one tell you?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Practical Windows investigation fluency. This is very commonly asked and easy to score well on.

**Model answer (say this aloud):**
> The ones I use constantly are 4624 for a successful logon and 4625 for a failed logon, and with both the logon type field is what tells me what actually happened. 4672 tells me a logon was granted administrative privileges. 4688 gives me process creation with the command line, which is the single most useful endpoint event. 4720 is account creation and 4728 or 4732 is a user being added to a privileged group. 4740 is an account lockout. On the Kerberos side, 4768 is a ticket-granting ticket request, 4769 is a service ticket request, and 4771 is pre-authentication failure. 7045 in the system log is a new service installed, which is a classic persistence and lateral movement artefact. And 1102 means the security log was cleared, which I treat as an incident on its own.

**Deeper explanation:**

| Event ID | Log | Meaning | Why it matters in an investigation |
|---|---|---|---|
| 4624 | Security | Successful logon | Logon type and source IP reveal how access was gained |
| 4625 | Security | Failed logon | Brute force, spraying, misconfigured service |
| 4634 / 4647 | Security | Logoff / user-initiated logoff | Session duration for timeline building |
| 4648 | Security | Logon using explicit credentials | Runas and lateral movement with alternate credentials |
| 4672 | Security | Special privileges assigned | Administrative logon occurred |
| 4688 | Security | Process creation | Command line, parent process — core execution evidence |
| 4697 | Security | Service installed | Persistence and remote execution |
| 4720 / 4726 | Security | Account created / deleted | Attacker-created accounts |
| 4722 / 4724 / 4738 | Security | Account enabled / password reset / account changed | Account manipulation |
| 4728 / 4732 / 4756 | Security | Member added to global / local / universal group | Privilege escalation |
| 4740 | Security | Account locked out | Brute force or bad cached credential |
| 4768 / 4769 / 4771 | Security (DC) | TGT request / service ticket request / pre-auth failure | Kerberos abuse, Kerberoasting, spraying |
| 4776 | Security | NTLM credential validation | NTLM authentication and legacy protocol use |
| 4662 | Security (DC) | Operation on a directory object | DCSync when replication rights are used |
| 5140 / 5145 | Security | Network share accessed / detailed share access | Lateral movement, data staging |
| 1102 | Security | Audit log cleared | Anti-forensics — treat as an incident |
| 7045 | System | New service installed | Persistence, remote execution tools |
| 4104 | PowerShell Operational | Script block logging | The actual PowerShell code that ran, decoded |

Add that **Sysmon** enriches this heavily where deployed: Event ID 1 process creation with hashes, 3 network connection, 7 image loaded, 8 remote thread creation, 10 process access (used to catch LSASS access), 11 file creation, 13 registry value set, and 22 DNS query.

**Key terms to mention:** logon type, parent process, command line auditing, script block logging, Sysmon, audit policy, anti-forensics.

**Weak answer to avoid:** Reciting numbers without saying what you would do with them. Always pair the ID with the investigative use.

**Likely follow-up:** "You see 4688 but the command line field is empty. Why, and what do you do?"

---

### Q29. Explain Windows logon types and why they matter in an investigation.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Depth on the single most important field in Windows authentication events.

**Model answer (say this aloud):**
> The logon type tells me how the account authenticated, which is often more important than the fact that it authenticated. Type 2 is interactive at the console. Type 3 is network, which is what you see for file shares and most remote access. Type 4 is batch, usually scheduled tasks. Type 5 is a service starting. Type 7 is unlocking a workstation. Type 8 is network logon with cleartext credentials, which usually points at older applications. Type 9 is new credentials, which is what runas with network credentials produces and is a strong lateral movement indicator. Type 10 is remote interactive, meaning RDP. Type 11 is cached interactive, meaning the domain controller was not reachable. In practice, type 10 on a server at an unusual hour and type 9 on a workstation are the two that make me look closely.

**Deeper explanation:**

| Type | Name | Typical cause | Investigative significance |
|---|---|---|---|
| 2 | Interactive | Console logon | Physical or virtual console access |
| 3 | Network | SMB, remote admin, authenticated API | Most lateral movement appears here |
| 4 | Batch | Scheduled task | Persistence via scheduled tasks |
| 5 | Service | Service account starting a service | Service creation for persistence |
| 7 | Unlock | Workstation unlock | Presence of the user at the machine |
| 8 | NetworkCleartext | Basic auth over the wire | Credential exposure risk |
| 9 | NewCredentials | `runas /netonly` | Strong lateral movement indicator |
| 10 | RemoteInteractive | RDP | Hands-on-keyboard remote access |
| 11 | CachedInteractive | Cached domain credentials | Logon while DC unreachable |

Two practical pivots: for type 3 and 10 always take the **source IP or workstation name** and ask whether that machine has any business reason to reach this target; and check the **authentication package** — NTLM appearing where Kerberos is expected can indicate pass-the-hash or a deliberate downgrade.

**Key terms to mention:** logon type, source workstation, authentication package, NTLM versus Kerberos, `runas /netonly`, cached credentials.

**Weak answer to avoid:** "A logon is a logon." The type is what distinguishes a user sitting down at their desk from an attacker moving laterally.

**Likely follow-up:** "You see logon type 3 with NTLM to five servers from one workstation in two minutes. What is your hypothesis?"

---

### Q30. `Scenario-based` A domain controller shows 400 failed logons for different usernames from one internal IP in ten minutes, followed by one successful logon. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Recognising password spraying from inside the network and responding to a successful compromise.

**Model answer (say this aloud):**
> Many usernames from one source is password spraying, not brute force — the attacker is trying one or two passwords against many accounts to avoid lockouts. The critical fact is that it came from an internal IP, which means something inside the network is already compromised and is now enumerating credentials. And there is a success at the end, so I have to assume that account is compromised. So I work three things at once: identify and contain the source host, contain the successful account, and determine what that account can reach. This is a High or Critical incident from the first minute because it indicates an attacker already has an internal foothold.

**Scenario walkthrough**

**Initial alert or situation**
High volume of failed authentications across many distinct accounts from a single internal source, ending in a success.

**Investigation steps, in order**
1. Identify the source host from the IP — asset inventory, DHCP records, and the workstation name in the events — and determine its owner and role.
2. On the source host, identify the process performing the authentications: process name, full command line, parent process, and the user context it ran under.
3. Determine how that host was compromised — first suspicious process, recent downloads, email, or a prior alert on the same machine.
4. Confirm the successful account: which account, what privileges it holds, what logon type, and what it did after authenticating.
5. Check whether the successful account authenticated anywhere else, and whether the attacker moved on to a second host.
6. Determine the scope of accounts targeted — the account list itself may reveal how the attacker enumerated the directory.
7. Check for follow-on activity: service creation, scheduled tasks, share access, or privilege changes.

**Evidence and log sources to review**
Domain controller security logs for 4625, 4768 and 4771 grouped by source; 4624 for the success with its logon type; EDR process and network telemetry on the source host; directory enumeration activity; the successful account's subsequent authentications and share access events; DHCP and asset inventory.

**Severity and business impact reasoning**
Critical. Internal password spraying means an existing foothold and active credential access. If the successful account is privileged or has access to sensitive systems, this can progress to domain compromise quickly. Impact spans confidentiality and integrity of directory-controlled resources.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the source host via EDR | Disabling a privileged or service account |
| Revoke sessions and force password reset on the compromised account | Domain-wide password reset |
| Preserve volatile evidence on the source host | Blocking authentication from a network segment |
| Sweep for the same behaviour from other sources | Taking a domain controller offline |

**Escalation and communication**
Escalate immediately as a suspected internal compromise with credential access. Notify the identity and Active Directory team so they can watch for privilege changes. State clearly which account succeeded and what it can access, since that drives the urgency for management.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the source host, rotate the compromised credentials, verify no persistence was established, and monitor the targeted accounts for a defined period. Lessons learned: how did the source host get compromised, and why did internal spraying reach 400 attempts before anyone acted. Detection improvement: alert on a single source producing failed authentications against a high count of distinct accounts within a short window — distinct-account count is the correct signal for spraying, not failure count per account — and alert on any success from a source that has just sprayed.

**Say this aloud to the interviewer:**
> "Many usernames from one internal source is spraying, and internal means we already have a foothold. I would contain the source host, contain the account that succeeded, and map what that account can reach — all three in parallel, and escalate immediately because the success makes this a live compromise."

**Key terms to mention:** password spraying versus brute force, distinct-account count, internal foothold, credential access, lockout threshold evasion, lateral movement.

**Weak answer to avoid:** "It is a brute force attack, I would block the IP." Blocking an internal IP without investigating the compromised host leaves the attacker in place.

**Likely follow-up:** "Why would an attacker use spraying instead of brute force?"

---

### Q31. What is Kerberoasting and how would you detect it?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Real Active Directory attack knowledge, not textbook definitions.

**Model answer (say this aloud):**
> Kerberoasting abuses a normal feature of Kerberos. Any authenticated domain user can request a service ticket for any account that has a service principal name registered. That ticket is encrypted with the service account's password hash, so the attacker requests tickets, takes them offline, and cracks them at their own pace to recover the plaintext password. It is attractive because it needs no special privileges and generates no failed logons. I detect it by looking at service ticket requests — event 4769 — and looking for one account requesting tickets for an unusual number of distinct services in a short window, especially when the encryption type is the weaker RC4 rather than AES, because attackers often request RC4 deliberately as it cracks faster.

**Deeper explanation:**
Mechanism: the attacker enumerates accounts with a service principal name, requests TGS tickets with a standard API call, exports them, then cracks offline. Detection signals in order of value: (1) a single user requesting 4769 events for many distinct service names in a short window; (2) ticket encryption type `0x17` (RC4-HMAC) where the environment normally uses `0x12` (AES256); (3) requests for service accounts that the requesting user has no business reason to use; (4) enumeration activity — LDAP queries filtering on servicePrincipalName — just before the requests. Prevention matters and is worth mentioning: long random passwords or group managed service accounts for any account with an SPN, disabling RC4 where possible, and not granting SPNs to privileged accounts.

Detection sketch:

```kusto
SecurityEvent
| where EventID == 4769
| where TicketEncryptionType == "0x17"          // RC4 — weaker, preferred by attackers
| where ServiceName !endswith "$"                // exclude machine accounts
| summarize DistinctServices = dcount(ServiceName),
            Requests = count(),
            Services = make_set(ServiceName, 20)
          by Account, bin(TimeGenerated, 15m)
| where DistinctServices >= 8
```

**Key terms to mention:** service principal name, TGS request, event 4769, RC4 versus AES encryption type, offline cracking, group managed service accounts, no failed logons generated.

**Weak answer to avoid:** "It is an attack on Kerberos." Name the SPN and the offline cracking — that is what shows real understanding.

**Likely follow-up:** "How would you find which accounts in the domain are most at risk of Kerberoasting?"

---

### Q32. Explain pass-the-hash, pass-the-ticket and Golden Ticket, and how you would detect each.

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Credential attack depth — a strong differentiator for L2.

**Model answer (say this aloud):**
> All three are credential theft techniques where the attacker never needs the plaintext password. Pass-the-hash uses a stolen NTLM hash to authenticate directly, so I look for NTLM network logons from workstations to many systems, especially where Kerberos would be normal. Pass-the-ticket steals an existing Kerberos ticket from memory and reuses it, so I look for tickets used from a host that never requested them and for unusual ticket lifetimes. A Golden Ticket is the most serious: the attacker has stolen the krbtgt account hash and can forge ticket-granting tickets for any user, including accounts that do not exist, with any validity period. The detection signals are service ticket requests with no corresponding ticket-granting ticket request, tickets for non-existent accounts, and anomalous ticket lifetimes. A Golden Ticket means the domain is fully compromised and the recovery is a double krbtgt password reset, which is a major planned operation.

**Deeper explanation:**

| Technique | What is stolen | Key detection signals |
|---|---|---|
| **Pass-the-hash** | NTLM password hash | 4624 logon type 3 with NTLM package from workstations; same account authenticating to many hosts rapidly; 4776 volume anomalies |
| **Overpass-the-hash** | NTLM hash used to obtain Kerberos tickets | 4768 TGT requests with RC4 from a host where AES is normal |
| **Pass-the-ticket** | Kerberos TGT or TGS from memory | Ticket used from a host that never requested it; mismatched source; unusual ticket lifetime |
| **Golden Ticket** | krbtgt hash — forge any TGT | 4769 without a preceding 4768; tickets for non-existent or disabled accounts; ticket lifetime far beyond policy; domain field anomalies |
| **Silver Ticket** | Service account hash — forge a TGS for one service | Service access with no corresponding DC ticket request at all |

Emphasise the response difference: pass-the-hash is contained by isolating hosts and rotating the affected account's credentials, whereas a Golden Ticket requires resetting the krbtgt password **twice**, with the correct interval between resets so replication completes, and it should be treated as a full domain compromise with a planned, coordinated recovery.

**Key terms to mention:** NTLM hash, krbtgt, ticket-granting ticket, ticket lifetime, forged ticket, double krbtgt reset, tier-0 assets, credential hygiene.

**Weak answer to avoid:** Conflating the three. Interviewers listen specifically for whether you know a Golden Ticket involves krbtgt.

**Likely follow-up:** "You suspect a Golden Ticket. What is the first thing you check?"

---

### Q33. `Scenario-based` At 03:00 a new user account is created and added to Domain Admins by an account that belongs to a member of IT staff. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Privilege escalation detection plus professional handling of a possible insider or compromised administrator.

**Model answer (say this aloud):**
> I treat this as high severity immediately, because creating an account and granting it Domain Admin is exactly what an attacker does to establish durable privileged access. I do not accuse anyone — the most likely explanations are a compromised administrator account or unapproved out-of-hours work, and I need evidence before I decide which. So I check whether there is a change request authorising it, I check how that administrator account authenticated at 03:00 and from where, and I check what else that account did around the same time. I escalate to the incident manager straight away because privileged group changes are not something to sit on, and because if it involves a staff member the handling has to follow the proper process rather than my own judgement.

**Scenario walkthrough**

**Initial alert or situation**
Account creation (4720) followed by addition to a privileged group (4728) outside business hours, performed by a staff administrator account.

**Investigation steps, in order**
1. Confirm the exact sequence and timing: creation event, group addition event, the target account name, and the account that performed both.
2. Check for an authorising change request or ticket covering the action.
3. Examine how the administrator account authenticated: time, source IP or workstation, logon type, whether MFA or a privileged access workstation was used, and whether that source is normal for them.
4. Look at everything else that account did in the surrounding hours — other account changes, group changes, share access, remote sessions.
5. Examine the newly created account: naming convention, whether it mimics a service account, password settings, and whether it has already been used to authenticate anywhere.
6. Check the administrator's endpoint for signs of compromise — suspicious processes, credential dumping, remote access tools.
7. Determine whether the administrator was physically or remotely working at that time through independent evidence such as VPN records.

**Evidence and log sources to review**
Domain controller security logs for 4720, 4722, 4724, 4728 and 4672; the administrator account's 4624 events with logon type and source; VPN and remote access logs; EDR telemetry on the administrator's workstation; change management records; and directory replication metadata for who made the change.

**Severity and business impact reasoning**
High to Critical. Domain Admin membership grants control over the entire directory and everything it authenticates. Whether the cause is compromise or unapproved administration, the risk is the same until proven otherwise.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Preserve all evidence and build the timeline | Disabling the new account, if business impact is unknown |
| Raise monitoring on the new account and the admin account | Disabling the administrator's account |
| Isolate the administrator's workstation if it shows compromise indicators | Any action framed as an accusation against staff |
| Verify whether the new account has been used | Contacting the staff member directly |

**Escalation and communication**
Escalate immediately to the incident manager, and follow the organisation's process for anything involving personnel — that usually means the case is handled with restricted distribution and with management deciding who is contacted. Report facts only: what was created, by which account, from where, and whether authorisation exists. Do not speculate about intent in writing.

**Recovery, lessons learned, detection improvement**
If unauthorised: remove the account, rotate the administrator's credentials, review everything the accounts touched, and check for other persistence. Lessons learned: privileged group changes were possible without approval workflow or out-of-hours control. Improvement: real-time alerting on any privileged group modification, require just-in-time privileged access with approval rather than standing membership, and reconcile all privileged group changes against change records daily.

**Say this aloud to the interviewer:**
> "I would treat it as high severity but I would not accuse anyone. The two likely explanations are a compromised admin account or unapproved out-of-hours work, and evidence decides which — the authentication source, the change record, and what else that account did. I would escalate immediately because anything involving a staff member has to be handled through the proper process, not by me."

**Key terms to mention:** privileged group modification, standing versus just-in-time privilege, change authorisation, compromised administrator, personnel-sensitive handling, restricted distribution.

**Weak answer to avoid:** "I would call the administrator and ask him." If the account is compromised you tip off the attacker; if it is an insider you contaminate the investigation. That contact decision belongs to management.

**Likely follow-up:** "The administrator says he did it and forgot to raise a ticket. Are you satisfied?"

---

### Q34. How do you investigate whether a service account has been compromised?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Understanding that service accounts are the most abused and least monitored identities.

**Model answer (say this aloud):**
> Service accounts are predictable, which is what makes them investigable. A service account should authenticate from a fixed set of hosts, to a fixed set of targets, at regular intervals, with a consistent logon type. So I compare current behaviour against that baseline: is it logging on from a host it has never used, is it authenticating interactively when it should only ever be a service or network logon, is it accessing resources it has never touched, and is it being used at times outside its normal pattern. Interactive or RDP logon by a service account is a very strong indicator, because a service account should almost never log on interactively. I also check whether its password was changed recently and whether it has been added to any group.

**Deeper explanation:**
Specific checks: baseline the account's source hosts, destination hosts, logon types, and daily rhythm over 30 days, then look for deviation. Red flags in priority order: interactive (type 2) or remote interactive (type 10) logon; authentication from a workstation rather than a server; a sudden increase in distinct destinations, which suggests it is being used for lateral movement; execution of interactive tooling under the account context; group membership changes; and use outside the service's normal schedule. Also check whether the account is over-privileged in the first place, whether its password is old and non-rotating, and whether it has a service principal name that exposes it to Kerberoasting. Containment is delicate: disabling a service account can break production, so the containment plan must be agreed with the application owner before you act.

**Key terms to mention:** behavioural baseline, logon type anomaly, interactive logon by a service account, distinct destination count, over-privileged service account, group managed service account, coordinated containment with the application owner.

**Weak answer to avoid:** "I would disable the account." Disabling a service account without coordination can cause a larger outage than the incident.

**Likely follow-up:** "The service account is used by a critical application and you suspect compromise. How do you contain it without an outage?"

---

### Q35. `Scenario-based` The EDR raises an alert that a process accessed LSASS memory on a finance workstation. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Credential dumping response — recognising that the damage is to credentials, not just to one host.

**Model answer (say this aloud):**
> LSASS holds credential material in memory, so an unexpected process reading it is credential dumping until proven otherwise. The key insight is that the impact is not limited to this workstation: every credential that was cached on that machine must now be treated as compromised, including any administrator who logged into it recently. So my response has two halves. First, contain the host — isolate it and preserve evidence. Second, identify every account whose credentials could have been in memory and get those rotated, prioritising privileged accounts. I escalate immediately, because credential compromise spreads quietly and the window to act is short.

**Scenario walkthrough**

**Initial alert or situation**
EDR detects process access to LSASS memory with suspicious access rights from a non-standard process.

**Investigation steps, in order**
1. Identify the accessing process: full path, hash, digital signature, parent process, command line, and the account it ran as.
2. Determine whether it is a legitimate tool — some security and backup products access LSASS legitimately, so confirm against the known-good inventory rather than assuming.
3. Establish how the process got on the host and when it first appeared, which points to the initial access.
4. Determine what credentials could have been in LSASS memory: enumerate every account that logged on to this host since its last reboot, and identify which are privileged.
5. Look for exfiltration of the dump — file creation of a large dump file, followed by an upload, an SMB copy, or archiving.
6. Check for lateral movement already using the harvested credentials: unusual logons by those accounts elsewhere in the estate.
7. Sweep the estate for the same process hash and the same behaviour on other hosts.

**Evidence and log sources to review**
EDR process access telemetry (Sysmon Event ID 10 where deployed, with the LSASS target and granted access mask), process creation events with command lines, file creation events for dump files, 4624 history on the host to enumerate exposed accounts, network telemetry for upload, and estate-wide indicator sweeps.

**Severity and business impact reasoning**
Critical if any privileged account logged on to the host; High otherwise. The business impact is potential compromise of every credential exposed on that machine, which is a lateral movement enabler rather than a single-host problem.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the workstation via EDR | Domain-wide or tier-0 credential rotation |
| Preserve memory and disk evidence before changes | Reimaging before evidence collection completes |
| Block the malicious binary hash across the estate | Disabling privileged accounts still in use |
| Sweep for the same indicators elsewhere | Forcing a mass password reset across the organisation |

**Escalation and communication**
Escalate immediately with an explicit list of accounts exposed on that host, prioritised by privilege. That list is the most useful thing you can hand to the incident manager, because it defines the credential rotation scope. Recommend rotation order: tier-0 and domain administrative accounts first, then service accounts used on the host, then standard users.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the host from clean media, rotate exposed credentials in the agreed order, and monitor the affected accounts closely for a defined period. Lessons learned: why did a privileged account log on to a standard workstation, and how did the tool arrive. Improvement: enable credential protection features such as LSASS protected process and Credential Guard where supported, enforce tiered administration so administrators never log on to workstations, and alert on any non-allowlisted process opening a handle to LSASS.

**Say this aloud to the interviewer:**
> "I would treat it as credential dumping until proven otherwise, and the important point is that the blast radius is every credential cached on that machine, not just the host. So I isolate and preserve evidence, then immediately produce the list of accounts that logged on since last reboot so credential rotation can start with the privileged ones."

**Key terms to mention:** LSASS, credential dumping, access mask, cached credentials, tiered administration, Credential Guard, protected process, rotation scope and order.

**Weak answer to avoid:** "I would quarantine the file and close it." Removing the tool does nothing about the credentials it already stole.

**Likely follow-up:** "How do you determine exactly which credentials were exposed on that machine?"

---

### Q36. What is DCSync, why is it dangerous, and how would you detect it?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Awareness of high-impact Active Directory attacks that leave subtle traces.

**Model answer (say this aloud):**
> DCSync abuses the directory replication mechanism. An account that has replication rights can ask a domain controller to send it account data, including password hashes, exactly as another domain controller would. The attacker does not need code execution on the domain controller at all, which is what makes it dangerous and quiet — it looks like normal replication traffic. The most valuable target is the krbtgt account hash, because with it the attacker can forge Golden Tickets. I detect it by watching for directory replication requests coming from anything that is not a domain controller: event 4662 showing access to the replication rights, where the requesting account is not a DC computer account. Any account with those rights that is not a domain controller should be a standing alert.

**Deeper explanation:**
The relevant extended rights are **DS-Replication-Get-Changes** and **DS-Replication-Get-Changes-All**. Detection approach: alert on event 4662 on domain controllers where the properties field references those replication rights and the subject account is not a domain controller computer account or a known replication service. Network-level detection is also possible by watching for the replication protocol from non-DC sources. Preventive control: audit which principals hold replication rights — attackers frequently grant these rights to a normal account as a stealthy backdoor, so periodic review of directory permissions is itself a detection. If DCSync is confirmed against krbtgt, treat it as full domain compromise with the double krbtgt reset and coordinated recovery.

**Key terms to mention:** directory replication rights, DS-Replication-Get-Changes-All, event 4662, non-DC source, krbtgt hash, Golden Ticket, permission audit as detection.

**Weak answer to avoid:** "It is a way to steal passwords from AD." Correct but shallow — name the replication rights and the non-DC-source detection.

**Likely follow-up:** "How would you audit who currently holds replication rights?"

---

### Q37. `Scenario-based` A user reports that their account keeps locking out several times a day. How do you investigate?

- **Difficulty:** Core
- **Type:** Scenario-based
- **What the interviewer is testing:** Methodical everyday investigation — and whether you check for attack before assuming user error.

**Model answer (say this aloud):**
> Lockouts are usually caused by a stale cached credential somewhere, but they can also be an attacker spraying that account, so I check both. The method is to find the source. The lockout event on the domain controller records the calling computer, and the failed logon events tell me which machine and which logon type the bad password attempts came from. If the source is the user's own devices and the pattern is regular, it is almost always a saved credential — a mapped drive, a scheduled task, a service, or a phone with an old password. If the source is a machine the user does not use, or an external address, then it is an attack and the ticket becomes a security incident rather than a support issue.

**Scenario walkthrough**

**Initial alert or situation**
Repeated account lockouts for a single user across the day.

**Investigation steps, in order**
1. Pull lockout events (4740) on the domain controllers and note the caller computer name recorded in each.
2. Pull the failed authentication events (4625, plus 4771 and 4776) for that account and group them by source workstation, source IP, and logon type.
3. Determine the interval pattern: precisely regular intervals point to a scheduled task or service; irregular bursts point at a human or a tool.
4. If the source is the user's own machine, enumerate stored credentials, mapped drives, scheduled tasks, and services running under that account.
5. Check mobile device and mail client sessions, which are a very common cause after a password change.
6. Check whether other accounts are locking out from the same source — several accounts means spraying, not a stale credential.
7. Check whether any authentication for that account succeeded from an unexpected source during the same period.

**Evidence and log sources to review**
Domain controller security logs (4740, 4625, 4771, 4776) with caller computer and source address; the user's endpoint credential manager, scheduled tasks and services; mail and mobile device access logs; and authentication logs for other accounts from the same source.

**Severity and business impact reasoning**
Low if the cause is a stale cached credential — an availability annoyance for one user. High if the source is unexpected or if multiple accounts are affected, because that is a credential attack. Do not classify until the source is identified.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Identify the source and advise the user to clear stored credentials | Blocking a source used by legitimate business systems |
| Force a password reset if compromise is suspected | Changing domain lockout policy |
| Check other accounts for the same source | Disabling the user's account during business hours |

**Escalation and communication**
If it is a stale credential, resolve it with the user and IT and close it with the cause documented. If it is an attack, convert it to a security incident, escalate, and check for successful authentications immediately. Tell the user what to do in plain language rather than technical detail.

**Recovery, lessons learned, detection improvement**
Recovery: clear the offending stored credential or contain the attack source. Lessons learned: repeated lockouts were treated as a helpdesk annoyance rather than checked for spraying. Improvement: alert when a single source causes lockouts across multiple distinct accounts, and enrich lockout alerts automatically with the caller computer name so the source is visible without manual lookup.

**Say this aloud to the interviewer:**
> "Lockouts are usually a stale saved credential, but they can be spraying, so I find the source first — the caller computer in the lockout event and the source in the failed logons. Own device with a regular interval means a saved credential. Unknown source, or several accounts from the same source, and it becomes a security incident."

**Key terms to mention:** event 4740, caller computer name, stale cached credential, interval analysis, multiple accounts from one source, spraying, lockout threshold.

**Weak answer to avoid:** "I would just reset the password." That resolves nothing if the source keeps presenting the old password, and it misses an attack entirely.

**Likely follow-up:** "The lockouts continue after a password reset. What does that tell you?"

---

[⬅ Previous: SIEM & Detection](02-siem-logging-and-detection-engineering.md) · [Back to README](../README.md) · [Next: Network Security ➡](04-network-security-and-traffic-analysis.md)
