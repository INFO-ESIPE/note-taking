[[Network Infrastructures]]
***
**Table of Contents**
```table-of-contents
```

****
**Note:** This lecture will focus on LANs using ethernet ((IEEE 802.3). For wireless networks (e.g., IEEE 802.11), see [[02 - Wireless Networks|the dedicated lecture]].
***
## Refresher

### LAN?

A Local Area Network is a private network typically spanning a building or campus (up to a few kilometres). Devices communicate via the **data link layer** (Layer 2).

![[lan-osi-tcpip.png|600]]

### IEEE model

This model divides the link layer into two sublayers:
1. **LLC (Logical Link Control, IEEE 802.2)**: Communication management, interface with upper layers
	*needed by some LAN management protocols, like Spanning Tree*
2. **MAC (Medium Access Control)**:
	- **Access method** (e.g., CSMA/CD): distribution of the bandwidth between the connected devices
	- Local addressing/local scope: exclusively used inside the LAN to route frames

> [!info]- About LLC functionalities
> Historically, this was used for multiplexing different network layer protocols (e.g., IP, IPX)
> Nowadays, protocol are identified by the « type » field of the Ethernet frame.

### MAC Address

A **48-bit (6-byte)** identifier for network interfaces, formatted as `XX:XX:XX:XX:XX:XX` or `XX-XX-XX-XX-XX-XX`.
Structure:
- **OUI (Organisationally Unique Identifier)**: First 3 bytes, assigned by IEEE to vendors
	- **I/G bit** (LSB of 1st byte): `0` = unicast, `1` = multicast/broadcast
	- **U/L bit** (2nd LSB of 1st byte): `0` = globally unique, `1` = locally administered
- **Card serial number**: Last 3 bytes, assigned by the vendor

![[mac-address-oui.png|400]]

> [!tip]
> Convenient websites can find the vendor for you if you paste it (e.g., [maclookup](https://maclookup.app/search/result?mac=CE%3AC0%3A73), [macaddress_io](https://macaddress.io/))
> OUIs are directly accessible [on the IEEE website](https://standards-oui.ieee.org/). Keep in mind that some OUIs are hidden or withheld from public databases for several reasons:
> - Confidentiality request
> - Specialised equipment (industrial, military, government, prototypes/experimental)
> - IEEE policy allowing a vendor to pay some money to stay private
> - Limit possibilities for an adversary to fingerprint the device vendor and exploit vulnerabilities 

Broadcast address is `FF:FF:FF:FF:FF:FF`. Multicast addresses are identified by reserved OUIs:

| **OUI**             | **Role**                                        |
| ------------------- | ----------------------------------------------- |
| `01:00:5E:...`      | IPv4 multicast                                  |
| `33:33:`            | IPv6 multicast                                  |
| `01:80:C2:00:00:00` | Spanning Tree Protocol                          |
| `01:80:C2:00:00:01` | Pause frame (flow control in switched Ethernet) |


***
## IEEE 802.3

### Fundamentals

"Ethernet" was developed by Xerox PARC in the 70s, and inspired the interoperable "IEEE 802.3" standard. It only applies to the physical and MAC layer.
Ethernet defines **Layer 1 (physical)** and **Layer 2 (MAC)** standards.

![[ethernet-layers.png|400]]

Bit rates can go from 1 Mb/s to 1,6 Tb/s, mostly depending on the medium used (coaxial/twinaxial, optical fibre, twisted pair) and the version (10BASE2, 10GBASE-T, 1000BASE-KX…).
Each version has differences in terms of:
- Physical layer (topology, bit rate, medium, coding…)
- MAC layer (half-duplex, full-duplex)

> [!info] Notation
> Examples:
> 
> | **Rate** | **Transmission** | **Mode** | **Clause equivalent** |
> | -------- | ---------------- | -------- | -------------- |
> | 10       | BASE             | 2        | IEEE 802.3a |
> | 10G      | BASE             | -T       | IEEE 802.3an |
> | 1000     | BASE             | -KX      | IEEE 802.3ap |
> 
> Rate is, by default, expressed in Mb/s. Letter "G" used to express in G/s instead.
> 
> Transmission can either be BASE (basebase) or BROAD (broadband, obsolete).
> 
> The medium is encoded like so:
> - T = twisted pair; F = optical fibre; K = backplane; S = short wavelength fibre; L = long wavelength fibre; C = twinaxial; K = backplane; X = 4B5B or 8B10B precoding; R = 64B66B...
> - Or maximum segment length of coaxial cable expressed in multiple of `100m` 

All 802.3 standards follows a same frame format:

![[ethernet-frame.png|650]]
- **Preamble** is here to synchronise clocks; `10101010`
- **Start Frame Delimiter (SFD)**, always `10101011`
- **Length/Type**:
	- **Type (EtherType)**: Identifies payload protocol (e.g., `0x0800` = IPv4, `0x86DD` = IPv6, `0x0806` = ARP).
	- **Length**: Frame size (if ≤ 1500 bytes).
- **Frame Check Sequence (FCS)** is a 4 bytes cyclic redundancy check (CRC) protecting addresses, length/type, data/padding field. It serves as a checksum to detect errors.


### Ethernet medias
#### Twisted pair

Used in LANs, mostly in a star topology. It contains four pairs of wires which may be foiled (FTP), shielded (STP), without protection (unshielded, UTP) or combined (U/FTP, F/UTP, S/FTP F/FTP).
It is used with RJ45 connectors.

![[twisted-pair.png|500]]
![[rj45.png|300]]

> [!example]- Categories
> | **Category** | **Bandwidth (MHz)** | **Used in**                            |
> | ------------ | ------------------- | -------------------------------------- |
> | 3            | 16                  | 10BASE-T                               |
> | 4            | 20                  | 10BASE-T                               |
> | 5            | 100                 | 100BASE-TX,<br>1000BASE-T              |
> | 5e           | 100                 | 1000BASE-T,<br>2,5GBASE-T,<br>5GBASE-T |
> | 6            | 250                 | 5GBASE-T,<br>10GBASE-T                 |
> | 6a           | 500                 | 10GBASE-T                              |
> | 8            | 1600—2000           | 25GBASE-T,<br>40GBASE-T                |
> 

#### Twinaxial cable (twinax)

Short-distance, high-speed links (e.g., server-to-switch in data centres).
Shielded, low latency, supports 10/25/40/100 Gbps (e.g., SFP+ DAC).

![[twinax.png|400]]

#### Optical Fibre

Can be used in both LANs (only to interconnect sites a few km away) and WANs (up to a 80 km distance).
Can be of two types:
1. **Single-Mode Fibre (SMF)**
    - **Core**: 9 µm
    - **Use**: Long-distance (up to 80 km), lasers
    - **Standards**: 1000BASE-LX, 10GBASE-LR
2. **Multimode Fibre (MMF)**
    - **Core**: 50/62.5 µm
    - **Use**: Shorter distances (up to 550m), LEDs
    - **Standards**: 1000BASE-SX, 10GBASE-SR


### PoE
*Power over Ethernet; IEEE 802.3af*

Purpose is to provide both electrical power and data to terminal equipment (e.g., Wireless Access Point, IP Camera, printers) through twister-pair ethernet.
The **Link Layer Discovery Protocol Media Endpoint Discovery** (LLDP-MED, IEEE 802.1AB) negotiates the power required or available.
	*Proprietary protocols such as Cisco CDP may also negotiate power*

Three techniques are defined for transmitting power over Ethernet cabling:

| **Mode**          | **Rates**   | **Max power** | **Mechanism description**                      |
| ----------------- | ----------- | ------------- | ---------------------------------------------- |
| **Alternative A** | 10—100 Mb/s | 25.50W        | Power and data on the same wires (1-2 and 3-6) |
| **Alternative B** | 10—100 Mb/s | 25.50W        | Power on the unused twisted pairs (4-5 ; 7-8)  |
| **4PPoE**         | >= 1 Gb/s   | 71W           | Power and data on all four pairs               |


### EEE
*Energy-Efficient Ethernet; IEEE 802.3az*

Set of enhancement which can be applied to pretty much any ethernet media we listed above in order to decrease by half the energy consumption from equipment in absence of
trafic in ≥ 100 Mb/s standards. 

Without EEE, equipment permanently send **Idle codes** to maintain synchronisation: high energy consumption!
EEE provides a sleep mode to equipment, considerably lowering the frequency of communication in this time span. 

![[eee.png|550]]


***
## Switched Ethernet

### From half-duplex to full-duplex

Until the 90s, we relied on half-duplex: no simultaneous transmissions because of collisions due to topology:
- **Bus**: Electrical signal are propagated on the wire
- **Star using a hub**: A bit incoming on a port is repeated on all other connected ports, no bufferisation

![[half-duplex.png|600]]

To solve collisions, half-duplex Ethernet used CSMA/CD protocol, inducing:
- Limitation of the distance between two stations
- Minimal frame length (64 bytes from destination address to FCS). 
	*Note that the later remains in all 802.3 standards*


### Full-duplex
*Or "Switched Ethernet"*

Unlike the legacy half-duplex, its full counterpart relies on **switches** or **bridges**, and necessarily follows a star topology.
	*The later interconnects segments of different technology, like ethernet + token bus + token ring, while switches are interconnecting segments of same technology*

Each host is connected by two wires (inside a same cable) to a dedicated port on the switch: One wire for transmission, other for reception.
Each port has two buffers: one stores received frames, the other stores sent frames.

![[switch.png|600]]

A **switching table** (or MAC address table) is here to associate each host MAC address to the switch port they are connected to.
This way, a received frame is stored in the output buffer of the port associated with the destination MAC address. It is only pushed to host when No other frame is being sent.
	*This prevents collisions from happening*

> [!info]- Switch synoptic
> ![[switch-synoptic.png|600]]

When receiving a frame, the switch process it in two steps:
1. **Reads the destination MAC address**
	- If broadcast (`FF:FF:FF:FF:FF:FF`), the frame is **copied to the output buffer of all the connected ports** (except the receiving port) 
	- Otherwise, looks for the port associated to this address in the table:
		- If port is known, we copy it to its output buffer
		- Otherwise, the frame is copied to the output buffer of all the connected ports (except the receiving port). This is known as the **flooding mechanism**

2. **Reads the source MAC address**. If its not known yet:
	- The source address is associated to the receiving port. This is known as the **learning mechanism**
	- A **timer** is triggered and updated each time a frame is received from this source MAC address. If it expires, the MAC/port association is removed from the table
		*Note that the timer duration is not standardised*

> [!example]- Simultaneous transmissions towards the same destination MAC address
> ![[simultaneous-dest.png|700]]

> [!example]- Learning and flooding
> ![[flooding.webm]]


### Switching process

The switching process (treatment inside the switch) influences both the treatment duration and the error rate. It is not standardised by IEEE and totally depends on the manufacturer. 
Two main processes:
1. **Cut-through (on-the-fly) switching**: The switch starts copying the frame to the output buffer as soon as the destination MAC address is completely received
2. **Store-and-forward switching**: The frame must have been completely received before being copied to the output buffer

| **Process**       | **Advantages**              | **Drawbacks**                               | **Fields of application**                                                 |
| ----------------- | --------------------------- | ------------------------------------------- | ------------------------------------------------------------------------- |
| Cut-through       | Reduced processing duration | Frames with errors may<br>be propagated     | Networks with a low<br>error rate and requiring<br>a very high throughput |
| Store-and-forward | Reduced error rates         | Increase delays, Higher CPU/mem consumption | Adapted to the ends of<br>the network (more<br>frequent errors)           |


### STP
*Spanning Tree Protocol; standardised in IEEE 802.1D*

#### Problem of loops

Fleshed-out topologies might include **loops**. Let's take the following scenario where a broadcasting frame is sent:
 1. SW1 copies it to all connected ports → SW2 received it on two ports, 1 and 2
 2. SW2 copies the frame received on port 2 to all connected port (including port 1), and the frame received on port 1 to all connected port (including port 2)
 3. Back to step 1... etc...

![[topology-loop.png|650]]

Here is the logic: **redundant links** creates loops, so, we want to only activate one link at a time and keep redundant ones as backups.
The Spanning Tree Algorithm (STA) superimposes a tree covering the network topology without a loop, which ends up **disabling ports causing the loop**.

![[spanning-tree.png|650]]

#### STP flavours

Three versions of the protocol:
1. **Spanning Tree Protocol (STP)**, IEEE 802.1D (1990)
	- Sends control frame (BPDU) every 2s
	- Converges in 30—50s.
2. **Rapid Spanning Tree Protocol (RSTP)**, IEEE 802.1w (1998)
	- Switches keep alternate paths
	- Excludes ports connected to stations from the execution
	- Converges in 6s (→ less numerous port states)
3. **Multiple Spanning Tree Protocol (MSTP)**, IEEE 802.1Q (2003)
	- Defines RSTP instances by VLAN groups

#### RSTP overview

Just like the classical STP, the purpose is to deactivates inter-switch links to remove loops in the topology by superimposing a logical tree topology.
The logic is the following:
1. The switches **elect a root switch** (distributed method)
2. They **determine the “shortest" path to the Root** (shortest = lowest cost of the link, in terms of rate)
3. They **determine their own ports status**: forwarding state or discarding state

![[rstp.png|600]]

Switches sends **Bridge Protocol Data Unit (BPDU)** every two seconds; from MAC source to `01:80:C2:00:00:00` (multicast). It carries, among others:
- The **BID (Bridge Identifier)** of the transmitter made of a configurable priority and the switch MAC address
- The identifier of the port transmitting the BPDU (Port ID), made of a configurable priority and a number
- The identifier of the Root switch: Root ID
- The cost of the path to the Root switch (depends on the links’ rate)

> [!info]- BPDU frame
> ![[bpdu.png|750]]

The **root bridge** is the switch (bridge) owning the lowest BID. So, the switch with the lowest priority value first, or if none is superior the one with the lowest MAC address.
	*Usually chosen in the centre of the topology in order to reduce the path length (done by adjusting the priority field in BID)*

On every other bridges, the closest port to the root bridge (lowest cost) is designated as the **root port**.

![[root-bridge.png|650]]
> [!caution]
> Note that in this example we chose bridge A for convenience, but it isn't the best choice, as B, C, D or E would have induced shortest paths...

The **designated bridge** is the closest switch to the root bridge (lowest cost).
	*If two bridges have the same cost, the lowest BID is chosen*

![[bridgeon.png|700]]

We can now focus on which ports to disable or not. Port can be of two states:
1. **Forwarding state**: Port is allowed to both transmit traffic and learn MAC addresses
2. **Discarding state**: Cannot learn nor transmit

All ports on the root bridge are immediately put in the forwarding state.
On other bridges, the **designated port** on a link is the port belonging to the designated bridge of the link.
	*When RSTP has converged, all designated ports are in the forwarding state*

Ports connected to a machine (so, not a switch) are called **edge ports**. They are always in the forwarding state, and aren't included in the RSTP execution as they can't create any loop. 

All ports that are not root ports nor designated ports are in the discarding state. They may be:
- An **alternate port**: In case of root port failure, it may become the new root port
- A **backup port**: In case of a failure of the designated port on a collision domain, the backup port may become the new designated port
	*Note that backup ports only exist when a switch is connected to a collision domain by more than one port (hub or bus). It's half-duplex-oriented*

![[rstp-example.png|650]]

Now, RSTP needs to update itself. Every two seconds, a designated port sends BPDUs carrying its configuration.
	*If we don't receive any frame in 6 seconds (3 Hello), we consider the link broken*

![[rstp-update.png|650]]

#### Building RSTP tree

At initialisation, all ports are **designated** and are in the **discarding state**. All switches assumes they are the root bridge.
A switch records a **priority vector** carrying (among other parameters): 
Root ID / cost to the Root bridge / BID and Port ID of the switch that sent the “best” received BPDU / its own root port

![[rstp-base.png|625]]

A switch receiving a BPDU compares it to its priority vector:
It updates the PV and modifies the receiving port’s status if the BPDU‘s parameters are « better », i.e. :
- Received Root ID < Root Id in PV
- Or received Root ID better than the one in PV and (received cost < cost in PV)
- Or received Root ID and cost better than those in PV and (sender’s BID < designated port in PV)
- Or received Root ID, cost, sender’s BID better than those in PV and (sender’s Port ID < Port ID in PV)
	 *may occur when multiple links between two switches*
- Or received Root ID, cost, sender’s BID, sender’s Port Id better than those in PV and (received Port ID < receiving Port ID)
	 *may occur when two ports of the bridge are inter-connected*

![[rstp-coverage.webm]]

#### State transition

A port that becomes an alternate port immediately goes to discarding state.
A port that becomes a root port immediately goes into the forwarding state.

On the other hand, a designated port cannot immediately go to the forwarding state because it is connected to the tree’s leaves where loops may occur.
If a designated port wishes to go to the forwarding state, it requests authorisation from the switch it is connected to:
	It sends a BPDU carrying the **proposal flag**;
	- If the neighbour sends an **agreement flag**, it goes to the forwarding state
	- Otherwise, it remains in the discarding state

When receiving a proposal flag, a switch:
- Sends the agreement flag only if the received BPDU is better than its priority vector
- Makes the receiving port the new root port
- Puts all other non-edge ports in the discarding state (prevents from loops below)
 - Sends a proposal to the switches below (downstream)
	*So, this process is repeated towards the leaves of the tree*

![[state-transition.png|675]]

#### Topology change

A topology change happens when a non-edge port goes to the forwarding state (e.g., a new link is connected).
	*A port going to discarding state does not induce a topology change*

When a switch notices the topology change, it:
- Triggers the TC timer (= 2 x hello timer)
- Sends BPDU carrying the TC flag on all non-edge designated ports and on its root port, as long as the timer does not expire
- Withdraws all MAC addresses it learnt on its non-edge designated ports and on its root port

When receiving a BPDU with TC=1, a switch:
- Withdraws all MAC addresses it learnt on its non-edge ports except on the receiving port
- Triggers the TC timer
- Sends BPDU carrying the TC flag on all non-edge designated ports and on its root port, as long as the timer does not expire
	 *The information propagates quickly in the whole network*

> [!info]
> To limit all the flooding this induces, addresses associated to the previous root port are associated to the new root port


### Flow Control

This is an optional mechanism intending to prevent buffer overflow in switches and DTE, especially when there is a huge amount of incoming traffic intended to a same output port of low bit rate.

![[flow-control.png|350]]

It allows the receiver of traffic to send **pause frames** in multicast (`01-80-C2-00-00-01`), indicating that the receiving device will stops transmission for the duration defined in the frame.


### LACP
*Link Aggregation Control Protocol; IEEE 802.1AX, IEEE 802.3ad (old)*

**Link aggregation** (or **trunking**) bundles several physical links in a logical one. If a link fails, the trunk is dynamically rebuilt.
Aggregation is done between LACP-enabled devices, which can be two switches or a switch and a host (e.g., highly solicited server).

![[lacp.png|500]]

LACP is a standardised and vendor-independent protocol which uses **LACPDU** frames periodically sent to the multicast address `01:80:C2:00:00:02`.
A device may be active (LACP unconditionally enabled) or passive (sends LACPDU only when solicited by an active device).

Equipment can now locate and merge redundant links, as long as:
- They share the same rate
- They belong to the same VLAN(s)
- They use the same transmission mode (half or full duplex)


### IEEE 802.1X

This standard provides an authentication mechanism controlling connection of equipment to an infrastructure.
- Not limited to layer 2, may involve higher layer protocols
- Not designed to MAC address learning (not standardised use case): based on proprietary extensions (e.g., MAC Authentication Bypass (MAB) by Cisco)

The model admits three entities:
1. **Supplicant**: Any device intending to connect to the Ethernet network
2. **Authenticator**: A network equipment (switch in our case) that physically connects the supplicant to the infrastructure
3. **Authentication server**: A server executing authentication protocols.
   For example RADIUS (Remote Authentication Dial In User Service, RFC 2865) and EAP (Extensible Authentication Protocol, RFC 3748), which we will see in a future lecture.

![[802.1x.png|650]]


***
## VLAN
*Virtual LAN*
### Fundamentals

We call a **broadcast domain** an area inside a network where broadcast frames are received by all equipment. They are limited by routers.

![[broadcast-domain.png|600]]

Now, two things matter in a wired LAN:
- **Performance**: Broadcast frames generally concern a few equipment (e.g., DHCP server, destination host of an ARP request).
  The scope of broadcast domains should be restricted
- **Security**: Unicast traffic inside of a LAN should be restricted to a model following the company's internal organisation.
  We would like to isolate alike machines in a dedicated environment, isolated from unrelated ones

VLANs are here to serve this purpose. They are implemented on switches, and only allow stations belonging to the same VLAN to exchange frames.
	*Stations are logically grouped: security
	Broadcast domains are restricted: performance*

![[vlan.png|550]]

There are three VLAN "levels":
- **Layer 1 - Port-based VLANs**: A station’s belonging to a VLAN is defined by assigning the port to which it is connected to a VLAN
- **Layer 2 - MAC-based VLANs**: The station’s MAC address is assigned to a VLAN
- **Layer 3 - Subnet-based VLANs; Protocol-based VLANs**: The stations’s IP address (or the subnet, or the network layer protocol) is assigned to the VLAN

The higher the level, the easier the implementation and the lower the security.

> [!info]
> Only Port-based VLANs is standardised (IEEE 802.1Q) and will be studied in this class.

Port-based VLANs can be:
- **Untagged**: using standardised 802.3 frame
- **Tagged**: Ethernet frame carries a tag indicating to which VLAN it belongs


### Port-based VLAN
*IEEE 802.1Q*

The standard defines:
- Tagged frames format and their processing by switches
- Frames’ priority levels for QoS processing in switches
- MVRP (Multiple VLAN Registration Protocol), protocol that dynamically configures inter-switch links
- MSTP (Multiple Spanning Tree Protocol), a version of RSTP adapted to VLANs
- A method to create link-level VPN in ISP networks (studied in a future course...)

VLANs are identified on a switch by a 12-bit number called **VID (VLAN Identifier)**.
	*There are a few reserved values we cannot use: 
	`0` (priority tagging), `1` (default VLAN), `4095` (implementation-specific)*
When no VLAN has been created, all ports—even unconnected ones—belong to the **default VLAN** of VID=1.
	*The default VLAN cannot be removed*

To keep track of everything, the switch owns a port/VLAN table configured by an administrator. It indicates for each port the VLAN(s) to which it belongs.
Protocols like GVRP or MVRP can be used to auto-configure ports on the inter-switch links, without requiring administrator privileges each time.

![[vlan-table.png|600]]


### Learning mechanisms

IEEE 802.1Q (annexe F) allows two operating modes:
1. **SVL mode (Shared VLAN Learning)**: MAC address learning is common to all VLANs.
	*Equivalent of having a common switching table for all VLANs*
2. **IVL mode (Independent VLAN Learning)**: MAC address learning is performed inside VLANs independently of other VLANs
	*Equivalent of having a dedicated switching table per VLAN*

> [!info]- About IVL
> IVL mode used to be required in infrastructures where several interfaces share the same MAC address, which can therefore exist on several VLANs.
> For example: In a DECnet architecture (obsolete) where all router interfaces used identical MACs

#### SVL

The switch has a single switching table, no matter the amount of VLANs.
1. The switch consults the VLAN table and determines the VLAN to which the ingress port belongs
2. It reads the destination MAC address
	- If broadcast address, it copies the frame to the output buffer of all connected ports belonging to the same VLAN as the input port
	- Otherwise it looks for it in the switching table
	  If the address is associated with a port, it reads the VLAN to which this port belongs in the VLAN table. If input and output ports belong to the same VLAN, it copies the frame to the output buffer of that port. Otherwise, the frame is discarded.
	  If the switching table does not contain the destination address, it copies the frame to the output buffer of all ports belonging to the same VLAN as the input port (flooding)
3. The switch reads the source MAC address and reads the switching table
	- If it is already recorded, it resets the timer
	- Otherwise, it associates it with the input port in the switching table

![[svl.webm]]

#### IVL

Unlike SVL, the switch owns a switching table per VLAN.
1. The switch consults the VLAN table and determines the VLAN to which the ingress port belongs
2. It reads the destination MAC address
	- If broadcast address, it copies the frame to the output buffer of all connected ports belonging to the same VLAN as the input port
	- Otherwise, it looks for it in the switching table associated with the VLAN to which the input port belongs.
	  If the address is associated with a port, it copies the frame to the output buffer of this port
	  Otherwise, it copies the frame to the output buffer of all ports belonging to the same VLAN as the input port (flooding)
3. The switch reads the source MAC address and reads the switching table associated with the VLAN to which the input port belongs
	- If it is already recorded, it resets the timer
	- Otherwise, it associates it with the input port in the switching table of this VLAN

![[ivl.webm]]


### Interconnecting switches

A first approach would be to have an inter-switch link per VLAN.

![[interswitch-link.png|550]]
> [!info]
> Low risks of congestion, but uses several ports on the switches.


A better solution would be to rely on a **tagged inter-switch link**.
To achieve this, we must use an updated version of the tagged frame format which **inserts a 4-byte field** (between source address and length/type fields) **carrying the VID**.

![[tagged-frame.png|650]]
- **TPID (Tag Protocol Identifier)**: Ethertype which identifies the type of 802.1Q frame
	`0x8100` = C-VLAN (Customer, client network of an ISP FAI)
	`0x88A8` = S-VLAN (Service or Backbone, on an ISP network)
	`0x88E7` = Backbone Service Instance Tag (on an ISP network)
- **TCI (Tag Control Information)**
- **PCP (Priority Code Point)**: Priority level to enable service classes (voice, data, vidéo...)
- **DEI (Drop Eligible Indicator)**: Marks frames eligible for dropping during congestion (e.g., best-effort traffic)

This way, a single link may be used to interconnect switches.

![[tagged-links.png|650]]
> [!info]
> The ports at the ends of the link belong to all the VLANs: when they send a frame, they add a tag to indicate which VLAN it comes from.
> It ultimately save ports on switches, but the inter-switch link is more liable to be congested.


### VLAN-aware stations

Some stations should be able to communicate with devices belonging to different VLANs (router, DNS server, DHCP server...)
Such stations must manage tags: « VLAN-aware ». They use **trunk ports** configured to carry multiple VLANs.
The support of tags by a host requires configuration on its OS and a suitable IP addressing plan (detail in lab).


### MVRP
*Multiple VLAN Registration Protocol; part of IEEE 802.1Q*

Replaces GARP VLAN Registration Protocol (GVRP; IEEE 802.1D).
It enables dynamic configuration of ports connected to inter-switch links through multicast frames called MVRPDU.
	*Purpose is to reduce manual intervention when possible*

When an edge port is assigned to a VLAN on a switch, the switch sends MVRPDU frames to ils neighbours. 
The receiving switches assign the receiving port to the VLAN identified in the frame (tagged port).

> [!example]-
> Port 1 of SW1 is manually assigned to red VLAN
> 1. Port is assigned
> 2. Port informs other ports of SW1 that it belongs to red VLAN
> 3. Ports 6 and 8 of SW1 propagate the information « SW1 needs red VLAN » → port 7 of SW2 and port 1 of SW3 are assigned to red VLAN and are tagged
> 4. Port 7 of SW2 and port 1 of SW3 informs the other ports of their respective switch that they belong to red VLAN
> ![[mvrp-1.png|625]]
> 
> Port 1 of SW2 is manually assigned to red VLAN → ports connected to the link between SW1 and SW2 belong to red VLAN
> 1. Port is assigned
> 2. Port informs other ports of SW2 that it belongs to red VLAN
> 3. Port 7 of SW2 propagates the information « SW2 needs red VLAN » → port 6 of SW1 is assigned to red VLAN and is tagged
> 4. Port 6 of SW1 informs the other ports of SW1 that it belongs to red VLAN
> 5. Port 8 of SW1 propagates the information « SW1 needs red VLAN » → port 1 of SW3 already tagged in the red VLAN, nothing changes
> ![[mvrp-2.png|625]]
> 
> Now, red VLAN traffic can be exchanged between SW1 and SW2, but not SW1 and SW3
> ![[mvrp-3.png|625]]


***
## Common attacks

### MAC flooding, MAC overflow

An attacker sends numerous frames carrying random MAC source addresses.
	*Can be tested with the `macof` (for "MAC overflow") command included in the `dsniff` package*

The learning process overflows the switching table, which means that switches:
- Cannot learn « official » stations’ addresses anymore
- Must flood frames intended to stations

This allows the attacker to receive—and potentially sniff—all traffic.

> [!success] Solutions
> - Restriction of the number of learnt addresses per port
> - Listing of the addresses a port is allowed to record
> - Using [[01 - Wired Networks#IEEE 802.1X|802.1X authentication]]

### Double tagging attack

Leverages poor configuration of the inter-switch link (like not tagging unconnected ports) to enables a VLAN-aware attacker to send trafic to a station which either belongs to a different VLAN, or is connected to another switch.

![[double-tagging.png|675]]

> [!success] Solutions (and general guidelines to configure secure VLANs)
> - Set VLANs on the switches as restrictively as possible. On inter-switch links, the traffic of a VLAN is only allowed when machines of this VLAN are connected on both sides of the link.
> - When a station is disconnected, the port is assigned to the Default VLAN again
> - Start by allowing the minimum of transactions. Communications are authorised only when the need arises
> - Tags are required for all VLANs on the inter-switch links (to avoid double tagging)

