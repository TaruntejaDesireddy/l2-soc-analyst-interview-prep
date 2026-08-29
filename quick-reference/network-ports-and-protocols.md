# Quick Reference · Network Ports and Protocols

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)

## Ports to know cold

| Port | Protocol | SOC-relevant note |
|---|---|---|
| 20/21 | FTP | Cleartext credentials — a finding if seen |
| 22 | SSH | Watch for brute force |
| 23 | Telnet | Cleartext — should not exist on a modern network |
| 25 | SMTP | Mail relay |
| 53 | DNS | Name resolution **and** a major abuse channel (tunnelling, DGA, fast flux) |
| 80 / 443 | HTTP / HTTPS | Web traffic — watch for port/protocol mismatch (non-TLS on 443) |
| 88 | Kerberos | Watch 4768/4769 activity |
| 135 | RPC | Lateral movement |
| 139 / 445 | NetBIOS / SMB | File sharing, lateral movement, ransomware spread |
| 161 / 162 | SNMP | v1/v2c cleartext is a finding |
| 389 / 636 | LDAP / LDAPS | Cleartext LDAP is a finding |
| 464 | Kerberos password change | |
| 465 / 587 | SMTP over TLS / submission | Authenticated mail |
| 1433 | MSSQL | Should never be internet-facing |
| 3306 | MySQL | Should never be internet-facing |
| 3389 | RDP | Common ransomware entry point if exposed to the internet |
| 5985 / 5986 | WinRM | Remote PowerShell management |

**High-value fact:** a well-known port carrying the *wrong* protocol (e.g., raw non-TLS traffic on 443) is itself a detection technique — malware reuses common ports specifically to blend into normal traffic.

## DNS abuse — the three patterns

| Technique | What it looks like |
|---|---|
| **Tunnelling** | Long/high-entropy subdomains, high query rate to one parent domain, TXT/NULL record abuse |
| **DGA** (domain generation algorithm) | High NXDOMAIN ratio from one host, high-entropy names, no dictionary structure |
| **Fast flux** | One domain resolving to many distinct IPs, very low TTL |

## TLS metadata you can still use without decrypting anything

| Signal | What it tells you |
|---|---|
| **SNI** (Server Name Indication) | Client-declared hostname, visible even over HTTPS |
| **Certificate** | Issuer, age, self-signed status, subject mismatch |
| **JA3 / JA3S fingerprint** | Identifies the client/server TLS implementation — can fingerprint a malware family regardless of domain used |
| **Cipher suite** | Unusual selections can stand out |
| **Connection size/timing** | Beaconing is visible even with zero content visibility |

## Beaconing detection — the shape of the logic

```
Group by (source, destination) pair
  → compute inter-arrival time between connections
  → low variance / coefficient of variation = regular beacon (jitter tolerance ~small % of base interval)
  → combine with: destination rarity, payload size consistency, destination reputation/age
```

## Lateral movement protocols — the pair to watch together

**SMB (445) + RDP (3389)** from one internal host to many internal destinations in a short window is one of the highest-value lateral movement signals a SOC can build. Always check **success vs rejection** first — it changes urgency completely.

## IDS vs IPS — the one-line distinction

| | Placement | Action |
|---|---|---|
| **IDS** | Out-of-band / passive tap | Detects and alerts only — always needs full triage |
| **IPS** | Inline | Can block in real time — stage new rules as alert-only before promoting to blocking mode |

[⬅ Back to README](../README.md) · [Quick Reference Index](README.md)
