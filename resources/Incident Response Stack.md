# **Incident Response Stack: TheHive, Shuffle & MISP**

These three tools form the response side of the SOC — everything that happens _after_ Wazuh raises an alert. They live in **VLAN 20 (SOC & Monitoring)** and work as a chain rather than independently:

```
Wazuh alert → Shuffle (automates) → TheHive (case) → MISP (enrich / share IOCs)
```

See `architecture.md` for how this fits into the wider network.


---
## Shuffle — SOAR Automation

**What it is**: an open source Security Orchestration, Automation and Response (SOAR) platform. It watches for triggers (like a webhook from Wazuh) and runs a **workflow** — a visual sequence of actions — in response, without a human having to do each step by hand.

**Key concepts**:

- **Workflow**: the automated sequence of steps, built as a drag-and-drop flow.
- **Trigger**: what starts a workflow — typically a webhook receiving a Wazuh alert.
- **App**: a pre-built integration (TheHive, MISP, email, Slack, etc.) that a workflow step can call.

**Role in this lab**: Shuffle is the glue. A Wazuh alert above a severity threshold hits a Shuffle webhook, which then:

1. Formats the alert data
2. Creates a case in TheHive automatically
3. Queries MISP for any matching IOCs (IPs, hashes, domains) in the alert
4. Attaches whatever MISP finds back onto the TheHive case

Without Shuffle, this would mean manually copying alert details into TheHive and manually checking MISP every time — Shuffle removes that toil.


---
## TheHive — Case Management

**What it is**: an open source Security Incident Response Platform (SIRP) for tracking investigations as structured **cases**, rather than as scattered notes or tickets.

**Key concepts**:

- **Alert**: a raw signal coming in (from Wazuh via Shuffle, in this lab) — not yet a confirmed incident.
- **Case**: a promoted alert that's now being actively investigated.
- **Observable**: an indicator attached to a case (IP, hash, domain, URL) — these can be checked against MISP directly from TheHive.
- **Task**: a checklist item within a case (e.g. "check DNS logs," "isolate host").

**Role in this lab**: TheHive is where triage actually happens. Every alert Shuffle forwards becomes a case with pre-filled observables, so the analyst (you) opens TheHive already knowing what to look at, instead of starting from a blank page.


---
## MISP — Threat Intelligence Platform

**What it is**: a platform for storing, correlating, and sharing threat intelligence — known-bad IPs, file hashes, domains, and the context around them (which campaign, which malware family, etc.).

**Key concepts**:

- **Event**: a bundle of related indicators (e.g. everything tied to one campaign).
- **Attribute**: a single indicator inside an event (an IP, a hash, a URL).
- **IOC**: Indicator of Compromise — the general term for these attributes.
- **Feed**: an external source of events MISP can subscribe to, to stay current without manual entry.

**Role in this lab**: MISP is the "have we seen this before?" check. It closes the threat intelligence gap this project originally had — instead of judging an alert purely on Wazuh's rule severity, an analyst can check whether the IP or hash involved is already known-malicious elsewhere.

---

## End-to-end example

A concrete walkthrough of one alert traveling through the whole chain:

1. **Wazuh** detects a brute-force pattern on VLAN 30 (Target zone) and fires an alert.
2. **Shuffle**'s webhook receives it, extracts the source IP, and:
    - Creates a case in **TheHive** titled "Possible brute-force — 10.10.30.x"
    - Queries **MISP** for that source IP
3. **MISP** returns a match: the IP is tagged as part of a known scanning botnet.
4. Shuffle attaches that MISP context to the TheHive case automatically.
5. The analyst opens **TheHive**, sees the case already has the botnet tag attached, and can skip straight to deciding on containment instead of manually researching the IP first.


---
## Where things live in the repo

|Item|Location|
|---|---|
|Shuffle workflow exports|`/configs/shuffle/`|
|TheHive case templates|`/configs/thehive/`|
|MISP feed configuration|`/configs/misp/`|
|Installation steps for each tool|`/setup/` (see numbered guides)|
|Day-to-day issues encountered|`/docs/journal/`|

## Further reading

- [Shuffle documentation](https://shuffler.io/docs)
- [TheHive documentation](https://docs.strangebee.com/thehive/)
- [MISP documentation](https://www.misp-project.org/documentation/)