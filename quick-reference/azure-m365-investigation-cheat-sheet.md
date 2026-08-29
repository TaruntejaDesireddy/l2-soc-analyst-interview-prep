# Quick Reference · Azure / Microsoft 365 Investigation Cheat Sheet

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

## KQL tables — where to look first

| Table | Contents | Common use |
|---|---|---|
| `SigninLogs` | Interactive sign-ins | Risky sign-in, impossible travel, MFA verification |
| `AADNonInteractiveUserSignInLogs` | Token refresh, background auth | **Often missed** — check this alongside SigninLogs every time |
| `AuditLogs` | Directory changes | Role assignment, group membership, app consent |
| `OfficeActivity` | Exchange/SharePoint/OneDrive/Teams actions | Inbox rules, mail sent, file sharing |
| `DeviceProcessEvents` | Endpoint process creation (Defender for Endpoint) | Execution, command-line analysis |
| `DeviceNetworkEvents` | Endpoint network connections | C2, beaconing |
| `DeviceLogonEvents` | Endpoint logon activity | Logon type, source, account |
| `IdentityLogonEvents` | On-prem AD auth via Defender for Identity | Kerberos abuse, NTLM relay |
| `EmailEvents` / `EmailUrlInfo` / `EmailAttachmentInfo` | Mail flow and content metadata | Phishing triage |

## Identity investigation checklist — every risky sign-in

1. Which specific risk detection fired? (each implies different evidence)
2. Source IP reputation, location, device, browser
3. Interactive or non-interactive?
4. **How was MFA satisfied** — real push, or legacy auth that bypassed it entirely?
5. Compare against the account's normal baseline
6. Post-sign-in: new inbox rules, new MFA methods, new app consents
7. What did the session actually touch?

## Legacy authentication — why it matters (say this cold)

> Legacy protocols (POP, IMAP, SMTP AUTH, older ActiveSync) **cannot enforce MFA** and are barely evaluated by Conditional Access. A stolen password alone is enough. Any successful legacy sign-in deserves more scrutiny than a modern one — check the client app field in sign-in logs.

## MFA fatigue — the one non-negotiable sequence

**Revoke sessions FIRST, investigate second.** Then check specifically whether the attacker registered their own MFA method — a password reset alone does not remove that.

## OAuth consent phishing — the fact that trips people up

> The attacker never needed the password. It's a **token grant**, not a credential — so **a password reset does nothing**. You must find the specific app and revoke its access directly.

## BEC / mailbox rule red flags

- Hidden inbox rule: forward externally + delete/move to hidden folder = near-zero false positive rate
- Transport-level mail flow rule created by a non-admin account
- Rule filtering on finance/invoice keywords

## Azure control-plane — the equivalent of "Domain Admin" here

**Subscription Owner** / **User Access Administrator** role assignment = treat exactly like a Domain Admin group addition on-premises. First-time assignment, unusual hour, no change record = high severity immediately.

## Shared Responsibility Model — the one thing the customer ALWAYS owns

Regardless of IaaS / PaaS / SaaS: **identity and access management, data classification, and secure configuration are always the customer's job.** Microsoft never monitors your tenant's identity misuse or a misconfigured storage account for you.

## Conditional Access — what to actually check in the sign-in log

The **Conditional Access section of the sign-in log** shows every applicable policy and whether it was **satisfied**, **not applied**, or **report-only**. This tells you in seconds whether a suspicious sign-in was actually challenged for MFA and passed — or slipped through because a policy didn't apply to that app or client type.

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
