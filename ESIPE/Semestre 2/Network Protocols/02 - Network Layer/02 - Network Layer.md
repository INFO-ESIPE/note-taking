[[Network Protocols]]
***
**Table of Contents**
```table-of-contents
```

****
## Refresher

### Network Layer

The network layer enables communication across heterogeneous networks by:
- Providing **addressing** (IPv4/IPv6) for device identification
- Implementing **routing** to find paths between networks
- Handling **interconnection** of diverse link-layer technologies (Ethernet, Wi-Fi, etc.)

Other protocols of this layer (ICMP, ARP, IGMP) assists in these tasks.

### IP Address

An IP address represents a machine or a group of machine on the Internet. An address is unique at a given moment (besides private IPs which we will talk about later on)
	*A networking card = An IP address*

Two types of IP addresses:
- IPv4: 32-bit, a total of 4.3 billion available addresses (exhausted since February 2011)
- IPv6: 128-bit, a total of 3,4\*10³⁸ available addresses


***
## IPv4
*Internet Protocol; RFC 791*
### Fundamentals

Composed of four bytes, where each part is written in decimal and separated by a dot
`11000001001101110010110011001000 -> 193.55.44.200`

Three types of IPv4 addresses:
1. **Unicast**: Theoretically uniquely identifies an interface on the Internet
	*byte1 < 224*
2. **Multicast**: Used to send the same packet to a group of interfaces
	*byte1 >= 224*
3. **Broadcast**: Sends a same packet to all interfaces on the network 


### CIDR
*Classless Inter Domain Routing; RFC 1467, 1993*

Its a method for allocating IP addresses for IP routing.
An IPv4 address is composed of two parts:
- The `n` first bits represents the **Net-id (or "prefix")**, which identifies the network
- The `32 - n` remaining bits represents the **Host-id**, which identifies the machine on the network

> [!example]-
> - `200.10.111.0/24`: The network is represented by the 24 first bits. So, only the last byte is usable to identify machines on this network. 
> 	For instance, machine 3 will have the address `200.10.111.3`
> - `210.64.0.0/11`: The network is represented by the 11 first bits  (`11010010 . 010`). So, the 21 remaining bits can be used to identify machines on this network.
> 	  For instance, `210.67.128.0` can be the address of a machine in this network.
>	```
>	11010010 . 010 00011 . 10000000 . 00000000
>	|____________| |_________________________|
> 	    Net-id               Host-id
>	```

#### Reserved addresses

Two of the available host addresses of a network are always reserved:
1. **Network address**: All bits of host-id equals 0
	`193.55.44.0/24` represents a network
2. **Broadcast**: All bits of host-id equals 1
	`193.55.44.255` is the broadcast address for the network of address `193.55.44.0/24` 
*This means, the smallest machine address is `network-address + 1`, and the largest machine address is `broadcast-address - 1`.*

Below are the main reserved address ranges:

| **Address block**  | **Address range**           | **Scope**       | **Description**                                                                                                 |
| ------------------ | --------------------------- | --------------- | --------------------------------------------------------------------------------------------------------------- |
| 0.0.0.0/8          | 0.0.0.0–0.255.255.255       | Host            | Used as a source address by a machine to find out its own IP address at start-up using DHCP                     |
| 10.0.0.0/8         | 10.0.0.0–10.255.255.255     | Private network | Used for local communications within a private network                                                          |
| 100.64.0.0/10      | 100.64.0.0–100.127.255.255  | Private network | Shared address space for CGN, more details in [[02 - Network Layer#Address translation with CGN\|this chapter]] |
| 127.0.0.0/8        | 127.0.0.0–127.255.255.255   | Host            | **Loopback** address, used for IPCs on a same machine                                                           |
| 172.16.0.0/12      | 172.16.0.0–172.31.255.255   | Private network | Used for local communications within a private network                                                          |
| 192.168.0.0/16     | 192.168.0.0–192.168.255.255 | Private network | Used for local communications within a private network                                                          |
| 224.0.0.0/4        | 224.0.0.0–239.255.255.255   | Internet        | In use for multicast                                                                                            |
| 240.0.0.0/4        | 240.0.0.0–255.255.255.254   | Internet        | Reserved for future use                                                                                         |
| 255.255.255.255/32 | 255.255.255.255             | Subnet          | Reserved for the "limited broadcast" destination address                                                        |

The three "private network" address blocks cannot route to the Internet without **Network Address Translation (NAT)**.

![[private-network.png|600]]

> [!info]- Before 1993 - Class addresses
> The following logic was used before the arrival of CIDR.
> IPs were distributed in 5 classes, and each had a prefix which was necessarily a multiple of 8 (`/8`, `/16`, `/24`), big waste of IPs...
> 
> | **Role**  | **Class** | **Range**                 | **Possible Networks** | **Possible Machines** |
> | --------- | --------- | ------------------------- | --------------------- | --------------------- |
> | Unicast   | A         | 1.0.0.0–126.255.255.255         | 126                   | 16 777 214            |
> | Unicast   | B         | 128.0.0.0–191.255.0.0     | 16 382                | 65 534                |
> | Unicast   | C         | 192.0.0.0–223.255.255.0   | 2 097 150             | 254                   |
> | Multicast | D         | 224.0.0.0–239.255.255.0   | 268 435 455           |                       |
> | Research  | E         | 240.0.0.0–247.255.255.255 |                       |                       |

#### Netmask

It is associated to a network address; it depends on its network prefix, and is used for routing process.
The netmask appears as an IP address, but:
- All net-id bits equals 1
- All host-id bits equals 0

> [!example]-
> - `200.10.111.0/24`:
>   In decimal: `255.255.255.0`
>   In binary:
>	```
>	11111111 . 11111111 . 11111111 . 00000000
>	|____________________________|   |______|
> 	           Net-id               Host-id
>	```
> - `210.64.0.0/11`:
>   In decimal: `255.224.0.0`
>   In binary:
>	```
>	11111111 . 11100000 . 00000000 . 00000000
>	|____________||_________________________|
> 	   Net-id             Host-id
>	```

When a router receives a packet, it performs a **bitwise AND** between the destination IP and the subnet mask to determine the network address.

> [!example]-
> - Machine `200.10.111.3` on a `/24`
>	```
> 	  1100 1000 . 0000 1010 . 0110 1111 . 0000 0011
>	& 1111 1111 . 1111 1111 . 1111 1111 . 0000 0000
>	_______________________________________________
> 	  1100 1000 . 0000 1010 . 0110 1111 . 0000 0000
>	```
> - Machine `210.67.128.0` on a `/11`
>	```
> 	  1101 0010 . 0100 0011 . 1000 0000 . 0000 0000
>	& 1111 1111 . 1110 0000 . 0000 0000 . 0000 0000
>	_______________________________________________
> 	  1101 0010 . 0100 0000 . 0000 0000 . 0000 0000
>	```


### Packet format

v4 packets are structured like so:

![[ipv4-header.png|600]]
- **Version** indicates the IP version (4 in our case)
- **IP Header Length** indicates the header length in 32-bits words
- **TOS** indicates the QoS required by the datagram 
- **Total Length** indicates the entire length of the packet (header + data) in bytes
- **Identification + Flag + FO** are used for fragmentation
- **TTL** indicates the packet lifetime.
	*This was initially in seconds, but it is now in hops.
	Each time a packet traverses a router, its TTL decreases by one. If it reaches 0, the packet is discarded and an ICMP "Time exceeded" message is sent back to the source. This is very useful for "traceroute".*

- **Protocol** is the protocol number the packet must be transmitted to (TCP=6, UDP=17, ICMP=1)
- **Header Checksum** detects errors in the header. If invalid checksum, a router destroys the packet
- **Source & Destination IP Address** contains IPs of end services (not intermediate nodes)
- **Options (+ padding)** to specify debugging, error logging...


### Routing

As we have seen above, the packet carries the source IP (machine which created the packet) and the destination IP (recipient machine). We know that routing is stateless and following a hop by hop logic, and this is done thanks to the **routing table** which maps the destination network to a next router.

#### Routing table

Mentioned in RFC 1156, it is present in every layer 3 equipment.
Minimal content:
- **Destination + Mask**: Identifies the target network (e.g., `192.168.1.0/24`). Combined with the **netmask**, it defines the range of IP addresses in the network
- **Next Hop (gateway)**: IP address of the next router to forward the packet to (or `*` for directly connected networks)
- **Interface**: Physical/virtual port used to send the packet (e.g., `eth0`, `wlan0`)
- **Cost**: Numerical value representing the "cost" of the route (e.g., hop count, bandwidth)
  Lower values indicate preferred paths.

#### Routing procedure

The routing process follows these steps:
1. **Extract Destination IP**: The router reads the destination IP from the packet header.
2. **Check Local Interface**:
    - If the destination IP matches one of the router’s interfaces, the packet is forwarded to the upper layer (e.g., TCP/UDP).
    - Otherwise, the router performs a **bitwise AND** between the destination IP and the subnet masks in the routing table, **starting with the longest prefix** (most specific routes)
3. **Match Routing Entry**:
    - **Local Network**: If the result matches a directly connected network (e.g., `192.168.1.0/24`), the router sends the packet directly to the destination via the linked interface
    - **Remote Network**: If the result matches a non-local network, the router forwards the packet to the **next-hop router** specified in the routing table
    - **No Match**: If no entry matches, the router discards the packet and sends an **ICMP Destination Unreachable** error to the source

#### Default route

Only Tier 1 routers maintain full routing tables for all networks. For smaller networks (LANs, Tier 2/3 ISPs), a **default route** simplifies routing.

> [!example]- Example of routing table with default route
> | **Destination** | **Netmask**   | **Next Hop** | **Interface** |
> | --------------- | ------------- | ------------ | ------------- |
> | 192.168.1.0     | 255.255.255.0 | *            | eth0          |
> | 0.0.0.0         | 0.0.0.0       | 192.168.1.1  | eth0          |

**Logic**: A `0.0.0.0/0` destination/netmask (`0.0.0.0` mask) matches **any IP address**.

A logical AND between any destination address and mask `0.0.0.0` always return `0.0.0.0`, which means the packet is necessarily routed to the next router (in this case, `192.168.1.1`.
	*In this case, `192.168.1.1` is known as the default gateway. This avoids the need to manually configure routes for every possible destination...
	Typically, the default gateway is the ISP's router or firewall.*

> [!example]- (Non optimal) routing table for private networks interconnection
> ![[routing-example.png|650]]
> 
> R1's routing table:
> 
> | **Destination** | **Netmask**   | **Next Hop** | **Interface** |
> | --------------- | ------------- | ------------ | ------------- |
> | 192.168.4.0     | 255.255.255.0 | \*           | eth2          |
> | 192.168.3.0     | 255.255.255.0 | \*           | eth1          |
> | 192.168.1.0     | 255.255.255.0 | 192.168.4.2  | eth2          |
> | 192.168.2.0     | 255.255.255.0 | 192.168.4.2  | eth2          |
> | 172.16.0.0      | 255.255.0.0   | 192.168.4.2  | eth2          |
> | 10.0.0.0        | 255.0.0.0     | 192.168.4.2  | eth2          |
> | 0.0.0.0         | 0.0.0.0       | 192.168.4.2  | eth2          |
> 

An optimised routing table is one with as few lines as possible (fewer lookup operations, decreasing resource consumption).

> [!example]- Optimal routing table alternative for previous example
> In R1's table, networks `192.168.1.0/24`, `192.168.2.0/24`, `172.16.0.0/16` and `10.0.0.0/8` are all accessible from the same router (`192.168.4.2`) and interface (`eth2`).
> So, routes sharing the same next-hop gateway and egress interface can be aggregated into broader network ranges:
> 
> | **Destination** | **Netmask**   | **Next Hop** | **Interface** |
> | --------------- | ------------- | ------------ | ------------- |
> | 192.168.4.0     | 255.255.255.0 | \*           | eth2          |
> | 192.168.3.0     | 255.255.255.0 | \*           | eth1          |
> | 0.0.0.0         | 0.0.0.0       | 192.168.4.2  | eth2          |

> [!example]- Usage of R1's routing table
> Router R1 receives a packet from Host A (`192.168.3.10`) to Host B (`172.16.0.10`).
> - Checks if the destination is on the local network:
>   `172.16.0.10 & 255.255.255.0 = 172.16.0.0`, this does not match `192.168.3.0`.
> - Tries routing table lines one by one:
> 	- `172.16.0.10 & 255.255.255.0 = 172.16.0.0` ≠ 192.168.4.0
> 	- `172.16.0.10 & 0.0.0.0 = 0.0.0.0`, it (obviously) fits
> - Packet leaves by `eth2` and is sent to next hop `192.168.4.2` (R4)
> - Retrieve MAC address corresponding to `192.168.4.2` via ARP (more details on this later on) to fill the ethernet header
> 
> ![[routing-optimised-example.png|650]]

#### Configuration

**IP Address Assignment:**
- **Static Allocation:** Manual configuration (prone to human error, ideal for servers)
- **Dynamic Allocation (DHCP):** Automatic assignment by a DHCP server (common for end-user devices)

**Routing Table Management:**
- **Static Routing:** Manual entries (simple, but inflexible for large networks)
- **Dynamic Routing:** Protocols like RIP, OSPF, or BGP automatically update routes (scalable, resilient, but resource-intensive)


### DHCP
*Dynamic Host Configuration Protocol; RFC 145*

Application-layer protocol which **automatically assign IP configurations (IP, subnet mask, gateway, DNS) to hosts**.
	*UDP (Client port 68, Server port 67)*

> [!example]- Address allocation with DHCP
> ![[dhcp-lease.png|600]]
> - **DHCP discover** to localise all available DHCP servers
> - **DHCP offer** identifies the server and provide an IP configuration to the client
> - **DHCP request** tells which server we chose and confirm the received address
> - **DHCP ack** confirms address allocation

It attributes everything in two primary ways:
1. **Static IP Assignment (Reservation):** A specific IP address is **permanently bound** to a device's MAC address. The device always receives the same IP address, even after reboots or lease expirations.
    *Configured on the DHCP server (e.g., `reserve 192.168.1.10 for MAC AA:BB:CC:DD:EE:FF`)*

2. **Dynamic IP Assignment (Lease):** IP addresses are **temporarily assigned** from a predefined pool (e.g., 192.168.1.100–192.168.1.200).
    - Leases have a **configurable duration** (e.g., 24 hours) managed by the DHCP server
    - Clients automatically renew leases (via `DHCPREQUEST` messages) to retain their IP addresses
    - Un-renewed leases expire, returning the IP to the pool for reuse

> [!example]- isc-dhcp-server daemon configuration
> ![[dhcp-daemon.png|600]]
> 
> ```bash
> subnet 172.16.0.0 netmask 255.255.0.0 {      # Definition of the target network
>     option routers 172.16.0.254;             # Client devices' default gateway
>     option domain-name "network.edu";        # Domain name for client devices
>     option domain-name-servers 172.16.2.200, 193.55.44.126; # DNS servers for clients
>     
>     range 172.16.2.1 172.16.2.10;            # Dynamic IP address pool (10 addresses)
> 	
>     host pc1 {                               # Configuration for fixed address of client pc1
>         hardware ethernet 00:01:02:03:04:05; # MAC address of pc1
>         fixed-address 172.16.1.10;           # Reserved IP address for pc1
>     }
> 	
>     host pc2 {                               # Configuration for fixed address of client pc2
>         hardware ethernet 00:0A:0B:0C:0D:0E; # MAC address of pc2
>         fixed-address 172.16.1.20;           # Reserved IP address for pc2
>     }
> }
> ```


### Subnetting

We can divide a network into subnetworks. We reserve most significant bits of the host-id to reference those subnetworks.

> [!example]- Subnetting `200.10.111.0/24` into 3 subnetworks
> We must reserve at least two bits from the host-id (which gives us four subnetworks in total, 62 machines per subnetwork):
> ![[subnetting.png|600]]


### Fragmentation

As explained in the very first lecture, internet is about connecting heterogeneous networks which does not rely on the same protocols for the physical and data link layer. Differences on the later are abstracted by IP.
This is because the data link layer has a great variety of standards, distinguished by:
- Their **addressing schemes**; MAC address for ethernet & WiFi, VPI/VCI for Asynchronous Transfer Mode (ATM) networks...
- Their **Maximum Transfer Unit (MTU)**, so the maximum accepted size of the SDU for our IP packet

The solution for the later issue is **IP fragmentation and reassembly**:
Routers break down (fragment) our IPv4 packet when entering a network with an inferior MTU, and let the recipient reassemble pieces.
	*This is a costly operation (both in time and resources); exploitable for attacks, which is why IPv6 routers does not use it*

![[ESIPE/Semestre 2/Network Protocols/02 - Network Layer/fragmentation.png|675]]

Four header fields are used to achieve fragmentation:
- **Identification**: A common number for all fragments of a same packet
- **Fragment offset**: Position of fragment data in the original packet, **in 8-byte blocks**
- **Flag More Fragment**: Set to 1 in all fragments except the last
- **Flag Don't Fragment**: If set, packet must be dropped if fragmentation is required

> [!example]- 4000 bytes packet entering a 1500-MTU network
> ![[header-fragment.png|675]]

> [!info] Method for IPv6
> In IPv6, routers do not fragment packets. Instead, sources use ICMPv6 Packet Too Big messages to discover the MTU and adjust packet sizes accordingly.


### ARP
*Address Resolution Protocol; RFC 826. This one is technically more in the second layer but its debatable*

#### Concept

ARP maps **IP addresses** (network layer) to **MAC addresses** (data link layer) for local network delivery.
- **IP header destination:** Final recipient's IP address
- **Frame header destination:** Next-hop device's MAC address (router or end host)

#### Packet

![[arp-packet.png|600]]
- **Hardware type**, `0x0001` for Ethernet
- **Protocol type**, `0x0800` for IPv4
- **Hardware address length**, in bytes (6 for MAC Address)
- **Protocol address length**, in bytes (4 for IPv4)
- **Operation code**, respectively 1 and 2 for request and answer
- **Source hardware**, MAC address of packet sender
- **Source protocol**, IP address of packet sender
- **Target hardware**, MAC address of packet recipient if answer, 0 if request
- **Target protocol**, IP address of packet recipient

#### Resolve

It follows this scheme:

![[arp-resolve.png|600]]
> [!info]
> `FF:FF:FF:FF:FF:FF` is the broadcast MAC address


### TTL

Value initialised by the operating system, between 1 and 255.
	*typically 64 for Linux, 128 for Windows, 255 for routers*

As explained before, the TTL is decremented at each router hop. If TTL=0, packet is discarded and an **ICMP Time Exceeded (Type 11)** message is sent to the source.

![[ttl.png|550]]

> [!question] What is the purpose of this?
> - Destroy a packet caught in a loop (possible if dynamic routing algorithm does not converge)
> - Discover the path followed by a packet via the `traceroute` command


### ICMP
*Internet Control Message Protocol; RFC 792*

#### Concept & packet

Encapsulated in IP, it completes the later by providing information/error messages on the network's state.
The nature of a message is mostly defined by two fields:
- **Type**, 22 of them in total
- **Code**, adding more details to the type field 
Key Message Types:

|Type|Code|Description|
|---|---|---|
|0|0|Echo Reply (Ping response)|
|3|3|Port Unreachable|
|8|0|Echo Request (Ping)|
|11|0|TTL Exceeded|

#### Use cases

A very common use case is the `ping` (Packet INternet Groper) command, which tests connectivity and measure latency (RTT) with a distant interface.
	*The content of the request and response are the same*

> [!example]- Ping command
> ![[ping.png|450]]
> ```bash
> derelict@pc:~$ ping 200.0.0.1
> PING 200.0.0.1 (200.0.0.1) 56(84) bytes of data.
> 64 bytes from 200.0.0.1: icmp_seq=1 ttl=49 time=111 ms
> 64 bytes from 200.0.0.1: icmp_seq=2 ttl=49 time=88.9 ms
> 64 bytes from 200.0.0.1: icmp_seq=3 ttl=49 time=93.7 ms
> --- 200.0.0.1 ping statistics ---
> 3 packets transmitted, 3 received, 0% packet loss, time 5ms
> rtt min/avg/max/mdev = 88.902/97.762/110.643/9.323 ms
> derelict@pc:~$ ^C
> ```

Another very common use case we mentioned above is the `traceroute` command, which displays all traversed interfaces to reach a destination.

> [!example]- Traceroute command
> ![[traceroute.png|550]]
> ```bash
> derelict@pc:~$ traceroute 200.0.0.1
> traceroute to 200.0.0.1 (200.0.0.1), 30 hops max, 60 byte packets
> 1 100.0.0.10 0.786 ms 0.913 ms 1.592 ms
> 2 50.0.0.10 2.188 ms 2.670 ms 2.796 ms
> 3 200.0.0.1 3.318 ms 3.387 ms 3.967 ms
> derelict@pc:~$
> ```


### Address translation with NAT

#### NAT
*Network Address Translation*

As briefly mentioned earlier, addresses in these three ranges requires NAT to go beyond the LAN and become routable on the internet:
- `10.0.0.0/8` (10.0.0.0—10.255.255.255)
- `172.16.0.0/12` (172.16.0.0—172.31.255.255)
- `192.168.0.0/16` (192.168.0.0—192.168.255.255)

NAT is here to hide the internal organisation of the network. It does so by **replacing the address of packets leaving the private network for the Internet with a public address**. Two ways of doing it:
- **Basic NAT**: Maps private IPs to a pool of public IPs (rarely used)
- **Network Address Port Translation (NAPT)**: Maps multiple private IPs to a single public IP using port translation

![[nat.png|600]]
> [!info]
> This translation requires to re-calculate checksums of IP, TCP/UDP and ICMP headers

> [!question] What if the same port is used by two local machines?
> As you can see on the figure below, two applications using the same port does not permit to retrieve the original machine, as the **quintuplet is not unique** anymore... 
> 
> ![[nat-problem.png|600]]

#### Port translation

The solution to the previous problem is port translation. The NAT service has an internal **translation table** which maps the port used by a local machine (LAN side) with its translated version (Public side).

![[port-translation.png|600]]

#### Port mapping

Also known as **static NAT**, it Expose internal services via manual port forwarding. This is used to make an internal server accessible from the outside:

![[port-mapping.png|600]]

#### ICMP and NAT

ICMP lacks port numbers, requiring NATs to use:
1. **ICMP Query Messages (e.g., Ping):** Track **ICMP Identifier** field (analogous to ports).
2. **ICMP Error Messages:** Contain the original error-ed packet's IP header in their payload.
   NAT extracts original source IP/port from this payload for reverse translation.

Our NAT gateway will also have to maintain state for both outgoing ICMP requests (Identifier mapping) and incoming ICMP errors (Original packet header analysis):

![[nat-icmp.png|600]]


### Address translation with CGN
#### CGN
*Carrier-grade NAT, a.k.a. Large Scale NAT, NAT444*

CGN allows an ISP to share a same public IPv4 between several clients (mostly to make up for the lack of IPv4 addresses...).
ISPs must use the **shared address space** `100.64.0.0/10` (RFC 6598) for this purpose.
	*Packets from this IP range cannot be routed over the Internet*

![[cgn.png|600]]

#### CGN with Dynamic port translation

CGN dynamically allocates unique ports to differentiate clients (similar to NAPT).

![[cgn-port.png|600]]
> [!caution] Disadvantages
> - Problem if too many connections are opened as it quickly exhausts the 65535 available ports
> - Difficult to log

#### CGN with Port range assignment

Each client has its own range of ports, which limits the amount of simultaneous connections from a client.
Appears as a more suitable choice, but predictable port allocation increases vulnerability to targeted attacks.

![[cgn-range.png|600]]

#### Static port mapping with PCP
*Port Control Protocol; RFC 6887*

Allows clients to configure port forwarding through CGN.

![[gcn-pcp.png|550]]
> [!info]
> Requires PCP support on both CGN and client routers


### IP addresses management

IANA is the authority in charge of this, but delegates IP blocks to five **Regional Internet Registries (RIR)**.

![[rir.svg|600]]

RIRs then allocates those IPs to Local IR (LIR) or National IR (NIR), which are usually Internet service provides or telecom operators.
Organisations and individuals obtains their IP addresses from their ISP.
	*However, some historically large and important organisations (MIT, IBM, Xerox...) have been allocated addresses directly by the RIRs, without going through an ISP.*

All allocations are documented by [IANA](https://www.iana.org/numbers)


***
## Addressing and Dynamic Routing in WANs

### AS
*Autonomous System; RFC 1930*

An AS is a collection of networks under a single administrative domain, governed by a unified routing policy. It is identified by a 2 or 4 bytes **AS Number (ASN)**.
	*Assignment follows the procedure describes above for IP management: IANA -> RIR -> ISP/Telecom*

> [!example]- ASN examples
> AS2200 (Renater), AS12322 (Free), AS3215 (Orange)


### IGP, EGP
*Interior Gateway Protocol; Exterior Gateway Protocol*

IGPs (e.g., RIP, OSPF) manages routing **within** an AS, maintaining up-to-date topology information by correctly completing routing tables.

> [!info]- IGP examples
> - **RIP (Routing Information Protocol, RFC 1058)** is a distance-vector protocol using hop count as a metric, and periodically broadcasts routing tables to neighbours,
>   However, it has limited scalability (max 15 hops)
> - **OSPF (Open Shortest Path First, RFC 2328)** is a link-state protocol using Dijkstra’s algorithm. It floods Link-State Advertisements (LSAs) to build a topology map.
>   Supports hierarchical routing with areas

EGPs (e.g., BGP), on the other hand, are here to exchange routing information (address ranges) **between** ASes, so each AS can learn others' addresses.

> [!info]- EGP example
> - **BGP (Border Gateway Protocol, RFC 4271)** is a path-vector protocol which advertises routes as sequences of ASes. Those routers are selected based on attributes like AS path length, local preference, and MED (Multi-Exit Discriminator).

![[igp-egp.png|600]]


### Route Aggregation and VLSM
*Variable Length Subnet Masking*

The aforementioned mechanisms will heavily increase the amount of lines in the routing tables (which will increase latency and mem/CPU consumption).
A mechanism of **route aggregation (supernetting)** is here to reduce routing table size by merging contiguous prefixes.

The logic is: IANA attributes contiguous blocks of addresses to RIRs. The later must keep the contiguous aspect when re-distributing it to ISPs.

![[supernetting.png|500]]
> [!caution] Limitation
> Historically-attributed static addresses disturb the contiguity of the ranges

> [!example]- Supernetting example
> ![[supernetting-example.png|650]]


Now, VLSM is here to allocate subnets hierarchically to minimise wasted addresses:
1. Assign the largest subnet first
2. Allocate smaller subnets from remaining contiguous blocks

> [!example]- ISP owns `200.100.128.0/19`
> So, we begin with the following:
>```
>11001000 . 01100100 . 100 00000 . 0000 0000
> |_______________________| |_______________|
>         Net-id                Host-id
>```
>
> Now, client A needs 2000 addresses: 11 bits host-id -> `/21` prefix
> 	Range from `200.100.128.1 `to `200.100.135.254`
> Client B needs 1000 addresses: 10 bits host-id -> `/22` prefix
>	Range from `200.100.136.1` to `200.100.139.254`
> Client C needs 100 addresses: 7 bits host-id -> `/25` prefix
>	Range from `200.100.140.1` to `200.100.140.127`
> 
> The best way to visualise this is as a tree:
> ![[vlsm-tree.png|600]]


### `/31` prefix
*RFC 3021, 2000*

This addressing scheme only gives room for two addresses: Network and broadcast address. Both addresses are usable for interface assignments, unlike larger subnets where these are typically reserved.
In general, this is used for **point-to-point links**, such as direct connections between two routers.
	*Purpose is to avoid wasting addresses, dynamic routing protocols are unaffected as long as both endpoints support RFC 3021 (which is the case for most modern Operating Systems)*

![[thirtyone.png|600]]


### Dynamic routing

Protocols fulfilling this purpose are based algorithms searching for the "best path". Two large kinds:
1. **Distance vector protocols**, shortest path algorithm (e.g., Belman-Ford)
	Periodic transmission of known routes to neighbouring routers → update if route is shorter or new 
		*[[02 - Network Layer#IGP, EGP|RIP]], RIPv2, IGRP...*

> [!example]- Routing table construction
> ![[vector-rtable.png|600]]

> [!example]- Topology change
> ![[vector-topology-change.png|600]]

2. **Link state protocols**, usually relies on Dijkstra
	Attributes a metric to links (depending on robustness, capacity...).
	Periodic transmission of the state of links between routers → establishment of a complete map of the topology → calculation of the ‘best’ routes (lowest cost)
		*[[02 - Network Layer#IGP, EGP|OSPF]], IS-IS...*

> [!example]- Neighbourhood discovery
> ![[link-neighbourhood.png|600]]

> [!example]- Topology information propagation
> ![[link-topology.png|600]]

> [!example]- Common topology and routing tables
> ![[link-routing.png|600]]

> [!example]- Topology change
> ![[link-topology-change.png|600]]

Comparison:

| **Criteria**                  | **Distance-Vector (e.g., RIP)**       | **Link-State (e.g., OSPF)**                |
| ----------------------------- | ------------------------------------- | ------------------------------------------ |
| **Convergence Speed**         | Slower (relies on periodic updates)   | Faster (uses triggered updates)            |
| **Transient Loops**           | Possible (split-horizon mitigates)    | None (loop-free via SPF algorithm)         |
| **Bandwidth Usage**           | Higher (full routing table exchanges) | Lower (sends incremental updates)          |
| **Resource Consumption**      | Lower CPU/memory (small tables)       | Higher CPU/memory (maintains topology)     |
| **Implementation Complexity** | Simpler (easy to configure)           | More complex (requires topology setup)     |
| **Hierarchical Support**      | Not natively supported                | Supported (e.g., OSPF areas, IS-IS levels) |


***
## IPv6
*Internet Protocol, Version 6; RFC 2460, 8200*

Introduced for two reasons:
- Remedy IPv4 address exhaustion
- Improve weak points of IPv4 (fragmentation, QoS, security...)

Transition form IPv4 to IPv6 is still ongoing, with more or less success on specific infrastructures

![[ipv6-transition.png|600]]
> For April 2023

IPv6 are managed similarly to IPv4 (IANA, RIR).


### IP Format

Address coded on 16 bytes (128 bits), and follows those guidelines:
- **Grouping**: Bytes are grouped into **four hexadecimal digits** (two bytes per group), separated by colons (`:`).
- **Prefix Notation**: The prefix length (CIDR notation) is appended with a `/`, similar to IPv4.

>Here's an example of an IPv6 address with a `/64` prefix:

```
0010 0000 0010 0101  1010 1011 0000 0001  0000 1100 0010 0100  0001 0000 0100 1010  0000 0000 0000 0000  0000 0000 0000 0000  0001 0011 1101 1110  0001 1111 0000 0000
2025               : AB01               : 0C24               : 104A               : 0000               : 0000               : 13DE               : 1F00               /64
|_________________________________________________________________________________| |________________________________________________________________________________|
								    Net-id                                                                             Host-id
```

However, an IPv6 can be abbreviated:
- **Leading Zeros**: Omit leading zeros in each group.
- **Consecutive Zeros**: Replace one or more consecutive groups of zeros with `::`.
	*Note that this last shortcut can only be used once per address*

> [!example]- Abbreviate the IPv6 above
>```
>   2025 : AB01 : 0C24 : 104A : 0000 : 0000 : 13DE : 1F00 / 64
>= 2025 : AB01 :  C24 : 104A ::              13DE : 1F00 / 64
>```


### Scope

Three main entities:
3. **Node**: Any device on the network (e.g., host, router, server)
4. **Link**: A collection of interfaces that can communicate at the data link layer without traversing a router
	*(e.g., Ethernet/WiFi, PPP)*
5. **Site**: An interconnected set of subnets within an organisation.

![[ipv6-scope.png|600]]

| **Type**           | **Usage**                                                                                                | **Prefix**                     |
| ------------------ | -------------------------------------------------------------------------------------------------------- | ------------------------------ |
| Link-Local Unicast | Identifies an interface on a local network (link)<br>Not routable beyond the local network<br>           | `FE80::/10`                    |
| Global unicast     | Identifies an interface globally on the Internet<br>Routable across the entire IPv6 internet             | `2000::/3`                     |
| Multicast          | Represents a group of interfaces<br>Packets sent to a multicast address are delivered to all members<br> | `FF00::/8`                     |
| Anycast            | Assigned to multiple interfaces<br>Packets are routed to the nearest interface with this address<br><br> | Shared with <br>Global Unicast |
> [!info]
> There is no broadcast address for IPv6

> [!info]- Multicast prefixes
> | **Prefix**                  | **Scope** | **Meaning**                                            |
> | --------------------------- | --------- | ------------------------------------------------------ |
> | `FF01::1`                   | Node      | All node addresses on the local device                 |
> | `FF01::2`                   | Node      | All router addresses on the local device               |
> | `FF02::1`                   | Link      | All node addresses on the local network                |
> | `FF02::2`                   | Link      | All router addresses on the local network              |
> | `FF02::1:2`                 | Link      | All DHCPv6 servers/relays                              |
> | `FF02:0:0:0:0:1:FF00::/104` | Link      | Solicited-node multicast (used for neighbor discovery) |
> | `FF02::9`                   | Link      | RIPng routers                                          |
> | `FF05::2`                   | Site      | All routers within the organization                    |
> | `FF05::1:3`                 | Site      | All DHCPv6 servers within the organization             |
> 

> [!info]- Unspecified addresses
> | **Type**            | **Notation** | **Remarks**                       |
> | ------------------- | ------------ | --------------------------------- |
> | Unspecified address | `::/128`     | Same usage as `0.0.0.0` in IPv4   |
> | Loopback            | `::1/128`    | Same usage as `127.0.0.1` in IPv4 |
> 
> 

#### Interface Identifier

**64-bit value** that uniquely identifies an interface when combined with a network prefix.

**Original Method (Deprecated)** - Derived from the **MAC address**:
      1. Invert the _Universal/Local (U) bit_ (7th bit of the first byte)
      2. Insert `FF:FE` between the 3rd and 4th bytes of the MAC.    
        - **Example**: `00:1C:98:3A:6D:25` → `021C:98FF:FE3A:6D25`

> [!caution] Issue
> This method gives too much room for device tracking, as the identifiers are predictable...
   
**RFC 7217 (Secure Method)** - Generates a **non-reversible interface identifier** using:
    - A cryptographic hash (SHA-1) of the MAC address
    - Network prefix
    - Secret key (optional).

> [!success] Better

#### Link-Local address

Local scope, identifies an interface on a link. Used for auto-configuration of the machine (global unicast address, neighbourhood & router discovery...)
**Format**
- **Prefix**: `FE80::/64` (first 64 bits).
- **Interface Identifier**: Last 64 bits (derived from MAC or RFC 7217).

> [!example]- For MAC Address `00:1C:98:3A:6D:25`
> Interface Id: `00:1C:98:3A:6D:25` -> ``21C:98FF:FE3A:6D25``
> Link-Local Address: `FE80:: 21C:98FF:FE3A:6D25 / 64`
> 
> ![[link-local.png|600]]

This local address can be auto-configured, thus enabling communication within the local network segment (link), three steps:
1. Generate interface-id
2. Address formation (combine link-local prefix and interface-id like described above)
3. **Uniqueness check**: Host sends an **ICMPv6 Neighbour Solicitation** message to the multicast address `FF02::1` (all nodes on the link).
	If no response (Neighbour Advertisement) is received, the address is considered unique

![[link-local-config.png|550]]


#### Global unicast address

Uniquely identifies an interface over the internet. Composed of three parts:
1. **Global routing prefix**: 48 bits, assigned by a RIR or ISP. Begins by bits `001`
2. **Subnet-id**: 16 bits to segment the network, managed by the organisation
3. **Interface Identifier (interface-id)**, 64 bits

![[global-address.png|700]]

> [!example]- MAC `00:1C:98:3A:6D:25` ; global prefix `2018:77:1 ::/48` ; subnet-id `1`
> `2018:77:1: 1 :21C:98FF:FE3A:6D25 / 64`

Again, this can be auto-configured for global internet communication, four steps:
1. **Prefix Acquisition**: Routers broadcast the global prefix (e.g., `2001:db8::/48`) via **ICMPv6 Router Advertisement (RA)** messages
2. **Address Formation**: Combine the global prefix, subnet ID (assigned by the organisation), and interface ID (like described above)
3. **Uniqueness Check**: Host performs **Duplicate Address Detection (DAD)** via multicast *Neighbour Solicitation*
4. **Source Address**: All configuration messages (e.g., DAD, router solicitation) use the **link-local address** as the source

![[global-config.png|600]]

#### Anycast address

As mentioned earlier, those addresses are here to deliver packets to the **nearest** instance of a service (e.g., DNS, CDN nodes).
Services shares the same address across multiple interfaces, and the routing protocol select the "closest" instance based on metrics (e.g., hop count).

![[anycast.png|650]]


### ICMPv6

Critical for most of IPv6, it combines—and replaces—IPv4's IGMP, ICMP and ARP.
It keeps being used for error handling, but also:
- **Neighbour Discovery (ND)**, Allows to resolve IPv6 to MAC, and detect duplicate addresses
- **Path MTU Discovery (PMTUd)**, which lets the source dynamically determines the maximum MTU along a path

ND and Neighbour Advertisement (NA) allows ICMPv6 to completely replace ARP.


### Modes of configuration

IPv6 supports two methods for address configuration:
1. **Stateless Address Autoconfiguration (SLAAC)**
    The OS automatically configures addresses at startup **without a central server**        
    Uses **ICMPv6 messages** for:
        - **Router Discovery**: Routers periodically send *Router Advertisement (RA)* messages containing the global prefix
        - **Duplicate Address Detection (DAD)**: Hosts verify address uniqueness via *Neighbour Solicitation* multicast messages
        - **DHCPv6 Optional**: Additional parameters (domain name, DNS servers) can be obtained via DHCPv6 without relying on it for IP assignment

2. **Stateful Configuration (DHCPv6)**
    A **DHCPv6 server** assigns all configuration parameters, including:
        - IPv6 addresses
        - Domain name, DNS servers, and other network settings
        - Used when centralised control over address allocation is required


