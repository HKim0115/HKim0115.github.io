# concept of Defensive security and network 

Basic concept of defensive security is :
1. Preventing intrusions from occurring
2. Detecting intrusions when they occur and responding properly

some task of defensive security include :
- User cyber security awareness: Training users about cyber security helps protect against attacks targeting their systems.
- Documenting and managing assets: We need to know the systems and devices we must manage and protect adequately.
- Updating and patching systems: Ensuring that computers, servers, and network devices are correctly updated and patched against any known vulnerability (weakness).
- Setting up preventative security devices: firewall and intrusion prevention systems (IPS) are critical components of preventative security. Firewalls control what network traffic can go inside and what can leave the system or network. IPS blocks any network traffic that matches present rules and attack signatures.
- Setting up logging and monitoring devices: Proper network logging and monitoring are essential for detecting malicious activities and intrusions. If a new unauthorized device appears on our network, we should be able to detect it.

# What is network?

networks are simply things connected.
It can be anything from your computer to your phone, traffic light and any technological devices. 

# What is Internet?

The Internet is one giant network that consists of many, many small networks within itself.
![imagie](/assets/internet-diagram.png)

## Understanding Private and Public IP Addresses
Q1: Why do two devices have the same public IP but different private IPs?

A: They are connected to the same router (Wi-Fi network). The router has one public IP, and each device connected to it gets a unique private IP inside the local network.

Q2: Does each device have both a public and a private IP address?

A: Not exactly. Each device has only a private IP. The public IP belongs to the router, which communicates with the internet on behalf of all devices.

Q3: If two people use the same internet company, can their public IPs be the same?

A: Normally, no — each router gets its own public IP from the ISP.
However, with CGNAT (Carrier-Grade NAT), multiple users may share the same public IP.

Q4: Do public IP addresses change over time?

A: Yes, if the ISP assigns a dynamic IP, it may change periodically or after restarting the router. A static IP stays the same but usually requires a paid plan.

Q5: When devices in the same network communicate, do they use private IPs?

A: Yes! Devices in the same local network use private IPs to communicate.
However, the data still goes through the router, which handles and directs the traffic inside the network.

# What is MAC address?

MAC address stands for Media Access Control
It's a physical indentifying number of the devices but it can be faked or 'spoofed'.

