---
title: "Day 12: Brute Force Authentication Simulation"
categories: [HomeLab]
---


## Overveiw

This is a record of simulating a brute force attack against the RDP service on the Windows 11 VM using Hydra from Kali Linux, then detecting and tuning the alert in Splunk.

---

## Attack Simulation

I built a password list with 9 entries and placed the correct password on the last line.

![passwords.txt contents](/assets/images/HomeLab/day12/1kali.png)

Ran Hydra against the RDP service.

```bash
hydra -l "kim's windows" -P passwords.txt rdp://192.168.56.5
```

![Hydra output](/assets/images/HomeLab/day12/2hydra.png)

The 9th password succeeded.

---

## Detection in Splunk

The existing Failed Login - Brute Force Detection alert triggered as expected.

![Triggered Alerts list](/assets/images/HomeLab/day12/4alert.png)
![Triggered Alerts list](/assets/images/HomeLab/day12/5viewresult.png)

But opening View Results only showed `_time` and `Account_Name`. There was no way to tell which IP the attack came from or why each attempt failed. The alert query was missing fields.

I added `Source_Network_Address` to the group-by and re-saved the alert.

![Alert list after the update - Tuned version](/assets/images/HomeLab/day12/6edit.png)

The next trigger included the source IP.

![Statistics results with Source_Network_Address](/assets/images/HomeLab/day12/7viewresult.png)

---

## Investigation

I wrote an investigation query keyed on Account_Name and Source_Network_Address.

![image](/assets/images/HomeLab/day12/8inves.png)

```spl
index=wineventlog (EventCode=4624 OR EventCode=4625)
Source_Network_Address="192.168.56.101"
earliest="06/26/2026:00:15:00" latest="06/26/2026:00:20:00"
| table _time EventCode host Account_Name Source_Network_Address Failure_Reason Logon_Type
| sort _time
```

In the Events tab, I expanded a raw event and checked the Network Information section.

![Network Information from the raw event](/assets/images/HomeLab/day12/9event.png)
![image](/assets/images/HomeLab/day12/10fromkali.png)


`Workstation Name: kali` and `Source Network Address: 192.168.56.101` confirm the attack originated from the Kali VM, directly in the log.

---

## Timeline

![image](/assets/images/HomeLab/day12/8inves.png)

Between 00:17:19.322 and 00:17:21.431, roughly 2 seconds, eight 4625 (failure) events were logged, immediately followed by one 4624 (success) at 00:17:23.451. Hydra ran 4 parallel tasks, so the order of attempts doesn't map 1:1 to the password list order, but eight failures followed by one success matches the list (correct password on the 9th of 9 entries).

Every event recorded `Logon_Type` as 3 (Network). The reason an RDP attack shows up as a network logon rather than a remote desktop logon is that Hydra only checks whether the password is correct, it doesn't go as far as actually opening a login session. So from Windows' point of view, this gets logged as plain network authentication instead of a remote desktop logon (Logon_Type 10).

![image](/assets/images/HomeLab/day12/11NTLM.png)

---

## MITRE ATT&CK Mapping

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Repeated RDP logon failures followed by success | Credential Access | Brute Force: Password Guessing | T1110.001 |

Since the environment only has one account, password spraying (T1110.003), spreading few attempts across many accounts, couldn't be simulated. This was repeated guessing against a single account, so it's classified as T1110.001.

---

## Anticipated Interview Questions

**Q. Why wasn't the alert built in a complete form from the start?**

When I first designed the alert, the goal was just to confirm that detection worked and that the alert fired correctly. Today, once a real brute force attack triggered it and I went to investigate, I realized the fields I needed weren't there. The first version was built to validate detection, not to support investigation, and that gap only became obvious once I tried to actually use it.

**Q. Wasn't it odd that Logon_Type came back as 3?**

Yes. I expected Logon_Type 10 for an RDP attack, so seeing 3 was unexpected. If I had built the alert around Logon_Type=10, this attack would have been missed entirely. Designing it around EventCode and Account_Name instead turned out to be the safer choice.

---

## What Went Wrong

- The initial alert query was missing `Source_Network_Address`, which made investigation impossible. Group-by fields need to account for what's needed during investigation, not just whether the alert fires.
- I didn't know in advance that an RDP attack gets logged as Logon_Type 3 instead of 10. Filtering on Logon_Type alone is risky; the EventCode + Account_Name combination is more reliable.