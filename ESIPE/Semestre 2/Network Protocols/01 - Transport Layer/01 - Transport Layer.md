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
*Transmission Control Protocol; RFC 793, 9293*

An application interacts with TCP via **calls** (e.g., Open, Send, Receive, Close, Abort...): App transmits a command to TCP, and the later replies with information on the success/failure of the command execution, along with the connection state.
	*Those calls are defined by the OS, and may differ depending on the implementation of TCP.*


### Segment format

TCP segments are structured like so:

![[tcp.png]]
- **Sequence Number** and **Acknowledgement Number** are used for error control and maintaining the correct order of packets
- **Header Length (HLEN** is a **4-bit field** representing the header length in **32-bit words**.  
	*e.g., HLEN = 5 → Header size = 5×325×32 bits = 20 bytes*
- **Reserved** is always equal to 0
- **Flags**, 6 bits, one per flag. They are used to manage the session (acknowledge segments, open connection, close connection)
- **Window Size** is used for flow control. It defines the number of bytes one can receive before requiring an acknowledgement
- **Checksum** is used to detect error on header or data
- **Urgent** is a pointer indicating the last urgent byte to transmit (only if URG bit is set to 1)
- **Options** allows to provide options defined in RFCs (e.g., MSS, SACK)

About the flags:
- **Urgent (URG)**: Used for TCP to transmit this segment as fast as possible to the application layer, without respecting buffer order (e.g., Ctrl C in telnet)
	*Deprecated nowadays*
- **Acknowledgement (ACK)**: Acknowledges a segment (via Acknowledgement Number field)
- **Synchronisation (SYN)**: Used to establish a TCP connection
- **Push (PSH)**: Receiver does not have to wait until the queue is full before emitting data
- **Reset (RST)**: Requires a reset of the connection (usually due to interrupted/inexistant connection, error in segments...)
- **FIN**: No more data to transmit, used to end a TCP connection

> [!info]- Additional flags
> Modern TCP also includes two more optional flags which are negotiated during the handshake (per [RFC 3168](https://datatracker.ietf.org/doc/html/rfc3168)): 
> **CWR** (Congestion Window Reduced)
> **ECN** (Explicit Congestion Notification)

### Handshake

TCP **Three-way Handshake** is used to connect two machines. It defines terms and conditions of exchange (Buffers size, MSS, supported options...), and reserve resources on both ends (Memory space for buffers, CPU time).
Initiated by the application layer, it is opened via three segments (hence the name).
	*Two semi-openings, and a final control segment*

![[handshake.png|600]]
> [!info]
> By convention, we only mention a flag when its value is set to 1. If we take the second segment, it specifies "SYN/ACK" because both the SYN and ACK flags are true (other flags are ignored).

### Closing

Can be initiated by any end when no more resources to send. It frees the allocated resources.
It is composed of four segments.
	*The second segment on the figure bellow is actually done in two steps: FIN is sent first, and then ACK (each on its own segment)*

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
	*MSS = MTU - IP_header (20 bytes) - TCP_header (20 bytes)*

The smallest MSS is conserved out of the two: SDU ≤ min (MSSc ; MSSs)
![[mss.png|600]]
> [!info]
> `A` has the smallest MSS, which means segments emitted by both `A` and `B` will contain 1460 bytes of data at most.

The data provided by the application layer will be divided into `n` TCP segments, and each will have a size inferior or equal to MSSmin:
![[segmentation.png|500]]

*Each* segment is assigned a **sequence number** and an **acknowledgement number** in order to detect lost data or data arriving in the wrong order.
Those numbers are defined by an **Initial Sequence Number (ISN)** which corresponds to the very first segment emitted (SYN segment). Next segments's numbers will be added to this ISN **(offset)**.
	*The ISN value is selected by the OS, usually a random number to prevent TCP spoofing attacks*

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
- **Send buffer**: Stores data the application has written to the socket but has not yet been transmitted over the network
- **Receive buffer**: Stores data that has been received from the network but has not yet been read by the application
- **Retransmission buffer**: Stores copies of unacknowledged data
	When data is sent, a **Retransmission Timeout (RTO)** is started. If no Acknowledgement is received, data is re-sent, otherwise the RTO can be terminated

The PSH flag instructs the receiver to deliver buffered data to the application immediately, bypassing any buffering delays.
	*e.g., typing a command in SSH (small data chunks) uses PSH to ensure real-time responsiveness.*

When the recipient's receive buffer is full, TCP informs the application that it must read the data. When the application informs back that it has read all the data, the receive buffer is emptied and an acknowledgement segment is sent to the other end.
	*Reminder: receive buffer free space is indicated by the window field*

> [!caution]
> If the sender has insufficient data to fill the window, it transmits available data and relies on the receiver’s delayed ACK mechanism (no PSH required).

![[window.png|600]]
> [!info]
> The `W` value is the window field, indicating how much space is left in the machine's receive buffer


As we can see, to avoid overflow of receive buffer, segments flow is controlled by the receiver which sends its window value.
This means the window varies during connection: it's **sliding**.

![[sliding-window.png|700]]


### State machine

Our TCP connection traverse several states, depending on events from:
- **Service Primitives** (open, send, receive...)
- **Control segments** (SYN, FIN, ACK, RST)

![[states.png|450]]

Here is the TCP state machine which describes generic behaviours a connection will experience:
![[state-machine.png|675]]
> [!info]- Key states transitions
> - **LISTEN**: Server waits for SYN after passive open
>- **SYN-SENT**: Client waits for SYN-ACK after sending SYN
>- **TIME-WAIT**: After closing, waits 2×MSL2×MSL (Maximum Segment Lifetime) to handle delayed segments


***
## TCP advanced features

### Delayed ACK

Without any proper mechanism to manage acknowledgements, receiver should send an ACK each time it receives a segment.
	*So, an acknowledgement per data segment, huge waste of bandwidth...*

Piggybacking is a first solution we saw already, but **delayed ACK** is another mechanism that comes handy.
Here, the receiver does not emit an ACK for each received segment. Instead, it store received segments in the buffer and delays their acknowledgement to acknowledge them cumulatively in one single control frame.

> [!question] So when do we send the ACK segment?
> - When a segment containing the PSH flag is received
> - When a segment is stored since at least 500ms
> - If received two in-order segments

Overall, it optimises bandwidth but may negatively affect performance of the applications (latency).
It can, however, be turned off by the application (`TCP_QUICKACK` option).


### Silly Window Syndrome

Phenomenon happening when the receiver is overloaded, it keeps getting ridiculously small volumes of data, causing **important overhead**.

![[silly-window.png|600]]

> [!success] Solution
> Receiver announces `w = 0` until `w >= min(MSS;receiver-buffer/2`


### Nagle's Algorithm

We keep segments with a size inferior to MSS in the send buffer while previously emitted segments aren't acknowledged.
When the expected ACK is received, we send collected segments in one single large segment (or several if needed, logic is we just merge them).

The purpose of this is to **limit overhead for applications generating small volumes of data**.
It can be turned off by the application (`TCP_NODELAY` option).

> [!success] Advantage
> Protects from Silly Window Syndrome

> [!fail] Disadvantages
> - Hardly compatible with delayed ACK
> - Harmful increase in latency for interactive real-time applications (e.g. mouse movements in online games)


### RTO handling

Too short RTO = Useless retransmissions
Too long RTO = Huge latency
RTO should adapt to the network's current state (bottlenecks, route changing...)

#### Jacobson's algorithm

We can use Jacobson's algorithm to dynamically calculate the RTO, depending on:
- **Smoother RTT**: The average **Round Trip Time (RTT)**, which is the duration between transmission of a segment and receipt of its acknowledgement 
- **RTT Variance**: The variance of the average RTT
$$RTO=SRTT+4×RTTVAR$$

#### Karn's Algorithm

The aforementioned algorithm can be combined with Karn's: Resolve ambiguities in RTT measurements during retransmissions, via two strategies:
- **Ignore RTT samples for retransmitted segments**: ACKs for retransmitted packets cannot be reliably attributed to the original or retransmitted segment
- **Exponential backoff**: If RTO expires, Double the RTO on retransmission to reduce congestion (e.g., RTO = 2 × RTO).


### Keep-alive

This mechanism is often user to avoid a **middlebox (FW, NAT)** to interrupt an **idle** connection (valid but unused).
	*Turned off by default but can be activated by applications. Its usage is not recommended by IETF though...*

A keep-alive segment: 
- Has an empty SDU (or a single byte used for padding)
- Sequence number = (Sequence number of next segment to transmit) - 1, to avoid advancing the window
- Emitted if inactivity >= 2 hours


### Out-of-Order segments handling

The sender can't know if segments following a lost/delayed segment are well received, which means he cannot move his sliding window (latency issue).

> [!question] How to handle RTO expiration of lost/delayed segments?
> Retransmit the concerned segment only?
> Also retransmit all following segments without waiting for their RTO expiration (global retransmission)? 
>
> It depends of the situation:
> - Isolated loss: global retransmission is inefficient
> - Congestion (loss after loss...): global retransmission is preferable

#### Selective Acknowledgement (SACK)
*RFC 2018*

Optional TCP feature negotiated during the opening handshake (in the `Options` field).
A control segment now carries:
- The sequence number of the **next sequenced segment to be expected** (In the acknowledgement number field) 
- The **range of out-of-order bytes received correctly** (in the option field) 

Here, we retransmit the lost segment only.

#### Fast Retransmit

This feature is natively included in TCP congestion control (e.g., Reno, NewReno).
It works like this:
1. **Duplicate ACKs**: When the receiver gets an out-of-order segment, it sends a duplicate ACK for the last contiguous byte received (e.g., ACK 1001 for missing segment `1001–2000`)
2. **Trigger**: If the sender receives **3 duplicate ACKs** (ACK 1001 three times), it assumes segment `1001–2000` is lost
3. **Retransmit**: The sender immediately retransmits the missing segment (`1001–2000`), bypassing the retransmission timeout (RTO)

Without SACK, Fast Retransmit retransmits **only the oldest unacknowledged segment** (even if multiple losses occur). SACK enables retransmitting multiple gaps.


### Congestion handling

We need to manage those correctly, avoiding **congestion collapse** situations
	*congestion -> loss -> retransmission -> congestion...*

 RFC 9293 defines mostly three mechanisms to manage congestions:
 - Slow start
 - Congestion avoidance
 - RTO Exponential backoff
*Other mechanisms exists and can be applied by the Operating System, as long as they remain conform to RFC 2914, 5033 & 8961*

Furthermore, the **Explicit Congestion Notification** (ECN, RFC 3168) also aims at preventing congestion:
- Think ahead, before losses
- Involves routers and TCP endpoints;

#### Window(s)

Receiver defines the **Receiver window (rwnd)**, which indicates room left in the receive buffer. It is present in all segments.
Sender defines the **Congestion window (cwnd)**, which is calculated dynamically according to the network's state
When connection is opened, network state is unknown. An weak value is chosen, known as the **Initial window (IW)** (10 times the size of MSSsender, per [RFC 6928](https://datatracker.ietf.org/doc/html/rfc6928)).
	In general, rwnd > cwnd.

> [!example]- Evolution of cwnd when no congestion or loss
> ![[cwnd.png|600]]

Let's take a closer look at how cwnd behaves:
1. **Slow start**: When ACK is received, cwnd increased by `1 * MSS` (effectively doubling cwnd per RTT).
	- Slow start ends when cwnd reaches the threshold `ssthresh`, or when an RTO expiration occurs (potential congestion)
	- `ssthresh` is initially a large value, lowers when congestion detected

2. **Congestion avoidance**: Ends when congestion is detected or when rwnd was reached

> [!example]- Evolution of cwnd when congestion
> ![[cwnd-congestion.png|600]]

#### Algorithms

Several of them respects the aforementioned principles, their performance varies depending on the context.
The most popular ones are: **Reno**, **Tahoe**, **New Reno**, BIC, CUBIC
	*Reno: Fast retransmit + fast recovery
	Tahoe: Uses Fast Retransmit but reverts to slow start after loss detection*


***
## UDP
*User Datagram Protocol; RFC 768*

It offers minimal features:
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
