---
categories: [Honey Pot]
title: "Phase 3. Splunk Integration"
---


With both honeypots live on the VPS, the goal of this phase was to get their logs into Splunk in real time and build a dashboard that actually makes sense of the data.

## 1. Enable HTTP Event Collector (HEC)

Enabled HEC on my local Splunk instance, created a dedicated token, and routed it to a new `honeypot` index.

## 2. Test event send from VPS

Connected the VPS and my laptop over Tailscale (no ports opened on my home network), then confirmed the link with a HEC health check and a test SSH login that appeared in Splunk within seconds.

## 3. Forward Cowrie + web honeypot logs to Splunk

Set up a lightweight forwarder (`tail` + `jq` + `curl`, running as a systemd service per honeypot) instead of the full Splunk Universal Forwarder, to keep the footprint small on a 512MB VPS.

## 4. Field extraction

Both honeypots already log in JSON, so each line is sent to Splunk as a structured event. This meant every field was extracted automatically, with no regex or manual configuration. The key fields for this project:

- `src_ip`: who is attacking
- `username` / `password`: what credentials bots are trying
- `input`: commands typed inside the fake SSH shell
- `path`, `user_agent`, `form_data`: what the web scanners are requesting and sending
- `timestamp`: passed through as the event time, so delayed events still land at their real attack time

## 5. Dashboard

Each panel on the "Honeypot Overview" dashboard was added to answer a specific question.

### When is it happening?

**Event Volume Over Time** shows activity per honeypot on one timeline. Spikes are the starting point for any investigation: when something jumps, that is where to look first.

**Unique Attackers Per Day** separates "one noisy bot" from "many different sources". A high event count from a small number of IPs means something very different from the same count spread across dozens.

![Event Volume Over Time](/assets/images/Honeypot/3-1overtime.png)

### Who is attacking?

**Top Attacker IPs** surfaces the noisiest sources. These are the first candidates to pivot on in the analysis phase.

**Country Distribution** gives a quick geographic overview. One thing I noticed early: the top countries mostly reflect where cloud providers host servers, not where attackers actually are. For example, the busiest IP belongs to a cloud hosting range. Geography is useful context, but not attribution.

![Top Attacker IPs](/assets/images/Honeypot/3-2topattacker.png)

### What are they trying on SSH?

**Cowrie Credential Attempts**, **Top Usernames** and **Top Passwords** show the wordlists bots are using. Almost every attempt targets `root`, with keyboard patterns and simple number sequences as passwords, which is typical of automated brute-force tools.

**Login Success vs Failed** is included, but it needs context: Cowrie accepts almost any password by design, so a high success rate reflects the honeypot's configuration rather than attacker skill. What matters is what attackers do *after* getting in, which is the focus of Phase 5.

![Credential Attempts](/assets/images/Honeypot/3-3attempts.png)


### What are they looking for on the web?

**Web Honeypot Attack Type** breaks requests down by the fake endpoint they hit (WordPress login, `.env`, `.git/config`, and a catch-all for everything else).

**Top User-Agents** turned out to be one of the most useful panels. A large share of web traffic came from legitimate crawlers and AI bots, not attackers. Separating that background noise from real malicious scanners (such as `libredtail-http`, a user-agent linked to cryptomining campaigns) is exactly the kind of filtering a SOC analyst does every day.

![Top User-Agents](/assets/images/Honeypot/3-4useragent.png)

## Milestone 3

Cowrie and web honeypot logs are flowing into Splunk in real time, fields are extracted automatically, and the dashboard is working with real attack data.

**Next:** Phase 4 (data collection) and Phase 5 (analysis and MITRE ATT&CK mapping).