## What it is

OpenVAS (Open Vulnerability Assessment Scanner, now maintained as part of the **Greenbone Vulnerability Management** suite) is an open source vulnerability scanner. It runs authenticated or unauthenticated scans against hosts and reports known vulnerabilities (CVEs), missing patches, misconfigurations, and weak service exposure, ranked by severity (CVSS).

## Role in this project

OpenVAS runs from **VLAN 20 (SOC & Monitoring)** and scans the **Target/Victim VLAN (30)** on a schedule. Its job is proactive, not reactive: instead of waiting for an attack to trigger a Wazuh/Suricata alert, OpenVAS tells you _in advance_ what's exploitable on those hosts — which is exactly why the Target VMs (DVWA, Metasploitable) are useful in the first place: they give OpenVAS something realistic to find.

This is the lab's only vulnerability management piece — everything else in the stack (Wazuh, Suricata, ModSecurity) detects activity; OpenVAS assesses exposure before activity happens.

## Justification

OpenVAS was chosen over commercial scanners (Nessus, Qualys, Tenable.io) because it's fully open source with no licensed-host limits, which matters for a home lab scanning multiple VMs freely and repeatedly. 

Nessus Essentials would also work and is free for small scopes, but ties the choice to a vendor account and a host cap — OpenVAS avoids both, at the cost of a somewhat rougher UI and heavier resource footprint to run yourself.

## Limitations

- **Signature/CVE-based** — like ClamAV, it identifies _known_ vulnerabilities against its own vulnerability test (NVT) feed. Zero-days and logic flaws specific to custom code won't be found.
- **Noisy without tuning** — default scan configs can generate a large volume of low-severity findings; without filtering by severity/relevance, reports become hard to act on.
- **Scanning has side effects** — some vulnerability tests can crash fragile or intentionally-vulnerable services (which DVWA/Metasploitable are, by design), so scan scheduling needs to account for that.
- **No exploitation, no remediation** — OpenVAS reports what's wrong; it doesn't verify exploitability beyond version/banner checks, and it doesn't fix anything itself.

## Integration with the stack

OpenVAS's output isn't wired into the automated Wazuh→Shuffle→TheHive alert chain the way ClamAV's is — vulnerability scan results are a different kind of signal (exposure, not an event) and are reviewed on their own cadence:

1. OpenVAS runs a scheduled scan against the Target VLAN.
2. Scan reports are exported and reviewed manually (or optionally imported into TheHive as a case for tracking remediation as a task list).
3. Findings that reveal _why_ a later Wazuh/Suricata alert succeeded (e.g. "this exploited service was already flagged as vulnerable last week") are cross-referenced manually — this is a natural fit for a journal entry documenting the connection.

## Where configs live

|Item|Location|
|---|---|
|Scan schedules / task configs|`/configs/openvas/`|
|Exported scan reports (samples)|`/data/`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [Greenbone Community documentation](https://greenbone.github.io/docs/)