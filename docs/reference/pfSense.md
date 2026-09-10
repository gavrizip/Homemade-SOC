## What it is

pfSense is an open source firewall/router distribution built on FreeBSD. Beyond basic routing and NAT, it supports VLAN tagging, stateful firewall rules per interface, VPN, and a package system that adds features like IDS/IPS (Suricata/Snort), giving it capabilities closer to a small commercial UTM appliance than a plain router.

## Role in this project

pfSense is the **perimeter firewall and router** for the entire lab — it sits between the home network/internet and all four internal VLANs (Management, SOC & Monitoring, Target/Victim, Attacker). Every packet crossing a VLAN boundary passes through pfSense, which is also where Suricata runs inline (see `architecture.md` and `setup/02-network-segmentation-vlans.md`).

In short: pfSense is what turns "a bunch of VMs on one flat network" into an actual segmented lab with enforced boundaries.

## Justification

pfSense was chosen over a Linux box with `iptables`/`nftables` for a few concrete reasons: it bundles VLAN interface management, per-interface firewall rules, and a package ecosystem (Suricata/Snort included) behind a single web UI, which cuts setup time considerably for a solo home lab compared to hand-rolling the same with raw netfilter rules and a separate IDS install. The trade-off is giving up some of the fine-grained control a hand-built Linux firewall would offer, in exchange for a setup that's faster to build, easier to document, and easier for someone else to replicate from this repo.

OPNsense (a pfSense fork) was the closest alternative; pfSense was picked mainly for its larger documentation footprint and package ecosystem maturity, which matters when troubleshooting a home lab solo.

## Limitations

- **Single point of failure** — every VLAN's connectivity and inline IDS/IPS inspection depends on one pfSense instance; there's no redundancy in this lab (acceptable for a home lab, not for production).
- **Suricata/Snort packages lag upstream** — the pfSense package versions of Suricata/Snort trail the standalone releases, so the newest detection features arrive later than if run bare-metal.
- **Web UI-centric** — configuration is mostly done through the GUI; while there is a config-export/XML backup, it's less naturally version-controllable than plain text config files, which is why `/configs/pfsense/` stores exported XML snapshots rather than live-editable configs.
- **Performance ceiling** — running firewall, routing, and inline IPS inspection on one VM sharing host resources with everything else means throughput and latency don't reflect what dedicated hardware would give.

## Integration with the stack

pfSense doesn't feed alerts into Wazuh/TheHive directly by default — its involvement is at the network layer:

1. pfSense enforces the firewall rules that define which VLANs can reach which (see the allow/deny table in the VLAN setup guide).
2. Suricata, running as a pfSense package in inline IPS mode, inspects traffic crossing VLAN boundaries and can drop malicious packets before they reach their destination.
3. Suricata's alerts are what actually reach **Wazuh** — a Wazuh agent (or the Suricata-to-Wazuh log forwarding) reads Suricata's eve.json log and turns network-layer detections into Wazuh alerts, which then flow into the same incident response chain as any other alert.
4. The daily `vlan-healthcheck.sh` script actively verifies pfSense's segmentation is still enforced as configured.

## Where configs live

|Item|Location|
|---|---|
|Exported pfSense config (XML backup)|`/configs/pfsense/`|
|VLAN + firewall rule setup steps|`/setup/02-network-segmentation-vlans.md`|
|Daily segmentation check|`/scripts/automation/vlan-healthcheck.sh`|

## Further reading

- [pfSense documentation](https://docs.netgate.com/pfsense/en/latest/)