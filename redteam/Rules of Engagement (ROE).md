
# Rules of Engagement (ROE)

This document defines the scope, authorization, and ethical boundaries for every exercise documented under `/redteam`. Every playbook in this repo operates under these rules — they aren't restated per playbook, they're assumed.

## Authorization

All activity documented in `/redteam` is performed against **systems I own and control**, inside an isolated home lab with no connection to production systems, third-party networks, or the public internet beyond PfSense's WAN for updates. No external target, real organization, or third party is ever in scope. This repository documents personal, self-authorized security research for learning and portfolio purposes — it is not a template for testing systems you don't own or lack explicit written authorization to test.

## Scope

**In scope**:

- The Target/Victim VLAN (30) and its intentionally vulnerable hosts (DVWA, Metasploitable)
- Traffic between the Attacker VLAN (40) and the Target VLAN (30), as inspected by pfSense/Suricata

**Out of scope**:

- The Management VLAN (10) and SOC & Monitoring VLAN (20) — these should be unreachable from the Attacker VLAN by design (see segmentation checks in `vlan-healthcheck.sh`); any playbook that succeeds in reaching them is itself a finding, not a technique to continue exploiting
- Anything outside this lab's private address space
- Destructive actions with no rollback (e.g. actions that would require rebuilding a VM from scratch) unless explicitly noted as the exercise's purpose and reversible via snapshot

## Handling of data

No real personal data, credentials, or third-party information is used anywhere in this lab. Any "sensitive" data referenced in playbooks or reports (credentials, hashes, PII-like fields) is synthetic, generated for the exercise, and never reused elsewhere.

## Why this document exists

Beyond the practical scope definition, this file exists to make explicit something that should be second nature for anyone doing offensive security work: **authorization and scope come first, technique comes second.** A red team exercise without a defined ROE isn't a red team exercise — it's unauthorized access, regardless of intent. Documenting this here is as much a portfolio signal as any tool skill.

## Legal & ethical note

The techniques documented under `/redteam` are for authorized testing only. Do not run any of these techniques, tools, or scripts against systems you do not own or have explicit written permission to test. Unauthorized access to computer systems is illegal in most jurisdictions.