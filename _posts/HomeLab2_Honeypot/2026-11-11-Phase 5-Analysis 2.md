---
categories: [Honey Pot]
title: "Phase 5-B. SOC Investigation Flow"
---

> Project Environment: AWS (LightSail) + Splunk |  The data is **real attack traffic from the internet**.

## Summary
A bot logged into my honeypot and ran **9 commands, one second apart, in about 10 seconds**.
It checked the device type, hardware, rival crypto miners, and Telegram and SMS-modem files.
It looks like a **"find a useful machine" script**.
Blocking the IP is low value. **Behavior-based detection** is the better response.

## The Flow (7 questions)
What happened? → Which session looks odd? → What exactly did it do? → Why? → Is this the first time? → Is it a known bad IP? → What's the next step?

---

## 1. What happened? Get the big picture
My honeypot accepts most logins, so a successful login tells me little. What matters is what the attacker does **after** logging in. I counted commands per session to get a rough picture first.

📷 [Screenshot: commands per session](/assets/images/Honeypot/phase5b/1_ip-command-counts.png)

- **Observed:** a few sessions run many commands.
- **Next question:** which one looks suspicious?

## 2. Which session looks odd? Pick a target
I removed my own demo sessions and picked `83.145.42.126`: one session, 9 commands.

## 3. What exactly did it do? Build the timeline
I searched by **session ID**, not by IP. One session ID is one connection, so other connections do not mix into the story. I also filtered to command events only, which removed connection noise.

📷 [Screenshot: session timeline](/assets/images/Honeypot/phase5b/2_session-timeline.png)

| Time | Command |
|---|---|
| 09:23:14 | `/ip cloud print` |
| 09:23:15 | `ifconfig` |
| 09:23:17 | `uname -a` |
| 09:23:18 | `cat /proc/cpuinfo` |
| 09:23:19–20 | `ps | grep '[Mm]iner'` and a variant |
| 09:23:22 | `ls -la ...TelegramDesktop/tdata ...ttyGSM... smsd...` |
| 09:23:23 | `locate D877F783D5D3EF8Cs` |
| 09:23:24 | `echo Hi | cat -n` |

(Times as shown in the Splunk table.)

- **Observed:** the login failed once, then succeeded 1.4 seconds later, and the commands are one second apart. A human does not type like this. It is a **bot**.
- **Next question:** what is it trying to do?

## 4. Group the commands by purpose
I grouped the commands by what they are for.

| Purpose | Commands |
|---|---|
| Identify the device | `/ip cloud print` (a MikroTik router command) |
| Collect system info | `ifconfig`, `uname -a`, `cat /proc/cpuinfo` |
| Check for rivals | `ps | grep '[Mm]iner'` (is a miner already running?) |
| Look for things to steal | Telegram `tdata`, SMS modem paths, `locate D877F...` |

- **Observed:** it searched for a Telegram session folder and SMS modem paths. `D877F...` is the name of the folder inside Telegram Desktop's `tdata` directory that holds the first logged-in account's session data.
- **Learned:** the order is *what device → how powerful → any rivals → anything to steal*.

## 5. Is this the first time? Check the IP history
I searched the IP across all time. It appeared once in my honeypot: 1 session, 34 events, SSH only, on September 6.

📷 [Screenshot: IP history](/assets/images/Honeypot/phase5b/)

## 6. Is it a known bad IP? Check reputation
I looked up the IP on VirusTotal.

📷 [Screenshot: VirusTotal summary, account name hidden](/assets/images/Honeypot/phase5b/)

- 5 of 92 vendors flag it as malicious. It belongs to a Swedish ISP (GlobalConnect AB).
- Guardpot reported **SSH brute force and port scanning from this IP between April and August**.
- It was new to **my** honeypot, but not new to the internet.
- A business-network IP scanning for months may be a **compromised device**. (Inference.)

---

## Conclusion
| Item | Assessment |
|---|---|
| Goal | Evaluate the machine, then look for mining or account-theft targets (inferred) |
| Impact | None (honeypot) |
| Severity | Low in the honeypot (no real impact). **High if this were a real server:** an external IP logged in as root with a weak password. The bot only ran recon, but on a real system those commands would have returned real data and it could have continued. |
| Limits | I could not confirm real theft or who controls the IP |

**MITRE ATT&CK:** T1078.001 (Default Accounts), T1082 (System Information Discovery), T1016 (Network Configuration Discovery), T1057 (Process Discovery), T1083 (File and Directory Discovery)

## Recommended Response
1. **Do not rely on blocking this IP.** It visited once, and a compromised host changes IPs.
2. **Detect behavior, not IPs.** Alert when one session shows system info collection, a miner check, and a theft-target search within 30 seconds.
3. **Record IOCs:** the unusual strings and the command order.
4. **Check the attack surface:** disable SSH password login and direct root login, use keys, and put SSH behind a VPN.

## What I Learned
Good investigation is not about knowing every command. It is about asking the next right question, following a logical flow, and putting each clue (commands, timing, IP history, reputation) into the right context.

