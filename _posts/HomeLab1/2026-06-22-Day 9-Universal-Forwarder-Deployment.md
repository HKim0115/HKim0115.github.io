---
title: "Day 9 — Universal Forwarder Deployment: Windows Event Log + Sysmon → Splunk"
categories: [HomeLab]
---

## Overview

Installed the Splunk Universal Forwarder on the Windows 11 VM and configured it to forward Windows Event Logs and Sysmon telemetry to the Ubuntu Splunk indexer (`192.168.56.103:9997`) built on Day 8.

Quick note on what the Universal Forwarder actually is: it's a lightweight Splunk agent that doesn't index or search anything itself. It just reads logs locally on the endpoint and ships them off to a separate indexer. That's why this isn't a second Splunk Enterprise install; it's a much smaller component built specifically for this job.

---

## 1. Receiving Indexer

`192.168.56.103:9997`, the Ubuntu Splunk indexer from Day 8. Deployment Server left blank (not needed for a single forwarder).

![image](/assets/images/HomeLab/day9/1uniforip.png)

---

## 2. Windows Event Log Forwarding

Enabled Application, Security, and System logs.

`index=wineventlog` returned 700 events. `host=kim`, `source=WinEventLog:Security` confirms it's genuinely coming from the Windows VM.

![image](/assets/images/HomeLab/day9/2spwin.png)

---

## 3. Sysmon Forwarding

Sysmon wasn't on the installer's checklist, so the channel was added manually in `inputs.conf`.

`index=sysmon` returned 855 events, shown as raw XML (`renderXml = true`).

![image](/assets/images/HomeLab/day9/3spsys.png)

---

## 4. inputs.conf

![image](/assets/images/HomeLab/day9/4input.png)


**Q: What is it?**
The config file that tells the Universal Forwarder what to collect and where to send it.

**Q: Why edit it by hand?**
The installer only checks for standard, built-in Windows Event Log channels. Sysmon is a third-party tool, so it never showed up as an option. Had to add the stanza myself:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = false
index = sysmon
renderXml = true
```

**Q: When would I need to check this again?**
Any time a log source isn't showing up in Splunk. If the channel was never defined here, the forwarder never even attempted to read it, so this is the first place to look before assuming the problem is on the indexer side.

