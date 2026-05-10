---
categories: [HomeLab]
title: "Day 2. Nmap Scanning and SMB Enumeration"
---

### 1. Lab Overview

* Learn basic reconnaissance using Nmap
* Identify exposed services on a Windows VM
* Explore SMB enumeration and authenticated access
* Test file transfer between Kali Linux and Windows

---

### 2. Initial Nmap Scan

I started by scanning the Windows VM from Kali Linux using Nmap.

Command:

nmap 192.168.56.2


Initial scans showed all ports as filtered even though ping communication between the VMs worked successfully.

![imagie](/assets/nmap.png)
![imagie](/assets/nmap-p445.png)

---

### 3. Troubleshooting

After investigating the issue, I found that the VirtualBox Host-only network was being treated as a Public network by Windows 11, which restricted SMB exposure and inbound connections.

I temporarily changed the network profile to Private using PowerShell and confirmed that the SMB service became visible through Nmap scanning.

![imagie](/assets/adminpowershell.png)
![imagie](/assets/nmapopen.png)

---

### 4. Discovering SMB Service

After adjusting the network profile, I rescanned port 445.

Command:

nmap -p 445 192.168.56.2

Result: 

445/tcp open microsoft-ds


This confirmed that the Windows SMB service was exposed and reachable from the Kali VM.

---

### 5. SMB Enumeration

Next, I used smbclient from Kali Linux to enumerate SMB shares on the Windows VM.

Command:

smbclient -L //192.168.56.2/ -U hkim

After authentication, I was able to identify available SMB shares including:

* ADMIN$
* C$
* IPC$
* LabShare (created for this Lab)

![imagie](/assets/smbclientu.png)

---

### 6. SMB Share Interaction

I connected to the shared folder and tested file transfer between Kali Linux and Windows.

Command:

smbclient //192.168.56.2/LabShare -U hkim


Inside the SMB session, I tested:

#### ls 
   (Checking what files are in the LabShare folder)

#### get secret.txt
   (Download secret.txt file from Windows to Kali Linux)

#### put hello.txt
   (Share hello.txt file from Kali Linux to Windows)


This confirmed successful authenticated SMB interaction and file transfer between both VMs.

![imagie](/assets/smbget.png)
![imagie](/assets/smbput.png)

---

### 7. Observations & Reflections

#### Defender Perspective

During the lab, I observed that Windows 11 initially filtered SMB traffic even though the SMB service was running internally. From a defensive perspective, this behavior is important because reducing exposed services helps minimize the attack surface and prevents unauthorized enumeration.

This lab also demonstrated how layered security controls work together. Even after the SMB service became discoverable through Nmap scanning, authentication was still required before accessing shared resources. This highlighted the role of:

* firewalls
* network trust profiles
* authentication mechanisms

in protecting systems from unauthorized access.

In addition, this exercise made me think more about credential security in real environments. I have often seen workplaces leave usernames and passwords near shared workstations for convenience during shift changes. However, this lab helped me better understand how weak credential handling practices can undermine otherwise secure systems.

Overall, this reinforced the idea that cybersecurity depends not only on technical controls, but also on operational security and user behavior.

---


