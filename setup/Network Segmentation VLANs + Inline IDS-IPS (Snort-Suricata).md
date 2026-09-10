
This guide walks through segmenting the lab into VLANs and forcing all inter-VLAN traffic through Snort/Suricata for **inline** inspection — meaning packets are actually held up and analyzed (and can be dropped) before reaching their destination, not just copied to a monitor port after the fact.

## Goal

Traffic path for every packet crossing a VLAN boundary:

```
Source VM → pfSense VLAN interface → Suricata (inline IPS) → Firewall rules → Destination VLAN
```

## Prerequisites

- pfSense already installed and reachable (see `01-getting-started.md`)
- Hypervisor: VMware ESXi (recommended path below) or VMware Workstation (alternative note in Step 2b)
- pfSense VM with at least one extra virtual NIC beyond WAN, OR a single LAN NIC if you plan to trunk (see Step 2)


___
## Step 1 — Plan VLANs and addressing

Reuse the segmentation already defined in `docs/reference/architecture.md`:

|VLAN ID|Name|Subnet (example)|Purpose|
|---|---|---|---|
|10|Management|10.10.10.0/24|Hypervisor + jump box|
|20|SOC & Monitoring|10.10.20.0/24|Wazuh, OpenVAS, Shuffle, TheHive, MISP|
|30|Target / Victim|10.10.30.0/24|Vulnerable VMs, web app + ModSecurity|
|40|Attacker|10.10.40.0/24|Kali|

Write these down before touching pfSense — renumbering later means redoing firewall rules and DHCP scopes.

## Step 2 — Create the VLAN trunk (ESXi)

1. In **vSwitch** settings, create one portgroup per VLAN (e.g. `VLAN10-MGMT`, `VLAN20-SOC`, `VLAN30-TARGET`, `VLAN40-ATTACKER`), each with its **VLAN ID** set to the matching number (10, 20, 30, 40).
2. Attach each lab VM's vNIC to the portgroup matching its VLAN — a Wazuh VM's NIC goes on `VLAN20-SOC`, a DVWA VM's NIC goes on `VLAN30-TARGET`, etc.
3. Create one more portgroup for pfSense's internal NIC, but set its VLAN ID to **4095 ("All")** — this makes it a trunk port that passes all tagged traffic through untouched.
4. Attach pfSense's LAN-side vNIC to this trunk portgroup.

### Step 2b — Alternative: VMware Workstation

Workstation doesn't support 802.1Q VLAN tagging on its virtual switches, so let pfSense do the tagging instead:

1. Create a single custom network (e.g. `VMnet2`) and attach pfSense's LAN vNIC and every VM's vNIC to it — pfSense will emit and expect tagged frames, and Workstation's switch just passes them through as ordinary payload.
2. Everything else in this guide (VLAN interfaces, firewall rules, Suricata) works identically.

## Step 3 — Create VLAN interfaces in pfSense

1. **Interfaces → Assignments → VLANs tab → Add**
2. For each VLAN: set **Parent interface** to the trunk NIC, enter the **VLAN tag** (10, 20, 30, 40), and give it a description (`MGMT`, `SOC`, `TARGET`, `ATTACKER`).
3. Go to **Interfaces → Assignments** and assign each new VLAN as a new interface (`OPT1`, `OPT2`...).
4. Rename each interface to match (`MGMT`, `SOC`, `TARGET`, `ATTACKER`) under **Interfaces → [OPTx]**.

## Step 4 — Configure IPs and DHCP per VLAN

For each VLAN interface:

1. Enable the interface, set **Static IPv4**, and assign the gateway address for that subnet (e.g. `10.10.20.1/24` for SOC).
2. **Services → DHCP Server** → enable on that interface, set a pool range (e.g. `10.10.20.100–10.10.20.200`).
3. Repeat for all four VLANs.

## Step 5 — Baseline firewall rules (deny by default)

Segmentation only matters if VLANs can't freely talk to each other:

1. On each VLAN interface's firewall rules tab, the default pfSense behavior already blocks traffic _into_ an interface from other interfaces unless a rule allows it — leave it that way.
2. Add explicit **allow** rules only where needed, for example:
    - `TARGET → SOC` on port 1514/1515 (Wazuh agent-to-manager) — required for agents to report.
    - `SOC → TARGET` for OpenVAS scans, if desired.
3. Do **not** add a rule allowing `ATTACKER → MGMT` or `ATTACKER → SOC` — this isolation is the point of the Attacker VLAN.

## Step 6 — Install Suricata in pfSense

1. **System → Package Manager → Available Packages** → search `Suricata` → Install.
2. Once installed, go to **Services → Suricata → Interfaces → Add**.

## Step 7 — Enable inline IPS mode (this is what makes it "before reaching destination")

By default, Suricata packages in pfSense run in **Legacy/IDS mode** — promiscuous, alert-only, traffic is copied and inspected _after_ it's already been forwarded. To make it truly inline:

1. On the interface you just added (repeat once per VLAN interface — MGMT, SOC, TARGET, ATTACKER), open its settings.
2. Under **General Settings**:
    - **Suricata mode**: select **Legacy Blocking Mode is off** → toggle to **Inline (IPS) Mode**, which uses the `netmap` framework to intercept and hold packets rather than just mirror them.
    - Enable **Block Offenders**.
3. Under **WAN/interface settings**, make sure **Home Net** includes the subnet for that VLAN.
4. Save and start Suricata on that interface. Repeat for each of the four VLAN interfaces so all inter-VLAN traffic passes through inline inspection regardless of direction.

> If you'd rather use **Snort** instead of Suricata, the same package menu and Inline IPS Mode toggle exist under **Services → Snort** — the steps are functionally identical.

## Step 8 — Load detection rules

1. **Services → Suricata → Global Settings** → enable a free ruleset (e.g. **ET Open**) and add your Snort/Suricata subscriber code if you have one.
2. **Update Rules** to download them.
3. On each interface's **Categories** tab, enable the rule categories relevant to that VLAN (e.g. `web-attacks` for TARGET, `scan` for ATTACKER).

## Step 9 — Verify it's actually inline

1. From a VM on `ATTACKER`, run a basic scan against a VM on `TARGET` (e.g. `nmap -sS`).
2. Check **Services → Suricata → Alerts** on the relevant interface — you should see alerts.
3. Confirm blocking works: check **Services → Suricata → Blocks** for the offending IP, and verify further traffic from that IP is dropped (ping should stop responding) until you clear the block.
4. Cross-check the same event lands as an alert in **Wazuh** (VLAN 20), confirming the log pipeline end-to-end.

## Troubleshooting

- **No traffic reaching Suricata at all**: the trunk portgroup's VLAN ID isn't set to 4095, or the VM's vNIC is on the wrong portgroup.
- **High false positives / dropped legitimate traffic**: start with alert-only categories enabled, confirm expected traffic isn't flagged, then enable blocking category by category.
- **Suricata interface won't start**: another interface already claimed the same physical adapter in inline mode — each inline instance needs its own VLAN interface, not the raw parent NIC.

## Where this lives in the repo

- This file: `/setup/02-network-segmentation-vlans.md`
- Referenced architecture/diagram: `/docs/reference/architecture.md` and `/diagrams/network-architecture.svg`
- Once you've run through this on your own lab, log the actual outcome (what broke, what you tweaked) in `/docs/journal/` using the five-field template.