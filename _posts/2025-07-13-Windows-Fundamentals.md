---
categories: [Cyber Security 101]
---

# Windows Fundamentals 

## 1. File Systems in Windows
NTFS (New Technology File System) is the modern default file system.

Supports files > 4GB, file/folder permissions, compression, encryption (EFS), and journaling.

Journaling helps restore data after failures, unlike FAT16/FAT32.

Alternate Data Streams (ADS) allow hiding extra data in files; often abused by malware but also used by Windows (e.g., tagging downloaded files).

Checking File System: Right-click drive > Properties to check if NTFS is used.

## 2. File/Folder Permissions
NTFS allows setting granular permissions: Full Control, Modify, Read, Write, etc.

View/edit permissions: Right-click file/folder > Properties > Security tab.

## 3. System Environment
%windir% points to the Windows folder (e.g., C:\Windows).

The System32 folder stores critical OS files; altering it may break the system.

## 4. User Accounts
Two types: Administrator (full system access) and Standard User (limited).

Profiles are created under C:\Users\ upon first login.

Use lusrmgr.msc to manage local users and groups.

## 5. User Account Control (UAC)
Prevents automatic elevation of privileges.

Admins get a prompt when an action requires elevation.

UAC doesn’t apply to the built-in Administrator account by default.

## 6. Control Panel vs Settings
Control Panel offers deep configuration tools.

Settings Menu (since Windows 8) is more user-friendly and touch-oriented.

Some settings (e.g., network adapter options) still redirect to Control Panel.

## 7. Task Manager
Use to monitor CPU, RAM, disk, and apps.

Access by right-clicking the taskbar or pressing Ctrl+Shift+Esc.

## 8. System Configuration (msconfig)
Diagnose startup issues.

Tabs: General, Boot, Services, Startup (now managed by Task Manager), Tools.

## 9. Computer Management
Includes:

Task Scheduler: Automates tasks.

Event Viewer: Logs and monitors system activity.

Shared Folders: Shows active shares, sessions, and open files.

Local Users and Groups

Performance Monitor (perfmon): Analyzes system performance.

Device Manager: Manages hardware devices.

## 10. Disk Management
Used to manage partitions, assign drive letters, and prepare drives.

## 11. WMI (Windows Management Instrumentation)
Allows remote/local scripting and management of Windows systems.

WMIC is deprecated; use PowerShell for WMI operations.

## 12. System Information (msinfo32)
Provides detailed system specs, hardware, and environment variables.

Components tab includes details like display and network info.

## 13. Resource Monitor (resmon)
Real-time monitoring of CPU, Memory, Disk, and Network usage.

Useful for identifying deadlocks and resource conflicts.

## 14. Command Line Tools
hostname, whoami, ipconfig, netstat, net help

Helpful for networking and user identification.

cls clears the screen.

## 15. Windows Registry (regedit)
Hierarchical database used for user profiles, installed programs, system configs.

Dangerous to modify unless you're confident.

## 16. Windows Update
Provides security patches and feature updates.

Can be triggered manually via control /name Microsoft.WindowsUpdate.

## 17. Windows Security
Virus & Threat Protection: Includes scans, quarantine, exclusions.

Firewall & Network Protection: Manages access by domain, private, or public networks.

SmartScreen: Blocks suspicious apps and sites.

Exploit Protection & Core Isolation: Additional layers against malware.

BitLocker: Encrypts drives using TPM (Trusted Platform Module).

Volume Shadow Copy Service (VSS): Enables restore points and backup snapshots.

# Introduction to Active Directory (AD)

## 1. Windows Domains & AD DS
A Windows domain groups users and computers under centralized management.

The Domain Controller (DC) runs Active Directory Domain Services (AD DS).

Benefits:

Centralized identity and access management.

Group-wide policy control.

## 2. AD Object Types
Users: Security principals representing people or services.

Machines: Every computer in the domain is an object (e.g., PC01$).

Groups: Users/computers can be grouped for shared permissions (e.g., Domain Admins).

## 3. Default AD Groups
Domain Admins: Full control of the domain.

Server Operators, Backup Operators, Account Operators: Limited, specific administrative rights.

Domain Users/Computers/Controllers: Include all respective objects.

## 4. Organizational Units (OUs)
Logical containers used to group users, computers, and other objects.

Commonly structured by departments (e.g., IT, HR).

Policies are applied at the OU level.

## 5. AD Users and Computers Tool
Used to view, manage, and modify domain objects.

Accessible via Start Menu > Active Directory Users and Computers.

Supports tasks like creating users, resetting passwords, organizing OUs.

