---
categories: [Pre Security]
---

# Basic Linux Commands & Concepts Summary

## File Editing & Download Tools

### `nano`
A simple text editor used in the terminal.  
Usage:
nano filename.txt
Save: Ctrl + O
Exit: Ctrl + X

### wget
Command-line tool to download files from the internet.
Usage:
wget http://example.com/file.txt

### scp (Secure Copy)
Used to securely copy files between machines over SSH.
Usage:
scp file.txt user@host:/destination/path

### python3 -m http.server
Starts a simple HTTP server to serve files in the current directory.
Default port: 8000
Usage:
python3 -m http.server

### Downloading a File from a Web Server
Use wget or curl:
wget http://server/file.txt
curl -O http://server/file.txt

### Process & System Monitoring
-ps
Displays currently running processes for the current user.

-ps aux
Shows all running processes with detailed information.

-top
Real-time view of system resources (CPU, memory, etc.).

-kill
Terminates a process by its PID (Process ID).
Usage:
kill PID

-Ctrl + Z
Pauses a running process and sends it to the background.

-fg
Brings the most recent backgrounded process to the foreground.

### Background vs Foreground
Foreground: A process running directly in the terminal.

Background: A process running detached from the terminal (e.g., with & or Ctrl + Z).

### Services & Scheduling
-systemd
The modern Linux system and service manager (init system).

-systemctl
Used to manage systemd services.
Examples:
systemctl status apache2
systemctl start nginx
systemctl enable ssh

-cron & crontab
Used for scheduling automatic tasks.
Edit user cron jobs:
crontab -e

Example entry (runs a script daily at 9 AM):
0 9 * * * /home/user/script.sh

### Package Management
-apt-repository
Software sources used by APT.

Add a repository:
sudo add-apt-repository ppa:some/ppa

-dpkg
Installs or removes .deb packages.
Examples:
sudo dpkg -i package.deb
sudo dpkg -r package-name

-apache2
A widely-used HTTP server (web server).
Common commands:
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2


