---
categories: [Cyber Security 101]
---

# Windows Command Line

Learning Objectives
•Display basic system information
•Check and troubleshoot network configuration
•Manage files and folders
•Check running processes

## Key Commands
|Command|Description|Example|
|----------|----------|----------|
|ver|	Display Windows version	|ver
|set|	View or set environment variables	|set PATH
|systeminfo|	Display detailed system information	|systeminfo
|driverquery|	List installed drivers	|driverquery
|more|	View long output one screen at a time	| "type file.txt | more"
|help|	Show help for commands	|help copy
|cls|	Clear the screen	|cls
|ipconfig|	Show network configuration	|ipconfig
|ping|	Test network connection	|ping google.com
|tracert|	Trace route to a host	|tracert google.com
|nslookup|	Query DNS information	|nslookup google.com
|netstat|	Display network connections	|netstat -an
|cd|	Change directory	|cd C:\Users
|dir|	List directory contents	|dir
|tree|	Display folder structure	|tree /F
|mkdir|	Create a new folder	mkdir| newFolder
|type|	Show file contents	type |notes.txt
|copy|	Copy files	|copy a.txt b.txt
|move|	Move files	|move a.txt D:\
|del / erase|	Delete files	|del test.txt
|*	|Wildcard for multiple files	|del *.txt
|tasklist|	List running processes	|tasklist
|/FI|	Filter results	tasklist |/FI "IMAGENAME eq chrome.exe"
|chkdsk|	Check disk for errors	|chkdsk C:
|sfc /scannow|	Scan system files for integrity	|sfc /scannow

### PowerShell

Learning Objectives
•Understand what PowerShell is and its capabilities
•Learn the basic structure of PowerShell language
•Run basic PowerShell commands
•Understand PowerShell's role in cybersecurity


Common Cmdlets

|Cmdlet|	Description|	Example
|----------|----------|----------|
|Get-Content|	Read file content|	Get-Content file.txt|
|Set-Location|	Change directory|	Set-Location C:\Users|
|Get-Command|	List available commands|	Get-Command|
|CommandType|	View command type|	Get-Command ls | Select-Object CommandType|
|Get-Help|	Show help for cmdlets|	Get-Help Get-Content|
|Get-Date|	Display current date/time|	Get-Date|
|Get-Alias|	View command aliases|	Get-Alias|
|Get-ChildItem|	List directory contents|	Get-ChildItem|
|Find-Module|	Search for modules|	Find-Module -Name Az|
|Install-Module|	Install a module|	Install-Module Az|
|New-Item|	Create a file or folder|	New-Item -Path . -Name file.txt -ItemType File|
|Remove-Item|	Delete a file or folder|	Remove-Item file.txt|
|Copy-Item|	Copy files/folders|	Copy-Item a.txt b.txt|
|Move-Item|	Move files/folders|	Move-Item a.txt D:\|

Piping, Filtering, and Sorting
-Pipe output to another command

|Cmdlet|	Description|	Example|
|----------|----------|----------|
|Sort-Object|	Sort objects|	Sort-Object Name|
|Where-Object|	Filter objects based on condition|	Where-Object {$_.CPU -gt 100}|
|-eq, -ne, -gt, etc.|	Comparison operators	| |
|Select-Object|	Choose specific properties|	Select-Object Name, ID|
|Select-String|	Search for text in a file|	Select-String "error" file.txt|
|Get-ComputerInfo|	System details| |	
|Get-LocalUser|	View local users	| |
|Get-NetIPConfiguration|	Network settings| |	
|Get-Process|	Show running processes| |	
|Get-Service|	Show services| |	
|Get-NetTCPConnection|	View TCP connections|	|
|Get-FileHash|	Get file hash| |
|Invoke-Command|	Run command remotely|	|
|ScriptBlock|	Code block for remote execution|	|

### Linux Shell

Learning Objectives
•Interact with Linux shell
•Use basic shell commands
•Explore different types of shells (bash, zsh, etc.)
•Write basic shell scripts

Common Commands
|Command|	Description|	Example|
|----------|----------|----------|
|pwd|	Show current directory|	pwd|
|cd|	Change directory|	cd /home/user|
|ls|	List directory contents|	ls -l|
|cat|	Display file content|	cat file.txt|
|grep|	Search for patterns|	grep "error" log.txt|
|echo|	Print text|	echo Hello|
|/etc/shells|	List available shells|	cat /etc/shells|
|bash, zsh, fish|	Types of shells|	|
|nano|	Command-line text editor|	nano file.txt|
|#! (shebang)|	Declare script interpreter|	#!/bin/bash|
|read|	Accept user input|	read name|
|chmod +x|	Make script executable|	chmod +x script.sh|
|loops|	Control structure for repetition|	|
|sudo, su|	Superuser or admin privileges|	sudo apt update|

