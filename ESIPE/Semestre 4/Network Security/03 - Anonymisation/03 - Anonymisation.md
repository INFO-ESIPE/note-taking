[[Network Security]]
***
**Table of Contents**
```table-of-contents
```

****
## VPN
*Virtual Private Network*
### Fundamentals

As explained in RFC 2764, the purpose is to emulate a private Wide Area Network. It extends the range of a LAN by operating the facilities of one or more public or private WANs.

It can execute mechanisms of QoS, filtering and encryption. It can be:
- **Host-to-site**: Connect a machine to a distant LAN
- **Site-to-site**: Connect networks together

For the later, three architectures:
![[vpn-points.png|425]]

VPNs can interconnect private networks at:
- Data link layer, we call them **L2VPN**
- Network layer, we call them **L3VPN**

> [!info] About VPNs and encryption
> Despite a common belief perpetually associating both, confidentiality is not necessarily the goal of a VPN.
> For instance, a BGP/MPLS VPN:
> - Ensures that flows are "sealed"
> - reserve resources on the WAN for those flows 
> - but provides no encryption
> 
> For this lecture, we will only focus on VPNs using encryption
> 

VPNs operates by **tunneling**: When traffic travels within the VPN, it is encapsulated in a new header (either at Layer 2 or Layer 3, depending on the VPN type).

![[tunneling.png|350]]

### Establish connection

"For **permanent connections** between multiple sites of the same organisation, the end of the tunnel can be:
- **Custom Edge (CE)**: Refers to the equipment at the extremity (edge) of the site
	*ISP Ignores existence of the VPN*
- **Provider Edge (PE)**: Refers to the equipment at the edge of the ISP's network responsible of establishing the VPN
	*PPVPN (Provider Provisioned VPN, RFC 3809)*

Otherwise, it is temporarily between a travelling host and a site.

![[vpn-config.png|700]]

> [!example]- VPN Use case - (Partial) anonymisation of Internet communications
> VPN established between the host and a commercial VPN gateway.
> Data from the host is transferred to the destination server via the gateway, the later hides the host's identity.
> 
> ![[vpn-anon.png|425]]
> 
> Inconvenients:
> - Host's identity and data content is known by the gateway
> - Protocols and algorithms used between gateway and server aren't chosen by the host (same problem as [[01 - Filtering#TLS interception and MITM|TLS interception with proxies]])
> 
> For both reasons, it is important to chose the solution wisely.

> [!example]- VPN Use case - Avoid censorship, download resources illegally
> The principle is: If the context involves a packets filtering depending on a geographical region, the host connects to a VPN gateway located in an authorised region.
> 
> ![[vpn-censorship.png|600]]
> 
> Inconvenient remains the same as above: The gateway knows the host's identity and the exchanged data content...
 
*Note: VPNs are a rabbit hole. We won't explore the entire topic as it requires more time than we have for this lecture. Here is the classification of VPNs according to RFC 4026:*
![[vpn-classification.png|780]]


***
## IPsec
*RFC 4301. Fundamental aspects are also described [[15 - IPsec|in this lecture]]*
### IPsec Fundamentals

This architecture is dedicated to L3VPNs. It guarantees confidentiality, integrity and authentication of IP packets (both v4 and v6).
It can be implemented in a host, a router, a firewall, or other network devices. it defines;
- A **Security Policy Database (SPD)** specifies how to handle each packet (`protected`, `discarded`, `bypass IPsec protection`)
- Encapsulation protocols (e.g., ESP, AH) to protect the traffic
- Protocols managing the encryption keys (mostly IKE)

The purpose is to **protect traffic at the network layer**, thus protecting all applications.

IPsec has 3 versions.
IPsec-v1 is obsolete and should not be used. We will study IPsec-v3 in this lecture, even though the v2 remains widely used.

IPsec is, in fact, an entire architecture composed of several protocols and algorithms:

![[ipsec-archi.png|600]]

For the IPsec architecture to function, several components are required:
- **IPsec gateway**: A device (e.g., router, firewall, or dedicated appliance) responsible for **encrypting and tunneling IP packets**
- **Security Policy Database (SPD)**: A local database on each gateway that defines **how to handle each IP packet**.
  Rules specify whether to **protect**, **discard**, or **bypass IPsec** for incoming and outgoing traffic.
> [!example]
> A rule might state, "Protect all traffic from `192.168.1.0/24` to `10.0.0.0/8` using ESP."
- **Ancillary equipment** (Certification Authority, Logwatch server...)

A **Security Association (SA)** defines all the parameters required to securely process IP packets between two IPsec gateways. 
Key characteristics:
- **Unidirectional**: Each SA applies to traffic in one direction. Thus, a bidirectional connection requires **two SAs** (one for each direction).
- **Contents of an SA**:
    - Identifiers for the gateways (e.g., IP addresses).
    - Encryption keys and algorithms (e.g., AES, SHA-256).
    - Encapsulation protocol (e.g., ESP or AH).
    - Sequence number (to prevent replay attacks).
    - Lifetime (duration before the SA expires and must be renegotiated).

![[ipsec-sa.png|600]]
> [!info]
> - **One SA per protocol**: Separate SAs are created for ESP and AH if both are used.
>- **Security Associations Database (SAD)**: Stores all active SAs on a gateway.  
>- **Security Parameter Index (SPI)**: A unique identifier for each SA, included in the IPsec header to help the receiving gateway locate the correct SA in its SAD.


### Modes of operation

IPsec operates in two modes: 
1. **Transport Mode** - Only the payload (SDU) is protected, the IP header remains unchanged
   Mostly used for host-to-host 
2. **Tunnel Mode** - The entire original IP packet is encapsulated within a new IP packet
   For network-to-network and host-to-network (VPNs, for instance)

![[ipsec-modes.png|600]]

IPsec offers several levels of granularity, for instance:
1. A same tunnel between two gateways carries all the traffic
2. A tunnel per TCP connection
3. Or, a tunnel for routing protocols (e.g., BGP), and none for other flows

![[ipsec-granularity.png|450]]


### Encapsulation

Two options: AH or ESP.
These protocols protect packets by providing several services, which can be selectively enabled or disabled as needed.
They can use several algorithms; each is defined in its own RFC.

The following table summarises the services provided by each protocol:

|         | **Authentication** | **Integrity Check** | **Anti-Replay** | **Confidentiality** | **IP Header protocol** |
| ------- | ------------------ | ------------------- | --------------- | ------------------- | ---------------------- |
| **AH**  | X                  | X                   | X               |                     | 51                     |
| **ESP** | X                  | X                   | X               | X                   | 50                     |

> [!info]
> Both protocols can be used in combination, but ESP is now generally used alone.

#### ESP
*Encapsulating Security Payload; RFC 4303*

In the case of EAP, confidentiality and integrity can be managed by the same algorithm, or by two separate ones.
![[ipsec-esp.svg|600]]
> [!info] Integrity Check Value (ICV)
> An optional ICV can be concatenated at the very end of the packet. It is in cleartext and of variable size.

#### AH
*Authentication Header; RFC 4302*

Mandatory for IPsec-v2, but became optional in the v3. It ensures authentication and integrity of:
- **Non-modifiable** fields of the IP header (which excludes `DSCP/ECN`, `Flags`, `Fragment offset`, `TTL`, `Header checksum`)
  **Unusable with NAT !!**
- Fields of AH header (besides ICV)
- All fields following the AH header
![[ipsec-ah.svg|600]]
> [!info]
> AH is never used on its own nowadays. Always with ESP.
> Main purpose of AH is to protect fields which aren't covered by ESP.


### IKE
*Internet Key Exchange; RFC 6071*

IKE is used for the **dynamic negotiation and management** of encryption parameters required to establish and maintain the VPN.
	*Uses UDP port 500, or 4500 in case of CGN*

In two phases:
1. **Establish a secured channel**
	Negotiate hashing and encryption algorithms, and create a shared key via Diffie-Hellman
2. **Define SAs for ESP and/or AH** 
	Identifies extremity equipment and exchange certificates, negotiate hashing, compression and encryption algorithms, and build keys used by SAs


***
## TOR
*The Onion Router. Fundamental aspects are also described [[16 - Anonymity#The Onion Router (TOR)|in this lecture]], I will avoid repeating information...*
### Concept

Initially a DARPA and NRL project in the 90s, it became open-source in 2006 under the name "The Tor Project". 
It provides **end-to-end encryption** of TCP communication, and *partially* anonymises the author of this connection. Only the first node knows the client's IP address, while "hidden services" can be leveraged to anonymise the destination IP.

So, TOR is for TCP only, client-server model (no P2P), and its anonymisation efficiency depends on the number of users and nodes available.
TOR operates as an overlay network on top of the Internet, relying on standard IP routing between nodes. TLS Sessions are established between the client and **nodes**, also known as **Onion Routers (OR)**.

**Directory Servers (DS)** maintain signed directories that list all active **Onion Routers (ORs)** and their current status (e.g., availability, bandwidth, supported protocols)
The **Onion Proxy (OP)** is the client-side that **queries Directory Servers** to gather information about ORs and **constructs circuits** for routing traffic.
	*This software periodically downloads DS via HTTP*

![[tor-actors.png|600]]

Client choses three ORs composing the **circuit**. An OR only knows its predecessor and successor.
	*This is why, as we mentioned above, only the first node is aware of the client's IP.*


### Cells

TCP segments are divided in 512-bytes cells which do not include the source IP. Each cell is **encrypted in multiple layers** by the client (once for each OR in the circuit). As the cell traverses the circuit, each OR decrypts one layer to reveal the next destination.
The **exit node** is the final OR in the circuit, responsible for forwarding traffic to the destination server.

![[tor-cells.png|600]]

Two types of cells:
1. **Relay cells**: Contains data of the TCP connection
2. **Control cells**: Used to establish, maintain and destroy the circuit

Both cell types include a header with the following fields:
- A **local-scope Circuit Identifier (CircID)**
- A **command (CMD)** telling the receiver how to handle the data:
	- `Relay`: Forward a *relay cell* to the next node on the circuit
	- `Padding`: Keep-alive purpose to maintain the TLS connection opened
	- `Create`/`Created`: To establish a new circuit
	- `Destroy`: Delete the current circuit

![[tor-cells-format.png|600]]

Now, the **relay cell header** contains additional fields specific to relay operations:
- **StreamID**: Id of the flow on the circuit (as a circuit can multiplex several streams)
- **Len**: Length of the payload (data)
- **CMD**: Several payload commands are available, this field specifies which one to use:

| **Command**                        | **Meaning**                                             |
| ---------------------------------- | ------------------------------------------------------- |
| `Relay Data`                       | Forwarding data from the flow                           |
| `Relay Begin`                      | Open a flow on the circuit                              |
| `Relay End`                        | Close the flow                                          |
| `Relay Teardown`                   | Close the flow in case of an anomaly                    |
| `Relay Connected`                  | Inform that the flow is opened                          |
| `Relay Extend`/`Relay Extended`    | Extend the circuit by one node / Acknowledge the extent |
| `Relay Truncate`/`Relay Truncated` | Delete a part of the circuit / Acknowledge the deletion |

![[tor-relay-cell.png|600]]
> [!info]
> So, a relay cell has the `Relay` CMD and an additional header of four fields right after. The specific Relay command is specified in this second CMD field.


### Circuit

As mentioned above, the circuit consists of **three nodes** selected by the OP based on information from Directory Servers, including:
- Node status (e.g., online/offline)
- Available bandwidth and resources
- Supported application protocols (e.g., HTTP, SMTP)
- Node family (to avoid selecting nodes managed by the same organisation)
- Guard node status (for entry nodes)

During startup, the OP choses more than a circuit. It selects:
- **Three guard nodes**, highly available and with large bandwidth. They will be used for future circuits.
- **An entry node among those three guard nodes**
- **An Exit node**, supporting the application protocol used by the client (e.g., HTTP, SMTP), with enough resources to handle the streams, and **distinct from guard nodes**

![[tor-circuit.png|450]]

Nodes are selected based on strict criteria to enhance security and performance:
1. Nodes must **belong to different families**. A family is the collection of nodes managed by a same organisation.
2. Nodes should **not share the same `/16` prefix.**

Furthermore, additional circuits are constructed in advance, for two reasons:
- Circuit changes every minute, to help preserve anonymity
- A backup circuit should be ready to use in case of problem on the current circuit  

![[tor-circuit-change.png|450]]

As described earlier, **multiple TCP streams** can be multiplexed over a single circuit to reduce latency.
	*This is very useful when several connections are opened at the same time, which typically happens when you request data from a website (main page, scripts, CSS, multimedia...).*

![[tor-multiplexing.png|650]]


### Manage circuits and streams
#### Creation of the circuit(s)

An OR possess a key pair reserved for construction of the circuit. The public key is called the **Onion Key**.
Incremental construction of the circuit (one OR after another). 
	*Circuit between `X` and `Y` identified by a CircID denoted `Cy`. 
A secret key `KsecN` is created between source and node `N` via Diffie-Hellman.
Once the circuit is assembled, node `N` forwards any incoming cell with CircID `Cn` to CircID `Cn+1`.

![[tor-circuit-creation.png|750]]

#### Opening stream on a circuit

To open a stream (TCP connection) on an established circuit:
- Source choses a StreamID, and sends a `Relay Begin` cell on the circuit 
- Exit node opens a TCP connection with the destination server, and sends a `Relay Connected` cell on the circuit

![[stream-open.png|700]]

#### Closing stream on a circuit

There is a two-way handshake:
- **Relay End** cells are forwarded
- **Only exit nodes know that the stream is closed**

![[stream-close.png|700]]

#### Destruction of a circuit

To destroy the entire circuit:
1. Source sends a cell carrying the `Destroy` command
2. Upon receiving the cell, a node:
	- Destroy all streams associated to the circuit
	- Destroys the circuit
	- Passes the cell to its successor (chain reaction)

![[tor-circuit-destroy.png|700]]

To destroy a subset of the circuit only, we send a cell carrying the `Relay Truncate` command to a specific node on the circuit.
This allows us to rebuild the end of the circuit, starting from the node we sent the cell to.

![[tor-circuit-destroy-steps.png|700]]


### Directory servers

Deeper look at directory servers. They are HTTP servers that maintain and distribute a description of the TOR network's current state.
DS public keys are pre-registered in OPs, similar to how web browsers come pre-loaded with trusted CA certificates.
They periodically receives information from the OR about their status, inside signed messages.

Each server create a **directory** listing all active ORs and their attributes.

![[directory-server.png|525]]
> [!info]
> As of march 2025, a total of 10 directory servers and 8500 ORs.
>- Check directory servers: https://metrics.torproject.org/rs.html#search/flag:authority
>- Number of relays of the TOR network: https://metrics.torproject.org/networksize.html

DS together synchronise their directories to produce a **consensus directory**, updated hourly, which provides a consistent view of the TOR network
OPs can maintain a copy of the directory in cache, mostly to reduce traffic on directory servers.

If a DS is censored, clients can connect to a **bridge node**, which acts as a proxy to bypass censorship
	*The complete list of bridge nodes is **never fully published** to prevent censorship*

> [!caution] DS Weaknesses
> - Possible bottlenecks
> - Servers are public, they are vulnerable to censorship (hence the existence of bridges)
> - If managed by an attacker, can forward only ORs under its control, or can provide specific ORs to a client in order to track him


### Hidden services
*Also known as "Onion services"*
#### Hidden services concept

An ingenious system which allows users to host services (e.g., websites, file servers) anonymously, hiding both the **server's IP address** and the **client's IP address**. This is achieved through a unique addressing system and a multi-step process for establishing connections.

Hidden services use `.onion` addresses, which are derived from the server's public key. The address is a **base32-encoded hash** of the server's public key, ensuring it is unique and cryptographically tied to the service.
	*The `.onion` TLD is a special-use domain name which is not resolved by DNS servers, but is still reserved by IANA specifically for the TOR network (RFC 7686)*

#### Setup

When the server comes online, it picks three nodes called **Introduction Points**, establish a **circuit** towards them and **forwards them their public key**. Those introduction points now awaits for incoming client requests. 
	*So, there is a circuit between the server and each IntPt, with three hops for each, just like we've seen above for clients-to-server circuits.*

Now, the server creates and sign a **Hidden Service Descriptor**, containing the server's public key (authentication purpose) and the IP addresses of all Introduction points it selected.
The descriptor is then published into the **Distributed Hash Table (DHT)**.
	*As its name implies, the DHT is distributed among several nodes, which means each node contains a fraction of information for the entire table. This information can be localised by the client thanks to an index allowing him to locate the node hosting the specific part of the hash table he needs.
	The keys of this hash table are (digests of) onion addresses. This allows us to find the descriptor based on the onion address (simple key-value pairs).*

![[hidden-service-setup.png|350]]
> [!info]
> Here we consider that `xyz` is the hash.

#### Rendez-vous mechanism

For now, the server (and the introduction points) are just waiting for incoming clients.
If a client wants to access our hidden service (and knows the onion domain, that he got somewhere idk), it can ask the DHT for the descriptor and obtain information about the Introduction Points to contact.

Client now choses both a random IntPt among the three, and a random node on the TOR network which will be the **Rendez-vous Point (RP)**.
1. Client creates a circuit towards the RP, and transmits it a one time password of its own creation known as the **cookie** (unrelated to web browser cookies)
2. Client creates a circuit towards the IntPt of its choice, and sends him a message encrypted via the destination server's public key
   The message contains:
	- The address of the RP
	- The cookie
	- The first half of the DH parameters
3. The contacted IntPt forwards the message it received—from the client—to the server.
4. If the server accepts, it creates a circuit towards RP and sends the cookie (pair the client and the server), the second half of the DH parameters, and a hash of the secret key (authentication) to the RP
5. The RP now connects the client circuit with the server circuit
6. The client sends the `Relay Begin` cell on the full circuit

![[hidden-service-rendezvous.png|500]]
> [!info]
> Its the cookie that allows the RP to match the client and hidden service.

In case of an HTTP hidden service, the configuration is nothing too far-fetched and remain pretty standard. The only important thing is to never insert any piece of information which could disclose the identity or location of the machine.
	*Do not specify a `ServerAdmin`, etc*

#### Single onion

This is kind of like a trade-off between anonymity and performance for hidden services.
If you followed all along, you noticed that the circuit between the client and the hidden service is a total of 6 hops (3 on each side).
	*This is a major problem with TOR and especially with hidden services, it's extremely slow as the entire process takes quite a lot of time, and intermediate nodes can be all around the planet.*

As its name implies, single onion simply lowers the server-side hops from three to one.

> [!question] But what's the point of doing this instead of just using the clear website?
> It is possible for an adversary to sniff both ends of a normal TOR connection (correlation) and identify the client. 
> However this does not happen with hidden services as **the server is fully part of the TOR network** and never leaves it. 
> 	Single Onion Services still protect the server’s IP from the public internet (unlike clearnet hosting).
