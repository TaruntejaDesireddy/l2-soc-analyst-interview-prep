# Quick Reference · Red Flags and Escalation Triggers

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

The specific things that should make you say "this is not routine" out loud in an interview answer — and exactly what to do the moment you see them.

## Near-zero false-positive indicators — act on these fast

| Indicator | Why it's high-fidelity |
|---|---|
| Shadow copy / backup deletion command (`vssadmin`, `wmic shadowcopy delete`) | Almost no legitimate business reason to run this |
| Security tooling disabled via registry | Same — very rarely legitimate |
| Audit log cleared (event 1102) | Deliberate anti-forensics |
| Hidden inbox rule: forward externally + delete/hide | Classic BEC pattern, very low false-positive rate |
| DCSync from a non-domain-controller principal | The replication rights themselves are the tell |
| One source, failed auth against many *distinct* accounts | Password spraying, not noise |
| LSASS memory access by a non-allowlisted process | Credential dumping |

## Escalate immediately — don't wait to finish the investigation first

- Scope expands mid-investigation (one host → multiple hosts/servers)
- Any evidence of active data exfiltration
- A privileged/tier-0 account or domain controller becomes involved
- You find precursor ransomware behaviour, even with **no ransom note yet**
- The same low-severity alert pattern appears linked across multiple entities (correlation, not coincidence)
- A detection rule that normally fires regularly has gone silent — silence is a finding, not a relief
- Anything involving a colleague, personnel matter, or conflict of interest

## Wake-your-manager-at-2am triggers

- Ransomware on shared/critical infrastructure
- Confirmed domain-wide or multi-server compromise
- Confirmed data exfiltration involving sensitive data
- Active DDoS on a primary public-facing service
- Suspected insider threat or personnel-sensitive finding
- Anything where the honest answer to "can this wait until morning" is no

## The authority boundary — say it in one breath

> "I can isolate a single host, revoke sessions, block a confirmed-malicious indicator, and preserve evidence — all under standing procedure. Anything domain-wide, anything touching a privileged account, anything that reverses someone else's decision, or anything involving direct contact with a person in a personnel-sensitive case — that goes to the incident manager, not me alone."

## Signs an alert needs re-scoping, not just closing

- The "explanation" is a guess, not a checked fact (no change ticket found, no confirmed scanner schedule)
- Corroborating telemetry from a second source doesn't actually agree with the first
- The account or host involved is privileged, even if the activity looks minor
- Closing it would mean not documenting *why* — if you can't write one clean sentence justifying closure, it isn't ready to close

## Integrity red flags — in yourself and in a request from others

| Situation | What to do |
|---|---|
| You made a containment mistake | Report it yourself, immediately, before being asked |
| Asked to skip evidence documentation "just this once" | Push back respectfully, offer a faster compliant alternative |
| You recognise a personal connection to a case | Disclose immediately, request reassignment, discuss with no one |
| A colleague accessed something outside their scope | Report factually through the proper channel — don't handle it informally |
| Conflicting instructions from two people with authority | Facilitate them talking directly; don't silently pick a side |

## The single sentence to keep in your back pocket for any pressure question

> "I don't skip the process under pressure — a rushed shortcut that damages evidence or trust almost always costs more time later than the delay would have."

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
