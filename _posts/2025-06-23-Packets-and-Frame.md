---
categories: [Pre Security]
---

#  Understanding Ports, TCP, and UDP in Networking

##  What Are Ports?

In networking, a **port** is a logical access point through which data is sent and received.  
Just like a port in a harbor allows ships to dock, network ports allow data to reach the correct service or application on a device.

- Port numbers range from **0 to 65535**, based on the **16-bit** address space (2¹⁶ = 65536).
- Ports help organize and direct traffic to the appropriate application or protocol.

###  Port Ranges

| Range          | Name               | Usage                             |
|----------------|--------------------|------------------------------------|
| 0 - 1023       | Well-known ports   | Common protocols (HTTP, HTTPS...) |
| 1024 - 49151   | Registered ports   | User-registered applications       |
| 49152 - 65535  | Dynamic/private    | Temporary or custom usage         |

---

##  Common Protocols and Their Ports

| Protocol | Port | Description |
|----------|------|-------------|
| FTP      | 21   | File transfer between client and server |
| SSH      | 22   | Secure text-based remote login |
| HTTP     | 80   | Basic web traffic |
| HTTPS    | 443  | Secure encrypted web traffic |
| SMB      | 445  | File and printer sharing on local network |
| RDP      | 3389 | Remote desktop access |

Note: Applications **can use non-standard ports**, such as running a web server on port 8080 instead of 80. In that case, a URL must specify the port (e.g., `http://localhost:8080`).

---

##  My Question

###  Do websites use port 21 for downloads?

- **No.** Most websites use **port 80 (HTTP)** or **443 (HTTPS)** for everything — including file downloads.
- **Port 21 (FTP)** is only used when downloading from a dedicated FTP server.

###  Can a device use multiple ports at once?

- **Yes!** A single device can open **many ports simultaneously**.
- Each connection uses a unique combination of:


For example:
- YouTube on tab 1 (port 443)
- GitHub on tab 2 (port 443)
- Each uses a different **source port** on your device (e.g., 54321, 54322)

---

##  TCP vs UDP – What's the Difference?

###  TCP (Transmission Control Protocol)

- **Connection-oriented**
- Establishes a connection before data is sent (3-way handshake)
- **Reliable**: Guarantees delivery and order of packets
- Retransmits lost packets
- **Slower** but secure and accurate
- Used in: web (HTTP/HTTPS), email (SMTP/POP3), file transfer (FTP)

###  UDP (User Datagram Protocol)

- **Connectionless**
- Sends data without checking if it arrives
- **Unreliable**: No delivery or order guarantees
- No retransmission
- **Faster**, lightweight
- Used in: streaming (YouTube, VoIP), online games, DNS

###  Comparison Table

| Feature         | TCP                            | UDP                            |
|----------------|---------------------------------|--------------------------------|
| Connection      | Yes                            | No                             |
| Reliability     | High (guaranteed delivery)      | Low (no guarantee)             |
| Speed           | Slower                          | Faster                         |
| Packet Order    | Maintains order                 | No order guarantee             |
| Retransmission  | Yes                             | No                             |
| Use Cases       | Web, email, file transfers      | Streaming, gaming, VoIP, DNS   |

###  Real-Life Analogy

- **TCP = Registered Mail** 
→ Confirmed delivery, tracking, and retry if lost.

- **UDP = Postcard**   
→ Sent quickly, but may be lost or out of order.

---

##  Summary

- **Ports** help direct traffic to the right service on a device.
- **Devices can use many ports at the same time**, especially in modern multitasking.
- Websites typically use **port 80 (HTTP)** or **443 (HTTPS)**, not FTP unless explicitly set up.
- **TCP** is reliable and ordered but slower, ideal for critical communications.
- **UDP** is faster but unreliable, ideal for real-time data where speed matters more than accuracy.
