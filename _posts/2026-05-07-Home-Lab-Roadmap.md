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

* Kali Linux VM & Windows 11 VM - 08 May 2026
* Nmap - 10 May 2026
* Wireshark - 15 May 2026
* Windows Event Viewer - 18 May 2026
* Sysmon - 25 May 2026

I plan to continue expanding this environment as I progress.

---

# Homelab Roadmap


### Phase 1 — Foundations

✅ [Day 1 — VM Networking Fundamentals](https://hkim0115.github.io/homelab/Day-1-VM-Networking-Fundamentals/)

Topics

Kali Linux VM setup

Windows 11 VM setup

NAT vs Host-Only

IP address checking

connectivity testing

<br>

✅ [Day 2 — Nmap Scanning and SMB Enumeration](https://hkim0115.github.io/homelab/Day-2-Nmap-and-SMB/)


Topics :

nmap scanning

SMB enumeration

smbclient usage

authentication attempts

Windows firewall behavior

<br>

✅ [Day 3 — Wireshark Traffic Analysis and Nmap Reconnaissance](https://hkim0115.github.io/homelab/Day-3-Wireshark/)

Topics :

ICMP traffic

SMB traffic

packet capture basics

filtered ports

Windows Public vs Private profile effects


## Milestone 1 — Basic Network Visibility 🌱


<br>

### Phase 2 — Windows Logging & Security Monitoring

✅ [Day 4 — Event Viewer & Security Monitoring](https://hkim0115.github.io/homelab/Day-4-Event-Viewer-Monitoring/)

Topics :

Windows Event Viewer

Security logs

SMBServer logs

failed login monitoring

successful authentication logs

Event Viewer vs Wireshark

understanding SIEM vs native logs


## Milestone 2 — Host-Level Visibility ☘️


<br>

### Phase 3 — Endpoint Telemetry & Sysmon 

- [ ] Day 5 — Sysmon Installation & Process Monitoring

Topics :

Sysmon installation

Event Viewer integration

Event ID 1

process creation monitoring

cmd.exe / powershell.exe / ping.exe logging

command-line visibility basics

<br>

- [ ] Day 6 — Network Connections & PowerShell Visibility

Topics :

SwiftOnSecurity Sysmon config

Sysmon Event ID 3

network connection telemetry

PowerShell activity

parent-child process relationships

improved logging visibility

why Sysmon configuration matters

<br>

- [ ] Day 7 — File Creation & DNS Monitoring

Topics :

Sysmon Event ID 11

Event ID 22

file creation logging

DNS query visibility

suspicious activity observations


## Milestone 3 — Endpoint Monitoring Fundamentals


<br>

### Phase 4 — SIEM & Centralized Monitoring

- [ ] Day 8 — SIEM Introduction & Architecture

Topics :

what SIEM does

centralized logging concepts

Windows → SIEM workflow

Splunk vs Wazuh overview

log forwarding concepts

<br>

- [ ] Day 9 — Installing Splunk or Wazuh

Topics :

SIEM installation

basic configuration

connecting Windows logs

verifying ingestion

<br>

- [ ] Day 10 — Sysmon Log Ingestion into SIEM

Topics :

forwarding Sysmon logs

indexes

searching logs

event visibility inside SIEM

<br>

- [ ] Day 11 — Basic Detection & Alert Monitoring

Topics :

failed login detection

suspicious PowerShell detection

simple alert creation

investigation workflow basics


## Milestone 4 — SOC-Style Monitoring Lab


<br>

### Phase 5 — Incident Simulation & Investigation

- [ ] Day 12 — Brute-Force Authentication Simulation

Topics :

repeated failed logins

Event ID analysis

SIEM correlation

detection observations

<br>

- [ ] Day 13 — Suspicious SMB Activity Investigation

Topics :

abnormal SMB access

SMB logs

Wireshark analysis

investigation workflow

<br>

- [ ] Day 14 — PowerShell Abuse Simulation

Topics :

encoded PowerShell commands

Sysmon telemetry

SIEM detection

suspicious process analysis

<br>

- [ ] Day 15 — IOC Investigation & Log Correlation

Topics :

indicators of compromise

log correlation

endpoint + network analysis

investigation notes


## Milestone 5 — Basic Incident Investigation Capability


<br>

### Phase 6 — Documentation & Portfolio Development


- [ ] Day 16 — Network Diagram & Lab Architecture

Topics :

network diagrams

VM relationships

monitoring flow

SIEM architecture visualization

<br>

- [ ] Day 17 — Detection Rules & Monitoring Summary

Topics :

detection examples

alert workflow summary

monitoring observations

lessons learned

<br>

- [ ] Day 18 — Final Home Lab Portfolio Summary

Topics :

project reflection

technical growth

Blue Team concepts learned

future improvements

<br>

### portfolio summary

### Final Portfolio Goal

Demonstrate practical understanding of:


network visibility

Windows authentication logging

endpoint telemetry

SIEM monitoring

incident investigation

technical documentation
