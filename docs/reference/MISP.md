## What it is

MISP (Malware Information Sharing Platform) is an open source threat intelligence platform for storing, correlating, and sharing Indicators of Compromise (IOCs) — IPs, file hashes, domains, URLs — along with the context around them (which campaign, which malware family, which report they came from). It can also subscribe to external **feeds** to pull in community threat intelligence automatically.

## Role in this project

MISP sits in **VLAN 20 (SOC & Monitoring)** and acts as the lab's threat intelligence lookup layer. When an alert comes in, MISP is what answers the question _"has this indicator been seen before, and by whom?"_ — turning a bare IP or hash into context an analyst can actually act on.

## Justification

MISP was added specifically to close a gap this project had: without it, every alert was judged purely on Wazuh's rule severity, with no way to check whether an indicator was already known-malicious elsewhere. Rather than build ad-hoc IOC lookups, MISP was chosen because it's the de facto open source standard for threat intel sharing — well documented, widely integrated (Shuffle and TheHive both have native MISP apps/connectors), and backed by an active community feed ecosystem, so the lab benefits from external intelligence without needing a paid feed subscription.

## Limitations

- **MISP doesn't detect anything itself** — it's a lookup/correlation layer, not a sensor. Its value depends entirely on the quality and freshness of the data fed into it (feeds, manual entries, other tools' output).
- **Feed quality varies** — free community feeds range from excellent to noisy; without curation, MISP can return low-confidence matches that waste analyst time.
- **No automatic blocking** — a MISP match doesn't do anything on its own; it has to be wired into Shuffle/pfSense to trigger an action, which means the enrichment is only as useful as the automation built around it.
- **Manual upkeep** — feeds need to be reviewed periodically, and internally-discovered IOCs need to be manually added as events for MISP to "learn" from this lab's own incidents.

## Integration with the stack

MISP is queried, not standalone:

1. **Shuffle** calls MISP's API during an automated workflow, passing along an indicator (IP, hash) extracted from a Wazuh alert.
2. MISP returns a match (or nothing) with tags/context (e.g. "known scanning botnet").
3. Shuffle attaches that result to the corresponding **TheHive** case.
4. Analysts can also query MISP directly from within TheHive when investigating an observable manually.

Full walkthrough of this chain: `incident-response-stack.md`.

## Where configs live

|Item|Location|
|---|---|
|MISP feed configuration|`/configs/misp/`|
|Shuffle → MISP workflow app config|`/configs/shuffle/`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [MISP documentation](https://www.misp-project.org/documentation/)
- [MISP feeds list](https://www.misp-project.org/feeds/)