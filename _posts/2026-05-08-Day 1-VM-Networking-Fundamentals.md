---
categories: [HomeLab]
title: "Day 1. VM Networking Fundamentals"
---

### Environment
- Host OS: macOS
- Hypervisor: VirtualBox
- Kali Linux VM
- Windows 11 VM

### Lab Overview

I created an isolated virtual lab environment using Kali Linux and Windows 11 VMs.

In this lab:

- Kali Linux acts as the attacker machine
- Windows 11 acts as the target/end-user machine

The purpose of this setup is to understand how systems communicate within a private network and how attackers interact with target systems.

In future labs, I plan to add another Linux VM for SIEM tools to monitor and analyse attacker activity.


## 1. Checking IP Addresses

### Kali Linux
![imagie](/assets/kali_ip.png)

### Windows 11
![imagie](/assets/window_ip.png)

## 2. Ping Test

Sending **ping** from Kali to Windows
![imagie](/assets/first_ping.png)
- Initially, the ping request failed because Windows Firewall blocked ICMP echo requests by default. 

 To resolve this issue:

![imagie](/assets/firewall.png)

1. Opened Windows Defender Firewall with Advanced Security
2. Selected Inbound Rules
3. Enabled:
    - File and Printer Sharing (Echo Request - ICMPv4-In)


### Successful Communication

After enabling the firewall rule, the ping test succeeded and confirmed communication between both VMs.

![imagie](/assets/second_ping.png)
![imagie](/assets/ping_wtk.png)



## 3. Observations & Reflections

During the lab, I noticed that the ping request from Kali Linux to Windows 11 initially failed because Windows Firewall blocked ICMP echo requests.

This raised an important question:

> In real-world environments, administrators would not intentionally open firewall rules for attackers. So how are connections actually established?

Through researching and troubleshooting this issue, I learned that firewalls do not block all communication completely. Instead, systems selectively allow specific services and ports that are required for normal operations, such as:

* HTTPS (443)
* SMB (445)
* RDP (3389)

Attackers often begin by identifying which ports and services are exposed, which is why reconnaissance tools such as Nmap are widely used in cybersecurity.

This helped me better understand:

* the role of firewalls
* attack surface exposure
* why service enumeration is important in security assessments
