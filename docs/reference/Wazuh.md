
# Wazuh

## What it is

Wazuh is an open source SIEM and XDR platform. A central **manager** collects data from lightweight **agents** installed on endpoints (log collection, file integrity monitoring, rootkit detection, vulnerability detection) and from external sources like network IDS logs, correlates it against a ruleset, and raises alerts ranked by severity. It also ships a web dashboard for searching and visualizing everything it collects.

## Role in this project

Wazuh is the detection core of the SOC — it's what everything else in the stack ultimately feeds into or reads from. It runs in **VLAN 20 (SOC & Monitoring)** as the manager, with agents deployed on the Target/Victim VLAN (30) hosts and on the Management jump box.

Concretely, Wazuh is where three different signal types converge into one place:

- **Host-based events** from its own agents (log data, file changes, ClamAV scan results)
- **Network-based events** forwarded from Suricata's `eve.json` running inline on pfSense
- Alerts crossing a severity threshold trigger the automated response chain: Shuffle → TheHive → MISP (see `incident-response-stack.md`)

## Justification

Wazuh was chosen over Splunk or the Elastic Stack's paid security tiers because it's free, open source, and purpose-built for security use cases out of the box — agent-based collection, a security ruleset, and file integrity monitoring are all included, whereas Elastic/Splunk require assembling equivalent functionality from more generic building blocks (or paying for it). Compared to running raw Elasticsearch + a custom rule engine, Wazuh gives up some flexibility in exchange for a security-specific data model and ruleset that didn't need to be built from scratch — which matters for a solo project with a documentation-first focus.

## Limitations

- **Ruleset-driven detection** — like the other signature/rule-based tools in this stack, Wazuh's out-of-the-box rules catch known patterns; custom detections need to be written and tuned per environment, which is ongoing work rather than a one-time setup.
- **Single manager, no clustering** — this lab runs one Wazuh manager with no high-availability; acceptable for a home lab, not representative of a production multi-node deployment.
- **Resource-hungry at scale** — the manager plus its own Elasticsearch/OpenSearch-based indexer can be heavy on a shared VM host, especially once historical data accumulates.
- **Dashboard learning curve** — while functional out of the box, building genuinely useful custom visualizations/dashboards takes real time to learn, beyond the default views.

## Integration with the stack

Wazuh is the hub the rest of the pipeline is built around:

1. **Agents** on Target VLAN hosts send logs, file integrity events, and ClamAV scan output to the manager.
2. **Suricata**, running inline on pfSense, writes `eve.json`; Wazuh reads it (via agent or log forwarding) and turns network detections into Wazuh alerts.
3. When an alert clears a defined severity threshold, Wazuh calls a **Shuffle** webhook.
4. From there: Shuffle creates a case in **TheHive**, checks **MISP** for known indicators, and attaches the result — all detailed in `incident-response-stack.md`.
5. The daily `vlan-healthcheck.sh` script's findings can also be logged as Wazuh events for a single pane of glass on lab health, not just security alerts.


___
# **Wazuh Core Capabilities**

Wazuh is the core of this home SOC: it operates as a unified SIEM and XDR platform, correlating events from every other tool in the stack. This document details the capabilities that make it central to the project's detection and response workflow.

## **1. Active Response**

### 1.1 Purpose and Operational Role

Active Response is the capability of Wazuh to execute automatic actions when relevant security events are detected. It allows closing the detection loop with fast responses, helping to handle high-priority incidents in real time. For example, Wazuh can apply network blocks or remove malicious files automatically through preconfigured scripts, reducing the workload of the security team and alert noise.

### 1.2 Triggering Mechanism

Wazuh Active Response is configured in the `ossec.conf` file, where commands (scripts) and activation conditions are defined. When a security event triggers a specific rule (by ID, level, or rule group), Wazuh executes one or more scripts associated with that alert.

These scripts can run in different scopes:
- **Local**: executed on the agent that generated the alert.
- **Server**: executed on the Wazuh server (manager).
- **Defined agent**: executed on specific agents even if the alert originated elsewhere.
- **All**: executed on all agents (dangerous usage, may affect the entire environment).

The module logs these actions in a dedicated "active-responses" log file for auditing purposes. In the interface, an additional alert is generated when a script is triggered, confirming the action taken (e.g., "Host Blocked by firewall-drop" when an IP is blocked).

### 1.3 Types of Active Response

Wazuh supports two operational modes. The choice between both depends on the required response (whether reversal is expected) and the operational impact on the system.

- **Stateless responses**: one-time actions that are not automatically reversed. They are used for permanent or irreversible mitigations, such as deleting malicious files or disabling accounts.
- **Stateful responses**: temporary actions that are automatically reverted after a configured time period. Typically used for network-level containment, such as temporary IP blocking.

### 1.4 Default Response Capabilities

Wazuh includes several preconfigured scripts ready to use on Linux/macOS/Unix and Windows agents. The most relevant include:

- Blocking malicious IP addresses at the firewall level
- Denying access via host-based restrictions
- Disabling compromised user accounts
- Restarting Wazuh services or agents
- Sending notifications to external systems

These built-in actions provide immediate usability without requiring custom development:

**Linux/macOS/Unix**
- `disable-account`: disables a local user account
- `firewall-drop` / `firewalld-drop` / `ipfw` / `npf` / `pf`: add the source IP to firewall blocklists (iptables, firewalld, IPFW, NPF, PF, depending on the system)
- `host-deny`: adds the IP to `/etc/hosts.deny` to block it
- `ip-customblock`: a customizable IP blocking template
- `wazuh-slack`: sends notifications to Slack (requires webhook)
- `restart-wazuh` (or `restart.sh`): restarts the Wazuh agent or manager

**Windows**
- `netsh.exe`: blocks an IP using the Windows `netsh` command
- `restart-wazuh.exe`: restarts the Wazuh agent on Windows
- `route-null.exe`: adds the attacker IP to a null route (drops traffic)

These scripts cover common containment tasks (blocking attackers, isolating hosts or users, restarting components, etc.) and reduce the need to develop custom code from scratch.

### 1.5 Security Considerations and Constraints

Although powerful, Active Response introduces operational risk if not properly configured:

- Incorrect rules can trigger unintended system actions
- Broad execution scopes (e.g., all agents) can amplify impact
- Poor event filtering may lead to false-positive responses
- Actions must be carefully validated before deployment in production

Because of this, Active Response requires strict rule tuning and controlled activation thresholds.

### 1.6 Key Value in Incident Response

Active Response provides a direct automation layer between detection and mitigation. When properly configured, it enables:

- Faster containment of threats
- Reduction of manual intervention
- Consistent response to repeated incidents
- Improved efficiency for resource-constrained security teams

### 1.7 Summary

Wazuh Active Response acts as an automated remediation engine integrated into the detection pipeline. Its effectiveness depends heavily on correct rule configuration, controlled execution scope, and careful selection of response actions to balance automation with operational safety.

## **2. Vulnerability Detection**

Wazuh offers users a means to manage vulnerabilities within an IT infrastructure using the Vulnerability Detection module. The module functions using one of the following vulnerability sources:

- The repository in Wazuh's Cyber Threat Intelligence (CTI) Platform
- An offline vulnerabilities repository — a locally hosted copy of the Wazuh threat intelligence repository from the Wazuh CTI platform

The Wazuh agent collects a list of installed applications (software inventory data) from monitored endpoints and sends it to the Wazuh server. The Vulnerability Detection module then correlates this software inventory data with vulnerability information obtained from the vulnerability repository.

### 2.1 Alert Generation

The Vulnerability Detection module generates alerts when new vulnerabilities are discovered or when existing vulnerabilities are resolved due to package updates, removals, or operating system changes. However, alerts are only generated under specific detection scenarios and not for every inventory synchronization or reevaluation — the module focuses on alerting on actual changes in vulnerability status rather than existing conditions discovered during initial synchronizations.

The principal functionalities are:

- Detection of new and resolved vulnerabilities
- Operating system-related alerts
- Package-related alerts
- Cluster environment considerations
- Vulnerability content updates

The Wazuh documentation includes a Compatibility Matrix listing the operating systems officially supported by the Vulnerability Detection module.

### 2.2 Configuration

This module is enabled by default on the manager. It works with the System Inventory module and is able to detect vulnerabilities in packages installed on the desired endpoint. Example configuration to enable vulnerability detection:

```xml
<vulnerability-detection>
   <enabled>yes</enabled>
   <index-status>yes</index-status>
   <feed-update-interval>60m</feed-update-interval>
</vulnerability-detection>
```

Further details on local configuration are available in the `ossec.conf` documentation on the Wazuh website. Detected vulnerabilities are forwarded to the Wazuh indexer for querying, visualization, and deeper analysis using the indexer connector setting, which is enabled by default in the `/var/ossec/etc/ossec.conf` file of the Wazuh manager.

## **3. File Integrity Monitoring (FIM)**

Wazuh's File Integrity Monitoring (FIM) capability continuously monitors files and directories to detect unauthorized or unexpected changes. It establishes a baseline by storing cryptographic checksums and file attributes, then compares subsequent file states against this baseline. Alerts are generated when files are created, modified, deleted, or otherwise altered. The module supports both real-time monitoring and scheduled scans, providing continuous visibility into critical system and application files.

### 3.1 Change Detection and Threat Response

The FIM module enhances security operations by identifying changes to sensitive files and providing detailed information about what changed, when it changed, and who or what performed the action. This visibility helps organizations validate change management processes, detect unauthorized modifications, monitor file permissions, and identify indicators of compromise. When integrated with other Wazuh capabilities, FIM becomes an effective tool for threat detection, investigation, and incident response.

## **4. Malware Detection**

Wazuh uses a broad-spectrum malware detection strategy that combines signature-based detection, behavioral analysis, and threat intelligence. This approach helps identify both known malware and suspicious activities that may indicate malicious software, including threats that attempt to evade traditional detection mechanisms.

### 4.1 Detection Methods and Integrations

FIM plays a key role in malware detection by monitoring file changes on endpoints. While FIM alone cannot determine whether a file is malicious, it can be integrated with VirusTotal, YARA scans, threat intelligence feeds, and CDB lists containing malicious file hashes to identify malware. Additionally, the Rootcheck module detects rootkits, trojans, and anomalous system behavior through both signature-based detection and behavioral monitoring, helping uncover threats that may bypass conventional security controls.

- **File integrity monitoring and threat detection rules**: monitors file changes and applies rules to detect malware-related modifications.
- **Rootkit behavior detection**: detects behaviors and signatures of rootkits and trojans attempting to hide within the system.
- **CDB lists and threat intelligence**: compares hashes or indicators against known threat lists to identify malicious files.
- **VirusTotal integration**: queries the VirusTotal platform to check whether a file has been flagged as malicious by multiple antivirus engines.
- **File integrity monitoring and YARA**: combines file monitoring with YARA rules to identify malware-characteristic patterns.
- **ClamAV logs collection**: collects and analyzes logs generated by ClamAV to centralize malware detection.
- **Windows Defender logs collection**: collects and analyzes Windows Defender events to detect threats from within Wazuh.
- **Custom rules to detect malware IOCs**: allows creating custom rules to detect indicators of compromise (IOCs), such as hashes, domains, IPs, or malicious file names.

## **5. Security Configuration Assessment (SCA)**

The Security Configuration Assessment (SCA) module helps organizations reduce their attack surface by identifying security misconfigurations and configuration weaknesses on monitored endpoints. It evaluates systems against predefined security policies to verify compliance with hardening standards and security best practices. Typical assessments include password policies, unnecessary services, installed software, network configurations, files, directories, registry settings, and running processes.

### 5.1 Policy-Based Security Validation

SCA performs scans using YAML-based policy files that define security checks and remediation recommendations. These policies compare the actual endpoint configuration against expected secure settings and classify the results as Passed, Failed, or Not Applicable. Wazuh includes a comprehensive set of preconfigured policies based primarily on industry-recognized CIS Benchmarks, while also allowing organizations to create or customize policies to meet their specific security requirements.

### 5.2 Compliance Monitoring and Continuous Assessment

The SCA module provides continuous visibility into endpoint security posture and helps maintain compliance with frameworks and standards such as CIS, PCI-DSS, NIST, ISO 27001, and other regulatory requirements. By generating alerts when configuration states change and providing remediation guidance, SCA enables proactive security hardening, compliance validation, and ongoing monitoring of system security configurations.

## **6. System Call Monitoring**

Wazuh leverages the Linux Audit System (`auditd`) to monitor system calls and collect detailed security-relevant events from Linux endpoints. Audit records generated by the Linux kernel contain information such as the user, process, timestamp, and action performed. Wazuh automatically collects and analyzes these events, providing centralized visibility into system activity and helping security teams identify suspicious behavior, investigate incidents, and maintain compliance requirements.

### 6.1 Detection of Critical Security Activities

Using predefined and customizable audit rules, Wazuh can monitor a wide range of activities, including file access, command execution, privilege escalation, malware-related behavior, and configuration changes. Audit rules can be configured to track:

- File system activity (file and directory access or modifications)
- System calls executed by specific users or processes
- Audit system controls and policies governing audit behavior

By correlating audit events with built-in detection rules, Wazuh provides real-time alerts for potentially malicious or unauthorized actions on Linux systems.

### 6.2 Flexible Auditing and Compliance Support

Wazuh allows organizations to tailor auditing policies to their security and compliance requirements through custom Linux Audit rules. Events are tagged using unique keys, making investigation and correlation easier. By centralizing audit logs and automating analysis, Wazuh simplifies security monitoring, improves threat detection capabilities, and supports compliance with regulatory and auditing standards while reducing the complexity of managing large volumes of audit data.

### 6.3 Use Cases

The following use cases demonstrate how Wazuh leverages the Linux Audit system to provide visibility into critical system activities on Linux endpoints. These examples are based on an Ubuntu 22.04 endpoint with a Wazuh agent installed and enrolled to a Wazuh server.

**Monitoring file and directory access** Wazuh can monitor access to sensitive files and directories by using Linux Audit file system rules. This capability provides detailed information about who accessed a file, when the access occurred, and what action was performed. Monitoring critical resources such as system configuration files, authentication databases, application directories, and confidential data repositories helps detect unauthorized modifications, data tampering attempts, and potential indicators of compromise.

**Monitoring commands run as root** Privileged commands executed by the root user represent a significant security risk if misused or compromised. Wazuh can track command execution events through `auditd` rules and generate alerts whenever critical administrative commands are executed with elevated privileges. This visibility allows organizations to audit administrative activities, identify unauthorized changes to system configurations, and maintain accountability for privileged operations — particularly valuable for forensic investigations and compliance requirements.

**Privilege abuse** Privilege abuse occurs when users or processes attempt to gain unauthorized access to elevated permissions or misuse existing privileges. Wazuh helps detect these activities by monitoring system calls related to privilege escalation mechanisms, user account modifications, permission changes, and the execution of privileged binaries. By correlating audit events with built-in detection rules, Wazuh can identify suspicious behaviors such as unauthorized use of `sudo`, execution of privilege escalation tools, or attempts to modify security-sensitive files.

Together, these use cases illustrate how Wazuh transforms raw Linux Audit events into actionable security intelligence, providing enhanced visibility, threat detection capabilities, and support for security auditing and compliance initiatives.

## **Conclusion**

Wazuh combines SIEM and XDR capabilities within a single open-source platform, providing comprehensive visibility across endpoints, networks, cloud environments, and external services while integrating detection, analysis, and automated response. Its modular architecture, broad compatibility, and extensive customization options allow this project to centralize security operations, reduce infrastructure complexity, and strengthen the overall defensive posture of the home SOC. The following capabilities summarize the main strengths that make Wazuh the right foundation for this project:

1. **Unified SIEM + XDR platform**: Wazuh consolidates functionalities that typically require disparate toolsets into a single agent and architecture, gathering telemetry from endpoints, network infrastructure, cloud-based workloads, and external APIs to deliver centralized security monitoring. This consolidation minimizes tool sprawl and eliminates licensing costs thanks to its open-source nature.

2. **Continuous vulnerability management**: moving beyond periodic scanning, Wazuh provides persistent visibility and threat-based prioritization by correlating system activity with threat intelligence. The agent uses the Syscollector module for software inventorying, which the server then cross-references against the Wazuh CTI database to identify active CVEs, giving a real-time view of exposure rather than a static list of flaws.

3. **Cross-platform vulnerability detection**: by correlating agent-collected software inventories with vulnerability content, the module generates comprehensive dashboard alerts, ensuring broad coverage across Windows, Linux (RHEL, Ubuntu, Debian, CentOS, Amazon Linux), macOS, and containerized workloads.

4. **Network anomaly and behavioral detection**: Wazuh extends its capabilities beyond signature matching to identify and mitigate threats based on atypical behavioral patterns, such as network anomalies or baseline deviations, adding defensive value against emerging threats not yet categorized by static rules or CVEs.

5. **Active Response (automated remediation)**: Wazuh delivers granular, automated actions directly on the endpoint, such as blocking malicious IPs, terminating suspicious processes, and isolating hosts. These responses significantly reduce mean time to respond by mitigating incidents before human intervention is required.

6. **Correlation and analyst context (SIEM core)**: the platform provides real-time correlation, mapping detected events to known adversary tactics and techniques. By ingesting external threat intelligence and supporting custom threat hunting queries, it facilitates automatic mapping to the MITRE ATT&CK framework, enabling effective alert prioritization.

7. **Integration and broad compatibility**: detection capabilities are extended through the integration of third-party solutions and the unification of telemetry from syslog or APIs across cloud providers and SaaS applications, making Wazuh a viable event source for integration into existing architectures.

8. **Open-source foundation**: the platform is highly customizable and backed by an extensive community of developers and users, ensuring total control over detection logic and source code without proprietary licensing constraints — though it requires internal expertise for ongoing maintenance and management.


___
## Where configs live

|Item|Location|
|---|---|
|Wazuh manager config, custom decoders/rules|`/configs/wazuh/`|
|Custom rules for ClamAV / Suricata log parsing|`/detections/wazuh/`|
|Installation steps|`/setup/` (see numbered guides)|

## Further reading

- [Wazuh documentation](https://documentation.wazuh.com/)







