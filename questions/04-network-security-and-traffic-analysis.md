# 04 · Network Security and Traffic Analysis

**10 questions · Q38–Q47 · 4 scenario-based**

[⬅ Previous: Windows & AD](03-windows-active-directory-and-identity.md) · [Back to README](../README.md) · [Next: Azure & M365 ➡](05-azure-microsoft-365-and-cloud-security.md)

| # | Question | Difficulty | Type |
|---|----------|-----------|------|
| Q38 | DNS in attacks — tunnelling, fast flux, DGA | Advanced | Standard |
| Q39 | Proxy log fields you check first | Core | Standard |
| Q40 | Reading a firewall/NetFlow log for lateral movement | Advanced | Scenario-based |
| Q41 | TLS/HTTPS visibility without decrypting content | Core | Standard |
| Q42 | Large outbound transfer at 23:00 | Advanced | Scenario-based |
| Q43 | VPN log anomaly — split geography | Advanced | Scenario-based |
| Q44 | Identifying C2 beaconing in NetFlow | Advanced | Standard |
| Q45 | DNS tunnelling detection walkthrough | Advanced | Scenario-based |
| Q46 | Common ports and protocols you must know | Core | Standard |
| Q47 | Difference between IDS and IPS in your workflow | Core | Standard |

---

### Q38. How do attackers abuse DNS, and what should you look for?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Whether you treat DNS as a security telemetry source, not just name resolution.

**Model answer (say this aloud):**
> DNS is abused in three main ways. Tunnelling encodes data inside DNS queries and responses to exfiltrate data or run command and control through a channel that is rarely blocked. Domain generation algorithms let malware compute a new list of possible command and control domains every day so a static blocklist cannot keep up. Fast flux rotates the IP addresses behind a domain rapidly to resist takedown and blocking. What I look for is query volume and pattern rather than any single query: unusually long subdomains or high entropy in the subdomain, a high rate of NXDOMAIN responses from one host suggesting it is cycling through generated names, TXT record queries that are unusual for normal traffic, and a domain whose resolved IP changes very frequently.

**Deeper explanation:**
Detection signals in more detail: **tunnelling** — abnormally long query names, high query rate to a single parent domain, unusual record types such as TXT or NULL, high entropy or base32/base64-looking subdomain labels, and response sizes larger than typical resolution traffic; **DGA** — high NXDOMAIN ratio from a single host, queries with high lexical entropy and no dictionary structure, and many distinct never-before-seen domains queried in sequence; **fast flux** — a domain resolving to many distinct IPs with very low TTL, often across many different networks or countries. Practical response: query the DNS logs grouped by source host for query volume and NXDOMAIN ratio over a rolling window, and treat DNS as an equal-priority log source to proxy and firewall, not an afterthought.

**Key terms to mention:** DNS tunnelling, DGA, NXDOMAIN ratio, subdomain entropy, fast flux, low TTL, TXT record abuse.

**Weak answer to avoid:** "DNS just resolves names to IPs." Misses that it is one of the richest and least-filtered channels attackers use.

**Likely follow-up:** "Write the logic for detecting a high NXDOMAIN ratio from a single host."

---

### Q39. When you open a proxy log for an investigation, which fields do you check first and why?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Practical web-traffic triage skill.

**Model answer (say this aloud):**
> I start with destination — the domain, the category, and its reputation and first-seen date, because a domain registered days ago is far more suspicious than an established one. Then the HTTP method and response code, because a series of POST requests with 200 responses to an unfamiliar domain looks like data going out successfully. I check the user agent, because malware often uses a generic or outdated user agent that does not match the browser actually installed on the host. I check bytes sent versus bytes received, since a request sending far more than it receives is consistent with exfiltration. And I check the requesting process on the endpoint if that field is available, because a browser making the request is very different from an unknown executable making it directly.

**Deeper explanation:**
Priority fields: destination domain and category, domain reputation and age, full URI path and query string, HTTP method, response code, request and response size, user agent string, referrer, TLS SNI, and the source process where the proxy or EDR integration provides it. Red flags: mismatched user agent versus actual browser; direct-to-IP HTTPS requests with no SNI; POST-heavy traffic to an uncategorised or newly registered domain; user agents naming a scripting library (a generic HTTP client) rather than a browser; and a URI that looks like it is carrying encoded data rather than a normal page path.

**Key terms to mention:** SNI, domain reputation and age, method and response code pairing, byte asymmetry, user agent mismatch, source process attribution.

**Weak answer to avoid:** "I check the URL." Name the specific fields and what each one tells you.

**Likely follow-up:** "The domain is categorised as 'business' and has existed for ten years, but the traffic pattern still looks wrong. Do you dismiss it?"

---

### Q40. `Scenario-based` You are reviewing firewall and NetFlow logs and see one internal workstation initiating SMB and RDP connections to fifteen other internal hosts within twenty minutes. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Recognising lateral movement from flow data, and containment judgement for internal spread.

**Model answer (say this aloud):**
> A single workstation reaching out to fifteen internal hosts over SMB and RDP in twenty minutes is not normal user behaviour — it is scanning or lateral movement. My first move is to confirm whether those connections actually succeeded or were rejected, because that changes urgency enormously. Then I go to the source host and find the process responsible, because NetFlow tells me the pattern but not the cause. I treat this as an active spreading event until proven otherwise, so I isolate the source host quickly, and I check every one of the fifteen destinations for signs the connection actually landed and did something.

**Scenario walkthrough**

**Initial alert or situation**
One internal workstation shows a burst of SMB and RDP connection attempts to fifteen internal hosts in a short window.

**Investigation steps, in order**
1. Separate successful from rejected connections in the flow data — a high rejection rate suggests scanning; a high success rate suggests active lateral movement already occurring.
2. On the source host, identify the responsible process, its parent, command line, and the account context via EDR telemetry.
3. Check whether the account driving these connections is the logged-on user or a different, higher-privileged account, which points at credential theft.
4. For each destination with a successful connection, check for a new logon (4624 type 3 or 10) around the same timestamp and any subsequent process creation.
5. Check whether any destination is a domain controller or another tier-0 asset, since that changes severity immediately.
6. Determine how the source host was itself compromised — the true origin of the spread.
7. Check whether this pattern is now repeating from any of the fifteen destination hosts, which would confirm active propagation.

**Evidence and log sources to review**
Firewall and NetFlow connection records with success/reject state, EDR process and network telemetry on the source host, security event logs (4624, 4688) on each destination host, asset inventory for destination criticality, and directory logs for the account used.

**Severity and business impact reasoning**
Critical if any connection succeeded, especially to a server or domain controller — this is active lateral movement with the potential for rapid, wide compromise. High if all connections were rejected, since scanning still indicates an active internal threat gathering targets.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the source host via EDR | Isolating multiple production servers simultaneously |
| Preserve evidence on the source host | Segmenting a network zone |
| Check and contain the account used, per procedure | Domain-wide credential rotation |
| Alert-check the fifteen destinations for follow-on activity | Taking a domain controller offline |

**Escalation and communication**
Escalate immediately as suspected active lateral movement, listing the destination count and which, if any, are tier-0 assets. This is exactly the kind of finding that justifies declaring a major incident before full scope is known, because speed of containment matters more than a complete picture.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the source host, rotate any credentials used during the spread, and individually verify each of the fifteen destinations before declaring them clean. Lessons learned: how far did detection lag behind the actual spread. Detection improvement: a correlation rule for one source host connecting via SMB or RDP to an unusual number of distinct internal destinations within a short window — this specific pattern is one of the highest-value lateral movement detections a SOC can build.

**Say this aloud to the interviewer:**
> "Fifteen internal hosts over SMB and RDP in twenty minutes from one workstation is lateral movement or scanning, not normal use. I would check which connections actually succeeded, find the process and account driving it on the source host, isolate immediately, and check every successful destination for a new logon and follow-on activity."

**Key terms to mention:** lateral movement, connection success versus rejection, SMB and RDP as lateral movement protocols, tier-0 asset, propagation, correlation on distinct-destination count.

**Weak answer to avoid:** "I would monitor it a bit longer to be sure." With SMB and RDP fan-out already observed, monitoring alone risks losing control of an active spread.

**Likely follow-up:** "The connections were all rejected by host firewalls. Does that change your severity?"

---

### Q41. TLS encrypts the content of web traffic. What can you still learn from encrypted connections without decrypting them?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Whether you understand metadata-based detection, since full decryption is often unavailable or undesirable.

**Model answer (say this aloud):**
> Even without decrypting anything, TLS metadata tells me a lot. The Server Name Indication field in the handshake shows which domain the client is connecting to, even over HTTPS. The certificate itself has an issuer, validity period, and subject that can look suspicious — a certificate issued minutes before use, or a self-signed certificate where a trusted one is expected. JA3 or JA3S fingerprints characterise the client and server's TLS implementation, which can identify a specific malware family or tool regardless of the domain it is using. And connection metadata — timing, size, frequency — still shows beaconing patterns even when I cannot see a single byte of content. So my analysis shifts from content to behaviour and fingerprint.

**Deeper explanation:**
Useful signals without decryption: SNI (client-declared hostname), certificate chain details (issuer, age, self-signed status, subject alternative names), JA3/JA3S client and server fingerprints, cipher suite selection, connection size and timing patterns, and destination IP and ASN reputation. Where an organisation runs TLS inspection at the proxy for managed devices, full content becomes visible there, but privacy and policy constraints usually apply and should be respected — inspection typically excludes categories like banking and health. A practical point to make: a mismatch between the SNI and the certificate's subject, or a JA3 fingerprint matching a known malware family regardless of the domain used, are both strong, decryption-free detections.

**Key terms to mention:** SNI, JA3/JA3S fingerprint, certificate metadata, self-signed certificate, cipher suite, TLS inspection and its policy limits, beaconing by size and timing.

**Weak answer to avoid:** "You can't see anything if it's encrypted." Undersells a large amount of legitimate, widely used detection capability.

**Likely follow-up:** "What is a JA3 fingerprint and why is it useful even if the attacker changes domains?"

---

### Q42. `Scenario-based` At 23:00 a workstation on a standard user's account uploads 4 GB to an external cloud storage domain that the company does not officially use. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Data exfiltration triage — distinguishing insider action from account compromise, and evidence-first handling.

**Model answer (say this aloud):**
> A large after-hours upload to an unsanctioned cloud service is a strong exfiltration indicator, so I treat it seriously immediately, but I keep two hypotheses open — account compromise, or a genuine insider action — until the evidence points one way. I check whether the login session itself looks legitimate: was it from the user's normal device and location, was MFA satisfied properly. I check what was actually uploaded, without unnecessarily exposing the content, by looking at file names, types, and the source folders that were accessed just before the upload. Given the sensitivity of this in a government environment, this is exactly the kind of case I escalate immediately and let the incident manager decide how HR and legal are engaged — that is not a decision for me to make alone.

**Scenario walkthrough**

**Initial alert or situation**
Large after-hours outbound data transfer from a standard user's workstation to an unsanctioned cloud storage domain.

**Investigation steps, in order**
1. Confirm the transfer details from proxy and firewall logs: destination domain, total bytes, duration, and whether it was continuous or in bursts.
2. Check the identity session: was this the legitimate user's normal device, location, and time pattern, and was authentication fully satisfied including MFA.
3. Check endpoint activity immediately before the upload: which folders and files were accessed, copied, or compressed, using file access and process telemetry.
4. Determine file types and naming from metadata where possible, without opening or exposing sensitive content unnecessarily.
5. Check whether this pattern is new for this user or has happened before, and whether other users show the same pattern to the same destination.
6. Check for other exfiltration indicators around the same time — email attachments sent externally, USB device activity, or printing.
7. Preserve the evidence trail carefully, since this may become a personnel matter as well as a security one.

**Evidence and log sources to review**
Proxy and firewall logs for the transfer, identity sign-in logs for the session, EDR file access and process telemetry, DLP alerts if available, email and removable media logs for corroborating activity, and HR-relevant context only through the proper channel, never accessed directly by the analyst.

**Severity and business impact reasoning**
High to Critical depending on data sensitivity, which in a government or high-trust environment should be assumed sensitive until classified otherwise. Confidentiality impact drives severity here, and the after-hours timing raises concern regardless of the eventual cause.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Preserve logs and evidence immediately, before they age out | Disabling the user's account |
| Block the destination domain per standing procedure if it is clearly unauthorised | Contacting the user directly |
| Revoke active sessions if account compromise is suspected | Engaging HR or legal |
| Document facts precisely and objectively | Any communication framing this as confirmed wrongdoing |

**Escalation and communication**
Escalate immediately and factually, with no speculation about motive in writing. This is a case where the incident manager, HR, and potentially legal need to be involved, and the decision to contact the user or their manager belongs above the analyst level. Confidentiality of the investigation itself matters — this is not discussed outside the defined reporting chain.

**Recovery, lessons learned, detection improvement**
Recovery depends on findings: if compromise, standard account recovery; if insider action, handled entirely outside the SOC through HR and legal process, with the SOC providing evidence only. Lessons learned: was there a DLP or upload-size control on unsanctioned cloud destinations. Improvement: alert on large uploads to uncategorised or unsanctioned cloud storage domains, particularly outside business hours, and consider blocking unsanctioned cloud storage categories at the proxy by policy.

**Say this aloud to the interviewer:**
> "I would keep both account compromise and insider action open as hypotheses and let the evidence decide — checking the session legitimacy and what was accessed before the upload. I would preserve evidence immediately and escalate straight away, because engaging HR or contacting the user is a decision above my level, not something I do myself."

**Key terms to mention:** exfiltration indicator, unsanctioned cloud storage, DLP, chain of custody, personnel-sensitive escalation, confidentiality of the investigation itself.

**Weak answer to avoid:** "I would message the user to ask what they uploaded." Directly contacting a possible insider or a possibly compromised user before containment and proper authorisation can destroy evidence and is not the analyst's call.

**Likely follow-up:** "The user is on a performance improvement plan, information you were not supposed to know. Does that change what you write in the ticket?"

---

### Q43. `Scenario-based` VPN logs show the same user account connecting from two countries eight hours apart, both sessions apparently active. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** Impossible travel reasoning applied to VPN rather than cloud sign-in, and precise evidence checking.

**Model answer (say this aloud):**
> Two active sessions from two countries is either impossible travel, meaning the credentials are compromised, or a legitimate explanation like a VPN client roaming across exit nodes, split tunnelling, or corporate infrastructure using shared address space. I do not assume either. I check the physical plausibility first — could a person actually travel that distance in that time. Then I check whether both sessions are genuinely concurrent or whether one simply has a stale session that never properly closed. I check the device fingerprint and MFA method for each session, because if the two sessions used different devices and different MFA methods, that strongly supports compromise rather than a client quirk.

**Scenario walkthrough**

**Initial alert or situation**
Same VPN account shows connections from two geographically distant countries within a timeframe that appears to make legitimate travel impossible.

**Investigation steps, in order**
1. Confirm both sessions are genuinely active concurrently, not one stale session still showing as connected after the client disconnected uncleanly.
2. Calculate travel feasibility between the two locations in the elapsed time.
3. Compare device identifiers, client versions, and MFA method used for each session — a real user typically has one device signature.
4. Check what each session actually did: which internal resources were accessed, and whether the activity looks like normal work or reconnaissance.
5. Check the authentication method for each session — was MFA satisfied by push approval, a code, or something else, and was it prompted and approved quickly, which suggests attentiveness, or approved instantly without user awareness, which suggests fatigue or automation.
6. Check whether the account uses a corporate VPN client with a known exit node pattern, or third-party infrastructure that could explain apparent geographic movement.
7. Contact the user through an out-of-band channel to confirm their actual location and activity, once other checks support that this is warranted.

**Evidence and log sources to review**
VPN concentrator session logs with concurrent session state, device and client fingerprint data, MFA logs showing method and response time, resource access logs during each session, and identity provider sign-in logs if the VPN is federated.

**Severity and business impact reasoning**
High until explained. A confirmed compromised VPN credential grants network-level access, which is a serious foothold regardless of what has been accessed yet.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Terminate the suspicious session per standing procedure | Disabling the account entirely before confirming with the user |
| Force MFA re-authentication or password reset | Blocking a whole geography or exit-node range that serves legitimate users |
| Preserve session and access logs | Notifying the user's management |

**Escalation and communication**
If compromise is confirmed or strongly suspected, escalate immediately with both sessions' details and what each accessed. If the explanation turns out to be benign — a known client quirk — document it clearly so the same alert is not re-investigated from scratch next time.

**Recovery, lessons learned, detection improvement**
Recovery: terminate compromised sessions, rotate credentials, and review everything accessed during the illegitimate session. Lessons learned: whether the VPN platform reliably reports session state, since stale-session false alarms waste investigation time if not handled well. Detection improvement: an impossible-travel rule for VPN sessions using the same logic as cloud sign-in impossible travel, and device-fingerprint mismatch as a corroborating signal.

**Say this aloud to the interviewer:**
> "I would check plausibility of travel, then check whether the sessions are truly concurrent or one is stale, then compare device and MFA signatures between them. A same-device explanation points to a client or session-state quirk; different devices and different MFA methods points to compromise, and I would terminate the suspicious session and force re-authentication."

**Key terms to mention:** impossible travel, concurrent session verification, device fingerprint, MFA method comparison, out-of-band user verification, stale session state.

**Weak answer to avoid:** "I would assume it's just a VPN glitch." Dismissing without checking device and MFA evidence is exactly how a real compromise gets waved through.

**Likely follow-up:** "The user confirms they are only in one of the two locations. What is your next step?"

---

### Q44. How would you identify command-and-control beaconing purely from NetFlow data, with no payload visibility?

- **Difficulty:** Advanced
- **Type:** Standard
- **What the interviewer is testing:** Statistical/behavioural analysis skill beyond signature matching.

**Model answer (say this aloud):**
> Beaconing has a statistical signature even without seeing any content: regular timing between connections to the same destination, low variance in that interval, and often a consistent or near-consistent payload size. So I group flow records by source and destination pair, calculate the time between successive connections, and look for a low coefficient of variation in that interval — real user browsing is bursty and irregular, while malware beacons on a near-fixed schedule with small random jitter added deliberately to evade naive detection. I also look at the destination: is it a rare or unique destination for the organisation, does it have a low reputation or a very recent registration, and is the connection count high relative to how well-known the destination is.

**Deeper explanation:**
Practical approach: group by source-destination pair over a rolling window, compute inter-arrival times, and calculate variance or coefficient of variation — beaconing typically shows a tight interval distribution even with jitter, since jitter is usually a small percentage of the base interval rather than fully random. Combine with: connection count (persistent, repeated contact), payload size consistency, destination rarity across the environment (a destination only one host talks to is more suspicious than one many hosts share), and destination age or reputation. This is the same underlying technique used by beaconing-detection features in modern network security tools, and describing it in these terms — interval regularity, jitter tolerance, destination rarity — signals genuine understanding rather than a memorised buzzword.

**Key terms to mention:** inter-arrival time, coefficient of variation, jitter, destination rarity, payload size consistency, rolling window analysis.

**Likely follow-up:** "How would jitter of up to 20% affect your detection logic, and how would you tolerate it?"

**Weak answer to avoid:** "I would look for a lot of connections to the same IP." Volume alone misses low-and-slow beacons and false-positives on legitimate polling services.

---

### Q45. `Scenario-based` A threat hunt turns up a host making an extremely high volume of DNS queries to subdomains of one external domain, all with long, random-looking labels. What do you do?

- **Difficulty:** Advanced
- **Type:** Scenario-based
- **What the interviewer is testing:** End-to-end handling of a DNS tunnelling finding, since it is a favourite scenario topic.

**Model answer (say this aloud):**
> This is a strong DNS tunnelling pattern — long, high-entropy subdomains and high query volume to one parent domain is exactly what encoding data into DNS looks like. I would not wait for more proof before acting, because tunnelling is an active exfiltration or C2 channel by definition once confirmed. I identify the process and account on the host driving the queries, I check what data could be leaving through this channel, and I contain the host. Then I check whether other hosts show the same pattern to the same or similar parent domains, because tunnelling tools are often deployed identically across compromised machines.

**Scenario walkthrough**

**Initial alert or situation**
One host generating a very high volume of DNS queries with long, random-looking subdomain labels against a single external parent domain, found during proactive hunting.

**Investigation steps, in order**
1. Confirm the pattern statistically: query volume per minute, label length distribution, and entropy of the subdomain labels compared with normal traffic.
2. Identify the querying process on the host through EDR or DNS client correlation, along with its parent process and command line.
3. Determine how long this has been happening by checking DNS logs historically for the same host and domain.
4. Check the record types being queried — TXT and NULL records are common in tunnelling tools, more so than typical A record lookups.
5. Estimate what could realistically have been exfiltrated given the volume and duration, understanding DNS tunnelling throughput is low but sustained over time can move meaningful data.
6. Check the parent domain's registration date and reputation, and check whether it resolves anywhere or is only ever queried, which is also consistent with tunnelling.
7. Sweep other hosts for queries to the same parent domain or the same query pattern.

**Evidence and log sources to review**
DNS query logs (source host, full query name, record type, response, volume over time), EDR process telemetry correlated to the DNS client activity, domain registration and reputation data, and an estate-wide DNS query sweep.

**Severity and business impact reasoning**
High to Critical. A live tunnelling channel is both a C2 and exfiltration risk simultaneously, and the fact it survived until a proactive hunt found it means it evaded existing detections.

**Containment actions — analyst vs approval required**

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the host via EDR | Blocking the parent domain organisation-wide if it has any shared legitimate use |
| Block the specific malicious subdomain pattern per standing procedure | Changes to DNS resolver policy |
| Preserve evidence and the responsible binary | Estate-wide DNS filtering policy changes |
| Sweep for the same pattern across the estate | |

**Escalation and communication**
Escalate as a confirmed or near-confirmed active tunnelling channel with likely exfiltration. State the estimated duration and, if calculable, a rough estimate of data volume that could have moved, since that materially affects how leadership treats the case.

**Recovery, lessons learned, detection improvement**
Recovery: rebuild the host, rotate any credentials it held, and review what sensitive data it had access to. Lessons learned: why did existing detections miss a high-volume, long-duration channel — often because DNS query volume per host was never baselined. Detection improvement: build a rule on subdomain label entropy and length combined with query volume per host per parent domain, and add DNS to the standard set of sources reviewed during every hunt, not just an occasional check.

**Say this aloud to the interviewer:**
> "Long, high-entropy subdomains at high volume to one domain is a strong tunnelling signature. I would not wait for more proof — I would identify the process and account, contain the host, estimate what could have moved through that channel given the volume and duration, and sweep the estate for the same pattern, since tunnelling tools are usually deployed identically across compromised hosts."

**Key terms to mention:** DNS tunnelling, subdomain entropy, TXT and NULL record abuse, query volume baseline, exfiltration throughput estimate, estate-wide sweep.

**Weak answer to avoid:** "I would block the domain and move on." Without identifying the process, the account, and the duration, you cannot scope what already left the network.

**Likely follow-up:** "Roughly how would you estimate how much data could have exfiltrated given the query volume you observed?"

---

### Q46. Which ports and protocols do you consider essential to know cold for this role?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Fast factual recall — this comes up almost every interview.

**Model answer (say this aloud):**
> The ones I use constantly are 53 for DNS, 80 and 443 for HTTP and HTTPS, 25, 465 and 587 for SMTP mail submission, 22 for SSH, 23 for Telnet which should almost never be seen, 3389 for RDP, 445 and 139 for SMB, 389 and 636 for LDAP and LDAPS, 88 for Kerberos, 464 for the Kerberos password change protocol, 135 plus a dynamic range for RPC, 3306 for MySQL, 1433 for MSSQL, 5985 and 5986 for WinRM, and 161 for SNMP. The ones I flag as immediately worth attention if seen unexpectedly on the perimeter are Telnet, unencrypted SNMP, and any database port exposed externally.

**Deeper explanation:**

| Port | Protocol | Notes for a SOC analyst |
|---|---|---|
| 20/21 | FTP | Cleartext credentials — a finding if seen |
| 22 | SSH | Remote administration; watch for brute force |
| 23 | Telnet | Cleartext — should not exist on a modern network |
| 25 / 465 / 587 | SMTP | Mail transport; 25 for relay, 587 for authenticated submission |
| 53 | DNS | Name resolution — also an abuse channel, see Q38 |
| 80 / 443 | HTTP / HTTPS | Web traffic |
| 88 | Kerberos | Authentication — watch 4768/4769 activity here |
| 135 / 445 / 139 | RPC / SMB | Lateral movement and file sharing |
| 161 / 162 | SNMP | Device management — v1/v2c cleartext is a finding |
| 389 / 636 | LDAP / LDAPS | Directory queries; cleartext LDAP is a finding |
| 464 | Kerberos password change | Password change traffic |
| 1433 / 3306 | MSSQL / MySQL | Databases — should never be internet-facing |
| 3389 | RDP | Remote desktop — common ransomware entry point if exposed |
| 5985 / 5986 | WinRM | Remote PowerShell management |

Mention that seeing a well-known port used for the wrong protocol — for example, non-HTTP traffic on port 443 — is itself a detection technique, since malware often reuses common ports to blend in.

**Key terms to mention:** cleartext protocols as findings, exposed database ports, RDP as a ransomware entry vector, port-protocol mismatch.

**Weak answer to avoid:** Listing ports with no security relevance attached. Always pair the port with why it matters to a SOC.

**Likely follow-up:** "You see traffic on port 443 that is not actually TLS. What does that suggest?"

---

### Q47. What is the difference between an IDS and an IPS, and how does that affect your day-to-day workflow?

- **Difficulty:** Core
- **Type:** Standard
- **What the interviewer is testing:** Basic but frequently asked network security fundamentals.

**Model answer (say this aloud):**
> An intrusion detection system monitors traffic and generates an alert, but it does not act on the traffic itself — it is out of band or passively tapped. An intrusion prevention system sits inline in the traffic path and can actively block or drop malicious traffic in real time. The practical difference for me is workload and risk: IDS alerts always need my triage because nothing was blocked automatically, whereas IPS blocks reduce my triage burden but carry the risk of blocking legitimate traffic, so IPS rules need to be much more confidently tuned before they go into blocking mode. Many organisations run new detections in IDS or alert-only mode first, then promote them to IPS blocking mode once confidence in low false positives is established.

**Deeper explanation:**
Key operational implications: IDS requires no availability risk to deploy since it does not sit inline, but it means the attack traffic already reached its destination — response is always after the fact. IPS can stop the attack in real time but a badly tuned inline rule can cause a self-inflicted outage, which is why staged rollout — alert-only, then blocking — is standard practice. From an analyst's perspective, an IDS alert queue needs full triage discipline since nothing has been mitigated, while an IPS block event still needs review to confirm it was correct and was not a false positive causing business impact, similar to the containment-mistake scenario in incident response.

**Key terms to mention:** inline versus out-of-band, real-time blocking, staged rollout from alert-only to blocking, false-positive blocking risk, post-event triage versus prevention.

**Weak answer to avoid:** "IPS is just a better IDS." Missing the inline placement and the availability trade-off misses the point entirely.

**Likely follow-up:** "Would you ever put a brand-new detection straight into IPS blocking mode? Why or why not?"

---

[⬅ Previous: Windows & AD](03-windows-active-directory-and-identity.md) · [Back to README](../README.md) · [Next: Azure & M365 ➡](05-azure-microsoft-365-and-cloud-security.md)
