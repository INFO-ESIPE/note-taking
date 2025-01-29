[[Network Protocols]]
***
**Table of Contents**
```table-of-contents
```

****
## Refresher

### Transport Layer

The **network layer** is the part responsible for routing packets (making hop-by-hop decisions).
- A router does not store information about individual flows.
- A router only knows the next router on the path to the destination.
- Routing tables are dynamically updated in case of topology changes.
- It provides **best-effort delivery** (no Quality of Service guarantees).
- There is no access control by default (any packet is accepted unless explicitly filtered).

![[topology.png|600]]

What are the possible problems?
- Possible loss of packets
- Unpredictable arrival order
- No guarantee on delivery delays
- No mechanism to handle network congestion
Those problem *can* be addressed by the transport layer.

The two main transport protocols are **TCP** and **UDP** (QUIC also exists but is not covered here). Each has its own use cases and characteristics:

| **Feature**                                   | **TCP** | **UDP** |
| --------------------------------------------- | ------- | ------- |
| Application identification by port numbers    | Yes     | Yes     |
| Connection-oriented                           | Yes     | No      |
| Flow control                                  | Yes     | No      |
| Loss handling (acknowledgment/retransmission) | Yes     | No      |
| Sequencing control                            | Yes     | No      |
| Congestion control                            | Yes     | No      |
| Segmentation                                  | Yes     | No      |

### Ports

A port number identifies an application running on a machine. 
	*Represented by two bytes, so it ranges from 0 to 65535.*

Port Categories:
1. **Well-known ports (0–1023)**: Reserved by IANA for applications requiring root privileges.
    Examples: HTTP (80), FTP (20, 21), DNS (53), POP3 (110).
2. **Registered ports (1024–49151)**: Can be used by applications with ordinary privileges.
    Examples: OpenVPN (1194), XMPP (5222, 5269), ICQ (4000).
3. **Dynamic/private ports (49152–65535)**: Not officially allocated. Randomly assigned by the operating system for client applications (e.g., Firefox, qBittorrent).

A port is considered **open** if an active service is associated with it.

### Sockets

A socket is an access point to the lower layers (which are treated as "black boxes" by applications and developers). It is provided by TCP or UDP.

A socket is defined by the combination of an **IP address** and a **port number**.    
    *- Client socket: Client IP + Client port.
    - Server socket: Server IP + Server port.*

An exchange is uniquely identified by the **quintuplet**:  `Protocol + Client socket + Server socket`  
	*This combination is unique at any given moment across the entire internet.*


***
## TCP general principles


***
## TCP advanced features


***
## UDP

User Datagram Protocol (RFC 768). It offers minimal features:
- Not connection-oriented (Datagrams)
- Doesn't handle congestion control, sequencing or recovery. However, it does provide a minimal form of error handling through a **checksum**.
- Stateless
- Usable for **broadcast** and **multicast**

The concept is straightforward: We send a datagram, recipient may or may not receive it.

The datagram looks like this:
![[udp.png|550]]
- **Length** (for the entire datagram)
- **Checksum** is used for basic error detection. It is calculated over:
	- The UDP header
	- The Encapsulated data
	- A "pseudo-header" that includes fields from the IP header (source IP, destination IP, protocol, and UDP length). This pseudo-header is not transmitted but is used only for checksum calculation.
