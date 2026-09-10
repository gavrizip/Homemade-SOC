# Shuffle

## What it is

Shuffle is an open source Security Orchestration, Automation and Response (SOAR) platform. It lets you build **workflows** — visual, drag-and-drop sequences of actions — that trigger automatically (e.g. on a webhook) and call other tools' APIs (TheHive, MISP, email, Slack, and hundreds of other pre-built "apps") without a human running each step by hand.

## Role in this project

Shuffle is the automation glue of the SOC VLAN. It sits between **Wazuh** (which detects) and **TheHive/MISP** (which investigate and enrich), turning what would otherwise be a manual, multi-tool triage routine into a workflow that runs the moment an alert fires.

Concretely, in this lab: Wazuh alert → Shuffle webhook → case created in TheHive → MISP queried for known IOCs → result attached back to the case. See the full walkthrough in `incident-response-stack.md`.

## Justification

Shuffle was chosen over commercial SOAR platforms (Splunk SOAR, Palo Alto XSOAR) because those are priced and licensed for enterprise deployments, not a solo home lab. Compared to writing custom Python glue scripts to connect Wazuh, TheHive, and MISP directly, Shuffle was preferred because its visual workflow builder is easier to document and show in a portfolio context (a screenshot of a workflow communicates the automation logic faster than a script would to someone skimming a GitHub repo), and it comes with maintained apps for TheHive/MISP already built, instead of hand-rolling API calls.

## Limitations

- **Self-hosting overhead** — unlike a SaaS SOAR, Shuffle here needs its own upkeep (updates, backups of workflow exports) as one more service in the lab.
- **Debugging workflows is visual, not code-first** — tracing why a workflow step failed means clicking through the UI's execution log rather than reading a stack trace, which is slower for complex logic.
- **App coverage varies** — Shuffle's app ecosystem is large but community-maintained; not every integration is as polished as the built-in TheHive/MISP apps used here.
- **A failed webhook is a silent failure** — if Wazuh's webhook call to Shuffle fails (network blip, Shuffle down), the alert simply never gets automated triage; there's no automatic fallback to manual handling unless one is explicitly built in.

## Integration with the stack

Shuffle is the connector, not an endpoint — its value only exists in relation to the other tools:

1. **Wazuh** calls a Shuffle webhook when an alert crosses a defined severity threshold.
2. The Shuffle workflow extracts the relevant fields (source IP, hash, rule ID) from the alert payload.
3. It calls **TheHive**'s API to create a case pre-filled with those observables.
4. It calls **MISP**'s API to check those observables against known threat intelligence.
5. Any MISP match gets written back onto the TheHive case, so the analyst sees the enrichment immediately when they open it.

## Where configs live

|Item|Location|
|---|---|
|Shuffle workflow exports (JSON)|`/configs/shuffle/`|
|Wazuh → Shuffle webhook config|`/configs/wazuh/`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [Shuffle documentation](https://shuffler.io/docs)