# Scenario Playbooks

**12 end-to-end incident playbooks**, each with a Mermaid flowchart showing the full lifecycle: **alert → validation → investigation → containment → escalation → eradication/recovery → lessons learned.**

These are designed to be read the night before or morning of the interview — skim the flowchart to fix the shape of the response in your head, then read the "say this aloud" line so you have a ready answer if the interviewer opens with "walk me through how you'd handle X."

Each playbook follows the same structure as the scenario-based questions in [`questions/`](../questions/), but presented as a complete standalone walkthrough rather than tied to one specific question.

[⬅ Back to README](../README.md)

## The 12 Playbooks

| # | Playbook | Core Theme |
|---|----------|-------------|
| 1 | [Ransomware on a Critical Server](01-ransomware-critical-server.md) | Active destruction, speed of containment, backup integrity |
| 2 | [Microsoft 365 Account Compromise](02-m365-account-compromise.md) | Identity compromise, session tokens, mailbox rules |
| 3 | [MFA Fatigue Attack](03-mfa-fatigue-attack.md) | Push bombing, approved-prompt compromise |
| 4 | [Phishing / BEC Targeting an Executive](04-phishing-bec-executive.md) | Impersonation, financial fraud, urgency |
| 5 | [Suspicious PowerShell Execution](05-suspicious-powershell-execution.md) | Encoded commands, living-off-the-land, decode-first |
| 6 | [Brute Force Followed by Successful Login](06-bruteforce-then-success.md) | Credential access, immediate compromise response |
| 7 | [Impossible-Travel / Cloud Sign-In Alert](07-impossible-travel-signin.md) | Identity risk triage, false-positive discipline |
| 8 | [Malware / EDR Alert on a Privileged Workstation](08-malware-privileged-workstation.md) | Blast radius of privileged credentials |
| 9 | [Suspected Lateral Movement](09-suspected-lateral-movement.md) | Internal spread, SMB/RDP fan-out, rapid isolation |
| 10 | [Data Exfiltration Alert](10-data-exfiltration-alert.md) | Confidentiality impact, insider vs compromise |
| 11 | [Insider-Threat Warning](11-insider-threat-warning.md) | Personnel sensitivity, evidence discipline, restricted handling |
| 12 | [DDoS / Major Availability Incident](12-ddos-availability-incident.md) | Availability impact, coordinated response at scale |

## How to use these under time pressure

If you only have a few minutes before the interview, read the **flowchart** and the **"say this aloud"** closing line of each playbook — that combination gives you the shape of a complete answer for almost any incident scenario an interviewer improvises on the spot, even one not listed here exactly, because the underlying lifecycle is always the same six stages.

[⬅ Back to README](../README.md)
