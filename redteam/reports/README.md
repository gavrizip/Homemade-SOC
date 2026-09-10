
# Reports

Actual results from running each playbook — this is where offensive action turns into blue-team feedback. Every report follows up with what the SOC detected, what it missed, and what changed as a result.

## Index

|Report|Corresponding playbook|Status|
|---|---|---|
|`01-example-report.md`|`../playbooks/01-example-playbook.md`|Template — fill in after running|

## Why reports exist separately from playbooks

A playbook describes what _should_ happen; a report records what _actually_ happened. Keeping them separate means a playbook can be reused multiple times (after a detection gap is fixed, for example) while each run gets its own dated report — so improvement over time is visible instead of overwritten.

## Format every report follows

1. **Exercise metadata** — date, attacker/target VMs, playbook used
2. **Timeline** — each action, whether it was detected, by which tool, and how fast
3. **Detection chain outcome** — did the full Wazuh → Shuffle → TheHive → MISP pipeline fire correctly
4. **False negatives / false positives** — the most valuable section; gaps found here drive the next fix
5. **Follow-up actions** — concrete rule/config changes made, logged in `../../docs/journal/`

## Adding a new report

1. Copy the structure from `01-example-report.md`
2. Fill it in immediately after running the corresponding playbook, while details are fresh
3. Update `../mitre-attack-mapping.md` if this closes out a "planned coverage" entry