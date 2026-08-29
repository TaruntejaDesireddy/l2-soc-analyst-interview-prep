# Playbook 5 · Suspicious PowerShell Execution

[⬅ Previous: Phishing/BEC](04-phishing-bec-executive.md) · [Back to Scenarios](README.md) · [Back to README](../README.md) · [Next: Brute Force → Success ➡](06-bruteforce-then-success.md)

## Flowchart

```mermaid
flowchart TD
    A[EDR alert: PowerShell with<br/>encoded command argument] --> B{Validate}
    B -->|DECODE the command first -<br/>never skip this step| C[Investigation]
    C --> C1[Identify parent process<br/>and full lineage]
    C1 --> C2[Read decoded script:<br/>network, file, registry,<br/>credential access]
    C2 --> C3[Check 4104 script block log<br/>for corroboration]
    C3 --> C4[Check what ran AFTER -<br/>child processes, persistence]
    C4 --> D{Containment}
    D -->|Analyst can do now| D1[Isolate endpoint if malicious<br/>Kill process, block hash<br/>Preserve decoded script]
    D -->|Needs approval| D2[Isolate server running<br/>scheduled automation<br/>Domain-wide PS restrictions]
    D1 --> E[Escalation]
    D2 --> E
    E --> E1[Escalate WITH decoded<br/>content included, not just<br/>"malicious PowerShell"]
    E1 --> F[Eradication / Recovery]
    F --> F1[Rebuild if execution succeeded<br/>with material impact]
    F1 --> F2[Identify how trigger reached user]
    F2 --> G[Lessons Learned]
    G --> G1[Enable script block logging<br/>everywhere]
    G1 --> G2[Alert: encoded command +<br/>non-admin parent process]
```

## Initial alert or situation

EDR flags a PowerShell process launched with a base64-encoded command argument on a user's laptop.

## Investigation steps, in order

1. **Decode the command first — this is the step that must never be skipped.** Encoding alone proves nothing; the decoded content is what actually matters.
2. **Identify the parent process and full process lineage.** PowerShell spawned by Word, Excel, or a browser is a strong indicator of a malicious document or drive-by; spawned by a management tool is more likely benign.
3. **Read the decoded script for what it actually does**: network calls and destinations, file writes, registry changes, credential access, disabling of security tools, or a download-and-execute pattern.
4. **Check PowerShell script block logging (event 4104)** for independent corroboration of the same content.
5. **Check what happened after execution** — child processes, new files, new persistence artefacts, outbound connections.
6. **Check the delivery vector** — a recently opened email attachment, document, or link that could explain the launch.
7. **Check the script against known offensive tooling patterns** — reflective loading, AMSI bypass strings, obfuscation techniques.

## Evidence and log sources to review

EDR process telemetry with the full command line, PowerShell script block logging (4104) and module logging, process lineage, file creation events following execution, network telemetry for any destination contacted, and the source of the initial trigger.

## Severity and business impact reasoning

Severity is driven entirely by the **decoded content and parent process**, never by the presence of encoding itself. A download-and-execute pattern from an Office parent process is High to Critical; an encoded command from a known management tool with benign content is Low.

## Containment actions — analyst vs approval required

| I can do immediately as L2 | Requires approval / escalation |
|---|---|
| Isolate the endpoint if decoded content is malicious | Isolating a server running scheduled automation without checking impact first |
| Kill the process and block the associated hash | Domain-wide PowerShell execution restrictions |
| Preserve the decoded script and process tree as evidence | |

## Escalation and communication

If malicious, escalate with the **decoded script content included** in the report — never summarise it as just "malicious PowerShell." Show the actual commands so the incident manager and any responders understand exactly what it attempted.

## Recovery, lessons learned, detection improvement

**Recovery:** rebuild if execution succeeded with material impact, otherwise clean and verify.

**Lessons learned:** how did the triggering file or link reach the user in the first place.

**Detection improvement:** enable and centrally collect script block logging everywhere, alert on encoded commands combined with a non-administrative parent process, and alert on known AMSI-bypass or obfuscation strings appearing in decoded content.

## Say this aloud to the interviewer

> "Encoding by itself proves nothing — admins use it too — so I always decode first and read exactly what the script does before deciding anything. The parent process tells me a lot: PowerShell from Word or a browser is a strong signal on its own. I set severity from the decoded content and what happened afterward, never from the fact that it was encoded in the first place."

---

[⬅ Previous: Phishing/BEC](04-phishing-bec-executive.md) · [Back to Scenarios](README.md) · [Back to README](../README.md) · [Next: Brute Force → Success ➡](06-bruteforce-then-success.md)
