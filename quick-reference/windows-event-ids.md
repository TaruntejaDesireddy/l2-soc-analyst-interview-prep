# Quick Reference · Windows Event IDs

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

The list to have cold. Grouped by what you're investigating, not numerical order — that's how you'll actually be asked about them.

## Authentication & logon

| Event ID | Meaning | Why it matters |
|---|---|---|
| **4624** | Successful logon | Check the **logon type** field — it tells you *how* |
| **4625** | Failed logon | Brute force / spraying — group by distinct-account count for spraying |
| **4634 / 4647** | Logoff (system / user-initiated) | Session duration for timelines |
| **4648** | Logon with explicit credentials | `runas`, lateral movement with alternate creds |
| **4672** | Special privileges assigned | Admin-level logon occurred — is it expected here? |
| **4740** | Account locked out | Check the **caller computer name** field for the source |
| **4776** | NTLM credential validation | Legacy/NTLM authentication in use |

## Logon types (the field inside 4624/4625 that matters most)

| Type | Name | Meaning |
|---|---|---|
| 2 | Interactive | Console logon |
| 3 | Network | SMB, remote admin — most lateral movement shows here |
| 4 | Batch | Scheduled task |
| 5 | Service | Service starting |
| 7 | Unlock | Workstation unlock |
| 8 | NetworkCleartext | Basic auth over the wire — credential exposure |
| 9 | NewCredentials | `runas /netonly` — strong lateral movement indicator |
| 10 | RemoteInteractive | RDP — hands-on-keyboard remote access |
| 11 | CachedInteractive | DC unreachable, cached creds used |

## Process, service & persistence

| Event ID | Meaning | Why it matters |
|---|---|---|
| **4688** | Process creation | The single most useful endpoint event — full command line |
| **4697** | Service installed (Security log) | Persistence, remote execution |
| **7045** | New service installed (System log) | Same idea — classic persistence artefact |
| **4698** | Scheduled task created | Persistence — check for disguised legitimate-looking names |

## Account & privilege management

| Event ID | Meaning | Why it matters |
|---|---|---|
| **4720** | Account created | Attacker-created accounts |
| **4722 / 4724 / 4738** | Account enabled / password reset / account changed | Account manipulation |
| **4726** | Account deleted | Anti-forensics or cleanup |
| **4728 / 4732 / 4756** | Member added to global / local / universal group | **Privilege escalation** — treat additions to Domain Admins like a fire alarm |

## Kerberos (domain controller logs)

| Event ID | Meaning | Why it matters |
|---|---|---|
| **4768** | TGT request | Baseline Kerberos auth |
| **4769** | Service ticket (TGS) request | **Kerberoasting** — many distinct services, one account, short window, RC4 encryption |
| **4771** | Kerberos pre-authentication failed | Spraying/brute force against Kerberos |
| **4662** | Operation on a directory object | **DCSync** when replication rights are used by a non-DC principal |

## Anti-forensics — always high priority

| Event ID | Meaning |
|---|---|
| **1102** | Security audit log cleared — treat as an incident on its own, never dismiss |

## Sysmon (where deployed) — the enrichment layer

| Event ID | Meaning |
|---|---|
| 1 | Process creation (with hashes) |
| 3 | Network connection |
| 7 | Image/DLL loaded |
| 8 | Remote thread creation (injection) |
| 10 | Process access — **used to catch LSASS credential dumping** |
| 11 | File creation |
| 13 | Registry value set |
| 22 | DNS query |

## PowerShell logging

| Log | What it captures |
|---|---|
| **Event 4104** (PowerShell Operational) | Script block logging — the actual decoded code that ran |
| Module logging | Pipeline execution details |

## Two facts worth memorising verbatim

- **Kerberoasting** generates **no failed logons** — it abuses a legitimate feature (TGS requests for any SPN account), so 4769 volume-per-account is your only lever.
- **DCSync** never touches the domain controller's shell — it's a replication *request*, so the tell is the requesting principal not being a DC computer account.

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
