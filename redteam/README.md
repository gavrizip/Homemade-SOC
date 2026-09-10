
# Red Team

Offensive-side content for the lab, run from the isolated Attacker VLAN (40) against the Target/Victim VLAN (30). This is the counterpart to the rest of the repo, which is entirely blue-team/defensive — see `docs/reference/architecture.md` for how the two are segmented.

**Start here**: `ROE.md` — scope, authorization, and ethical boundaries that apply to everything below.

## Structure

```
redteam/
├── ROE.md                    # Rules of Engagement — read this first
├── setup/                    # ParrotOS + tooling installation
├── tools/                    # Notes per individual tool (nmap, Metasploit, Burp, etc.)
├── playbooks/                # Step-by-step attack scenarios against the Target VLAN
├── reports/                  # Results of each exercise, including what the SOC did/didn't detect
└── mitre-attack-mapping.md   # Techniques used across playbooks, mapped to MITRE ATT&CK
```

|Folder/file|Contents|
|---|---|
|`ROE.md`|Authorization, scope, and ethics for all exercises|
|`setup/`|ParrotOS installation and core toolset|
|`tools/`|What each tool is, when it's used, and why it was chosen|
|`playbooks/`|Step-by-step attack scenarios, each with expected detection points|
|`reports/`|Actual results per playbook run, cross-referenced against blue-team detection|
|`mitre-attack-mapping.md`|Which ATT&CK techniques each playbook exercises|

## Purpose

Every playbook here exists to test the SOC stack built in the rest of this repo — this is a **purple team** loop, not offensive security for its own sake. Each report closes by feeding findings back into blue-team tuning (Wazuh rules, Suricata categories, ModSecurity config), documented in `docs/journal/`.