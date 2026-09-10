# TheHive

## What it is

TheHive is an open source Security Incident Response Platform (SIRP). It organizes security work into structured **cases** — with observables, tasks, and timelines — instead of scattered notes, tickets, or spreadsheets. It's built to be used alongside a SOAR (Shuffle, here) and a threat intel platform (MISP, here), rather than as a standalone tool.

## Role in this project

TheHive is where triage and investigation actually happen. Every alert that clears Wazuh's severity threshold gets automatically turned into a TheHive case by Shuffle, pre-filled with the relevant observables (source IP, hash, affected host) and any MISP context already attached. The analyst's job starts from an organized case, not a raw log line.

See the full walkthrough in `incident-response-stack.md`.

## Justification

TheHive was chosen over a generic ticketing system (Jira, a plain spreadsheet, or a shared doc) because it's purpose-built for security investigations: observables are first-class objects that can be checked against MISP with one click, cases have a defined severity/TLP taxonomy analysts already recognize, and it has a native, well-maintained Shuffle app for automated case creation. A generic ticketing tool would require bolting all of that on manually. TheHive was preferred over the commercial alternative (Splunk SOAR's case management, or a paid SIRP) simply because it's free and self-hostable, matching the rest of the stack.

## Limitations

- **Not a SIEM** — TheHive doesn't collect or correlate raw logs itself; it depends entirely on Wazuh (via Shuffle) to feed it alerts in the first place. Without that pipeline, TheHive is an empty case tracker.
- **Self-hosted maintenance** — as with Shuffle and MISP, running it yourself means handling its own updates and backups (case data included) rather than relying on a managed service.
- **Taxonomy requires discipline** — severity levels, TLP markings, and tags are only as useful as the consistency with which they're applied; a solo analyst can drift into using them loosely over time.
- **No built-in automation** — TheHive tracks and organizes; it doesn't execute response actions (blocking an IP, isolating a host) itself — that responsibility sits with Shuffle.

## Integration with the stack

TheHive is the case-management hub the rest of the pipeline writes into and reads from:

1. **Shuffle** creates a case automatically via TheHive's API when a Wazuh alert clears the severity threshold, pre-filling observables extracted from the alert.
2. Any **MISP** matches Shuffle found are attached to the case as context.
3. The analyst works the case directly in TheHive — adding tasks, notes, and closing it once resolved.
4. Closed cases (and any newly-confirmed IOCs found during investigation) can be manually pushed back into MISP as new events, so the lab's own incidents feed future lookups.

## Where configs live

|Item|Location|
|---|---|
|TheHive case templates|`/configs/thehive/`|
|Shuffle → TheHive workflow app config|`/configs/shuffle/`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [TheHive documentation](https://docs.strangebee.com/thehive/)