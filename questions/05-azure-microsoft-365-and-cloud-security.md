# 05 · Azure, Microsoft 365 and Cloud Security

**10 questions · Q48–Q57 · 6 scenario-based**

[⬅ Previous: Network Security](04-network-security-and-traffic-analysis.md) · [Back to README](../README.md) · [Next: EDR & Malware ➡](06-edr-malware-and-endpoint-security.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q48 | Investigating a risky sign-in in Entra ID | Advanced | Scenario-based |
| Q49 | Impossible travel — real vs false positive | Advanced | Scenario-based |
| Q50 | MFA fatigue attack — recognise and respond | Advanced | Scenario-based |
| Q51 | OAuth consent phishing / illicit consent grant | Advanced | Scenario-based |
| Q52 | Conditional Access — what it is, how it helps triage | Core | Standard |
| Q53 | Legacy authentication risk | Core | Standard |
| Q54 | Compromised mailbox with a hidden forwarding rule | Advanced | Scenario-based |
| Q55 | Azure activity log — a new Owner role assignment | Advanced | Scenario-based |
| Q56 | Shared responsibility model in Azure | Core | Standard |
| Q57 | Key KQL tables for Entra/M365 investigations | Core | Standard |

---

### Q48. `Scenario-based` Microsoft Entra ID flags a "risky sign-in" for a user. Walk me through your investigation.

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you can work Entra ID's risk signals methodically rather than just clicking "confirm compromised."

**Model answer (say this aloud):**
> I start by reading exactly what risk detection fired — anonymous IP, unfamiliar sign-in properties, password spray, leaked credentials, or something else — because each implies a different investigation. Then I check the sign-in details: source IP and its reputation, location, device, browser, and whether it was interactive or a background token refresh. I check whether MFA was satisfied, and specifically how — a real user approving a push is very different from a sign-in that succeeded through a legacy protocol with no MFA at all. Then I check what happened after sign-in: any new mailbox rules, new registered devices, or app consents. Based on all of that I either confirm compromise and contain, or dismiss the risk with a documented reason.

**Scenario walkthrough**

**Initial alert or situation**
Entra ID Identity Protection raises a risky sign-in alert for a user account.

**Investigation steps, in order**
1. Identify the specific risk detection type that triggered — this determines what evidence is relevant.
2. Review sign-in log details: IP address and its ASN/reputation, geolocation, device compliance state, browser or client app, and whether the sign-in was interactive or non-interactive.
3. Check the authentication requirement satisfied and the MFA method used — push, code, phone call — and whether it was legacy authentication that bypassed modern MFA entirely.
4. Compare against the user's normal sign-in pattern: usual countries, usual devices, usual hours.
5. Check post-sign-in activity: new inbox rules, forwarding, newly registered MFA methods or devices, and new OAuth app consents.
6. Check the user's risk history — is this a first-time flag or a repeat, and what is the user's current risk level in Identity Protection.
7. If genuinely uncertain, contact the user through an out-of-band channel — not by replying to any email associated with the session — to confirm.

**Evidence and log sources to review**
Entra ID sign-in logs (interactive and non-interactive), Identity Protection risk detections, audit logs for mailbox rule and app consent changes, device compliance data, and the user's historical sign-in baseline.

**Severity and business impact reasoning**
Escalates from Low to High based on: privileged account involvement, MFA bypass via legacy auth, post-sign-in persistence artefacts such as inbox rules, and a source IP with genuine malicious reputation rather than just an unfamiliar location.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke the user's active sessions per standing procedure | Disabling the account, especially if privileged |
| Force MFA re-registration and password reset | Blocking a whole country or IP range |
| Remove a malicious inbox rule after preserving it as evidence | Removing legitimate business Conditional Access exceptions |
| Confirm with the user via a separate verified channel | |

**Escalation and communication**
Escalate confirmed compromises immediately with the specific evidence: which risk detection, which MFA gap, what persistence was found. State plainly whether legacy authentication was involved, since that is usually the fixable root cause.

**Recovery, lessons learned, detection improvement**
Recovery: sessions revoked, credentials rotated, malicious artefacts removed, and the account monitored at elevated sensitivity for a period. Lessons learned: was legacy authentication still enabled for this user. Detection improvement: enforce Conditional Access blocking legacy authentication entirely, and require sign-in risk plus user risk together to drive automatic session revocation for high-confidence detections.

**Say this aloud to the interviewer:**
> "I would read the specific risk detection first, since it tells me what to check. Then I would look at the source IP, device, and how MFA was actually satisfied, and check post-sign-in activity like new inbox rules or app consents for persistence. Legacy authentication satisfying the sign-in with no real MFA is the strongest single indicator of compromise I look for."

**Key terms to mention:** Identity Protection risk detection type, non-interactive sign-in, legacy authentication, MFA method verification, inbox rule persistence, out-of-band verification.

**Weak answer to avoid:** "I would reset the password and close it." Skips verifying whether MFA was actually satisfied and whether persistence was already established — both of which change the entire response.

**Likely follow-up:** "The sign-in used legacy authentication with no MFA prompt at all. What does that tell you about your environment's exposure?"

---

### Q49. `Scenario-based` A sign-in appears from Doha at 09:00 and from a different country at 09:20 for the same user. Is this impossible travel, and how do you confirm it?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Whether you validate the physics before declaring compromise — a very common trick question.

**Model answer (say this aloud):**
> Twenty minutes is not enough time to physically travel between most countries, so on the surface this looks like impossible travel. But before I call it a compromise I check for the common legitimate causes: was one of the sign-ins through a VPN or corporate proxy that exits in a different country than the user's real location, was it a non-interactive token refresh rather than a fresh interactive logon, and does the geolocation data come from a reliable source or a shared corporate egress IP that Entra ID has mislabeled. If both sign-ins are genuinely interactive, from different devices, with no shared corporate infrastructure explanation, I treat it as compromise.

**Scenario walkthrough**

**Initial alert or situation**
Two sign-ins for the same user twenty minutes apart from geographically distant locations.

**Investigation steps, in order**
1. Confirm both events are interactive sign-ins, not one interactive and one automatic token refresh, which can appear from a background service with a different apparent location.
2. Check whether either IP belongs to known corporate VPN, proxy, or cloud infrastructure that would misrepresent the user's real location.
3. Compare device identifiers and browser/client fingerprints between the two sign-ins.
4. Check MFA satisfaction and method for both sign-ins.
5. Check what each session actually did — mailbox access, file access, application usage.
6. Attempt to reach the user through a verified out-of-band channel to establish their actual location at each time.
7. Check the account's sign-in history for whether this pattern of shifting apparent geography is a recurring, already-explained pattern.

**Evidence and log sources to review**
Entra ID sign-in logs including interactive/non-interactive flag, IP address and known infrastructure ranges, device and client details, MFA logs, and Conditional Access evaluation results for both sign-ins.

**Severity and business impact reasoning**
High until explained — impossible travel that survives the VPN and token-refresh checks strongly indicates the credential is in two places at once, which is compromise by definition.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke sessions for the account pending confirmation | Disabling the account before confirming with the user |
| Force re-authentication with MFA | Broad geographic blocking |
| Preserve both sessions' activity logs | |

**Escalation and communication**
If confirmed compromise, escalate with both session details side by side. If explained by VPN infrastructure, document the corporate egress ranges so future alerts on the same pattern are triaged faster.

**Recovery, lessons learned, detection improvement**
Recovery: session revocation and credential rotation if compromised. Lessons learned: if VPN egress caused repeated false impossible-travel alerts, that is a tuning opportunity. Detection improvement: maintain a known-infrastructure allowlist for corporate VPN and proxy egress ranges so Identity Protection's location logic does not misfire on legitimate traffic, and weight non-interactive sign-ins lower in impossible-travel logic.

**Say this aloud to the interviewer:**
> "Twenty minutes is not enough time to travel between most countries, so on the surface it looks impossible. But I check for corporate VPN egress and non-interactive token refresh first, since both commonly produce this exact pattern legitimately. If neither explains it and the devices differ, I treat it as compromise and revoke sessions."

**Key terms to mention:** interactive versus non-interactive sign-in, VPN egress misattribution, device fingerprint comparison, known-infrastructure allowlist, out-of-band verification.

**Weak answer to avoid:** "It's clearly a compromised account, disable it immediately." Skipping the VPN and token-refresh check causes frequent false escalations and burns credibility with the team.

**Likely follow-up:** "How would you build an allowlist so this stops generating false alerts without losing real detections?"

---

### Q50. `Scenario-based` A user reports receiving dozens of MFA push notifications overnight that they did not initiate, and this morning they tapped "Approve" by accident to make them stop. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** MFA fatigue recognition — a very current, high-frequency real-world attack and a near-guaranteed interview topic.

**Model answer (say this aloud):**
> This is an MFA fatigue attack, and the critical fact is that the user says they approved one, which means the attacker likely has valid access right now. I treat this as an active, ongoing compromise, not a near-miss. My first action is immediate session revocation and forcing re-authentication, because every minute the attacker holds that approved session is a minute they can act. Then I check what that session actually did before I contained it — sign-in location, applications accessed, any changes made. I also check whether the attacker already registered a new MFA method of their own during the session, because that would let them regain access even after I reset the password.

**Scenario walkthrough**

**Initial alert or situation**
User reports repeated unsolicited MFA prompts overnight and confirms they approved one by mistake this morning.

**Investigation steps, in order**
1. Revoke the user's sessions and force re-authentication immediately — this is time-critical, done before deep investigation, not after.
2. Pull the full MFA prompt history overnight: count, source IP for each attempt, and the exact time of the approved one.
3. Identify what the approved session accessed: applications, mailbox, files, and any administrative actions if the account is privileged.
4. Check whether a new MFA method or authenticator device was registered by the attacker during or after the approved session — this is the single most important check, since it is how attackers survive a password reset.
5. Check the source IP and infrastructure behind the push flood for any pattern matching known attack infrastructure.
6. Check whether other users received similar unsolicited pushes around the same time, which would indicate a broader campaign.
7. Review whether number-matching or additional context is enabled on the MFA method, since its absence is usually what makes fatigue attacks possible.

**Evidence and log sources to review**
Entra ID sign-in and MFA logs (all prompts, not just the approved one), audit logs for MFA method registration changes, mailbox and application audit logs for the compromised session window, and Conditional Access evaluation logs.

**Severity and business impact reasoning**
Critical. An approved MFA push during a fatigue attack means the attacker has valid, MFA-satisfied access, which bypasses the primary control the organisation relies on. Treat exactly like a confirmed account compromise from the first minute.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke all active sessions immediately | Disabling the account if it is privileged or a VIP |
| Force password reset and MFA re-registration | Organisation-wide MFA method policy changes |
| Remove any attacker-registered MFA method after evidence capture | |
| Notify the user of exactly what to expect next | |

**Escalation and communication**
Escalate immediately as a confirmed account compromise via MFA fatigue. Recommend an organisation-wide check for similar overnight push patterns on other users, since fatigue attacks are frequently run against several accounts at once. This is a strong candidate for a broader campaign alert to management given the technique's prevalence.

**Recovery, lessons learned, detection improvement**
Recovery: credentials rotated, all MFA methods re-registered from a verified clean state, session and access history reviewed in full. Lessons learned: was number-matching enabled, and were push notifications rate-limited. Detection improvement: alert on high-volume MFA prompts to a single user in a short window regardless of outcome, and push for number-matching or phishing-resistant MFA (FIDO2 keys, certificate-based auth) as a structural fix, since fatigue attacks specifically exploit simple push-approve MFA.

**Say this aloud to the interviewer:**
> "The user approved one, so I treat this as an active compromise right now, not a near-miss. I would revoke sessions immediately before anything else, then check what the approved session did and — most importantly — whether the attacker registered their own MFA method during it, because that is how they survive a password reset."

**Key terms to mention:** MFA fatigue, push bombing, number-matching, phishing-resistant MFA, attacker-registered MFA method, immediate session revocation, campaign-wide check.

**Weak answer to avoid:** "I would just reset their password." A password reset alone does not remove an attacker-registered MFA method, so the attacker keeps access.

**Likely follow-up:** "What structural control would you recommend to prevent this attack class entirely, not just respond to it?"

---

### Q51. `Scenario-based` A user clicked a link that led to a Microsoft-branded consent screen asking to grant a third-party app permissions to their mailbox, and they clicked "Accept." What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** OAuth consent phishing awareness — increasingly common and often missed by candidates who only think in terms of stolen passwords.

**Model answer (say this aloud):**
> This is illicit consent grant, and the important thing to understand is that the attacker never needed the user's password at all — the user directly authorised a malicious application to access their data, and that access persists even through a password reset, because it is a token grant, not a credential. So my response is not about resetting the password, it is about finding and revoking that specific app's permissions. I look at exactly what permissions were granted — read mail, send mail, read files — because that defines the blast radius precisely. I revoke the app's access, then check what it actually did with that access while it had it.

**Scenario walkthrough**

**Initial alert or situation**
User granted OAuth consent to a third-party application via a phishing-style consent prompt.

**Investigation steps, in order**
1. Identify the exact application: its app ID, publisher, verification status, and precisely which permissions or scopes were granted.
2. Check whether the app is a known-malicious or newly registered app, or a legitimate-looking clone of a trusted brand.
3. Determine the scope of access granted — read-only mail, send-as, full mailbox access, file access — since this defines what the attacker could do.
4. Check the mailbox and file audit logs for activity attributable to the app since consent was granted: mail read, mail sent, rules created, files accessed.
5. Check whether other users in the organisation also granted consent to the same application.
6. Revoke the application's access at the tenant or user level per standing procedure.
7. Check whether user consent to third-party apps is broadly permitted in the tenant, since that is the underlying control gap.

**Evidence and log sources to review**
Entra ID application consent audit logs, OAuth application permission grants and scopes, mailbox audit logs for activity during the access window, sign-in logs for the consenting session, and tenant app consent policy settings.

**Severity and business impact reasoning**
High. Illicit consent grant persists independent of password resets and often independent of MFA, and mailbox-scope access can expose the full history of a user's correspondence, which is a significant confidentiality impact.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke the specific application's access per standing procedure | Disabling broad user consent to all third-party apps tenant-wide |
| Notify the user of what happened and what to expect | Removing legitimate business-approved third-party integrations |
| Preserve consent and audit logs as evidence | |

**Escalation and communication**
Escalate with the specific application name, permissions, and what the audit logs show it actually did — this is more actionable to management than "a user clicked a phishing link." Recommend checking for other affected users in the same message.

**Recovery, lessons learned, detection improvement**
Recovery: application access revoked, mailbox reviewed for attacker-created rules or forwarding left behind, user educated on the specific technique since it looks legitimate. Lessons learned: was tenant-wide user consent to unverified apps still permitted. Detection improvement: restrict user consent to admin-approved applications only, alert on new high-privilege OAuth grants especially to unverified publishers, and periodically audit existing app consents across the tenant for anything unused or suspicious.

**Say this aloud to the interviewer:**
> "This is illicit consent grant — the attacker never needed the password, so a password reset does nothing here. I would identify the exact app and the permissions it was granted, check the mailbox audit log for what it actually did with that access, revoke the app specifically, and check whether other users consented to the same app."

**Key terms to mention:** OAuth consent phishing, illicit consent grant, token-based persistence, app permission scope, tenant consent policy, admin-approved apps only.

**Weak answer to avoid:** "I would reset the user's password." This is the classic mistake — it does not touch the OAuth grant at all, and the attacker keeps access.

**Likely follow-up:** "How would you find every user in the tenant who consented to a specific malicious application?"

---

### Q52. What is Conditional Access, and how does it help you as an analyst during an investigation?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Practical understanding of a control you will read logs from constantly.

**Model answer (say this aloud):**
> Conditional Access is a policy engine that evaluates every sign-in against conditions — user, location, device state, application, and risk level — and then applies a control: allow, block, require MFA, require a compliant device, and so on. For me as an analyst it is extremely useful during investigation because every sign-in log shows which policies were evaluated and what the outcome was, so I can immediately see whether MFA was actually enforced, whether the device had to be compliant, and whether the sign-in was blocked or allowed and why. It essentially gives me a pre-computed risk decision I can read rather than reconstruct myself.

**Deeper explanation:**
Conditional Access policies combine signals — user or group, cloud app, device platform and compliance state, location (including named locations and trusted network ranges), client app type, and sign-in or user risk level from Identity Protection — into grant or block controls. For investigations, the sign-in log's Conditional Access section shows each applicable policy and whether it was satisfied, not applied, or reported-only. This tells an analyst instantly whether a sign-in that looks suspicious was actually challenged for MFA and passed, or slipped through because a policy did not apply to that app or that legacy client. A common real gap to mention: policies that exclude legacy authentication clients from evaluation entirely, which is exactly the loophole attackers use.

**Key terms to mention:** policy conditions and controls, named locations, device compliance, report-only mode, sign-in risk versus user risk, legacy authentication exclusion gap.

**Weak answer to avoid:** "It's a way to require MFA." True but incomplete — mention that it is a full policy engine and, crucially, that its evaluation result is visible per sign-in for investigation.

**Likely follow-up:** "A sign-in log shows a Conditional Access policy in 'report-only' mode. What does that mean for your investigation?"

---

### Q53. Why is legacy authentication considered a serious risk, and how would you identify its use in an environment?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** A specific, high-value, frequently tested cloud security fact.

**Model answer (say this aloud):**
> Legacy authentication protocols like POP, IMAP, SMTP AUTH, and older Exchange ActiveSync clients cannot support modern authentication, which means they cannot enforce multi-factor authentication or be evaluated by Conditional Access policies properly. That makes them the preferred entry point for attackers doing password spraying or credential stuffing, because a stolen or guessed password alone is enough to succeed, with no MFA challenge at all. I identify it by filtering sign-in logs for the client app field showing legacy protocols, and I treat any successful legacy authentication sign-in with more scrutiny than a modern one, because it means MFA was never in the picture.

**Deeper explanation:**
Legacy authentication cannot prompt for MFA and predates Conditional Access support for many of its evaluation signals, so a Conditional Access policy that does not explicitly block legacy clients may effectively skip modern controls for those connections. In sign-in logs, the client app field distinguishes modern (browser, mobile apps using modern auth) from legacy (Other clients, POP, IMAP, older ActiveSync, authenticated SMTP). The standard remediation is a Conditional Access policy that blocks legacy authentication entirely, though this requires first auditing which applications or devices still depend on it, since blocking blindly can break business processes like old scanners or line-of-business apps sending mail via SMTP AUTH. From a detection standpoint, any spray or brute force campaign should be checked specifically against legacy sign-in attempts, since that is where it is most likely to succeed.

**Key terms to mention:** legacy protocols (POP/IMAP/SMTP AUTH/older ActiveSync), no MFA support, client app field in sign-in logs, blocking policy, dependency audit before blocking.

**Weak answer to avoid:** "Legacy auth is old and insecure." Say specifically why — no MFA support — since that is the fact interviewers are checking for.

**Likely follow-up:** "You want to block legacy authentication tenant-wide, but you're worried about breaking something. How do you find out what still uses it first?"

---

### Q54. `Scenario-based` During investigation of a compromised mailbox, you find a hidden inbox rule that forwards all emails containing the word "invoice" to an external address and then deletes them from the inbox. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** BEC-pattern recognition and correct evidence handling of the rule itself.

**Model answer (say this aloud):**
> This is a classic Business Email Compromise setup — the attacker is filtering for financial correspondence and hiding the evidence by deleting the forwarded copy, so the real user never sees it happening. Before I touch anything I document the rule exactly as it is — its conditions, the destination address, and when it was created — because that evidence is critical for anyone downstream, including for notifying anyone who received a fraudulent invoice as a result. Then I remove the rule, revoke sessions, and reset credentials. I also have to think beyond this mailbox: was a fraudulent payment request sent to a customer or supplier from this account while the attacker had control, because that is a business harm that needs its own urgent notification.

**Scenario walkthrough**

**Initial alert or situation**
A hidden inbox rule found on a compromised mailbox, forwarding and deleting finance-related emails.

**Investigation steps, in order**
1. Document the rule precisely: exact conditions, destination address, actions, and its creation timestamp, before making any change.
2. Identify when the rule was created and correlate that to a sign-in event to establish the compromise window.
3. Review sent items and any drafts within the compromise window for signs the attacker sent fraudulent messages, particularly changed banking details or invoice redirection.
4. Check for other persistence: delegated mailbox access, additional forwarding at the transport level (which is separate from inbox rules and easy to miss), and any registered MFA changes.
5. Identify who received emails as a result of the rule while it was active, and specifically whether any external party received a fraudulent request.
6. Check whether other mailboxes show the same or similar rule pattern, since BEC campaigns often hit several accounts.

**Evidence and log sources to review**
Mailbox audit logs (rule creation, sent items, rule conditions and actions), transport-level mail flow rules separate from inbox rules, sign-in logs correlated to the rule creation time, and any external delivery reports for messages sent during the compromise window.

**Severity and business impact reasoning**
High to Critical. BEC targeting finance correspondence often precedes fraudulent payment redirection, which is a direct and often large financial loss, and in a government context could also mean sensitive correspondence disclosure.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Preserve the rule's exact configuration before removing it | Notifying external parties who may have received fraudulent messages |
| Remove the malicious rule | Contacting finance or leadership about a potential fraud attempt |
| Revoke sessions and reset credentials | |
| Check for transport-level forwarding as well | |

**Escalation and communication**
Escalate immediately, and flag explicitly whether any outbound message during the compromise window looks like a fraud attempt, since that requires urgent notification to finance and potentially to the external party who received it — a decision made by the incident manager, not the analyst alone.

**Recovery, lessons learned, detection improvement**
Recovery: rule removed, credentials rotated, delegated access reviewed, and any fraudulent communication retracted or flagged to the recipient through the proper channel. Lessons learned: how long did the rule exist undetected. Detection improvement: alert on any inbox rule that forwards to an external domain combined with a delete or move-to-a-hidden-folder action, and alert on transport rule creation by non-administrative accounts — both are high-fidelity BEC indicators with very low false positive rates.

**Say this aloud to the interviewer:**
> "This is a classic BEC setup — filtering finance keywords, forwarding externally, deleting the evidence. I would document the rule exactly before touching it, then check sent items during the compromise window for any fraudulent request that went out, because that is a business harm that needs urgent notification, separate from just cleaning the mailbox."

**Key terms to mention:** Business Email Compromise, hidden inbox rule, transport-level forwarding, fraudulent payment redirection, compromise window, external notification decision.

**Weak answer to avoid:** "I would delete the rule and reset the password." Deleting the rule first without documenting it loses evidence, and missing the check for fraudulent outbound mail misses the actual business harm.

**Likely follow-up:** "How is a transport-level mail flow rule different from an inbox rule, and why would an attacker prefer one over the other?"

---

### Q55. `Scenario-based` The Azure Activity Log shows a new Owner role assignment on a subscription, made by an account that has never made role assignments before, at an unusual hour. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Cloud control-plane investigation — privilege escalation in Azure specifically, not just identity.

**Model answer (say this aloud):**
> Owner on a subscription is one of the most powerful roles that exists in Azure, since it can manage every resource and grant any other role including its own. So a first-time role assignment like this from an unusual account at an unusual hour is treated the same way I would treat a Domain Admin addition on-premises — high severity from the first minute. I check how the assigning account itself authenticated, what its normal activity looks like, and whether it holds Owner or User Access Administrator itself, since only certain roles can grant Owner. Then I check what the newly granted account has done with that access since, because subscription Owner can create resources, exfiltrate data, or grant itself even more paths of access.

**Scenario walkthrough**

**Initial alert or situation**
A first-time Owner role assignment on a subscription, made by an account with no history of making role assignments, occurring outside business hours.

**Investigation steps, in order**
1. Confirm the exact role assignment details: assigning principal, assigned principal, scope, and timestamp from the Activity Log.
2. Check how the assigning account authenticated around that time — sign-in log, MFA satisfaction, source IP, and device.
3. Confirm the assigning account genuinely held a role capable of granting Owner, such as Owner or User Access Administrator itself, and how it obtained that role.
4. Check for a change record or approval authorising the assignment.
5. Review everything the newly granted principal has done since receiving Owner: resource creation, key or secret access, data exports, network rule changes, or further role assignments.
6. Check whether the newly granted principal is a user, a service principal, or a guest account, since each implies a different risk profile.
7. Check the subscription's resource inventory for anything sensitive — key vaults, storage accounts with sensitive data, production workloads.

**Evidence and log sources to review**
Azure Activity Log for role assignment and subsequent resource operations, Entra ID sign-in logs for the assigning account, Azure AD audit logs for any related directory changes, Key Vault access logs, and storage account access logs if applicable.

**Severity and business impact reasoning**
Critical. Subscription Owner is effectively full control of every resource in that subscription. Unexplained, first-time, out-of-hours assignment of this role should be treated as an active privilege escalation until proven otherwise.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Preserve the Activity Log evidence | Removing the Owner role assignment |
| Check the assigning and assigned accounts' recent activity | Disabling either account |
| Raise monitoring on both accounts and the subscription | Locking down or restricting the subscription |

**Escalation and communication**
Escalate immediately to the incident manager and the cloud platform team together, since removing a live Owner assignment or restricting a subscription needs coordination to avoid breaking production. State clearly what resources the subscription contains and what the newly privileged account has already done.

**Recovery, lessons learned, detection improvement**
Recovery: remove the unauthorised assignment through a coordinated change, rotate any secrets or keys the account could have accessed, and review every resource change made during the window of elevated access. Lessons learned: was the assigning account itself compromised, and why was it able to grant Owner without an approval gate. Detection improvement: alert on every subscription Owner or User Access Administrator role assignment in real time regardless of who performs it, and implement Privileged Identity Management for just-in-time, approval-gated elevation instead of standing high-privilege roles.

**Say this aloud to the interviewer:**
> "Subscription Owner is as powerful as Domain Admin, so I would treat this with the same urgency. I would check how the assigning account authenticated and whether it genuinely held the rights to grant Owner, then review everything the newly privileged account has done since — resource changes, key access, data exports — and escalate immediately for a coordinated removal rather than acting alone."

**Key terms to mention:** subscription Owner role, User Access Administrator, Privileged Identity Management, just-in-time elevation, Azure Activity Log, coordinated removal.

**Weak answer to avoid:** "I would just remove the role assignment myself." Removing a live privileged role without coordination can break production dependencies and should go through the incident manager and platform team.

**Likely follow-up:** "What is Privileged Identity Management and how would it have prevented this from being a standing risk in the first place?"

---

### Q56. Explain the shared responsibility model in Azure, and why it matters to a SOC analyst.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand what your organisation is actually responsible for detecting and securing in the cloud.

**Model answer (say this aloud):**
> The shared responsibility model splits security duties between Microsoft and the customer, and where that line falls depends on the service model. For infrastructure as a service, Microsoft secures the physical datacentre, host, and hypervisor, while the customer is responsible for the operating system, network configuration, identity, and data. For platform as a service, Microsoft also manages the runtime and operating system, and the customer focuses on identity, configuration, and data. For software as a service like Microsoft 365, Microsoft manages almost the entire stack, and the customer is mainly responsible for identity, access management, data classification, and endpoint security. As a SOC analyst this matters because it tells me exactly what I need to monitor myself — Microsoft is not watching my tenant's identity misuse or my misconfigured storage account for me.

**Deeper explanation:**
Regardless of service model, the customer is **always** responsible for: identity and access management, data classification and protection, endpoint security, and secure configuration of anything they control. This is why identity and configuration monitoring dominate cloud SOC work — the attack surface a customer actually owns has shifted from "patch the server" to "manage who can authenticate and what they can touch." A practical example worth giving: in SaaS like Microsoft 365, Microsoft secures the application itself, but a misconfigured sharing link, an over-permissive app consent, or a compromised identity are entirely the customer's responsibility to detect and respond to — which maps directly onto much of the identity, OAuth, and mailbox-rule content already covered in this section.

**Key terms to mention:** IaaS/PaaS/SaaS boundaries, customer always owns identity and data, configuration responsibility, security "of" the cloud versus security "in" the cloud.

**Weak answer to avoid:** "Microsoft handles security." Missing the customer's always-owned responsibilities is a common and serious misunderstanding.

**Likely follow-up:** "Give an example of a security failure that is entirely the customer's fault, not Microsoft's, under this model."

---

### Q57. Which KQL tables do you go to first for an Entra ID or Microsoft 365 investigation, and what does each contain?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Practical Sentinel/Microsoft 365 Defender fluency.

**Model answer (say this aloud):**
> For identity, SigninLogs gives me interactive sign-ins, and AADNonInteractiveUserSignInLogs gives me token refreshes and background authentication, which people often forget to check. AuditLogs covers directory changes — role assignments, group membership, application consent. For mailbox activity, OfficeActivity or the unified audit log covers rule creation, mail sent, and file access in SharePoint and OneDrive. For endpoint, DeviceProcessEvents, DeviceNetworkEvents, and DeviceLogonEvents from Defender for Endpoint give me the same kind of telemetry Sysmon would, but centrally queryable. And IdentityLogonEvents and related identity tables in Defender for Identity cover on-premises Active Directory signals forwarded into the same workspace.

**Deeper explanation:**

| Table | Contents | Common use |
|---|---|---|
| `SigninLogs` | Interactive user sign-ins | Risky sign-in, impossible travel, MFA verification |
| `AADNonInteractiveUserSignInLogs` | Token refresh, background auth | Often missed source of "hidden" sign-in activity |
| `AuditLogs` | Directory changes | Role assignment, group membership, app consent grants |
| `OfficeActivity` | Exchange, SharePoint, OneDrive, Teams actions | Inbox rules, mail sent, file sharing, mailbox delegation |
| `DeviceProcessEvents` | Endpoint process creation | Execution, command-line analysis (Defender for Endpoint) |
| `DeviceNetworkEvents` | Endpoint network connections | C2, beaconing, lateral movement source |
| `DeviceLogonEvents` | Endpoint logon activity | Logon type, source, account used |
| `IdentityLogonEvents` | On-prem AD authentication via Defender for Identity | Kerberos abuse, NTLM relay, DCSync-adjacent signals |
| `EmailEvents` / `EmailUrlInfo` / `EmailAttachmentInfo` | Mail flow and content metadata | Phishing triage, malicious attachment or URL tracing |

A practical habit worth stating: always check both `SigninLogs` and the non-interactive table together, since an attacker's malicious app or persisted token activity frequently only appears in the non-interactive log and is invisible if you only check the interactive one.

**Key terms to mention:** SigninLogs vs non-interactive sign-ins, AuditLogs for directory changes, OfficeActivity/unified audit log, Defender for Endpoint device tables, Defender for Identity tables, EmailEvents.

**Weak answer to avoid:** Naming only `SigninLogs`. Missing the non-interactive table and `AuditLogs` misses most persistence and privilege-escalation evidence.

**Likely follow-up:** "Write a query joining SigninLogs and AuditLogs to find role changes shortly after a risky sign-in."

---

[⬅ Previous: Network Security](04-network-security-and-traffic-analysis.md) · [Back to README](../README.md) · [Next: EDR & Malware ➡](06-edr-malware-and-endpoint-security.md)
