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

| **Address block**  | **Address range**           | **Scope**       | **Description**                                                                             |
| ------------------ | --------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| 0.0.0.0/8          | 0.0.0.0–0.255.255.255       | Host            | Used as a source address by a machine to find out its own IP address at start-up using DHCP |
| 10.0.0.0/8         | 10.0.0.0–10.255.255.255     | Private network | Used for local communications within a private network                                      |
| 127.0.0.0/8        | 127.0.0.0–127.255.255.255   | Host            | **Loopback** address, used for IPCs on a same machine                                       |
| 172.16.0.0/12      | 172.16.0.0–172.31.255.255   | Private network | Used for local communications within a private network                                      |
| 192.168.0.0/16     | 192.168.0.0–192.168.255.255 | Private network | Used for local communications within a private network                                      |
| 224.0.0.0/4        | 224.0.0.0–239.255.255.255   | Internet        | In use for multicast                                                                        |
| 240.0.0.0/4        | 240.0.0.0–255.255.255.254   | Internet        | Reserved for future use                                                                     |
| 255.255.255.255/32 | 255.255.255.255             | Subnet          | Reserved for the "limited broadcast" destination address                                    |
The three "private network" address blocks cannot route to the Internet without **Network Address Translation (NAT)**.

![[private-network.png|600]]

> [!info]- Before 1993 - Class addresses
> The following logic was used before the arrival of CIDR.
> IPs were distributed in 5 classes, and each had a prefix which was necessarily a multiple of 8 (`/8`, `/16`, `/24`), big waste of IPs...
> 
> | **Role**  | **Class** | **Range**                 | **Possible Networks** | **Possible Machines** |
> | --------- | --------- | ------------------------- | --------------------- | --------------------- |
> | Unicast   | A         | 1.0.0.0–126.0.0.0         | 126                   | 16 777 214            |
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

As explained in the very first lecture, internet is about connecting heterogeneous networks which does not rely on the same protocols for the physical and data link layer (done by IP).
This is because the data link layer has a great variety of standards, distinguished by:
- Their **addressing method**; MAC address for ethernet & WiFi, VPI/VCI for Asynchronous Transfer Mode (ATM) networks...
- Their **Maximum Transfer Unit (MTU)**, so the maximum accepted size of the SDU for our IP packet

The solution for the later issue is **IP fragmentation and reassembly**:
We break down (fragment) our packet when entering a network with an inferior MTU, and let the recipient reassemble pieces.
	*This is a costly operation (both in time and resources); exploitable for attacks, which is why IPv6 routers does not use it*

![[ESIPE/Semestre 2/Network Protocols/02 - Network Layer/fragmentation.png|675]]

Four header fields are used to achieve fragmentation:
- **Identification**: A common number for all fragments of a same packet
- **Fragment offset**: Position of the fragment in the original packet's SDU
- **Flag More Fragment**: 1 if the current fragment isn't the last
- **Flag Don't Fragment**: 1 if fragmentation is not allowed

> [!example]- 4000 bytes packet entering a 1500-MTU network
> ![[header-fragment.png|675]]


### ARP
*Address Resolution Protocol; RFC 826*

