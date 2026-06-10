---
title: "Day 6 — Network Connections & PowerShell Visibility"
categories: [HomeLab]
---

## Overview

Today's focus was on applying the SwiftOnSecurity Sysmon configuration and observing how Sysmon records network connections (Event ID 3) and PowerShell activity (Event ID 1). 

The goal was to understand what normal and suspicious process behaviour looks like in logs before moving into attack simulations.

---

## Step 1 — Applying SwiftOnSecurity Sysmon Config

To improve endpoint visibility, I applied the SwiftOnSecurity Sysmon configuration.

This configuration enables additional telemetry such as:

- Process Creation (Event ID 1)
- Network Connections (Event ID 3)
- File Creation (Event ID 11)
- DNS Queries (Event ID 22)

![image](/assets/1swift_down.png)
![image](/assets/2swift_config.png)

Downloaded the SwiftOnSecurity configuration file from GitHub using PowerShell on the Windows VM.

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\sysmonconfig.xml"
```

Applied the config via CMD (Administrator):

```cmd
Sysmon64.exe -c C:\sysmonconfig.xml
```

Output confirmed successful load:

```
Loading configuration file with schema version 4.50
Sysmon schema version: 4.91
Configuration file validated.
Configuration updated.
```
![image](/assets/3swift_config_check.png)

Verified the active config with `Sysmon64.exe -c`, which showed `Config file: C:\sysmonconfig.xml`.

> The SwiftOnSecurity config is a community-maintained ruleset that filters out noisy, low-value events and keeps focus on behaviours relevant to detection — such as suspicious process creation and unexpected network connections.

---

## Step 2 — Network Connection Monitoring (Event ID 3)

![image](/assets/4kali_ping_nmap.png)

Triggered network activity from Kali Linux targeting the Windows VM.

```bash
ping 192.168.56.5
nmap -p 80,443,445 192.168.56.5
```

Ping was successful with 0% packet loss. Nmap reported all three ports as `filtered`, consistent with the Windows firewall behaviour observed in earlier days.

![image](/assets/5event3.png)

In Event Viewer, Sysmon logged Event ID 3 entries for outbound connections from the Windows VM. One notable entry showed **OneDrive.exe** connecting to an external IP on port 443 (HTTPS) which is a normal background sync process, but a good example of how Sysmon captures all network activity regardless of source.

A second Event ID 3 confirmed **powershell.exe** making an outbound connection to port 443 after running `Invoke-WebRequest -Uri "http://example.com"`


![image](/assets/7event3.png)

Key fields in Event ID 3:

| Field | Value |
|---|---|
| Image | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| Protocol | tcp |
| SourceIp | 10.0.2.15 |
| DestinationIp | 172.66.147.243 |
| DestinationPort | 443 |
| DestinationPortName | https |

---

## Step 3 — PowerShell Activity Monitoring (Event ID 1)

### Process creation — notepad.exe

![image](/assets/5ps_note.png)

Ran `Start-Process notepad.exe` from PowerShell to generate a clean Event ID 1 for comparison.

```powershell
Start-Process notepad.exe
```
![imgae](/assets/6event1_note.png)

Sysmon logged the process creation. Notably, the CommandLine field included a long `/SESSION:...` token — an internal Windows session identifier, not user input. This is normal for Store apps like the modern Notepad.

### Encoded PowerShell execution

![image](/assets/6ps.png)

```powershell
$cmd = "whoami"
$encoded = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes($cmd))
powershell.exe -EncodedCommand $encoded
```

Output: `kim\hkim`

![image](/assets/8event1.png)

Sysmon Event ID 1 captured this with full detail:

| Field | Value |
|---|---|
| Image | `powershell.exe` |
| CommandLine | `powershell.exe -EncodedCommand dwBoAG8AYQBtAGkA` |
| ParentImage | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| ParentCommandLine | `powershell.exe -EncodedCommand dwBoAG8AYQBtAGkA` |

---

## Step 4 — Parent-Child Process Analysis

The most interesting log of the day came from the encoded PowerShell execution chain. Sysmon captured a separate Event ID 1 for `whoami.exe` being spawned as a child process:

| Field | Value |
|---|---|
| Image | `C:\WINDOWS\system32\whoami.exe` |
| CommandLine | `"C:\WINDOWS\system32\whoami.exe"` |
| ParentImage | `powershell.exe` |
| ParentCommandLine | `powershell.exe -EncodedCommand dwBoAG8AYQBtAGkA` |

This shows the full execution chain: an encoded PowerShell command spawning `whoami.exe`. 

From a detection perspective, this is exactly the kind of parent-child relationship that would trigger an alert in a real environment — a process launched by a PowerShell session that was itself running an encoded command.

---

## Key Observations

**Sysmon logs everything, not just threats.** Background processes like OneDrive, CameraMonitor, and Windows services all generate Event ID 1 and Event ID 3 entries. Identifying signals in that noise is a core SOC skill.

![image](/assets/camera.png)

During this session, the `svchost.exe -k CameraMonitor` entry briefly caught my attention — the name suggested something was accessing the camera. On closer inspection, it was a standard Windows service managed by `services.exe` running as SYSTEM, which is entirely normal behaviour. This was a useful reminder that process names alone can be misleading. A similar thing came up during an internship, where the same log prompted a second look before being cleared. Without prior exposure to what normal system logs look like, it is easy to either over-investigate routine events or, conversely, dismiss something genuinely suspicious because it looks unfamiliar. Building that baseline understanding of normal behaviour is what makes the difference in a real SOC environment.

**Encoded commands leave clear evidence.** Even though `-EncodedCommand` is designed to obscure the payload, Sysmon records the full command line including the Base64 string. The encoding itself becomes a detection indicator.

---

## MITRE ATT&CK Mapping

| Observation | Tactic | Technique | ID |
|---|---|---|---|
| PowerShell execution with `-EncodedCommand` | Execution | Command and Scripting Interpreter: PowerShell | T1059.001 |

