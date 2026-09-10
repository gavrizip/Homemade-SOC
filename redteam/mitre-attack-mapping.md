
# MITRE ATT&CK Mapping

Maps the techniques exercised across `/redteam/playbooks/` to the [MITRE ATT&CK](https://attack.mitre.org/) framework (Enterprise Matrix). Updated every time a new playbook is added — this is a living document, not a one-time write-up.

## Why this exists

A playbook that says "I ran sqlmap" tells you _what tool_ was used. Mapping it to ATT&CK tells you _what technique class_ it represents — which is what actually matters for measuring SOC coverage: the goal isn't "did we detect sqlmap specifically," it's "do we detect Exploitation of Public-Facing Applications regardless of which tool produces it."

## Coverage table

|Playbook|Tactic|Technique ID|Technique Name|Detected in this lab?|
|---|---|---|---|---|
|`01-example-playbook.md`|Reconnaissance|[T1595](https://attack.mitre.org/techniques/T1595/)|Active Scanning|See `reports/01-example-report.md`|
|`01-example-playbook.md`|Discovery|[T1046](https://attack.mitre.org/techniques/T1046/)|Network Service Discovery|See report|
|`01-example-playbook.md`|Credential Access|[T1110.001](https://attack.mitre.org/techniques/T1110/001/)|Brute Force: Password Guessing|See report|
|`01-example-playbook.md`|Initial Access|[T1190](https://attack.mitre.org/techniques/T1190/)|Exploit Public-Facing Application (SQL injection)|See report|
|`01-example-playbook.md`|Discovery|[T1082](https://attack.mitre.org/techniques/T1082/)|System Information Discovery (post-exploitation)|See report|

_"Detected in this lab?" links out to the report rather than duplicating a yes/no here — the report is the source of truth and this table can go stale if results are copied in twice._

## Planned coverage (future playbooks)

|Planned playbook|Tactic|Technique ID|Technique Name|
|---|---|---|---|
|Privilege escalation / lateral movement|Privilege Escalation|[T1068](https://attack.mitre.org/techniques/T1068/)|Exploitation for Privilege Escalation|
|Privilege escalation / lateral movement|Lateral Movement|[T1021](https://attack.mitre.org/techniques/T1021/)|Remote Services|
|Phishing / social engineering|Initial Access|[T1566](https://attack.mitre.org/techniques/T1566/)|Phishing|
|Network attacks (ARP/MITM)|Credential Access|[T1557](https://attack.mitre.org/techniques/T1557/)|Adversary-in-the-Middle|
|Additional web scenarios|Initial Access|[T1190](https://attack.mitre.org/techniques/T1190/)|Exploit Public-Facing Application (other vuln classes)|

## How to update this file

When adding a new playbook:

1. Identify the ATT&CK tactic/technique each step maps to (attack.mitre.org has a searchable technique index).
2. Add a row to the coverage table once the playbook has a corresponding report with results.
3. Move the entry out of "Planned coverage" once it's real.


