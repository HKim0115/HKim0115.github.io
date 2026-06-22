---
title: "Day 10 — SPL Fundamentals & Search Verification"
categories: [HomeLab]
---

## Overview

With both indexes receiving data from Day 9, today was about actually querying that data using SPL (Search Processing Language). Started with a basic health check across both indexes, then moved into filtering and aggregating login events, including some I generated myself to have real failure data to work with.

---

## 1. Multi-Index Health Check

```spl
index=wineventlog OR index=sysmon
| stats count by index, sourcetype
```

Confirms both indexes are still actively receiving data, broken down by sourcetype.

![image](/assets/images/HomeLab/day10/1query.png)

---

## 2. EventCode 4624 (Logon Success)

```spl
index=wineventlog EventCode=4624
```

252 events came back, which was more than expected. Cross-checking `Logon_Type` and `Account_Name` showed why: most of these were `Logon_Type=5` (service logons) tied to system accounts like `SYSTEM` and `NETWORK SERVICE`, not actual interactive logins.

A quick reference for the Logon Types that actually came up:
 
| Logon Type | Meaning |
|---|---|
| 2 | Interactive (typed at the machine) |
| 5 | Service (a service starting up) |

![image](/assets/images/HomeLab/day10/2event4624.png)

Worth noting: `Logon_Type=2` alone wasn't enough either. Some Windows system processes (`DWM`, `UMFD`) also show up as type 2, even though they're not real logins. Checking `Account_Name` as well, and matching it against the actual Windows username, was what actually separated real logins from system noise.

```spl
index=wineventlog EventCode=4624 Account_Name="kim's windows"
```

![image](/assets/images/HomeLab/day10/2.png)

---

## 3. EventCode 4625 (Logon Failure)

No failed logins existed yet, so I generated some by intentionally entering the wrong password a few times on the lock screen.

```spl
index=wineventlog EventCode=4625
```

![image](/assets/images/HomeLab/day10/3event4625.png)

Expanding one event shows the full failure detail:

![image](/assets/images/HomeLab/day10/4event25detail.png)

---

## 4. Aggregating with stats

```spl
index=wineventlog (EventCode=4624 OR EventCode=4625) Logon_Type=2
| stats count by EventCode
```

6 failures, 8 successes, matching the manual attempts made above.

![image](/assets/images/HomeLab/day10/5stat.png)

---

## 5. Timeline View

```spl
index=wineventlog (EventCode=4624 OR EventCode=4625) Account_Name="kim's windows"
| table _time, EventCode
| sort _time
```

![image](/assets/images/HomeLab/day10/6time.png)

The interesting cluster: five 4625s within about 17 seconds, followed by a 4624. That's the same shape a brute-force detection rule looks for, just done manually instead of automated. This same logic gets built into an actual alert on Day 11, and simulated at scale with Kali on Day 12.


## Self Q&A
 
**Q: 252 EventCode=4624 events came back, more than expected. Just looking at that raw number, is that something to worry about? What would actually tell you whether it's normal or not?**
Not by itself. What matters is the baseline: how many logon events this host normally generates per day, across that same mix of service and interactive logons. A spike well above that baseline is worth investigating, the raw count in isolation isn't.
 
**Q: You filtered on Account_Name to separate real logins from system noise like DWM and UMFD. If an attacker logged in using that exact same account name, would Account_Name alone be enough to call it a legitimate login?**
No. A correct account name only proves the authentication succeeded with valid credentials, not that the legitimate user is the one who typed them. Credentials can be phished, reused from a breach, or brute-forced, so account name narrows things down but has to be cross-checked with other context, like source IP, login time relative to normal working hours, and location, before calling it legitimate.
 
**Q: You called the five-failures-then-success pattern a small-scale version of brute-force. What actually separates that from a real user just fumbling their password a few times?**
Volume and speed. Five failures in 17 seconds is already faster than someone could physically type and second-guess a password, and an automated brute-force tool would push both numbers much further. A real detection rule for this isn't based on a raw failure count alone, it's a threshold paired with a time window, e.g. more than N failures within X minutes, since that's what separates automated attempts from normal human error.
 