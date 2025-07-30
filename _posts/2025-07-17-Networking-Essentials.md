---
categories: [Cyber Security 101]
---

# Networking Essentials

## Learning Objectives
Various standard protocols and technologies that glue things together:

Dynamic Host Configuration Protocol (DHCP)  
Address Resolution Protocol (ARP)  
Network Address Translation (NAT)  
Internet Control Message Protocol (ICMP)  
    Ping  
    Traceroute

### DHCP : Dynamic Host Configuration Protocol  
Whenever we want to access a network, at the very least, we need to configure the following:

    IP address along with subnet mask
    Router (or gateway)
    DNS server

  
DHCP follows four steps: Discover, Offer, Request, and Acknowledge (DORA):

DHCP Discover: The client broadcasts a DHCPDISCOVER message seeking the local DHCP server if one exists.  
DHCP Offer: The server responds with a DHCPOFFER message with an IP address available for the client to accept.  
DHCP Request: The client responds with a DHCPREQUEST message to indicate that it has accepted the offered IP.  
DHCP Acknowledge: The server responds with a DHCPACK message to confirm that the offered IP address is now assigned to this client.  

### ARP : Address Resolution Protocol

ARP Summary
1. Purpose

ARP (Address Resolution Protocol) translates an IP address (logical address) into a MAC address (physical address).

Devices on the same network (LAN) must know each other’s MAC addresses to communicate at Layer 2.

2. How It Works

Host A wants to communicate with Host B. It knows B’s IP address but not its MAC address.

Host A sends an ARP Request as a broadcast (ff:ff:ff:ff:ff:ff):

“Who has 192.168.66.1? Tell me your MAC address.”

Host B (the one with that IP) sends an ARP Reply back:

“192.168.66.1 is at 44:df:65:d8:fe:6c.”

Host A stores that MAC address and uses it in the Ethernet frame header to send data.

3. Key Points

ARP messages are encapsulated directly in Ethernet frames (not inside IP or UDP).

It’s often considered Layer 2 because it deals with MAC addresses, but it also supports Layer 3 operations.

Uses broadcast for requests and unicast for replies.

Simple Analogy
IP address = Street address of a house.

MAC address = Unique door number or nameplate on that house.

In the same apartment complex (LAN), you might know the street address (IP), but to deliver a package you need the exact door number (MAC).

ARP is like asking: “Who lives at this address? What’s your door number?” and getting a reply.

Example Packet Capture
ARP Request: Who has 192.168.66.1? Tell 192.168.66.89
ARP Reply: 192.168.66.1 is at 44:df:65:d8:fe:6c
Request → Broadcast

Reply → Unicast

In short

ARP is essential inside a LAN.

It resolves IP addresses into MAC addresses so Ethernet frames can be delivered.

Works via broadcast request → unicast reply.

### ICMP – Troubleshooting Networks
ICMP (Internet Control Message Protocol) is used for sending error messages and operational information about network communication.

Commonly used for troubleshooting with tools like ping (to test reachability) and traceroute (to trace the path packets take).

Example: If a host is unreachable, ICMP sends a “Destination Unreachable” message.

### Routing
Routing is the process of selecting the best path for network traffic to travel from the source to the destination.

Done by routers, which use routing tables to decide where to forward packets.

Can be static (manually configured) or dynamic (learned automatically via routing protocols like OSPF, BGP).

### NAT (Network Address Translation)
NAT translates private IP addresses inside a local network into a public IP address for communication over the internet.

Helps conserve IPv4 addresses and adds a layer of security by hiding internal IPs.

Common types: SNAT (Source NAT) for outgoing traffic, DNAT (Destination NAT) for incoming traffic, and PAT (Port Address Translation) for many-to-one mapping.

### Networking core protocols

1. DNS (Domain Name System)
Translates human-readable domain names (e.g., example.com) into IP addresses so computers can locate websites and services.

Works like the phonebook of the internet.

2. WHOIS
A protocol/service used to look up registration details of a domain name or IP address.

Can reveal the registrar, owner, and contact information (unless privacy-protected).

3. HTTP / HTTPS (HyperText Transfer Protocol / Secure)
HTTP: Protocol used for transferring web pages and data between a browser and a web server.

HTTPS: HTTP with encryption (TLS/SSL) to protect data in transit.

4. FTP (File Transfer Protocol)
Used for transferring files between computers over a network.

Can be plain (FTP) or secure (FTPS, SFTP).

5. SMTP (Simple Mail Transfer Protocol)
Used to send emails from a client to a mail server or between mail servers.

Works mainly for outgoing mail.

6. POP3 (Post Office Protocol v3)
Used by email clients to download messages from a mail server to a local device.

Usually removes messages from the server after downloading.

7. IMAP (Internet Message Access Protocol)
Used by email clients to read and manage emails directly on the mail server.

Keeps messages on the server, allowing access from multiple devices.

### Wireshark – Overview and Features
1. Tool Overview
Wireshark is a free and open-source network protocol analyzer.

It captures network traffic in real-time and lets you inspect packets at a very detailed level.

Commonly used for:

Troubleshooting networks (find connection issues, delays, or drops)

Analyzing protocols (see exactly how TCP, HTTP, DNS, etc., behave)

Security analysis (detect suspicious activity, investigate breaches)

Learning (see how networking protocols work in real time)

2. Packet Dissection
Wireshark shows captured packets in three main panes:

Packet List Pane – A summary of all captured packets (time, source, destination, protocol, info).

Packet Details Pane – A tree-like breakdown of the selected packet (Ethernet, IP, TCP/UDP, application data).

Packet Bytes Pane – The raw data of the packet in hexadecimal and ASCII.

You can expand each protocol layer to see headers, flags, fields, and values.

3. Packet Navigation
You can scroll through all captured packets in the Packet List Pane.

Click on a packet to see its details and raw data.

Follow TCP/UDP Streams – Lets you view the entire conversation between two hosts.

Color Coding – Wireshark highlights different protocols in different colors to help you quickly identify traffic types.

4. Packet Filtering
Capture Filters – Set before capturing to limit what packets are recorded (e.g., host 192.168.1.10).

Display Filters – Apply after capturing to show only specific packets (e.g., http, ip.addr == 192.168.1.5, tcp.port == 80).

Display filters are powerful and can be combined with logic operators (and, or, not).

Example:

http → Show only HTTP packets

ip.src == 192.168.0.10 → Show packets sent from that IP

tcp.port == 443 → Show only HTTPS packets

📌 In short:

Wireshark lets you capture and analyze network packets in detail.

Packet dissection breaks packets into layers (Ethernet → IP → TCP/UDP → Application).

Navigation helps you follow streams and view conversations.

Filtering allows you to focus only on relevant packets.

### Tcpdump – Overview & Usage
1. Basic Packet Capture
Tcpdump is a command-line packet capture tool for Linux/Unix systems.

Captures network packets in real time and displays them in text format.

-Basic usage:
tcpdump

-Capture on a specific interface:
tcpdump -i eth0

-Save capture to a file (for later analysis in Wireshark):
tcpdump -i eth0 -w capture.pcap

2. Filtering Expressions
Tcpdump uses BPF (Berkeley Packet Filter) syntax to filter traffic.

-Common examples:

tcpdump host 192.168.1.10        # Traffic to/from a specific host
tcpdump src host 192.168.1.10    # Traffic from a specific source IP
tcpdump dst host 192.168.1.10    # Traffic to a specific destination IP
tcpdump port 80                  # Traffic on port 80 (HTTP)
tcpdump tcp                      # Only TCP packets
tcpdump udp                      # Only UDP packets

3. Advanced Filtering
-Combine filters with logical operators:

tcpdump host 192.168.1.10 and port 80     # HTTP traffic to/from host
tcpdump tcp and portrange 20-80           # TCP traffic in port range
tcpdump src net 192.168.0.0/24            # Traffic from a subnet
tcpdump 'tcp[tcpflags] & tcp-syn != 0'    # Only TCP SYN packets

-Negation:

tcpdump not port 22     # All traffic except SSH

4. Displaying Packets
-Show packets with timestamps:

tcpdump -tt

-Show verbose details:

tcpdump -v       # Verbose
tcpdump -vv      # More verbose
tcpdump -vvv     # Very verbose

-Show packet contents in ASCII/hex:

tcpdump -A       # ASCII
tcpdump -XX      # Hex + ASCII

-Read from a saved capture file:

tcpdump -r capture.pcap

📌 In short:

Tcpdump is lightweight and perfect for quick packet captures.

Filtering expressions keep captures focused and efficient.

Advanced filtering uses logical operators and field matching.

Displaying packets can be adjusted for summary or detailed analysis.

### Nmap – Overview
Nmap (Network Mapper) is a free, open-source tool for:

Host discovery – finding devices on a network

Port scanning – identifying open ports

Service/version detection – learning what services are running

OS detection – guessing the operating system

Security auditing – finding vulnerabilities

1. Host Discovery – "Who is Online"
-Find devices currently connected to a network.

nmap -sn 192.168.1.0/24

-sn: Ping scan (no port scanning).

Uses ICMP, TCP, or ARP to detect live hosts.

Output shows only active devices.

2. Port Scanning – "Who is Listening"
-Check which ports are open (listening for connections).

nmap 192.168.1.10

Default scans 1–1000 TCP ports.

Common types:

TCP SYN Scan (-sS): Stealthy and fast.

TCP Connect Scan (-sT): Full TCP connection.

UDP Scan (-sU): For UDP services.
-Example:

nmap -p 22,80,443 192.168.1.10
(Scan only ports 22, 80, 443.)

3. Version Detection – "Extract More Information"
-Identify application name and version running on an open port.

nmap -sV 192.168.1.10

Example output:

80/tcp open  http    Apache httpd 2.4.41
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu

4. Timing – "How Fast is Fast"
-Adjust scan speed.

nmap -T0   # Paranoid (very slow)
nmap -T3   # Normal (default)
nmap -T4   # Aggressive (fast, may be detected)
nmap -T5   # Insane (very fast, may cause packet loss)

Higher timing = faster scan but more chance of being blocked or missing packets.

5. Output – "Controlling What You See"
-Choose how results are displayed or saved:

nmap -oN result.txt 192.168.1.10   # Normal format
nmap -oG result.gnmap 192.168.1.10 # Greppable format
nmap -oX result.xml 192.168.1.10   # XML format

-v = Verbose (more details in terminal).

-vv = Very verbose.

-d = Debug mode.

📌 Summary Table

|Task	|Command Example	|Purpose|
|------|----------------|-----------|
|Host Discovery	|nmap -sn 192.168.1.0/24	|Find who’s online|
|Port Scanning	|nmap -p 80,443 192.168.1.10	|Find open ports|
|Version Detection	|nmap -sV 192.168.1.10	|Identify services & versions|
|Timing	|nmap -T4 192.168.1.10	|Control scan speed|
|Output Control	|nmap -oN scan.txt 192.168.1.10	|Save or format results|

