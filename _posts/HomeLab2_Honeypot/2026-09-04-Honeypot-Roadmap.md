---
categories: [Honey Pot]
title: "Honeypot Roadmap"
---


Honeypot Project



Cloud VPS honeypot (AWS + Cowrie + Splunk)

## Phase 1: Environment Setup

- Spin up VPS (AWS Lightsail)
- Harden: new SSH key, change admin port, firewall rules
- Install Cowrie
- Customize hostname / banner
- Local test, confirm logging works

**Milestone 1:** VPS hardened, Splunk ready, Cowrie live and logging locally.

## Phase 2: Web Honeypot

- Build/deploy a fake web app (login page or common admin panel look-alike)
- Cover common attacker-probed paths (e.g. /wp-admin, /.env, /admin)
- Run it on an internal port, redirect public port 80 (and 443 if used) to it via iptables — same pattern as the Cowrie port redirect
- Log every request: path, method, headers/User-Agent, submitted payloads (SQLi attempts, credential stuffing, etc.)
- Register as systemd service
- Local test, confirm logging works alongside Cowrie

**Milestone 2:** Web honeypot live, public port 80 routes to the fake app, requests logged locally.

## Phase 3: Splunk Integration

- Enable HTTP Event Collector (HEC)
- Test event send from VPS
- Forward Cowrie + web honeypot logs to Splunk
- Set up field extraction (source IP, credentials, commands, request paths/payloads)
- Build dashboard (attempt counts, country distribution, credential patterns, attack type breakdown)

**Milestone 3:** Cowrie and web honeypot logs flowing into Splunk in real time, dashboard working.

## Phase 4: Data Collection

- Let honeypots run and collect real attack traffic
- Daily check: services up, logs accumulating
- Take an instance snapshot at some point in this window

**Milestone 4:** A week of real attack data collected across both honeypots, no downtime, snapshot saved.

## Phase 5: Analysis and Documentation

- Review data: top IPs, credential patterns, notable command sequences, web attack types
- Map findings to MITRE ATT&CK
- Finalize dashboard for screenshots
- Write analysis post (results-focused, not setup steps)
- Publish to GitHub portfolio, update README

**Milestone 5:** Analysis complete, mapped to ATT&CK, published as results-focused post.

## Phase 6: Wrap-up

- Save final snapshot, decide keep-running vs. teardown
- Update resume and LinkedIn
- Short retrospective: what was learned, what's next (RDP honeypot, other protocols, etc.)

**Milestone 6:** Project closed out, reflected in portfolio, resume, and LinkedIn.
