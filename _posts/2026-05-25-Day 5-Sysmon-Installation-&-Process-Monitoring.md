---
categories: [HomeLab]
title: "Day 5. Sysmon Installation & Process Monitoring"
---

<br>

### Lab Overview

In previous labs, I monitored authentication and SMB-related activity using Windows Event Viewer.

Although Event Viewer provided useful security logs, I noticed that the default Windows logging had limited visibility into process activity and command-line execution.

During my cybersecurity internship, I also used an EDR platform for basic threat hunting through process logs. However, due to the limited visibility available within my role and access level, further investigation into process activity was sometimes difficult.

To improve my understanding of endpoint visibility and process telemetry, I decided to explore Sysmon in this lab.

The goal of this lab is to observe process creation events and understand how Sysmon provides more detailed monitoring compared to standard Windows logs.



### What is Sysmon?

Sysmon (System Monitor) is a Windows system monitoring tool.

Unlike basic Windows Event Viewer logs, Sysmon provides more detailed telemetry such as:

- process creation
- command-line arguments
- parent processes
- network connections
- file creation activity

Sysmon logs are stored inside Event Viewer.


### Installing Sysmon

I downloaded Sysmon from Microsoft Sysinternals and installed it.

![image](/assets/sysmon.png)

After installation, Sysmon logs became available in Event Viewer:

Applications and Services Logs  
→ Microsoft  
→ Windows  
→ Sysmon  
→ Operational

![image](/assets/eventviewer.png)


### Generating Process Activity

To generate logs, I opened several processes:

- Command Prompt
- PowerShell
- Notepad
- ping.exe

![image](/assets/exe.png)


### Event ID 1 — Process Creation

One of the most important Sysmon events is:

Event ID 1 — Process Create

This event records detailed information whenever a new process is executed.

![image](/assets/notepad.png)

Compared to standard Event Viewer logs, Sysmon provided much more detailed visibility.


### Command-Line Visibility

I tested the ping command:

```cmd
ping 8.8.8.8
```

![image](/assets/ping.png)

Sysmon successfully recorded:

- executable path
- command-line arguments
- process information
- parent process

![image](/assets/ping2.png)
![image](/assets/ping3.png)

This was important because Sysmon was not only logging that `PING.EXE` was executed, but also exactly how it was executed.

<br>

### Why This Matters

During investigations, process names alone are often not enough to determine whether activity is normal or suspicious.

<br>

### Potential Detection Opportunities

The activities observed during this lab could potentially be monitored through:

- unusual PowerShell execution
- suspicious command-line arguments
- abnormal parent-child process relationships
- unexpected administrative tool usage

This helped me better understand how endpoint telemetry can support detection and investigation workflows.


### Observations & Reflections

During the lab, I realized that different monitoring tools provide different types of visibility.

For example:

- Wireshark captures network packets
- Event Viewer records authentication and Windows events
- Sysmon focuses on endpoint process activity

I also learned that Sysmon does not directly capture ICMP packets like Wireshark does.

Instead, Sysmon records the process that generated the activity.

For example:

- Wireshark can capture ICMP ping packets
- Sysmon can record that `ping.exe` was executed

This helped me better understand the difference between:

- network visibility
- endpoint visibility

Another important realization was understanding how SIEM platforms work.

At first, I thought SIEM tools generated logs themselves.

However, through this lab, I learned that:

- Windows
- Sysmon
- Event Viewer
- EDR agents

generate telemetry first, and then SIEM platforms collect and correlate those logs centrally for monitoring and investigation.

This gave me a much clearer understanding of how security monitoring environments are structured in real-world SOC environments.
