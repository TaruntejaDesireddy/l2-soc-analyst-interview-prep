# Quick Reference · MITRE ATT&CK Cheat Sheet

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

## The structure, in one line

**Tactics** (the *why* — attacker's objective) → **Techniques** and **Sub-techniques** (the *how*) → each technique has documented **procedures** (real-world examples), **detection guidance**, and **mitigations**.

## The 14 tactics, in typical attack order

| # | Tactic | Plain-language meaning |
|---|---|---|
| 1 | Reconnaissance | Gathering information about the target |
| 2 | Resource Development | Building attack infrastructure |
| 3 | **Initial Access** | Getting in the door |
| 4 | **Execution** | Running malicious code |
| 5 | **Persistence** | Surviving a reboot |
| 6 | **Privilege Escalation** | Gaining higher-level permissions |
| 7 | **Defense Evasion** | Avoiding detection |
| 8 | **Credential Access** | Stealing account credentials |
| 9 | **Discovery** | Mapping the environment |
| 10 | **Lateral Movement** | Moving to other systems |
| 11 | **Collection** | Gathering the target data |
| 12 | **Command and Control** | Remote communication with the attacker |
| 13 | **Exfiltration** | Getting data out |
| 14 | **Impact** | Disruption, destruction, or manipulation (e.g., ransomware) |

**A typical ransomware intrusion, tactic order:** Initial Access → Execution → Persistence → Privilege Escalation → Defense Evasion → Credential Access → Discovery → Lateral Movement → Collection → Exfiltration (often, before encryption) → Impact.

## High-value techniques to be able to name and explain

| Technique | ID | Tactic | One-line description |
|---|---|---|---|
| Kerberoasting | T1558.003 | Credential Access | Request TGS tickets for SPN accounts, crack offline |
| DCSync | T1003.006 | Credential Access | Abuse replication rights to pull password hashes |
| Pass-the-Hash | T1550.002 | Lateral Movement | Authenticate with a stolen NTLM hash, no plaintext needed |
| OS Credential Dumping (LSASS) | T1003.001 | Credential Access | Read credentials from LSASS process memory |
| Phishing | T1566 | Initial Access | Malicious link or attachment |
| PowerShell | T1059.001 | Execution | Living-off-the-land script execution |
| Scheduled Task/Job | T1053 | Persistence / Priv Esc | Common persistence mechanism |
| Valid Accounts | T1078 | Defense Evasion / Persistence | Using legitimate stolen credentials — inherently hard to detect |
| Exfiltration Over Web Service | T1567 | Exfiltration | Data sent to cloud storage or similar |
| Data Encrypted for Impact | T1486 | Impact | Ransomware encryption |
| Inhibit System Recovery | T1490 | Impact | Shadow copy / backup deletion — a ransomware precursor |

## Pyramid of Pain — bottom to top, memorise the order

```
TTPs                      ← detecting this forces the attacker to rethink their whole approach (max pain)
  Tools
    Network/Host Artefacts
      Domain Names
        IP Addresses
          Hash Values      ← trivial to change, near-zero pain (min pain)
```

**The point to make out loud:** IOCs (hash/IP/domain) are fast and precise but disposable — an attacker rotates them cheaply. Behavioural/TTP-level detection is durable because changing it requires the attacker to change *how* they operate, not just *what* infrastructure they use.

## How you actually use ATT&CK day to day

1. **Incident timelines** — annotate each stage with a technique ID for unambiguous, standard language.
2. **Coverage mapping** — build a heatmap of which techniques have detections, partial detections, or none — this is how you evidence detection coverage to management rather than asserting it.
3. **Threat intel → hunts** — a report naming a group's known techniques becomes a direct checklist of hunt hypotheses.

## IOC vs behavioural detection — the one-sentence answer

> "IOCs are fast and disposable; behavioural/TTP detection is slower to build but durable. I use both — IOCs for immediate tactical blocking of a known active threat, and I build the durable behavioural detection alongside it so the next variant is still caught."

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
