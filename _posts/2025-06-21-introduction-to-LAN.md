# Introduction to LAN (Local Area Network) 

## What is a LAN?
A **Local Area Network (LAN)** connects devices within a limited area (like a home, school, or office) to communicate and share resources such as files, printers, or internet access.

---

## Common LAN Topologies

### Star Topology
- Devices are connected to a **central switch or hub**.
- **Pros**: Easy to scale, reliable structure.
- **Cons**: Expensive, central device failure brings down the network.

### Bus Topology
- Devices connect along a **single backbone cable**.
- **Pros**: Cheap and easy to set up.
- **Cons**: Prone to bottlenecks, hard to troubleshoot, no redundancy.

### Ring Topology
- Devices form a **loop**, passing data in one direction.
- **Pros**: Less network traffic congestion.
- **Cons**: One break disrupts the entire network.

---

## What is a Switch?
- A **Switch** connects devices in a LAN using Ethernet cables.
- Sends data **only to the intended device**, reducing unnecessary traffic.
- Common in **schools, businesses, and large networks**.

---

## What is a Router?
- A **Router** connects **multiple networks**, like your home LAN to the internet.
- It uses **routing** to find the best path for data.
- Can be connected with switches to create larger, more reliable networks.

---

## What is Subnetting?
**Subnetting** divides a large network into smaller, manageable subnetworks.

### Benefits:
- Improved **efficiency**, **security**, and **control**.

### Subnet Address Roles:

| Type             | Purpose                                       | Example          |
|------------------|-----------------------------------------------|------------------|
| Network Address   | Identifies the network itself                | `192.168.1.0`    |
| Host Address      | Identifies a specific device in the network  | `192.168.1.100`  |
| Default Gateway   | Routes data outside the network              | `192.168.1.254`  |

> Example: A cafe uses subnetting to separate customer Wi-Fi from internal systems (cash registers, staff devices).

---

## What is ARP (Address Resolution Protocol)?
- ARP links **IP addresses** to **MAC addresses**.
- Uses:
  - **ARP Request**: "Who has this IP?"
  - **ARP Reply**: "I do. Here’s my MAC address."
- Devices store these in the **ARP cache** for future use.

---

## IP Address Assignment with DHCP
**DHCP (Dynamic Host Configuration Protocol)** automatically assigns IP addresses to devices when they connect to the network.

### DHCP Process:
1. **Discover** – Device asks for IP
2. **Offer** – Server offers available IP
3. **Request** – Device accepts the offer
4. **Acknowledge** – Server confirms
