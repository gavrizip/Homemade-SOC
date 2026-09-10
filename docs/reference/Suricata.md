# Suricata

## What it is

Suricata is an open source network IDS/IPS (Intrusion Detection/Prevention System) engine. It inspects network traffic against a ruleset (signatures + protocol anomaly detection) and can run in two modes: **IDS** (passive — alert only, traffic is just copied for inspection) or **IPS** (inline — traffic is actually held up and can be dropped before reaching its destination).

## Role in this project

Suricata runs as a package **inside pfSense**, in **inline IPS mode**, on every VLAN interface (see `setup/02-network-segmentation-vlans.md`). It's the network-layer detection sensor of the lab: while Wazuh watches hosts and logs, Suricata watches the wire — every packet crossing a VLAN boundary passes through it before reaching its destination.

This is what makes the segmentation in `architecture.md` more than just routing: traffic isn't just separated by VLAN, it's actively inspected as it moves between them.

## Justification

Suricata was chosen over Snort — its closest and most direct alternative — mainly for **multi-threading**: Suricata was built from the ground up to use multiple CPU cores, while Snort (in its 2.x branch) is largely single-threaded per instance. On a home lab VM with limited vCPUs shared across four inline instances (one per VLAN), that throughput difference matters more than it would on dedicated hardware. Snort 3 closes much of this gap, but Suricata's multi-threading has been production-proven for longer, and both are drop-in equivalents inside the same pfSense package menu — so switching later, if ever needed, is a low-cost decision.

## Limitations

- **Signature and anomaly-based** — like ClamAV and OpenVAS, Suricata's detection depends on its ruleset (ET Open, in this lab). Novel attack traffic without a matching rule won't be flagged.
- **Encrypted traffic is mostly opaque** — without TLS interception (not configured in this lab, and invasive to set up), Suricata can inspect metadata (JA3 fingerprints, certificate info) but not the encrypted payload itself.
- **Inline mode adds latency and a failure mode** — every packet is held for inspection, and if Suricata crashes or misbehaves, it can become a bottleneck or fail-closed point for that VLAN's traffic (mitigated by the daily healthcheck script, but worth stating plainly).
- **Rule tuning is ongoing work** — the ET Open free ruleset is broad but not tuned to this specific lab; false positives are expected until categories are adjusted per VLAN (see Step 8 of the VLAN setup guide).

## Integration with the stack

Suricata is the sensor; everything downstream depends on its output reaching the rest of the pipeline:

1. Suricata inspects traffic inline on each VLAN interface inside pfSense.
2. Detections are written to `eve.json` (Suricata's structured alert log).
3. A **Wazuh agent** (or direct log forwarding from pfSense) reads `eve.json` and converts network-layer detections into Wazuh alerts.
4. From there, the alert follows the same path as any host-based Wazuh alert — into the incident response chain.
5. The `vlan-healthcheck.sh` script checks daily that a Suricata instance is actually running per VLAN interface — an inline IPS that silently stopped is worse than no IPS at all, since rules may assume it's filtering.

## Where configs live

|Item|Location|
|---|---|
|Suricata interface + rule category config|`/configs/suricata/`|
|Inline IPS setup steps|`/setup/02-network-segmentation-vlans.md`|
|Wazuh decoder for `eve.json`|`/detections/wazuh/suricata-rules.xml`|

## Further reading

- [Suricata documentation](https://docs.suricata.io/)
- [Emerging Threats (ET Open) ruleset](https://rules.emergingthreats.net/)