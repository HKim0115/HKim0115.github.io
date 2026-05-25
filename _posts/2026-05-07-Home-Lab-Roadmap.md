---
categories: [HomeLab]
title: "Cybersecurity Homelab Roadmap"
---


## Building My Cybersecurity Homelab Journey

The main goal of this lab is to better understand how systems, networks, and security monitoring work from both attacker and defender perspectives. I also want to build stronger practical experience related to SOC analyst and cybersecurity analyst roles.

## Final Goal

The long-term goal is to build a small lab environment where I can practice:

* Network analysis
* Traffic monitoring
* Vulnerability assessment
* Windows and Linux administration
* Log analysis
* Basic incident response workflows
* Security monitoring and detection concepts

## Current Environment

Current setup:

* Kali Linux VM
* Windows 11 VM
* VirtualBox
* Nmap
* Wireshark

I plan to continue expanding this environment as I progress.

---

# Homelab Roadmap

### Phase 1 — Foundations

Topics:

* VM setup and networking
* Connectivity testing
* Network scanning and service enumeration
* SMB analysis and shared folder access
* Wireshark traffic analysis
* Windows firewall and network profile observations
* Basic Linux and Windows troubleshooting

#### Related Posts

* [Day 1 — VM Networking Fundamentals](https://hkim0115.github.io/homelab/Day-1-VM-Networking-Fundamentals/)
* [Day 2 — Nmap Scanning and SMB Enumeration](https://hkim0115.github.io/homelab/Day-2-Nmap-and-SMB/)
* [Day 3 — Wireshark Traffic Analysis and Nmap Reconnaissance](https://hkim0115.github.io/homelab/Day-3-Wireshark/)

---

### Phase 2 — System Monitoring & Visibility

Topics:

* Windows Event Viewer
* Authentication and failed login logs
* Windows Defender Firewall monitoring
* Linux log monitoring
* Monitoring active network connections
* Identifying suspicious activity and processes


### Related Posts

* [Day 4 — Event Viewer & Security Monitoring](https://hkim0115.github.io/homelab/Day-4-Event-Viewer-Monitoring/)

---

### Phase 3 — Vulnerability Assessment

Topics:

* Advanced Nmap scanning
* Nmap NSE scripts
* SMB version and OS discovery
* Vulnerability detection
* Exposure analysis
* Basic CVE research
* Security hardening concepts

### Related Posts

* Coming soon

---

### Phase 4 — Active Directory Fundamentals

Topics:

* Windows Server setup
* Domain controller configuration
* Active Directory users and groups
* Authentication concepts
* Group Policy basics
* Domain enumeration

### Related Posts

* Coming soon

---

### Phase 5 — SIEM & Security Monitoring

Topics:

* Sysmon installation
* Centralized logging
* Wazuh or Splunk setup
* Windows and Linux log ingestion
* Basic detection rules
* Alert investigation workflows

### Related Posts

* Coming soon

---

### Phase 6 — Incident Response Simulations

Topics:

* Brute-force simulations
* Suspicious SMB activity analysis
* PowerShell abuse simulations
* IOC identification
* Basic incident investigation
* Incident documentation and reporting

### Related Posts

* Coming soon

---

### Phase 7 — Documentation & Portfolio Development

Topics:

* Technical lab documentation
* Network diagrams
* Security observations and analysis
* GitHub blog documentation
* LinkedIn project summaries
* Portfolio development

### Related Posts

* Coming soon







### Phase 1 — Foundations

- [x][Day 1 — VM Networking Fundamentals](https://hkim0115.github.io/homelab/Day-1-VM-Networking-Fundamentals/)

Topics

Kali Linux VM setup

Windows 11 VM setup

NAT vs Host-Only

IP address checking

connectivity testing



- [x] [Day 2 — Nmap Scanning and SMB Enumeration](https://hkim0115.github.io/homelab/Day-2-Nmap-and-SMB/)


Topics :

nmap scanning

SMB enumeration

smbclient usage

authentication attempts

Windows firewall behavior



- [x] [Day 3 — Wireshark Traffic Analysis and Nmap Reconnaissance](https://hkim0115.github.io/homelab/Day-3-Wireshark/)

Topics :

ICMP traffic

SMB traffic

packet capture basics

filtered ports

Windows Public vs Private profile effects


## Milestone 1 — Basic Network Visibility



### Phase 2 — Windows Logging & Security Monitoring

- [x] [Day 4 — Event Viewer & Security Monitoring](https://hkim0115.github.io/homelab/Day-4-Event-Viewer-Monitoring/)

Topics :

Windows Event Viewer

Security logs

SMBServer logs

failed login monitoring

successful authentication logs

Event Viewer vs Wireshark

understanding SIEM vs native logs


## Milestone 2 — Host-Level Visibility



### Phase 3 — Endpoint Telemetry & Sysmon

- [ ] Day 5 — Sysmon Installation & Process Monitoring

Topics :

Sysmon installation

SwiftOnSecurity config

Event ID 1

process creation monitoring

cmd.exe / powershell.exe logging



- [ ] Day 6 — Network Connections & PowerShell Visibility

Topics :

Sysmon Event ID 3

PowerShell activity

network connection telemetry

command-line visibility

parent-child process relationships



- [ ] Day 7 — File Creation & DNS Monitoring

Topics :

Sysmon Event ID 11

Event ID 22

file creation logging

DNS query visibility

suspicious activity observations


## Milestone 3 — Endpoint Monitoring Fundamentals



### Phase 4 — SIEM & Centralized Monitoring

- [ ] Day 8 — SIEM Introduction & Architecture

Topics :

what SIEM does

centralized logging concepts

Windows → SIEM workflow

Splunk vs Wazuh overview

log forwarding concepts


- [ ] Day 9 — Installing Splunk or Wazuh

Topics :

SIEM installation

basic configuration

connecting Windows logs

verifying ingestion



- [ ] Day 10 — Sysmon Log Ingestion into SIEM

Topics :

forwarding Sysmon logs

indexes

searching logs

event visibility inside SIEM



- [ ] Day 11 — Basic Detection & Alert Monitoring

Topics :

failed login detection

suspicious PowerShell detection

simple alert creation

investigation workflow basics


## Milestone 4 — SOC-Style Monitoring Lab



### Phase 5 — Incident Simulation & Investigation

- [ ] Day 12 — Brute-Force Authentication Simulation

Topics :

repeated failed logins

Event ID analysis

SIEM correlation

detection observations



- [ ] Day 13 — Suspicious SMB Activity Investigation

Topics :

abnormal SMB access

SMB logs

Wireshark analysis

investigation workflow



- [ ] Day 14 — PowerShell Abuse Simulation

Topics :

encoded PowerShell commands

Sysmon telemetry

SIEM detection

suspicious process analysis



- [ ] Day 15 — IOC Investigation & Log Correlation

Topics :

indicators of compromise

log correlation

endpoint + network analysis

investigation notes


## Milestone 5 — Basic Incident Investigation Capability



### Phase 6 — Documentation & Portfolio Development


- [ ] Day 16 — Network Diagram & Lab Architecture

Topics :

network diagrams

VM relationships

monitoring flow

SIEM architecture visualization



- [ ] Day 17 — Detection Rules & Monitoring Summary

Topics :

detection examples

alert workflow summary

monitoring observations

lessons learned



- [ ] Day 18 — Final Home Lab Portfolio Summary

Topics :

project reflection

technical growth

Blue Team concepts learned

future improvements


### portfolio summary

### Final Portfolio Goal

Demonstrate practical understanding of:


network visibility

Windows authentication logging

endpoint telemetry

SIEM monitoring

incident investigation

technical documentation
