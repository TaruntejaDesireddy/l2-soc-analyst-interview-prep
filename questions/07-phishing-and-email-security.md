# 07 · Phishing and Email Security

**6 questions · Q66–Q71 · 3 scenario-based**

[⬅ Previous: EDR & Malware](06-edr-malware-and-endpoint-security.md) · [Back to README](../README.md) · [Next: Threat Hunting & MITRE ➡](08-threat-hunting-mitre-and-threat-intelligence.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q66 | How you analyse a suspicious email end to end | Core | Standard |
| Q67 | Reading email headers to trace true origin | Advanced | Standard |
| Q68 | User reports they clicked a link and entered credentials | Advanced | Scenario-based |
| Q69 | Safe URL and attachment analysis | Core | Standard |
| Q70 | Executive-targeted BEC wire transfer request | Advanced | Scenario-based |
| Q71 | One phishing email, multiple recipients | Advanced | Scenario-based |

---

### Q66. Walk me through how you analyse a suspicious email end to end.

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** A structured, repeatable process rather than "it looked dodgy so I reported it."

**Model answer (say this aloud):**
> I work outside-in: envelope first, then headers, then content, then any links or attachments, and I never click anything directly — everything is checked through safe tooling. I check the envelope sender against the display name, because a mismatch there is one of the strongest signals. I read the headers to trace the actual sending infrastructure and check SPF, DKIM and DMARC results. I read the content for urgency, authority pressure, and unusual requests. Any URL gets checked for its real destination, not the display text, using a sandboxed or reputation tool. Any attachment gets a hash lookup and, if needed, detonation in a sandbox rather than opened locally. Then I decide: malicious, suspicious but inconclusive, or benign, and I act accordingly on the mailbox and on anyone else who received it.

**Deeper explanation:**
Structured sequence: (1) envelope-from versus display name versus reply-to, looking for mismatches; (2) header trace through `Received` lines to find the true originating server, since intermediate hops can look legitimate while the true origin is not; (3) authentication results — SPF pass/fail, DKIM signature validity, DMARC alignment and policy; (4) content analysis for social engineering markers — urgency, authority, unusual payment or credential requests, poor grounding in normal business context; (5) URL analysis via de-obfuscation and reputation/sandbox checking of the true destination, not the anchor text; (6) attachment hash and, where warranted, sandbox detonation; (7) search the mail environment for other recipients of the same or similar message. Never interact with suspected phishing content directly from a production browser or mail client — always through purpose-built, isolated tooling.

**Key terms to mention:** envelope-from vs display name, SPF/DKIM/DMARC, header trace, URL de-obfuscation, sandbox detonation, other-recipient search.

**Weak answer to avoid:** "I look at it and decide if it looks suspicious." No structure, no mention of headers or authentication results — this reads as untrained.

**Likely follow-up:** "SPF passes but DMARC still fails. What does that tell you?"

---

### Q67. How do you read email headers to trace the true origin of a message?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Whether you can actually do this, since it is asked constantly and often skipped by candidates who only know the theory.

**Model answer (say this aloud):**
> I read the `Received` headers from the bottom up, because that is the order the message actually travelled — the bottom entry is the earliest hop, closest to the true sender, and the top is the last hop before it reached me. Each hop shows which server received it from which other server, so I am looking for the first external hop into an infrastructure I do not recognise, and I check whether that IP matches the domain it claims to be from. I also check `Authentication-Results` for the SPF, DKIM and DMARC verdicts as evaluated by the receiving server, since that is more trustworthy than anything in the visible sender fields, which the attacker fully controls.

**Deeper explanation:**
Key header fields: `Return-Path` (envelope sender, used for SPF), `From` (display sender, easily spoofed), `Reply-To` (where replies actually go — a very common phishing tell when it differs from `From`), `Received` chain (actual server-to-server hops, read bottom-to-top), `Authentication-Results` (the receiving server's own SPF/DKIM/DMARC verdicts), and `Message-ID` (can reveal the true originating mail system's naming convention). Practical check: take the originating IP from the earliest external `Received` hop and look up its reverse DNS and ASN — legitimate mail infrastructure for a claimed sender domain should resolve sensibly; a mismatch is a strong indicator. Remember that `From` and `Reply-To` are attacker-controlled and prove nothing on their own — the `Received` chain and `Authentication-Results` are the fields that are hard to forge.

**Key terms to mention:** Received chain read bottom-up, Return-Path, Reply-To mismatch, Authentication-Results, reverse DNS and ASN of originating IP, Message-ID.

**Weak answer to avoid:** "I check the From address." The From field is exactly what attackers spoof; explain why the Received chain and authentication results matter more.

**Likely follow-up:** "Show me, in words, how you'd identify the true originating IP from a header block with five Received lines."

---

### Q68. `Scenario-based` A user calls you directly, panicked, saying they clicked a link in an email, it took them to a Microsoft-looking login page, and they entered their username and password. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Fast, correct credential-phishing response — one of the highest-frequency real scenarios.

**Model answer (say this aloud):**
> I treat the account as compromised immediately, because a working credential is now in an attacker's hands, and I do not wait to analyse the phishing page first — that comes after containment. So the very first thing I do is revoke the user's active sessions and force a password reset, because Entra ID sign-in tokens can outlive a password change if I do not explicitly revoke them. Then I check whether the attacker actually used the credential yet — sign-in logs right after the click — and if MFA is enabled, whether it was also phished or bypassed. In parallel I get the actual email and URL so it can be blocked for everyone else, and I check whether anyone else received the same message.

**Scenario walkthrough**

**Initial alert or situation**
User self-reports entering credentials on a phishing page reached via an email link.

**Investigation steps, in order**
1. Revoke all active sessions for the account and force a password reset immediately — before any further analysis, because this is the time-critical action.
2. Check sign-in logs immediately after the reported click time for any successful authentication from an unfamiliar source.
3. Check whether MFA is enabled on the account and whether the phishing page attempted to capture it too — some kits proxy the real login and steal the MFA-satisfied session token directly.
4. If a session token may have been stolen via an adversary-in-the-middle kit, revoking sessions is essential since a password reset alone does not invalidate an already-issued token.
5. Obtain the original email and the exact URL from the user, and submit both for blocking at the mail gateway and web proxy.
6. Search the mail environment for the same sender or same URL sent to other recipients.
7. Check for post-compromise indicators if any sign-in did occur: new inbox rules, new MFA methods registered, mailbox access.

**Evidence and log sources to review**
Entra ID sign-in logs immediately following the reported time, session and token issuance logs, the original email with full headers, URL reputation and sandbox analysis, mail environment search for other recipients, and mailbox/audit logs for any post-compromise activity.

**Severity and business impact reasoning**
High immediately, regardless of whether a successful malicious sign-in is confirmed yet, because a valid credential is confirmed compromised. Escalates to Critical if the account is privileged or if evidence shows the attacker already authenticated.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Revoke sessions and force password reset per standard procedure | Disabling the account if privileged |
| Block the sender and URL at the gateway/proxy per standing policy | Organisation-wide communication about the campaign |
| Search for and remove the same email from other mailboxes per procedure | |
| Praise the user for reporting quickly — not a technical action, but say it | |

**Escalation and communication**
Escalate as a confirmed credential compromise, and note explicitly whether MFA protected the account or whether it may have been phished too — that materially changes urgency. Thank the user for reporting promptly; a blame-free response is what keeps future reports coming in fast, which is a genuine security control in itself.

**Recovery, lessons learned, detection improvement**
Recovery: credentials rotated, sessions confirmed revoked, mailbox reviewed for any changes made during exposure, user briefed on what happened. Lessons learned: did the phishing page bypass MFA, and how did the email reach the inbox past existing filtering. Detection improvement: submit the URL and sender to threat intelligence and mail security for blocking, and if an adversary-in-the-middle kit is suspected, recommend phishing-resistant MFA as the structural fix.

**Say this aloud to the interviewer:**
> "I treat the account as compromised the moment they tell me, so the first action is revoking sessions and forcing a password reset, before I even finish analysing the email. Then I check the sign-in logs for what happened right after the click, and whether MFA may have been phished too, because a password reset alone doesn't kill an already-issued session token."

**Key terms to mention:** immediate session revocation, password reset versus token invalidation, adversary-in-the-middle phishing kit, MFA-aware phishing, other-recipient search, blame-free reporting culture.

**Weak answer to avoid:** "I would analyse the phishing page first to understand it." Analysis before containment leaves a confirmed compromised credential live for longer than necessary.

**Likely follow-up:** "The user says they also approved an MFA prompt right after. Does your response change?"

---

### Q69. How do you safely analyse a suspicious URL and a suspicious attachment without putting yourself or the organisation at risk?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Safe handling practice — a basic but essential operational security habit.

**Model answer (say this aloud):**
> I never open a suspicious link or attachment directly on my own machine or a production system. For URLs, I first de-obfuscate and read the true destination rather than trusting the display text, then I check it through reputation and sandbox tools that render it in an isolated environment. For attachments, I calculate the file hash first and check it against reputation sources before doing anything else, because a known-bad hash answers the question immediately with zero risk. If it is unknown, I submit it to a sandbox for detonation rather than opening it myself, and I look at the sandbox's behavioural report — what it connected to, what it wrote, what it modified — rather than the file's static appearance.

**Deeper explanation:**
URL handling: expand shortened or redirect-chained URLs to their final destination, check domain age and reputation, and use an isolated browsing or sandbox tool for genuinely unknown links — never click through directly. Attachment handling: compute and check the hash first against reputation and any internal threat intelligence; if unknown, submit for automated sandbox detonation and review the behavioural report (network connections, file and registry changes, process spawned); for documents, check for macros, embedded objects, or exploit indicators without enabling content. General discipline: work from a purpose-built analysis environment, never your primary workstation, and never disable your own endpoint protections to "see what happens." If in doubt about your tooling's isolation, escalate rather than improvise.

**Key terms to mention:** URL de-obfuscation and final destination, sandbox detonation, hash-first attachment triage, isolated analysis environment, macro and embedded object checks.

**Weak answer to avoid:** "I would open it in a private browser tab to check." A private/incognito tab provides no real isolation from malicious content and is not a safe analysis method.

**Likely follow-up:** "The sandbox report shows no malicious behaviour at all. Do you close the case?"

---

### Q70. `Scenario-based` An email appearing to be from your CEO asks a finance manager to urgently process a wire transfer to a new supplier account, marked confidential and time-sensitive. The finance manager forwards it to you before acting. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** BEC recognition and correct handling — including praising the reporting behaviour that stopped the fraud.

**Model answer (say this aloud):**
> Every marker here is classic Business Email Compromise — impersonating an executive, urgency, confidentiality, a new payment destination, and a request that deliberately bypasses normal verification. The finance manager did exactly the right thing by not acting and forwarding it instead, and I say that clearly, because reinforcing that behaviour matters as much as the technical analysis. I check the email's authenticity technically — headers, sender infrastructure, whether it matches the CEO's normal sending pattern — while in parallel advising that no payment should be processed until verified through a separate, trusted channel like a phone call to a known number, never by replying to the email itself.

**Scenario walkthrough**

**Initial alert or situation**
A finance manager forwards a suspected BEC email impersonating the CEO, requesting an urgent, confidential wire transfer to a new account.

**Investigation steps, in order**
1. Check envelope sender, display name, and reply-to for mismatches against the CEO's genuine address.
2. Trace the header chain to identify the true sending infrastructure and compare it against the organisation's legitimate mail sources.
3. Check SPF, DKIM and DMARC results — a spoofed exact address usually fails these, though a lookalike domain can pass them legitimately while still being fraudulent.
4. Check whether the domain is an exact match or a lookalike (character substitution, extra letter, different top-level domain).
5. Advise immediately, in parallel with the technical check, that the payment must not be processed until verified out-of-band.
6. Search the mail environment for the same or similar messages sent to other staff, since BEC campaigns frequently target several people at once.
7. If the CEO's real account shows any sign-in anomaly, treat it as a possible actual account compromise rather than pure impersonation, and investigate that account directly.

**Evidence and log sources to review**
Email headers and authentication results, sender domain registration and reputation, the CEO's actual mailbox sign-in logs (to rule out real account compromise versus impersonation), and a mail-environment search for related messages to other recipients.

**Severity and business impact reasoning**
High immediately, given direct financial fraud risk and potential impersonation of leadership. If it turns out the CEO's actual account is compromised rather than just spoofed, severity rises further because of everything else that account can reach.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Advise finance to hold the payment pending verification | Confirming to finance whether to proceed — that is a business decision made with the facts I provide, not by me alone |
| Block the sending domain/address at the mail gateway per procedure | Contacting the CEO directly about impersonation, depending on process |
| Search for and quarantine the same message sent to others | External notification to the supposed new supplier |
| Check the CEO's real account for compromise indicators | |

**Escalation and communication**
Escalate immediately to the incident manager and finance leadership together, since this is a fraud-prevention race against time. State clearly and immediately whether this is impersonation (spoofed/lookalike domain, real account untouched) or actual account compromise, since the two demand very different responses.

**Recovery, lessons learned, detection improvement**
Recovery: confirm no payment was made, block the fraudulent infrastructure, and brief the finance team on the outcome. Lessons learned: did the finance team have a defined out-of-band verification step for payment changes, and did they follow it correctly this time. Detection improvement: implement DMARC enforcement to reduce exact-domain spoofing, flag external emails claiming to be from internal executives with a visible banner, and establish a mandatory callback-verification policy for any payment detail change, regardless of how the request arrives.

**Say this aloud to the interviewer:**
> "Every marker here is classic BEC — executive impersonation, urgency, confidentiality, a new payment account. I would tell the finance manager clearly they did the right thing by not acting, verify the email's authenticity technically, and make sure no payment is processed until it is verified through a separate trusted channel, not by replying to the email."

**Key terms to mention:** Business Email Compromise, executive impersonation, lookalike domain, DMARC enforcement, out-of-band payment verification, external-sender banner.

**Weak answer to avoid:** "I would tell them to check if it's really from the CEO by replying to the email." Replying to the email reaches the attacker directly, not the real CEO — verification must always be out-of-band.

**Likely follow-up:** "The domain is an exact match to the real company domain and DKIM passes. What does that tell you, and what do you check next?"

---

### Q71. `Scenario-based` You confirm one phishing email is malicious, and a mail search shows it was delivered to 40 mailboxes, with 6 users having clicked the link. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Scaling a single-user response into an organisation-wide campaign response.

**Model answer (say this aloud):**
> The moment I know the real numbers, the scope of my response has to match them — this is no longer a one-user cleanup, it is a campaign. I get the email removed from all 40 mailboxes as fast as possible, since every minute it sits in an inbox is another possible click. For the 6 who clicked, I need to know individually what each one did after clicking — did they just land on the page, or did they also enter credentials — because those two groups need different levels of response. I do not treat all 6 the same until I know that.

**Scenario walkthrough**

**Initial alert or situation**
A confirmed phishing email delivered to 40 mailboxes, with 6 confirmed clicks on the malicious link.

**Investigation steps, in order**
1. Initiate removal of the email from all remaining mailboxes that have not yet opened it, as the top priority action.
2. For each of the 6 clickers, determine individually what happened after the click: page view only, credential entry, or file download and execution — treat each account's follow-up investigation separately.
3. For any of the 6 with credential entry, follow the full account-compromise response: revoke sessions, reset credentials, check sign-in logs, check for post-compromise activity.
4. For any of the 6 with only a page view and no credential entry, confirm no further action occurred and monitor rather than fully containing.
5. Block the sending infrastructure and the malicious URL organisation-wide.
6. Check whether the 40 recipients share something in common — a distribution list, a department, a public-facing directory — which tells you how the attacker built the target list.
7. Determine whether any of the 6 accounts are privileged, since that changes prioritisation within the group.

**Evidence and log sources to review**
Mail environment-wide search results for delivery and interaction, URL click-tracking logs per user if available, sign-in logs for each of the 6 clicking accounts, and distribution list or directory data to understand targeting.

**Severity and business impact reasoning**
High, scaling with the number of confirmed clicks and especially with the number of confirmed credential entries. A 40-recipient campaign against a defined group suggests deliberate targeting rather than random spam, which itself is worth noting.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Purge the email from remaining mailboxes per standing procedure | Organisation-wide announcement about the campaign |
| Revoke sessions and reset credentials for confirmed compromises | Disabling accounts, if any are privileged or VIP |
| Block sender and URL at the gateway | |
| Prioritise investigation of the 6 clickers by privilege level | |

**Escalation and communication**
Escalate as a confirmed multi-user phishing campaign, giving exact numbers: delivered, clicked, credentials entered, confirmed compromised. Recommend a brief, clear organisation-wide notice describing the email so anyone who has not yet reported it recognises it — this is a communication decision made with leadership, not sent unilaterally by the analyst.

**Recovery, lessons learned, detection improvement**
Recovery: complete individual remediation for each compromised account, confirm the email is fully purged, and monitor for any secondary activity such as internal phishing sent from a compromised account. Lessons learned: why did the email pass existing mail filtering to reach 40 mailboxes. Detection improvement: submit the confirmed indicators — sender, URL, attachment hash if any — to improve mail filtering, and consider targeted awareness follow-up for the group that was specifically targeted.

**Say this aloud to the interviewer:**
> "The response has to scale with the real numbers. I would purge the email from the remaining mailboxes immediately, then treat the 6 clickers individually rather than as one group — full compromise response for anyone who entered credentials, and just monitoring for anyone who only viewed the page — while blocking the sender and URL organisation-wide."

**Key terms to mention:** environment-wide mail purge, per-user outcome differentiation, credential entry versus page-view-only, targeted campaign recognition, indicator submission for filtering improvement.

**Weak answer to avoid:** "I would reset all 6 users' passwords to be safe." Treating a page-view-only click identically to a credential-entry click wastes response effort and can mask which cases are genuinely urgent.

**Likely follow-up:** "One of the 6 is a domain administrator. How does that change your prioritisation?"

---

[⬅ Previous: EDR & Malware](06-edr-malware-and-endpoint-security.md) · [Back to README](../README.md) · [Next: Threat Hunting & MITRE ➡](08-threat-hunting-mitre-and-threat-intelligence.md)
