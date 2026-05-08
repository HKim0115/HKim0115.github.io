---
categories: [Cybersecurity HomeLab]
---

# Day 1. VM settings and Communication
---

## 1. VM Settings

### Environment
- Host OS: macOS
- Hypervisor: VirtualBox
- Kali Linux VM
- Windows 11 VM

### Kali Linux and Windows 11
- I generated 2 isolated network VMs. Kali Linux is attacker and Windows vm is end user. 
- Through this lab, I want to simulate how the attacker contact and communicate to user's computer. 
- Eventually, I'm going to make another VM for SIEM tool, so that I can monitor and analyse the attacker's behaviors.


## 2. Checking IP Addresses

### Kali Linux
![imagie](/assets/kali_ip.png)

### Windows 11
![imagie](/assets/window_ip.png)

## 3. Ping Test

Sending **ping** from Kali to Windows
![imagie](/assets/first_ping.png)
- Initially the ping failed because of windows firewall setting. 

![imagie](/assets/firewall.png)
- Configure window's firewall rule (Enable file share)
Step 1: Windows Defender Firewall with Advanced Security
Step 2: Inbound Rules
Step 3: File and Printer sharing (Echo Request - ICMPv4-In)
Step 4: Enable

![imagie](/assets/second_ping.png)
![imagie](/assets/ping_wtk.png)
- Successfull ping confirmed communication between both VMs

### 4. What I Learned

1. Basic VM network isolation (host-only network)
2. Private IP addressing
3. Basic firewall troubleshooting
4. VM-to-VM communication testing