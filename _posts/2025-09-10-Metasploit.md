---
categories: [Cyber Security 101]
---

# Metasploit Study Notes
---

## 1. Core Concepts of Metasploit

### msfconsole
- `msfconsole` is the main interactive console for the Metasploit Framework. Use it to search, configure, and run modules.

### Modules
Modules are small components that perform a specific function. Main categories:

- **Exploit**: Code that takes advantage of a vulnerability to achieve a goal (such as remote code execution).
- **Auxiliary**: Tools for scanning, enumeration, fuzzing, and other non-exploit tasks (e.g., port scanning, service/version detection).
- **Payload**: Code that runs on the target after a successful exploit (e.g., reverse shells, Meterpreter).
- **Post**: Modules used after session establishment (post-exploitation tasks like data collection, privilege escalation, persistence).

### Vulnerability, Exploit, Payload — quick definitions
- **Vulnerability**: A flaw in design, code, or logic that may allow confidentiality, integrity, or availability breaches.
- **Exploit**: A technique or code that abuses a vulnerability to produce an effect (e.g., code execution).
- **Payload**: The action executed on the target after exploitation (what the attacker wants the exploit to do).

### Common Options / Parameters
- `RHOSTS` — Remote host(s) IP address or range.
- `RPORT` — Remote port where the target service listens.
- `LHOST` — Local host (attacker) IP for reverse connections.
- `LPORT` — Local port for reverse connections.
- `PAYLOAD` — Payload name to use with an exploit.
- `SESSION` — Session ID for post-exploitation modules.


---

## 2. Common Commands (Reference)
> The following list is educational. It shows common Metasploit operations but does not give values for attacking real targets.

- `msfconsole` — Start the Metasploit console.
- `search <keyword>` — Search available modules (e.g., `search portscan`).
- `use <module/path>` — Select a module (e.g., `use auxiliary/scanner/smb/smb_version`).
- `set <option> <value>` — Set module options (e.g., `set RHOSTS 10.0.0.5`).
- `show options` — Show configurable options for the selected module.
- `run` / `exploit` — Execute the currently selected module.
- `msfvenom --list formats` — List supported payload output formats.
- `msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> > payload.exe` — Generate an executable payload file.
- `use exploit/multi/handler` — Start a handler to catch incoming reverse connections.
- `hashdump` — Dump password hashes from a compromised session (requires an active, authorized session).


---

## 3. SMB / MS17-010 (EternalBlue) — Conceptual Overview
> This section explains the background and conceptual workflow for studying MS17-010. It does **not** provide exploit commands or parameter values for attacking systems.

### Background
- MS17-010 is a critical remote code execution vulnerability in Microsoft SMB (Server Message Block) that affected SMBv1 handling. It was widely exploited in the wild and used by malware such as WannaCry.

### Conceptual Attack Workflow (Learning Focus)
1. **Scoping** — Define your practice environment. Use isolated VMs and never scan or attack external systems without permission.
2. **Discovery & Enumeration** — Determine whether SMB is available and which SMB version is active. Collect service banners and version info.
3. **Vulnerability Verification** — Safely check whether the target is vulnerable to MS17-010 using passive or non-destructive detection techniques.
4. **Exploit Selection (Conceptual)** — Understand the exploit module's purpose, its required options, and how it works. Emphasize learning the code flow rather than running it against unauthorized targets.
5. **Payload Consideration** — Choose a benign payload for testing (e.g., non-destructive checks) or a controlled Meterpreter session in a lab environment.
6. **Post-exploitation** — After obtaining a session, use post modules for data collection, privilege checks, or cleanup — but always within authorized scope.


---

## 4. Common Module Types (Examples)
- `auxiliary/scanner/*` — Scanning and enumeration modules (e.g., `auxiliary/scanner/smb/smb_version`, `auxiliary/scanner/smb/smb_enumshares`).
- `exploit/windows/smb/*` — Exploit modules targeting SMB-related vulnerabilities.
- `post/windows/gather/*` — Post modules to collect information from a compromised Windows host.

