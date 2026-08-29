# Quick Reference · L2 Incident Triage Framework

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

One-page version of "what do I actually do, in order" — read this last, right before you go in.

## The universal sequence

```
ACKNOWLEDGE → READ → SCOPE → VALIDATE → TIMELINE → CLASSIFY → CONTAIN → ESCALATE → ERADICATE → RECOVER → DOCUMENT → LESSONS LEARNED
```

| Step | What you actually do | Time budget |
|---|---|---|
| **Acknowledge** | Take ownership so no one duplicates work | Immediate |
| **Read** | Detection rule, timestamp (UTC), source, account, severity | 1–2 min |
| **Scope** | What asset — DC? server? standard laptop? — decides business impact | 1–2 min |
| **Validate** | Pull raw logs, not just the alert summary. Check historical disposition of this rule | 3–5 min |
| **Timeline** | Build what happened just before / after in UTC | 5–10 min |
| **Classify** | Severity + true positive / false positive / benign true positive, with a written reason | — |
| **Contain** | Only within your authority — see the boundary table below | Immediate once decided |
| **Escalate** | Shift lead / incident manager, factual, with what's confirmed vs unconfirmed | Immediate for High/Critical |
| **Eradicate** | Only after scope is understood — not before | — |
| **Recover** | Coordinated, monitored, criterion for "closed" agreed in advance | — |
| **Document** | UTC timeline, facts vs assessment, approver recorded, evidence hashed | Throughout, not just at the end |
| **Lessons learned** | What detection gap let this go undetected longer than it should have | Post-incident |

## Severity quick-reference

| Signal | Push severity UP |
|---|---|
| Domain controller, tier-0 asset, or privileged account involved | ↑ |
| Confirmed data exfiltration or staging | ↑↑ |
| Active spread / lateral movement confirmed | ↑↑ |
| MFA bypassed via legacy auth | ↑ |
| Backup / shadow copy deletion attempted | ↑↑ Critical |
| Security tooling disabled | ↑↑ Critical |
| Multiple hosts, same indicator | ↑↑ |
| Benign explanation confirmed (change ticket, known admin tool) | ↓ |

## The containment authority boundary — memorise this shape

**I can do myself, immediately, under standing procedure:**
- Isolate a single endpoint via EDR
- Revoke sessions / force password reset on a compromised account
- Block a confirmed-malicious IP/domain/hash per pre-approved lists
- Preserve evidence (memory, logs, hashes)
- Remove a malicious mailbox rule *after* documenting it first

**Always needs approval / escalation:**
- Disabling a privileged, VIP, or service account
- Isolating multiple production servers or a domain controller
- Any domain-wide or organisation-wide action
- Contacting the user/individual directly in a personnel-sensitive case
- Powering off a host (destroys volatile evidence)
- Reversing someone else's containment decision

## The three-question gut check for any new scenario

Ask yourself, in order, out loud if needed:
1. **Is it real?** (validate against raw evidence, not the alert summary)
2. **How far has it spread?** (scope — one host? one account? multiple?)
3. **Does it need to stop right now, or can it wait for full investigation?** (contain vs monitor)

If you can answer these three in under two minutes for any scenario the interviewer throws at you, you have a complete opening answer.

## Documentation habit — one sentence to remember

> **Facts. Assessment. Action. Approver. Timestamp. Every time.**

Never close an alert with no written reason. Never edit history — append corrections with a new timestamp instead.

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
