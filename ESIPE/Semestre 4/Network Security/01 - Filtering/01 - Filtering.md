[[Network Security]]
***
**Table of Contents**
```table-of-contents
```

****
## Middleboxes
*RFC 8517*

Any equipment placed between the source and destination, and providing services other than IP routing (most of the time, to increase performance or to add extra security).
This includes firewalls, NAT gateways, VPN gateways, proxies, authentication servers...
	*It can be placed anywhere on the infrastructure (inside the network, interconnection of operator networks...)*

![[middleboxes.png|600]]
	*CPE: Customer-Premises Equipment; LB: Load Balancer; DPI: Deep Packet Inspection*

> [!caution] Downsides to all of this
>- Access control to the network or applications (Firewall, proxy)
>- End-to-end not respected
>- Unequal treatment of flows (Load balancer, WAN optimiser)
>- Surveillance (DPI)
>- Censorship (Firewall, Proxy)


***
## Firewall
### Blocking and filtering
*RFC 2979, 7754*

**Blocking** voluntarily prevent access to an aggregate of resources.
	*Blocking access to Exploit-DB, Youtube....*
**Filtering** voluntarily prevent access to one resource or a subset of resources in an aggregate.
	*Blocking access to a specific CVE on Exploit-DB, a specific video on Youtube...*

The software (or hardware) responsible for this task is the firewall. It can be executed either at the network entrance (router), or on a host of the network.
It can work at any layer of the TCP/IP model:
- Network access (MAC address, in/out interface)
- Internet (Src/dest IP address, protocol field)
- Transport (Src/dest port, TCP header flags...)
- Application (Direct content)

![[firewall.png|500]]

It follows those configuration principles:
- Either a **blacklist** (allow everything by default and then create rules blocking specific resources/actions), or a **whitelist** (blocking everything by default and then create rules allowing specific actions).
	*The second choice is usually wiser*
- Rules are tested one after the other: as soon as a rule matches, it is applied
- Rules should be listed from the most generic to the most specific
- The more precise a rule, the less likely it is to be flawed
	*Think about IP spoofing, use of non-standard ports, fragmentation, etc*

### Types of firewalls

1. **First generation: Stateless Packet filter**
   Packets are treated independently
2. **Second generation: Stateful inspection**
   Aware of the overall flow and checks that the packet conforms to previous packets in the flow
   (e.g., TCP connection, ICMP error corresponding to a previously-treated IP packet...)
3. **Third generation: Application-layer**
   Provides support for application-layer protocols, applies filtering on it
   (e.g., Prevent `PUT` or `DELETE` HTTP requests)
4. **Next-generation firewall: Deep Packet Inspection (DPI)**
   Firewall filters packets depending of the content of it, not only the headers
   (e.g., Verify a message signature, look for malicious code)

### Manage IP fragmentation
*RFC 3128*

The challenge is: In general, only the first fragment carries the transport layer information necessary for the firewall (source port, destination port, flags).
How do we treat other fragments?

![[fragmentation-firewall.png|650]]

A solution would be to do the following:
- We apply the filtering rule to the first fragment (`FO=0`, Fragment Offset set to 0 and measured in **8-byte blocks**). We then register parameters of the packet (IP, protocol field, ID and filtering decision)
- Now, we apply the same rule to all fragments carrying the same parameters
- Once the last fragment arrives (`MF=0`), we erase saved parameters

![[firewall-fragment.png|650]]

> [!caution] Problems
> - Possible de-sequencing: `FO=0` not necessarily the first frame to arrive
> - If the fragments follow various paths (asymmetric routing, load balancing) and the firewall only processes one path, not all the fragments will have been processed
> - Requires the firewall to act "stateful" as it needs to take into account data from previous packets (so, doesn't work on stateless firewalls)

In general, the stateless firewall behaviour is the following:
- It accepts all fragments with a non-null `FO`
- Applies filtering rules to the `FO=0` fragment only. If rejected, reassembling becomes impossibles (respects rule)

![[fragmentation-manage.png|650]]

### Tiny fragment attack

Let's have a small refresher on IP properties (RFC 791):
- IP header size: `20 ≤ x ≤ 60 bytes`
- IP fragment payload size: `≥ 8 bytes`
- Offset in IP header = `(n° first SDU byte) / 8`

Purpose of this attack is to bypass firewall rules by exploiting IP fragmentation to hide critical TCP header information (e.g., flags like `SYN`, `ACK`).
The attack is the following:
- **First Fragment**: Contains the IP header and **only 8 bytes of the TCP header** (ports, sequence number
    Critical TCP flags (e.g., `ACK=1`) are excluded because they reside in bytes 13–14 of the TCP header
- **Subsequent Fragments**: Contain the remaining TCP header/data (including flags or malicious payloads)

![[tiny-fragment.png|675]]

As explained above, stateless firewalls inspect only the first fragment for TCP/UDP rules (e.g., port blocking). Since the first fragment lacks flags (e.g., `ACK=0`), rules tied to flags are not triggered.
Later fragments are allowed through because they lack transport-layer headers, and firewalls treat them as "data" without context.

> [!success] Defense mechanisms
> The first fragments requires at least the first 16 bytes of TCP header for the firewall to correctly inspect:
> - Source/destination ports (bytes 0–3)
>- Sequence number (bytes 4–7)
>- **Flags** (bytes 13–14).
> 
> So, drop all fragments where `FO=0` (first fragment) and TCP header length < 16 bytes

### Stateful firewall

Conceived for TCP, but can be adapted to UDP and ICMP (stateless protocols).
It analyses IP packets in a context, and keeps track of each packet's data in a **state table**.
	*Addresses, ports, flags, seq numbers for TCP, addresses & ports for UDP, addresses and types/codes for ICMP*

There is a predefined filtering policy for packets initiating the TCP connection or the UDP/ICMP flow.
Once accepted (handshake over for TCP, first packet for UDP/ICMP), two things are done:
- Register characteristics of the established flow inside the state table
- For UDP/ICMP flows: Trigger a timer which is refreshed only when a new packet is received

As long as the timer doesn't expire, or as long as no `FIN/ACK` segments are received, packets having correct characteristics keeps being accepted.

> [!caution] Performance
>More secure than stateless firewalls as it provides more strict controls, but:
>- Consumes more resources
>- Possible attack vector on the state table (table overloading)
>- Restarting the firewall will interrupt active connections or ongoing flows
>- Conversation over UDP can be interrupted if the timer expires too early (e.g., long moment of silence during the conversation)

*Note that QUIC packets—which are encapsulated in UDP datagrams—are resilient to firewall restarts as they are identified by a Connection Identifier (CID) instead of sockets.
This allow each side to know where the session was left thanks to this id.*

### Application-layer firewall

This generation knows the procedures for all application protocols it supports, it directly analyses their PDUs to take a decision.
This type of firewall is way more complex and resource-hungry. Furthermore, application-layer protocols which aren't supported cannot be analysed and are therefore systematically discarded (or allowed, depending on default policies but you still lack control over it brah).

> [!example]- Firewall for FTP
> A statically-opened channel for commands (port 21)
> A dynamically-opened channel for data, depending on the mode
> - Active: Client choses a port and the server opens a connection on port 20
> - Passive: Server choses a port > 1023 and the client opens the connection
>
> The firewall analyse the command channel to determine the opening mode
> ![[firewall-ftp.png|650]]
> 
> ![[firewall-ftp2.png|650]]

### DMZ
*Demilitarised zone*

There are two basic types of machines on a network:
1. Hosts on which externally initiated access is undesirable
   It can be workstations authorised to open connections to external servers (but not vice versa), or critical servers for internal use, with external connections prohibited (e.g., Database, LDAP, GitLab)
2. Internal servers requiring externally initiated access (e.g., Web, DNS, Email)

The idea is that we split the network into two subnetworks, each controlled by one or several firewall(s).
Rules are strict in the first case, as it belongs in the private network.
Rules are more permissive in the second scenario, we call it a demilitarised zone.

Mostly two DMZ designs:
1. **Three-legged model**, using one firewall
![[dmz.png|550]]

2. **Dual**, using two firewalls
![[dmz2.png|550]]


***
## PCP for NAT/CGN
*Port Control Protocol, RFC 6887*
### Concern

NAT and CGN disrupt the end-to-end Internet model, causing critical issues:
1. **End-to-End Connectivity Breakdown**
    NAT/CGN gateways rewrite IP/port headers, hiding internal hosts. This breaks protocols relying on direct addressing (e.g., peer-to-peer apps, VoIP)
    Applications requiring **inbound connections** (e.g., gaming servers, IoT devices) cannot function without explicit port mappings.
    
2. **Dynamic Port Limitations**
    Firewalls/NATs often block unsolicited inbound traffic. While **application-layer gateways** (ALGs) can dynamically open ports, they only support standardised protocols (e.g., FTP, SIP).
    New or proprietary applications (e.g., custom P2P apps) lack ALG support, failing behind NAT/CGN
    
3. **Static Redirection Challenges**
    Users can manually configure **static port forwarding** on home routers (NAT), but this is impossible under CGN
    CGN assigns shared public IPs to multiple users, making per-user port control infeasible without protocol support
    
4. **Stateful Interruptions**
    NAT/CGN mappings are stateful. Rebooting the gateway clears mappings, disrupting active connections (e.g., VoIP calls, VPNs)

PCP is here to allow dynamic port opening, and so despite firewalls, NAT and CGN.
It must, however, be embedded in all actors.

![[pcp-setup.png|450]]

### Concept

Host of the private network asks the router to open a port dynamically for a given period of time
1. Client → server (UDP port 5351): PCP request defining the port and the period of time
2. Server → client: Notification of rejection or acceptance.

![[pcp-example.png|650]]
> [!success] Benefits (especially over UPnP IGD)
> - Restores the end-to-end principle
> - Reduces the need for application firewalls
> - Possibility for the client to send an extension message to prevent the middlebox from interrupting the connection
> - Can use authentication mechanisms (e.g., EAP) to prevent rogue mappings
>- Interoperates across ISPs/CGN architectures

### PCP Proxy
*RFC 7648*

The problem we have is: Generally, the list of successive NAT/CGN routers is unknown to a host, and even if it's known, not necessarily possibility/permission to run PCP on all routers.
The solution is the PCP Proxy router: It receives the host's PCP request and opens the desired port, and then sends a PCP request to the **upstream PCP router**, which opens the requested port. 

![[pcp-proxy.png|650]]

> [!example]- FTP Example
> ![[pcp-proxy-example.png|700]]


***
## Proxy

We have looked at what a proxy was in the context of PCP, but maybe it's worth taking a look at the overall concept of proxies (as we have only skimmed the surface thus far).

It's simply a program acting as an intermediate between clients and servers.
It deals with application layer protocols (usually HTTP, FTP, DNS, IMAP/POP3, SMTP), and relays client requests and server responses.

Several kinds:
1. **Forward proxy**, controls the client-side
   Present on an equipment inside the host's LAN or the ISP's network, it collects client requests and forwards them to the server.
   Can put data in cache. Can hide client's IP (if anonymous), but might result in losing some information needed by the server.
   Used to filter requests/responses (IP Blacklist, URI, keywords, MIME types, file analysis), log activity or control access via authentication.

![[forward-proxy.png|300]]

2. **Reverse proxy**, controls the server-side
	Placed at the entrance of the servers' LAN, it forwards client requests to **one of** the servers providing the requested service. Purposes:
	- Balance the workload
	- Manage TLS on behalf of the server
	- Reduce overall server load (host static content, cache recently-asked resources...)

![[reverse-proxy.png|300]]

3. **Open proxy**, usable by any client on the Internet
	The most common type of proxy in the wilderness. Most of the time, used to bypass censorship

![[open-proxy.png|450]]

A proxy is called **anonymous** if it hides the client's IP address.
An **Interception proxy; transparent proxy/cache** is one that can be used without any configuration from the client (opposite of an **explicit** proxy)

### "Forwarded" extension
*RFC 7239*

Some proxies may hide or modify information concerning the incoming request (client IP, `host` field, usage of HTTP/HTTPS ...).
This can delete information useful for the server to correctly carry out a procedure, and fill log files with inaccurate and inoperable information.

The `Forwarded` field can be inserted in the HTTP request re-transmitted by the proxy to specify all the deleted information.
It is here to standardise the various non-standardised fields simulating this feature (`X-Forwarded-For`, `X-Forwarded-By`, `X-Forwarded-Proto`).
	*It is purely optional and not even turned on by default.*

| **Parameter** | **Argument**                               | **Role**                                                                                            |
| ------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| For           | IP+port, local or "unknown" (if anonymous) | Identifies the client which created this request, or a proxy which treated it                       |
| By            | IP+port, local or "unknown" (if anonymous) | Identifies interface of the proxy which received the request                                        |
| Host          | Original value of the `host` field         | Needed by the server if the reverse proxy insert a local name in the `host` field                   |
| Proto         | Original protocol (HTTP or HTTPS)          | Write a correct URI in a response, even if the client and the proxy doesn't share the same protocol |

> [!example]
> ![[proxy-forwarded.png|675]]

### CONNECT Method
*Specified in RFC 7231*

This HTTP method allows a client to establish an HTTPS connection with the server **via an explicit proxy**.
	Proxy opens the TCP connection with the server, and an HTTPS tunnel is materialised between client and server.

![[proxy-connect.png|650]]

> [!success] Advantages
> - Preserve confidentiality of the HTTP session between client and server
> - The client is the one choosing TLS parameters (algorithm, key length)

> [!fail] Disadvantages
> - Cannot cache any response
> - Impossible to filter. Security risk: Forces user to auth via `Proxy-Authorization` field
> - Target of the CONNECT request may be intended for a port other than 80 or 443 (e.g.: port 25 → risk of spam)

### TLS interception and MITM

The CONNECT method prevents the proxy from accessing request/response. However, it is *sometimes* the role of a proxy:
- Organisation network, to control connections or accessed websites 
- Parental control software (e.g.: ESET, Kindergate) 

TLS interception is about encrypting two times: Client -> Proxy, and Proxy -> Server. This way, the proxy can have access to the frames.
This requires the proxy to install a **custom CA certificate** on the client, enabling decryption. This introduces a trust dependency on the proxy operator...

However, this procedure gives room for a malicious proxy to perform a Man in the middle attack.

![[mitm-proxy.png|675]]
> [!caution] Risks
> - Proxy sees the content of requests and responses, which prevents confidentiality
> - Parameters of Proxy -> Server TLS session are unknown by the client, and potentially poorly chosen
