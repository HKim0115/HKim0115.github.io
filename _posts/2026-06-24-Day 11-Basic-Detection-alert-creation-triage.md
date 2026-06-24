---
title: "Day 11 - Basic Detection, Alert Creation & Triage"
categories: [HomeLab]
---

## Overeview

Move from collecting logs to detecting on them. Build two Splunk alerts and verify they actually fire.

1. Failed login brute-force alert (EventCode 4625)
2. Encoded PowerShell execution alert (Sysmon EventCode 1)

---

## Alert 1: Failed Login Brute Force Detection

**SPL:**
```spl
index=wineventlog EventCode=4625
| bucket _time span=5m
| stats count by _time, Account_Name
| where count >= 5
```

![SPL query for brute force detection](/assets/images/HomeLab/day11/1spl.png)

**Alert settings:**

![alert settings part 1](/assets/images/HomeLab/day11/2alertset1.png)
![alert settings part 2](/assets/images/HomeLab/day11/2alertset2.png)

- Trigger: Number of Results > 0, Once
- Cron: `*/5 * * * *`
- Time range: Last 15 minutes (later corrected, see below)
- Severity: Medium

Saved and confirmed in the Alerts list, enabled and scheduled correctly.

![saved alert confirmation](/assets/images/HomeLab/day11/3alert.png)

**Test:** Entered the wrong password 6 times on the Windows 11 login screen.

![invalid credentials login screen](/assets/images/HomeLab/day11/4loginattempt.png)

**Result:** 6 separate 4625 events confirmed in Splunk.

![4625 events in Splunk](/assets/images/HomeLab/day11/5failedlogin.png)

**Problem:** The alert fired three times in a row for the same brute-force burst.

![duplicate alert triggers](/assets/images/HomeLab/day11/6alert.png)

**Cause:** Time range (15 minutes) was wider than the cron interval (5 minutes), so each run re-evaluated overlapping data and kept re-triggering on the same 6 events.

**Fix:** Time range narrowed to match the cron interval, plus throttling added as a second layer of protection.

![throttle and time range fix](/assets/images/HomeLab/day11/7timerange.png)
![throttle and time range fix](/assets/images/HomeLab/day11/7throttle.png)
---

## Alert 2: Encoded PowerShell Execution Detection

**SPL:**
```spl
index=sysmon EventCode=1 Image="*\\powershell.exe"
(CommandLine="*-EncodedCommand*" OR CommandLine="*-enc *" OR CommandLine="*FromBase64String*")
| table _time, Computer, ParentImage, Image, CommandLine
```

![SPL query for encoded PowerShell detection](/assets/images/HomeLab/day11/8psquery.png)

**Alert settings:**

![powershell alert settings](/assets/images/HomeLab/day11/9alertset.png)

- Trigger: Number of Results > 0, Once
- Cron: `*/5 * * * *`
- Time range: Last 6 minutes
- Throttle: 10 minutes
- Severity: High (encoded execution is a higher-confidence indicator than a single failed login)

**Test:**
```powershell
powershell.exe -EncodedCommand ([Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes("Get-Host")))
```

![running encoded powershell command](/assets/images/HomeLab/day11/10ps.png)

**Result:** Alert triggered correctly.

![encoded powershell alert triggered](/assets/images/HomeLab/day11/11alert.png)

---

## What went wrong, and why it matters

**Alert 1 fired three times for one event.**

Time range was set to "Last 15 minutes" while the cron ran every 5 minutes. Each run re-evaluated the same 15-minute window, so one brute-force burst got caught and re-alerted on for three consecutive cron cycles.

Fix: match the search time range to the cron interval (Last 6 minutes for a 5-minute cron, with 1 minute of buffer for indexing lag), and add throttling as a second layer of protection against duplicate alerts on the same account.

---

## Self Q&A

**Q: Your brute-force alert triggers on 5+ failures in 5 minutes for one account. What's a realistic way an attacker bypasses this exact logic, and how would you adjust for it?**

Low-and-slow brute-forcing, spacing attempts further apart than the detection window, or rotating across many accounts at a low per-account rate (password spraying) so no single account crosses the threshold. Addressing it means also tracking failure counts aggregated by source IP or workstation, not just by target account.

**Q: If this alert fires at 3am with no other context, what's your first move before escalating?**

Pull the account's recent login history and check whether this matches a known pattern (e.g. a forgotten password, locked-out service account) versus something with no innocent explanation. Check source IP/workstation against expected values. The decision isn't "is this suspicious" in isolation, it's "does this deviate from what's normal for this specific account."

**Q: Why flag encoded PowerShell at all? Plenty of legitimate admin scripts use -EncodedCommand.**

It's true, base64 encoding alone isn't malicious. The value of the alert is as a flag for investigation, not a verdict. What matters next is the decoded content, the parent process, and whether this matches known admin tooling for the environment. Treating "fired" and "confirmed malicious" as the same thing is exactly how alert fatigue starts.

