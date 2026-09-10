
![[network-architecture.png]]


## Overview

The lab runs on **VMware (Workstation/ESXi)** and is segmented into four VLANs behind a **pfSense** perimeter firewall. All inter-VLAN traffic is mirrored through **Snort/Suricata** for inline IDS/IPS inspection before reaching its destination.

## VLANs

|VLAN|Purpose|Key hosts|
|---|---|---|
|**10 · Management**|Hypervisor and out-of-band access|VMware host, SSH/RDP jump box|
|**20 · SOC & Monitoring**|Detection, vulnerability scanning, incident response tooling|Wazuh Manager, OpenVAS, Shuffle, TheHive, MISP|
|**30 · Target / Victim Zone**|Intentionally vulnerable systems generating telemetry|Web app + ModSecurity (WAF), DVWA/Metasploitable, Wazuh Agent + ClamAV|
|**40 · Attacker Zone**|Isolated offensive testing|ParrotOS|

## Alert → Response Workflow

Endpoint and network events flow into **Wazuh Manager**, which triggers automated playbooks in **Shuffle (SOAR)**. Cases are opened in **TheHive** for investigation, and indicators are enriched or shared through **MISP**.

## Design rationale

- **Perimeter vs. internal segmentation**: pfSense sits at the network edge rather than between internal VLANs, so all inter-VLAN policy is enforced at a single, auditable choke point alongside the IDS/IPS.
- **Attacker isolation**: VLAN 40 has no direct path to Management, preventing lab compromise from spilling into host infrastructure.
- **Full context**: see `network-architecture.svg` for the visual, and the project overview for tool-by-tool detail.