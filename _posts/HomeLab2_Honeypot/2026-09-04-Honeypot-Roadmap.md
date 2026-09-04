---
categories: [Honey Pot]
title: "Honeypot Roadmap"
---


Cloud VPS honeypot (Cowrie + Splunk)

## Step 1: Environment Setup (Day 1-2)

- Day 1
  - Spin up VPS (DigitalOcean / Vultr / Lightsail)
  - Set billing alerts
  - Harden: new SSH key, change admin port, firewall rules
  - install Splunk
- Day 2
  - Install Cowrie
  - Customize hostname / banner
  - Local test, confirm logging works
  - Register as systemd service (auto-restart)

**Milestone 1:** VPS hardened, Splunk installed, Cowrie live and logging locally.

## Step 2: Splunk Integration (Day 3-4)

- Day 3
  - Enable HTTP Event Collector (HEC)
  - Test event send from VPS
  - Forward Cowrie JSON logs to Splunk
  - Set up field extraction (source IP, credentials, commands)
- Day 4
  - Build dashboard (attempt counts, country distribution, credential patterns)

**Milestone 2:** Cowrie logs flowing into Splunk in real time, dashboard working.

## Step 3: Data Collection (Day 5-11)

- Let honeypot run and collect real attack traffic
- Daily check: service up, logs accumulating
- Take an instance snapshot at some point in this window

**Milestone 3:** A week of real attack data collected, no downtime, snapshot saved.

## Step 4: Analysis and Documentation (Day 12-14)

- Day 12
  - Review data: top IPs, credential patterns, notable command sequences
  - Map findings to MITRE ATT&CK
- Day 13
  - Finalize dashboard for screenshots
  - Write analysis post (results-focused, not setup steps)
- Day 14
  - Publish to GitHub portfolio, update README

**Milestone 4:** Analysis complete, mapped to ATT&CK, published as results-focused post.

## Step 5: Wrap-up (Day 15)

- Save final snapshot, decide keep-running vs. teardown
- Update resume and LinkedIn
- Short retrospective: what was learned, what's next (web honeypot, other protocols, etc.)

**Milestone 5:** Project closed out, reflected in portfolio, resume, and LinkedIn.



