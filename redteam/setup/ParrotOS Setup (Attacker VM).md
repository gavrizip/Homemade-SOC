___
Tags: #redteam #attack #pentest #SOC 
___

Setup for the offensive tooling VM used in **VLAN 40 (Attacker Zone)**. See `docs/reference/architecture.md` for how this VLAN is isolated from the rest of the lab.

## Why ParrotOS

ParrotOS was chosen over Kali Linux mainly for its lighter default footprint (useful on a home lab host already running several other VMs) and its built-in **AnonSurf** and hardened defaults, which fit the theme of running the attacker box as a contained, disposable environment rather than a daily-driver pentest machine. Both distros ship essentially the same core toolset (Metasploit, Nmap, Burp Suite, etc.) via Debian packages, so the choice is about footprint and defaults, not tool availability.

## Prerequisites

- VMware (Workstation/ESXi) — see `setup/01-getting-started.md`
- ParrotOS Security Edition ISO (includes the full pentesting toolset out of the box)

## Steps

1. **Create the VM** Allocate at least 4 GB RAM / 2 vCPU / 40 GB disk. Attach the VM's vNIC to the `VLAN40-ATTACKER` portgroup (see `setup/02-network-segmentation-vlans.md`) — never to Management or SOC directly.

2. **Install ParrotOS Security Edition** Standard installation; the Security Edition ISO already includes the pentesting toolset, so no separate `apt install parrot-tools` step is needed.

3. **Confirm network isolation** From the ParrotOS VM, attempt to reach the Management VLAN gateway (`10.10.10.1`) and the SOC VLAN gateway (`10.10.20.1` on a management port like SSH). Both should fail — this confirms pfSense's segmentation is enforced before running any exercises. This is the same check the `vlan-healthcheck.sh` script runs automatically every day.

4. **Update tools**

```bash
sudo apt update && sudo apt full-upgrade -y
sudo msfupdate    # keep Metasploit modules current
```

1. **Verify core tools are present**

```bash
 nmap --version
 msfconsole --version
 sqlmap --version
```


## Core toolset used in this lab's playbooks

|Tool|Purpose|
|---|---|
|Nmap|Host/service discovery and reconnaissance|
|Metasploit Framework|Exploitation|
|Burp Suite (Community)|Web app testing (against ModSecurity + DVWA)|
|sqlmap|SQL injection testing|
|Hydra|Credential brute-forcing|

## Where this fits in the repo

- This file: `/redteam/setup/01-parrotos-setup.md`
- Attack scenarios run from this VM: `/redteam/playbooks/`
- Results and blue-team detection outcomes: `/redteam/reports/`