---
title: "Day 13 — Active Directory Deployment & Domain Integration"
categories: [HomeLab]
---

## Overview

The lab so far was a single-hop structure: Kali (attacker) → Win11 (target). This structure cannot accurately demonstrate "Lateral Movement" as a technique, since there is no second internal system to move to. To fix this, a Windows Server VM was added and configured as an Active Directory Domain Controller.

---

## Steps

- **Installed the AD DS role, then promoted the server to a Domain Controller**
  Created a new forest, `homelab.local`. Confirmed that installing the AD DS role and actually promoting the server to a DC are two separate steps in Server Manager.

  ![AD DS role installed](/assets/images/HomeLab/day13/1ADinstall.png)

- **Created two domain user accounts**
  - `Elkin E. Jose` (`elkin`) — standard user account
  - `Sid Swar` — added to Domain Admins

  ![Creating a user account](/assets/images/HomeLab/day13/2user.png)
  ![Adding Sid Swar to Domain Admins](/assets/images/HomeLab/day13/3sidadmin.png)

- **Pointed Win11's DNS to the DC (192.168.56.10)**
  Required beforehand, since AD relies on DNS to locate the domain.

  ![Win11 DNS configuration](/assets/images/HomeLab/day13/4winjoin.png)

- **Joined Win11 to the homelab.local domain**

  ![Domain join successful](/assets/images/HomeLab/day13/5winjoin.png)
  ![Domain join completed](/assets/images/HomeLab/day13/6domainjoin.png)

- **Tested login with a domain account**

  ![Logging in as a domain user via "Other user"](/assets/images/HomeLab/day13/7login.png)
  ![Confirming the domain account with whoami](/assets/images/HomeLab/day13/8elkin.png)

- **Verified Kerberos authentication logs on the DC**
  Each time a login occurred from Win11, the DC's Security log recorded a matching set of events: 4768 (TGT issued), 4769 (service ticket issued), and 4624 (logon success).

  ![Confirming elkin's logon event in DC Event Viewer](/assets/images/HomeLab/day13/9elkinlog.png)

- **Installed Splunk Universal Forwarder on the DC, configured Security Log forwarding via inputs.conf**

  ![Splunk Forwarder service running correctly](/assets/images/HomeLab/day13/10splunk.png)

- **Confirmed the DC's logs were arriving in Splunk**

  ![Search results showing KIMDC logs in Splunk](/assets/images/HomeLab/day13/11splunkcheck.png)

---

## What Went Wrong

Searching `index=wineventlog host=KIMDC*` in Splunk Web returned 0 results across every short time range tried (15 minutes, 60 minutes, 24 hours). The cause was narrowed down step by step:

1. **Checked whether the Splunk Forwarder was actually running** — `Get-Service SplunkForwarder` showed `Running`. 

2. **Checked whether `inputs.conf` was actually being applied** — Opening the file revealed that it was saved as `inputs.conf.txt`. Fixed the extension and restarted the service.

3. **Still no results, so checked whether Win11's own logs were still arriving normally** — searching `index=wineventlog` with no host filter showed existing Win11 data just fine. This ruled out a problem with Splunk or the index itself.

4. **Widened the time range to All time** — DC logs were actually present. However, the timestamps were two days behind the current search time.

5. **Checked the DC's system clock** — the DC's time zone had been changed earlier, but the change had not yet taken effect. The fix only applied after a reboot, which is why the clock still showed the old, unsynced time until then.

6. **Rebooted the DC** — after rebooting, the time zone change took effect and the clock matched the host. A search for Last 15 minutes immediately afterward showed DC logs arriving normally.

---

## Q&A

**Q. Why add Active Directory at this point?**

The previous structure (Kali → Win11) could only demonstrate an external attacker reaching a single target, which falls under Initial Access or Credential Access. Demonstrating real Lateral Movement — moving from an already-compromised internal host to a different internal system — requires a second system to move to. Adding AD/a DC is what makes that structure possible.

**Q. What is Kerberos authentication?**

It's the default authentication protocol in a Windows domain environment. When a user logs in, they first prove their identity to the DC and receive a TGT (Ticket Granting Ticket, Event ID 4768). From then on, every time they need to access a resource, that TGT is used to request a service ticket (Event ID 4769) instead of resending a password each time.

**Q. How does this differ from a local logon?**

In a local logon (workgroup environment), the machine itself verifies the credentials and that's the end of it. In a domain logon, the verification is actually performed by the DC, and the machine simply trusts that result. This is why domain logons leave logs in two places: the machine where the login was attempted (Win11) records the logon event, while the DC that actually verified the identity records both the Kerberos authentication events (4768/4769) and a matching logon event (4624).