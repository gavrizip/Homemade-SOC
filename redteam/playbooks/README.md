
# Playbooks

Step-by-step offensive scenarios run from the ParrotOS VM against the Target/Victim VLAN. All playbooks operate under `../ROE.md` — scope and authorization are not restated per file.

## Index

|Playbook|Scenario|ATT&CK coverage|
|---|---|---|
|`01-example-playbook.md`|Recon → credential brute-force → SQL injection|See `../mitre-attack-mapping.md`|

## Format every playbook follows

To keep results comparable across exercises, each playbook uses the same structure:

1. **Objective** — what the exercise is trying to achieve
2. **Scope** — target, attacker VM, explicit boundaries beyond the standing ROE
3. **Steps** — commands run, in order, each with an **expected detection point** called out explicitly (which blue-team tool _should_ catch this step)
4. **What to record** — pointer to the matching file in `../reports/`

## Adding a new playbook

1. Copy the structure from `01-example-playbook.md`
2. Add a row to the index table above
3. Add the corresponding technique(s) to `../mitre-attack-mapping.md`
4. Create the matching report file in `../reports/` before running it, so results get captured immediately