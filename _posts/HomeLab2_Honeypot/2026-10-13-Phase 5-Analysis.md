---
categories: [Honey Pot]
title: "Phase 5. Analysis: Investigating a 3-Hour SSH Brute Force Burst"
---

In this post I analyse real attack traffic that my Honeypot server forwarded to Splunk. The Honeypot is a real server I deployed on a cloud VPS, so everything here came from actual attackers on the internet.

One of the main reasons to run a Honeypot is to study attack patterns, then use what you see to set baselines and prepare detection and response.

**Key findings**

- A single IP (2.3.98.95) made 2,701 SSH login attempts against `root` in under 3 hours, about one every 4 seconds.
- After each login it ran one command to check whether the shell was real, then disconnected. No files were downloaded.
- Threat intelligence showed it was one part of a two-week internet-wide scanning campaign, most likely coming from a compromised home device.
- I mapped the activity to MITRE ATT&CK and built a Splunk alert with a threshold based on the data.

**Before you read**

1. The server itself is real, but the services attackers reach on it are fake. SSH is handled by Cowrie, a fake SSH service that gives attackers a simulated shell and logs everything they type. The website is a fake login site I built with Flask that records every request. Nothing an attacker does touches the real system.

2. Cowrie is configured to accept almost any password, so most login attempts show up as successful.

---

## SOC Analyst Workflow

In a real SOC, an investigation usually starts when an alert fires. Here I started the other way round: I looked through the data first and hunted for anything unusual. From there I followed a standard L1 triage process (scoping, reputation checks, verdict and response), and at the end I turned the pattern I found into a detection rule, so the next attack like this gets caught as an alert.

The data covers 8 to 16 September 2026 (UTC). The chart below shows events received by the SSH Honeypot and the web Honeypot over that window.

![Event volume over time](/assets/images/Honeypot/phase5/01-event-volume.png)

Five spikes stand out. I started with the biggest one: the SSH spike on 16 September.

> **Note on data:** SSH events stop reaching Splunk shortly after this spike. Troubleshooting the forwarding pipeline is out of scope for this project, since the window already had enough data to analyse.

---

## Investigating the 16 September Spike

The questions I wanted to answer: **who, how many, how, and did it work?**

**Who.** If the traffic comes from one or two IPs, it points to a single-source brute force. If it comes from many, it is a distributed attack, and the analysis and response change.

**Did it work?** In this project, login success on its own does not mean much, because Cowrie accepts most passwords. What matters is what happened *after* login. Running commands or downloading files separates a simple scan from a real intrusion attempt.

### Step 1. Who sent the traffic?

```spl
index=Honeypot sourcetype=cowrie_json
| where timestamp>="2026-09-15T21" AND timestamp<"2026-09-16T03"
| stats count dc(session) as sessions values(eventid) as events by src_ip
| sort -count | head 15
```

![Top source IPs during the spike](/assets/images/Honeypot/phase5/02-top-ips.png)

Out of 21,345 events in this window, 21,314 came from a single IP: **2.3.98.95**. The other IPs sent only two to five events each, which is normal scanning noise. This IP also produced `cowrie.command.input` events, so it went beyond guessing passwords.

### Step 2. When was it active?

I checked whether the IP showed up on any other day.

![Activity timeline for 2.3.98.95](/assets/images/Honeypot/phase5/03-timeline.png)

It was only active from about 23:00 on 15 September to 02:00 on 16 September (UTC), roughly three hours, and produced over 20,000 events. Each SSH session generates about eight events (connect, version exchange, key exchange, login, command, close), so this works out to around 2,700 sessions.

### Step 3. Threat intelligence (reputation check)

**VirusTotal**

![VirusTotal result](/assets/images/Honeypot/phase5/04-virustotal.png)

Only 5 of 92 vendors flag this IP, but the count is not a score. "Clean" means a vendor has no data on the IP, not that it is safe. Most vendors focus on malware hosting and phishing, so they rarely have an opinion on scanners. The vendor that matters most here is **GreyNoise**, which specialises in internet-wide scanning and marks the IP as malicious. Two vendors label it as phishing, but that does not match anything my Honeypot saw, so I noted it and left it out of the conclusion.

Other details from VirusTotal:

- No communicating or downloaded files are linked to the IP.
- The IP belongs to Orange (AS3215), a French consumer ISP.
- The only certificate seen on it is a free Let's Encrypt certificate for a personal website.

Together this points to a scanning node on a home connection, not a malware or command-and-control server.

**AbuseIPDB**

![AbuseIPDB summary](/assets/images/Honeypot/phase5/05-abuseipdb-summary.png)

![AbuseIPDB reports](/assets/images/Honeypot/phase5/06-abuseipdb-reports.png)

The IP was reported 10 times by 8 independent sources between 5 and 21 September 2026, many of them other Honeypot operators. The reports cover SSH and FTP brute force and probes on web and other service ports. About four hours after it left my Honeypot, another Honeypot reported it.

So this was not an attack aimed at my server. It was one stop in a two-week campaign that was sweeping the internet for SSH, FTP, web and other services.

### Step 4. Credential pattern

```spl
index=Honeypot src_ip=2.3.98.95 eventid=cowrie.login.*
| sort timestamp
| table timestamp eventid password
```

![Login attempts in order](/assets/images/Honeypot/phase5/07-login-order.png)

Every attempt used `root`. The bot started with `123456` (the only attempt that failed), tried a few number-based passwords, then went through a wordlist in reverse alphabetical order. That is clear evidence of brute force.

Attempts came every 3 to 4 seconds on average. Each cycle was the same: open a session, log in, run a check, disconnect. That rhythm is an automated script, not a person.

### Step 5. What did it do after logging in?

```spl
index=Honeypot src_ip=2.3.98.95 eventid=cowrie.command.input
| stats count by input
| sort -count
```

![Command run after login](/assets/images/Honeypot/phase5/08-command.png)

Every session ran the same single command:

```
echo -e "\x6F\x6B"
```

`\x6F` and `\x6B` are the hex codes for `o` and `k`, so on a real shell this prints `ok`. Writing it in hex instead of plain `echo ok` means a fake shell that just repeats the input back would fail the check. It looks like the bot was confirming that commands really execute on the server before moving on.

### Summary

> **2.3.98.95** (Orange S.A., French consumer ISP)
>
> UTC 2026-09-15 23:06:33 to 2026-09-16 01:58:22 (about 2 hours 52 minutes). 2,666 sessions and 2,665 login attempts against `root`, roughly one every 3 to 4 seconds, a new session each time. Common passwords first, then a wordlist in reverse alphabetical order. Only the first attempt (`123456`) was rejected; Cowrie accepted the remaining 2,700. Every session ran a hex-encoded `echo` to check the shell, then disconnected. No payload downloaded. Assessed as the credential-checking stage of a botnet.

---

## MITRE ATT&CK Mapping

Mapped to MITRE ATT&CK

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| 2,701 password attempts against a single account (`root`) | Credential Access (TA0006) | Brute Force: Password Guessing | T1110.001 |
| `echo` command run through the shell after login | Execution (TA0002) | Command and Scripting Interpreter: Unix Shell | T1059.004 |


---

## Detection Rule

### Setting the threshold

To pick a threshold, I looked at the highest number of login attempts each IP made in any 5-minute window, and how many alerts a 10 or 20 threshold would have produced.

```spl
index=Honeypot sourcetype=cowrie_json eventid=cowrie.login.*
| bin _time span=5m
| stats count as attempts by _time src_ip
| stats max(attempts) as peak_5m count(eval(attempts>=10)) as over_10 count(eval(attempts>=20)) as over_20 by src_ip
| sort -peak_5m
```

![Peak attempts per 5 minutes by IP](/assets/images/Honeypot/phase5/09-threshold.png)

There is a clear gap. The two attack sources peaked at 92 and 34 attempts per 5 minutes, while normal background IPs peaked at 7 and 5. I set the threshold where the numbers drop off: **10 or more attempts in 5 minutes**. It sits just above normal traffic, and is low enough to catch a slower attacker that a threshold of 20 might miss.


### The alert

```spl
index=Honeypot sourcetype=cowrie_json (eventid=cowrie.login.* OR eventid=cowrie.command.input) src_ip!=127.0.0.1
| stats count(eval(like(eventid, "cowrie.login.%"))) as attempts
        count(eval(eventid="cowrie.login.success")) as successes
        count(eval(eventid="cowrie.command.input")) as commands
        dc(username) as unique_users
        values(username) as usernames
        min(_time) as first_seen
        max(_time) as last_seen
        by src_ip
| where attempts >= 10
| iplocation src_ip
| eval first_seen=strftime(first_seen, "%Y-%m-%d %H:%M:%S"), last_seen=strftime(last_seen, "%Y-%m-%d %H:%M:%S")
| table src_ip Country attempts successes commands unique_users usernames first_seen last_seen
```

The alert runs every 5 minutes over the last 5 minutes. The output fields answer what an analyst asks first: did it succeed, did it run commands, which accounts, and is it still going.

**Throttling.** Without throttling, 2.3.98.95 alone would have triggered 35 alerts over three hours. With a 1-hour throttle on `src_ip`, the same attacker alerts at most once an hour (3 alerts instead of 35), while a different IP still alerts straight away.


---

## Verdict and Response

**Verdict:** True positive. Automated brute force attack.
**Severity:** Low (Honeypot, no real credentials or data).

**If this were a real server:**

1. Temporarily block 2.3.98.95 at the firewall. Since it is a home IP that can change, blocking alone is not enough.
2. Check whether any login from this IP succeeded on a real account, and review the session if it did.
3. Disable SSH password login and allow key-based authentication only (the real admin port on this VPS is already set up this way), and change any default or weak passwords.
4. Add automatic blocking for repeated failures, such as fail2ban.

---

## Lessons Learned

- I was surprised by how many attempts a single automated bot makes. Seeing thousands of common and weak passwords tried in a few hours made the risk of weak credentials feel real.
- On VirusTotal, a mostly "clean" result does not mean safe. Some vendors carry more weight than others depending on what you observed.
- Threat intelligence is not proof on its own. It adds context to what your own logs already show.

## Limitations

Working with real attack data was interesting, but almost all of it was password attacks, so I did not get to see a wide range of techniques. Analysing the web Honeypot and collecting data over a longer period are next steps.
