---
title: "Day 14 - Privileged Group Membership and Account Lockout Investigation"
categories: [HomeLab]
---

## Overview

Day 13 set up the Active Directory environment: a Domain Controller running `homelab.local`, two domain accounts (`elkin` as a standard user, `Sid Swar` as a Domain Admin), and Splunk collecting logs from both the DC and Win11.

Day 14 covers two types of AD security events that come up regularly in SOC work: privilege group membership changes and account lockouts. Both are only logged on the DC, which is why getting DC logs into a SIEM matters.

---

## Key AD Security Event IDs

| Event ID | Description | SOC Relevance | MITRE ATT&CK |
|---|---|---|---|
| 4720 | User account created | Attacker creating a backdoor account | T1136 - Create Account |
| 4722 | Account enabled | Re-enabling a dormant account for access | T1098 - Account Manipulation |
| 4724 | Password reset | Credential manipulation | T1098 - Account Manipulation |
| 4725 | Account disabled | Attacker disabling accounts to cause disruption | T1531 - Account Access Removal |
| 4728 | Member added to security group | Privilege escalation, e.g. adding a user to Domain Admins | T1098 - Account Manipulation |
| 4729 | Member removed from security group | Cleaning up traces after privilege abuse | T1098 - Account Manipulation |
| 4740 | Account locked out | Indicator of brute force activity | T1110 - Brute Force |

All of these events are recorded on the DC only. They do not show up in Win11's local event log. Without DC logs in your SIEM, you have no visibility into any of this activity.

---

## Stage 1 - Privileged Group Membership Changes

### Adding Elkin to Domain Admins

In `Active Directory Users and Computers`, I opened `Elkin E. Jose` properties, went to the `Member Of` tab, and added `Domain Admins`.

![Elkin added to Domain Admins](/assets/images/HomeLab/day14/1elkinadmin.png)

`Domain Admins` now shows up alongside `Domain Users` in the membership list.

### Removing Elkin from Domain Admins

Right after, I selected `Domain Admins` in the same tab and clicked `Remove`.

![Elkin removed from Domain Admins](/assets/images/HomeLab/day14/2elkinremove.png)

Back to `Domain Users` only.

### Splunk Detection

```spl
index=wineventlog EventCode IN (4728, 4729)
| table _time, EventCode, Account_Name, Member_Security_ID, Group_Name, ComputerName
| sort _time
```

![Splunk 4728 and 4729 events](/assets/images/HomeLab/day14/3event.png)

Both events show up in Splunk within seconds. `ComputerName` is `KIMDC.homelab.local` for both, confirming these were logged on the DC only, not on Win11.

The `Account_Name` field shows `Administrator` (who made the change) and `CN=Elkin E. Jose` (who was affected). In a real investigation, the Subject account is the first thing to check. If it is not a known IT admin, or the change happened outside business hours with no change ticket, that is a red flag.

#### Event ID 4728 (raw)

![4728 raw event](/assets/images/HomeLab/day14/4e4728.png)

- **Subject: Administrator** - who made the change
- **Member: CN=Elkin E. Jose** - who received the privilege
- **Group Name: Domain Admins**
- **ComputerName: KIMDC.homelab.local**

#### Event ID 4729 (raw)

![4729 raw event](/assets/images/HomeLab/day14/5e4729.png)

Same structure, showing the privilege was removed one minute later.

---

## Stage 2 - Account Lockout Policy and 4740 Detection

### Configuring Account Lockout Policy

Before this change, no lockout policy was in place.

![Policy before configuration](/assets/images/HomeLab/day14/6policy.png)

`Account lockout threshold` was at `0 invalid logon attempts`, meaning accounts could never be locked out no matter how many failed attempts happened. This is the same state the lab was in during Day 12, where Hydra ran unlimited RDP brute force attempts with nothing stopping it.

After editing the Default Domain Policy:

![Policy after configuration](/assets/images/HomeLab/day14/7policyedit.png)

| Setting | Value |
|---|---|
| Account lockout threshold | 5 invalid logon attempts |
| Account lockout duration | 30 minutes |
| Reset account lockout counter after | 30 minutes |

The 30-minute reset window matters. A short window like 5 minutes lets an attacker try 4 failed logins, wait for the counter to reset, and repeat without ever getting locked out. 30 minutes makes that much harder.


### Triggering a Lockout

On Win11, I logged in as `homelab\elkin` with the wrong password repeatedly.

![Invalid credentials message](/assets/images/HomeLab/day14/8loginfail.png)

After 5 failed attempts:

![Account locked out message](/assets/images/HomeLab/day14/8locked.png)

### Splunk Detection

```spl
index=wineventlog EventCode=4740
| table _time, Account_Name, Caller_Computer_Name, ComputerName
| sort _time
```

![Splunk 4740 event table](/assets/images/HomeLab/day14/9event.png)

![4740 raw event](/assets/images/HomeLab/day14/10raw.png)

Key fields:
- **Account That Was Locked Out: elkin** - which account got locked
- **Caller Computer Name: KIM** - where the failed attempts came from (Win11)
- **ComputerName: KIMDC.homelab.local** - where the event was recorded (DC)

`Caller Computer Name` and `ComputerName` point to different machines. The lockout is always recorded on the DC because the DC handles domain authentication, but `Caller Computer Name` shows which host actually sent the bad login attempts. In a large environment, this is the first field to check when a 4740 alert fires, because it tells you where to look next.

---

## Day 12 vs Day 14 - Local Account vs Domain Account

[Day 12](https://hkim0115.github.io/homelab/Day-12-Brute-Force-Authentication-Simulation/) showed Hydra running unlimited RDP brute force against a local account with no lockout policy.

| | Day 12 (Local Account) | Day 14 (Domain Account) |
|---|---|---|
| Account type | Local | Domain (`homelab.local`) |
| Lockout policy | None | 5 attempts, 30 min lockout |
| Brute force result | Unlimited attempts, no lockout | Locked after 5 failures, 4740 generated |
| Log location | Win11 only | DC only, forwarded to Splunk |
| Detection in Splunk | High volume of 4625 events | Single 4740 event |

Day 12 required looking at a spike in 4625 volume to suspect something was happening. Day 14 produces one clear 4740 event that tells you exactly what happened, which account, and where it came from. Domain environments have a built-in lockout mechanism that local-only setups do not.

---

## MITRE ATT&CK Mapping

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| Standard user added to Domain Admins | Persistence | Account Manipulation | T1098 |
| Repeated failed logins triggering lockout | Credential Access | Brute Force | T1110.001 |

---

## Lessons Learned

Day 14 introduced a set of event IDs that look like routine admin activity on the surface: creating accounts, resetting passwords, adding users to groups. The key insight was understanding why each of these matters from both an attacker and defender perspective. An attacker who gains admin access will often perform exactly these same actions, such as adding a backdoor account or escalating a compromised user to Domain Admins, and the logs they leave behind are identical to what a legitimate IT admin would generate.

That shift in perspective changed how I read these logs. Rather than treating a 4728 or 4720 as a routine record to be filed away, the question becomes: who did this, when, and does it match any expected change activity? The log itself is not enough. The context around it, such as the Subject account, the time of day, whether a change ticket exists, and what else was happening on the same host at the same time, is what separates a normal admin action from an attacker covering their tracks or establishing persistence.


---

## Self Q&A

**Q: A 4728 event fires in your SIEM. What is the first thing you check?**

The Subject field, meaning which account made the change. If it is a known IT admin account during business hours, it is probably routine. If it is a service account, a regular user account, or anything outside normal admin identities, that needs investigation. Timing also matters: was this change part of a ticket or a scheduled change window? A privilege change with no corresponding record is a red flag regardless of who the Subject is.

**Q: How do you tell the difference between a 4740 caused by a user forgetting their password and one caused by a brute force attack?**

Start with `Caller Computer Name`. If it matches the user's own workstation, it is probably a forgotten password or a cached credential issue. If it points to an unexpected host, a server, or a machine that does not belong to that user, that is worth escalating. Volume also matters: a single 4740 when someone returns from leave is normal. Multiple 4740 events across different accounts in a short time window, all from the same `Caller Computer Name`, looks like a brute force pattern.

**Q: The 4740 event is recorded on the DC, but the failed logins happened on Win11. Why?**

Domain account authentication is handled by the DC, not the endpoint. When someone logs into a domain-joined machine, the authentication request goes to the DC via Kerberos or NTLM. The DC tracks the failed attempt counter and enforces the lockout policy. The endpoint just passes the request through. That is why the event shows `ComputerName: KIMDC` and why `Caller Computer Name` is the field that tells you where the attempts actually came from.

**Q: Why does cross-referencing DC logs with endpoint logs matter in a real investigation?**

The 4740 on the DC tells you that elkin was locked out and that Win11 (KIM) was the source. But it does not tell you what was running on Win11 at that time. To complete the picture, you would pivot to Win11's Sysmon and Security logs to look for suspicious processes, remote connections, or anything that preceded the failed logins. DC logs tell you what happened to the account. Endpoint logs tell you what was happening on the machine. A SIEM that has both makes that pivot a single query.