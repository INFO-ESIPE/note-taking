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

An application interacts with TCP via **calls** (e.g., Open, Send, Receive, Close, Abort...): App transmits a command to TCP, and the later replies with information on the success/failure of the command execution, along with the connection state.
	*Those calls are defined by the OS, and may differ depending on the implementation of TCP.*


### Segment format

TCP segments are structured like so:

![[tcp.png]]
- **Sequence Number** and **Acknowledgement Number** are used for error control and maintaining the correct order of packets
- **Header Length (HLEN)** is in number of bits, will be multiplied by 32 (usually its value is 5)
- **Reserved** is always equal to 0
- **Flags**, 6 bits, one per flag. They are used to manage the session (acknowledge segments, open connection, close connection)
- **Window Size** is used for flow control. It defines the number of bytes one can receive before requiring an acknowledgement
- **Checksum** is used to detect error on header or data
- **Urgent** is a pointer indicating the last urgent byte to transmit (only if URG bit is set to 1)
- **Options** allows to provide options defined in RFCs (e.g., MSS, SACK)

About the flags:
- **Urgent (URG)**: Used for TCP to transmit this segment as fast as possible to the application layer, without respecting buffer order (e.g., Ctrl C in telnet)
- **Acknowledgement (ACK)**: Acknowledges a segment (via Acknowledgement Number field)
- **Synchronisation (SYN)**: Used to establish a TCP connection
- **Push (PSH)**: Receiver does not have to wait until the queue is full before emitting data
- **Reset (RST)**: Requires a reset of the connection (usually due to interrupted/inexistant connection, error in segments...)
- **FIN**: No more data to transmit, used to end a TCP connection


### Handshake

TCP **Three-way Handshake** is used to connect two machines. It defines terms and conditions of exchange (Buffers size, MSS, supported options...), and reserve resources on both ends (Memory space for buffers, CPU time).
Initiated by the application layer, it is opened via three segments (hence the name).
	*Two semi-openings, and a final control segment*

![[handshake.png|600]]
> [!info]
> By convention, we only mention a flag when its value is set to 1. If we take the second segment, it specifies "SYN/ACK" because both the SYN and ACK flags are true (other flags are ignored).

### Closing

Can be initiated by any end when no more resources to send. It frees the allocated resources.
It is composed of two semi-closing segments using the FIN flag.

![[closing.png|600]]

Furthermore, a connection can also be brutally closed (or refused) by using the RST flag.

![[reset.png|600]]


### Segmentation
#### Sequence number

Two types of TCP segments:
1. **Control segments**: Empty SDU (only a header, no data carried). Solely used to manage the connection via flags
	*E.g., Simple acknowledgement (ACK), Opening (SYN/ACK), Closing (FIN/ACK)*

2. **Data segments**: Encapsulates data. Those segments can also carry informative flags via **piggybacking**
	*Piggybacking is carried to optimise the traffic. It avoids sending both a data and a control segment when both can be merged into one segment that performs both actions.*

The largest size an SDU can have is defined by the **Maximum Segment Size (MSS)**. There is one MSS per end (client MSS and server MSS), they are both announced in the opening segments of the handshake. This value is here to limit IP fragmentation when entering either the client or server's LAN.
	*MSS = MTU - network_header - tcp_header.*

The smallest MSS is conserved out of the two: SDU ≤ min (MSSc ; MSSs)
![[mss.png|600]]
> [!info]
> `A` has the smallest MSS, which means segments emitted by both `A` and `B` will contain 1460 bytes of data at most.

The data provided by the application layer will be divided into `n` TCP segments, and each will have a size inferior or equal to MSSmin:
![[segmentation.png|500]]

*Each* segment is assigned a **sequence number** and an **acknowledgement number** in order to detect lost data or data arriving in the wrong order.
Those numbers are defined by an **Initial Sequence Number (ISN)** which corresponds to the very first segment emitted (SYN segment). Next segments's numbers will be added to this ISN **(offset)**.
	*The ISN value is selected by the Operating System, usually a large number.*

> [!caution]
> For this lecture, we will consider the ISN to be 0, but keep in mind this is not the case in real-world scenarios.

![[sequence.png|500]]
> [!important]
> A segment's sequence number corresponds to its first byte's number, as we can see above.

#### Acknowledgement number

The acknowledgement number is a cumulative number that indicates the **sequence number of the next expected segment**.
Positioning the ACK flag to 1 indicates that the acknowledgement number has a real meaning. All segments should have this flag set to true, except the segment imitating the connection... 
	*Acknowledgement number has no meaning for the first segment, which is why it only contains a SYN flag.*

> [!example]- Basic example
> ![[acknowledgement.png|600]]

> [!example]- Handshake + Data transmission example
> ![[handshake-ack.png|600]]
> ![[handshake-ack2.png|600]]

About **piggybacking** we mentioned above, its simply a data segment also transporting acknowledgement information. It
saves bandwidth...

>[!example]- Handshake + Data transmission with piggybacking example
> ![[piggybacking.png|600]]

> [!example]- Closing example
> Follow up of our previous example
> ![[closing-ack.png|600]]

#### Flow control with buffers

Both ends of a TCP connection manages three buffers, each one serves as a queue for a specific situation:
- **Send buffer**: Stores data the application has written to the socket but has not yet been transmitted over the network.
- **Receive buffer**: Stores data that has been received from the network but has not yet been read by the application.
- **Retransmission buffer**: Stores copies of data that has been sent but not yet acknowledged by the receiver
	When data is sent, a **Retransmission Timeout (RTO)** is started. If no Acknowledgement is received, data is re-sent, otherwise the RTO can be terminated.

The PSH flag tells the sender that it can keep transmitting data until the recipient's receive buffer is full.
When full, TCP informs the application that it must read the data. When the application informs back that it has read all the data, the receive buffer is emptied and an acknowledgement segment is sent to the other end.
	*Reminder: receive buffer free space is indicated by the window field*

> [!caution]
> If the sender does not have enough bytes to send to fill the receive buffer, it must force TCP to transmit data to application by setting the PSH flag to 1 on the last segment. 

![[window.png|600]]
> [!info]
> The `W` value is the window field, indicating how much space is left in the machine's receive buffer


As we can see, to avoid overflow of receive buffer, segments flow is controlled by the receiver which sends its window value.
This means the window varies during connection: it's sliding.

![[sliding-window.png|700]]


### State machine

Our TCP connection traverse several states, depending on events from:
- **Service Primitives** (open, send, receive...)
- **Control segments** (SYN, FIN, ACK, RST)

![[states.png|450]]

Here is the TCP state machine which describes generic behaviours a connection will experience:
![[state-machine.png|675]]


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
