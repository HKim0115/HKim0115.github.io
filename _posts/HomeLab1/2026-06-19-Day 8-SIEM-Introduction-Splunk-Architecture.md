---
title: "Day 8 — SIEM Introduction & Splunk Architecture"
categories: [HomeLab]
---

## What is a SIEM?

A SIEM(Security Information and Event Management) collects logs from multiple systems and stores them in one place. This makes it easier to monitor activity, investigate incidents, and retain logs even if an endpoint is compromised.


Up until now, I had been looking at logs directly through Event Viewer and Sysmon. It worked fine for a single machine, but I started wondering how logs would be handled in a real SOC environment where multiple systems generate events at the same time. That led me to SIEM platforms, and for this lab I decided to learn Splunk.

## Separating the Log Server from the Target Machine

At first, I was going to install Splunk directly on my Windows VM because it seemed easier. After reading more about how SIEMs are used in real environments, I realised it made more sense to separate the log server from the machine generating the logs.

Running the SIEM on the same machine I'm attacking in later simulations didn't make much sense to me. If the "compromised" endpoint and the log server are the same machine, log integrity becomes questionable as soon as the attack starts. I know my home lab is much smaller than a real SOC, but I wanted to follow the same idea of keeping log storage separate from monitored endpoints, so I built a third VM for it.

**Updated lab network:**

| Role | IP |
|---|---|
| Kali (attacker) | 192.168.56.101 |
| Windows 11 (target + forwarder) | 192.168.56.5 |
| Splunk Indexer (Ubuntu 26.04 LTS) | 192.168.56.103 |

The indexer VM has two adapters: NAT (for package updates / downloading Splunk) and a Host-Only adapter on the same `192.168.56.0/24` segment as Kali and Windows. All three VMs now have static IPs instead of DHCP leases — Universal Forwarder configs and SPL queries will hardcode these addresses later, so they need to stay fixed.

![image](/assets/images/HomeLab/day8/1splunkvm.png)

## Installing Splunk Enterprise

Downloaded the Linux `.deb` package directly via `wget` from the Ubuntu VM itself:

![image](/assets/images/HomeLab/day8/2splunkwg.png)

```bash
wget -O splunk-10.4.0-linux-amd64.deb "https://download.splunk.com/..."
sudo dpkg -i splunk-10.4.0-linux-amd64.deb
```

![image](/assets/images/HomeLab/day8/3splunkdown.png)


Once running, Splunk Web came up at `http://192.168.56.103:8000`:

![image](/assets/images/HomeLab/day8/4splunkweb.png)

## Configuring the Indexer

**Opened the forwarder receiving port** so the indexer is ready before Day 9's Universal Forwarder setup:

Settings → Forwarding and receiving → Configure receiving → port `9997`

![image](/assets/images/HomeLab/day8/7config.png)

**Created two custom indexes** instead of dumping everything into `main` — separating by source makes retention policy and search performance easier to manage, and mirrors how real indexers are typically organised:

- `wineventlog` — Windows Security log events (4624/4625)
- `sysmon` — Sysmon telemetry (Event IDs 1, 3, 11, 22)

![image](/assets/images/HomeLab/day8/5newindex.png)

## Quick Validation

Before moving on to the next stage, I wanted to make sure Splunk was actually working rather than assuming the installation had succeeded.

```spl
index=_internal
```

38,294 events came back — Splunk's own internal logs about its own operation. No Windows or Sysmon data yet (that's Day 9), but it confirms the search pipeline is healthy.

![image](/assets/images/HomeLab/day8/6search.png)

## What's Next

Day 9: install Splunk Universal Forwarder on the Windows 11 VM, point it at `192.168.56.103:9997`, and verify Windows Event Log + Sysmon data actually lands in the `wineventlog` and `sysmon` indexes.
