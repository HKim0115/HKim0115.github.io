---
title: "Cybersecurity Homelab Roadmap"
categories: [HomeLab]
---

# Building My Cybersecurity Homelab Journey

The main goal of this lab is to better understand how systems, networks, and security monitoring work from both attacker and defender perspectives. I also want to build stronger practical experience related to SOC analyst and cybersecurity analyst roles.

---

## Final Goal

The long-term goal is to build a small lab environment where I can practice:

- Network analysis and traffic monitoring
- Windows and Linux administration
- Log analysis with a SIEM (Splunk)
- Basic incident response workflows
- Security monitoring and detection concepts
- MITRE ATT&CK framework mapping

---

## What is MITRE ATT&CK?

MITRE ATT&CK is a publicly available knowledge base that catalogues the tactics and techniques used by real-world attackers. In a SOC role, analysts use it to describe and classify suspicious activity, not to perform attacks.

In this lab, I use ATT&CK to map what I observe during simulations to a recognised framework. For example:

- Detecting encoded PowerShell execution → `T1059.001 (Command and Scripting Interpreter: PowerShell)`
- Observing repeated failed logins → `T1110 (Brute Force)`
- Capturing SMB enumeration traffic → `T1135 (Network Share Discovery)`

---

## Current Environment

| Tool | Date Added |
|---|---|
| Kali Linux VM & Windows 11 VM | 08 May 2026 |
| Nmap | 10 May 2026 |
| Wireshark | 15 May 2026 |
| Windows Event Viewer | 18 May 2026 |
| Sysmon | 25 May 2026 |

---

## Lab Network Diagram (soon)

> Created with draw.io. Shows the VM layout, network configuration, and monitoring flow from endpoint to SIEM.

> (Insert network diagram image here)

**Network summary:**

- Kali Linux VM — attacker / red team perspective
- Windows 11 VM — target + defender / blue team perspective
- Network type: Host-Only  — isolated from the internet
- Monitoring flow: `Sysmon → Event Viewer → Splunk → Alert`

---

## Homelab Roadmap

---

### Phase 1 — Foundations

---

#### ✅ [Day 1 — VM Networking Fundamentals](https://hkim0115.github.io/homelab/Day-1-VM-Networking-Fundamentals/)

**Topics:**

- Kali Linux VM setup
- Windows 11 VM setup
- NAT vs Host-Only networking
- IP address verification
- Connectivity testing between VMs

---

#### ✅ [Day 2 — Nmap Scanning and SMB Enumeration](https://hkim0115.github.io/homelab/Day-2-Nmap-and-SMB/)

**Topics:**

- Nmap port scanning
- SMB enumeration
- smbclient usage
- Authentication attempts
- Windows firewall behaviour

---

#### ✅ [Day 3 — Wireshark Traffic Analysis and Nmap Reconnaissance](https://hkim0115.github.io/homelab/Day-3-Wireshark/)

**Topics:**

- ICMP traffic capture and analysis
- SMB traffic observation
- Packet capture basics
- Filtered vs open ports
- Windows Public vs Private firewall profile effects

---

> **Milestone 1 — Basic Network Visibility**
> Completed: VM environment running, network traffic visible and analysable.

---

### Phase 2 — Windows Logging & Security Monitoring

---

#### ✅ [Day 4 — Event Viewer & Security Monitoring](https://hkim0115.github.io/homelab/Day-4-Event-Viewer-Monitoring/)

**Topics:**

- Windows Event Viewer navigation
- Security log review
- SMBServer logs
- Failed login monitoring — Event ID 4625
- Successful authentication logs — Event ID 4624
- Event Viewer vs Wireshark — when to use each
- Understanding the role of SIEM vs native Windows logging

---

> **Milestone 2 — Host-Level Visibility**
> Completed: Windows authentication events are visible and interpretable without third-party tools.

---

### Phase 3 — Endpoint Telemetry & Sysmon

---

#### ✅ [Day 5 — Sysmon Installation & Process Monitoring](https://hkim0115.github.io/homelab/Day-5-Sysmon-Installation-&-Process-Monitoring/)

**Topics:**

- Sysmon installation and configuration
- Event Viewer integration with Sysmon logs
- Event ID 1 — Process Creation
- Monitoring cmd.exe, powershell.exe, and ping.exe activity
- Command-line argument visibility

---

#### ✅ [Day 6 — Network Connections & PowerShell Visibility](https://hkim0115.github.io/homelab/Day-6-Network-Connections-Powershell/)

**Topics:**

- SwiftOnSecurity Sysmon config applied
- Sysmon Event ID 3 — Network Connection
- Network connection telemetry
- PowerShell activity monitoring
- Parent-child process relationship analysis
- Why Sysmon configuration matters

**MITRE ATT&CK mapping:**

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| PowerShell execution detected | Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |

---

#### ✅ [Day 7 — File Creation & DNS Monitoring](https://hkim0115.github.io/homelab/Day-7-File-Creation-&-DNS-Monitoring/)

**Topics:**

- Sysmon Event ID 11 — File Creation
- Sysmon Event ID 22 — DNS Query
- File creation logging and suspicious path observation
- DNS query visibility

**MITRE ATT&CK mapping:**

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Suspicious file written to disk | Defense Evasion | Masquerading | T1036 |
| Kali drops executable via SMB | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| Unusual DNS query observed | Command and Control | Application Layer Protocol | T1071 |

---

> **Milestone 3 — Endpoint Monitoring Fundamentals**
> Completed: Process creation, network connections, file writes, and DNS queries are all visible via Sysmon.

---

### Phase 4 — SIEM & Centralized Monitoring

---

#### ✅ [Day 8 — SIEM Introduction & Splunk Architecture](https://hkim0115.github.io/homelab/Day-8-SIEM-Introduction-Splunk-Architecture/)

> **Tool decision: Splunk Free.**

**Topics:**

- What a SIEM does and why centralised logging matters
- Windows → Splunk log forwarding workflow
- Splunk Enterprise (Trial license)
- Indexes `wineventlog` and `sysmon` created; receiving port 9997 opened

---

#### ✅ [Day 9 — Universal Forwarder Deployment: Windows Event Log + Sysmon → Splunk](https://hkim0115.github.io/homelab/Day-9-Universal-Forwarder-Deployment/)

**Topics:**

- Splunk Universal Forwarder installed on Windows 11 VM
- Receiving indexer configured during install: 192.168.56.103:9997
- `inputs.conf` configured — Security/System Event Log → `wineventlog` index
- `inputs.conf` configured — Sysmon channel (`Microsoft-Windows-Sysmon/Operational`) → `sysmon` index
- Forwarder service verified (`SplunkForwarder` running)
- Ingestion verified via search on Ubuntu Splunk
- Troubleshooting notes: firewall/port 9997, `outputs.conf`, `splunkd.log`

---

#### ✅ [Day 10 — SPL Fundamentals & Search Verification](https://hkim0115.github.io/homelab/Day-10-SPL-Fundamentals/)

**Topics:**

- Basic SPL syntax: `index=`, `sourcetype=`, `stats`, `table`, `timechart`
- Searching specific Event IDs (4624, 4625, Sysmon EID 1/3/11/22) across both indexes
- Building a multi-index search to confirm forwarder health end-to-end
- Splunk dashboard screenshot added to portfolio

---

#### ✅ [Day 11 — Basic Detection, Alert Creation & Triage](https://hkim0115.github.io/homelab/Day-11-Basic-Detection-alert-creation-triage/)

**Topics:**

- Failed login detection alert — Event ID 4625
- Suspicious PowerShell detection rule
- Simple alert creation workflow
- **Alert triage decision** — when the alert fires, document whether it's a true or false positive and why
- Investigation workflow basics
- Splunk dashboard screenshot added to portfolio

---

> **Milestone 4 — SOC-Style Monitoring Lab**

---

### Phase 5 — Incident Simulation & Investigation

---

#### Day 12 — Brute-Force Authentication Simulation

**Topics:**

- Repeated failed login simulation from Kali
- Event ID 4625 analysis in Splunk
- Splunk correlation across multiple events
- Detection observations and investigation notes

**MITRE ATT&CK mapping:**

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Repeated failed logins | Credential Access | Brute Force | T1110 |

---

#### Day 13 — Suspicious SMB Activity Investigation

**Topics:**

- Abnormal SMB access simulation
- SMB log analysis in Event Viewer and Splunk
- Wireshark packet capture alongside log review
- Investigation workflow documentation

**MITRE ATT&CK mapping:**

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| SMB enumeration observed | Discovery | Network Share Discovery | T1135 |
| Lateral movement via SMB | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |

---

#### Day 14 — PowerShell Abuse Simulation

**Topics:**

- Encoded PowerShell command execution from Kali
- Sysmon Event ID 1 and 3 telemetry analysis
- Splunk detection of encoded commands
- Suspicious parent-child process analysis

**MITRE ATT&CK mapping:**

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Encoded PowerShell executed | Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |

---

#### Day 15 — IOC Investigation & Log Correlation

**Topics:**

- Indicators of Compromise (IOC) identification
- Cross-source log correlation — endpoint and network
- MITRE ATT&CK mapping table for all simulations
- Investigation notes written in structured format

---

#### Day 15-B — End-to-End Attack Scenario Investigation ⭐

> This is the centrepiece of the portfolio. All individual simulations are connected into a single investigation that mirrors a real SOC workflow.

**Scenario flow:**

```
Kali Linux (attacker)
  └─ Brute-force login attempts       → Event ID 4625 (multiple failures)
      └─ Successful authentication    → Event ID 4624
          └─ SMB lateral movement     → Sysmon Event ID 3 (network connection, T1021.002)
              └─ PowerShell execution → Sysmon Event ID 1 (process creation)
                  └─ (Optional) outbound C2-style connection → Sysmon Event ID 3 (T1071)
                      └─ Splunk detects the full chain
                          └─ Investigation report written
```

**Topics:**

- Full attack Kill Chain reproduced in the lab
- End-to-end event tracking in Splunk using SPL
- MITRE ATT&CK tactic mapping across the chain — Initial Access → Credential Access → Execution → Lateral Movement → (Command and Control, if simulated)
- **Investigation report written in standard IR format:** Timeline / Affected Assets / MITRE ATT&CK Mapping / Containment Action / Lessons Learned
- Published as a blog post

---

> **Milestone 5 — Basic Incident Investigation Capability**
> Completed: A realistic multi-stage attack has been simulated, detected, and documented end-to-end in a standard incident report format.

---

### Phase 6 — Documentation & Portfolio Development

---

#### Day 16 — Network Diagram & Lab Architecture (Final)

**Topics:**

- Final network diagram reflecting the actual architecture: dedicated Ubuntu Splunk indexer + Universal Forwarder on Windows 11
- VM relationships and monitoring flow finalised
- Splunk architecture added to the diagram
- Portfolio front page updated

---

#### Day 17 — Detection Rules & Monitoring Summary

**Topics:**

- Detection examples with SPL queries
- Alert workflow summary
- **False positive tuning** — note on any alert adjusted after being too sensitive, and why
- Full MITRE ATT&CK mapping table (all techniques observed across the lab)
- Lessons learned

---

#### Day 18 — Final Portfolio Summary

**Topics:**

- Full project reflection
- Technical growth documented
- Blue Team concepts and skills demonstrated
- GitHub blog final review and clean-up
- Future improvements noted

---

## MITRE ATT&CK Mapping Summary

> This table consolidates all techniques observed across the lab. It demonstrates the ability to connect raw log observations to a recognised threat framework — a core SOC analyst skill.

| Lab exercise | Tactic | Technique | ID |
|---|---|---|---|
| Nmap port scanning | Reconnaissance | Active Scanning | T1595 |
| SMB enumeration | Discovery | Network Share Discovery | T1135 |
| Brute-force login simulation | Credential Access | Brute Force | T1110 |
| PowerShell execution monitoring | Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |
| SMB lateral movement | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| DNS query monitoring (Sysmon ID 22) | Command and Control | Application Layer Protocol | T1071 |
| Suspicious file creation (Sysmon ID 11) | Defense Evasion | Masquerading | T1036 |

---

## Portfolio Summary

**Practical skills demonstrated:**

- Network visibility — Nmap, Wireshark
- Windows authentication logging — Event Viewer, Event ID 4624 / 4625
- Endpoint telemetry — Sysmon Event ID 1, 3, 11, 22
- SIEM monitoring — Splunk (detection, alerting, dashboard)
- Incident investigation — end-to-end attack scenario
- Threat framework mapping — MITRE ATT&CK
- Technical documentation — GitHub blog with structured investigation notes
