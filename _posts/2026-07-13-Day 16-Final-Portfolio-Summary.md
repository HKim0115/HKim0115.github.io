---
title: "Day 16 — Final Portfolio Summary"
categories: [HomeLab]
---

This is the last post in the series. closing a detection gap I noticed while
reviewing the project, and being honest about what worked and what didn't.

---

## Closing a Detection Gap

Going back through the earlier posts to write this summary, I noticed
something. Day 13 added the Domain Controller and, with it, Kerberos
authentication logging (Event ID 4768, 4769, 4776). Those log sources
have been sitting in Splunk since then, but I never actually wrote a
detection rule against them.

Having a log source isn't the same as having detection coverage. So before
closing out the project, I wrote SPL alerts for repeated 4768 requests in a
short window, a single account requesting an unusually high number of
service tickets (4769) in a short window, which is a common signal for
Kerberoasting, and repeated 4776 NTLM validation failures. No new
screenshots for this one, just the queries added to the existing alert set
from Day 11.

It's a small addition, but it's a useful reminder: forwarding a log source
doesn't mean it's being watched.

---

## MITRE ATT&CK Mapping Summary

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Nmap port scanning | Reconnaissance | Active Scanning | T1595 |
| SMB enumeration | Discovery | Network Share Discovery | T1135 |
| Repeated failed RDP logins (Hydra) | Credential Access | Brute Force | T1110.001 |
| PowerShell / ADSI execution | Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |
| Suspicious file creation (Sysmon ID 11) | Defense Evasion | Masquerading | T1036 |
| DNS query monitoring (Sysmon ID 22) | Command and Control | Application Layer Protocol | T1071 |
| ACL enumeration on Domain Admins (dacledit.py) | Discovery | Permission Groups Discovery | T1069.002 |
| GenericAll abused for self-elevation | Persistence / Privilege Escalation | Account Manipulation | T1098 |
| Win11 → DC via PsExec | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| whoami / net user / net group post-exploitation | Discovery | Account Discovery | T1087 |
| Kerberos ticket request pattern (4769) | Credential Access | Kerberoasting | T1558.003 |

---

## Technical Growth

This project didn't start with a SIEM. It started with two VMs and a ping
test.

Tools used across the lab: Kali Linux, Nmap, Wireshark, Windows Event
Viewer, Sysmon, Splunk Enterprise with Universal Forwarder, Windows Server
AD DS as a Domain Controller, Hydra, Impacket (psexec.py, dacledit.py),
ADSI PowerShell, dsacls, and GPO.

Building the environment piece by piece, instead of starting from a
pre-built SIEM setup, meant understanding why each layer exists before
relying on it. Network visibility came before host visibility, host
visibility came before Sysmon telemetry, and Sysmon came before
centralizing everything in Splunk. By the time alerts were being written in
Day 11, the underlying log sources weren't a black box.

Splunk specifically mattered because it's a real-world SIEM I hadn't
worked with hands-on before, and because I was studying for CompTIA
Security+ before this project. Concepts from Domain 2 (Threats,
Vulnerabilities, and Mitigations), like password attacks and account
lockout indicators, and Domain 4 (Security Operations), like log analysis,
incident response, and monitoring concepts, had mostly existed as theory
up to that point. Building alerts, triaging them, and writing an actual
incident report in Day 16 turned that theory into something I had
practiced, not just memorized.

The project was framed around SOC analyst workflows, but building the lab
required running the attacker side too. Brute force with Hydra, ACL abuse
with Impacket, lateral movement with PsExec. During my internship, I used
Eclipse XDR to analyze logs that were already sitting there, and the work
was mostly interpreting what a given log meant. Here the question was
often what an attacker would do next, and what that would leave behind.
Thinking from both sides made the defender side sharper. It's easier to
recognize a GenericAll abuse pattern in a log when you've set one up
yourself.

---

## What Went Wrong

A good amount of time went into building the environment from the ground
up, setting up each VM, wiring up log forwarding, getting Sysmon configs
right, rather than running through realistic attack cases. That was a
deliberate choice at the start, since understanding the plumbing mattered
more than speed early on, but it meant less time was spent on case-based
practice than I would have liked.

A lab environment is also inherently limited. Defender had to be disabled
to let the attack chain complete, timing and scale don't match production
traffic, and there's no real user noise to sift through alerts against.

---

## Lessons Learned

- Having a log source forwarded to the SIEM does not mean it's being
  monitored. Detection coverage has to be built and checked deliberately,
  the Kerberos gap above is a direct example.

- Windows has a built-in safety net that automatically resets permissions
  on sensitive groups like Domain Admins on a schedule. During the lab,
  a GenericAll permission I had manually granted for the attack chain got
  quietly removed by this process without any action on my part. It's a
  good reminder that some AD misconfigurations don't stay put, which
  matters when timing an attack or trying to reproduce one.

- An alert built only to confirm that detection logic fires can still lack
  the fields needed for a real investigation. That gap doesn't show up
  until an actual incident needs to be traced through it.

---

## Future Improvements

The next project will lean more toward realistic case simulation, using
existing detection datasets, working through public incident writeups, or
reproducing published attack case studies, rather than building
infrastructure from scratch again.

---

## Q&A

**What's the biggest gap between this lab and a real SOC environment?**

Noise. Every alert in this lab fires against a clean signal because
there's no legitimate background traffic to filter it out from. A real
environment means tuning against thousands of benign events, not zero.

**What was the most useful mistake you made in this project?**

Disabling Defender to let the attack chain complete. It was necessary to
finish the simulation, but it's also a reminder that AV/EDR blocking
something and logging it (Event ID 1116/1117) is itself valid detection
content, and turning it off removes a layer that would normally be part of
the story.