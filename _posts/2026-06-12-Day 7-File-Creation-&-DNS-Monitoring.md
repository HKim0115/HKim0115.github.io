---
title: "Day 7 — File Creation & DNS Monitoring with Sysmon"
categories: [HomeLab]
---

## Overview

Days 5 and 6 covered process creation and network connections. Today adds the remaining two core Sysmon event types: file writes and DNS queries. Together, these four event types give full endpoint visibility — the foundation for everything in Splunk from Day 9 onwards.

---

## Event ID 11 — File Creation

Sysmon logs a file creation event whenever a file is written to disk, provided the path or extension matches the active configuration. The SwiftOnSecurity config watches high-risk paths and executable extensions by default.

### Lab 1 — PowerShell drops an executable

![image](/assets/1.file.png)

On the Windows VM, I ran:

```powershell
New-Item -Path "C:\Users\hkim\AppData\Local\Temp\update3.exe" -ItemType File -Force
```

**Sysmon Event ID 11 result:**

![image](/assets/2event11.png)

Three things make this suspicious:

- **PowerShell wrote an executable** — legitimate software doesn't usually drop `.exe` files this way
- **AppData\Local\Temp** — a user-writable path requiring no admin rights, commonly used by malware to stage payloads
- **The filename `update.exe`** — a generic name designed to look like a legitimate Windows update

This matches the pattern of a malware dropper staging a payload before execution.

---

### Lab 2 — SMB file drop from Kali

![image](/assets/3smb.png)

From the Kali VM, I created a fake file and dropped it onto the Windows shared folder via SMB:

![image](/assets/3put.png)

```bash
smbclient //192.168.56.5/LabShare -U hkim
put update.exe
```

**Sysmon Event ID 11 result:**

![image](/assets/4event11.png)

This looks very different from the PowerShell drop:

- **Image: System / ProcessId: 4** — the Windows kernel handled the write, not a user process. The file arrived over the network.
- **NT AUTHORITY\SYSTEM** — no interactive user account was involved
- **LabShare path** — a network-accessible folder

A SOC analyst seeing `Image: System` write an `.exe` to a shared folder would immediately correlate this with network logs to find where the connection came from. This is the classic lateral movement file drop pattern.

---

### Comparing the two Event ID 11s

| | PowerShell drop | SMB drop (Kali) |
|---|---|---|
| `Image` | `powershell.exe` | `System` |
| `TargetFilename` | `AppData\Local\Temp\update3.exe` | `C:\LabShare\update.exe` |
| `User` | `kim\hkim` | `NT AUTHORITY\SYSTEM` |
| `ProcessId` | 2252 | 4 |

Different mechanisms, same outcome — an executable appeared on disk that wasn't there before.

---

## Event ID 22 — DNS Query

Sysmon logs DNS queries made by processes on the machine. This is one of the most valuable C2 detection signals available because malware almost always starts with a DNS lookup before attempting to connect to an attacker-controlled server.

### Lab — PowerShell queries a nonexistent domain

![image](/assets/5ps.png)

```powershell
Resolve-DnsName "fakedomain123.xyz" -ErrorAction SilentlyContinue
```

**Sysmon Event ID 22 result:**

![image](/assets/6event22.png)

Key observations:

- **powershell.exe making a DNS query** — browsers and system services query DNS constantly, but PowerShell doing it directly is unusual unless a script is explicitly calling out to an external host
- **QueryStatus 9003 (NXDOMAIN)** — the domain doesn't exist. In real malware using Domain Generation Algorithms (DGA), hundreds of these failed lookups appear in rapid succession as the malware cycles through generated domain names looking for an active C2 server
- **QueryResults shows a nameserver record** — the DNS infrastructure exists but no host is registered, consistent with a randomly generated domain

A sudden spike of NXDOMAIN responses from `powershell.exe` is a strong detection signal in a real SOC environment.

---

## MITRE ATT&CK Mapping

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| PowerShell drops executable in Temp folder | Defense Evasion | Masquerading | T1036 |
| Kali drops executable via SMB | Lateral Movement | SMB/Windows Admin Shares | T1021.002 |
| PowerShell queries nonexistent domain | Command and Control | Application Layer Protocol | T1071 |

---

## Key Takeaways

- Sysmon Event ID 11 captures file writes but only for paths and extensions the active config monitors. SwiftOnSecurity watches `.exe` drops in user-writable locations by default.
- The `Image` field tells you *what* wrote the file. `powershell.exe` writing an executable and `System` writing one via SMB are both suspicious but for different reasons.
- Sysmon Event ID 22 exposes DNS queries at the process level. Knowing *which process* made a lookup is what makes it useful — the same domain queried by a browser vs `powershell.exe` tells a very different story.
- These four event types together (ID 1, 3, 11, 22) give complete endpoint telemetry. From Day 9 onwards, all of this feeds into Splunk for centralised detection.
