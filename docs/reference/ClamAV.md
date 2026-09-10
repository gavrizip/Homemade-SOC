
## What it is

ClamAV is an open source antivirus engine built around signature-based detection. It scans files, directories, or live processes against a database of known malware signatures and flags matches. It ships with a command-line scanner (`clamscan`), an optional daemon (`clamd`) for faster repeated scans, and `freshclam` for keeping signatures up to date.

## Role in this project

ClamAV runs on the **Target/Victim VLAN (30)** endpoints, alongside the Wazuh agent. Its job is narrow and specific: catch known-signature malware dropped onto a host (e.g. a downloaded EICAR test file, a known webshell, a known malicious script) that a human or an exploit places on disk.

It is not the lab's primary detection layer — that's Wazuh (behavioral/log-based) and Suricata (network-based). ClamAV covers the specific gap of **file-based, signature-known threats on disk**, which the other two tools don't directly address.

## Justification

ClamAV was chosen over commercial AV/EDR agents (CrowdStrike, SentinelOne, Defender for Endpoint) for one simple reason: it's free, open source, and scriptable, which matters for a home lab meant to be fully reproducible by anyone cloning the repo. A commercial EDR would add licensing friction and, in most cases, cloud dependencies that don't fit an isolated lab.

The trade-off is explicit: ClamAV gives up the behavioral/heuristic detection and centralized management a commercial EDR provides, in exchange for being free and fully under local control.

## Limitations

- **Signature-based only** — no heuristics, no behavioral analysis, no memory-scanning. It will miss any malware without a matching signature, including most zero-days and custom/obfuscated payloads.
- **Not real-time by default** — `clamscan` alone is on-demand; without `clamd` running as a resident daemon, nothing is caught the moment a file lands, only when a scan is explicitly run.
- **No EDR capabilities** — no process tree visibility, no network correlation, no rollback/remediation actions. It only answers "does this file match a known-bad signature," nothing more.
- **Signature freshness depends on `freshclam`** — if the update cron job silently fails, ClamAV keeps running but with a stale (and increasingly useless) database.

## Integration with the stack

ClamAV doesn't talk to the rest of the stack on its own — the integration is done through Wazuh:

1. ClamAV runs scheduled scans (via cron) on the Target VLAN hosts.
2. Scan output/logs are read by the **Wazuh agent** on that host.
3. A custom Wazuh decoder/rule parses ClamAV's log format and raises an alert when a detection occurs.
4. From there, the alert follows the same path as any other Wazuh alert — into the incident response chain (Shuffle → TheHive → MISP).

Without this Wazuh integration, a ClamAV detection would just sit in a local log file, unseen.

## Where configs live

|Item|Location|
|---|---|
|ClamAV scan schedule (cron)|`/configs/clamav/`|
|Wazuh decoder/rule for ClamAV logs|`/detections/wazuh/clamav-rules.xml`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [ClamAV documentation](https://docs.clamav.net/)