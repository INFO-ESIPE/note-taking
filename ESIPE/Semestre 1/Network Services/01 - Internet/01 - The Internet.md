[[Network Services]]
****
**Table of Contents**
```table-of-contents
```

****
## Fundamentals

### What is a Network?

A network is an **interconnection of machines that exchange data**. Networks can be classified by their scope:
- **PAN (Personal Area Network)**: Small range (a few meters), private.
- **LAN (Local Area Network)**: Covers small areas like a building; usually private.
- **MAN (Metropolitan Area Network)**: City-wide or campus-sized network.
- **WAN (Wide Area Network)**: Covers large areas, like countries or continents.

![[networks.jpg|525]]

### Early Networking

Originated from telecommunication networks, mostly Public Switched Telephone Network (PSTN).
This model introduced mostly two mechanics:
- **Circuit Switching**: Dedicated paths for communication, ensuring **QoS (Quality of Service)** but inflexible for varying data loads.
- **Packet Switching**: Data split into packets with headers and payloads, sent asynchronously across the network. Basis for modern internet.

![[telephone-network.png|525]]

### Internet

In a nutshell, Internet is an open architecture allowing interconnection of heterogeneous networks.
	*In other words, it's a network of networks. Each network can have its own characteristics (Operating System, hardware) and still communicate over the internet with different specimens of networks.*

It was built on the following principles:
- **Hop-by-hop decision-making for routing**: Networks are interconnected by "black boxes" called **routers**, which:
	- Will forward what they receive in a simplistic and correct manner
	- Will not retain any information about individual flows
	- Will not resolve errors
- **Open Architecture**: No changes required in a network architecture for it to connect to the Internet.
- **Best Effort Delivery**: No guarantees for delivery; if packets are lost, retransmission (if any) is handled by higher-layer protocols such as [[01 - The Internet#Transport Layer|TCP]].
- **Decentralisation**: No central governing authority ensures reliability or performance, making the Internet resilient to failures in individual networks.
- **End-to-End Communication**: Hosts handle data integrity; routers and switches just forward packets.
- **Neutrality**: Flows of data are treated equally, independently of their source or destination.
- **No Access Control**: Any packet is accepted by default.

> [!info]- Key Milestones of the Internet
> - **1960s**: Development of packet switching concepts (MIT, RAND, NPL).
> - **1969**: ARPANET's first transmission.
> - **1972**: Invention of email, and overall concept of internet with birth of TCP/IP.
> - **1983**: Transition to TCP/IP protocols, introduction of DNS.
> - **1989**: Creation of the web (HTTP).


***
## Internet governance

### Organisation

Basic hierarchy:
![[governance.png|600]]
> [!info]
> Another important standard organisation, **Internet Assigned Numbers Authority** (**IANA**), is a function of ICANN.

- **ISOC** supervises the evolution of the Internet and ensures it remains an open model. Has authority over ICANN and IAB.
- **IANA** maintains the DNS root zone, and is in charge of distributing:
	- IP addresses (both v4 & v6)
	- Top level domains (TLDs)
	- Assigned numbers, which identifies protocols
- **IAB** is a committee supervising evolution of protocols taking part in the TCP/IP model.
- **IRTF** plans the evolution of architectures and protocols of the internet on long-term.
- **IESG** ensures correct evolution of protocols on short-term (6 months to 2 years).
- **IETF** produces internet's technical specifications, called **Request For Comments (RFC)**, and works on the first implementations of new TCP/IP protocols.

### RFC

Those are documents—published by the IETF and identified by an unique number—describing how [[01 - The Internet#Architecture|protocols]] works in depth.

They can have different status:
- **Internet Standard**: Official norm, tested and approved
- **Proposed Standard**: Proposal made by a community selected by IESG, good practices...

They either take the form of a text file, a PDF or a simple HTML file, and must specify:
Author(s) and the laboratory they are attached to, publication date, RFC unique number, status.

> [!info]- RFC Search engines
> - https://datatracker.ietf.org/
> - https://www.rfc-editor.org/


***
## Actors & Infrastructure

It can be broken down like so: 
LANs can be found at the extremity. Those LANs are linked to the core network by MANs & WANs managed by Internet Service Providers.

### Internet Transit

How is traffic passed from an ISP to another?
Two solutions to interconnect ISP networks:
1. **Peering**, usually a free and informal agreement. 

2. **Transit**, networks which knows a route towards every destination on the internet. They handle data incoming from any ISP's network as long as they have an official and fee-based agreement.


ISP Networks can belong to 3 tiers:
- **Tier 1**: Global backbone providers, interconnecting without costs (peering).
	*Orange OpenTransit, AT&T, Verizon, Sprint (USA)*
- **Tier 2**: Regional providers buying access from Tier 1 and selling to Tier 3 (transit).
	*Vodafone (GB), Hurricane Electric (USA), Geant (Europe)*
- **Tier 3**: Local ISPs providing internet access to end-users.

![[tiers.png|575]]

### Content Access Infrastructure

Until early 2000, resources were uniquely stored on a server inside the owner/ISP's network. 
Drawbacks:
- Latency (a person located in Japan could need a resource on an Italian server far away)
- Bottlenecks (resources are located on a single machine, every person who needs it will request it to the same server)

Two solutions were quickly brought up:
- **Content Delivery Network (CDN)**: We host **replicas** of content on several servers around the world, which reduces both aforementioned issues (closer to users, several access points).
- **Peer to Peer (P2P)**: Decentralised networks where pairs interacts directly, (almost) without any intermediate server.

> [!caution]
Those solutions are widespread nowadays, but it is worth noting that they aren't normalised by IETF and—in the case of CDN—introduces problematic modifications of the DNS protocol in order to know the geographical location of an IP.


***
## Network Models

How can the Internet connects heterogeneous networks? Via **protocols** which intervenes in various scenarios (client-server, router-router...).

We organise the logic in vertical layers. Two of those models are important:
1. **Open Systems Interconnection (OSI)**  
    The OSI model is a **theoretical framework** that defines seven distinct layers for standardising network communication. Each layer interacts only with the layer directly above or below it, ensuring modularity and clarity in network operations.
    
![[osi-model.jpg|750]]

2. **TCP/IP**
	A practical framework defining four layers, **implemented universally by all devices on the Internet**. 
	Unlike OSI, it is less theoretical and directly aligns with real-world networking.
![[tcpip-model.webp|600]]
> [!info]
> The application layer is managed by the developers, both the transport and network layers are managed by the Operating System, and the last layer is managed by the network card.  

> [!info]- Main protocols of TCP/IP
> - Application: HTTP, FTP, SSH, XMPP, SMTP, IMAP, SIP, DNS, DHCP, SSL/TLS ...
> - Transport: TCP, UDP, RTCP/RTP, QUIC
> - Network: IP, ARP, ICMP
> - Network Access: Ethernet, WiFi, ATM, SDH ... 
> 	*Those protocols aren't created by IETF*


***
## Data Communication

### Encapsulation and Decapsulation

As our data moves down the OSI layers, the **encapsulation** phenomenon occurs as **each layer is adding its own header** (containing information specific to the layer). Upon reception by the distant recipient, **headers are stripped in reverse order**, resulting in a **decapsulation** phenomenon.

![[encapsulation.png|600]]
> [!info]
> Each layer adds its own header, as its a way to forward information for the protocol of the same layer (information retrieval, parameters, error control...)

If we want to use a more appropriate vocabulary, we say that the **Service Data Unit (SDU)** is the data we pass to the next layer, the **Protocol Control Information (PCI)** is the header we add to the SDU we retrieve, which results in a **Protocol Data Unit (PDU)** that contains both the SDU and the PCI.

![[pdu.png|600]]

> [!info]
> As PDUs move through layers, they are referred to differently: 'segment' at the transport layer, 'packet' at the network layer, and 'frame' at the data link layer.

### Data Link Layer

This layer has a **local** reach, as its purpose is to transmit data in between **machines inside a same network** (limited by the router).
Locally, we identify machines by their **MAC Address**, which is **the physical address of the machine's network card**.

### Network Layer

Unlike the data link layer, this layer has a **global** reach, which means it can transfer data from a source to another **through several other networks**.
Globally, machines are identified by their IP Address, which are logical addresses.
Several mechanisms are put into place:
- **Routing**: Forward data from source to destination
- **Fragmentation**: Adapt frames when traversing various networks, depending on the Maximum Transfer Unit (MTU). 

IP routing is "best effort", "hop by hop" and dynamic, which means the route is automatically re-calculated if the topology changes.
	*Consequences: Loss are not detected nor managed, no mechanism against bottlenecks, no guarantee on the order of arrival.*

![[ip-routing.png|650]]

All of those consequences must be managed by the upper layer: Transport.

### Transport Layer

This layer is called an **"end to end" transport service**, which means it is executed by the hosts on the extremity.
	In other words, it is only executed by the two machines communicating, not the intermediate machines (routers, switches, access point...) in between.

Two main protocols for this layer:
- **Transmission Control Protocol (TCP)**, RFC 793
	*Connected, flow control, retransmission of lost frames, segmentation of data from the application layer*
- **User Datagram Protocol (UDP)**, RFC 768
	*Handles "datagrams" without connection establishment. It does not guarantee reliability, order, or segmentation beyond the network layer MTU limits.*

In other words, UDP is more adapted for real-time applications, while TCP emphasise more on reliability and data integrity.

### Application Layer

In a nutshell, there are two types of protocol for this layer:
- **User protocols** are an interface between the user and internet services
	*HTTP, FTP, SSH/Telnet, SIP, XMPP, SMTP/POP3/IMAP...*
- **Support protocols** are here for managing the network infrastructure
	*Host configuration with DHCP, Address resolution with DNS...*

and can usually work in two different modes:
- **Client -> Server**, which is a centralised architecture
	*The server is a machine that awaits a request from a client in order to execute the service it supports:
	Apache/Nginx for webservers, Postfix for mails, Bind9 for DNS...*
- **Peer-to-peer**, which is a distributed architecture

For machines to communicate on this layer, they must implement the same protocol, which defines a formal syntax and format for requests and responses.
The protocol for this layer requests the creation of a **socket** to the underlying transport protocol.
A socket is an access point to the layers under the application layer.
- Client and server both creates a socket
- The socket is defined by an **IP address and a port number**

> [!tip]
> The socket quintuplet (transport protocol, client IP, client port, server IP, server port) uniquely identifies a connection on the Internet.

