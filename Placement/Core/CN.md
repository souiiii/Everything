# Computer Networks Interview Notes

### Interview Preparation

These notes focus on the concepts that are commonly tested in entry-level technical interviews. The aim is to help you understand the ideas clearly and explain them naturally.

---

## 1. What Is a Computer Network?

A computer network is a collection of devices connected to exchange data and share resources such as files, printers, applications, and internet access.

### Types of Networks

| Network | Coverage |
| --- | --- |
| LAN | Small area such as an office, building, or college |
| MAN | City-sized geographical area |
| WAN | Large geographical area, often across countries |
| PAN | Personal devices within a short distance |

The internet is the largest example of a WAN.

### Common interview question

**What is the difference between the internet and the web?**

The internet is the underlying network connecting devices worldwide. The World Wide Web is a service that runs on the internet and allows users to access webpages through browsers.

---

# 2. OSI Model

The OSI model divides network communication into seven layers. Each layer performs a specific responsibility.

| Layer | Main Responsibility | Examples |
| --- | --- | --- |
| 7. Application | Provides network services to applications | HTTP, DNS, SMTP |
| 6. Presentation | Data format, encryption and compression | SSL/TLS, JPEG |
| 5. Session | Creates and manages communication sessions | Session control |
| 4. Transport | End-to-end delivery and reliability | TCP, UDP |
| 3. Network | Routing and logical addressing | IP, routers |
| 2. Data Link | Frame delivery and MAC addressing | Ethernet, switches |
| 1. Physical | Transmission of raw bits | Cables, radio signals |

### Easy way to understand the flow

When sending data:

```
Application → Transport → Network → Data Link → Physical
```

At the receiver, the process happens in reverse.

### Encapsulation

Each layer adds its own control information to the data.

```
Application data
↓
TCP/UDP segment
↓
IP packet
↓
Ethernet frame
↓
Bits
```

### Common interview questions

**Why do we use layered network models?**

Layering separates responsibilities. A change in one layer can be made without redesigning the entire network system.

**At which layer does a router work?**

A router primarily works at the Network Layer because it forwards packets using IP addresses.

**At which layer does a switch work?**

A normal Ethernet switch works at the Data Link Layer and forwards frames using MAC addresses.

---

# 3. TCP/IP Model

The TCP/IP model is the practical networking model used by the internet.

| TCP/IP Layer | Corresponding OSI Layers |
| --- | --- |
| Application | Application, Presentation and Session |
| Transport | Transport |
| Internet | Network |
| Network Access | Data Link and Physical |

### OSI vs TCP/IP

The OSI model is mainly a conceptual reference model. TCP/IP is the protocol suite actually used for internet communication.

---

# 4. Network Devices

## Hub

A hub sends incoming data to every connected device.

It does not understand MAC addresses and creates unnecessary network traffic.

## Switch

A switch forwards a frame only to the appropriate device using its MAC address table.

Each switch port creates a separate collision domain.

## Router

A router connects different networks and forwards packets using IP addresses.

It selects a path using its routing table.

## Modem

A modem converts signals so that a local network can communicate through an internet service provider.

## Access Point

A wireless access point allows Wi-Fi devices to connect to a wired network.

### Hub vs Switch vs Router

| Device | Uses | Main Purpose |
| --- | --- | --- |
| Hub | No address intelligence | Broadcast data |
| Switch | MAC address | Connect devices within a LAN |
| Router | IP address | Connect different networks |

### Common interview question

**Can a switch connect two different IP networks?**

A normal Layer 2 switch cannot route traffic between different IP networks. A router or Layer 3 switch is required.

---

# 5. MAC Address and IP Address

## MAC Address

A MAC address identifies a network interface within a local network.

Example:

```
00:1A:2B:3C:4D:5E
```

It is mainly used at the Data Link Layer.

## IP Address

An IP address identifies a device logically across networks.

Example:

```
192.168.1.10
```

It is used at the Network Layer.

### Difference

| MAC Address | IP Address |
| --- | --- |
| Used within the local network | Used across different networks |
| Data Link Layer | Network Layer |
| Identifies a network interface | Identifies the device’s network location |
| Used by switches | Used by routers |

### Interview-ready explanation

The IP address helps a packet reach the correct network. The MAC address helps deliver the frame to the correct device within the local network.

---

# 6. IPv4 Addressing

An IPv4 address contains 32 bits and is normally written as four decimal numbers.

```
192.168.1.20
```

Each part contains 8 bits and ranges from 0 to 255.

## Network and Host Portions

An IP address contains:

- A network portion that identifies the network
- A host portion that identifies the device within that network

The subnet mask determines which bits belong to each portion.

Example:

```
IP address:   192.168.1.20
Subnet mask:  255.255.255.0
```

The network address is:

```
192.168.1.0
```

## Private IPv4 Ranges

Private addresses are used inside local networks and are not directly routed on the public internet.

```
10.0.0.0      to 10.255.255.255
172.16.0.0    to 172.31.255.255
192.168.0.0   to 192.168.255.255
```

## Public IP Address

A public IP address is globally unique and can be used to communicate over the internet.

## Loopback Address

```
127.0.0.1
```

It represents the local machine and is commonly called `localhost`.

## APIPA Address

If a device cannot obtain an IP address from DHCP, it may assign itself an address from:

```
169.254.0.0/16
```

---

# 7. Subnetting Basics

Subnetting divides one network into smaller networks.

It helps:

- Reduce broadcast traffic
- Use IP addresses efficiently
- Improve network organisation
- Separate departments or security zones

## CIDR Notation

```
192.168.1.0/24
```

`/24` means the first 24 bits represent the network.

For a `/24` network:

```
Total addresses = 2⁸ = 256
Usable host addresses = 256 − 2 = 254
```

Two addresses are normally reserved:

- Network address
- Broadcast address

### Common formula

```
Usable hosts = 2ʰ − 2
```

Here, `h` is the number of host bits.

### Interview example

**How many usable hosts are available in a /26 subnet?**

```
Host bits = 32 − 26 = 6
Usable hosts = 2⁶ − 2 = 62
```

---

# 8. ARP

ARP stands for Address Resolution Protocol.

It is used to find the MAC address corresponding to an IPv4 address within the local network.

### Example

Suppose a computer knows the destination IP address but not its MAC address.

It broadcasts:

```
Who has 192.168.1.5?
```

The device with that IP replies with its MAC address.

The result is temporarily stored in the ARP cache.

### Important point

ARP works only within the local network. If the destination is outside the local network, the sender resolves the MAC address of the default gateway instead.

---

# 9. DNS

DNS stands for Domain Name System.

It converts human-readable domain names into IP addresses.

```
www.example.com → 93.184.216.34
```

### Simplified DNS resolution

1. The browser checks its cache.
2. The operating system checks its cache.
3. The request is sent to a DNS resolver.
4. The resolver contacts the required DNS servers.
5. The IP address is returned.

### Common DNS records

| Record | Purpose |
| --- | --- |
| A | Domain to IPv4 address |
| AAAA | Domain to IPv6 address |
| CNAME | Alias for another domain |
| MX | Mail server |
| TXT | Text-based verification or policy information |

### Interview question

**What happens if DNS is unavailable?**

Users may be unable to access websites using domain names, although direct communication using a known IP address may still work.

---

# 10. DHCP

DHCP stands for Dynamic Host Configuration Protocol.

It automatically provides devices with network configuration such as:

- IP address
- Subnet mask
- Default gateway
- DNS server

## DHCP Process

The process is remembered as DORA:

```
Discover
Offer
Request
Acknowledge
```

1. The client broadcasts a DHCP Discover message.
2. The server offers an IP address.
3. The client requests the offered address.
4. The server acknowledges the allocation.

### Static IP vs Dynamic IP

A static IP is manually configured and usually remains fixed.

A dynamic IP is assigned automatically by DHCP and may change.

---

# 11. Default Gateway

The default gateway is the router used when the destination lies outside the sender’s local network.

### Example

Suppose:

```
Computer: 192.168.1.10
Destination: 8.8.8.8
```

Since the destination is outside the local network, the computer sends the frame to the MAC address of its default gateway.

### Important clarification

The packet keeps the final destination IP address. Only the frame’s destination MAC address is set to the gateway’s MAC address.

---

# 12. NAT and PAT

## NAT

Network Address Translation converts private IP addresses into public IP addresses.

It allows devices using private addresses to access the internet.

## PAT

Port Address Translation allows many private devices to share a single public IP address by using different port numbers.

Example:

```
192.168.1.10:5000 → Public-IP:30001
192.168.1.11:5000 → Public-IP:30002
```

PAT is the form commonly used in home and office routers.

### Interview question

**Why is NAT required?**

NAT conserves public IPv4 addresses and allows private networks to communicate with the internet.

---

# 13. TCP and UDP

Both TCP and UDP operate at the Transport Layer.

## TCP

TCP is connection-oriented and provides reliable, ordered data delivery.

Features:

- Connection establishment
- Acknowledgements
- Retransmission
- Flow control
- Congestion control
- Ordered delivery

Used by:

- HTTP and HTTPS
- File transfer
- Email
- SSH

## UDP

UDP is connectionless and does not guarantee delivery or ordering.

It has lower overhead and is useful when speed is more important than perfect reliability.

Used by:

- Live video and audio
- Online gaming
- DNS
- Voice calls

## TCP vs UDP

| TCP | UDP |
| --- | --- |
| Connection-oriented | Connectionless |
| Reliable | No delivery guarantee |
| Ordered delivery | Packets may arrive out of order |
| Higher overhead | Lower overhead |
| Slower | Faster |
| Uses acknowledgements | No built-in acknowledgements |

### Interview-ready answer

TCP is preferred when losing data is unacceptable. UDP is preferred when occasional data loss is better than delay.

---

# 14. TCP Three-Way Handshake

TCP establishes a connection using three steps:

```
Client → SYN → Server
Client ← SYN-ACK ← Server
Client → ACK → Server
```

## Why three steps?

Both sides must confirm that they can send and receive data and must synchronise their sequence numbers.

### TCP connection termination

TCP usually terminates a connection using four messages:

```
FIN
ACK
FIN
ACK
```

Four messages are needed because TCP is full-duplex. Each direction is closed independently.

---

# 15. TCP Reliability

## Sequence Numbers

TCP assigns sequence numbers to data so that the receiver can arrange it correctly.

## Acknowledgements

The receiver confirms the successful receipt of data.

## Retransmission

If an acknowledgement is not received within a certain time, TCP retransmits the missing data.

## Flow Control

Flow control prevents a fast sender from overwhelming a slow receiver.

TCP uses a receiver window to indicate how much data the receiver can currently accept.

## Congestion Control

Congestion control prevents excessive traffic from overloading the network.

### Flow control vs Congestion control

Flow control protects the receiver.

Congestion control protects the network.

---

# 16. Ports and Sockets

A port number identifies a particular application or service running on a device.

Common ports:

| Protocol | Port |
| --- | --- |
| HTTP | 80 |
| HTTPS | 443 |
| FTP | 21 |
| SSH | 22 |
| DNS | 53 |
| SMTP | 25 |
| DHCP | 67 and 68 |

A socket is commonly represented as:

```
IP address + Port number
```

Example:

```
192.168.1.10:8080
```

### Interview question

**Why are port numbers required?**

An IP address identifies the device. A port number identifies the particular application on that device.

---

# 17. HTTP and HTTPS

HTTP is an application-layer protocol used for communication between web clients and servers.

## HTTP request methods

| Method | Purpose |
| --- | --- |
| GET | Retrieve data |
| POST | Create or submit data |
| PUT | Replace a resource |
| PATCH | Partially update a resource |
| DELETE | Remove a resource |

## Common status codes

| Code | Meaning |
| --- | --- |
| 200 | Successful |
| 201 | Resource created |
| 301 | Permanent redirect |
| 400 | Bad request |
| 401 | Authentication required |
| 403 | Access forbidden |
| 404 | Resource not found |
| 500 | Server error |

## HTTPS

HTTPS is HTTP protected using TLS encryption.

It provides:

- Confidentiality
- Data integrity
- Server authentication

### HTTP vs HTTPS

HTTP sends data without transport-level encryption. HTTPS encrypts the communication using TLS and generally uses port 443.

---

# 18. What Happens When You Enter a URL?

This is one of the most important networking interview questions.

Suppose the user enters:

```
https://example.com
```

The simplified process is:

1. The browser checks cached information.
2. DNS resolves the domain name into an IP address.
3. The device checks whether the destination is local or remote.
4. ARP is used to obtain the gateway’s MAC address if required.
5. A TCP connection is established.
6. For HTTPS, a TLS handshake occurs.
7. The browser sends an HTTP request.
8. The server processes the request and returns a response.
9. The browser renders the received HTML, CSS and JavaScript.

---

# 19. Unicast, Broadcast and Multicast

## Unicast

One sender communicates with one receiver.

## Broadcast

One sender communicates with every device in the local broadcast domain.

ARP requests are an example.

## Multicast

One sender communicates with a selected group of receivers.

It is useful for applications such as live streaming and conferencing.

---

# 20. Collision Domain and Broadcast Domain

## Collision Domain

A collision domain is an area where transmitted frames could interfere with one another.

Each switch port creates a separate collision domain.

## Broadcast Domain

A broadcast domain is the group of devices that receive the same Layer 2 broadcast.

Routers separate broadcast domains.

### Interview question

**Does a switch reduce broadcast traffic?**

A Layer 2 switch forwards broadcast frames to all ports in the same VLAN. It separates collision domains, but it does not normally separate broadcast domains.

---

# 21. Routing Basics

Routing is the process of selecting a path for packets between networks.

A router examines the destination IP address and checks its routing table.

A routing table may contain:

- Destination network
- Subnet mask or prefix
- Next-hop router
- Outgoing interface
- Route metric

## Static Routing

Routes are manually configured.

It is simple but difficult to maintain in large networks.

## Dynamic Routing

Routers learn and update routes using routing protocols.

Examples include RIP, OSPF and BGP.

For an entry-level interview, understanding the purpose of routing is more important than memorising every protocol.

---

# 22. Packet, Frame, Segment and Datagram

These terms describe data at different networking layers.

| Term | Layer |
| --- | --- |
| Segment | TCP transport data |
| Datagram | UDP transport data |
| Packet | Network-layer data |
| Frame | Data-link-layer data |
| Bits | Physical-layer transmission |

### Interview-ready explanation

Data is called a segment at the TCP layer, a packet at the IP layer and a frame at the Data Link Layer.

---

# 23. Error Detection

Networks use error-detection methods to determine whether data was damaged during transmission.

## Checksum

The sender calculates a value based on the data. The receiver performs the same calculation and compares the result.

## CRC

Cyclic Redundancy Check is commonly used at the Data Link Layer and is more effective than a basic checksum for detecting transmission errors.

### Important distinction

Error detection identifies corrupted data. Error correction or retransmission determines how the problem is fixed.

---

# 24. Frequently Asked Interview Questions

## What is the difference between bandwidth and latency?

Bandwidth is the maximum amount of data that can be transmitted per unit time.

Latency is the time taken for data to travel from source to destination.

A network can have high bandwidth but still feel slow if its latency is high.

---

## What is packet loss?

Packet loss occurs when some packets fail to reach their destination.

It may be caused by:

- Network congestion
- Faulty hardware
- Wireless interference
- Routing issues
- Buffer overflow

TCP retransmits lost data, while UDP does not provide automatic retransmission.

---

## What is ping?

`ping` checks whether a destination is reachable and measures approximate round-trip time.

It commonly uses ICMP Echo Request and Echo Reply messages.

---

## What is traceroute?

`traceroute` displays the sequence of routers through which packets travel before reaching the destination.

It is useful for identifying where delay or failure occurs.

---

## What is a firewall?

A firewall monitors and controls incoming and outgoing network traffic according to security rules.

It can allow or block traffic based on information such as:

- IP address
- Port number
- Protocol
- Connection state

---

## What is a proxy server?

A proxy acts as an intermediary between a client and another server.

It may be used for:

- Access control
- Caching
- Content filtering
- Privacy
- Monitoring

---

## What is a VPN?

A VPN creates an encrypted tunnel between a device and a remote VPN server.

It protects data while travelling through untrusted networks and can provide access to a private organisational network.

---

## Why does a device need both an IP address and a MAC address?

The IP address is used for communication across networks.

The MAC address is used for frame delivery within the local network.

Routers work with IP addresses, while switches mainly work with MAC addresses.

---

## Can two devices have the same private IP address?

Two devices in different private networks can use the same private IP address.

Two devices in the same network should not use the same IP address because it would create an IP conflict.

---

## What is the difference between authentication and encryption?

Authentication verifies identity.

Encryption protects the content of the communication from unauthorised reading.

---

# 25. Scenario-Based Questions

## A website opens using its IP address but not its domain name. What is likely wrong?

The most likely cause is a DNS problem because the network can reach the server, but the domain name is not being translated into an IP address.

---

## A computer receives an address beginning with 169.254. What does it indicate?

The computer failed to obtain an IP address from a DHCP server and assigned itself an APIPA address.

---

## Devices inside the same office can communicate, but none can access the internet. What should be checked?

Check:

- Default gateway
- Router connectivity
- NAT configuration
- DNS configuration
- Internet service provider connection

---

## A browser shows “server not found.” What could be the reason?

Possible causes include:

- Incorrect domain name
- DNS failure
- No internet connection
- Server unavailable
- Firewall blocking the request

---

## Why can video calls continue even if a few packets are lost?

Real-time applications often use UDP. Retransmitting old voice or video packets would create more delay, so occasional loss is tolerated.

---

# Final Revision Checklist

Before the interview, make sure you can confidently explain:

- OSI and TCP/IP models
- Hub, switch and router
- MAC address vs IP address
- TCP vs UDP
- TCP three-way handshake
- DNS, DHCP, ARP and NAT
- Public and private IP addresses
- Basic subnetting
- Ports and sockets
- HTTP and HTTPS
- What happens when a URL is entered
- Bandwidth, latency and packet loss
- Default gateway and routing
- Collision and broadcast domains

The goal is not to recite definitions. Explain each concept as a sequence of events and connect it with a practical example.

# Computer Networks — Layers, URL Flow, TCP vs UDP

## 1. Why Do Networks Use Layers?

Computer networks are divided into **layers** so that each layer handles one specific responsibility.

Instead of one giant system doing everything, each layer focuses on a particular job and communicates with the layer above and below it.

A useful mental model is:

> **Each layer adds something necessary to move data closer to its destination.**
> 

This makes networks easier to design, debug, and standardize.

---

## 2. OSI and TCP/IP Models

The **OSI model** has seven layers, but for interviews and practical understanding, the most important flow is:

```
Application
Transport
Network
Data Link
Physical
```

These roughly correspond to the practical TCP/IP model.

### Application Layer

This is where user-facing network protocols operate.

Examples:

```
HTTP / HTTPS
DNS
SMTP
FTP
```

Its job is to define **how applications communicate over the network**.

For example, a browser uses HTTP or HTTPS to request a web page.

---

### Transport Layer

The Transport layer handles communication between **processes running on different machines**.

The two major protocols are:

```
TCP
UDP
```

It introduces the idea of **ports**, allowing multiple applications on the same machine to communicate independently.

For example:

```
IP address → identifies the machine
Port       → identifies the application/process
```

---

### Network Layer

The Network layer is responsible for moving packets between different networks.

Its most important protocol is:

```
IP
```

The IP address identifies the source and destination machines.

Routers primarily operate at this layer and determine where packets should be forwarded next.

---

### Data Link Layer

The Data Link layer handles communication between devices on the **same local network**.

Examples include:

```
Ethernet
Wi-Fi
```

It uses **MAC addresses** to identify network interfaces locally.

---

### Physical Layer

The Physical layer deals with the actual transmission of raw bits.

Examples:

```
electrical signals
radio waves
fiber-optic signals
```

At this point, the information is ultimately represented as:

```
0s and 1s
```

---

# 3. Encapsulation

When an application sends data, that data travels **down through the networking layers**.

Each layer adds its own control information.

This process is called **encapsulation**.

```
Application Data
      ↓
Transport Layer
      ↓
TCP Segment / UDP Datagram
      ↓
Network Layer
      ↓
IP Packet
      ↓
Data Link Layer
      ↓
Frame
      ↓
Physical Layer
      ↓
Bits
```

At the receiving machine, the reverse process occurs.

Each layer removes the information intended for it until the original application data reaches the application.

This is called **decapsulation**.

### Interview-friendly definition

> **Encapsulation is the process of adding protocol-specific headers as data moves down the networking stack. Decapsulation removes those headers as the data moves back up the stack at the receiver.**
> 

---

# 4. What Happens When You Type a URL?

Suppose you enter:

```
https://example.com
```

into your browser.

A simplified but strong interview explanation is:

```
URL entered
    ↓
DNS resolution
    ↓
Destination IP obtained
    ↓
Local routing / ARP if required
    ↓
TCP connection
    ↓
TLS handshake
    ↓
HTTP request
    ↓
Server processes request
    ↓
HTTP response
    ↓
Browser renders page
```

Let's understand each step.

---

## Step 1 — Browser Parses the URL

The browser first understands the different parts of the URL.

For:

```
https://example.com
```

it identifies:

```
https        → protocol
example.com  → domain name
```

Because HTTPS is being used, secure HTTP communication will be required.

---

## Step 2 — DNS Resolves the Domain Name

Computers communicate using IP addresses, but humans usually use domain names.

The browser therefore needs to determine:

```
example.com → IP address
```

This is the job of **DNS — Domain Name System**.

Conceptually:

```
Domain Name
     ↓ DNS
IP Address
```

DNS therefore acts somewhat like the Internet's naming system.

---

## Step 3 — The Machine Determines Where to Send the Packet

Once the destination IP address is known, the operating system determines how to reach it.

If the destination is outside the local network, the packet will normally be sent toward the **default gateway/router**.

The IP address tells the system **where the final destination is**.

---

## Step 4 — ARP May Be Used

To actually send a frame over the local network, the sender needs the appropriate **MAC address**.

This is where ARP can be involved.

ARP stands for:

> **Address Resolution Protocol**
> 

It is used to find the MAC address associated with an IP address on the local network.

A useful distinction:

```
IP address  → identifies destination across networks
MAC address → identifies a device/interface on the local network
```

If the destination is on another network, the machine generally needs the MAC address of its **next hop**, such as the router.

---

# 5. Why Do We Need Both IP and MAC Addresses?

This is a common interview question.

IP addresses are used for **end-to-end routing across networks**.

MAC addresses are used to deliver frames **within the current local network**.

Think of it like:

```
IP  → Where is the final destination?

MAC → Which device should receive this frame right now?
```

As a packet moves through routers, the IP destination generally continues to represent the final destination, while the Data Link information can change from hop to hop.

---

# 6. TCP Connection Is Established

For normal HTTPS communication, the browser usually establishes a TCP connection with the server.

TCP is **connection-oriented**, so communication is established before application data is exchanged.

The classic process is the **three-way handshake**:

```
Client                   Server

SYN  -------------------->

     <---------------- SYN-ACK

ACK  -------------------->
```

After this, the TCP connection is established.

TCP then provides features such as:

- reliable delivery
- ordered delivery
- retransmission of lost data
- flow control
- congestion control

---

# 7. TLS Handshake for HTTPS

Because the URL uses:

```
https://
```

the browser and server establish a secure TLS connection.

TLS provides important security properties such as:

- **encryption** — others should not be able to read the traffic
- **integrity** — traffic should not be silently modified
- **authentication** — the browser can verify the server's identity

Only after the secure connection is established does normal HTTPS communication continue.

---

# 8. The Browser Sends an HTTP Request

The browser can now request the resource.

Conceptually:

```
GET / HTTP/1.1
Host: example.com
```

The request may also contain:

- headers
- cookies
- authentication information
- accepted content types
- other metadata

HTTP operates at the **Application layer**.

---

# 9. The Server Processes the Request

The server receives the request and determines how to respond.

Depending on the application, it may:

```
run backend logic
query a database
verify authentication
read files
call another service
```

The server then constructs an HTTP response.

---

# 10. The Server Sends an HTTP Response

The response contains a status code, headers, and usually some content.

For example:

```
HTTP/1.1 200 OK
```

followed by HTML or other data.

That response travels back through the network to the browser.

---

# 11. The Browser Renders the Page

Once the browser receives the HTML, it begins processing it.

The HTML may reference additional resources such as:

```
CSS
JavaScript
images
fonts
API requests
```

The browser may then perform additional network requests for these resources.

Finally, the page is rendered for the user.

---

# 12. The URL Story as a 60-Second Answer

A strong interview answer:

> **When I type a URL, the browser first parses it and resolves the domain name to an IP address using DNS. The operating system determines how to reach that IP, and ARP may be used to find the MAC address of the local next hop. For HTTPS, the browser establishes a TCP connection with the server, followed by a TLS handshake to create a secure channel. The browser then sends an HTTP request, the server processes it and returns an HTTP response, and finally the browser processes the returned resources and renders the page.**
> 

That answer is detailed enough to demonstrate understanding while still being concise.

---

# 13. TCP vs UDP

TCP and UDP are both **Transport-layer protocols**, but they make different trade-offs.

| TCP | UDP |
| --- | --- |
| Connection-oriented | Connectionless |
| Reliable | Best-effort delivery |
| Maintains order | Ordering not guaranteed |
| Retransmits lost data | No built-in retransmission |
| More protocol overhead | Lower overhead |
| Useful when correctness matters | Useful when low latency matters |

---

## TCP

TCP is designed for applications where receiving data **correctly and in order** matters.

If a packet is lost, TCP can retransmit it.

Typical examples include:

```
web traffic
file transfers
email
```

If part of a file is lost, simply ignoring the missing data is usually unacceptable.

---

## UDP

UDP is much simpler.

It sends datagrams without first establishing a TCP-style connection and does not itself guarantee that packets will:

- arrive
- arrive once
- arrive in order

That sounds worse, but it can be exactly what some applications need.

---

# 14. Why Would Anyone Choose UDP?

Because sometimes:

> **Getting the newest data quickly is more important than recovering old lost data.**
> 

Consider a live voice call.

If a tiny piece of audio is lost:

```
"Hel— how are you?"
```

the conversation can continue.

Waiting to retransmit that old audio could instead introduce noticeable delay.

This makes UDP useful in scenarios such as:

```
real-time voice/video
online gaming
some DNS communication
other latency-sensitive applications
```

So the distinction is not:

```
TCP = good
UDP = bad
```

It is:

```
TCP → prioritize reliable, ordered delivery

UDP → prioritize simplicity and low latency
```

---

# Interview Quick Revision

### What is encapsulation?

> Adding headers at each network layer as data moves from the application toward the physical network.
> 

### HTTP belongs to?

> Application layer.
> 

### TCP and UDP belong to?

> Transport layer.
> 

### IP belongs to?

> Network layer.
> 

### Ethernet and Wi-Fi belong to?

> Data Link layer.
> 

### IP vs MAC?

> IP is used for routing between networks; MAC addresses are used for local frame delivery.
> 

### TCP vs UDP?

> TCP provides reliable ordered communication, while UDP offers lightweight connectionless communication without delivery or ordering guarantees.
> 

### Why use UDP?

> When low latency and fresh data are more important than retransmitting every lost packet.
> 

### What happens when you type a URL?

```
URL
 ↓
DNS
 ↓
IP / Routing
 ↓
ARP for local delivery if needed
 ↓
TCP
 ↓
TLS
 ↓
HTTP Request
 ↓
Server
 ↓
HTTP Response
 ↓
Browser Rendering
```

Below are **interview-focused notes for Computer Networks 2 — IP, DNS, HTTP**, following exactly the scope in your placement calendar: public/private IP, NAT, subnet masks, DNS hierarchy/propagation, HTTP statelessness, methods/idempotency/status codes, and HTTP/1.1 vs 2 vs 3.

# Computer Networks 2 — IP, DNS & HTTP

## 1. Public IP vs Private IP

An **IP address** identifies a device/interface on an IP network so packets can be routed to it.

### Public IP

A **public IP address** is globally routable on the Internet.

Your home router typically receives a public IP from your ISP.

```
Internet
   |
Public IP: 49.x.x.x
   |
 Router
```

Public IPv4 addresses must be globally unique at a given time.

### Private IP

Private IPs are used **inside local networks** and are not directly routable across the public Internet.

Common IPv4 private ranges:

```
10.0.0.0      – 10.255.255.255
172.16.0.0    – 172.31.255.255
192.168.0.0   – 192.168.255.255
```

Example:

```
Laptop      192.168.1.5
Phone       192.168.1.8
TV          192.168.1.12
               |
             Router
               |
        Public IP: 49.x.x.x
```

Different homes can both have a device with `192.168.1.5` because private addresses only need to be unique **inside their own networks**.

### Interview answer

> A public IP is globally routable on the Internet, while a private IP is used within a local network and cannot normally be routed directly over the public Internet.
> 

---

# 2. NAT — Network Address Translation

The obvious problem is:

> If my laptop has a private IP, how does it communicate with Google on the Internet?
> 

Through **NAT**, usually performed by your router.

Suppose:

```
Laptop:
192.168.1.5:51000
```

sends a request to:

```
142.x.x.x:443
```

The router replaces the private source address with its public address.

Conceptually:

```
192.168.1.5:51000
        ↓ NAT
49.x.x.x:62001
        ↓
Internet
```

The router remembers the mapping:

```
49.x.x.x:62001 → 192.168.1.5:51000
```

When the response returns, it knows which internal device should receive it.

### NAT vs PAT

What home routers commonly perform is technically **PAT — Port Address Translation**.

Multiple devices can share one public IPv4 address because different port numbers distinguish their connections.

```
Laptop → 49.x.x.x:50001
Phone  → 49.x.x.x:50002
Tablet → 49.x.x.x:50003
```

### Why NAT became important

One major reason is the shortage of IPv4 addresses.

Instead of giving every device a public IPv4 address:

```
100 devices → 100 public IPs
```

a private network can use:

```
100 private IPs
        ↓
1 public IP
```

### Important interview nuance

NAT is **not fundamentally a security mechanism**.

It happens to make inbound connections less straightforward, but its primary purpose is address translation.

Firewalls are what explicitly enforce network access policies.

---

# 3. Subnet Mask

An IPv4 address has two conceptual parts:

```
Network portion | Host portion
```

The **subnet mask** tells us where that division occurs.

Example:

```
IP:      192.168.1.25
Mask:    255.255.255.0
```

This is commonly written:

```
192.168.1.25/24
```

`/24` means the first **24 bits belong to the network portion**.

Therefore:

```
Network: 192.168.1.0
Hosts:   192.168.1.x
```

Two devices such as:

```
192.168.1.10/24
192.168.1.80/24
```

belong to the same subnet.

But:

```
192.168.2.10/24
```

belongs to another subnet.

### Why this matters

Before sending a packet, a host effectively asks:

> Is the destination on my local subnet?
> 

If yes:

```
send directly on local network
```

If no:

```
send packet to default gateway/router
```

This connects directly to what you learned earlier about **ARP**:

```
Same subnet
→ find destination's MAC using ARP

Different subnet
→ find router/default gateway's MAC using ARP
```

### Interview answer

> A subnet mask determines which bits of an IP address identify the network and which identify the host. It allows a machine to determine whether a destination is local or must be reached through a router.
> 

---

# 4. DNS — Domain Name System

Humans prefer:

```
google.com
```

Networks ultimately need an address such as:

```
142.x.x.x
```

DNS performs that mapping.

Think of it as a **distributed hierarchical naming system**, not simply one giant database.

---

## DNS hierarchy

Consider:

```
www.example.com
```

The hierarchy is roughly:

```
Root DNS
   ↓
.com TLD server
   ↓
example.com authoritative server
   ↓
record for www.example.com
```

### The important players

**1. Stub resolver**

Usually part of your OS. It asks a DNS resolver for the answer.

**2. Recursive resolver**

Often provided by:

- ISP
- Google DNS
- Cloudflare
- company network

It performs the lookup on your behalf.

**3. Root server**

Doesn't usually know the site's IP.

It says approximately:

> “Ask the `.com` nameservers.”
> 

**4. TLD nameserver**

For `.com`, `.org`, `.in`, etc.

It says:

> “Ask the authoritative server responsible for `example.com`.”
> 

**5. Authoritative nameserver**

Contains the actual DNS records for the domain.

---

## Example DNS resolution

You enter:

```
www.example.com
```

Resolver:

```
Do I already have it cached?
        |
       No
        ↓
Root
        ↓
.com nameserver
        ↓
example.com authoritative DNS
        ↓
IP address
```

That result is then cached.

---

# 5. DNS Records You Should Know

You don't need twenty record types for placements.

Know these:

### A

Maps a hostname to an IPv4 address.

```
example.com → 203.0.113.10
```

### AAAA

Maps to an IPv6 address.

### CNAME

Creates an alias for another hostname.

```
www.example.com
→ example.vercel.app
```

### MX

Specifies mail servers for a domain.

### NS

Specifies the authoritative nameservers for the domain.

---

# 6. Why DNS Changes Take Time to Propagate

Suppose your domain currently resolves to:

```
Old IP
```

and you change it to:

```
New IP
```

The authoritative DNS server may update immediately.

But DNS resolvers around the world may have **cached the old value**.

Each DNS record has a:

### TTL — Time To Live

Example:

```
TTL = 3600 seconds
```

A resolver may reuse that answer for up to an hour before asking again.

Therefore:

```
Authoritative record updated
         ↓
Some caches still contain old value
         ↓
TTL expires
         ↓
Resolvers query again
         ↓
New value appears
```

That is what people commonly call **DNS propagation**.

### Interview answer

> DNS changes may appear gradually because recursive resolvers cache DNS records according to their TTL. Until those cached entries expire, some users may continue receiving the previous address.
> 

---

# 7. HTTP is Stateless

HTTP is fundamentally a **stateless protocol**.

This means:

> Each HTTP request is independent. The server does not inherently remember previous requests merely because they came from the same browser.
> 

For example:

```
POST /login
```

followed by:

```
GET /profile
```

HTTP itself does not automatically know:

```
"this GET belongs to the person who logged in earlier"
```

Something must carry that identity between requests.

Hence mechanisms such as:

```
Cookies
Sessions
JWTs
```

### Connection to Velvet

This is exactly why an authentication cookie is sent with later requests.

Conceptually:

```
HTTP is stateless
       ↓
Need persistent identity
       ↓
Cookie/JWT/session
       ↓
Server associates request with user
```

### Important distinction

**Stateless HTTP does NOT mean TCP reconnects for every request.**

HTTP can reuse an underlying connection while the **application protocol remains stateless**.

---

# 8. HTTP Methods

Methods describe the intended operation.

### GET

Retrieve a resource.

```
GET /users/123
```

Should generally not modify server state.

---

### POST

Create something or perform an operation.

```
POST /users
```

POST is generally **not idempotent**.

Sending the request twice may create two resources.

---

### PUT

Usually replaces an entire resource.

```
PUT /users/123
```

Typically idempotent.

---

### PATCH

Partially updates a resource.

```
PATCH /users/123
```

Example:

```json
{
  "name": "Shahid"
}
```

---

### DELETE

Removes a resource.

```
DELETE /users/123
```

Generally considered idempotent.

---

# 9. Idempotency

An operation is **idempotent** if performing it multiple times has the same intended final effect as performing it once.

Example:

```
DELETE user 123
```

Run once:

```
user gone
```

Run again:

```
user still gone
```

Final state is unchanged.

Therefore DELETE is idempotent.

Likewise:

```
PUT name = "John"
```

running it five times still results in:

```
name = John
```

But:

```
POST /orders
```

five times might create:

```
5 orders
```

so POST isn't inherently idempotent.

### Standard classification

```
GET      idempotent
PUT      idempotent
DELETE   idempotent

POST     generally not idempotent
PATCH    not guaranteed to be idempotent
```

### Important interview point

Idempotent does **not** mean the response must be identical every time.

It refers primarily to the intended effect on server state.

---

# 10. HTTP Status Codes

Don't memorize dozens.

Understand the families:

```
1xx → informational
2xx → success
3xx → redirection
4xx → client-side/request problem
5xx → server-side problem
```

### Important codes

**200 OK**

Request succeeded.

**201 Created**

New resource successfully created.

**204 No Content**

Request succeeded but there is no response body.

---

**301 Moved Permanently**

Permanent redirect.

**302 Found**

Temporary-style redirect.

---

**400 Bad Request**

Malformed/invalid request.

**401 Unauthorized**

Despite the name, usually means:

> Authentication is missing or invalid.
> 

**403 Forbidden**

Identity is known, but the user is **not allowed** to perform the operation.

This maps nicely to:

```
401 → Who are you?
403 → I know who you are, but you can't do this.
```

**404 Not Found**

Resource does not exist.

**409 Conflict**

Request conflicts with current resource state.

**429 Too Many Requests**

Rate limit exceeded.

---

**500 Internal Server Error**

Unexpected server-side failure.

**502 Bad Gateway**

An upstream server returned a bad response.

**503 Service Unavailable**

Service temporarily unavailable/overloaded.

---

# 11. HTTP/1.1 vs HTTP/2 vs HTTP/3

This is commonly asked because the versions improve **transport efficiency**, while HTTP semantics remain broadly recognizable.

## HTTP/1.1

HTTP/1.1 supports persistent connections, so one TCP connection can serve multiple requests.

But requests on a connection traditionally suffer from limitations around request ordering.

Browsers therefore often open several TCP connections to the same server.

Conceptually:

```
TCP connection 1 → request
TCP connection 2 → request
TCP connection 3 → request
```

---

# 12. HTTP/2

HTTP/2 keeps the same general HTTP semantics but changes how messages are transported.

Its major improvement is:

### Multiplexing

Multiple request/response streams can share **one TCP connection**.

```
One TCP connection
      |
 ┌────┼────┐
 ↓    ↓    ↓
Req1 Req2 Req3
```

It also introduces:

- binary framing
- header compression
- multiplexed streams

### But there is still a problem

HTTP/2 runs over **TCP**.

TCP guarantees ordered delivery.

If one TCP packet is lost, later received TCP data may have to wait for retransmission.

This creates **transport-level head-of-line blocking** across the connection.

---

# 13. HTTP/3

HTTP/3 changes the underlying transport.

Instead of:

```
HTTP
 ↓
TCP
```

it uses:

```
HTTP/3
 ↓
QUIC
 ↓
UDP
```

QUIC provides reliability, encryption, congestion control, and independent streams while being built over UDP.

The major advantage is that packet loss affecting one stream does not necessarily block unrelated streams in the same way TCP can.

It also integrates TLS more tightly, allowing faster connection establishment.

---

# 14. HTTP Versions — Interview Comparison

|  | HTTP/1.1 | HTTP/2 | HTTP/3 |
| --- | --- | --- | --- |
| Transport | TCP | TCP | QUIC over UDP |
| Multiplexing | Limited | Yes | Yes |
| Binary framing | No | Yes | Yes |
| Header compression | Limited | HPACK | QPACK |
| TCP-level HOL blocking | Yes | Still possible | Avoided between independent streams |
| TLS | Separate | Usually TLS | Built into QUIC |

The most useful answer:

> HTTP/2 introduced multiplexing so multiple HTTP streams can share one TCP connection, but packet loss can still cause TCP-level head-of-line blocking. HTTP/3 moves to QUIC over UDP, where independent streams can recover from loss separately and connection establishment is also faster.
> 

---

# The entire section as one story

You should be able to mentally connect everything:

```
User enters a domain
        ↓
DNS resolves hostname → IP
        ↓
Subnet mask determines local vs remote destination
        ↓
Remote destination → router/default gateway
        ↓
NAT translates private IP → public IP
        ↓
Connection established
        ↓
HTTP request sent
        ↓
HTTP is stateless
        ↓
Cookie/token identifies user across requests
        ↓
Server responds with HTTP status + data
```

## 8 lines worth memorizing before an interview

> **Private IP:** locally routable address, not directly Internet-routable.
> 
> 
> **NAT:** translates private addresses to public addresses; PAT lets many hosts share one public IP using ports.
> 
> **Subnet mask:** separates network bits from host bits and tells us whether a destination is local.
> 
> **DNS:** hierarchical distributed system translating names into records such as IP addresses.
> 
> **DNS propagation:** mostly cached records remaining until TTL expiry.
> 
> **HTTP statelessness:** each request carries no inherent memory of previous requests, hence cookies/sessions/tokens.
> 
> **Idempotency:** repeating an operation has the same intended final effect as doing it once.
> 
> **HTTP/2 vs 3:** HTTP/2 multiplexes over TCP; HTTP/3 uses QUIC over UDP to avoid cross-stream TCP head-of-line blocking.
> 

# Computer Networks 3: HTTPS, TLS & Web Security

> **Core idea:** HTTP moves data between a client and server. HTTPS is HTTP protected by **TLS**, which adds confidentiality, integrity, and authentication.
> 

---

## 1. Why plain HTTP is dangerous

With plain HTTP, data travels without cryptographic protection.

If you send:

```
username = alice
password = secret123
```

an attacker who can observe or interfere with the network may be able to:

- read the data,
- modify the data,
- impersonate the server.

HTTPS exists to prevent these problems.

---

# 2. What is HTTPS?

HTTPS is simply:

```
HTTP + TLS
```

HTTP still defines things like:

```
GET /products
POST /login
headers
cookies
status codes
```

TLS protects the connection carrying those HTTP messages.

Think:

```
Application:     HTTP
                   ↓
Security:        TLS
                   ↓
Transport:       TCP
```

---

# 3. What does TLS guarantee?

Remember these **three guarantees**.

## 1. Confidentiality

> Someone observing the network should not be able to understand the data.
> 

TLS encrypts the communication.

Without TLS:

```
password=hello123
```

may be visible.

With TLS:

```
7A F2 91 ...
```

an observer sees encrypted data instead.

---

## 2. Integrity

> The data should not be secretly modified while travelling.
> 

Suppose you send:

```
Transfer ₹100
```

An attacker should not be able to change it to:

```
Transfer ₹10000
```

without the modification being detected.

TLS provides cryptographic integrity checks.

---

## 3. Authentication

> The client needs confidence that it is communicating with the real server.
> 

If you visit:

```
https://example.com
```

TLS helps verify that the server presenting itself as `example.com` is authorized to do so.

This is where **digital certificates** become important.

### Easy recall

> **TLS = privacy + tamper protection + identity verification.**
> 

---

# 4. Why not use asymmetric encryption for everything?

There are two broad kinds of encryption involved.

### Asymmetric cryptography

Uses a key pair:

```
Public key
Private key
```

It is extremely useful for:

- authentication,
- digital signatures,
- establishing shared secrets.

But it is computationally expensive.

---

### Symmetric cryptography

Uses the same shared secret key for encryption and decryption.

It is much faster and therefore suitable for encrypting large amounts of application data.

---

## TLS therefore combines both ideas

Very simplified:

```
Client                    Server
  |                         |
  |----- TLS handshake ---->|
  |                         |
  | establish shared secret |
  |                         |
  |==== symmetric crypto ===|
  | HTTP data               |
  | HTTP data               |
```

The handshake uses public-key cryptography for authentication/key agreement.

After both sides establish shared session keys, **fast symmetric encryption** protects the actual HTTP traffic.

### Interview answer

> TLS does not use asymmetric cryptography for the entire connection because it is much more computationally expensive. Public-key cryptography is mainly used during the handshake for authentication and key establishment, after which symmetric encryption protects the bulk data.
> 

---

# 5. The TLS handshake: mental model

You do not need to memorize every packet.

Understand the story.

### Step 1: Client connects

The browser says roughly:

> “I want to establish a secure TLS connection, and here are the cryptographic options I support.”
> 

---

### Step 2: Server identifies itself

The server sends information including its **certificate**.

The certificate contains, among other things, the server's public-key information and the domain identity it is valid for.

---

### Step 3: Browser verifies the certificate

The browser checks things such as:

- Is it valid for this domain?
- Has it expired?
- Is it signed through a trusted certificate chain?
- Is the signature valid?

If verification fails, you may see:

> **Your connection is not private**
> 

---

### Step 4: Session keys are established

The client and server perform cryptographic key agreement.

They end up with shared secret session keys.

---

### Step 5: Encrypted communication begins

Now normal HTTP requests and responses travel through the protected TLS connection.

---

# 6. What is a digital certificate?

A certificate connects:

```
domain identity
       +
public key
       +
trusted signature
```

Conceptually:

> “This public key belongs to a server authorized for `example.com`.”
> 

Certificates are normally issued/signed through a **Certificate Authority**, or CA.

Examples of trusted certificate authorities are installed in browsers and operating systems.

---

# 7. What does a certificate actually prove?

This distinction is very important.

A valid certificate helps prove:

> **You are communicating with a party that controls the private key corresponding to a certificate valid for this domain.**
> 

It does **not** prove:

- the website is morally trustworthy,
- the company is legitimate in every sense,
- the site contains no malware,
- the site will not scam you,
- the site's application code has no vulnerabilities.

A phishing website can itself have perfectly valid HTTPS.

So:

> **HTTPS means the connection is secure, not that the website itself is safe.**
> 

Excellent interview line.

---

# 8. CORS, XSS and CSRF are completely different problems

These three are often confused.

Use this mental map:

```
CORS → Which websites may READ responses?

XSS  → Can attacker-controlled JavaScript RUN inside my site?

CSRF → Can another site make the user's browser SEND an unwanted request?
```

That distinction alone solves most interview confusion.

---

# 9. CORS

## Cross-Origin Resource Sharing

Browsers enforce a **same-origin policy**.

An origin is primarily based on:

```
protocol + host + port
```

For example:

```
https://example.com
```

and:

```
https://api.example.com
```

are different origins.

So JavaScript running on one origin normally cannot freely read responses from another origin.

---

## Why CORS exists

Suppose your frontend runs on:

```
https://frontend.com
```

and your API runs on:

```
https://api.example.com
```

The API can explicitly tell the browser:

> “I allow `https://frontend.com` to access my responses.”
> 

Using headers such as:

```
Access-Control-Allow-Origin: https://frontend.com
```

---

## Important: CORS is enforced by the browser

CORS does **not** stop someone from directly sending requests to your API using:

```
curl
Postman
another backend server
```

It primarily controls what **browser JavaScript** is allowed to read across origins.

### Interview line

> CORS is a browser security mechanism that controls whether JavaScript from one origin may access responses from another origin.
> 

---

# 10. XSS

## Cross-Site Scripting

XSS happens when attacker-controlled content is treated as executable JavaScript by your website.

Example of dangerous application behavior:

```html
Welcome, <USER_INPUT>
```

Suppose the attacker manages to provide:

```html
<script>
    stealSomething();
</script>
```

If the application blindly inserts this as HTML, the browser may execute it as part of your site.

That is XSS.

---

## Why XSS is dangerous

The malicious script executes **inside your site's origin**.

So it may be able to:

- read page data,
- make requests as the user,
- alter the page,
- steal accessible tokens,
- perform actions using the user's session.

That is why XSS is extremely serious.

---

# 11. How do we prevent XSS?

Main defenses:

### Escape output

Treat user input as **text**, not executable HTML.

Modern frameworks such as React escape ordinary rendered strings by default.

---

### Sanitize HTML

If your application genuinely needs users to submit HTML, sanitize it using a trusted library before rendering it.

---

### Avoid unsafe HTML insertion

Be very careful with things equivalent to:

```
innerHTML
```

or React's:

```tsx
dangerouslySetInnerHTML
```

The name is intentionally scary.

---

### Content Security Policy

A strong **CSP** can limit which scripts the browser is allowed to execute.

It acts as an additional defense layer.

### Recall

> **XSS = attacker gets JavaScript to execute as your website.**
> 

---

# 12. CSRF

## Cross-Site Request Forgery

CSRF exploits something browsers normally do automatically:

> **Attach authentication cookies to requests.**
> 

Imagine you are logged into:

```
bank.com
```

Your browser has:

```
session cookie
```

Now you visit:

```
evil.com
```

The malicious site somehow causes your browser to request:

```
POST bank.com/transfer
```

Your browser may automatically attach the bank's authentication cookie.

The bank sees:

```
valid session cookie
```

and might think:

> “This request came from the logged-in user.”
> 

But the user never intended to make the request.

That is CSRF.

---

# 13. Why HttpOnly does not stop CSRF

Suppose your cookie is:

```
HttpOnly
```

This prevents JavaScript from reading the cookie.

Excellent for reducing cookie theft through XSS.

But during a request, the **browser can still send the cookie automatically**.

Therefore:

> `HttpOnly` helps protect against token theft, but by itself does not prevent CSRF.
> 

This distinction is frequently asked.

---

# 14. How do we prevent CSRF?

Common defenses include:

### SameSite cookies

Example:

```
SameSite=Lax
```

or, when suitable:

```
SameSite=Strict
```

This restricts when cookies are sent with cross-site requests.

---

### CSRF tokens

The legitimate application includes a secret/unpredictable token with sensitive requests.

An attacking website cannot normally know that token.

The server verifies:

```
valid session
+
valid CSRF token
```

before performing the operation.

---

### Origin / Referer validation

For sensitive requests, the server can verify where the request originated.

---

# 15. The crucial CORS vs CSRF distinction

A common mistake:

> “CORS protects against CSRF.”
> 

Not reliably.

CORS mainly prevents malicious JavaScript from **reading a cross-origin response**.

But an attacker may sometimes still cause the browser to **send** a request.

CSRF is concerned with:

> **The unwanted request being executed.**
> 

Not whether the attacker gets to read the response.

### Recall

> **CORS controls reading. CSRF is about unwanted sending.**
> 

---

# 16. XSS vs CSRF

Another commonly confused pair.

### XSS

The attacker gets code to execute **inside your website**.

```
attacker code
      ↓
your origin
```

---

### CSRF

The attacker stays on **another website**, but tricks the user's browser into making an authenticated request to yours.

```
evil.com
   ↓
user's browser
   ↓
your website + user's cookie
```

Easy memory trick:

> **XSS = bad code gets IN.**
> 
> 
> **CSRF = bad request gets SENT.**
> 

---

# 17. Which defence protects against which problem?

### XSS

Think:

```
escaping
sanitization
CSP
avoid unsafe HTML
```

### CSRF

Think:

```
SameSite cookies
CSRF token
Origin checking
```

### CORS

Think:

```
Access-Control-Allow-Origin
allowed methods
allowed headers
credentials policy
```

Do not mix these categories.

---

# 18. A full browser example

Suppose:

```
Frontend:
https://app.example.com

API:
https://api.example.com
```

### CORS question

Should JavaScript on:

```
app.example.com
```

be allowed to read API responses from:

```
api.example.com
```

The API's CORS configuration controls this.

---

### XSS question

Can malicious user input cause JavaScript to execute inside:

```
app.example.com
```

Output escaping/sanitization/CSP help here.

---

### CSRF question

Can another website make the browser send an authenticated request to:

```
api.example.com
```

using the user's existing cookies?

SameSite cookies/CSRF defenses address this.

---

# 19. Authentication vs authorization

This connects directly to web security.

### Authentication

> **Who are you?**
> 

Example:

```
User has a valid session.
```

### Authorization

> **Are you allowed to perform this action?**
> 

Example:

```
User is authenticated
but
does not own this file
```

The server must still reject the operation.

Remember:

> **Authentication does not imply authorization.**
> 

And client-side checks are never sufficient security because the server must enforce permissions.

---

# 20. Interview rapid-fire answers

### What does TLS provide?

> Confidentiality, integrity and authentication.
> 

---

### Why use symmetric encryption after the handshake?

> Symmetric cryptography is much faster for encrypting large amounts of data, while public-key cryptography is mainly used for authentication and key establishment.
> 

---

### What does an HTTPS certificate prove?

> It helps establish that the server controls the private key corresponding to a certificate valid for the requested domain. It does not prove that the website itself is trustworthy or harmless.
> 

---

### What is CORS?

> A browser mechanism controlling whether JavaScript from one origin may access responses from another origin.
> 

---

### What is XSS?

> A vulnerability where attacker-controlled content causes JavaScript to execute in the context of a trusted website.
> 

---

### What is CSRF?

> An attack where another site tricks a logged-in user's browser into sending an unintended authenticated request to a trusted website.
> 

---

### Does CORS prevent CSRF?

> No. CORS mainly controls whether cross-origin JavaScript can read responses, while CSRF is about causing an unwanted authenticated request to be sent.
> 

---

### Does HttpOnly prevent CSRF?

> No. HttpOnly prevents JavaScript from reading the cookie, but the browser may still automatically attach that cookie to requests.
> 

---

# The 6 things to remember

If you forget everything else before an interview:

1. **HTTPS = HTTP over TLS.**
2. **TLS gives confidentiality, integrity and authentication.**
3. **Public-key crypto establishes trust/keys; symmetric crypto handles bulk traffic.**
4. **A certificate authenticates a domain connection, not whether the website is trustworthy.**
5. **XSS = attacker code runs inside your site.**
6. **CSRF = another site tricks the browser into sending an authenticated request; CORS mainly controls cross-origin response access.**

That covers the actual CN 3 target for this block without dragging you into unnecessary cryptography details.

# Computer Networks — The Journey of a Web Request

The easiest way to understand computer networks for interviews is not to memorize isolated definitions. Instead, follow one real event:

> **You type `https://example.com` into your browser and press Enter. What happens next?**
> 

Almost every important networking concept fits somewhere inside this journey.

---

## 1. First, the browser needs an IP address

Humans prefer names such as:

```
example.com
```

but networks ultimately route packets using IP addresses such as:

```
93.184.216.34
```

The browser therefore needs to answer:

> **Which IP address corresponds to `example.com`?**
> 

This is the job of **DNS — Domain Name System**.

Conceptually:

```
example.com
     ↓ DNS
93.184.216.34
```

DNS is therefore often described as the **phonebook of the Internet**: we provide a human-readable name and receive the network address needed to contact the server.

The system may first check whether the answer is already cached in the browser, operating system, or a nearby DNS resolver. If not, the DNS infrastructure is queried until the appropriate record is found.

### Interview distinction

**DNS resolves domain names to IP addresses.**

It does not establish the connection to the web server itself. It merely tells us **where the server is**.

---

# 2. The machine now knows where to send the request

Suppose DNS returns:

```
93.184.216.34
```

The browser now wants to communicate with that machine.

But before discussing the Internet as a whole, consider something closer to home.

Your laptop itself usually does not directly know the physical hardware address of every device it communicates with.

On a local IPv4 network, **ARP — Address Resolution Protocol** helps map an IP address to a MAC address.

Conceptually:

```
IP address
    ↓ ARP
MAC address
```

A useful distinction is:

```
DNS
Domain name → IP address

ARP
Local IP address → MAC address
```

DNS answers **which network address?**

ARP helps answer **which local network interface should receive this frame?**

---

# 3. Your router and NAT

Your laptop may have a private address such as:

```
192.168.1.10
```

This address is meaningful inside your local network but is not normally routed across the public Internet.

Your router has a public IP address and commonly performs **NAT — Network Address Translation**.

Conceptually:

```
Laptop
192.168.1.10
      |
      | NAT
      ▼
Router
Public IP
      |
      ▼
Internet
```

The router rewrites addressing information so multiple devices in your home or office can share a public Internet connection.

So when a response eventually returns, the router knows which internal device it belongs to and forwards it appropriately.

### Why NAT matters

Without NAT, every device would generally require its own globally routable IPv4 address.

NAT helped reduce pressure on the limited IPv4 address space and allows private addressing inside local networks.

---

# 4. Ports identify the application

An IP address tells us **which machine** we want.

But one machine may simultaneously run:

- a web server
- SSH
- a database
- an email server

We therefore also need to identify the intended application.

That is where **ports** come in.

Think:

```
IP address → which machine?
Port       → which service/process on that machine?
```

For example:

```
HTTPS → commonly port 443
HTTP  → commonly port 80
SSH   → commonly port 22
```

So a connection might conceptually target:

```
93.184.216.34:443
```

meaning:

> Contact this machine, specifically the service listening on port 443.
> 

---

# 5. TCP establishes a reliable connection

Normal HTTPS web traffic commonly runs over **TCP**.

TCP is connection-oriented and designed to provide reliable, ordered delivery.

Before application data is exchanged, TCP traditionally establishes a connection using the **three-way handshake**:

```
Client                         Server

   -------- SYN ------------>

   <----- SYN + ACK ----------

   -------- ACK ------------>
```

### What does this accomplish?

The client essentially says:

> “I would like to establish a connection.”
> 

The server replies:

> “I received your request, and I am ready too.”
> 

The client acknowledges that reply.

Now both sides know that communication can proceed.

---

# 6. Why TCP instead of simply sending packets?

Networks are imperfect.

Packets may:

- arrive late
- arrive out of order
- be duplicated
- be lost

TCP provides mechanisms to deal with these problems.

It uses concepts such as:

- sequence numbers
- acknowledgements
- retransmission
- flow control

The application therefore receives a reliable ordered byte stream rather than having to solve all of these problems itself.

---

# 7. TCP vs UDP

This is one of the most common networking interview questions.

## TCP

TCP emphasizes **reliability and ordering**.

It establishes a connection and ensures that missing data can be retransmitted.

Good examples include:

- traditional HTTP/HTTPS
- file transfers
- SSH

If part of a webpage arrives incorrectly, receiving the correct data is generally more important than saving a tiny amount of latency.

---

## UDP

UDP provides a much lighter communication model.

There is no TCP-style connection establishment and no built-in guarantee that every packet arrives or arrives in order.

This can be useful when **speed and low overhead matter more than perfect delivery**.

Examples can include:

- real-time voice/video
- online gaming
- some DNS communication

The intuition is:

> **TCP asks, “Did everything arrive correctly?”UDP largely says, “Here is the data; send it quickly.”**
> 

Neither is universally better. The appropriate protocol depends on what the application needs.

---

# 8. HTTPS adds TLS

We typed:

```
https://example.com
```

rather than merely:

```
http://example.com
```

The `S` represents secure communication.

HTTPS uses **TLS — Transport Layer Security** to protect communication between the client and server.

TLS mainly provides:

- **Confidentiality** — outsiders should not be able to read the traffic.
- **Integrity** — the data should not be silently modified in transit.
- **Authentication** — the client can verify that it is communicating with the intended server.

Conceptually:

```
Browser
   |
   | encrypted connection
   |
   ▼
Server
```

Before normal HTTP application data is exchanged, a TLS handshake establishes the secure session and validates the server's certificate.

---

# 9. Then comes HTTP

After the connection is ready, the browser can finally make an HTTP request.

For example:

```
GET /products HTTP/1.1
Host: example.com
```

This roughly means:

> “Give me the `/products` resource from `example.com`.”
> 

The server might answer:

```
HTTP/1.1 200 OK
Content-Type: application/json
```

followed by the actual data.

So:

```
TCP
→ reliable transport

TLS
→ secure communication

HTTP
→ meaning of the web request/response
```

These protocols solve different problems and work together.

---

# 10. What reaches the backend?

In a very small application, the request could go directly to your backend server.

```
Browser
   ↓
Node/Express server
```

In real deployments, there are often additional components in front of it.

For example:

```
Browser
   ↓
Reverse Proxy
   ↓
Backend
   ↓
Database
```

The backend processes the request, may query a database, constructs a response and sends it back through the network.

---

# 11. Proxy vs Reverse Proxy

These names are confusing until you ask:

> **Whose side is being represented?**
> 

## Forward proxy

A normal or **forward proxy** represents the **client**.

```
Client
   ↓
Forward Proxy
   ↓
Internet
   ↓
Server
```

The server sees communication coming through the proxy rather than directly from the original client.

It may be used for:

- access control
- filtering
- privacy
- corporate network policies

Think:

> **Forward proxy stands in front of clients.**
> 

---

## Reverse proxy

A **reverse proxy** represents servers.

```
Client
   ↓
Reverse Proxy
   ↓
Backend Server
```

The client communicates with the reverse proxy and may not know which internal backend ultimately serves the request.

A reverse proxy can perform tasks such as:

- routing
- TLS termination
- caching
- load balancing
- hiding internal application servers

Think:

> **Reverse proxy stands in front of servers.**
> 

This distinction is frequently asked in interviews.

---

# 12. Load Balancer

Suppose your application becomes popular.

One backend server may no longer be enough.

Instead:

```
                 ┌→ Server 1
Client → Load Balancer → Server 2
                 └→ Server 3
```

A **load balancer distributes incoming requests across multiple servers**.

The basic goal is to prevent one server from handling all traffic while others sit idle.

This provides benefits such as:

- scalability
- better resource utilization
- improved availability

If one server fails, the load balancer may route traffic to healthy servers instead.

---

# 13. What happens when you type a URL? — Complete picture

Now combine everything.

You enter:

```
https://example.com/products
```

### Step 1 — DNS

The system determines the server's IP address.

```
example.com
     ↓
DNS
     ↓
IP address
```

### Step 2 — Routing toward the destination

Your system prepares packets for that destination. On the local network, mechanisms such as ARP help deliver frames to the appropriate next-hop hardware interface.

### Step 3 — NAT

If your device uses a private IP address, the router may translate it to the public address used on the Internet.

### Step 4 — TCP

A connection is established with the destination service, typically using the three-way handshake.

### Step 5 — TLS

For HTTPS, the client and server establish secure communication and authenticate the server.

### Step 6 — HTTP

The browser sends the actual application request:

```
GET /products
```

### Step 7 — Infrastructure

The request may pass through:

```
Reverse proxy
      ↓
Load balancer
      ↓
Backend
```

### Step 8 — Application processing

The backend may:

- validate the request
- authenticate the user
- query a database
- perform application logic

### Step 9 — Response

The server sends an HTTP response back through the connection.

The browser receives it and displays or processes the result.

The complete mental picture is therefore:

```
URL
 ↓
DNS
 ↓
IP Address
 ↓
Local Network / ARP
 ↓
Router / NAT
 ↓
Internet
 ↓
TCP Connection
 ↓
TLS
 ↓
HTTP Request
 ↓
Reverse Proxy / Load Balancer
 ↓
Backend
 ↓
Database
 ↓
HTTP Response
 ↓
Browser
```

---

# Placement-Level Distinctions

### DNS vs ARP

> DNS translates a domain name into an IP address. ARP is used within IPv4 local-network communication to determine the MAC address associated with an IP address.
> 

### IP vs Port

> An IP address identifies the destination machine or network interface, while a port identifies the application/service to which the traffic should be delivered.
> 

### TCP vs UDP

> TCP provides connection-oriented, reliable and ordered delivery. UDP provides a lighter datagram-based mechanism without those delivery guarantees, making it useful where low latency and reduced overhead are more important.
> 

### HTTP vs HTTPS

> HTTPS is HTTP communication protected by TLS, providing encryption, integrity and server authentication.
> 

### Proxy vs Reverse Proxy

> A forward proxy sits on the client side and represents clients to external servers. A reverse proxy sits on the server side and represents backend servers to clients.
> 

### Reverse Proxy vs Load Balancer

> A reverse proxy is a broader server-side intermediary that may perform routing, TLS termination, caching and other functions. Load balancing is specifically concerned with distributing traffic among multiple backend instances; a reverse proxy can also act as a load balancer.
> 

---

# Final Mental Model

Do not remember Computer Networks as twenty unrelated protocols.

Remember one journey:

> **I know a domain name, DNS gives me an address. I need a particular service, so I use a port. TCP establishes reliable communication. TLS secures it. HTTP expresses what I actually want. Network infrastructure routes the request to the appropriate backend, and the response travels back to my browser.**
> 

Once that flow is clear, concepts such as **DNS, ARP, NAT, ports, TCP, TLS, HTTP, reverse proxies and load balancers** stop being disconnected definitions and become different stages of the same real-world request.

## PROVE checkpoint — 1 Sep 2026: **PASS**

Scenario-based recall was strong across default gateway/ARP, NAT/PAT, TCP/TLS/HTTP layering, CORS vs CSRF, switch broadcast behavior, DHCP/APIPA, TCP handshake, TTL/traceroute, certificates, authentication vs authorization, and TLS cryptography.

Only narrow precision/repair points:

- **TCP flow control:** the receiver explicitly advertises a receive window; it is not primarily just “slow ACKs.”
- **TCP termination:** the usual four-message close exists because TCP is full-duplex, so each direction closes independently.
- **Reverse proxy vs load balancer:** a reverse proxy can also perform load balancing; in an Nginx setup doing both server-side proxying and traffic distribution, it is both.

**Status:** MAINTAIN. No broad CN relearning needed.